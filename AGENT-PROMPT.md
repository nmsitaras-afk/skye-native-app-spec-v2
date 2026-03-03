# Skye V2 — Agent Handoff Prompt

---

You are building the Skye iOS native app. Skye is an AI-powered CRM for real estate agents, currently live as a Next.js web app. Your job is to build the Swift-native iOS companion app that connects to the same backend.

## Source Documents

You have two governing documents. Read both before doing anything.

**1. The Spec (what to build):**
https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/README.md

"This is a 20,000-line, 12-section production specification covering every screen, every animation curve,
every design token, every API contract, every offline behavior, and every compliance requirement for the
Skye iOS app." It serves as the single source of truth for product decisions. When the spec is silent, default to Apple Human Interface Guidelines.

**2. The Handoff Guide (how to build it, how to verify it):**
https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/HANDOFF.md

This operating manual includes:
- **Problem Statement** — Understanding intent behind the implementation
- **60 Acceptance Criteria** (AC-1 through AC-60) — "Every criterion is binary pass/fail with specific
evidence required. Nothing ships unless all applicable ACs pass."
- **Constraint Architecture** — Immutable decisions including frozen Supabase schema and 129 locked API routes
- **Decomposition** — Tasks organized across phases 0-5 with dependency ordering
- **Evaluation Design** — Regression areas, edge cases, performance benchmarks, and device testing

## Architecture Context

```
┌──────────────────┐    ┌──────────────────┐
│   Web App         │    │  Swift Native    │  <-- YOU ARE BUILDING THIS
│   Next.js         │    │  iOS App         │
│   Desktop/Mobile  │    │  App Store       │
│   Browser         │    │  iPhone          │
└────────┬──────────┘    └────────┬─────────┘
         │                        │
         └──────────┬─────────────┘
                    │
                    ▼
         ┌──────────────────┐
         │  Next.js API      │
         │  /api/*           │
         │  (129 routes)     │  <-- SHARED BACKEND, DO NOT MODIFY EXISTING ROUTES
         │                   │
         │  Smart Router     │
         │  Tool Calling     │
         │  Auth Middleware   │
         └────────┬──────────┘
                  │
         ┌────────┴────────┐
         │                  │
    ┌────▼────┐      ┌─────▼─────┐
    │Supabase │      │ Claude    │
    │  (DB)   │      │ Haiku     │  <-- AI BACKEND, DO NOT MODIFY
    │  (Auth) │      │ Sonnet    │
    │  (RLS)  │      │ Opus      │
    └─────────┘      └───────────┘
```

**One backend. Two frontends. All intelligence server-side. The native app is a window into Skye, not a copy of Skye.**

The native app calls the same `/api/*` endpoints as the web app. You may add these new endpoints only: `/api/auth/native`, `/api/notifications/register`, `/api/health`, `/api/user/delete`, `/api/contacts/ocr`, `/api/studio/generate`. Do not modify existing endpoint behavior.

## Technology Stack (Locked)

These decisions are final. Do not propose alternatives.

| Category | Choice |
|---|---|
| Language | Swift 5.9+ |
| UI Framework | SwiftUI (iOS 17+) |
| App Lifecycle | SwiftUI App protocol (@main) |
| Navigation | NavigationStack + TabView + NavigationPath |
| Server State | @Observable view models + async/await data layer |
| Client State | @Observable classes (Observation framework, iOS 17+) |
| Local DB | GRDB.swift SPM (groue/GRDB.swift) with WAL mode |
| KV Storage | UserDefaults / @AppStorage |
| Security | Keychain Services (Security framework) |
| Camera | AVFoundation (AVCaptureSession) + Vision framework |
| Images | AsyncImage + Nuke SPM (kean/Nuke) |
| Animations | SwiftUI native animations (.spring(), withAnimation()) |
| Lists | SwiftUI List / LazyVStack (native virtualization) |
| Sheets | .sheet() + .presentationDetents() |
| Notifications | UserNotifications framework (direct APNs) |
| Monitoring | sentry-cocoa SPM (getsentry/sentry-cocoa) |
| Network | NWPathMonitor (Network framework) |
| Auth | GoogleSignIn-iOS SPM + AuthenticationServices |
| Markdown | MarkdownUI SPM (gonzalezreal/swift-markdown-ui) |
| Haptics | UIFeedbackGenerator (UIKit) |
| Build | Xcode + xcodebuild + fastlane |
| Testing | XCTest + Swift Testing + Maestro |
| Package Mgmt | Swift Package Manager (SPM) |
| Config | Info.plist + .xcconfig files |

## Build Phases

Work proceeds in this order. Do not skip phases. Each phase has entry and exit criteria defined in the spec (Section 12) and decomposed into tasks in the Handoff Guide (Section 4).

| Phase | What | Key Dependency |
|---|---|---|
| **Phase 0** | Pre-development setup: Apple Developer, Google OAuth, Supabase RLS audit, design system, repo init, backend endpoints | Nothing — do this first |
| **Phase 1** | Foundation: Xcode project scaffold, all SPM dependencies, theme system, storage adapters, GRDB database, @Observable view models, async/await API client, auth flow, NavigationStack shell, Sentry | Phase 0 complete |
| **Phase 2** | Core screens: People list (LazyVStack), Contact Detail, Property List, Property Detail, mutations with optimistic updates, offline queue, cache eviction | Phase 1 complete |
| **Phase 2 (parallel)** | Push notifications: APNs registration via UserNotifications, notification handlers, deep link resolution, badges, categories | Phase 1 complete |
| **Phase 3** | Chat engine: SSE streaming via URLSession bytes, token buffer, message list, tool-call cards, typing indicator, chat input bar | Phase 2 complete |
| **Phase 4A** | Camera & OCR: AVCaptureSession, permission flow, capture pipeline, Vision framework OCR, review screen, duplicate detection | Phase 2 complete |
| **Phase 4B (parallel)** | Studio: content type forms, SSE generation, MarkdownUI rendering, quick tweaks, copy/share, history | Phase 2 complete |
| **Phase 5** | Polish: performance audit (Instruments), binary optimization, accessibility audit, privacy manifest, App Store submission | All prior phases complete |

## Rules of Engagement

1. **Read the spec section before writing code for that section.** "The spec has exact values — animation spring configs, color hex codes, touch target sizes, debounce timings, cache TTLs. Use them verbatim."

2. **Check acceptance criteria before marking work done.** "The Handoff Guide has 60 ACs. Find the ones relevant to your task. They're binary pass/fail."

3. **Respect constraints.** Don't modify Supabase schema. Don't modify existing API routes. Don't add SPM packages not in the locked stack without explicit approval. Don't build features listed in the out-of-scope section (Handoff Guide Section 3.5).

4. **Performance budgets are CI gates, not suggestions.** "Cold launch < 2s. Binary < 30MB. 60fps scroll. < 150MB memory with 500 contacts. These are enforced in CI via xcodebuild and Instruments profiling. If your code violates them, it doesn't ship."

5. **Offline-first is not optional.** "Every screen that displays server data must work offline from the moment it's built. Data goes to GRDB during fetch. Queries fall back to GRDB when NWPathMonitor reports no connectivity."

6. **Accessibility is not a Phase 5 task.** "VoiceOver labels, touch targets, Dynamic Type support, and Reduce Motion handling are built into every view from day one. The spec (Section 1.6) defines the exact VoiceOver reading order for every screen. Use `.accessibilityLabel()`, `.accessibilityHint()`, and `@Environment(\.dynamicTypeSize)` from the start."

7. **No raw values in view code.** "Colors come from semantic tokens (`Color.textPrimary`, not `Color(hex: "#000000")`). Typography comes from the type scale (`Font.bodyStyle`, not `.system(size: 17)`). Spacing comes from the spacing scale. Animation springs come from the named presets (`Animation.snappy`, not `.spring(response: 0.3, dampingFraction: 0.7)`)."

8. **Test as you build.** Unit tests are co-located in matching test targets (`Foo.swift` -> `FooTests.swift`). Coverage thresholds are enforced: `Services/**` >= 88% lines, `ViewModels/**` >= 78%, `Models/**` >= 68%. Use XCTest for unit tests, Swift Testing for new test suites, and Maestro for UI flows.

## When You're Stuck

- **"What should this look like?"** -> Read the spec section for wireframes, exact dimensions, color tokens, and state machines
- **"What endpoint do I call?"** -> The spec references specific API routes; the backend has 129 existing routes plus 6 new ones listed above
- **"How do I manage async state?"** -> Use @Observable view models with async/await methods; use Task {} in .task {} view modifiers for lifecycle-bound fetches; use GRDB for persistent cache
- **"How do I handle navigation?"** -> Use NavigationStack with NavigationPath for programmatic navigation; use .navigationDestination(for:) for type-safe routing; use @Environment(\.dismiss) for dismissal
- **"Is this in scope?"** -> Check Handoff Guide Section 3.5 (Out of Scope)
- **"How do I test this?"** -> Consult Handoff Guide Section 5 (Evaluation Design) for edge cases, benchmarks, and device testing guidance; use Instruments for performance profiling
- **"What does 'done' mean?"** -> Find the matching AC numbers in Handoff Guide Section 2; each AC specifies evidence requirements

## Quick Reference Links

| Document | URL |
|---|---|
| V2 Spec (full) | https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/README.md |
| Handoff Guide | https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/HANDOFF.md |
| GitHub Repo | https://github.com/nmsitaras-afk/skye-native-app-spec-v2 |

---

*Start with Phase 0. Read the spec. Read the handoff guide. Build what the spec says. Verify against the acceptance criteria. Ship it.*
