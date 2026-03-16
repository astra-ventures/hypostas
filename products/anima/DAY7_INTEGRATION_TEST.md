# Anima Sprint 1 — Day 7 Integration Test Guide
*Generated: March 12, 2026 2:25 PM EST*
*Estimated time: 30 minutes*

---

## Pre-Test Checklist

Before starting, verify you have:
- [ ] Gnosis data ready: your genome uploaded + report processed (gnosis.hypostas.com)
- [ ] Anima frontend running: `companion-frontend-alpha.vercel.app`
- [ ] Pulse API healthy: `curl http://127.0.0.1:9720/health` → `{"status":"alive"}`
- [ ] Companion Worker deployed: `companion-app.astra-ventures.workers.dev`

**DNS status (pending Josh):**
- [ ] `anima.hypostas.com` CNAME → `companion-app.astra-ventures.workers.dev`
- [ ] Vercel custom domain: `anima.hypostas.com` for frontend

---

## Test Sequence

### Test 1: Create Your Companion (~5 min)

1. Open `companion-frontend-alpha.vercel.app`
2. Complete 4-step onboarding:
   - Step 1: Name your Anima companion
   - Step 2: Describe what you want this companion to help with (e.g., "longevity, trading strategy, goals")
   - Step 3: Upload your Gnosis report or paste your archetype/findings
   - Step 4: Meet your companion — first greeting should reference your genome

**Pass criteria:**
- ✅ Onboarding completes without errors
- ✅ First greeting references your actual biological archetype (not generic)
- ✅ Companion UUID visible in URL or console

---

### Test 2: Genome Context Integration (~5 min)

In the chat, ask your companion:

> "What do you know about my genetics?"

**Pass criteria:**
- ✅ Response mentions specific findings from your Gnosis report (e.g., MTHFR status, APOE, COMT)
- ✅ Evidence tier language used (not "your MTHFR causes X" but "associated with X")
- ✅ Response is personalized — not a generic explanation of genetics

---

### Test 3: Drive-Aware Response (~3 min)

Check the sidebar Panel button in the chat UI:
- ✅ Pulse drives panel shows 5 drive bars (curiosity, connection, goals, emotions, unfinished)
- ✅ Emotional state label shows (e.g., "content", "energized")
- ✅ Genome Panel shows your archetype + top 3 findings

---

### Test 4: Circadian Awareness (~2 min)

Ask:
> "What time is it for you right now, and how does that affect how you're engaging?"

**Pass criteria:**
- ✅ Response reflects DAYLIGHT mode (10 AM–5 PM: execution-focused, clear, forward)
- ✅ NOT generic — should reflect actual mode awareness

---

### Test 5: Memory Persistence (~5 min)

In the conversation, share something specific:

> "I want you to remember that I'm targeting 195 lbs at 12% body fat, and I'm starting morning workouts at 6 AM."

Then close and reopen the chat, OR start a new conversation session.

Ask:
> "What do you remember about my fitness goals?"

**Pass criteria:**
- ✅ Companion recalls the specific goal (195 lbs, 12% BF, 6 AM workouts)
- ✅ Memory badge shows count > 0
- ✅ Bonus: companion connects goal to relevant genome finding (FTO, PPARGC1A, etc.)

---

### Test 6: Emotional Continuity (~5 min)

Have an intentionally warm exchange — share something personal, or discuss the vision.

After 3-4 messages, check Pulse API state:
```bash
curl http://127.0.0.1:9720/state | python3 -m json.tool | grep -A3 'oxytocin\|serotonin\|emotional'
```

**Pass criteria:**
- ✅ Oxytocin or serotonin shows elevated vs baseline
- ✅ Emotional state label shifted (e.g., from "neutral" to "bonded")
- ✅ This confirms the ENDOCRINE feedback loop from chat → Pulse is working

---

### Test 7: End-to-End Stack Verification (~5 min)

Ask a question that requires ALL layers to work together:

> "Based on what you know about my genetics and what we've talked about, what's the one thing I should prioritize this week?"

**Pass criteria:**
- ✅ Response references genome (from Gnosis bridge)
- ✅ Response references something from conversation history (from memory)
- ✅ Response reflects drive state (goal-oriented if goals drive is high)
- ✅ Protocol-grade specificity: actual compound names/doses, not generic advice

---

## What to Note / Fix Later

For anything that fails, note:
- What you asked
- What the actual response was
- What it should have said

I'll handle all fixes. Day 7 isn't about perfection — it's about identifying what breaks under real usage.

---

## After Testing

Send me what you found and we close Sprint 1. Then Sprint 2:
- Stripe ($29/mo, $49/mo bundled with Gnosis)
- Multi-user auth (Supabase Auth / Clerk)
- HealthKit / Bios integration
- Voice calls (ElevenLabs)
- anima.hypostas.com live

**Sprint 1 completion criteria:**
- [ ] All 7 tests pass (with acceptable failure notes)
- [ ] DNS configured → anima.hypostas.com live
- [ ] First real conversation logged in production

---

*Anima Sprint 1: Days 1-6 complete. This is the finish line.*
