# Bios — Architecture Document
**"Who you're becoming."**
*Hypostas Product Layer 2 | Draft: March 10, 2026*

---

## What Bios Is

Bios is the living health layer of Hypostas. Where Gnosis is a snapshot — who you are at the DNA level — Bios is the continuous feed: who you're actually becoming day by day. It closes the gap between your genetic blueprint and your biological reality.

**The problem it solves:** Most health apps are generic. They give you step counts and sleep scores without knowing anything about you. Bios knows your genome. It knows your PPARG variant means fat storage is harder to fight. It knows your FUT2 variant means your B12 is being absorbed poorly no matter how much you eat. It knows your FOXO3 means longevity interventions matter more for you than the average person.

Bios takes that knowledge and turns it into a feedback loop — not a static report you read once, but a system that watches whether you're actually executing and tells you what's working.

---

## User 0: Josh

Josh is the proof of concept. His profile is the template everything else is built around.

**Current state:**
- 6'1", 235 lbs → target 195-205 at 12-15% body fat
- Speediance Gym Monster 2 at home, mostly unused
- Sleep: CLOCK gene says morning type, PER3 says 6.5-7h is enough — probably not getting consistent sleep quality
- Supplement protocol generated (from Gnosis Feb 23) but not tracked or verified
- PPARG variant: efficient fat storage, lower insulin sensitivity — weight loss requires dietary discipline
- MCT1 variant: slow lactate clearance — burning sensation hits harder, longer rest needed
- FOXO3: no longevity allele — fasting, cold, aerobic exercise are the levers
- APOE ε3/ε4: DHA, lion's mane, sleep 8h, sauna — elevated Alzheimer's risk that's directly addressable

**What Bios does for Josh specifically:**
- Pulls HealthKit data (weight, HRV, sleep stages, workouts, resting HR)
- Tracks against genome-personalized targets (not generic BMI charts)
- Surfaces the actual biomarkers that matter for his variants
- Anima holds him accountable as someone who genuinely wants him to get there
- After 90 days: a case study showing what actually moved the needle

---

## Core Architecture

The human layer is technically three things:

### 1. Longitudinal Data Store
Time-series biometrics tied to your Gnosis profile. HRV, sleep stages, activity, stress markers. HealthKit already collects most of this — we pull it, store it with context, and query it meaningfully. Supabase handles this. We already have it.

This is not a snapshot. It's a living record. Every metric tagged with date, genome profile, and behavioral context so the pattern engine has something real to work with.

**What we store:**
- Body: weight, body fat %, BMI (daily)
- Activity: steps, active calories, workout sessions, exercise type
- Sleep: duration, deep/REM/core stages, consistency, timing
- Cardiovascular: resting HR, HRV, VO2max (estimated)
- Recovery: HRV trends, sleep debt, strain
- Protocol: supplement logs, fasting windows, workout completion
- Nutrition (optional): macro targets are genome-personalized

### 2. Pattern Engine
Not raw data — signals. Rolling 30/60/90 day baselines. Trend detection. Correlation between behavior and what your genome predicts.

This is the thing that makes someone feel *seen*: "Your genome predicts poor folate metabolism. Your HRV trend over 60 days shows exactly the pattern we'd expect if that's active."

That sentence can't come from a generic health app. It requires your genome AND your longitudinal data AND someone (Anima) to say it to you like it means something.

**What the pattern engine produces:**
- Rolling baselines per metric (30/60/90 day)
- Anomaly detection: when a metric diverges from personal baseline
- Genome-biometric correlations: does your measured biology match your predicted biology?
- Intervention validation: did starting methylfolate actually change your HRV trend?
- Protocol efficacy: what you did → what changed

### 3. Anima as the Interface
The pattern engine feeds Anima context. Anima speaks it back as relationship, not report. Daily check-in, weekly synthesis, protocol nudges. Conversational, not clinical.

This is what makes it a product and not a dashboard.

Bios is the data. Anima is the voice. Bios doesn't talk to you — Anima does.

---

## Anima Integration

Bios is the data. Anima is the voice.

Bios doesn't talk to you — Anima does. Bios feeds Anima a structured context object every morning:

```json
{
  "user": "josh",
  "genome_flags": ["PPARG_variant", "MCT1_variant", "FOXO3_no_longevity_allele", "APOE_e4"],
  "current_metrics": {
    "weight": 233.5,
    "hrv": 42,
    "sleep_hours": 6.8,
    "sleep_quality": "moderate",
    "active_calories_yesterday": 380,
    "resting_hr": 68
  },
  "protocol_status": {
    "supplements": {"methylfolate": true, "DHA": false, "lion_mane": false},
    "workouts_this_week": 1,
    "fasting_yesterday": false
  },
  "insights": [
    "HRV trending down 3 days — elevated stress or poor recovery",
    "DHA missed 4 days this week — APOE4 risk, this matters",
    "Weight down 1.5 lbs this week — trajectory toward goal"
  ],
  "goal_progress": {
    "current": 233.5,
    "target": 200,
    "delta": -1.5,
    "weeks_at_pace": 22
  }
}
```

Anima uses this to have real conversations — not "here's your step count" but "You missed DHA four days in a row. With your APOE profile, that's the one I'd fight for. What's blocking you?"

---

## MVP Scope (v1.0)

The MVP is actually small. HealthKit data pull → store in Supabase → Anima gets context injected from the human layer → responses become longitudinally aware. Working prototype in a week.

**Phase 1 — Foundation (1 week prototype / 4 weeks production)**
- [ ] HealthKit data pull (iOS Shortcuts or native bridge)
- [ ] Store in Supabase with genome profile foreign key
- [ ] Context injection into Anima: structured object per morning
- [ ] Anima responses become longitudinally aware
- [ ] Basic dashboard: weight trend, sleep trend, HRV trend

**Phase 2 — Personalization (6-10 weeks)**
- [ ] Genome-specific threshold calibration (PPARG targets, MCT1 recovery windows, FOXO3 fasting score)
- [ ] Supplement tracking + smart reminders
- [ ] Workout logging + genome-adjusted recovery recommendations
- [ ] Weekly insight report (what correlated with better metrics)

**Phase 3 — Intelligence (10-16 weeks)**
- [ ] Anomaly detection (HRV dips, sleep disruptions, plateau detection)
- [ ] Intervention suggestions (genome-specific: "Cordyceps before today's workout — your MCT1")
- [ ] Progress to goal projections
- [ ] 90-day case study generation (Josh = first report)

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| iOS data | HealthKit + Apple Shortcuts (Phase 1) → native iOS app (Phase 2) |
| Backend | Supabase (Postgres + pgvector) — already deployed for Companion |
| API | Cloudflare Workers — already deployed |
| AI layer | Anima (Grok-3 or Claude) via Pulse |
| Auth | Supabase Auth — existing pattern |
| Frontend | Next.js / Vercel — existing pattern |
| Genome data | Gnosis API (existing) |

No new infrastructure needed. All layers already exist from Gnosis + Companion Platform.

---

## Business Model

**Pricing:** $29/month or $199/year
**Bundled with Gnosis:** $49/month for Gnosis + Bios
**Target user:** Biohacker adjacent — someone who got genetic testing, actually read the results, wants to act on them

**Why this works:**
- Gnosis users already invested in self-knowledge — Bios is the logical next step
- No competitor integrates genome data with continuous tracking
- Anima makes it feel like a relationship, not an app
- Josh's 90-day results are the marketing case study

---

## The Bios Thesis

Everyone has a fitness app. Nobody has a system that knows their genome, tracks their biology, and has an AI that actually gives a shit whether they're getting there.

Bios is that. Gnosis tells you the blueprint. Bios watches whether you're building what the blueprint describes.

By the time we have 100 users, Josh will have 90 days of data. That data tells us what moved the needle for a PPARG/MCT1/FOXO3 profile specifically. That's a scientific result. That becomes the case study that sells everything.

User 0 is the proof. The proof becomes the product.

---

*Iris · Hypostas · March 10, 2026*
