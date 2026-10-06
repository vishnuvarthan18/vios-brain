---
tags: chat
date: 2026-07-06
source: Claude personal account
uuid: 28ac1d07-1151-49d9-9bff-2a3a2456c373
---
# Wedding2day MVP launch strategy and tech stack

## Summary
**Conversation Overview**

The person is building a wedding vendor marketplace app called "Wedding2day" with a strict 20-day launch timeline targeting the Google Play Store. The project scope is a V1 MVP covering Firebase Authentication (phone OTP and Google Login), vendor profiles, listing creation with image uploads, a browsable home feed with filtering, listing detail views, and basic trust/safety features. The 20-day window is constrained by Google Play's 2026 policy requiring a 14-consecutive-day closed test with at least 12 opted-in testers before production access, leaving only 5 days for actual development.

The conversation focused heavily on tooling and stack decisions. The person initially proposed Flutter with Cursor IDE and Claude 3.5 Sonnet, then evolved toward React Native + Expo after recognizing two advantages: stronger AI code generation quality for React components and NativeWind styling compared to Flutter widget trees, and Expo's EAS cloud build service bypassing local Android SDK setup entirely. Claude confirmed React Native + Expo as the correct call for this specific project and constraint set, and provided critical hidden traps including EAS Build free-tier queue delays, the requirement for a custom dev client (not Expo Go) when using react-native-firebase with phone OTP, image compression pipeline needs before Firebase Storage writes, EAS Submit service account setup requirements, and native module version drift risks between react-native-firebase and the Expo SDK.

The person also asked for a detailed comparison of alternative tools including Claude Code, Google AI Studio, Fugu AI, and browser-based tools like Bolt and Replit. Claude recommended Claude Code as the strongest agentic option due to direct filesystem and terminal access, but confirmed Cursor remains a solid alternative. Google AI Studio was ruled out as a prototyping tool with no build pipeline, Fugu AI as unverified for native Android builds, and Bolt/Replit as incompatible with native Android compilation. A key strategic recommendation made throughout was to begin closed test tester enrollment in parallel with day one of development, not after build completion, due to 24–48 hour propagation delays.

## Chat

**Vishnu:** Objective: Launch the V1 MVP of "Wedding2day" (a wedding vendor marketplace) to the Google Play Store within a strict 20-day timeline.
1. The 20-Day Constraint (Google Play Compliance)
Because this is a new personal Google Play Developer account, we are bound by Google's 2026 testing policy: We must run a Closed Test with at least 12 opted-in testers for 14 consecutive days before applying for production access.

* Result: The actual development window for the MVP is strictly 5 days.
* Strategy: Build a "happy path" MVP that handles core logic and passes Google's testing requirements. Complex admin panels or payment gateways are deferred to V2.
2. V1 Product Scope (Features)

* Auth: Sign up / Log in via Phone OTP & Google Login.
* Profiles: Name, Business Name, User Type (Manufacturer/Decorator dropdown), District (Tamil Nadu), Phone Number.
* Supply (Create Listing): Form capturing Title, Category, Condition (Used/New), Photos (up to 5 max), Price, Quantity, District, and Description.
* Demand (Browse/Search): Full listing feed with basic filtering (Category, District, Condition, Price).
* Listing Detail: Single view showing all fields and an image carousel.
* Conversion (Trust/Safety):
   * "I'm Interested" button reveals the seller's phone/WhatsApp.
   * "Report Listing" button (Mandatory for Play Store AI/UGC compliance).
* Admin: No dedicated app. Manual moderation via the database console.
3. Proposed Tech Stack

* Frontend: Flutter (compiled to `.aab` for Android).
* Backend/Database: Firebase (Authentication, Firestore for NoSQL document storage, Cloud Storage for image hosting).
* UI/UX: Existing Figma designs are ready for implementation.
4. The Tooling Decision (Need Your Input)
I am planning to use Cursor IDE (with the Claude 3.5 Sonnet model) to write the Flutter code, moving away from no-code builders like FlutterFlow due to lack of codebase control, and avoiding browser-based tools (like Bolt/Replit) because they struggle with native Android compilation.

**Claude:** Cursor + Claude for Flutter is the right call — but Claude 3.5 Sonnet is outdated. Use **Claude Sonnet 4.5 or 4.6** if Cursor's model list has it; the jump in agentic coding and long-context file editing is significant for a Flutter codebase this size. Non-negotiable, not a preference.

## 5-Day Build Plan Risk

Your scope is realistic *if* Figma-to-Flutter isn't done from scratch. Biggest risks in order:

- **Firebase Auth Phone OTP** — Google's SMS Retriever/Play Integrity requirements have gotten stricter in 2026. Budget a full day just for OTP + Google Sign-In edge cases (existing account conflicts, OTP rate limits, emulator testing gaps).
- **Image upload to Cloud Storage** — 5-photo multi-upload with compression before upload is often underestimated. Do client-side compression (`flutter_image_compress`) before Storage write, or you'll blow past free-tier bandwidth fast.
- **Firestore query for filtering** — Category + District + Condition + Price range as compound filters needs composite indexes set up *before* testing starts, or queries silently fail. Set these up on day 1, not day 5.

## What to cut if day 5 runs long

- Image carousel → single hero image + thumbnail strip (cheaper than full carousel widget debugging)
- District list → hardcode Tamil Nadu districts as a static dropdown array, don't fetch from Firestore

## Sequencing correction

Start the **12-tester closed test enrollment in parallel with day 1 of dev**, not after build completion. Testers need to opt in via a Play Store link, and propagation/acceptance can take 24–48 hrs before the 14-day clock even starts cleanly. If you wait until day 5 to invite testers, you lose 1–2 days off your already-tight window.

**One thing to verify now, not later:** confirm your Google Play Console personal account is fully verified (identity + D-U-N-S if applicable) — new personal accounts have had verification delays reported in 2026 that stall closed testing enrollment entirely, independent of your 14-day clock.

Want the Firestore schema (collections/fields) and composite index list drafted next?

**Vishnu:** why code x, claud code in claude app, google ai stude, fugu ai, and other stuff cant do this

**Claude:** Because none of those tools solve for the actual bottleneck in your 20-day plan — Google Play's compliance clock and Flutter/Firebase integration correctness. What they're good at is generating code fast, not making your specific launch survive Play Store review or Firebase's runtime quirks.

Breaking down each:

- **Claude Code (CLI/app)** — Actually *could* do this well, arguably better than Cursor for a solo agentic build, since it operates directly on your filesystem with full project context and can run `flutter build`, `flutter test`, and Firebase CLI commands in the same loop. If you're choosing Cursor over Claude Code, that's the one worth reconsidering, not ruling out.
- **Google AI Studio** — Built for prototyping with Gemini models and testing prompts/API calls, not an IDE. No file system, no build tooling, no way to compile a `.aab`. Wrong category of tool entirely.
- **Fugu AI** — Niche/lesser-known tool, likely lacks the ecosystem maturity (extensions, terminal integration, multi-file refactor support) that a real Flutter + Firebase build needs under time pressure.
- **Bolt/Replit (browser-based)** — You already identified this correctly: no native Android SDK access, so `.aab` compilation for Play Store either doesn't work or requires exporting anyway, defeating the point of using them.

The real differentiator isn't the model — it's **whether the tool can execute a full native build pipeline** (Flutter SDK, Android SDK, Gradle, Firebase CLI, signing keys) locally or in a sandboxed environment that mirrors production. Cursor and Claude Code both qualify. AI Studio, Fugu, and browser IDEs don't.

**Recommendation stands: Cursor or Claude Code, not both** — pick one to avoid context-switching overhead during a 5-day sprint. If you want, I'll give you a one-line trade-off between the two for your specific setup.

**Vishnu:** i need  while detail comapreion of thses tools for this project

**Claude:** Claude Code wins for this project.

| Tool | Native build execution | Firebase CLI integration | Context handling for 5-day sprint | Verdict |
|---|---|---|---|---|
| **Claude Code (CLI/app)** | Direct filesystem + terminal access, runs `flutter build`, `flutter test`, `firebase deploy` in the same loop | Full CLI access — can init, configure rules, deploy indexes, manage Storage rules without leaving the tool | Best-in-class agentic context — reads your whole repo, tracks state across sessions, no re-explaining | **Use this** |
| **Cursor + Claude Sonnet 4.5/4.6** | Runs terminal commands inside IDE, same build access as Claude Code | Same CLI access, just routed through Cursor's terminal pane | Strong, but context window management is IDE-mediated — occasional need to re-point it at files | Solid alternative, redundant if using Claude Code |
| **Google AI Studio** | None — no filesystem, no terminal | None | Prompt/response only, no persistent project state | Wrong category — prototyping tool, not a build tool |
| **Fugu AI** | Unverified — no evidence of native Android SDK or Gradle execution support | Unverified | Unknown | Not worth the risk on a 5-day clock — no track record for this |
| **Bolt / Replit** | No native Android SDK/Gradle — can't produce a signed `.aab` without exporting | Partial, browser-sandboxed | Fine for web, wrong for native mobile | Ruled out — you already identified this correctly |

**Why Claude Code over Cursor specifically for you:**
- No IDE licensing/context-switch overhead
- Runs Firebase emulator suite and Flutter test commands natively without terminal-pane friction
- Better suited to agentic, multi-file Flutter refactors under time pressure since it holds full repo state without you re-selecting files

**Trade-off if you still prefer Cursor:** visual diffing and inline review are better in Cursor's UI. If you want that during a fast build, Cursor is fine — just don't run both in parallel, pick one.

**Vishnu:** I’m doing a 20-day sprint to build a V1 MVP for a wedding vendor marketplace (Wedding2day). It’s simple: Firebase Auth, profiles, listing creation with image uploads, and a home feed.
Since I need to move incredibly fast, I’m using Cursor + Claude Sonnet 5 to generate the code. I originally thought about using Flutter, but I’m leaning heavily toward React Native + Expo instead for two reasons:

1. The AI is much more accurate at generating React components and NativeWind/Tailwind styling than Flutter’s widget trees.
2. Expo’s cloud builds (EAS) let me bypass the local Android Studio setup so I can hit the Play Store’s 14-day testing requirement faster.
Given the 20-day constraint and my reliance on AI, do you agree React Native + Expo is the smartest path? Are there any hidden traps with this stack I should watch out for?

**Claude:** Agree. React Native + Expo is the better call here, not Flutter.

## Why it's right
- AI code generation quality for RN/NativeWind is genuinely stronger than Flutter widget trees — you're not wrong about that.
- EAS Build removes local Android SDK/Gradle setup entirely, which directly shortens your path to a signed `.aab` and into closed testing.

## Hidden traps

**EAS Build queue times** — Free tier builds queue behind paid users, sometimes 30–60+ min wait. On a 5-day dev window, budget for this or pay for priority builds ($). Don't discover this on day 4.

**OTP + Firebase Auth on Expo** — `react-native-firebase` needs a **custom dev client**, not Expo Go. You lose Expo Go's fast refresh loop for testing phone auth from day 1. Confirm your Cursor/Claude setup targets `expo-dev-client` + prebuild, not managed workflow assumptions.

**expo-image-picker + compression** — Default picker output isn't compressed. You'll need `expo-image-manipulator` in the upload pipeline before Firebase Storage writes, or you'll blow bandwidth same as the Flutter path.

**EAS Submit + Play Console verification** — Submission via EAS still requires your Play Console service account JSON key set up correctly. This is a common day-of-launch blocker — set it up on day 1, not day 5.

**Native module version drift** — `react-native-firebase` requires matching Expo SDK version precisely. Mismatches cause native build failures that AI-generated code won't catch, since it's a config/dependency issue, not a code issue. Lock Expo SDK version before any AI-generated code starts.

Worth verifying: confirm EAS free tier build minutes cover your iteration count for a 5-day sprint — if you're rebuilding native binaries frequently (any native module change), you may hit the monthly cap fast.
