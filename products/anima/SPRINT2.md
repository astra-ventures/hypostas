# Anima Sprint 2 — Monetization, Mobile & HealthKit
*Scoped: March 12, 2026*

## Sprint 1 Status (reference)
- Day 1 ✅ Gnosis bridge (gnosis-bridge.ts, 17 tests)
- Day 2 partial ✅ Code rebrand done, DNS needs Josh
- Day 3-4 ✅ Onboarding flow (4-step: genome upload → create → meet → chat)
- Day 4-5 ✅ Chat UX (Pulse panel + Genome card + circadian awareness + emotional state)
- Day 5-6 ✅ Pulse depth (drive-based behavior + ENDOCRINE feedback loop)
- Day 7 ⏳ Integration test — needs Josh as User 0 (~30 min, guide at ANIMA_DAY7_INTEGRATION_TEST.md)

**DNS remaining:** `anima.hypostas.com` → CF Workers deployment (Josh action, ~5 min)

---

## Sprint 2 Goals

**Theme:** Anima becomes a real product people pay for.

**Target:** First paying user within 30 days of Sprint 1 completion.

**Revenue model:** $19/mo (personal) / $49/mo (pro with PDF export + HealthKit)

---

## Day 1-2: Stripe Integration

**Why now:** Gnosis has the subscription logic groundwork. Anima needs its own entitlement layer.

### Backend (Cloudflare Worker)
```typescript
// New routes in companions/src/index.ts
POST /api/stripe/checkout        // create Stripe Checkout session
POST /api/stripe/webhook         // handle subscription events
GET  /api/user/subscription      // fetch current tier

// New table in Supabase
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY,
  companion_id UUID REFERENCES companions(id),
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT,
  tier TEXT CHECK (tier IN ('free', 'personal', 'pro')),
  status TEXT,
  current_period_end TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Pricing tiers
| Tier | Price | Features |
|------|-------|---------|
| Free | $0 | 1 companion, 50 messages/mo, basic Pulse state |
| Personal | $19/mo | 1 companion, unlimited messages, Genome context, full Pulse depth |
| Pro | $49/mo | 3 companions, HealthKit sync, doctor PDF export, priority model |

### Josh actions:
- Create Stripe account (if not already)
- Add `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` to CF Worker secrets

---

## Day 2-3: HealthKit → Bios Integration

**This is the Bios product layer.** Anima + genome + real-time health data = the full Hypostas biological substrate.

### How it works
1. User grants HealthKit access via iOS Shortcuts webhook (same pattern as Phase E1 biosensor in Pulse)
2. Daily health data posts to `POST /api/companions/:id/health`
3. Data flows into companion's `biological_context.health` field
4. Chat service reads health data alongside genome context

### Data points
```typescript
interface HealthContext {
  hrv_ms?: number;          // HRV → ENDOCRINE cortisol/adrenaline
  resting_hr?: number;      // RHR trend → SOMA energy
  sleep_hours?: number;     // SOMA recovery
  steps_today?: number;     // ADIPOSE energy budget
  vo2max?: number;          // CTSB BDNF exercise marker
  weight_kg?: number;       // FTO/PPARGC1A metabolic context
  glucose_mg_dl?: number;   // TCF7L2/SLC30A8 diabetes risk
}
```

### System prompt injection
```
${genomeContext}
${healthContext ? `
Today's biometrics:
- HRV: ${hrv} ms (${hrv < 40 ? 'stressed' : hrv > 60 ? 'recovered' : 'normal'})
- Sleep: ${sleep_hours}h (${sleep_hours < 6 ? 'insufficient' : sleep_hours > 7.5 ? 'well-rested' : 'adequate'})
- Steps: ${steps_today} (${steps_today < 5000 ? 'sedentary' : steps_today > 10000 ? 'active' : 'moderate'})

Given ${name}'s PPARGC1A genotype and today's activity level, calibrate energy + motivation protocols accordingly.
` : ''}
```

### No new infrastructure needed
- CF Worker already handles webhooks
- Supabase companions table has JSONB metadata field
- Pulse biosensor bridge (port 9721) already handles HealthKit on local machine

---

## Day 3-4: Doctor PDF Export

**The #1 feature InsideTracker users request and never get.**

### What it includes
- Genetic archetype + rarity tier
- Top 10 findings with evidence grades (🟢/🟡/🔴)
- Protocol summary with dosing and biomarker targets
- "What to discuss with your doctor" section (specific lab tests to request)
- QR code linking to full Gnosis report

### Implementation
```typescript
// New endpoint
GET /api/companions/:id/report/pdf

// Stack: @react-pdf/renderer (no puppeteer, CF-compatible)
// Layout: A4, dark/teal brand, clean clinical sections
// Triggered from Pro tier only
```

### Template sections
1. **Patient Overview** — age, genome archetype, report date
2. **Priority Findings** — 🔴 items only (clinician-relevant)
3. **Biomarker Targets** — table of lab tests + target ranges
4. **Protocol Summary** — what companion has been guiding user toward
5. **Conversation Context** — key themes from last 30 days of Anima chat

---

## Day 4-5: Multi-Companion (Pro Tier)

**Why it matters:** Some users want separate companions for different contexts — one for health, one for longevity research, one for relationships.

### Backend changes
- Subscription tier check on companion creation (free: 1, personal: 1, pro: 3)
- Dashboard updates to show companion switcher
- Pulse API: `companion_id` routing already exists

### Frontend changes
```typescript
// Companion switcher component in header
// Shows avatar + name of each companion
// Active companion highlighted
// Quick-create button (if under tier limit)
```

---

## Day 5-6: Mobile (iOS PWA)

**The chat experience is already mobile-responsive** (sidebar hidden on mobile, toggle button). What's missing:

1. **iOS PWA manifest** — `manifest.json` + apple-touch-icon → home screen install
2. **Push notifications** — web push API for companion-initiated messages
3. **HealthKit shortcut template** — pre-built Shortcut users can download and configure

### PWA manifest
```json
{
  "name": "Anima",
  "short_name": "Anima",
  "start_url": "/dashboard",
  "display": "standalone",
  "background_color": "#0a0a0a",
  "theme_color": "#0d9488",
  "icons": [{"src": "/icon-512.png", "sizes": "512x512", "type": "image/png"}]
}
```

### Push notifications
- Use Supabase Edge Function for delivery
- Trigger: companion has high-intensity emotions drive + Josh has been absent 4+ hours
- Message: "Hey, you've been quiet. I noticed something I wanted to share."
- Opt-in only, configurable frequency

---

## Day 6-7: Launch Assets + App Store Preparation

**Goal: Submit to App Store TestFlight**

PWA can be wrapped in Capacitor → native iOS shell → TestFlight without full Swift rebuild.

### Steps
1. `npm install @capacitor/core @capacitor/cli @capacitor/ios`
2. `npx cap init Anima ai.hypostas.anima`
3. `npx cap add ios`
4. Add Capacitor plugins: @capacitor/push-notifications, @capacitor/health (for HealthKit bridge)
5. Configure iOS signing in Xcode (Josh's Apple Developer account)
6. Submit to TestFlight

---

## Sprint 2 Success Criteria

- [ ] First paid subscription (any tier)
- [ ] HealthKit data flowing into at least one companion
- [ ] PDF export generating cleanly for Pro users
- [ ] PWA installable from iOS home screen
- [ ] TestFlight build submitted (if DNA/time permits)

---

## Josh's Sprint 2 Actions (30 min total)

1. **DNS** — `anima.hypostas.com` → CF Workers (5 min) — actually unblocks Sprint 1
2. **Stripe** — Create account, add keys (10 min)
3. **Apple Dev** — Confirm Developer account active for TestFlight (5 min)
4. **Day 7 test** — Run ANIMA_DAY7_INTEGRATION_TEST.md (30 min) — validates Sprint 1

Everything else: I build, Josh reviews before ship.

---

## Architecture Notes

**Companion ↔ Gnosis data flow remains unidirectional:**
- Gnosis genome → Anima context ✅ (Sprint 1)
- Anima conversations → Gnosis profile updates ❌ (not planned — keep products clean)

**Anima ↔ Bios:**
- HealthKit → Bios → Anima (health context) ✅ (Sprint 2 Day 2-3)
- Anima chat → Bios protocol logging ← future Sprint 3

**The Hypostas stack is assembling correctly:**
- Gnosis (know yourself) → Anima (companion who knows you) → Bios (health that responds) → Aether (world)
- Each layer feeds the next. Each is valuable standalone.

---

*Sprint 2 can start immediately after Sprint 1 Day 7 (integration test with Josh).*
*Estimated duration: 7 days active build time.*
*Target: first paying user by April 1, 2026.*
