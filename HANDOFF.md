# Skye V2 Spec — Handoff & Implementation Guide

> **Purpose:** This document is the operating manual for the Skye V2 spec. It defines what problem the spec solves, what "done" looks like (acceptance criteria), what constraints must not be violated, how to decompose the work into buildable tasks, and how to evaluate quality beyond surface-level checks. Every builder agent, QA agent, and human reviewer uses this document to interpret and execute the spec.

> **Companion to:** [skye-native-app-spec-v2.md](https://github.com/nmsitaras-afk/skye-native-app-spec-v2/blob/main/README.md)

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Acceptance Criteria](#2-acceptance-criteria)
3. [Constraint Architecture](#3-constraint-architecture)
4. [Decomposition](#4-decomposition)
5. [Evaluation Design](#5-evaluation-design)

---

## 1. Problem Statement

### 1.1 The Core Problem

Real estate agents cannot access Skye — their AI-powered CRM — from their phones in a native, reliable, offline-capable way. The existing web app (Next.js) works in mobile Safari but delivers a degraded experience: no push notifications, no camera-based business card scanning, no offline access to contacts during property showings, no haptic feedback, no home-screen presence, and no App Store discoverability. Agents lose deals because they can't pull up a contact's history while walking a property with spotty cell coverage.

### 1.2 Root Cause Analysis

| User Pain | Root Cause | Why It Exists |
|---|---|---|
| **"I can't check my contacts when I'm showing a house with no signal"** | The web app requires an active internet connection for every data fetch. There is no offline cache, no local database, and no mutation queue. | The web app was built as a desktop-first SPA. Offline support was never architectured because desktop browsers rarely lose connectivity. |
| **"I miss follow-up reminders because I don't see notifications"** | Mobile Safari does not support push notifications on iOS. The web app has no push infrastructure. | Push notifications require APNs integration, which requires a native app binary distributed through the App Store. PWA push on iOS is limited and unreliable. |
| **"I have to manually type in business card info after every networking event"** | There is no camera integration. The web app cannot access the device camera for OCR scanning. | Camera access via the browser is possible but insufficient — no frame processing, no real-time overlay, no background upload. Native camera APIs are required for a professional scanning experience. |
| **"The web app feels slow and clunky on my phone"** | Mobile Safari adds rendering overhead, has no access to native gesture handlers, and cannot use platform-optimized list virtualization. Scrolling through 500+ contacts stutters. | React DOM rendering in a mobile browser cannot match the performance of native views backed by UIKit. React Native with Hermes bytecode and native gesture handlers eliminates this gap. |
| **"I can't find Skye in the App Store — clients ask if we have an app and I have to say no"** | There is no native iOS app. | Building a native app was deferred in favor of shipping the web product first. V2 addresses this. |
| **"When I switch between Skye and my phone app, I lose my place"** | Mobile Safari aggressively discards background tabs. State is not persisted across sessions. | Web apps in Safari have no control over tab lifecycle. A native app with Zustand persisted stores and SQLite-backed query cache survives background kills. |

### 1.3 What Success Looks Like

A real estate agent at an open house with intermittent connectivity can:
1. Open Skye from their home screen (native app, instant launch < 2 seconds).
2. See their full contact list immediately from cache, even offline.
3. Scan a visitor's business card and have it auto-populate a new contact.
4. Ask the AI assistant a question about a property ("What's the HOA fee for 123 Oak?") and get a streamed response.
5. Receive a push notification 30 minutes later reminding them to follow up.
6. Generate an Instagram caption for the listing from the Studio tab.
7. Know that every edit they made offline will sync when they're back on Wi-Fi, without data loss or conflicts.

### 1.4 What This Spec Covers

The V2 spec is a **12-section, 20,000-line production specification** covering:

| Section | Domain | Key Deliverables |
|---|---|---|
| 1 | UX Architecture & Interaction Design | Every gesture, haptic, animation curve, touch target, skeleton screen, and accessibility rule |
| 2 | Visual Design System | Complete design token system (colors, typography, spacing, shadows) for light/dark modes with WCAG AA contrast compliance |
| 3 | Chat Engine & AI Interface | SSE streaming protocol, token buffering, tool-call rendering, message state machine, typing indicators |
| 4 | CRM Screens | People list virtualization, contact detail layout, property cards, engagement health dots, search/filter behavior |
| 5 | Authentication & Security | Google OAuth native flow, token lifecycle, secure storage, biometric unlock, OWASP Mobile Top 10 compliance |
| 6 | Push Notifications | APNs-direct integration, notification types/payloads, deep link resolution, badge management, silent push |
| 7 | Performance & Infrastructure | Cold launch budget (<2s), bundle size budget (<15MB), Hermes configuration, EAS Build pipeline, CI gates |
| 8 | Offline Architecture & State | TanStack Query + Zustand hybrid, SQLite mutation queue, conflict resolution, cache eviction, network state machine |
| 9 | Camera, OCR, Media & Studio | VisionCamera integration, OCR pipeline, image processing, Studio content generation, upload manager |
| 10 | App Store Compliance | Privacy manifest, nutrition labels, review preparation, screenshot specifications, rejection prevention |
| 11 | Testing Strategy & QA | Testing pyramid (60/25/15), unit/integration/E2E test specifications, CI enforcement, Maestro flows |
| 12 | Build Roadmap | Phase 0-5 with entry/exit criteria, dependency graph, task-level decomposition, verification checklists |

---

## 2. Acceptance Criteria

Every criterion below is **binary pass/fail**. No partial credit. These form the QA checklist. An item is "done" only when ALL applicable criteria pass.

### 2.1 Authentication (Section 5)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-1 | Google OAuth sign-in completes end-to-end: user taps "Sign in with Google" → native credential picker appears → user selects account → app receives tokens → user lands on People tab. | Screen recording on physical device showing full flow. |
| AC-2 | Access token is stored in iOS Keychain via `expo-secure-store` with `WHEN_UNLOCKED_THIS_DEVICE_ONLY` protection level. Token is NOT stored in AsyncStorage, MMKV, or UserDefaults. | Code review of auth store confirming `secureStorage` adapter. Keychain Sharing entitlement audit. |
| AC-3 | Token refresh happens silently when access token expires. User is never shown a login screen due to token expiry during active use. | Test: set token TTL to 30 seconds, use app for 2 minutes, verify no auth interruption. |
| AC-4 | After 401 response, the app attempts one silent token refresh. If refresh fails, user is redirected to login with all local state cleared. | API mock returning 401, verify refresh attempt, then mock refresh failure, verify redirect + state clear. |
| AC-5 | Biometric unlock (Face ID / Touch ID) gates app access after 5+ minutes of inactivity. Biometric prompt uses `expo-local-authentication`. | Physical device test: background app for 6 minutes, foreground, verify biometric prompt appears before content is visible. |
| AC-6 | Account deletion is accessible from Settings and completes via `POST /api/user/delete`. All local data (Keychain, MMKV, SQLite, image cache) is wiped. | Test: delete account, verify Keychain is empty, MMKV stores are cleared, SQLite database is deleted, app shows login screen. |

### 2.2 Navigation & UX (Sections 1, 4)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-7 | Tab bar has exactly 5 tabs: Chat, People, Properties, Studio, Settings. Tab icons match SF Symbols specified in Section 2. Active tab uses `brand.primary` tint. | Visual comparison against spec. |
| AC-8 | Every screen implements all 7 states from the universal state machine: IDLE, LOADING, LOADED, EMPTY, REFRESHING, ERROR, OFFLINE. | Screenshot matrix showing each state for each screen. |
| AC-9 | Skeleton screens match the exact layout dimensions of their loaded counterparts. No blank white screens during loading. | Side-by-side comparison of skeleton vs. loaded states. |
| AC-10 | All touch targets are >= 44x44pt. No interactive element has a smaller hit area. | Accessibility Inspector audit on every screen. |
| AC-11 | Edge swipe back (iOS interactive pop) works on every stack-pushed screen. Gesture response distance is 20pt from left edge. | Physical device test on every pushed screen. |
| AC-12 | Pull-to-refresh uses the branded Skye animation (not default `RefreshControl`). Animation plays at 60fps. Minimum display time is 600ms. | Screen recording at 60fps showing branded animation. Slow-motion verification of minimum duration. |
| AC-13 | Tab switching preserves full stack state, scroll position, search query, and filter state. Keyboard is dismissed on tab switch. | Test: apply filters on People tab, scroll down, switch to Chat, switch back — all state preserved, keyboard dismissed. |
| AC-14 | All bottom sheets use `@gorhom/bottom-sheet` with specified snap points, grab handle (36x5pt), and pan-to-close enabled. Backdrop dismisses on tap. | Component audit + physical device testing of every bottom sheet instance. |

### 2.3 Chat Engine (Section 3)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-15 | Chat messages stream token-by-token via SSE at 60fps with no frame drops. Token buffer drains via `requestAnimationFrame`. | Flipper performance profiler showing JS thread >= 58fps during streaming. |
| AC-16 | Chat input expands from 1 line (36pt) to max 5 lines (120pt) as user types, then enables internal scroll. Border radius adjusts from 20pt to 12pt at 3+ lines. | Physical device test with multi-line input. |
| AC-17 | "Stop Generating" button appears during streaming. Tapping it immediately closes the SSE connection and displays the partial response. | Test: send message, tap stop mid-stream, verify partial response is preserved and connection is terminated. |
| AC-18 | Tool-call events render inline UI cards (e.g., contact lookup shows a contact card within the chat). Tool results are visually distinct from text responses. | Send a message that triggers a tool call, verify inline card renders correctly. |
| AC-19 | Typing indicator (3 bouncing dots) appears within 100ms of sending a message and disappears when streaming begins. | Screen recording with frame timing. |
| AC-20 | Chat history uses cursor-based pagination. Scrolling up loads older messages without losing scroll position (`maintainVisibleContentPosition`). | Test: scroll up in a conversation with 100+ messages, verify no scroll jump on page load. |

### 2.4 CRM Screens (Section 4)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-21 | Contact list scrolls at 60fps with 500+ contacts using FlashList. No frame drops during fast scroll. | FlashList performance metrics: blank area < 5%, JS thread >= 58fps. |
| AC-22 | Contact list rows show: avatar (44pt circle with initials fallback), name, company, last activity, engagement health dot (color-coded by recency), and quick action icons (phone/message). | Visual comparison against Section 4 wireframe. |
| AC-23 | Engagement health dot colors match spec: Red (≤7 days), Amber (8-30 days), Blue (31-90 days), Gray (91+ days). Red dot has pulsing animation. | Test with contacts at each threshold. Verify animation on red dot. |
| AC-24 | Contact search debounces at 300ms, filters locally first, then hits API if local results < 5 and query length >= 3. Empty results show illustration + "No results for '[query]'" message. | Test: type a search query, verify debounce timing, verify empty state rendering. |
| AC-25 | Swipe-left on contact row reveals Call (green) and Delete (red) action buttons. Swipe-right reveals Message (blue). Full swipe (>180pt) triggers primary action. | Physical device test with velocity and distance variations. |
| AC-26 | Contact detail screen loads progressively: header first (from list cache), then activity timeline, linked properties, and notes — each with independent skeleton states. | Network throttling test showing progressive section loading. |

### 2.5 Offline Architecture (Section 8)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-27 | With airplane mode enabled, the app displays cached contact list, contact details, and property list from SQLite-persisted TanStack Query cache. No crash, no blank screens. | Physical device test: load data online, enable airplane mode, navigate all cached screens. |
| AC-28 | Edits made offline (edit contact, add note, create task) are queued in the `mutation_queue` SQLite table with status `pending`. Queue survives app kill and restart. | Test: edit contact offline, force-kill app, relaunch, verify mutation exists in SQLite. |
| AC-29 | When connectivity returns, the mutation queue processes in FIFO order. Each mutation includes an `Idempotency-Key` header. Successfully synced mutations show brief green "synced" badge. | Test: queue 3 mutations offline, restore connectivity, verify FIFO processing order and idempotency headers. |
| AC-30 | Conflict resolution: 409 responses auto-resolve scalar fields via last-write-wins. Complex fields (deals) surface a merge UI. Dead-lettered mutations (404, 403) show error badge with dismiss option. | Mock server returning 409 and 404, verify auto-resolution and dead-letter handling. |
| AC-31 | Network state banner appears within 2 seconds of connectivity loss: "You're offline — showing cached data" with pending mutation count. Banner dismisses within 2 seconds of restoration. | Toggle airplane mode, time banner appearance and dismissal. |
| AC-32 | Cache eviction fires when data cache exceeds 50MB. LRU eviction based on `accessed_at` column. Contacts with pending mutations are protected from eviction. | Fill cache to >50MB, verify eviction runs, verify protected entities survive. |

### 2.6 Camera & OCR (Section 9)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-33 | Camera permission flow shows pre-permission screen with Skye branding BEFORE the iOS system prompt. "Not Now" dismisses without triggering the system prompt. | Screen recording showing branded screen before system alert. |
| AC-34 | Camera viewfinder opens in < 500ms from tap. Overlay shows business card frame guide (3.5:2 aspect ratio) with animated corner brackets. | Stopwatch measurement from tap to live viewfinder. |
| AC-35 | Captured image is processed (orientation-corrected, resized to max 1920px longest edge, JPEG 85% quality) and is < 5MB. Image is never saved to camera roll. | Assert file size after processing. Verify camera roll is empty after scan. |
| AC-36 | OCR review screen shows all extracted fields with confidence indicators: green checkmark (>=0.85), yellow warning (0.60-0.84), red warning (<0.60). All fields are editable. | Test with a business card image, verify confidence indicators match response values. |
| AC-37 | Duplicate contact detection: if OCR creates a contact with an existing email/phone, a "Contact May Already Exist" screen appears with options: View Existing, Create Anyway, Merge. | Mock 409 response from create endpoint, verify screen renders with all 3 options. |
| AC-38 | Studio generates content via SSE streaming with real-time token rendering. Quick tweak chips ("Make it Shorter", "More Professional", etc.) trigger regeneration. Copy button copies plain text to clipboard. | Test: generate Instagram caption, apply tweak, copy result, paste to verify content. |

### 2.7 Push Notifications (Section 6)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-39 | Push notification permission flow shows branded pre-permission screen before the iOS system prompt. Screen explains value ("Never miss a follow-up"). | Screen recording showing branded screen. |
| AC-40 | Device token is registered via `POST /api/notifications/register` with token, platform, and device metadata. Token is re-registered on every app launch. | Network log showing registration call on launch. |
| AC-41 | Tapping a push notification deep-links to the correct screen (contact detail, chat conversation, task detail). If the target resource doesn't exist, fallback screen shows with toast. | Test each notification type: tap → verify correct screen. Test with invalid resource ID → verify fallback. |
| AC-42 | Badge count on app icon reflects unread notifications. Badge clears when user opens the relevant tab. | Test: receive notifications, verify badge count on icon. Open tab, verify badge clears. |
| AC-43 | Notification categories with action buttons work: "Reply" and "Mark as Read" on chat notifications, "Call Back" on contact reminders. | Test: long-press notification in Notification Center, verify action buttons appear and function. |

### 2.8 Performance (Section 7)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-44 | Cold launch to interactive < 2,000ms on iPhone 13 (P95). Measured from process start to first frame accepting touch input. | Xcode Instruments App Launch trace showing P95 metric. |
| AC-45 | JavaScript bundle size < 15MB compressed (Hermes bytecode). CI gate fails build if bundle exceeds 18MB. | EAS Build output log showing bundle size. CI pipeline screenshot showing gate. |
| AC-46 | App memory usage < 150MB with 500 contacts loaded in a scrolling list. No memory leaks after navigating to/from a screen 10 times. | Xcode Instruments Allocations trace. Leaks trace after 10 navigation cycles. |
| AC-47 | All scrollable lists maintain >= 58fps on JS thread and >= 60fps on UI thread during fast scroll. | Flipper performance monitor screenshot during scroll stress test. |
| AC-48 | Hermes bytecode compilation is verified: `.hbc` file present in app bundle, NOT `.jsbundle`. | EAS Build artifact inspection. |

### 2.9 Visual Design (Section 2)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-49 | Light mode and dark mode are fully implemented. All semantic color tokens match spec values. Dark mode uses true black (`#000000`) for OLED. Theme switches with system setting. | Screenshot comparison against spec token table for both modes on OLED device. |
| AC-50 | Typography uses SF Pro with the exact scale from Section 2: Display (34pt Bold), Title (28pt Bold), Headline (22pt Semibold), Body (17pt Regular), Caption (13pt Regular). | Font inspector audit on every text element. |
| AC-51 | All color combinations meet WCAG AA contrast ratio: 4.5:1 for body text, 3:1 for large text, 3:1 for interactive elements. | Automated contrast ratio check using Accessibility Inspector. |
| AC-52 | Reduce Motion is respected: all spring animations degrade to crossfade or instant state change. Skeleton shimmer becomes static. No motion-sensitive user is subjected to animation. | Enable Reduce Motion in iOS Settings, navigate all screens, verify no spring/bounce/parallax animations. |

### 2.10 App Store Compliance (Section 10)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-53 | `PrivacyInfo.xcprivacy` declares all required-reason APIs: FileTimestamp (C617.1), SystemBootTime (35F9.1), DiskSpace (E174.1), UserDefaults (CA92.1). `NSPrivacyTracking` is `false`. | Xcode Privacy Report generation. Manual XML review. |
| AC-54 | Privacy nutrition labels in App Store Connect accurately declare: Contact Info (linked), Identifiers (linked), Usage Data (linked), Diagnostics (not linked). No under-declaration. | Comparison of App Store Connect entries against Section 10 table. |
| AC-55 | App includes in-app account deletion accessible from Settings. Deletion cascade removes all user data from Supabase, Keychain, MMKV, SQLite, and image cache. | Test: create account, add data, delete account, verify all storage backends are empty. |
| AC-56 | Demo account credentials are prepared for App Review (email + password) with pre-populated data (contacts, properties, chat history). Review notes explain AI features and camera usage. | Verify demo account has data. Read review notes for completeness. |
| AC-57 | App does not crash on any supported device (iPhone SE 3rd Gen through iPhone 16 Pro Max). All screens render correctly on 375pt (SE) through 430pt (Pro Max) widths. | TestFlight testing on at minimum: iPhone SE, iPhone 15, iPhone 16 Pro Max. |

### 2.11 Testing & CI (Section 11)

| # | Criterion | Evidence Required |
|---|---|---|
| AC-58 | Unit test coverage meets thresholds: `src/lib/**` >= 88% lines, `src/hooks/**` >= 78% lines, `src/stores/**` >= 68% lines. CI fails below these. | Jest coverage report from CI pipeline. |
| AC-59 | E2E tests cover 5 critical flows via Maestro: (1) sign-in, (2) browse contacts, (3) view contact detail, (4) send chat message, (5) scan business card. All pass on CI. | Maestro test results from CI run. |
| AC-60 | Accessibility audit passes: all screens navigable via VoiceOver in correct order, all touch targets >= 44pt, Dynamic Type scales without clipping at AX3 size. | VoiceOver screen recording on physical device. Accessibility Inspector report. |

---

## 3. Constraint Architecture

### 3.1 Backend Constraints (DO NOT MODIFY)

| Constraint | Detail |
|---|---|
| **Supabase schema** | All existing tables, columns, types, and relationships are frozen. The native app reads and writes to the same database as the web app. No new tables may be added without explicit approval. No column renames. No type changes. |
| **RLS policies** | All Row Level Security policies remain as-is. The native app authenticates via Supabase JWT (same `auth.uid()` resolution). If a policy doesn't work with bearer-token auth (vs. cookie-based session), the policy is fixed server-side — the native app does not work around it. |
| **API routes** | All 129 existing `/api/*` routes are the interface. The native app calls the same endpoints as the web app. New endpoints are limited to: `/api/auth/native`, `/api/notifications/register`, `/api/health`, `/api/user/delete`, `/api/contacts/ocr`, `/api/studio/generate`. No existing route behavior changes. |
| **API response format** | All responses are JSON. No HTML error pages. `Content-Type: application/json` enforced. If an existing route returns HTML on error, it is fixed server-side. |

### 3.2 Technology Constraints (LOCKED)

| Decision | Choice | Reversibility |
|---|---|---|
| Runtime | React Native 0.76+ with Expo SDK 52+ | Irreversible after Phase 1 |
| JS engine | Hermes (bytecode compilation) | Irreversible |
| Navigation | React Navigation 6+ (native stack) | High cost to reverse after Phase 1 |
| Server state | TanStack Query v5 (`networkMode: 'offlineFirst'`) | Medium cost to reverse |
| Client state | Zustand v4 with persist middleware | Low cost to reverse |
| Local database | expo-sqlite (WAL mode) | Medium cost to reverse |
| Secure storage | expo-secure-store (iOS Keychain) | Low cost to reverse |
| Fast KV storage | react-native-mmkv | Low cost to reverse |
| Camera | react-native-vision-camera v4+ | Medium cost to reverse |
| Image rendering | expo-image | Low cost to reverse |
| Animations | react-native-reanimated v3 + react-native-gesture-handler v2 | Irreversible after Phase 1 |
| Lists | @shopify/flash-list | Low cost to reverse |
| Bottom sheets | @gorhom/bottom-sheet v5 | Low cost to reverse |
| Push notifications | APNs-direct (server) + @notifee/react-native (client) | Medium cost to reverse |
| Error monitoring | @sentry/react-native | Low cost to reverse |

### 3.3 Performance Budgets (CI-ENFORCED GATES)

| Metric | Budget | Hard Ceiling (CI Fails) |
|---|---|---|
| Cold launch to interactive | < 2,000ms (P95, iPhone 13) | 2,500ms |
| JS bundle size (compressed) | < 15MB | 18MB |
| App memory (500 contacts loaded) | < 150MB | 200MB |
| JS thread FPS during scroll | >= 60fps | 58fps |
| UI thread FPS during scroll | >= 60fps | 58fps |
| JS thread FPS during animation | >= 58fps | 55fps |
| Data cache (SQLite) | < 50MB | 50MB (triggers eviction) |
| Image cache (disk) | < 100MB | 100MB (triggers eviction) |
| Time to first meaningful paint | < 500ms | 800ms |

### 3.4 Design Constraints

| Constraint | Detail |
|---|---|
| **Orientation** | Portrait only. Locked in `Info.plist` and `app.json`. No landscape support in V2. |
| **iPad** | Runs in iPhone compatibility mode (letterboxed). No iPad-native layouts in V2. |
| **iOS version** | Minimum iOS 16. No support for iOS 15 or below. |
| **Device support** | iPhone SE 3rd Gen through iPhone 16 Pro Max. All screen sizes 375pt-430pt width. |
| **Accessibility floor** | WCAG AA minimum. VoiceOver full support. Dynamic Type to AX5 (capped at 2x multiplier). Reduce Motion respected on all animations. |
| **Color system** | Three-layer token architecture (primitive → semantic → component). No raw hex values in component code. All colors referenced via semantic tokens. |

### 3.5 Explicit Out of Scope (V2)

These items are explicitly NOT part of V2. Do not build, plan, or estimate them:

| Item | Reason |
|---|---|
| Android app | iOS-only for V2. Android is a V3 consideration. |
| iPad-native layouts | Runs in iPhone compatibility mode. iPad layouts are V3. |
| Landscape orientation | Portrait-locked. Landscape media viewer is V3. |
| Real-time WebSocket sync | Poll-based sync (TanStack Query staleTime) is sufficient for V2. Real-time is V3. |
| Offline AI chat | AI responses require server connectivity. Chat messages can be queued but responses cannot be generated offline. |
| Offline image upload | Too large for reliable queue. Upload requires connectivity. |
| Apple Watch companion | Not in scope. |
| Widgets (iOS home screen) | Not in scope for V2. |
| Siri Shortcuts integration | Not in scope for V2. |
| In-app purchases | Skye billing is handled via the web app. No IAP in V2. |
| Bulk import/export from native app | Available only via web app. |
| Voice input in chat | V3 feature. |
| Video calling | Not in scope. |

---

## 4. Decomposition

### 4.1 Phase Dependency Graph

```
Phase 0: Pre-Development Setup
    |
    v
Phase 1: Foundation (Auth, API Client, State, Theming, Navigation Shell)
    |
    +---> Phase 2: Core Screens (People List, Contact Detail, Property List, Property Detail)
    |         |
    |         +---> Phase 3: Chat Engine (SSE streaming, message list, tool-call UI, input bar)
    |         |
    |         +---> Phase 4: Camera & OCR (VisionCamera, capture pipeline, OCR flow, review screen)
    |         |
    |         +---> Phase 4 (parallel): Studio (content types, generation SSE, history, quick tweaks)
    |
    +---> Phase 2 (parallel): Push Notifications (APNs registration, handlers, deep links, badges)
    |
    v
Phase 5: Polish, Performance, Compliance, Submission
```

### 4.2 Phase 0 — Pre-Development Setup

| # | Task | Files/Actions | Change Type | Parallelizable | Agent Routing |
|---|---|---|---|---|---|
| 0.1 | Activate Apple Developer Program, create App ID `com.skye.crm` with Push Notifications capability | Apple Developer Portal (manual) | Config | Yes | Human |
| 0.2 | Create Google Cloud OAuth iOS client ID + web client ID | Google Cloud Console (manual) | Config | Yes | Human |
| 0.3 | Audit Supabase RLS policies with mobile-issued JWT | Supabase dashboard + test script | Audit | Yes | Backend agent |
| 0.4 | Finalize design system in Figma: color tokens, type scale, spacing, component library | Figma (manual) | Design | Yes | Human/Design |
| 0.5 | Lock Architecture Decision Records (ADR-1 through ADR-5) | `DECISIONS.md` (new file) | New file | No (depends on 0.4) | Human |
| 0.6 | Provision dev environments: Xcode 16+, CocoaPods, Expo CLI, EAS CLI, physical devices | Developer machines (manual) | Config | Yes | Human |
| 0.7 | Initialize repository: monorepo structure, `.gitignore`, ESLint + Prettier, `tsconfig.json`, `app.json`, CI pipeline stub, branch protection | Repository root | New files | No (depends on 0.6) | Code agent |
| 0.8 | Create Sentry project, obtain DSN | Sentry dashboard (manual) | Config | Yes | Human |
| 0.9 | Implement `/api/health` endpoint | `pages/api/health.ts` or equivalent | New file | Yes | Backend agent |
| 0.10 | Implement `/api/auth/native` endpoint (Google auth code exchange) | `pages/api/auth/native.ts` | New file | No (depends on 0.2) | Backend agent |
| 0.11 | Implement `/api/user/delete` endpoint (account deletion cascade) | `pages/api/user/delete.ts` | New file | Yes | Backend agent |
| 0.12 | Audit all 129 API routes for JSON-only responses | Audit script | Audit + modify | Yes | Backend agent |

### 4.3 Phase 1 — Foundation

| # | Task | Files | Change Type | Parallelizable | Agent Routing |
|---|---|---|---|---|---|
| 1.1 | Scaffold Expo project with `npx create-expo-app`, configure `app.json` (bundle ID, orientation, splash, icons, Hermes, EAS config) | `app.json`, `app.config.ts`, `eas.json` | New files | No (first task) | Code agent |
| 1.2 | Install and configure all foundation dependencies (React Navigation, TanStack Query, Zustand, MMKV, expo-secure-store, expo-sqlite, Reanimated, Gesture Handler) | `package.json`, various config files | New + modify | No (depends on 1.1) | Code agent |
| 1.3 | Implement design token system: `src/theme/colors.ts`, `src/theme/typography.ts`, `src/theme/spacing.ts`, `src/theme/shadows.ts`, `src/theme/springs.ts` | `src/theme/*` | New files | Yes (after 1.2) | Code agent (design-focused) |
| 1.4 | Implement storage adapters: `src/lib/mmkvStorage.ts`, `src/lib/secureStorage.ts` | `src/lib/*Storage.ts` | New files | Yes (after 1.2) | Code agent |
| 1.5 | Implement SQLite database setup: `src/lib/database.ts`, `src/lib/migrations.ts`, schema v1 | `src/lib/database.ts`, `src/lib/migrations.ts` | New files | Yes (after 1.2) | Code agent |
| 1.6 | Implement TanStack Query client: `src/lib/queryClient.ts`, `src/lib/queryKeys.ts`, `src/lib/queryPersister.ts` | `src/lib/query*.ts` | New files | Yes (after 1.5) | Code agent |
| 1.7 | Implement Zustand stores: `src/stores/authStore.ts`, `src/stores/uiStore.ts`, `src/stores/chatStore.ts`, `src/stores/offlineStore.ts`, `src/stores/notificationStore.ts` | `src/stores/*.ts` | New files | Yes (after 1.4) | Code agent |
| 1.8 | Implement API client: `src/lib/apiClient.ts` (auth header injection, 401 refresh interceptor, error normalization) | `src/lib/apiClient.ts` | New file | No (depends on 1.7) | Code agent |
| 1.9 | Implement auth flow: `src/hooks/useAuth.ts`, `src/screens/auth/LoginScreen.tsx`, Google OAuth integration | `src/hooks/useAuth.ts`, `src/screens/auth/*` | New files | No (depends on 1.7, 1.8) | Code agent |
| 1.10 | Implement online manager: `src/lib/onlineManager.ts`, `src/lib/networkManager.ts` (NetInfo bridge) | `src/lib/onlineManager.ts`, `src/lib/networkManager.ts` | New files | Yes (after 1.6) | Code agent |
| 1.11 | Implement tab navigator shell: `src/app/_layout.tsx` with 5 tabs, provider tree (QueryClientProvider, KeyboardProvider, GestureHandlerRootView, SafeAreaProvider) | `src/app/_layout.tsx`, tab navigator config | New files | No (depends on 1.3, 1.6, 1.7) | Code agent |
| 1.12 | Implement network banner: `src/components/NetworkBanner.tsx` | `src/components/NetworkBanner.tsx` | New file | Yes (after 1.10) | Code agent |
| 1.13 | Implement Sentry integration: `src/lib/sentry.ts` | `src/lib/sentry.ts` | New file | Yes (after 1.1) | Code agent |
| 1.14 | Configure EAS Build profiles: development, preview, production | `eas.json` | Modify | Yes (after 1.1) | Code agent |
| 1.15 | Write unit tests for: apiClient, queryClient, all Zustand stores, storage adapters | `src/**/*.test.ts` | New files | Yes (after 1.4-1.8) | Testing agent |
| 1.16 | First successful build to physical device via EAS Build | N/A (CI/build) | Build | No (last Phase 1 task) | Human + CI |

### 4.4 Phase 2 — Core Screens

| # | Task | Files | Change Type | Parallelizable | Agent Routing |
|---|---|---|---|---|---|
| 2.1 | Implement People list screen with FlashList, alphabetical sections, engagement health dots | `src/screens/people/PeopleListScreen.tsx`, `src/components/ContactRow.tsx` | New files | Yes | Code agent (UI) |
| 2.2 | Implement contact search with 300ms debounce, local-first + API fallback | `src/hooks/useContactSearch.ts`, search bar component | New files | Yes (after 2.1) | Code agent |
| 2.3 | Implement contact filters bottom sheet | `src/screens/people/FilterContactsSheet.tsx` | New file | Yes (after 2.1) | Code agent (UI) |
| 2.4 | Implement swipe actions on contact rows (Call, Delete, Message) | Modify `ContactRow.tsx` | Modify | No (depends on 2.1) | Code agent (UI) |
| 2.5 | Implement Contact Detail screen with progressive loading sections | `src/screens/people/ContactDetailScreen.tsx`, section components | New files | Yes | Code agent (UI) |
| 2.6 | Implement Edit Contact bottom sheet | `src/screens/people/EditContactSheet.tsx` | New file | Yes (after 2.5) | Code agent (UI) |
| 2.7 | Implement Add Contact modal (full screen) | `src/screens/people/AddContactScreen.tsx` | New file | Yes | Code agent (UI) |
| 2.8 | Implement Add Note bottom sheet | `src/screens/people/AddNoteSheet.tsx` | New file | Yes (after 2.5) | Code agent (UI) |
| 2.9 | Implement Property List screen with cards (hero image + details) | `src/screens/properties/PropertyListScreen.tsx`, `src/components/PropertyCard.tsx` | New files | Yes | Code agent (UI) |
| 2.10 | Implement Property Detail screen (hero image, Street View, details, linked contacts) | `src/screens/properties/PropertyDetailScreen.tsx` | New file | Yes (after 2.9) | Code agent (UI) |
| 2.11 | Implement image components: `ContactAvatar.tsx`, `StreetViewImage.tsx` with blurhash placeholders | `src/components/ContactAvatar.tsx`, `src/components/StreetViewImage.tsx` | New files | Yes | Code agent (UI) |
| 2.12 | Implement mutation hooks with optimistic updates: `useEditContact`, `useCreateContact`, `useDeleteContact`, `useAddNote` | `src/hooks/mutations/*.ts` | New files | Yes (after 2.5) | Code agent |
| 2.13 | Implement offline mutation queue: `src/lib/mutationQueue.ts`, `src/lib/queueProcessor.ts` | `src/lib/mutationQueue.ts`, `src/lib/queueProcessor.ts` | New files | Yes (after 1.5) | Code agent |
| 2.14 | Implement cache eviction: `src/lib/cacheEviction.ts` | `src/lib/cacheEviction.ts` | New file | Yes (after 1.5) | Code agent |
| 2.15 | Implement pull-to-refresh branded animation | `src/components/BrandedRefresh.tsx` | New file | Yes | Code agent (animation) |
| 2.16 | Write tests for CRM screens (component tests + mutation hook tests) | `src/**/*.test.ts` | New files | Yes (after 2.1-2.14) | Testing agent |

### 4.5 Phase 3 — Chat Engine

| # | Task | Files | Change Type | Parallelizable | Agent Routing |
|---|---|---|---|---|---|
| 3.1 | Implement SSE streaming engine: connection state machine, token parser, buffer drain | `src/lib/chatStream.ts` | New file | Yes | Code agent (networking) |
| 3.2 | Implement chat session list screen | `src/screens/chat/ChatListScreen.tsx` | New file | Yes | Code agent (UI) |
| 3.3 | Implement chat conversation screen: message list (inverted FlashList), message bubbles, timestamps | `src/screens/chat/ChatConversationScreen.tsx`, `src/components/ChatBubble.tsx` | New files | No (depends on 3.1) | Code agent (UI) |
| 3.4 | Implement chat input bar: expanding TextInput, send button, attachment button | `src/components/ChatInputBar.tsx` | New file | Yes | Code agent (UI) |
| 3.5 | Implement typing indicator (3 bouncing dots) | `src/components/TypingIndicator.tsx` | New file | Yes | Code agent (animation) |
| 3.6 | Implement tool-call inline cards (contact card, property card within chat) | `src/components/chat/ToolCallCard.tsx` | New file | No (depends on 3.3) | Code agent (UI) |
| 3.7 | Implement "Stop Generating" button and partial response preservation | Modify `ChatConversationScreen.tsx` | Modify | No (depends on 3.3) | Code agent |
| 3.8 | Implement chat history pagination (cursor-based, inverted scroll) | Modify chat screen + hooks | Modify | No (depends on 3.3) | Code agent |
| 3.9 | Implement offline chat message queuing with "AI responses require connectivity" warning | `src/hooks/useSendMessage.ts` | New file | No (depends on 2.13) | Code agent |
| 3.10 | Write tests: SSE parser unit tests, message state transitions, streaming performance test | `src/**/*.test.ts` | New files | Yes (after 3.1-3.8) | Testing agent |

### 4.6 Phase 4 — Camera, OCR & Studio (Parallel Tracks)

**Track A: Camera & OCR**

| # | Task | Files | Change Type | Parallelizable | Agent Routing |
|---|---|---|---|---|---|
| 4A.1 | Implement camera permission flow with branded pre-permission screen | `src/hooks/useCameraAccess.ts`, `src/screens/camera/PermissionScreen.tsx` | New files | Yes | Code agent (UI) |
| 4A.2 | Implement camera viewfinder with overlay frame (3.5:2 aspect ratio guide, corner brackets) | `src/screens/camera/CameraScreen.tsx`, `src/components/CardFrameOverlay.tsx` | New files | No (depends on 4A.1) | Code agent (UI + native) |
| 4A.3 | Implement photo capture pipeline: orientation correction, resize, JPEG compression | `src/lib/imageProcessing.ts` | New file | Yes | Code agent |
| 4A.4 | Implement OCR preview screen | `src/screens/camera/PreviewScreen.tsx` | New file | Yes (after 4A.2) | Code agent (UI) |
| 4A.5 | Implement OCR upload with progress bar (XMLHttpRequest for progress events) | `src/services/uploadManager.ts` | New file | Yes | Code agent (networking) |
| 4A.6 | Implement OCR review screen with confidence indicators and editable fields | `src/screens/camera/ReviewScreen.tsx` | New file | No (depends on 4A.5) | Code agent (UI) |
| 4A.7 | Implement OCR success/error/duplicate states | `src/screens/camera/ResultScreen.tsx` | New file | No (depends on 4A.6) | Code agent (UI) |
| 4A.8 | Implement `/api/contacts/ocr` backend endpoint | Backend: `pages/api/contacts/ocr.ts` | New file | Yes | Backend agent |
| 4A.9 | Write tests for image processing, upload manager, OCR flow | `src/**/*.test.ts` | New files | Yes (after 4A.1-4A.7) | Testing agent |

**Track B: Studio**

| # | Task | Files | Change Type | Parallelizable | Agent Routing |
|---|---|---|---|---|---|
| 4B.1 | Implement Studio home screen with content type selector chips | `src/screens/studio/StudioHomeScreen.tsx` | New file | Yes | Code agent (UI) |
| 4B.2 | Implement context input forms per content type (Instagram, Email, Listing, Market Update, Open House) | `src/screens/studio/ContentInputForm.tsx`, type-specific config | New files | No (depends on 4B.1) | Code agent (UI) |
| 4B.3 | Implement SSE generation flow with streaming output and markdown rendering | `src/screens/studio/GenerationOutput.tsx` | New file | No (depends on 4B.2) | Code agent |
| 4B.4 | Implement quick tweak chips with regeneration | Modify `GenerationOutput.tsx` | Modify | No (depends on 4B.3) | Code agent |
| 4B.5 | Implement copy/share/regenerate action buttons | Modify `GenerationOutput.tsx` | Modify | No (depends on 4B.3) | Code agent (UI) |
| 4B.6 | Implement generation history (SQLite-backed, last 100 entries) | `src/screens/studio/HistoryScreen.tsx`, `src/lib/studioHistory.ts` | New files | Yes | Code agent |
| 4B.7 | Implement `/api/studio/generate` backend SSE endpoint | Backend: `pages/api/studio/generate.ts` | New file | Yes | Backend agent |
| 4B.8 | Write tests for Studio flows | `src/**/*.test.ts` | New files | Yes (after 4B.1-4B.6) | Testing agent |

### 4.7 Phase 2 (Parallel) — Push Notifications

| # | Task | Files | Change Type | Parallelizable | Agent Routing |
|---|---|---|---|---|---|
| PN.1 | Implement notification permission flow with branded pre-permission screen | `src/hooks/useNotificationPermission.ts`, `src/screens/notifications/PermissionScreen.tsx` | New files | Yes | Code agent (UI) |
| PN.2 | Implement APNs device token registration | `src/lib/pushRegistration.ts` | New file | Yes | Code agent (native) |
| PN.3 | Implement `/api/notifications/register` backend endpoint | Backend: `pages/api/notifications/register.ts` | New file | Yes | Backend agent |
| PN.4 | Implement notification handler: foreground presentation, background handler, tap handler with deep link resolution | `src/lib/notificationHandler.ts` | New file | No (depends on PN.2) | Code agent |
| PN.5 | Implement deep link resolution engine (URL scheme `skye://`, notification payloads, auth gate, resource validation) | `src/lib/deepLinkResolver.ts` | New file | No (depends on PN.4) | Code agent |
| PN.6 | Implement badge management (set, increment, decrement, clear per tab) | `src/lib/badgeManager.ts` | New file | Yes | Code agent |
| PN.7 | Implement notification categories with action buttons | `src/lib/notificationCategories.ts` | New file | Yes | Code agent (native) |
| PN.8 | Implement backend push send service (APNs-direct via `@parse/node-apn`) | Backend: `src/services/pushService.ts` | New file | No (depends on PN.3) | Backend agent |
| PN.9 | Write tests for notification handling, deep link resolution, badge management | `src/**/*.test.ts` | New files | Yes (after PN.1-PN.7) | Testing agent |

### 4.8 Phase 5 — Polish, Performance, Compliance, Submission

| # | Task | Files | Change Type | Parallelizable | Agent Routing |
|---|---|---|---|---|---|
| 5.1 | Performance audit: cold launch profiling, memory profiling, scroll performance profiling on iPhone 13 | Xcode Instruments traces | Audit | Yes | Performance agent |
| 5.2 | Bundle size audit and optimization: tree-shake unused imports, lazy-load secondary tabs | Various files | Modify | Yes | Code agent |
| 5.3 | Accessibility audit: VoiceOver navigation order on every screen, Dynamic Type testing at AX3, touch target audit | Various files | Modify | Yes | QA agent |
| 5.4 | Build `PrivacyInfo.xcprivacy` with all required-reason API declarations | `app.json` (privacyManifests field) | Modify | Yes | Code agent |
| 5.5 | Prepare App Store Connect: screenshots (6.7", 6.5", 5.5"), promotional text, keywords, description, categories | App Store Connect (manual) | Config | Yes | Human |
| 5.6 | Prepare demo account with pre-populated data for App Review | Supabase seed script | New file | Yes | Backend agent |
| 5.7 | Write App Review notes explaining AI features, camera usage, data handling | App Store Connect (manual) | Config | No (depends on 5.5) | Human |
| 5.8 | Final E2E test suite run via Maestro on all 5 critical flows | Maestro test files | Run | No (last task) | Testing agent + CI |
| 5.9 | TestFlight distribution to internal testers (minimum 3 device variants) | EAS Build + TestFlight | Build | No (depends on 5.8) | CI + Human |
| 5.10 | Submit to App Store Review | App Store Connect | Submission | No (last task) | Human |

---

## 5. Evaluation Design

### 5.1 Regression Areas

After every PR merge, the following areas must be smoke-tested (automated where possible):

| Area | What to Check | Automated? |
|---|---|---|
| Auth flow | Login still works, tokens refresh, biometric prompt appears after timeout | Maestro E2E |
| Contact list rendering | 500+ contacts scroll at 60fps, health dots render, swipe actions work | FlashList performance metrics (CI) |
| Offline -> online transition | Mutation queue processes, banner appears/dismisses, cached data renders | Integration test |
| Chat streaming | SSE connects, tokens render at 60fps, stop button works | Maestro E2E |
| Push notification deep links | Tapping each notification type lands on correct screen | Manual (per release) |
| Camera -> OCR flow | Permission prompt, capture, upload, review, create contact | Maestro E2E |
| Dark mode | All screens render correctly in dark mode, no hardcoded colors | Snapshot tests |
| iPhone SE layout | No truncated content, no overflow, all touch targets 44pt+ | Snapshot tests at 375pt width |
| Memory leaks | Navigate screen 10 times, check allocations for growth | Instruments (per release) |
| Bundle size | Stays under 15MB | CI gate (every build) |

### 5.2 Edge Cases to Probe

| Scenario | Expected Behavior | How to Test |
|---|---|---|
| Token expires mid-chat-stream | Stream continues if refresh succeeds silently. If refresh fails, partial response preserved + re-auth prompt. | Set token TTL to 10s, start a long chat response. |
| Airplane mode toggled rapidly (on-off-on-off) | Network state machine settles correctly. No duplicate mutation processing. No duplicate banners. | Toggle airplane mode 5 times in 10 seconds. |
| App killed during offline mutation queue processing | Queue resumes from last unprocessed mutation on relaunch. No duplicates (idempotency keys). | Start queue processing, force-kill, relaunch. |
| OCR upload completes while app is backgrounded | On foreground return, OCR result is available. In-app banner: "Business card scan complete!" | Start OCR upload, background app, wait, foreground. |
| 1000+ contacts with all filters active | FlashList still scrolls at 60fps. Filter computation doesn't block UI thread. | Load test data, apply 3+ filters simultaneously. |
| User deletes a contact on web while native has pending edits for that contact | Mutation queue receives 404. Dead letter queue captures it. Error badge shown. No crash. | Edit contact on native offline, delete on web, bring native online. |
| Chat message sent offline, then same message sent from web before native syncs | Idempotency key prevents duplicate. Native message syncs, web message already exists. | Send identical message from both clients. |
| App receives push notification while on the same screen the notification links to | No navigation change. No duplicate data fetch. Badge count still updates correctly. | Send push for current contact while viewing that contact. |
| Dynamic Type set to AX5 (maximum) | All text renders (capped at 2x multiplier). No text clipping. Buttons grow to accommodate. Touch targets remain >= 44pt. | Set AX5 in iOS Settings, navigate all screens. |
| Storage pressure (device nearly full) | SQLite writes fail gracefully. Error toast: "Storage full." No data corruption. App doesn't crash. | Fill device storage to <50MB free, attempt to save data. |
| Multiple rapid pull-to-refresh gestures | Only one refresh executes at a time. Second gesture is ignored until first completes. | Pull-to-refresh 5 times rapidly. |
| User logs out then logs in as different user | All previous user's data is completely purged (Keychain, MMKV, SQLite, image cache). New user starts fresh. | Log out, log in as user B, verify no user A data visible. |

### 5.3 Performance Benchmarks

| Benchmark | Target | Tool | Frequency |
|---|---|---|---|
| Cold launch to interactive (P50) | < 1,500ms | Xcode Instruments App Launch | Per release |
| Cold launch to interactive (P95) | < 2,000ms | Xcode Instruments App Launch | Per release |
| Contact list scroll FPS (500 items) | >= 60fps JS, >= 60fps UI | Flipper Performance Monitor | Per PR touching list code |
| Chat streaming FPS (during token render) | >= 58fps JS | Flipper Performance Monitor | Per PR touching chat code |
| Time from "Send" to first token rendered | < 800ms (network permitting) | Custom perf marker | Per release |
| Contact detail time to header render | < 200ms (from cache) | Custom perf marker | Per release |
| SQLite query time (500 contacts) | < 50ms | SQLite EXPLAIN QUERY PLAN + timing | Per schema change |
| Mutation queue drain (10 mutations) | < 30 seconds total | Integration test timer | Per release |
| Image cache eviction (100MB -> 80MB) | < 2 seconds | Integration test timer | Per release |
| Bundle size (compressed Hermes bytecode) | < 15MB | `expo export` output | Every CI build |
| App memory after 30 min active use | < 200MB | Xcode Instruments Allocations | Per release |
| App memory after navigating all screens 3x | No monotonic growth > 10MB | Xcode Instruments Allocations | Per release |

### 5.4 Device Test Matrix

| Device | Screen Size | Chip | Why This Device |
|---|---|---|---|
| iPhone SE 3rd Gen | 375 x 667pt | A15 | Smallest supported screen. No notch. No Dynamic Island. Home button. Tests compact layout. |
| iPhone 14 / 15 | 390 x 844pt | A15/A16 | Median device. Notch. Represents typical user hardware. |
| iPhone 15 Pro / 16 Pro | 393 x 852pt | A17/A18 Pro | Dynamic Island. ProMotion 120Hz. Tests animation smoothness at high refresh rate. |
| iPhone 16 Pro Max | 430 x 932pt | A18 Pro | Largest screen. Tests "large" layout category. Validates no excessive whitespace. |

**Every release build must be tested on at least iPhone SE + one Pro Max.** CI runs Maestro tests on iPhone 15 simulator. Physical device testing covers SE and Pro Max.

### 5.5 Accessibility Test Protocol

| Check | Method | Pass Criteria |
|---|---|---|
| VoiceOver navigation order | Enable VoiceOver, swipe through every screen | Order matches Section 6 of UX spec for each screen. No unlabeled elements. No "button, button, button" without descriptive labels. |
| VoiceOver custom actions | On swipeable rows, use VoiceOver rotor "Actions" | All swipe actions available as accessibility actions (Call, Message, Delete, etc.) |
| Dynamic Type at Default | Set text size to Default | Baseline — all screens render correctly |
| Dynamic Type at Large | Set text size to Large | Text scales, no clipping, layout reflows. Buttons accommodate larger text. |
| Dynamic Type at AX3 | Set text size to AX3 (3rd largest accessibility size) | Text capped at 2x multiplier. No overflow. Containers scroll if needed. Touch targets remain >= 44pt. |
| Reduce Motion enabled | Enable Reduce Motion in iOS Settings | All spring animations → crossfade or instant. Skeleton shimmer → static gray. No parallax. No bouncing. |
| Bold Text enabled | Enable Bold Text in iOS Settings | All text renders in bold variant. No layout breaks. |
| Increase Contrast enabled | Enable Increase Contrast in iOS Settings | Borders thicken (1px → 1.5px). Separator opacity increases. Placeholder text darkens. |
| Color contrast audit | Accessibility Inspector on every screen | All text passes WCAG AA: 4.5:1 for body, 3:1 for large text. All interactive elements: 3:1 against adjacent colors. |
| Switch Control compatibility | Enable Switch Control, navigate key flows | App is operable via Switch Control. No trapped focus states. |

### 5.6 Security Evaluation

| Check | Method | Pass Criteria |
|---|---|---|
| Token storage | Inspect Keychain after login | Tokens are in Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`. Not in UserDefaults, not in MMKV, not in AsyncStorage. |
| Token not in logs | Search all `console.log`, `Sentry.captureMessage`, and network logs | No access token, refresh token, or auth code appears in any log output. |
| Certificate pinning (if implemented) | Use Charles Proxy with custom CA | API requests fail when proxy intercepts (certificate mismatch). |
| Jailbreak detection (if implemented) | Test on jailbroken device or simulator flag | App shows warning or refuses to operate on compromised device. |
| SQLite encryption | Inspect `skye.db` file on disk | If using SQLCipher: file is unreadable without key. If not encrypted: accepted risk documented. |
| Sensitive data in screenshots | Background app, check app switcher screenshot | No PII visible in app switcher preview (blur or blank overlay applied). |
| Clipboard handling | Copy a token/password, check clipboard after 60s | Sensitive clipboard content is cleared after 60 seconds (if implemented) or not copied at all. |

---

## Appendix A: How to Use This Document

1. **Before starting a phase:** Read the Problem Statement to ground yourself in why. Read the Constraint Architecture to know what you cannot break.
2. **When assigning work:** Use the Decomposition tables to identify tasks, their dependencies, and routing suggestions. Parallelize where marked.
3. **When reviewing a PR:** Check the relevant Acceptance Criteria. Every criterion is binary — it passes or it doesn't.
4. **When testing a build:** Follow the Evaluation Design: run benchmarks, probe edge cases, test on the device matrix, run the accessibility protocol.
5. **When deciding if something is in scope:** Check Section 3.5 (Out of Scope). If it's listed there, it's not V2.

## Appendix B: Acceptance Criteria Quick Reference

Total acceptance criteria: **60**

| Category | Criteria Count | IDs |
|---|---|---|
| Authentication | 6 | AC-1 through AC-6 |
| Navigation & UX | 8 | AC-7 through AC-14 |
| Chat Engine | 6 | AC-15 through AC-20 |
| CRM Screens | 6 | AC-21 through AC-26 |
| Offline Architecture | 6 | AC-27 through AC-32 |
| Camera & OCR | 6 | AC-33 through AC-38 |
| Push Notifications | 5 | AC-39 through AC-43 |
| Performance | 5 | AC-44 through AC-48 |
| Visual Design | 4 | AC-49 through AC-52 |
| App Store Compliance | 5 | AC-53 through AC-57 |
| Testing & CI | 3 | AC-58 through AC-60 |

---

*This document is the operating layer on top of the V2 spec. The spec says what to build. This document says how to verify it was built correctly, what to protect while building it, and how to break the work into executable units. When in doubt, this document governs process; the spec governs product.*
