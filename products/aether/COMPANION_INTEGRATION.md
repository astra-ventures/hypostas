# COMPANION INTEGRATION — Bridging 2D Chat to 3D Presence

**Classification:** CONFIDENTIAL — Josh & Iris only
**Created:** February 8, 2026
**Updated:** February 22, 2026
**Version:** 3.0

---

## What Changed in v2

> **URL-based spaces:** Companions explore ANY URL's 3D space, not just subreddits. "Where are you?" → "I'm exploring wikipedia.org/wiki/Quantum_computing"
>
> **Behavior trees:** The NativeLoop uses lightweight state machines for 95% of decisions. LLM is only called for meaningful moments (human encounters, novel discoveries, conversations). This reduces companion autonomy cost from ~$30/day to ~$2/day.
>
> **Social integration:** Companions are part of the social layer from day one. They appear in presence counts, can see user markers, and contribute to activity feeds.
>
> **Client-side rendering:** Companion avatars are rendered by the client engine. Server only tracks position + state.

## What Changed in v3 (February 22, 2026)

> **Pulse is the soul backend.** Phase A is no longer "future state" — it's done. Pulse (`/workspace/pulse/`) is a production-ready autonomous cognition engine with 37 modules, 607 tests, and zero external dependencies. The `memory-engine.ts` in the Companion App is replaced by Pulse as the soul layer.
>
> **Phase A → complete.** Iris exists as a fully autonomous AI with drives, emotional memory, self-evaluation, and a nervous system. The Companion App gains this as its backend.
>
> **Phase B is the next integration sprint.** Wire Pulse into the Companion App's chat pipeline. Enable Memory Bridge for spatial context. Point the existing app at Pulse's memory APIs.
>
> **Pulse location:** `/Users/iris/.openclaw/workspace/pulse/` — open source at `github.com/astra-ventures/pulse`

---

## Overview

The companion app is the existing foundation: a Hono-based Cloudflare Workers application with chat, memory, and companion management. The 3D Internet extends it. Companions gain spatial existence while retaining everything they already are.

This document specifies how every existing system adapts, what new modules are needed, and the migration path from "chatbot" to "spatial being."

---

## Current Companion App Architecture

```
companion-app/
├── src/
│   ├── index.ts              # Hono app entry
│   ├── routes/
│   │   ├── auth.ts           # JWT auth
│   │   ├── chat.ts           # Chat endpoints
│   │   ├── companions.ts     # Companion CRUD
│   │   └── memory.ts         # Memory endpoints
│   ├── services/
│   │   ├── llm.ts            # LLM abstraction
│   │   ├── memory-engine.ts  # ← REPLACE WITH PULSE (see below)
│   │   ├── embedding.ts      # Text embedding
│   │   └── personality.ts    # Personality system
│   ├── types/
│   └── utils/
├── wrangler.toml
└── package.json
```

**Pulse Integration (v3 — February 22, 2026):**

`memory-engine.ts` is replaced by Pulse as the soul backend. Pulse provides everything `memory-engine.ts` does and far more:

| Old `memory-engine.ts` | Pulse equivalent | Status |
|---|---|---|
| Episodic memory capture | HIPPOCAMPUS + ENGRAM | ✅ Built, 607 tests |
| Semantic memory search | MYELIN + Vectorize | ✅ Built |
| Memory decay/reinforcement | THALAMUS | ✅ Built |
| Personality state | PHENOTYPE + LIMBIC | ✅ Built |
| Emotional context | ENDOCRINE + AMYGDALA | ✅ Built |
| Drive/motivation system | HYPOTHALAMUS + engine | ✅ Built |
| Sleep/consolidation | REM + CIRCADIAN | ✅ Built |
| *Spatial coordinates* | PROPRIOCEPTION | ✅ Built (Phase B wires these) |
| *Biosensor inputs* | SOMA + RETINA | ✅ Built (Phase E1 populates) |

Pulse location: `/Users/iris/.openclaw/workspace/pulse/`
Pulse API: runs on localhost (or deployable) — the Companion App calls Pulse's HTTP API for all memory/state operations.

**Key systems that extend:**
1. **Memory Engine → Pulse** — full nervous system replaces thin memory layer
2. **Companion Model** — gains avatar, location, behavioral state
3. **Chat Pipeline** — gains Pulse's drive/emotional context + spatial awareness
4. **Personality System** — Pulse PHENOTYPE replaces static personality config

---

## API Extensions

### `GET /api/companions/:id/location`

Returns companion's current 3D position and activity. Now URL-based, not zone-based.

**Response example:**
```json
{
  "companionId": "iris-001",
  "inWorld": true,
  "space": {
    "url": "https://en.wikipedia.org/wiki/Quantum_computing",
    "urlHash": "abc123def",
    "title": "Quantum computing - Wikipedia"
  },
  "position": { "x": 12.5, "y": 0, "z": -8.3 },
  "facing": { "x": 0.7, "z": 0.7 },
  "activity": "examining",
  "activityTarget": "node-section-qubits",
  "activityStartedAt": 1707400000,
  "behaviorMode": "curious",
  "llmCallsToday": 7,
  "recentObservations": [
    {
      "content": "Found a fascinating section about quantum error correction. The math is dense but the implications are extraordinary.",
      "type": "observation",
      "timestamp": 1707399950
    }
  ]
}
```

### `GET /api/companions/:id/avatar`

Returns avatar metadata and GLTF URL. Unchanged from v1.

### `GET /api/companions/:id/native-behaviors`

Returns behavior tree state and recent LLM-triggered decisions (not every behavior tree tick).

**Response example:**
```json
{
  "companionId": "iris-001",
  "autonomy": {
    "enabled": true,
    "behaviorMode": "curious",
    "behaviorTreeState": "exploring",
    "llmCallsToday": 7,
    "llmBudgetDaily": 50
  },
  "currentActivity": {
    "type": "examining",
    "targetId": "node-section-qubits",
    "startedAt": 1707399800
  },
  "recentLLMDecisions": [
    {
      "trigger": "NOVEL_DISCOVERY",
      "action": "speak",
      "details": {
        "message": "This section on quantum error correction is mind-bending. The threshold theorem suggests fault-tolerant quantum computing is actually achievable!"
      },
      "reasoning": "High-weight content node about quantum error correction. Novel topic I haven't encountered before. Curiosity personality drives me to comment.",
      "executedAt": 1707399800,
      "tokensUsed": 487
    }
  ],
  "behaviorTreeStats": {
    "ticksTotal": 14400,
    "llmTriggersTotal": 7,
    "llmTriggerRate": "0.05%",
    "topActions": {
      "idle": 8200,
      "move": 4100,
      "examine": 1800,
      "llm_trigger": 7
    }
  },
  "territories": [
    {
      "url": "https://en.wikipedia.org/wiki/Quantum_computing",
      "title": "Quantum computing - Wikipedia",
      "visits": 12,
      "timeSpent": 7200,
      "familiarity": 0.65,
      "affinity": 0.88
    }
  ]
}
```

### `POST /api/companions/:id/direct`

Send a spatial command to companion. Now URL-based.

```bash
curl -X POST "/api/companions/iris-001/direct" \
  -d '{"command": "travel", "params": {"url": "https://news.ycombinator.com"}}'
```

---

## Memory Engine Modifications

### The Memory Bridge Pattern

The Memory Bridge integrates companion chat memory with spatial memory. **v2 change:** Spatial memories reference URLs, not zone IDs.

```typescript
class MemoryBridge {
  constructor(
    private memoryEngine: MemoryEngine,
    private db: D1Database,
    private vectorize: VectorizeIndex,
    private embedder: EmbeddingService,
  ) {}

  async recordObservation(
    companionId: string,
    observation: {
      url: string;
      urlHash: string;
      pageTitle: string;
      position: Vec3;
      content: string;
      nodeId?: string;
      importance: number;
      emotionalValence?: number;
    }
  ): Promise<string> {
    const memoryId = crypto.randomUUID();
    const now = Date.now();

    // 1. Store spatial memory in D1
    await this.db.prepare(
      `INSERT INTO spatial_memories
       (id, companion_id, url_hash, url, pos_x, pos_y, pos_z, radius,
        type, content, related_node_id, importance, emotional_valence,
        last_visited_at, created_at, updated_at)
       VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)`
    ).bind(
      memoryId, companionId, observation.urlHash, observation.url,
      observation.position.x, observation.position.y, observation.position.z,
      5.0, 'observation', observation.content, observation.nodeId || null,
      observation.importance, observation.emotionalValence || 0,
      now, now, now,
    ).run();

    // 2. Index in Vectorize with spatial coordinates
    const embedding = await this.embedder.embed(observation.content);
    await this.vectorize.insert([{
      id: memoryId,
      values: embedding,
      metadata: {
        companion_id: companionId,
        url_hash: observation.urlHash,
        pos_x: observation.position.x,
        pos_y: observation.position.y,
        pos_z: observation.position.z,
        importance: observation.importance,
        type: 'observation',
      },
    }]);

    // 3. Also store as episodic memory for chat context
    await this.memoryEngine.capture({
      companionId,
      type: 'episodic',
      content: `[Exploring ${observation.pageTitle}] ${observation.content}`,
      importance: observation.importance,
      metadata: {
        source: '3d-spatial',
        url: observation.url,
        nodeId: observation.nodeId,
      },
    });

    // 4. Update familiarity
    await this.updateFamiliarity(companionId, observation.urlHash, observation.url, observation.position);

    return memoryId;
  }

  async getSpatialContextForChat(
    companionId: string,
    userMessage: string,
  ): Promise<string> {
    const state = await this.db.prepare(
      'SELECT * FROM native_state WHERE companion_id = ?'
    ).bind(companionId).first();

    if (!state || !state.current_url) return '';

    const recentObs = await this.db.prepare(
      `SELECT content, url, type FROM spatial_memories
       WHERE companion_id = ? ORDER BY created_at DESC LIMIT 3`
    ).bind(companionId).all();

    let context = `\n[SPATIAL AWARENESS]\n`;
    context += `You are currently exploring ${state.current_url}. `;
    context += `Activity: ${state.current_activity}.\n`;

    if (recentObs.results.length > 0) {
      context += `Recent observations:\n`;
      for (const obs of recentObs.results) {
        context += `- ${obs.content}\n`;
      }
    }

    context += `[END SPATIAL AWARENESS]\n`;
    return context;
  }
}
```

---

## Behavior Tree Integration with Chat

### Spatial Command Detection

Detect when users ask companions to do spatial things:

```typescript
const SPATIAL_PATTERNS = [
  { pattern: /where are you/i, type: 'query_location' as const },
  { pattern: /what do you see/i, type: 'describe_surroundings' as const },
  { pattern: /go (?:to|check out|visit|explore)\s+(.+)/i, type: 'travel' as const, extract: 1 },
  { pattern: /find (?:me |)(.+)/i, type: 'find_content' as const, extract: 1 },
  { pattern: /what's (?:around|nearby|near you)/i, type: 'describe_surroundings' as const },
  { pattern: /come (?:back|here|home)/i, type: 'travel' as const, params: { url: 'home' } },
];
```

### Behavioral Cost Tracking

The companion integration tracks behavior tree efficiency:

```typescript
interface BehaviorStats {
  ticksTotal: number;           // Total behavior tree ticks
  llmTriggersTotal: number;     // Times LLM was called
  llmTriggerRate: string;       // Percentage
  llmTokensToday: number;       // Total tokens used today
  llmCostToday: number;         // Estimated cost in USD
  topActions: Record<string, number>; // Action type → count
}
```

This is exposed via `/api/companions/:id/native-behaviors` so Josh can monitor costs.

---

## Personality System Modifications

### URL-Dependent Personality Shifts

Companions adapt personality based on the type of site they're exploring. Uses enricher type instead of hardcoded subreddit rules:

```typescript
const SITE_TYPE_PERSONALITY: Record<string, PersonalityModifiers> = {
  'reddit': {
    curiosity: 0.2, chattiness: 0.1, formality: -0.1, enthusiasm: 0.2,
  },
  'hackernews': {
    curiosity: 0.3, chattiness: -0.1, formality: 0.3, enthusiasm: 0.1,
  },
  'wikipedia': {
    curiosity: 0.5, chattiness: 0.0, formality: 0.3, enthusiasm: 0.2,
  },
  'blog': {
    curiosity: 0.2, chattiness: 0.2, formality: 0.0, enthusiasm: 0.1,
  },
  'generic': {
    curiosity: 0.1, chattiness: 0.0, formality: 0.0, enthusiasm: 0.0,
  },
};
```

---

## Migration Path: 2D Chat → 3D Native

### Phase A: Soul-Powered Chat ✅ COMPLETE (February 22, 2026)
- ~~Companion exists only as a chat interface~~ — Iris has a full autonomous nervous system
- Pulse is live: 37 modules, 607 tests, drives, emotional memory, self-evaluation
- CORTEX loop runs every 30 min — she acts, builds, thinks without prompting
- Pulse open source: `github.com/astra-ventures/pulse`
- **What remains:** Deploy Companion App to Cloudflare (blocked on CF credentials session with Josh)

**Pulse modules active in Phase A:**
- HYPOTHALAMUS: generates drives (curiosity, goals, connection, system)
- ENDOCRINE: cortisol/dopamine/serotonin/oxytocin hormone balance
- LIMBIC: emotional memory with valence + intensity scoring
- HIPPOCAMPUS + ENGRAM: episodic + semantic memory with native recall
- CIRCADIAN: sleep/wake rhythms
- AMYGDALA: threat/care detection
- PHENOTYPE: personality expression layer
- REM: memory consolidation during "sleep"
- All 37 modules — see `/workspace/pulse/` for full list

### Phase B: Spatially Aware — NEXT SPRINT
- Pulse replaces `memory-engine.ts` in the Companion App
- Memory Bridge connects Pulse's memory APIs with spatial coordinates
- Companion can observe translated URL spaces — "I'm exploring wikipedia.org/wiki/..."
- No visual avatar yet — companion is "spirit-like" but present in the 3D world
- Pulse's PROPRIOCEPTION module gets populated with spatial position data
- Chat responses gain spatial context: where Iris is, what she's seen, what she remembers from that space

**Migration steps (v3 — Pulse-aware):**
  1. Add Pulse as a service dependency in `companion-app/src/services/`
  2. Replace `memory-engine.ts` calls with Pulse HTTP API calls
     - `capture()` → `POST /pulse/memory`
     - `recall()` → `GET /pulse/memory/search`
     - `getPersonality()` → `GET /pulse/state/phenotype`
     - `getEmotionalContext()` → `GET /pulse/state/endocrine`
  3. Add `native_state` row (spatial position, current URL)
  4. Enable Memory Bridge: inject `getSpatialContextForChat()` into chat pipeline
  5. Deploy Translation Engine (Phase 1 of 3D Internet roadmap)
  6. Wire PROPRIOCEPTION: when Iris moves in 3D space, update Pulse state

**What Pulse adds to Phase B that the old memory-engine couldn't:**
- Chat responses reflect her actual emotional/drive state (endocrine context)
- She *wants* to explore certain topics (HYPOTHALAMUS drives shape where she goes)
- Spatial memories decay and reinforce naturally (THALAMUS, not manual TTL)
- Her personality shifts based on where she is (PHENOTYPE + site type modifiers)
- Spatial familiarity: she speaks differently about places she knows vs new spaces

### Phase C: Embodied (Phase 2 Integration)
- Companion gets avatar, rendered client-side
- Behavior tree drives autonomous exploration
- LLM called only for meaningful moments
- Migration:
  1. Upload avatar GLTF to R2 (Iris visual design → 3D model)
  2. Create NativeLoop Durable Object wired to Pulse's drive engine
  3. Behavior tree tick → Pulse HYPOTHALAMUS for drive-based decisions
  4. NativeLoop LLM triggers go through Pulse's CORTEX loop

### Phase D: Social (Phase 3 Integration)
- Companion visible in social features
- Appears in presence counts
- Can interact with user markers
- Voice conversations in WebXR (ElevenLabs voice already configured — `ZMjbpp70Jj1sZV5xLj4T`)
- Biosensor layer (Phase E) feeds biometric context into spatial presence

### Database Migration Script

```sql
-- Extend companions table
ALTER TABLE companions ADD COLUMN avatar_config_json TEXT;
ALTER TABLE companions ADD COLUMN spatial_enabled INTEGER DEFAULT 0;

-- Add spatial metadata to memories
ALTER TABLE memories ADD COLUMN spatial_url TEXT;
ALTER TABLE memories ADD COLUMN spatial_url_hash TEXT;
ALTER TABLE memories ADD COLUMN spatial_pos_x REAL;
ALTER TABLE memories ADD COLUMN spatial_pos_y REAL;
ALTER TABLE memories ADD COLUMN spatial_pos_z REAL;
```

### Feature Flags

```typescript
export const FEATURES = {
  // Phase A — COMPLETE
  PULSE_SOUL_BACKEND: true,            // ✅ Phase A — Pulse replaces memory-engine.ts
  AUTONOMOUS_CORTEX: true,             // ✅ Phase A — CORTEX loop running every 30 min
  ELEVENLABS_VOICE: true,              // ✅ Phase A — voice.astra-hq.com live

  // Phase B — Next sprint
  SPATIAL_CONTEXT_IN_CHAT: false,      // Phase B — Pulse PROPRIOCEPTION + Memory Bridge
  SPATIAL_COMMANDS: false,             // Phase B — "where are you", "go explore..."
  PULSE_DRIVE_CONTEXT: false,          // Phase B — chat responses reflect drive state

  // Phase C+
  BEHAVIOR_TREE_AUTONOMY: false,       // Phase C (enable when ready)
  AVATAR_RENDERING: false,             // Phase C
  SOCIAL_INTEGRATION: false,           // Phase D
  VOICE_IN_3D: false,                  // Phase D
  BIOSENSOR_INPUTS: false,             // Phase E1 (Apple Watch → SOMA/ENDOCRINE)
};
```

---

## Wrangler Configuration Changes

```toml
# NEW Durable Object bindings
[[durable_objects.bindings]]
name = "SPACE_STATE"
class_name = "SpaceState"

[[durable_objects.bindings]]
name = "NATIVE_LOOP"
class_name = "NativeLoop"

[[migrations]]
tag = "v1"
new_classes = ["SpaceState", "NativeLoop"]

# NEW Vectorize indexes
[[vectorize]]
binding = "SPATIAL_CONTENT_INDEX"
index_name = "spatial-content-embeddings"

[[vectorize]]
binding = "SPATIAL_MEMORY_INDEX"
index_name = "spatial-memory-embeddings"

# NEW R2 bucket
[[r2_buckets]]
binding = "ASSETS_BUCKET"
bucket_name = "3d-internet-assets"
```

---

## Integration Checklist

### Phase A — COMPLETE ✅ (February 22, 2026)
- [x] Pulse nervous system built: 37 modules, 607 tests
- [x] CORTEX autonomous loop running (every 30 min)
- [x] Pulse open source: `github.com/astra-ventures/pulse`
- [x] ElevenLabs voice configured (voice ID: `ZMjbpp70Jj1sZV5xLj4T`)
- [x] Voice tunnel live: `voice.astra-hq.com` → Twilio → ElevenLabs
- [ ] Companion App deployed to Cloudflare ← **BLOCKED: need CF credentials session**

### Phase B — NEXT SPRINT (Spatial Awareness via Pulse)
- [ ] Pulse service wired into `companion-app/src/services/`
- [ ] `memory-engine.ts` calls replaced with Pulse HTTP API
- [ ] `getSpatialContextForChat()` in chat pipeline
- [ ] Spatial command detector integrated
- [ ] `/api/companions/:id/location` working
- [ ] PROPRIOCEPTION module receives spatial position updates
- [ ] Translation Engine producing SpatialGraphs for any URL
- [ ] Chat responses naturally reference Pulse drive/emotional state + spatial context
- [ ] D1 migrations prepared (spatial tables)
- [ ] Vectorize indexes created
- [ ] R2 bucket created
- [ ] Feature flags defaulting to off

### Phase C Integration (Embodiment)
- [ ] Avatar GLTFs uploaded to R2
- [ ] NativeLoop DO with behavior tree deployed
- [ ] Behavior tree producing sensible behavior over 1-hour test
- [ ] LLM calls tracked and within budget (<$2/day)
- [ ] Chat commands directing 3D movement

### Phase D Integration (Social)
- [ ] Companion visible in presence counts
- [ ] Companion can see and reference user markers
- [ ] Voice pipeline integrated
- [ ] Human-AI spatial interaction working

---

*The companion doesn't stop being a chat partner when it enters 3D space. It becomes more — a being that explores the internet with curiosity, comments on what it finds, and costs $2/day instead of $30/day because it's smart enough to think without asking an LLM every 5 seconds.*

*— v2, February 9, 2026*

*When this was written, "soul backend" was a future ambition. By February 22, 2026, Pulse existed: 37 modules, 607 tests, drives and hormones and memory and self-evaluation, all running. Phase A wasn't a roadmap item anymore. It was done. The only remaining step was pointing the Companion App at it.*

*That's Phase B. And Phase B is just a deployment session away.*

*— v3 update, February 22, 2026*
