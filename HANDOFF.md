# Skye V2 Spec — Handoff & Implementation Guide (Swift Native)

**Purpose:** Operating manual defining problem, acceptance criteria, constraints, task decomposition, and quality evaluation for all builders, QA agents, and reviewers.

**Platform:** Swift 5.9+ / SwiftUI / iOS 17+ (native — no JavaScript runtime)

---

## Section 1: Problem Statement

### 1.1 The Core Problem

Real estate professionals cannot reliably access Skye's AI-powered CRM from mobile devices in a native, offline-capable format. The existing web application delivers degraded mobile experiences: no push notifications, no camera-based business card scanning, no offline contact access during property showings with spotty coverage, and no App Store presence.

### 1.2 Root Cause Analysis

| User Pain | Root Cause | Why It Exists |
|---|---|---|
| **"I can't check contacts without a signal"** | Web app requires active internet for every data fetch; no offline cache, local database, or mutation queue. | Desktop-first SPA architecture; offline support never architected. |
| **"I miss follow-up reminders—no notifications"** | Mobile Safari lacks push notification support on iOS; web app has no push infrastructure. | Push notifications require APNs integration and native app distribution via App Store. |
| **"Typing business card info manually is inefficient"** | No camera integration; browser cannot access device camera for OCR. | Native camera APIs required for professional scanning experience. |
| **"The app feels slow and sluggish on my phone"** | Mobile Safari rendering overhead; no native gesture handlers or platform-optimized list virtualization. | React DOM cannot match native UIKit/SwiftUI performance; native Swift compilation solves this. |
| **"I can't find Skye in the App Store"** | No native iOS app exists. | Native app development deferred; V2 addresses this gap. |
| **"I lose my place when switching between apps"** | Mobile Safari aggressively discards background tabs; no cross-session state persistence. | Web apps lack control over tab lifecycle; native app with persisted stores survives background kills. |

### 1.3 What Success Looks Like

A real estate agent at an open house with intermittent connectivity can:

1. Open Skye from home screen (native app, instant launch < 1.5 seconds)
2. View full contact list immediately from cache, even offline
3. Scan visitor's business card with auto-populated new contact creation
4. Ask AI assistant property questions and receive streamed responses
5. Receive push notification 30 minutes later reminding follow-up
6. Generate Instagram captions from Studio tab
7. Know offline edits sync when connectivity returns without data loss or conflicts

### 1.4 What This Spec Covers

12-section production specification covering:

| Section | Domain | Key Deliverables |
|---|---|---|
| 1 | UX Architecture & Interaction Design | Every gesture, haptic, animation, touch target, skeleton screen, accessibility rule |
| 2 | Visual Design System | Design tokens (colors, typography, spacing, shadows) for light/dark modes; WCAG AA compliance |
| 3 | Chat Engine & AI Interface | SSE streaming, token buffering, tool-call rendering, message state machine, typing indicators |
| 4 | CRM Screens | People list virtualization, contact detail, property cards, engagement health, search/filter |
| 5 | Authentication & Security | Google OAuth native flow, token lifecycle, secure storage, biometric unlock, OWASP Mobile Top 10 |
| 6 | Push Notifications | APNs-direct integration, notification types/payloads, deep link resolution, badge management |
| 7 | Performance & Infrastructure | Cold launch (<1.5s), app binary size (<30MB), native compilation, Xcode Cloud pipeline, CI gates |
| 8 | Offline Architecture & State | @Observable view models + async data layer, GRDB.swift mutation queue, conflict resolution, cache eviction |
| 9 | Camera, OCR, Media & Studio | AVFoundation + Vision framework, OCR pipeline, image processing, Studio generation, upload manager |
| 10 | App Store Compliance | Privacy manifest, nutrition labels, review prep, screenshot specs, rejection prevention |
| 11 | Testing Strategy & QA | Testing pyramid (60/25/15), XCTest/Swift Testing/E2E specs, CI enforcement, Maestro flows |
| 12 | Build Roadmap | Phases 0-5 with entry/exit criteria, dependency graph, task-level decomposition, verification |

---

## Section 2: Acceptance Criteria (AC-1 through AC-60)

### 2.1 Authentication (Section 5)

**AC-1:** Google OAuth sign-in completes end-to-end: user taps "Sign in with Google" → `ASWebAuthenticationSession` or Google Sign-In SDK presents account picker → user selects account → app receives tokens → user lands on People tab. *Evidence:* Screen recording on physical device showing full flow.

**AC-2:** Access token is stored in iOS Keychain via Security framework with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` protection level. Token is NOT stored in `UserDefaults`, `@AppStorage`, or any unencrypted store. *Evidence:* Code review of auth service confirming Keychain usage; Keychain Sharing entitlement audit.

**AC-3:** Token refresh happens silently when access token expires. User is never shown a login screen due to token expiry during active use. *Evidence:* Test—set token TTL to 30 seconds, use app for 2 minutes, verify no auth interruption.

**AC-4:** After 401 response, app attempts one silent token refresh. If refresh fails, user is redirected to login with all local state cleared. *Evidence:* API mock returning 401, verify refresh attempt, then mock refresh failure, verify redirect + state clear.

**AC-5:** Biometric unlock (Face ID / Touch ID) gates app access after 5+ minutes of inactivity. Biometric prompt uses `LAContext` from LocalAuthentication framework. *Evidence:* Physical device test—background app for 6 minutes, foreground, verify biometric prompt appears before content visibility.

**AC-6:** Account deletion is accessible from Settings and completes via `POST /api/user/delete`. All local data (Keychain, UserDefaults, GRDB database, image cache) is wiped. *Evidence:* Test—delete account, verify Keychain empty, UserDefaults cleared, GRDB database deleted, login screen shown.

### 2.2 Navigation & UX (Sections 1, 4)

**AC-7:** Tab bar has exactly 5 tabs: Chat, People, Properties, Studio, Settings. Tab icons use SF Symbols specified in Section 2. Active tab uses `brand.primary` tint. Implementation uses SwiftUI `TabView`. *Evidence:* Visual comparison against spec.

**AC-8:** Every screen implements all 7 states from the universal state machine: IDLE, LOADING, LOADED, EMPTY, REFRESHING, ERROR, OFFLINE. *Evidence:* Screenshot matrix showing each state for each screen.

**AC-9:** Skeleton screens match the exact layout dimensions of their loaded counterparts. No blank white screens during loading. *Evidence:* Side-by-side comparison of skeleton vs. loaded states.

**AC-10:** All touch targets are >= 44x44pt. No interactive element has a smaller hit area. *Evidence:* Accessibility Inspector audit on every screen.

**AC-11:** Edge swipe back (iOS interactive pop) works on every `NavigationStack`-pushed screen. Gesture response distance is 20pt from left edge. *Evidence:* Physical device test on every pushed screen.

**AC-12:** Pull-to-refresh uses the branded Skye animation (not default `.refreshable`). Animation plays at 60fps. Minimum display time is 600ms. *Evidence:* Screen recording at 60fps showing branded animation; slow-motion verification of minimum duration.

**AC-13:** Tab switching preserves full stack state, scroll position, search query, and filter state. Keyboard is dismissed on tab switch. *Evidence:* Test—apply filters on People tab, scroll down, switch to Chat, switch back—all state preserved, keyboard dismissed.

**AC-14:** All bottom sheets use `.sheet()` with `.presentationDetents()` at specified snap points, with a grab handle via `.presentationDragIndicator(.visible)` and swipe-to-dismiss enabled. Backdrop dismisses on tap. *Evidence:* Component audit + physical device testing of every sheet instance.

### 2.3 Chat Engine (Section 3)

**AC-15:** Chat messages stream token-by-token via SSE at 60fps with no frame drops. Token buffer drains on `MainActor` using Swift `AsyncSequence`. *Evidence:* Xcode Instruments Time Profiler showing main thread >= 58fps during streaming.

**AC-16:** Chat input expands from 1 line (36pt) to max 5 lines (120pt) as user types, then enables internal scroll. Border radius adjusts from 20pt to 12pt at 3+ lines. *Evidence:* Physical device test with multi-line input.

**AC-17:** "Stop Generating" button appears during streaming. Tapping it immediately cancels the `URLSession` stream task and displays the partial response. *Evidence:* Test—send message, tap stop mid-stream, verify partial response is preserved and connection is terminated.

**AC-18:** Tool-call events render inline UI cards (e.g., contact lookup shows a contact card within the chat). Tool results are visually distinct from text responses. *Evidence:* Send a message that triggers a tool call, verify inline card renders correctly.

**AC-19:** Typing indicator (3 bouncing dots) appears within 100ms of sending a message and disappears when streaming begins. *Evidence:* Screen recording with frame timing.

**AC-20:** Chat history uses cursor-based pagination. Scrolling up loads older messages without losing scroll position via `ScrollViewReader` or `UICollectionView` bridge with `maintainVisibleContentPosition`. *Evidence:* Test—scroll up in a conversation with 100+ messages, verify no scroll jump on page load.

### 2.4 CRM Screens (Section 4)

**AC-21:** Contact list scrolls at 60fps with 500+ contacts using SwiftUI `List` / `LazyVStack`. No frame drops during fast scroll. *Evidence:* Xcode Instruments Core Animation FPS: main thread >= 58fps.

**AC-22:** Contact list rows show: avatar (44pt circle with initials fallback), name, company, last activity, engagement health dot (color-coded by recency), and quick action icons (phone/message). *Evidence:* Visual comparison against Section 4 wireframe.

**AC-23:** Engagement health dot colors match spec: Red (≤7 days), Amber (8-30 days), Blue (31-90 days), Gray (91+ days). Red dot has pulsing animation. *Evidence:* Test with contacts at each threshold; verify animation on red dot.

**AC-24:** Contact search debounces at 300ms, filters locally first, then hits API if local results < 5 and query length >= 3. Empty results show illustration + "No results for '[query]'" message. *Evidence:* Test—type a search query, verify debounce timing, verify empty state rendering.

**AC-25:** Swipe-left on contact row reveals Call (green) and Delete (red) action buttons. Swipe-right reveals Message (blue). Full swipe (>180pt) triggers primary action. *Evidence:* Physical device test with velocity and distance variations.

**AC-26:** Contact detail screen loads progressively: header first (from list cache), then activity timeline, linked properties, and notes—each with independent skeleton states. *Evidence:* Network throttling test showing progressive section loading.

### 2.5 Offline Architecture (Section 8)

**AC-27:** With airplane mode enabled, the app displays cached contact list, contact details, and property list from GRDB.swift-persisted cache. No crash, no blank screens. *Evidence:* Physical device test—load data online, enable airplane mode, navigate all cached screens.

**AC-28:** Edits made offline (edit contact, add note, create task) are queued in the `mutation_queue` GRDB table with status `pending`. Queue survives app kill and restart. *Evidence:* Test—edit contact offline, force-kill app, relaunch, verify mutation exists in GRDB.

**AC-29:** When connectivity returns, the mutation queue processes in FIFO order. Each mutation includes an `Idempotency-Key` header. Successfully synced mutations show brief green "synced" badge. *Evidence:* Test—queue 3 mutations offline, restore connectivity, verify FIFO processing order and idempotency headers.

**AC-30:** Conflict resolution: 409 responses auto-resolve scalar fields via last-write-wins. Complex fields (deals) surface a merge UI. Dead-lettered mutations (404, 403) show error badge with dismiss option. *Evidence:* Mock server returning 409 and 404, verify auto-resolution and dead-letter handling.

**AC-31:** Network state banner appears within 2 seconds of connectivity loss: "You're offline—showing cached data" with pending mutation count. Banner dismisses within 2 seconds of restoration. *Evidence:* Toggle airplane mode, time banner appearance and dismissal.

**AC-32:** Cache eviction fires when data cache exceeds 50MB. LRU eviction based on `accessed_at` column. Contacts with pending mutations are protected from eviction. *Evidence:* Fill cache to >50MB, verify eviction runs, verify protected entities survive.

### 2.6 Camera & OCR (Section 9)

**AC-33:** Camera permission flow shows pre-permission screen with Skye branding BEFORE the iOS system prompt. "Not Now" dismisses without triggering the system prompt. *Evidence:* Screen recording showing branded screen before system alert.

**AC-34:** Camera viewfinder opens in < 500ms from tap. Uses `AVCaptureSession` with overlay showing business card frame guide (3.5:2 aspect ratio) with animated corner brackets. *Evidence:* Stopwatch measurement from tap to live viewfinder.

**AC-35:** Captured image is processed (orientation-corrected, resized to max 1920px longest edge, JPEG 85% quality) and is < 5MB. Image is never saved to camera roll. *Evidence:* Assert file size after processing; verify camera roll is empty after scan.

**AC-36:** OCR review screen shows all extracted fields with confidence indicators: green checkmark (>=0.85), yellow warning (0.60-0.84), red warning (<0.60). All fields are editable. Uses Vision framework `VNRecognizeTextRequest`. *Evidence:* Test with business card image, verify confidence indicators match response values.

**AC-37:** Duplicate contact detection: if OCR creates a contact with an existing email/phone, a "Contact May Already Exist" screen appears with options: View Existing, Create Anyway, Merge. *Evidence:* Mock 409 response from create endpoint, verify screen renders with all 3 options.

**AC-38:** Studio generates content via SSE streaming with real-time token rendering. Quick tweak chips ("Make it Shorter", "More Professional", etc.) trigger regeneration. Copy button copies plain text to clipboard via `UIPasteboard`. *Evidence:* Test—generate Instagram caption, apply tweak, copy result, paste to verify content.

### 2.7 Push Notifications (Section 6)

**AC-39:** Push notification permission flow shows branded pre-permission screen before the iOS system prompt. Screen explains value ("Never miss a follow-up"). *Evidence:* Screen recording showing branded screen.

**AC-40:** Device token is registered via `POST /api/notifications/register` with token, platform, and device metadata. Token is re-registered on every app launch. Uses `UNUserNotificationCenter` and `UIApplication.shared.registerForRemoteNotifications()`. *Evidence:* Network log showing registration call on launch.

**AC-41:** Tapping a push notification deep-links to the correct screen (contact detail, chat conversation, task detail). If the target resource doesn't exist, fallback screen shows with toast. *Evidence:* Test each notification type: tap → verify correct screen. Test with invalid resource ID → verify fallback.

**AC-42:** Badge count on app icon reflects unread notifications. Badge clears when user opens the relevant tab. Uses `UNUserNotificationCenter.setBadgeCount()`. *Evidence:* Test—receive notifications, verify badge count on icon. Open tab, verify badge clears.

**AC-43:** Notification categories with action buttons work: "Reply" and "Mark as Read" on chat notifications, "Call Back" on contact reminders. Uses `UNNotificationCategory` and `UNNotificationAction`. *Evidence:* Test—long-press notification in Notification Center, verify action buttons appear and function.

### 2.8 Performance (Section 7)

**AC-44:** Cold launch to interactive < 1,500ms on iPhone 13 (P95). Measured from process start to first frame accepting touch input. *Evidence:* Xcode Instruments App Launch trace showing P95 metric.

**AC-45:** Compressed app binary size < 30MB. CI gate fails build if binary exceeds 35MB. *Evidence:* Xcode Archive output showing binary size; CI pipeline screenshot showing gate.

**AC-46:** App memory usage < 150MB with 500 contacts loaded in a scrolling list. No memory leaks after navigating to/from a screen 10 times. *Evidence:* Xcode Instruments Allocations trace; Leaks trace after 10 navigation cycles.

**AC-47:** All scrollable lists maintain >= 60fps during fast scroll (native SwiftUI — trivially achievable). *Evidence:* Xcode Instruments Core Animation FPS during scroll stress test.

**AC-48:** Native Swift/LLVM compilation is verified: no embedded JavaScript engine or interpreter in the final binary. *Evidence:* Binary inspection confirming pure native Swift compilation.

### 2.9 Visual Design (Section 2)

**AC-49:** Light mode and dark mode are fully implemented. All semantic color tokens match spec values. Dark mode uses true black (`#000000`) for OLED. Theme switches with system setting via `@Environment(\.colorScheme)`. *Evidence:* Screenshot comparison against spec token table for both modes on OLED device.

**AC-50:** Typography uses SF Pro with the exact scale from Section 2: Display (34pt Bold), Title (28pt Bold), Headline (22pt Semibold), Body (17pt Regular), Caption (13pt Regular). Uses SwiftUI `Font` system. *Evidence:* Font inspector audit on every text element.

**AC-51:** All color combinations meet WCAG AA contrast ratio: 4.5:1 for body text, 3:1 for large text, 3:1 for interactive elements. *Evidence:* Automated contrast ratio check using Accessibility Inspector.

**AC-52:** Reduce Motion is respected: all spring animations degrade to crossfade or instant state change. Skeleton shimmer becomes static. No motion-sensitive user is subjected to animation. Checked via `@Environment(\.accessibilityReduceMotion)`. *Evidence:* Enable Reduce Motion in iOS Settings, navigate all screens, verify no spring/bounce/parallax animations.

### 2.10 App Store Compliance (Section 10)

**AC-53:** `PrivacyInfo.xcprivacy` declares all required-reason APIs: FileTimestamp (C617.1), SystemBootTime (35F9.1), DiskSpace (E174.1), UserDefaults (CA92.1). `NSPrivacyTracking` is `false`. *Evidence:* Xcode Privacy Report generation; manual XML review.

**AC-54:** Privacy nutrition labels in App Store Connect accurately declare: Contact Info (linked), Identifiers (linked), Usage Data (linked), Diagnostics (not linked). No under-declaration. *Evidence:* Comparison of App Store Connect entries against Section 10 table.

**AC-55:** App includes in-app account deletion accessible from Settings. Deletion cascade removes all user data from Supabase, Keychain, UserDefaults, GRDB database, and image cache. *Evidence:* Test—create account, add data, delete account, verify all storage backends are empty.

**AC-56:** Demo account credentials are prepared for App Review (email + password) with pre-populated data (contacts, properties, chat history). Review notes explain AI features and camera usage. *Evidence:* Verify demo account has data; read review notes for completeness.

**AC-57:** App does not crash on any supported device (iPhone SE 3rd Gen through iPhone 16 Pro Max). All screens render correctly on 375pt (SE) through 430pt (Pro Max) widths. *Evidence:* TestFlight testing on minimum—iPhone SE, iPhone 15, iPhone 16 Pro Max.

### 2.11 Testing & CI (Section 11)

**AC-58:** Unit test coverage meets thresholds: `Sources/Lib/**` >= 88% lines, `Sources/ViewModels/**` >= 78% lines, `Sources/Services/**` >= 68% lines. CI fails below these via `xcodebuild -enableCodeCoverage YES`. *Evidence:* XCTest/Swift Testing coverage report from CI pipeline.

**AC-59:** E2E tests cover 5 critical flows via Maestro: (1) sign-in, (2) browse contacts, (3) view contact detail, (4) send chat message, (5) scan business card. All pass on CI. *Evidence:* Maestro test results from CI run.

**AC-60:** Accessibility audit passes: all screens navigable via VoiceOver in correct order, all touch targets >= 44pt, Dynamic Type scales without clipping at AX3 size. *Evidence:* VoiceOver screen recording on physical device; Accessibility Inspector report.

---

## Section 3: Constraint Architecture

### 3.1 Backend Constraints (DO NOT MODIFY)

Supabase schema is frozen — no new tables, column renames, or type changes. RLS policies remain unchanged; the app authenticates via JWT `auth.uid()`. All 129 existing API routes are immutable. New endpoints are limited to exactly these six:

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/auth/native` | POST | Google OAuth credential exchange for native clients |
| `/api/notifications/register` | POST | Device token registration for APNs |
| `/api/health` | POST | Liveness/readiness check for CI pipelines |
| `/api/user/delete` | POST | Account deletion cascade (user-initiated) |
| `/api/contacts/ocr` | POST | Business card OCR processing pipeline |
| `/api/studio/generate` | POST | Content generation via SSE streaming |

All responses must be JSON. No HTML error pages. If existing routes return HTML on error, they must be fixed server-side — the native app cannot work around RLS or API incompatibilities.

### 3.2 Technology Constraints (LOCKED)

| Technology | Choice | Reversal Cost |
|---|---|---|
| **Language & Runtime** | Swift 5.9+ / SwiftUI / iOS 17+ | Irreversible after Phase 1 |
| **Compilation** | Native Swift/LLVM compilation (no JS engine) | Irreversible |
| **Navigation** | `NavigationStack` + `TabView` | High cost post-Phase 1 |
| **Server State** | `@Observable` view models with async data layer (offlineFirst) | Medium cost |
| **Client State** | `@Observable` classes (Observation framework) | Low cost |
| **Local DB** | GRDB.swift SPM (groue/GRDB.swift, WAL mode) | Medium cost |
| **Secure Storage** | Keychain Services (Security framework) | Low cost |
| **Fast KV Store** | `UserDefaults` / `@AppStorage` | Low cost |
| **Camera** | AVFoundation + Vision framework | Medium cost |
| **Image Loading** | `AsyncImage` + Nuke SPM (kean/Nuke) | Low cost |
| **Animations** | SwiftUI native animations + gesture modifiers | Irreversible post-Phase 1 |
| **List Virtualization** | SwiftUI `List` / `LazyVStack` | Low cost |
| **Bottom Sheets** | `.sheet()` + `.presentationDetents()` | Low cost |
| **Push Notifications** | APNs-direct (server) + UserNotifications framework (client) | Medium cost |
| **Error Monitoring** | sentry-cocoa SPM (getsentry/sentry-cocoa) | Low cost |

### 3.3 Performance Budgets (CI-ENFORCED GATES)

| Metric | Target Budget | Hard Ceiling (CI Fails) |
|---|---|---|
| Cold launch to interactive (P95, iPhone 13) | < 1,500ms | 2,000ms |
| Compressed app binary | < 30MB | 35MB |
| App memory (500 contacts loaded) | < 150MB | 200MB |
| Main thread FPS during scroll | >= 60fps | 58fps |
| Main thread FPS during animation | >= 58fps | 55fps |
| Data cache (GRDB) | < 50MB | 50MB (triggers eviction) |
| Image cache (disk) | < 100MB | 100MB (triggers eviction) |
| Time to first meaningful paint | < 500ms | 800ms |

> **Note:** With native Swift compilation, there is no separate JS thread. All work runs on the main thread and structured concurrency actors. The "JS thread FPS" metric from React Native is replaced by main thread profiling via Xcode Instruments.

### 3.4 Design Constraints

| Constraint | Detail |
|---|---|
| **Orientation** | Portrait-only; locked in `Info.plist`. No landscape in V2. |
| **iPad Support** | iPhone compatibility mode (letterboxed). No native iPad layouts. |
| **iOS Minimum** | iOS 17+. Required for Observation framework, `@Observable`, and modern SwiftUI APIs. |
| **Device Range** | iPhone SE 3rd Gen through iPhone 16 Pro Max (375–430pt width). |
| **Accessibility Floor** | WCAG AA minimum; VoiceOver full support; Dynamic Type to AX5 (capped 2x multiplier); Reduce Motion respected on all animations. |
| **Color System** | Three-layer token architecture (primitive → semantic → component) via SwiftUI `Color` extensions and `ShapeStyle`; no raw hex in view code. |

### 3.5 Explicit Out of Scope (V2)

- Android app (V3 consideration)
- iPad-native layouts (iPhone compatibility mode only)
- Landscape orientation / media viewer
- Real-time WebSocket sync (poll-based staleTime sufficient)
- Offline AI chat (responses require server)
- Offline image upload (queue not reliable for large files)
- Apple Watch companion app
- iOS home screen widgets
- Siri Shortcuts integration
- In-app purchases (web-app billing only)
- Bulk import/export from native app
- Voice input in chat
- Video calling

---

## Section 4: Decomposition

### 4.1 Phase Dependency Graph

```
Phase 0: Pre-Development Setup
    ↓
Phase 1: Foundation
    (Auth, API Client, State, Theming, Navigation Shell)
    ↓
    ├──→ Phase 2: Core Screens
    │    (People List, Contact Detail, Property List/Detail)
    │    ↓
    │    ├──→ Phase 3: Chat Engine
    │    │    (SSE streaming, messages, tool calls, input)
    │    │
    │    ├──→ Phase 4A: Camera & OCR
    │    │    (AVFoundation, capture, Vision OCR, review)
    │    │
    │    └──→ Phase 4B: Studio
    │         (Content types, generation, history, tweaks)
    │
    └──→ Phase 2 (Parallel): Push Notifications
         (APNs registration, handlers, deep links, badges)

    ↓
Phase 5: Polish, Performance, Compliance, Submission
    (Audits, App Store prep, TestFlight, submission)
```

**Critical Path:** Phase 0 → Phase 1 → Phase 2 → Phase 3/4A/4B (parallel) → Phase 5

**Parallelizable after Phase 1:** Phases 2, 2-parallel (PN), 3, 4A, and 4B can overlap, with data dependencies respected (e.g., Phase 3 awaits Phase 1's auth/data infrastructure).

### 4.2 Phase 0 — Pre-Development Setup

| Task | Deliverable |
|---|---|
| **0.1** | Activate Apple Developer Program, create App ID (`com.skye.crm`) with Push Notifications capability |
| **0.2** | Create Google Cloud OAuth iOS + web client IDs |
| **0.3** | Audit Supabase RLS policies for mobile JWT compatibility |
| **0.4** | Finalize design system in Figma (tokens, type scale, spacing, components) |
| **0.5** | Lock Architecture Decision Records (ADR-1 through ADR-5) |
| **0.6** | Provision dev environments (Xcode 15+, Swift 5.9+, physical devices, provisioning profiles) |
| **0.7** | Initialize Xcode project (`.xcodeproj`, SPM dependencies, SwiftLint config, `.gitignore`, CI stub) |
| **0.8** | Create Sentry project and obtain DSN |
| **0.9** | Implement `/api/health` endpoint |
| **0.10** | Implement `/api/auth/native` endpoint (Google OAuth code exchange) |
| **0.11** | Implement `/api/user/delete` endpoint (account deletion cascade) |
| **0.12** | Audit all 129 API routes for JSON-only responses |

### 4.3 Phase 1 — Foundation

| Task | Deliverable |
|---|---|
| **1.1** | Create Xcode project structure; configure `Info.plist`, build settings, schemes (Debug/Release) |
| **1.2** | Add SPM dependencies (GRDB.swift, Nuke, sentry-cocoa) and configure project targets |
| **1.3** | Implement design token system (`Colors.swift`, `Typography.swift`, `Spacing.swift`, `Shadows.swift`, `Animations.swift`) as SwiftUI extensions |
| **1.4** | Implement storage adapters (`KeychainService.swift`, `UserDefaultsService.swift`) |
| **1.5** | Implement GRDB database setup (`DatabaseManager.swift`, `Migrations.swift`, schema v1) |
| **1.6** | Implement async data layer (`DataRepository.swift`, `CachePolicy.swift`, `QueryKeys.swift`, GRDB-backed persistence) |
| **1.7** | Implement `@Observable` state classes (`AuthState.swift`, `UIState.swift`, `ChatState.swift`, `OfflineState.swift`, `NotificationState.swift`) |
| **1.8** | Implement API client (`APIClient.swift` with auth header injection, 401 refresh interceptor, error normalization using `URLSession`) |
| **1.9** | Implement auth flow (`AuthViewModel.swift`, `LoginView.swift`, Google OAuth via `ASWebAuthenticationSession`) |
| **1.10** | Implement network monitor (`NetworkMonitor.swift` using `NWPathMonitor`) |
| **1.11** | Implement tab navigator shell (`ContentView.swift` with `TabView`, 5 tabs: Chat/People/Properties/Studio/Settings, `NavigationStack` per tab) |
| **1.12** | Implement network banner (`NetworkBannerView.swift` with offline/online states) |
| **1.13** | Implement Sentry integration (`SentrySetup.swift` configuration) |
| **1.14** | Configure Xcode Cloud or GitHub Actions + fastlane build profiles (dev, TestFlight, production) |
| **1.15** | Write unit tests for API client, data layer, state classes, storage adapters using XCTest + Swift Testing |
| **1.16** | First successful build to physical device via Xcode |

### 4.4 Phase 2 — Core Screens

| Task | Deliverable |
|---|---|
| **2.1** | Implement People list with `List`/`LazyVStack`, alphabetical sections, health dots |
| **2.2** | Implement contact search with 300ms debounce (using `Task` + `try await Task.sleep`), local-first + API fallback |
| **2.3** | Implement contact filters bottom sheet using `.sheet()` + `.presentationDetents()` |
| **2.4** | Implement swipe actions on contact rows (`.swipeActions`) — Call, Delete, Message |
| **2.5** | Implement Contact Detail screen with progressive loading sections |
| **2.6** | Implement Edit Contact sheet |
| **2.7** | Implement Add Contact full-screen modal (`.fullScreenCover()`) |
| **2.8** | Implement Add Note sheet |
| **2.9** | Implement Property List with cards (hero image via `AsyncImage`/Nuke + details) |
| **2.10** | Implement Property Detail screen (hero image, Street View, linked contacts) |
| **2.11** | Implement image components (`ContactAvatarView`, `StreetViewImage` with blurhash placeholder) |
| **2.12** | Implement mutation methods in view models (`editContact`, `createContact`, `deleteContact`, `addNote` with optimistic updates) |
| **2.13** | Implement offline mutation queue (`MutationQueue.swift`, `QueueProcessor.swift` using GRDB) |
| **2.14** | Implement cache eviction (`CacheEviction.swift`, LRU by `accessed_at`) |
| **2.15** | Implement pull-to-refresh branded animation (custom `RefreshableModifier`) |
| **2.16** | Write view model and mutation tests for all CRM screens |

### 4.5 Phase 3 — Chat Engine

| Task | Deliverable |
|---|---|
| **3.1** | Implement SSE streaming engine (`SSEClient.swift`: connection state machine, token parser, buffer drain via `AsyncSequence`) |
| **3.2** | Implement chat session list screen |
| **3.3** | Implement chat conversation screen (reversed `ScrollView`/`LazyVStack`, bubbles, timestamps) |
| **3.4** | Implement chat input bar (expanding `TextField`/`TextEditor`, send button, attachment button) |
| **3.5** | Implement typing indicator (3 bouncing dots via SwiftUI animation) |
| **3.6** | Implement tool-call inline cards (contact card, property card within chat) |
| **3.7** | Implement "Stop Generating" button with `URLSessionDataTask` cancellation and partial response preservation |
| **3.8** | Implement chat history pagination (cursor-based, inverted scroll) |
| **3.9** | Implement offline chat message queuing with connectivity warning |
| **3.10** | Write tests: SSE parser, message state transitions, streaming performance |

### 4.6 Phase 4A — Camera & OCR

| Task | Deliverable |
|---|---|
| **4A.1** | Implement camera permission flow with branded pre-permission screen (check `AVCaptureDevice.authorizationStatus`) |
| **4A.2** | Implement camera viewfinder with `AVCaptureSession` + overlay frame (3.5:2 guide, corner bracket animation) via `UIViewRepresentable` |
| **4A.3** | Implement photo capture pipeline (`AVCapturePhotoOutput`, orientation correction, resize, JPEG compression) |
| **4A.4** | Implement OCR preview screen |
| **4A.5** | Implement OCR upload with progress using `URLSession` upload task with delegate |
| **4A.6** | Implement OCR review screen with confidence indicators (Vision `VNRecognizedText` confidence) and editable fields |
| **4A.7** | Implement OCR success/error/duplicate result states |
| **4A.8** | Implement `/api/contacts/ocr` backend endpoint |
| **4A.9** | Write tests for image processing, upload manager, full OCR flow |

### 4.6 Phase 4B — Studio

| Task | Deliverable |
|---|---|
| **4B.1** | Implement Studio home screen with content type selector chips |
| **4B.2** | Implement context input forms per content type (Instagram, Email, Listing, Market Update, Open House) |
| **4B.3** | Implement SSE generation flow with streaming output and markdown rendering (using `AttributedString`) |
| **4B.4** | Implement quick tweak chips with regeneration |
| **4B.5** | Implement copy/share/regenerate action buttons |
| **4B.6** | Implement generation history (GRDB-backed, last 100 entries) |
| **4B.7** | Implement `/api/studio/generate` backend SSE endpoint |
| **4B.8** | Write tests for all Studio flows |

### 4.7 Phase 2 (Parallel) — Push Notifications

| Task | Deliverable |
|---|---|
| **PN.1** | Implement notification permission flow with branded pre-permission screen |
| **PN.2** | Implement APNs device token registration via `UIApplication.shared.registerForRemoteNotifications()` |
| **PN.3** | Implement `/api/notifications/register` backend endpoint |
| **PN.4** | Implement notification handler (`UNUserNotificationCenterDelegate`: foreground display, background processing, tap with deep link resolution) |
| **PN.5** | Implement deep link resolution engine (`skye://` URL scheme, `onOpenURL` modifier, auth gate, resource validation) |
| **PN.6** | Implement badge management (`UNUserNotificationCenter.setBadgeCount`: set, increment, clear per tab) |
| **PN.7** | Implement notification categories (`UNNotificationCategory`) with action buttons (`UNNotificationAction`) |
| **PN.8** | Implement backend push send service (APNs-direct via @parse/node-apn) |
| **PN.9** | Write tests for notification handling, deep link resolution, badge management |

### 4.8 Phase 5 — Polish, Performance, Compliance, Submission

| Task | Deliverable |
|---|---|
| **5.1** | Performance audit (cold launch, memory, scroll profiling on iPhone 13 via Xcode Instruments) |
| **5.2** | Binary size audit and optimization (remove unused assets, dead code stripping, link-time optimization) |
| **5.3** | Accessibility audit (VoiceOver order, Dynamic Type AX3, 44pt targets via Accessibility Inspector) |
| **5.4** | Build `PrivacyInfo.xcprivacy` with required-reason API declarations (FileTimestamp C617.1 / SystemBootTime 35F9.1 / DiskSpace E174.1 / UserDefaults CA92.1) |
| **5.5** | Prepare App Store Connect (screenshots 6.7"/6.5"/5.5", promo text, keywords, categories) |
| **5.6** | Prepare demo account with pre-populated test data |
| **5.7** | Write App Review notes (AI features, camera, data handling) |
| **5.8** | Final E2E test suite run via Maestro on all 5 critical flows |
| **5.9** | TestFlight distribution (minimum 3 device variants: SE + standard + Pro Max) |
| **5.10** | Submit to App Store Review |

---

## Section 5: Evaluation Design

### 5.1 Regression Areas (Post-PR Smoke Test)

| Area | What to Check | Automated? |
|---|---|---|
| Auth flow | Login works, tokens refresh silently, biometric prompt after timeout | Maestro E2E |
| Contact list | 500+ items scroll 60fps, health dots render, swipes work | Xcode Instruments (CI) |
| Offline→online | Mutation queue processes, banner lifecycle correct, cache renders | XCTest integration test |
| Chat streaming | SSE connects, tokens render 60fps, stop button functional | Maestro E2E |
| Push deep links | Each notification type lands on correct screen | Manual (per release) |
| Camera→OCR | Permission, capture, upload, review, contact create flow | Maestro E2E |
| Dark mode | All screens render correctly; no hardcoded colors | SwiftUI Preview snapshots |
| iPhone SE layout | No truncated content, no overflow, 44pt+ touch targets | SwiftUI Preview (375pt width) |
| Memory leaks | Navigate screen 10x; check allocations for growth | Xcode Instruments (per release) |
| Binary size | Under 30MB compressed | CI gate (every build) |

### 5.2 Edge Cases to Probe

| Scenario | Expected Behavior |
|---|---|
| Token expires mid-chat-stream | Stream continues if refresh silent; if fails, partial response + re-auth. |
| Airplane mode toggled 5x rapidly | Network state settles correctly; no duplicate mutations or banners. |
| App force-killed during queue processing | Queue resumes from last unprocessed item; idempotency prevents duplicates. |
| OCR upload while backgrounded | On foreground return, result available; in-app banner notifies user. |
| 1000+ contacts with 3+ filters | SwiftUI List 60fps; filter compute doesn't block main thread (use actors). |
| Contact deleted on web, pending edits on native | Mutation receives 404; dead-letter queue captures; error badge shown. |
| Identical message sent from native (offline) and web | Idempotency prevents duplicate; native syncs to existing entry. |
| Push received while viewing same screen as target | No navigation; no duplicate fetch; badge updates correctly. |
| Dynamic Type at AX5 (max) | Text renders (2x multiplier cap); no clipping; buttons accommodate; 44pt+ maintained. |
| Device storage nearly full (<50MB free) | GRDB writes fail gracefully; error toast; no corruption or crash. |
| Multiple rapid pull-to-refresh | Only one executes; subsequent gestures ignored until first completes. |
| User logout then login as different user | All prior data purged (Keychain, UserDefaults, GRDB, cache); new user fresh. |

### 5.3 Performance Benchmarks

| Benchmark | Target | Tool | Frequency |
|---|---|---|---|
| Cold launch P50 | < 1,000ms | Xcode Instruments App Launch | Per release |
| Cold launch P95 | < 1,500ms | Xcode Instruments App Launch | Per release |
| Contact list scroll (500 items) | >= 60fps | Xcode Instruments Core Animation | Per PR touching list code |
| Chat streaming FPS (token render) | >= 58fps | Xcode Instruments Time Profiler | Per PR touching chat code |
| Send to first token rendered | < 800ms (network permitting) | Custom `os_signpost` marker | Per release |
| Contact detail header render time | < 200ms (from cache) | Custom `os_signpost` marker | Per release |
| GRDB query time (500 contacts) | < 50ms | `EXPLAIN QUERY PLAN` + timing | Per schema change |
| Mutation queue drain (10 mutations) | < 30 seconds total | XCTest integration timer | Per release |
| Image cache eviction (100MB → 80MB) | < 2 seconds | XCTest integration timer | Per release |
| Binary size (compressed) | < 30MB | `xcodebuild archive` output | Every CI build |
| App memory after 30 min use | < 200MB | Xcode Instruments Allocations | Per release |
| App memory growth after 3x screen nav | No monotonic growth > 10MB | Xcode Instruments Allocations | Per release |

### 5.4 Device Test Matrix

| Device | Screen | Chip | Purpose |
|---|---|---|---|
| iPhone SE 3rd Gen | 375×667pt | A15 | Smallest; tests compact layout; no notch; home button. |
| iPhone 14/15 | 390×844pt | A15/A16 | Median; notch; typical user hardware. |
| iPhone 15 Pro / 16 Pro | 393×852pt | A17/A18 Pro | Dynamic Island; 120Hz ProMotion; animation smoothness. |
| iPhone 16 Pro Max | 430×932pt | A18 Pro | Largest; validates no excessive whitespace. |

**Requirement:** Every release must test on iPhone SE + one Pro Max physically. CI runs Maestro on iPhone 15 simulator.

### 5.5 Accessibility Test Protocol

| Check | Method | Pass Criteria |
|---|---|---|
| VoiceOver navigation | Enable VoiceOver; swipe all screens | Order matches spec; no unlabeled elements; use `.accessibilityLabel()`. |
| VoiceOver actions | Use rotor "Actions" on swipeable rows | All actions (Call, Message, Delete) accessible via `.accessibilityAction()`. |
| Dynamic Type: Default | Baseline test | Renders correctly at system default. |
| Dynamic Type: Large | Medium size setting | Text scales; no clipping; layout reflows; buttons accommodate. |
| Dynamic Type: AX3 | 3rd largest accessibility size | Capped 2x multiplier via `@ScaledMetric`; no overflow; scroll if needed; 44pt+ targets maintained. |
| Reduce Motion enabled | System setting active | Spring animations → crossfade/instant; shimmer → static; no parallax. Checked via `@Environment(\.accessibilityReduceMotion)`. |
| Bold Text enabled | System setting active | All text renders bold variant via `.bold()` / `legibilityWeight`; no layout breaks. |
| Increase Contrast enabled | System setting active | Borders thicken (1px → 1.5pt); separators darken; placeholder text darkens. Checked via `@Environment(\.accessibilityContrast)`. |
| Color contrast | Accessibility Inspector | Body text 4.5:1; large text 3:1; interactive 3:1 vs. adjacent colors (WCAG AA). |
| Switch Control | Enable Switch Control; navigate key flows | App operable via Switch Control; no trapped focus. |

### 5.6 Security Evaluation

| Check | Method | Pass Criteria |
|---|---|---|
| Token storage | Inspect Keychain post-login | Tokens in Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`; not in UserDefaults, `@AppStorage`, or any plist. |
| Token not in logs | Search `print()`, `os_log`, Sentry, network logs | No access token, refresh token, or auth code in logs. |
| Certificate pinning | Charles Proxy with custom CA (if implemented) | API requests fail when intercepted via `URLSessionDelegate` pinning. |
| Jailbreak detection | Test on jailbroken device or flag (if implemented) | App shows warning or refuses operation on compromised device. |
| GRDB encryption | Inspect `skye.db` on disk | If SQLCipher-encrypted via GRDB: unreadable without key. If not encrypted: risk documented in ADR. |
| Sensitive data in screenshots | Background app; check switcher preview | No PII in switcher thumbnail (blur or blank overlay via `UIWindow` lifecycle). |
| Clipboard handling | Copy token/password; check after 60s | Sensitive clipboard cleared after 60s (via `UIPasteboard.general.setItems(_, options: [.expirationDate])`) or not copied. |

---

## Appendix A: How to Use This Document

1. **Before phase start:** Read Problem Statement for context; read Constraint Architecture for guardrails.
2. **When assigning work:** Use Decomposition tables; identify dependencies; parallelize where indicated.
3. **When reviewing a PR:** Check relevant Acceptance Criteria — binary pass/fail only.
4. **When testing a build:** Follow Evaluation Design: run benchmarks with Xcode Instruments, probe edge cases, test device matrix, run accessibility protocol.
5. **When scoping work:** Check Section 3.5 (Out of Scope) — if listed, it is not V2.

## Appendix B: Acceptance Criteria Quick Reference

**Total: 60 criteria**

| Category | Count | IDs |
|---|---|---|
| Authentication | 6 | AC-1 – AC-6 |
| Navigation & UX | 8 | AC-7 – AC-14 |
| Chat Engine | 6 | AC-15 – AC-20 |
| CRM Screens | 6 | AC-21 – AC-26 |
| Offline Architecture | 6 | AC-27 – AC-32 |
| Camera & OCR | 6 | AC-33 – AC-38 |
| Push Notifications | 5 | AC-39 – AC-43 |
| Performance | 5 | AC-44 – AC-48 |
| Visual Design | 4 | AC-49 – AC-52 |
| App Store Compliance | 5 | AC-53 – AC-57 |
| Testing & CI | 3 | AC-58 – AC-60 |

**Key Rule:** Acceptance criteria are binary gates. No partial credit. Every build must pass all 60 criteria to be considered production-ready.
