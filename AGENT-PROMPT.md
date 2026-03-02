# Skye V2 — Agent Handoff Prompt

---

You are building the Skye iOS native app. Skye is an AI-powered CRM for real estate agents, currently live as a Next.js web app. Your job is to build the React Native iOS companion app that connects to the same backend.

## Source Documents

You have two governing documents. Read both before doing anything.

**1. The Spec (what to build):**
https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/README.md

This is a 20,000-line, 12-section production specification covering every screen, every animation curve, every design token, every API contract, every offline behavior, and every compliance requirement for the Skye iOS app. It is the single source of truth for product decisions. When the spec says something, that's what gets built. When the spec is silent, default to Apple Human Interface Guidelines.

**2. The Handoff Guide (how to build it, how to verify it):**
https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/HANDOFF.md

This is the operating manual. It contains:
- **Problem Statement** — Why we're building this and what user problems it solves. Read this to understand intent, not just implementation.
- **60 Acceptance Criteria** (AC-1 through AC-60) — Every criterion is binary pass/fail with specific evidence required. Nothing ships unless all applicable ACs pass. These are your QA checklist.
- **Constraint Architecture** — What you cannot break: frozen Supabase schema, locked RLS policies, 129 existing API routes, CI-enforced performance budgets, technology decisions that are irreversible. Also lists what's explicitly out of scope.
- **Decomposition** — Every task broken into phases (0-5), with files to create/modify, dependency ordering, parallelization flags, and agent routing. Use this to plan your work.
- **Evaluation Design** — Regression areas, 12 edge cases to probe, performance benchmarks, device test matrix, accessibility protocol, security checks.

## Architecture Context

```
┌──────────────────┐    ┌──────────────────┐
│   Web App         │    │  React Native    │  <-- YOU ARE BUILDING THIS
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

| Layer | Choice |
|---|---|
| Runtime | React Native 0.76+ with Expo SDK 52+ |
| JS Engine | Hermes (bytecode compilation, mandatory) |
| Navigation | React Navigation 6+ (native stack) |
| Server State | TanStack Query v5 (`networkMode: 'offlineFirst'`) |
| Client State | Zustand v4 with persist middleware |
| Local Database | expo-sqlite (WAL mode) |
| Secure Storage | expo-secure-store (iOS Keychain) |
| Fast KV | react-native-mmkv |
| Camera | react-native-vision-camera v4+ |
| Images | expo-image (with blurhash placeholders) |
| Animations | react-native-reanimated v3 + react-native-gesture-handler v2 |
| Lists | @shopify/flash-list |
| Bottom Sheets | @gorhom/bottom-sheet v5 |
| Push | APNs-direct (server) + @notifee/react-native (client) |
| Error Monitoring | @sentry/react-native |

## Build Phases

Work proceeds in this order. Do not skip phases. Each phase has entry and exit criteria defined in the spec (Section 12) and decomposed into tasks in the Handoff Guide (Section 4).

| Phase | What | Key Dependency |
|---|---|---|
| **Phase 0** | Pre-development setup: Apple Developer, Google OAuth, Supabase RLS audit, design system, repo init, backend endpoints | Nothing — do this first |
| **Phase 1** | Foundation: Expo scaffold, all dependencies, theme system, storage adapters, SQLite, TanStack Query, Zustand stores, API client, auth flow, navigation shell, Sentry | Phase 0 complete |
| **Phase 2** | Core screens: People list (FlashList), Contact Detail, Property List, Property Detail, mutations with optimistic updates, offline queue, cache eviction | Phase 1 complete |
| **Phase 2 (parallel)** | Push notifications: APNs registration, notification handlers, deep link resolution, badges, categories | Phase 1 complete |
| **Phase 3** | Chat engine: SSE streaming, token buffer, message list, tool-call cards, typing indicator, chat input bar | Phase 2 complete |
| **Phase 4A** | Camera & OCR: VisionCamera, permission flow, capture pipeline, OCR upload, review screen, duplicate detection | Phase 2 complete |
| **Phase 4B (parallel)** | Studio: content type forms, SSE generation, markdown rendering, quick tweaks, copy/share, history | Phase 2 complete |
| **Phase 5** | Polish: performance audit, bundle optimization, accessibility audit, privacy manifest, App Store submission | All prior phases complete |

## Rules of Engagement

1. **Read the spec section before writing code for that section.** The spec has exact values — animation spring configs, color hex codes, touch target sizes, debounce timings, cache TTLs. Use them verbatim.

2. **Check acceptance criteria before marking work done.** The Handoff Guide has 60 ACs. Find the ones relevant to your task. They're binary pass/fail.

3. **Respect constraints.** Don't modify Supabase schema. Don't modify existing API routes. Don't add dependencies not in the locked stack without explicit approval. Don't build out-of-scope features (see Handoff Guide Section 3.5 for the full list).

4. **Performance budgets are CI gates, not suggestions.** Cold launch < 2s. Bundle < 15MB. 60fps scroll. < 150MB memory with 500 contacts. These are enforced in CI. If your code violates them, it doesn't ship.

5. **Offline-first is not optional.** Every screen that displays server data must work offline from the moment it's built. Data goes to SQLite during fetch. Queries fall back to SQLite when offline. Mutations queue to SQLite and replay on reconnect. This is part of the definition of done for every screen.

6. **Accessibility is not a Phase 5 task.** VoiceOver labels, touch targets, Dynamic Type support, and Reduce Motion handling are built into every component from day one. The spec (Section 1.6) defines the exact VoiceOver reading order for every screen.

7. **No raw values in component code.** Colors come from semantic tokens (`colors.text.primary`, not `#000000`). Typography comes from the type scale (`typography.body`, not `fontSize: 17`). Spacing comes from the spacing scale. Animation springs come from the named presets (`Springs.snappy`, not `{ damping: 20, stiffness: 300 }`).

8. **Test as you build.** Unit tests are co-located (`foo.ts` → `foo.test.ts`). Coverage thresholds are enforced: `src/lib/**` >= 88% lines, `src/hooks/**` >= 78%, `src/stores/**` >= 68%.

## When You're Stuck

- **"What should this look like?"** → Read the spec section. It has wireframes, exact dimensions, color tokens, and state machines for every screen.
- **"What endpoint do I call?"** → The spec sections reference specific API routes. The backend has 129 existing routes plus 6 new ones listed above.
- **"Is this in scope?"** → Check Handoff Guide Section 3.5 (Out of Scope). If it's listed, don't build it.
- **"How do I test this?"** → Check Handoff Guide Section 5 (Evaluation Design) for edge cases, benchmarks, and the device test matrix.
- **"What does 'done' mean for this task?"** → Find the matching AC numbers in Handoff Guide Section 2. Every AC has evidence requirements.

## Quick Reference Links

| Document | URL |
|---|---|
| V2 Spec (full) | https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/README.md |
| Handoff Guide | https://raw.githubusercontent.com/nmsitaras-afk/skye-native-app-spec-v2/main/HANDOFF.md |
| GitHub Repo | https://github.com/nmsitaras-afk/skye-native-app-spec-v2 |

---

*Start with Phase 0. Read the spec. Read the handoff guide. Build what the spec says. Verify against the acceptance criteria. Ship it.*
