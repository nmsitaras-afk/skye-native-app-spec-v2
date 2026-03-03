# Skye iOS — Swift Native Builder Brief

**Project:** Skye CRM iOS App (Swift Native / SwiftUI / iOS 17+)
**Repo:** `nmsitaras-afk/skye-native-app-spec-v2`
**Commit:** `9d75144`

---

## Documents — Read in This Order

### 1. Agent Prompt (Start Here)
**What it is:** Locked tech stack, rules of engagement, phase ordering, and "when you're stuck" guidance.
**Lines:** 150
**Link:** [AGENT-PROMPT.md](https://github.com/nmsitaras-afk/skye-native-app-spec-v2/blob/main/AGENT-PROMPT.md)
**Raw:** https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/AGENT-PROMPT.md

### 2. Handoff & Implementation Guide
**What it is:** Problem statement, 60 binary acceptance criteria (AC-1 through AC-60), constraint architecture, full task decomposition (70+ tasks across 6 phases), evaluation design (benchmarks, edge cases, device matrix, accessibility protocol, security checks).
**Lines:** 569
**Link:** [HANDOFF.md](https://github.com/nmsitaras-afk/skye-native-app-spec-v2/blob/main/HANDOFF.md)
**Raw:** https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/HANDOFF.md

### 3. Full Production Specification
**What it is:** The complete 12-section spec covering every screen, gesture, animation, token, API call, and edge case.
**Lines:** 26,375
**Link:** [README.md](https://github.com/nmsitaras-afk/skye-native-app-spec-v2/blob/main/README.md)
**Raw:** https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/README.md

| # | Section | Domain |
|---|---------|--------|
| 1 | UX Architecture & Interaction Design | Gestures, haptics, animations, skeletons, accessibility |
| 2 | Visual Design System | Color tokens, typography, spacing, shadows, dark mode |
| 3 | Chat Engine & AI Interface | SSE streaming, token buffering, tool-call cards, input bar |
| 4 | CRM Screens | Contact list, detail, properties, search, filters, swipe actions |
| 5 | Authentication & Security | Google OAuth, Keychain, biometric unlock, token lifecycle |
| 6 | Push Notifications | APNs-direct, deep links, badge management, categories |
| 7 | Performance & Infrastructure | Launch budgets, Instruments profiling, CI gates |
| 8 | Offline Architecture & State | GRDB, mutation queue, conflict resolution, cache eviction |
| 9 | Camera, OCR, Media & Studio | AVFoundation, Vision OCR, upload pipeline, Studio generation |
| 10 | App Store Compliance | PrivacyInfo.xcprivacy, nutrition labels, review prep |
| 11 | Testing Strategy & QA | XCTest, Swift Testing, Maestro E2E, CI enforcement |
| 12 | Build Roadmap | Phases 0-5, dependency graph, entry/exit criteria |

---

## Tech Stack (Locked)

| Layer | Technology |
|-------|-----------|
| Language | Swift 5.9+ |
| UI Framework | SwiftUI (iOS 17+) |
| Navigation | NavigationStack + TabView |
| State | @Observable classes (Observation framework) |
| Networking | URLSession + async/await |
| Local DB | GRDB.swift (SPM) |
| Secure Storage | Keychain Services (Security framework) |
| Camera | AVFoundation |
| OCR | Vision framework (VNRecognizeTextRequest) |
| Push | UserNotifications (APNs-direct) |
| Images | AsyncImage + Nuke (SPM) |
| Monitoring | sentry-cocoa (SPM) |
| Testing | XCTest + Swift Testing + Maestro |
| CI/CD | Xcode Cloud or GitHub Actions + fastlane |
| Package Manager | Swift Package Manager (SPM) |

---

## Phase Order

```
Phase 0  →  Phase 1  →  Phase 2  ──→  Phase 3   ──→  Phase 5
                     │              ├──→  Phase 4A  ──→
                     │              └──→  Phase 4B  ──→
                     └──→  Push Notifications (parallel) ──→
```

---

## Rules

1. iOS 17+ minimum. No UIKit unless SwiftUI has no equivalent.
2. All dependencies via SPM. No CocoaPods. No Carthage.
3. No raw hex colors in views — use semantic token extensions only.
4. Every screen implements all 7 states: IDLE, LOADING, LOADED, EMPTY, REFRESHING, ERROR, OFFLINE.
5. All touch targets >= 44x44pt.
6. Tokens in Keychain only. Never UserDefaults, never @AppStorage, never plist.
7. Offline edits queued in GRDB mutation table with idempotency keys.
8. 60 acceptance criteria are binary pass/fail. No partial credit.
