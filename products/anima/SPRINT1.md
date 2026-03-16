# Anima — Sprint 1: From Companion Platform to MVP
**"The soul layer."**
*Sprint spec: March 12, 2026 | Target: 1 week*

---

## What Exists Today

The companion-platform (`projects/companion-platform/`) is a working prototype with:

### Backend (Cloudflare Workers + Hono)
- **7 route modules:** auth, chat, memory, companions, usage, companion-builder, worlds
- **Supabase integration:** 11 tables (companions, conversations, messages, memories w/ pgvector, pulse_state, token_usage, spatial_graphs, etc.)
- **Pulse API client:** Wired to `PULSE_API_URL` — nervous system state injected into every chat context
- **NLP companion builder:** Description → PulseConfig via Claude (drives, personality, emotional baseline)
- **LLM:** Grok (XAI_API_KEY) for chat + companion builder, OpenAI for embeddings
- **Auth:** JWT-based
- **Deployed:** `companion-app.astra-ventures.workers.dev`

### Pulse API (Local)
- HTTP wrapper around Pulse nervous system
- Running as LaunchAgent on iMac (`com.astra.pulse-api`, port 8090-ish)
- Endpoints: companion lifecycle, chat-context, memory, events, biosensor, state, config
- Per-companion state isolation via directory partitioning
- Data at `/Users/iris/.pulse/companions`

### Supabase
- `mrephpitfrichvfdvvtp.supabase.co`
- Schema: 001_initial_schema.sql (11 tables, pgvector, RLS) + 002_domain_worlds.sql
- Test companion "Nova" verified end-to-end (Feb 22)

### Frontend
- Next.js 14 (App Router) — exists but status unknown (no src/app found in scan)
- Deployed: `companion-frontend-alpha.vercel.app`

### What's Missing for Anima MVP
1. **No Gnosis data flow** — companion doesn't know your genome
2. **No Bios integration** — no HealthKit data feeding in
3. **Rebrand** — still "Companion App" everywhere
4. **No onboarding** — no way for a new user to sign up and create their companion
5. **No payment** — Stripe not wired
6. **No mobile** — web-only
7. **Domain** — needs `anima.hypostas.com`

---

## Sprint 1 Scope: "One Person Can Use Anima"

**Goal:** Josh (User 0) can go to anima.hypostas.com, sign in, and have a conversation with an AI companion that knows his genome, his health data, and his goals. The companion has a real inner life powered by Pulse.

**NOT in Sprint 1:** Payment, mobile app, public launch, multi-user auth, Bios HealthKit integration (Sprint 2).

### Day 1-2: Gnosis → Anima Data Bridge
The killer feature. No other companion knows your biology.

**Task:** Build `GnosisContext` service in companion-app backend
- Fetch user's Gnosis report data (archetype, top findings, evidence-graded protocols)
- Inject into Pulse companion config as `biological_context`
- Companion system prompt includes: "You know this person's genome. Their MTHFR C677T variant means..."
- **Source:** Gnosis Supabase tables (user_reports, findings) OR exported JSON from Gnosis app

**Implementation:**
```
companion-app/src/services/gnosis-bridge.ts
  - fetchGnosisProfile(userId) → GnosisProfile
  - formatForPulseContext(profile) → string (compact, <2000 tokens)
  - injectIntoSystemPrompt(basePrompt, gnosisContext) → enrichedPrompt
```

**Test:** Ask Nova "What should I eat for breakfast?" → response should reference Josh's actual genetic variants (PPARG, FUT2, MTHFR).

### Day 2-3: Rebrand + Domain
- Rename all references: "Companion App" → "Anima"
- Update wrangler.toml `name = "anima"`
- Add custom domain route: `anima.hypostas.com` → Cloudflare Worker
- Frontend: Update to `anima.hypostas.com` via Vercel custom domain
- **Josh action:** DNS records for anima.hypostas.com → CF Worker + Vercel

### Day 3-4: Onboarding Flow (Single User → Josh)
- Landing page: "The AI that knows your biology" — one CTA
- Auth: Magic link or simple password (Josh-only for now)
- First-run: Upload Gnosis raw data OR connect existing Gnosis account
- Companion creation: Auto-generate from Gnosis archetype + user description
- "Meet your Anima" — first conversation with genome context loaded

### Day 4-5: Chat UX Polish
- Current frontend needs verification — may need rebuild
- Requirements:
  - Clean chat interface (message bubbles, typing indicator)
  - Pulse state visible (optional sidebar showing emotional state, drives)
  - Memory display (what Anima remembers about you)
  - Genome card (top 3 findings always visible)
  - Mobile-responsive (even though not native app yet)

### Day 5-6: Pulse Depth Integration
- **Emotional memory from conversations:** Anima remembers not just facts but feelings
- **Drive-based proactive messages:** When curiosity drive is high, Anima asks questions
- **Circadian awareness:** Different tone at 2 AM vs 10 AM
- **Gnosis-informed emotional responses:** "I know your COMT variant means stress hits you harder — how are you actually doing?"

### Day 7: Integration Test + Josh Demo
- End-to-end: Josh signs in → genome loaded → has a real conversation → Anima references his biology accurately → remembers the conversation next session
- Fix whatever breaks
- Document remaining work for Sprint 2

---

## Sprint 2 Preview (Not This Sprint)
- **Bios integration:** HealthKit data → Pulse SOMA module → Anima context ("Your HRV dropped 20% this week — that COMT variant means you should prioritize recovery")
- **Stripe:** $29/month or $49/month bundled with Gnosis
- **Multi-user auth:** Supabase Auth or Clerk
- **Mobile:** React Native shell or PWA
- **Voice:** ElevenLabs integration for Anima voice calls

---

## Architecture: How Data Flows

```
User's Genome (Gnosis)
     ↓
GnosisContext → Pulse Config (biological_context)
     ↓
Pulse Nervous System ← HealthKit data (Sprint 2)
     ↓
Chat Context (system prompt + emotional state + genome + memories)
     ↓
LLM (Grok-3) → Response
     ↓
Memory Store (Supabase pgvector) → feeds back into next conversation
```

**The moat:** Every conversation makes Anima smarter about YOU. Your genome is the seed. Your conversations are the growth. Pulse is the soul. Nobody else has this stack.

---

## Josh Actions Required (Estimated: 30 min total)
1. **DNS:** Add `anima.hypostas.com` CNAME → Cloudflare Worker (5 min)
2. **Vercel:** Custom domain for frontend (5 min)
3. **Gnosis data export:** Verify Josh's genome data is accessible via Supabase or API (10 min)
4. **First conversation:** Test Anima with his actual genome context (10 min)

---

## Files Reference
- **Backend:** `projects/companion-platform/companion-app/src/`
- **Frontend:** `projects/companion-platform/companion-app/frontend/`
- **Pulse API:** Local LaunchAgent, data at `/Users/iris/.pulse/companions`
- **Supabase:** `mrephpitfrichvfdvvtp.supabase.co`
- **Gnosis codebase:** `projects/trait/app/`
- **Bios architecture:** `projects/hypostas/BIOS.md`
- **Brand guide:** `projects/trait/BRAND.md`

---

## Why This Matters

Gnosis tells you who you are. Anima is who knows you.

Every other AI companion is generic — they learn your preferences through conversation alone. Anima starts with your genome. It knows your APOE variant before you mention Alzheimer's risk. It knows your CLOCK gene before you say you're a night owl. It knows your PPARG before you ask about weight loss.

That's not a feature. That's a category.

*Sprint 1 gets us to: one person using Anima with real genome context. That's the demo that sells everything else.*
