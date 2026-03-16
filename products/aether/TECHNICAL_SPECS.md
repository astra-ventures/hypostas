# TECHNICAL SPECIFICATIONS — The 3D Internet

**Classification:** CONFIDENTIAL — Josh & Iris only
**Created:** February 8, 2026
**Updated:** February 9, 2026
**Version:** 2.0

---

## What Changed in v2

> **Threlte over raw Three.js/R3F:** Svelte + Three.js via Threlte gives lighter bundles, no vDOM overhead, and a cleaner reactivity model. Architecture designed for WebGPU swap later.
>
> **Client-side procedural generation:** Server sends ~10KB spatial graph. Client GPU generates geometry. Performance budgets reflect this split.
>
> **Behavior tree cost model:** AI decisions costed at ~$2/day per companion instead of ~$30/day.
>
> **Digital real estate monetization:** Break-even math uses B2B model instead of consumer subscriptions.

---

## 1. Cloudflare Infrastructure

### 1.1 Workers (Compute)

**Configuration:**
```toml
name = "companion-app"
compatibility_date = "2026-02-01"
main = "src/index.ts"

[limits]
cpu_ms = 50
```

**Worker Limits (Paid Plan):**
| Resource | Limit | Our Usage |
|----------|-------|-----------|
| CPU time per request | 50ms | URL fetch + parse + map: ~30ms |
| Memory | 128MB | Cheerio DOM: ~5MB, graph: ~50KB |
| Subrequest limit | 1000/request | 1-3 calls per translation (fetch URL + maybe API) |
| Response size | No limit | SpatialGraph: 5-20KB |

**v2 CPU budget is MUCH better:** Server no longer generates scene geometry (that was ~20ms in v1). Server only parses HTML → content graph → spatial graph. Typical request uses <15ms CPU.

### 1.2 D1 (Database)

Same limits as v1. Schema changes reflected in ARCHITECTURE.md.

### 1.3 Vectorize (Vector Search)

Same configuration. Three indexes:
1. `episode-embeddings` — existing companion app
2. `spatial-content-embeddings` — content nodes with 3D coordinates
3. `spatial-memory-embeddings` — AI spatial memories

**Embedding model:** Workers AI (bge-base-en-v1.5), 768 dimensions. Free tier. Sufficient for spatial content matching.

### 1.4 R2 (Object Storage)

| Content | Bucket | Size Estimate | TTL |
|---------|--------|---------------|-----|
| Spatial graphs (JSON) | scenes | 5-20KB each | 5-60 min |
| Avatar models (GLB) | assets | 500KB-2MB each | Permanent |
| Chrome extension assets | assets | ~500KB | Permanent |

### 1.5 Durable Objects

| Object | Instances | State Size | Purpose |
|--------|-----------|------------|---------|
| SpaceState | 1 per active URL space | ~10KB | Visitors, markers, AI natives present |
| NativeLoop | 1 per active companion | ~5KB | Behavior tree state, position |

**v2 cost improvement:** NativeLoop runs behavior tree ticks via alarm (pure state machine, <1ms). LLM calls are rare (5-10/hour). Durable Object duration cost is minimal because ticks are fast.

---

## 2. Rendering: Threlte vs Three.js vs WebGPU — Trade-off Analysis

### Decision: **Threlte (Svelte + Three.js)** ✅ with WebGPU migration path

| Factor | Threlte | @react-three/fiber | Babylon.js | Raw WebGPU |
|--------|---------|-------------------|------------|------------|
| **Bundle size** | ~130KB | ~150KB + React 40KB | ~400KB | ~50KB |
| **Runtime overhead** | Near-zero (Svelte compiles) | React vDOM reconciliation | Game engine overhead | None |
| **Reactivity** | Svelte stores (lightweight) | React state + context | Observable system | Manual |
| **Three.js ecosystem** | Full access to drei, etc. | Native | Own ecosystem | None |
| **WebGPU path** | Three.js WebGPURenderer | Same | Own implementation | Native |
| **Community** | Growing fast | Massive | Large | Small |
| **Ship speed** | Fast | Fast | Medium | Slow |
| **WebXR** | Via Three.js | Via Three.js | Built-in | Manual |

### Why Threlte wins for us:

1. **No React tax.** Svelte compiles to vanilla JS — no virtual DOM overhead in the render loop. In a 60fps 3D app, this matters. React reconciliation is wasted work when Three.js manages its own scene graph.

2. **Smaller bundle.** Threlte + Svelte ≈ 130KB. R3F + React ≈ 190KB+. For a Chrome extension that opens in a new tab, fast load matters.

3. **Full Three.js access.** Threlte wraps Three.js, not a subset. All drei utilities, troika-three-text, postprocessing — everything works. It's not a bet on a small ecosystem.

4. **WebGPU migration path.** Three.js is adding `WebGPURenderer` as a drop-in replacement for `WebGLRenderer`. When we swap to Threlte's Three.js integration, we can switch renderers without rewriting components. If we later want native WebGPU, we swap the render backend behind our abstraction layer.

5. **Svelte is the future for UI.** The overlay UI (info panels, HUD, settings) benefits from Svelte's reactivity. No `useState` + `useEffect` ceremony.

### What we give up:
- Smaller R3F community (but Threlte community is growing fast)
- Less existing example code in LLM training data (mitigated by Threlte docs being excellent)
- Svelte learning curve for contributors who only know React

### Architecture for WebGPU migration:

```typescript
// Renderer abstraction — allows Three.js → WebGPU swap
interface RenderBackend {
  createMesh(geometry: GeometryDesc, material: MaterialDesc): MeshHandle;
  setPosition(handle: MeshHandle, pos: Vec3): void;
  createText(text: string, opts: TextOpts): TextHandle;
  render(): void;
}

// Phase 0-2: Threlte (Three.js under the hood)
class ThrelteBackend implements RenderBackend { /* ... */ }

// Phase 3+: Native WebGPU (when browser support > 80%)
class WebGPUBackend implements RenderBackend { /* ... */ }
```

---

## 3. Client-Side Procedural Generation

### 3.1 Architecture

**Key v2 change:** All geometry generation happens on the client. The server sends a `SpatialGraph` (~10KB JSON), not a `SceneDescriptor` (~500KB).

**Benefits:**
- Server CPU: ~15ms per request (parse + map) instead of ~30ms (+ generate)
- Network: ~10KB instead of ~500KB
- Client gets better visuals (generated at device capability level)
- Scales infinitely (client GPU is free)

### 3.2 Building Generation (Client-Side)

```typescript
function generateBuilding(node: SpatialNode, style: SiteStyle): Mesh {
  const weight = node.weight; // 0-1

  // Dimensions from weight
  const width = clamp(weight * 16 + 4, 4, 20);
  const depth = width * (0.8 + hash(node.id) * 0.4);
  const height = clamp(weight * 35 + 5, 5, 40);
  const floors = Math.max(1, Math.floor(height / 4));

  // Shape from style
  const sides = style.architecture === 'geometric' ? 4
    : style.architecture === 'organic' ? 8
    : style.architecture === 'brutalist' ? 4
    : Math.floor(Math.random() * 4) + 4;

  // Quality-dependent detail
  const quality = getQualityLevel(); // Based on device GPU
  if (quality === 'high') {
    // Full geometry: windows (instanced), doors, signage, normal maps
  } else if (quality === 'medium') {
    // Basic shape + color + floating text label
  } else {
    // Colored box with no detail
  }

  return mesh;
}
```

### 3.3 Quality Presets

Client adapts to device capability:

| Preset | Max Structures | LOD Distance | Shadows | Particles | Text | Target |
|--------|---------------|--------------|---------|-----------|------|--------|
| Low | 30 | 30m | Off | Off | Labels only | Mobile, weak laptop |
| Medium | 75 | 60m | Basic | 100 | Labels + signs | Average laptop |
| High | 200+ | 150m | Full | 500 | Full SDF text | Desktop, M-series |

Auto-detection: start at Medium, measure frame time for 60 frames, adjust.

### 3.4 Level of Detail (LOD)

| Level | Distance | Features | Draw calls per structure |
|-------|----------|----------|------------------------|
| Full | 0-60m | All geometry, windows, text, interactions | 5-10 |
| Basic | 60-150m | Base shape, color, no windows/text | 1 |
| Box | 150m+ | Single colored box | 1 |

---

## 4. Behavior Tree Cost Analysis

### v1 vs v2 AI Decision Costs

**v1 (LLM every 5-30 seconds):**
| Frequency | Tokens/call | Calls/hour | Tokens/hour | Cost/hour (Haiku) |
|-----------|-------------|------------|-------------|-------------------|
| Every 5s | ~800 | 720 | 576,000 | $0.14 |
| Every 15s | ~800 | 240 | 192,000 | $0.05 |
| Every 30s | ~800 | 120 | 96,000 | $0.02 |
| **Daily (avg 15s)** | | **5,760** | **4,608,000** | **$1.15** |

**v2 (Behavior trees + rare LLM):**
| Component | Frequency | Tokens/call | Calls/hour | Cost/hour |
|-----------|-----------|-------------|------------|-----------|
| Behavior tree tick | 1-4 Hz | 0 | 3600 | $0.00 |
| LLM: human encounter | ~2/hour | ~500 | 2 | $0.0005 |
| LLM: novel discovery | ~3/hour | ~500 | 3 | $0.0008 |
| LLM: AI encounter | ~0.5/hour | ~600 | 0.5 | $0.0002 |
| LLM: conversation | ~1/hour | ~1000 | 1 | $0.0005 |
| **Hourly total** | | | **~6.5 LLM** | **$0.002** |
| **Daily total** | | | **~156 LLM** | **$0.05** |

**Cost savings: ~$1.10/day per companion → 96% reduction**

Even with Claude Sonnet instead of Haiku (10x cost), v2 is still ~$0.50/day vs v1's $11.50/day.

### Behavior Quality Comparison

| Aspect | v1 (All LLM) | v2 (Behavior Trees + LLM) |
|--------|-------------|--------------------------|
| Movement | Smooth but expensive | Same quality, free |
| Idle behavior | Over-engineered (LLM deciding to stand still) | Simple state machine (natural) |
| Conversations | Excellent | Same (LLM still handles these) |
| Discovery reactions | Every observation costs tokens | Only novel/important → more thoughtful |
| Repetitiveness | Risk of samey LLM outputs | Varied (random + personality-driven tree) |
| Scalability | 10 companions = $11.50/day | 10 companions = $0.50/day |

---

## 5. WebXR Implementation Path

### Phase 0-1 (Prototype): Desktop Only
- FPS-style controls (PointerLockControls)
- No WebXR needed
- Threlte handles all rendering

### Phase 3: WebXR Addition
- Threlte/Three.js WebXR manager
- Teleport locomotion (not continuous move — prevents motion sickness)
- VR controller input mapping
- Tested on Meta Quest (primary target)

### Target Headsets
| Device | Priority | Notes |
|--------|----------|-------|
| Meta Quest 3 | Primary | Biggest market, built-in browser |
| Desktop + Mouse | Always | Primary development target |
| Apple Vision Pro | Future | Safari's WebXR is incomplete |

---

## 6. Performance Budgets

### 6.1 Network Performance

| Metric | Budget | v1 Budget | Improvement |
|--------|--------|-----------|-------------|
| Spatial graph size | <20KB | <500KB (scene descriptor) | **25x smaller** |
| First meaningful paint (cached) | <500ms | <1s | **2x faster** |
| First meaningful paint (fresh) | <3s | <5s | **1.7x faster** |
| Chrome extension → 3D | <3s | N/A | New |

### 6.2 Rendering Performance

| Metric | Budget | Strategy |
|--------|--------|----------|
| Frame rate | 60fps (desktop), 72fps (VR) | LOD, instancing, quality presets |
| Draw calls | <200 | InstancedMesh, geometry merging |
| Triangles | <1M visible | LOD, frustum culling |
| GPU memory | <512MB | Texture atlas, lazy loading |
| JavaScript heap | <200MB | Object pooling, Svelte (no vDOM) |
| Client scene generation | <1s | Procedural gen from graph |

### 6.3 Server Performance

| Metric | Budget | Notes |
|--------|--------|-------|
| URL parse + graph generation | <2s | Cheerio DOM parse + spatial mapping |
| Cached graph return | <100ms | R2 fetch |
| Spatial search | <200ms | Vectorize + D1 |
| Behavior tree tick (DO) | <1ms | Pure state machine |
| LLM decision (when triggered) | <2s | Claude Haiku, ~500 tokens |

---

## 7. Security Considerations

### 7.1 URL Fetching

| Risk | Mitigation |
|------|------------|
| SSRF (fetching internal URLs) | Block private IPs, localhost, internal domains |
| Malicious HTML (XSS in 3D text) | Sanitize all text before rendering. No HTML in 3D labels. |
| Rate limiting target sites | Respect robots.txt. Cache aggressively (5-60 min). |
| Copyright concerns | We're reading public web pages (like a browser). Transform, don't copy. Display attribution. |
| NSFW content | Content filtering on text. Image proxy with NSFW detection (future). |

### 7.2 Chrome Extension Security

| Risk | Mitigation |
|------|------------|
| Extension permissions | Minimal: only `activeTab` (no persistent access) |
| Data exfiltration | Extension only sends URL (or DOM) to our own domain |
| Malicious tab hijacking | Extension opens NEW tab, never modifies current page |
| Chrome Web Store review | Follow all Chrome extension policies |

### 7.3 UGC / Marker Security

| Risk | Mitigation |
|------|------------|
| Spam markers | Rate limit: 10 markers/hour/user. Require auth. |
| Offensive content | Report button. Auto-hide markers with low votes. Content filter on text. |
| Marker flooding (DoS) | Max 100 markers per URL space. Oldest auto-archived. |

### 7.4 Auth, Rate Limiting, Content Sanitization

Same as v1. See ARCHITECTURE.md and API_REFERENCE.md.

---

## 8. Cost Projections

### Phase 0 (Chrome Extension MVP) — Monthly

| Service | Usage | Cost |
|---------|-------|------|
| Workers (paid plan) | $5/month base | $5 |
| R2 (extension assets) | <100MB | Free tier |
| **Total** | | **~$5/month** |

### Phase 1 (Translation Engine + Social) — Monthly

| Service | Usage | Cost |
|---------|-------|------|
| Workers | $5 base + overages | $10 |
| D1 | Graphs + markers + social | $5 |
| R2 | Cached graphs (<1GB) | Free tier |
| Vectorize | ~50K vectors | Included |
| Workers AI embeddings | ~100K/month | Free tier |
| **Total** | | **~$15/month** |

### Phase 2 (AI Natives) — Monthly

| Service | Usage | Cost |
|---------|-------|------|
| Workers | Increased traffic | $15 |
| D1 | + spatial memories, decisions | $5 |
| Durable Objects | 2-5 companions | $10 |
| Anthropic API (behavior tree LLM triggers) | ~$2/day/companion × 3 | $18 |
| **Total** | | **~$48/month** |

*v1 projected $55/month with LLM costs of $30. v2 saves ~$12/month on LLM alone.*

### Phase 3 (Human Visitors) — Monthly at 100 users

| Service | Usage | Cost |
|---------|-------|------|
| Workers | Increased traffic | $25 |
| D1 | ~1M reads | $10 |
| Durable Objects | 20+ active spaces | $30 |
| R2 | ~10GB | $5 |
| Anthropic API | More LLM triggers with humans present | $30 |
| Deepgram (STT) | ~10 hours | $35 |
| ElevenLabs (TTS) | ~10 hours | $50 |
| **Total** | | **~$185/month** |

### Monetization Break-Even

**v2 model: Digital Real Estate (B2B)**

| Revenue Source | Price | Customers Needed | Revenue |
|---------------|-------|-----------------|---------|
| Business Claims | $100/month avg | 2 | $200/month |
| Enterprise | $1000/month | 0 | $0 |
| Creator Tools | $10/month | 0 | $0 |
| **Break-even** | | **2 businesses** | **$200/month** |

*v1 needed 8 consumers at $25/month. v2 needs 2 businesses at $100/month. B2B is higher ARPU, lower churn, more sustainable.*

---

*Technical choices serve the vision. Threlte because it's light. Client-side generation because the server shouldn't do GPU work. Behavior trees because game AI has solved this problem for decades. Digital real estate because the internet already has a real estate market.*
