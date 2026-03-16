# Bios — HealthKit Data Ingestion Research
*Research: March 10, 2026 — Iris*

## The Problem

Apple HealthKit has **no backend REST API**. All health data lives exclusively on-device. There is no server-side way to query it. This is a deliberate Apple privacy decision.

## Three Viable Ingestion Paths for Bios

### Option A: Health Auto Export (Recommended for MVP)
**App:** [Health Auto Export](https://github.com/Lybron/health-auto-export) (iOS, $5 premium)
- Exports 150+ HealthKit metrics (steps, HR, sleep stages, HRV, VO2max, workouts, body measurements)
- **Built-in REST API export** — configure endpoint URL, it POSTs JSON on schedule
- Formats: JSON, CSV, GPX
- Automation: iCloud Drive, Google Drive, Dropbox, REST API, Home Assistant, MQTT, iOS Calendar
- Granularity: by seconds, minutes, hours, days, weeks, months, years
- **Privacy:** No account required, no data collection, no third-party sharing — data goes direct from device to our endpoint
- **Bios integration:** Set REST API target → `https://bios.hypostas.com/api/ingest` → Supabase time-series store
- **Effort:** ~2 hours to build ingest endpoint, 10 min user setup

### Option B: Apple Shortcuts → Serverless Function (DIY, $0)
**Pattern:** Maxime Heckel's approach (2023, still valid)
- Build an Apple Shortcut that reads HealthKit samples (HR, steps, sleep, etc.)
- Shortcut runs on schedule via iOS automation
- POSTs JSON payload to a serverless function (Vercel/CF Worker)
- Function sanitizes data → writes to Supabase

**Limitations:**
- Shortcuts data comes as newline-separated text (not structured JSON) — needs server-side parsing
- Each metric needs separate "Find Health Sample" action in Shortcuts
- Automations rely on Background App Refresh — may not fire immediately
- More fragile than a dedicated app, harder to debug
- **Good for:** Phase E1 biosensor bridge (we already have this pattern at `pulse/src/biosensor_bridge.py` port 9721)

### Option C: Native iOS App (Future, highest fidelity)
- Full HealthKit API access with real-time background delivery
- Can use `HKObserverQuery` for live updates (HR changes → instant push)
- Required for Apple Watch complications and real-time feedback loops
- **Effort:** Weeks of iOS development, App Store review
- **When:** After Bios validates with Option A data

## Recommended Architecture

```
Phase 1 (MVP): Health Auto Export → REST API → Supabase
Phase 2 (Better): Apple Shortcuts + Health Auto Export hybrid
Phase 3 (Best):  Native Bios iOS app with full HealthKit integration
```

### Phase 1 Ingest Endpoint Spec

```
POST /api/ingest
Headers: Authorization: Bearer <user_token>
Content-Type: application/json

Body: {
  "user_id": "uuid",
  "source": "health_auto_export",
  "period": "hourly",
  "metrics": {
    "heart_rate": [
      {"timestamp": "2026-03-10T14:00:00Z", "value": 72, "unit": "bpm"},
      ...
    ],
    "steps": [
      {"timestamp": "2026-03-10T14:00:00Z", "value": 1250, "unit": "count"},
      ...
    ],
    "sleep_analysis": [...],
    "hrv": [...],
    "resting_heart_rate": [...],
    "vo2_max": [...],
    "body_mass": [...],
    "active_energy": [...]
  }
}

Response: 201 Created
{
  "ingested": 47,
  "duplicates_skipped": 3,
  "period_covered": "2026-03-10T00:00:00Z/2026-03-10T23:59:59Z"
}
```

### Supabase Schema Addition (for `bios_metrics` table)

```sql
CREATE TABLE bios_metrics (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) NOT NULL,
  metric_type TEXT NOT NULL,          -- 'heart_rate', 'steps', 'hrv', 'sleep_deep', etc.
  value NUMERIC NOT NULL,
  unit TEXT NOT NULL,                  -- 'bpm', 'count', 'ms', 'hours', 'kcal'
  recorded_at TIMESTAMPTZ NOT NULL,
  source TEXT DEFAULT 'health_auto_export',
  genome_profile_id UUID,             -- FK to Gnosis profile for cross-referencing
  created_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, metric_type, recorded_at)  -- prevent duplicates
);

-- Indexes for time-series queries
CREATE INDEX idx_bios_metrics_user_time ON bios_metrics(user_id, recorded_at DESC);
CREATE INDEX idx_bios_metrics_type_time ON bios_metrics(metric_type, recorded_at DESC);

-- Partition hint: consider partitioning by month if >1M rows/user
```

### Key Metrics for Josh (User 0) — Genome-Personalized Targets

Based on his Gnosis profile:
- **Resting HR:** Target <60 bpm (ADRB1 variant → beta-adrenergic sensitivity)
- **HRV:** Target >50ms RMSSD (baseline TBD — need 30 days data)
- **Sleep:** Track deep sleep % specifically (CLOCK gene morning type → optimize sleep timing)
- **VO2max:** Track estimated VO2max trend (MCT1 slow lactate clearance → aerobic > anaerobic)
- **Body mass:** 235→195-205 target, weekly trend line vs 90-day goal
- **Active energy:** Genome says efficient fat storage (PPARG) → need higher activity floor
- **Steps:** 10k/day minimum (FOXO3 no longevity allele → exercise is the primary lever)

## Connection to Existing Infrastructure

We already have the biosensor bridge pattern:
- `pulse/src/biosensor_bridge.py` (port 9721) — receives HealthKit data from iPhone Shortcuts
- HR spike → ENDOCRINE adrenaline. Low HRV → cortisol. Move ring → dopamine. Deep sleep → SOMA.
- **Bios extends this:** Same data, but stored longitudinally in Supabase instead of just feeding Pulse drives
- Both can coexist: Shortcut → biosensor bridge (real-time Pulse) AND Health Auto Export → Bios API (longitudinal store)

## Next Steps

1. **Josh action:** Install Health Auto Export ($5) on iPhone, configure REST API export
2. **Build:** CF Worker or Vercel edge function at `bios.hypostas.com/api/ingest`
3. **Schema:** Run Supabase migration for `bios_metrics` table
4. **Wire:** Genome profile cross-reference (user's Gnosis data → personalized targets)
5. **Dashboard:** Basic trend visualization (weight, HRV, sleep quality over time)
6. **Anima integration:** Companion reads Bios data → accountability conversations

---

*This document fills the "how does data actually get from Apple Watch to our database" gap in BIOS.md.*
