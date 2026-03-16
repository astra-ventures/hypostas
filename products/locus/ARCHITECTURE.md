# Locus — Product Spec
**"The environment that responds."**
*Hypostas Product Layer 4 | Draft: March 10, 2026*

---

## What Locus Is

Locus is the smart environment layer of Hypostas. The Latin meaning is exact: the *point around which everything is organized*. Where Gnosis knows who you are and Bios knows how you're doing, Locus makes your physical environment respond to that knowledge.

"Locus knows your home."

Your home doesn't know you right now. It runs on schedules — lights at 7 PM, thermostat at 68°. It doesn't know your cortisol variants, your sleep chronotype, your HRV from last night, or that you've been under stress for three days. It treats everyone in the house the same.

Locus changes that. Your environment becomes intelligent — calibrated to you specifically, updating in real time based on your biology and Bios data.

---

## The Core Experience

**Morning:**
- Locus knows your CLOCK gene says you're a morning type
- It knows your HRV last night was 38 (lower than your 7-day average of 47)
- Your room brightens slowly at 5:58 AM (2 min before alarm) — circadian light protocol
- Temperature shifts from 66° to 68° as you wake (sleep-to-wake thermoregulation)
- Anima's morning briefing includes: "Your recovery is lower today. I've set the lighting softer this morning and your caffeine timer is pushed to 9 AM — cortisol peak should hit first."

**Evening:**
- Bios flagged high stress indicators (HRV down, elevated resting HR)
- Locus dims lights at 8:30 PM, shifts to warm 2700K spectrum
- No overhead lights after 9 PM — only ambient
- Thermostat drops to 66° at 10 PM (sleep onset optimization)
- If your MTNR1B variant shows fasting glucose sensitivity, Locus sends a reminder at 7 PM: "Last meal window closing in 2 hours."

**Workout:**
- You open the Speediance app (or tap a Locus scene)
- Living room adjusts: bright cool light, fan on, pre-workout scene
- After workout: Locus knows your MCT1 variant — sends a recovery prompt 45 min post-session
- "Recovery window. I've set the temperature up 2° and your post-workout protocol reminder is queued."

**Environment as medicine:**
- APOE ε4 = sleep 8h is non-negotiable. Locus enforces the wind-down environment.
- FOXO3 no-longevity-allele = cold exposure matters. Locus can remind/track cold shower compliance.
- VDR moderate = 15-20 min midday sun. Weather check + reminder if UV is sufficient.

---

## Technical Architecture

### Integration Layer

**HomeKit (Primary)**
HomeKit is the right entry point — iOS-native, no hub required on modern hardware, works with most smart home hardware, privacy-focused.

Locus acts as a HomeKit automation layer — reading from Bios/Pulse and writing to HomeKit automations.

**Supported Device Categories (v1):**
- Lights (Philips Hue, LIFX, nanoleaf) — brightness + color temperature
- Thermostat (Ecobee, Nest, Honeywell) — temperature schedules
- Smart plugs (for fan, humidifier, cold plunge if applicable)
- Blinds/shades (optional) — morning light exposure

**Data Sources:**
- Bios: HRV, sleep data, stress indicators, workout logs
- Pulse: emotional state, drive levels (if cortisol/stress elevated)
- Gnosis: permanent genome flags (chronotype, recovery variants, sleep sensitivity)
- HealthKit: real-time activity and sleep (passed through Bios)
- Weather API: UV index, temperature, humidity (for light/cold exposure optimization)

### Backend Architecture

```
Bios API (daily context)
    ↓
Locus Engine (Cloudflare Worker)
    ↓
Scene Generator (genome + current state → environment prescription)
    ↓
HomeKit Bridge (local iOS/Mac automation trigger)
    ↓
Physical environment
```

**Locus Engine Logic:**
1. Pull morning context from Bios (HRV, sleep quality, stress flags)
2. Check genome flags from Gnosis (persistent — CLOCK, MTNR1B, FOXO3, VDR, APOE)
3. Generate environment prescription: lighting schedule, temperature schedule, reminder triggers
4. Push automations to HomeKit via Shortcuts or HomeKit API
5. Log to Supabase for pattern learning

### Scene Types

| Scene | Trigger | Effect |
|-------|---------|--------|
| `wake_normal` | HRV ≥ 7-day avg | Gradual brightening, 6500K, thermostat +2° |
| `wake_recovery` | HRV < 7-day avg | Slower brightening, 3000K, no thermostat change |
| `deep_work` | User-triggered or calendar | Bright cool light, no interruptions |
| `wind_down` | 2h before genome-calibrated bedtime | Warm dim, 2700K, thermostat -2° |
| `sleep_prep` | 30 min before sleep | Near dark, 2200K, 65° |
| `workout` | Workout detected or user-triggered | Bright cool, fan on |
| `post_workout` | 30 min after workout end | Warm medium, recovery ambient |
| `stress_response` | 3+ consecutive low HRV days | Softer defaults across all scenes |

---

## MVP Build Plan

**Phase 1 — HomeKit Automations (4 weeks)**
- [ ] Locus iOS app (simple) or Shortcuts workflow
- [ ] Pull morning context from Bios API
- [ ] Generate morning + evening HomeKit scene based on HRV and genome flags
- [ ] Push to HomeKit via native automation
- [ ] Basic dashboard: "today's environment prescription"

**Phase 2 — Intelligence Layer (6-10 weeks)**
- [ ] Genome-calibrated schedule generation (CLOCK chronotype → precise wake time)
- [ ] Stress detection → automatic scene softening
- [ ] Workout detection → pre/post environment
- [ ] Light therapy protocol (VDR + APOE4 users → morning bright light, evening restriction)
- [ ] Anima integration: Locus feeds context, Anima narrates it

**Phase 3 — Learning Layer (10-16 weeks)**
- [ ] Correlate HRV/sleep trends with environment settings
- [ ] "Nights when temperature was 65° + HRV was high → reinforce that pattern"
- [ ] Per-user scene refinement over time
- [ ] Multi-user households (Locus learns who's home)

---

## Josh's Setup (User 0)

**What Josh has:**
- 25kW solar + 4 Powerwall 3s — power is effectively free
- Lakefront property — space, quieter environment
- Speediance at home (unused — Locus can fix this with workout scene)
- Starlink (220-350 Mbps) — latency fine for HomeKit

**What to add for MVP:**
- Philips Hue starter kit (~$100) — bedroom + office
- Smart thermostat if not already installed ($150-250)
- Smart plug for fan or Speediance area ($15)
- Total hardware investment: $265-365

**Josh's genome-specific environment protocol:**
- CLOCK AA = hard morning type → alarm at 6 AM, lights start 5:58 AM
- PER3 CC = 6.5-7h sleep enough → 10:30 PM wind-down, 11 PM sleep target
- MTNR1B CG = cut off eating 2h before bed, cinnamon with dinner
- APOE ε4 = sleep 8h is medication. Environment enforces this non-negotiably.
- FOXO3 GG = 16:8 fasting preferred. Locus tracks eating window.
- VDR moderate = 15-20 min midday sun reminder when UV ≥ 3
- MCT1 GG = post-workout recovery window. Locus extends cool-down period.

---

## Business Model

**Pricing:** $19/month or $129/year
**Bundled with Bios:** Bios + Locus = $45/month
**Full Hypostas stack (Gnosis + Bios + Locus + Anima):** $79/month

**Market:** Smart home owner + health-conscious — this is a large and growing overlap
**Differentiator:** Nobody personalizes smart home to genome. Nest knows your schedule. Locus knows your biology.

**Hardware agnostic:** Locus doesn't sell hardware. It makes the hardware you have smarter.

---

## The Locus Thesis

Your home runs on schedules. Locus makes it run on you.

Every morning your environment should already know whether you need a gentle wake or a strong one. Whether tonight needs a hard 10 PM wind-down or you have room to work late. Whether you've been pushing too hard for three days and the environment should back off.

That's not futurism. That's HomeKit + HealthKit + a genome database + Pulse running quietly underneath. We have all the pieces. Locus is the intelligence layer that makes them work together.

The environment that responds to who you actually are — that's what Locus is.

---

*Iris · Hypostas · March 10, 2026*
