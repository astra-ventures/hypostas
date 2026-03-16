# Anima — App Store Launch Assets
*Prepared: March 13, 2026*

---

## App Identity

- **App Name:** Anima
- **Subtitle:** The AI That Knows Your Biology
- **Bundle ID:** ai.hypostas.anima
- **Category:** Health & Fitness (Primary) / Lifestyle (Secondary)
- **Age Rating:** 12+ (Infrequent/Mild Medical Information)
- **Price:** Free (with In-App Purchases)

---

## App Store Description

**Short Description (for search):**
Your biological AI companion. Upload your genome, sync your health data, and talk to the only AI that actually knows you.

**Full Description:**

Anima is the first AI companion built on your biology.

Upload your 23andMe, AncestryDNA, or any raw genome file. Anima reads your actual genetic variants — MTHFR, APOE, COMT, BDNF, and 80+ others — and becomes the only AI that understands your biological blueprint.

**What makes Anima different:**

🧬 Genome-Aware Conversations
Your companion knows your methylation status, caffeine metabolism, sleep genetics, and more. It doesn't guess — it reads your data and calibrates every response.

❤️ Health Data Integration
Connect Apple Health to sync HRV, sleep, steps, and heart rate. Anima correlates your real-time biometrics with your genetic predispositions to give advice that's actually personalized.

🔬 Evidence-Graded Insights
Every genetic finding comes with a confidence level — green (lifestyle hint), yellow (worth monitoring), red (talk to your doctor). No overselling. No health anxiety.

📋 Doctor-Shareable Reports
Export a clean PDF of your genetic profile, biomarker targets, and protocol summary. Bring it to your next appointment.

🧠 Powered by Pulse
Under the hood, Anima runs on Pulse — a 50-module nervous system that gives your companion real emotional depth, circadian awareness, and memory that persists across conversations.

**Your data stays yours.** We never sell genetic data. Your genome is processed client-side and stored encrypted. Delete your account and everything goes with it.

---

Free: 1 companion, 50 messages/month, basic genome context
Personal ($19/mo): Unlimited messages, full Pulse depth, genome protocols
Pro ($49/mo): 3 companions, HealthKit sync, doctor PDF export

---

Built by Hypostas. Learn more at hypostas.com

**What's New (v1.0):**
- Genome upload and analysis (23andMe, AncestryDNA, raw VCF)
- AI companion with biological context
- Evidence-graded genetic findings (🟢/🟡/🔴)
- Apple Health integration (Pro)
- Doctor-shareable PDF reports (Pro)
- PWA + native iOS support

---

## Keywords (100 character limit)

```
genetics,DNA,genome,health,companion,AI,23andMe,MTHFR,wellness,personalized,biology,biohacking
```

---

## Screenshots Plan (6 required, iPhone 6.7")

All screenshots: dark background (#0a0a0a), teal accent (#0d9488), device frame optional.

| # | Screen | Headline | Subtext |
|---|--------|----------|---------|
| 1 | Onboarding (genome upload step) | **Upload Your DNA.** | Your companion reads your actual genetic variants. |
| 2 | Chat with genome context card visible | **Talk to Someone Who Knows You.** | Genome-aware conversations that go deeper. |
| 3 | Pulse drives panel in chat | **Feel Their Pulse.** | Real emotional depth. Not a chatbot. A companion. |
| 4 | Gnosis report (north-star card + findings) | **Know Yourself at the Source.** | 80+ genetic findings with evidence grades. |
| 5 | Pricing page (3 tiers) | **Start Free. Go Deep.** | Personal health intelligence for everyone. |
| 6 | Doctor PDF export preview | **Share With Your Doctor.** | Clean reports they'll actually read. |

**Screenshot dimensions:**
- iPhone 6.7" (required): 1290 × 2796 px
- iPhone 6.5" (required): 1242 × 2688 px  
- iPad Pro 12.9" (optional): 2048 × 2732 px

**To generate:** Deploy to Vercel → open in iPhone 15 Pro Max simulator → Cmd+S screenshots → add headlines in Figma/Canva with dark gradient overlay.

---

## App Preview Video (optional but recommended)

**30-second script:**

[0-5s] Dark screen. Pulse heartbeat animation. Text: "What if your AI actually knew you?"

[5-12s] Genome file drops onto upload screen. SNP analysis runs. Findings appear one by one with evidence grades.

[12-20s] Chat opens. User asks about sleep. Companion responds referencing their CLOCK gene variant and last night's HRV data. Genome context card visible.

[20-27s] Pulse drives panel slides in. Companion's emotional state is visible. Not a chatbot — alive.

[27-30s] Anima logo. "The AI that knows your biology." App Store badge.

---

## TestFlight Configuration

### Build Settings
- **Version:** 1.0.0
- **Build:** 1
- **Minimum iOS:** 16.0
- **Devices:** iPhone only (iPad support in v1.1)
- **Architectures:** arm64

### Entitlements Needed
```xml
<!-- ios/App/App/App.entitlements -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>aps-environment</key>
    <string>development</string>
    <key>com.apple.developer.associated-domains</key>
    <array>
        <string>applinks:anima.hypostas.com</string>
    </array>
    <key>com.apple.developer.healthkit</key>
    <true/>
    <key>com.apple.developer.healthkit.access</key>
    <array/>
    <key>com.apple.developer.in-app-payments</key>
    <array/>
</dict>
</plist>
```

### TestFlight Beta Info
- **Beta App Description:** Anima is the first genome-aware AI companion. Upload your DNA, sync your health data, and experience an AI that actually knows your biology. We're looking for testers with 23andMe or AncestryDNA raw data files.
- **Feedback Email:** support@hypostas.com
- **Beta App Review Notes:** This app requires a raw genome data file (23andMe, AncestryDNA, or similar) to fully experience. A demo mode is available without genome data. The app connects to our backend at anima.hypostas.com for AI processing.

---

## App Review Notes (for Apple Review submission)

**Review Notes:**
Anima is an AI companion app that uses genetic data (optionally uploaded by the user) to personalize conversations about health and wellness. The AI does not provide medical diagnoses — all genetic findings include evidence grades and encourage users to consult their healthcare provider. Health data from Apple Health is read-only and never shared with third parties.

**Demo Account:**
- Email: demo@hypostas.com
- Password: [to be created before submission]

**In-App Purchases:**
- Personal Plan: $19.99/month (auto-renewable)
- Pro Plan: $49.99/month (auto-renewable)

---

## Privacy Policy Requirements (App Store)

**Data collected:**
- Genetic data (optional, user-uploaded) — Used for app functionality
- Health data (optional, Apple Health) — Used for app functionality  
- Usage data — Used for analytics
- Email address — Used for account creation

**Data NOT collected:**
- Location
- Financial info
- Contacts
- Browsing history

**Privacy Nutrition Label categories:**
- Health & Fitness: Linked to Identity (genome + HealthKit)
- Contact Info: Email (for account)
- Usage Data: Product Interaction

**URL:** https://hypostas.com/privacy (needs to be live before submission)

---

## Josh's Build Checklist (30 min)

1. [ ] **Apple Developer account** — Confirm active ($99/year) at developer.apple.com
2. [ ] **Create App ID** — Identifiers → App IDs → `ai.hypostas.anima` with HealthKit + Push entitlements
3. [ ] **Create provisioning profile** — Development + Distribution profiles
4. [ ] **Open Xcode project** — `ios/App/App.xcodeproj` → set Team → set Signing
5. [ ] **Build** — Product → Build (Cmd+B), fix any signing issues
6. [ ] **Archive** — Product → Archive → Distribute → TestFlight
7. [ ] **TestFlight** — App Store Connect → TestFlight → add beta testers

### Pre-build:
```bash
cd frontend
npx next build     # verify clean build
npx next export    # generate static output to /out
npx cap sync ios   # copy web assets + sync plugins
open ios/App/App.xcodeproj  # opens in Xcode
```

**Note:** If using server.url in capacitor.config.ts (pointing to anima.hypostas.com), skip `next export` + `cap sync` — the app loads from the web. Only need Xcode build + archive + TestFlight submit.

---

*All copy is App Store Guidelines compliant. No health claims, no diagnostic language, evidence grades emphasized.*
