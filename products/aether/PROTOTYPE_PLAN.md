# PROTOTYPE PLAN — Week 1-4 Detailed Build Spec

**Classification:** CONFIDENTIAL — Josh & Iris only
**Created:** February 8, 2026
**Version:** 1.0

---

## Objective

Build a working proof-of-concept: **any webpage in 3D**. A user enters any URL and navigates a generated 3D space representing that page's content. Headings become districts, paragraphs become buildings, links become portals. Any site, any page, out of the box.

Reddit is used as an *internal test case* for the enricher plugin system — not the product. The product works on everything.

This is Phase 1 of the Master Roadmap, weeks 1-4 specifically (the core pipeline). Weeks 5-6 (polish + API hardening) follow naturally once this works.

---

## Technology Stack

### Server (Cloudflare Workers)

| Library | Version | Purpose |
|---------|---------|---------|
| `hono` | `^4.4.0` | HTTP framework (already used in companion app) |
| `cheerio` | `^1.0.0` | HTML parsing for Reddit scraping |
| `uuid` | `^9.0.0` | ID generation (or `crypto.randomUUID()`) |
| `zod` | `^3.23.0` | Request/response validation |
| `@cloudflare/workers-types` | `^4.20240208.0` | TypeScript types for Workers APIs |

### Client (Next.js)

| Library | Version | Purpose |
|---------|---------|---------|
| `next` | `^14.2.0` | React framework with App Router |
| `react` | `^18.3.0` | UI library |
| `three` | `^0.163.0` | 3D rendering engine |
| `@react-three/fiber` | `^8.16.0` | React bindings for Three.js |
| `@react-three/drei` | `^9.105.0` | Three.js helpers (controls, loaders, text) |
| `@tanstack/react-query` | `^5.40.0` | Data fetching + caching |
| `troika-three-text` | `^0.49.0` | SDF text rendering in 3D |
| `tailwindcss` | `^3.4.0` | UI overlay styling |
| `zustand` | `^4.5.0` | Client-side state management |

### Infrastructure (Cloudflare)

| Service | Purpose |
|---------|---------|
| Workers | API server, scraping, spatial mapping |
| D1 | Structured data (zones, elements, paths) |
| Vectorize | Semantic + spatial search |
| R2 | Scene descriptor blobs, textures, cached assets |
| Pages | Static client hosting |

---

## Project File Structure

```
3d-internet-engine/
├── package.json
├── tsconfig.json
├── wrangler.toml
├── vitest.config.ts
│
├── src/                              # Worker source (server-side)
│   ├── index.ts                      # Hono app entry
│   │
│   ├── routes/
│   │   ├── translate.ts              # /api/translate/* endpoints
│   │   ├── spatial.ts                # /api/spatial/* endpoints
│   │   └── health.ts                 # /api/health
│   │
│   ├── scrapers/
│   │   ├── base.ts                   # BaseScraper abstract class
│   │   ├── reddit.ts                 # RedditScraper implementation
│   │   └── types.ts                  # Scraper output types
│   │
│   ├── pipeline/
│   │   ├── normalizer.ts             # Raw data → SiteGraph
│   │   ├── spatial-mapper.ts         # SiteGraph → SpatialMap
│   │   ├── procedural-generator.ts   # SpatialMap → SceneDescriptor
│   │   ├── style-engine.ts           # Zone style derivation
│   │   └── types.ts                  # Pipeline types (SiteGraph, SpatialMap, etc.)
│   │
│   ├── services/
│   │   ├── cache.ts                  # D1 + R2 caching layer
│   │   ├── embedding.ts              # Vectorize embedding service
│   │   └── rate-limit.ts             # Rate limiting
│   │
│   ├── types/
│   │   ├── index.ts                  # Shared types
│   │   ├── geometry.ts               # Vec3, Quat, BoundingBox3D
│   │   └── env.ts                    # Worker env bindings
│   │
│   └── utils/
│       ├── math.ts                   # clamp, lerp, hashString
│       ├── color.ts                  # Color manipulation (hue shift, palette)
│       └── validation.ts            # Zod schemas
│
├── client/                           # Next.js client app
│   ├── package.json
│   ├── next.config.js
│   ├── tsconfig.json
│   ├── tailwind.config.ts
│   │
│   ├── app/
│   │   ├── layout.tsx                # Root layout
│   │   ├── page.tsx                  # Landing page
│   │   ├── explore/
│   │   │   └── [...slug]/
│   │   │       └── page.tsx          # /explore/reddit.com/r/programming
│   │   └── api/
│   │       └── proxy/
│   │           └── [...path]/
│   │               └── route.ts      # Proxy to Worker API (dev only)
│   │
│   ├── components/
│   │   ├── scene/
│   │   │   ├── SceneLoader.tsx       # Fetches + displays scene
│   │   │   ├── SceneRenderer.tsx     # Three.js scene from descriptor
│   │   │   ├── Building.tsx          # Building mesh component
│   │   │   ├── Furniture.tsx         # Furniture mesh component
│   │   │   ├── Ground.tsx            # Ground plane
│   │   │   ├── Portal.tsx            # Zone transition portal
│   │   │   ├── Signage.tsx           # 3D text labels
│   │   │   └── Environment.tsx       # Skybox, fog, lights, particles
│   │   │
│   │   ├── navigation/
│   │   │   ├── FPSControls.tsx       # WASD + mouse look
│   │   │   ├── Minimap.tsx           # 2D overhead minimap
│   │   │   └── Crosshair.tsx         # Aim indicator
│   │   │
│   │   ├── interaction/
│   │   │   ├── Raycaster.tsx         # Click detection
│   │   │   ├── InfoPanel.tsx         # Content overlay on click
│   │   │   └── Tooltip.tsx           # Hover tooltip
│   │   │
│   │   └── ui/
│   │       ├── LoadingScreen.tsx     # Scene loading state
│   │       ├── HUD.tsx               # Heads-up display
│   │       └── ZoneBar.tsx           # Current zone breadcrumb
│   │
│   ├── hooks/
│   │   ├── useScene.ts              # Scene data fetching
│   │   ├── useNavigation.ts         # Player movement state
│   │   └── useInteraction.ts        # Click/hover state
│   │
│   ├── stores/
│   │   ├── player.ts                # Player position, rotation
│   │   └── ui.ts                    # UI state (panel open, loading)
│   │
│   ├── lib/
│   │   ├── scene-builder.ts         # SceneDescriptor → Three.js objects
│   │   ├── material-factory.ts      # MaterialDescriptor → Three.js material
│   │   └── api.ts                   # API client
│   │
│   └── public/
│       ├── textures/                # Default textures
│       └── skyboxes/                # Skybox images
│
├── test/
│   ├── scrapers/
│   │   └── reddit.test.ts
│   ├── pipeline/
│   │   ├── normalizer.test.ts
│   │   ├── spatial-mapper.test.ts
│   │   └── procedural-generator.test.ts
│   ├── routes/
│   │   └── translate.test.ts
│   └── fixtures/
│       ├── reddit-programming.json   # Cached Reddit response
│       ├── reddit-gaming.json
│       └── reddit-askreddit.json
│
├── scripts/
│   ├── seed-db.ts                   # Seed D1 with test data
│   ├── generate-fixtures.ts         # Capture Reddit responses
│   └── benchmark.ts                 # Performance benchmarks
│
└── migrations/
    └── 0001_initial.sql             # All CREATE TABLE statements
```

---

## Day-by-Day Build Plan

### Week 1: Scraping + Normalization (Days 1-7)

#### Day 1: Project Setup

**Deliverables:** Initialized project with CI working.

**Tasks:**
1. Initialize Worker project:
   ```bash
   mkdir 3d-internet-engine && cd 3d-internet-engine
   npm init -y
   npx wrangler init . --yes
   npm install hono cheerio zod
   npm install -D vitest @cloudflare/vitest-pool-workers typescript @cloudflare/workers-types
   ```
2. Configure `wrangler.toml`:
   ```toml
   name = "3d-internet-engine"
   main = "src/index.ts"
   compatibility_date = "2024-02-08"

   [vars]
   ENVIRONMENT = "development"

   [[d1_databases]]
   binding = "DB"
   database_name = "3d-internet"
   database_id = "local"

   [[r2_buckets]]
   binding = "SCENES_BUCKET"
   bucket_name = "3d-scenes"

   [[vectorize]]
   binding = "SPATIAL_INDEX"
   index_name = "spatial-content-embeddings"
   ```
3. Create basic Hono app with health endpoint:
   ```typescript
   // src/index.ts
   import { Hono } from 'hono';
   import { translateRoutes } from './routes/translate';
   import { spatialRoutes } from './routes/spatial';

   const app = new Hono();
   app.get('/api/health', (c) => c.json({ status: 'ok', version: '0.1.0' }));
   app.route('/api/translate', translateRoutes);
   app.route('/api/spatial', spatialRoutes);
   export default app;
   ```
4. Set up TypeScript config (strict mode)
5. Set up Vitest with Workers pool
6. Run D1 migration:
   ```bash
   wrangler d1 migrations apply 3d-internet --local
   ```
7. Verify `wrangler dev` starts and `/api/health` returns 200

**Exit criteria:** `npm test` passes, `wrangler dev` runs, health endpoint works.

---

#### Day 2: Reddit Scraper — JSON API

**Deliverables:** Working Reddit data fetcher using JSON API.

**Tasks:**
1. Create `src/scrapers/types.ts`:
   ```typescript
   export interface ScrapedPost {
     id: string;
     title: string;
     author: string;
     score: number;
     numComments: number;
     createdUtc: number;
     url: string;
     selftext?: string;
     thumbnail?: string;
     imageUrl?: string;
     permalink: string;
     flair?: string;
     isStickied: boolean;
   }

   export interface ScrapedComment {
     id: string;
     author: string;
     body: string;
     score: number;
     createdUtc: number;
     depth: number;
     parentId: string;
     replies: ScrapedComment[];
   }

   export interface ScrapedSubreddit {
     name: string;
     displayName: string;
     description: string;
     subscribers: number;
     activeUsers: number;
     iconUrl?: string;
     bannerUrl?: string;
     primaryColor?: string;
     rules: string[];
     moderators: string[];
     posts: ScrapedPost[];
   }

   export interface ScrapedPostDetail extends ScrapedPost {
     comments: ScrapedComment[];
   }
   ```

2. Implement `src/scrapers/reddit.ts`:
   ```typescript
   export class RedditScraper {
     private baseUrl = 'https://www.reddit.com';
     private userAgent = '3DInternet/0.1 (educational project)';

     async scrapeSubreddit(
       subreddit: string,
       sort: 'hot' | 'top' | 'new' = 'hot',
       limit: number = 25,
     ): Promise<ScrapedSubreddit> {
       // Fetch subreddit about
       const aboutUrl = `${this.baseUrl}/r/${subreddit}/about.json`;
       const aboutResp = await fetch(aboutUrl, {
         headers: { 'User-Agent': this.userAgent },
       });
       const aboutData = await aboutResp.json();

       // Fetch posts
       const listingUrl = `${this.baseUrl}/r/${subreddit}/${sort}.json?limit=${limit}`;
       const listingResp = await fetch(listingUrl, {
         headers: { 'User-Agent': this.userAgent },
       });
       const listingData = await listingResp.json();

       return this.normalizeSubreddit(aboutData.data, listingData.data.children);
     }

     async scrapePost(subreddit: string, postId: string): Promise<ScrapedPostDetail> {
       const url = `${this.baseUrl}/r/${subreddit}/comments/${postId}.json`;
       const resp = await fetch(url, {
         headers: { 'User-Agent': this.userAgent },
       });
       const data = await resp.json();
       return this.normalizePostDetail(data);
     }

     private normalizeSubreddit(about: any, children: any[]): ScrapedSubreddit {
       return {
         name: about.display_name,
         displayName: about.display_name_prefixed,
         description: about.public_description || about.description || '',
         subscribers: about.subscribers || 0,
         activeUsers: about.active_user_count || about.accounts_active || 0,
         iconUrl: about.icon_img || about.community_icon || undefined,
         bannerUrl: about.banner_background_image || about.banner_img || undefined,
         primaryColor: about.primary_color || about.key_color || undefined,
         rules: [],
         moderators: [],
         posts: children
           .filter((c: any) => c.kind === 't3')
           .map((c: any) => this.normalizePost(c.data)),
       };
     }

     private normalizePost(data: any): ScrapedPost {
       return {
         id: data.id,
         title: data.title,
         author: data.author,
         score: data.score,
         numComments: data.num_comments,
         createdUtc: data.created_utc,
         url: data.url,
         selftext: data.selftext || undefined,
         thumbnail: data.thumbnail !== 'self' && data.thumbnail !== 'default'
           ? data.thumbnail : undefined,
         imageUrl: this.extractImageUrl(data),
         permalink: data.permalink,
         flair: data.link_flair_text || undefined,
         isStickied: data.stickied || false,
       };
     }

     private extractImageUrl(data: any): string | undefined {
       if (data.post_hint === 'image') return data.url;
       if (data.preview?.images?.[0]?.source?.url) {
         return data.preview.images[0].source.url.replace(/&amp;/g, '&');
       }
       return undefined;
     }

     private normalizePostDetail(data: any[]): ScrapedPostDetail {
       const post = this.normalizePost(data[0].data.children[0].data);
       const comments = this.normalizeComments(data[1].data.children);
       return { ...post, comments };
     }

     private normalizeComments(children: any[], depth = 0): ScrapedComment[] {
       return children
         .filter((c: any) => c.kind === 't1')
         .map((c: any) => ({
           id: c.data.id,
           author: c.data.author,
           body: c.data.body,
           score: c.data.score,
           createdUtc: c.data.created_utc,
           depth,
           parentId: c.data.parent_id,
           replies: c.data.replies?.data?.children
             ? this.normalizeComments(c.data.replies.data.children, depth + 1)
             : [],
         }));
     }
   }
   ```

3. Write tests with fixtures:
   ```typescript
   // test/scrapers/reddit.test.ts
   import { describe, it, expect, vi } from 'vitest';
   import { RedditScraper } from '../../src/scrapers/reddit';
   import programmingFixture from '../fixtures/reddit-programming.json';

   describe('RedditScraper', () => {
     it('normalizes subreddit data', async () => {
       vi.spyOn(globalThis, 'fetch').mockImplementation(async (url) => {
         if (String(url).includes('/about.json')) {
           return Response.json({ data: programmingFixture.about });
         }
         return Response.json({ data: programmingFixture.listing });
       });

       const scraper = new RedditScraper();
       const result = await scraper.scrapeSubreddit('programming');

       expect(result.name).toBe('programming');
       expect(result.posts.length).toBeGreaterThan(0);
       expect(result.posts[0]).toHaveProperty('id');
       expect(result.posts[0]).toHaveProperty('title');
       expect(result.posts[0]).toHaveProperty('score');
     });
   });
   ```

4. Capture fixture data:
   ```typescript
   // scripts/generate-fixtures.ts
   // Run manually: npx tsx scripts/generate-fixtures.ts
   const subreddits = ['programming', 'gaming', 'askreddit'];
   for (const sub of subreddits) {
     const about = await fetch(`https://www.reddit.com/r/${sub}/about.json`).then(r => r.json());
     const listing = await fetch(`https://www.reddit.com/r/${sub}/hot.json?limit=25`).then(r => r.json());
     writeFileSync(`test/fixtures/reddit-${sub}.json`, JSON.stringify({ about: about.data, listing: listing.data }));
   }
   ```

**Exit criteria:** Scraper fetches and normalizes 3 different subreddits. Tests pass with fixtures.

---

#### Day 3: Normalizer — SiteGraph

**Deliverables:** SiteGraph intermediate representation from scraped data.

**Tasks:**
1. Define SiteGraph types in `src/pipeline/types.ts`:
   ```typescript
   export interface SiteGraph {
     site: string;
     zone: string;
     nodes: SiteGraphNode[];
     edges: SiteGraphEdge[];
     metadata: ZoneMetadata;
   }

   export interface SiteGraphNode {
     id: string;
     sourceId: string;
     type: 'post' | 'comment' | 'sidebar' | 'moderator' | 'rule';
     title?: string;
     content?: string;
     author?: string;
     score: number;
     timestamp: number;
     imageUrl?: string;
     url?: string;
     depth: number;
     childCount: number;
   }

   export interface SiteGraphEdge {
     from: string;
     to: string;
     type: 'reply_to' | 'linked_from' | 'authored_by' | 'contains';
     weight: number;
   }

   export interface ZoneMetadata {
     displayName: string;
     description: string;
     subscribers: number;
     activeUsers: number;
     primaryColor?: string;
     iconUrl?: string;
     bannerUrl?: string;
     totalScore: number;
     averageScore: number;
     postCount: number;
   }
   ```

2. Implement `src/pipeline/normalizer.ts`:
   ```typescript
   export class Normalizer {
     normalize(subreddit: ScrapedSubreddit): SiteGraph {
       const nodes: SiteGraphNode[] = [];
       const edges: SiteGraphEdge[] = [];

       // Sidebar node
       nodes.push({
         id: `sidebar-${subreddit.name}`,
         sourceId: subreddit.name,
         type: 'sidebar',
         title: subreddit.displayName,
         content: subreddit.description,
         score: subreddit.subscribers,
         timestamp: Date.now(),
         depth: 0,
         childCount: 0,
       });

       // Post nodes
       for (const post of subreddit.posts) {
         const nodeId = `post-${post.id}`;
         nodes.push({
           id: nodeId,
           sourceId: post.id,
           type: 'post',
           title: post.title,
           content: post.selftext,
           author: post.author,
           score: post.score,
           timestamp: post.createdUtc * 1000,
           imageUrl: post.imageUrl || post.thumbnail,
           url: post.permalink,
           depth: 0,
           childCount: post.numComments,
         });

         // Edge: post contained in zone
         edges.push({
           from: `sidebar-${subreddit.name}`,
           to: nodeId,
           type: 'contains',
           weight: post.score,
         });
       }

       // Moderator nodes
       for (const mod of subreddit.moderators) {
         nodes.push({
           id: `mod-${mod}`,
           sourceId: mod,
           type: 'moderator',
           title: mod,
           score: 0,
           timestamp: Date.now(),
           depth: 0,
           childCount: 0,
         });
       }

       const scores = subreddit.posts.map(p => p.score);
       const totalScore = scores.reduce((a, b) => a + b, 0);

       return {
         site: 'reddit.com',
         zone: `r/${subreddit.name}`,
         nodes,
         edges,
         metadata: {
           displayName: subreddit.displayName,
           description: subreddit.description,
           subscribers: subreddit.subscribers,
           activeUsers: subreddit.activeUsers,
           primaryColor: subreddit.primaryColor,
           iconUrl: subreddit.iconUrl,
           bannerUrl: subreddit.bannerUrl,
           totalScore,
           averageScore: scores.length ? totalScore / scores.length : 0,
           postCount: subreddit.posts.length,
         },
       };
     }
   }
   ```

3. Write normalizer tests — verify node/edge counts, types, metadata.

**Exit criteria:** `ScrapedSubreddit → SiteGraph` for all 3 fixture subreddits. All nodes have valid IDs, all edges reference existing nodes.

---

#### Day 4: Spatial Mapper — Positions

**Deliverables:** SiteGraph → SpatialMap with 3D positions for every element.

**Tasks:**
1. Define SpatialMap types (extends `src/pipeline/types.ts`)
2. Implement `SpatialMapper` class:
   - Posts sorted by score → placed in concentric rings from center
   - Higher score = closer to center + taller building
   - `importance` derived: `normalize(log(score + 1)) * 0.7 + recency * 0.3`
   - Sidebar/info → fixed position at zone entrance (0, 0, -bounds.z/2)
   - Paths auto-generated between post clusters
3. Layout algorithm:
   ```typescript
   // Golden angle spiral for post placement
   function spiralLayout(posts: SiteGraphNode[], centerX = 0, centerZ = 0): Position[] {
     const goldenAngle = Math.PI * (3 - Math.sqrt(5));
     const spacing = 12; // units between buildings

     return posts.map((post, i) => {
       const r = spacing * Math.sqrt(i + 1);
       const theta = goldenAngle * i;
       return {
         x: centerX + r * Math.cos(theta),
         y: 0,
         z: centerZ + r * Math.sin(theta),
       };
     });
   }
   ```
4. Write tests: verify positions don't overlap, high-score posts are more central, bounds are reasonable.

**Exit criteria:** Every SiteGraph node has a 3D position. No overlaps (min 8 units between building centers). Layout looks reasonable when plotted as 2D scatter.

---

#### Day 5: Spatial Mapper — Embeddings + Storage

**Deliverables:** Spatial elements stored in D1, embeddings with coordinates in Vectorize.

**Tasks:**
1. After mapping, insert all elements into `spatial_elements` table
2. Generate text embeddings for post titles + content previews
3. Insert into Vectorize with spatial coordinates as metadata
4. Create zone record in `zones` table
5. Implement `SpatialSearchService`:
   ```typescript
   async function spatialSearch(
     zoneId: string,
     query: string,
     position: Vec3,
     radius: number,
   ): Promise<SpatialElement[]> {
     const embedding = await embed(query);
     const results = await vectorize.query(embedding, {
       filter: {
         zone_id: { $eq: zoneId },
         pos_x: { $gte: position.x - radius, $lte: position.x + radius },
         pos_z: { $gte: position.z - radius, $lte: position.z + radius },
       },
       topK: 10,
     });
     // Fetch full records from D1
     // ...
   }
   ```
6. Write integration tests with local D1 + Vectorize.

**Exit criteria:** 25 posts stored in D1 with positions. Vectorize search returns relevant results for "typescript" query near the center of r/programming.

---

#### Day 6-7: End-to-End Scrape-to-Store + Caching

**Deliverables:** Full pipeline from URL to stored SpatialMap. Caching layer.

**Tasks:**
1. Wire up the full pipeline in `/api/translate` route:
   ```typescript
   app.get('/', async (c) => {
     const site = c.req.query('site');
     const zone = c.req.query('zone');

     // Check cache
     const cached = await cacheService.getScene(site, zone);
     if (cached) return c.json(cached);

     // Run pipeline
     const scraped = await scraper.scrapeSubreddit(zone.replace('r/', ''));
     const graph = normalizer.normalize(scraped);
     const spatialMap = await mapper.map(graph);

     // Store in D1 + Vectorize
     await storage.storeSpatialMap(spatialMap);

     // Cache metadata in D1, full scene placeholder in R2
     await cacheService.cacheScene(site, zone, spatialMap);

     return c.json(spatialMap);
   });
   ```
2. Implement cache service:
   - D1: zone metadata + cache timestamps
   - R2: full scene blob (when generated in Week 3)
   - TTL: 5 min for hot, 60 min for stable (based on activity level)
3. Rate limiting: 60 req/min per IP
4. Error handling: Reddit 429 → retry with backoff, 404 → clear error message
5. Integration tests: full round-trip from API call to stored data

**Exit criteria:**
- `GET /api/translate?site=reddit.com&zone=r/programming` returns valid SpatialMap
- Second request returns cached data in <50ms
- Rate limiting rejects 61st request in same minute
- r/programming, r/gaming, r/askreddit all produce valid, different SpatialMaps

---

### Week 2: Procedural Generation (Days 8-14)

#### Day 8: Style Engine

**Deliverables:** Zone style derivation from subreddit metadata.

**Tasks:**
1. Implement `src/pipeline/style-engine.ts`:
   ```typescript
   export function deriveStyle(metadata: ZoneMetadata): SubredditStyle {
     // Extract color from subreddit's primary color, or derive from name
     const baseColor = metadata.primaryColor
       ? parseHexColor(metadata.primaryColor)
       : hashToColor(metadata.displayName);

     // Derive architecture from activity level and subscriber count
     const activityRatio = metadata.activeUsers / Math.max(metadata.subscribers, 1);
     const architecture = activityRatio > 0.01 ? 'whimsical'
       : metadata.subscribers > 1_000_000 ? 'geometric'
       : metadata.subscribers > 100_000 ? 'organic'
       : 'brutalist';

     // Generate full palette from base color
     const palette = generatePalette(baseColor);

     return {
       palette,
       architecture,
       density: clamp(metadata.postCount / 50, 0.3, 1.0),
       verticalScale: clamp(Math.log10(metadata.averageScore + 1) / 4, 0.3, 1.0),
       ambientSound: mapToAmbient(architecture),
       particleEffect: mapToParticles(activityRatio),
       skybox: mapToSkybox(architecture),
     };
   }

   function generatePalette(base: RGB): ColorPalette {
     return {
       primary: rgbToHex(base),
       secondary: rgbToHex(shiftHue(base, 30)),
       accent: rgbToHex(shiftHue(base, 180)),
       background: rgbToHex(desaturate(darken(base, 0.7), 0.5)),
       text: '#ffffff',
     };
   }
   ```
2. Test with 3+ subreddits: verify distinct styles
3. Style should be deterministic (same input → same output)

**Exit criteria:** 3 subreddits produce visibly different style configs. Deterministic.

---

#### Day 9-10: Procedural Generator — Buildings

**Deliverables:** SceneDescriptor with building geometry for all posts.

**Tasks:**
1. Implement `src/pipeline/procedural-generator.ts`:
   - `generateScene(map: SpatialMap, style: SubredditStyle): SceneDescriptor`
   - Building generator: footprint, height, floors from score/importance
   - Material assignment from style palette
   - Signage: post title as text descriptor
   - LOD hints: far (box), medium (basic), close (full)
2. Scene descriptor format:
   ```typescript
   interface SceneDescriptor {
     version: string;
     zone: { id: string; name: string; style: SubredditStyle };
     objects: ObjectDescriptor[];
     lights: LightDescriptor[];
     environment: EnvironmentDescriptor;
     bounds: BoundingBox3D;
   }

   interface ObjectDescriptor {
     id: string;
     type: 'building' | 'ground' | 'path' | 'sign' | 'portal' | 'furniture';
     geometry: GeometrySpec;
     material: MaterialSpec;
     position: Vec3;
     rotation?: Vec3;
     scale?: Vec3;
     children?: ObjectDescriptor[];
     userData?: Record<string, unknown>;
     lod?: LODSpec[];
   }
   ```
3. Ground plane generation
4. Path geometry between buildings (road meshes)
5. Tests: verify scene has correct number of buildings, all have valid geometry

**Exit criteria:** SceneDescriptor JSON for r/programming with 25 buildings, ground plane, paths. JSON size <300KB.

---

#### Day 11-12: Procedural Generator — Details

**Deliverables:** Windows, doors, signage, portals, environment.

**Tasks:**
1. Window generation: instanced rectangles on building faces
2. Door geometry: entrance at ground level per building
3. Signage: text descriptors with post titles
4. Portal objects: for links to other subreddits
5. Environment descriptor:
   ```typescript
   const environment: EnvironmentDescriptor = {
     skybox: style.skybox,
     fog: { color: style.palette.background, near: 50, far: 200 },
     ambientLight: { color: '#ffffff', intensity: 0.4 },
     directionalLight: {
       color: '#ffeedd',
       intensity: 0.8,
       position: { x: 50, y: 100, z: 30 },
       castShadow: true,
     },
     particles: {
       type: style.particleEffect,
       count: 200,
       color: style.palette.accent,
       area: { x: 200, y: 30, z: 200 },
     },
   };
   ```
6. Full pipeline test: scrape → normalize → map → generate → valid SceneDescriptor

**Exit criteria:** SceneDescriptor includes windows, doors, signage, environment. Complete pipeline runs end-to-end in <3s.

---

#### Day 13: R2 Storage + Cache Integration

**Deliverables:** Generated scenes cached in R2, served on repeat requests.

**Tasks:**
1. Store SceneDescriptor as JSON blob in R2:
   ```typescript
   const key = `scenes/${siteId}/${zoneId}/v${version}.json`;
   await env.SCENES_BUCKET.put(key, JSON.stringify(sceneDescriptor), {
     httpMetadata: { contentType: 'application/json' },
     customMetadata: { generatedAt: Date.now().toString(), version: version.toString() },
   });
   ```
2. Update zone record with R2 key and version
3. On cache hit: fetch from R2, return directly (skip entire pipeline)
4. Cache invalidation: TTL-based + manual refresh endpoint
5. Test: first request = full pipeline (~3s), second request = R2 fetch (<200ms)

**Exit criteria:** Cached scene served in <200ms. R2 blob matches generated scene exactly.

---

#### Day 14: Week 2 Integration + Testing

**Deliverables:** Complete server pipeline, all tests passing.

**Tasks:**
1. Integration test: full flow for 3 subreddits
2. Verify SceneDescriptor JSON structure is stable
3. Performance benchmark: generation time per subreddit
4. Error handling: invalid subreddit, Reddit API errors, malformed data
5. Document API response format

**Exit criteria:**
- 3 subreddits generate complete SceneDescriptors
- All tests pass
- Generation: <5s for 25 posts fresh, <200ms cached
- SceneDescriptor JSON is well-documented and stable

---

### Week 3: Client Renderer (Days 15-21)

#### Day 15: Next.js Setup + Scene Loading

**Deliverables:** Next.js app that fetches scene data from API.

**Tasks:**
1. Initialize Next.js project in `client/`:
   ```bash
   cd client
   npx create-next-app@latest . --typescript --tailwind --app --src-dir=no
   npm install three @react-three/fiber @react-three/drei @tanstack/react-query zustand
   npm install -D @types/three
   ```
2. Set up API proxy for development (Next.js → Wrangler dev server)
3. Create `SceneLoader` component:
   ```tsx
   'use client';
   import { useQuery } from '@tanstack/react-query';
   import { Canvas } from '@react-three/fiber';
   import { SceneRenderer } from './SceneRenderer';
   import { LoadingScreen } from '../ui/LoadingScreen';

   export function SceneLoader({ site, zone }: { site: string; zone: string }) {
     const { data, isLoading, error } = useQuery({
       queryKey: ['scene', site, zone],
       queryFn: () => fetch(`/api/translate?site=${site}&zone=${zone}`).then(r => r.json()),
       staleTime: 5 * 60 * 1000, // 5 min
     });

     if (isLoading) return <LoadingScreen zone={zone} />;
     if (error) return <div>Error loading {zone}</div>;

     return (
       <Canvas shadows camera={{ fov: 75, near: 0.1, far: 500 }}>
         <SceneRenderer descriptor={data} />
       </Canvas>
     );
   }
   ```
4. URL routing: `/explore/[...slug]/page.tsx` extracts site + zone
5. Verify: page loads, fetches data, shows loading state

**Exit criteria:** `/explore/reddit.com/r/programming` fetches scene data and logs it to console.

---

#### Day 16-17: Scene Renderer — Buildings + Ground

**Deliverables:** 3D scene with buildings and ground plane visible.

**Tasks:**
1. Implement `SceneRenderer` — iterate over objects, dispatch to sub-components
2. `Ground` component: plane mesh with zone-appropriate material
3. `Building` component:
   ```tsx
   function Building({ descriptor }: { descriptor: ObjectDescriptor }) {
     const { geometry, material, position, scale, userData } = descriptor;

     return (
       <mesh
         position={[position.x, position.y + geometry.height / 2, position.z]}
         scale={scale ? [scale.x, scale.y, scale.z] : [1, 1, 1]}
         userData={userData}
       >
         <boxGeometry args={[geometry.width, geometry.height, geometry.depth]} />
         <meshStandardMaterial
           color={material.color}
           metalness={material.metalness || 0.1}
           roughness={material.roughness || 0.7}
           emissive={material.emissive}
           emissiveIntensity={material.emissiveIntensity || 0}
         />
       </mesh>
     );
   }
   ```
4. Environment: lights, fog, skybox from descriptor
5. Camera positioned at zone entrance, facing inward

**Exit criteria:** Navigate to `/explore/reddit.com/r/programming` and see colored 3D boxes representing posts on a ground plane. Different subreddits have different colors.

---

#### Day 18: Navigation — FPS Controls

**Deliverables:** WASD + mouse look movement through the scene.

**Tasks:**
1. `FPSControls` component using PointerLockControls from drei:
   ```tsx
   import { PointerLockControls } from '@react-three/drei';
   import { useFrame, useThree } from '@react-three/fiber';
   import { useKeyboard } from '../../hooks/useNavigation';

   export function FPSControls() {
     const { camera } = useThree();
     const keys = useKeyboard(); // tracks WASD state
     const speed = 15; // units per second

     useFrame((_, delta) => {
       const direction = new THREE.Vector3();
       if (keys.forward) direction.z -= 1;
       if (keys.backward) direction.z += 1;
       if (keys.left) direction.x -= 1;
       if (keys.right) direction.x += 1;

       direction.normalize().multiplyScalar(speed * delta);
       direction.applyQuaternion(camera.quaternion);
       direction.y = 0; // stay on ground plane

       camera.position.add(direction);
       camera.position.y = 2; // eye height
     });

     return <PointerLockControls />;
   }
   ```
2. Click canvas to lock pointer, Escape to unlock
3. Basic collision: raycast downward to stay on ground, simple distance check vs buildings
4. Minimap overlay (2D top-down view of positions)

**Exit criteria:** Walk around the 3D subreddit with WASD + mouse look at 60fps.

---

#### Day 19: Interaction — Click to Inspect

**Deliverables:** Click a building to see post content.

**Tasks:**
1. Raycaster integration: detect clicked mesh
2. InfoPanel component (HTML overlay):
   ```tsx
   function InfoPanel({ element }: { element: ObjectDescriptor }) {
     const { title, score, contentUrl, content } = element.userData;
     return (
       <div className="fixed right-4 top-4 w-96 bg-gray-900/90 text-white p-6 rounded-xl">
         <h2 className="text-xl font-bold mb-2">{title}</h2>
         <div className="flex gap-2 text-sm text-gray-400 mb-4">
           <span>▲ {score}</span>
           <span>by {element.userData.author}</span>
         </div>
         {content && <p className="text-gray-300 mb-4">{content.slice(0, 300)}...</p>}
         <a href={`https://reddit.com${contentUrl}`}
            target="_blank"
            className="text-blue-400 hover:underline">
           View on Reddit →
         </a>
       </div>
     );
   }
   ```
3. Hover effect: highlight building on mouseover
4. Tooltip: post title on hover (before clicking)

**Exit criteria:** Hover a building → see title tooltip. Click → see full content panel with link to Reddit.

---

#### Day 20: Text + Signs in 3D

**Deliverables:** Post titles visible as floating signs in the 3D scene.

**Tasks:**
1. Integrate `troika-three-text` or `@react-three/drei` Text component
2. Signage component: text floating above building entrance
   ```tsx
   import { Text } from '@react-three/drei';

   function Signage({ text, position }: { text: string; position: Vec3 }) {
     return (
       <Text
         position={[position.x, position.y + 1, position.z]}
         fontSize={0.5}
         maxWidth={8}
         textAlign="center"
         color="#ffffff"
         outlineWidth={0.05}
         outlineColor="#000000"
         anchorX="center"
         anchorY="bottom"
       >
         {text}
       </Text>
     );
   }
   ```
3. Score badges: floating upvote count near signage
4. Performance: only render text for buildings within 50 units

**Exit criteria:** Walking through the scene, you can read post titles floating above buildings.

---

#### Day 21: Week 3 Integration + Performance

**Deliverables:** Complete client with all features working together.

**Tasks:**
1. Full integration: navigate scene, click buildings, read titles
2. Performance profiling:
   - Verify 60fps with 25 buildings + text
   - Measure memory usage (<200MB target)
   - Measure initial load time (<3s target)
3. LOD implementation (if needed for performance):
   - Far buildings: simple boxes
   - Close buildings: full geometry + text
4. Polish: loading screen, error states, smooth camera

**Exit criteria:**
- Navigate r/programming at 60fps
- Click any building to see content
- Text labels readable from 15+ units away
- Load time <3s from URL to rendered scene
- Works on M1 Mac in Chrome

---

### Week 4: Polish + Multi-Subreddit (Days 22-28)

#### Day 22-23: Visual Polish

**Deliverables:** Scenes that look distinct and atmospheric.

**Tasks:**
1. Image textures: load post images as building facade textures (lazy loading)
2. Portal objects: glowing archways for cross-subreddit links
3. Particle effects: floating elements matching zone theme
4. Dynamic lighting: warm glow from buildings, ambient variation
5. Ground material improvement: grid pattern, zone-appropriate texture

**Exit criteria:** r/programming, r/gaming, r/askreddit look visually distinct. Screenshots look interesting.

---

#### Day 24-25: Multi-Zone Navigation

**Deliverables:** Navigate between subreddits via portals.

**Tasks:**
1. Portal interaction: click portal → transition to new zone
2. Scene transition: fade out → load new scene → fade in
3. Zone bar UI: shows current zone, history, back button
4. URL updates: navigation updates browser URL
5. Scene caching: keep visited scenes in memory for fast back-navigation

**Exit criteria:** Click a portal in r/programming → smoothly arrive in r/gaming. Back button returns.

---

#### Day 26: API Hardening

**Deliverables:** Robust API ready for demo.

**Tasks:**
1. Comprehensive error handling (all edge cases)
2. Input validation with Zod schemas
3. CORS configuration for client → worker communication
4. Rate limiting verified under load
5. Logging: structured logs for debugging

**Exit criteria:** API handles invalid inputs gracefully, rate limits work, CORS configured.

---

#### Day 27: Testing + Performance

**Deliverables:** All tests passing, performance within budgets.

**Tasks:**
1. Server: unit tests + integration tests for all pipeline stages
2. Client: component tests for key components
3. E2E: Playwright test — navigate to URL, verify scene loads, click building
4. Performance:
   - Lighthouse score >80 for initial load
   - 60fps maintained with 50+ visible objects
   - API response times within budget
5. Fix any issues found during testing

**Exit criteria:** All tests green. Performance within budgets documented in TECHNICAL_SPECS.md.

---

#### Day 28: Demo + Documentation

**Deliverables:** Deployable demo, documentation.

**Tasks:**
1. Deploy Worker to Cloudflare Workers (production)
2. Deploy client to Cloudflare Pages
3. Create demo script:
   - Navigate r/programming
   - Explore buildings, click to read
   - Travel to r/gaming via portal
   - Show visual difference between zones
4. Record demo video (screen capture)
5. Write DEMO.md with:
   - Live URL
   - Local development setup instructions
   - Architecture overview
   - Known limitations

**Exit criteria:**
- Live demo accessible at URL
- 3 subreddits navigable with distinct visual styles
- Another developer can run locally from docs
- Demo video captured

---

## Testing Strategy

### Unit Tests (Vitest)
- **Scraper:** Mock fetch, verify normalization of Reddit JSON → ScrapedSubreddit
- **Normalizer:** ScrapedSubreddit → SiteGraph, verify node/edge integrity
- **Spatial Mapper:** SiteGraph → SpatialMap, verify positions don't overlap, importance ordering
- **Procedural Generator:** SpatialMap → SceneDescriptor, verify geometry validity
- **Style Engine:** Metadata → SubredditStyle, verify determinism and variation
- **Cache Service:** Store/retrieve/TTL logic
- **Rate Limiter:** Window-based counting

### Integration Tests (Vitest + Miniflare)
- Full pipeline: API request → scrape (mocked) → store → return
- Cache hit/miss paths
- Vectorize search relevance
- Error handling paths (404, 429, 500)

### Client Tests (Vitest + Testing Library)
- SceneRenderer: renders correct number of meshes from descriptor
- InfoPanel: displays content correctly
- Navigation: movement math correct

### E2E Tests (Playwright)
- Load URL → scene renders (verify canvas has content)
- Click building → info panel appears
- Zone transition via portal
- Performance: frame time measurements

### Test Coverage Targets
| Module | Target |
|--------|--------|
| Scrapers | >90% |
| Pipeline (normalizer, mapper, generator) | >85% |
| Routes | >80% |
| Client components | >70% |
| E2E | 3+ happy-path scenarios |

---

## Success Metrics

### Phase 1 Prototype "Done" Criteria

| Metric | Target | How to Measure |
|--------|--------|----------------|
| Subreddits renderable | ≥3 | r/programming, r/gaming, r/askreddit |
| Visual distinction | Subjective pass | Side-by-side screenshots, 3 people agree they look different |
| Scene load (cached) | <1s | Performance timing in browser |
| Scene load (fresh) | <5s | Performance timing from API call |
| Frame rate | ≥60fps | Chrome DevTools performance monitor |
| Client memory | <200MB | Chrome DevTools memory profiler |
| Scene size (network) | <500KB | Network tab, gzipped |
| Click interaction | Works | Click building → content panel |
| Text readability | Readable at 15 units | Manual verification |
| Portal navigation | Works | Click portal → new zone loads |
| API error rate | <1% | Worker analytics |
| Test coverage | >80% | Vitest coverage report |
| Can be deployed by another dev | Yes | Kimi/Grok deploys from DEMO.md |

### Stretch Goals (if ahead of schedule)
- [ ] Image textures on building facades
- [ ] Comment visualization inside buildings (walk in → see comments as furniture)
- [ ] Sound: ambient audio per zone style
- [ ] Minimap shows building labels
- [ ] Mobile touch controls

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Reddit API rate limiting | Use JSON API (more lenient than OAuth), cache aggressively, test with fixtures |
| Three.js performance with many objects | Instanced meshes for repeated geometry, LOD, frustum culling built-in |
| Scene descriptor too large | Compress with gzip, use quantized positions (2 decimal places), lazy-load details |
| Development velocity (1 developer) | Strict daily deliverables, cut scope not quality, Kimi/Grok can parallelize |
| Cloudflare Workers size limit (10MB) | Keep Worker lean, heavy data in R2, client handles rendering |
| Visual quality vs performance | Start ugly, optimize later. Colored boxes that run at 60fps > pretty scenes at 15fps |

---

*Week 1-4 is the skeleton. If the skeleton works — if you can walk through Reddit as a 3D space and it feels like something — then everything else in the Master Roadmap is achievable. This is the proof.*
