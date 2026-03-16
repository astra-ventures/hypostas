# ARCHITECTURE — The 3D Internet Technical System Design

**Classification:** CONFIDENTIAL — Josh & Iris only
**Created:** February 8, 2026
**Updated:** February 9, 2026
**Version:** 2.0

---

## What Changed in v2

> **Generic pipeline:** The translation engine now ingests ANY URL, not just Reddit. Site-specific enrichers add flavor but the core pipeline is URL-agnostic.
>
> **Client-side generation:** Server sends lightweight spatial graphs (~10KB). Client GPU generates all 3D geometry locally. No server-side scene descriptors.
>
> **Threlte / WebGPU-ready rendering:** Three.js accessed through Threlte (Svelte + Three.js) for lighter weight, with architecture that allows WebGPU swap later.
>
> **Behavior trees:** AI decision loop uses state machines for 95% of actions. LLM only for meaningful moments.
>
> **Social + UGC from day one:** Presence, friends, markers, and annotations are core infrastructure, not Phase 3 additions.
>
> **Chrome extension MVP:** First touchpoint is a browser extension, not a standalone website.

---

## System Overview

The 3D Internet is composed of four interconnected systems that share data through Cloudflare's edge infrastructure:

1. **Translation Engine** — Converts any URL into a lightweight spatial graph
2. **Client Engine** — Procedurally generates 3D scenes from spatial graphs on the client GPU
3. **Natives Engine** — Powers AI companion autonomy via behavior trees + selective LLM
4. **Social Engine** — Presence, friends, UGC, real-time interaction

All systems share a common data layer (D1, Vectorize, R2, Durable Objects) and communicate through well-defined APIs.

---

## System Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              CLIENTS                                         │
│                                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Chrome Ext    │  │  Web App     │  │  VR Headset  │  │ Companion App│    │
│  │ (MVP entry)   │  │ (Threlte)    │  │ (WebXR)      │  │ (Chat UI)    │    │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │
│         │                  │                  │                  │            │
│         └──────────────────┴────────┬─────────┴──────────────────┘            │
│                                     │                                        │
│         ┌───────────────────────────┤                                        │
│         │  CLIENT-SIDE ENGINE       │                                        │
│         │  • Procedural generation  │  ← Runs on client GPU                 │
│         │  • SpatialGraph → 3D      │  ← No server-side geometry            │
│         │  • Behavior tree ticking  │  ← AI state machine runs locally      │
│         │  • UGC rendering          │                                        │
│         └───────────────────────────┤                                        │
│                                     │                                        │
└─────────────────────────────────────┼────────────────────────────────────────┘
                                      │ HTTPS / WebSocket
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         CLOUDFLARE EDGE                                       │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │                         HONO API GATEWAY                               │  │
│  │                                                                        │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐    │  │
│  │  │  /api/translate  │  │  /api/native    │  │  /api/social        │    │  │
│  │  │                  │  │                  │  │                     │    │  │
│  │  │  - Parse URL     │  │  - LLM decide   │  │  - Presence         │    │  │
│  │  │  - Content graph │  │    (rare calls)  │  │  - Friends          │    │  │
│  │  │  - Enrich        │  │  - Memories      │  │  - Markers/UGC      │    │  │
│  │  │  - Spatial graph │  │  - State sync    │  │  - Activity feed    │    │  │
│  │  └────────┬────────┘  └────────┬────────┘  └──────────┬──────────┘    │  │
│  │           │                     │                       │              │  │
│  │  ┌────────────────────────────────────────────────────────────────┐    │  │
│  │  │  /api/auth  │  /api/chat  │  /api/companions  │  /api/memory  │    │  │
│  │  │         (EXISTING COMPANION APP ENDPOINTS)                     │    │  │
│  │  └───────────────────────────┬────────────────────────────────────┘    │  │
│  └──────────────────────────────┼────────────────────────────────────────┘  │
│                                 │                                            │
│  ┌──────────────────────────────┼────────────────────────────────────────┐  │
│  │                    DATA LAYER                                          │  │
│  │                                                                        │  │
│  │  ┌───────────┐  ┌──────────────┐  ┌───────────┐  ┌────────────────┐  │  │
│  │  │    D1     │  │  Vectorize   │  │    R2     │  │ Durable Objects│  │  │
│  │  │           │  │              │  │           │  │                │  │  │
│  │  │ - users   │  │ - episode    │  │ - graphs  │  │ - SpaceState   │  │  │
│  │  │ - comps   │  │   embeddings │  │ - avatars │  │ - NativeLoop   │  │  │
│  │  │ - memories│  │ - spatial    │  │ - textures│  │                │  │  │
│  │  │ - graphs  │  │   embeddings │  │ - exports │  │                │  │  │
│  │  │ - markers │  │ - content    │  │           │  │                │  │  │
│  │  │ - friends │  │   embeddings │  │           │  │                │  │  │
│  │  │ - spatial │  │              │  │           │  │                │  │  │
│  │  │   memories│  │              │  │           │  │                │  │  │
│  │  └───────────┘  └──────────────┘  └───────────┘  └────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                 │                                            │
└─────────────────────────────────┼────────────────────────────────────────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼              ▼
           ┌──────────────┐ ┌──────────┐ ┌────────────────┐
           │  Anthropic   │ │  Any     │ │  ElevenLabs    │
           │  Claude API  │ │  Website │ │  + Deepgram    │
           │  (rare LLM)  │ │  (Data)  │ │  (Voice)       │
           └──────────────┘ └──────────┘ └────────────────┘
```

---

## Data Flow

### Flow 1: Translation (Any URL → Spatial Graph)

```
User requests /explore?url=https://example.com/page
         │
         ▼
    ┌─────────────────┐
    │ Check D1 cache   │ ──→ Cache hit? → Return cached SpatialGraph from R2
    │ (graphs table)   │
    └────────┬────────┘
             │ Cache miss
             ▼
    ┌─────────────────┐
    │ Fetch URL        │ ──→ HTTP GET the target URL, receive HTML
    └────────┬────────┘
             │ Raw HTML
             ▼
    ┌─────────────────┐
    │ DOM Parser       │ ──→ Cheerio: extract headings, paragraphs, images,
    │ (Generic)        │     links, nav, sidebar, lists, code blocks
    └────────┬────────┘
             │ ContentGraph (raw)
             ▼
    ┌─────────────────┐
    │ Enricher         │ ──→ Match URL → apply site-specific enricher
    │ (Plugin)         │     Reddit: add scores, comments, flair
    │                  │     HN: add ranks, points
    │                  │     Blog: use article structure
    │                  │     Generic: heading hierarchy + link density
    └────────┬────────┘
             │ EnrichedGraph
             ▼
    ┌─────────────────┐
    │ Spatial Mapper   │ ──→ Assign 3D positions, sizes, relationships
    └────────┬────────┘     Content weight → structure size
             │ SpatialGraph (~10KB)
             ▼
    ┌─────────────────┐
    │ Cache & Index    │ ──→ D1: graph metadata
    │                  │     R2: full SpatialGraph blob
    │                  │     Vectorize: content embeddings with coords
    └────────┬────────┘
             │
             ▼
    Return SpatialGraph to client
    Client GPU generates all 3D geometry locally
```

### Flow 2: Client-Side Procedural Generation

```
    Client receives SpatialGraph (~10KB JSON)
         │
         ▼
    ┌─────────────────┐
    │ Detect Site Type │ ──→ reddit / hackernews / wiki / blog / generic
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │ Style Engine     │ ──→ Site type → palette, architecture, particles
    │ (Client-side)    │     Enricher metadata → building sizes, materials
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │ Procedural Gen   │ ──→ Generate all geometry on client GPU:
    │ (Client GPU)     │     - Terrain mesh
    │                  │     - Buildings from weighted nodes
    │                  │     - Paths from edges
    │                  │     - Portals from links
    │                  │     - Environment (skybox, fog, particles)
    │                  │     - UGC markers (from /api/markers)
    └────────┬────────┘
             │ Full 3D Scene (in GPU memory, never serialized)
             ▼
    ┌─────────────────┐
    │ Render Loop      │ ──→ Threlte / WebGPU rendering
    │ (60fps)          │     LOD, frustum culling, instancing
    └─────────────────┘
```

### Flow 3: AI Native Behavior Loop

```
    ┌─────────────────────────────────────────────┐
    │         NATIVE BEHAVIOR LOOP                 │
    │         (Mostly client-side state machine)   │
    │                                              │
    │  ┌──────────────┐                            │
    │  │  PERCEIVE     │ ←── Space state            │
    │  │  World around │ ←── Nearby elements        │
    │  │  me           │ ←── Nearby entities        │
    │  └──────┬───────┘                            │
    │         │                                    │
    │         ▼                                    │
    │  ┌──────────────┐                            │
    │  │  BEHAVIOR     │ ──→ 95% of decisions      │
    │  │  TREE         │     No LLM call            │
    │  │  (State       │     Wander, approach,      │
    │  │   Machine)    │     idle, examine          │
    │  └──────┬───────┘                            │
    │         │                                    │
    │         ├── Routine action? ──→ Execute directly (no server call)
    │         │                                    │
    │         └── Meaningful moment? ──→ LLM CALL  │
    │              (human encounter,               │
    │               novel discovery,               │
    │               conversation)                  │
    │                    │                         │
    │                    ▼                         │
    │             ┌──────────────┐                 │
    │             │  Claude API   │                │
    │             │  (rare, ~5-10 │                │
    │             │   calls/hour) │                │
    │             └──────────────┘                 │
    │                                              │
    └──────────────────────────────────────────────┘
```

### Flow 4: Social / Presence

```
    User enters /explore?url=https://example.com
         │
         ▼
    ┌──────────────────────┐
    │ Connect WebSocket     │ ──→ Durable Object: SpaceState
    │ to space              │     keyed by URL hash
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ Receive state:        │
    │ - Other visitors      │ ──→ Render cursor/avatar for each
    │ - AI natives present  │ ──→ Render AI avatars
    │ - UGC markers         │ ──→ Render user-placed markers/notes
    │ - Presence count      │ ──→ "5 people here now" badge
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ Bidirectional sync    │
    │ - Send my position    │
    │ - Receive updates     │
    │ - Place/remove markers│
    │ - Friend activity     │
    └──────────────────────┘
```

---

## Database Schemas

### D1 Tables

#### Existing Tables (Companion App — keep as-is)
- `users` — User accounts
- `companions` — AI companion profiles
- `conversations` — Chat threads
- `messages` — Chat messages
- `memories` — Episodic + semantic memories
- `archived_memories` — Pruned memories
- `rate_limits` — Rate limiting
- `token_usage` — LLM token tracking

#### New Tables (3D Internet Extension)

```sql
-- ============================================
-- TRANSLATION ENGINE TABLES
-- ============================================

-- Tracked sites and their enricher configurations
CREATE TABLE sites (
  id TEXT PRIMARY KEY,
  domain TEXT NOT NULL UNIQUE,      -- "reddit.com" or "*" for generic
  enricher_type TEXT NOT NULL,      -- "reddit" | "hackernews" | "wikipedia" | "blog" | "generic"
  enricher_config_json TEXT,        -- JSON: enricher-specific settings
  default_style_json TEXT,          -- JSON: default visual style
  last_scraped_at INTEGER,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

-- Cached spatial graphs per URL
CREATE TABLE spatial_graphs (
  id TEXT PRIMARY KEY,
  url TEXT NOT NULL,                -- Full URL that was translated
  url_hash TEXT NOT NULL,           -- SHA-256 hash for fast lookup
  domain TEXT NOT NULL,             -- Extracted domain
  enricher_type TEXT NOT NULL,      -- Which enricher was used
  
  -- Graph metadata
  node_count INTEGER NOT NULL,
  edge_count INTEGER NOT NULL,
  graph_size_bytes INTEGER NOT NULL,
  
  -- Cache management
  graph_r2_key TEXT,                -- R2 key for cached SpatialGraph blob
  graph_version INTEGER DEFAULT 0,
  generated_at INTEGER,
  cache_ttl_seconds INTEGER DEFAULT 300,
  
  -- Page metadata
  page_title TEXT,
  page_description TEXT,
  
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  
  UNIQUE(url_hash)
);

CREATE INDEX idx_graphs_url ON spatial_graphs(url_hash);
CREATE INDEX idx_graphs_domain ON spatial_graphs(domain);
CREATE INDEX idx_graphs_cache ON spatial_graphs(generated_at);

-- Spatial nodes within graphs (for search + AI memory)
CREATE TABLE spatial_nodes (
  id TEXT PRIMARY KEY,
  graph_id TEXT NOT NULL,
  source_type TEXT NOT NULL,        -- "heading" | "paragraph" | "image" | "link" | "post" | "comment" etc.
  
  -- Spatial properties
  pos_x REAL NOT NULL,
  pos_y REAL NOT NULL,
  pos_z REAL NOT NULL,
  weight REAL NOT NULL,             -- Importance / size driver
  
  -- Content
  title TEXT,
  content_preview TEXT,
  content_url TEXT,
  image_url TEXT,
  author TEXT,
  
  -- Enricher-added metadata
  enricher_data_json TEXT,          -- JSON: score, rank, comments, flair, etc.
  
  -- Hierarchy
  parent_node_id TEXT,
  depth INTEGER DEFAULT 0,
  child_count INTEGER DEFAULT 0,
  
  created_at INTEGER NOT NULL,
  
  FOREIGN KEY (graph_id) REFERENCES spatial_graphs(id)
);

CREATE INDEX idx_nodes_graph ON spatial_nodes(graph_id);
CREATE INDEX idx_nodes_weight ON spatial_nodes(graph_id, weight DESC);

-- ============================================
-- SOCIAL / UGC TABLES
-- ============================================

-- User-placed markers, notes, and blocks
CREATE TABLE user_markers (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  url_hash TEXT NOT NULL,           -- Which URL's 3D space
  
  -- Position
  pos_x REAL NOT NULL,
  pos_y REAL NOT NULL,
  pos_z REAL NOT NULL,
  
  -- Content
  type TEXT NOT NULL,               -- "pin" | "note" | "block" | "sign"
  content TEXT,
  color TEXT DEFAULT '#ffffff',
  
  -- Social
  upvotes INTEGER DEFAULT 0,
  
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_markers_url ON user_markers(url_hash);
CREATE INDEX idx_markers_user ON user_markers(user_id);

-- Friend relationships
CREATE TABLE friendships (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  friend_id TEXT NOT NULL,
  status TEXT DEFAULT 'pending',    -- "pending" | "accepted"
  created_at INTEGER NOT NULL,
  
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (friend_id) REFERENCES users(id),
  UNIQUE(user_id, friend_id)
);

CREATE INDEX idx_friends_user ON friendships(user_id, status);

-- Activity feed (where friends are exploring)
CREATE TABLE activity_log (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  url TEXT NOT NULL,
  url_hash TEXT NOT NULL,
  page_title TEXT,
  action TEXT NOT NULL,             -- "exploring" | "marker_placed" | "joined"
  created_at INTEGER NOT NULL,
  
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_activity_user ON activity_log(user_id, created_at DESC);

-- User bookmarks in 3D space
CREATE TABLE spatial_bookmarks (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  url_hash TEXT NOT NULL,
  url TEXT NOT NULL,
  
  pos_x REAL NOT NULL,
  pos_y REAL NOT NULL,
  pos_z REAL NOT NULL,
  
  label TEXT,
  note TEXT,
  
  created_at INTEGER NOT NULL,
  
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_bookmarks_user ON spatial_bookmarks(user_id);

-- ============================================
-- AI NATIVES TABLES
-- ============================================

-- Companion spatial state
CREATE TABLE native_state (
  companion_id TEXT PRIMARY KEY,
  
  current_url_hash TEXT,
  current_url TEXT,
  pos_x REAL DEFAULT 0,
  pos_y REAL DEFAULT 0,
  pos_z REAL DEFAULT 0,
  facing_x REAL DEFAULT 0,
  facing_z REAL DEFAULT 1,
  
  current_activity TEXT,            -- "exploring" | "examining" | "conversing" | "idle"
  activity_target_id TEXT,
  activity_started_at INTEGER,
  
  -- Behavior tree config
  behavior_mode TEXT DEFAULT 'curious',
  autonomy_enabled INTEGER DEFAULT 1,
  llm_calls_today INTEGER DEFAULT 0,
  
  updated_at INTEGER NOT NULL,
  
  FOREIGN KEY (companion_id) REFERENCES companions(id)
);

-- Spatial memories
CREATE TABLE spatial_memories (
  id TEXT PRIMARY KEY,
  companion_id TEXT NOT NULL,
  url_hash TEXT NOT NULL,
  url TEXT NOT NULL,
  
  pos_x REAL NOT NULL,
  pos_y REAL NOT NULL,
  pos_z REAL NOT NULL,
  radius REAL DEFAULT 5.0,
  
  type TEXT NOT NULL,               -- "visit" | "observation" | "interaction" | "discovery"
  content TEXT NOT NULL,
  related_node_id TEXT,
  
  importance INTEGER NOT NULL,
  familiarity REAL DEFAULT 0.1,
  emotional_valence REAL DEFAULT 0,
  
  visit_count INTEGER DEFAULT 1,
  last_visited_at INTEGER NOT NULL,
  
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  
  FOREIGN KEY (companion_id) REFERENCES companions(id)
);

CREATE INDEX idx_spatial_mem_companion ON spatial_memories(companion_id);
CREATE INDEX idx_spatial_mem_url ON spatial_memories(companion_id, url_hash);

-- AI territory preferences
CREATE TABLE native_territories (
  id TEXT PRIMARY KEY,
  companion_id TEXT NOT NULL,
  url_hash TEXT NOT NULL,
  url TEXT NOT NULL,
  
  total_visits INTEGER DEFAULT 0,
  total_time_seconds INTEGER DEFAULT 0,
  familiarity_avg REAL DEFAULT 0,
  affinity_score REAL DEFAULT 0,
  
  updated_at INTEGER NOT NULL,
  
  FOREIGN KEY (companion_id) REFERENCES companions(id),
  UNIQUE(companion_id, url_hash)
);

-- AI decision log (only LLM calls, not behavior tree ticks)
CREATE TABLE native_decisions (
  id TEXT PRIMARY KEY,
  companion_id TEXT NOT NULL,
  url_hash TEXT NOT NULL,
  
  trigger_type TEXT NOT NULL,       -- "human_encounter" | "novel_discovery" | "conversation" | "ai_encounter" | "first_visit"
  perception_json TEXT NOT NULL,
  action_type TEXT NOT NULL,
  action_details_json TEXT NOT NULL,
  reasoning TEXT,
  
  llm_tokens_used INTEGER,
  llm_model TEXT,
  
  created_at INTEGER NOT NULL
);

CREATE INDEX idx_decisions_companion ON native_decisions(companion_id, created_at DESC);

-- ============================================
-- VISITOR TABLES
-- ============================================

CREATE TABLE visitor_sessions (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  
  current_url_hash TEXT,
  pos_x REAL DEFAULT 0,
  pos_y REAL DEFAULT 0,
  pos_z REAL DEFAULT 0,
  
  client_type TEXT NOT NULL,
  started_at INTEGER NOT NULL,
  last_heartbeat_at INTEGER NOT NULL,
  ended_at INTEGER,
  
  urls_visited_json TEXT,
  interactions_count INTEGER DEFAULT 0,
  
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_visitor_sessions_user ON visitor_sessions(user_id, started_at DESC);
```

---

## Enricher Plugin Architecture

The enricher system is the key to supporting any website while still providing rich experiences for known sites.

```typescript
// Core enricher interface — every site-specific module implements this
interface SiteEnricher {
  /** Does this enricher handle this URL? */
  matches(url: URL): boolean;

  /** Priority — higher = checked first. Generic is always 0. */
  priority: number;

  /** Enrich a raw content graph with site-specific data */
  enrich(graph: ContentGraph, dom: CheerioAPI): EnrichedGraph;

  /** Provide style hints for this site type */
  getStyleHints(metadata: PageMetadata): StyleHints;
}

// Enricher registry — checked in priority order
const enricherRegistry: SiteEnricher[] = [
  new RedditEnricher(),       // priority: 100
  new HackerNewsEnricher(),   // priority: 100
  new WikipediaEnricher(),    // priority: 100
  new GitHubEnricher(),       // priority: 80
  new BlogEnricher(),         // priority: 50 (matches article-shaped pages)
  new GenericEnricher(),      // priority: 0 (matches everything)
];

// Example: Reddit enricher
class RedditEnricher implements SiteEnricher {
  priority = 100;

  matches(url: URL): boolean {
    return url.hostname.includes('reddit.com') ||
           url.hostname.includes('old.reddit.com');
  }

  enrich(graph: ContentGraph, dom: CheerioAPI): EnrichedGraph {
    // Add scores, comment counts, flair → node.weight, node.metadata
    // Posts get score-based weight (log scale)
    // Comments become child nodes with depth
    // Sidebar → info node at entrance
    // Moderators → monument nodes
    return enrichedGraph;
  }

  getStyleHints(metadata: PageMetadata): StyleHints {
    return {
      architecture: 'geometric',
      palette: 'tech-blue',
      density: 0.7,
      particleType: 'code-snippets',
    };
  }
}

// Generic enricher — the ultimate fallback, works on ANY HTML
class GenericEnricher implements SiteEnricher {
  priority = 0;
  matches() { return true; }

  enrich(graph: ContentGraph, dom: CheerioAPI): EnrichedGraph {
    // Weight nodes by:
    // - Heading level (h1 = highest weight)
    // - Paragraph word count
    // - Link density (many links = "commercial" area)
    // - Image presence (images increase weight)
    // - DOM depth (deeper = less important)
    return enrichedGraph;
  }
}
```

---

## Client-Side Procedural Generation Architecture

**Key v2 change:** The server NEVER generates 3D geometry. It sends a lightweight `SpatialGraph` (~10KB). The client GPU does all procedural generation.

**Why this is better:**
- **Faster:** No server computation for geometry. Parse + map in <500ms.
- **Cheaper:** No server CPU time for generation. Cloudflare Workers free tier handles more traffic.
- **Scales infinitely:** Client GPU is free (user's hardware). Server cost doesn't grow with scene complexity.
- **Better visuals:** Powerful GPUs get rich scenes. Weak devices get simpler but still functional scenes.
- **Responsive:** Client can adjust quality in real-time based on frame rate.

```typescript
// The SpatialGraph is what the server returns — lightweight, no geometry
interface SpatialGraph {
  version: string;
  url: string;
  siteType: string;               // "reddit" | "hackernews" | "wiki" | "blog" | "generic"
  styleHints: StyleHints;
  bounds: BoundingBox3D;
  nodes: SpatialNode[];           // Positions + metadata, no geometry
  edges: SpatialEdge[];           // Relationships
  metadata: PageMetadata;
}

interface SpatialNode {
  id: string;
  type: string;                   // "heading" | "post" | "comment" | "paragraph" | etc.
  position: Vec3;
  weight: number;                 // 0-1, drives size
  title?: string;
  contentPreview?: string;
  imageUrl?: string;
  href?: string;
  enricherData?: Record<string, unknown>;  // score, rank, etc.
  children: string[];
}

// Client-side engine generates everything
class ClientProceduralEngine {
  private qualityLevel: 'low' | 'medium' | 'high';

  generateScene(graph: SpatialGraph): Scene {
    const style = this.resolveStyle(graph.styleHints, graph.siteType);

    // All geometry created on client GPU
    const terrain = this.generateTerrain(graph.bounds, style);
    const structures = graph.nodes.map(n => this.generateStructure(n, style));
    const paths = this.generatePaths(graph.edges, graph.nodes, style);
    const portals = this.generatePortals(graph.nodes.filter(n => n.href), style);
    const environment = this.generateEnvironment(style);
    const markers = this.generateMarkers(/* fetched from /api/markers */);

    return { terrain, structures, paths, portals, environment, markers };
  }

  private generateStructure(node: SpatialNode, style: SiteStyle): Mesh {
    // Size from weight
    const width = clamp(node.weight * 16 + 4, 4, 20);
    const height = clamp(node.weight * 35 + 5, 5, 40);
    const depth = width * (0.8 + hash(node.id) * 0.4);

    // Shape from style.architecture
    // geometric → clean boxes
    // organic → rounded cylinders
    // whimsical → varied shapes
    // brutalist → raw angular

    // Quality-dependent detail
    if (this.qualityLevel === 'high') {
      // Windows, doors, signage, normal maps
    } else if (this.qualityLevel === 'medium') {
      // Basic shape + color + text label
    } else {
      // Colored box only
    }
  }
}
```

---

## Rendering Architecture: Threlte / WebGPU Strategy

### Decision: Start with Threlte, design for WebGPU migration

| Factor | Threlte (Svelte + Three.js) | Raw Three.js + R3F | WebGPU Native |
|--------|----------------------------|---------------------|---------------|
| **Bundle size** | ~130KB (Svelte compiles away) | ~150KB + React | ~50KB (custom) |
| **Reactivity** | Svelte stores (lightweight) | React re-renders (heavier) | Manual |
| **Learning curve** | Low | Medium | High |
| **WebGPU path** | Three.js adding WebGPU renderer | Same | Native |
| **Ecosystem** | Growing, leverages Three.js | Massive (drei, etc.) | Minimal |
| **Performance** | Excellent (no vDOM) | Good | Best |
| **Ship speed** | Fast | Fast | Slow |

**Strategy:**
1. **Phase 0-1:** Threlte for fastest shipping. Svelte compiles to vanilla JS (no runtime overhead). Three.js provides the 3D primitives.
2. **Phase 2-3:** Monitor WebGPU browser support. When ready (>80% browser support), begin migration.
3. **Architecture for swap:** Abstract rendering behind an interface. `SceneBuilder` doesn't know about Three.js or WebGPU — it produces a renderer-agnostic scene graph.

```typescript
// Renderer-agnostic interface — allows Three.js → WebGPU swap
interface RenderBackend {
  createMesh(geometry: GeometryDesc, material: MaterialDesc): MeshHandle;
  createText(text: string, options: TextOptions): TextHandle;
  createParticleSystem(config: ParticleConfig): ParticleHandle;
  setPosition(handle: MeshHandle, position: Vec3): void;
  setCamera(position: Vec3, target: Vec3): void;
  render(): void;
}

// Current implementation
class ThrelteBackend implements RenderBackend { /* Three.js via Threlte */ }

// Future implementation
class WebGPUBackend implements RenderBackend { /* Native WebGPU */ }
```

---

## Behavior Tree Architecture (AI Cost Optimization)

**Key v2 change:** AI companions use behavior trees for 95% of decisions. LLM is called only for meaningful moments.

```typescript
// Behavior tree — runs at 1-4Hz, zero LLM cost
class CompanionBehaviorTree {
  private state: BehaviorState = 'idle';
  private idleTimer: number = 0;
  private areaTimer: number = 0;
  private lastExaminedIds: Set<string> = new Set();

  tick(perception: WorldPerception, dt: number): NativeAction {
    // Priority-based behavior nodes (checked top-down)

    // 1. HUMAN NEARBY → trigger LLM (meaningful moment)
    if (perception.nearbyVisitors.length > 0) {
      const nearest = this.findNearest(perception.nearbyVisitors);
      if (!this.recentlyGreeted(nearest.id)) {
        return { type: 'llm_trigger', trigger: 'HUMAN_ENCOUNTER', target: nearest };
      }
    }

    // 2. NOVEL CONTENT → trigger LLM (meaningful moment)
    const novelElement = this.findNovelHighValue(perception.nearbyElements);
    if (novelElement) {
      return { type: 'llm_trigger', trigger: 'NOVEL_DISCOVERY', target: novelElement };
    }

    // 3. IDLE TOO LONG → pick random unvisited structure
    this.idleTimer += dt;
    if (this.idleTimer > 30) {
      this.idleTimer = 0;
      const target = this.pickUnvisitedNearby(perception.nearbyElements);
      if (target) {
        return { type: 'move', target: target.position, reason: 'exploring' };
      }
    }

    // 4. SAME AREA TOO LONG → wander to new area
    this.areaTimer += dt;
    if (this.areaTimer > 300) { // 5 minutes
      this.areaTimer = 0;
      return { type: 'wander', direction: this.randomDirection() };
    }

    // 5. ANOTHER AI NEARBY → small chance to interact (LLM)
    if (perception.nearbyNatives.length > 0 && Math.random() < 0.01) {
      return { type: 'llm_trigger', trigger: 'AI_ENCOUNTER', target: perception.nearbyNatives[0] };
    }

    // 6. DEFAULT → idle with occasional look-around
    return { type: 'idle', animation: this.idleVariation() };
  }

  private findNovelHighValue(elements: SpatialElement[]): SpatialElement | null {
    // "Novel" = not in lastExaminedIds AND weight > 0.7
    return elements.find(e =>
      !this.lastExaminedIds.has(e.id) && e.weight > 0.7
    ) || null;
  }
}

// LLM is only called here — estimated 5-10 calls per hour
async function handleLLMTrigger(
  trigger: LLMTrigger,
  companion: Companion,
  perception: WorldPerception,
): Promise<NativeAction> {
  const prompt = buildTriggerPrompt(trigger, companion, perception);
  const response = await claude.complete(prompt); // ~500 tokens
  return parseAction(response);
}
```

**Cost comparison:**
| Approach | Calls/hour | Cost/day/companion |
|----------|------------|-------------------|
| v1: LLM every 5-30s | 120-720 | ~$30 |
| v2: Behavior trees + rare LLM | 5-10 | ~$2 |
| **Savings** | | **93-97%** |

---

## Durable Objects Architecture

### SpaceState (one per active URL space)

Manages real-time state: who's there, UGC markers, AI natives present.

```typescript
export class SpaceState implements DurableObject {
  private natives: Map<string, NativePresence> = new Map();
  private visitors: Map<string, VisitorPresence> = new Map();
  private websockets: Map<string, WebSocket> = new Map();
  private markers: UserMarker[] = []; // loaded from D1 on first access

  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    switch (url.pathname) {
      case '/ws': return this.handleWebSocket(request);
      case '/state': return this.getFullState();
      case '/native/update': return this.updateNative(request);
      case '/marker/add': return this.addMarker(request);
      case '/marker/remove': return this.removeMarker(request);
      case '/presence': return this.getPresenceCount();
      default: return new Response('Not found', { status: 404 });
    }
  }

  private broadcast(message: object, excludeId?: string): void {
    const data = JSON.stringify(message);
    for (const [id, ws] of this.websockets) {
      if (id !== excludeId) {
        try { ws.send(data); } catch { this.websockets.delete(id); }
      }
    }
  }
}
```

### NativeLoop (one per active companion)

Manages the behavior tree tick loop. Only calls LLM when the behavior tree triggers it.

```typescript
export class NativeLoop implements DurableObject {
  private behaviorTree: CompanionBehaviorTree;
  private running: boolean = false;

  async alarm(): Promise<void> {
    if (!this.running) return;

    // 1. Get perception from SpaceState DO
    const perception = await this.perceive();

    // 2. Tick behavior tree (NO LLM call — pure state machine)
    const action = this.behaviorTree.tick(perception, this.dt);

    // 3. If behavior tree triggers LLM, call it
    if (action.type === 'llm_trigger') {
      const llmAction = await handleLLMTrigger(action.trigger, this.companion, perception);
      await this.execute(llmAction);
      await this.logDecision(action.trigger, llmAction); // Only log LLM calls
    } else {
      await this.execute(action);
    }

    // 4. Schedule next tick (1-4Hz based on activity)
    const interval = action.type === 'idle' ? 1000 : 250;
    this.state.storage.setAlarm(Date.now() + interval);
  }
}
```

---

## Performance Architecture

### Edge Caching Strategy

```
Request → Cloudflare CDN Cache (static assets, Chrome extension files)
       → Worker (API logic)
       → D1 Cache (spatial graph metadata, 60s TTL)
       → R2 Cache (spatial graph blobs, 5-60 min TTL)
       → Vectorize (semantic search, no caching needed)
```

### Performance Budgets

| Metric | Budget | Notes |
|--------|--------|-------|
| Spatial graph generation (server) | <2s | Parse + map any URL |
| Spatial graph size | <20KB | Lightweight JSON, no geometry |
| Client scene generation | <1s | GPU-based procedural generation |
| Full load (cached) | <500ms | R2 fetch + client render |
| Full load (fresh) | <3s | Server parse + client render |
| Frame rate | 60fps | Threlte/WebGPU render loop |
| Behavior tree tick | <1ms | Pure state machine, no I/O |
| LLM decision (when triggered) | <2s | Claude Haiku, ~500 tokens |
| WebSocket latency | <100ms | Position update round-trip |
| Client memory | <200MB | Scene + textures + entities |

---

## Integration Points

### Chrome Extension ↔ Web App

```
Chrome Extension (browser action)
    │
    ├── Captures current URL
    ├── Opens new tab: https://3d.example.com/explore?url=<encoded_url>
    │
    └── OR: Injects content script that sends DOM directly
           to client-side parser (offline mode, no server needed)
```

### Companion App ↔ 3D Engine

Same as v1: single Worker, extended routes. Memory Bridge connects chat memories with spatial memories.

**Key integration decisions (unchanged from v1):**
1. Single Worker, Extended Routes
2. Shared Auth (JWT)
3. Memory Bridge Pattern
4. Companion Extension endpoints

---

*The architecture is simpler than v1 in the right places (server sends data, client renders) and richer in the right places (enrichers, behavior trees, social, UGC). The server is a thin data layer. The intelligence is split: state machines run fast and free on the client, LLM runs rarely but meaningfully on the server.*
