# Skye Native App Spec V2 (React Native / iOS)

> **V2 Produced by 12-Specialist Agent System.** Supersedes V1.5. Every section has been independently researched against current Apple Human Interface Guidelines, Expo SDK 52, React Native 0.76, and 2025 App Store Review Guidelines. This document is the single source of truth for the Skye iOS app build.

> **Architecture Principle:** One backend. Two frontends. All intelligence server-side. The native app is a window into Skye, not a copy of Skye.

```
┌──────────────────┐    ┌──────────────────┐
│   Web App         │    │  React Native    │
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
         │  (129 routes)     │
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
    │  (DB)   │      │ Haiku     │
    │  (Auth) │      │ Sonnet    │
    │  (RLS)  │      │ Opus      │
    └─────────┘      └───────────┘
```

---

## Table of Contents

1. [Section 1: UX Architecture & Interaction Design](#section-1-ux-architecture--interaction-design)
2. [Section 2: Visual Design System](#section-2-visual-design-system)
3. [Section 3: Chat Engine & AI Interface](#section-3-chat-engine--ai-interface)
4. [Section 4: CRM Screens — People, Contact Detail, Properties](#section-4-crm-screens--people-contact-detail-properties)
5. [Section 5: Authentication, Security & Data Protection](#section-5-authentication-security--data-protection)
6. [Section 6: Push Notifications](#section-6-push-notifications)
7. [Section 7: Performance, Infrastructure & Build Pipeline](#section-7-performance-infrastructure--build-pipeline)
8. [Section 8: Offline Architecture & State Management](#section-8-offline-architecture--state-management)
9. [Section 9: Camera, OCR, Media & Studio](#section-9-camera-ocr-media--studio)
10. [Section 10: App Store Compliance & Submission](#section-10-app-store-compliance--submission)
11. [Section 11: Testing Strategy, QA & Acceptance Criteria](#section-11-testing-strategy-qa--acceptance-criteria)
12. [Section 12: Build Roadmap & Phased Implementation](#section-12-build-roadmap--phased-implementation)

---


## Section 1: UX Architecture & Interaction Design

**Document Version:** 2.0.0
**Last Updated:** 2026-03-02
**Status:** APPROVED FOR IMPLEMENTATION
**Owner:** UX Architecture Lead
**Target Platform:** iOS 16+ (React Native)
**Design Language:** Apple HIG-native, Skye brand overlay

---

### Table of Contents

1. Interaction Design System
2. Navigation Flow Diagrams
3. Screen State Machines
4. Keyboard and Input Behavior
5. Scroll Behavior and Performance
6. Accessibility-First Design
7. Responsive Layout Rules

---

### 1. INTERACTION DESIGN SYSTEM

#### 1.1 Micro-Interaction Vocabulary

Every interactive element in Skye follows a strict interaction vocabulary. No interaction may exist that is not catalogued below.

##### 1.1.1 Tap Interactions

| Element | Tap Behavior | Visual Feedback | Duration |
|---------|-------------|-----------------|----------|
| Primary Button | Scale to 0.97, opacity to 0.9, execute action on release | Background darkens 10% | Press: 60ms, Release: 120ms |
| Secondary Button | Scale to 0.98, execute action on release | Border color intensifies 20% | Press: 40ms, Release: 100ms |
| Ghost/Text Button | Opacity to 0.6, execute action on release | None (opacity only) | Press: 0ms (instant), Release: 150ms |
| List Row | Background highlight (systemGray5), execute on release | Row background shifts to highlight color | Press: 0ms (instant), Release: 200ms fade |
| Tab Bar Icon | Scale to 0.90 then spring to 1.0, switch tab | Icon fills/activates with spring animation | 300ms total spring |
| Avatar/Profile Image | Scale to 0.95, navigate to profile | Subtle shadow increase | Press: 60ms, Release: 100ms |
| FAB (Floating Action) | Scale to 0.92, shadow elevation increase, execute | Shadow radius 4 -> 8, color brightens 5% | Press: 80ms, Release: 200ms spring |
| Card Element | Scale to 0.98, slight shadow lift, navigate | Shadow offset Y: 2 -> 6, opacity 0.98 | Press: 100ms, Release: 150ms |
| Chip/Tag | Scale to 0.95, toggle state | Background color crossfade | 200ms |
| Icon Button (toolbar) | Opacity to 0.4, execute on release | None (opacity only) | Press: 0ms, Release: 150ms |
| Toggle/Switch | Track slides with gesture, snaps to state | Track color crossfade, thumb slides | 250ms spring |
| Segmented Control | Selected segment slides with spring | Background indicator translates | 350ms spring |

##### 1.1.2 Long-Press Interactions

| Element | Trigger Threshold | Behavior | Visual Feedback |
|---------|-------------------|----------|-----------------|
| Contact List Row | 500ms | Context menu appears (Call, Message, Email, Edit, Delete) | Row scales to 0.97, blurs background at 200ms mark |
| Property Card | 500ms | Context menu (Share, Edit, Archive, View on Map) | Card lifts with shadow, background blurs |
| Chat Message (own) | 500ms | Context menu (Copy, Edit, Delete, Reply) | Message bubble scales to 0.97, haptic at trigger |
| Chat Message (AI) | 500ms | Context menu (Copy, Regenerate, Share, Save to Notes) | Message bubble scales to 0.97, haptic at trigger |
| Studio Content Item | 500ms | Context menu (Share, Download, Edit, Delete, Duplicate) | Item lifts, background dims to 0.6 opacity |
| Image/Photo | 500ms | Preview popover (ala Peek), options on further press | Image zooms to 1.1 scale in context |
| Phone Number | 300ms | Copy to clipboard (shorter threshold for utility) | Flash highlight, toast confirmation |
| Email Address | 300ms | Copy to clipboard | Flash highlight, toast confirmation |
| URL/Link | 300ms | Preview + Copy option | Link text highlights in brand color |

All long-press interactions MUST include haptic feedback at the trigger threshold (see Section 1.2).

##### 1.1.3 Swipe Interactions

| Element | Swipe Direction | Distance Threshold | Behavior |
|---------|----------------|-------------------|----------|
| Contact List Row | Left (partial, 80pt) | 80pt to reveal | Reveal action buttons: Call (green), Delete (red) |
| Contact List Row | Left (full, >180pt) | 180pt to trigger | Execute primary trailing action (Call) |
| Contact List Row | Right (partial, 80pt) | 80pt to reveal | Reveal action button: Message (blue) |
| Contact List Row | Right (full, >180pt) | 180pt to trigger | Execute primary leading action (Message) |
| Property List Row | Left (partial) | 80pt to reveal | Archive (yellow), Share (blue) |
| Chat Message | Right (partial, 60pt) | 60pt to trigger | Reply to message (spring snap back) |
| Screen (from left edge) | Right (from 0-20pt edge) | 50% screen width or velocity > 500pt/s | Navigate back (interactive pop) |
| Modal/Bottom Sheet | Down | 100pt or velocity > 800pt/s | Dismiss modal (interactive) |
| Image Gallery | Left/Right | 50% width or velocity > 300pt/s | Navigate to next/previous image |
| Notification Banner | Up | 40pt | Dismiss notification |
| Notification Banner | Down | 60pt | Expand notification details |
| Settings Toggle Row | N/A (not swipeable) | N/A | Settings rows are tap-only, no swipe actions |

**Swipe Mechanics:**
- All swipe actions use `react-native-gesture-handler` `Swipeable` or custom `PanGestureHandler`
- Swipe reveal actions have a rubberband effect beyond their snap point: `overshootLeft: false, overshootRight: false` for action reveals
- Full-swipe actions have a commit threshold at which point the row auto-completes the action with a spring animation
- Velocity-based detection: if horizontal velocity > 500pt/s at release, the swipe completes regardless of distance
- Opposite-direction interruption: starting a swipe in the opposite direction of an open action immediately closes the open action

##### 1.1.4 Pull-to-Refresh

| Screen | Pull Distance | Trigger Threshold | Animation |
|--------|--------------|-------------------|-----------|
| People (Contact List) | 0-120pt elastic | 80pt to trigger | Skye logo mark rotates 360 degrees, scales from 0.5 to 1.0 |
| Properties List | 0-120pt elastic | 80pt to trigger | Same branded animation |
| Chat History | 0-120pt elastic (inverted) | 80pt to trigger | Loading older messages, skeleton rows appear at top |
| Studio Gallery | 0-120pt elastic | 80pt to trigger | Same branded animation |
| Settings | N/A | N/A | No pull-to-refresh (static content) |

**Branded Pull-to-Refresh Animation Specification:**
- Implementation: Custom `PanGestureHandler` + Reanimated shared values (NOT default `RefreshControl`)
- Phase 1 (0-40pt pull): Skye icon fades in from 0 to 1 opacity, positioned at pull distance Y
- Phase 2 (40-80pt pull): Skye icon rotates proportionally to pull distance (0 to 180 degrees), scales from 0.6 to 1.0
- Phase 3 (80pt+ trigger): Icon snaps to center, begins continuous rotation at 1 revolution per 800ms
- Phase 4 (refresh complete): Icon scales to 1.2, haptic success, fades out over 200ms, content springs into place
- Rubberband physics: `translationY * 0.5` beyond trigger threshold (content resists further pull)
- Color: Brand primary color icon on semi-transparent (`rgba(0,0,0,0.03)`) background circle
- Implementation library: Reanimated shared values + Lottie for the icon animation sequence

##### 1.1.5 Shake Gesture

| Context | Behavior | Confirmation |
|---------|----------|-------------|
| Any screen | Presents "Report a Problem" / "Undo" action sheet | System alert with options: "Report a Problem", "Undo Last Action" (if applicable), "Cancel" |
| Chat (after send) | Shake within 3 seconds of sending -> Undo Send option | Undo confirmation with 2-second auto-dismiss countdown |

Detection: Use `react-native-shake` or accelerometer-based detection with threshold of 2.5g over 500ms window.

##### 1.1.6 Pinch-to-Zoom

| Element | Behavior | Bounds |
|---------|----------|--------|
| Property Street View | Zoom 1.0x to 4.0x with pan | Clamp at bounds, rubberband at edges, double-tap to toggle 1x/2x |
| Property Photos | Zoom 1.0x to 5.0x with pan | Full-screen takeover at >1.5x, dismiss on pinch back to <1.0x |
| Studio Generated Images | Zoom 1.0x to 3.0x with pan | Same as Property Photos |
| Profile Photos | Zoom 1.0x to 3.0x | Same as Property Photos |
| Chat Images | Zoom 1.0x to 4.0x with pan | Full-screen with dark background, dismiss on pinch to <1.0x |

**Zoom Implementation:**
- Use `PinchGestureHandler` composed with `PanGestureHandler` via `Gesture.Simultaneous()`
- Scale and translation are Reanimated shared values on the UI thread
- When zoom > 1.0x, disable parent scroll (`simultaneousHandlers` must reference parent scroll)
- Double-tap: animated zoom to 2.0x centered on tap point, or back to 1.0x if already zoomed
- Double-tap animation: `withSpring(targetScale, { damping: 20, stiffness: 200 })`

---

#### 1.2 Haptic Feedback Map

Skye uses iOS haptic feedback through `expo-haptics` (or `react-native-haptic-feedback` for bare workflow). Every haptic is mapped to a specific semantic action category. Haptics MUST be called synchronously from the UI thread where possible (use Reanimated `runOnJS` bridge if needed).

**Pre-warming:** Call `Haptics.prepare()` (or the equivalent `prepare()` on the native feedback generator) when a screen mounts that will use haptics, to eliminate the ~50ms cold-start latency of the Taptic Engine.

##### 1.2.1 Impact Feedback (Physical Metaphor)

| Haptic Type | Trigger | Semantic Meaning |
|-------------|---------|-----------------|
| `impactLight` | Tab bar icon tap | Lightweight UI element selection |
| `impactLight` | Chip/tag selection | Small element state change |
| `impactLight` | Toggle switch flip | Lightweight binary state change |
| `impactLight` | Segmented control change | Selection within a group |
| `impactLight` | Pull-to-refresh crosses trigger threshold | Threshold acknowledgment |
| `impactMedium` | Button tap (Primary, Secondary) | Standard interactive element activation |
| `impactMedium` | Card tap/press | Navigational element activation |
| `impactMedium` | Swipe action reveal (actions become visible) | Physical "landing" of revealed actions |
| `impactMedium` | Bottom sheet snap to point | Sheet settling at a snap position |
| `impactHeavy` | Long-press trigger (context menu appears) | Significant UI state change |
| `impactHeavy` | Full-swipe action commit | Destructive or significant action commitment |
| `impactHeavy` | FAB tap | High-importance action initiation |

##### 1.2.2 Selection Feedback

| Haptic Type | Trigger | Semantic Meaning |
|-------------|---------|-----------------|
| `selection` | Scrolling through a picker wheel | Detent-by-detent selection change |
| `selection` | Dragging a slider to discrete values | Value change at each discrete step |
| `selection` | Reordering list items (as item crosses boundary) | Item has crossed a positional threshold |
| `selection` | Scrubbing through dates in a calendar | Each date boundary crossing |

##### 1.2.3 Notification Feedback

| Haptic Type | Trigger | Semantic Meaning |
|-------------|---------|-----------------|
| `notificationSuccess` | Message sent successfully | Positive completion |
| `notificationSuccess` | Contact saved | Data persistence confirmed |
| `notificationSuccess` | Content generated in Studio | AI task completed |
| `notificationSuccess` | Pull-to-refresh complete | Refresh completed with new data |
| `notificationSuccess` | Property saved/favorited | Positive action committed |
| `notificationWarning` | Form validation error (inline) | Non-blocking issue requires attention |
| `notificationWarning` | Network degraded (offline banner appears) | System state change requiring awareness |
| `notificationWarning` | Approaching character limit in chat input | Threshold warning |
| `notificationError` | Action failed (API error) | Operation could not complete |
| `notificationError` | Delete confirmation (at the moment of deletion) | Destructive action executed |
| `notificationError` | Authentication failure | Security-related failure |
| `notificationError` | Network request timeout | Connection failure |

##### 1.2.4 Haptic Rules

1. **Never double-fire:** If a single user action produces both a visual change and a navigation, fire haptic once (on the action, not the navigation).
2. **No haptics on passive events:** Content loading, background sync, time-based updates produce zero haptics.
3. **No haptics on scroll:** Scrolling through content must never produce haptic feedback (except picker wheels).
4. **Respect system settings:** Check `CHHapticEngine.capabilitiesForHardware()` and gracefully degrade on devices without Taptic Engine. If the device is in Low Power Mode, skip haptics.
5. **Timing:** Haptics must fire within 10ms of the triggering gesture. Any delay >50ms creates a perceptible disconnect.
6. **iPad:** iPads do not support haptic feedback. All haptic calls must be wrapped in a platform/device check that gracefully no-ops on iPad.

---

#### 1.3 Gesture Vocabulary

All gesture recognizers use `react-native-gesture-handler` v2+ with the declarative `Gesture` API. Gestures are composed using `Gesture.Simultaneous()`, `Gesture.Exclusive()`, and `Gesture.Race()` as appropriate.

##### 1.3.1 Standardized Gestures Across All Screens

| Gesture | Implementation | Behavior | Scope |
|---------|---------------|----------|-------|
| Edge Swipe Back | `gestureEnabled: true` on React Navigation stack | Interactive pop with parallax (current screen slides right, previous screen slides from -33% to 0%) | All stack-pushed screens |
| Swipe-to-Dismiss Modal | `PanGestureHandler` on modal container | Modal translates down with finger, opacity fades, dismiss on release past threshold | All modals and bottom sheets |
| Tap-Outside-to-Dismiss | `TapGestureHandler` on overlay/backdrop | Single tap on dimmed backdrop dismisses modal/sheet | All modals, bottom sheets, dropdowns |
| Pull-to-Refresh | Custom `PanGestureHandler` | Branded refresh animation (see 1.1.4) | Scrollable list screens |
| Pinch-to-Zoom | `PinchGestureHandler` + `PanGestureHandler` | Scale + translate image content | Image viewers only |
| Double-Tap-to-Zoom | `TapGestureHandler(numberOfTaps: 2)` | Toggle between 1x and 2x zoom | Image viewers only |
| Long-Press | `LongPressGestureHandler(minDurationMs: 500)` | Context menu / preview | List items, messages, images |

##### 1.3.2 Gesture Conflict Resolution

When multiple gesture recognizers exist on the same view hierarchy, conflicts are resolved as follows:

1. **Scroll vs. Swipe-to-Action:** Horizontal swipe on list rows is detected first. If the initial gesture direction is >30 degrees from horizontal, it falls through to the vertical scroll. Implementation: `activeOffsetX: [-10, 10]` on the swipe handler, `failOffsetX: [-10, 10]` on the scroll handler.

2. **Scroll vs. Edge Swipe Back:** The navigation edge swipe recognizer is limited to the leftmost 20pt of the screen. Any touch originating >20pt from the left edge is not captured by the back gesture. Implementation: React Navigation's default `gestureResponseDistance` set to 20.

3. **Pinch-to-Zoom vs. Scroll:** When a pinch gesture is detected (two fingers), scroll is disabled. When zoom > 1.0x, panning within the zoomed content takes priority over parent scroll. When zoom returns to 1.0x, parent scroll is re-enabled.

4. **Swipe-to-Dismiss vs. Scroll (Bottom Sheets):** Bottom sheet's `BottomSheetScrollView` handles this: when scrolled to top and dragging down, the sheet drag takes over from the scroll. Implementation handled by `@gorhom/bottom-sheet` internally.

5. **Long-Press vs. Swipe:** Long-press and swipe are mutually exclusive. If horizontal movement >10pt occurs during the long-press delay, the long-press is cancelled and swipe takes over. Implementation: `maxDist: 10` on the `LongPressGestureHandler`.

---

#### 1.4 Touch Target Sizes

Per [Apple HIG](https://developer.apple.com/design/human-interface-guidelines/), the absolute minimum touch target is 44x44pt. Skye enforces this strictly and exceeds it where possible.

| Element Type | Minimum Touch Target | Content Size | Padding Strategy |
|-------------|---------------------|-------------|-----------------|
| Primary Button | 44pt height, full-width or min 120pt wide | 17pt text, 16pt horizontal padding | `paddingVertical: 12, paddingHorizontal: 24` |
| Secondary Button | 44pt height, min 100pt wide | 15pt text | `paddingVertical: 10, paddingHorizontal: 20` |
| Icon Button (Toolbar) | 44x44pt | 22x22pt icon | Invisible padding extends hit area to 44x44 |
| Tab Bar Item | 49pt height (standard), width = screen / tabCount | 25x25pt icon + 10pt label | Centered in tab width |
| List Row | 56pt minimum height | Content dependent | `paddingVertical: 8, paddingHorizontal: 16` |
| Text Link (inline) | 44pt height (with invisible padding) | Text natural height | `paddingVertical: 10` invisible extension |
| Checkbox / Radio | 44x44pt | 22x22pt visible element | 11pt invisible padding per side |
| Close / Dismiss Button | 44x44pt | 15x15pt "X" icon | Center icon in 44x44 hit area |
| Back Button | 44x44pt | 12x20pt chevron + label | Standard UINavigationBar sizing |
| Search Bar | 44pt height | 34pt visible bar height | 5pt vertical padding |
| Chat Send Button | 44x44pt | 28x28pt icon | 8pt padding |
| Swipe Action Button | 74pt width x row height | 22pt icon + 12pt label | Centered in 74pt width |
| Avatar (tappable) | 44x44pt minimum | 36x36pt image | 4pt invisible padding |
| FAB | 56x56pt | 24x24pt icon | 16pt padding |
| Segmented Control | 44pt height | Per Apple HIG standard | Segments divide evenly |

**Implementation Rule:** Every `TouchableOpacity`, `Pressable`, or `GestureDetector` that wraps an interactive element MUST have `hitSlop` set to ensure the touch target is at least 44x44pt:

```typescript
const ensureMinTouchTarget = (contentWidth: number, contentHeight: number) => ({
  hitSlop: {
    top: Math.max(0, (44 - contentHeight) / 2),
    bottom: Math.max(0, (44 - contentHeight) / 2),
    left: Math.max(0, (44 - contentWidth) / 2),
    right: Math.max(0, (44 - contentWidth) / 2),
  },
});
```

---

#### 1.5 Spring Animation Curves

All spring animations in Skye use `react-native-reanimated` v3's `withSpring()`. Below are the exact configurations for every animation type. Engineers MUST use these values verbatim. No ad-hoc spring configurations are permitted.

##### 1.5.1 Named Spring Presets

```typescript
// springs.ts -- Canonical spring configurations
// Import and use ONLY these presets. Do not create ad-hoc springs.

export const Springs = {
  // Snappy -- Used for small, immediate UI responses (button presses, toggles)
  snappy: {
    damping: 20,
    stiffness: 300,
    mass: 0.8,
    overshootClamping: false,
    restDisplacementThreshold: 0.01,
    restSpeedThreshold: 2,
  },

  // Gentle -- Used for content transitions (page pushes, card reveals)
  gentle: {
    damping: 20,
    stiffness: 120,
    mass: 1,
    overshootClamping: false,
    restDisplacementThreshold: 0.01,
    restSpeedThreshold: 2,
  },

  // Bouncy -- Used for playful elements (FAB press, tab icon, empty state illustrations)
  bouncy: {
    damping: 12,
    stiffness: 150,
    mass: 0.9,
    overshootClamping: false,
    restDisplacementThreshold: 0.01,
    restSpeedThreshold: 2,
  },

  // Heavy -- Used for large elements (bottom sheet, modal, full-screen transitions)
  heavy: {
    damping: 28,
    stiffness: 200,
    mass: 1.2,
    overshootClamping: false,
    restDisplacementThreshold: 0.01,
    restSpeedThreshold: 2,
  },

  // Sheet -- Used specifically for bottom sheet snap animations
  sheet: {
    damping: 30,
    stiffness: 250,
    mass: 0.9,
    overshootClamping: true,
    restDisplacementThreshold: 0.01,
    restSpeedThreshold: 2,
  },

  // Navigation -- Used for screen push/pop transitions
  navigation: {
    damping: 25,
    stiffness: 180,
    mass: 1,
    overshootClamping: true,
    restDisplacementThreshold: 0.01,
    restSpeedThreshold: 2,
  },

  // Keyboard -- Used for keyboard avoidance translations
  keyboard: {
    damping: 30,
    stiffness: 250,
    mass: 0.8,
    overshootClamping: true,
    restDisplacementThreshold: 0.01,
    restSpeedThreshold: 2,
  },

  // Micro -- Used for very small, subtle animations (icon state changes, indicator dots)
  micro: {
    damping: 15,
    stiffness: 400,
    mass: 0.5,
    overshootClamping: false,
    restDisplacementThreshold: 0.001,
    restSpeedThreshold: 0.5,
  },
} as const;
```

##### 1.5.2 Spring Usage Map

| Animation | Spring Preset | Animated Property | From | To |
|-----------|--------------|-------------------|------|-----|
| Button press down | `snappy` | `transform.scale` | 1.0 | 0.97 |
| Button release | `bouncy` | `transform.scale` | 0.97 | 1.0 |
| Tab bar icon activate | `bouncy` | `transform.scale` | 0.90 | 1.0 |
| Tab bar indicator slide | `snappy` | `transform.translateX` | Previous position | New position |
| Screen push (forward) | `navigation` | `transform.translateX` | Screen width | 0 |
| Screen pop (back) | `navigation` | `transform.translateX` | 0 | Screen width |
| Screen push (previous screen) | `navigation` | `transform.translateX` | 0 | -screenWidth * 0.33 |
| Modal present (bottom sheet rise) | `heavy` | `transform.translateY` | Screen height | Snap point |
| Modal dismiss | `sheet` | `transform.translateY` | Current position | Screen height |
| Bottom sheet snap | `sheet` | `transform.translateY` | Current | Target snap point |
| Card press | `snappy` | `transform.scale` | 1.0 | 0.98 |
| Card release | `gentle` | `transform.scale` | 0.98 | 1.0 |
| FAB press | `bouncy` | `transform.scale` | 1.0 | 0.92 |
| FAB release | `bouncy` | `transform.scale` | 0.92 | 1.0 |
| Toast enter | `bouncy` | `transform.translateY` | -80 | 0 |
| Toast exit | `gentle` | `opacity` + `translateY` | 1.0, 0 | 0, -40 |
| Skeleton shimmer | N/A (timing) | `opacity` | 0.3 | 0.7 (loop, 1000ms) |
| Content fade-in | N/A (timing) | `opacity` | 0 | 1 (200ms ease-out) |
| List item enter (staggered) | `gentle` | `opacity` + `translateY` | 0, 20 | 1, 0 (stagger: 50ms) |
| Swipe action reveal | `snappy` | `transform.translateX` | 0 | -80 (action width) |
| Swipe action snap back | `bouncy` | `transform.translateX` | Current | 0 |
| Keyboard avoidance | `keyboard` | `transform.translateY` | 0 | -keyboardHeight |
| Segmented control indicator | `snappy` | `transform.translateX` + `width` | Previous segment | Selected segment |
| Toggle switch thumb | `snappy` | `transform.translateX` | 0 or 20 | 20 or 0 |
| Pull-to-refresh icon rotation | N/A (timing) | `transform.rotate` | 0deg | 360deg (800ms linear loop) |
| Context menu scale-in | `micro` | `transform.scale` + `opacity` | 0.8, 0 | 1.0, 1.0 |
| Empty state illustration | `bouncy` | `transform.scale` + `opacity` | 0.9, 0 | 1.0, 1.0 |
| Header collapse (scroll-linked) | N/A (interpolation) | `height` + `opacity` | Expanded | Collapsed (scroll-driven) |

##### 1.5.3 Timing Animation Exceptions

Some animations use `withTiming()` rather than springs, for predictability:

```typescript
export const Timings = {
  // Opacity transitions -- springs overshoot on opacity looks wrong
  fadeIn: { duration: 200, easing: Easing.out(Easing.ease) },
  fadeOut: { duration: 150, easing: Easing.in(Easing.ease) },

  // Skeleton shimmer loop
  shimmer: { duration: 1000, easing: Easing.inOut(Easing.ease) },

  // Color transitions (background, text color changes)
  colorChange: { duration: 250, easing: Easing.out(Easing.ease) },

  // Scroll-linked interpolations use interpolate() with Extrapolation.CLAMP
  // -- not withTiming/withSpring; they are driven directly by scroll position
} as const;
```

##### 1.5.4 Reduce Motion Overrides

When `useReducedMotion()` returns `true`:

| Standard Behavior | Reduced Motion Replacement |
|-------------------|---------------------------|
| Spring scale animations | Instant state change (no animation) |
| Slide transitions (push/pop) | Cross-dissolve: `withTiming(opacity, { duration: 250 })` |
| Bottom sheet spring to snap | `withTiming(position, { duration: 300 })` -- no spring, no overshoot |
| Bouncy elements (FAB, tab icon) | Simple opacity change, no scale |
| Pull-to-refresh rotation | Static icon (no rotation), progress indicated by opacity |
| Staggered list entry | Instant render (no stagger, no translate) |
| Skeleton shimmer | Solid placeholder color (no shimmer animation) |
| Auto-scrolling content | Static display |
| Parallax effects | No parallax (static positioning) |

Implementation: Wrap all animations in a utility function:

```typescript
import { useReducedMotion } from 'react-native-reanimated';

export const useAdaptiveSpring = (
  targetValue: number,
  springConfig: SpringConfig,
  reducedMotionDuration: number = 250
) => {
  const reducedMotion = useReducedMotion();
  return reducedMotion
    ? withTiming(targetValue, { duration: reducedMotionDuration })
    : withSpring(targetValue, springConfig);
};
```

---

### 2. NAVIGATION FLOW DIAGRAMS

#### 2.1 Complete Navigation Graph

##### 2.1.1 Root Architecture

```
AppRoot
+-- AuthGate (conditional)
|   +-- [NOT_AUTHENTICATED] -> AuthStack (Modal presentation)
|   |   +-- Welcome Screen
|   |   +-- Sign In (Email + Password)
|   |   +-- Sign In (Apple)
|   |   +-- Sign Up
|   |   +-- Forgot Password
|   |   +-- Email Verification
|   |
|   +-- [AUTHENTICATED] -> MainTabNavigator
|       +-- Tab 1: Chat (AI Assistant)
|       |   +-- ChatStack
|       |       +-- ChatList (conversation history)
|       |       +-- ChatConversation (streaming AI chat)
|       |       +-- -> ContactDetail (push, from mentioned contact)
|       |       +-- -> PropertyDetail (push, from mentioned property)
|       |       +-- -> DocumentPreview (modal, from shared document)
|       |
|       +-- Tab 2: People (CRM)
|       |   +-- PeopleStack
|       |       +-- PeopleList (searchable contact list)
|       |       +-- ContactDetail (full contact profile)
|       |       |   +-- -> EditContact (modal bottom sheet, snap: 60%, 90%)
|       |       |   +-- -> AddNote (modal bottom sheet, snap: 40%, 70%)
|       |       |   +-- -> ActivityLog (push)
|       |       |   +-- -> PropertyDetail (push, from linked properties)
|       |       |   +-- -> ChatConversation (push, "Ask AI about this contact")
|       |       |   +-- -> Phone/SMS/Email (system action, external)
|       |       +-- AddContact (modal, full screen)
|       |       +-- ImportContacts (modal, full screen)
|       |       +-- FilterContacts (modal bottom sheet, snap: 50%, 85%)
|       |
|       +-- Tab 3: Properties
|       |   +-- PropertiesStack
|       |       +-- PropertiesList (searchable property list)
|       |       +-- PropertyDetail
|       |       |   +-- -> StreetView (push, full screen with zoom)
|       |       |   +-- -> PhotoGallery (modal, full screen with zoom/swipe)
|       |       |   +-- -> EditProperty (modal bottom sheet, snap: 60%, 90%)
|       |       |   +-- -> LinkedContacts (push)
|       |       |   +-- -> ShareProperty (modal bottom sheet, snap: 40%)
|       |       |   +-- -> MapView (push)
|       |       |   +-- -> ChatConversation (push, "Ask AI about this property")
|       |       +-- AddProperty (modal, full screen)
|       |       +-- FilterProperties (modal bottom sheet, snap: 50%, 85%)
|       |
|       +-- Tab 4: Studio (Content Generation)
|       |   +-- StudioStack
|       |       +-- StudioHome (gallery of generated content)
|       |       +-- CreateContent (push)
|       |       |   +-- -> SelectTemplate (push)
|       |       |   +-- -> SelectProperty (modal bottom sheet, snap: 60%, 90%)
|       |       |   +-- -> SelectContact (modal bottom sheet, snap: 60%, 90%)
|       |       |   +-- -> PreviewContent (push)
|       |       +-- ContentDetail (push)
|       |       |   +-- -> ShareContent (system share sheet)
|       |       |   +-- -> EditContent (push)
|       |       |   +-- -> FullScreenPreview (modal, full screen)
|       |       +-- ContentHistory (push)
|       |
|       +-- Tab 5: Settings
|           +-- SettingsStack
|               +-- SettingsHome
|               +-- ProfileSettings (push)
|               |   +-- -> EditProfile (push)
|               +-- NotificationSettings (push)
|               +-- SubscriptionSettings (push)
|               |   +-- -> PaywallModal (modal, full screen)
|               +-- IntegrationSettings (push)
|               |   +-- -> IntegrationDetail (push)
|               +-- AppearanceSettings (push)
|               +-- PrivacySettings (push)
|               +-- DataExport (push)
|               +-- HelpAndSupport (push)
|               |   +-- -> WebView (push)
|               +-- About (push)
|               +-- DeleteAccount (modal, full screen with confirmation)
|
+-- Global Overlays (presented over any tab)
|   +-- IncomingCallBanner (custom overlay, not a screen)
|   +-- NetworkOfflineBanner (custom overlay, not a screen)
|   +-- ToastNotification (custom overlay, not a screen)
|   +-- ForceUpdateModal (modal, full screen, non-dismissible)
|
+-- Deep Link Handler (see 2.2)
```

##### 2.1.2 Back-Navigation Path Rules

| Screen | Back Button Target | Gesture Back Target | Tab Memory |
|--------|-------------------|-------------------|------------|
| ChatConversation | ChatList | ChatList | Preserves conversation scroll position |
| ContactDetail | PeopleList | PeopleList | Preserves list scroll position + search state |
| ContactDetail (from Chat link) | ChatConversation | ChatConversation | Cross-tab: returns to Chat tab |
| PropertyDetail | PropertiesList | PropertiesList | Preserves list scroll position |
| PropertyDetail (from Contact link) | ContactDetail | ContactDetail | Returns to referring contact |
| PropertyDetail (from Chat link) | ChatConversation | ChatConversation | Cross-tab: returns to Chat tab |
| EditContact (bottom sheet) | Dismiss sheet | Swipe down dismisses | No navigation impact |
| AddContact (modal) | Dismiss modal | Swipe down dismisses | If saved: PeopleList with new contact at top |
| Any Settings sub-screen | Parent Settings screen | Parent Settings screen | Settings stack unaffected by tab switching |

**Rule:** When navigating cross-tab (e.g., from Chat to a Contact Detail), the target screen is pushed onto the originating tab's stack. It does NOT switch tabs. This preserves the user's mental model of "where they are."

---

#### 2.2 Deep Link Resolution Map

##### 2.2.1 URL Scheme

```
skye://                         -> App root (current tab)
skye://chat                     -> Chat tab, ChatList
skye://chat/{conversationId}    -> Chat tab, ChatConversation
skye://people                   -> People tab, PeopleList
skye://people/{contactId}       -> People tab, ContactDetail
skye://properties               -> Properties tab, PropertiesList
skye://properties/{propertyId}  -> Properties tab, PropertyDetail
skye://studio                   -> Studio tab, StudioHome
skye://studio/{contentId}       -> Studio tab, ContentDetail
skye://settings                 -> Settings tab, SettingsHome
skye://settings/profile         -> Settings tab, ProfileSettings
skye://settings/subscription    -> Settings tab, SubscriptionSettings
```

##### 2.2.2 Push Notification Deep Link Resolution

| Notification Type | Payload Key | Target Screen | Fallback Screen | Auth Gate |
|-------------------|-------------|--------------|----------------|-----------|
| `new_message` | `conversationId` | ChatConversation | ChatList (if conversationId invalid) | YES -- queue deep link, present auth, navigate after success |
| `contact_reminder` | `contactId` | ContactDetail | PeopleList (if contactId not found) | YES |
| `property_update` | `propertyId` | PropertyDetail | PropertiesList (if propertyId not found) | YES |
| `studio_complete` | `contentId` | ContentDetail | StudioHome (if contentId not found) | YES |
| `subscription_expiring` | none | SubscriptionSettings | SettingsHome | YES |
| `system_announcement` | `url` (optional) | WebView or SettingsHome | SettingsHome | YES (but show announcement regardless) |
| `referral_invite` | `referralCode` | Sign Up (with pre-filled code) | Welcome Screen | NO -- auth gate not required for sign up flow |

##### 2.2.3 Deep Link Resolution Logic

```
1. App receives deep link (URL or notification tap)
2. Parse link into { screen, params }
3. Check authentication state:
   a. IF authenticated:
      - Check if target screen exists in navigation state
      - IF resource ID provided: validate resource exists (API call with 3s timeout)
        - IF valid: navigate to target screen with params
        - IF invalid (404): navigate to fallback screen, show toast "Item not found"
        - IF timeout: navigate to target screen (optimistic), let screen handle error state
      - IF no resource ID: navigate to target screen
   b. IF NOT authenticated:
      - Store deep link in AsyncStorage: { pendingDeepLink: { screen, params, timestamp } }
      - Present AuthStack
      - After successful authentication:
        - Read pendingDeepLink from AsyncStorage
        - IF timestamp < 15 minutes old: execute navigation
        - IF timestamp >= 15 minutes old: discard, navigate to default tab
        - Clear pendingDeepLink from AsyncStorage
4. App state handling:
   a. App killed (cold start): getInitialURL() + getInitialNotification()
   b. App backgrounded: Linking.addEventListener + onNotificationOpenedApp()
   c. App foregrounded: onForegroundEvent listener
```

---

#### 2.3 Tab-Switching State Preservation

| Tab | Preserved State on Tab Switch Away | Restored State on Tab Return |
|-----|-----------------------------------|------------------------------|
| Chat | Full stack (all pushed screens remain), scroll positions, draft message text in input, conversation scroll position | Full stack restored, scroll positions restored, draft message persists, keyboard state: dismissed (never auto-show keyboard on tab return) |
| People | Full stack, list scroll position, active search query text, active filter state, sort order | Full stack restored, scroll position restored. Search bar state: if was active, remains active with query text but keyboard dismissed. Filter badges persist. |
| Properties | Full stack, list scroll position, active search query, filter state, map vs list view toggle state | Full stack restored, scroll position restored. Map/List toggle state preserved. |
| Studio | Full stack, gallery scroll position, in-progress content generation (continues in background) | Full stack restored. If generation completed while away, shows completion badge on tab icon. |
| Settings | Full stack, scroll positions | Full stack restored. No special persistence needed. |

**Implementation:** React Navigation's default tab behavior preserves the stack. Enhancement:
- `unmountOnBlur: false` on all tabs (default, but explicitly set)
- `freezeOnBlur: true` on all tabs (React Native Screens optimization -- prevents re-renders while tab is backgrounded)
- Scroll positions stored in React refs (not state) to avoid unnecessary re-renders
- Draft text stored in a Zustand store slice keyed by screen identifier

---

#### 2.4 Modal vs. Push Navigation Decisions

##### 2.4.1 Decision Framework

```
Should this screen be MODAL or PUSH?

1. Is this a self-contained task with a clear Save/Cancel outcome?
   -> YES -> MODAL (bottom sheet or full-screen modal)

2. Is the user drilling deeper into a content hierarchy?
   -> YES -> PUSH (stack navigation with back button)

3. Does the user need to reference the screen behind this one?
   -> YES -> BOTTOM SHEET (partial modal)
   -> NO  -> Full-screen push or full-screen modal

4. Can the task be completed in <3 taps?
   -> YES -> BOTTOM SHEET (40-60% snap)

5. Does the task require significant scrollable content or a form with many fields?
   -> YES -> FULL-SCREEN MODAL (with close button and save/done)

6. Is this a destructive or irreversible action confirmation?
   -> YES -> ALERT DIALOG (not a screen)

7. Is this selecting from a list to populate a field on the previous screen?
   -> YES -> BOTTOM SHEET (60-90% snap) with search
```

##### 2.4.2 Complete Modal/Push Classification

| Screen Transition | Presentation Type | Justification |
|-------------------|-------------------|---------------|
| PeopleList -> ContactDetail | **Push** | Drilling into hierarchy, needs back navigation context |
| ContactDetail -> EditContact | **Bottom Sheet** (60%, 90%) | Self-contained edit task, user may reference profile behind |
| ContactDetail -> AddNote | **Bottom Sheet** (40%, 70%) | Quick task, benefits from seeing contact context |
| PeopleList -> AddContact | **Full-Screen Modal** | Multi-field form requiring focus, Save/Cancel outcome |
| PeopleList -> ImportContacts | **Full-Screen Modal** | Complex multi-step flow, needs full screen |
| PeopleList -> FilterContacts | **Bottom Sheet** (50%, 85%) | Quick selection task, user references list behind |
| PropertiesList -> PropertyDetail | **Push** | Drilling into hierarchy |
| PropertyDetail -> PhotoGallery | **Full-Screen Modal** | Immersive media experience requiring full screen |
| PropertyDetail -> EditProperty | **Bottom Sheet** (60%, 90%) | Self-contained edit, reference property behind |
| PropertyDetail -> ShareProperty | **Bottom Sheet** (40%) | Quick action with few options |
| PropertyDetail -> StreetView | **Push** | Content exploration, needs back navigation |
| ChatList -> ChatConversation | **Push** | Drilling into hierarchy, frequent back-and-forth |
| ChatConversation -> DocumentPreview | **Full-Screen Modal** | Immersive document view |
| StudioHome -> CreateContent | **Push** | Multi-step wizard, needs back navigation through steps |
| StudioHome -> ContentDetail | **Push** | Drilling into hierarchy |
| ContentDetail -> FullScreenPreview | **Full-Screen Modal** | Immersive preview |
| SettingsHome -> Any sub-screen | **Push** | Standard drill-down hierarchy |
| Settings -> DeleteAccount | **Full-Screen Modal** | Destructive action requiring focused confirmation |
| Any -> System Share Sheet | **System Modal** | iOS system-provided |
| Any -> Phone Call | **System Action** | iOS system-provided |

##### 2.4.3 Bottom Sheet Specifications

All bottom sheets use `@gorhom/bottom-sheet` v5 with Reanimated v3 and Gesture Handler v2.

| Bottom Sheet Context | Snap Points | Initial Snap | Enable Pan-to-Close | Handle Style |
|---------------------|-------------|-------------|---------------------|-------------|
| EditContact | ['60%', '90%'] | 0 (60%) | Yes | Visible grab handle, 36x5pt, 4pt border-radius |
| AddNote | ['40%', '70%'] | 0 (40%) | Yes | Visible grab handle |
| FilterContacts | ['50%', '85%'] | 0 (50%) | Yes | Visible grab handle |
| FilterProperties | ['50%', '85%'] | 0 (50%) | Yes | Visible grab handle |
| ShareProperty | ['40%'] | 0 (40%) | Yes | Visible grab handle |
| SelectProperty (Studio) | ['60%', '90%'] | 0 (60%) | Yes | Visible grab handle |
| SelectContact (Studio) | ['60%', '90%'] | 0 (60%) | Yes | Visible grab handle |
| Quick Actions (from FAB) | ['CONTENT_HEIGHT'] | 0 | Yes | Visible grab handle |

**Bottom Sheet Shared Configuration:**

```typescript
const bottomSheetDefaults = {
  enableDynamicSizing: false,
  enablePanDownToClose: true,
  enableOverDrag: true,
  overDragResistanceFactor: 6,
  animationConfigs: Springs.sheet,
  backdropComponent: BottomSheetBackdrop, // dims to rgba(0,0,0,0.4)
  backgroundStyle: {
    backgroundColor: Colors.surface,
    borderTopLeftRadius: 16,
    borderTopRightRadius: 16,
  },
  handleIndicatorStyle: {
    backgroundColor: Colors.handleIndicator, // rgba(0,0,0,0.2)
    width: 36,
    height: 5,
    borderRadius: 2.5,
  },
};
```

---

#### 2.5 Gesture-Based Navigation

##### 2.5.1 Interactive Screen Pop (Edge Swipe Back)

```typescript
const stackScreenOptions = {
  gestureEnabled: true,
  gestureDirection: 'horizontal',
  gestureResponseDistance: 20, // Only 20pt from left edge triggers back gesture
  cardStyleInterpolator: ({ current, next, layouts }) => ({
    cardStyle: {
      transform: [
        {
          translateX: current.progress.interpolate({
            inputRange: [0, 1],
            outputRange: [layouts.screen.width, 0],
          }),
        },
      ],
      shadowColor: '#000',
      shadowOffset: { width: -3, height: 0 },
      shadowOpacity: current.progress.interpolate({
        inputRange: [0, 1],
        outputRange: [0, 0.15],
      }),
      shadowRadius: 10,
    },
    overlayStyle: {
      opacity: current.progress.interpolate({
        inputRange: [0, 1],
        outputRange: [0, 0.07],
      }),
    },
  }),
  transitionSpec: {
    open: { animation: 'spring', config: Springs.navigation },
    close: { animation: 'spring', config: Springs.navigation },
  },
};
```

**Interactive pop behavior:**
- Gesture is velocity-sensitive: if finger velocity > 500pt/s at release, pop completes regardless of distance
- If finger has traveled > 50% of screen width at release, pop completes
- If finger has traveled < 50% and velocity < 500pt/s, pop cancels (screen springs back)
- During gesture: previous screen is visible with parallax offset, dimming overlay on previous screen decreases proportionally

##### 2.5.2 Interactive Modal Dismissal

```typescript
const modalDismissConfig = {
  velocityThreshold: 800,     // Downward velocity > 800pt/s dismisses regardless
  distanceThreshold: 0.35,    // Translated > 35% of modal height dismisses
  overDragResistanceFactor: 6, // Rubberband resistance when dragging beyond top
};

const fullScreenModalDismiss = {
  gestureEnabled: true,
  gestureDirection: 'vertical',
  cardStyleInterpolator: ({ current }) => ({
    cardStyle: {
      opacity: current.progress.interpolate({
        inputRange: [0, 0.5, 0.9, 1],
        outputRange: [0, 0.25, 0.7, 1],
      }),
      transform: [
        {
          scale: current.progress.interpolate({
            inputRange: [0, 1],
            outputRange: [0.93, 1],
          }),
        },
      ],
    },
    overlayStyle: {
      opacity: current.progress.interpolate({
        inputRange: [0, 1],
        outputRange: [0, 0.5],
      }),
    },
  }),
};
```

---

### 3. SCREEN STATE MACHINES

#### 3.1 Universal State Machine

Every data-driven screen in Skye implements the following state machine. No screen may deviate from this pattern without documented justification.

```
                        +-------------+
                        |    IDLE     |  (Screen mounted, no data request yet)
                        +------+------+
                               | onMount / onFocus (if stale)
                               v
                        +-------------+
                   +--->|  LOADING    |  (First load, show skeleton)
                   |    +------+------+
                   |           |
                   |    +------+------------------+
                   |    |                         |
                   |    v                         v
                   | +-------------+     +-------------+
                   | |   LOADED   |     |    EMPTY    |  (0 items)
                   | | (has data) |     +------+------+
                   | +------+------+           |
                   |        |                  | user action (add item)
                   |        |                  +-------> LOADING
                   |        |
                   |   +----+----------------+
                   |   |                     |
                   |   v                     v
                   | +-------------+  +-------------+
                   | | REFRESHING |  |   STALE     |
                   | | (pull-to-  |  +------+------+
                   | |  refresh)  |         |
                   | +------+------+        +-------> REFRESHING
                   |        |
                   |   +----+----------+
                   |   |               |
                   |   v               v
                   | LOADED     +-------------+
                   |            |   ERROR     |
                   |            +------+------+
                   |                   | retry
                   |                   +-------> LOADING (no cache) / REFRESHING (has cache)
                   |
                   |    +-------------+
                   +----+  OFFLINE    |
                        +------+------+
                               | network restored
                               +-------> REFRESHING (has cache) / LOADING (no cache)
```

##### 3.1.1 State Definitions

| State | Definition | UI Behavior |
|-------|-----------|-------------|
| `IDLE` | Screen is mounted but has not initiated a data fetch | Instantaneous, transitions to LOADING on mount. Users should never perceive this state. |
| `LOADING` | First data fetch in progress, no cached data available | Show skeleton screen (see 3.3). No spinner. No blank screen. |
| `LOADED` | Data successfully fetched and rendered | Full content visible. Pull-to-refresh enabled. |
| `EMPTY` | Data fetch succeeded but returned zero items | Empty state illustration + message + primary CTA (e.g., "Add your first contact"). |
| `REFRESHING` | Data refresh in progress, stale/cached data still visible | Current data remains visible. If pull-to-refresh: branded animation runs. If background refresh: subtle inline indicator (not full skeleton). |
| `STALE` | Cached data is older than staleness threshold, refresh needed | Identical to LOADED visually, but auto-triggers REFRESHING on next focus. |
| `ERROR` | Data fetch failed | If has cached data: show cached data + inline error banner at top ("Couldn't refresh. Tap to retry."). If no cached data: full-screen error state with illustration + message + retry button. |
| `OFFLINE` | Device has no network connectivity | If has cached data: show cached data + persistent offline banner at top. If no cached data: full-screen offline state with illustration + "You're offline" message. |

##### 3.1.2 Staleness Thresholds

| Screen | Staleness Threshold | Auto-Refresh Trigger |
|--------|--------------------|--------------------|
| ChatConversation | 30 seconds | On focus (tab switch or app foreground) |
| ChatList | 60 seconds | On focus |
| PeopleList | 5 minutes | On focus |
| ContactDetail | 2 minutes | On focus |
| PropertiesList | 5 minutes | On focus |
| PropertyDetail | 2 minutes | On focus |
| StudioHome | 5 minutes | On focus |
| Settings screens | Never stale | Manual refresh only (where applicable) |

##### 3.1.3 Error Retry Strategy

```
Attempt 1: Immediate retry
Attempt 2: After 2 second delay
Attempt 3: After 5 second delay
After 3 failures: Stop auto-retry. Show persistent error state with manual "Retry" button.
```

Each retry uses exponential backoff. During retry delay, show a subtle progress indicator (not full skeleton) and text "Retrying..." beneath the error message.

---

#### 3.2 Per-Screen State Machines

##### 3.2.1 Chat Conversation Screen

```
States: IDLE -> LOADING -> LOADED -> STREAMING -> ERROR -> OFFLINE

Additional states specific to Chat:
- STREAMING: AI is generating a response (tokens arriving via SSE/WebSocket)
  UI: Message bubble appears immediately with typing indicator,
      then text streams in token-by-token. User can scroll up during stream.
      "Stop Generating" button appears below streaming message.
- WAITING_FOR_AI: User message sent, waiting for AI response to begin
  UI: User message appears immediately (optimistic), typing indicator
      pulses in AI message area. Duration: typically 500ms-3s.

State transitions:
- User sends message -> LOADED (optimistic user message) -> WAITING_FOR_AI -> STREAMING -> LOADED
- Network loss during STREAMING -> Show partial response + "Response interrupted" label + Retry button
- Error during WAITING_FOR_AI -> Show failed message with retry icon on user's message bubble
```

##### 3.2.2 People List Screen

```
States: IDLE -> LOADING -> LOADED/EMPTY -> REFRESHING -> ERROR -> OFFLINE

Additional states:
- SEARCHING: User has activated search, keyboard is visible
  UI: List filters in real-time (debounced 300ms). Empty search state:
      "No contacts match [query]" with suggestion to broaden search.
- FILTERING: User has active filters applied
  UI: Filter badge count on filter button. Inline chip bar showing active filters.
      "Clear all" option. Empty filtered state: "No contacts match these filters."
```

##### 3.2.3 Contact Detail Screen

```
Sub-sections load independently:
1. Header + Contact Info (from cache: instant; from API: skeleton for 200-500ms)
2. Activity Timeline (skeleton, loads within 500ms-1s)
3. Linked Properties (skeleton, loads within 500ms-1.5s)
4. Notes (skeleton, loads within 500ms-1.5s)

Each section independently transitions through LOADING -> LOADED/EMPTY/ERROR.
The screen as a whole is LOADED when section 1 (header) is loaded.
```

##### 3.2.4 Property Detail Screen

```
Sub-sections load independently:
1. Header (address, price, hero image): Loads first (from list cache if available)
2. Property details (beds, baths, sqft, description): Loads with header
3. Street View thumbnail: Loads second (separate image load)
4. Photo gallery thumbnails: Loads third (lazy, loads as user scrolls to section)
5. Linked contacts: Loads fourth (separate API)
6. Market data: Loads fifth (separate API, may be slowest)
```

##### 3.2.5 Studio Home Screen

```
Additional states:
- GENERATING: Content generation is in progress
  UI: Generating item appears at top of gallery with indeterminate progress bar
      + "Generating your listing description..." text.
      Persists if user navigates away and returns.
- GENERATED: Content just completed generation
  UI: Item at top of gallery pulses with brand highlight ring for 3 seconds,
      success haptic fires if user is on this screen when generation completes.
```

---

#### 3.3 Skeleton Screen Specifications

Every skeleton screen MUST match the exact layout of its loaded state. Skeleton placeholders must be the same size and position as the content they replace.

##### 3.3.1 Skeleton Visual Style

```typescript
const SkeletonStyle = {
  backgroundColor: 'rgba(0, 0, 0, 0.06)', // Light mode
  // backgroundColor: 'rgba(255, 255, 255, 0.08)', // Dark mode
  borderRadius: 4,
  shimmerHighlight: 'rgba(0, 0, 0, 0.02)',
  shimmerDuration: 1000, // ms per cycle
  shimmerDelay: 0,
};
```

Shimmer animation: A linear gradient sweeps left-to-right across the skeleton at 45 degrees. When Reduce Motion is enabled, skeleton is solid gray with no shimmer.

##### 3.3.2 Per-Screen Skeleton Layouts

**People List Skeleton:**
```
+--------------------------------+
| [Search Bar Skeleton]          |  <- 36pt height, full width - 32pt margin
|                                |
| [O] ===============           |  <- 56pt row: 40pt circle avatar + name line
|     ============               |     Name: 140pt width, 14pt height
|     =========                  |     Subtitle: 100pt width, 12pt height
|--------------------------------|
| [O] ===============           |  <- Repeat 8 rows (fills typical viewport)
|     ============               |
|     =========                  |
|--------------------------------|
|        (repeat x6 more)       |
+--------------------------------+
```

**Contact Detail Skeleton:**
```
+--------------------------------+
|          [  O  ]               |  <- 80pt circle avatar, centered
|       ================         |  <- Name: 160pt centered, 18pt height
|         ============           |  <- Role: 120pt centered, 14pt height
|          ==========            |  <- Company: 100pt centered, 14pt height
|                                |
| [O]      [O]      [O]         |  <- Action buttons: 3x 44pt circles
|                                |
| -- SECTION DIVIDER --          |
| ========                       |  <- Section header: 80pt, 12pt height
| =======================        |  <- Detail rows
| ===================            |
+--------------------------------+
```

**Properties List Skeleton:**
```
+--------------------------------+
| [Search Bar Skeleton]          |
|                                |
| +----------------------------+ |  <- Property card: 200pt height
| | ========================== | |     Hero image placeholder: 140pt
| |                            | |
| | ================           | |     Address: 180pt
| | ============               | |     Price: 120pt
| | ===  ===  ====             | |     Beds/Baths/Sqft
| +----------------------------+ |
|                                |
| +----------------------------+ |  <- Repeat 3 cards
| | ========================== | |
| | ================           | |
| | ============               | |
| +----------------------------+ |
+--------------------------------+
```

**Chat Conversation Skeleton:**
```
+--------------------------------+
|                                |
|          ================      |  <- AI message bubble (left-aligned)
|          ============          |
|                                |
|    ================            |  <- User message bubble (right-aligned)
|                                |
|          ====================  |  <- AI message (left)
|          ==================    |
|          ============          |
|                                |
+--------------------------------+
| [Message Input Bar]            |  <- Input bar is NOT skeleton; real and interactive immediately
+--------------------------------+
```

##### 3.3.3 Progressive Loading Transitions

Content replaces skeleton with a crossfade:
- Skeleton element fades out: `withTiming(0, { duration: 150 })` opacity
- Real content fades in: `withTiming(1, { duration: 200, delay: 50 })` opacity
- The 50ms overlap creates a seamless crossfade, not a flash
- If a section loads in <100ms, skip the skeleton entirely (show content directly) to avoid flicker
- Staggered sections: each section's skeleton crossfades independently as its data arrives

---

### 4. KEYBOARD AND INPUT BEHAVIOR

#### 4.1 Keyboard Avoidance Strategy

Skye uses `react-native-keyboard-controller` as the primary keyboard management library across all screens. This replaces the built-in `KeyboardAvoidingView` for consistent behavior and smooth, native-matched keyboard animations.

##### 4.1.1 Global Setup

```typescript
// In root _layout.tsx / App.tsx
import { KeyboardProvider } from 'react-native-keyboard-controller';

export default function RootLayout() {
  return (
    <KeyboardProvider>
      <GestureHandlerRootView style={{ flex: 1 }}>
        <NavigationContainer>
          {/* App content */}
        </NavigationContainer>
      </GestureHandlerRootView>
    </KeyboardProvider>
  );
}
```

##### 4.1.2 Per-Screen Keyboard Behavior

| Screen | Input Elements | Avoidance Strategy | Keyboard Type |
|--------|---------------|-------------------|---------------|
| Chat Conversation | Message TextInput (expandable) | `padding` mode: content pushes up, input bar stays above keyboard | `default` |
| AddContact | Name, Phone, Email, Address, Notes | `padding` mode with `KeyboardAwareScrollView`: focused field scrolls into view | `default`, `phone-pad`, `email-address` per field |
| EditContact | Same as AddContact | Same as AddContact | Same as AddContact |
| AddProperty | Address, Price, Description, etc. | `padding` mode with `KeyboardAwareScrollView` | `default`, `decimal-pad` for price |
| Search (People) | Search TextInput | No avoidance needed (search bar is at top, list shrinks) | `default` |
| Search (Properties) | Search TextInput | No avoidance needed | `default` |
| Sign In | Email, Password | `padding` mode: form scrolls up to keep active field + submit button visible | `email-address`, `default` |
| Sign Up | Name, Email, Password, Confirm | `padding` mode with scroll | `default`, `email-address`, `default`, `default` |
| AddNote | Note TextInput | `padding` mode (bottom sheet context: sheet pushes up above keyboard) | `default` |
| Studio CreateContent | Prompt TextInput | `padding` mode: prompt field + generate button remain visible | `default` |
| Settings (Profile Edit) | Name, Bio, etc. | `padding` mode with `KeyboardAwareScrollView` | `default` |

##### 4.1.3 Keyboard Dismiss Behavior

| Trigger | Behavior |
|---------|----------|
| Tap outside text input | Dismiss keyboard (global `Keyboard.dismiss()` on background tap) |
| Scroll list | Dismiss keyboard on scroll begin (`keyboardDismissMode: 'on-drag'`) |
| Tab switch | Dismiss keyboard (fires `Keyboard.dismiss()` on `tabPress` listener) |
| Back navigation | Dismiss keyboard (handled by navigation transition) |
| Bottom sheet drag | Dismiss keyboard (handled by `@gorhom/bottom-sheet` `keyboardBehavior: 'interactive'`) |
| Swipe-to-dismiss modal | Dismiss keyboard at start of gesture |

---

#### 4.2 Chat Input: Expanding Text Input Behavior

The chat message input is the most complex input in the app. It must match the behavior of Apple Messages.

```typescript
const ChatInputConfig = {
  multiline: true,
  minHeight: 36,              // Single line height
  maxHeight: 120,             // Maximum expansion (approximately 5 lines of text)
  lineHeight: 22,             // Per-line height at default font size
  fontSize: 16,               // Body text size
  paddingVertical: 8,
  paddingHorizontal: 12,
  borderRadius: 20,           // Pill shape when single-line
  // borderRadius adjusts: 20 at 1 line, 16 at 2 lines, 12 at 3+ lines

  // Expansion behavior:
  // - TextInput grows height dynamically based on content
  // - Uses onContentSizeChange to track intrinsic content height
  // - Height is animated with Springs.keyboard
  // - When content exceeds maxHeight, internal scroll is enabled
  // - scrollEnabled transitions from false -> true when content > maxHeight

  // Send button behavior:
  // - Send button appears when text is non-empty (trim whitespace)
  // - Send button replaces voice/attachment button with crossfade (200ms)
  // - Send button is 30x30pt, positioned vertically centered to last line of text
  // - Send button has impactMedium haptic on tap
};
```

**Chat Input Layout (single line):**
```
+--------------------------------------------------+
| [+]  | Message...                        |  [>]  |
+--------------------------------------------------+
  |      |                                    |
  |      TextInput (expands upward)           Send (appears when text non-empty)
  Attachment button                           OR microphone (when text empty)
```

**When expanded (multi-line):**
```
+--------------------------------------------------+
| [+]  | This is a longer message that     |       |
|      | wraps to multiple lines. The      | [>]   |
|      | input grows upward.               |       |
+--------------------------------------------------+
```

When scrollable (content exceeds maxHeight):
- A subtle top border (1px, systemGray5) appears at the top of the TextInput to indicate scrollable content above
- Scroll indicator is visible inside the TextInput
- The input container stops growing; internal scroll takes over

---

#### 4.3 Search Input Behavior

| Property | Value | Rationale |
|----------|-------|-----------|
| Debounce timing | 300ms | Balances responsiveness with API/filter performance |
| Minimum query length | 1 character | Start filtering immediately; real-time feel |
| Cancel button | Appears when search bar is focused (animated slide-in from right, 200ms) | Apple HIG standard pattern |
| Cancel button tap | Clears text, dismisses keyboard, hides cancel button | Full reset |
| Clear button ("X") | Appears when text is non-empty, inside search bar at trailing edge | Clears text only, keyboard stays, search remains focused |
| Keyboard dismiss on scroll | Yes (`keyboardDismissMode: 'on-drag'`) | Standard iOS behavior |
| Search bar style | 36pt height visible, system gray background, 8pt border-radius, magnifying glass icon leading | Matches UISearchBar |
| Placeholder text | People: "Search contacts...", Properties: "Search properties...", Studio: "Search content..." | Context-specific |
| Voice search | Not supported in V2 | Planned for V3 |
| Search results | Local-first (filter cached list), then API (if query length >= 3 and local results < 5) | Instant local results + server augmentation |
| Empty search results | Illustration + "No results for '[query]'" + suggestion text | Never show a blank list |
| Recent searches | Show 5 most recent searches when search bar is focused and empty | Stored in AsyncStorage, clearable |

---

#### 4.4 Form Input Behavior

##### 4.4.1 Focus Order (Tab Order)

Forms follow a strict top-to-bottom, left-to-right focus order. The Return key on the keyboard advances to the next field.

**AddContact Form:**
```
1. First Name       -> Return key: "Next" -> focuses Last Name
2. Last Name        -> Return key: "Next" -> focuses Phone
3. Phone            -> Return key: "Next" -> focuses Email
4. Email            -> Return key: "Next" -> focuses Company
5. Company          -> Return key: "Next" -> focuses Role/Title
6. Role/Title       -> Return key: "Next" -> focuses Address
7. Address          -> Return key: "Next" -> focuses Notes
8. Notes (multiline)-> Return key: "Return" (new line) -> Submit via toolbar "Done" button
```

**Sign In Form:**
```
1. Email            -> Return key: "Next" -> focuses Password
2. Password         -> Return key: "Go"   -> submits form
```

**Sign Up Form:**
```
1. Full Name        -> Return key: "Next" -> focuses Email
2. Email            -> Return key: "Next" -> focuses Password
3. Password         -> Return key: "Next" -> focuses Confirm Password
4. Confirm Password -> Return key: "Join" -> submits form
```

Implementation: Each TextInput stores a `ref`. `onSubmitEditing` on field N calls `fieldRefs[N+1].current.focus()`. The last field either submits the form or (for multiline) inserts a newline.

##### 4.4.2 Return Key Configuration

| Field Type | `returnKeyType` | Behavior |
|-----------|-----------------|----------|
| Any field with a next field | `'next'` | Focus next field |
| Last single-line field in form | `'go'` or `'done'` or `'join'` | Submit form |
| Multiline field (Notes, Description) | `'default'` | Insert newline |
| Search field | `'search'` | Execute search (secondary, since real-time filtering is primary) |
| Chat input | `'default'` | Return inserts newline; send is button-based |

##### 4.4.3 Inline Validation Timing

| Validation Type | Trigger Timing | Visual Feedback |
|----------------|---------------|-----------------|
| Required field empty | On blur (field loses focus) only if user interacted with field | Red border, error text below: "This field is required" |
| Email format invalid | On blur + after 500ms of no typing (live) | Red border, error text: "Enter a valid email address" |
| Phone format invalid | On blur | Red border, error text: "Enter a valid phone number" |
| Password too short | Live (as user types) once >= 1 character typed | Strength indicator: red (<8), yellow (8-11), green (12+) |
| Password mismatch | On blur of confirm password field | Red border on confirm field: "Passwords don't match" |
| Duplicate contact | On blur of email or phone (async check) | Warning (yellow) border: "A contact with this email already exists" |

**Validation visual pattern:**
- Error state: border color transitions to `systemRed` (250ms), error text fades in below field (150ms), field label turns red
- Recovery: when error is corrected, border transitions back to default (250ms), error text fades out (150ms)
- Haptic: `notificationWarning` on first error appearance per field (not on every keystroke)

---

### 5. SCROLL BEHAVIOR AND PERFORMANCE

#### 5.1 List Optimization Strategy (500+ Items)

Skye uses `@shopify/flash-list` (FlashList) as the primary list component for all scrollable lists. FlatList is used only for lists guaranteed to have <50 items.

##### 5.1.1 FlashList Configuration by Screen

| Screen | List Component | `estimatedItemSize` | `drawDistance` | Item Type |
|--------|---------------|--------------------|----|-----------|
| PeopleList | FlashList | 64 | 250 | Uniform height contact rows |
| PropertiesList | FlashList | 220 | 300 | Uniform height property cards |
| ChatConversation | FlashList (inverted) | 80 | 500 | Variable height message bubbles |
| StudioHome | FlashList | 180 | 300 | Uniform height content cards |
| ChatList | FlashList | 72 | 250 | Uniform height rows |
| ActivityLog | FlashList | 56 | 200 | Uniform height rows |
| Search Results | FlashList | 64 | 200 | Uniform height rows |

##### 5.1.2 FlashList Performance Configuration

```typescript
// Image optimization:
// - All list images use react-native-fast-image with disk + memory cache
// - Avatar images: 40x40pt rendered size, source images max 80x80px (2x Retina)
// - Property thumbnails: exact rendered size, not full-resolution
// - Image placeholder: brand-colored background while loading

// Render optimization:
// - All list item components wrapped in React.memo()
// - Key extractors use stable IDs (never array index)
// - No inline function definitions in renderItem
// - Event handlers use useCallback with stable dependencies
// - Complex computations memoized with useMemo

const renderContactRow = useCallback(({ item }: { item: Contact }) => (
  <ContactRow contact={item} onPress={handleContactPress} />
), [handleContactPress]);

const ContactRow = React.memo(({ contact, onPress }: Props) => {
  // render
}, (prev, next) =>
  prev.contact.id === next.contact.id &&
  prev.contact.updatedAt === next.contact.updatedAt
);
```

##### 5.1.3 Section List Pattern (Alphabetical Contacts)

```
FlashList with sticky headers:
- stickyHeaderIndices computed from section positions
- Section headers: 16pt height, system gray background, bold 13pt label
- Section index (A-Z sidebar): custom component overlaid on right edge
  - Touch + drag scrolls to section (with selection haptic at each letter)
  - Sidebar letters: 10pt font, 16pt touch target height
  - Large letter overlay appears center-screen during sidebar scrub
```

---

#### 5.2 Scroll-Linked Animations

##### 5.2.1 Header Collapse (Large Title to Small Title)

Applicable screens: PeopleList, PropertiesList, StudioHome, ChatList, Settings

```typescript
const headerCollapseConfig = {
  // Large title height: 52pt (text) + 16pt padding = 68pt collapsible area
  // Navigation bar height: 44pt (standard) + status bar
  // Total header: 44 + 68 = 112pt expanded, 44pt collapsed

  // Scroll range: 0 -> 68pt of scroll triggers full collapse
  // Animation: scroll-linked interpolation (direct 1:1 mapping, no springs)

  expandedTitleFontSize: 34,   // SF Pro Display Bold
  collapsedTitleFontSize: 17,  // SF Pro Text Semibold

  // Large title translateY: interpolate(scrollY, [0, 68], [0, -68], CLAMP)
  // Large title opacity: interpolate(scrollY, [0, 40, 68], [1, 1, 0], CLAMP)
  // Small title opacity: interpolate(scrollY, [50, 68], [0, 1], CLAMP)
  // Search bar: scrolls with content, sticks when reaching nav bar bottom

  // Snap behavior: if scroll position between 20-48pt on release, snap to 0 or 68
  // Snap threshold: 34pt (midpoint). Below -> snap expanded. Above -> snap collapsed.
  // Snap animation: Springs.snappy
};
```

##### 5.2.2 Tab Bar Auto-Hide

Applicable: ChatConversation only (tab bar hides to maximize chat space)

```typescript
const tabBarAutoHideConfig = {
  // Trigger: scrolling DOWN by >20pt
  // Animation: tab bar translates down by 83pt (including safe area on notched iPhones)
  //   withTiming(83, { duration: 250, easing: Easing.out(Easing.ease) })

  // Trigger show: scrolling UP by >10pt OR reaching bottom of chat
  //   withTiming(0, { duration: 250, easing: Easing.out(Easing.ease) })

  // Hides ONLY in ChatConversation
  // On tab switch: tab bar always resets to visible

  // When tab bar is hidden, chat input extends to safe area bottom
  // Input bar animates bottom padding from tabBarHeight to safeAreaBottom
};
```

##### 5.2.3 FAB Behavior

Applicable: PeopleList, PropertiesList, StudioHome

```typescript
const fabScrollBehavior = {
  // Default: visible at bottom-right, 16pt from right, 16pt above tab bar
  // Scroll DOWN >30pt: FAB shrinks to mini (scale 0.7, opacity 0.8) via Springs.snappy
  // Scroll UP >10pt: FAB returns to full (scale 1.0, opacity 1.0)
  // At scroll top (offset < 50): always fully visible
  // At scroll bottom: always fully visible

  // FAB does NOT fully hide -- reduces to mini but remains tappable
  // Mini: icon only, reduced shadow
  // Full: icon + label (if applicable), full shadow
};
```

---

#### 5.3 Momentum Scrolling

All scrollable containers use `decelerationRate: 'normal'` (iOS default, 0.998).

```typescript
const scrollConfig = {
  decelerationRate: 'normal',     // Matches UIScrollView default
  showsVerticalScrollIndicator: true,
  showsHorizontalScrollIndicator: false,
  scrollIndicatorInsets: { right: 1 },

  // Horizontal (photo gallery):
  pagingEnabled: true,
  decelerationRate: 'fast',       // Snappier for paged content

  bounces: true,
  alwaysBounceVertical: true,
  scrollEventThrottle: 16,        // 60fps for Reanimated
};
```

---

#### 5.4 Pull-to-Refresh Implementation

See Section 1.1.4 for animation specification. Technical details:

```typescript
const pullToRefreshConfig = {
  // Custom PanGestureHandler (NOT RefreshControl)
  // Threshold: 80pt to trigger
  // Elastic resistance: beyond 80pt, 0.5x factor
  // Max pull: 160pt
  // Trigger: instant impactLight haptic
  // Minimum display time: 600ms (even if API returns faster)
  // Maximum display time: 15s (timeout, show error toast)
  // Complete: notificationSuccess haptic, content springs to top
  // Edge case: ignore pull while already refreshing
  // Edge case: quick pull-and-release before 80pt -> elastic snap back, no trigger
};
```

---

#### 5.5 Infinite Scroll / Pagination Strategy

##### 5.5.1 Contact List Pagination

```typescript
const contactListPagination = {
  pageSize: 50,
  prefetchThreshold: 0.7,      // onEndReachedThreshold={0.3}
  // Loading indicator: subtle ActivityIndicator at list bottom
  // End-of-list: "You've reached the end" (12pt, systemGray, centered)
  // Error during pagination: inline error row with "Tap to retry"
  // Cursor-based pagination: { cursor: lastItemId, limit: 50 }
};
```

##### 5.5.2 Chat History Pagination (Inverted)

```typescript
const chatHistoryPagination = {
  pageSize: 30,
  prefetchThreshold: 0.8,      // 80% toward oldest
  direction: 'backward',

  // FlashList inverted, onEndReached fires when scrolling UP
  // Skeleton rows appear at top during load
  // maintainVisibleContentPosition preserves scroll position on prepend

  // Initial load: 30 most recent messages
  // Auto-scroll to bottom on initial load
  // New message: auto-scroll ONLY if within 100pt of bottom
  //   Otherwise: "New message" floating badge (tap to scroll)
  // End-of-history: "This is the beginning of your conversation with Skye"
};
```

##### 5.5.3 Properties List Pagination

```typescript
const propertiesListPagination = {
  pageSize: 20,                 // Fewer (larger cards with images)
  prefetchThreshold: 0.6,      // Earlier trigger for larger items
  // Prefetch hero images for next page via react-native-fast-image preload()
};
```

---

### 6. ACCESSIBILITY-FIRST DESIGN

#### 6.1 VoiceOver Navigation Order

VoiceOver reads elements in a defined order on each screen. This order MUST be tested on a physical device (not simulator) and MUST match the visual reading order.

##### 6.1.1 Per-Screen VoiceOver Order

**People List:**
```
1. Navigation bar title ("People") -- accessibilityRole: "header"
2. Search bar -- accessibilityLabel: "Search contacts", accessibilityRole: "search"
3. Filter button -- accessibilityLabel: "Filter contacts, [N] active filters"
4. Section header (e.g., "A") -- accessibilityRole: "header"
5. Contact row -- grouped as single element
   accessibilityLabel: "[Name], [Company], [Role]"
   accessibilityHint: "Double tap to view contact details"
6. [Repeat for each contact row]
7. [Repeat for each section]
8. Tab bar -- each tab individually focusable
   accessibilityLabel: "[Tab name], tab, [position] of 5"
   accessibilityState: { selected: true/false }
```

**Contact Detail:**
```
1. Back button -- accessibilityLabel: "Back to People"
2. More options button
3. Avatar image -- accessibilityLabel: "[Name]'s profile photo"
4. Name -- accessibilityRole: "header"
5. Role / Company
6. Call button -- accessibilityLabel: "Call [Name]"
   accessibilityHint: "Double tap to call [phone number]"
7. Message button -- accessibilityLabel: "Message [Name]"
8. Email button -- accessibilityLabel: "Email [Name]"
9. Section header ("Contact Info") -- accessibilityRole: "header"
10. Phone number -- accessibilityLabel: "Phone: [number]"
11. Email -- accessibilityLabel: "Email: [address]"
12. Address
13. Section header ("Activity") -- accessibilityRole: "header"
14. Activity items (each grouped)
15. Section header ("Notes")
16. Note items
17. Section header ("Properties")
18. Linked property items
```

**Chat Conversation:**
```
1. Back button -- "Back to conversations"
2. Title -- "Conversation with Skye AI"
3. Messages (newest first in VoiceOver order):
   Each: "[Sender] said: [message text], [relative time]"
4. Message input -- "Message input"
   accessibilityHint: "Type a message to Skye"
5. Send button -- "Send message"
   accessibilityState: { disabled: [true if empty] }
6. Attachment button -- "Add attachment"
```

**Property Detail:**
```
1. Back button
2. Share button -- "Share this property"
3. Hero image -- "Property photo, [address]"
4. Address -- accessibilityRole: "header"
5. Price -- "Price: [formatted price]"
6. Stats row -- "[N] bedrooms, [N] bathrooms, [N] square feet"
   (grouped as single element)
7. Description
8. Section headers and content
9. Street View -- "Street View of [address]"
   accessibilityHint: "Double tap to expand"
10. Photo gallery -- "Property photos, [count] photos"
```

---

#### 6.2 Accessibility Grouping

| Element Group | Container `accessible={true}` | Combined Label |
|-------------|------|---------------|
| Contact list row | Entire row | "[Name], [Company], [Role]" |
| Property card | Entire card | "[Address], [Price], [Beds] beds, [Baths] baths" |
| Chat message bubble | Bubble + timestamp | "[Sender] said: [text], [time]" |
| Property stats | Row grouped | "[N] bedrooms, [N] bathrooms, [N] square feet" |
| Action button group (Call/Msg/Email) | NOT grouped -- each individually focusable | Individual labels |
| Form field + label + error | Grouped | "[Label], [current value], [error if present], text field" |
| Tab bar item | Icon + label grouped | "[Tab name], tab, [position] of [total], [selected/not selected]" |
| Section header + count | Grouped | "[Section name], [count] items" |
| Toggle row (Settings) | Row + toggle grouped | "[Setting name], [state: on/off], switch" |

**Key rule:** Interactive sub-elements within a grouped element are NOT individually focusable. Secondary actions must be exposed via `accessibilityActions`.

---

#### 6.3 Custom VoiceOver Actions for Complex Elements

##### 6.3.1 Swipeable List Items

Swipe gestures are not accessible to VoiceOver users. Swipe actions must be exposed as custom accessibility actions:

```typescript
const contactRowAccessibility = {
  accessible: true,
  accessibilityRole: 'button' as const,
  accessibilityLabel: `${contact.name}, ${contact.company}, ${contact.role}`,
  accessibilityHint: 'Double tap to view details. Use actions for more options.',
  accessibilityActions: [
    { name: 'activate', label: 'View details' },
    { name: 'call', label: `Call ${contact.name}` },
    { name: 'message', label: `Message ${contact.name}` },
    { name: 'email', label: `Email ${contact.name}` },
    { name: 'delete', label: `Delete ${contact.name}` },
  ],
  onAccessibilityAction: (event) => {
    switch (event.nativeEvent.actionName) {
      case 'activate': navigateToContact(contact.id); break;
      case 'call': callContact(contact.phone); break;
      case 'message': messageContact(contact.phone); break;
      case 'email': emailContact(contact.email); break;
      case 'delete': confirmDeleteContact(contact.id); break;
    }
  },
};
```

##### 6.3.2 Chat Messages

```typescript
const chatMessageAccessibility = (message, isOwnMessage) => ({
  accessible: true,
  accessibilityRole: 'text',
  accessibilityLabel: `${isOwnMessage ? 'You' : 'Skye'} said: ${message.text}, ${formatRelativeTime(message.timestamp)}`,
  accessibilityActions: [
    { name: 'copy', label: 'Copy message text' },
    ...(isOwnMessage
      ? [{ name: 'edit', label: 'Edit message' }, { name: 'delete', label: 'Delete message' }]
      : [{ name: 'regenerate', label: 'Regenerate response' }, { name: 'share', label: 'Share response' }]
    ),
    { name: 'reply', label: 'Reply to this message' },
  ],
});
```

##### 6.3.3 Property Photos

```typescript
const photoAccessibility = (index, total) => ({
  accessible: true,
  accessibilityRole: 'image',
  accessibilityLabel: `Property photo ${index + 1} of ${total}`,
  accessibilityHint: 'Double tap to view full screen. Swipe left or right for more.',
  accessibilityActions: [
    { name: 'activate', label: 'View full screen' },
    { name: 'share', label: 'Share photo' },
    { name: 'save', label: 'Save to camera roll' },
  ],
});
```

---

#### 6.4 Reduce Motion Support

When "Reduce Motion" is enabled in iOS Settings, Skye detects via `useReducedMotion()` from `react-native-reanimated`:

| Standard Motion | Reduced Motion Replacement |
|----------------|---------------------------|
| Screen push (slide from right) | Cross-dissolve (200ms fade) |
| Screen pop (slide to right) | Cross-dissolve (200ms fade) |
| Modal present (slide from bottom) | Fade in (200ms) |
| Modal dismiss (slide to bottom) | Fade out (150ms) |
| Bottom sheet spring to snap | Linear timing (300ms, no overshoot) |
| Tab switch bounce | Instant state change |
| Button press scale | Opacity change only (1.0 to 0.7) |
| Pull-to-refresh rotation | Static progress icon, fill animation |
| Skeleton shimmer | Solid gray placeholder |
| Chat typing indicator (bouncing dots) | Static ellipsis "..." |
| List item stagger | Instant render (all at once) |
| Toast slide-in | Instant appear/disappear |
| Card lift on press | Opacity change only |
| FAB scale | Opacity change only |
| Empty state illustration bounce | Static display |
| Header collapse | Instant collapse at threshold |

**Global implementation:**

```typescript
import { ReducedMotionConfig, ReduceMotion } from 'react-native-reanimated';

// At app root, above all animated components:
<ReducedMotionConfig mode={ReduceMotion.System} />
```

---

#### 6.5 Dynamic Type Scaling Rules

Skye supports iOS Dynamic Type from xSmall (~0.82x) through AX5 (~3.12x).

##### 6.5.1 Font Size Scaling Configuration

```typescript
const TextScaling = {
  displayLarge:  { maxFontSizeMultiplier: 1.3 },  // 34pt base, max 44pt
  heading1:      { maxFontSizeMultiplier: 1.4 },  // 28pt base, max 39pt
  heading2:      { maxFontSizeMultiplier: 1.4 },  // 24pt base, max 34pt
  heading3:      { maxFontSizeMultiplier: 1.5 },  // 22pt base, max 33pt
  body:          { maxFontSizeMultiplier: 2.0 },  // 17pt base, max 34pt
  bodySmall:     { maxFontSizeMultiplier: 2.0 },  // 15pt base, max 30pt
  caption:       { maxFontSizeMultiplier: 1.8 },  // 13pt base, max 23pt
  captionSmall:  { maxFontSizeMultiplier: 1.8 },  // 12pt base, max 22pt
  button:        { maxFontSizeMultiplier: 1.3 },  // 17pt base, max 22pt
  buttonSmall:   { maxFontSizeMultiplier: 1.3 },  // 15pt base, max 19pt
  tabLabel:      { maxFontSizeMultiplier: 1.2 },  // 10pt base, max 12pt
  numericLarge:  { maxFontSizeMultiplier: 1.5 },
  numericSmall:  { maxFontSizeMultiplier: 1.8 },
};

// Global defaults
Text.defaultProps = {
  ...Text.defaultProps,
  allowFontScaling: true,
  maxFontSizeMultiplier: 2.0,
};

TextInput.defaultProps = {
  ...TextInput.defaultProps,
  allowFontScaling: true,
  maxFontSizeMultiplier: 1.5,
};
```

##### 6.5.2 Layout Reflow Rules at Large Sizes

| Element | Default Layout | At Largest Accessibility Sizes |
|---------|---------------|--------------------------------------|
| Contact list row | Avatar + Name + Subtitle in single row | Avatar fixed, Name wraps to 2 lines, row height increases. Min height: 64pt. |
| Property stats row | Horizontal: "3 bd \| 2 ba \| 1,500 sqft" | Wrap to 2 lines or stack vertically |
| Action buttons (Call/Msg/Email) | 3 buttons horizontal with labels | At 1.5x: labels truncate; at 2x+: labels hidden (icon-only with accessibilityLabel) |
| Tab bar | Icon + label vertically stacked | Label may truncate; icons constant (25pt). Height does NOT increase. |
| Navigation bar title | Centered, single line | Truncates with ellipsis. Large title area accommodates 2 lines. |
| Chat bubbles | Max 75% screen width | Text reflows, bubble grows taller. Max width stays 75%. |
| Bottom sheet content | Standard layout | Content scrolls if exceeds sheet height. Sheet does NOT auto-expand. |
| Form fields | Label above input, fixed height | Label wraps. Input height increases proportionally. |
| Buttons | Fixed 44-50pt height | Height: min(textHeight + 24pt, 64pt). Prefer full-width at large sizes. |

**Critical rule:** No content may be clipped or hidden due to Dynamic Type scaling. If layout cannot accommodate text, the container scrolls.

##### 6.5.3 Icon Scaling

```typescript
import { PixelRatio } from 'react-native';

const fontScale = PixelRatio.getFontScale();
const iconScale = Math.min(fontScale, 1.5);
const scaledIconSize = (baseSize: number) => baseSize * iconScale;

// Usage: <Icon size={scaledIconSize(22)} />
```

---

#### 6.6 High Contrast Mode Support

When "Increase Contrast" is enabled:

| Element | Standard | High Contrast |
|---------|----------|---------------|
| Input borders | 1px, rgba(0,0,0,0.1) | 1.5px, rgba(0,0,0,0.4) |
| Separator lines | rgba(0,0,0,0.08) | rgba(0,0,0,0.25) |
| Placeholder text | rgba(0,0,0,0.3) | rgba(0,0,0,0.5) |
| Disabled buttons | 0.4 opacity | 0.5 opacity + "Disabled" label |
| Card shadows | Standard | Shadow + 1px border |
| Skeleton placeholders | rgba(0,0,0,0.06) | rgba(0,0,0,0.12) |
| Tab bar inactive icons | rgba(0,0,0,0.3) | rgba(0,0,0,0.5) |
| Active tab indicator | Brand color | Brand color + 2px underline |

Minimum contrast ratios (WCAG AA):
- Body text: 4.5:1
- Large text (>= 18pt or >= 14pt bold): 3:1
- Interactive elements: 3:1 against adjacent colors
- Focus indicators: 3:1

---

### 7. RESPONSIVE LAYOUT RULES

#### 7.1 iPhone Screen Size Breakpoints

| Device | Width (pt) | Height (pt) | Safe Top | Safe Bottom | Category |
|--------|-----------|------------|---------|------------|----------|
| iPhone SE 3rd Gen | 375 | 667 | 20 | 0 | `compact` |
| iPhone 13/12 Mini | 375 | 812 | 47 | 34 | `regular` |
| iPhone 14/15/16 | 390 | 844 | 47 | 34 | `regular` |
| iPhone 14/15/16 Plus | 428 | 926 | 47 | 34 | `large` |
| iPhone 15/16 Pro | 393 | 852 | 59 | 34 | `regular` |
| iPhone 15/16 Pro Max | 430 | 932 | 59 | 34 | `large` |

##### 7.1.1 Layout Categories

```typescript
type LayoutCategory = 'compact' | 'regular' | 'large';

const getLayoutCategory = (screenWidth: number): LayoutCategory => {
  if (screenWidth <= 375) return 'compact';
  if (screenWidth <= 400) return 'regular';
  return 'large';
};
```

##### 7.1.2 Layout Adjustments by Category

| Property | Compact (SE) | Regular (14/15/16) | Large (Plus/Max) |
|----------|-------------|-------------------|-----------------|
| Horizontal padding | 12pt | 16pt | 20pt |
| Card horizontal margin | 12pt | 16pt | 16pt |
| Grid columns (Studio) | 2 | 2 | 3 |
| Property card image height | 140pt | 160pt | 180pt |
| Chat bubble max width | 80% | 75% | 70% |
| Large title font size | 30pt | 34pt | 34pt |
| FAB position from right | 12pt | 16pt | 20pt |
| Bottom sheet border radius | 12pt | 16pt | 16pt |
| Search bar horizontal margin | 12pt | 16pt | 20pt |

##### 7.1.3 iPhone SE-Specific Accommodations

The iPhone SE 3rd Generation (375x667pt, no notch, no home indicator) requires special handling:

- **No safe area bottom inset:** Tab bar sits flush. No additional padding.
- **No Dynamic Island / notch:** Status bar is 20pt, not 47-59pt.
- **Reduced vertical space:** ~177pt less than standard iPhones.
  - Contact Detail: reduce header padding by 8pt
  - Property Detail: hero image reduced to 180pt (from 220pt)
  - Bottom sheets: use absolute pixel snap points when percentage would be <350pt
- **Home button:** No swipe-up gesture area. Bottom content extends to screen edge.

---

#### 7.2 Safe Area Handling

##### 7.2.1 Per-Screen-Type Safe Area Rules

| Screen Type | Top Safe Area | Bottom Safe Area |
|------------|--------------|-----------------|
| Standard tab screen | Navigation bar respects safe area (automatic) | Content above tab bar. Tab bar respects home indicator. |
| Full-screen modal | Close button inside safe area top. Content below status bar. | Content respects home indicator. No tab bar. |
| Bottom sheet | N/A (overlays existing screen) | Sheet content respects bottom safe area when expanded. |
| Keyboard-present | Top unchanged | Bottom = keyboard top edge. Safe area replaced by keyboard height. |
| Chat screen | Navigation bar standard | Input bar above keyboard OR above tab bar (when keyboard hidden). |
| Full-screen image viewer | Status bar hidden. Close button at safeAreaInsets.top + 8pt. | Toolbar at bottom with safeAreaInsets.bottom padding. |

##### 7.2.2 Implementation

```typescript
import { useSafeAreaInsets } from 'react-native-safe-area-context';

// EVERY screen uses this hook. No hardcoded safe area values.
const Screen = ({ children }) => {
  const insets = useSafeAreaInsets();
  return (
    <View style={{ flex: 1, paddingTop: insets.top }}>
      {children}
    </View>
  );
};
```

---

#### 7.3 Orientation Support

**Skye is locked to portrait orientation.**

**Justification:**
1. CRM workflows are optimized for one-handed, portrait use. Real estate agents use Skye while walking properties, driving between showings, and in meetings.
2. Chat interfaces are fundamentally portrait-optimized.
3. Landscape support doubles QA surface area without proportional user benefit.
4. Apple HIG accepts portrait-only when justified by use case.
5. Full-screen media viewers may support landscape in V3.

```xml
<!-- Info.plist -->
<key>UISupportedInterfaceOrientations</key>
<array>
  <string>UIInterfaceOrientationPortrait</string>
</array>
```

**V3 Exception:** Full-screen media viewers may unlock orientation temporarily using `Orientation.unlockAsync()`, locking back to portrait on dismissal.

---

#### 7.4 iPad Compatibility Mode

Skye V2 runs on iPad in iPhone compatibility mode (letterboxed). Native iPad layout is planned for V3.

**V2 iPad behavior:**
- Runs in iPhone simulator mode (centered on iPad screen)
- User can tap 2x button to scale
- No iPad-specific layouts, split views, or multitasking
- No pointer/trackpad interaction
- No keyboard shortcuts

```xml
<!-- Info.plist -->
<key>UIRequiresFullScreen</key>
<true/>
```

**V3 iPad plans (out of scope for V2):**
- SplitView: Master list on left, Detail on right
- Multitasking: Slide Over and Split View
- Pointer support: hover states
- Keyboard shortcuts: Cmd+N, Cmd+F, etc.
- Drag and drop: contacts to groups

---

### Appendix A: Library Dependency Map

| Concern | Library | Min Version | Purpose |
|---------|---------|-------------|---------|
| Animations | `react-native-reanimated` | 3.6+ | UI thread animations, springs, scroll-linked |
| Gestures | `react-native-gesture-handler` | 2.14+ | Pan, pinch, tap, long-press recognizers |
| Navigation | `@react-navigation/native` | 6.1+ | Tab, stack, modal navigation |
| Bottom sheets | `@gorhom/bottom-sheet` | 5.0+ | All bottom sheet presentations |
| Lists | `@shopify/flash-list` | 1.6+ | Lists with 50+ items |
| Keyboard | `react-native-keyboard-controller` | 1.10+ | Keyboard avoidance and animations |
| Haptics | `expo-haptics` | Latest | All haptic feedback |
| Safe areas | `react-native-safe-area-context` | 4.8+ | Safe area inset management |
| Fast images | `react-native-fast-image` | 8.6+ | Cached, optimized image loading |
| SVG | `react-native-svg` | 13+ | Icon rendering |
| Lottie | `lottie-react-native` | 6.4+ | Branded animations |
| Screens | `react-native-screens` | 3.29+ | Native screen containers, freezeOnBlur |

### Appendix B: Animation Performance Budget

| Metric | Target | Measurement |
|--------|--------|-------------|
| JS Thread FPS during scroll | >= 60 FPS | Flipper Performance |
| UI Thread FPS during scroll | >= 60 FPS | Flipper Performance |
| JS Thread FPS during animation | >= 58 FPS | Flipper |
| Time to Interactive (screen push) | < 300ms | Custom perf marks |
| Time to first meaningful paint | < 500ms | Custom perf marks |
| Skeleton to content transition | < 100ms perceived | Visual inspection |
| Haptic latency | < 10ms from gesture | UI thread execution |
| Memory: 500-item list | < 150MB total app | Xcode Instruments |
| Memory: 1000-item list | < 200MB total app | Xcode Instruments |
| Image cache size | 100MB disk, 50MB memory | react-native-fast-image config |

### Appendix C: Testing Checklist

Before any screen ships, it must pass ALL of the following:

- [ ] VoiceOver navigation order matches spec (Section 6.1) -- tested on physical device
- [ ] All touch targets >= 44x44pt -- verified with Accessibility Inspector
- [ ] All states (loading, loaded, empty, error, offline, refreshing) implemented and visually correct
- [ ] Skeleton layout matches loaded layout dimensions
- [ ] Dynamic Type: tested at Default, Large, and AX3 sizes -- no clipping, no overflow
- [ ] Reduce Motion: all animations gracefully degrade
- [ ] iPhone SE: layout fits, no cut-off content
- [ ] iPhone 16 Pro Max: no excessive whitespace, content fills appropriately
- [ ] Pull-to-refresh: branded animation runs at 60fps
- [ ] Haptic feedback: correct type fires for every interaction (physical device)
- [ ] Keyboard: all inputs accessible, avoidance works, dismiss on scroll
- [ ] Edge swipe back: works on all pushed screens
- [ ] Bottom sheet: drag-to-dismiss works, snap points correct, backdrop dismisses
- [ ] FlashList: scroll performance >= 60fps with 500+ items
- [ ] Offline mode: graceful degradation, cached data shown, error states correct
- [ ] Deep link: screen reachable from cold start, background, and foreground states
- [ ] Memory: no leaks after navigating to/from screen 10 times (Xcode Instruments)

---

*End of V2: UX Architecture & Interaction Design specification.*
*This document is the single source of truth for all interaction and layout behavior in Skye.*
*Any deviation requires written approval from the UX Architecture Lead with documented rationale.*

---

### Sources

- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [Apple HIG - Playing Haptics](https://developer.apple.com/design/human-interface-guidelines/playing-haptics)
- [Apple HIG - Sheets](https://developer.apple.com/design/human-interface-guidelines/sheets)
- [Apple HIG - Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [React Native Reanimated - withSpring](https://docs.swmansion.com/react-native-reanimated/docs/animations/withSpring/)
- [React Native Reanimated - Accessibility / useReducedMotion](https://docs.swmansion.com/react-native-reanimated/docs/guides/accessibility/)
- [React Native Reanimated - ReducedMotionConfig](https://docs.swmansion.com/react-native-reanimated/docs/device/ReducedMotionConfig/)
- [React Native Reanimated - Customizing Animations](https://docs.swmansion.com/react-native-reanimated/docs/fundamentals/customizing-animation/)
- [React Native Reanimated - Performance Guide](https://docs.swmansion.com/react-native-reanimated/docs/guides/performance/)
- [FlashList by Shopify](https://shopify.github.io/flash-list/)
- [Shopify Engineering - FlatList to FlashList](https://shopify.engineering/instant-performance-upgrade-flatlist-flashlist)
- [React Native Accessibility Docs](https://reactnative.dev/docs/accessibility)
- [React Native Accessibility Best Practices 2025](https://www.accessibilitychecker.org/blog/react-native-accessibility/)
- [React Native Screen Reader Support (Jan 2026)](https://oneuptime.com/blog/post/2026-01-15-react-native-screen-reader-support/view)
- [React Native Keyboard Controller - KeyboardAvoidingView](https://kirillzyusko.github.io/react-native-keyboard-controller/docs/api/components/keyboard-avoiding-view)
- [Expo - Keyboard Handling](https://docs.expo.dev/guides/keyboard-handling/)
- [Expo Haptics Documentation](https://docs.expo.dev/versions/latest/sdk/haptics/)
- [@gorhom/bottom-sheet Props](https://gorhom.dev/react-native-bottom-sheet/props)
- [React Navigation - Deep Linking](https://reactnavigation.org/docs/deep-linking/)
- [Modern iOS Navigation Patterns - Frank Rausch](https://frankrausch.com/ios-navigation/)
- [Apple WWDC22 - Explore Navigation Design for iOS](https://developer.apple.com/videos/play/wwdc2022/10001/)
- [React Native Dynamic Font Scaling (Jan 2026)](https://oneuptime.com/blog/post/2026-01-15-react-native-dynamic-font-scaling/view)
- [Pull to Refresh Beyond the Default Spinner (Jan 2026)](https://medium.com/@nomanakram1999/pull-to-refresh-in-react-native-beyond-the-default-spinner-01998230c9b2)
- [React Native Reanimated 3 Ultimate Guide](https://dev.to/erenelagz/react-native-reanimated-3-the-ultimate-guide-to-high-performance-animations-in-2025-4ae4)
- [Josh Comeau - Accessible Animations with prefers-reduced-motion](https://www.joshwcomeau.com/react/prefers-reduced-motion/)
- [Apple HIG Complete iOS Design 2026](https://www.nadcab.com/blog/apple-human-interface-guidelines-explained)

---


## Section 2: Visual Design System

> **Design North Star:** Skye must achieve Apple-level visual quality -- the polish of Things 3, the elegance of Linear, the sophistication of Arc Browser. Every pixel must be intentional. Every color must be justified. Every animation must feel inevitable. This is not a design guideline -- it is a visual constitution.

> **Compatibility Note:** This system is designed with awareness of Apple's Liquid Glass design language (introduced WWDC 2025 / iOS 26). Where relevant, guidance is provided for integrating with the new translucent material system. However, the core design tokens are engineered to work across iOS 17+ and will gracefully adopt Liquid Glass affordances when running on iOS 26+.

---

### 2.1 Color System

#### 2.1.1 Design Philosophy

Skye's color system follows a three-layer token architecture:

1. **Primitive tokens** -- Raw color values (hex/rgba). Never referenced directly in component code.
2. **Semantic tokens** -- Context-aware aliases that reference primitives (e.g., `color.text.primary`). Used in all component styles.
3. **Component tokens** -- Scoped overrides for specific components (e.g., `chatBubble.background.ai`). Used sparingly when semantic tokens are insufficient.

Color is used with restraint. The UI is predominantly neutral -- brand color appears only on interactive elements, AI-generated content indicators, and the primary call-to-action. This trains the user to recognize that color = actionable or meaningful.

#### 2.1.2 Brand Colors

| Token | Light Mode | Dark Mode | Usage |
|---|---|---|---|
| `brand.primary` | `#0066FF` | `#3D8AFF` | Primary actions, links, active tab, toggle on-state |
| `brand.primaryHover` | `#0052CC` | `#5C9FFF` | Pressed/hover state of primary elements |
| `brand.primaryMuted` | `rgba(0, 102, 255, 0.12)` | `rgba(61, 138, 255, 0.16)` | Primary tinted backgrounds (selected row, chip) |
| `brand.secondary` | `#6E42D2` | `#9B7AE8` | AI-specific accent, Studio features, generative content |
| `brand.secondaryMuted` | `rgba(110, 66, 210, 0.10)` | `rgba(155, 122, 232, 0.14)` | AI content backgrounds, Studio card tints |

**Contrast Ratios (brand.primary):**
- `#0066FF` on `#FFFFFF` (light canvas): **4.75:1** -- passes WCAG AA for normal text
- `#3D8AFF` on `#000000` (dark canvas): **7.22:1** -- passes WCAG AA and AAA
- `#3D8AFF` on `#1C1C1E` (dark surface): **5.98:1** -- passes WCAG AA

#### 2.1.3 Semantic Colors

| Token | Light Mode | Dark Mode | Usage |
|---|---|---|---|
| `semantic.success` | `#28A745` | `#34D058` | Positive actions, confirmations, deal closed |
| `semantic.warning` | `#E8910D` | `#FFB340` | Caution states, approaching deadlines |
| `semantic.error` | `#DC3545` | `#FF6B6B` | Destructive actions, validation errors, deal lost |
| `semantic.info` | `#0A84FF` | `#4DA3FF` | Informational banners, tips, system messages |

**Contrast Ratios (semantic colors on respective backgrounds):**
- `#28A745` on `#FFFFFF`: **4.52:1** -- passes AA for normal text (use bold/semibold for extra legibility)
- `#DC3545` on `#FFFFFF`: **4.63:1** -- passes AA
- `#34D058` on `#1C1C1E`: **6.84:1** -- passes AA and AAA
- `#FF6B6B` on `#1C1C1E`: **5.91:1** -- passes AA

#### 2.1.4 Background Hierarchy

Skye uses a four-tier background system to create visual depth without relying on shadows (especially critical in dark mode).

**Light Mode:**

| Token | Value | Usage |
|---|---|---|
| `bg.canvas` | `#F2F2F7` | Root app background (matches iOS `systemGroupedBackground`) |
| `bg.surface` | `#FFFFFF` | Cards, sheets, list items, input fields |
| `bg.elevated` | `#FFFFFF` | Modals, popovers, floating action menus |
| `bg.overlay` | `rgba(0, 0, 0, 0.40)` | Scrim behind modals and bottom sheets |
| `bg.sunken` | `#E8E8ED` | Inset containers, search bar backgrounds, code blocks |

**Dark Mode:**

| Token | Value | Usage |
|---|---|---|
| `bg.canvas` | `#000000` | Root app background (true black for OLED power savings) |
| `bg.surface` | `#1C1C1E` | Cards, sheets, list items, input fields |
| `bg.elevated` | `#2C2C2E` | Modals, popovers, floating action menus |
| `bg.overlay` | `rgba(0, 0, 0, 0.60)` | Scrim behind modals and bottom sheets |
| `bg.sunken` | `#0D0D0F` | Inset containers, search bar backgrounds, code blocks |

**Elevated Dark Mode** (for modals presented as sheets on iOS):

| Token | Value | Usage |
|---|---|---|
| `bg.canvas.elevated` | `#1C1C1E` | Base of a presented modal sheet |
| `bg.surface.elevated` | `#2C2C2E` | Cards inside a modal sheet |
| `bg.elevated.elevated` | `#3A3A3C` | Nested elevated surfaces inside modals |

#### 2.1.5 Text Color Hierarchy

**Light Mode:**

| Token | Value | Usage |
|---|---|---|
| `text.primary` | `#000000` (alpha 1.0) | Headlines, body text, primary labels |
| `text.secondary` | `rgba(60, 60, 67, 0.60)` | Subheadlines, secondary labels, metadata |
| `text.tertiary` | `rgba(60, 60, 67, 0.30)` | Placeholder text, disabled labels, tertiary info |
| `text.quaternary` | `rgba(60, 60, 67, 0.18)` | Watermarks, ultra-subtle decorative text |
| `text.inverse` | `#FFFFFF` | Text on dark/colored backgrounds (buttons, badges) |
| `text.brand` | `#0066FF` | Links, tappable text, active navigation labels |
| `text.onPrimary` | `#FFFFFF` | Text rendered on `brand.primary` backgrounds |

**Dark Mode:**

| Token | Value | Usage |
|---|---|---|
| `text.primary` | `#FFFFFF` (alpha 1.0) | Headlines, body text, primary labels |
| `text.secondary` | `rgba(235, 235, 245, 0.60)` | Subheadlines, secondary labels, metadata |
| `text.tertiary` | `rgba(235, 235, 245, 0.30)` | Placeholder text, disabled labels, tertiary info |
| `text.quaternary` | `rgba(235, 235, 245, 0.16)` | Watermarks, ultra-subtle decorative text |
| `text.inverse` | `#000000` | Text on light backgrounds in dark mode |
| `text.brand` | `#3D8AFF` | Links, tappable text, active navigation labels |
| `text.onPrimary` | `#FFFFFF` | Text rendered on `brand.primary` backgrounds |

**Contrast Ratios (text on backgrounds):**
- `text.primary` (#000000) on `bg.surface` (#FFFFFF): **21:1** -- maximum contrast
- `text.secondary` (rgba(60,60,67,0.60) composited) on `bg.surface` (#FFFFFF): **5.54:1** -- passes AA
- `text.primary` (#FFFFFF) on `bg.surface` (#1C1C1E): **16.15:1** -- passes AAA
- `text.secondary` (rgba(235,235,245,0.60) composited) on `bg.surface` (#1C1C1E): **7.28:1** -- passes AA

#### 2.1.6 Real Estate Engagement Health Colors

These colors represent contact engagement temperature and appear as 8pt indicator dots, ring fills, and chart segments throughout the CRM.

| Token | Light Mode | Dark Mode | Meaning |
|---|---|---|---|
| `engagement.hot` | `#E53935` | `#FF5252` | Active buyer/seller, responded within 24h, high intent |
| `engagement.warm` | `#F4A623` | `#FFB74D` | Engaged in last 7 days, moderate intent |
| `engagement.cool` | `#2196F3` | `#64B5F6` | Last contact 7-30 days ago, passive interest |
| `engagement.cold` | `#78909C` | `#90A4AE` | No engagement 30-90 days, requires re-engagement |
| `engagement.inactive` | `#BDBDBD` | `#616161` | No engagement 90+ days, possibly churned |

**Engagement dot specifications:**
- Size: 8pt diameter circle
- Border: none
- Position: right-aligned in contact list row, vertically centered with contact name
- Animation: subtle pulse (scale 1.0 to 1.15 to 1.0 over 2000ms) on `engagement.hot` only
- Accessibility: always paired with text label (e.g., "Hot lead") for VoiceOver

#### 2.1.7 Fill & Border Colors

| Token | Light Mode | Dark Mode | Usage |
|---|---|---|---|
| `fill.primary` | `rgba(120, 120, 128, 0.20)` | `rgba(120, 120, 128, 0.36)` | Switch tracks, slider tracks, segmented control bg |
| `fill.secondary` | `rgba(120, 120, 128, 0.16)` | `rgba(120, 120, 128, 0.32)` | Text field backgrounds, search bars |
| `fill.tertiary` | `rgba(120, 120, 128, 0.12)` | `rgba(120, 120, 128, 0.24)` | Button subtle backgrounds, tag backgrounds |
| `fill.quaternary` | `rgba(120, 120, 128, 0.08)` | `rgba(120, 120, 128, 0.18)` | Hover states, skeleton screen base |
| `border.default` | `rgba(60, 60, 67, 0.12)` | `rgba(84, 84, 88, 0.65)` | Card borders, dividers, input field borders |
| `border.focused` | `#0066FF` | `#3D8AFF` | Focused input field border |
| `border.error` | `#DC3545` | `#FF6B6B` | Input validation error border |
| `separator.default` | `rgba(60, 60, 67, 0.29)` | `rgba(84, 84, 88, 0.60)` | List separators, section dividers |
| `separator.opaque` | `#C6C6C8` | `#38383A` | Opaque dividers where transparency causes issues |

#### 2.1.8 Gradient Specifications

Gradients are used sparingly for premium UI moments only -- never for standard UI chrome.

| Token | Values | Usage |
|---|---|---|
| `gradient.aiShimmer` | Linear, 135deg, `#6E42D2` -> `#3D8AFF` -> `#6E42D2` | AI thinking indicator border, Studio generation progress |
| `gradient.premium` | Linear, 180deg, `#0066FF` -> `#6E42D2` | Onboarding hero, upgrade CTA, premium feature badges |
| `gradient.warmGlow` | Radial, center, `rgba(244, 166, 35, 0.15)` -> `rgba(244, 166, 35, 0.00)` | Subtle warm glow behind "hot lead" spotlight cards |
| `gradient.surfaceFade` | Linear, 180deg, `bg.surface` at 0% opacity -> `bg.surface` at 100% opacity | Scroll fade-out at bottom of truncated content |
| `gradient.skeleton` | Linear, 90deg, `fill.quaternary` -> `fill.tertiary` -> `fill.quaternary` | Skeleton loading shimmer sweep |

**AI Shimmer animation specification:**
- Gradient position animates from -100% to 200% over 2000ms
- Easing: linear (continuous motion)
- Repeats infinitely while AI is processing
- Reduce Motion alternative: static border with `brand.secondary` color, pulsing opacity (0.5 to 1.0)

#### 2.1.9 System Color Integration Rules

| Scenario | Decision | Justification |
|---|---|---|
| Destructive action buttons | Use Skye `semantic.error` | Brand consistency across platforms |
| Toggle switches (on-state) | Use Skye `brand.primary` | Reinforces brand color = interactive |
| System alert dialogs | Use iOS `systemBlue` / native defaults | System-presented UI should match OS |
| Status bar tint | Use iOS system automatic | Proper light/dark switching |
| Activity indicators (system) | Use iOS `systemGray` | Native loading spinners match OS |
| Pull-to-refresh tint | Use Skye `brand.primary` | Custom refresh control, brand moment |
| Selection highlight (text) | Use iOS `tintColor` mapped to `brand.primary` | System text selection uses tint |
| Keyboard appearance | `.light` / `.dark` per mode | Matches app appearance mode |

---

### 2.2 Typography System

#### 2.2.1 Font Family Selection

**Primary typeface:** SF Pro (San Francisco) -- Apple's system font.

**Justification:** SF Pro is the definitive choice for Skye because:
1. **Native performance** -- zero bundle size cost, rendered by the OS text engine with optimal hinting.
2. **Dynamic Type support** -- automatic scaling across all accessibility text sizes without custom font metric tables.
3. **Optical sizing** -- SF Pro automatically adjusts letterspacing and stroke weight at different sizes (Text variant at 19pt and below, Display variant at 20pt and above).
4. **Variable width support** -- Condensed, Regular, Expanded variants available for data-dense views.
5. **9 weights** -- Ultralight through Black, with precise weight-matching to SF Symbols.
6. **Professional authority** -- reinforces that Skye is a serious business tool, not a lifestyle app.

**Secondary typeface:** SF Mono -- Apple's monospaced system font.
- Usage: Code blocks in AI chat responses, structured data display, JSON previews.

**Tertiary typeface:** SF Pro Rounded -- Apple's rounded system font variant.
- Usage: Reserved exclusively for the Skye logo wordmark and empty state illustration captions where a friendlier tone is appropriate. Never used for body text or navigation.

#### 2.2.2 Type Scale

All sizes specified at the iOS default Dynamic Type setting ("Large" -- the 4th of 12 stops on the accessibility slider).

| Token | Style Name | Size (pt) | Weight | Line Height (pt) | Letter Spacing (pt) | Usage |
|---|---|---|---|---|---|---|
| `type.largeTitle` | Large Title | 34 | Bold (700) | 41 | 0.37 | Screen titles when scrolled to top (People, Properties) |
| `type.title1` | Title 1 | 28 | Bold (700) | 34 | 0.36 | Section headers, modal titles |
| `type.title2` | Title 2 | 22 | Bold (700) | 28 | 0.35 | Card titles, contact names in detail view |
| `type.title3` | Title 3 | 20 | Semibold (600) | 25 | 0.38 | Sub-section headers, property price display |
| `type.headline` | Headline | 17 | Semibold (600) | 22 | -0.41 | List item primary text, emphasized body text |
| `type.body` | Body | 17 | Regular (400) | 22 | -0.41 | Default reading text, chat messages, form labels |
| `type.bodyEmphasized` | Body Emphasized | 17 | Semibold (600) | 22 | -0.41 | Bold body text for emphasis within paragraphs |
| `type.callout` | Callout | 16 | Regular (400) | 21 | -0.32 | Secondary content, card descriptions |
| `type.subheadline` | Subheadline | 15 | Regular (400) | 20 | -0.24 | Metadata, timestamps, secondary labels |
| `type.subheadlineEmphasized` | Subheadline Emphasized | 15 | Semibold (600) | 20 | -0.24 | Emphasized metadata, active filter labels |
| `type.footnote` | Footnote | 13 | Regular (400) | 18 | -0.08 | Auxiliary text, legal text, attribution |
| `type.footnoteEmphasized` | Footnote Emphasized | 13 | Semibold (600) | 18 | -0.08 | Footnote with emphasis (e.g., "NEW" badge text) |
| `type.caption1` | Caption 1 | 12 | Regular (400) | 16 | 0.00 | Timestamps, engagement labels, chart axis labels |
| `type.caption1Emphasized` | Caption 1 Emphasized | 12 | Medium (500) | 16 | 0.00 | Active caption, badge counts |
| `type.caption2` | Caption 2 | 11 | Regular (400) | 13 | 0.07 | Smallest text -- fine print, tab bar labels |

#### 2.2.3 Dynamic Type Mapping

Every text style in Skye maps to an iOS Dynamic Type text style, ensuring that the system accessibility slider controls all text sizes.

| Skye Token | iOS TextStyle | xSmall | Small | Medium | Large (Default) | xLarge | xxLarge | xxxLarge | AX1 | AX2 | AX3 | AX4 | AX5 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `type.largeTitle` | `.largeTitle` | 31 | 32 | 33 | 34 | 36 | 38 | 40 | 44 | 48 | 52 | 56 | 60 |
| `type.title1` | `.title1` | 25 | 26 | 27 | 28 | 30 | 32 | 34 | 38 | 43 | 48 | 53 | 58 |
| `type.title2` | `.title2` | 19 | 20 | 21 | 22 | 24 | 26 | 28 | 34 | 39 | 44 | 50 | 56 |
| `type.title3` | `.title3` | 17 | 18 | 19 | 20 | 22 | 24 | 26 | 31 | 37 | 43 | 49 | 55 |
| `type.headline` | `.headline` | 14 | 15 | 16 | 17 | 19 | 21 | 23 | 28 | 33 | 38 | 44 | 50 |
| `type.body` | `.body` | 14 | 15 | 16 | 17 | 19 | 21 | 23 | 28 | 33 | 38 | 44 | 50 |
| `type.callout` | `.callout` | 13 | 14 | 15 | 16 | 18 | 20 | 22 | 26 | 32 | 38 | 44 | 50 |
| `type.subheadline` | `.subheadline` | 12 | 13 | 14 | 15 | 17 | 19 | 21 | 25 | 30 | 36 | 42 | 48 |
| `type.footnote` | `.footnote` | 12 | 12 | 12 | 13 | 15 | 17 | 19 | 23 | 27 | 33 | 38 | 44 |
| `type.caption1` | `.caption1` | 11 | 11 | 11 | 12 | 14 | 16 | 18 | 22 | 26 | 32 | 37 | 43 |
| `type.caption2` | `.caption2` | 11 | 11 | 11 | 11 | 13 | 15 | 17 | 20 | 24 | 29 | 34 | 40 |

**Implementation note:** Use React Native's `Text` component with `allowFontScaling={true}` (default) and specify styles using the iOS text style constants. For React Native, use `fontFamily: 'System'` which automatically resolves to SF Pro with proper optical sizing.

#### 2.2.4 Font Weight Usage Rules

| Weight | Token | When to Use | When NOT to Use |
|---|---|---|---|
| Regular (400) | `weight.regular` | Body text, secondary labels, input values, chat messages | Never for navigation labels or section headers |
| Medium (500) | `weight.medium` | Emphasized captions, tab bar labels, badge counts, data table headers | Never for body text -- too similar to Regular at body size |
| Semibold (600) | `weight.semibold` | Headlines, emphasized body, navigation bar titles, list primary labels, button text | Never for large titles (use Bold), never for fine print |
| Bold (700) | `weight.bold` | Large titles, screen titles, modal headers, marketing/onboarding text | Never for body text -- creates visual shouting |

**Weights to AVOID:** Ultralight, Thin, Light -- these are difficult to read at small sizes and fail contrast requirements in many scenarios. Heavy and Black -- too aggressive for a professional CRM tool.

#### 2.2.5 Monospace Specifications

| Token | Font | Size | Line Height | Usage |
|---|---|---|---|---|
| `type.code` | SF Mono Regular | 15 | 22 | Inline code in AI chat, structured data |
| `type.codeBlock` | SF Mono Regular | 14 | 20 | Multi-line code blocks in AI chat responses |
| `type.codeBlockEmphasized` | SF Mono Semibold | 14 | 20 | Syntax highlighting keywords |

**Code block container:** `bg.sunken` background, `border.default` border, `borderRadius.md` (10pt), padding `spacing.md` (12pt) on all sides.

#### 2.2.6 Number Formatting

| Context | Figure Style | Implementation |
|---|---|---|
| Property prices, financial data | Tabular (monospaced) figures | `fontVariant: ['tabular-nums']` |
| Phone numbers, addresses | Tabular figures | `fontVariant: ['tabular-nums']` |
| Body text with inline numbers | Proportional (default) figures | No special fontVariant needed |
| Data tables, statistics | Tabular figures | `fontVariant: ['tabular-nums']` |
| Badge counts | Tabular figures | `fontVariant: ['tabular-nums']` |

---

### 2.3 Spacing & Layout System

#### 2.3.1 Base Grid

Skye uses a **4pt base grid**. Every spacing value, padding, margin, and size dimension is a multiple of 4. The most commonly used values are multiples of 8 (the "8pt grid"), with 4pt values reserved for fine adjustments.

#### 2.3.2 Named Spacing Tokens

| Token | Value (pt) | Usage |
|---|---|---|
| `spacing.xxs` | 2 | Hairline gaps, icon-to-badge offset, optical adjustments |
| `spacing.xs` | 4 | Tight gaps: icon-to-label within a button, stacked avatar overlap offset |
| `spacing.sm` | 8 | Compact spacing: between related elements, inline tag gaps, list item internal padding |
| `spacing.md` | 12 | Standard spacing: input field padding, card internal element gaps |
| `spacing.lg` | 16 | Default padding: screen edge margins, card padding, section title to content |
| `spacing.xl` | 24 | Generous spacing: between cards in a list, between sections |
| `spacing.xxl` | 32 | Major spacing: screen header to content, between major screen sections |
| `spacing.xxxl` | 48 | Dramatic spacing: empty state illustration to text, onboarding hero spacing |
| `spacing.xxxxl` | 64 | Maximum spacing: splash screen element separation |

#### 2.3.3 Component Spacing Rules

**Screen Layout:**

| Rule | Value | Notes |
|---|---|---|
| Screen edge horizontal padding | `spacing.lg` (16pt) | All screens. Consistent left/right inset. |
| Safe area top inset | System-provided | Never hardcode; use `SafeAreaView` |
| Safe area bottom inset | System-provided | Tab bar and home indicator handled by system |
| Max content width (iPad/large) | 640pt | Centered with auto horizontal margins on screens wider than 640pt |
| Navigation bar height | 44pt (standard) / 96pt (large title) | iOS system standard |

**Card Components:**

| Rule | Value |
|---|---|
| Card padding (all sides) | `spacing.lg` (16pt) |
| Card internal element gap | `spacing.md` (12pt) |
| Card-to-card vertical gap | `spacing.sm` (8pt) |
| Card border radius | `borderRadius.lg` (14pt) |

**List Items:**

| Rule | Value |
|---|---|
| List item vertical padding | `spacing.md` (12pt) |
| List item horizontal padding | `spacing.lg` (16pt) |
| List item minimum height | 44pt (Apple touch target minimum) |
| List item avatar to text gap | `spacing.md` (12pt) |
| List item separator inset (left) | 68pt (avatar width + left padding + gap) |

**Forms and Inputs:**

| Rule | Value |
|---|---|
| Input field height | 44pt |
| Input field horizontal padding | `spacing.md` (12pt) |
| Input label to field gap | `spacing.sm` (8pt) |
| Between form fields | `spacing.lg` (16pt) |
| Form section gap | `spacing.xxl` (32pt) |

**Section Spacing:**

| Rule | Value |
|---|---|
| Section title to content gap | `spacing.sm` (8pt) |
| Between sections (same screen) | `spacing.xxl` (32pt) |
| Screen title to first content | `spacing.lg` (16pt) |

#### 2.3.4 Vertical Rhythm

All text blocks maintain consistent vertical rhythm using the base grid:

- **Single line spacing:** Defined by the line height of the text style (see type scale).
- **Paragraph spacing:** Equal to 0.5x the line height of the body style (11pt for body text).
- **Heading to body gap:** `spacing.sm` (8pt) -- tight coupling shows relationship.
- **Body to next heading gap:** `spacing.xl` (24pt) -- clear separation between sections.

---

### 2.4 Iconography

#### 2.4.1 Icon Library Selection

**Primary:** SF Symbols 7 (6,900+ symbols).

**Justification:**
1. **Weight matching** -- SF Symbols automatically match the weight of adjacent SF Pro text, creating visual harmony that no third-party icon library can replicate.
2. **Dynamic Type scaling** -- icons scale with text size when using text style configurations.
3. **Rendering modes** -- Monochrome, Hierarchical, Palette, and Multicolor provide rich expression without custom artwork.
4. **Animation presets** -- Built-in Bounce, Pulse, Variable Color, and the new Draw animations (SF Symbols 7) reduce custom animation code.
5. **Accessibility** -- SF Symbols include built-in accessibility labels, and localized variants for RTL languages.
6. **Native feel** -- users subconsciously recognize SF Symbols as "belonging" to iOS.

**Supplementary:** Custom SVG icons are used only when SF Symbols lacks a needed glyph (see Section 2.4.5). These must be designed to match SF Symbols' optical weight and metrics.

#### 2.4.2 Icon Size Variants

| Token | Point Size | Scale | Usage |
|---|---|---|---|
| `icon.xs` | 12pt | Small | Inline indicators, disclosure arrows within caption text |
| `icon.sm` | 16pt | Small | Inline icons within body text, compact list accessories |
| `icon.md` | 20pt | Medium | Standard UI icons, toolbar items, form field leading icons |
| `icon.lg` | 24pt | Large | Tab bar icons, navigation bar buttons, card action icons |
| `icon.xl` | 28pt | Large | Prominent action icons, empty state secondary icons |
| `icon.xxl` | 36pt | Large | Feature icons, settings section headers, onboarding steps |
| `icon.hero` | 56pt | Large | Empty state hero icons, onboarding illustration accents |

**Minimum touch target:** Regardless of icon size, all tappable icons must have a minimum touch target of 44x44pt. Use transparent hit-slop expansion when needed.

#### 2.4.3 Icon Weight Matching Rules

| Adjacent Text Weight | Icon Weight | Example |
|---|---|---|
| Regular (400) | Regular | Body text with inline info icon |
| Medium (500) | Medium | Tab bar labels with tab bar icons |
| Semibold (600) | Semibold | Navigation title with nav bar buttons |
| Bold (700) | Bold | Section header with section action icon |

**Implementation:** In React Native, use `react-native-sf-symbols` or render SF Symbols through a native module bridge. Configure the symbol weight to match adjacent text. When using `react-native-vector-icons` as a fallback, map icon stroke widths to approximate SF Symbol weight matching.

#### 2.4.4 Tab Bar Icon Specifications

| State | Variant | Color | Weight | Size |
|---|---|---|---|---|
| Active | Filled | `brand.primary` (#0066FF light / #3D8AFF dark) | Medium | 24pt (large scale) |
| Inactive | Outline | `text.tertiary` (rgba(60,60,67,0.30) light / rgba(235,235,245,0.30) dark) | Regular | 24pt (large scale) |

**Tab bar configuration:**

| Property | Value |
|---|---|
| Tab bar height | 49pt (standard) + safe area bottom inset |
| Tab bar background | `bg.surface` with thin material blur (iOS `systemChromeMaterial` equivalent) |
| Tab bar border top | `separator.default` at 0.33pt height (retina hairline) |
| Tab label size | `type.caption2` (11pt, Medium weight) |
| Tab label-to-icon gap | `spacing.xxs` (2pt) |
| Active tab label color | `brand.primary` |
| Inactive tab label color | `text.tertiary` |

**Tab icons (SF Symbols glyph names):**

| Tab | Active (Filled) | Inactive (Outline) |
|---|---|---|
| Chat | `bubble.left.and.bubble.right.fill` | `bubble.left.and.bubble.right` |
| People | `person.2.fill` | `person.2` |
| Properties | `building.2.fill` | `building.2` |
| Studio | `wand.and.stars` | `wand.and.stars` (hierarchical rendering for active) |
| Settings | `gearshape.fill` | `gearshape` |

#### 2.4.5 Custom Icons Needed

These icons do not exist in SF Symbols and must be custom-designed to match SF Symbols' visual language.

| Icon | Description | Size(s) | Usage |
|---|---|---|---|
| `skye-ai-sparkle` | Skye AI indicator -- a stylized 4-point star with subtle gradient fill (brand.secondary) | 16pt, 20pt, 24pt | AI-generated content indicator, chat AI avatar accent |
| `engagement-dot` | Filled circle for engagement health display | 8pt, 10pt, 12pt | Contact list engagement indicator |
| `property-marker` | Simplified house pin -- combines `mappin` with small house silhouette | 24pt, 32pt | Map view property pins |
| `deal-pipeline` | Horizontal funnel with 4 segments | 20pt, 28pt | Pipeline stage indicator |
| `skye-logo-mark` | Skye app logo as a compact symbol (for use in chat as AI avatar) | 28pt, 36pt | AI chat avatar, loading states |

**Custom icon design constraints:**
- Must be drawn on SF Symbols grid templates (exported from SF Symbols app)
- Stroke weight must correspond to SF Pro Regular weight at the intended display size
- Must include Regular, Medium, Semibold, Bold weight variants at minimum
- Must include both outline and filled variants
- Export as PDF vector (for native iOS) and SVG (for React Native)

---

### 2.5 Component Design Tokens

#### 2.5.1 Border Radius Scale

| Token | Value (pt) | Usage |
|---|---|---|
| `borderRadius.none` | 0 | Full-bleed elements, inline text highlights |
| `borderRadius.xs` | 4 | Inline code snippets, small badges, tag pills |
| `borderRadius.sm` | 6 | Buttons (small), chips, input field inner elements |
| `borderRadius.md` | 10 | Input fields, search bars, segmented controls, toast notifications |
| `borderRadius.lg` | 14 | Cards, bottom sheets (top corners only), action sheets |
| `borderRadius.xl` | 20 | Modal sheets (top corners only), floating action menus, large image containers |
| `borderRadius.xxl` | 24 | Onboarding cards, premium feature showcase cards |
| `borderRadius.full` | 9999 | Avatars, FABs, circular buttons, status dots, pill badges |

**Continuous corner radius:** All rounded elements in Skye use iOS-style continuous (superellipse) corner curves, not standard circular arcs. In React Native, set `borderCurve: 'continuous'` (available in RN 0.71+). This creates the smooth "squircle" effect that distinguishes Apple-quality UI from generic rounded rectangles.

#### 2.5.2 Shadow / Elevation Scale

**Light Mode Shadows:**

| Token | Values | Usage |
|---|---|---|
| `shadow.none` | No shadow | Flat elements, inline content |
| `shadow.xs` | `shadowColor: #000, shadowOffset: {0, 1}, shadowOpacity: 0.04, shadowRadius: 3` | Subtle lift: list item hover state, input field focus |
| `shadow.sm` | `shadowColor: #000, shadowOffset: {0, 2}, shadowOpacity: 0.06, shadowRadius: 6` | Card elevation: standard cards, dropdown menus |
| `shadow.md` | `shadowColor: #000, shadowOffset: {0, 4}, shadowOpacity: 0.08, shadowRadius: 12` | Modal elevation: bottom sheets, popovers |
| `shadow.lg` | `shadowColor: #000, shadowOffset: {0, 8}, shadowOpacity: 0.12, shadowRadius: 24` | Overlay elevation: floating action buttons, tooltips |
| `shadow.xl` | `shadowColor: #000, shadowOffset: {0, 16}, shadowOpacity: 0.16, shadowRadius: 48` | Dramatic elevation: dragged elements, top-level modals |

**Dark Mode Shadow Strategy:**

Shadows are functionally invisible in dark mode (black shadow on near-black background). Skye uses alternative elevation cues in dark mode:

| Light Mode Equivalent | Dark Mode Treatment |
|---|---|
| `shadow.none` | No treatment |
| `shadow.xs` | 1pt top border using `rgba(255, 255, 255, 0.04)` (luminance edge) |
| `shadow.sm` | Background shifts to next tier (`bg.surface` -> `bg.elevated`) |
| `shadow.md` | Background shifts + 1pt top border `rgba(255, 255, 255, 0.06)` |
| `shadow.lg` | Background shifts two tiers + 1pt border `rgba(255, 255, 255, 0.08)` |
| `shadow.xl` | Background `bg.elevated` + 1pt border `rgba(255, 255, 255, 0.10)` + subtle inner glow |

**`boxShadow` specification (React Native 0.76+):**

For apps targeting RN 0.76+, use the `boxShadow` style property for cross-platform consistency:

```
shadow.sm = { boxShadow: '0px 2px 6px rgba(0, 0, 0, 0.06)' }
shadow.md = { boxShadow: '0px 4px 12px rgba(0, 0, 0, 0.08)' }
shadow.lg = { boxShadow: '0px 8px 24px rgba(0, 0, 0, 0.12)' }
```

#### 2.5.3 Border Specifications

| Token | Width | Color (Light) | Color (Dark) | Style | Usage |
|---|---|---|---|---|---|
| `border.hairline` | 0.33pt (1px on 3x retina) | `separator.default` | `separator.default` | solid | List separators, horizontal rules |
| `border.thin` | 1pt | `border.default` | `border.default` | solid | Card borders, input field borders, dividers |
| `border.medium` | 1.5pt | `border.focused` | `border.focused` | solid | Focused input fields, active selection borders |
| `border.thick` | 2pt | `brand.primary` | `brand.primary` | solid | Selected card borders, active segment indicator |
| `border.dashed` | 1pt | `border.default` | `border.default` | dashed (6pt dash, 4pt gap) | Drop zones, optional content placeholders |

#### 2.5.4 Opacity Scale

| Token | Value | Usage |
|---|---|---|
| `opacity.disabled` | 0.38 | Disabled buttons, disabled text, disabled icons |
| `opacity.muted` | 0.60 | Secondary text (matches `text.secondary` alpha), deemphasized elements |
| `opacity.overlay.light` | 0.40 | Light mode scrim behind modals |
| `opacity.overlay.dark` | 0.60 | Dark mode scrim behind modals |
| `opacity.hover` | 0.08 | Subtle background tint on hover/press state |
| `opacity.pressed` | 0.12 | Background tint on press state |
| `opacity.imageOverlay` | 0.15 | Dark mode image dimming overlay |
| `opacity.skeleton` | 0.60 | Skeleton placeholder element opacity |

#### 2.5.5 Blur / Vibrancy Specifications

| Token | Blur Radius | Background | Usage |
|---|---|---|---|
| `blur.thinMaterial` | 10pt | `rgba(255, 255, 255, 0.60)` light / `rgba(30, 30, 30, 0.60)` dark | Navigation bar, tab bar backgrounds |
| `blur.regularMaterial` | 20pt | `rgba(255, 255, 255, 0.72)` light / `rgba(30, 30, 30, 0.72)` dark | Bottom sheet backgrounds, overlay surfaces |
| `blur.thickMaterial` | 30pt | `rgba(255, 255, 255, 0.85)` light / `rgba(30, 30, 30, 0.85)` dark | Modal backgrounds, popover backgrounds |
| `blur.ultraThinMaterial` | 5pt | `rgba(255, 255, 255, 0.40)` light / `rgba(30, 30, 30, 0.40)` dark | Floating labels over images, map overlays |
| `blur.chromeMaterial` | 15pt | `rgba(242, 242, 247, 0.80)` light / `rgba(28, 28, 30, 0.80)` dark | System chrome: toolbars, segmented controls on blur bg |

**Implementation:** Use `@react-native-community/blur` (`BlurView`) with `blurType` set to `'light'` or `'dark'`, and `blurAmount` mapped to the blur radius tokens above. On iOS, this maps directly to `UIVisualEffectView` with the appropriate `UIBlurEffect.Style`.

**iOS 26 Liquid Glass integration:** When running on iOS 26+, replace `blur.thinMaterial` and `blur.regularMaterial` with the system `liquidGlassMaterial` modifier on navigation bars, tab bars, and toolbars. The Skye design system should detect the OS version and conditionally apply Liquid Glass treatments. Content areas and cards retain the custom blur specifications above regardless of OS version.

---

### 2.6 Animation & Motion Design

#### 2.6.1 Design Philosophy

Skye animations follow three principles:
1. **Purposeful** -- Every animation communicates something: a state change, spatial relationship, or confirmation of user action. No decorative animation.
2. **Physics-based** -- Prefer spring animations over timing-based animations. Springs feel natural because they model real-world physics (mass, tension, friction).
3. **Interruptible** -- All animations can be interrupted by new user input without visual glitching.

#### 2.6.2 Duration Scale

Used only for timing-based animations (not springs). Springs define their own duration through physics.

| Token | Duration | Usage |
|---|---|---|
| `duration.instant` | 100ms | Opacity changes, color transitions, icon state toggles |
| `duration.fast` | 200ms | Button press feedback, chip selection, toggle switches |
| `duration.normal` | 300ms | Screen transitions, card expand/collapse, modal present |
| `duration.slow` | 400ms | Complex layout transitions, shared element transitions |
| `duration.dramatic` | 500ms | Onboarding animations, celebration moments, first-run reveals |
| `duration.skeleton` | 1500ms | One full cycle of skeleton shimmer sweep |
| `duration.aiShimmer` | 2000ms | One full cycle of AI processing shimmer |

#### 2.6.3 Easing Curves

| Token | Type | Values | Usage |
|---|---|---|---|
| `easing.standard` | Cubic bezier | `cubic-bezier(0.25, 0.1, 0.25, 1.0)` (ease) | Default for non-spring animations |
| `easing.enter` | Cubic bezier | `cubic-bezier(0.0, 0.0, 0.2, 1.0)` (ease-out / decelerate) | Elements entering the screen |
| `easing.exit` | Cubic bezier | `cubic-bezier(0.4, 0.0, 1.0, 1.0)` (ease-in / accelerate) | Elements leaving the screen |
| `easing.emphasized` | Cubic bezier | `cubic-bezier(0.2, 0.0, 0.0, 1.0)` | Important transitions, modal presentations |
| `easing.linear` | Linear | `cubic-bezier(0.0, 0.0, 1.0, 1.0)` | Progress animations, shimmer sweeps |

#### 2.6.4 Spring Configurations

| Token | Mass | Damping | Stiffness | Usage |
|---|---|---|---|---|
| `spring.snappy` | 1 | 20 | 300 | Button scale feedback, toggle switches, quick UI updates |
| `spring.responsive` | 1 | 18 | 200 | List item interactions, card press states, chip selections |
| `spring.smooth` | 1 | 22 | 150 | Screen transitions, shared element transitions |
| `spring.gentle` | 1 | 26 | 120 | Modal presentations, bottom sheet open/close |
| `spring.bouncy` | 1 | 12 | 180 | Success animations, celebration moments, first-time reveals |
| `spring.stiff` | 1 | 30 | 400 | Gesture-driven snapping, tab selection, segmented control |

**Reanimated `withSpring` usage:**
```javascript
withSpring(targetValue, {
  mass: spring.smooth.mass,       // 1
  damping: spring.smooth.damping, // 22
  stiffness: spring.smooth.stiffness, // 150
  overshootClamping: false,
  restDisplacementThreshold: 0.01,
  restSpeedThreshold: 2,
})
```

#### 2.6.5 Screen Transition Animations

| Transition | Trigger | Animation | Duration/Spring |
|---|---|---|---|
| **Push** | Navigate to detail screen | Incoming screen slides from right (100% to 0%), outgoing slides left (0% to -33%). Both use `spring.smooth`. | `spring.smooth` (mass:1, damping:22, stiffness:150) |
| **Pop** | Navigate back (swipe or button) | Reverse of push. Gesture-driven: progress-based spring tracks finger position. | `spring.smooth` or gesture-driven |
| **Modal present** | Open modal/bottom sheet | Incoming: slide up from bottom with `spring.gentle`. Outgoing: scale down to 0.94, borderRadius increases to 12pt, opacity to 0.9. | `spring.gentle` |
| **Modal dismiss** | Close modal (swipe down or button) | Reverse of present. Gesture-driven: interactive if swiped down past 25% threshold, dismisses; otherwise snaps back. | `spring.gentle` or gesture-driven |
| **Fade** | Tab switch, sheet content change | Cross-fade with `duration.fast` (200ms), `easing.standard`. | 200ms, ease |
| **Shared element** | Contact list -> Contact detail | Avatar morphs position and size from list row to detail header using `SharedTransition`. | `spring.smooth`, 500ms max |

#### 2.6.6 Micro-Interaction Animations

**Button press feedback:**
- Scale: `1.0 -> 0.97` using `spring.snappy`
- Opacity: unchanged (scale alone is sufficient)
- Duration: immediate response on touch-down, spring back on touch-up

**Toggle switch:**
- Thumb position: animated with `spring.snappy`
- Track color: cross-fade `duration.instant` (100ms)
- Haptic: `UIImpactFeedbackGenerator` style `light` on toggle

**Pull-to-refresh:**
- Spinner appears with `spring.bouncy`
- Color: `brand.primary`
- Rotation: continuous 360deg at `duration.skeleton` (1500ms) per revolution, `easing.linear`
- Dismiss: fade out over `duration.fast` (200ms)

**Swipe actions (list item):**
- Reveal: gesture-driven, follows finger position with no spring (direct manipulation)
- Snap open: `spring.stiff` to snap to 80pt reveal width
- Full swipe: when gesture passes 60% of screen width, auto-complete to full width with `spring.responsive`
- Action execution: icon scales from 1.0 to 1.2 with `spring.bouncy`, then disappears

#### 2.6.7 Loading State Animations

**AI thinking indicator (pulsing dots):**
- Three dots, each 6pt diameter
- Color: `brand.secondary`
- Animation: sequential scale pulse (1.0 -> 1.4 -> 1.0) with 200ms stagger between dots
- Total cycle: 1200ms, repeats indefinitely
- Easing: `easing.standard`

**AI streaming text:**
- Characters appear with no animation (instant render as received from stream)
- Cursor: blinking vertical bar (|), 1.5pt wide, `brand.secondary` color
- Blink cycle: 530ms visible, 530ms hidden (standard cursor blink rate)
- Cursor disappears when streaming completes

**Skeleton shimmer:**
- Base color: `fill.quaternary`
- Shimmer highlight: `fill.tertiary`
- Gradient width: 40% of container width
- Animation: gradient translates from -100% to 200% over `duration.skeleton` (1500ms)
- Easing: `easing.linear`
- Repeats indefinitely until content loads
- Skeleton shapes match the layout of the content they replace (text lines, avatars, images)

**Progress indicators:**
- Circular: 24pt diameter, 2.5pt stroke, `brand.primary` color, indeterminate rotation at 1000ms/revolution
- Linear: 2pt height, full width, `brand.primary` fill with `brand.primaryMuted` track

#### 2.6.8 Success/Error Animations

**Success checkmark:**
- Container: 64pt circle, `semantic.success` background with `opacity.hover` (0.08)
- Checkmark: drawn with a path animation (stroke-dashoffset technique)
- Draw duration: `duration.normal` (300ms) with `easing.enter`
- After draw: circle scales from 1.0 to 1.05 to 1.0 with `spring.bouncy`
- Haptic: `UINotificationFeedbackGenerator` type `success`

**Error shake:**
- Horizontal shake: translateX oscillation: 0 -> 8 -> -8 -> 6 -> -6 -> 4 -> -4 -> 2 -> -2 -> 0
- Duration: `duration.normal` (300ms)
- Easing: `easing.standard`
- Applied to: the form field or element with the error
- Haptic: `UINotificationFeedbackGenerator` type `error`
- Visual: border color transitions to `border.error` simultaneously

#### 2.6.9 Shared Element Transitions

**Contact avatar (list to detail):**
- `sharedTransitionTag`: `"contact-avatar-{id}"`
- Properties animated: `width`, `height`, `originX`, `originY`, `borderRadius`
- List avatar: 44pt x 44pt, `borderRadius.full`
- Detail avatar: 80pt x 80pt, `borderRadius.full`
- Spring: `spring.smooth` (mass:1, damping:22, stiffness:150)

**Property card (list to detail):**
- `sharedTransitionTag`: `"property-image-{id}"`
- Properties animated: `width`, `height`, `originX`, `originY`, `borderRadius`
- List thumbnail: 100% width x 180pt height, `borderRadius.lg` (14pt)
- Detail hero: 100% width x 280pt height, `borderRadius.none` (0pt)
- Spring: `spring.smooth`

#### 2.6.10 Gesture-Driven Animations

**Interactive modal dismissal:**
- Trigger: pan gesture downward on modal content
- Progress mapping: finger Y translation / 300pt = progress (0.0 to 1.0)
- Modal translation: follows finger Y directly (no spring, direct manipulation)
- Modal scale: interpolate from 1.0 to 0.9 over progress 0.0 to 1.0
- Background scrim opacity: interpolate from `opacity.overlay` to 0.0
- Dismiss threshold: progress > 0.25 (75pt of drag)
- If released above threshold: snap back with `spring.gentle`
- If released below threshold: animate to dismiss with `spring.gentle`
- Rubber-band resistance: when dragging up (above starting position), apply 0.3x dampening factor

**Rubber-band overscroll:**
- Applied automatically by iOS ScrollView
- Custom implementation for non-scroll containers: `translation * (1 - translation / (translation + 300))`
- Maximum rubber-band distance: 80pt
- Release: spring back with `spring.stiff`

**Swipe-to-navigate-back (iOS):**
- Uses React Navigation's native stack with system gesture recognizer
- Shared element transitions are progress-based (track finger position)
- Cancel threshold: less than 50% progress when released
- Complete threshold: greater than 50% progress or velocity > 500pt/s

#### 2.6.11 Reduce Motion Alternatives

When the user has enabled "Reduce Motion" in iOS Settings (`UIAccessibility.isReduceMotionEnabled`), every animation is replaced:

| Standard Animation | Reduce Motion Alternative |
|---|---|
| Push/Pop slide transitions | Cross-dissolve fade, `duration.normal` (300ms) |
| Modal slide-up | Cross-dissolve fade, `duration.normal` |
| Spring bounces | Timing-based with `easing.standard`, no overshoot |
| Shared element transitions | Cross-dissolve fade |
| Skeleton shimmer sweep | Static skeleton with pulsing opacity (0.4 to 0.7) |
| AI shimmer gradient | Static border in `brand.secondary` with opacity pulse |
| Success checkmark draw | Instant full checkmark, no animation |
| Error shake | Red border flash (opacity 0 to 1 to 0), no position change |
| Pull-to-refresh rotation | Static spinner icon with opacity pulse |
| Engagement dot pulse (hot) | No pulse, static dot |
| Button scale press | Opacity change (1.0 to 0.7) instead of scale |

**Implementation:** Create a `useReducedMotion()` hook that reads `AccessibilityInfo.isReduceMotionEnabled` and returns the appropriate animation configuration. All animated components must accept this flag.

---

### 2.7 Dark Mode Design Philosophy

#### 2.7.1 Core Principles

Skye's dark mode is not a color-inverted light mode. It is a purpose-built visual system designed for:

1. **OLED power efficiency** -- True black (`#000000`) canvas background saves significant battery on OLED displays (iPhone X and later).
2. **Reduced eye strain** -- Dark surfaces with carefully calibrated contrast reduce eye fatigue during evening and nighttime use (common for agents reviewing contacts after showings).
3. **Maintained hierarchy** -- The four-tier background system (canvas, surface, elevated, overlay) uses luminance steps that preserve the same depth hierarchy as light mode.
4. **Content prominence** -- Property photos, agent headshots, and AI-generated content are the stars. Dark chrome recedes, making content pop.

#### 2.7.2 Material Hierarchy (Dark Mode)

Skye maps to Apple's material system for blur/vibrancy effects:

| Material | Blur Radius | Background Alpha (Dark) | iOS Equivalent | Usage in Skye |
|---|---|---|---|---|
| Ultra Thin Material | 5pt | 0.40 | `.ultraThinMaterial` | Floating labels over property images |
| Thin Material | 10pt | 0.60 | `.thinMaterial` | Tab bar, navigation bar |
| Regular Material | 20pt | 0.72 | `.regularMaterial` | Bottom sheets, action sheets |
| Thick Material | 30pt | 0.85 | `.thickMaterial` | Full-screen modals, alerts |
| Chrome Material | 15pt | 0.80 | `.bar` | Toolbar backgrounds, segmented controls on bar |

In dark mode, the material tint color shifts from white-based to a warm gray (`#1E1E1E`) to avoid a cold, clinical feel. This matches Apple's own dark mode materials.

#### 2.7.3 Image Treatment in Dark Mode

Property photos, contact avatars, and any user-uploaded images receive special treatment in dark mode to prevent them from "blowing out" against the dark UI:

| Image Type | Treatment |
|---|---|
| Property photos (full width) | 15% opacity black overlay on top (`rgba(0, 0, 0, 0.15)`), rounded corners maintained |
| Property photos (thumbnail in list) | No overlay (too small to cause blow-out) |
| Contact avatars | No overlay (circular crop + small size limits contrast shock) |
| AI-generated content images (Studio) | 10% opacity black overlay (`rgba(0, 0, 0, 0.10)`) |
| Onboarding/marketing images | Dedicated dark mode image variants (pre-authored for dark backgrounds) |
| Map views | Dark map style (`mapbox://styles/mapbox/dark-v11` or Apple Maps dark) |
| Empty state illustrations | Separate dark mode illustration set with muted colors and dark-friendly palette |

**Implementation:** Wrap image components in a container that conditionally renders an overlay `View` with `position: 'absolute'` and the appropriate background color based on the current color scheme.

#### 2.7.4 Shadow Behavior in Dark Mode

As specified in Section 2.5.2, shadows are replaced in dark mode:

- **Elevation via luminance:** Higher-elevation surfaces are literally lighter (higher luminance). `bg.canvas` (#000000) -> `bg.surface` (#1C1C1E) -> `bg.elevated` (#2C2C2E).
- **Luminance edge borders:** A subtle top border (`rgba(255, 255, 255, 0.04)` to `rgba(255, 255, 255, 0.10)`) simulates the light catching the top edge of an elevated element.
- **No drop shadows:** `shadowOpacity: 0` in dark mode. Shadows on black are invisible and waste GPU resources.

#### 2.7.5 Color Adjustments for Dark Mode

Not all light mode colors can simply be brightened for dark mode. Specific adjustments:

| Element | Light Mode | Dark Mode | Why Different |
|---|---|---|---|
| `brand.primary` | `#0066FF` | `#3D8AFF` | Lighter blue reduces vibration against dark backgrounds |
| `semantic.error` | `#DC3545` | `#FF6B6B` | Desaturated red prevents harsh vibration on black |
| `semantic.success` | `#28A745` | `#34D058` | Brighter green maintains visibility without neon appearance |
| `semantic.warning` | `#E8910D` | `#FFB340` | Brighter amber ensures visibility against dark surfaces |
| Engagement dot (hot) | `#E53935` | `#FF5252` | Slightly brightened for visibility, reduced saturation |
| Card borders | `rgba(60,60,67,0.12)` | `rgba(84,84,88,0.65)` | Higher opacity in dark mode to be visible against dark bg |
| Separator | `rgba(60,60,67,0.29)` | `rgba(84,84,88,0.60)` | Higher opacity for visibility |

**Desaturation rule:** Any fully-saturated color (S > 85% in HSL) used in light mode must be desaturated by 10-15% in dark mode to prevent visual vibration against dark backgrounds. This is why `#DC3545` becomes `#FF6B6B` (softer, warmer red) rather than simply brightness-boosted.

#### 2.7.6 Status Bar Handling

| Mode | Status Bar Style | Background Treatment |
|---|---|---|
| Light mode | `.darkContent` (dark text/icons) | Transparent, content scrolls under |
| Dark mode | `.lightContent` (light text/icons) | Transparent, content scrolls under |
| Over hero image (light mode) | `.lightContent` | Gradient overlay on image: transparent to `rgba(0,0,0,0.30)` top 80pt |
| Over hero image (dark mode) | `.lightContent` | Gradient overlay on image: transparent to `rgba(0,0,0,0.50)` top 80pt |
| During modal presentation | Matches modal appearance | Automatic if using native stack |

**Implementation:** Use React Navigation's `headerTransparent: true` and manage status bar style with `<StatusBar barStyle={colorScheme === 'dark' ? 'light-content' : 'dark-content'} />`. For screens with hero images, use `StatusBar.setBarStyle('light-content')` on mount and restore on unmount.

#### 2.7.7 Dark Mode Transition

When the user switches between light and dark mode (either via system settings or in-app toggle):

- Transition duration: `duration.normal` (300ms)
- All color properties cross-fade simultaneously
- No flicker: use React Native's `Appearance` API with a debounced listener (50ms debounce to prevent rapid toggling)
- Persistent preference: store user preference in AsyncStorage. Default to `'system'` (follow device setting).
- Three modes supported: `'light'`, `'dark'`, `'system'`

---

### 2.8 Brand Identity Integration

#### 2.8.1 Skye Logo

**Logo concept:** The Skye wordmark uses SF Pro Rounded Bold with custom letter-spacing (+1.2pt tracking). The "S" in Skye features a subtle gradient treatment (from `brand.primary` to `brand.secondary`) that evokes the sky/horizon metaphor.

**Logo mark:** A simplified abstract symbol representing a horizon line with a rising arc -- suggesting skyline, aspiration, and vision. Clean geometric construction with 2pt stroke weight at 24pt size.

**Logo usage specifications:**

| Variant | Usage | Minimum Size |
|---|---|---|
| Logo mark + wordmark (horizontal) | Splash screen, onboarding, marketing materials | 120pt wide |
| Logo mark only | App icon, chat AI avatar, compact header | 24pt |
| Wordmark only | Navigation bar title (Settings > About), legal | 60pt wide |

**Color variations:**

| Context | Logo Colors |
|---|---|
| Light background | `brand.primary` mark + `text.primary` wordmark |
| Dark background | `#FFFFFF` mark + `#FFFFFF` wordmark |
| On brand.primary background | `#FFFFFF` mark + `#FFFFFF` wordmark |
| Monochrome (legal/print) | Single color -- `text.primary` for both |

**Clear space:** Minimum clear space around the logo equals the height of the "k" in "Skye" (cap height) on all four sides.

#### 2.8.2 Splash Screen Design

**Static splash (LaunchScreen.storyboard):**
- Background: `bg.canvas` (white in light mode, black in dark mode)
- Center: Skye logo mark at 80pt, `brand.primary` color
- No text on static splash (text renders inconsistently across devices during cold start)

**Animated transition (after app load, before first screen):**
1. **Frame 0-300ms:** Logo mark holds at center, 80pt.
2. **Frame 300-600ms:** Logo mark scales down from 80pt to 28pt using `spring.smooth`, translates to final position (top-left of first screen header or center of tab bar chat icon). Simultaneously, the first screen content fades in from `opacity: 0` to `opacity: 1` using `easing.enter`.
3. **Frame 600-800ms:** First screen fully visible. Logo mark settles into its final resting position (AI chat avatar or app header).

**Reduce Motion alternative:** Logo holds for 500ms, then cross-dissolve to first screen over 300ms. No position animation.

**Implementation:** Use `react-native-bootsplash` for the native static splash. The animated transition is a custom React component that renders over the first screen, animates, then unmounts.

#### 2.8.3 App Icon Specifications

**Source artwork:** 1024x1024px PNG, no transparency, no rounded corners (iOS applies the mask automatically).

**Design description:**
- Background: Linear gradient from `#0052CC` (top-left) to `#6E42D2` (bottom-right)
- Foreground: White Skye logo mark, centered, sized at approximately 60% of the icon area
- No text in the icon (illegible at small sizes)
- Subtle inner glow effect at the top to add dimension (10% white overlay, feathered 200px)

**Required sizes (iOS):**

| Size (px) | Scale | Usage |
|---|---|---|
| 1024x1024 | 1x | App Store listing |
| 180x180 | 3x | iPhone home screen (60pt @3x) |
| 120x120 | 2x | iPhone home screen (60pt @2x) |
| 167x167 | 2x | iPad Pro home screen (83.5pt @2x) |
| 152x152 | 2x | iPad home screen (76pt @2x) |
| 120x120 | 3x | iPhone Spotlight (40pt @3x) |
| 80x80 | 2x | iPhone Spotlight (40pt @2x) |
| 87x87 | 3x | iPhone Settings (29pt @3x) |
| 58x58 | 2x | iPhone Settings (29pt @2x) |
| 60x60 | 3x | iPhone Notification (20pt @3x) |
| 40x40 | 2x | iPhone Notification (20pt @2x) |

**Testing:** The icon must be visually tested at all sizes, especially 58x58 (Settings) and 40x40 (Notification) to ensure the logo mark remains legible.

#### 2.8.4 Empty State Illustration Style Guide

Empty states occur on: empty contact list, empty property list, empty chat (first launch), no search results, offline state, error state.

**Illustration style:**
- **Technique:** Flat vector illustrations with limited color palette
- **Color palette:** Maximum 4 colors per illustration: `brand.primary`, `brand.secondary`, `fill.secondary` (for gray elements), and one semantic color if contextually appropriate
- **Line weight:** 2pt strokes, matching the SF Symbols regular weight visual density
- **Dimensions:** 200pt x 160pt maximum bounding box (centered in available space)
- **Character style:** No human figures (avoids cultural/demographic assumptions). Use abstract shapes, buildings, chat bubbles, stars, and property-related objects.
- **Dark mode variants:** Each illustration has a dedicated dark mode version with colors adjusted per Section 2.7.3 -- muted palette, reduced brightness, warm undertones.

**Empty state layout:**
1. Illustration (centered, 200x160pt max)
2. Gap: `spacing.xl` (24pt)
3. Title: `type.title3` (20pt, Semibold), `text.primary`, centered
4. Gap: `spacing.sm` (8pt)
5. Description: `type.body` (17pt, Regular), `text.secondary`, centered, max 2 lines
6. Gap: `spacing.lg` (16pt)
7. CTA button (if applicable): primary button style, centered

#### 2.8.5 Onboarding Visual Style

**Screen count:** 3 onboarding screens maximum (research shows completion rate drops after 3).

**Visual specifications per screen:**
- Hero illustration: 280pt x 220pt, centered in top 60% of screen
- Illustration style: matches empty state style guide (Section 2.8.4) but larger and more detailed
- Background: `bg.canvas` with optional subtle gradient overlay (`gradient.premium` at 5% opacity)
- Title: `type.title1` (28pt, Bold), `text.primary`, center-aligned
- Subtitle: `type.body` (17pt, Regular), `text.secondary`, center-aligned, max 3 lines
- Page indicator: 3 dots, 8pt diameter, `spacing.sm` (8pt) gap between dots
  - Active: `brand.primary`, scale 1.0
  - Inactive: `fill.secondary`, scale 1.0
  - Transition: dot color cross-fades over `duration.fast` (200ms)
- CTA button: full-width, 50pt height, `brand.primary` background, `text.onPrimary` label, `borderRadius.md` (10pt)
- Skip button: `type.body` (17pt, Regular), `text.brand` color, top-right corner

**Transition between onboarding screens:**
- Swipe gesture-driven, follows finger position
- Page indicator updates in sync with gesture progress
- Spring: `spring.responsive` on release

---

### 2.9 Design Token Implementation Guide

#### 2.9.1 Token File Structure

```
src/
  theme/
    tokens/
      colors.ts          # All color primitives and semantic tokens
      typography.ts       # Type scale, font weights, font families
      spacing.ts          # Spacing tokens and layout constants
      borderRadius.ts     # Corner radius scale
      shadows.ts          # Shadow/elevation definitions (light + dark)
      animation.ts        # Duration, easing, spring configurations
      blur.ts             # Material/blur specifications
      icons.ts            # Icon size and weight mappings
    index.ts              # Unified theme export
    ThemeProvider.tsx      # React Context provider for theme
    useTheme.ts           # Hook for accessing current theme
    useReducedMotion.ts   # Hook for accessibility motion preference
    tokens.types.ts       # TypeScript type definitions for all tokens
```

#### 2.9.2 Color Token Resolution

```typescript
// Simplified example of token resolution
type ColorScheme = 'light' | 'dark';

const colors = {
  light: {
    brand: { primary: '#0066FF', secondary: '#6E42D2' },
    bg: { canvas: '#F2F2F7', surface: '#FFFFFF', elevated: '#FFFFFF' },
    text: { primary: '#000000', secondary: 'rgba(60, 60, 67, 0.60)' },
    // ... all light mode tokens
  },
  dark: {
    brand: { primary: '#3D8AFF', secondary: '#9B7AE8' },
    bg: { canvas: '#000000', surface: '#1C1C1E', elevated: '#2C2C2E' },
    text: { primary: '#FFFFFF', secondary: 'rgba(235, 235, 245, 0.60)' },
    // ... all dark mode tokens
  },
};

// Usage in components:
const theme = useTheme(); // resolves to light or dark based on current scheme
// <View style={{ backgroundColor: theme.colors.bg.surface }} />
// <Text style={{ color: theme.colors.text.primary }} />
```

#### 2.9.3 Accessibility Audit Checklist

Every new screen and component must pass these checks before merge:

- All `text.primary` on `bg.surface` combinations meet 4.5:1 contrast ratio (WCAG AA)
- All `text.secondary` on `bg.surface` combinations meet 4.5:1 contrast ratio
- All `text.primary` on `bg.canvas` combinations meet 4.5:1 contrast ratio
- All large text (18pt+ or 14pt+ bold) on all backgrounds meets 3:1 contrast ratio
- All interactive icon colors on their backgrounds meet 3:1 contrast ratio
- All engagement health dot colors are paired with text labels for VoiceOver
- All animations have Reduce Motion alternatives implemented
- All text uses Dynamic Type text styles (not hardcoded sizes)
- All interactive elements have minimum 44x44pt touch targets
- Dark mode has been tested on actual OLED device (not just simulator)
- Images in dark mode have appropriate overlay treatment
- Status bar style is correct in both light and dark modes

---

### 2.10 Quick Reference: Token Summary

**Color Tokens (35 tokens):**
`brand.primary`, `brand.primaryHover`, `brand.primaryMuted`, `brand.secondary`, `brand.secondaryMuted`, `semantic.success`, `semantic.warning`, `semantic.error`, `semantic.info`, `bg.canvas`, `bg.surface`, `bg.elevated`, `bg.overlay`, `bg.sunken`, `text.primary`, `text.secondary`, `text.tertiary`, `text.quaternary`, `text.inverse`, `text.brand`, `text.onPrimary`, `engagement.hot`, `engagement.warm`, `engagement.cool`, `engagement.cold`, `engagement.inactive`, `fill.primary`, `fill.secondary`, `fill.tertiary`, `fill.quaternary`, `border.default`, `border.focused`, `border.error`, `separator.default`, `separator.opaque`

**Typography Tokens (17 tokens):**
`type.largeTitle`, `type.title1`, `type.title2`, `type.title3`, `type.headline`, `type.body`, `type.bodyEmphasized`, `type.callout`, `type.subheadline`, `type.subheadlineEmphasized`, `type.footnote`, `type.footnoteEmphasized`, `type.caption1`, `type.caption1Emphasized`, `type.caption2`, `type.code`, `type.codeBlock`

**Spacing Tokens (9 tokens):**
`spacing.xxs` (2), `spacing.xs` (4), `spacing.sm` (8), `spacing.md` (12), `spacing.lg` (16), `spacing.xl` (24), `spacing.xxl` (32), `spacing.xxxl` (48), `spacing.xxxxl` (64)

**Border Radius Tokens (8 tokens):**
`borderRadius.none` (0), `borderRadius.xs` (4), `borderRadius.sm` (6), `borderRadius.md` (10), `borderRadius.lg` (14), `borderRadius.xl` (20), `borderRadius.xxl` (24), `borderRadius.full` (9999)

**Shadow Tokens (6 tokens):**
`shadow.none`, `shadow.xs`, `shadow.sm`, `shadow.md`, `shadow.lg`, `shadow.xl`

**Animation Tokens (18 tokens):**
`duration.instant` (100ms), `duration.fast` (200ms), `duration.normal` (300ms), `duration.slow` (400ms), `duration.dramatic` (500ms), `duration.skeleton` (1500ms), `duration.aiShimmer` (2000ms), `easing.standard`, `easing.enter`, `easing.exit`, `easing.emphasized`, `easing.linear`, `spring.snappy`, `spring.responsive`, `spring.smooth`, `spring.gentle`, `spring.bouncy`, `spring.stiff`

---

*This specification is the authoritative source for all visual design decisions in Skye V2. No engineer should need to guess a value. No designer should need to reinvent a pattern. When in doubt, refer to this document. When this document is silent, refer to the Apple Human Interface Guidelines.*

---

**Sources consulted during specification authoring:**

- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [Apple HIG - Color](https://developer.apple.com/design/human-interface-guidelines/color)
- [Apple HIG - Typography](https://developer.apple.com/design/human-interface-guidelines/typography)
- [Apple HIG - SF Symbols](https://developer.apple.com/design/human-interface-guidelines/sf-symbols)
- [Apple HIG - Dark Mode](https://developer.apple.com/design/human-interface-guidelines/dark-mode)
- [WWDC25 - Meet Liquid Glass](https://developer.apple.com/videos/play/wwdc2025/219/)
- [WWDC25 - Get to know the new design system](https://developer.apple.com/videos/play/wwdc2025/356/)
- [WWDC20 - The Details of UI Typography](https://developer.apple.com/videos/play/wwdc2020/10175/)
- [WWDC22 - Meet the Expanded San Francisco Font Family](https://developer.apple.com/videos/play/wwdc2022/110381/)
- [WWDC24 - Get Started with Dynamic Type](https://developer.apple.com/videos/play/wwdc2024/10074/)
- [Apple Standard Colors Documentation](https://developer.apple.com/documentation/uikit/standard-colors)
- [Noah Gilmore - iOS 13 System Colors Reference](https://noahgilmore.com/blog/dark-mode-uicolor-compatibility)
- [Apple Colors Interactive Palette](https://mar.codes/apple-colors)
- [Dark Color Cheat Sheet](https://sarunw.com/posts/dark-color-cheat-sheet/)
- [React Native Reanimated - withSpring](https://docs.swmansion.com/react-native-reanimated/docs/animations/withSpring/)
- [React Native Reanimated - Shared Element Transitions](https://docs.swmansion.com/react-native-reanimated/docs/shared-element-transitions/overview/)
- [React Native Shadow Props](https://reactnative.dev/docs/shadow-props)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [WCAG 2.1 Contrast Requirements](https://www.makethingsaccessible.com/guides/contrast-requirements-for-wcag-2-2-level-aa/)
- [Linear App - How We Redesigned the UI](https://linear.app/now/how-we-redesigned-the-linear-ui)
- [SF Symbols 7 Release Notes](https://9to5mac.com/2025/06/11/apple-releases-sf-symbols-7-beta/)
- [react-native-bootsplash](https://github.com/zoontek/react-native-bootsplash)

---


## Section 3: Chat Engine & AI Interface

---

### 2.1 Streaming Engine Architecture

#### 2.1.1 SSE Connection State Machine

The streaming engine operates as a deterministic finite state machine with six states. Every SSE interaction follows this lifecycle exactly.

```
States:
  IDLE          → No active connection. Default state on mount and after graceful close.
  CONNECTING    → POST request initiated to /api/chat. Awaiting first byte.
  STREAMING     → First SSE event received. Tokens flowing and rendering.
  COMPLETING    → Final SSE event received ([DONE] sentinel). Flushing remaining buffer.
  ERROR         → Connection failed or stream interrupted. Triggers reconnection protocol.
  CLOSED        → Connection explicitly torn down. Terminal state before returning to IDLE.

Transitions:
  IDLE → CONNECTING          User taps send or retry.
  CONNECTING → STREAMING     First SSE data event arrives (within 10s timeout).
  CONNECTING → ERROR         Timeout (10s), network failure, or HTTP error (4xx/5xx).
  STREAMING → COMPLETING     Server sends [DONE] event or SSE stream closes cleanly.
  STREAMING → ERROR          Connection drops, malformed event, heartbeat timeout (15s).
  COMPLETING → CLOSED        Final buffer flush complete, response persisted to local store.
  CLOSED → IDLE              Cleanup complete, ready for next request.
  ERROR → CONNECTING         Auto-reconnect triggered (within retry budget).
  ERROR → IDLE               Retry budget exhausted. Manual retry required.
```

**Implementation:** Store the state in a `useRef` (not `useState`) to avoid re-renders on high-frequency transitions. Expose a derived `streamingStatus` via `useSyncExternalStore` for UI consumption, batched to update at most once per frame.

#### 2.1.2 SSE Connection Protocol

```
Request:
  POST /api/chat
  Headers:
    Content-Type: application/json
    Accept: text/event-stream
    Authorization: Bearer {accessToken}
    X-Chat-Session-Id: {sessionId}
    X-Client-Message-Id: {uuidv4}  // Idempotency key for deduplication
  Body:
    {
      "message": string,
      "sessionId": string,
      "attachments": Attachment[] | null
    }

Response Stream (SSE format):
  event: token
  data: {"content": "Hello", "index": 0}

  event: token
  data: {"content": " there", "index": 1}

  event: tool_call_start
  data: {"toolCallId": "tc_001", "toolName": "lookupContact", "args": {"name": "Marcus Johnson"}}

  event: tool_call_result
  data: {"toolCallId": "tc_001", "result": {ContactObject}, "status": "success"}

  event: token
  data: {"content": "I found Marcus...", "index": 47}

  event: done
  data: {"messageId": "msg_xyz", "usage": {"promptTokens": 340, "completionTokens": 289}}
```

#### 2.1.3 Chunked Response Parsing and Token Buffering

Tokens arrive as individual SSE `data:` lines. The parser operates as follows:

1. **Line accumulator:** An internal `StringBuilder` accumulates bytes until a double newline (`\n\n`) is detected, signaling a complete SSE event.
2. **Event parser:** Each complete event is split into `event:` and `data:` fields. The `data:` field is parsed as JSON. If JSON parsing fails, the raw string is treated as a text token (defensive fallback).
3. **Token buffer:** Parsed tokens are appended to a `tokenBuffer` array (a `useRef<string[]>`). A `requestAnimationFrame` loop drains the buffer and appends tokens to the displayed message string. This decouples network I/O frequency from render frequency.
4. **Render batching:** The RAF loop concatenates all buffered tokens into a single string append per frame. At 60fps, this means the UI updates at most every 16.67ms, regardless of how rapidly tokens arrive. Typical LLM output (40-80 tokens/second) results in 1-2 tokens per frame.
5. **Flush on complete:** When the `done` event arrives, the buffer is immediately flushed (bypassing RAF scheduling) to ensure the complete response is visible without a frame delay.

```typescript
// Pseudocode for token buffer drain
const tokenBufferRef = useRef<string[]>([]);
const displayTextRef = useRef<string>('');

function onSSEToken(token: string) {
  tokenBufferRef.current.push(token);
}

function drainBuffer() {
  if (tokenBufferRef.current.length > 0) {
    const batch = tokenBufferRef.current.splice(0);
    displayTextRef.current += batch.join('');
    // Trigger re-render of only the active message component
    setActiveMessageText(displayTextRef.current);
  }
  if (streamStateRef.current === 'STREAMING') {
    requestAnimationFrame(drainBuffer);
  }
}
```

#### 2.1.4 Reconnection Protocol

When a connection drops during streaming, the following protocol activates:

| Parameter | Value |
|---|---|
| Detection method | Heartbeat timeout: server sends `:heartbeat\n\n` every 10s. If no event (data or heartbeat) received for 15s, connection is considered dead. |
| Base delay | 1,000ms |
| Backoff formula | `min(baseDelay * 2^attempt + jitter, maxDelay)` |
| Jitter | Random 0-500ms added to each delay to prevent thundering herd |
| Max delay | 30,000ms |
| Max automatic retries | 3 |
| After max retries | Transition to IDLE. Display inline error: "Connection lost. Tap to retry." with manual retry button. |
| Reconnection request | Includes `X-Resume-After-Token: {lastReceivedTokenIndex}` header so the server can resume from the correct position. If server does not support resumption, full response is re-fetched and diffed against local buffer. |

**Backoff sequence:** 1s → 2s → 4s (plus jitter), then manual retry.

#### 2.1.5 Partial Response Preservation

If the stream drops after receiving N tokens, those N tokens remain permanently visible in the chat. The message is marked with a `partial: true` flag in local state.

- **Visual treatment:** The partial message renders normally with a subtle footer indicator: "Response interrupted" in muted text with a "Retry" link.
- **On retry:** A new SSE connection is opened. If the server supports resumption (via `X-Resume-After-Token`), new tokens are appended. If not, the AI regenerates the full response — the previous partial is replaced with the new complete response using a crossfade animation (200ms).
- **Persistence:** Partial messages are saved to local storage with `status: 'partial'`. On app relaunch, they display with the interrupted indicator.

#### 2.1.6 Concurrent Request Handling

**Rule: Only one active SSE stream per chat session at any time.**

If the user sends a new message while a previous response is still streaming:

1. The current SSE connection is aborted via `AbortController.abort()`.
2. The partial AI response is preserved in the message list with `status: 'partial'` and no interrupted indicator (it was intentionally cut short).
3. The new user message is appended to the message list.
4. A new SSE connection is established for the new message.
5. The send button is disabled (grayed, non-interactive) while the AI is actively streaming. The input field remains editable so the user can compose their next message, but they cannot send until streaming completes or they explicitly cancel the current stream via a "Stop generating" button that replaces the send button during streaming.

#### 2.1.7 Background Behavior (iOS)

iOS grants approximately 30 seconds of background execution time after the app moves to the background.

| App State | Behavior |
|---|---|
| **Active → Background** | Stream continues for the iOS background execution window (~30s). Register a background task via `UIApplication.beginBackgroundTask` (bridged through `react-native-background-actions` or a thin native module). |
| **Background, stream completes within 30s** | Response is persisted to local storage. A local notification is posted: "Skye has a response ready." |
| **Background, 30s elapses, stream still active** | Gracefully close the SSE connection. Persist the partial response with `status: 'partial'`. When app returns to foreground, display partial + "Response interrupted" indicator with retry. |
| **Background → Foreground** | Check `streamStateRef`. If IDLE with a pending partial, show retry UI. If IDLE with completed response, no action needed — the response is already in the list. If somehow still STREAMING (within 30s window), continue rendering as normal. |

Listen for `AppState` changes via React Native's `AppState` API:

```typescript
useEffect(() => {
  const sub = AppState.addEventListener('change', (nextState) => {
    if (nextState === 'active') {
      // Reconnect if stream was interrupted
      if (streamStateRef.current === 'ERROR') {
        attemptReconnection();
      }
    }
  });
  return () => sub.remove();
}, []);
```

#### 2.1.8 Network Transition Handling

| Transition | Behavior |
|---|---|
| **WiFi → Cellular (mid-stream)** | The underlying TCP connection will break. The SSE `onerror` fires. The reconnection protocol (Section 2.1.4) activates immediately. Because cellular is now available, the first reconnection attempt (1s delay) typically succeeds. Partial tokens are preserved. |
| **Cellular → WiFi (mid-stream)** | Same behavior — TCP connection breaks, reconnection protocol activates. WiFi is typically faster, so reconnection succeeds on first attempt. |
| **Any → Offline** | `NetInfo` listener detects offline state. Reconnection attempts are paused (no point retrying with no network). A banner appears at the top of the chat: "No internet connection." When connectivity is restored (`NetInfo` fires), reconnection resumes automatically. |
| **Implementation** | Use `@react-native-community/netinfo` to subscribe to connectivity changes. Combine with the SSE state machine: if `NetInfo.isConnected === false`, transition to a `WAITING_FOR_NETWORK` sub-state that bypasses the retry counter. |

#### 2.1.9 Token-by-Token Rendering Performance

**Target: 60fps (16.67ms per frame) during rapid token delivery.**

Strategies:

1. **Isolate re-renders:** Only the actively-streaming message component re-renders on each token batch. All other messages in the list are wrapped in `React.memo` with stable props and do not re-render.
2. **RAF-batched updates:** As described in Section 2.1.3, tokens are batched per animation frame. The string concatenation happens outside React's render cycle (in refs), and a single `setState` call per frame triggers the re-render.
3. **Avoid layout thrashing:** The streaming message uses a fixed-width container. As text grows, only the height changes. Use `onContentSizeChange` on the text container to drive height animation via `react-native-reanimated` shared values — no layout recalculation cascade.
4. **Markdown parsing deferral:** During active streaming, raw text is rendered (no markdown parsing). Markdown is parsed and rendered only after: (a) a 300ms pause in token delivery, or (b) the stream completes. This eliminates the cost of re-parsing growing markdown on every token. During streaming, basic formatting (bold with `**`, inline code with backticks) is applied via regex-based lightweight styling, not full AST parsing.
5. **Off-thread markdown parsing:** When full markdown parsing triggers, it runs on a JS worker thread (via `react-native-multithreading` or a dedicated Hermes worker) or uses incremental parsing — parse only the delta since the last parse, not the entire string.

#### 2.1.10 Memory Management for Long Sessions

**Target: <150MB for 500 messages. Graceful handling of 1000+ messages.**

| Strategy | Detail |
|---|---|
| **Virtualized list** | FlashList (v2) with cell recycling. Only messages in the viewport plus a 5-screen buffer above and below are mounted. All other messages exist only as data in the messages array. |
| **Message data pagination** | On initial load, fetch the most recent 50 messages from `/api/chat/history`. When the user scrolls to the top, fetch the next 50 (paginated via cursor). Maximum 200 messages in memory at any time. When a new page is loaded at the top, the oldest page at the bottom is evicted from the rendered array (but the full message IDs are retained for potential re-fetch). |
| **Image memory management** | Tool result card images (property thumbnails, avatars) use `react-native-fast-image` with aggressive disk caching and memory cache limits. Images not in the viewport are evicted from memory cache. Blurhash placeholders (stored as 20-30 byte strings) are used when images are recycled out and back in. |
| **Markdown AST caching** | Parsed markdown ASTs are memoized per `messageId`. When a message scrolls out of the virtualization window and back in, the cached AST is reused — no re-parsing. Cache is an LRU with a 100-entry limit. |
| **String interning** | Repeated strings in tool results (status labels, field names) use a shared string pool to reduce memory duplication. |
| **Memory pressure response** | Subscribe to iOS memory warnings via a native module. On `didReceiveMemoryWarning`, aggressively evict: image caches, markdown AST cache, and reduce the virtualization buffer from 5 screens to 2 screens. Log the event for analytics. |

---

### 2.2 Message Type System

Every message conforms to a base type and is rendered by a dedicated component. The message list is a heterogeneous list — FlashList's `getItemType` returns the message type string, enabling optimal cell recycling.

#### 2.2.1 Base Message Schema

```typescript
interface BaseMessage {
  id: string;                        // UUID, globally unique
  sessionId: string;                 // Chat session this message belongs to
  type: MessageType;                 // Discriminated union key
  timestamp: string;                 // ISO 8601
  status: 'sending' | 'sent' | 'partial' | 'failed' | 'delivered';
}

type MessageType =
  | 'user_text'
  | 'ai_text'
  | 'ai_tool_call'
  | 'ai_tool_result'
  | 'ai_action_confirmation'
  | 'system';
```

#### 2.2.2 User Text Message

```typescript
interface UserTextMessage extends BaseMessage {
  type: 'user_text';
  content: string;
  attachments?: Attachment[];  // Images from OCR flow
}

interface Attachment {
  id: string;
  uri: string;
  mimeType: 'image/jpeg' | 'image/png' | 'image/heic';
  thumbnailUri: string;
  width: number;
  height: number;
  ocrText?: string;  // Extracted text if OCR was performed
}
```

**Layout specification:**

| Property | Value |
|---|---|
| Alignment | Right-aligned (trailing edge) |
| Max width | 78% of screen width |
| Bubble background | `#007AFF` (Skye brand blue) |
| Bubble corner radius | 18pt. Bottom-right corner: 4pt (tail effect for last message in group) |
| Text color | `#FFFFFF` |
| Text font | SF Pro, 16pt, regular weight |
| Horizontal padding (inside bubble) | 14pt left, 14pt right |
| Vertical padding (inside bubble) | 10pt top, 10pt bottom |
| Bubble margin from screen edge | 16pt right |
| Bubble margin from opposite edge | Minimum 54pt left (ensures asymmetry) |
| Avatar | None displayed for user messages |
| Attachment rendering | Images render as rounded rectangles (12pt radius) above the text, max 240pt wide, aspect-ratio preserved, tap to view full-screen. Multiple images arranged in a 2-column grid. |

**Sending states:**

| State | Visual Treatment |
|---|---|
| `sending` | Bubble at 60% opacity. No timestamp yet. Subtle upward slide-in animation (translateY: 20 → 0, 200ms ease-out). |
| `sent` | Full opacity. Timestamp available on tap. |
| `failed` | Bubble background shifts to `#FF3B30` (system red) at 80% opacity. Below the bubble: "Not sent. Tap to retry." in 13pt, `#FF3B30`. Tap retries the send. |

#### 2.2.3 AI Text Message

```typescript
interface AITextMessage extends BaseMessage {
  type: 'ai_text';
  content: string;          // Markdown string, grows during streaming
  isStreaming: boolean;      // True while tokens are still arriving
  modelId?: string;          // For display if needed in future
}
```

**Layout specification:**

| Property | Value |
|---|---|
| Alignment | Left-aligned (leading edge) |
| Max width | 85% of screen width |
| Bubble background | `#F2F2F7` (system gray 6, light mode) / `#1C1C1E` (dark mode) |
| Bubble corner radius | 18pt. Bottom-left corner: 4pt (tail effect for last message in group) |
| Text color | `#000000` (light mode) / `#FFFFFF` (dark mode) |
| Text font | SF Pro, 16pt, regular weight. Markdown-derived styles override as needed. |
| Horizontal padding (inside bubble) | 14pt left, 14pt right |
| Vertical padding (inside bubble) | 10pt top, 10pt bottom |
| Bubble margin from screen edge | 16pt left |
| Bubble margin from opposite edge | Minimum 40pt right |
| Avatar | 28pt diameter Skye logo, displayed at bottom-left of the bubble, only for the last message in a consecutive AI group. Vertically aligned with the bottom of the bubble, 8pt to the left of the bubble. |
| Streaming cursor | A blinking vertical bar (`|`) appended to the last character during streaming. Blink cycle: 530ms on, 530ms off. Disappears when streaming completes. |

#### 2.2.4 AI Tool Call Message

Displayed when the AI invokes a tool. Appears inline in the message stream as a compact status indicator.

```typescript
interface AIToolCallMessage extends BaseMessage {
  type: 'ai_tool_call';
  toolCallId: string;
  toolName: string;
  displayLabel: string;     // Human-readable: "Looking up contact..."
  args: Record<string, any>;
  status: 'running' | 'completed' | 'failed';
}
```

**Layout specification:**

| Property | Value |
|---|---|
| Alignment | Left-aligned, same indent as AI text messages |
| Max width | 85% of screen width |
| Background | None (no bubble). Rendered as a single line with icon. |
| Icon | 16pt animated icon left of text. Running: spinning circle (Lottie, 1s loop). Completed: checkmark, green `#34C759`. Failed: exclamation triangle, red `#FF3B30`. |
| Text | `displayLabel` in 14pt, SF Pro, medium weight, `#8E8E93` (secondary label color). |
| Vertical spacing | 4pt above, 4pt below. Compact — these should feel like status annotations, not full messages. |
| Animation | Slide-in from left (translateX: -10 → 0, 150ms ease-out) on appearance. Icon cross-fades between states (running → completed/failed) over 200ms. |

**Tool name to display label mapping (examples):**

| `toolName` | `displayLabel` |
|---|---|
| `lookupContact` | "Looking up contact..." |
| `searchProperties` | "Searching properties..." |
| `createContact` | "Creating contact..." |
| `generateContent` | "Generating content..." |
| `getMarketAnalysis` | "Analyzing market data..." |
| `scheduleFollowUp` | "Scheduling follow-up..." |

#### 2.2.5 AI Tool Result Message

Displayed after a tool call completes. Renders as a rich, interactive card. The specific card type depends on the tool that was called. See Section 2.4 for detailed card specifications.

```typescript
interface AIToolResultMessage extends BaseMessage {
  type: 'ai_tool_result';
  toolCallId: string;       // Links to the corresponding tool call
  toolName: string;
  result: ToolResultPayload;
  displayType: 'contact_card' | 'property_card' | 'content_preview' | 'market_analysis' | 'task_card' | 'generic';
  resultCount: number;      // If >1, renders as carousel
}
```

**Layout specification:**

| Property | Value |
|---|---|
| Alignment | Left-aligned, full width minus 32pt margins (16pt each side) |
| Max width | `screenWidth - 32pt` |
| Card styling | See Section 2.4 for per-card-type specs. All cards share: 12pt corner radius, 1pt border (`#E5E5EA` light / `#38383A` dark), subtle shadow (`color: #000, opacity: 0.08, offset: {0, 2}, blur: 8`). |
| Carousel (multiple results) | Horizontal `FlatList`, paging enabled, 8pt gap between cards. Page indicator dots below carousel: 6pt diameter, `#C7C7CC` inactive, `#007AFF` active. Cards are `screenWidth - 64pt` wide each (peek next card by 32pt). |
| Loading state | Skeleton card: same dimensions as final card, 3 animated shimmer bars (gray gradient sweep, 1.5s loop). Appears immediately when tool call starts. |
| Error state | Card with light red background (`#FFF2F2`), exclamation icon, "Couldn't complete this action" text, and "Retry" button. |

#### 2.2.6 AI Action Confirmation Message

Used for destructive or significant actions that require user approval before execution.

```typescript
interface AIActionConfirmationMessage extends BaseMessage {
  type: 'ai_action_confirmation';
  prompt: string;           // "I'll create this contact with the following details:"
  previewData: Record<string, any>;  // Structured preview of the action
  confirmLabel: string;     // "Create Contact"
  cancelLabel: string;      // "Cancel"
  actionStatus: 'pending' | 'confirmed' | 'cancelled' | 'executed';
  toolName: string;
  toolArgs: Record<string, any>;
}
```

**Layout specification:**

| Property | Value |
|---|---|
| Alignment | Left-aligned |
| Max width | 85% of screen width |
| Background | White bubble with 1pt `#E5E5EA` border (light) / `#2C2C2E` with 1pt `#48484A` border (dark) |
| Corner radius | 16pt |
| Prompt text | 15pt, SF Pro, regular, `#000000` / `#FFFFFF`. Padding: 14pt all sides. |
| Preview section | Below prompt text. Key-value pairs rendered as: key in 13pt medium `#8E8E93`, value in 15pt regular `#000000`. 8pt vertical gap between pairs. 12pt top padding, separated from prompt by a 1pt `#E5E5EA` divider. |
| Button row | Below preview, separated by 1pt divider. Two buttons side by side. |
| Confirm button | Left button: `confirmLabel` text in 15pt semibold `#007AFF`, centered, 44pt tap target height. Haptic: `impactMedium` on tap. |
| Cancel button | Right button: `cancelLabel` text in 15pt regular `#FF3B30`, centered, 44pt tap target height. |
| Button divider | 1pt vertical `#E5E5EA` line between buttons. |
| Post-action state | After confirm: buttons replaced with "Confirmed" text + checkmark in green. After cancel: buttons replaced with "Cancelled" text in muted gray. Both states are non-interactive and final. |

#### 2.2.7 System Message

Used for non-conversational information: date separators, connection status changes, errors.

```typescript
interface SystemMessage extends BaseMessage {
  type: 'system';
  subtype: 'date_separator' | 'connection_status' | 'error' | 'session_start';
  content: string;
}
```

**Layout specification:**

| Subtype | Rendering |
|---|---|
| `date_separator` | Centered text: "Today", "Yesterday", or "Mon, Jan 15". Font: 12pt, SF Pro, semibold, `#8E8E93`. 20pt vertical margin above and below. No bubble. Optional: subtle horizontal lines extending from text to edges (hairline, `#E5E5EA`). |
| `connection_status` | Centered text with icon. "Reconnecting..." with spinning indicator. "Connected" with green dot (auto-dismisses after 2s fade-out). Font: 12pt, SF Pro, medium, `#8E8E93`. |
| `error` | Centered text, red-tinted. "Something went wrong. Tap to retry." Font: 12pt, SF Pro, medium, `#FF3B30`. Tap handler for retry. |
| `session_start` | Centered: "New conversation" with a subtle horizontal rule. Font: 12pt, SF Pro, medium, `#8E8E93`. 24pt vertical margin. |

#### 2.2.8 Message Grouping

Consecutive messages from the same sender are visually grouped:

| Behavior | Specification |
|---|---|
| Avatar display | Only the last message in a consecutive group shows the avatar (AI messages only). |
| Bubble tail | Only the last message in a group has the small corner radius (tail). All other messages in the group have uniform 18pt radius on all corners. |
| Intra-group spacing | 3pt between consecutive messages from the same sender. |
| Inter-group spacing | 12pt between message groups from different senders. |
| Timestamp grouping | If all messages in a group are within 60 seconds of each other, a single timestamp applies to the group (shown on the last message on tap). If messages span >60 seconds, a subtle inline timestamp is shown at the breakpoint. |
| Tool call/result grouping | Tool call status lines and their result cards are grouped with surrounding AI text messages into a single visual unit, with no extra spacing. The entire AI "turn" (text + tool calls + results + more text) is one group. |

---

### 2.3 Markdown Rendering

#### 2.3.1 Supported Syntax (Full CommonMark + Extensions)

| Element | Syntax | Rendering |
|---|---|---|
| Bold | `**text**` | SF Pro, 16pt, **semibold** weight |
| Italic | `*text*` | SF Pro, 16pt, *italic* style |
| Bold Italic | `***text***` | SF Pro, 16pt, ***semibold italic*** |
| Strikethrough | `~~text~~` | SF Pro, 16pt, ~~line-through~~ decoration |
| Inline code | `` `code` `` | SF Mono, 14pt, background `#E8E8ED` (light) / `#3A3A3C` (dark), 3pt horizontal padding, 2pt corner radius |
| Code block | ` ```lang ` | See Section 2.3.2 |
| Links | `[text](url)` | Text in `#007AFF`, underlined on long-press only. Tap opens SFSafariViewController. |
| Headers | `# H1` through `### H3` | H1: 22pt bold. H2: 19pt bold. H3: 17pt semibold. H4-H6: 16pt semibold. All have 8pt top margin, 4pt bottom margin. |
| Blockquote | `> text` | 3pt left border in `#C7C7CC`, 12pt left padding, text in `#6C6C70` italic. Nested blockquotes increase left padding by 12pt each level. |
| Unordered list | `- item` | 6pt bullet (`#8E8E93`), 8pt left indent per nesting level, 4pt vertical gap between items. |
| Ordered list | `1. item` | Number in `#8E8E93` followed by period, same spacing as unordered. |
| Horizontal rule | `---` | 1pt line, `#E5E5EA`, full bubble width minus 28pt (14pt margin each side), 12pt vertical margin. |
| Tables | `| col | col |` | See Section 2.3.4 |

#### 2.3.2 Code Block Rendering

```
┌─────────────────────────────────────────────────┐
│ javascript                            [  Copy  ]│  ← Header bar
├─────────────────────────────────────────────────┤
│ function greetAgent(name) {                     │
│   return `Welcome, ${name}!`;                   │
│ }                                               │
└─────────────────────────────────────────────────┘
```

| Property | Value |
|---|---|
| Container background | `#1E1E1E` (always dark, regardless of app theme — convention from all major chat apps) |
| Container corner radius | 10pt |
| Container margin | 0pt (fills full bubble width minus bubble padding) |
| Header bar | 32pt height, `#2D2D2D` background, contains language label (left) and copy button (right). |
| Language label | 12pt SF Mono, `#A0A0A0`, lowercase. Detected from the code fence language tag. If unspecified, label reads "code". |
| Copy button | "Copy" text in 12pt SF Pro medium `#A0A0A0`. On tap: text changes to "Copied!" with checkmark for 2s, haptic `notificationSuccess`. Copies code content (not the fence markers or language label) to clipboard. |
| Code text | 13pt SF Mono, syntax-highlighted. Colors follow a VS Code Dark+ inspired palette. |
| Horizontal overflow | Code block scrolls horizontally. No wrapping. Horizontal scroll indicator visible during scroll. |
| Vertical overflow | No max height — code blocks expand to show all lines. Very long blocks (>30 lines) show a "Show more" / "Show less" toggle at 15 lines, with smooth expand animation. |
| Syntax highlighting | Use `react-syntax-highlighter` with a custom theme mapped to native Text components (not WebView). Support languages: javascript, typescript, python, json, html, css, sql, bash, swift, markdown. Unrecognized languages render without highlighting. |

#### 2.3.3 Link Handling

| Interaction | Behavior |
|---|---|
| Tap | Opens URL in `SFSafariViewController` (in-app browser). The chat remains in the background — user can swipe down to return. If URL is a deep link to an in-app screen (e.g., a contact or property URL matching Skye's scheme), navigate to that screen instead. |
| Long-press (500ms) | Show a context menu (native iOS `UIContextMenuInteraction` via `react-native-context-menu-view`): (1) "Open" — same as tap, (2) "Copy Link" — copies URL to clipboard with `notificationSuccess` haptic, (3) "Share" — opens iOS share sheet. A preview of the URL destination appears above the menu (iOS link preview). |
| URL display | Inline in text, styled as `#007AFF`. No underline by default (matches iOS convention). Underline appears on long-press to indicate the link is being activated. |

#### 2.3.4 Table Rendering

When the AI returns tabular data in markdown:

| Property | Value |
|---|---|
| Container | Horizontally scrollable if table exceeds bubble width. Rounded corners (8pt), 1pt border `#E5E5EA`. |
| Header row | Background `#F2F2F7` (light) / `#2C2C2E` (dark). Text: 13pt SF Pro semibold. |
| Body rows | Alternating: white / `#F9F9F9` (light) or `#1C1C1E` / `#242426` (dark). Text: 13pt SF Pro regular. |
| Cell padding | 8pt horizontal, 6pt vertical. |
| Column alignment | Auto-detect from markdown alignment syntax (`:---`, `:---:`, `---:`). Default: left-aligned. |
| Max visible rows | 10 rows visible before vertical scroll activates within the table container. |

#### 2.3.5 Extensibility Architecture

The markdown renderer is designed as a pipeline with pluggable stages:

```
Raw Markdown String
  → Tokenizer (markdown-it or mdast-util-from-markdown)
  → AST (Abstract Syntax Tree)
  → Transform plugins (optional: LaTeX, custom card embeds, future extensions)
  → Renderer (AST → React Native components)
  → Memoized component tree
```

Each stage is independently replaceable. To add LaTeX support in the future, add a transform plugin that recognizes `$...$` and `$$...$$` delimiters and produces custom AST nodes, then add a renderer for those nodes (likely using `react-native-math-view`). No existing code changes required.

#### 2.3.6 Performance Requirements

| Requirement | Target |
|---|---|
| Parse time for a 500-word message | <10ms on iPhone 12 and newer |
| Incremental parse during streaming | Parse only the new delta (appended tokens), not the full string. Maintain a running AST that is extended, not rebuilt. |
| Thread | Full markdown parsing (for completed messages) runs off the main JS thread. During streaming, the lightweight regex-based formatter runs on-thread (it is <1ms per invocation). |
| Memoization | Each completed message's rendered component tree is memoized by `messageId + content hash`. Re-renders of the message list do not re-parse markdown for unchanged messages. |
| Crash resilience | If the markdown parser throws on malformed input, catch the error and fall back to rendering the raw string as plain text. Never crash the app due to markdown parsing. |

---

### 2.4 Tool Result Card System

All cards share a common elevated surface style and are rendered as `Pressable` components that navigate to the relevant detail screen on tap.

#### 2.4.0 Common Card Properties

| Property | Value |
|---|---|
| Background | `#FFFFFF` (light) / `#2C2C2E` (dark) |
| Corner radius | 12pt |
| Border | 1pt, `#E5E5EA` (light) / `#38383A` (dark) |
| Shadow | `color: #000000, opacity: 0.08, offset: {x: 0, y: 2}, blur: 8pt` |
| Padding | 14pt all sides |
| Tap feedback | Slight scale reduction (0.98) over 100ms on press-in, return to 1.0 on press-out. Haptic: `selection`. |
| Tap action | Navigate to the relevant detail screen via React Navigation `push`. |
| Type icon | 20pt icon in top-left corner of card, color-coded by type (see per-card specs). 6pt right margin from card title. |

#### 2.4.1 Contact Card

```
┌─────────────────────────────────────────────────┐
│ 👤 CONTACT                                      │
│                                                  │
│  ┌────┐  Marcus Johnson                         │
│  │ MJ │  📱 (555) 123-4567                      │
│  └────┘  ✉️  marcus@email.com                    │
│          🟢 Last engaged 2 days ago              │
│                                                  │
│  [Call]  [Text]  [Email]               ▸        │
└─────────────────────────────────────────────────┘
```

| Element | Specification |
|---|---|
| Type badge | "CONTACT" in 10pt SF Pro bold, `#007AFF`, uppercase, letter-spacing 0.5pt. Person icon (`person.fill` SF Symbol) in `#007AFF`. |
| Avatar | 44pt diameter circle. If contact has a photo, show photo. If not, show initials on a deterministic color background (hash of name → color from a predefined palette of 12 colors). |
| Name | 17pt SF Pro semibold, `#000000` / `#FFFFFF`. Single line, truncate with ellipsis. |
| Phone | 14pt SF Pro regular, `#3C3C43` / `#EBEBF5`. Phone icon 14pt, `#8E8E93`. Tap the phone number directly to initiate a call (via `Linking.openURL('tel:...')`). |
| Email | 14pt SF Pro regular, `#3C3C43` / `#EBEBF5`. Mail icon 14pt, `#8E8E93`. Tap to compose email. |
| Engagement dot | Green (`#34C759`) if engaged within 7 days. Yellow (`#FF9500`) if 7-30 days. Red (`#FF3B30`) if >30 days. Dot is 8pt diameter, inline with engagement text. Text: "Last engaged X days ago" in 13pt, `#8E8E93`. |
| Quick action buttons | Horizontal row. Each button: 32pt height, rounded rect (8pt radius), `#F2F2F7` background, 13pt SF Pro medium text. Tap triggers the action directly (call, text, email) without leaving chat. |
| Chevron | Right-pointing chevron, `#C7C7CC`, 14pt, right-aligned vertically centered. Indicates tappability for full detail navigation. |

#### 2.4.2 Property Card

```
┌─────────────────────────────────────────────────┐
│ ┌─────────────────────────────────────────────┐ │
│ │                                             │ │
│ │        [Street View / Property Image]       │ │
│ │                                             │ │
│ └─────────────────────────────────────────────┘ │
│ 🏠 1234 Oak Avenue, Austin, TX 78701            │
│                                                  │
│ $875,000           3 bd  │  2 ba  │  2,100 sqft │
│                                                  │
│ Listed 5 days ago                     Active  🟢│
└─────────────────────────────────────────────────┘
```

| Element | Specification |
|---|---|
| Image | Full card width minus 14pt padding each side. 160pt height. Corner radius 8pt. `resizeMode: cover`. Source: Google Street View API thumbnail or MLS photo. Loading: blurhash placeholder (computed server-side, sent with property data as a ~30-character string). Progressive JPEG rendering. If no image available: gradient placeholder `#E5E5EA` → `#F2F2F7` with a house icon centered. |
| Address | 16pt SF Pro semibold, `#000000` / `#FFFFFF`. Max 2 lines, truncate with ellipsis. House icon (`house.fill` SF Symbol) 16pt, `#007AFF`. |
| Price | 20pt SF Pro bold, `#000000` / `#FFFFFF`. Formatted with locale-aware currency (`$875,000`). |
| Metrics row | Three items separated by 1pt vertical dividers. Each: value in 15pt SF Pro semibold + label in 13pt SF Pro regular `#8E8E93`. E.g., "3 bd", "2 ba", "2,100 sqft". |
| Status | "Listed X days ago" in 13pt `#8E8E93`. Status badge: "Active" in 11pt SF Pro bold, `#34C759`, with green dot. Or "Pending" in `#FF9500`, "Sold" in `#FF3B30`. Badge has a subtle background tint matching the color at 10% opacity, 4pt horizontal padding, 4pt corner radius. |

#### 2.4.3 Content Preview Card

For AI-generated marketing content (Instagram captions, email drafts, listing descriptions).

```
┌─────────────────────────────────────────────────┐
│ ✏️ GENERATED CONTENT                              │
│                                                  │
│ Instagram Caption                                │
│ ─────────────────────────────────────            │
│ "🏡 Just listed! This stunning 3-bed             │
│ craftsman in East Austin features..."            │
│                                                  │
│                                        [Copy 📋]│
└─────────────────────────────────────────────────┘
```

| Element | Specification |
|---|---|
| Type badge | "GENERATED CONTENT" in 10pt SF Pro bold, `#AF52DE` (purple), uppercase. Pencil icon. |
| Content type label | "Instagram Caption" or "Email Draft" or "Listing Description" in 15pt SF Pro semibold. |
| Divider | 1pt `#E5E5EA`, full width minus 28pt, 8pt vertical margin. |
| Content preview | 14pt SF Pro regular, `#3C3C43`. Max 6 lines visible. If content exceeds 6 lines, truncate with "... Show more" in `#007AFF`. Tap "Show more" expands inline with animation. |
| Copy button | Bottom-right corner. "Copy" + clipboard icon. 13pt SF Pro medium, `#007AFF`. On tap: copies full content to clipboard, button text changes to "Copied!" with checkmark for 2s, haptic `notificationSuccess`. |
| Content is markdown-capable | The generated content may include emoji, hashtags, line breaks. Render faithfully. |

#### 2.4.4 Market Analysis Card

```
┌─────────────────────────────────────────────────┐
│ 📊 MARKET ANALYSIS — 78701                       │
│                                                  │
│  Median Price     Avg DOM      Inventory         │
│  $542,000         28 days      156 listings      │
│  ↑ 4.2%           ↓ 3 days     ↑ 12%            │
│                                                  │
│ ┌─────────────────────────────────────────────┐ │
│ │  ╱\     Price Trend (6mo)                   │ │
│ │ ╱  \  ╱\                                    │ │
│ │╱    \/  \╱                                  │ │
│ └─────────────────────────────────────────────┘ │
│                                                  │
│ Seller's Market · 4.2 months supply              │
└─────────────────────────────────────────────────┘
```

| Element | Specification |
|---|---|
| Type badge | "MARKET ANALYSIS" in 10pt SF Pro bold, `#FF9500` (orange), uppercase + zip code or area name. Chart icon. |
| Key metrics | 3-column layout. Each column: value in 18pt SF Pro bold, label in 12pt `#8E8E93`, trend in 13pt. Trend colors: up = `#34C759` with ↑ arrow, down = `#FF3B30` with ↓ arrow, flat = `#8E8E93` with → arrow. |
| Mini chart | 120pt height. Rendered using `react-native-svg` (not a WebView). Simple line chart with gradient fill below the line (brand blue to transparent). 6 data points (months). No axis labels (too small) — the chart is illustrative, not analytical. Tap the chart to navigate to a full market detail screen. |
| Summary line | Bottom of card. "Seller's Market" / "Buyer's Market" / "Balanced" in 14pt SF Pro medium, with supply months. |

#### 2.4.5 Task / Follow-up Card

```
┌─────────────────────────────────────────────────┐
│ ☑️ FOLLOW-UP                                     │
│                                                  │
│ Call Marcus Johnson re: 1234 Oak Ave showing     │
│ 📅 Tomorrow at 2:00 PM                           │
│                                                  │
│ [Mark Complete ✓]                                │
└─────────────────────────────────────────────────┘
```

| Element | Specification |
|---|---|
| Type badge | "FOLLOW-UP" or "TASK" in 10pt SF Pro bold, `#34C759` (green), uppercase. Checkmark icon. |
| Task description | 15pt SF Pro regular, `#000000` / `#FFFFFF`. Max 3 lines. |
| Due date | 14pt SF Pro medium, `#8E8E93`. Calendar icon 14pt. If overdue: text in `#FF3B30`, icon in `#FF3B30`. If today: text in `#FF9500`. If future: default `#8E8E93`. |
| Contact reference | If the task references a contact, the contact name is rendered as a tappable link (`#007AFF`). Tap navigates to Contact Detail. |
| Mark Complete button | Full width, 40pt height, `#34C759` background, white text "Mark Complete" in 14pt SF Pro semibold, 10pt corner radius. On tap: haptic `notificationSuccess`, button collapses to a centered checkmark with "Done" text over 300ms, card background gets a subtle green tint (`#F0FFF4`), button becomes non-interactive. The completion is sent to the API: `POST /api/tasks/{taskId}/complete`. |

#### 2.4.6 Card Loading State (Skeleton)

While a tool call is executing and the result has not yet arrived:

```
┌─────────────────────────────────────────────────┐
│ ████████████                                     │  ← shimmer bar 1 (type badge)
│                                                  │
│ ████████████████████████████                     │  ← shimmer bar 2 (title)
│ ██████████████████                               │  ← shimmer bar 3 (subtitle)
│                                                  │
│ ███████████████████████████████████████          │  ← shimmer bar 4 (content)
└─────────────────────────────────────────────────┘
```

| Property | Value |
|---|---|
| Dimensions | Match the expected card type. If the card type is unknown (generic tool call), use a default 200pt height. |
| Shimmer bars | Rounded rectangles (4pt radius), `#E5E5EA` base color with a gradient highlight (`#F5F5F5`) sweeping left-to-right at 1.5s interval. Use `react-native-reanimated` for the gradient animation to run on the UI thread. |
| Shimmer count | 3-4 bars of varying widths (70%, 50%, 85%, 40% of card width) to suggest the structure of the eventual content. |
| Transition | When the result arrives, the skeleton cross-fades to the real card over 250ms (`opacity: 0 → 1` on real card, `opacity: 1 → 0` on skeleton, simultaneous). |

#### 2.4.7 Card Error State

```
┌─────────────────────────────────────────────────┐
│ ⚠️                                               │
│ Couldn't complete this action                    │
│ There was a problem looking up this contact.     │
│                                                  │
│                    [Retry]                        │
└─────────────────────────────────────────────────┘
```

| Property | Value |
|---|---|
| Background | `#FFF2F2` (light) / `#3A2020` (dark) |
| Icon | Warning triangle, 32pt, `#FF3B30`, centered at top. |
| Title | "Couldn't complete this action" in 15pt SF Pro semibold, `#000000` / `#FFFFFF`. |
| Detail | Contextual error message in 13pt SF Pro regular, `#8E8E93`. E.g., "There was a problem looking up this contact." |
| Retry button | Centered, 36pt height, `#007AFF` text "Retry" in 14pt SF Pro medium. On tap: re-dispatches the original tool call. Skeleton loading state replaces the error card. |

---

### 2.5 Input Bar Design

The input bar is a persistent component anchored to the bottom of the chat screen. It must handle keyboard interaction flawlessly — this is the most interaction-dense area of the entire app.

#### 2.5.1 Layout Structure

```
┌──────────────────────────────────────────────────────────┐
│ [Attachment preview chips — if any attachments selected]  │
├──────────────────────────────────────────────────────────┤
│  [+]  │ Message Skye...                    │ [🎤] [➤]   │
│       │                                    │             │
└──────────────────────────────────────────────────────────┘
│                    Safe area padding                      │
└──────────────────────────────────────────────────────────┘
```

**Component hierarchy:** `KeyboardStickyView` (from `react-native-keyboard-controller`) wrapping the entire input bar ensures it stays pinned above the keyboard at all times, with smooth animated transitions.

#### 2.5.2 Text Input Field

| Property | Value |
|---|---|
| Component | `TextInput` with `multiline={true}` |
| Placeholder | "Message Skye..." in 16pt SF Pro regular, `#C7C7CC` |
| Font | 16pt SF Pro regular, `#000000` / `#FFFFFF` |
| Background | `#F2F2F7` (light) / `#2C2C2E` (dark) |
| Corner radius | 20pt (pill shape when single line) |
| Horizontal padding | 16pt left, 16pt right (additional right padding when send button is visible) |
| Vertical padding | 10pt top, 10pt bottom |
| Min height | 40pt (single line) |
| Growth behavior | Starts at 1 line (40pt). Grows line-by-line as text wraps. Maximum visible height: 6 lines (~132pt). After 6 lines, internal scroll activates. Growth animation: `react-native-reanimated` spring animation (stiffness: 500, damping: 30) on height change. |
| Max characters | 4,000 (API limit). No visible counter until 3,800 characters, then a subtle "200 remaining" label appears above the input bar in 11pt `#FF9500`. At 4,000: input stops accepting characters, label turns red "Character limit reached". |
| Return key | Default return key inserts a newline (multiline input). Send is always via the send button. |
| Autocorrect | Enabled. `autoCorrect={true}`, `spellCheck={true}`. |
| Content type | `textContentType="none"` to prevent password autofill suggestions. |

#### 2.5.3 Send Button

| Property | Value |
|---|---|
| Visibility | Hidden when text input is empty AND no attachments are present. Appears when `text.trim().length > 0 OR attachments.length > 0`. |
| Appearance animation | Scale from 0 → 1 over 150ms (spring: stiffness 600, damping 15). Position: right side of the input field, vertically centered with the last line of text. |
| Icon | Up-arrow in filled circle (SF Symbol: `arrow.up.circle.fill`). 32pt diameter. Color: `#007AFF`. |
| Tap behavior | (1) Haptic `impactLight`. (2) Dispatch message to send queue. (3) Clear input field. (4) Input field height animates back to single line. (5) Scroll to bottom of message list. |
| Disabled state | During active AI streaming: send button is replaced with a "Stop" button (square icon in circle, `#FF3B30`). Tapping stop aborts the current SSE stream. After abort, send button reappears if text is present. |
| Tap target | Minimum 44x44pt, even though the visible icon is 32pt. |

#### 2.5.4 Attachment Button

| Property | Value |
|---|---|
| Position | Left side of the input field, vertically centered. |
| Icon | Plus icon in circle (SF Symbol: `plus.circle.fill`), 28pt, `#007AFF`. |
| Tap behavior | Haptic `selection`. Opens an iOS action sheet (`ActionSheetIOS`) with options: "Take Photo" (camera), "Choose from Library" (photo picker), "Cancel". |
| Camera | Opens the device camera via `react-native-image-picker` with `mediaType: 'photo'`. Max resolution: 2048px on longest edge (downscaled before upload to control payload size). |
| Photo Library | Opens the iOS photo picker (PHPickerViewController via `react-native-image-picker`) with `selectionLimit: 3`. |
| Attachment preview | Selected images appear as chips above the input field: 56pt tall, 56pt wide, 8pt corner radius, with a small "X" button (20pt circle, top-right corner) to remove. Multiple chips scroll horizontally. |
| Upload behavior | Attachments are uploaded immediately on selection to a presigned S3 URL (via `POST /api/upload`). A circular progress indicator overlays each chip during upload. If upload fails, chip shows a red exclamation badge — tap to retry upload. |

#### 2.5.5 Voice Input Button

| Property | Value |
|---|---|
| Position | To the left of the send button (when send button is hidden, the microphone occupies the right position). |
| Icon | Microphone (SF Symbol: `mic.fill`), 24pt, `#007AFF`. |
| Tap behavior | Activates iOS system dictation via `TextInput`'s built-in dictation support (no custom speech recognition). The keyboard switches to dictation mode. |
| Visibility | Hidden when the send button is visible (text is present). Appears only when the input field is empty. This avoids cluttering the input bar. |
| Fallback | If dictation is disabled in iOS settings, the button is hidden entirely. Check `Settings.isAvailable('dictation')` (not a real API — instead, simply always render the button; iOS handles the unavailable state gracefully by showing a system alert). |

#### 2.5.6 Safe Area and Keyboard Handling

| Behavior | Implementation |
|---|---|
| Safe area (no keyboard) | Input bar bottom padding includes the device safe area inset (home indicator area on Face ID devices, ~34pt). Use `useSafeAreaInsets()` from `react-native-safe-area-context`. |
| Keyboard open | `react-native-keyboard-controller`'s `KeyboardStickyView` handles positioning. The input bar animates upward in sync with the keyboard animation (matched duration and curve). No jumping, no gap. |
| Keyboard dismiss (tap) | Tapping the message list area above the keyboard dismisses the keyboard. Implemented via `keyboardDismissMode="interactive"` on the FlashList's underlying ScrollView. |
| Keyboard dismiss (drag) | Interactive dismissal: dragging down on the message list pulls the keyboard down. The input bar tracks the keyboard position frame-by-frame (via `react-native-keyboard-controller`'s animated values). If the user drags <50% of keyboard height and releases, keyboard snaps back open. If >50%, keyboard dismisses fully. |
| Keyboard type switch | When switching between text and emoji keyboards, the input bar smoothly tracks the height change (no jump). `react-native-keyboard-controller` provides continuous height values via `useKeyboardAnimation`. |
| Rotation | On device rotation (if supported), the input bar width adjusts. Text input reflows. Keyboard re-renders at new width. Safe area insets update. All transitions animated. |

---

### 2.6 Chat UX Behaviors

#### 2.6.1 Auto-Scroll Logic

The auto-scroll behavior must match user expectations set by iMessage and WhatsApp: new messages scroll into view automatically, unless the user has deliberately scrolled up to read earlier messages.

**Algorithm:**

```typescript
const NEAR_BOTTOM_THRESHOLD = 100;   // pt from bottom
const SHOW_PILL_THRESHOLD = 200;     // pt from bottom

function onNewMessage(message: Message) {
  const distanceFromBottom = getDistanceFromBottom(); // contentSize.height - offset - viewportHeight

  if (distanceFromBottom <= NEAR_BOTTOM_THRESHOLD) {
    // User is at/near bottom → auto-scroll to new message
    scrollToEnd({ animated: true, duration: 250 });
  } else {
    // User has scrolled up → don't scroll, show pill
    showNewMessagePill(message);
  }
}
```

| Component | Specification |
|---|---|
| Auto-scroll animation | `scrollToEnd` with duration 250ms, ease-out curve. Smooth, not jarring. |
| "New message" pill | Floating pill at bottom-center of message list, 24pt above the input bar. Design: `#007AFF` background, white text "New message ↓" in 13pt SF Pro semibold, 20pt height, 16pt horizontal padding, fully rounded corners (pill shape). Shadow: `opacity 0.15, offset {0, 2}, blur 4`. |
| Pill behavior | Tap scrolls to bottom (animated, 300ms). Pill dismisses when the user scrolls to within `NEAR_BOTTOM_THRESHOLD` of the bottom (regardless of how they get there). Pill auto-dismisses after 5s if not tapped (fade-out 200ms). |
| Scroll-to-bottom button | A separate button from the pill. Appears when the user is >200pt from the bottom AND no new-message pill is active. Circular, 36pt diameter, `#FFFFFF` background, subtle shadow, down-arrow icon `#007AFF`. Positioned bottom-right of the message list, 16pt from right edge, 8pt above input bar. Tap scrolls to bottom. |
| During streaming | If user is near bottom, each token batch triggers a micro-scroll to keep the streaming content in view. Use `scrollToEnd({ animated: false })` (instant, no animation) to avoid jarring repeated animations. |

#### 2.6.2 Typing Indicator

Shown after the user sends a message and before the first SSE token arrives.

```
┌──────────────┐
│  ●  ●  ●     │   ← animated dots inside an AI-style bubble
└──────────────┘
```

| Property | Value |
|---|---|
| Trigger | Appears when a user message is sent and the SSE state transitions to `CONNECTING`. Removed when the state transitions to `STREAMING` (first token received). |
| Position | Rendered as the last item in the message list (below the user's message), left-aligned like an AI message. |
| Bubble | Same styling as AI text message bubble, but smaller: 52pt wide, 36pt tall. |
| Dots | Three circles, 8pt diameter each, `#8E8E93`, spaced 6pt apart, centered vertically and horizontally in the bubble. |
| Animation | Sequential bounce: each dot translates Y from 0 → -6pt → 0 over 400ms, staggered 150ms apart. Loop continuously. Use `react-native-reanimated` `withSequence` + `withDelay` for UI-thread animation. |
| Timeout | If typing indicator is visible for >15s (no first token), append a subtitle below the bubble: "This is taking longer than usual..." in 12pt `#8E8E93`. |
| Removal | On first token, the typing indicator cross-fades out (opacity 1 → 0, 150ms) while the AI message bubble fades in at the same position. No layout jump. |

#### 2.6.3 Timestamp Display

| Behavior | Specification |
|---|---|
| Inline timestamps | Automatically inserted between messages when the time gap exceeds **5 minutes**. Rendered as `system` messages with subtype `date_separator`. Format: "2:34 PM" (same day), "Yesterday 2:34 PM", or "Mon, Jan 15, 2:34 PM" (older than 2 days). |
| Per-message timestamp | Tap any message bubble to reveal its exact timestamp. Timestamp appears above the bubble in 11pt `#8E8E93`, with a slide-down animation (translateY: -8 → 0, opacity: 0 → 1, 200ms). Tapping again or tapping elsewhere hides it. |
| Date separators | Full date separators ("Today", "Yesterday", "January 15, 2026") are inserted between messages from different calendar days. Rendered per Section 2.2.7. |

#### 2.6.4 Message Sending States

| State | Visual | Transition |
|---|---|---|
| **Composing** | Text in input field. Not yet in message list. | User taps send → `sending` |
| **Sending** | Message appears in list immediately (optimistic). Bubble at 60% opacity. No timestamp. Subtle upward slide-in animation. | API acknowledges → `sent`. API fails → `failed`. |
| **Sent** | Full opacity. Timestamp available on tap. | — |
| **Failed** | Bubble turns `#FF3B30` tinted. Below bubble: "Not sent. Tap to retry." Tap the error text or the bubble to retry. | Retry → `sending`. User deletes → removed from list. |

**Optimistic insertion timing:** The user message appears in the list within the same frame as the send button tap (before the network request is even initiated). This creates the perception of instant responsiveness.

#### 2.6.5 Long-Press Message Actions

Long-pressing (500ms) on a message triggers a context menu with haptic feedback.

| Message Type | Available Actions |
|---|---|
| User text message | **Copy Text** — Copies message content to clipboard. **Delete** — "Delete Message" with confirmation alert ("Are you sure? This cannot be undone."). On confirm, message is removed from local list and `DELETE /api/chat/messages/{id}` is called. **Share** — Opens iOS share sheet with message text. |
| AI text message | **Copy Text** — Copies the full markdown source (not rendered text) to clipboard. **Share** — Opens iOS share sheet. **Regenerate** — Re-sends the previous user message to get a new AI response. The current AI response is replaced. |
| Tool result card | **Copy** — Copies the primary content (contact name, address, generated text) depending on card type. Other actions per card type (e.g., "Share Property" for property cards). |
| System message | No long-press action. |

**Implementation:** Use `react-native-context-menu-view` for native iOS context menu presentation (UIContextMenuInteraction). The menu appears with the standard iOS blur background and haptic feedback (handled natively). If the library is unavailable, fall back to a custom bottom sheet action list.

#### 2.6.6 Pull-to-Load Earlier History

| Behavior | Specification |
|---|---|
| Trigger | User scrolls to the top of the message list. `onStartReached` (or equivalent) fires. |
| Loading indicator | A `RefreshControl`-style spinner appears at the top of the list while loading. Alternative: 3 skeleton message bubbles at the top (left-aligned for AI, right-aligned for user, in a plausible pattern). |
| API call | `GET /api/chat/history?sessionId={id}&cursor={oldestMessageId}&limit=50` |
| Scroll position | After new messages are prepended, the scroll position is maintained so the user continues reading from where they were. Use FlashList's `maintainVisibleContentPosition` (enabled by default in v2). |
| End of history | When the API returns fewer than 50 messages (or an empty array), stop requesting. Show a subtle "Beginning of conversation" label at the top. |
| Performance | Prepending 50 messages should not cause a visible stutter. FlashList's cell recycling handles this. No full re-render of existing messages. |

#### 2.6.7 Empty Chat State

When the chat has no messages (new session or first launch):

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│                                                          │
│                     [Skye Logo]                          │
│                     64pt, brand blue                     │
│                                                          │
│              Hi! I'm Skye, your AI                       │
│              real estate assistant.                       │
│                                                          │
│              How can I help you today?                    │
│                                                          │
│    ┌──────────────────────────────────────────┐          │
│    │ 🔍  "Who should I follow up with today?" │          │
│    └──────────────────────────────────────────┘          │
│    ┌──────────────────────────────────────────┐          │
│    │ 🏠  "Find listings under $500K in 78704" │          │
│    └──────────────────────────────────────────┘          │
│    ┌──────────────────────────────────────────┐          │
│    │ ✏️  "Draft an open house Instagram post"  │          │
│    └──────────────────────────────────────────┘          │
│    ┌──────────────────────────────────────────┐          │
│    │ 📊  "What's the market like in 78701?"   │          │
│    └──────────────────────────────────────────┘          │
│                                                          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

| Element | Specification |
|---|---|
| Logo | Skye app icon or wordmark, 64pt, centered. |
| Greeting | "Hi! I'm Skye, your AI real estate assistant." in 20pt SF Pro semibold, centered. "How can I help you today?" in 16pt SF Pro regular, `#8E8E93`, centered. 8pt gap between lines. |
| Suggested prompts | 4 tappable cards, vertically stacked, 12pt gap between cards. Each card: full width minus 48pt margins, 48pt height, `#F2F2F7` background (light) / `#2C2C2E` (dark), 12pt corner radius. Left icon (relevant emoji), text in 15pt SF Pro regular, left-aligned with 12pt padding. |
| Prompt tap behavior | Tap inserts the prompt text into the input field AND immediately sends it (single tap = full send, not just insertion). The empty state disappears as the first message appears. |
| Animation | Prompts stagger-fade-in on mount: each card fades and slides up (translateY: 10 → 0, opacity: 0 → 1, 200ms) with 50ms stagger between cards. |
| Visibility | The empty state disappears permanently once the first message exists in the session. It does not reappear if all messages are deleted. |

#### 2.6.8 Chat Session Management

| Concept | Specification |
|---|---|
| Session | A chat session is a single continuous conversation with the AI. It has a unique `sessionId` (UUID). All messages within a session share this ID. |
| Session persistence | Sessions are persisted server-side. The client stores the `currentSessionId` in async storage. On app launch, the most recent session is loaded. |
| New Chat | The user can start a new chat session from the navigation header (icon: `square.and.pencil` SF Symbol, top-right). Tap creates a new `sessionId`, clears the message list, and shows the empty state. The previous session remains accessible from a session list/drawer. |
| Session list | Accessible from the navigation header (icon: hamburger/sidebar or `list.bullet`). Shows previous sessions with: first user message as title (truncated to 40 chars), last message timestamp, AI's first response as subtitle (truncated to 60 chars). Tap loads that session. |
| Session title | Auto-generated by the AI after the first exchange. The server returns a `sessionTitle` field on the first AI response. E.g., "Follow-up with Marcus Johnson". Updated if the conversation topic shifts significantly (server-driven). |
| Session deletion | Swipe-to-delete on session list items. Confirmation alert. Deletes all messages via `DELETE /api/chat/sessions/{sessionId}`. |

---

### 2.7 Performance Specifications

These are hard targets. The app must meet these benchmarks on an iPhone 12 (A14 Bionic) running iOS 16 or later. If benchmarks are met on iPhone 12, they will be exceeded on newer hardware.

| Metric | Target | Measurement Method |
|---|---|---|
| **First token to screen** | <100ms from the moment the first SSE `data:` event is received by the client to the moment the token is rendered in the message list. | Instrument with `performance.now()` timestamps at SSE `onmessage` and `onLayout` of the streaming message component. |
| **Time to first byte (TTFB)** | <500ms from send button tap to first SSE event. This is primarily server-side latency (AI model inference startup). | Timestamp at send dispatch vs. first `onmessage`. Monitored via analytics. |
| **Message list scroll FPS** | 60fps sustained with 200+ messages rendered (via virtualization). 0 blank cells during normal-speed scrolling. | Measure with Flipper Performance plugin or Xcode Instruments (Core Animation FPS). |
| **Input keystroke latency** | <16ms (1 frame) from physical key press to character rendered. | Profile with Instruments. Ensure no JS-thread blocking during typing. |
| **Memory — 50 messages** | <80MB total app memory. | Xcode Memory Graph. |
| **Memory — 200 messages** | <120MB total app memory. | Xcode Memory Graph. |
| **Memory — 500 messages** | <150MB total app memory. This requires aggressive virtualization and image cache management. | Xcode Memory Graph. |
| **App launch to chat ready** | <1.5s from app icon tap to chat screen rendered with the most recent 20 messages visible and input bar interactive. | Profile with Instruments or `react-native-startup-time`. |
| **Image load (card thumbnails)** | Blurhash placeholder visible <50ms. Full image loaded <500ms on WiFi, <1500ms on cellular. | Instrument `onLoadStart` to `onLoadEnd` on FastImage. |
| **Markdown parse (completed message)** | <10ms for a 500-word message. <50ms for a 2,000-word message. | Benchmark in isolation with `performance.now()`. |

#### 2.7.1 Virtualization Configuration

```typescript
<FlashList
  data={messages}
  renderItem={renderMessage}
  estimatedItemSize={80}                  // Average message height estimate
  getItemType={(item) => item.type}       // Enable per-type cell recycling
  keyExtractor={(item) => item.id}
  inverted={false}                        // Use startRenderingFromBottom instead
  startRenderingFromBottom={true}
  maintainVisibleContentPosition={{
    minIndexForVisible: 0,
    autoscrollToTopThreshold: 10,
  }}
  drawDistance={5 * screenHeight}          // 5 screens above and below
  overrideItemLayout={(layout, item) => {
    // Provide size hints for common message types to reduce blank cells
    if (item.type === 'system' && item.subtype === 'date_separator') {
      layout.size = 52;
    }
  }}
  keyboardDismissMode="interactive"
  contentContainerStyle={{ paddingTop: 16, paddingBottom: 8 }}
/>
```

#### 2.7.2 Image Loading Strategy

| Stage | Implementation |
|---|---|
| **Placeholder** | Blurhash string (sent from server with every image URL). Decoded to a low-res image (~4x3 pixels) and displayed as the blurred placeholder. Decode time: <5ms. |
| **Thumbnail** | Request a 200px-wide thumbnail via CDN image transformation URL parameter (e.g., `?w=200&q=70`). This loads quickly on any connection. |
| **Full image** | On tap (full-screen view), load the full-resolution image. Progressive JPEG — image renders at increasing quality as bytes arrive. |
| **Cache** | `react-native-fast-image` with disk cache (LRU, 200MB limit) and memory cache (50MB limit, auto-evicted under pressure). Cache key is the URL without query parameters (so different size transformations of the same image share a cache group). |
| **Prefetch** | When a tool result card with an image enters the virtualization buffer (before becoming visible), prefetch the thumbnail URL. This ensures images are loaded by the time the user scrolls to them. |

---

### 2.8 Error States and Edge Cases

#### 2.8.1 Network Offline During Send

| Behavior | Specification |
|---|---|
| Detection | `@react-native-community/netinfo` reports `isConnected: false` at the time the user taps send. |
| User experience | Message is inserted into the list with `status: 'queued'`. Bubble at 50% opacity with a clock icon (⏳) to the right. Below the bubble: "Will send when connected" in 12pt `#8E8E93`. |
| Queue | Messages are queued in an ordered array persisted to async storage. Maximum queue depth: 5 messages. If the user tries to send a 6th, show an alert: "You have unsent messages waiting for a connection." |
| Reconnection | When `NetInfo` reports `isConnected: true`, the queue is drained sequentially (one at a time, awaiting each response before sending the next). Each message transitions from `queued` → `sending` → `sent` as it processes. |
| Queue persistence | The queue survives app restart. On relaunch, check connectivity and drain if online. |

#### 2.8.2 API Returns 401 During Chat

| Behavior | Specification |
|---|---|
| Detection | SSE connection or any API call returns HTTP 401 or 403. |
| Handling | **Silently** attempt token refresh: `POST /api/auth/refresh` with the stored refresh token. This is invisible to the user. |
| Success | New access token is stored. The failed request is retried automatically with the new token. The user never sees an error. |
| Failure | If refresh fails (refresh token also expired), redirect to the login screen with a message: "Your session has expired. Please sign in again." Preserve the current chat state so it's available after re-authentication. |
| Rate limit | Maximum 2 silent refresh attempts per minute to prevent infinite loops. |

#### 2.8.3 API Returns 500 During Chat

| Behavior | Specification |
|---|---|
| Display | An inline error message appears below the user's message (where the AI response would be). Styled as a system message with red tint: "Something went wrong on our end." with a "Retry" button. |
| Retry | Tap "Retry" re-sends the last user message to the API. Maximum 2 automatic retries with 2s delay between them. After 2 failures, show: "We're having trouble right now. Please try again later." with no automatic retry — only a manual "Try again" button. |
| Logging | All 500 errors are logged to the analytics service with: endpoint, timestamp, request payload hash (no PII), response body if available, device info. |

#### 2.8.4 SSE Stream Malformed

| Scenario | Handling |
|---|---|
| Invalid JSON in `data:` field | Skip the malformed event. Log the raw event to analytics. Continue processing subsequent events. The token buffer is unaffected — only valid tokens are appended. |
| Incomplete SSE event (no `\n\n` terminator) | The line accumulator holds the partial data until the next chunk arrives and completes it. If the connection closes with an unterminated event, discard the partial event. |
| Unexpected event type | Ignore unknown `event:` types. Forward-compatible — new event types can be added server-side without breaking the client. |
| Empty `data:` field | Skip. No token appended. |
| Stream delivers valid JSON but unexpected schema | Validate each event against expected shape. If validation fails, skip and log. Do not crash. |

#### 2.8.5 Tool Call Takes >15 Seconds

| Timing | User Experience |
|---|---|
| 0-5s | Skeleton card with standard shimmer animation. Tool call status line shows "Looking up contact..." (spinner). |
| 5-15s | No change. This is within normal range for some operations (e.g., market analysis with heavy computation). |
| 15s+ | Text appears below the skeleton card: "This is taking longer than usual..." in 13pt `#8E8E93`. A "Cancel" link appears next to it. |
| 30s+ | Text updates to: "Still working on it..." with the "Cancel" link still available. |
| Cancel behavior | Tapping "Cancel" sends an abort signal to the server (via `AbortController` on the SSE connection). The tool call message transitions to `status: 'failed'` with an error card: "Action was cancelled." No retry button (user initiated the cancellation). The AI may send a follow-up text acknowledging the cancellation. |
| Server timeout (60s) | If the server itself times out, it sends an error event via SSE. Client renders the error card (Section 2.4.7). |

#### 2.8.6 Empty Response from AI

| Scenario | Handling |
|---|---|
| SSE stream opens, sends `done` event with 0 tokens | Display a system-style message in the AI position: "I didn't have a response for that. Try rephrasing your question." in 14pt SF Pro regular, `#8E8E93`. Italic styling. No bubble — renders as inline text, left-aligned. |
| SSE stream delivers only whitespace tokens | Same as empty response — trim and check. If the trimmed concatenation is empty, treat as empty response. |
| AI response is only a tool call with no text | This is valid — the tool result card is the response. No empty message indicator needed. |

#### 2.8.7 Extremely Long AI Response (10,000+ Tokens)

| Concern | Mitigation |
|---|---|
| Memory | The response string grows in a `useRef`, not in repeated `setState` calls. Only one state update per frame (RAF-batched). The string itself at 10,000 tokens is ~40-60KB — trivial for memory. |
| Markdown parsing | After streaming completes, the full markdown parse of a 10,000-token response may take 50-100ms. Run this parse on a background thread. While parsing, the raw text (with lightweight regex formatting) remains visible — no blank flash. Once parsing completes, the rendered markdown replaces the raw text with a crossfade. |
| Scroll performance | The message bubble will be very tall. FlashList handles this via dynamic item sizing. The bubble is a single list item — it may be 2000+ pt tall. Ensure `estimatedItemSize` is overridden for this item (or FlashList v2's auto-sizing handles it). |
| Display | No "show more" / truncation on AI responses. The user should see the full response. The virtualized list ensures off-screen portions don't consume render resources. |
| Scroll-to-bottom during streaming | If the user is near the bottom, the auto-scroll keeps pace with token delivery. If the response is generating faster than the user can read, the scroll remains at the bottom (most recent content visible). The user can scroll up at any time to read earlier parts — auto-scroll pauses per Section 2.6.1. |

#### 2.8.8 Rapid-Fire User Messages

| Scenario | Handling |
|---|---|
| User sends while AI is streaming | The send button is replaced with a "Stop" button during streaming (Section 2.5.3). The user must either wait for streaming to complete or tap "Stop" first. After stopping, the send button reappears, and they can send their next message. |
| User sends multiple messages quickly (no streaming active) | Messages are queued and sent sequentially. Each message is dispatched only after the previous SSE stream completes. The queue is processed FIFO. All queued messages appear in the chat list immediately (optimistic, `status: 'sending'`). |
| User pastes and sends very rapidly | Debounce is NOT applied to sends — every tap of the send button dispatches. But the sequential queue prevents concurrent SSE streams. |
| Typing while AI streams | Allowed and encouraged. The input field is always editable. Keystrokes must not be affected by streaming render activity (isolated re-render scopes ensure this). |

#### 2.8.9 App Kill During Active Stream

| Scenario | Handling |
|---|---|
| User force-quits the app while AI is streaming | The SSE connection is severed. Partial response is NOT saved (no opportunity to persist). On next launch, the message list loads from the server-side history (`/api/chat/history`). The server should have persisted whatever was generated before the disconnect. If the server persisted a complete response, it appears in full. If the server only persisted a partial, it appears with `status: 'partial'`. |
| iOS terminates the app for memory pressure | Same as force-quit. No cleanup opportunity. Rely on server-side state as source of truth. |
| Crash during streaming | Same handling. The crash reporter logs the crash. On relaunch, the chat loads from server history. |

---

### 2.9 Technology Stack Summary

| Concern | Library / Approach |
|---|---|
| SSE Client | `react-native-sse` with custom exponential backoff wrapper |
| Virtualized List | FlashList v2 (`@shopify/flash-list`) |
| Keyboard Handling | `react-native-keyboard-controller` (`KeyboardStickyView`, `KeyboardGestureArea`) |
| Animations | `react-native-reanimated` (v3+) for all animations — runs on UI thread |
| Markdown Rendering | `@docren/react-native-markdown` (MDAST-based, lighter parser) with custom memoization layer. Fallback: `react-native-markdown-display`. |
| Syntax Highlighting | `react-syntax-highlighter` with native Text component renderer (not WebView) |
| Image Loading | `react-native-fast-image` with blurhash placeholders (`react-native-blurhash`) |
| Haptic Feedback | `react-native-haptic-feedback` (or `expo-haptics` if using Expo) |
| Context Menus | `react-native-context-menu-view` (native UIContextMenuInteraction) |
| Network State | `@react-native-community/netinfo` |
| In-App Browser | `react-native-inappbrowser-rebridge` (SFSafariViewController) |
| Charts (Market Analysis) | `react-native-svg` with custom chart component (not a charting library — keep it minimal) |
| Safe Area | `react-native-safe-area-context` |
| Navigation | React Navigation v7 (native stack) |
| Local Storage | `react-native-mmkv` for synchronous KV (session IDs, queue). `@react-native-async-storage/async-storage` for larger data. |
| Image Picker | `react-native-image-picker` |
| Gesture Handling | `react-native-gesture-handler` (peer dependency of Reanimated and Keyboard Controller) |

---

### 2.10 Accessibility Requirements

The chat screen must be fully usable with VoiceOver enabled.

| Element | Accessibility Specification |
|---|---|
| Message bubbles | `accessibilityRole="text"`. Label: "[Sender] said: [message content]". Hint: "Double tap to reveal timestamp. Long press for options." |
| Tool result cards | `accessibilityRole="button"`. Label: "[Card type]: [summary]". E.g., "Contact card: Marcus Johnson, phone 555-123-4567." Hint: "Double tap to view details." |
| Action confirmation buttons | `accessibilityRole="button"`. Labels: "Confirm: [action]" and "Cancel". |
| Send button | `accessibilityLabel="Send message"`. `accessibilityState={{ disabled: !hasText }}`. |
| Attachment button | `accessibilityLabel="Add attachment"`. Hint: "Opens camera and photo options." |
| Typing indicator | `accessibilityRole="text"`. `accessibilityLabel="Skye is thinking"`. `accessibilityLiveRegion="polite"`. |
| New message pill | `accessibilityRole="button"`. `accessibilityLabel="New message. Tap to scroll to bottom."`. |
| Suggested prompts (empty state) | `accessibilityRole="button"`. Label: "Suggested prompt: [prompt text]". |
| Dynamic content | Streaming messages use `accessibilityLiveRegion="polite"` so VoiceOver announces when the response is complete (not every token — that would be unusable). Announce once when streaming ends: "Skye responded: [first 100 characters]." |
| Reduce Motion | If `AccessibilityInfo.isReduceMotionEnabled`, disable: typing indicator bounce animation (show static dots), message slide-in animations (instant appear), pill animations. Functional behavior is unchanged. |

---

This specification defines every aspect of the Skye chat system with implementation-ready precision. Every visual measurement, timing value, state transition, and error handling path is explicitly defined. A development team should be able to implement this screen from this document alone, without design ambiguity or architectural questions.

---


## Section 4: CRM Screens — People, Contact Detail, Properties

---

### 1. People Tab — Contact List

#### 1.1 FlatList Virtualization Strategy

The People tab renders the full contact list using a single `FlatList` component from React Native. With a target of 500+ contacts at 60fps, the following virtualization parameters are required:

| Parameter | Value | Rationale |
|---|---|---|
| `getItemLayout` | `(data, index) => ({ length: 76, offset: 76 * index, index })` | Fixed-height rows (76pt) eliminate measurement passes. Offset is cumulative. When alphabetical section headers are active, offset calculation must account for section header height (32pt) interspersed at alphabet boundaries. |
| `windowSize` | `11` | Renders 5 viewport-heights above and below the visible area (default 21 is excessive; 11 retains smooth fast-scroll while reducing off-screen node count). |
| `maxToRenderPerBatch` | `15` | Renders 15 items per JS frame tick. At 76pt row height on a ~812pt viewport (~10.7 visible rows), 15 items provides adequate look-ahead without frame drops. |
| `removeClippedSubviews` | `true` | Detaches off-screen native views from the hierarchy. Critical for iOS to reclaim compositor memory beyond the render window. |
| `initialNumToRender` | `15` | Renders enough rows to fill ~1.4 viewport-heights on first mount, preventing visible blank space on initial load. |
| `updateCellsBatchingPeriod` | `50` | 50ms batching interval. Lower than default (50 vs. 100) to keep up with fast scrolling without overloading the bridge. |
| `keyExtractor` | `(item) => item.id` | Stable string ID from API. Never use array index. |

**Section Header Consideration:** When sort mode is `alphabetical`, the list switches to `SectionList`. Section headers are 32pt tall and sticky. `getItemLayout` must account for the cumulative section header offsets. A precomputed offset map is built once on data change:

```typescript
interface LayoutEntry {
  length: number;
  offset: number;
  index: number;
}

function buildSectionLayout(
  sections: SectionListData<Contact>[]
): (sectionIndex: number, itemIndex: number) => LayoutEntry {
  const ITEM_HEIGHT = 76;
  const HEADER_HEIGHT = 32;
  let cumulativeOffset = 0;
  const sectionOffsets: number[] = [];

  for (const section of sections) {
    sectionOffsets.push(cumulativeOffset);
    cumulativeOffset += HEADER_HEIGHT + section.data.length * ITEM_HEIGHT;
  }

  return (sectionIndex, itemIndex) => ({
    length: ITEM_HEIGHT,
    offset: sectionOffsets[sectionIndex] + HEADER_HEIGHT + itemIndex * ITEM_HEIGHT,
    index: itemIndex,
  });
}
```

#### 1.2 Contact List Item Layout

Each contact row is exactly **76pt tall** with the following internal layout:

```
┌─────────────────────────────────────────────────────────────────────┐
│ 16pt padding-left                                                   │
│  ┌──────┐  12pt gap  ┌─────────────────────────────┐   Quick Icons │
│  │Avatar│             │ Name              ●(health) │   📞  💬     │
│  │ 44pt │             │ Company · Last Activity     │              │
│  └──────┘             └─────────────────────────────┘   16pt pad-R │
│ 16pt padding-left                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

**Component breakdown:**

| Element | Spec |
|---|---|
| **Row container** | Height: 76pt. Padding: 16pt horizontal, 16pt vertical. `flexDirection: 'row'`, `alignItems: 'center'`. Background: `#FFFFFF` (light), `#1C1C1E` (dark). Bottom border: 0.5pt `#E5E5EA` (light) / `#38383A` (dark). |
| **Avatar** | 44pt x 44pt circle. If `contact.avatar_url` exists: `<Image>` with `borderRadius: 22`. If null: initials fallback — background color derived from `hashCode(contact.id) % 8` mapping to palette `['#FF3B30','#FF9500','#FFCC00','#34C759','#007AFF','#5856D6','#AF52DE','#FF2D55']`. Initials: first letter of first name + first letter of last name, uppercase, `fontSize: 17`, `fontWeight: '600'`, color `#FFFFFF`. |
| **Name line** | `fontSize: 17`, `fontWeight: '600'`, `color: label` (system). Max 1 line, `numberOfLines: 1`, `ellipsizeMode: 'tail'`. Engagement health dot appears 6pt to the right of the name text, vertically centered with the name baseline. |
| **Subtitle line** | `fontSize: 15`, `fontWeight: '400'`, `color: secondaryLabel` (`#8E8E93`). Format: `"{company} · {lastActivityRelative}"`. If no company: `"{contactType} · {lastActivityRelative}"`. `lastActivityRelative` uses: "Just now", "2h ago", "Yesterday", "3d ago", "2w ago", "Mar 15". Max 1 line, truncated. |
| **Quick action icons** | Two icons in a vertical stack (or horizontal row, spaced 12pt apart), positioned at the trailing edge. Phone icon: `SF Symbol phone.fill`, tint `#34C759`, 20pt. Message icon: `SF Symbol message.fill`, tint `#007AFF`, 20pt. Each wrapped in 36pt x 36pt `TouchableOpacity` hit target. `onPress` triggers the respective action directly (calls `Linking.openURL('tel:')` or `Linking.openURL('sms:')`). Icons hidden if contact lacks phone/email respectively. |

#### 1.3 Engagement Health Dot System

The engagement health dot is a critical at-a-glance indicator. It encodes the recency of last meaningful interaction (call, email, meeting, note — not automated system events).

| Status | Color | Hex | Condition | Animation |
|---|---|---|---|---|
| **Hot** | Red | `#FF3B30` | Last contact within 7 calendar days | Pulsing glow animation (see below) |
| **Warm** | Amber | `#FF9500` | Last contact 8–30 calendar days ago | Static |
| **Cold** | Blue | `#007AFF` | Last contact 31–90 calendar days ago | Static |
| **Inactive** | Gray | `#8E8E93` | Last contact 91+ days ago, or never | Static |

**Dot specifications:**

- **Size:** 8pt diameter circle (`width: 8, height: 8, borderRadius: 4`).
- **Position on list row:** 6pt to the right of the last character of the contact name, vertically centered with the name text baseline. If name is truncated, dot appears 6pt after the ellipsis.
- **Position on detail header:** 8pt to the right of the name, vertically centered.
- **Shadow (all states):** `shadowColor` matches dot color, `shadowOffset: { width: 0, height: 0 }`, `shadowOpacity: 0.4`, `shadowRadius: 2`.

**Hot pulsing animation:**

```typescript
// Uses React Native Reanimated 3 shared values
const pulseScale = useSharedValue(1);
const pulseOpacity = useSharedValue(0.4);

useEffect(() => {
  if (engagementStatus === 'hot') {
    pulseScale.value = withRepeat(
      withSequence(
        withTiming(1.8, { duration: 1000, easing: Easing.out(Easing.ease) }),
        withTiming(1.8, { duration: 0 }) // hold (reset handled by repeat)
      ),
      -1, // infinite
      false // do not reverse — restart from 1
    );
    pulseOpacity.value = withRepeat(
      withSequence(
        withTiming(0, { duration: 1000, easing: Easing.out(Easing.ease) }),
        withTiming(0.4, { duration: 0 })
      ),
      -1,
      false
    );
  }
}, [engagementStatus]);
```

The animation creates a ring that expands from 8pt to ~14.4pt (1.8x) while fading from 40% to 0% opacity over 1000ms, then instantly resets. The core dot remains static at 8pt with full opacity. The pulse ring is rendered as a separate `Animated.View` absolutely positioned behind the dot with the same background color.

**Computation:** The engagement status is computed client-side from `contact.last_activity_date` (ISO 8601 timestamp from the API). The computation runs once when data is loaded/updated and the result is memoized per contact. Never recomputed during scroll.

```typescript
function computeEngagement(lastActivityDate: string | null): EngagementStatus {
  if (!lastActivityDate) return 'inactive';
  const daysSince = differenceInCalendarDays(new Date(), parseISO(lastActivityDate));
  if (daysSince <= 7) return 'hot';
  if (daysSince <= 30) return 'warm';
  if (daysSince <= 90) return 'cold';
  return 'inactive';
}
```

#### 1.4 Search

**Architecture:** The search bar is rendered as a persistent element above the FlatList (not inside it as a header, which would scroll away). It uses `position: 'sticky'` behavior via being a sibling rendered before the FlatList inside the parent container.

| Aspect | Specification |
|---|---|
| **Search bar component** | Height: 52pt (36pt input + 8pt top/bottom padding). Background: `#F2F2F7` (light) / `#2C2C2E` (dark). Corner radius: 10pt. Left icon: magnifying glass `#8E8E93`. Placeholder: "Search contacts...". `fontSize: 17`, `fontWeight: '400'`. Clear button appears when text is non-empty. |
| **Debounce** | 300ms debounce on text input. Uses a `useRef`-based timer that resets on each keystroke. |
| **Search pipeline** | 1. On text change (after debounce): search local SQLite cache first using `LIKE '%query%'` on `first_name`, `last_name`, `company`, `email`, `phone`. 2. Display local results immediately. 3. Simultaneously fire `GET /api/contacts/search?q={query}` to the server. 4. When server results arrive, merge with local results (deduplicate by `contact.id`), preferring server data for any conflicts. 5. Update display. |
| **Result highlighting** | Matching substrings in name, company, email, and phone are wrapped in `<Text style={{ backgroundColor: '#FFFF00' + '40', fontWeight: '700' }}>`. The highlight color is `#FFFF00` at 25% opacity (light mode) or `#FFFF00` at 15% opacity (dark mode). |
| **Empty search state** | If query returns 0 results: centered illustration (magnifying glass with question mark), text "No contacts found", subtext "Try a different search or add a new contact", and a "Add Contact" button. |
| **Cancel behavior** | Tapping the search bar shows a "Cancel" text button to the right (animated slide-in, 200ms). Tapping Cancel clears the query, dismisses the keyboard, and restores the full contact list. The FlatList scroll position is preserved across search/cancel cycles. |
| **Keyboard handling** | `keyboardShouldPersistTaps: 'handled'` on the FlatList. Scrolling the list dismisses the keyboard (`keyboardDismissMode: 'on-drag'`). |

#### 1.5 Filter Chips

**Layout:** A horizontally scrollable `ScrollView` rendered between the search bar and the FlatList (or section list). Height: 48pt (including 8pt top + 8pt bottom padding). `showsHorizontalScrollIndicator: false`. Content inset: 16pt on both sides.

**Chip design:**

| Property | Value |
|---|---|
| Height | 32pt |
| Padding | 8pt horizontal, 0pt vertical |
| Border radius | 16pt (fully rounded pill) |
| Background (unselected) | `#F2F2F7` (light) / `#2C2C2E` (dark) |
| Background (selected) | `#007AFF` |
| Text color (unselected) | `#3C3C43` (light) / `#EBEBF5` (dark) |
| Text color (selected) | `#FFFFFF` |
| Font | `fontSize: 14`, `fontWeight: '500'` |
| Spacing between chips | 8pt |
| Press animation | Scale to 0.95 over 100ms (Reanimated `withSpring`) |

**Filter categories and their chip options:**

1. **Status:** Active, Inactive. Single-select within category.
2. **Type:** Buyer, Seller, Investor, Vendor. Multi-select. Each selection is an independent chip.
3. **Tags:** Dynamically loaded from `GET /api/crm/tags`. Multi-select. Rendered after static filters. If more than 5 tags, show first 5 + a "More..." chip that opens a bottom sheet with full tag list.
4. **Engagement:** Hot, Warm, Cold, Inactive. Multi-select. Each chip shows its respective color dot (8pt) to the left of the label text, inset by 4pt.
5. **Last Contact:** Today, This Week, This Month, Older. Single-select within category.

**Multi-select behavior:** Tapping a chip toggles it. Multiple chips across different categories combine with AND logic. Multiple chips within the same category combine with OR logic (e.g., selecting "Buyer" AND "Seller" returns contacts who are either). Active filter count badge: if filters are active, a small red badge (16pt circle) appears at the top-right corner of a leading "Filter" icon chip showing the count of active filters.

**Filter application:** Filters are applied client-side against the cached contact list. No server round-trip is needed for filtering. Filter state is persisted to `AsyncStorage` under key `@skye/contact_filters` so filters survive app restart.

#### 1.6 Sort Options

**Trigger:** A sort button in the top-right of the navigation bar (`SF Symbol arrow.up.arrow.down`, 22pt, tint `#007AFF`). Tapping opens an action sheet (`ActionSheetIOS.showActionSheetWithOptions`) with four options:

| Sort Option | Implementation | Default Direction |
|---|---|---|
| **Alphabetical** | `contact.last_name.localeCompare()` then `first_name` | A → Z (tap again to reverse) |
| **Last Contacted** | `contact.last_activity_date` descending | Most recent first |
| **Engagement Score** | Numeric sort by `contact.engagement_score` (API field, 0–100 integer) | Highest first |
| **Date Added** | `contact.created_at` descending | Newest first |

**Section headers:** Only active when sort is `Alphabetical`. Section headers display a single uppercase letter, height 32pt, background `#F2F2F7` (light) / `#1C1C1E` (dark), text `fontSize: 13`, `fontWeight: '600'`, `color: secondaryLabel`, left-padded 16pt. Headers are sticky (`stickySectionHeadersEnabled: true`, default on iOS).

**Current sort indicator:** The active sort option shows a checkmark in the action sheet. The sort button in the nav bar subtly changes: a small label beneath it reads the current sort name in `fontSize: 10`, `color: secondaryLabel`.

#### 1.7 Swipe Actions

Implemented via `react-native-gesture-handler` `Swipeable` component wrapping each contact row. Swipe physics must match iOS system behavior.

**Left swipe (trailing actions — revealed on left swipe, i.e., finger moves left):**

| Action | Color | Icon | Width | Behavior |
|---|---|---|---|---|
| **Call** | `#34C759` (system green) | `SF Symbol phone.fill`, white, 22pt | 80pt | Opens phone dialer with primary phone number. If no phone, show toast "No phone number". |
| **Text** | `#007AFF` (system blue) | `SF Symbol message.fill`, white, 22pt | 80pt | Opens SMS compose with primary phone number. |

**Right swipe (leading actions — revealed on right swipe, i.e., finger moves right):**

| Action | Color | Icon | Width | Behavior |
|---|---|---|---|---|
| **Snooze** | `#FF9500` (system orange) | `SF Symbol moon.fill`, white, 22pt | 80pt | Opens a snooze picker bottom sheet: 1 hour, 3 hours, Tomorrow 9am, Next Week Monday 9am, Custom date/time. Snoozing hides the contact from default list until snooze expires. |
| **Archive** | `#FF3B30` (system red) | `SF Symbol archivebox.fill`, white, 22pt | 80pt | Prompts confirmation: "Archive {name}? They won't appear in your active contacts." Confirm archives (soft delete). |

**Swipe physics:**

- **Activation threshold:** 40pt of horizontal translation before the swipe is "captured" (prevents accidental activation during vertical scroll).
- **Velocity-based full reveal:** If swipe velocity exceeds 500pt/s, the actions fully reveal with a spring animation (`damping: 15, stiffness: 150`).
- **Haptic feedback:** `Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium)` fires when swipe crosses the first action's full-reveal threshold (80pt). A second `Heavy` impact fires at full-row-width for destructive actions.
- **Snap points:** Actions snap open at their combined width (160pt for the two-action side) or snap closed. No partial states.
- **Auto-close:** Swiping one row auto-closes any previously open row. Scrolling the list auto-closes all open rows.
- **Overshoot behavior:** If swiped beyond full reveal, the last action's background extends to fill the overshoot area (the "full-swipe" pattern). Full swipe past 75% of row width on the trailing side triggers Call directly.

#### 1.8 Pull-to-Refresh

| Aspect | Specification |
|---|---|
| **Trigger** | Standard `RefreshControl` attached to the FlatList/SectionList. |
| **Pull threshold** | 80pt of overscroll before refresh triggers. |
| **Custom animation** | Replace the default spinner with a custom Skye-branded animation: a small Skye logo (24pt) that rotates 360 degrees per 1000ms while pulling, then transitions to a checkmark on completion. Implemented via Reanimated `useAnimatedStyle` driving `transform: [{ rotate }]` from a shared value. |
| **API call** | `GET /api/crm/contacts?updatedSince={lastSyncTimestamp}`. If server returns full list, replace cache. If incremental, merge. |
| **Duration** | Minimum display time: 600ms (prevents jarring flash if server responds instantly). Maximum: 10000ms timeout, then show error toast and dismiss. |
| **Haptic** | `Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light)` on refresh trigger. |
| **Optimistic UI** | The list does not flash/reorder during refresh. New data is diffed against current state: new contacts are inserted, updated contacts are patched in-place, deleted contacts are removed with `LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut)`. |

#### 1.9 Contact Count Badge

- **Position:** Between the filter chips and the FlatList, left-aligned with 16pt left padding.
- **Height:** 28pt (including 4pt top + 4pt bottom padding).
- **Text:** `"{count} Contacts"` — `fontSize: 13`, `fontWeight: '400'`, `color: secondaryLabel`.
- **Dynamic updates:** Count reflects the current filtered/searched subset. If filters are active: `"{filteredCount} of {totalCount} Contacts"`. If search is active: `"{resultCount} results"`.
- **Animation:** Count changes animate with a vertical slide transition (old number slides up and fades out, new number slides up from below and fades in, 200ms, `Easing.out(Easing.ease)`).

#### 1.10 Floating Action Button

- **Size:** 56pt x 56pt circle.
- **Position:** Bottom-right corner, `right: 16pt`, `bottom: tabBarHeight + 16pt` (where `tabBarHeight` is computed from `useSafeAreaInsets().bottom + 49pt`).
- **Background:** `#007AFF`.
- **Icon:** `SF Symbol plus`, white, 24pt, centered.
- **Shadow:** `shadowColor: '#000000'`, `shadowOffset: { width: 0, height: 4 }`, `shadowOpacity: 0.3`, `shadowRadius: 8`, `elevation: 8`.
- **Press animation:** Scale to 0.9 over 100ms with haptic `Medium`.
- **Scroll hide behavior:** FAB hides (translates down by 80pt with spring animation) when user scrolls down, reappears when user scrolls up. Uses `onScroll` event with `useAnimatedScrollHandler` to track scroll direction.
- **Action:** Opens "Add Contact" modal screen (presented modally, slide-up from bottom).

---

### 2. Contact Detail Screen

#### 2.1 Navigation & Transition

- **Entry transition:** Push navigation with standard iOS back-swipe gesture enabled. The contact avatar in the list row animates to the large avatar position in the detail header via a shared element transition (using `react-native-shared-element` or React Navigation's built-in shared element support). Transition duration: 350ms, `Easing.bezier(0.4, 0.0, 0.2, 1.0)`.
- **Back button:** Standard iOS back chevron with `previousScreenTitle` label.
- **Right nav buttons:** Edit button (text "Edit"), overflow menu (`SF Symbol ellipsis.circle`).

#### 2.2 Header

The header is a collapsible section that compresses on scroll (parallax effect).

**Expanded state (scroll position 0):**

```
┌─────────────────────────────────────────────────┐
│              ┌──────────┐                       │
│              │  Avatar   │                       │
│              │   80pt    │                       │
│              └──────────┘                       │
│            Jane Smith ● (hot)  ★                │
│          Senior Agent · Realty Co                │
└─────────────────────────────────────────────────┘
```

| Element | Spec |
|---|---|
| **Avatar** | 80pt x 80pt circle, centered horizontally. Same initials fallback logic as list rows but larger (`fontSize: 32`, `fontWeight: '600'`). Border: 3pt white (`#FFFFFF`) border giving a card-lifted appearance. Shadow: `shadowColor: '#000'`, `shadowOffset: { width: 0, height: 2 }`, `shadowOpacity: 0.15`, `shadowRadius: 8`. |
| **Name** | `fontSize: 24`, `fontWeight: '700'`, centered. Max 2 lines. |
| **Engagement dot** | 10pt diameter (slightly larger than list), same color system, positioned 8pt right of name text, vertically centered with first line of name. Hot pulse animation active. |
| **Favorite star** | `SF Symbol star.fill` (if favorited, tint `#FFCC00`) or `star` (unfavorited, tint `#8E8E93`), 22pt. Positioned 8pt right of engagement dot. Tappable — toggles with `Haptics.impactAsync(Light)` and fires `PATCH /api/crm/contacts/{id}` with `{ is_favorite: true/false }`. |
| **Subtitle** | `fontSize: 15`, `fontWeight: '400'`, `color: secondaryLabel`, centered. Format: `"{title} · {company}"`. If either is missing, omit the separator and the missing field. |

**Collapsed state (scrolled 120pt+):** Avatar shrinks to 36pt and moves to the left of the navigation title area. Name appears as the navigation bar title. Engagement dot and star remain visible at reduced size (8pt, 18pt). Transition is driven by scroll offset via `Animated.interpolate` on `scrollY` shared value.

#### 2.3 Quick Action Row

Positioned immediately below the header. Height: 72pt. Horizontal layout, evenly spaced across the screen width.

| Action | Icon (SF Symbol) | Tint | Label | Behavior |
|---|---|---|---|---|
| **Call** | `phone.fill` | `#34C759` | "Call" | `Linking.openURL('tel:{primary_phone}')`. If multiple phones, show action sheet to choose. |
| **Text** | `message.fill` | `#007AFF` | "Text" | `Linking.openURL('sms:{primary_phone}')`. |
| **Email** | `envelope.fill` | `#5856D6` | "Email" | `Linking.openURL('mailto:{primary_email}')`. If multiple emails, action sheet. |
| **Directions** | `location.fill` | `#FF9500` | "Directions" | `Linking.openURL('maps:?daddr={encoded_address}')`. Uses primary address. |

**Button spec:**

- Container: `flex: 1`, centered content, `alignItems: 'center'`.
- Icon container: 44pt x 44pt circle, background `{tintColor}15` (15% opacity of the tint), icon 22pt centered.
- Label: `fontSize: 12`, `fontWeight: '500'`, `color: {tintColor}`, `marginTop: 4pt`.
- Press: scale 0.9 over 100ms, `Haptics.impactAsync(Light)`.
- Disabled state: if contact lacks the relevant field, icon container background becomes `#8E8E9320`, icon tint `#8E8E93`, label tint `#8E8E93`, `opacity: 0.5`, press does nothing.

#### 2.4 Segmented Sections

Below the quick action row, a segmented control or tab bar switches between five content sections. Implementation: custom segmented control pinned below the quick action row, becoming sticky at the top of the scroll area when the header collapses.

**Segment bar spec:**

- Height: 44pt.
- Background: view background color.
- Segments: `Info`, `Notes`, `Deals`, `Tasks`, `Timeline`.
- Active indicator: 3pt bottom border, `#007AFF`, animated with Reanimated `withSpring` to slide to the active tab's position.
- Text: `fontSize: 14`, `fontWeight: '600'` (active, `#007AFF`), `fontWeight: '500'` (inactive, `secondaryLabel`).
- Swipe gesture: horizontal swipe between sections is supported via a `PagerView` (from `react-native-pager-view`) beneath the segment bar.

##### 2.4.1 Info Section

Structured as a grouped list of fields, matching iOS Settings aesthetic.

**Field groups:**

1. **Phone Numbers** — label, number, type badge (Mobile/Work/Home), tap-to-call, long-press for copy. Multiple entries supported.
2. **Email Addresses** — label, address, type badge, tap-to-email, long-press for copy.
3. **Addresses** — full address block, type badge (Home/Work/Investment Property), tap-for-directions, long-press for copy.
4. **Social Links** — LinkedIn, Facebook, Instagram, X/Twitter, website. Each shows platform icon + URL/handle. Tap opens in-app browser or platform app via deep link.
5. **Custom Fields** — label-value pairs, rendered dynamically. Fields fetched from `GET /api/crm/contacts/{id}/detail` in `custom_fields` array. Each has `{ label: string, value: string, type: 'text' | 'date' | 'number' | 'url' }`.

**Field row spec:**

- Height: auto (minimum 44pt).
- Label: `fontSize: 13`, `fontWeight: '400'`, `color: secondaryLabel`, `marginBottom: 2pt`.
- Value: `fontSize: 17`, `fontWeight: '400'`, `color: label`.
- Group separator: 16pt vertical spacing between groups. Section title: `fontSize: 13`, `fontWeight: '600'`, `color: secondaryLabel`, uppercase, left-padded 16pt.
- Copy feedback: on long-press, `Haptics.notificationAsync(Success)`, toast "Copied to clipboard", value saved to `Clipboard.setString()`.

##### 2.4.2 Notes Section

- **List:** Reverse-chronological FlatList of notes.
- **Note card:** Background `#F2F2F7` (light) / `#2C2C2E` (dark), corner radius 12pt, padding 12pt, margin 8pt horizontal + 4pt vertical.
  - **Author + timestamp:** `"{authorName} · {relativeTime}"`, `fontSize: 13`, `color: secondaryLabel`.
  - **Body:** Rendered markdown (using `react-native-markdown-display`). `fontSize: 15`, `color: label`. Images in markdown are supported (rendered inline, max width 100%, lazy loaded). Code blocks have `backgroundColor: #E5E5EA` / `#3A3A3C`, `fontFamily: 'Menlo'`, corner radius 6pt, padding 8pt.
  - **Actions:** Long-press opens context menu: Edit, Delete, Copy Text, Share.
- **Add note FAB:** Smaller than the list FAB — 48pt circle, `#007AFF`, `SF Symbol square.and.pencil` white 20pt, positioned bottom-right of the notes section content area, `right: 16pt`, `bottom: 16pt`. Tapping opens a modal note editor.
- **Note editor modal:** Full-screen modal. Top bar: Cancel (left), "New Note" (center title), Save (right, `#007AFF`, `fontWeight: '600'`). Body: `TextInput` with `multiline: true`, `placeholder: "Write a note..."`, `fontSize: 17`. Toolbar above keyboard: bold, italic, bullet list, numbered list, link insertion. Save fires `POST /api/crm/contacts/{id}/notes` with `{ body: markdownString }`.

##### 2.4.3 Deals Section

- **Deal card:** Full-width card, corner radius 12pt, border 1pt `#E5E5EA`, padding 16pt, margin 8pt vertical.
  - **Deal name:** `fontSize: 17`, `fontWeight: '600'`.
  - **Stage badge:** Pill shape (same spec as filter chips), background color mapped to stage: Prospect `#007AFF`, Qualification `#5856D6`, Proposal `#FF9500`, Negotiation `#FFCC00`, Closed Won `#34C759`, Closed Lost `#FF3B30`.
  - **Value:** `fontSize: 20`, `fontWeight: '700'`. Formatted with locale currency (e.g., "$425,000").
  - **Expected close:** `fontSize: 13`, `color: secondaryLabel`. Format: `"Expected close: Mar 15, 2026"`. If overdue: `color: #FF3B30`.
  - **Probability:** `fontSize: 13`, `color: secondaryLabel`. Format: `"75% probability"`. Rendered as a thin (4pt) progress bar beneath the card, filled to the probability percentage, color matches stage badge.
- **Tap action:** Navigates to deal detail screen (outside scope of this spec section).
- **Empty state:** "No deals yet", with "Create Deal" button.

##### 2.4.4 Tasks Section

- **Task row:** Height: auto (minimum 52pt). Checkbox (24pt circle, `borderWidth: 2`, `borderColor: #8E8E93` unchecked, filled `#34C759` with white checkmark when checked) on the left. Task title `fontSize: 15`. Due date below title `fontSize: 13`, `color: secondaryLabel`.
- **Overdue highlighting:** If `task.due_date < now && !task.completed`: due date text `color: #FF3B30`, `fontWeight: '600'`, and a subtle red left border (3pt, `#FF3B30`) on the task row.
- **Completion toggle:** Tapping the checkbox fires `PATCH /api/crm/tasks/{taskId}` with `{ completed: true, completed_at: ISO }`. Optimistic update: checkbox fills immediately, task fades (opacity 0.5) and moves to "Completed" section at bottom after 1500ms delay (allowing undo). Undo: toast appears "Task completed" with "Undo" button for 3000ms.
- **Add task:** "Add Task" row at bottom of list with `+ Add Task` text, tint `#007AFF`. Tapping shows an inline text input that expands with title, due date picker (native iOS date picker in compact mode), and assignee selector.
- **Sort:** Overdue first (sorted by due date ascending), then upcoming (due date ascending), then no due date, then completed (completion date descending).

##### 2.4.5 Timeline Section

A reverse-chronological feed of all contact interactions.

**Event types and their rendering:**

| Event Type | Icon (SF Symbol) | Color | Title Format | Detail |
|---|---|---|---|---|
| Phone call | `phone.fill` | `#34C759` | "Called {name}" or "{name} called" | Duration, outcome (connected/voicemail/no answer) |
| Email | `envelope.fill` | `#5856D6` | "Emailed {name}" | Subject line preview, first 2 lines of body |
| Text/SMS | `message.fill` | `#007AFF` | "Texted {name}" | Message preview (first line) |
| Note added | `square.and.pencil` | `#FF9500` | "Note added" | First 2 lines of note content |
| Deal change | `chart.line.uptrend` | `#FFCC00` | "Deal moved to {stage}" | Deal name, old stage → new stage |
| Task created | `checkmark.circle` | `#34C759` | "Task created" | Task title, due date |
| Task completed | `checkmark.circle.fill` | `#34C759` | "Task completed" | Task title |
| AI interaction | `sparkles` | `#AF52DE` | "AI summary generated" | Preview of AI output |
| Meeting | `calendar` | `#FF2D55` | "Meeting scheduled" | Date, time, location |

**Event card spec:**

- Left rail: vertical line (2pt, `#E5E5EA` light / `#38383A` dark) connecting event icons. Each icon is 28pt circle with the event-type color at 15% opacity background, icon at 14pt.
- Content area: 12pt left of the icon rail. Event title `fontSize: 15`, `fontWeight: '500'`. Detail `fontSize: 13`, `color: secondaryLabel`. Timestamp at top-right `fontSize: 12`, `color: tertiaryLabel`, relative format.
- Tap action: expands the event card to show full detail inline (animated height expansion, 200ms).
- **Pagination:** Initial load: 20 events. "Load more" button at the bottom fetches next 20 from `GET /api/crm/contacts/{id}/timeline?cursor={lastEventId}&limit=20`.

#### 2.5 Edit Mode

- **Activation:** Tapping "Edit" in the nav bar transitions all Info section fields to editable state.
- **Nav bar changes:** "Edit" becomes "Cancel" (left) and "Save" (right, `#007AFF`, `fontWeight: '700'`, disabled until changes are detected).
- **Field editing:** Each field transforms from a display `<Text>` to a `<TextInput>` with a bottom border (1pt, `#007AFF`). Pre-filled with current value. Keyboard type matches field type (`phone-pad` for phones, `email-address` for emails, `default` for text).
- **Add field:** Each field group has an "+ Add {Field}" row at the bottom (e.g., "+ Add Phone Number"). Tapping inserts a new empty field row.
- **Delete field:** Each field row in edit mode shows a red minus circle (`SF Symbol minus.circle.fill`, `#FF3B30`) on the left. Tapping removes the field with `LayoutAnimation`.
- **Save:** `PATCH /api/crm/contacts/{id}` with only changed fields (delta payload). Optimistic update applied immediately.
- **Cancel:** Reverts all fields to pre-edit state. If changes exist, shows confirmation: "Discard changes?" with "Keep Editing" and "Discard" options.
- **Validation:** Phone numbers validated with `libphonenumber-js` (E.164 format). Emails validated with RFC 5322 regex. Invalid fields show red border and inline error message.

#### 2.6 Edit Conflict Handling

**Optimistic update flow:**

```
User taps Save
  → Immediately update local UI and SQLite cache
  → Mark record as `syncStatus: 'pending'` in SQLite
  → Show subtle "Saving..." indicator (small spinner in nav bar)
  → Fire PATCH to server

Server responds 200:
  → Update SQLite `syncStatus: 'synced'`, store server's `updated_at`
  → Remove "Saving..." indicator

Server responds 409 (Conflict):
  → Show conflict resolution modal (see below)
  → Do NOT revert local UI yet

Network unavailable:
  → Record queued in `sync_queue` SQLite table with payload and timestamp
  → Show "Pending sync" badge: small amber dot on the contact in the list,
    and a banner on the detail screen: "Changes saved locally · Will sync when online"
  → When network restored: process queue in FIFO order
```

**Conflict resolution modal:**

- Title: "This contact was updated elsewhere"
- Body: side-by-side diff showing "Your changes" and "Server version" for each conflicting field.
- Options: "Keep Mine", "Keep Theirs", "Merge" (shows field-by-field picker).
- "Keep Mine" fires `PATCH` with `force: true` (or `If-Match` header with client's `updated_at`).
- "Keep Theirs" discards local changes and refreshes from server.

#### 2.7 Related Contacts

- **Position:** Below the segmented sections, before the delete button.
- **Header:** "Related Contacts" with count, `fontSize: 17`, `fontWeight: '600'`.
- **Logic:** Contacts that share `company` name, or are linked in the same deal, or are manually linked via `contact.related_contact_ids`.
- **Rendering:** Horizontal scroll of compact contact cards (120pt wide, 140pt tall): avatar (36pt), name (2 lines max), relationship badge ("Same Company", "Co-Buyer", etc.).
- **Tap:** Navigates to that contact's detail screen (push onto nav stack).

#### 2.8 Delete Contact

- **Position:** Bottom of the scroll view, `marginTop: 32pt`, centered.
- **Button:** Text-only, `color: #FF3B30`, `fontSize: 17`, `fontWeight: '400'`. Text: "Delete Contact".
- **Confirmation:** Alert dialog: title "Delete {name}?", message "This will permanently delete this contact, all their notes, tasks, and timeline history. This action cannot be undone.", buttons: "Cancel" (default), "Delete" (destructive).
- **Execution:** `DELETE /api/crm/contacts/{id}`. Optimistic removal from local cache. Navigation pops back to People tab. If offline, queue the deletion and show the contact as "Pending deletion" (strikethrough name, reduced opacity).

---

### 3. Properties Tab — Property List

#### 3.1 Property List Item (List View)

Each property row is **100pt tall** with the following layout:

```
┌─────────────────────────────────────────────────────────────────────┐
│  ┌──────────────┐  12pt  ┌──────────────────────────────┐          │
│  │  Street View  │       │ 123 Main Street               │          │
│  │  Thumbnail    │       │ Anytown, ST 12345              │          │
│  │  120x80pt     │       │ $425,000 · 3bd/2ba · 1,850sf  │          │
│  │  cornerR: 8   │       │ ┌──────────┐                  │          │
│  └──────────────┘        │ │ Active ●  │                  │          │
│                           │ └──────────┘                  │          │
│                           └──────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────────┘
```

| Element | Specification |
|---|---|
| **Row container** | Height: 100pt. Padding: 12pt all sides. `flexDirection: 'row'`, `alignItems: 'center'`. |
| **Thumbnail** | 120pt x 80pt, `borderRadius: 8`. Source: `property.street_view_url` or `property.primary_photo_url`. Placeholder: blurhash string from `property.thumbnail_blurhash` rendered via `expo-image` `placeholder` prop, transition 300ms. If no image available: gray placeholder with `SF Symbol house.fill` centered, 32pt, `#8E8E93`. Lazy loaded: `<Image loading="lazy">` (expo-image handles this natively). |
| **Address** | Line 1: street address, `fontSize: 17`, `fontWeight: '600'`. Line 2: city, state, zip, `fontSize: 14`, `fontWeight: '400'`, `color: secondaryLabel`. |
| **Metrics** | `fontSize: 14`, `fontWeight: '400'`, `color: secondaryLabel`. Format: `"$425,000 · 3bd/2ba · 1,850sf"`. Price formatted with locale-aware number formatting. |
| **Status badge** | Pill: height 22pt, `borderRadius: 11`, `paddingHorizontal: 8`, `fontSize: 12`, `fontWeight: '600'`. Colors: Active `bg: #34C75920, text: #34C759`, Pending `bg: #FF950020, text: #FF9500`, Sold `bg: #FF3B3020, text: #FF3B30`, Off-Market `bg: #8E8E9320, text: #8E8E93`. |

#### 3.2 Property List Item (Grid View)

2-column grid, each cell is a card:

| Property | Value |
|---|---|
| **Card width** | `(screenWidth - 16*2 - 8) / 2` (16pt margin on each side, 8pt gap between columns) |
| **Card height** | Auto. Image: aspect ratio 3:2 (width-determined). Content padding: 10pt. |
| **Image** | Full card width, aspect ratio 3:2, `borderTopLeftRadius: 12`, `borderTopRightRadius: 12`. Same blurhash placeholder strategy. |
| **Content** | Below image. Address: `fontSize: 14`, `fontWeight: '600'`, 2 lines max. Price: `fontSize: 16`, `fontWeight: '700'`. Beds/baths/sqft: `fontSize: 12`, `color: secondaryLabel`. Status badge (same spec, reduced to 20pt height). |
| **Card styling** | `borderRadius: 12`, `backgroundColor: systemBackground`, `shadowColor: '#000'`, `shadowOffset: { width: 0, height: 1 }`, `shadowOpacity: 0.1`, `shadowRadius: 4`. |

**Toggle:** An icon button in the nav bar: `SF Symbol list.bullet` (list active) or `SF Symbol square.grid.2x2` (grid active). Tapping switches with a crossfade animation (200ms). Preference persisted to `AsyncStorage` key `@skye/property_view_mode`.

#### 3.3 Search

- **Search bar:** Same spec as People tab search bar, but placeholder: "Search address, MLS#, or price range...".
- **Search behavior:** Debounce 300ms. Local SQLite search on `address`, `mls_number`, `price`. Server fallback to `GET /api/properties/search?q={query}`.
- **Price range parsing:** If user types "$300k-$500k" or "300000-500000", the search recognizes this as a price range filter and applies `WHERE price >= 300000 AND price <= 500000` locally.

#### 3.4 Filters

**Filter panel:** Unlike the People tab chips (which are inline), the Properties tab uses a filter bottom sheet triggered by a filter icon in the nav bar (`SF Symbol line.3.horizontal.decrease.circle`). Active filter count shown as badge on the icon.

**Filter fields:**

| Filter | UI Component | Spec |
|---|---|---|
| **Status** | Horizontal chip group | Active, Pending, Sold, Off-Market. Multi-select. |
| **Price Range** | Dual-thumb range slider | Min: $0, Max: $10M (or max in dataset). Step: $10,000. Labels show formatted values. Slider track: 4pt, inactive `#E5E5EA`, active `#007AFF`. Thumbs: 28pt circles, white, shadow. |
| **Bedrooms** | Segmented chips | Any, 1+, 2+, 3+, 4+, 5+. Single-select. |
| **Bathrooms** | Segmented chips | Any, 1+, 2+, 3+, 4+. Single-select. |
| **Square Footage** | Dual-thumb range slider | Min: 0, Max: 10,000sf. Step: 100. |
| **Property Type** | Chip group | Single Family, Condo, Townhouse, Multi-Family, Land, Commercial. Multi-select. |

**Bottom sheet spec:** Presented via `@gorhom/bottom-sheet`. Snap points: 60% of screen height (default), 90% (expanded). Background dimming at 40% black. Handle bar: 36pt x 5pt, centered, corner radius 2.5pt, `backgroundColor: #E5E5EA`. "Apply Filters" button at bottom: full-width, height 50pt, `backgroundColor: #007AFF`, white text, `borderRadius: 12`. "Reset All" text button next to "Apply".

#### 3.5 Sort

Same action sheet pattern as People tab. Options: Price (high to low), Price (low to high), Date Added (newest), Address (A-Z).

#### 3.6 Map View Toggle

- **Toggle:** Tab bar-like segment at the top of the Properties tab: `List | Map`. Active segment has bottom border `#007AFF`.
- **Map implementation:** `react-native-maps` with `provider={PROVIDER_DEFAULT}` (uses Apple MapKit on iOS).
- **Property pins:** Custom markers using `<Marker>`. Pin color matches status: Active `#34C759`, Pending `#FF9500`, Sold `#FF3B30`, Off-Market `#8E8E93`. Custom callout on tap: mini property card (thumbnail, address, price, beds/baths) in a 260pt x 100pt floating card above the pin.
- **Clustering:** When zoomed out, pins cluster using `react-native-map-clustering`. Cluster circle shows count, background `#007AFF`.
- **Region:** Initial region centers on the user's location (if permitted) with a 10-mile radius. Falls back to the centroid of all properties.
- **Interaction:** Tapping a callout navigates to Property Detail. Tapping the map background dismisses any open callout. Pinch-to-zoom and pan are native.
- **Filter sync:** Map view respects all active filters — only matching properties show pins.
- **Performance:** Maximum 200 markers rendered simultaneously. If filtered set exceeds 200, cluster more aggressively. Never render more than 200 individual marker views.

#### 3.7 States

- **Pull-to-refresh:** Same spec as People tab but calls `GET /api/properties`.
- **Empty state:** House illustration, "No properties yet", "Add your first property or connect an MLS feed." Buttons: "Add Property", "Connect MLS".
- **Error state:** Warning triangle illustration, "Couldn't load properties", "Check your connection and try again." Button: "Retry". Also shown as inline banner if partial load fails.
- **Loading state:** 4 skeleton cards matching list item dimensions — pulsing gray rectangles (opacity oscillating 0.3 to 0.7 over 1000ms, `Easing.inOut(Easing.ease)`).

---

### 4. Property Detail Screen

#### 4.1 Navigation & Transition

- **Entry:** Push navigation. If entering from map callout, the callout card morphs into the hero image area via a shared element transition (350ms).
- **Nav bar:** Transparent initially, becomes opaque (`systemBackground`) as user scrolls past the hero image. Back chevron and action buttons (`SF Symbol square.and.arrow.up` for share, `SF Symbol ellipsis.circle` for menu) are white when over the hero, switching to system tint when the bar becomes opaque.

#### 4.2 Hero Image Gallery

- **Height:** 300pt (including safe area top inset).
- **Component:** Horizontal `FlatList` with `pagingEnabled: true`. Each item is a full-width image.
- **Page indicator:** Centered at the bottom of the gallery, 16pt from bottom. Dots: 6pt circles, active `#FFFFFF`, inactive `#FFFFFF` at 50% opacity. Spacing: 8pt between dots. Max 5 dots with shrinking/sliding behavior for 6+ images.
- **Images:** Source priority: (1) agent-uploaded photos, (2) MLS photos, (3) Street View fallback. Loaded via `expo-image` with `contentFit: 'cover'` and blurhash placeholders.
- **Zoom:** Double-tap to zoom 2x (animated 300ms). Pinch-to-zoom up to 4x. Panning while zoomed. Single tap dismisses zoom or toggles UI overlay visibility.
- **Gradient overlay:** Bottom 100pt of each image has a linear gradient from `transparent` to `rgba(0,0,0,0.5)` for text legibility of overlaid elements.

#### 4.3 Address & Price Block

Positioned immediately below the hero gallery (or overlaid at the bottom of the hero with the gradient).

| Element | Spec |
|---|---|
| **Address line 1** | Street address. `fontSize: 22`, `fontWeight: '700'`. |
| **Address line 2** | City, State Zip. `fontSize: 15`, `fontWeight: '400'`, `color: secondaryLabel`. |
| **Price** | `fontSize: 28`, `fontWeight: '800'`. If status is Sold, show strikethrough on list price and "Sold for $X" below. |
| **Status badge** | Same pill spec as list items but larger: height 26pt, `fontSize: 14`. |
| **MLS number** | `fontSize: 12`, `color: tertiaryLabel`. Format: `"MLS# 12345678"`. |

#### 4.4 Key Metrics Row

Horizontal row of icon-value pairs. Evenly spaced across width. Padding: 16pt vertical, 16pt horizontal. Background: `systemGroupedBackground`. Corner radius: 12pt. Margin: 16pt horizontal.

| Metric | Icon (SF Symbol) | Format |
|---|---|---|
| **Bedrooms** | `bed.double.fill` | "3 bd" |
| **Bathrooms** | `shower.fill` | "2 ba" |
| **Square Footage** | `square.split.diagonal.2x2` | "1,850 sf" |
| **Lot Size** | `leaf.fill` | "0.25 ac" or "10,890 sf" |
| **Year Built** | `calendar` | "2005" |

Each metric: icon 18pt, `color: secondaryLabel`, value below `fontSize: 15`, `fontWeight: '600'`. Icon and text centered vertically in their column. Separator: 1pt vertical line, `#E5E5EA`, height 28pt, between each pair.

#### 4.5 AVM (Automated Valuation Model) Section

- **Section header:** "Estimated Value", `fontSize: 17`, `fontWeight: '600'`, padding 16pt horizontal.
- **Estimated value:** `fontSize: 28`, `fontWeight: '700'`, `color: #34C759` (if above list price) or `#FF3B30` (if below). Source: `property.avm.estimated_value`.
- **Confidence range:** `fontSize: 14`, `color: secondaryLabel`. Format: `"$410,000 – $440,000"`. Rendered as a horizontal bar beneath the estimate: thin (6pt) bar, background `#E5E5EA`, filled portion `#34C759`, with a diamond marker at the point estimate.
- **Value history chart:** Line chart showing `property.avm.history` (array of `{ date, value }` objects). Chart library: `react-native-charts-wrapper` or `victory-native`. Chart height: 180pt. X-axis: dates (monthly). Y-axis: dollar values (auto-scaled with $50K increments). Line: 2pt, `#007AFF`. Fill: gradient from `#007AFF20` to `transparent`. Interactive: tap-and-hold to scrub — tooltip shows date and value at the touch point. Horizontal dashed line at current list price, labeled "List Price".
- **Data source attribution:** `fontSize: 11`, `color: tertiaryLabel`. Text: "Estimated by Skye AI · Updated {date}".

#### 4.6 Comparable Properties

- **Section header:** "Comparable Properties" with count, "See All" trailing link (`color: #007AFF`).
- **Layout:** Horizontal `FlatList`, `showsHorizontalScrollIndicator: false`, `contentContainerStyle: { paddingHorizontal: 16 }`.
- **Comp card:** Width: 220pt. Height: auto. Corner radius: 12pt. Shadow: same as grid view card.
  - Image: 220pt x 130pt, `borderTopLeftRadius: 12`, `borderTopRightRadius: 12`.
  - Address: `fontSize: 14`, `fontWeight: '500'`, 1 line.
  - Price: `fontSize: 16`, `fontWeight: '700'`.
  - Beds/baths/sqft: `fontSize: 12`, `color: secondaryLabel`.
  - Distance: `fontSize: 12`, `color: tertiaryLabel`. Format: `"0.3 mi away"`.
  - Price delta: `fontSize: 12`, `color: #34C759` or `#FF3B30`. Format: `"+$15,000"` or `"-$20,000"` relative to subject property.
- **Spacing:** 12pt between cards.
- **Tap:** Navigates to that property's detail screen.

#### 4.7 Market Stats

- **Section header:** "Market Insights — {Neighborhood Name}".
- **Cards:** Vertical stack of stat cards, each 72pt tall, full width minus 32pt margin, corner radius 12pt, background `systemGroupedBackground`.

| Stat | Label | Value Format | Trend |
|---|---|---|---|
| Median Sale Price | "Median Sale Price" | "$425,000" | Arrow up/down + percentage: "+3.2% YoY" |
| Days on Market | "Avg. Days on Market" | "24 days" | Arrow + delta: "-5 days" |
| List-to-Sale Ratio | "List-to-Sale Ratio" | "98.5%" | Arrow + delta: "+1.2%" |
| Active Listings | "Active Listings" | "47" | Arrow + delta: "+8 this month" |

**Trend indicators:** Up arrow `#34C759` for favorable trends (price up, days down, ratio up), down arrow `#FF3B30` for unfavorable. Arrow icon: `SF Symbol arrow.up.right` or `arrow.down.right`, 12pt.

#### 4.8 Associated Contacts

- **Section header:** "Associated Contacts" with count.
- **Contact cards:** Similar to Related Contacts in Contact Detail. Vertical list (not horizontal scroll, since there are typically fewer). Each row: avatar 40pt, name, role badge (Buyer, Seller, Agent, Lender), phone quick-action icon.
- **Role badge:** Pill, height 20pt, `fontSize: 11`, `fontWeight: '600'`. Buyer: `#007AFF` bg 15%, `#007AFF` text. Seller: `#FF9500` bg 15%, `#FF9500` text. Agent: `#5856D6` bg 15%. Lender: `#34C759` bg 15%.
- **Tap:** Navigates to Contact Detail.
- **Add association:** "+ Link Contact" row at bottom. Opens a contact picker modal (searchable list of all contacts).

#### 4.9 Notes Section

Identical spec to Contact Detail Notes (section 2.4.2) but scoped to `property.id`. API: `GET /api/properties/{id}/notes`, `POST /api/properties/{id}/notes`.

#### 4.10 Share Property

- **Button position:** In the nav bar (share icon) and as a full-width button in the content area below Market Stats.
- **Share sheet options:**
  1. **Generate Link:** Calls `POST /api/properties/{id}/share-link`, returns a short URL. Copies to clipboard and opens `Share.share({ url })` (iOS native share sheet).
  2. **Generate Summary:** Calls `POST /api/properties/{id}/summary` (AI-generated text summary). Opens share sheet with text.
  3. **Export PDF:** Calls `POST /api/properties/{id}/export-pdf`. Downloads PDF and opens share sheet with the file.

#### 4.11 "Ask Skye" Action Button

- **Position:** Floating at the bottom of the screen (above the tab bar), full-width minus 32pt margin, height 50pt, `borderRadius: 25pt`.
- **Design:** Gradient background from `#5856D6` to `#AF52DE` (left to right). Text: "Ask Skye about this property", `fontSize: 16`, `fontWeight: '600'`, white. `SF Symbol sparkles` icon 18pt to the left of text.
- **Action:** Navigates to the Chat tab with a pre-loaded system context message: `"The user is viewing {address}, a {beds}bd/{baths}ba {sqft}sf {type} listed at {price}. Status: {status}. MLS#: {mls}."` The chat opens with a suggested prompt: "What can you tell me about this property?".
- **Animation:** Subtle shimmer effect — a diagonal white gradient (10% opacity, 40pt wide) translates across the button from left to right over 2000ms, repeating every 4000ms. Implemented with Reanimated `withRepeat(withTiming(...))` driving `translateX` on an `Animated.View` overlay with `overflow: 'hidden'`.

---

### 5. Offline Data Strategy

#### 5.1 Cache Architecture

**Database:** SQLite via `expo-sqlite` (synchronous API for reads, async for writes). Single database file: `skye_cache.db`.

**Schema:**

```sql
-- Core tables mirror API response shapes
CREATE TABLE contacts (
  id TEXT PRIMARY KEY,
  data TEXT NOT NULL,          -- JSON blob of full contact object
  engagement_status TEXT,      -- Precomputed: 'hot' | 'warm' | 'cold' | 'inactive'
  last_activity_date TEXT,     -- Indexed for sort
  created_at TEXT,             -- Indexed for sort
  updated_at TEXT,             -- Server timestamp for conflict detection
  cached_at INTEGER NOT NULL,  -- Unix ms timestamp when cached
  last_viewed_at INTEGER,      -- For LRU eviction
  sync_status TEXT DEFAULT 'synced' -- 'synced' | 'pending' | 'conflict'
);

CREATE INDEX idx_contacts_engagement ON contacts(engagement_status);
CREATE INDEX idx_contacts_last_activity ON contacts(last_activity_date);
CREATE INDEX idx_contacts_cached_at ON contacts(cached_at);
CREATE INDEX idx_contacts_last_viewed ON contacts(last_viewed_at);
CREATE INDEX idx_contacts_sync_status ON contacts(sync_status);

-- Full-text search virtual table
CREATE VIRTUAL TABLE contacts_fts USING fts5(
  id,
  first_name,
  last_name,
  company,
  email,
  phone,
  content='contacts',
  content_rowid='rowid'
);

CREATE TABLE contact_details (
  contact_id TEXT PRIMARY KEY,
  notes TEXT,        -- JSON array
  deals TEXT,        -- JSON array
  tasks TEXT,        -- JSON array
  timeline TEXT,     -- JSON array (first 50 events)
  cached_at INTEGER NOT NULL,
  last_viewed_at INTEGER,
  sync_status TEXT DEFAULT 'synced'
);

CREATE TABLE properties (
  id TEXT PRIMARY KEY,
  data TEXT NOT NULL,
  status TEXT,
  price REAL,
  cached_at INTEGER NOT NULL,
  last_viewed_at INTEGER,
  sync_status TEXT DEFAULT 'synced'
);

CREATE INDEX idx_properties_status ON properties(status);
CREATE INDEX idx_properties_price ON properties(price);
CREATE INDEX idx_properties_cached_at ON properties(cached_at);

CREATE TABLE property_details (
  property_id TEXT PRIMARY KEY,
  avm TEXT,            -- JSON object
  comparables TEXT,    -- JSON array
  market_stats TEXT,   -- JSON object
  notes TEXT,          -- JSON array
  contacts TEXT,       -- JSON array
  cached_at INTEGER NOT NULL,
  last_viewed_at INTEGER,
  sync_status TEXT DEFAULT 'synced'
);

-- Offline mutation queue
CREATE TABLE sync_queue (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  entity_type TEXT NOT NULL,  -- 'contact' | 'property' | 'note' | 'task'
  entity_id TEXT NOT NULL,
  action TEXT NOT NULL,       -- 'create' | 'update' | 'delete'
  payload TEXT NOT NULL,      -- JSON of the mutation
  created_at INTEGER NOT NULL,
  retry_count INTEGER DEFAULT 0,
  last_error TEXT,
  status TEXT DEFAULT 'pending'  -- 'pending' | 'in_progress' | 'failed'
);

CREATE INDEX idx_sync_queue_status ON sync_queue(status);

-- Image cache metadata
CREATE TABLE image_cache (
  url TEXT PRIMARY KEY,
  local_path TEXT NOT NULL,
  size_bytes INTEGER NOT NULL,
  cached_at INTEGER NOT NULL,
  last_accessed_at INTEGER NOT NULL
);
```

#### 5.2 Cache Invalidation

**Stale-after strategy:**

| Data Type | Stale After | Refresh Trigger |
|---|---|---|
| Contact list | 5 minutes | App foreground, pull-to-refresh, network restore |
| Contact detail | 5 minutes | Screen focus, pull-to-refresh |
| Property list | 15 minutes | App foreground, pull-to-refresh, network restore |
| Property detail | 15 minutes | Screen focus, pull-to-refresh |
| AVM data | 24 hours | Manual refresh button in AVM section |
| Market stats | 24 hours | Implicit on property detail refresh |

**Staleness check:**

```typescript
function isStale(cachedAt: number, maxAgeMs: number): boolean {
  return Date.now() - cachedAt > maxAgeMs;
}

const STALE_THRESHOLDS = {
  contactList: 5 * 60 * 1000,      // 5 minutes
  contactDetail: 5 * 60 * 1000,
  propertyList: 15 * 60 * 1000,    // 15 minutes
  propertyDetail: 15 * 60 * 1000,
  avm: 24 * 60 * 60 * 1000,        // 24 hours
  marketStats: 24 * 60 * 60 * 1000,
};
```

**Read strategy:** Always read from cache first (instant UI), then check staleness. If stale and online, fetch fresh data in the background and update UI when received. If not stale, do nothing. This ensures the UI always appears instantly, with fresh data arriving 100-500ms later if needed.

#### 5.3 What is Cached

| Data | Scope | Notes |
|---|---|---|
| Full contact list | All contacts, all fields | Stored as individual rows in `contacts` table for per-record access. FTS index populated for search. |
| Contact detail | Notes, deals, tasks, timeline | Cached on first view. Timeline limited to most recent 50 events in cache. |
| Property list | All properties, all fields | Including thumbnail URLs (not image binaries — those are handled by expo-image disk cache). |
| Property detail | AVM, comparables, market stats, notes, associated contacts | Cached on first view. |
| Thumbnail images | Street View, property photos, contact avatars | Managed by `expo-image` disk cache (separate from SQLite). Blurhash strings stored in the JSON data blobs. |

#### 5.4 Cache Size Management

- **Target max cache size:** 50MB for SQLite database. Image cache managed separately by `expo-image` (configurable to 200MB).
- **LRU eviction:** When SQLite database file exceeds 50MB (checked on app foreground and after bulk writes):
  1. Identify contacts/properties not viewed in the last 30 days (`last_viewed_at < now - 30 days`).
  2. Delete their detail records (`contact_details`, `property_details`).
  3. If still over 50MB, delete contact/property list entries by `last_viewed_at` ascending until under 45MB (5MB buffer).
  4. Run `VACUUM` to reclaim space (expensive — run only when eviction occurs, and only when app is in background via a background task).
- **Never evict:** Records with `sync_status: 'pending'` (unsynced mutations must be preserved).
- **Priority:** Recently viewed contacts are never evicted. Contacts viewed in the current session get `last_viewed_at` updated on each view.

#### 5.5 Sync Strategy

**Sync triggers (in priority order):**

1. **Network restoration:** `NetInfo.addEventListener` detects connectivity change from offline to online. Immediate sync of pending queue.
2. **App foreground:** `AppState.addEventListener` detects transition from `background`/`inactive` to `active`. Check staleness and sync pending queue.
3. **Pull-to-refresh:** User-initiated. Full refresh of the current screen's data.
4. **Screen focus:** React Navigation `useIsFocused` or `useFocusEffect`. Refresh the current screen's data if stale.
5. **Periodic (when app is active):** Every 5 minutes, check for pending queue items and attempt sync. Uses `setInterval` cleared on app background.

**Sync queue processing:**

```typescript
async function processSyncQueue(): Promise<void> {
  const pending = await db.getAllAsync<SyncQueueItem>(
    'SELECT * FROM sync_queue WHERE status = ? ORDER BY created_at ASC',
    ['pending']
  );

  for (const item of pending) {
    try {
      await db.runAsync(
        'UPDATE sync_queue SET status = ? WHERE id = ?',
        ['in_progress', item.id]
      );

      await executeSync(item); // Fires the appropriate API call

      await db.runAsync('DELETE FROM sync_queue WHERE id = ?', [item.id]);
      await updateEntitySyncStatus(item.entity_type, item.entity_id, 'synced');
    } catch (error) {
      const retryCount = item.retry_count + 1;
      if (retryCount >= 5) {
        await db.runAsync(
          'UPDATE sync_queue SET status = ?, retry_count = ?, last_error = ? WHERE id = ?',
          ['failed', retryCount, error.message, item.id]
        );
        // Surface to user: "Some changes couldn't sync. Tap to review."
      } else {
        await db.runAsync(
          'UPDATE sync_queue SET status = ?, retry_count = ?, last_error = ? WHERE id = ?',
          ['pending', retryCount, error.message, item.id]
        );
        // Exponential backoff: wait 2^retryCount seconds before next attempt
      }
    }
  }
}
```

#### 5.6 Offline Indicators

**Global offline banner:**

- Appears at the top of every screen (below the status bar, above the navigation bar).
- Height: 32pt. Background: `#8E8E93`. Text: "You're offline · Viewing cached data", `fontSize: 13`, `fontWeight: '500'`, white, centered.
- Animation: slides down from under the status bar over 300ms on network loss, slides up to dismiss over 300ms on network restoration.
- After dismissal, a temporary green banner "Back online · Syncing..." appears for 2000ms.

**Disabled action buttons:**

When offline, the following actions are disabled (opacity 0.4, non-interactive):

- Share Property (requires server to generate link)
- "Ask Skye" (requires server AI)
- Map View (tiles may not load — show cached tiles if available, otherwise show "Map unavailable offline" placeholder)
- MLS search (server-only)
- Export PDF

Actions that work offline (queue mutations):

- Edit contact fields → queued
- Add/edit notes → queued
- Create/complete tasks → queued
- Call/Text/Email/Directions → use native OS capabilities, work offline
- Search contacts/properties → searches local cache

**Per-record sync badges:**

- On contact/property list items with pending changes: small amber circle (6pt) at the top-right of the avatar/thumbnail. Tooltip on tap: "Changes pending sync".
- On the detail screen: banner below the header: "Edited offline · Will sync when connected", background `#FF950020`, `borderColor: #FF9500`, height 36pt.

#### 5.7 Conflict Resolution

**Strategy by field type:**

| Field Type | Strategy | Example |
|---|---|---|
| Simple scalar fields (name, email, phone, company) | Last-write-wins by timestamp | If client writes `name: "John"` at T1 and server has `name: "Johnny"` at T2 where T2 > T1, server wins. If T1 > T2, client wins. |
| Array fields (notes, tasks) | Merge | Both client additions and server additions are kept. Deletions: if client deletes item X and server still has X, delete wins. |
| Engagement score | Server always wins | Computed server-side, not editable by client. |
| Custom fields | Last-write-wins per field | Each custom field tracked independently. |

**Timestamp collision window:** If client and server timestamps are within 60 seconds of each other, the system cannot confidently determine the "last" write. In this case:

1. Mark the contact/property as `sync_status: 'conflict'`.
2. Show a conflict badge (red exclamation) on the contact/property in the list.
3. On entering detail screen: conflict resolution modal (spec in section 2.6).
4. User must manually resolve.

**Merge algorithm for arrays (notes example):**

```typescript
function mergeNotes(
  localNotes: Note[],
  serverNotes: Note[],
  baseNotes: Note[]   // The state at last sync
): Note[] {
  const localAdded = localNotes.filter(n => !baseNotes.find(b => b.id === n.id));
  const serverAdded = serverNotes.filter(n => !baseNotes.find(b => b.id === n.id));
  const localDeleted = baseNotes.filter(b => !localNotes.find(n => n.id === b.id));
  const serverDeleted = baseNotes.filter(b => !serverNotes.find(n => n.id === b.id));

  const allDeleted = new Set([
    ...localDeleted.map(n => n.id),
    ...serverDeleted.map(n => n.id),
  ]);

  // Start with server notes (source of truth for existing items)
  const merged = serverNotes.filter(n => !allDeleted.has(n.id));

  // Add local additions that aren't already on the server
  for (const note of localAdded) {
    if (!merged.find(m => m.id === note.id)) {
      merged.push(note);
    }
  }

  // Sort by created_at descending
  return merged.sort((a, b) =>
    new Date(b.created_at).getTime() - new Date(a.created_at).getTime()
  );
}
```

#### 5.8 Background Sync

- **Trigger:** `AppState` change to `active` from `background`.
- **Sequence:**
  1. Process sync queue (pending mutations).
  2. Fetch updated contacts since `lastSyncTimestamp` via `GET /api/crm/contacts?updatedSince={ts}`.
  3. Merge server updates into local cache.
  4. Update FTS index for any changed contacts.
  5. Only after sync completes: update the UI. Use a React context `SyncContext` that provides `lastSyncTimestamp` — screens observe this and re-query SQLite when it changes.
- **Throttle:** Background sync runs at most once per 30 seconds (prevents rapid foreground/background cycling from flooding the API).
- **Silent execution:** No loading spinners or UI disruption. Changes appear as if the data was always there. If significant changes occurred (e.g., contact deleted by another user), show a toast: "Contact list updated".

---

### 6. Performance Targets

#### 6.1 Contact List Rendering

| Metric | Target | Measurement Method |
|---|---|---|
| Scroll frame rate | 60fps sustained with 500+ contacts | React Native Perf Monitor (`__DEV__` frames) and Instruments Core Animation FPS |
| Time to interactive | < 800ms from tab tap to scrollable list | `performance.now()` marks: tab press → first `onLayout` of FlatList |
| Scroll jank (frame drops) | 0 dropped frames during continuous 5s scroll | Instruments → Core Animation → Frame drops |
| Memory during scroll | No increase beyond initial render allocation | Instruments → Allocations, monitor heap during 30s scroll |

**Optimization checklist:**

- [ ] `getItemLayout` returns correct pre-computed offsets (no runtime measurement).
- [ ] Row component wrapped in `React.memo` with custom `areEqual` comparing only `id` and `updated_at`.
- [ ] No inline function definitions in render (all callbacks `useCallback`-wrapped).
- [ ] No `new Object()` or `new Array()` in render path (style objects are static `StyleSheet.create` references).
- [ ] Avatar images use `expo-image` with `cachePolicy: 'memory-disk'` and fixed dimensions.
- [ ] Engagement status precomputed, not calculated per-render.
- [ ] Swipeable component uses `react-native-gesture-handler` (native thread gesture processing, not JS).

#### 6.2 Search Performance

| Metric | Target |
|---|---|
| Local cache search (SQLite FTS5) | < 100ms for any query against 500 contacts |
| Server search (API round-trip) | < 500ms p95 on LTE |
| Search result display | < 1 frame (16.7ms) from data arrival to UI update |
| Debounce latency | 300ms from last keystroke to search execution |
| Total perceived latency | < 400ms from last keystroke to visible results (300ms debounce + <100ms local search) |

**FTS5 query:**

```sql
SELECT c.id, c.data,
  highlight(contacts_fts, 1, '<mark>', '</mark>') AS first_name_hl,
  highlight(contacts_fts, 2, '<mark>', '</mark>') AS last_name_hl,
  highlight(contacts_fts, 3, '<mark>', '</mark>') AS company_hl
FROM contacts_fts
JOIN contacts c ON c.id = contacts_fts.id
WHERE contacts_fts MATCH ?
ORDER BY rank
LIMIT 50;
```

#### 6.3 Image Loading

| Metric | Target |
|---|---|
| Street View thumbnail first load (LTE) | < 1000ms |
| Street View thumbnail cached load | < 50ms (memory cache), < 200ms (disk cache) |
| Contact avatar first load | < 500ms |
| Blurhash placeholder render | < 16ms (single frame) |
| Image cache hit rate (returning user) | > 95% |

**expo-image configuration:**

```typescript
<Image
  source={{ uri: property.street_view_url }}
  placeholder={{ blurhash: property.thumbnail_blurhash }}
  contentFit="cover"
  transition={300}
  cachePolicy="memory-disk"
  recyclingKey={property.id}
  style={{ width: 120, height: 80, borderRadius: 8 }}
/>
```

#### 6.4 Detail Screen Loading

| Metric | Target |
|---|---|
| Time to skeleton display | < 100ms from navigation start |
| Time to cached data display | < 200ms from navigation start |
| Time to fresh data display | < 1000ms from navigation start |
| Section tab switch | < 100ms (content pre-rendered in PagerView) |
| Shared element transition | 350ms total, no dropped frames |

**Loading sequence:**

```
T+0ms:    Navigation starts, shared element transition begins
T+16ms:   Skeleton layout renders (header skeleton + section skeleton)
T+100ms:  SQLite cache read completes, cached data replaces skeletons
T+350ms:  Shared element transition completes
T+500ms:  Network request completes (if cache was stale), data merges in
          If network is slow, cached data remains visible — no flash
T+1000ms: Hard timeout — if network hasn't responded, stop waiting,
          show cached data with "Last updated {time}" subtitle
```

#### 6.5 Memory Budget

| Component | Budget | Notes |
|---|---|---|
| People tab (500 contacts) | < 100MB total | Includes FlatList render window, JS objects, image cache in memory |
| Contact list data (JS heap) | < 15MB | 500 contacts x ~30KB JSON each = ~15MB. Parsed lazily (JSON.parse on row render, not on bulk load). |
| Visible row images | < 20MB | ~15 visible rows x 44pt avatar = ~1.3MB. Well under budget. |
| Off-screen buffer | < 10MB | windowSize=11 means ~55 rows buffered (5 screens) x ~180KB per row view tree. |
| SQLite in-memory page cache | < 5MB | Default SQLite page cache, sufficient for FTS queries. |
| Properties tab | < 80MB | Fewer items, larger images. Grid view uses more memory than list (4 images per row vs 1). |
| Property detail (single screen) | < 40MB | Hero gallery (3-5 images at display resolution), chart rendering, comps scroll. |

**Memory monitoring:**

- In development: `Performance` observer tracking JS heap size.
- In production: `expo-application` `getMemoryUsage()` logged to analytics on screen transitions. Alert if People tab exceeds 100MB.
- **Automatic mitigation:** If memory warning received (`MemoryWarning` event from `AppState`), reduce `windowSize` to 5, flush image memory cache (`Image.clearMemoryCache()`), collapse off-screen detail sections.

#### 6.6 Offline-to-Online Transition

| Metric | Target |
|---|---|
| Queue processing start | < 1000ms after network detected |
| Individual mutation sync | < 500ms per item (server-side) |
| Total queue drain (10 pending items) | < 5000ms |
| UI update after sync | < 500ms after data received |
| Conflict detection and prompt | < 2000ms after sync attempt |

**Sequence timing:**

```
T+0ms:      NetInfo fires 'connected' event
T+200ms:    Sync manager starts processing queue (200ms debounce
            to avoid false positives from flaky connections)
T+200-5000ms: Queue items processed sequentially
T+5000ms:   All pending mutations synced (target for 10 items)
T+5200ms:   Fresh data fetched from server (incremental)
T+5500ms:   UI updated with fresh data, offline banner dismissed
T+5500ms:   Green "Back online" banner shown for 2000ms
```

---

*End of V2 CRM Screens specification. All measurements are in logical points (pt) unless otherwise specified. All colors are specified in hex and must support both light and dark mode via system dynamic colors where noted. All animations use React Native Reanimated 3 unless otherwise specified.*

---


## Section 5: Authentication, Security & Data Protection

**Document Version:** 2.0
**Last Updated:** 2026-03-02
**Classification:** Internal -- Engineering
**OWASP Alignment:** Mobile Top 10 (2024 Final Release)
**Apple Compliance Target:** iOS 17+, App Store Review Guidelines 5.1.1, Privacy Manifest (PrivacyInfo.xcprivacy)

---

### Table of Contents

1. Google OAuth Native Flow
2. Token Management
3. API Client Security
4. Data Protection
5. Apple Privacy Compliance
6. Security Threat Model
7. Session Management

---

### 1. Google OAuth Native Flow

#### 1.1 Library Decision: `@react-native-google-signin/google-signin`

**Decision: Use `@react-native-google-signin/google-signin` (v13+).**

| Criterion | `@react-native-google-signin/google-signin` | `expo-auth-session` |
|---|---|---|
| **Sign-in UX** | Native iOS system dialog (credential manager) | Web-based browser modal |
| **Expo recommendation** | Officially recommended for Google auth | Deprecated for Google auth (guide removed) |
| **Token delivery** | Returns server auth code directly | Returns tokens via redirect URI |
| **Keychain integration** | Automatic via Google SDK | Manual |
| **Offline access** | Supports `serverAuthCode` for backend exchange | Requires additional configuration |
| **Expo Go** | Not supported (requires dev build) | Limited support |
| **Web support** | Paid tier only | Built-in |
| **Config plugin** | Yes (`expo-config-plugin` available) | Built-in |

**Justification:** Skye is an iOS-first native app. The native Google Sign-In SDK provides a system-level credential selector that feels integrated with iOS, respects the user's saved Google accounts, and directly produces a `serverAuthCode` suitable for backend exchange. Since Expo has officially deprecated `expo-auth-session` for Google authentication and now recommends `@react-native-google-signin/google-signin`, this is the correct forward-looking choice. The lack of Expo Go support is irrelevant since Skye uses development builds via EAS Build. Web authentication is handled separately by the existing NextAuth implementation and does not need to be unified in this library.

#### 1.2 Complete OAuth Flow Sequence

```
+------------+    +--------------+    +--------------+    +--------------+    +----------+
|  Skye      |    |  Google SDK  |    |  Google      |    |  Skye        |    | Supabase |
|  App       |    |  (Native)    |    |  Servers     |    |  Backend     |    |          |
+-----+------+    +------+-------+    +------+-------+    +------+-------+    +----+-----+
      |                  |                    |                    |                |
      |  1. User taps    |                    |                    |                |
      |  "Sign in with   |                    |                    |                |
      |   Google"        |                    |                    |                |
      |----------------->|                    |                    |                |
      |                  |                    |                    |                |
      |  2. GoogleSignin |                    |                    |                |
      |  .signIn()       |                    |                    |                |
      |                  |  3. Present native |                    |                |
      |                  |  credential picker |                    |                |
      |                  |------------------->|                    |                |
      |                  |                    |                    |                |
      |                  |  4. User selects   |                    |                |
      |                  |  Google account &  |                    |                |
      |                  |  consents          |                    |                |
      |                  |<-------------------|                    |                |
      |                  |                    |                    |                |
      |  5. Return       |                    |                    |                |
      |  { serverAuthCode,                    |                    |                |
      |    idToken,      |                    |                    |                |
      |    user }        |                    |                    |                |
      |<-----------------|                    |                    |                |
      |                  |                    |                    |                |
      |  6. POST /api/auth/native             |                    |                |
      |  { serverAuthCode, platform: "ios" }  |                    |                |
      |------------------------------------------------------>|                |
      |                  |                    |                    |                |
      |                  |                    |  7. Exchange       |                |
      |                  |                    |  serverAuthCode    |                |
      |                  |                    |  for Google tokens |                |
      |                  |                    |<-------------------|                |
      |                  |                    |                    |                |
      |                  |                    |  8. Return         |                |
      |                  |                    |  { access_token,   |                |
      |                  |                    |    id_token,       |                |
      |                  |                    |    refresh_token } |                |
      |                  |                    |------------------->|                |
      |                  |                    |                    |                |
      |                  |                    |  9. Verify         |                |
      |                  |                    |  id_token (Google  |                |
      |                  |                    |  public keys)      |                |
      |                  |                    |  Extract email,    |                |
      |                  |                    |  sub, name         |                |
      |                  |                    |                    |                |
      |                  |                    |  10. Upsert user   |                |
      |                  |                    |  in shared users   |                |
      |                  |                    |  table             |                |
      |                  |                    |                    |--------------->|
      |                  |                    |                    |                |
      |                  |                    |                    |<---------------|
      |                  |                    |                    |                |
      |                  |                    |  11. Generate      |                |
      |                  |                    |  Skye session:     |                |
      |                  |                    |  - access_token    |                |
      |                  |                    |    (JWT, 1hr)      |                |
      |                  |                    |  - refresh_token   |                |
      |                  |                    |    (opaque, 30d)   |                |
      |                  |                    |  - Sign Supabase   |                |
      |                  |                    |    JWT with        |                |
      |                  |                    |    SUPABASE_JWT_   |                |
      |                  |                    |    SECRET          |                |
      |                  |                    |                    |                |
      |  12. Return { accessToken, refreshToken, expiresAt,       |                |
      |               supabaseAccessToken, user }                 |                |
      |<-------------------------------------------------------|                |
      |                  |                    |                    |                |
      |  13. Store in    |                    |                    |                |
      |  SecureStore:    |                    |                    |                |
      |  - accessToken   |                    |                    |                |
      |  - refreshToken  |                    |                    |                |
      |  - supabaseToken |                    |                    |                |
      |  - expiresAt     |                    |                    |                |
      |                  |                    |                    |                |
      |  14. Navigate    |                    |                    |                |
      |  to Home screen  |                    |                    |                |
      v                  v                    v                    v                v
```

#### 1.3 Redirect URI Configuration

**Decision: Neither custom scheme nor Universal Links is required for the primary flow.**

The `@react-native-google-signin/google-signin` library uses the native Google Sign-In SDK, which handles the OAuth flow entirely within the native layer (ASWebAuthenticationSession on iOS or the Credential Manager API). No redirect URI is configured in the app itself; the Google SDK manages the callback internally using the reversed iOS client ID as a URL scheme (e.g., `com.googleusercontent.apps.XXXX`).

However, a custom URL scheme is still registered for other deep linking purposes:

| Purpose | URI | Registration |
|---|---|---|
| Google SDK internal callback | `com.googleusercontent.apps.{IOS_CLIENT_ID}` | Automatic via `GoogleService-Info.plist` |
| App deep links (notifications, etc.) | `com.skyecrm://` | `app.json` > `scheme` |
| Universal Links (future web-to-app) | `https://app.skyecrm.com/.well-known/apple-app-site-association` | AASA file on server |

The deep link scheme `com.skyecrm://` will be validated as described in Section 6.6 (Deep Link Injection).

#### 1.4 Google Cloud Console Setup

Create a **separate iOS OAuth Client ID** in the Google Cloud Console. Do NOT reuse the web client ID.

**Configuration:**

| Setting | Value |
|---|---|
| **Project** | `skye-crm` (same GCP project as web) |
| **Application type** | iOS |
| **Bundle ID** | `com.skyecrm.app` |
| **App Store ID** | (add after first App Store submission) |
| **Team ID** | (Apple Developer Team ID) |

**Required OAuth Scopes:**

| Scope | Purpose |
|---|---|
| `openid` | Required for ID token |
| `email` | User email for account lookup |
| `profile` | User name and avatar |

No additional Google API scopes are required. Skye does not access Google Calendar, Gmail, or Drive. Requesting minimal scopes keeps the consent screen simple and approval fast.

**Consent Screen Configuration:**

| Field | Value |
|---|---|
| App name | Skye CRM |
| User support email | support@skyecrm.com |
| App logo | 120x120 Skye logo |
| Application home page | https://skyecrm.com |
| Application privacy policy | https://skyecrm.com/privacy |
| Application terms of service | https://skyecrm.com/terms |
| Authorized domains | skyecrm.com |
| Publishing status | Production (after verification) |

**Credential Files:**

- Download `GoogleService-Info.plist` for iOS and place it at `ios/GoogleService-Info.plist`.
- Set the `iosClientId` in the `@react-native-google-signin/google-signin` config plugin in `app.json`.
- The **web client ID** (from the existing web OAuth credential) is required as the `webClientId` parameter to enable `serverAuthCode` generation for backend exchange.

```json
{
  "expo": {
    "plugins": [
      [
        "@react-native-google-signin/google-signin",
        {
          "iosUrlScheme": "com.googleusercontent.apps.IOS_CLIENT_ID_HERE"
        }
      ]
    ]
  }
}
```

#### 1.5 Web App Compatibility

The native auth flow and the existing web NextAuth flow MUST coexist without interference. This is achieved through architectural separation at the OAuth layer and convergence at the user and session data layer.

**Architecture:**

```
                    +--------------------------------+
                    |       Google Cloud Console      |
                    |       (Single GCP Project)      |
                    +----------------+---------------+
                    |  Web Client ID | iOS Client ID  |
                    |  (type: web)   | (type: iOS)    |
                    +-------+--------+-------+--------+
                            |                |
                  +---------v------+  +------v---------+
                  |  NextAuth      |  |  Native Google  |
                  |  (Web App)     |  |  Sign-In SDK    |
                  |  Cookie-based  |  |  serverAuthCode |
                  |  sessions      |  |  to backend     |
                  +-------+--------+  +------+---------+
                          |                  |
                          v                  v
                  +----------------------------------+
                  |     Next.js API Routes           |
                  +--------------+-------------------+
                  | /api/auth/*  | /api/auth/native   |
                  | (NextAuth)   | (Native endpoint)  |
                  +------+-------+----------+--------+
                         |                  |
                         v                  v
                  +----------------------------------+
                  |     Shared Supabase Database      |
                  |     +---------------------+      |
                  |     |   users table        |      |
                  |     |   (single source     |      |
                  |     |    of truth)          |      |
                  |     +---------------------+      |
                  |     +---------------------+      |
                  |     |   sessions table     |      |
                  |     |   (web + native)     |      |
                  |     +---------------------+      |
                  |     Row Level Security (RLS)      |
                  +----------------------------------+
```

**Key compatibility guarantees:**

1. **Separate Client IDs:** The web app uses a `Web Application` type OAuth client ID. The iOS app uses an `iOS` type OAuth client ID. Both are in the same GCP project and share the same consent screen.

2. **Shared User Table:** Both flows resolve to the same `users` row. The `/api/auth/native` endpoint looks up users by `google_sub` (Google's stable user identifier) or `email`. If a user signed up via web, their first native sign-in finds the existing record. If a user signs up via native first, their first web sign-in finds the existing record.

3. **Compatible Session Format:** The backend mints a Supabase-compatible JWT for both flows. The JWT payload structure:

```json
{
  "aud": "authenticated",
  "exp": 1709423400,
  "iat": 1709419800,
  "iss": "https://your-project.supabase.co/auth/v1",
  "sub": "user-uuid-from-users-table",
  "email": "agent@example.com",
  "role": "authenticated",
  "app_metadata": {
    "provider": "google",
    "client": "native"
  },
  "user_metadata": {
    "full_name": "Jane Agent",
    "avatar_url": "https://..."
  }
}
```

This JWT is signed with the `SUPABASE_JWT_SECRET` and is accepted by Supabase RLS policies. The `sub` claim matches the user's UUID in the `users` table, so `auth.uid()` in RLS policies resolves correctly regardless of whether the session originated from web or native.

4. **Independent Sessions:** Web sessions are cookie-based (managed by NextAuth). Native sessions are token-based (managed by the native app via SecureStore). A user can be logged into both simultaneously. Revoking one does not revoke the other unless "Log out all devices" is invoked (see Section 7.2).

5. **No NextAuth modification required:** The `/api/auth/native` endpoint is a standalone Next.js API route. It does not import or modify NextAuth configuration. NextAuth continues to operate via `/api/auth/[...nextauth]` exactly as before.

**Backend endpoint specification: `POST /api/auth/native`**

```
Request:
  POST /api/auth/native
  Content-Type: application/json
  {
    "serverAuthCode": "4/0AX4X...",
    "platform": "ios",
    "deviceId": "device-fingerprint-uuid",
    "deviceName": "iPhone 15 Pro"
  }

Response (success):
  200 OK
  {
    "accessToken": "eyJhbG...",          // JWT, 1hr expiry
    "refreshToken": "rt_a8f3c...",       // Opaque, 30-day expiry
    "expiresAt": 1709423400,             // Unix timestamp
    "supabaseAccessToken": "eyJhbG...",  // Supabase-compatible JWT
    "user": {
      "id": "uuid",
      "email": "agent@example.com",
      "name": "Jane Agent",
      "avatarUrl": "https://...",
      "createdAt": "2024-01-15T00:00:00Z"
    }
  }

Response (error):
  401 Unauthorized
  { "error": "invalid_auth_code", "message": "..." }
```

#### 1.6 Error Handling Matrix

Every OAuth failure mode must be handled gracefully. The native app must NEVER show a raw error or crash.

| Error Scenario | Detection Method | User-Facing Behavior | Technical Action |
|---|---|---|---|
| **User cancellation** | `statusCodes.SIGN_IN_CANCELLED` from `GoogleSignin.signIn()` | Silently return to sign-in screen. No error message. | No-op. Log analytics event `auth_cancelled`. |
| **Network error (no internet)** | `statusCodes.IN_PROGRESS` timeout or `fetch` failure | Show inline banner: "No internet connection. Please check your connection and try again." | Check `NetInfo` before initiating. Retry button. |
| **Google server error** | `statusCodes.SIGN_IN_REQUIRED` or HTTP 5xx from Google | Show modal: "Google sign-in is temporarily unavailable. Please try again in a few minutes." | Exponential backoff retry (max 3 attempts). Log to Sentry. |
| **Play Services unavailable** | `statusCodes.PLAY_SERVICES_NOT_AVAILABLE` | iOS: Should not occur (no Play Services). If detected, log as anomaly. | Log to Sentry with device info. |
| **Invalid server auth code** | Backend `/api/auth/native` returns 401 `invalid_auth_code` | Show modal: "Authentication failed. Please try again." | Clear any partial state, redirect to sign-in. Log to Sentry. |
| **Account not authorized** | Backend returns 403 `account_not_authorized` | Show modal: "This Google account is not authorized to use Skye. Contact your administrator." | Do not retry. Show "Use a different account" link. |
| **Backend unreachable** | Fetch timeout (15s) or network error on `/api/auth/native` | Show inline banner: "Unable to connect to Skye. Please try again." | Retry with exponential backoff. Check server status endpoint first. |
| **Duplicate account (email conflict)** | Backend returns 409 `email_conflict` | Show modal: "An account with this email already exists. Please sign in with your existing account." | Offer "Contact support" link. |
| **Rate limited** | Backend returns 429 | Show modal: "Too many sign-in attempts. Please wait a moment." | Respect `Retry-After` header. Disable sign-in button with countdown. |
| **Token storage failure** | `SecureStore.setItemAsync` throws | Show modal: "Unable to save your session securely. Please restart the app." | Log to Sentry with error details. Attempt cleanup and retry once. |

**Implementation pattern (pseudocode):**

```typescript
async function handleGoogleSignIn(): Promise<void> {
  try {
    await GoogleSignin.hasPlayServices();
    const signInResult = await GoogleSignin.signIn();
    const { serverAuthCode } = signInResult.data;

    if (!serverAuthCode) {
      throw new Error('No server auth code received');
    }

    const response = await apiClient.post('/api/auth/native', {
      serverAuthCode,
      platform: 'ios',
      deviceId: await getDeviceId(),
      deviceName: await getDeviceName(),
    });

    await secureTokenStorage.saveTokens({
      accessToken: response.accessToken,
      refreshToken: response.refreshToken,
      supabaseAccessToken: response.supabaseAccessToken,
      expiresAt: response.expiresAt,
    });

    navigation.reset({ index: 0, routes: [{ name: 'Home' }] });

  } catch (error) {
    if (isErrorWithCode(error)) {
      switch (error.code) {
        case statusCodes.SIGN_IN_CANCELLED:
          analytics.track('auth_cancelled');
          return; // Silent return
        case statusCodes.IN_PROGRESS:
          return; // Already signing in
        case statusCodes.PLAY_SERVICES_NOT_AVAILABLE:
          Sentry.captureException(error);
          return;
        default:
          showErrorModal('google_error');
      }
    } else if (isApiError(error)) {
      handleApiAuthError(error);
    } else {
      Sentry.captureException(error);
      showErrorModal('unknown_error');
    }
  }
}
```

---

### 2. Token Management

#### 2.1 Token Types

Skye uses a dual-token architecture with an additional Supabase-specific token for direct database access.

| Token | Format | Lifetime | Purpose | Storage |
|---|---|---|---|---|
| **Access Token** | JWT (signed with app secret) | 1 hour | Authenticates API requests to Skye backend | SecureStore key: `skye_access_token` |
| **Refresh Token** | Opaque string (UUID-based, `rt_` prefix) | 30 days | Obtains new access tokens without re-authentication | SecureStore key: `skye_refresh_token` |
| **Supabase Access Token** | JWT (signed with `SUPABASE_JWT_SECRET`) | 1 hour (rotated alongside access token) | Direct Supabase queries with RLS enforcement | SecureStore key: `skye_supabase_token` |
| **Token Expiry** | Unix timestamp (number) | N/A | Client-side expiry check | SecureStore key: `skye_token_expires_at` |

**Access Token JWT Claims:**

```json
{
  "sub": "user-uuid",
  "email": "agent@example.com",
  "iat": 1709419800,
  "exp": 1709423400,
  "iss": "skye-api",
  "aud": "skye-native",
  "jti": "unique-token-id",
  "session_id": "session-uuid",
  "client": "native-ios"
}
```

**Refresh Token Server-Side Record:**

```sql
CREATE TABLE refresh_tokens (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  token_hash  TEXT NOT NULL UNIQUE,  -- SHA-256 hash, never store plaintext
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  session_id  UUID NOT NULL REFERENCES sessions(id) ON DELETE CASCADE,
  device_id   TEXT NOT NULL,
  expires_at  TIMESTAMPTZ NOT NULL,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  revoked_at  TIMESTAMPTZ,          -- Non-null = revoked
  replaced_by UUID REFERENCES refresh_tokens(id) -- Token rotation chain
);
```

#### 2.2 Secure Storage

**Mandatory: `expo-secure-store` (iOS Keychain). NEVER use AsyncStorage, MMKV, or in-memory storage for tokens.**

**Configuration:**

```typescript
import * as SecureStore from 'expo-secure-store';

const SECURE_STORE_OPTIONS: SecureStore.SecureStoreOptions = {
  keychainAccessible: SecureStore.AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY,
  // AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY:
  //   - Accessible after first unlock (supports background refresh)
  //   - THIS_DEVICE_ONLY prevents iCloud Keychain sync (tokens stay on-device)
  //   - NOT available before first unlock (acceptable trade-off)
  //
  // We do NOT use WHEN_UNLOCKED because:
  //   - It prevents background token refresh (e.g., push notification handling)
  //
  // We do NOT use ALWAYS because:
  //   - It is deprecated by Apple and is the least secure option
};

// Key constants
const KEYS = {
  ACCESS_TOKEN: 'skye_access_token',
  REFRESH_TOKEN: 'skye_refresh_token',
  SUPABASE_TOKEN: 'skye_supabase_token',
  TOKEN_EXPIRES_AT: 'skye_token_expires_at',
  BIOMETRIC_ENABLED: 'skye_biometric_enabled',
} as const;

// Storage wrapper
export const secureTokenStorage = {
  async saveTokens(tokens: TokenResponse): Promise<void> {
    await Promise.all([
      SecureStore.setItemAsync(KEYS.ACCESS_TOKEN, tokens.accessToken, SECURE_STORE_OPTIONS),
      SecureStore.setItemAsync(KEYS.REFRESH_TOKEN, tokens.refreshToken, SECURE_STORE_OPTIONS),
      SecureStore.setItemAsync(KEYS.SUPABASE_TOKEN, tokens.supabaseAccessToken, SECURE_STORE_OPTIONS),
      SecureStore.setItemAsync(KEYS.TOKEN_EXPIRES_AT, String(tokens.expiresAt), SECURE_STORE_OPTIONS),
    ]);
  },

  async getAccessToken(): Promise<string | null> {
    return SecureStore.getItemAsync(KEYS.ACCESS_TOKEN, SECURE_STORE_OPTIONS);
  },

  async isTokenExpired(): Promise<boolean> {
    const expiresAt = await SecureStore.getItemAsync(KEYS.TOKEN_EXPIRES_AT, SECURE_STORE_OPTIONS);
    if (!expiresAt) return true;
    // Add 60-second buffer to avoid edge-case expiry during request
    return Date.now() / 1000 >= Number(expiresAt) - 60;
  },

  async clearAll(): Promise<void> {
    await Promise.all(
      Object.values(KEYS).map(key => SecureStore.deleteItemAsync(key, SECURE_STORE_OPTIONS))
    );
  },
};
```

**Size constraint:** SecureStore values are limited to approximately 2048 bytes on some iOS versions. JWTs must be kept compact. If the access token exceeds 1500 bytes, audit the claims for unnecessary data.

**Persistence behavior:** iOS Keychain data may persist after app uninstall if the app is reinstalled with the same bundle ID. On reinstall, check for orphaned tokens and validate them against the server before use. If the server rejects them, clear them silently.

#### 2.3 Token Refresh Flow

```
+-----------------------------------------------------------------------------+
|                          TOKEN REFRESH STATE MACHINE                         |
+-----------------------------------------------------------------------------+
|                                                                              |
|   +----------+     token valid      +--------------+                         |
|   |  IDLE    | ------------------> |  MAKE API     |                         |
|   |          |                      |  REQUEST      |                         |
|   +----+-----+                      +------+--------+                         |
|        |                                   |                                  |
|        | token expired                     | 200 OK                           |
|        | (checked before                   v                                  |
|        |  every request)          +--------------+                            |
|        |                          |  RETURN       |                            |
|        |                          |  RESPONSE     |                            |
|        v                          +--------------+                            |
|   +----------+                                                                |
|   | ACQUIRE  |                                                                |
|   | LOCK     | <---- Other requests wait here                                 |
|   +----+-----+                                                                |
|        |                                                                      |
|        | lock acquired                                                        |
|        v                                                                      |
|   +--------------+     +--------------+                                      |
|   | CHECK AGAIN  |---->| TOKEN NOW    | (another request refreshed it)        |
|   | (double-     |     | VALID        |---------> MAKE API REQUEST            |
|   |  check)      |     +--------------+                                      |
|   +----+---------+                                                            |
|        |                                                                      |
|        | still expired                                                        |
|        v                                                                      |
|   +--------------+                                                            |
|   | POST         |                                                            |
|   | /api/auth/   |                                                            |
|   | refresh      |                                                            |
|   +----+----+----+                                                            |
|        |    |                                                                 |
|   200 OK    | 401 (refresh token                                              |
|        |    |  revoked/expired)                                                |
|        v    |                                                                 |
|   +---------+  |   +-------------+    +--------------+                       |
|   | SAVE    |  +-->| CLEAR       |--->| NAVIGATE TO  |                       |
|   | NEW     |      | ALL TOKENS  |    | AUTH SCREEN  |                       |
|   | TOKENS  |      +-------------+    +--------------+                       |
|   +----+----+                                                                 |
|        |                                                                      |
|        | release lock                                                         |
|        v                                                                      |
|   +--------------+                                                            |
|   | RETRY        |                                                            |
|   | ORIGINAL     |                                                            |
|   | REQUEST      |                                                            |
|   +--------------+                                                            |
|                                                                              |
+-----------------------------------------------------------------------------+
```

**Refresh endpoint specification:**

```
Request:
  POST /api/auth/refresh
  Content-Type: application/json
  {
    "refreshToken": "rt_a8f3c..."
  }

Response (success):
  200 OK
  {
    "accessToken": "eyJhbG...",           // New JWT
    "refreshToken": "rt_b9d4e...",        // NEW refresh token (old one invalidated)
    "supabaseAccessToken": "eyJhbG...",   // New Supabase JWT
    "expiresAt": 1709427000
  }

Response (failure):
  401 Unauthorized
  { "error": "refresh_token_revoked" }
```

#### 2.4 Race Condition Handling: Token Refresh Mutex

When multiple API calls fire simultaneously and the token is expired, only ONE refresh request must execute. All others must wait for its result.

```typescript
class TokenRefreshManager {
  private refreshPromise: Promise<TokenResponse> | null = null;
  private mutex = new Mutex(); // Use 'async-mutex' package

  async getValidAccessToken(): Promise<string> {
    const isExpired = await secureTokenStorage.isTokenExpired();

    if (!isExpired) {
      return (await secureTokenStorage.getAccessToken())!;
    }

    // Acquire mutex: only one refresh at a time
    const release = await this.mutex.acquire();

    try {
      // Double-check after acquiring lock (another caller may have refreshed)
      const isStillExpired = await secureTokenStorage.isTokenExpired();
      if (!isStillExpired) {
        return (await secureTokenStorage.getAccessToken())!;
      }

      // Perform the refresh
      const refreshToken = await secureTokenStorage.getRefreshToken();
      if (!refreshToken) {
        throw new AuthExpiredError('No refresh token available');
      }

      const response = await fetch(`${API_BASE_URL}/api/auth/refresh`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refreshToken }),
      });

      if (!response.ok) {
        if (response.status === 401) {
          await secureTokenStorage.clearAll();
          throw new AuthExpiredError('Refresh token revoked or expired');
        }
        throw new NetworkError(`Refresh failed: ${response.status}`);
      }

      const tokens: TokenResponse = await response.json();
      await secureTokenStorage.saveTokens(tokens);

      return tokens.accessToken;

    } finally {
      release(); // Always release the mutex
    }
  }
}

// Singleton instance
export const tokenManager = new TokenRefreshManager();
```

**Dependencies:** `async-mutex` (npm package, ~2KB, no native code, well-maintained).

#### 2.5 Token Rotation

On every refresh, the server performs the following atomically (within a database transaction):

1. Look up the refresh token by its SHA-256 hash.
2. Verify it is not revoked (`revoked_at IS NULL`) and not expired (`expires_at > now()`).
3. Generate a new refresh token.
4. Mark the old refresh token as revoked (`revoked_at = now()`) and set `replaced_by` to the new token's ID.
5. Insert the new refresh token record.
6. Generate new access token and Supabase access token.
7. Return all new tokens to the client.

**Reuse detection:** If a revoked refresh token is presented, it indicates potential token theft. The server must:

1. Walk the `replaced_by` chain to find the most recent token in the family.
2. Revoke the ENTIRE token family (all tokens in the chain).
3. Return 401 with `error: "token_family_revoked"`.
4. Send a security alert email to the user.
5. Log the incident to the security audit log.

This prevents stolen refresh tokens from being used even once after the legitimate client has already refreshed.

#### 2.6 Logout Flow

Logout must be thorough. Partial logout is a security vulnerability.

```typescript
async function logout(): Promise<void> {
  try {
    // 1. Revoke refresh token on server (best-effort)
    const refreshToken = await secureTokenStorage.getRefreshToken();
    if (refreshToken) {
      await fetch(`${API_BASE_URL}/api/auth/logout`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${await secureTokenStorage.getAccessToken()}`,
        },
        body: JSON.stringify({ refreshToken }),
      }).catch(() => {
        // Server revocation failed (offline, etc.)
        // Continue with local cleanup -- token will expire naturally
        // Server should also clean expired tokens via cron
      });
    }

    // 2. Clear SecureStore (all tokens)
    await secureTokenStorage.clearAll();

    // 3. Clear offline database
    await offlineDatabase.deleteAllData();

    // 4. Clear image cache
    await ImageCache.clearAll();

    // 5. Clear any in-memory caches (React Query, Zustand, etc.)
    queryClient.clear();
    useAuthStore.getState().reset();

    // 6. Clear Supabase client session
    await supabaseClient.auth.signOut();

    // 7. Sign out from Google (clears cached Google credential)
    await GoogleSignin.signOut();

    // 8. Navigate to auth screen (reset navigation stack)
    navigation.reset({
      index: 0,
      routes: [{ name: 'Auth' }],
    });

  } catch (error) {
    // Even if steps fail, ensure local state is cleared
    Sentry.captureException(error);
    await secureTokenStorage.clearAll();
    navigation.reset({ index: 0, routes: [{ name: 'Auth' }] });
  }
}
```

#### 2.7 Biometric Re-Authentication

Optional Face ID / Touch ID to unlock the app after it has been in the background. This is a user-configurable setting (disabled by default).

**When triggered:**

- App returns from background after more than 5 minutes (configurable).
- User opens the Settings > Security screen.
- Before executing high-sensitivity actions (optional, e.g., deleting a deal).

**Implementation:**

```typescript
import * as LocalAuthentication from 'expo-local-authentication';

async function biometricGate(): Promise<boolean> {
  const biometricEnabled = await SecureStore.getItemAsync('skye_biometric_enabled');
  if (biometricEnabled !== 'true') return true; // Not enabled, pass through

  const hasHardware = await LocalAuthentication.hasHardwareAsync();
  const isEnrolled = await LocalAuthentication.isEnrolledAsync();

  if (!hasHardware || !isEnrolled) {
    // Hardware unavailable or no biometrics enrolled
    // Fall back to allowing access (don't lock the user out)
    return true;
  }

  const result = await LocalAuthentication.authenticateAsync({
    promptMessage: 'Unlock Skye',
    fallbackLabel: 'Use Passcode',
    disableDeviceFallback: false, // Allow device passcode as fallback
    cancelLabel: 'Cancel',
  });

  if (result.success) {
    return true;
  }

  // Authentication failed or cancelled
  // User stays on the lock screen
  return false;
}
```

**Required `app.json` configuration for Face ID:**

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSFaceIDUsageDescription": "Skye uses Face ID to secure your account and protect sensitive client data."
      }
    }
  }
}
```

**App lifecycle integration:**

```typescript
const LOCK_TIMEOUT_MS = 5 * 60 * 1000; // 5 minutes
let backgroundTimestamp: number | null = null;

AppState.addEventListener('change', (nextState) => {
  if (nextState === 'background') {
    backgroundTimestamp = Date.now();
  } else if (nextState === 'active' && backgroundTimestamp) {
    const elapsed = Date.now() - backgroundTimestamp;
    backgroundTimestamp = null;
    if (elapsed > LOCK_TIMEOUT_MS) {
      showBiometricLockScreen();
    }
  }
});
```

---

### 3. API Client Security

#### 3.1 Base API Client Architecture

All HTTP communication flows through a single, centralized API client. No component or hook may make direct `fetch` calls to the backend.

```typescript
import axios, { AxiosInstance, AxiosRequestConfig, AxiosError } from 'axios';

const API_BASE_URL = __DEV__
  ? 'https://dev-api.skyecrm.com'
  : 'https://api.skyecrm.com';

const apiClient: AxiosInstance = axios.create({
  baseURL: API_BASE_URL,
  timeout: 15_000, // 15s default
  headers: {
    'Content-Type': 'application/json',
    'X-Client-Platform': 'ios',
    'X-Client-Version': Constants.expoConfig?.version ?? 'unknown',
  },
});
```

#### 3.2 Auth Header Injection

An Axios request interceptor injects the access token on every request. This is the ONLY place where the access token is read and attached.

```typescript
apiClient.interceptors.request.use(async (config) => {
  // Skip auth header for auth endpoints
  if (config.url?.startsWith('/api/auth/')) {
    return config;
  }

  try {
    const accessToken = await tokenManager.getValidAccessToken();
    config.headers.Authorization = `Bearer ${accessToken}`;
  } catch (error) {
    if (error instanceof AuthExpiredError) {
      // Trigger logout flow from interceptor
      authEventEmitter.emit('session_expired');
      return Promise.reject(error);
    }
    throw error;
  }

  return config;
});
```

**Response interceptor for 401 handling:**

```typescript
apiClient.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as AxiosRequestConfig & { _retry?: boolean };

    // If 401 and not already retried, attempt token refresh
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const newToken = await tokenManager.getValidAccessToken();
        originalRequest.headers = {
          ...originalRequest.headers,
          Authorization: `Bearer ${newToken}`,
        };
        return apiClient(originalRequest);
      } catch (refreshError) {
        authEventEmitter.emit('session_expired');
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);
```

#### 3.3 Certificate Pinning

**Decision: Use `react-native-ssl-public-key-pinning` for JavaScript-level pinning, backed by TrustKit on iOS and OkHttp CertificatePinner on Android.**

This library patches the standard React Native networking layer at the native level, so all `fetch` and `axios` requests are automatically covered without changing any JS code after initialization.

**Justification:** Public key pinning (rather than certificate pinning) is preferred because public keys survive certificate renewal. When the server certificate is rotated, the new certificate typically reuses the same public key, avoiding app breakage.

**Setup:**

```typescript
import { initializeSslPinning } from 'react-native-ssl-public-key-pinning';

// Call ONCE at app startup, before any API calls
await initializeSslPinning({
  'api.skyecrm.com': {
    includeSubdomains: true,
    publicKeyHashes: [
      'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=', // Current leaf cert SHA-256
      'BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=', // Backup intermediate cert SHA-256
    ],
  },
});
```

**Key rotation procedure:**

1. Before the current certificate expires, generate the new certificate with a new key pair.
2. Extract the new public key hash using OpenSSL:
   ```bash
   openssl x509 -in new_cert.pem -pubkey -noout | \
   openssl pkey -pubin -outform DER | \
   openssl dgst -sha256 -binary | base64
   ```
3. Add the new hash as a third pin in the next app release (keep both old pins).
4. After the app release has been adopted by >95% of users, rotate the server certificate.
5. In the following release, remove the old leaf cert hash (keep the intermediate as backup).

**Development builds:** Pinning MUST be disabled in `__DEV__` mode to allow proxy tools (Charles, Proxyman) for debugging.

```typescript
if (!__DEV__) {
  await initializeSslPinning({ /* ... */ });
}
```

#### 3.4 Request Signing (HMAC) for Sensitive Endpoints

For high-sensitivity mutations (contact creation, deal modification, financial data updates), requests are signed with HMAC-SHA256 to prevent tampering even if the TLS layer is compromised.

**Implementation:**

```typescript
import { HmacSHA256 } from 'crypto-js';

function signRequest(method: string, path: string, body: string, timestamp: number): string {
  // The signing key is derived from the access token (not stored separately)
  // This means the signature is only valid for the current session
  const signingKey = accessToken.substring(0, 32); // Derived from JWT
  const payload = `${method}\n${path}\n${timestamp}\n${body}`;
  return HmacSHA256(payload, signingKey).toString();
}

// Applied via interceptor to specific endpoints
const SIGNED_ENDPOINTS = [
  '/api/contacts',
  '/api/deals',
  '/api/properties',
  '/api/user/delete',
];

apiClient.interceptors.request.use(async (config) => {
  const isSignedEndpoint = SIGNED_ENDPOINTS.some(ep =>
    config.url?.startsWith(ep) && config.method !== 'get'
  );

  if (isSignedEndpoint && config.data) {
    const timestamp = Math.floor(Date.now() / 1000);
    const signature = signRequest(
      config.method!.toUpperCase(),
      config.url!,
      JSON.stringify(config.data),
      timestamp,
    );
    config.headers['X-Request-Timestamp'] = timestamp;
    config.headers['X-Request-Signature'] = signature;
  }

  return config;
});
```

**Server-side validation:** The backend recomputes the HMAC and compares. Requests with a timestamp older than 5 minutes are rejected (replay protection).

#### 3.5 Rate Limiting Awareness

```typescript
apiClient.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    if (error.response?.status === 429) {
      const retryAfter = error.response.headers['retry-after'];
      const waitMs = retryAfter
        ? parseInt(retryAfter, 10) * 1000
        : 5000; // Default 5s

      // Surface to UI
      rateLimitEventEmitter.emit('rate_limited', { waitMs });

      // Auto-retry after wait (for background requests only)
      if (error.config?.headers?.['X-Background-Request']) {
        await delay(waitMs);
        return apiClient(error.config);
      }

      return Promise.reject(new RateLimitError(waitMs));
    }

    return Promise.reject(error);
  }
);
```

#### 3.6 Request Timeouts

| Request Category | Timeout | Justification |
|---|---|---|
| Standard API calls | 15 seconds | Adequate for typical REST operations over cellular |
| AI chat streaming | 30 seconds (initial response) | LLM inference can take time; streaming begins within 30s |
| OCR image upload | 60 seconds | Image upload + server-side processing |
| File downloads (documents) | 60 seconds | Large PDF/document downloads |
| Token refresh | 10 seconds | Should be fast; fail quickly to surface issues |
| Health check / ping | 5 seconds | Lightweight; long timeout suggests server issues |

**Implementation:**

```typescript
// Per-request timeout overrides
export const chatApi = {
  sendMessage: (payload: ChatPayload) =>
    apiClient.post('/api/chat', payload, { timeout: 30_000 }),
};

export const ocrApi = {
  uploadImage: (formData: FormData) =>
    apiClient.post('/api/ocr/analyze', formData, {
      timeout: 60_000,
      headers: { 'Content-Type': 'multipart/form-data' },
    }),
};
```

#### 3.7 No Sensitive Data in URLs

**Rule: ALL sensitive parameters MUST be in the request body, NEVER in query strings.**

Query strings appear in:
- Server access logs
- CDN logs
- Browser history (if WebView is used)
- iOS `NSURLSession` logs
- Proxy tool logs
- Crash reports

**Prohibited patterns:**

```
BAD:  GET /api/contacts?phone=+15551234567
BAD:  GET /api/search?email=jane@example.com
BAD:  GET /api/deals?ssn=123-45-6789
```

**Required patterns:**

```
GOOD: POST /api/contacts/search   { "phone": "+15551234567" }
GOOD: POST /api/search            { "email": "jane@example.com" }
```

The only query parameters permitted are:
- Pagination: `?page=2&limit=20`
- Sorting: `?sort=created_at&order=desc`
- Non-sensitive filters: `?status=active&type=buyer`
- Resource IDs in path segments: `/api/contacts/uuid-here`

---

### 4. Data Protection

#### 4.1 Sensitive Data Classification

| Tier | Classification | Examples | Storage | Encryption |
|---|---|---|---|---|
| **Tier 1: Secrets** | Authentication credentials, API keys | Access tokens, refresh tokens, Supabase JWT, encryption keys | iOS Keychain only (via `expo-secure-store`) | Hardware-backed (Secure Enclave) |
| **Tier 2: PII** | Personally identifiable information of contacts and deals | Contact names, phone numbers, emails, addresses, deal amounts, property details, notes | Encrypted SQLite (SQLCipher) | AES-256 at rest |
| **Tier 3: App State** | Non-sensitive application state | UI preferences, theme, last viewed screen, feature flags, cached non-PII | AsyncStorage or MMKV | None required |

**Data flow principle:** Tier 1 data NEVER touches Tier 2 or Tier 3 storage. Tier 2 data NEVER downgrades to Tier 3 storage. Violations of this principle are treated as P0 security bugs.

#### 4.2 SQLite Encryption (SQLCipher)

The offline cache stores contact PII, deal details, and property information for offline access. This data MUST be encrypted at rest.

**Configuration:**

```json
// app.json
{
  "expo": {
    "plugins": [
      ["expo-sqlite", { "useSQLCipher": true, "enableFTS": true }]
    ]
  }
}
```

**Database initialization:**

```typescript
import * as SQLite from 'expo-sqlite';
import * as SecureStore from 'expo-secure-store';
import * as Crypto from 'expo-crypto';

const DB_KEY_STORAGE_KEY = 'skye_db_encryption_key';

async function getOrCreateDbKey(): Promise<string> {
  let key = await SecureStore.getItemAsync(DB_KEY_STORAGE_KEY, {
    keychainAccessible: SecureStore.AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY,
  });

  if (!key) {
    // Generate a 256-bit random key, encoded as hex
    const randomBytes = await Crypto.getRandomBytesAsync(32);
    key = Array.from(randomBytes)
      .map(b => b.toString(16).padStart(2, '0'))
      .join('');

    await SecureStore.setItemAsync(DB_KEY_STORAGE_KEY, key, {
      keychainAccessible: SecureStore.AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY,
    });
  }

  return key;
}

async function openDatabase(): Promise<SQLite.SQLiteDatabase> {
  const db = await SQLite.openDatabaseAsync('skye_offline.db');
  const dbKey = await getOrCreateDbKey();

  // Set SQLCipher encryption key immediately after opening
  await db.execAsync(`PRAGMA key = '${dbKey}'`);

  // Verify encryption is working
  await db.execAsync('SELECT count(*) FROM sqlite_master');

  // Performance tuning for encrypted database
  await db.execAsync('PRAGMA journal_mode = WAL');
  await db.execAsync('PRAGMA synchronous = NORMAL');

  return db;
}
```

**The database encryption key is stored in Tier 1 storage (Keychain) and is NEVER logged, transmitted, or exposed to JavaScript beyond the initialization call.**

#### 4.3 No Secrets in Bundle

**Rule: Zero API keys, zero client secrets, zero server URLs with credentials in the JavaScript bundle.**

What goes in the bundle (compile-time constants):
- API base URL (https://api.skyecrm.com) -- this is public.
- Google OAuth iOS client ID -- this is public (embedded in `GoogleService-Info.plist` by design).
- Sentry DSN -- this is public by design.
- App version and build number.

What NEVER goes in the bundle:
- `SUPABASE_JWT_SECRET` -- server-side only.
- `SUPABASE_SERVICE_ROLE_KEY` -- server-side only.
- Google OAuth client secret -- server-side only (the iOS client type does not have a client secret; the server uses the web client secret).
- Any database connection strings.
- Any third-party API keys (OpenAI, Twilio, etc.) -- server-side only.
- HMAC signing secrets -- derived from session tokens, not stored.

**Enforcement:**

1. Add a pre-commit hook that scans for common secret patterns (`sk_live_`, `SUPABASE_SERVICE`, `-----BEGIN PRIVATE KEY-----`, etc.) using `detect-secrets` or `gitleaks`.
2. Add an EAS Build check that scans the final JavaScript bundle for known secret patterns.
3. Environment variables in `app.json` / `app.config.ts` must be limited to the `EXPO_PUBLIC_` prefix, and these must only contain non-secret values.

#### 4.4 Screenshot Protection

When the app enters the background, iOS captures a snapshot for the app switcher. This snapshot can expose sensitive data (contact details, deal amounts).

**Decision: Use `expo-screen-capture` (Expo SDK built-in) for primary protection, supplemented by a native iOS lifecycle hook.**

```typescript
import * as ScreenCapture from 'expo-screen-capture';

// Activate privacy screen on app mount
useEffect(() => {
  if (!__DEV__) {
    // Prevents screenshots and screen recording
    ScreenCapture.preventScreenCaptureAsync('skye-privacy');

    // Add blur overlay for app switcher
    ScreenCapture.addScreenshotListener(() => {
      // Optional: log that a screenshot was attempted
      analytics.track('screenshot_attempted');
    });
  }

  return () => {
    ScreenCapture.allowScreenCaptureAsync('skye-privacy');
  };
}, []);
```

**Native iOS supplementary protection** (in `AppDelegate.mm`):

```objc
// In AppDelegate.mm
- (void)applicationWillResignActive:(UIApplication *)application {
  // Add a blur view when entering background
  UIBlurEffect *blurEffect = [UIBlurEffect effectWithStyle:UIBlurEffectStyleLight];
  UIVisualEffectView *blurView = [[UIVisualEffectView alloc] initWithEffect:blurEffect];
  blurView.frame = self.window.bounds;
  blurView.tag = 9999; // Tag for removal
  [self.window addSubview:blurView];
}

- (void)applicationDidBecomeActive:(UIApplication *)application {
  // Remove blur view when returning to foreground
  UIView *blurView = [self.window viewWithTag:9999];
  [blurView removeFromSuperview];
}
```

This is added via a custom Expo config plugin to avoid modifying `AppDelegate.mm` directly.

#### 4.5 Clipboard Security

When a user copies sensitive data (phone numbers, email addresses), the clipboard should be cleared after 60 seconds to prevent accidental paste into other apps.

```typescript
import * as Clipboard from 'expo-clipboard';

const CLIPBOARD_CLEAR_DELAY_MS = 60_000; // 60 seconds
let clipboardTimer: ReturnType<typeof setTimeout> | null = null;

export async function copyWithAutoExpiry(text: string, label?: string): Promise<void> {
  await Clipboard.setStringAsync(text);

  // Show brief toast: "Copied! Clipboard will clear in 60 seconds."
  showToast(`${label ?? 'Text'} copied. Clipboard clears in 60s.`);

  // Clear any existing timer
  if (clipboardTimer) clearTimeout(clipboardTimer);

  clipboardTimer = setTimeout(async () => {
    // Only clear if clipboard still contains our text
    const current = await Clipboard.getStringAsync();
    if (current === text) {
      await Clipboard.setStringAsync('');
    }
    clipboardTimer = null;
  }, CLIPBOARD_CLEAR_DELAY_MS);
}
```

**Usage:** All "copy" actions for contact phone numbers, emails, addresses, and deal IDs must use `copyWithAutoExpiry` instead of direct `Clipboard.setStringAsync`.

#### 4.6 Debug / Release Build Differences

| Feature | Debug (`__DEV__`) | Release |
|---|---|---|
| Console logging | Enabled | **Stripped** (via `babel-plugin-transform-remove-console`) |
| Flipper | Enabled | **Removed** (not included in production Podfile) |
| React DevTools | Enabled | **Disabled** |
| Network inspector | Enabled | **Disabled** |
| Certificate pinning | **Disabled** (allow proxy tools) | **Enabled** |
| Screenshot protection | **Disabled** (for testing screenshots) | **Enabled** |
| Hermes source maps | Available locally | **NOT uploaded to App Store** (uploaded to Sentry only) |
| Performance monitors | Enabled | **Disabled** |
| Error boundaries | Show error details | Show generic "Something went wrong" |
| ATS exceptions | localhost allowed | **No exceptions** |

**Babel configuration for console stripping:**

```javascript
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  const plugins = ['nativewind/babel'];

  if (process.env.NODE_ENV === 'production') {
    plugins.push('transform-remove-console');
  }

  return {
    presets: ['babel-preset-expo'],
    plugins,
  };
};
```

---

### 5. Apple Privacy Compliance

#### 5.1 Privacy Manifest (`PrivacyInfo.xcprivacy`)

The Privacy Manifest is required for App Store submission. It declares the Required Reason APIs used by the app and all third-party SDKs.

**Exact file contents for Skye:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>

  <!-- SECTION 1: Required Reason APIs -->
  <key>NSPrivacyAccessedAPITypes</key>
  <array>

    <!-- UserDefaults: Used by React Native, Expo, and app for preferences -->
    <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array>
        <string>CA92.1</string>
        <!-- CA92.1: Access UserDefaults to read/write app-specific
             configuration and state (e.g., onboarding completion,
             theme preference, feature flags) -->
      </array>
    </dict>

    <!-- File Timestamp: Used by expo-file-system, expo-updates, and
         SQLite WAL for file metadata -->
    <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array>
        <string>C617.1</string>
        <!-- C617.1: Access file timestamps of files within the
             app container for cache management and update checks -->
      </array>
    </dict>

    <!-- System Boot Time: Used by Hermes engine for performance timing
         and expo-updates for scheduling -->
    <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategorySystemBootTime</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array>
        <string>35F9.1</string>
        <!-- 35F9.1: Access system boot time for measuring elapsed time
             intervals within the app (performance monitoring) -->
      </array>
    </dict>

    <!-- Disk Space: Used by expo-file-system for cache management
         and image download pre-checks -->
    <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategoryDiskSpace</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array>
        <string>E174.1</string>
        <!-- E174.1: Access disk space to check available capacity
             before downloading large files (property images, documents) -->
      </array>
    </dict>

  </array>

  <!-- SECTION 2: Tracking Domains -->
  <key>NSPrivacyTrackingDomains</key>
  <array>
    <!-- Skye does NOT use any tracking domains.
         No advertising SDKs, no cross-app tracking.
         This array is intentionally empty. -->
  </array>

  <!-- SECTION 3: Tracking Declaration -->
  <key>NSPrivacyTracking</key>
  <false/>
  <!-- Skye does NOT track users across apps or websites.
       No IDFA access, no fingerprinting, no ad networks. -->

  <!-- SECTION 4: Collected Data Types -->
  <key>NSPrivacyCollectedDataTypes</key>
  <array>

    <!-- Email Address -->
    <dict>
      <key>NSPrivacyCollectedDataType</key>
      <string>NSPrivacyCollectedDataTypeEmailAddress</string>
      <key>NSPrivacyCollectedDataTypeLinked</key>
      <true/>
      <key>NSPrivacyCollectedDataTypeTracking</key>
      <false/>
      <key>NSPrivacyCollectedDataTypePurposes</key>
      <array>
        <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
      </array>
    </dict>

    <!-- Name -->
    <dict>
      <key>NSPrivacyCollectedDataType</key>
      <string>NSPrivacyCollectedDataTypeName</string>
      <key>NSPrivacyCollectedDataTypeLinked</key>
      <true/>
      <key>NSPrivacyCollectedDataTypeTracking</key>
      <false/>
      <key>NSPrivacyCollectedDataTypePurposes</key>
      <array>
        <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
      </array>
    </dict>

    <!-- User ID (Google sub, internal UUID) -->
    <dict>
      <key>NSPrivacyCollectedDataType</key>
      <string>NSPrivacyCollectedDataTypeUserID</string>
      <key>NSPrivacyCollectedDataTypeLinked</key>
      <true/>
      <key>NSPrivacyCollectedDataTypeTracking</key>
      <false/>
      <key>NSPrivacyCollectedDataTypePurposes</key>
      <array>
        <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
      </array>
    </dict>

    <!-- Photos (property images uploaded by user) -->
    <dict>
      <key>NSPrivacyCollectedDataType</key>
      <string>NSPrivacyCollectedDataTypePhotos</string>
      <key>NSPrivacyCollectedDataTypeLinked</key>
      <true/>
      <key>NSPrivacyCollectedDataTypeTracking</key>
      <false/>
      <key>NSPrivacyCollectedDataTypePurposes</key>
      <array>
        <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
      </array>
    </dict>

    <!-- Crash Data (via Sentry) -->
    <dict>
      <key>NSPrivacyCollectedDataType</key>
      <string>NSPrivacyCollectedDataTypeCrashData</string>
      <key>NSPrivacyCollectedDataTypeLinked</key>
      <false/>
      <key>NSPrivacyCollectedDataTypeTracking</key>
      <false/>
      <key>NSPrivacyCollectedDataTypePurposes</key>
      <array>
        <string>NSPrivacyCollectedDataTypePurposeAnalytics</string>
      </array>
    </dict>

    <!-- Performance Data (via Sentry) -->
    <dict>
      <key>NSPrivacyCollectedDataType</key>
      <string>NSPrivacyCollectedDataTypePerformanceData</string>
      <key>NSPrivacyCollectedDataTypeLinked</key>
      <false/>
      <key>NSPrivacyCollectedDataTypeTracking</key>
      <false/>
      <key>NSPrivacyCollectedDataTypePurposes</key>
      <array>
        <string>NSPrivacyCollectedDataTypePurposeAnalytics</string>
      </array>
    </dict>

  </array>

</dict>
</plist>
```

**This file is placed at `ios/PrivacyInfo.xcprivacy` and added to the Xcode project via an Expo config plugin.**

#### 5.2 App Tracking Transparency (ATT) Analysis

**Decision: ATT prompt is NOT required for Skye. Do NOT include the ATT framework.**

**Detailed analysis of every SDK in the Skye stack:**

| SDK / Library | Accesses IDFA? | Cross-app tracking? | Tracking domain? | ATT trigger? |
|---|---|---|---|---|
| `@react-native-google-signin/google-signin` | No | No | No | No |
| `expo-secure-store` | No | No | No | No |
| `expo-sqlite` (SQLCipher) | No | No | No | No |
| `expo-local-authentication` | No | No | No | No |
| `expo-screen-capture` | No | No | No | No |
| `expo-notifications` | No | No | No | No |
| `@sentry/react-native` | No | No (crash/perf only) | No (`sentry.io` is not a tracking domain) | No |
| `react-native-ssl-public-key-pinning` | No | No | No | No |
| `axios` | No (pure JS) | No | No | No |
| `@supabase/supabase-js` | No (pure JS) | No | No | No |
| `@tanstack/react-query` | No (pure JS) | No | No | No |
| `zustand` / `jotai` | No (pure JS) | No | No | No |

**Conclusion:** Skye does not access the IDFA, does not perform cross-app or cross-site tracking, does not use advertising SDKs, and does not share data with data brokers. The ATT prompt is not required and MUST NOT be included (showing it unnecessarily degrades user experience and may raise App Review questions).

**`NSUserTrackingUsageDescription` is NOT included in `Info.plist`.** Including it without a valid reason may trigger App Review flags.

#### 5.3 Privacy Nutrition Labels (App Store Connect)

When submitting to the App Store, the following entries must be declared in App Store Connect under "App Privacy":

**Data Linked to You:**

| Data Type | Category | Purpose | Linked to Identity | Used for Tracking |
|---|---|---|---|---|
| Name | Contact Info | App Functionality | Yes | No |
| Email Address | Contact Info | App Functionality | Yes | No |
| User ID | Identifiers | App Functionality | Yes | No |
| Photos or Videos | User Content | App Functionality | Yes | No |

**Data Not Linked to You:**

| Data Type | Category | Purpose | Linked to Identity | Used for Tracking |
|---|---|---|---|---|
| Crash Data | Diagnostics | Analytics | No | No |
| Performance Data | Diagnostics | Analytics | No | No |

**Data NOT Collected (explicit declarations):**

- Location: NOT collected (no geolocation features in v2).
- Contacts: NOT collected (Skye has its own contact database; it does NOT access the iOS Contacts app).
- Browsing History: NOT collected.
- Search History: NOT collected (search queries are processed but not logged for analytics).
- Advertising Data: NOT collected.
- Financial Information: NOT collected (deal amounts are user-entered CRM data, not financial instruments).
- Health & Fitness: NOT collected.
- Sensitive Information: NOT collected.

#### 5.4 Data Retention Policy

| Data Category | Local Retention | Server Retention | Purge Trigger |
|---|---|---|---|
| **Auth tokens** (Tier 1) | Until logout or expiry | Access: 1hr. Refresh: 30 days. | Logout, token rotation, or expiry |
| **Contact/Deal cache** (Tier 2) | 30 days since last sync | Per account terms (indefinite while active) | Logout, account deletion, or 30-day offline expiry |
| **Property images** (Tier 2) | 7 days (LRU cache, max 500MB) | Indefinite (cloud storage) | Cache eviction, logout, or manual clear |
| **AI chat history** (Tier 2) | 7 days (last 50 conversations) | 90 days | Logout or manual clear |
| **App preferences** (Tier 3) | Indefinite | Not stored on server | App uninstall |
| **Crash/perf data** | Not stored locally | 90 days (Sentry retention) | Automatic Sentry retention policy |

**Automatic purge:** A background task runs on app launch to check offline cache age. Data older than the retention period is deleted without user intervention.

```typescript
async function purgeStaleOfflineData(): Promise<void> {
  const db = await getDatabase();
  const thirtyDaysAgo = Date.now() - (30 * 24 * 60 * 60 * 1000);
  const sevenDaysAgo = Date.now() - (7 * 24 * 60 * 60 * 1000);

  await db.runAsync('DELETE FROM contacts_cache WHERE last_synced_at < ?', [thirtyDaysAgo]);
  await db.runAsync('DELETE FROM deals_cache WHERE last_synced_at < ?', [thirtyDaysAgo]);
  await db.runAsync('DELETE FROM chat_history WHERE created_at < ?', [sevenDaysAgo]);

  // Purge image cache exceeding 500MB
  await ImageCache.evictToSize(500 * 1024 * 1024);
}
```

#### 5.5 Account Deletion Flow

Apple requires that any app offering account creation must provide in-app account deletion (App Store Review Guideline 5.1.1(v)). The deletion option must be easy to find.

**User flow:**

```
Settings > Account > Delete My Account
       |
       v
+------------------------------------------+
|  DELETE YOUR ACCOUNT                     |
|                                          |
|  This action is permanent and cannot     |
|  be undone. All your data will be        |
|  deleted, including:                     |
|                                          |
|  - All contacts and client data          |
|  - All deals and transaction history     |
|  - All property listings                 |
|  - All AI conversation history           |
|  - All uploaded documents and images     |
|                                          |
|  +------------------------------------+  |
|  | Active Subscriptions Warning       |  |
|  |                                    |  |
|  | You have an active subscription.   |  |
|  | Deleting your account will NOT     |  |
|  | cancel your subscription billing.  |  |
|  | Please cancel your subscription    |  |
|  | first.                             |  |
|  |                                    |  |
|  | [Manage Subscription]              |  |
|  +------------------------------------+  |
|                                          |
|  To confirm, type "DELETE" below:        |
|                                          |
|  +------------------------------------+  |
|  |                                    |  |
|  +------------------------------------+  |
|                                          |
|  [Cancel]          [Delete My Account]   |
|                    (red, destructive)     |
+------------------------------------------+
       |
       | User types "DELETE" and taps button
       v
+------------------------------------------+
|  IDENTITY VERIFICATION                   |
|                                          |
|  A verification code has been sent to    |
|  a***@example.com                        |
|                                          |
|  Enter the 6-digit code:                 |
|  [_] [_] [_] [_] [_] [_]               |
|                                          |
|  [Resend Code]         [Verify & Delete] |
+------------------------------------------+
       |
       | Code verified
       v
+------------------------------------------+
|  Backend processing:                     |
|                                          |
|  1. Mark account as "pending_deletion"   |
|  2. Revoke ALL sessions (web + native)   |
|  3. Schedule data deletion job (24hr     |
|     grace period for accidental delete)  |
|  4. Send confirmation email              |
|  5. After 24hr: cascade delete all data  |
|     from all tables                      |
|  6. Revoke Google Sign-In token          |
|  7. Remove from Sentry user context      |
|  8. Purge from any email marketing lists |
+------------------------------------------+
       |
       v
+------------------------------------------+
|  ACCOUNT DELETION SCHEDULED              |
|                                          |
|  Your account will be permanently        |
|  deleted within 24 hours. You will       |
|  receive a confirmation email.           |
|                                          |
|  If you change your mind, contact        |
|  support@skyecrm.com within 24 hours.    |
|                                          |
|  [OK]                                    |
+------------------------------------------+
       |
       | User taps OK
       v
  Clear all local data + navigate to Auth screen
```

**Server endpoint:**

```
POST /api/user/delete
Authorization: Bearer <access_token>
X-Request-Signature: <HMAC signature>
{
  "verificationCode": "123456",
  "confirmationText": "DELETE"
}

Response:
200 OK
{
  "status": "pending_deletion",
  "deletionScheduledAt": "2026-03-03T12:00:00Z",
  "gracePeriodEndsAt": "2026-03-03T12:00:00Z"
}
```

---

### 6. Security Threat Model

This section maps threats from the OWASP Mobile Top 10 (2024) to Skye-specific mitigations.

#### 6.1 Jailbreak Detection

**OWASP M7: Insufficient Binary Protections**

**Decision: Detect and warn. Do NOT block.**

Apple may reject apps that refuse to run on jailbroken devices, as it could be seen as limiting device functionality. Instead, detect jailbreak status and:

1. Show a non-blocking warning dialog on first launch on a jailbroken device.
2. Log the event to the security audit log.
3. Optionally disable certain high-sensitivity features (e.g., biometric auth, since it may be compromised on jailbroken devices).

**Implementation using `jail-monkey`:**

```typescript
import JailMonkey from 'jail-monkey';

async function performSecurityChecks(): Promise<SecurityStatus> {
  const isJailbroken = JailMonkey.isJailBroken();
  const isDebugMode = JailMonkey.isDebuggedMode();
  const canMockLocation = JailMonkey.canMockLocation();
  const hookDetected = JailMonkey.hookDetected();

  if (isJailbroken && !__DEV__) {
    // Log to security audit
    await apiClient.post('/api/security/audit', {
      event: 'jailbreak_detected',
      details: JailMonkey.jailBrokenMessage(),
      deviceId: await getDeviceId(),
    });

    // Show warning (once per install)
    const warningShown = await AsyncStorage.getItem('jailbreak_warning_shown');
    if (!warningShown) {
      Alert.alert(
        'Security Warning',
        'This device appears to be modified (jailbroken). Your data may be at ' +
        'increased risk. For the best security, we recommend using Skye on an ' +
        'unmodified device.',
        [{ text: 'I Understand',
           onPress: () => AsyncStorage.setItem('jailbreak_warning_shown', 'true') }]
      );
    }

    // Disable biometric auth on jailbroken devices
    await SecureStore.setItemAsync('skye_biometric_available', 'false');
  }

  if (hookDetected && !__DEV__) {
    // Frida or similar hooking framework detected
    await apiClient.post('/api/security/audit', {
      event: 'hook_detected',
      deviceId: await getDeviceId(),
    });
  }

  return { isJailbroken, isDebugMode, canMockLocation, hookDetected };
}
```

**Limitations:** `jail-monkey` can be bypassed by sophisticated attackers using tools like Liberty Lite or Frida. This is a detection layer, not a prevention layer. Defense in depth (server-side validation, token rotation, certificate pinning) provides the actual security.

#### 6.2 Man-in-the-Middle (MITM) Protection

**OWASP M5: Insecure Communication**

Mitigations (layered):

| Layer | Mechanism | Implementation |
|---|---|---|
| 1. Transport | TLS 1.2+ enforced via ATS | iOS `Info.plist` configuration |
| 2. Certificate validation | SSL public key pinning | `react-native-ssl-public-key-pinning` (Section 3.3) |
| 3. Request integrity | HMAC signing for mutations | Custom interceptor (Section 3.4) |
| 4. Response validation | Verify JWT signatures client-side | Verify `iss`, `aud`, `exp` claims |

**ATS configuration for production (`Info.plist`):**

```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <false/>
  <!-- No exceptions. All connections must be HTTPS with TLS 1.2+. -->
  <!-- The React Native Metro dev server is handled by Expo's
       development build configuration and is NOT present in
       production builds. -->
</dict>
```

#### 6.3 Token Theft Prevention

**OWASP M1: Improper Credential Usage / M9: Insecure Data Storage**

| Attack Vector | Mitigation |
|---|---|
| Extracting tokens from device storage | Tokens stored in iOS Keychain (hardware-backed encryption) via `expo-secure-store` with `AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY` (prevents iCloud sync, requires device unlock) |
| Intercepting tokens in transit | TLS 1.2+ with certificate pinning |
| Stolen refresh token reuse | Token rotation with reuse detection (Section 2.5) -- reuse revokes entire token family |
| Physical device access (unlocked) | Optional biometric lock (Section 2.7) with configurable timeout |
| Token leaking via logs | Console logging stripped in release builds; tokens never logged even in debug |
| Token in memory after logout | All caches cleared, React Query invalidated, Zustand store reset |
| Token extracted from backup | `AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY` prevents token inclusion in iCloud backup |

#### 6.4 Reverse Engineering Protection

**OWASP M7: Insufficient Binary Protections**

| Layer | Protection | Details |
|---|---|---|
| JavaScript source | Hermes bytecode compilation | AOT compilation to Hermes bytecode format; decompilation requires version-specific tools (`hermes-dec`) that lag behind releases |
| JavaScript obfuscation | Pre-Hermes obfuscation via `metro-react-native-babel-preset` + `babel-plugin-transform-remove-console` | Variable renaming, dead code elimination, console stripping |
| Additional JS obfuscation | `react-native-obfuscating-transformer` (optional, for high-value logic) | Control flow flattening, string encryption for sensitive string constants |
| Native code | Xcode build settings: `STRIP_INSTALLED_PRODUCT = YES`, `DEPLOYMENT_POSTPROCESSING = YES` | Strips debug symbols from release binary |
| Source maps | NOT included in App Store build | Uploaded to Sentry only (authenticated access) |
| Sensitive logic location | **Server-side** | All business rules, pricing calculations, AI prompts, and authorization checks run on the backend. The client is a rendering layer only. |

**Fundamental principle:** Assume the client can be fully reverse-engineered. No client-side check should be the sole gate for any security-critical operation. Every authorization decision happens server-side with RLS enforcement.

#### 6.5 Insecure Network Detection

**OWASP M5: Insecure Communication**

App Transport Security (ATS) blocks all non-HTTPS connections by default in iOS 9+. However, additional application-level enforcement is implemented:

```typescript
import NetInfo from '@react-native-community/netinfo';

// On app launch and on every network state change
NetInfo.addEventListener(state => {
  if (state.isConnected && state.type === 'wifi') {
    // WiFi connection -- check if captive portal
    // (Captive portals can MITM traffic before user authenticates)
    checkForCaptivePortal().then(isCaptive => {
      if (isCaptive) {
        showBanner('You appear to be on a captive WiFi network. ' +
                   'Some features may be unavailable until you complete WiFi login.');
      }
    });
  }
});

// The API client NEVER downgrades to HTTP
// API_BASE_URL is hardcoded as https:// -- there is no fallback
const API_BASE_URL = 'https://api.skyecrm.com'; // HTTPS only, always
```

#### 6.6 Deep Link Injection

**OWASP M4: Insufficient Input/Output Validation**

Deep links and notification payloads are untrusted input. They can be crafted by malicious apps or push notification spoofing.

**Validation rules:**

```typescript
import { z } from 'zod';

// Define strict schemas for every deep link route
const deepLinkSchemas = {
  contact: z.object({
    id: z.string().uuid(), // MUST be valid UUID, not arbitrary string
  }),
  deal: z.object({
    id: z.string().uuid(),
  }),
  chat: z.object({
    conversationId: z.string().uuid().optional(),
  }),
  auth: z.object({
    // Auth deep links accept NO parameters from external sources
    // The OAuth callback is handled by the Google SDK internally
  }),
};

function handleDeepLink(url: string): void {
  const parsed = parseUrl(url);

  // 1. Validate scheme
  if (parsed.scheme !== 'com.skyecrm' && !parsed.host?.endsWith('skyecrm.com')) {
    Sentry.captureMessage(`Invalid deep link scheme: ${url}`);
    return; // Silently reject
  }

  // 2. Validate route exists
  const route = parsed.path?.split('/')[1];
  if (!route || !(route in deepLinkSchemas)) {
    return; // Unknown route, ignore
  }

  // 3. Validate parameters with Zod
  const schema = deepLinkSchemas[route as keyof typeof deepLinkSchemas];
  const result = schema.safeParse(parsed.params);

  if (!result.success) {
    Sentry.captureMessage(`Invalid deep link params: ${url}`, {
      extra: { errors: result.error.issues },
    });
    return; // Invalid params, ignore
  }

  // 4. Navigate only if authenticated
  if (!isAuthenticated()) {
    // Queue navigation for after auth
    pendingDeepLink.set({ route, params: result.data });
    return;
  }

  // 5. Navigate with validated params
  navigation.navigate(route, result.data);
}
```

**Push notification payload validation:** Notification payloads are treated identically to deep links. The `data` field of a push notification is parsed and validated through the same schema pipeline before any navigation occurs.

#### 6.7 SQL Injection in Offline Cache

**OWASP M4: Insufficient Input/Output Validation**

All SQLite queries MUST use parameterized queries. String concatenation for SQL is a bannable offense in code review.

**Correct patterns:**

```typescript
// CORRECT: Parameterized query
await db.runAsync(
  'INSERT INTO contacts_cache (id, name, email, phone) VALUES (?, ?, ?, ?)',
  [contact.id, contact.name, contact.email, contact.phone]
);

// CORRECT: Parameterized search
await db.getAllAsync(
  'SELECT * FROM contacts_cache WHERE name LIKE ? LIMIT ?',
  [`%${searchTerm}%`, limit]
);

// CORRECT: Parameterized update
await db.runAsync(
  'UPDATE deals_cache SET status = ? WHERE id = ? AND user_id = ?',
  [newStatus, dealId, userId]
);
```

**Prohibited patterns:**

```typescript
// BANNED: String concatenation (SQL injection vulnerable)
await db.execAsync(`SELECT * FROM contacts WHERE name = '${name}'`);

// BANNED: Template literal interpolation
await db.execAsync(`DELETE FROM deals WHERE id = '${dealId}'`);
```

**Enforcement:** Add an ESLint rule (custom or via `eslint-plugin-security`) that flags any `db.execAsync` call containing template literal expressions or string concatenation. The rule is configured as `error` severity, blocking CI.

---

### 7. Session Management

#### 7.1 Multi-Device Support

Users can be logged into the web app and the native app simultaneously. Sessions are completely independent.

**Session data model:**

```sql
CREATE TABLE sessions (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  client_type   TEXT NOT NULL CHECK (client_type IN ('web', 'native-ios', 'native-android')),
  device_id     TEXT,            -- Unique per device (generated on first install)
  device_name   TEXT,            -- e.g., "iPhone 15 Pro", "Chrome on macOS"
  ip_address    INET,
  user_agent    TEXT,
  last_active_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at    TIMESTAMPTZ NOT NULL,
  revoked_at    TIMESTAMPTZ,     -- Non-null = session revoked
  CONSTRAINT valid_expiry CHECK (expires_at > created_at)
);

CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_user_active ON sessions(user_id) WHERE revoked_at IS NULL;
```

**How web and native sessions coexist:**

| Aspect | Web (NextAuth) | Native (Skye) |
|---|---|---|
| Session storage | HTTP-only cookie | SecureStore (Keychain) |
| Session identifier | NextAuth session token (cookie) | Skye JWT + refresh token |
| Token format | NextAuth JWT (signed with `NEXTAUTH_SECRET`) | Skye JWT (signed with `SKYE_JWT_SECRET`) |
| Supabase access | Custom JWT signed with `SUPABASE_JWT_SECRET` in session callback | Custom JWT signed with `SUPABASE_JWT_SECRET` at token issue |
| Session duration | Configurable (typically 30 days with sliding window) | Access: 1hr, Refresh: 30 days |
| Session record | Stored in `sessions` table with `client_type = 'web'` | Stored in `sessions` table with `client_type = 'native-ios'` |

Both session types resolve to the same `user_id` in the `sessions` table, and both produce Supabase JWTs with the same `sub` claim, so RLS policies work identically.

#### 7.2 Session Invalidation (Remote Revocation)

The server provides a "Log out all devices" capability. This is accessible from:
- The web app Settings page.
- The native app Settings > Security > Active Sessions.
- The backend admin panel (for compromised accounts).

**Endpoint:**

```
POST /api/auth/sessions/revoke-all
Authorization: Bearer <access_token>
{
  "excludeCurrentSession": true  // Optional: keep current session active
}

Response:
200 OK
{
  "revokedCount": 3,
  "message": "3 sessions have been revoked."
}
```

**Server implementation:**

```sql
-- Revoke all sessions except the current one
UPDATE sessions
SET revoked_at = now()
WHERE user_id = $1
  AND id != $2           -- Exclude current session
  AND revoked_at IS NULL;

-- Also revoke all refresh tokens for those sessions
UPDATE refresh_tokens
SET revoked_at = now()
WHERE session_id IN (
  SELECT id FROM sessions
  WHERE user_id = $1
    AND id != $2
    AND revoked_at IS NOT NULL
    AND revoked_at > now() - interval '1 minute'  -- Just revoked
);
```

**Client-side enforcement:** On every API call, the server checks if the session has been revoked. If the session is revoked (`revoked_at IS NOT NULL`), the server returns 401 with `error: "session_revoked"`. The client clears local state and redirects to the auth screen.

#### 7.3 Concurrent Session Limit

**Default limit: 5 active sessions per user.**

This limit is configurable per account tier (e.g., enterprise accounts may allow 10).

**Enforcement logic (server-side, during login):**

```typescript
async function enforceSessionLimit(userId: string, maxSessions: number = 5): Promise<void> {
  const activeSessions = await db.query(
    `SELECT id, created_at, device_name, last_active_at
     FROM sessions
     WHERE user_id = $1 AND revoked_at IS NULL AND expires_at > now()
     ORDER BY last_active_at DESC`,
    [userId]
  );

  if (activeSessions.rows.length >= maxSessions) {
    // Revoke the OLDEST session (least recently active)
    const oldestSession = activeSessions.rows[activeSessions.rows.length - 1];

    await db.query(
      `UPDATE sessions SET revoked_at = now() WHERE id = $1`,
      [oldestSession.id]
    );

    await db.query(
      `UPDATE refresh_tokens SET revoked_at = now()
       WHERE session_id = $1 AND revoked_at IS NULL`,
      [oldestSession.id]
    );

    // Optionally notify the user that their oldest session was signed out
    await notifyUser(userId, {
      type: 'session_evicted',
      deviceName: oldestSession.device_name,
    });
  }
}
```

**Behavior:** When a user exceeds the limit, the least recently active session is revoked automatically. The user receives a push notification: "You were signed out on [device name] because you signed in on a new device."

#### 7.4 Session Activity Tracking

Each session's `last_active_at` is updated on every authenticated API call. This is done efficiently via a debounced server-side mechanism (update at most once per 5 minutes per session to avoid excessive writes).

**Settings > Security > Active Sessions UI:**

```
+------------------------------------------+
|  ACTIVE SESSIONS                         |
|                                          |
|  +------------------------------------+  |
|  |  iPhone 15 Pro               (you) |  |
|  |  Skye iOS App                      |  |
|  |  Last active: Just now             |  |
|  |  Signed in: Feb 28, 2026           |  |
|  |  IP: 192.168.1.xxx                 |  |
|  +------------------------------------+  |
|                                          |
|  +------------------------------------+  |
|  |  Chrome on macOS                   |  |
|  |  Skye Web App                      |  |
|  |  Last active: 2 hours ago          |  |
|  |  Signed in: Feb 15, 2026           |  |
|  |  IP: 10.0.0.xxx                    |  |
|  |                     [Sign Out]     |  |
|  +------------------------------------+  |
|                                          |
|  +------------------------------------+  |
|  |  iPad Air                          |  |
|  |  Skye iOS App                      |  |
|  |  Last active: 3 days ago           |  |
|  |  Signed in: Jan 20, 2026           |  |
|  |  IP: 172.16.0.xxx                  |  |
|  |                     [Sign Out]     |  |
|  +------------------------------------+  |
|                                          |
|  [Sign Out All Other Devices]            |
|                                          |
+------------------------------------------+
```

**API endpoint:**

```
GET /api/auth/sessions
Authorization: Bearer <access_token>

Response:
200 OK
{
  "sessions": [
    {
      "id": "session-uuid",
      "clientType": "native-ios",
      "deviceName": "iPhone 15 Pro",
      "lastActiveAt": "2026-03-02T10:30:00Z",
      "createdAt": "2026-02-28T09:00:00Z",
      "ipAddress": "192.168.1.xxx",
      "isCurrent": true
    }
  ],
  "maxSessions": 5
}
```

**IP address display:** The last octet of the IP address is masked (e.g., `192.168.1.xxx`) in the client display to prevent full IP exposure. The server stores the full IP for security audit purposes.

---

### Appendix A: OWASP Mobile Top 10 (2024) Coverage Matrix

| OWASP Category | Skye Mitigation | Spec Section |
|---|---|---|
| **M1: Improper Credential Usage** | Keychain storage, no secrets in bundle, server-side secret management | 2.2, 4.1, 4.3 |
| **M2: Inadequate Supply Chain Security** | Privacy manifests, SDK audit, dependency scanning | 5.1, 5.2 |
| **M3: Insecure Authentication/Authorization** | Google OAuth native flow, JWT validation, Supabase RLS, session management | 1.x, 2.x, 7.x |
| **M4: Insufficient Input/Output Validation** | Deep link validation (Zod), parameterized SQL, request body validation | 6.6, 6.7 |
| **M5: Insecure Communication** | TLS 1.2+ (ATS), certificate pinning, HMAC signing | 3.3, 3.4, 6.2, 6.5 |
| **M6: Inadequate Privacy Controls** | Privacy Manifest, no tracking, data classification, retention policy, account deletion | 4.1, 5.x |
| **M7: Insufficient Binary Protections** | Hermes bytecode, jailbreak detection, code stripping, no source maps in build | 6.1, 6.4 |
| **M8: Security Misconfiguration** | ATS enforcement, debug/release separation, no console in prod | 4.6, 6.5 |
| **M9: Insecure Data Storage** | SQLCipher, SecureStore, data tiering, screenshot protection, clipboard security | 4.1, 4.2, 4.4, 4.5 |
| **M10: Insufficient Cryptography** | AES-256 (SQLCipher), SHA-256 (token hashing), HMAC-SHA256 (request signing) | 2.1, 3.4, 4.2 |

### Appendix B: Dependency List (Security-Related)

| Package | Purpose | Native Code? | Privacy Manifest? |
|---|---|---|---|
| `@react-native-google-signin/google-signin` | Google OAuth | Yes | Yes (GoogleSignIn SDK) |
| `expo-secure-store` | Keychain access | Yes | Yes (Expo managed) |
| `expo-local-authentication` | Face ID / Touch ID | Yes | Yes (Expo managed) |
| `expo-sqlite` | Encrypted offline DB | Yes (SQLCipher) | Yes (Expo managed) |
| `expo-screen-capture` | Screenshot protection | Yes | Yes (Expo managed) |
| `expo-crypto` | Cryptographic operations | Yes | Yes (Expo managed) |
| `expo-clipboard` | Clipboard management | Yes | Yes (Expo managed) |
| `react-native-ssl-public-key-pinning` | Certificate pinning | Yes (TrustKit) | Must verify/add |
| `jail-monkey` | Jailbreak detection | Yes | Must verify/add |
| `async-mutex` | Token refresh mutex | No (pure JS) | N/A |
| `axios` | HTTP client | No (pure JS) | N/A |
| `zod` | Input validation | No (pure JS) | N/A |
| `@sentry/react-native` | Error tracking | Yes | Yes (on Apple's SDK list) |

### Appendix C: Security Audit Checklist (Pre-Release)

Before every App Store submission, the following must be verified:

- [ ] All tokens stored in SecureStore (not AsyncStorage, not MMKV).
- [ ] Console logging stripped from production bundle (`transform-remove-console`).
- [ ] Flipper removed from production Podfile.
- [ ] Source maps NOT included in IPA (uploaded to Sentry separately).
- [ ] Certificate pinning enabled and pins are current.
- [ ] ATS has no exceptions in production `Info.plist`.
- [ ] `PrivacyInfo.xcprivacy` is up to date with current SDK usage.
- [ ] No API keys or secrets in JavaScript bundle (scan with `gitleaks`).
- [ ] SQLCipher encryption verified (open DB, check `PRAGMA cipher_version`).
- [ ] Account deletion flow tested end-to-end.
- [ ] Biometric lock tested on Face ID and Touch ID devices.
- [ ] Deep link validation tested with malformed URLs.
- [ ] Token refresh race condition tested (5 simultaneous requests with expired token).
- [ ] Logout clears ALL local data (verify with filesystem inspection).
- [ ] Jailbreak warning displays correctly on jailbroken test device.
- [ ] Rate limit handling tested (mock 429 response).
- [ ] Offline cache purge verified (data older than retention period is deleted).
- [ ] App switcher screenshot shows blur overlay, not app content.
- [ ] Clipboard auto-clear verified after 60 seconds.

---

*End of V2: Authentication, Security & Data Protection specification.*

---

**Key sources referenced during research:**

- [OWASP Mobile Top 10 (2024)](https://owasp.org/www-project-mobile-top-10/)
- [Apple Privacy Manifest Documentation](https://developer.apple.com/documentation/bundleresources/privacy-manifest-files)
- [Apple Describing Use of Required Reason API](https://developer.apple.com/documentation/bundleresources/describing-use-of-required-reason-api)
- [Apple App Tracking Transparency](https://developer.apple.com/documentation/apptrackingtransparency)
- [Apple Account Deletion Requirements](https://developer.apple.com/support/offering-account-deletion-in-your-app/)
- [Apple Third-Party SDK Requirements](https://developer.apple.com/support/third-party-SDK-requirements)
- [Expo Authentication Guide](https://docs.expo.dev/develop/authentication/)
- [Expo SecureStore Documentation](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [Expo Privacy Manifests Guide](https://docs.expo.dev/guides/apple-privacy/)
- [Expo LocalAuthentication](https://docs.expo.dev/versions/latest/sdk/local-authentication/)
- [Expo SQLite Documentation](https://docs.expo.dev/versions/latest/sdk/sqlite/)
- [react-native-ssl-public-key-pinning](https://www.npmjs.com/package/react-native-ssl-public-key-pinning)
- [Callstack: SSL Pinning in React Native](https://www.callstack.com/blog/ssl-pinning-in-react-native-apps)
- [NextAuth FAQ on React Native](https://next-auth.js.org/faq)
- [Supabase Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [jail-monkey (npm)](https://www.npmjs.com/package/jail-monkey)
- [React Native Hermes Security Analysis](https://www.iteratorshq.com/blog/the-silent-security-revolution-how-react-native-hermes-turned-apps-from-a-data-goldmine-into-fort-knox/)
- [Apple App Privacy Details](https://developer.apple.com/app-store/app-privacy-details/)

---


## Section 6: Push Notifications

---

### Table of Contents

1. [APNs Integration Architecture](#1-apns-integration-architecture)
2. [Notification Types and Payloads](#2-notification-types-and-payloads)
3. [Notification Permission Flow](#3-notification-permission-flow)
4. [Deep Link Resolution Engine](#4-deep-link-resolution-engine)
5. [Badge Management](#5-badge-management)
6. [Notification Center Behavior](#6-notification-center-behavior)
7. [Silent Push and Background Refresh](#7-silent-push-and-background-refresh)

---

### 1. APNs Integration Architecture

#### 1.1 APNs-Direct vs Firebase Cloud Messaging (FCM)

**Recommendation: APNs-Direct (with `@parse/node-apn` on the backend)**

| Criterion | APNs-Direct | FCM (via Firebase) |
|---|---|---|
| Latency | Direct to Apple servers; lowest possible delivery latency | Additional hop through Google servers before routing to APNs |
| Payload control | Full, unrestricted access to every APNs header and payload field | FCM wraps payloads in its own envelope; some APNs-specific features require nested `apns.payload` overrides |
| iOS-specific features | Native support for Live Activities, Notification Service Extensions, `thread-id`, `interruption-level`, `relevance-score` | Requires platform-specific overrides within FCM message structure |
| Dependency surface | Single dependency: `@parse/node-apn` | Requires `@react-native-firebase/messaging` on client + Firebase project + Google service account on server |
| Token management | Device token obtained via native iOS API, posted to your own backend | FCM token wraps the APNs token; FCM token can change independently of APNs token |
| Cost | Free (APNs has no per-message charges) | Free tier, but adds Firebase dependency and telemetry |
| Debugging | Direct feedback from APNs on every send (HTTP/2 response per notification) | Error feedback routed through Firebase; harder to diagnose APNs-specific rejections |

**Justification:** Skye is an iOS-only application. APNs-direct eliminates the Firebase intermediary, reduces latency, gives full payload control for rich notification features (property images, actionable buttons, notification grouping), and removes an entire third-party dependency from the architecture. Every notification type Skye sends (engagement alerts, morning briefings, task reminders, property matches, AI completions) benefits from precise APNs payload control.

**If the app ever expands to Android:** At that point, adopt FCM for Android while keeping APNs-direct for iOS. The backend push service (Section 1.6) already uses a platform discriminator, making this a straightforward addition.

#### 1.2 Client-Side Library Selection

**Recommendation: Notifee (`@notifee/react-native`) + React Native's built-in `PushNotificationIOS`**

| Library | Role |
|---|---|
| `@notifee/react-native` | Local notification display, notification categories/actions registration, badge management, foreground presentation options, background event handling |
| React Native `PushNotificationIOS` (or direct native module) | APNs device token registration, remote notification receipt, background notification handling |

**Why Notifee over `expo-notifications`:** Skye is a bare React Native project (not Expo-managed). Notifee provides full native-level control over iOS notification categories, rich notification styling, badge count APIs (`setBadgeCount`, `incrementBadgeCount`, `decrementBadgeCount`), and background event handling without requiring the Expo runtime. Notifee is open-source, actively maintained, and has first-class support for the UNNotificationCategory/UNNotificationAction APIs that Skye's actionable notifications require.

#### 1.3 Device Token Registration Flow

```
App Launch
    |
    v
[Check notification permission status]
    |
    +--> "not_determined" --> [Show pre-permission priming screen (Section 3)]
    |                              |
    |                     [User taps "Enable Notifications"]
    |                              |
    |                              v
    |                     [Request iOS notification permission]
    |                              |
    |                     +--------+--------+
    |                     |                 |
    |                  granted           denied
    |                     |                 |
    |                     v                 v
    |              [Register for          [Store "denied" state]
    |               remote notifications]  [Skip token registration]
    |                     |
    +--> "authorized" ----+
    |                     |
    |                     v
    |              [iOS returns APNs device token via
    |               didRegisterForRemoteNotificationsWithDeviceToken]
    |                     |
    |                     v
    |              [Convert token to hex string]
    |                     |
    |                     v
    |              [POST /api/push/subscribe]
    |              {
    |                "token": "<apns_device_token_hex>",
    |                "platform": "ios",
    |                "device_id": "<unique_device_identifier>",
    |                "app_version": "2.0.0",
    |                "os_version": "18.2",
    |                "environment": "production" | "sandbox"
    |              }
    |                     |
    |                     v
    |              [Server stores token with user_id + device_id composite key]
    |              [Server responds 200 OK with subscription_id]
    |                     |
    +--> "denied" --------+
                          |
                          v
                   [No registration; show settings redirect if user
                    later attempts to enable from in-app preferences]
```

**Device ID generation:** Use a persistent device identifier stored in iOS Keychain (survives app reinstalls). Generate a UUID on first launch and persist it via `react-native-keychain` or a similar secure storage library. This prevents duplicate token registrations across reinstalls.

**Environment detection:** The app must detect whether it is running against the APNs sandbox (debug/TestFlight builds) or production environment and include this in the subscription payload. The backend uses this to route pushes to the correct APNs endpoint (`api.sandbox.push.apple.com` vs `api.push.apple.com`).

#### 1.4 Token Refresh Handling

APNs device tokens can rotate under the following conditions:
- App is restored to a new device from backup
- User reinstalls the app
- iOS decides to rotate the token (rare but documented)

**Detection strategy:**

```javascript
// In AppDelegate.swift (native module bridge) or via PushNotificationIOS
// This callback fires on every app launch with the current token

function onTokenReceived(newToken) {
  const storedToken = await SecureStorage.get('apns_device_token');

  if (storedToken !== newToken) {
    // Token has changed -- re-register silently
    await api.post('/api/push/subscribe', {
      token: newToken,
      platform: 'ios',
      device_id: getDeviceId(),
      app_version: getAppVersion(),
      os_version: getOsVersion(),
      environment: getAPNsEnvironment(),
      previous_token: storedToken  // Server uses this to invalidate old token
    });

    await SecureStorage.set('apns_device_token', newToken);
  }
}
```

**Backend behavior on token refresh:**
1. If `previous_token` is provided, mark the old subscription record as `invalidated`.
2. Upsert the new token record keyed on `(user_id, device_id)`.
3. If a push to a stale token returns APNs status `410 Gone` or `BadDeviceToken`, immediately delete that subscription record from the database to prevent future failed sends.

#### 1.5 Backend APNs Integration

**Library:** `@parse/node-apn` (v7.x) -- the actively maintained fork of the original `node-apn`, now with 107K+ weekly downloads and support for HTTP/2, Live Activities, and modern APNs features.

**Authentication: Token-Based (.p8 Key)**

Token-based authentication is mandatory for Skye. Justification:
- `.p8` keys never expire (unlike `.p12` certificates which expire annually)
- A single `.p8` key works across all app bundle IDs and both sandbox/production environments
- No certificate renewal process to manage or monitor
- Simpler server configuration (three values: key file, key ID, team ID)

**Required credentials (stored as environment variables, never in source code):**

| Credential | Source | Environment Variable |
|---|---|---|
| `.p8` auth key file | Apple Developer Portal > Keys > Create Key (with APNs enabled) | `APNS_AUTH_KEY_PATH` or `APNS_AUTH_KEY_BASE64` |
| Key ID | Displayed when key is created (10-character string) | `APNS_KEY_ID` |
| Team ID | Apple Developer Portal > Membership | `APNS_TEAM_ID` |
| Bundle ID | App's bundle identifier | `APNS_BUNDLE_ID` |

**Server-side provider initialization:**

```javascript
const apn = require('@parse/node-apn');

const apnsProvider = new apn.Provider({
  token: {
    key: process.env.APNS_AUTH_KEY_PATH,    // Path to .p8 file
    keyId: process.env.APNS_KEY_ID,         // e.g., "AB12CD34EF"
    teamId: process.env.APNS_TEAM_ID,       // e.g., "9GQ7JHT2C4"
  },
  production: process.env.NODE_ENV === 'production',  // false = sandbox
});
```

**APNs HTTP/2 Headers (set per-notification):**

| Header | Value | Purpose |
|---|---|---|
| `apns-push-type` | `alert` or `background` | Required on iOS 13+. Must match payload content. |
| `apns-priority` | `10` (alert) or `5` (background/silent) | `10` = immediate delivery. `5` = power-considerate delivery. |
| `apns-topic` | `com.skye.app` (bundle ID) | Required. Identifies the target app. |
| `apns-expiration` | Unix timestamp or `0` | `0` = deliver immediately or drop. Timestamp = APNs retries until expiration. |
| `apns-collapse-id` | String (max 64 bytes) | Optional. Replaces existing notification with same collapse ID. Use for badge-only updates. |

#### 1.6 APNs Payload Format

**Complete payload structure (maximum 4,096 bytes):**

```json
{
  "aps": {
    "alert": {
      "title": "Follow-up Reminder",
      "subtitle": "Hot Lead",
      "body": "Marcus Johnson -- last contact 14 days ago",
      "title-loc-key": null,
      "title-loc-args": null,
      "loc-key": null,
      "loc-args": null,
      "launch-image": ""
    },
    "badge": 3,
    "sound": {
      "critical": 0,
      "name": "default",
      "volume": 1.0
    },
    "thread-id": "engagement-alerts",
    "category": "ENGAGEMENT_ALERT",
    "mutable-content": 1,
    "content-available": 0,
    "interruption-level": "time-sensitive",
    "relevance-score": 0.8
  },
  "type": "engagement",
  "contact_id": "550e8400-e29b-41d4-a716-446655440000",
  "deep_link": "/contacts/550e8400-e29b-41d4-a716-446655440000",
  "notification_id": "uuid-for-tracking",
  "timestamp": "2026-03-02T09:00:00Z"
}
```

**Field-by-field specification:**

| Field | Type | Required | Description |
|---|---|---|---|
| `aps.alert.title` | String | Yes | Bold heading line. Max ~50 chars for lock screen visibility. |
| `aps.alert.subtitle` | String | No | Secondary line below title. Use for priority labels or context. |
| `aps.alert.body` | String | Yes | Main message body. Max ~150 chars for full display without truncation. |
| `aps.badge` | Integer | No | Badge count to display on app icon. `0` clears badge. `null` = no change. |
| `aps.sound` | String or Object | No | `"default"` for system sound. Object form for custom sounds (see Section 6.4). |
| `aps.thread-id` | String | No | Groups notifications in Notification Center. All notifications with the same `thread-id` are stacked together. |
| `aps.category` | String | Yes (for actionable) | Matches a `UNNotificationCategory` registered on the client. Triggers action buttons. |
| `aps.mutable-content` | Integer (0 or 1) | No | Set to `1` to enable Notification Service Extension interception. Required for rich notifications (images). |
| `aps.content-available` | Integer (0 or 1) | No | Set to `1` for silent/background push. Must not coexist with `alert` in the same payload for reliable delivery. |
| `aps.interruption-level` | String | No | iOS 15+. Values: `passive`, `active` (default), `time-sensitive`, `critical`. Use `time-sensitive` for engagement alerts and overdue tasks. |
| `aps.relevance-score` | Double (0.0-1.0) | No | iOS 15+. Determines notification ranking in summary. Higher = more prominent. |
| Custom data keys | Any JSON | No | Placed at root level alongside `aps`. Contains `type`, `deep_link`, entity IDs, `notification_id` for analytics. |

**Payload size budget:** With the 4,096-byte limit, the `aps` dictionary typically consumes 300-500 bytes. Custom data should remain under 500 bytes. This leaves ample room, but long property addresses or descriptions should be truncated server-side before payload construction.

#### 1.7 Web Push Coexistence

The existing backend serves a web application with Web Push (VAPID-based). The push infrastructure must support both APNs and Web Push simultaneously.

**`/api/push/subscribe` endpoint schema (updated):**

```typescript
// POST /api/push/subscribe
interface PushSubscriptionRequest {
  // For APNs (iOS)
  token?: string;               // APNs device token (hex string)
  platform: 'ios' | 'web';     // Platform discriminator
  device_id?: string;           // Unique device identifier (iOS only)

  // For Web Push (existing)
  endpoint?: string;            // Web Push subscription endpoint URL
  keys?: {                      // Web Push VAPID keys
    p256dh: string;
    auth: string;
  };

  // Common
  app_version?: string;
  user_agent?: string;
  environment?: 'production' | 'sandbox';  // iOS only
  previous_token?: string;                  // For token rotation (iOS only)
}
```

**Database schema for push subscriptions:**

```sql
CREATE TABLE push_subscriptions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  platform        VARCHAR(10) NOT NULL CHECK (platform IN ('ios', 'web')),
  device_id       VARCHAR(255),         -- iOS: keychain-persisted UUID
  token           TEXT,                 -- iOS: APNs device token (hex)
  endpoint        TEXT,                 -- Web Push: subscription endpoint
  web_push_keys   JSONB,               -- Web Push: { p256dh, auth }
  environment     VARCHAR(10),          -- iOS: 'production' or 'sandbox'
  app_version     VARCHAR(20),
  os_version      VARCHAR(20),
  is_active       BOOLEAN DEFAULT true,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  last_used_at    TIMESTAMPTZ,          -- Updated on each successful push

  -- Prevent duplicate registrations
  UNIQUE (user_id, platform, device_id),
  -- For web push, endpoint is the unique key
  UNIQUE (user_id, platform, endpoint)
);

CREATE INDEX idx_push_subscriptions_user_active
  ON push_subscriptions (user_id, is_active)
  WHERE is_active = true;
```

**Push dispatcher architecture:**

```
[Notification Event Trigger]
         |
         v
[Push Service: buildNotificationPayload(event, user)]
         |
         v
[Query push_subscriptions WHERE user_id = ? AND is_active = true]
         |
         +---> platform = 'ios'  --> [APNs Formatter] --> [APNs Provider: send()]
         |                                                       |
         |                                                  [Handle response]
         |                                                  - 200: update last_used_at
         |                                                  - 410/BadDeviceToken: set is_active = false
         |                                                  - 429: exponential backoff + retry
         |
         +---> platform = 'web'  --> [Web Push Formatter] --> [web-push: sendNotification()]
                                                                   |
                                                              [Handle response]
                                                              - 201: update last_used_at
                                                              - 404/410: set is_active = false
```

**Key design principle:** The notification event layer is platform-agnostic. It produces a canonical notification object (`{ type, title, body, data }`). Platform-specific formatters then translate this into APNs JSON or Web Push JSON. This separation means adding Android/FCM in the future requires only a new formatter, not changes to the event system.

---

### 2. Notification Types and Payloads

#### 2.1 Engagement Alert

**Trigger:** Scheduled job detects a contact has not been contacted within their configured follow-up window (default: 14 days for warm leads, 7 days for hot leads, 30 days for cold leads).

**APNs Payload:**

```json
{
  "aps": {
    "alert": {
      "title": "Follow-up Reminder",
      "body": "Marcus Johnson \u2014 last contact 14 days ago"
    },
    "badge": 5,
    "sound": "default",
    "thread-id": "engagement-alerts",
    "category": "ENGAGEMENT_ALERT",
    "mutable-content": 1,
    "interruption-level": "time-sensitive",
    "relevance-score": 0.8
  },
  "type": "engagement",
  "contact_id": "550e8400-e29b-41d4-a716-446655440000",
  "contact_name": "Marcus Johnson",
  "days_since_contact": 14,
  "contact_phone": "+14155551234",
  "deep_link": "/contacts/550e8400-e29b-41d4-a716-446655440000",
  "notification_id": "notif-uuid-001",
  "timestamp": "2026-03-02T09:00:00Z"
}
```

**Notification Category Registration (client-side):**

```javascript
import notifee from '@notifee/react-native';

await notifee.setNotificationCategories([
  {
    id: 'ENGAGEMENT_ALERT',
    actions: [
      {
        id: 'call_now',
        title: 'Call Now',
        foreground: true,   // Opens app to initiate call
      },
      {
        id: 'snooze_1d',
        title: 'Snooze 1 Day',
        foreground: false,  // Handled in background
      },
      {
        id: 'dismiss',
        title: 'Dismiss',
        foreground: false,
        destructive: true,  // Red text styling
      },
    ],
  },
]);
```

**Action Handlers:**

| Action ID | Behavior | Opens App? |
|---|---|---|
| `call_now` | Navigate to ContactDetail screen, then initiate phone call via `Linking.openURL('tel:+14155551234')` | Yes |
| `snooze_1d` | POST `/api/contacts/{id}/snooze` with `{ duration: "1d" }`. Reschedules the engagement alert for tomorrow. Decrement badge count. | No |
| `dismiss` | POST `/api/notifications/{notification_id}/dismiss`. Removes from pending queue. Decrement badge count. | No |

**Collapse behavior:** Use `apns-collapse-id: "engagement-{contact_id}"` so that if the server sends an updated reminder for the same contact (e.g., "15 days" replacing "14 days"), the notification is updated in place rather than creating a duplicate.

---

#### 2.2 Morning Briefing

**Trigger:** Scheduled job runs at user's preferred time (stored in user preferences, default: 7:30 AM in user's local timezone). The job computes the briefing content before sending the push.

**APNs Payload:**

```json
{
  "aps": {
    "alert": {
      "title": "Good morning, Sarah",
      "body": "4 follow-ups today \u00b7 2 new leads"
    },
    "badge": 1,
    "sound": {
      "critical": 0,
      "name": "skye_morning_chime.caf",
      "volume": 0.7
    },
    "thread-id": "morning-briefing",
    "category": "MORNING_BRIEFING",
    "mutable-content": 1,
    "interruption-level": "active",
    "relevance-score": 1.0
  },
  "type": "briefing",
  "first_name": "Sarah",
  "follow_up_count": 4,
  "new_lead_count": 2,
  "briefing_date": "2026-03-02",
  "deep_link": "/chat?briefing=today",
  "notification_id": "notif-uuid-002",
  "timestamp": "2026-03-02T14:30:00Z"
}
```

**Custom Sound Requirements:**

The custom morning briefing sound (`skye_morning_chime.caf`) must meet the following specifications:

| Requirement | Specification |
|---|---|
| Format | Linear PCM, MA4, uLaw, or aLaw. Recommended: CAF container (`.caf`) |
| Duration | Maximum 30 seconds. Recommended: 2-4 seconds for a gentle chime. |
| File location | Bundled in the app's main bundle (added to Xcode project, included in "Copy Bundle Resources" build phase) |
| Volume | Set via `volume` field in sound object (0.0 to 1.0). Use `0.7` for a non-jarring wake-up. |
| Fallback | If the sound file is not found, iOS falls back to the default sound. |
| Production | Provide the sound file as `skye_morning_chime.caf` at 44.1kHz sample rate, 16-bit depth. |

**Notification Category Registration:**

```javascript
{
  id: 'MORNING_BRIEFING',
  actions: [
    {
      id: 'view_briefing',
      title: 'View Briefing',
      foreground: true,
    },
    {
      id: 'dismiss_briefing',
      title: 'Dismiss',
      foreground: false,
    },
  ],
}
```

**Action Handlers:**

| Action ID | Behavior | Opens App? |
|---|---|---|
| `view_briefing` | Navigate to Chat screen with `?briefing=today` parameter. Chat screen auto-loads today's AI-generated briefing. | Yes |
| `dismiss_briefing` | Mark briefing notification as read. Decrement badge. | No |

**Scheduling considerations:**
- The backend must store each user's timezone and preferred briefing time.
- The scheduled job must fan out across timezones (do not send all briefings at one UTC time).
- Use a job queue (e.g., BullMQ) with delayed jobs keyed to each user's local delivery time.
- If the briefing data is not yet computed (e.g., AI processing delay), delay the push up to 15 minutes. If still not ready, send a simplified briefing: "Your daily briefing is ready. Tap to view."
- The `apns-collapse-id` should be `"briefing-{date}"` to prevent duplicate briefings on the same day.

---

#### 2.3 Overdue Task

**Trigger:** Scheduled job detects tasks with `due_date < NOW()` and `status != 'completed'`. Runs every hour. Only sends the notification once per task per day (tracked via `last_notified_at` on the task record).

**APNs Payload:**

```json
{
  "aps": {
    "alert": {
      "title": "Overdue Task",
      "body": "Call Sarah Chen \u2014 due yesterday"
    },
    "badge": 7,
    "sound": "default",
    "thread-id": "overdue-tasks",
    "category": "OVERDUE_TASK",
    "mutable-content": 1,
    "interruption-level": "time-sensitive",
    "relevance-score": 0.9
  },
  "type": "task",
  "contact_id": "660e8400-e29b-41d4-a716-446655440001",
  "task_id": "770e8400-e29b-41d4-a716-446655440002",
  "task_description": "Call Sarah Chen",
  "due_date": "2026-03-01",
  "deep_link": "/contacts/660e8400-e29b-41d4-a716-446655440001?tab=tasks",
  "notification_id": "notif-uuid-003",
  "timestamp": "2026-03-02T10:00:00Z"
}
```

**Notification Category Registration:**

```javascript
{
  id: 'OVERDUE_TASK',
  actions: [
    {
      id: 'mark_complete',
      title: 'Mark Complete',
      foreground: false,
    },
    {
      id: 'snooze_task',
      title: 'Snooze',
      foreground: false,
    },
    {
      id: 'view_task',
      title: 'View',
      foreground: true,
    },
  ],
}
```

**Action Handlers:**

| Action ID | Behavior | Opens App? |
|---|---|---|
| `mark_complete` | POST `/api/tasks/{task_id}/complete`. Update task status. Decrement badge. Remove from overdue notifications. | No |
| `snooze_task` | POST `/api/tasks/{task_id}/snooze` with `{ duration: "1d" }`. Updates `due_date` to tomorrow. Decrement badge. | No |
| `view_task` | Navigate to ContactDetail screen with tasks tab selected. | Yes |

**Due date formatting rules:**
- If due today: "due today"
- If due yesterday: "due yesterday"
- If due 2-7 days ago: "due {N} days ago"
- If due more than 7 days ago: "due {formatted_date}" (e.g., "due Feb 22")

---

#### 2.4 New Property Match

**Trigger:** New listing ingested from MLS feed matches a saved search criteria for one or more contacts. The matching engine evaluates criteria (price range, bedrooms, bathrooms, location, square footage) and triggers a notification per match.

**APNs Payload:**

```json
{
  "aps": {
    "alert": {
      "title": "New Property Match",
      "body": "742 Evergreen Terrace \u00b7 $875,000 \u00b7 4bd/3ba"
    },
    "badge": 8,
    "sound": "default",
    "thread-id": "property-matches",
    "category": "PROPERTY_MATCH",
    "mutable-content": 1,
    "interruption-level": "active",
    "relevance-score": 0.6
  },
  "type": "property",
  "property_id": "880e8400-e29b-41d4-a716-446655440003",
  "contact_id": "660e8400-e29b-41d4-a716-446655440001",
  "contact_name": "Sarah Chen",
  "property_address": "742 Evergreen Terrace",
  "property_price": 875000,
  "property_beds": 4,
  "property_baths": 3,
  "property_image_url": "https://cdn.skye.com/properties/880e8400/hero.jpg",
  "deep_link": "/properties/880e8400-e29b-41d4-a716-446655440003",
  "notification_id": "notif-uuid-004",
  "timestamp": "2026-03-02T11:30:00Z"
}
```

**Notification Category Registration:**

```javascript
{
  id: 'PROPERTY_MATCH',
  actions: [
    {
      id: 'view_property',
      title: 'View Property',
      foreground: true,
    },
    {
      id: 'share_with_client',
      title: 'Share with Client',
      foreground: true,
    },
  ],
}
```

**Action Handlers:**

| Action ID | Behavior | Opens App? |
|---|---|---|
| `view_property` | Navigate to PropertyDetail screen for the matched property. | Yes |
| `share_with_client` | Navigate to PropertyDetail screen with share sheet pre-opened, pre-populated with the matched contact. | Yes |

**Rich Notification (Notification Service Extension):**

Property match notifications use `mutable-content: 1` to trigger a Notification Service Extension that downloads and attaches the property hero image.

Implementation requirements:
1. Create a Notification Service Extension target in Xcode named `SkyeNotificationService`.
2. Configure App Groups shared between the main app and extension: `group.com.skye.app`.
3. In the extension's `didReceive(_:withContentHandler:)`:
   - Check for `property_image_url` in the notification's `userInfo`.
   - Download the image (with a 25-second timeout -- Apple gives ~30 seconds total).
   - Save to a temporary file with appropriate UTI (`.jpg`).
   - Create `UNNotificationAttachment` from the file.
   - Set `bestAttemptContent.attachments = [attachment]`.
   - Call `contentHandler(bestAttemptContent)`.
4. In `serviceExtensionTimeWillExpire()`: deliver the notification without the image (graceful degradation).

**Image requirements:**
- Maximum recommended size: 10MB (iOS limit), but target under 1MB for fast download.
- The CDN should serve property images at a notification-optimized resolution: 1024x1024 pixels max.
- Supported formats: JPEG, PNG, GIF (animated).

**Batching strategy:** If multiple properties match simultaneously (e.g., 5 new listings), do not send 5 separate notifications. Instead, batch into a single summary notification:
- Title: "5 New Property Matches"
- Body: "New listings match Sarah Chen's criteria"
- Deep link: `/contacts/{contact_id}?tab=matches`

Threshold: batch if more than 2 matches arrive within a 5-minute window for the same contact.

---

#### 2.5 AI Completion

**Trigger:** Long-running AI task completes (market analysis, CMA report, email draft, listing description). The AI task worker emits a completion event that triggers the push.

**APNs Payload:**

```json
{
  "aps": {
    "alert": {
      "title": "Skye",
      "body": "Your market analysis is ready to view"
    },
    "badge": 9,
    "sound": "default",
    "thread-id": "ai-completions",
    "category": "AI_COMPLETION",
    "mutable-content": 0,
    "interruption-level": "active",
    "relevance-score": 0.5
  },
  "type": "ai_complete",
  "chat_id": "990e8400-e29b-41d4-a716-446655440004",
  "task_type": "market_analysis",
  "deep_link": "/chat/990e8400-e29b-41d4-a716-446655440004",
  "notification_id": "notif-uuid-005",
  "timestamp": "2026-03-02T12:15:00Z"
}
```

**Notification Category Registration:**

```javascript
{
  id: 'AI_COMPLETION',
  actions: [
    {
      id: 'view_result',
      title: 'View Result',
      foreground: true,
    },
    {
      id: 'dismiss_ai',
      title: 'Dismiss',
      foreground: false,
    },
  ],
}
```

**Action Handlers:**

| Action ID | Behavior | Opens App? |
|---|---|---|
| `view_result` | Navigate to Chat screen, load the specific conversation with the completed AI result. | Yes |
| `dismiss_ai` | Mark notification as read. Decrement badge. | No |

**Task type display mapping:**

| `task_type` value | Display string |
|---|---|
| `market_analysis` | "market analysis" |
| `cma_report` | "CMA report" |
| `email_draft` | "email draft" |
| `listing_description` | "listing description" |
| `buyer_tour_plan` | "buyer tour plan" |
| `offer_summary` | "offer summary" |

---

### 3. Notification Permission Flow

#### 3.1 Pre-Permission Priming Screen

**Purpose:** iOS only allows one system-level notification permission prompt per app installation. If the user denies it, the app cannot programmatically re-trigger it. A pre-permission "priming" screen educates the user before the system prompt appears, dramatically increasing opt-in rates (industry average: 50-60% without priming, 70-80% with priming).

**Screen design specification:**

```
+--------------------------------------------------+
|                                                  |
|              [Skye Logo / Icon]                  |
|                                                  |
|         Stay on top of every deal                |
|                                                  |
|  Skye sends you:                                 |
|                                                  |
|  [Bell icon]  Daily morning briefings            |
|               Your schedule and new leads         |
|               at a glance                         |
|                                                  |
|  [Clock icon] Follow-up reminders                |
|               Never let a lead go cold            |
|                                                  |
|  [Home icon]  New property matches               |
|               Instant alerts when listings        |
|               match your clients' criteria        |
|                                                  |
|  [Check icon] Task reminders                     |
|               Stay ahead of deadlines             |
|                                                  |
|                                                  |
|  +--------------------------------------------+  |
|  |         Enable Notifications               |  |
|  +--------------------------------------------+  |
|                                                  |
|              Not Now                             |
|                                                  |
+--------------------------------------------------+
```

**Component:** `NotificationPrimingScreen`

**Display triggers (show priming screen when ALL of the following are true):**
1. Notification permission status is `not_determined` (never asked).
2. User has completed onboarding/authentication.
3. User has not dismissed the priming screen in the last 3 days.
4. User has not dismissed the priming screen more than 3 times total.

**Optimal timing for first display:** Show after the user's first meaningful interaction with the app -- specifically, after they add their first contact OR after their first chat conversation with Skye. This establishes value before asking for commitment.

#### 3.2 Permission Request Flow

```
[User taps "Enable Notifications"]
    |
    v
[iOS system permission dialog appears]
    |
    +---> User taps "Allow"
    |         |
    |         v
    |    [Permission granted]
    |    [Register for remote notifications]
    |    [Token received -> POST /api/push/subscribe]
    |    [Subscribe to all notification categories by default]
    |    [Store permission_granted_at timestamp]
    |    [Navigate to next screen / dismiss priming]
    |
    +---> User taps "Don't Allow"
              |
              v
         [Permission denied]
         [Store permission_denied_at timestamp]
         [Increment denial_count]
         [Navigate to next screen / dismiss priming]
         [DO NOT show system prompt again -- iOS blocks it]
```

#### 3.3 "Not Now" Handling

```
[User taps "Not Now"]
    |
    v
[Store priming_dismissed_at timestamp]
[Increment priming_dismiss_count]
[Dismiss priming screen]
    |
    v
[Re-show priming screen when:]
  - 3 days have elapsed since last dismissal
  - AND user opens the app
  - AND priming_dismiss_count < 3
  - OR user performs a high-value action (adds 5th contact,
    receives first AI result) that makes notifications more relevant
```

#### 3.4 Permission Denied Handling

Once iOS permission is denied, the app cannot re-request. The app must guide users to Settings.

**Settings redirect UI (shown in Notification Preferences screen):**

```
+--------------------------------------------------+
|  Notifications are disabled                       |
|                                                  |
|  To receive follow-up reminders, briefings,      |
|  and property alerts, enable notifications        |
|  in your device settings.                         |
|                                                  |
|  +--------------------------------------------+  |
|  |         Open Settings                      |  |
|  +--------------------------------------------+  |
+--------------------------------------------------+
```

**Implementation:**

```javascript
import { Linking, Platform } from 'react-native';

function openNotificationSettings() {
  if (Platform.OS === 'ios') {
    Linking.openURL('app-settings:');
    // This opens the app's Settings page in iOS Settings,
    // where the user can toggle notifications.
  }
}
```

**When to show the settings redirect:**
- On the Notification Preferences screen (always visible if permission is denied).
- As an inline banner at the top of the People tab, dismissible, shown once per session.
- When the user explicitly tries to enable a notification type in preferences while permissions are denied.

#### 3.5 Notification Preferences Screen

**Location:** Settings > Notification Preferences

**UI Specification:**

```
+--------------------------------------------------+
|  Notification Preferences                         |
+--------------------------------------------------+
|                                                  |
|  [If permission denied: Settings redirect banner] |
|                                                  |
|  DAILY BRIEFING                                  |
|  +----------------------------------------------+|
|  | Morning Briefing            [Toggle: ON/OFF] ||
|  | Delivery Time               [7:30 AM  >]    ||
|  +----------------------------------------------+|
|                                                  |
|  CONTACTS & FOLLOW-UPS                           |
|  +----------------------------------------------+|
|  | Engagement Reminders        [Toggle: ON/OFF] ||
|  | Reminder Frequency          [Default   >]    ||
|  +----------------------------------------------+|
|                                                  |
|  TASKS                                           |
|  +----------------------------------------------+|
|  | Overdue Task Alerts         [Toggle: ON/OFF] ||
|  +----------------------------------------------+|
|                                                  |
|  PROPERTIES                                      |
|  +----------------------------------------------+|
|  | New Property Matches        [Toggle: ON/OFF] ||
|  +----------------------------------------------+|
|                                                  |
|  AI ASSISTANT                                    |
|  +----------------------------------------------+|
|  | AI Task Completions         [Toggle: ON/OFF] ||
|  +----------------------------------------------+|
|                                                  |
+--------------------------------------------------+
```

**Persistence:** Preferences are stored on the server (via `PUT /api/users/me/notification-preferences`) and cached locally. The server checks user preferences before sending any push notification.

**API contract:**

```typescript
// PUT /api/users/me/notification-preferences
interface NotificationPreferences {
  briefing_enabled: boolean;           // Default: true
  briefing_time: string;               // Default: "07:30" (HH:mm, user's local time)
  briefing_timezone: string;           // Default: device timezone (e.g., "America/Los_Angeles")
  engagement_enabled: boolean;         // Default: true
  engagement_frequency: 'default' | 'daily_digest' | 'weekly_digest';
  tasks_enabled: boolean;              // Default: true
  properties_enabled: boolean;         // Default: true
  ai_completions_enabled: boolean;     // Default: true
}

// GET /api/users/me/notification-preferences
// Returns the same shape with current values.
```

**Default state:** All notification types are enabled on first registration. Users opt out explicitly.

---

### 4. Deep Link Resolution Engine

#### 4.1 URI Scheme and Universal Links

Skye supports two deep linking mechanisms:

| Mechanism | Format | Use Case |
|---|---|---|
| Custom URI scheme | `skyeapp://path` | Push notification tap handling, in-app cross-linking |
| Universal Links | `https://app.skye.com/path` | Web-to-app transitions, shared links, email campaigns |

**Custom URI scheme configuration (Info.plist):**

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>skyeapp</string>
    </array>
    <key>CFBundleURLName</key>
    <string>com.skye.app</string>
  </dict>
</array>
```

**Universal Links configuration:**

1. **Associated Domains entitlement (in Xcode):**
   Add `applinks:app.skye.com` to the Associated Domains capability.

2. **Apple App Site Association file** (hosted at `https://app.skye.com/.well-known/apple-app-site-association`):

```json
{
  "applinks": {
    "details": [
      {
        "appIDs": [
          "9GQ7JHT2C4.com.skye.app"
        ],
        "components": [
          {
            "/": "/contacts/*",
            "comment": "Contact detail pages"
          },
          {
            "/": "/properties/*",
            "comment": "Property detail pages"
          },
          {
            "/": "/chat",
            "comment": "Chat screen"
          },
          {
            "/": "/chat/*",
            "comment": "Specific chat conversations"
          },
          {
            "/": "/settings/*",
            "exclude": true,
            "comment": "Settings pages should open in browser"
          }
        ]
      }
    ]
  }
}
```

**AASA file requirements:**
- Must be served over HTTPS with no redirects.
- Content-Type: `application/json`.
- Maximum size: 128KB uncompressed.
- No `.json` file extension.
- Apple CDN caches the file; updates may take 24 hours to propagate. Use `?mode=developer` in the Associated Domains entitlement during development for immediate refresh.

#### 4.2 Deep Link Routing Table

| Deep Link Path | Screen | Parameters | Notes |
|---|---|---|---|
| `/contacts/{id}` | `ContactDetailScreen` | `contactId: string` | Loads contact profile, defaults to Overview tab |
| `/contacts/{id}?tab=tasks` | `ContactDetailScreen` | `contactId: string`, `initialTab: 'tasks'` | Opens contact with Tasks tab selected |
| `/contacts/{id}?tab=matches` | `ContactDetailScreen` | `contactId: string`, `initialTab: 'matches'` | Opens contact with Property Matches tab selected |
| `/properties/{id}` | `PropertyDetailScreen` | `propertyId: string` | Loads property listing detail |
| `/chat` | `ChatScreen` | (none) | Opens main chat interface |
| `/chat?briefing=today` | `ChatScreen` | `briefingDate: 'today'` | Opens chat and auto-loads today's morning briefing |
| `/chat/{id}` | `ChatScreen` | `chatId: string` | Opens specific chat conversation |
| `/` | `HomeScreen` | (none) | Fallback: opens app to default tab |

#### 4.3 React Navigation Linking Configuration

```javascript
// src/navigation/linking.js

const linking = {
  prefixes: [
    'skyeapp://',
    'https://app.skye.com',
  ],

  config: {
    screens: {
      // Main tab navigator
      MainTabs: {
        screens: {
          PeopleTab: {
            screens: {
              ContactDetail: {
                path: 'contacts/:contactId',
                parse: {
                  contactId: (id) => id,
                },
              },
            },
          },
          PropertiesTab: {
            screens: {
              PropertyDetail: {
                path: 'properties/:propertyId',
                parse: {
                  propertyId: (id) => id,
                },
              },
            },
          },
          ChatTab: {
            path: 'chat',
            screens: {
              ChatConversation: {
                path: ':chatId',
                parse: {
                  chatId: (id) => id,
                },
              },
            },
          },
        },
      },

      // Auth screens (for auth gate handling)
      Auth: {
        screens: {
          Login: 'login',
        },
      },
    },
  },

  // Custom URL resolution for push notification deep links
  getInitialURL: async () => {
    // Check if the app was opened from a push notification
    const initialNotification = await notifee.getInitialNotification();
    if (initialNotification?.notification?.data?.deep_link) {
      return 'skyeapp:/' + initialNotification.notification.data.deep_link;
    }

    // Check for standard deep link
    const url = await Linking.getInitialURL();
    return url;
  },

  subscribe: (listener) => {
    // Listen for deep link events while app is running
    const linkingSubscription = Linking.addEventListener('url', ({ url }) => {
      listener(url);
    });

    // Listen for notification taps while app is running
    const notifeeUnsubscribe = notifee.onForegroundEvent(({ type, detail }) => {
      if (type === EventType.PRESS && detail.notification?.data?.deep_link) {
        listener('skyeapp:/' + detail.notification.data.deep_link);
      }
    });

    return () => {
      linkingSubscription.remove();
      notifeeUnsubscribe();
    };
  },
};

export default linking;
```

#### 4.4 Auth Gate for Deep Links

Deep links may arrive when the user is not authenticated (e.g., token expired, app not opened in weeks). The app must preserve the deep link intent through the authentication flow.

**Implementation:**

```javascript
// src/navigation/AuthGate.js

const PENDING_DEEP_LINK_KEY = 'pending_deep_link';

class DeepLinkAuthGate {
  /**
   * Called when a deep link arrives and user is not authenticated.
   * Stores the intent for post-auth resolution.
   */
  static async preserveIntent(deepLinkUrl) {
    await AsyncStorage.setItem(PENDING_DEEP_LINK_KEY, deepLinkUrl);
  }

  /**
   * Called after successful authentication.
   * Returns the preserved deep link (if any) and clears it.
   */
  static async resolveIntent() {
    const pendingUrl = await AsyncStorage.getItem(PENDING_DEEP_LINK_KEY);
    if (pendingUrl) {
      await AsyncStorage.removeItem(PENDING_DEEP_LINK_KEY);
      return pendingUrl;
    }
    return null;
  }
}
```

**Flow:**

```
[Deep link arrives (push tap, universal link, etc.)]
    |
    v
[Check auth state]
    |
    +---> Authenticated
    |         |
    |         v
    |    [Resolve deep link normally]
    |    [Navigate to target screen]
    |
    +---> Not Authenticated
              |
              v
         [Preserve deep link intent in AsyncStorage]
         [Show login/auth screen]
              |
              v
         [User authenticates successfully]
              |
              v
         [Check for preserved intent]
              |
              +---> Intent exists
              |         |
              |         v
              |    [Navigate to deep link target]
              |    [Clear preserved intent]
              |
              +---> No intent
                        |
                        v
                   [Navigate to default home screen]
```

**TTL for preserved intents:** Deep link intents stored in AsyncStorage expire after 1 hour. If the user does not authenticate within that window, the intent is discarded to avoid navigating to stale content.

#### 4.5 Entity Not Found Handling

When a deep link points to a deleted or inaccessible entity:

```javascript
// In the target screen's data fetching logic

async function loadContact(contactId) {
  try {
    const contact = await api.get(`/api/contacts/${contactId}`);
    return contact;
  } catch (error) {
    if (error.status === 404) {
      // Entity not found -- navigate to parent tab and show toast
      navigation.navigate('PeopleTab', { screen: 'ContactList' });
      showToast({
        type: 'info',
        message: 'This contact is no longer available',
        duration: 3000,
      });
      return null;
    }
    throw error; // Re-throw other errors for error boundary
  }
}
```

**Fallback routing table:**

| Target Screen | Fallback on 404 | Toast Message |
|---|---|---|
| `ContactDetailScreen` | Navigate to People tab (contact list) | "This contact is no longer available" |
| `PropertyDetailScreen` | Navigate to Properties tab | "This property listing is no longer available" |
| `ChatConversation` | Navigate to Chat tab (conversation list) | "This conversation is no longer available" |

#### 4.6 Cold Start vs. Warm Start Deep Link Handling

| Scenario | Behavior |
|---|---|
| **Cold start** (app killed, user taps notification) | App launches from scratch. Deep link URL is captured via `getInitialURL()`. Navigation must wait for: (1) app initialization complete, (2) auth token validation complete, (3) navigation container mounted. Only then navigate to deep link target. |
| **Warm start** (app in background, user taps notification) | App is already initialized and authenticated. Deep link is received via the `subscribe` listener. Navigation happens immediately. |
| **Foreground** (app is active, notification arrives) | Notification is displayed as a banner (foreground presentation). User taps it. Deep link is received via Notifee's `onForegroundEvent`. Navigation happens immediately. |

**Cold start initialization sequence:**

```
[App process started by iOS]
    |
    v
[1. React Native bridge initializes]
    |
    v
[2. App component mounts]
    |
    v
[3. Auth token loaded from secure storage]
    |
    v
[4. Auth token validated (silent refresh if expired)]
    |
    +---> Token valid
    |         |
    |         v
    |    [5. NavigationContainer mounts with linking config]
    |    [6. getInitialURL() resolves the deep link]
    |    [7. React Navigation processes the URL]
    |    [8. User sees target screen]
    |
    +---> Token invalid / expired
              |
              v
         [5. Show auth screen]
         [6. Deep link preserved via AuthGate]
         [7. After auth, resolve preserved intent]
```

**Critical implementation detail:** The `NavigationContainer`'s `linking` prop handles the cold start case automatically via `getInitialURL`. However, the `fallback` prop must render a loading/splash screen while initialization is in progress:

```jsx
<NavigationContainer
  linking={linking}
  fallback={<SplashScreen />}
>
  {/* ... */}
</NavigationContainer>
```

---

### 5. Badge Management

#### 5.1 Badge Count Strategy

The badge count on Skye's app icon represents the total number of actionable, unread notifications across all notification types.

**Badge count composition:**

| Notification Type | Contributes to Badge? | Cleared When |
|---|---|---|
| Engagement Alert | Yes | User views the contact OR dismisses the notification |
| Morning Briefing | Yes (1 per day max) | User opens the briefing in chat |
| Overdue Task | Yes | User marks task complete, snoozes, or views task |
| Property Match | Yes | User views the property |
| AI Completion | Yes | User views the AI result |

#### 5.2 Server-Driven Badge Count

**Every push notification includes the current badge count** in the `aps.badge` field. The server is the source of truth for badge count.

**Server-side badge calculation:**

```sql
SELECT
  (SELECT COUNT(*) FROM engagement_alerts
   WHERE user_id = $1 AND status = 'pending' AND dismissed_at IS NULL)
  +
  (SELECT COUNT(*) FROM morning_briefings
   WHERE user_id = $1 AND date = CURRENT_DATE AND read_at IS NULL)
  +
  (SELECT COUNT(*) FROM task_notifications
   WHERE user_id = $1 AND status = 'pending' AND dismissed_at IS NULL)
  +
  (SELECT COUNT(*) FROM property_match_notifications
   WHERE user_id = $1 AND status = 'pending' AND viewed_at IS NULL)
  +
  (SELECT COUNT(*) FROM ai_completion_notifications
   WHERE user_id = $1 AND status = 'pending' AND viewed_at IS NULL)
AS total_badge_count;
```

**This query runs before every push send** to include the accurate badge count. For efficiency, maintain a materialized `notification_badge_count` column on the user table, updated via database triggers or application-level increment/decrement operations.

#### 5.3 Client-Side Badge Updates

```javascript
import notifee from '@notifee/react-native';

// When a notification action decrements the badge (e.g., "Dismiss", "Mark Complete")
async function handleBackgroundAction(action, notification) {
  switch (action.id) {
    case 'dismiss':
    case 'mark_complete':
    case 'dismiss_briefing':
    case 'dismiss_ai':
      await notifee.decrementBadgeCount(1);
      break;
    // ... other actions
  }
}

// When user opens a screen that clears relevant badges
async function onScreenFocus(screenName, params) {
  switch (screenName) {
    case 'ContactDetail':
      // Clear engagement alert badge for this contact
      await api.post(`/api/notifications/mark-read`, {
        type: 'engagement',
        contact_id: params.contactId,
      });
      // Server returns updated badge count
      const { badge_count } = await api.get('/api/users/me/badge-count');
      await notifee.setBadgeCount(badge_count);
      break;

    case 'ChatScreen':
      if (params?.briefingDate) {
        await api.post(`/api/notifications/mark-read`, {
          type: 'briefing',
          date: params.briefingDate,
        });
      }
      if (params?.chatId) {
        await api.post(`/api/notifications/mark-read`, {
          type: 'ai_complete',
          chat_id: params.chatId,
        });
      }
      const updated = await api.get('/api/users/me/badge-count');
      await notifee.setBadgeCount(updated.badge_count);
      break;
  }
}
```

#### 5.4 Badge Sync on App Foreground

When the app returns to the foreground, the badge count may be stale (e.g., the user dismissed a notification from the lock screen but the server-side count was not synced). Sync on every foreground event.

```javascript
import { AppState } from 'react-native';
import notifee from '@notifee/react-native';

let appState = AppState.currentState;

AppState.addEventListener('change', async (nextAppState) => {
  if (appState.match(/inactive|background/) && nextAppState === 'active') {
    // App has come to foreground -- sync badge
    try {
      const { badge_count } = await api.get('/api/users/me/badge-count');
      await notifee.setBadgeCount(badge_count);
    } catch (error) {
      // Network failure -- keep existing badge count
      console.warn('Badge sync failed:', error);
    }
  }
  appState = nextAppState;
});
```

#### 5.5 Badge Clear on Full App Open

**Policy decision:** Do NOT auto-clear the entire badge on app open. The badge represents actionable items. Only clear badges for items the user actually views or dismisses. This ensures the badge remains a meaningful signal.

**Exception:** If the user explicitly taps "Mark All as Read" in a future in-app notification center, clear the entire badge: `await notifee.setBadgeCount(0)`.

---

### 6. Notification Center Behavior

#### 6.1 Notification Grouping

iOS groups notifications using the `thread-id` field. Skye uses the following grouping strategy:

| Thread ID | Groups | Summary Text |
|---|---|---|
| `engagement-alerts` | All engagement/follow-up reminders | "%u+ contacts need follow-up" |
| `morning-briefing` | Morning briefings (typically only 1/day) | (No grouping needed) |
| `overdue-tasks` | All overdue task notifications | "%u+ overdue tasks" |
| `property-matches` | All property match notifications | "%u+ new property matches" |
| `ai-completions` | All AI task completion notifications | "%u AI results ready" |

**Summary text configuration (Notifee category registration):**

```javascript
{
  id: 'ENGAGEMENT_ALERT',
  summaryFormat: '%u+ contacts need follow-up',
  actions: [/* ... */],
}
```

When 4 or more notifications stack in a group, iOS displays the summary text with the count.

#### 6.2 Notification Actions (Background Processing)

Actions that execute without opening the app require a background event handler.

```javascript
// src/notifications/backgroundHandler.js

import notifee, { EventType } from '@notifee/react-native';

notifee.onBackgroundEvent(async ({ type, detail }) => {
  const { notification, pressAction } = detail;

  if (type === EventType.ACTION_PRESS) {
    const { data } = notification;

    switch (pressAction.id) {
      // Engagement Alert Actions
      case 'call_now':
        // Foreground action -- handled by navigation
        break;

      case 'snooze_1d':
        await fetch(`${API_BASE}/api/contacts/${data.contact_id}/snooze`, {
          method: 'POST',
          headers: { Authorization: `Bearer ${await getToken()}` },
          body: JSON.stringify({ duration: '1d' }),
        });
        await notifee.decrementBadgeCount(1);
        await notifee.cancelNotification(notification.id);
        break;

      case 'dismiss':
        await fetch(
          `${API_BASE}/api/notifications/${data.notification_id}/dismiss`,
          {
            method: 'POST',
            headers: { Authorization: `Bearer ${await getToken()}` },
          }
        );
        await notifee.decrementBadgeCount(1);
        await notifee.cancelNotification(notification.id);
        break;

      // Overdue Task Actions
      case 'mark_complete':
        await fetch(`${API_BASE}/api/tasks/${data.task_id}/complete`, {
          method: 'POST',
          headers: { Authorization: `Bearer ${await getToken()}` },
        });
        await notifee.decrementBadgeCount(1);
        await notifee.cancelNotification(notification.id);
        break;

      case 'snooze_task':
        await fetch(`${API_BASE}/api/tasks/${data.task_id}/snooze`, {
          method: 'POST',
          headers: { Authorization: `Bearer ${await getToken()}` },
          body: JSON.stringify({ duration: '1d' }),
        });
        await notifee.decrementBadgeCount(1);
        await notifee.cancelNotification(notification.id);
        break;

      // Morning Briefing
      case 'dismiss_briefing':
        await fetch(
          `${API_BASE}/api/notifications/${data.notification_id}/dismiss`,
          {
            method: 'POST',
            headers: { Authorization: `Bearer ${await getToken()}` },
          }
        );
        await notifee.decrementBadgeCount(1);
        await notifee.cancelNotification(notification.id);
        break;

      // AI Completion
      case 'dismiss_ai':
        await fetch(
          `${API_BASE}/api/notifications/${data.notification_id}/dismiss`,
          {
            method: 'POST',
            headers: { Authorization: `Bearer ${await getToken()}` },
          }
        );
        await notifee.decrementBadgeCount(1);
        await notifee.cancelNotification(notification.id);
        break;
    }
  }

  if (type === EventType.DISMISSED) {
    // User swiped away the notification (not via an action button)
    // Do NOT decrement badge -- item is still actionable
    // Optionally log this event for analytics
  }
});
```

**Critical detail:** Background action handlers run in a headless JS context. They do not have access to React Navigation or UI components. They can only perform network requests and Notifee API calls. Auth tokens must be retrieved from secure storage directly (not from React context/state).

#### 6.3 Rich Notifications (Notification Content Extension)

For property match notifications, a Notification Content Extension provides a rich, expanded view with the property image, address, price, and key details.

**Implementation requires a native iOS target:**

1. **Create target:** In Xcode, File > New > Target > Notification Content Extension. Name: `SkyeNotificationContent`.
2. **Configure Info.plist** of the extension:

```xml
<key>NSExtension</key>
<dict>
  <key>NSExtensionAttributes</key>
  <dict>
    <key>UNNotificationExtensionCategory</key>
    <string>PROPERTY_MATCH</string>
    <key>UNNotificationExtensionInitialContentSizeRatio</key>
    <real>0.5</real>
    <key>UNNotificationExtensionDefaultContentHidden</key>
    <true/>
  </dict>
  <key>NSExtensionPointIdentifier</key>
  <string>com.apple.usernotifications.content-extension</string>
  <key>NSExtensionPrincipalClass</key>
  <string>NotificationViewController</string>
</dict>
```

3. **NotificationViewController** layout:
   - Property hero image (full width, 16:9 aspect ratio)
   - Address label (bold, 17pt)
   - Price label (bold, 20pt, Skye brand color)
   - Bed/bath/sqft details in a horizontal stack
   - "Matched for: {contact_name}" subtitle

4. **Data flow:** The notification's `userInfo` contains `property_image_url`, `property_address`, `property_price`, `property_beds`, `property_baths`, `contact_name`. The Notification Service Extension (Section 2.4) downloads the image and attaches it. The Content Extension renders the custom UI.

**Scope note:** The Notification Content Extension is a native iOS component written in Swift. It cannot use React Native views. It is a standalone UI rendered by iOS when the user long-presses or expands the notification.

#### 6.4 Notification Sounds

| Notification Type | Sound | Specification |
|---|---|---|
| Engagement Alert | System default | `"sound": "default"` |
| Morning Briefing | Custom gentle chime | `"sound": { "name": "skye_morning_chime.caf", "volume": 0.7 }` |
| Overdue Task | System default | `"sound": "default"` |
| Property Match | System default | `"sound": "default"` |
| AI Completion | System default | `"sound": "default"` |

**Custom sound file (`skye_morning_chime.caf`) specification:**

| Property | Value |
|---|---|
| Format | Core Audio Format (CAF) |
| Encoding | Linear PCM or IMA4 |
| Sample rate | 44,100 Hz |
| Bit depth | 16-bit |
| Channels | Mono (1 channel) |
| Duration | 3 seconds |
| Character | Soft ascending chime -- three gentle bell tones ascending in pitch. Non-jarring, professional, warm. |
| File size | Target under 100KB |
| Bundle location | Main app bundle, added to "Copy Bundle Resources" build phase in Xcode |

**Conversion command (for audio team):**

```bash
afconvert input.wav skye_morning_chime.caf -d LEI16 -f caff -c 1
```

#### 6.5 Do Not Disturb and Scheduled Delivery

iOS handles DND automatically -- notifications are silenced during DND/Focus modes. Skye should design around this:

**Morning briefing scheduling:**
- The morning briefing push is scheduled based on the user's preferred delivery time (stored server-side).
- Default: 7:30 AM local time.
- The backend scheduler fans out briefing pushes per-user based on their timezone and preference.
- If the user's preferred time falls during typical sleeping hours (11 PM - 5 AM) due to timezone misconfiguration, cap at 7:00 AM local time.
- The briefing is generated and cached before the push is sent, so when the user taps "View Briefing," the content loads instantly.

**Interruption levels (iOS 15+):**
- `time-sensitive`: Used for engagement alerts and overdue tasks. These can break through Focus modes if the user has allowed Time Sensitive notifications for Skye. Users must explicitly enable this in iOS Settings > Focus > [Focus Name] > Apps > Skye > Time Sensitive Notifications.
- `active`: Used for morning briefings, property matches, AI completions. Standard delivery, respects DND/Focus.
- `passive`: Not used by Skye (these are delivered silently without any alert).

**Skye should NOT use `critical` interruption level.** Critical alerts require a special entitlement from Apple (granted only for health/safety/security apps) and always play sound even at full volume during DND. Real estate notifications do not qualify.

---

### 7. Silent Push and Background Refresh

#### 7.1 Silent Push for Background Data Sync

Silent push notifications (`content-available: 1`) wake the app in the background to perform data sync operations without displaying a visible notification to the user.

**Silent push payload format:**

```json
{
  "aps": {
    "content-available": 1
  },
  "sync_type": "contacts_updated",
  "sync_data": {
    "updated_contact_ids": ["uuid1", "uuid2"],
    "timestamp": "2026-03-02T10:00:00Z"
  }
}
```

**Required APNs headers for silent push:**

| Header | Value | Notes |
|---|---|---|
| `apns-push-type` | `background` | Required for silent push on iOS 13+ |
| `apns-priority` | `5` | Must be `5` (not `10`) for background push. Priority `10` will be rejected for background push type. |
| `apns-topic` | `com.skye.app` | Bundle ID |

**Important constraint:** A silent push payload must NOT include `alert`, `badge`, or `sound` in the `aps` dictionary. Including these alongside `content-available: 1` creates a "hybrid" push that may not reliably wake the app in the background.

#### 7.2 Silent Push Use Cases

| Use Case | Trigger | Sync Action |
|---|---|---|
| Contact data sync | Server-side contact update (e.g., CRM import, lead assignment) | Fetch updated contact records, update local cache |
| Badge count correction | Periodic (every 6 hours) or after batch server operations | Fetch current badge count, update app badge |
| Briefing pre-fetch | 30 minutes before scheduled briefing delivery | Download and cache briefing data so it loads instantly when the visible notification arrives |
| Property data refresh | New MLS feed ingestion complete | Fetch updated property listings for cached searches |
| Chat message sync | New message in an existing conversation (when real-time WebSocket is disconnected) | Fetch recent messages, update conversation list |

#### 7.3 Background Fetch Handler

```javascript
// Register headless task for silent push
import { AppRegistry } from 'react-native';

AppRegistry.registerHeadlessTask(
  'SkyeBackgroundSync',
  () => async (taskData) => {
    const { sync_type, sync_data } = taskData;

    try {
      switch (sync_type) {
        case 'contacts_updated':
          await syncContacts(sync_data.updated_contact_ids);
          break;

        case 'badge_sync':
          const { badge_count } = await api.get('/api/users/me/badge-count');
          await notifee.setBadgeCount(badge_count);
          break;

        case 'briefing_prefetch':
          await prefetchBriefing(sync_data.briefing_date);
          break;

        case 'property_refresh':
          await refreshPropertyCache();
          break;

        case 'chat_sync':
          await syncRecentMessages(sync_data.conversation_ids);
          break;

        default:
          console.warn(`Unknown sync type: ${sync_type}`);
      }
    } catch (error) {
      console.error('Background sync failed:', error);
    }

    // Must complete within ~30 seconds
    // iOS will terminate the process if exceeded
  }
);
```

#### 7.4 iOS Background Tasks Framework Integration

For more reliable background processing beyond silent push, register BGTaskScheduler tasks.

**Task registration (in native AppDelegate):**

```swift
import BackgroundTasks

func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions:
                   [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

  // Register background task identifiers
  BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "com.skye.app.refresh",
    using: nil
  ) { task in
    self.handleAppRefresh(task: task as! BGAppRefreshTask)
  }

  BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "com.skye.app.dataSync",
    using: nil
  ) { task in
    self.handleDataSync(task: task as! BGProcessingTask)
  }

  return true
}

func handleAppRefresh(task: BGAppRefreshTask) {
  // Schedule the next refresh
  scheduleAppRefresh()

  // Bridge to React Native for data fetch
  let bridge = RCTBridge.current()
  bridge?.enqueueJSCall("SkyeBackgroundTasks",
                         method: "onRefresh",
                         args: [],
                         completion: nil)

  task.expirationHandler = {
    // Clean up if time runs out
  }

  // Must call task.setTaskCompleted(success:) when done
}

func scheduleAppRefresh() {
  let request = BGAppRefreshTaskRequest(
    identifier: "com.skye.app.refresh")
  request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)
  try? BGTaskScheduler.shared.submit(request)
}
```

**Info.plist entries:**

```xml
<key>BGTaskSchedulerPermittedIdentifiers</key>
<array>
  <string>com.skye.app.refresh</string>
  <string>com.skye.app.dataSync</string>
</array>
```

**Xcode capabilities:** Ensure "Background Modes" includes both "Background fetch" and "Remote notifications."

#### 7.5 Battery Impact Mitigation

| Strategy | Implementation |
|---|---|
| Rate-limit silent pushes | Server sends a maximum of 2-3 silent pushes per hour per device. Batch updates instead of sending per-change. |
| Minimize network requests | Background sync fetches only delta/changed data, not full datasets. Use `If-Modified-Since` headers. |
| Cache aggressively | Store fetched data in local SQLite/MMKV. Only fetch what changed since `last_sync_timestamp`. |
| Respect Low Power Mode | Check `ProcessInfo.processInfo.isLowPowerModeEnabled` before heavy sync operations. If enabled, skip non-critical syncs. |
| Use BGProcessingTask for heavy work | Large data syncs (full MLS refresh) use `BGProcessingTask` which runs during charging/idle, not `BGAppRefreshTask`. |
| Complete handlers promptly | Always call `completionHandler(.newData)`, `.noData`, or `.failed` within 25 seconds. iOS tracks completion time and penalizes slow apps by reducing future background execution opportunities. |

#### 7.6 Testing Strategy for Background Push Delivery

| Method | How | Best For |
|---|---|---|
| **Xcode Debug** | In Xcode, while debugging: Debug > Simulate Background Fetch. Alternatively, edit scheme > Run > Options > Background Fetch checkbox. | Quick iteration during development. |
| **Push notification tester** | Use a tool like Pusher or a custom script with `@parse/node-apn` to send test silent pushes to a development device. | Testing actual APNs delivery path. |
| **Console.app** | Connect device to Mac, open Console.app, filter by `dasd` (Duet Activity Scheduler Daemon) and your app's bundle ID. Shows when iOS grants/denies background execution. | Diagnosing why background tasks are not running. |
| **Xcode device logs** | Filter device logs for `BackgroundTask` and `pushRegistry`. Shows silent push receipt and processing. | Verifying silent push delivery on real devices. |
| **TestFlight** | Distribute TestFlight builds to QA devices. Use sandbox APNs environment. Send test pushes from staging backend. | Production-like testing. Critical since simulators do not support push notifications. |
| **Simulating BGTaskScheduler** | Use `e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateLaunchForTaskWithIdentifier:@"com.skye.app.refresh"]` in Xcode LLDB debugger. | Testing BGTaskScheduler tasks without waiting for iOS to schedule them. |

**Testing checklist:**

- Silent push received while app is in foreground (should trigger handler immediately)
- Silent push received while app is in background (should wake app)
- Silent push received while app is terminated/killed by user (will NOT be delivered -- by Apple design)
- Silent push with `apns-priority: 5` delivers within 5-15 minutes (not guaranteed to be instant)
- Background task completes within 30 seconds
- Badge count updated correctly after background sync
- Completion handler called with correct result (`.newData`, `.noData`, `.failed`)
- Low Power Mode: background sync degrades gracefully (skips non-critical operations)
- No crashes in headless JS context (no UI references, no navigation calls)

---

### Appendix A: Notification Category Registration (Complete)

All notification categories must be registered at app launch, before any notifications arrive. Place this in the app initialization sequence, after authentication but before the main UI renders.

```javascript
// src/notifications/registerCategories.js

import notifee from '@notifee/react-native';

export async function registerNotificationCategories() {
  await notifee.setNotificationCategories([
    {
      id: 'ENGAGEMENT_ALERT',
      actions: [
        {
          id: 'call_now',
          title: 'Call Now',
          foreground: true,
        },
        {
          id: 'snooze_1d',
          title: 'Snooze 1 Day',
          foreground: false,
        },
        {
          id: 'dismiss',
          title: 'Dismiss',
          foreground: false,
          destructive: true,
        },
      ],
      summaryFormat: '%u+ contacts need follow-up',
    },
    {
      id: 'MORNING_BRIEFING',
      actions: [
        {
          id: 'view_briefing',
          title: 'View Briefing',
          foreground: true,
        },
        {
          id: 'dismiss_briefing',
          title: 'Dismiss',
          foreground: false,
        },
      ],
    },
    {
      id: 'OVERDUE_TASK',
      actions: [
        {
          id: 'mark_complete',
          title: 'Mark Complete',
          foreground: false,
        },
        {
          id: 'snooze_task',
          title: 'Snooze',
          foreground: false,
        },
        {
          id: 'view_task',
          title: 'View',
          foreground: true,
        },
      ],
      summaryFormat: '%u+ overdue tasks',
    },
    {
      id: 'PROPERTY_MATCH',
      actions: [
        {
          id: 'view_property',
          title: 'View Property',
          foreground: true,
        },
        {
          id: 'share_with_client',
          title: 'Share with Client',
          foreground: true,
        },
      ],
      summaryFormat: '%u+ new property matches',
    },
    {
      id: 'AI_COMPLETION',
      actions: [
        {
          id: 'view_result',
          title: 'View Result',
          foreground: true,
        },
        {
          id: 'dismiss_ai',
          title: 'Dismiss',
          foreground: false,
        },
      ],
    },
  ]);
}
```

---

### Appendix B: Server-Side Push Service (Complete)

```javascript
// src/services/pushService.js (backend)

const apn = require('@parse/node-apn');

class PushService {
  constructor() {
    this.apnsProvider = new apn.Provider({
      token: {
        key: process.env.APNS_AUTH_KEY_PATH,
        keyId: process.env.APNS_KEY_ID,
        teamId: process.env.APNS_TEAM_ID,
      },
      production: process.env.NODE_ENV === 'production',
    });
  }

  /**
   * Send a push notification to a specific user
   * across all their registered devices.
   */
  async sendToUser(userId, notificationPayload) {
    const subscriptions = await db.query(
      `SELECT * FROM push_subscriptions
       WHERE user_id = $1 AND is_active = true`,
      [userId]
    );

    const badgeCount = await this.getBadgeCount(userId);

    const results = await Promise.allSettled(
      subscriptions.map(async (sub) => {
        if (sub.platform === 'ios') {
          return this.sendAPNs(sub, notificationPayload, badgeCount);
        } else if (sub.platform === 'web') {
          return this.sendWebPush(sub, notificationPayload, badgeCount);
        }
      })
    );

    // Process results -- deactivate failed subscriptions
    for (let i = 0; i < results.length; i++) {
      const result = results[i];
      const sub = subscriptions[i];

      if (result.status === 'rejected' || result.value?.failed) {
        const failureReason = result.value?.reason || result.reason;
        if (this.isTokenInvalid(failureReason)) {
          await db.query(
            `UPDATE push_subscriptions
             SET is_active = false, updated_at = NOW()
             WHERE id = $1`,
            [sub.id]
          );
        }
      } else {
        await db.query(
          `UPDATE push_subscriptions SET last_used_at = NOW()
           WHERE id = $1`,
          [sub.id]
        );
      }
    }
  }

  /**
   * Send via APNs.
   */
  async sendAPNs(subscription, payload, badgeCount) {
    const notification = new apn.Notification();

    notification.alert = {
      title: payload.title,
      subtitle: payload.subtitle || undefined,
      body: payload.body,
    };
    notification.badge = badgeCount;
    notification.sound = payload.sound || 'default';
    notification.threadId = payload.threadId;
    notification.category = payload.category;
    notification.mutableContent = payload.mutableContent ? 1 : 0;
    notification.contentAvailable = payload.contentAvailable ? 1 : 0;

    if (payload.interruptionLevel) {
      notification.interruptionLevel = payload.interruptionLevel;
    }
    if (payload.relevanceScore !== undefined) {
      notification.relevanceScore = payload.relevanceScore;
    }

    notification.payload = {
      type: payload.type,
      deep_link: payload.deepLink,
      notification_id: payload.notificationId,
      timestamp: new Date().toISOString(),
      ...payload.customData,
    };

    notification.topic = process.env.APNS_BUNDLE_ID;
    notification.pushType =
      payload.contentAvailable ? 'background' : 'alert';
    notification.priority = payload.contentAvailable ? 5 : 10;

    if (payload.collapseId) {
      notification.collapseId = payload.collapseId;
    }
    if (payload.expiration) {
      notification.expiry =
        Math.floor(payload.expiration.getTime() / 1000);
    }

    const result = await this.apnsProvider.send(
      notification,
      subscription.token
    );

    if (result.failed.length > 0) {
      const failure = result.failed[0];
      return {
        failed: true,
        reason: failure.response?.reason || 'unknown',
        statusCode: failure.status,
      };
    }

    return { failed: false };
  }

  /**
   * Check if a failure reason indicates an invalid token.
   */
  isTokenInvalid(reason) {
    const invalidReasons = [
      'BadDeviceToken',
      'Unregistered',
      'ExpiredProviderToken',
      'TopicDisallowed',
      410,
      404,
    ];
    return invalidReasons.includes(reason);
  }

  /**
   * Get current badge count for a user.
   */
  async getBadgeCount(userId) {
    const result = await db.query(
      `SELECT badge_count FROM users WHERE id = $1`,
      [userId]
    );
    return result.rows[0]?.badge_count || 0;
  }

  /**
   * Graceful shutdown.
   */
  async shutdown() {
    await this.apnsProvider.shutdown();
  }
}

module.exports = new PushService();
```

---

### Appendix C: Notification Lifecycle Diagram

```
[SERVER EVENT]  (e.g., contact follow-up overdue)
       |
       v
[NOTIFICATION ENGINE]
  - Evaluate user preferences (is this type enabled?)
  - Calculate badge count
  - Build canonical notification payload
       |
       v
[PLATFORM FORMATTER]
  - APNs: Build APNs JSON with aps dict, custom data, headers
  - Web: Build Web Push JSON with VAPID signature
       |
       v
[DELIVERY]
  - APNs: HTTP/2 POST to api.push.apple.com (or sandbox)
  - Web: HTTP POST to push service endpoint
       |
       v
[iOS DEVICE RECEIVES PUSH]
       |
       +--> [Notification Service Extension]  (if mutable-content: 1)
       |        - Download property image
       |        - Modify content if needed
       |        - Pass to display
       |
       v
[iOS DISPLAYS NOTIFICATION]
  - Lock screen / Notification Center / Banner
  - Grouped by thread-id
  - Action buttons from category
       |
       v
[USER INTERACTS]
       |
       +--> [Tap notification body]
       |        - App opens (cold or warm start)
       |        - Deep link resolved
       |        - Navigate to target screen
       |
       +--> [Tap action button -- foreground]
       |        - App opens
       |        - Action handler runs with navigation
       |
       +--> [Tap action button -- background]
       |        - Background event handler fires
       |        - API call made (snooze, complete, dismiss)
       |        - Badge decremented
       |        - Notification removed
       |
       +--> [Swipe to dismiss]
       |        - Notification removed from center
       |        - Badge NOT decremented (item still actionable)
       |
       +--> [Ignore / not seen]
                - Notification persists in Notification Center
                - Badge persists on app icon
```

---

### Appendix D: Required Native Configuration Checklist

| Configuration | Location | Detail |
|---|---|---|
| Push Notifications capability | Xcode > Target > Signing & Capabilities | Enables APNs entitlement |
| Background Modes | Xcode > Target > Signing & Capabilities > Background Modes | Enable "Remote notifications" and "Background fetch" |
| Associated Domains | Xcode > Target > Signing & Capabilities | Add `applinks:app.skye.com` |
| Custom URL scheme | Info.plist > CFBundleURLTypes | `skyeapp` scheme |
| Notification Service Extension | Xcode > New Target | `SkyeNotificationService` for rich notifications |
| Notification Content Extension | Xcode > New Target | `SkyeNotificationContent` for property match rich UI |
| App Groups | Xcode > Main target + Extensions | `group.com.skye.app` -- shared between main app and extensions |
| Background Task identifiers | Info.plist > BGTaskSchedulerPermittedIdentifiers | `com.skye.app.refresh`, `com.skye.app.dataSync` |
| Custom sound file | Main bundle > Copy Bundle Resources | `skye_morning_chime.caf` |
| APNs environment entitlement | Automatically managed by Xcode | `aps-environment: production` (or `development` for debug) |

---

### Appendix E: Error Handling Matrix

| APNs Response | Status Code | Meaning | Action |
|---|---|---|---|
| `BadDeviceToken` | 400 | Token is invalid format | Deactivate subscription. Log error. |
| `Unregistered` | 410 | Device token is no longer active | Deactivate subscription immediately. |
| `PayloadTooLarge` | 413 | Payload exceeds 4,096 bytes | Log error. Truncate body text and retry. |
| `TooManyRequests` | 429 | Rate limited by APNs | Exponential backoff: 1s, 2s, 4s, 8s. Max 3 retries. |
| `InternalServerError` | 500 | APNs server error | Retry with exponential backoff. Max 5 retries. |
| `ServiceUnavailable` | 503 | APNs temporarily unavailable | Retry after `Retry-After` header value, or 60 seconds. |
| `ExpiredProviderToken` | 403 | JWT token expired (must refresh every 60 min) | Refresh JWT and retry immediately. `@parse/node-apn` handles this automatically. |
| `TopicDisallowed` | 400 | Bundle ID mismatch or APNs not enabled for key | Configuration error. Alert ops team. Do not retry. |
| `MissingTopic` | 400 | `apns-topic` header not set | Configuration error. Set topic to bundle ID. |
| `BadCertificate` | 403 | Certificate/key authentication failure | Configuration error. Verify .p8 key, key ID, team ID. |

---

### Appendix F: Analytics Events

Every notification interaction should be tracked for engagement analytics:

| Event | Properties | Trigger |
|---|---|---|
| `notification_sent` | `type`, `user_id`, `notification_id`, `platform` | Server successfully sends push |
| `notification_delivered` | `type`, `notification_id` | Notification Service Extension confirms delivery (requires mutable-content) |
| `notification_displayed` | `type`, `notification_id` | Notifee foreground display callback |
| `notification_tapped` | `type`, `notification_id`, `action_id` (null for body tap) | User taps notification or action button |
| `notification_dismissed` | `type`, `notification_id` | User swipes to dismiss |
| `notification_action` | `type`, `notification_id`, `action_id`, `action_result` | Background action completes (success/failure) |
| `deep_link_resolved` | `deep_link`, `target_screen`, `resolution_time_ms` | Deep link successfully navigated |
| `deep_link_failed` | `deep_link`, `error_type` (`not_found`, `auth_required`, `parse_error`) | Deep link resolution failed |
| `permission_prompted` | `screen` (`priming` or `system`) | Priming screen shown or system prompt triggered |
| `permission_result` | `granted: boolean`, `prompt_type` | User responds to priming or system prompt |

**Engagement metrics dashboard (derived):**
- Notification opt-in rate (permission granted / prompted)
- Delivery rate (delivered / sent)
- Tap-through rate (tapped / delivered) -- broken down by notification type
- Action completion rate (action / tapped) -- "Call Now" usage, "Mark Complete" usage
- Re-engagement rate (app sessions initiated via notification / total notifications)
- Time-to-action (seconds between notification delivery and user interaction)

---

This concludes the V2 specification for Push Notifications, Deep Linking, and Real-Time Updates. The spec covers the full lifecycle from server-side event generation through APNs delivery, client-side category registration, user interaction handling (foreground and background), deep link resolution with auth gating, badge management, rich notifications with property images, silent push background sync, and comprehensive analytics tracking.

**Key architectural decisions summarized:**
- APNs-direct over FCM for iOS-only, minimizing dependencies and maximizing payload control
- `@parse/node-apn` (v7.x) for backend APNs integration with token-based (.p8) authentication
- Notifee for client-side notification management (categories, badges, background events)
- Server-driven badge counts with client-side sync on foreground
- Pre-permission priming screen to maximize opt-in rates
- Auth-gated deep linking with preserved intent through login flow
- Silent push rate-limited to 2-3/hour with BGTaskScheduler for heavier background work

Sources:
- [Expo Notifications Documentation](https://docs.expo.dev/versions/latest/sdk/notifications/)
- [Notifee iOS Badges](https://notifee.app/react-native/docs/ios/badges/)
- [Notifee iOS Categories](https://notifee.app/react-native/docs/ios/categories/)
- [Apple: Establishing a Token-Based Connection to APNs](https://developer.apple.com/documentation/usernotifications/establishing-a-token-based-connection-to-apns)
- [Apple: Declaring Actionable Notification Types](https://developer.apple.com/documentation/usernotifications/declaring-your-actionable-notification-types)
- [Apple: UNNotificationCategory](https://developer.apple.com/documentation/usernotifications/unnotificationcategory)
- [Apple: Supporting Associated Domains](https://developer.apple.com/documentation/xcode/supporting-associated-domains)
- [React Navigation: Deep Linking](https://reactnavigation.org/docs/deep-linking/)
- [React Navigation: Configuring Links](https://reactnavigation.org/docs/configuring-links/)
- [@parse/node-apn on npm](https://www.npmjs.com/package/@parse/node-apn)
- [parse-community/node-apn on GitHub](https://github.com/parse-community/node-apn)
- [APNs Payload Cheatsheet](https://tanaschita.com/20230417-cheatsheet-for-anatomy-of-ios-push-notifications/)
- [iOS Silent Push Notifications Guide](https://medium.com/@vladosius/all-about-silent-push-notifications-step-by-step-guide-21e6a4eefd54)
- [React Native Deep Linking Guide](https://medium.com/@nikhithsomasani/react-native-deep-linking-that-actually-works-universal-links-cold-starts-oauth-aced7bffaa56)
- [iOS Notification Images with React Native Firebase](https://rnfirebase.io/messaging/ios-notification-images)

---


## Section 7: Performance, Infrastructure & Build Pipeline

---

### 1. App Startup Performance

#### 1.1 Cold Launch Budget Breakdown

**Target: <2,000ms from tap to interactive on iPhone 13 (A15 Bionic, iOS 16+).**

| Phase | Budget | What Happens | Failure Action |
|-------|--------|-------------|----------------|
| **Native Init** | <400ms | iOS launches process, loads dylibs, executes `main()`, initializes Expo runtime, displays native splash screen | Profile with Instruments > App Launch. Reduce embedded frameworks, strip unused architectures. |
| **JS Bundle Load** | <600ms | Hermes reads pre-compiled bytecode from disk into memory, parses top-level scope | Reduce bundle size. Ensure Hermes bytecode compilation is enabled (not plain JS parsing). |
| **React Tree Mount** | <400ms | Root component mounts, navigation container initializes, auth state resolves, active tab component renders | Defer non-critical providers. Lazy-load secondary tabs. Minimize providers in root tree. |
| **First Meaningful Paint** | <600ms | Splash screen hides. User sees real UI with either skeleton placeholders or cached data. App accepts touch input. | Audit paint waterfall with React DevTools Profiler. Reduce component depth. Use cached data for instant paint. |
| **Total** | **<2,000ms** | | CI gate: fail build if P95 cold launch exceeds 2,500ms on iPhone 13 test device. |

#### 1.2 Hermes Engine (Mandatory)

Hermes is non-negotiable. It is the only JavaScript engine that meets our startup budget.

```javascript
// app.json — Hermes must be explicitly enabled
{
  "expo": {
    "jsEngine": "hermes"
  }
}
```

**Why Hermes:**
- **Pre-compiled bytecode**: Hermes compiles JavaScript to bytecode at build time (`.hbc` files), eliminating JIT compilation at runtime. This reduces JS load phase from ~1,200ms (JSC) to ~400-600ms on iPhone 13.
- **Lower memory footprint**: Hermes uses ~30-50% less memory than JavaScriptCore for equivalent workloads. Garbage collector is optimized for mobile (GenGC with young/old generations).
- **Faster TTI (Time to Interactive)**: Bytecode execution begins immediately without parsing or compilation warmup.

**Hermes configuration requirements:**
- Verify bytecode compilation in EAS Build output logs: look for `Compiling JS to Hermes bytecode`.
- Confirm `.hbc` file is present in the app bundle, not `.jsbundle`.
- Enable Hermes debugging via Flipper for development builds (Flipper Hermes debugger plugin).
- If any dependency is incompatible with Hermes (rare in 2026), either find an alternative or file a compatibility shim — do not fall back to JSC.

#### 1.3 Bundle Size Budget

**Target: <15MB compressed JavaScript bundle. Hard ceiling: 18MB (triggers immediate audit).**

| Budget Category | Allocation | Notes |
|----------------|-----------|-------|
| React + React Native core | ~4MB | Non-negotiable baseline |
| Navigation (React Navigation) | ~1.5MB | Stack, bottom tabs, native stack |
| TanStack Query | ~0.5MB | Data fetching layer |
| UI components (Tamagui or equivalent) | ~2MB | Tree-shakeable required |
| Expo modules (SecureStore, Notifications, Image, etc.) | ~2MB | Only import used modules |
| App business logic + screens | ~3MB | Our code |
| Remaining headroom | ~2MB | Safety margin |
| **Total** | **<15MB** | |

**Bundle audit strategy:**

```bash
# Run on every PR via CI — part of the lint/check stage
npx react-native-bundle-visualizer

# OR using expo export and source-map-explorer
npx expo export --platform ios
npx source-map-explorer dist/_expo/static/js/ios/*.js
```

**Enforcement rules:**
1. CI step runs `npx expo export --platform ios` and measures the output `.hbc` file size.
2. If bundle exceeds 15MB compressed: warning annotation on PR.
3. If bundle exceeds 18MB compressed: CI fails with error `BUNDLE_SIZE_EXCEEDED`.
4. Every new dependency addition must include a bundle impact assessment in the PR description (before/after size).
5. Weekly automated report: top 20 largest modules by size, posted to team Slack channel.

**Anti-bloat practices:**
- Tree-shaking: ensure `sideEffects: false` in `package.json` for our code, use ESM imports exclusively (`import { specific } from 'lib'`, never `import * as lib`).
- No moment.js (use `date-fns` with tree-shaking, or `dayjs` at 2KB).
- No lodash full bundle (use `lodash-es` with specific imports or individual packages like `lodash.debounce`).
- Audit for duplicate dependencies: `npx depcheck` and `npm ls --all | grep <suspect>`.
- Images and assets: never bundle large images in JS. Use `expo-asset` for required assets, CDN for dynamic images.

#### 1.4 Lazy Loading Strategy

**Eagerly loaded screens** (included in initial bundle, mounted at startup):
| Screen | Reason |
|--------|--------|
| `AuthScreen` (Login/SignUp) | Must be immediately available if user is unauthenticated |
| `ChatScreen` (Home tab) | Primary landing screen — user sees this first on every launch |

**Lazily loaded screens** (loaded on first navigation, code-split via `React.lazy` + `Suspense`):
| Screen | Trigger | Estimated Load Time |
|--------|---------|-------------------|
| `PeopleScreen` (Contacts list) | User taps People tab | <200ms (prefetched after initial render) |
| `PeopleDetailScreen` | User taps a contact | <150ms |
| `PropertiesScreen` | User taps Properties tab | <200ms |
| `PropertyDetailScreen` | User taps a property | <150ms |
| `StudioScreen` | User taps Studio tab | <300ms (heaviest — includes rich media components) |
| `SettingsScreen` | User taps Settings tab | <100ms |
| `ProfileScreen` | User taps profile | <100ms |
| All modal screens | Triggered by user action | <150ms |

**Implementation:**

```typescript
// navigation/LazyScreens.ts
import { lazy } from 'react';

// Eagerly loaded — imported normally
export { ChatScreen } from '@/screens/Chat/ChatScreen';
export { AuthScreen } from '@/screens/Auth/AuthScreen';

// Lazily loaded — code-split
export const PeopleScreen = lazy(() => import('@/screens/People/PeopleScreen'));
export const PeopleDetailScreen = lazy(() => import('@/screens/People/PeopleDetailScreen'));
export const PropertiesScreen = lazy(() => import('@/screens/Properties/PropertiesScreen'));
export const PropertyDetailScreen = lazy(() => import('@/screens/Properties/PropertyDetailScreen'));
export const StudioScreen = lazy(() => import('@/screens/Studio/StudioScreen'));
export const SettingsScreen = lazy(() => import('@/screens/Settings/SettingsScreen'));
```

```typescript
// In navigator, wrap lazy screens with Suspense
import { Suspense } from 'react';
import { ScreenSkeleton } from '@/components/ScreenSkeleton';

function LazyScreen({ component: Component, ...props }) {
  return (
    <Suspense fallback={<ScreenSkeleton />}>
      <Component {...props} />
    </Suspense>
  );
}
```

**Background prefetching of lazy chunks:** After the app becomes interactive and the Chat screen is rendered, idle-time prefetch the People and Properties chunks (the next most likely navigations) using `import()` calls that are not awaited:

```typescript
// After splash hides and Chat renders:
requestIdleCallback(() => {
  import('@/screens/People/PeopleScreen');
  import('@/screens/Properties/PropertiesScreen');
});
```

#### 1.5 Splash Screen Strategy

**Requirement: Zero white flash. Zero layout jump. The native splash screen persists until the React Native app has rendered its first meaningful frame.**

**Implementation: `expo-splash-screen` with manual hide control.**

```javascript
// app.json
{
  "expo": {
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#0A0A0F"  // Matches app background — dark theme default
    },
    "ios": {
      "splash": {
        "image": "./assets/splash.png",
        "resizeMode": "contain",
        "backgroundColor": "#0A0A0F"
      }
    }
  }
}
```

**Manual hide control:**

```typescript
// App.tsx
import * as SplashScreen from 'expo-splash-screen';

// Prevent auto-hide — we control when it hides
SplashScreen.preventAutoHideAsync();

export default function App() {
  const [appReady, setAppReady] = useState(false);

  useEffect(() => {
    async function prepare() {
      try {
        // 1. Read auth token from SecureStore
        const token = await SecureStore.getItemAsync('auth_token');
        
        // 2. Initialize auth state (optimistic — details in 1.6)
        authStore.initialize(token);
        
        // 3. Prefetch active tab data (Chat history or contacts)
        await prefetchActiveTabData();
        
        // 4. Mark ready
        setAppReady(true);
      } catch (e) {
        // Even on error, show the app (will show auth screen)
        setAppReady(true);
        Sentry.captureException(e);
      }
    }
    prepare();
  }, []);

  const onLayoutRootView = useCallback(async () => {
    if (appReady) {
      // Hide splash AFTER the root view has been laid out
      // This eliminates any white flash between splash and app
      await SplashScreen.hideAsync();
    }
  }, [appReady]);

  if (!appReady) return null;

  return (
    <View style={{ flex: 1 }} onLayout={onLayoutRootView}>
      <AppProviders>
        <RootNavigator />
      </AppProviders>
    </View>
  );
}
```

**Critical rules:**
- The splash image background color (`#0A0A0F`) MUST exactly match the app's root background color. Any mismatch creates a visible flash.
- The splash screen is rendered by iOS natively from a storyboard (generated by Expo). It is NOT a React Native component. This means it is visible instantly, before any JS executes.
- `SplashScreen.hideAsync()` is called ONLY after the root view has laid out (via `onLayout`), guaranteeing the React tree has painted pixels to the screen.
- Maximum splash duration hard cap: 5,000ms. If `prepare()` has not completed by then, force-hide and show whatever state is available. This prevents the app from appearing frozen.

#### 1.6 App Initialization Sequence

Exact order of operations from the moment the user taps the app icon:

```
T+0ms     ── USER TAPS APP ICON ──────────────────────────────────────
           │
T+0-50ms  ── iOS PROCESS LAUNCH ──────────────────────────────────────
           │  iOS kernel creates process, loads Mach-O binary
           │  Dynamic linker loads dylibs (UIKit, Foundation, etc.)
           │  main() is called
           │
T+50ms    ── STEP 1: NATIVE SPLASH DISPLAYED ────────────────────────
           │  iOS displays LaunchScreen.storyboard (generated by Expo)
           │  User sees Skye logo on dark background immediately
           │  SplashScreen.preventAutoHideAsync() is registered
           │
T+50-400ms── STEP 2: HERMES ENGINE INITIALIZES ──────────────────────
           │  Expo native runtime initializes
           │  Hermes VM is created
           │  Global scope is set up (bridge, native modules registered)
           │
T+400ms   ── STEP 3: JS BUNDLE LOADS ────────────────────────────────
           │  Hermes reads pre-compiled bytecode (.hbc) from disk
           │  Top-level module execution begins
           │  All eagerly-imported modules execute their top-level code
           │  React, React Native, Navigation, TanStack Query initialize
           │  Target: complete by T+1000ms
           │
T+1000ms  ── STEP 4: AUTH TOKEN READ FROM SECURESTORE ───────────────
           │  await SecureStore.getItemAsync('auth_token')
           │  await SecureStore.getItemAsync('refresh_token')
           │  Read is fast (~5-20ms) — Keychain access on device
           │
T+1020ms  ── STEP 5: TOKEN VALIDATION (OPTIMISTIC) ──────────────────
           │  IF token exists:
           │    ├─ Decode JWT locally (check exp claim)
           │    ├─ IF not expired: assume valid, proceed authenticated
           │    ├─ IF expired but refresh token exists: proceed authenticated
           │    │   (refresh will happen async in background)
           │    └─ IF no tokens: proceed to auth screen
           │  IF no token:
           │    └─ Proceed to auth screen (login/signup)
           │  
           │  CRITICAL: Do NOT make a network call to validate the token
           │  during startup. This would add 200-800ms to launch time.
           │  Validate asynchronously after the app is interactive.
           │
T+1050ms  ── STEP 6: NAVIGATION TREE MOUNTS ─────────────────────────
           │  <NavigationContainer> mounts
           │  Initial route is determined:
           │    ├─ Authenticated → ChatScreen (home tab)
           │    └─ Unauthenticated → AuthScreen
           │  Bottom tab navigator mounts (tabs registered but
           │  only active tab screen component renders)
           │
T+1100ms  ── STEP 7: ACTIVE TAB DATA PREFETCH BEGINS ────────────────
           │  IF authenticated:
           │    ├─ queryClient.prefetchQuery('chat-history', ...)
           │    ├─ Read from TanStack Query persisted cache first
           │    └─ If cache hit: use immediately. If miss: fetch from API.
           │  This runs BEFORE splash hides. User sees splash during fetch.
           │  Timeout: 2000ms. If not complete, show skeleton instead.
           │
T+1400ms  ── STEP 8: SPLASH HIDES, APP IS INTERACTIVE ───────────────
(target)   │  Root view onLayout fires → SplashScreen.hideAsync()
           │  User sees either:
           │    ├─ Full Chat screen with data (cache hit or fast fetch)
           │    ├─ Chat screen with skeleton loading state (fetch pending)
           │    └─ Auth screen (unauthenticated)
           │  App accepts touch input. User can navigate.
           │
T+1400ms+ ── STEP 9: BACKGROUND TASKS ───────────────────────────────
           │  All of these run after splash hides, non-blocking:
           │  
           │  ├─ Token validation: POST /api/auth/validate
           │  │   IF invalid → force logout, redirect to AuthScreen
           │  │   IF valid → no-op (we were right to be optimistic)
           │  │
           │  ├─ Token refresh (if token was expired):
           │  │   POST /api/auth/refresh with refresh_token
           │  │   Store new tokens in SecureStore
           │  │
           │  ├─ Push notification registration:
           │  │   Request permission (if not already granted)
           │  │   Get Expo push token
           │  │   POST /api/users/push-token
           │  │
           │  ├─ Prefetch adjacent tab data:
           │  │   queryClient.prefetchQuery('contacts', ...)  // People tab
           │  │   queryClient.prefetchQuery('properties', ...) // Properties tab
           │  │
           │  ├─ Lazy chunk prefetch:
           │  │   import('@/screens/People/PeopleScreen')
           │  │   import('@/screens/Properties/PropertiesScreen')
           │  │
           │  ├─ Sync offline queue (if pending mutations exist):
           │  │   NetworkQueue.flush()
           │  │
           │  └─ Analytics: app_launch event with timing data
           │
           └── APP IS FULLY LOADED ───────────────────────────────────
```

**Initialization timeout safety:** If any step in the critical path (Steps 4-7) takes longer than 3,000ms total, force-proceed to Step 8. Show whatever is available. Log the timeout to Sentry as a performance warning. The user must never stare at a splash screen for more than 3 seconds.

---

### 2. Runtime Performance

#### 2.1 Frame Rate Targets

| Context | Target FPS | Measurement Method |
|---------|-----------|-------------------|
| Scrolling (FlatList, ScrollView) | 60fps (16.67ms/frame) | Flipper FPS monitor, Perf Monitor overlay |
| Scrolling on ProMotion devices (iPhone 13 Pro+) | 120fps (8.33ms/frame) | Must not lock to 60fps — use `CADisplayLink` preferred frame rate |
| Animations (Reanimated) | 60fps | All animations run on UI thread via Reanimated worklets |
| Screen transitions (React Navigation) | 60fps | Use `@react-navigation/native-stack` (native transitions, not JS-driven) |
| Keyboard open/close | 60fps | Use `react-native-keyboard-controller` for native-driven keyboard animation |
| Chat message streaming (typing indicator, token rendering) | 60fps | Batch state updates, avoid per-token re-renders (buffer 50ms) |

**ProMotion support:**

```typescript
// Enable ProMotion (120Hz) for scroll views
// React Native respects CADisplayLink by default on iOS,
// but ensure no forced 60fps caps exist in the app
// Reanimated 3 respects ProMotion natively
```

**FPS monitoring in development:**

```typescript
// Enable in dev builds only
if (__DEV__) {
  // React Native Perf Monitor
  // Accessible via Dev Menu → Show Perf Monitor
  // Shows JS thread FPS and UI thread FPS separately
}
```

**CI enforcement:** Maestro E2E tests include automated scroll tests. If any test records a frame drop below 55fps for more than 500ms continuously, the test fails.

#### 2.2 JS Thread Budget

**Hard limit: <16ms per JS frame execution.** Any operation exceeding this will cause frame drops.

**Operations that MUST run on native/UI threads (never on JS thread):**

| Operation | Solution | Thread |
|-----------|----------|--------|
| Scroll-linked animations | `Reanimated` shared values + `useAnimatedScrollHandler` | UI thread |
| Gesture handling | `react-native-gesture-handler` | UI thread |
| Screen transitions | `@react-navigation/native-stack` | Native (UIKit) |
| Image decoding | `expo-image` (native decoding, off-main-thread) | Background native thread |
| Keyboard animations | `react-native-keyboard-controller` | UI thread |
| Layout animations | `Reanimated` layout animations | UI thread |
| Heavy data transforms (500+ contacts sort/filter) | `InteractionManager.runAfterInteractions` or Web Worker via JSI | Deferred / Worker |

**Operations acceptable on JS thread (if <16ms):**

| Operation | Typical Duration | Acceptable? |
|-----------|-----------------|-------------|
| React state update (single component) | 1-3ms | Yes |
| TanStack Query cache read | <1ms | Yes |
| Navigation dispatch | 2-5ms | Yes |
| Small list filter (<100 items) | 1-5ms | Yes |
| Date formatting (single value) | <1ms | Yes |

**Operations that MUST be deferred or chunked:**

| Operation | Strategy |
|-----------|----------|
| Sorting 500+ contacts | `useMemo` with dependency on sort key. If >16ms, move to `InteractionManager.runAfterInteractions` |
| Full-text search across contacts | Debounce input by 300ms, run search in `requestIdleCallback` or after interactions complete |
| Chat history parsing (1000+ messages) | Paginate: load 50 messages initially, load more on scroll |
| JSON parsing of large API responses | Handled by Hermes natively (fast), but if >50KB response, parse in chunks |

#### 2.3 React Rendering Optimization

##### 2.3.1 `React.memo` Strategy

**Policy: Memoize components that receive object/array props from parent lists, or that render inside frequently-updating contexts. Do NOT memoize every component — only where re-render cost is measurable (>2ms).**

| Component | Memoized? | Reason |
|-----------|-----------|--------|
| `ContactListItem` | Yes | Rendered inside FlatList with 500+ items. Parent list re-renders on filter/sort. Each item receives contact object prop. |
| `PropertyListItem` | Yes | Same reasoning as ContactListItem. |
| `ChatMessage` | Yes | Rendered inside FlatList. New messages should not re-render all existing messages. |
| `ChatInput` | Yes | Parent (ChatScreen) re-renders on new messages. Input must not lose focus or re-render. |
| `TabBar` | Yes | Rendered on every screen. Must not re-render when screen content changes. |
| `Avatar` | Yes | Rendered in many lists. Receives `uri` and `name` props — cheap to compare. |
| `ScreenHeader` | No | Renders once per screen. No perf benefit from memoization. |
| `SettingsScreen` | No | Rarely visited, simple render. Memoization adds complexity for no gain. |

```typescript
// Example: ContactListItem
export const ContactListItem = React.memo(function ContactListItem({
  contact,
  onPress,
}: {
  contact: Contact;
  onPress: (id: string) => void;
}) {
  return (
    <Pressable onPress={() => onPress(contact.id)}>
      <Avatar uri={contact.avatar_url} name={contact.full_name} size={48} />
      <View>
        <Text>{contact.full_name}</Text>
        <Text>{contact.company}</Text>
      </View>
    </Pressable>
  );
}, (prev, next) => {
  // Custom comparator: only re-render if the contact data changed
  return prev.contact.id === next.contact.id
    && prev.contact.full_name === next.contact.full_name
    && prev.contact.avatar_url === next.contact.avatar_url
    && prev.contact.company === next.contact.company;
});
```

##### 2.3.2 `useMemo` / `useCallback` Policy

**Policy: Use `useMemo` for computationally expensive derivations (>2ms). Use `useCallback` for functions passed as props to memoized children. Do NOT use either for trivial computations — the overhead of memoization itself (~0.1ms) can exceed the cost of recomputation.**

| Hook | Where | What | Why |
|------|-------|------|-----|
| `useMemo` | PeopleScreen | Filtered + sorted contacts list | Filtering 500+ contacts on every render would take 5-15ms. Memoize on `[contacts, searchQuery, sortKey]`. |
| `useMemo` | ChatScreen | Grouped messages by date | Grouping 200+ messages is ~3ms. Memoize on `[messages]`. |
| `useMemo` | PropertiesScreen | Price-formatted property data | Number formatting 100+ items is ~2ms. |
| `useCallback` | PeopleScreen | `onContactPress` handler | Passed to memoized `ContactListItem`. Without `useCallback`, new function reference on every render defeats `React.memo`. |
| `useCallback` | ChatScreen | `onSendMessage` handler | Passed to memoized `ChatInput`. |
| **DO NOT** | SettingsScreen | Simple boolean toggles | Trivial computation. `useMemo` overhead exceeds savings. |
| **DO NOT** | Any | Inline styles that are static | Use `StyleSheet.create` instead, defined outside component. |

##### 2.3.3 FlatList Optimization

Every FlatList in the app MUST use these props:

```typescript
<FlatList
  // REQUIRED for all FlatLists
  keyExtractor={(item) => item.id}              // Stable, unique key
  removeClippedSubviews={true}                  // Detach off-screen views from native hierarchy
  maxToRenderPerBatch={10}                      // Render 10 items per JS frame
  windowSize={10}                               // Render 10 screens worth of content (5 above, 5 below)
  initialNumToRender={15}                       // Render 15 items initially (fills ~2 screens)
  updateCellsBatchingPeriod={50}                // 50ms between batch renders

  // REQUIRED for fixed-height rows (contacts, properties)
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,                        // e.g., 72 for contact rows
    offset: ITEM_HEIGHT * index,
    index,
  })}

  // RECOMMENDED
  onEndReachedThreshold={0.5}                   // Trigger load-more at 50% from bottom
  ListEmptyComponent={<EmptyState />}           // Always show empty state
  ListFooterComponent={isLoading ? <Spinner /> : null}
/>
```

**FlatList-specific rules:**
- Contact list (500+ items): MUST use `getItemLayout` with fixed 72pt row height.
- Chat messages (variable height): CANNOT use `getItemLayout`. Instead, use `estimatedItemSize={80}` if using FlashList, or accept the scroll-to-index limitation.
- **FlashList consideration:** If FlatList performance is insufficient for 500+ contacts, migrate to `@shopify/flash-list` which uses recycling (like UICollectionView). FlashList requires `estimatedItemSize` instead of `getItemLayout` and typically achieves 2-3x better scroll performance. **Recommendation: Start with FlatList, migrate to FlashList if profiling shows frame drops on 500+ item lists.**

##### 2.3.4 Context Splitting (Prevent Cascade Re-renders)

**Rule: Never put unrelated state in the same React Context. A context update re-renders ALL consumers of that context.**

```
WRONG:
  <AppContext.Provider value={{ user, theme, contacts, chatMessages, notifications }}>
    {children}
  </AppContext.Provider>
  // Problem: a new chat message re-renders EVERY component that reads theme or user

CORRECT — Split into focused contexts:
```

| Context | State | Update Frequency | Consumers |
|---------|-------|-----------------|-----------|
| `AuthContext` | `user`, `token`, `isAuthenticated` | Rare (login/logout only) | Navigation guard, profile displays |
| `ThemeContext` | `colorScheme`, `theme` object | Rare (user changes theme) | All UI components via styled-system |
| `NetworkContext` | `isOnline`, `connectionType` | Occasional (connectivity changes) | Offline banner, NetworkQueue |
| `NotificationContext` | `unreadCount`, `permissions` | Occasional | Tab badge, notification bell |

**Data fetching does NOT use Context.** All server data (contacts, properties, chat messages, etc.) flows through TanStack Query. TanStack Query has its own subscription model — components only re-render when THEIR specific query data changes, not when any query changes. This is superior to Context for server state.

```typescript
// CORRECT: Each screen subscribes only to its own data
function PeopleScreen() {
  const { data: contacts } = useQuery({ queryKey: ['contacts'], ... });
  // Only re-renders when contacts change
}

function ChatScreen() {
  const { data: messages } = useQuery({ queryKey: ['chat-messages'], ... });
  // Only re-renders when messages change
  // Does NOT re-render when contacts update
}
```

#### 2.4 Image Performance

**Library: `expo-image`** (preferred over FastImage for Expo-managed projects; wraps native SDImage on iOS with full caching, progressive loading, and memory management).

```typescript
import { Image } from 'expo-image';

// Contact avatar — 48x48pt display, load at 2x max (96x96px)
<Image
  source={{ uri: contact.avatar_url }}
  style={{ width: 48, height: 48, borderRadius: 24 }}
  contentFit="cover"
  placeholder={blurhash}                    // BlurHash placeholder for instant display
  transition={200}                          // 200ms fade-in when loaded
  cachePolicy="memory-disk"                 // Cache in memory AND on disk
  recyclingKey={contact.id}                 // Stable key for recycling in lists
/>

// Property photo — full width display
<Image
  source={{ uri: property.photo_url }}
  style={{ width: screenWidth, height: 240 }}
  contentFit="cover"
  placeholder={property.photo_blurhash}
  transition={300}
  cachePolicy="memory-disk"
  priority="high"                           // Load before other images on same screen
/>
```

**Image rules:**
1. **Never load full-resolution images into thumbnails.** If displaying a 48x48pt avatar, request `?width=96&height=96` from the CDN/image service (2x for Retina). If the backend serves images via Supabase Storage, use Supabase image transforms: `supabaseUrl/storage/v1/object/public/avatars/img.jpg?width=96&height=96`.
2. **BlurHash for all user-generated images.** When images are uploaded, generate a BlurHash server-side (4x3 components, ~20-30 bytes). Store in the database alongside the image URL. Display BlurHash instantly while full image loads.
3. **Memory-disk cache hierarchy.** `expo-image` manages this natively. Memory cache for recently viewed (last ~50 images). Disk cache for all downloaded images (cap at 200MB, LRU eviction).
4. **Prefetch images for visible list items.** When a FlatList is about to display items, prefetch their images: `Image.prefetch(urls)`.
5. **Cancel image loads on unmount.** `expo-image` handles this automatically when the component unmounts.

#### 2.5 Memory Management

##### 2.5.1 Memory Budgets

| Screen | Memory Budget | Rationale |
|--------|--------------|-----------|
| ChatScreen | <150MB | Chat history with message bubbles, typing indicators, streamed content. May include rendered tool results (contact cards, property cards). |
| PeopleScreen | <100MB | FlatList with 500+ contacts. Avatars in memory for visible + buffered rows (~40 items at any time). |
| PeopleDetailScreen | <120MB | Single contact with full details, activity timeline, notes, linked properties. May include multiple images. |
| PropertiesScreen | <120MB | Property cards with photos. Heavier images than People. |
| StudioScreen | <130MB | Rich media content, AI-generated marketing materials. |
| SettingsScreen | <50MB | Lightweight. Simple form fields. |
| **Total app at any time** | **<300MB** | iOS will terminate apps exceeding ~1.5GB on iPhone 13 (6GB RAM). We stay well under, leaving room for system + background apps. |

##### 2.5.2 Memory Warning Handling

```typescript
// Register for iOS memory warnings
import { AppState } from 'react-native';

useEffect(() => {
  const subscription = AppState.addEventListener('memoryWarning', () => {
    // 1. Clear image memory cache (disk cache persists)
    Image.clearMemoryCache();

    // 2. Reduce FlatList render windows
    setFlatListWindowSize(5);  // From 10 to 5

    // 3. Clear TanStack Query inactive caches
    queryClient.getQueryCache().getAll()
      .filter(q => q.state.status === 'success' && !q.getObserversCount())
      .forEach(q => queryClient.removeQueries({ queryKey: q.queryKey }));

    // 4. Log to Sentry
    Sentry.captureMessage('Memory warning received', {
      level: 'warning',
      extra: {
        screen: navigationRef.getCurrentRoute()?.name,
        // Note: direct heap size not available, but we can track query cache size
        queryCacheSize: queryClient.getQueryCache().getAll().length,
      },
    });
  });

  return () => subscription.remove();
}, []);
```

##### 2.5.3 Leak Detection

**Development-time leak detection:**

1. **Component unmount audit:** Every component that creates subscriptions, timers, or async operations MUST clean them up:

```typescript
// REQUIRED pattern for all components with side effects
useEffect(() => {
  const controller = new AbortController();
  
  fetchData({ signal: controller.signal });
  const timer = setInterval(pollData, 30000);
  const subscription = eventEmitter.addListener('event', handler);

  return () => {
    controller.abort();              // Cancel pending fetch
    clearInterval(timer);            // Clear timer
    subscription.remove();           // Remove event listener
  };
}, []);
```

2. **Flipper LeakCanary plugin** (development builds): monitor for components that remain in memory after unmount.

3. **CI memory regression test:** Maestro E2E navigates through all tabs 3 times, then checks memory. If memory exceeds 400MB after returning to ChatScreen (which would indicate leaked screens), the test fails.

4. **TanStack Query subscription audit:** In development, log a warning if a query has observers after the subscribing component unmounts:

```typescript
if (__DEV__) {
  queryClient.getQueryCache().subscribe((event) => {
    if (event.type === 'observerRemoved' && event.query.getObserversCount() === 0) {
      console.debug(`Query ${event.query.queryKey} has no observers — eligible for GC`);
    }
  });
}
```

---

### 3. Network Performance

#### 3.1 API Client Architecture

```typescript
// api/client.ts — Singleton API client

import { QueryClient } from '@tanstack/react-query';

const API_BASE = Config.API_URL; // e.g., https://api.skyecrm.com

class ApiClient {
  private pendingRequests = new Map<string, Promise<any>>();
  
  // ─── CONNECTION POOLING ──────────────────────────
  // React Native's fetch uses NSURLSession on iOS, which
  // maintains HTTP/2 connection pooling by default.
  // We enforce HTTP/2 at the server level (Next.js on Vercel/AWS).
  // NSURLSession multiplexes requests over a single TCP connection.
  // No additional client configuration needed — this is automatic.

  // ─── REQUEST DEDUPLICATION ───────────────────────
  async request<T>(
    method: string,
    path: string,
    options?: RequestOptions,
  ): Promise<T> {
    const dedupeKey = `${method}:${path}:${JSON.stringify(options?.params)}`;

    // If identical request is already in flight, return same promise
    if (method === 'GET' && this.pendingRequests.has(dedupeKey)) {
      return this.pendingRequests.get(dedupeKey) as Promise<T>;
    }

    const promise = this.executeRequest<T>(method, path, options);

    if (method === 'GET') {
      this.pendingRequests.set(dedupeKey, promise);
      promise.finally(() => this.pendingRequests.delete(dedupeKey));
    }

    return promise;
  }

  // ─── REQUEST EXECUTION ──────────────────────────
  private async executeRequest<T>(
    method: string,
    path: string,
    options?: RequestOptions,
  ): Promise<T> {
    const url = new URL(path, API_BASE);
    if (options?.params) {
      Object.entries(options.params).forEach(([k, v]) =>
        url.searchParams.set(k, String(v))
      );
    }

    const headers: Record<string, string> = {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      'Accept-Encoding': 'gzip, deflate, br',   // Compression
      'X-Client-Version': Config.APP_VERSION,
      'X-Request-Priority': options?.priority ?? 'normal',
    };

    // Auth header
    const token = await tokenManager.getAccessToken();
    if (token) {
      headers['Authorization'] = `Bearer ${token}`;
    }

    const controller = new AbortController();
    const timeoutId = setTimeout(
      () => controller.abort(),
      options?.timeout ?? 15000,   // Default 15s timeout
    );

    try {
      const response = await fetch(url.toString(), {
        method,
        headers,
        body: options?.body ? JSON.stringify(options.body) : undefined,
        signal: options?.signal ?? controller.signal,
      });

      if (response.status === 401) {
        // Token expired — attempt refresh
        const refreshed = await tokenManager.refresh();
        if (refreshed) {
          // Retry with new token
          return this.executeRequest(method, path, options);
        } else {
          // Refresh failed — logout
          authStore.logout();
          throw new AuthError('Session expired');
        }
      }

      if (!response.ok) {
        throw new ApiError(response.status, await response.text());
      }

      return response.json() as Promise<T>;
    } finally {
      clearTimeout(timeoutId);
    }
  }
}

export const apiClient = new ApiClient();
```

##### 3.1.1 Request Prioritization

| Priority | Examples | Behavior |
|----------|---------|----------|
| `critical` | Chat message send, auth token refresh | Immediate execution, no queuing, 30s timeout |
| `high` | Current screen data fetch, contact detail load | Immediate execution, 15s timeout |
| `normal` | Background prefetch, adjacent tab data | Queued if >4 concurrent requests, 15s timeout |
| `low` | Analytics events, push token registration | Queued if >2 concurrent requests, 30s timeout, retryable |

```typescript
// Priority-aware request queue
class RequestQueue {
  private concurrentLimit = 6;      // iOS NSURLSession default
  private activeCount = 0;
  private queue: PrioritizedRequest[] = [];

  enqueue(request: PrioritizedRequest) {
    if (request.priority === 'critical' || request.priority === 'high') {
      // Execute immediately, even if at concurrent limit
      // (iOS handles >6 connections with HTTP/2 multiplexing)
      this.execute(request);
    } else {
      // Queue and execute when slot available
      this.queue.push(request);
      this.queue.sort((a, b) => PRIORITY_ORDER[a.priority] - PRIORITY_ORDER[b.priority]);
      this.drain();
    }
  }
}
```

##### 3.1.2 Compression

- **Response compression:** Server MUST send `Content-Encoding: gzip` or `br` (Brotli). All Next.js API routes on Vercel/AWS automatically gzip responses. Verify by checking response headers.
- **Request compression:** For large request bodies (e.g., bulk contact import >10KB), compress with gzip before sending. For typical requests (chat messages, contact edits), compression overhead exceeds savings — send uncompressed.
- **Expected savings:** Typical JSON API response compresses 70-80% with gzip. A 50KB contact list response becomes ~12KB on the wire.

#### 3.2 Caching Strategy (TanStack Query Configuration)

```typescript
// query/queryClient.ts

import { QueryClient } from '@tanstack/react-query';
import { createAsyncStoragePersister } from '@tanstack/query-async-storage-persister';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // ─── GLOBAL DEFAULTS ────────────────────────
      staleTime: 5 * 60 * 1000,          // 5 minutes — data considered fresh
      gcTime: 30 * 60 * 1000,            // 30 minutes — keep in cache after last observer unmounts
      retry: 3,                           // Retry failed requests 3 times
      retryDelay: (attempt) =>            // Exponential backoff: 1s, 2s, 4s
        Math.min(1000 * 2 ** attempt, 8000),
      refetchOnWindowFocus: true,         // Refetch when app comes to foreground
      refetchOnReconnect: true,           // Refetch when network reconnects
      networkMode: 'offlineFirst',        // Return cached data when offline
    },
    mutations: {
      retry: 2,
      networkMode: 'offlineFirst',
    },
  },
});
```

##### 3.2.1 Per-Resource Cache Configuration

| Resource | Query Key | `staleTime` | `gcTime` | Refetch Strategy | Rationale |
|----------|-----------|-------------|----------|-----------------|-----------|
| Contacts list | `['contacts']` | 5min | 30min | On foreground, on reconnect | Contacts change occasionally; 5min staleness is acceptable. |
| Contact detail | `['contacts', id]` | 5min | 30min | On foreground | Same as list. Detail may be edited by user (optimistic update handles this). |
| Properties list | `['properties']` | 15min | 30min | On foreground | Properties change less frequently than contacts. |
| Property detail | `['properties', id]` | 15min | 30min | On foreground | Same as list. |
| Chat history | `['chat-messages']` | 1min | 30min | On foreground, polling every 30s when screen active | Chat is near-realtime. 1min staleness ensures quick updates. |
| Chat conversation | `['chat', conversationId]` | 30s | 15min | Polling every 15s when active | Active conversation needs freshest data. |
| User profile | `['user-profile']` | 30min | 60min | On foreground | Profile rarely changes. |
| Tasks list | `['tasks']` | 5min | 30min | On foreground | Tasks may be created by AI or user. |
| Notifications | `['notifications']` | 1min | 15min | Polling every 60s | Users expect near-realtime notification counts. |
| Search results | `['search', query]` | 0 (always stale) | 5min | Never auto-refetch | Search results are ephemeral — always fetch fresh on new search. |

##### 3.2.2 Query Persistence (Offline-First)

```typescript
// Persist query cache to AsyncStorage for instant startup data
const asyncStoragePersister = createAsyncStoragePersister({
  storage: AsyncStorage,
  key: 'SKYE_QUERY_CACHE',
  throttleTime: 2000,            // Throttle writes to every 2s
  serialize: JSON.stringify,
  deserialize: JSON.parse,
});

// Wrap app with PersistQueryClientProvider
import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client';

<PersistQueryClientProvider
  client={queryClient}
  persistOptions={{
    persister: asyncStoragePersister,
    maxAge: 24 * 60 * 60 * 1000,  // Persist for 24 hours
    dehydrateOptions: {
      shouldDehydrateQuery: (query) => {
        // Only persist successful queries for key resources
        const persistableKeys = ['contacts', 'properties', 'user-profile', 'tasks'];
        return query.state.status === 'success'
          && persistableKeys.some(key => 
            (query.queryKey as string[])[0] === key
          );
      },
    },
  }}
>
  <App />
</PersistQueryClientProvider>
```

**Result:** On subsequent app launches, the contacts list, properties, and user profile are available INSTANTLY from persisted cache (before any network request). The user sees real data, not a loading spinner, within milliseconds. Fresh data is fetched in the background and seamlessly replaces cached data when available.

##### 3.2.3 Optimistic Updates

**Policy: All user-initiated mutations that modify a single resource use optimistic updates. The UI updates immediately; the server request happens in the background. If the server rejects the mutation, the UI rolls back.**

| Mutation | Optimistic? | Rollback Strategy |
|----------|-------------|------------------|
| Edit contact fields | Yes | Snapshot previous contact, restore on error, show toast "Save failed, changes reverted" |
| Create new note | Yes | Add note to list immediately with temporary ID, replace with server ID on success, remove on error |
| Complete/uncomplete task | Yes | Toggle immediately, revert on error |
| Send chat message | Yes | Add message to list immediately with "sending" status, update to "sent" on success, show "failed to send" on error with retry button |
| Delete contact | **No** | Destructive action — confirm with user, wait for server confirmation, then remove from UI |
| Bulk operations | **No** | Too complex for optimistic updates — show loading state |

```typescript
// Example: Optimistic contact edit
const updateContact = useMutation({
  mutationFn: (data: ContactUpdate) =>
    apiClient.request('PATCH', `/api/contacts/${data.id}`, { body: data }),
  
  onMutate: async (newData) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['contacts', newData.id] });
    
    // Snapshot previous value
    const previousContact = queryClient.getQueryData(['contacts', newData.id]);
    
    // Optimistically update
    queryClient.setQueryData(['contacts', newData.id], (old: Contact) => ({
      ...old,
      ...newData,
    }));
    
    // Also update the contact in the list cache
    queryClient.setQueryData(['contacts'], (old: Contact[]) =>
      old?.map(c => c.id === newData.id ? { ...c, ...newData } : c)
    );
    
    return { previousContact };
  },
  
  onError: (err, newData, context) => {
    // Rollback
    queryClient.setQueryData(['contacts', newData.id], context?.previousContact);
    queryClient.invalidateQueries({ queryKey: ['contacts'] });
    showToast('Save failed. Changes reverted.', 'error');
  },
  
  onSettled: () => {
    // Always refetch to ensure server state
    queryClient.invalidateQueries({ queryKey: ['contacts'] });
  },
});
```

#### 3.3 Prefetching Strategy

| User Location | Prefetch Target | Trigger | Data Size |
|---------------|----------------|---------|-----------|
| People tab (list) | First 5 contact details | After list renders, idle time | ~5KB each = 25KB |
| People tab (list) | Contact avatars for visible rows | FlatList `onViewableItemsChanged` | Images (cached by expo-image) |
| Chat tab | Recent chat history (last 50 messages) | On app launch (Step 7 in init sequence) | ~20KB |
| Chat tab | User's task list | After chat renders | ~5KB |
| Properties tab | First 3 property details | After list renders | ~10KB each = 30KB |
| Any tab | Adjacent tab data | After current tab renders + 1s delay | Variable |
| Contact detail | Linked properties for that contact | On detail screen mount | ~10KB |

```typescript
// Prefetch hook used in PeopleScreen
function usePrefetchContactDetails(contacts: Contact[]) {
  const queryClient = useQueryClient();

  useEffect(() => {
    if (contacts.length === 0) return;

    // Prefetch first 5 contact details after a short delay
    const timer = setTimeout(() => {
      contacts.slice(0, 5).forEach(contact => {
        queryClient.prefetchQuery({
          queryKey: ['contacts', contact.id],
          queryFn: () => apiClient.request('GET', `/api/contacts/${contact.id}`),
          staleTime: 5 * 60 * 1000,
        });
      });
    }, 1000); // 1s delay — don't compete with initial render

    return () => clearTimeout(timer);
  }, [contacts]);
}
```

#### 3.4 Offline Queue (NetworkQueue)

```typescript
// network/NetworkQueue.ts

import NetInfo from '@react-native-community/netinfo';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface QueuedMutation {
  id: string;                          // UUID
  timestamp: number;                   // When queued
  method: 'POST' | 'PATCH' | 'PUT' | 'DELETE';
  path: string;
  body: any;
  retryCount: number;
  maxRetries: number;                  // Default: 5
  priority: 'critical' | 'high' | 'normal';
}

class NetworkQueue {
  private queue: QueuedMutation[] = [];
  private isFlushing = false;
  private readonly STORAGE_KEY = 'SKYE_OFFLINE_QUEUE';
  private readonly MAX_QUEUE_SIZE = 100;

  constructor() {
    // Restore queue from persistent storage on init
    this.restore();

    // Listen for connectivity changes
    NetInfo.addEventListener((state) => {
      if (state.isConnected && state.isInternetReachable) {
        this.flush();
      }
    });
  }

  // ─── ENQUEUE ─────────────────────────────────────
  async enqueue(mutation: Omit<QueuedMutation, 'id' | 'timestamp' | 'retryCount'>): Promise<void> {
    if (this.queue.length >= this.MAX_QUEUE_SIZE) {
      throw new Error('Offline queue is full. Please connect to sync pending changes.');
    }

    const entry: QueuedMutation = {
      ...mutation,
      id: generateUUID(),
      timestamp: Date.now(),
      retryCount: 0,
      maxRetries: mutation.maxRetries ?? 5,
    };

    this.queue.push(entry);
    this.queue.sort((a, b) => {
      // Sort by priority first, then by timestamp (FIFO within same priority)
      const priorityOrder = { critical: 0, high: 1, normal: 2 };
      if (priorityOrder[a.priority] !== priorityOrder[b.priority]) {
        return priorityOrder[a.priority] - priorityOrder[b.priority];
      }
      return a.timestamp - b.timestamp;
    });

    await this.persist();

    // Attempt flush immediately (will no-op if offline)
    this.flush();
  }

  // ─── FLUSH ───────────────────────────────────────
  async flush(): Promise<void> {
    if (this.isFlushing || this.queue.length === 0) return;

    const netState = await NetInfo.fetch();
    if (!netState.isConnected) return;

    this.isFlushing = true;

    try {
      // Process FIFO (queue is already sorted by priority + timestamp)
      while (this.queue.length > 0) {
        const mutation = this.queue[0];

        try {
          await apiClient.request(mutation.method, mutation.path, {
            body: mutation.body,
            timeout: 30000,   // 30s timeout for queued mutations
            priority: 'high',
          });

          // Success — remove from queue
          this.queue.shift();
          await this.persist();
        } catch (error) {
          mutation.retryCount++;

          if (mutation.retryCount >= mutation.maxRetries) {
            // Max retries exceeded — move to dead letter queue
            this.queue.shift();
            await this.persist();
            Sentry.captureException(error, {
              extra: { mutation, reason: 'max_retries_exceeded' },
            });
            showToast(`Failed to sync: ${mutation.path}. Change may be lost.`, 'error');
          } else if (error instanceof ApiError && error.status === 409) {
            // Conflict — server has newer data
            // Remove from queue, invalidate cache to fetch server state
            this.queue.shift();
            await this.persist();
            queryClient.invalidateQueries();
            showToast('A sync conflict occurred. Refreshing data.', 'warning');
          } else {
            // Network error or server error — stop flushing, retry later
            break;
          }
        }
      }
    } finally {
      this.isFlushing = false;
    }
  }

  // ─── PERSISTENCE ─────────────────────────────────
  private async persist(): Promise<void> {
    await AsyncStorage.setItem(this.STORAGE_KEY, JSON.stringify(this.queue));
  }

  private async restore(): Promise<void> {
    const stored = await AsyncStorage.getItem(this.STORAGE_KEY);
    if (stored) {
      this.queue = JSON.parse(stored);
    }
  }

  // ─── STATUS ──────────────────────────────────────
  get pendingCount(): number {
    return this.queue.length;
  }

  get hasPending(): boolean {
    return this.queue.length > 0;
  }
}

export const networkQueue = new NetworkQueue();
```

**Conflict resolution strategy:**
1. **Last-write-wins** for simple field edits (contact name, phone number). The most recent mutation overwrites.
2. **Server-wins** on 409 Conflict. Discard local mutation, refetch server state, show user a toast explaining the conflict.
3. **Merge** for note creation / task creation. These are additive operations — no conflict possible. If the server returns 409, retry with a new ID.

---

### 4. Build and CI/CD Pipeline

#### 4.1 Build System Recommendation

| Criteria | EAS Build | Fastlane |
|----------|-----------|----------|
| Setup complexity | Minimal — `eas build:configure` | Significant — Ruby, Bundler, match, Xcode CLI |
| Code signing | Managed by EAS (auto-provisions) | Manual via `fastlane match` (Git-encrypted certs) |
| Build environment | Cloud (Expo servers, M-series Mac) | Self-hosted or CI runner (must be macOS) |
| Expo compatibility | First-class (native support) | Works but requires `expo prebuild` first |
| OTA updates | Built-in via EAS Update | Requires separate setup (CodePush or custom) |
| Build time | ~15-25min (cloud queue + build) | ~10-15min (no queue, local/dedicated runner) |
| Cost | Free tier: 30 builds/month. Production: $99/month | Free (self-hosted) or CI runner costs |
| Native module support | Full (custom dev clients) | Full |

**Recommendation: EAS Build.** For an Expo-managed project, EAS Build is the clearly superior choice. It eliminates code signing complexity, integrates OTA updates natively, and removes the need for macOS CI runners. The minor queue delay (2-5min) is acceptable for a team-size project.

**Exception:** If the team already has Fastlane expertise and self-hosted Mac hardware, Fastlane is viable. But the EAS integration benefits (single tool for build + update + submit) outweigh Fastlane's speed advantage.

#### 4.2 Build Profiles

```json
// eas.json
{
  "cli": {
    "version": ">= 12.0.0",
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "simulator": false,
        "buildConfiguration": "Debug"
      },
      "env": {
        "APP_ENV": "development",
        "API_URL": "http://localhost:3000",
        "SENTRY_DSN": "",
        "ENABLE_FLIPPER": "true"
      },
      "channel": "development"
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "buildConfiguration": "Release"
      },
      "env": {
        "APP_ENV": "staging",
        "API_URL": "https://staging-api.skyecrm.com",
        "SENTRY_DSN": "https://xxx@sentry.io/yyy",
        "ENABLE_FLIPPER": "false"
      },
      "channel": "preview",
      "autoIncrement": true
    },
    "production": {
      "distribution": "store",
      "ios": {
        "buildConfiguration": "Release",
        "autoIncrement": true
      },
      "env": {
        "APP_ENV": "production",
        "API_URL": "https://api.skyecrm.com",
        "SENTRY_DSN": "https://xxx@sentry.io/yyy",
        "ENABLE_FLIPPER": "false"
      },
      "channel": "production"
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "team@skyecrm.com",
        "ascAppId": "1234567890",
        "appleTeamId": "XXXXXXXXXX"
      }
    }
  }
}
```

| Profile | JS Bundle | Signing | API Target | Distribution | Debug Tools |
|---------|-----------|---------|-----------|-------------|-------------|
| `development` | Debug (dev client, HMR, source maps) | Development | `localhost:3000` | Internal (device install via QR) | Flipper, React DevTools, Perf Monitor |
| `preview` | Release (Hermes bytecode, minified) | Ad Hoc | `staging-api.skyecrm.com` | Internal (TestFlight internal testing) | Sentry only |
| `production` | Release (Hermes bytecode, minified, optimized) | App Store | `api.skyecrm.com` | App Store (via App Store Connect) | Sentry only |

#### 4.3 CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
    tags: ['v*']

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # ═══════════════════════════════════════════════════
  # STAGE 1: Quality Gates (runs on every PR)
  # ═══════════════════════════════════════════════════
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx eslint . --max-warnings 0
      - run: npx prettier --check .

  typecheck:
    name: TypeScript Check
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx tsc --noEmit

  unit-tests:
    name: Unit & Component Tests
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx jest --coverage --ci --maxWorkers=2
      - name: Check coverage thresholds
        run: |
          npx jest --coverage --coverageReporters=json-summary --ci
          # Custom script validates: utils >= 80%, hooks >= 60%
          node scripts/check-coverage.js
      - uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

  bundle-size:
    name: Bundle Size Check
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx expo export --platform ios
      - name: Check bundle size
        run: |
          BUNDLE_SIZE=$(du -sk dist/_expo/static/js/ios/*.hbc | awk '{print $1}')
          echo "Bundle size: ${BUNDLE_SIZE}KB"
          if [ "$BUNDLE_SIZE" -gt 18432 ]; then  # 18MB in KB
            echo "::error::Bundle size ${BUNDLE_SIZE}KB exceeds 18MB hard limit"
            exit 1
          elif [ "$BUNDLE_SIZE" -gt 15360 ]; then  # 15MB in KB
            echo "::warning::Bundle size ${BUNDLE_SIZE}KB exceeds 15MB soft limit"
          fi

  # ═══════════════════════════════════════════════════
  # STAGE 2: Build Preview (runs on merge to main)
  # ═══════════════════════════════════════════════════
  build-preview:
    name: Build Preview & Deploy to TestFlight
    needs: [lint, typecheck, unit-tests, bundle-size]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    timeout-minutes: 45
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      - name: Build iOS Preview
        run: eas build --platform ios --profile preview --non-interactive
      - name: Upload Sentry source maps
        run: |
          npx sentry-cli releases new ${{ github.sha }}
          npx sentry-cli releases files ${{ github.sha }} upload-sourcemaps dist/
          npx sentry-cli releases finalize ${{ github.sha }}
        env:
          SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
          SENTRY_ORG: skye-crm
          SENTRY_PROJECT: skye-ios

  # ═══════════════════════════════════════════════════
  # STAGE 3: E2E Tests (runs after preview build)
  # ═══════════════════════════════════════════════════
  e2e-tests:
    name: E2E Tests (Maestro)
    needs: [build-preview]
    runs-on: macos-latest   # Requires macOS for iOS Simulator
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
      - name: Install Maestro
        run: |
          curl -Ls "https://get.maestro.mobile.dev" | bash
          export PATH="$PATH:$HOME/.maestro/bin"
      - name: Download preview build
        run: eas build:download --platform ios --profile preview --non-interactive
      - name: Boot iOS Simulator
        run: |
          xcrun simctl boot "iPhone 15"
      - name: Install app on Simulator
        run: xcrun simctl install booted ./build.app
      - name: Run Maestro tests
        run: |
          maestro test .maestro/ --format junit --output maestro-results.xml
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: maestro-results
          path: maestro-results.xml

  # ═══════════════════════════════════════════════════
  # STAGE 4: Production Release (runs on version tag)
  # ═══════════════════════════════════════════════════
  build-production:
    name: Build Production & Submit to App Store
    needs: [lint, typecheck, unit-tests, bundle-size]
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    timeout-minutes: 60
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      - name: Build iOS Production
        run: eas build --platform ios --profile production --non-interactive --auto-submit
      - name: Upload Sentry source maps (production)
        run: |
          VERSION=$(node -p "require('./package.json').version")
          npx sentry-cli releases new "com.skyecrm.app@${VERSION}"
          npx sentry-cli releases files "com.skyecrm.app@${VERSION}" upload-sourcemaps dist/
          npx sentry-cli releases finalize "com.skyecrm.app@${VERSION}"
        env:
          SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
          SENTRY_ORG: skye-crm
          SENTRY_PROJECT: skye-ios
```

**Pipeline summary:**

| Trigger | Steps | Duration Target |
|---------|-------|----------------|
| PR opened/updated | Lint + TypeCheck + Unit Tests + Bundle Size | <5 minutes |
| Merge to `main` | Above + EAS Build (preview) + Sentry source maps | <30 minutes |
| Post-preview build | E2E tests (Maestro on simulator) | <15 minutes |
| Tag `v*.*.*` | Quality gates + EAS Build (production) + App Store submit + Sentry | <45 minutes |

#### 4.4 Code Signing

**Recommended: EAS Managed Signing.**

EAS Build manages all iOS code signing automatically:
- Generates and stores distribution certificates in EAS servers (encrypted at rest).
- Creates and manages provisioning profiles.
- Handles certificate rotation and expiry.
- No manual Xcode, Apple Developer Portal, or Keychain management needed.

```bash
# First-time setup (one-time)
eas credentials
# Follow prompts to:
# 1. Log in to Apple Developer account
# 2. Let EAS generate/manage certificates
# 3. Let EAS create provisioning profiles
```

**If manual control is required** (e.g., enterprise policy mandates self-managed certs):
- Use `fastlane match` with a private Git repo for encrypted certificate storage.
- Configure in `eas.json` with `credentialsSource: "local"`.
- Store the match password in EAS Secrets, not in the repo.

#### 4.5 Environment Variables

**Rule: NEVER commit secrets to the repository. NEVER commit `.env` files.**

| Variable | Source | Used In |
|----------|--------|---------|
| `API_URL` | EAS Build profile `env` block | API client base URL |
| `APP_ENV` | EAS Build profile `env` block | Feature flags, logging level |
| `SENTRY_DSN` | EAS Secrets | Sentry initialization |
| `EXPO_TOKEN` | GitHub Secrets | CI/CD EAS authentication |
| `SENTRY_AUTH_TOKEN` | GitHub Secrets | Source map upload |
| `SUPABASE_URL` | EAS Secrets | Supabase client (if direct connection needed) |
| `SUPABASE_ANON_KEY` | EAS Secrets | Supabase client (if direct connection needed) |

**Access in app code:**

```typescript
// Using expo-constants (reads from app.json/eas.json env)
import Constants from 'expo-constants';

const API_URL = Constants.expoConfig?.extra?.API_URL;
const APP_ENV = Constants.expoConfig?.extra?.APP_ENV;

// app.config.ts
export default {
  expo: {
    extra: {
      API_URL: process.env.API_URL,
      APP_ENV: process.env.APP_ENV,
    },
  },
};
```

**`.gitignore` must include:**
```
.env
.env.*
*.p12
*.mobileprovision
*.keystore
secrets/
```

#### 4.6 OTA Updates (EAS Update)

**Purpose:** Deploy JavaScript-only changes (bug fixes, UI tweaks, content updates) instantly without App Store review. Native code changes (new native modules, Expo SDK upgrades) still require a full build.

```typescript
// App.tsx — Check for updates on launch
import * as Updates from 'expo-updates';

useEffect(() => {
  async function checkForUpdate() {
    if (__DEV__) return; // Skip in development

    try {
      const update = await Updates.checkForUpdateAsync();
      if (update.isAvailable) {
        await Updates.fetchUpdateAsync();
        // Apply on NEXT launch — do not restart mid-session
        // The update is cached and will load on next cold start
        
        // Exception: if update is marked critical (via EAS Update metadata),
        // show a modal asking the user to restart
        if (update.manifest?.extra?.critical) {
          showRestartModal();
        }
      }
    } catch (error) {
      // Silently fail — user keeps using current version
      Sentry.captureException(error, { level: 'warning' });
    }
  }
  
  checkForUpdate();
}, []);
```

**Update policy:**

| Aspect | Policy |
|--------|--------|
| Check frequency | On every cold launch |
| Apply timing | On next cold launch (not during active session) |
| Critical updates | Show restart prompt modal |
| Rollback | EAS Update supports automatic rollback if crash rate increases after update |
| Channel mapping | `development` channel -> dev builds, `preview` channel -> TestFlight builds, `production` channel -> App Store builds |
| Update size limit | <5MB differential (EAS Update sends only changed assets) |

```bash
# Deploy OTA update to preview channel (TestFlight users)
eas update --branch preview --message "Fix contact search performance"

# Deploy OTA update to production channel (App Store users)
eas update --branch production --message "Fix crash on property detail screen"
```

#### 4.7 Build Versioning

**Scheme: Semantic Versioning (SemVer) `MAJOR.MINOR.PATCH`**

| Component | When Incremented | Example |
|-----------|-----------------|---------|
| MAJOR | Breaking changes, major redesign, incompatible API changes | 1.0.0 -> 2.0.0 |
| MINOR | New features, non-breaking enhancements | 1.0.0 -> 1.1.0 |
| PATCH | Bug fixes, performance improvements, minor tweaks | 1.0.0 -> 1.0.1 |

**Build number:** Auto-incremented by EAS Build on every build. Independent of version number. Monotonically increasing integer.

```json
// eas.json — auto-increment build number
{
  "build": {
    "preview": {
      "autoIncrement": true    // Increments ios.buildNumber automatically
    },
    "production": {
      "ios": {
        "autoIncrement": true
      }
    }
  }
}
```

**Version management:**

```json
// app.json
{
  "expo": {
    "version": "1.0.0",        // User-facing version (manual)
    "ios": {
      "buildNumber": "1"        // Auto-incremented by EAS
    }
  }
}
```

**Release tagging:**
```bash
# When ready for App Store release:
npm version minor   # Bumps 1.0.0 -> 1.1.0 in package.json
# CI script syncs version to app.json
git tag v1.1.0
git push origin v1.1.0   # Triggers production build + submit pipeline
```

---

### 5. Testing Infrastructure

#### 5.1 Unit Tests (Jest)

**Framework: Jest with `jest-expo` preset (configures Hermes transforms, React Native mocks).**

```javascript
// jest.config.js
module.exports = {
  preset: 'jest-expo',
  setupFilesAfterSetup: ['./jest.setup.ts'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@sentry/react-native|@tanstack/.*)',
  ],
  coverageThresholds: {
    global: {
      branches: 60,
      functions: 60,
      lines: 60,
      statements: 60,
    },
    './src/utils/': {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    './src/hooks/': {
      branches: 60,
      functions: 60,
      lines: 60,
      statements: 60,
    },
    './src/api/': {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{ts,tsx}',
    '!src/**/index.{ts,tsx}',     // Re-export files
  ],
};
```

**Unit test targets:**

| Module | Coverage Target | Key Test Cases |
|--------|----------------|---------------|
| `api/client.ts` | 80% | Request deduplication, auth header injection, 401 retry, timeout handling, error parsing |
| `api/tokenManager.ts` | 90% | Token read/write SecureStore, JWT expiry check, refresh flow, concurrent refresh prevention |
| `utils/formatters.ts` | 95% | Phone number formatting, currency, dates, relative time, truncation |
| `utils/validators.ts` | 95% | Email, phone, required fields, URL, numeric ranges |
| `hooks/useContacts.ts` | 60% | Query key correctness, filter/sort logic, infinite scroll pagination |
| `hooks/useChat.ts` | 60% | Message sending, streaming response handling, error states |
| `network/NetworkQueue.ts` | 80% | Enqueue, flush order, retry logic, conflict handling, persistence, max queue size |
| `store/authStore.ts` | 80% | Login, logout, token refresh, optimistic auth state |

**Example unit test:**

```typescript
// __tests__/api/client.test.ts
describe('ApiClient', () => {
  describe('request deduplication', () => {
    it('should return same promise for identical concurrent GET requests', async () => {
      const mockFetch = jest.fn().mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve({ data: 'test' }),
      });
      global.fetch = mockFetch;

      const promise1 = apiClient.request('GET', '/api/contacts');
      const promise2 = apiClient.request('GET', '/api/contacts');

      expect(promise1).toBe(promise2);          // Same promise reference
      expect(mockFetch).toHaveBeenCalledTimes(1); // Only one fetch call

      const [result1, result2] = await Promise.all([promise1, promise2]);
      expect(result1).toEqual(result2);
    });

    it('should NOT deduplicate POST requests', async () => {
      const mockFetch = jest.fn().mockResolvedValue({
        ok: true,
        json: () => Promise.resolve({ id: '1' }),
      });
      global.fetch = mockFetch;

      const promise1 = apiClient.request('POST', '/api/contacts', { body: { name: 'A' } });
      const promise2 = apiClient.request('POST', '/api/contacts', { body: { name: 'B' } });

      expect(promise1).not.toBe(promise2);
      expect(mockFetch).toHaveBeenCalledTimes(2);
    });
  });

  describe('auth token refresh', () => {
    it('should retry request with new token after 401', async () => {
      const mockFetch = jest.fn()
        .mockResolvedValueOnce({ ok: false, status: 401, text: () => 'Unauthorized' })
        .mockResolvedValueOnce({ ok: true, json: () => Promise.resolve({ data: 'success' }) });
      global.fetch = mockFetch;

      jest.spyOn(tokenManager, 'refresh').mockResolvedValueOnce(true);
      jest.spyOn(tokenManager, 'getAccessToken')
        .mockResolvedValueOnce('expired-token')
        .mockResolvedValueOnce('fresh-token');

      const result = await apiClient.request('GET', '/api/contacts');

      expect(tokenManager.refresh).toHaveBeenCalledTimes(1);
      expect(mockFetch).toHaveBeenCalledTimes(2);
      expect(result).toEqual({ data: 'success' });
    });
  });
});
```

#### 5.2 Component Tests (React Native Testing Library)

**Framework: `@testing-library/react-native` — renders components in a test environment, queries by accessibility properties, simulates user interactions.**

**Policy: Every screen component MUST be tested in all 4 states.**

| State | What to Verify |
|-------|---------------|
| **Loading** | Skeleton/spinner is displayed. No data content visible. Accessible loading indicator present. |
| **Success** | Correct data rendered. All interactive elements present. Accessibility labels on all touchable elements. |
| **Empty** | Empty state illustration and message displayed. CTA button present (e.g., "Add your first contact"). |
| **Error** | Error message displayed. Retry button present and functional. Error is not a raw technical message. |

```typescript
// __tests__/screens/PeopleScreen.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { PeopleScreen } from '@/screens/People/PeopleScreen';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      <NavigationContainer>
        {children}
      </NavigationContainer>
    </QueryClientProvider>
  );
};

describe('PeopleScreen', () => {
  it('shows loading skeleton on initial render', () => {
    server.use(
      rest.get('*/api/contacts', (req, res, ctx) => {
        return res(ctx.delay('infinite'));   // Never resolves
      })
    );

    render(<PeopleScreen />, { wrapper: createWrapper() });

    expect(screen.getByAccessibilityHint('Loading contacts')).toBeTruthy();
    expect(screen.queryByText('John Doe')).toBeNull();
  });

  it('renders contact list on success', async () => {
    server.use(
      rest.get('*/api/contacts', (req, res, ctx) => {
        return res(ctx.json({
          contacts: [
            { id: '1', full_name: 'John Doe', company: 'Acme Realty' },
            { id: '2', full_name: 'Jane Smith', company: 'Prime Properties' },
          ],
        }));
      })
    );

    render(<PeopleScreen />, { wrapper: createWrapper() });

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeTruthy();
      expect(screen.getByText('Jane Smith')).toBeTruthy();
    });

    // Verify accessibility
    expect(screen.getByRole('button', { name: 'View John Doe' })).toBeTruthy();
  });

  it('shows empty state when no contacts exist', async () => {
    server.use(
      rest.get('*/api/contacts', (req, res, ctx) => {
        return res(ctx.json({ contacts: [] }));
      })
    );

    render(<PeopleScreen />, { wrapper: createWrapper() });

    await waitFor(() => {
      expect(screen.getByText('No contacts yet')).toBeTruthy();
      expect(screen.getByRole('button', { name: 'Add your first contact' })).toBeTruthy();
    });
  });

  it('shows error state with retry button on failure', async () => {
    server.use(
      rest.get('*/api/contacts', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );

    render(<PeopleScreen />, { wrapper: createWrapper() });

    await waitFor(() => {
      expect(screen.getByText(/something went wrong/i)).toBeTruthy();
      expect(screen.getByRole('button', { name: 'Try again' })).toBeTruthy();
    });

    // Verify retry works
    server.use(
      rest.get('*/api/contacts', (req, res, ctx) => {
        return res(ctx.json({ contacts: [{ id: '1', full_name: 'John Doe' }] }));
      })
    );

    fireEvent.press(screen.getByRole('button', { name: 'Try again' }));

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeTruthy();
    });
  });

  it('filters contacts by search query', async () => {
    server.use(
      rest.get('*/api/contacts', (req, res, ctx) => {
        return res(ctx.json({
          contacts: [
            { id: '1', full_name: 'John Doe' },
            { id: '2', full_name: 'Jane Smith' },
          ],
        }));
      })
    );

    render(<PeopleScreen />, { wrapper: createWrapper() });

    await waitFor(() => screen.getByText('John Doe'));

    fireEvent.changeText(screen.getByPlaceholderText('Search contacts...'), 'Jane');

    expect(screen.queryByText('John Doe')).toBeNull();
    expect(screen.getByText('Jane Smith')).toBeTruthy();
  });
});
```

**Accessibility testing (embedded in component tests):**

Every interactive element MUST have:
- `accessibilityLabel` — describes the element (e.g., "View John Doe")
- `accessibilityRole` — button, link, header, image, etc.
- `accessibilityHint` — describes what happens on interaction (for non-obvious actions)

```typescript
// Verify in tests:
expect(screen.getByRole('button', { name: 'Send message' })).toBeTruthy();
expect(screen.getByRole('header', { name: 'People' })).toBeTruthy();
expect(screen.getByLabelText('Contact avatar for John Doe')).toBeTruthy();
```

#### 5.3 Integration Tests

**Framework: Jest + MSW (Mock Service Worker) for API mocking, or real staging server calls.**

| Integration Test Suite | What It Verifies |
|----------------------|-----------------|
| **Auth Flow** | OAuth redirect -> callback -> token storage in SecureStore -> authenticated API call succeeds -> token expires -> refresh token used -> new access token stored -> API call retried and succeeds |
| **Chat Flow** | Send message via API -> receive streaming response (SSE) -> tokens rendered incrementally -> tool call result (contact card) rendered -> message persisted in history |
| **Contact CRUD** | Fetch contacts list -> create new contact -> verify appears in list -> edit contact -> verify changes persist -> delete contact -> verify removed from list |
| **Offline Queue** | Go offline -> create contact -> edit contact -> go online -> verify both mutations synced in order -> verify server state matches |
| **Deep Linking** | App receives `skyecrm://contacts/123` -> navigates to contact detail -> correct data loaded |

```typescript
// __tests__/integration/auth-flow.test.ts
describe('Authentication Flow', () => {
  it('completes full auth lifecycle', async () => {
    // 1. Start unauthenticated
    expect(await SecureStore.getItemAsync('auth_token')).toBeNull();
    
    // 2. Simulate OAuth callback with auth code
    const tokens = await authService.handleOAuthCallback('auth-code-123');
    
    // 3. Verify tokens stored
    expect(await SecureStore.getItemAsync('auth_token')).toBe(tokens.access_token);
    expect(await SecureStore.getItemAsync('refresh_token')).toBe(tokens.refresh_token);
    
    // 4. Verify authenticated API call works
    const contacts = await apiClient.request('GET', '/api/contacts');
    expect(contacts).toBeDefined();
    
    // 5. Simulate token expiry
    await SecureStore.setItemAsync('auth_token', 'expired-token');
    
    // 6. Make API call — should trigger refresh
    const refreshedContacts = await apiClient.request('GET', '/api/contacts');
    expect(refreshedContacts).toBeDefined();
    
    // 7. Verify new token stored
    const newToken = await SecureStore.getItemAsync('auth_token');
    expect(newToken).not.toBe('expired-token');
  });
});
```

#### 5.4 E2E Tests (Maestro)

**Framework: Maestro** (recommended over Detox for React Native in 2026 — YAML-based, no flaky waits, built-in mobile-specific commands, supports iOS simulator and real device).

**Why Maestro over Detox:**
| Criteria | Maestro | Detox |
|----------|---------|-------|
| Setup complexity | `brew install maestro` + YAML files | Complex native build config, detox.config.js, custom matchers |
| Test authoring | YAML (readable by QA, PM, anyone) | JavaScript (requires dev to write/maintain) |
| Flakiness | Low (built-in smart waits, retry logic) | Medium (requires manual `waitFor`, synchronization issues with animations) |
| React Navigation support | Works out of the box | Requires custom synchronization for animations |
| CI integration | Simple — single binary, no Xcode workspace manipulation | Requires pre-built app, complex build/test separation |
| Maintenance burden | Low | High |

**Maestro test suites:**

```yaml
# .maestro/login-and-chat.yaml
appId: com.skyecrm.app
---
# Critical Path: Login -> Chat -> Send Message -> Receive Response
- launchApp
- assertVisible: "Sign in with Google"
- tapOn: "Sign in with Google"
# OAuth flow completes (configured with test account in staging)
- waitForAnimationToEnd
- assertVisible: "Chat"                    # Chat tab is active
- tapOn: "Message Skye..."                 # Chat input
- inputText: "Show me my top 5 contacts"
- tapOn: "Send"                            # Send button
- waitForAnimationToEnd
- assertVisible:
    text: "Here are your top"              # AI response starts
    timeout: 15000                         # 15s for AI response
- assertVisible: "Contact"                 # Tool result card rendered
- scroll:
    direction: DOWN
    distance: 200
```

```yaml
# .maestro/crm-flow.yaml
appId: com.skyecrm.app
---
# CRM Flow: Search -> View -> Edit -> Save -> Verify
- launchApp
- tapOn: "People"                          # Navigate to People tab
- waitForAnimationToEnd
- tapOn: "Search contacts..."
- inputText: "John"
- assertVisible: "John Doe"
- tapOn: "John Doe"                        # Open contact detail
- waitForAnimationToEnd
- assertVisible: "John Doe"               # Detail screen
- assertVisible: "Phone"
- assertVisible: "Email"
- tapOn: "Edit"
- clearText: "Company"
- inputText: "New Company LLC"
- tapOn: "Save"
- waitForAnimationToEnd
- assertVisible: "New Company LLC"         # Verify edit persisted
- back                                     # Return to list
- assertVisible: "New Company LLC"         # Verify in list too
```

```yaml
# .maestro/push-notification.yaml
appId: com.skyecrm.app
---
# Push Notification: Receive -> Tap -> Deep Link
- launchApp
- waitForAnimationToEnd
# Simulate push notification via maestro
- sendNotification:
    title: "New message from Jane"
    body: "I'm interested in the property on Oak Street"
    data:
      type: "chat_message"
      conversationId: "conv-123"
- tapOnNotification: "New message from Jane"
- waitForAnimationToEnd
- assertVisible: "Jane"                    # Chat screen with Jane's conversation
- assertVisible: "Oak Street"             # Message content visible
```

#### 5.5 Performance Tests

**Automated performance regression gates (run in CI after E2E build):**

| Metric | Target | Failure Threshold | Measurement |
|--------|--------|------------------|-------------|
| Cold launch time | <2,000ms | >2,500ms fails CI | Maestro `measurePerformance` + Xcode Instruments trace |
| FPS during contact list scroll (500 items) | 60fps | <55fps average fails CI | Maestro scroll + FPS recording |
| FPS during chat scroll (200 messages) | 60fps | <55fps average fails CI | Maestro scroll + FPS recording |
| Memory on ChatScreen (after 5min interaction) | <150MB | >180MB fails CI | Xcode Memory Gauge via `xcrun simctl` |
| Memory on PeopleScreen (500 contacts loaded) | <100MB | >120MB fails CI | Xcode Memory Gauge |
| Time to interactive after tab switch | <300ms | >500ms fails CI | Maestro timing |

```yaml
# .maestro/perf-scroll-contacts.yaml
appId: com.skyecrm.app
---
- launchApp
- tapOn: "People"
- waitForAnimationToEnd
- repeat:
    times: 10
    commands:
      - scroll:
          direction: DOWN
          distance: 500
          duration: 300
- assertMinFps: 55      # Fail if average drops below 55fps
```

```bash
# scripts/perf-startup-test.sh — Run on CI
#!/bin/bash
# Measure cold launch time on iOS Simulator
RESULTS=()
for i in {1..5}; do
  xcrun simctl terminate booted com.skyecrm.app 2>/dev/null
  sleep 2
  START=$(date +%s%N)
  xcrun simctl launch booted com.skyecrm.app
  # Wait for "app ready" log marker
  xcrun simctl spawn booted log stream --predicate 'subsystem == "com.skyecrm.app" AND messageType == info' | \
    head -1 | while read line; do
      END=$(date +%s%N)
      DURATION=$(( ($END - $START) / 1000000 ))
      RESULTS+=($DURATION)
      echo "Launch $i: ${DURATION}ms"
    done
done

# Calculate P95
# Sort and take 95th percentile
P95=$(echo "${RESULTS[@]}" | tr ' ' '\n' | sort -n | tail -1)
echo "P95 cold launch: ${P95}ms"
if [ "$P95" -gt 2500 ]; then
  echo "FAIL: P95 cold launch ${P95}ms exceeds 2500ms threshold"
  exit 1
fi
```

#### 5.6 Test Environments

| Test Type | API Server | Data | Authentication |
|-----------|-----------|------|---------------|
| Unit tests | None (pure logic) | Mocked inline | Mocked |
| Component tests | MSW (in-process mock) | Fixture data per test | Mocked context |
| Integration tests | MSW or staging server | Seed data + test accounts | Test OAuth tokens |
| E2E tests (Maestro) | Staging server | Seeded staging DB (reset before each run) | Real test account on staging |
| Performance tests | Staging server | Pre-populated with 500+ contacts, 200+ messages | Real test account |

**Staging server requirements:**
- Mirrors production API exactly (same 129 routes).
- Seeded with realistic data: 500 contacts, 50 properties, 1000 chat messages.
- Test account: `test@skyecrm.com` with OAuth bypass for CI (API key auth fallback).
- Database reset script: `npm run db:seed:staging` — runs before E2E suite, takes <60s.

---

### 6. Monitoring and Observability

#### 6.1 Sentry Configuration

```typescript
// sentry.config.ts
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: Config.SENTRY_DSN,
  
  // ─── RELEASE TRACKING ──────────────────────────
  release: `com.skyecrm.app@${Config.APP_VERSION}`,
  dist: Config.BUILD_NUMBER,
  
  // ─── ENVIRONMENT ───────────────────────────────
  environment: Config.APP_ENV,     // 'development', 'staging', 'production'
  
  // ─── PERFORMANCE MONITORING ────────────────────
  tracesSampleRate: Config.APP_ENV === 'production' ? 0.2 : 1.0,
  // 20% of transactions in production (cost control)
  // 100% in staging (full visibility during testing)
  
  profilesSampleRate: Config.APP_ENV === 'production' ? 0.1 : 0.5,
  // Profile 10% of transactions in production
  
  // ─── BREADCRUMBS ───────────────────────────────
  enableAutoSessionTracking: true,
  sessionTrackingIntervalMillis: 30000,
  
  // Automatic breadcrumbs for:
  // - Navigation events (screen transitions)
  // - Network requests (URL, method, status code — NOT request/response bodies)
  // - Console logs (warn and error level only)
  // - User interactions (button taps with accessibility label)
  
  enableNativeCrashHandling: true,
  enableAutoPerformanceTracing: true,
  
  // ─── DATA SCRUBBING ────────────────────────────
  beforeSend(event) {
    // Remove any PII from breadcrumbs
    event.breadcrumbs = event.breadcrumbs?.map(b => {
      if (b.category === 'http' && b.data?.url) {
        // Redact query params that may contain PII
        const url = new URL(b.data.url);
        url.searchParams.forEach((_, key) => {
          if (['email', 'phone', 'name', 'token'].includes(key.toLowerCase())) {
            url.searchParams.set(key, '[REDACTED]');
          }
        });
        b.data.url = url.toString();
      }
      return b;
    });
    return event;
  },
  
  // ─── FILTERING ─────────────────────────────────
  beforeBreadcrumb(breadcrumb) {
    // Don't track console.log/debug in production
    if (breadcrumb.category === 'console' && 
        ['log', 'debug', 'info'].includes(breadcrumb.level ?? '')) {
      return null;
    }
    return breadcrumb;
  },
  
  // ─── INTEGRATIONS ──────────────────────────────
  integrations: [
    Sentry.reactNativeTracingIntegration({
      routingInstrumentation: Sentry.reactNavigationIntegration,
      enableStallTracking: true,       // Detect JS thread stalls >500ms
      enableNativeFramesTracking: true, // Track native frame rate
    }),
  ],
});

// Wrap root navigation with Sentry
const navigationRef = useNavigationContainerRef();
Sentry.reactNavigationIntegration.registerNavigationContainer(navigationRef);
```

**Source map upload (in CI):**

```bash
# Uploaded automatically in CI pipeline (see Section 4.3)
# Ensures stack traces in Sentry show original TypeScript source, not minified bytecode
npx sentry-cli releases files "com.skyecrm.app@${VERSION}" upload-sourcemaps \
  --rewrite \
  --strip-prefix /build/ \
  dist/
```

**Sentry alert rules:**

| Alert | Condition | Action |
|-------|-----------|--------|
| New crash type | First occurrence of a new unhandled exception | Slack notification to #skye-crashes + PagerDuty if P0 |
| Crash-free rate drop | Crash-free sessions drops below 99% in any 1-hour window | Slack alert + PagerDuty page |
| Performance regression | P95 screen load time increases by >50% compared to 7-day average | Slack notification to #skye-performance |
| High error rate | >100 errors of same type in 1 hour | Slack notification + auto-create GitHub issue |
| Memory warning spike | >10 memory warnings in 1 hour | Slack notification to #skye-performance |

#### 6.2 Custom Analytics Events

**If an analytics SDK is chosen (Mixpanel, Amplitude, PostHog, or Segment), track these events:**

| Event | Properties | Purpose |
|-------|-----------|---------|
| `app_launched` | `launch_type: cold/warm`, `duration_ms`, `cached_data: boolean` | Measure startup performance |
| `screen_viewed` | `screen_name`, `previous_screen`, `load_time_ms` | Navigation patterns |
| `chat_message_sent` | `message_length`, `has_attachment: boolean` | Chat engagement |
| `chat_response_received` | `response_time_ms`, `token_count`, `had_tool_call: boolean` | AI performance |
| `contact_viewed` | `contact_id` (hashed), `source: search/list/chat` | CRM engagement |
| `contact_action_taken` | `action: call/text/email/note`, `contact_id` (hashed) | Agent productivity |
| `property_viewed` | `property_id` (hashed), `source: search/list/chat` | Property engagement |
| `search_performed` | `query_length`, `result_count`, `time_to_first_result_ms` | Search quality |
| `error_displayed` | `error_type`, `screen`, `retry_tapped: boolean` | Error UX quality |
| `offline_queue_synced` | `mutation_count`, `conflict_count`, `total_sync_time_ms` | Offline reliability |
| `push_notification_received` | `notification_type`, `app_state: foreground/background` | Push delivery |
| `push_notification_tapped` | `notification_type`, `time_to_tap_ms` | Push engagement |

**Funnel tracking:**

```
Funnel: Agent Productivity
  1. app_launched
  2. screen_viewed (screen: People)
  3. contact_viewed
  4. contact_action_taken (action: call | text | email)

Funnel: AI Assistant Usage
  1. app_launched
  2. screen_viewed (screen: Chat)
  3. chat_message_sent
  4. chat_response_received
  5. contact_action_taken (from chat tool result)
```

**Analytics rules:**
- All PII is hashed before sending (contact IDs, property IDs). Never send names, emails, or phone numbers to analytics.
- Analytics events are batched and sent every 30s or when batch reaches 20 events (whichever comes first).
- Analytics is `priority: low` in the request queue — never competes with user-facing requests.
- Users can opt out of analytics in Settings (CCPA/GDPR compliance).

#### 6.3 Health Metrics Dashboard

**Dashboard (DataDog, Grafana, or Sentry Dashboard):**

| Metric | Data Source | Target | Warning | Critical |
|--------|-----------|--------|---------|----------|
| **Crash-free rate** | Sentry | >99.5% | <99.5% | <99.0% |
| **App launch time (P50)** | Sentry Performance / Custom analytics | <1,500ms | >1,800ms | >2,500ms |
| **App launch time (P95)** | Sentry Performance / Custom analytics | <2,000ms | >2,200ms | >3,000ms |
| **API response time (P50)** | Sentry Performance | <300ms | >500ms | >1,000ms |
| **API response time (P95)** | Sentry Performance | <800ms | >1,200ms | >2,000ms |
| **Chat response time (P50)** | Custom analytics | <2,000ms | >3,000ms | >5,000ms |
| **Push notification delivery** | Push service dashboard | >95% | <95% | <90% |
| **JS bundle size** | CI artifact | <15MB | >15MB | >18MB |
| **Active crash issues** | Sentry | <5 open | >5 open | >10 open |
| **Offline queue failure rate** | Custom analytics | <2% | >2% | >5% |
| **Memory warning rate** | Sentry | <0.1% of sessions | >0.1% | >0.5% |
| **FlatList frame drops** | Sentry native frames | <5% of frames dropped | >5% | >10% |

**Dashboard refresh interval:** Real-time for crash-free rate and error spikes. 5-minute aggregation for performance metrics. Daily roll-up for trends.

---

### 7. Dependency Management

#### 7.1 Version Locking

**Policy: Exact versions only. No ranges (`^`, `~`). Lock file MUST be committed.**

```json
// package.json — ALL dependencies use exact versions
{
  "dependencies": {
    "react": "18.3.1",
    "react-native": "0.76.3",
    "expo": "52.0.11",
    "@tanstack/react-query": "5.62.7",
    "@react-navigation/native": "7.0.14",
    "react-native-reanimated": "3.16.5",
    "expo-image": "2.0.4",
    "expo-secure-store": "14.0.1",
    "@sentry/react-native": "6.5.0"
  }
}
```

**Rules:**
- `package-lock.json` (or `yarn.lock` / `pnpm-lock.yaml`) is committed to the repository. Always.
- CI runs `npm ci` (not `npm install`) — installs from lock file exactly, fails if lock file is out of date.
- Dependabot or Renovate is configured to create PRs for dependency updates weekly. Each update is tested through the full CI pipeline before merge.
- Major version upgrades require a dedicated PR with manual testing.

#### 7.2 Dependency Audit

**Automated vulnerability scanning:**

```yaml
# .github/workflows/security.yml
name: Security Audit
on:
  schedule:
    - cron: '0 6 * * 1'   # Every Monday at 6 AM UTC
  push:
    paths:
      - 'package.json'
      - 'package-lock.json'

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - name: npm audit
        run: npm audit --audit-level=high
        # Fails on HIGH or CRITICAL vulnerabilities
      - name: Snyk scan (optional, more comprehensive)
        uses: snyk/actions/node@master
        with:
          args: --severity-threshold=high
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

**Policy:**
- CRITICAL vulnerabilities: fix within 24 hours (patch dependency or replace).
- HIGH vulnerabilities: fix within 1 week.
- MEDIUM vulnerabilities: fix within 1 month or document accepted risk.
- LOW vulnerabilities: address during regular maintenance.

#### 7.3 Native Module Compatibility

**Every native module in the project must pass this checklist before inclusion:**

| Check | Requirement | How to Verify |
|-------|------------|---------------|
| iOS version support | iOS 16.0+ | Check module's `Podspec` for `ios.deployment_target` |
| Architecture | arm64 (Apple Silicon + all iPhones) | Check module's `Podspec` for `EXCLUDED_ARCHS` — must NOT exclude arm64 |
| No private APIs | Does not use Apple private APIs | Check for App Store rejection history in module's GitHub issues |
| Expo compatibility | Works with Expo managed or has config plugin | Check Expo docs or module README for Expo support |
| Hermes support | Functions correctly with Hermes engine | Check module's issues for Hermes-related bugs. Test in development build. |
| Expo New Architecture | Compatible with React Native New Architecture (Fabric/TurboModules) | Check module docs/issues for New Architecture support |

**Currently approved native modules:**

| Module | Version | Purpose | Last Verified |
|--------|---------|---------|---------------|
| `expo-secure-store` | 14.x | Secure token storage (iOS Keychain) | Verified with Expo SDK 52 |
| `expo-notifications` | 0.29.x | Push notifications | Verified with Expo SDK 52 |
| `expo-image` | 2.x | High-performance image loading | Verified with Expo SDK 52 |
| `expo-splash-screen` | 0.29.x | Native splash screen control | Verified with Expo SDK 52 |
| `expo-updates` | 0.26.x | OTA updates | Verified with Expo SDK 52 |
| `expo-haptics` | 14.x | Haptic feedback | Verified with Expo SDK 52 |
| `expo-linking` | 7.x | Deep linking | Verified with Expo SDK 52 |
| `react-native-reanimated` | 3.16.x | Animations (UI thread) | Verified with RN 0.76 |
| `react-native-gesture-handler` | 2.20.x | Gesture recognition | Verified with RN 0.76 |
| `@react-native-community/netinfo` | 11.x | Network state detection | Verified with RN 0.76 |
| `react-native-keyboard-controller` | 1.x | Keyboard animation control | Verified with RN 0.76 |

#### 7.4 Dependency Review Checklist

**Before adding ANY new package to the project, the developer must answer ALL of the following. This checklist is included in the PR template.**

```markdown
## New Dependency Checklist

Package: `<package-name>@<version>`

- [ ] **Actively maintained?** Last commit within 6 months. Link to repo: ___
- [ ] **Hermes compatible?** No known Hermes issues. Tested in dev build: Yes/No
- [ ] **Privacy Manifest?** If the module uses Required Reason APIs (UserDefaults, disk space,
      file timestamps, system boot time, active keyboard list), it must include an
      `NSPrivacyAccessedAPITypes` declaration in its Privacy Manifest. 
      Does it use Required Reason APIs? Yes/No. If Yes, does it include a Privacy Manifest? Yes/No.
- [ ] **Bundle size impact?** Before: ___MB, After: ___MB, Delta: ___MB
      (measured via `npx expo export --platform ios`)
- [ ] **Duplicates existing functionality?** Does this package overlap with something
      we already have? If so, why not extend existing? Explanation: ___
- [ ] **iOS 16+ compatible?** Checked Podspec deployment target: ___
- [ ] **arm64 support?** Verified no architecture exclusions: Yes/No
- [ ] **New Architecture compatible?** Works with Fabric/TurboModules: Yes/No/Unknown
- [ ] **License compatible?** License type: ___. Compatible with commercial use: Yes/No
- [ ] **Tree-shakeable?** Supports ESM imports / `sideEffects: false`: Yes/No
- [ ] **Weekly downloads on npm?** ___k (as a proxy for community adoption)
- [ ] **Open issues count?** ___ open issues (red flag if >100 with many unanswered)
```

**Approval requirement:** Any new dependency with native code (Objective-C, Swift, C++) requires approval from the tech lead. Pure JavaScript dependencies can be added by any team member who completes the checklist.

---

### Appendix A: Performance Monitoring Cheat Sheet

Quick reference for the development team:

| What to Measure | Tool | How |
|----------------|------|-----|
| Cold launch time | Xcode Instruments > App Launch template | Profile on physical iPhone 13 |
| JS thread FPS | React Native Perf Monitor (Dev Menu) | Watch "JS" FPS counter during interaction |
| UI thread FPS | React Native Perf Monitor | Watch "UI" FPS counter during scroll |
| Component re-renders | React DevTools Profiler | Enable "Highlight updates" — look for unnecessary flashes |
| Bundle size | `npx source-map-explorer` | Run after `expo export` |
| Memory usage | Xcode > Debug Navigator > Memory | Profile on device during typical usage |
| Network waterfall | Flipper > Network plugin | Inspect request timing, caching, deduplication |
| Hermes heap | Flipper > Hermes Debugger | Inspect heap snapshots for leaks |
| Animation performance | Reanimated `useFrameCallback` | Log frame timing in development |

### Appendix B: Performance Budget Summary

Single-page reference of all numeric targets:

| Metric | Target | Hard Limit |
|--------|--------|-----------|
| Cold launch (iPhone 13) | <2,000ms | 2,500ms |
| JS bundle size (compressed) | <15MB | 18MB |
| FPS (all interactions) | 60fps | 55fps minimum |
| JS thread frame budget | <16ms | 16.67ms |
| Memory — ChatScreen | <150MB | 180MB |
| Memory — PeopleScreen | <100MB | 120MB |
| Memory — PropertiesScreen | <120MB | 144MB |
| Memory — Total app | <300MB | 360MB |
| API response time (P50) | <300ms | 500ms |
| API response time (P95) | <800ms | 1,200ms |
| Crash-free rate | >99.5% | 99.0% |
| Test coverage — utils | >80% | 70% |
| Test coverage — hooks | >60% | 50% |
| Offline queue capacity | 100 mutations | - |
| OTA update size | <5MB differential | 10MB |
| Tab switch TTI | <300ms | 500ms |
| TanStack Query staleTime (contacts) | 5min | - |
| TanStack Query staleTime (chat) | 1min | - |
| TanStack Query gcTime (all) | 30min | - |

---

This concludes the V2: Performance, Infrastructure & Build Pipeline specification. Every target is quantified, every configuration has exact values, and every architectural decision is justified with rationale and implementation guidance. The team should treat the "Hard Limit" column as CI-enforced gates — any violation blocks the build and requires immediate remediation.

---


## Section 8: Offline Architecture & State Management

> **Scope**: This section defines the complete offline-first data architecture for Skye, a React Native iOS CRM and AI assistant for real estate agents. It covers state management (server, UI, and offline queue), persistent caching, offline mutation queuing, conflict resolution, network state handling, and real-time sync considerations. Every configuration value, state transition, and data flow is explicit and implementation-ready.

---

### 1. State Management Architecture

Skye uses a **hybrid state management** approach: **TanStack Query v5** (React Query) owns all server/async state, and **Zustand v4** owns all client-side/UI state. These two systems are complementary and never overlap in responsibility.

#### 1.1 TanStack Query (Server State)

TanStack Query is the single source of truth for all data that originates from the Supabase backend. It handles fetching, caching, background refetching, optimistic updates, and mutation lifecycle.

##### 1.1.1 QueryClient Configuration

```typescript
// src/lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // Data is considered fresh for 5 minutes by default.
      // Individual query keys override this (see Section 3).
      staleTime: 5 * 60 * 1000, // 5 minutes

      // Cached data is retained for 7 days before garbage collection.
      // This MUST be >= the persister maxAge to prevent premature eviction
      // of persisted cache entries on hydration.
      gcTime: 7 * 24 * 60 * 60 * 1000, // 7 days

      // Retry policy: 3 retries with exponential backoff.
      retry: 3,
      retryDelay: (attemptIndex: number) =>
        Math.min(1000 * 2 ** attemptIndex, 30000), // 1s, 2s, 4s (cap 30s)

      // Do not refetch on window focus -- controlled manually via AppState.
      refetchOnWindowFocus: false,

      // Refetch when network reconnects (after offline period).
      refetchOnReconnect: 'always',

      // Do not refetch on component mount if data is fresh.
      refetchOnMount: true,

      // Network mode: offlineFirst ensures cached data is returned
      // immediately even when offline, and fetches are paused (not failed).
      networkMode: 'offlineFirst',
    },
    mutations: {
      // Mutations also use offlineFirst so they pause when offline
      // rather than immediately failing.
      networkMode: 'offlineFirst',

      // Retry policy for mutations: 3 retries with exponential backoff.
      retry: 3,
      retryDelay: (attemptIndex: number) =>
        Math.min(1000 * 2 ** attemptIndex, 30000),
    },
  },
});
```

**Key design decisions**:

- `networkMode: 'offlineFirst'` is critical. It tells TanStack Query to serve cached data first and pause network requests when offline, rather than failing them. When connectivity returns, paused queries and mutations resume automatically.
- `gcTime` is set to 7 days (matching the persister `maxAge`) so that cached data survives across app restarts for up to a week. Without this, the default 5-minute `gcTime` would discard persisted data on hydration.
- `refetchOnWindowFocus` is disabled because React Native does not have a browser-style "window focus" event. Instead, Skye manages refetch on app foreground via the `AppState` listener (see Section 5.3).

##### 1.1.2 Query Key Conventions

All query keys follow a hierarchical tuple structure: `[entity, scope, params?]`. This enables targeted invalidation at any level of granularity.

```typescript
// src/lib/queryKeys.ts

export const queryKeys = {
  // -- Contacts --
  contacts: {
    all:      ['contacts'] as const,
    lists:    () => [...queryKeys.contacts.all, 'list'] as const,
    list:     (filters: ContactFilters) =>
                [...queryKeys.contacts.lists(), filters] as const,
    details:  () => [...queryKeys.contacts.all, 'detail'] as const,
    detail:   (id: string) =>
                [...queryKeys.contacts.details(), id] as const,
    notes:    (contactId: string) =>
                [...queryKeys.contacts.all, 'notes', contactId] as const,
    tasks:    (contactId: string) =>
                [...queryKeys.contacts.all, 'tasks', contactId] as const,
    timeline: (contactId: string) =>
                [...queryKeys.contacts.all, 'timeline', contactId] as const,
  },

  // -- Properties --
  properties: {
    all:      ['properties'] as const,
    lists:    () => [...queryKeys.properties.all, 'list'] as const,
    list:     (filters?: PropertyFilters) =>
                [...queryKeys.properties.lists(), filters] as const,
    details:  () => [...queryKeys.properties.all, 'detail'] as const,
    detail:   (id: string) =>
                [...queryKeys.properties.details(), id] as const,
  },

  // -- Chat --
  chat: {
    all:        ['chat'] as const,
    sessions:   () => [...queryKeys.chat.all, 'sessions'] as const,
    history:    (sessionId: string) =>
                  [...queryKeys.chat.all, 'history', sessionId] as const,
    recent:     () => [...queryKeys.chat.all, 'recent'] as const,
  },

  // -- Tasks --
  tasks: {
    all:      ['tasks'] as const,
    lists:    () => [...queryKeys.tasks.all, 'list'] as const,
    list:     (filters?: TaskFilters) =>
                [...queryKeys.tasks.lists(), filters] as const,
    detail:   (id: string) =>
                [...queryKeys.tasks.all, 'detail', id] as const,
  },

  // -- User / Dashboard --
  user: {
    profile:  ['user', 'profile'] as const,
    dashboard: ['user', 'dashboard'] as const,
  },
} as const;
```

**Invalidation patterns by granularity**:

| Action | Invalidation Target | Effect |
|---|---|---|
| Create contact | `queryKeys.contacts.lists()` | Refetches all contact list variants (any filter combo) |
| Edit contact `abc` | `queryKeys.contacts.detail('abc')` | Refetches that contact's detail |
| Delete contact `abc` | `queryKeys.contacts.all` | Refetches all contact queries (lists + details) |
| Add note to contact `abc` | `queryKeys.contacts.notes('abc')` | Refetches that contact's notes |
| Complete task `xyz` | `queryKeys.tasks.detail('xyz')` and `queryKeys.contacts.tasks(contactId)` | Refetches the task and the parent contact's task list |
| Send chat message | `queryKeys.chat.history(sessionId)` | Refetches the active chat session |
| Pull-to-refresh on People tab | `queryKeys.contacts.lists()` | Refetches the current contact list |

##### 1.1.3 Mutation Conventions

All mutations follow a consistent pattern: **optimistic update first, then reconcile on success or rollback on error**.

```typescript
// Example: Edit Contact mutation
// src/hooks/mutations/useEditContact.ts

import { useMutation, useQueryClient } from '@tanstack/react-query';
import { queryKeys } from '@/lib/queryKeys';
import { useOfflineStore } from '@/stores/offlineStore';

export function useEditContact() {
  const queryClient = useQueryClient();
  const { isOnline, enqueueMutation } = useOfflineStore();

  return useMutation({
    mutationFn: async (variables: { id: string; data: Partial<Contact> }) => {
      const response = await api.patch(
        `/api/contacts/${variables.id}`,
        variables.data
      );
      return response.data;
    },

    // -- Optimistic Update --
    onMutate: async (variables) => {
      // 1. Cancel any outgoing refetches for this contact.
      await queryClient.cancelQueries({
        queryKey: queryKeys.contacts.detail(variables.id),
      });

      // 2. Snapshot the current cached value for rollback.
      const previousContact = queryClient.getQueryData<Contact>(
        queryKeys.contacts.detail(variables.id)
      );

      // 3. Optimistically update the cache.
      queryClient.setQueryData<Contact>(
        queryKeys.contacts.detail(variables.id),
        (old) => (old ? { ...old, ...variables.data } : old)
      );

      // 4. Also update the contact in list caches.
      queryClient.setQueriesData<ContactListResponse>(
        { queryKey: queryKeys.contacts.lists() },
        (old) => {
          if (!old) return old;
          return {
            ...old,
            contacts: old.contacts.map((c) =>
              c.id === variables.id ? { ...c, ...variables.data } : c
            ),
          };
        }
      );

      // 5. Return context for rollback.
      return { previousContact };
    },

    // -- Success --
    onSuccess: (_data, variables) => {
      queryClient.invalidateQueries({
        queryKey: queryKeys.contacts.detail(variables.id),
      });
      queryClient.invalidateQueries({
        queryKey: queryKeys.contacts.lists(),
      });
    },

    // -- Error / Rollback --
    onError: (_error, variables, context) => {
      if (context?.previousContact) {
        queryClient.setQueryData(
          queryKeys.contacts.detail(variables.id),
          context.previousContact
        );
      }
    },
  });
}
```

**Mutation pattern summary (applied to all mutations)**:
1. `onMutate`: Cancel related queries, snapshot current cache, apply optimistic update, return snapshot as context.
2. `onSuccess`: Invalidate related queries to refetch canonical server data.
3. `onError`: Restore snapshot from context (rollback).
4. `onSettled` (optional): Used for cleanup regardless of success/failure.

##### 1.1.4 Persisted Cache (SQLite-Backed Persister)

TanStack Query's cache is persisted to SQLite so that queries survive app restarts. Skye uses the `experimental_createQueryPersister` API (per-query persistence) rather than `persistQueryClient` (whole-cache single-key persistence). Per-query persistence avoids serializing the entire cache into a single storage key, which is critical for performance at scale.

```typescript
// src/lib/queryPersister.ts
import { experimental_createQueryPersister } from '@tanstack/query-persist-client-core';
import { db } from '@/lib/database';

const sqliteQueryStorage = {
  getItem: async (key: string): Promise<string | null> => {
    const result = await db.getFirstAsync<{ value: string }>(
      'SELECT value FROM query_cache WHERE key = ?',
      [key]
    );
    return result?.value ?? null;
  },
  setItem: async (key: string, value: string): Promise<void> => {
    await db.runAsync(
      `INSERT OR REPLACE INTO query_cache (key, value, updated_at)
       VALUES (?, ?, ?)`,
      [key, value, Date.now()]
    );
  },
  removeItem: async (key: string): Promise<void> => {
    await db.runAsync('DELETE FROM query_cache WHERE key = ?', [key]);
  },
};

export const queryPersister = experimental_createQueryPersister({
  storage: sqliteQueryStorage,
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days -- matches gcTime
  serialize: JSON.stringify,
  deserialize: JSON.parse,
});
```

**How it works**:
- Each query is persisted individually (keyed by its query hash).
- Queries are lazily restored when first accessed (not all at once on app launch).
- After each `queryFn` execution, the result is persisted to SQLite.
- This eliminates the need for throttled bulk writes and keeps the cache granular.

##### 1.1.5 Online Manager Setup (NetInfo Integration)

TanStack Query does not automatically detect network changes on React Native. Skye bridges `@react-native-community/netinfo` into TanStack Query's `onlineManager`.

```typescript
// src/lib/onlineManager.ts
import NetInfo, { NetInfoState } from '@react-native-community/netinfo';
import { onlineManager } from '@tanstack/react-query';

export function setupOnlineManager(): () => void {
  onlineManager.setEventListener((setOnline) => {
    return NetInfo.addEventListener((state: NetInfoState) => {
      const isConnected =
        state.isConnected != null &&
        state.isConnected &&
        Boolean(state.isInternetReachable);
      setOnline(isConnected);
    });
  });

  return NetInfo.addEventListener(() => {});
}
```

**Important**: `state.isConnected` alone is insufficient. A device can be "connected" to Wi-Fi but have no internet access. `state.isInternetReachable` provides the additional check. However, even this is advisory; Skye also performs a periodic health-check ping (see Section 5.4).

---

#### 1.2 Zustand Stores (Client/UI State)

Zustand manages all state that does NOT originate from the server: authentication state, UI preferences, chat drafts, offline queue metadata, and notification state. Each concern gets its own store (no mega-store).

##### 1.2.1 Store Definitions

**Auth Store** -- Persisted to `expo-secure-store` (encrypted, OS keychain-backed):

```typescript
// src/stores/authStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { secureStorage } from '@/lib/secureStorage';

interface AuthState {
  accessToken: string | null;
  refreshToken: string | null;
  user: UserProfile | null;
  authStatus: 'idle' | 'loading' | 'authenticated' | 'unauthenticated';
  setTokens: (access: string, refresh: string) => void;
  setUser: (user: UserProfile) => void;
  clearAuth: () => void;
  setAuthStatus: (status: AuthState['authStatus']) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      accessToken: null,
      refreshToken: null,
      user: null,
      authStatus: 'idle',
      setTokens: (access, refresh) =>
        set({ accessToken: access, refreshToken: refresh }),
      setUser: (user) => set({ user }),
      clearAuth: () =>
        set({
          accessToken: null,
          refreshToken: null,
          user: null,
          authStatus: 'unauthenticated',
        }),
      setAuthStatus: (status) => set({ authStatus: status }),
    }),
    {
      name: 'skye-auth',
      storage: createJSONStorage(() => secureStorage),
      partialize: (state) => ({
        accessToken: state.accessToken,
        refreshToken: state.refreshToken,
        user: state.user,
      }),
    }
  )
);
```

**UI Store** -- Persisted to MMKV (fast, synchronous key-value):

```typescript
// src/stores/uiStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { mmkvStorage } from '@/lib/mmkvStorage';

interface UIState {
  activeTab: 'people' | 'properties' | 'chat' | 'tasks' | 'settings';
  contactListFilters: ContactFilters;
  contactListSort: SortConfig;
  propertyListFilters: PropertyFilters;
  themeOverride: 'light' | 'dark' | 'system';
  hasCompletedOnboarding: boolean;
  lastViewedContactId: string | null;
  setActiveTab: (tab: UIState['activeTab']) => void;
  setContactListFilters: (filters: ContactFilters) => void;
  setContactListSort: (sort: SortConfig) => void;
  setPropertyListFilters: (filters: PropertyFilters) => void;
  setThemeOverride: (theme: UIState['themeOverride']) => void;
  setOnboardingComplete: () => void;
  setLastViewedContactId: (id: string | null) => void;
}

export const useUIStore = create<UIState>()(
  persist(
    (set) => ({
      activeTab: 'people',
      contactListFilters: { status: 'all', search: '' },
      contactListSort: { field: 'name', direction: 'asc' },
      propertyListFilters: { status: 'all', search: '' },
      themeOverride: 'system',
      hasCompletedOnboarding: false,
      lastViewedContactId: null,
      setActiveTab: (tab) => set({ activeTab: tab }),
      setContactListFilters: (filters) => set({ contactListFilters: filters }),
      setContactListSort: (sort) => set({ contactListSort: sort }),
      setPropertyListFilters: (filters) => set({ propertyListFilters: filters }),
      setThemeOverride: (theme) => set({ themeOverride: theme }),
      setOnboardingComplete: () => set({ hasCompletedOnboarding: true }),
      setLastViewedContactId: (id) => set({ lastViewedContactId: id }),
    }),
    {
      name: 'skye-ui',
      storage: createJSONStorage(() => mmkvStorage),
    }
  )
);
```

**Chat Store** -- Persisted to MMKV (draft message survives app kill):

```typescript
// src/stores/chatStore.ts
interface ChatState {
  activeSessionId: string | null;
  inputDraft: string;
  isStreaming: boolean;
  streamingMessageId: string | null;
  streamedContent: string;
  setActiveSession: (id: string | null) => void;
  setInputDraft: (text: string) => void;
  setStreaming: (streaming: boolean, messageId?: string | null) => void;
  appendStreamedContent: (chunk: string) => void;
  clearStreamedContent: () => void;
}

export const useChatStore = create<ChatState>()(
  persist(
    (set) => ({
      activeSessionId: null,
      inputDraft: '',
      isStreaming: false,
      streamingMessageId: null,
      streamedContent: '',
      setActiveSession: (id) => set({ activeSessionId: id }),
      setInputDraft: (text) => set({ inputDraft: text }),
      setStreaming: (streaming, messageId = null) =>
        set({ isStreaming: streaming, streamingMessageId: messageId }),
      appendStreamedContent: (chunk) =>
        set((state) => ({ streamedContent: state.streamedContent + chunk })),
      clearStreamedContent: () =>
        set({ streamedContent: '', streamingMessageId: null }),
    }),
    {
      name: 'skye-chat',
      storage: createJSONStorage(() => mmkvStorage),
      // Only persist the draft and active session -- NOT streaming state.
      partialize: (state) => ({
        activeSessionId: state.activeSessionId,
        inputDraft: state.inputDraft,
      }),
    }
  )
);
```

**Offline Store** -- Persisted to MMKV (metadata only; actual queue is in SQLite):

```typescript
// src/stores/offlineStore.ts
type NetworkQuality = 'strong' | 'weak' | 'offline';
type SyncStatus = 'idle' | 'syncing' | 'error';

interface OfflineState {
  networkQuality: NetworkQuality;
  syncStatus: SyncStatus;
  pendingMutationCount: number;
  lastSyncTimestamp: number | null;
  failedMutationCount: number;
  setNetworkQuality: (quality: NetworkQuality) => void;
  setSyncStatus: (status: SyncStatus) => void;
  setPendingMutationCount: (count: number) => void;
  setLastSyncTimestamp: (ts: number) => void;
  setFailedMutationCount: (count: number) => void;
}

export const useOfflineStore = create<OfflineState>()(
  persist(
    (set) => ({
      networkQuality: 'strong',
      syncStatus: 'idle',
      pendingMutationCount: 0,
      lastSyncTimestamp: null,
      failedMutationCount: 0,
      setNetworkQuality: (quality) => set({ networkQuality: quality }),
      setSyncStatus: (status) => set({ syncStatus: status }),
      setPendingMutationCount: (count) => set({ pendingMutationCount: count }),
      setLastSyncTimestamp: (ts) => set({ lastSyncTimestamp: ts }),
      setFailedMutationCount: (count) => set({ failedMutationCount: count }),
    }),
    {
      name: 'skye-offline',
      storage: createJSONStorage(() => mmkvStorage),
      partialize: (state) => ({ lastSyncTimestamp: state.lastSyncTimestamp }),
    }
  )
);
```

**Notification Store** -- Persisted to MMKV:

```typescript
// src/stores/notificationStore.ts
interface NotificationState {
  badgeCounts: { people: number; tasks: number; chat: number };
  pendingDeepLink: string | null;
  pushPermissionStatus: 'granted' | 'denied' | 'undetermined';
  setBadgeCount: (tab: keyof NotificationState['badgeCounts'], count: number) => void;
  clearBadgeCount: (tab: keyof NotificationState['badgeCounts']) => void;
  setPendingDeepLink: (link: string | null) => void;
  setPushPermissionStatus: (status: NotificationState['pushPermissionStatus']) => void;
}

export const useNotificationStore = create<NotificationState>()(
  persist(
    (set) => ({
      badgeCounts: { people: 0, tasks: 0, chat: 0 },
      pendingDeepLink: null,
      pushPermissionStatus: 'undetermined',
      setBadgeCount: (tab, count) =>
        set((state) => ({ badgeCounts: { ...state.badgeCounts, [tab]: count } })),
      clearBadgeCount: (tab) =>
        set((state) => ({ badgeCounts: { ...state.badgeCounts, [tab]: 0 } })),
      setPendingDeepLink: (link) => set({ pendingDeepLink: link }),
      setPushPermissionStatus: (status) => set({ pushPermissionStatus: status }),
    }),
    {
      name: 'skye-notifications',
      storage: createJSONStorage(() => mmkvStorage),
      partialize: (state) => ({
        badgeCounts: state.badgeCounts,
        pushPermissionStatus: state.pushPermissionStatus,
      }),
    }
  )
);
```

##### 1.2.2 Storage Adapters

```typescript
// src/lib/mmkvStorage.ts
import { MMKV } from 'react-native-mmkv';
import { StateStorage } from 'zustand/middleware';

export const mmkv = new MMKV({ id: 'skye-app-storage' });

export const mmkvStorage: StateStorage = {
  setItem: (name: string, value: string) => { mmkv.set(name, value); },
  getItem: (name: string) => mmkv.getString(name) ?? null,
  removeItem: (name: string) => { mmkv.delete(name); },
};
```

```typescript
// src/lib/secureStorage.ts
import * as SecureStore from 'expo-secure-store';
import { StateStorage } from 'zustand/middleware';

export const secureStorage: StateStorage = {
  setItem: async (name: string, value: string) => {
    await SecureStore.setItemAsync(name, value, {
      keychainAccessible: SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
    });
  },
  getItem: async (name: string) => await SecureStore.getItemAsync(name),
  removeItem: async (name: string) => { await SecureStore.deleteItemAsync(name); },
};
```

**Storage selection rationale**:

| Store | Storage Backend | Why |
|---|---|---|
| `useAuthStore` | `expo-secure-store` | Tokens are sensitive. iOS Keychain provides hardware-backed encryption. |
| `useUIStore` | MMKV | Fast synchronous reads. UI state is not sensitive. Accessed frequently. |
| `useChatStore` | MMKV | Draft persistence needs to be fast and survive app kill. Not sensitive. |
| `useOfflineStore` | MMKV (metadata only) | Only `lastSyncTimestamp` is persisted. Actual mutation queue lives in SQLite. |
| `useNotificationStore` | MMKV | Badge counts and permission status. Not sensitive. Fast reads for tab bar. |

##### 1.2.3 State Initialization Sequence

On every app launch, the following sequence executes in order. The app shows a splash screen until step 5 completes.

```
App Launch
  |
  v
Step 1: Restore Zustand persisted state (synchronous for MMKV stores)
  |   - useUIStore, useChatStore, useOfflineStore, useNotificationStore
  |     hydrate from MMKV synchronously (< 1ms each).
  |   - useAuthStore hydrates from expo-secure-store (async, ~10-50ms).
  |
  v
Step 2: Validate authentication
  |   - Read accessToken from useAuthStore.
  |   - If no token: authStatus = 'unauthenticated', navigate to login.
  |   - If token exists: decode JWT, check expiry.
  |     - If expired: attempt silent refresh via refreshToken.
  |       - Refresh success: update tokens, authStatus = 'authenticated'.
  |       - Refresh failure: authStatus = 'unauthenticated', navigate to login.
  |     - If valid: authStatus = 'authenticated'.
  |
  v
Step 3: Initialize TanStack Query online manager
  |   - Call setupOnlineManager() to bridge NetInfo into TanStack Query.
  |   - Determine initial network quality (strong / weak / offline).
  |
  v
Step 4: Hydrate TanStack Query cache from SQLite
  |   - experimental_createQueryPersister handles this lazily
  |     (queries are restored on first access, not all at once).
  |   - No blocking wait needed.
  |
  v
Step 5: App Ready
  |   - Hide splash screen.
  |   - If authenticated: show main app (People tab by default).
  |   - If unauthenticated: show login screen.
  |   - If pending deep link exists: navigate to target.
  |
  v
Step 6: Background cache warming (non-blocking, after UI is visible)
      - If online: prefetch contact list, first page of properties, recent chat.
      - If offline: skip -- cached data from persister already available.
      - Process any pending offline mutations.
```

**Timing budget**: Target < 500ms total. MMKV hydration is < 5ms. SecureStore hydration is < 50ms. JWT decode is < 5ms.

---

### 2. Offline Storage Schema (SQLite)

#### 2.1 Database Setup

```typescript
// src/lib/database.ts
import * as SQLite from 'expo-sqlite';

let _db: SQLite.SQLiteDatabase | null = null;

export async function initDatabase(): Promise<SQLite.SQLiteDatabase> {
  if (_db) return _db;
  _db = await SQLite.openDatabaseAsync('skye.db');

  // Enable WAL mode for concurrent read/write performance.
  await _db.execAsync('PRAGMA journal_mode = WAL;');
  await _db.execAsync('PRAGMA foreign_keys = ON;');

  await runMigrations(_db);
  return _db;
}
```

#### 2.2 Schema Definition (Migration v1)

```sql
-- Schema Version Tracking
CREATE TABLE IF NOT EXISTS schema_version (
  version     INTEGER NOT NULL,
  applied_at  INTEGER NOT NULL DEFAULT (strftime('%s', 'now') * 1000)
);

-- TanStack Query Cache (per-query persistence)
CREATE TABLE IF NOT EXISTS query_cache (
  key         TEXT    PRIMARY KEY NOT NULL,
  value       TEXT    NOT NULL,
  updated_at  INTEGER NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_query_cache_updated ON query_cache(updated_at);

-- Contacts Cache
CREATE TABLE IF NOT EXISTS contacts (
  id          TEXT    PRIMARY KEY NOT NULL,
  data        TEXT    NOT NULL,          -- JSON: full contact object
  updated_at  INTEGER NOT NULL,          -- server-side updated_at (ms)
  accessed_at INTEGER NOT NULL,          -- last time this row was read (ms)
  is_stale    INTEGER NOT NULL DEFAULT 0
);
CREATE INDEX IF NOT EXISTS idx_contacts_updated ON contacts(updated_at);
CREATE INDEX IF NOT EXISTS idx_contacts_accessed ON contacts(accessed_at);

-- Contact Details Cache (lazy-loaded, separate from list data)
CREATE TABLE IF NOT EXISTS contact_details (
  contact_id  TEXT    PRIMARY KEY NOT NULL,
  notes       TEXT,                      -- JSON array
  deals       TEXT,                      -- JSON array
  tasks       TEXT,                      -- JSON array
  timeline    TEXT,                      -- JSON array
  updated_at  INTEGER NOT NULL,
  FOREIGN KEY (contact_id) REFERENCES contacts(id) ON DELETE CASCADE
);
CREATE INDEX IF NOT EXISTS idx_contact_details_updated ON contact_details(updated_at);

-- Properties Cache
CREATE TABLE IF NOT EXISTS properties (
  id          TEXT    PRIMARY KEY NOT NULL,
  data        TEXT    NOT NULL,
  updated_at  INTEGER NOT NULL,
  accessed_at INTEGER NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_properties_updated ON properties(updated_at);
CREATE INDEX IF NOT EXISTS idx_properties_accessed ON properties(accessed_at);

-- Chat Sessions Cache
CREATE TABLE IF NOT EXISTS chat_sessions (
  id          TEXT    PRIMARY KEY NOT NULL,
  messages    TEXT    NOT NULL,          -- JSON array
  updated_at  INTEGER NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_chat_sessions_updated ON chat_sessions(updated_at);

-- Offline Mutation Queue
CREATE TABLE IF NOT EXISTS mutation_queue (
  id              TEXT    PRIMARY KEY NOT NULL,
  type            TEXT    NOT NULL,
  endpoint        TEXT    NOT NULL,
  method          TEXT    NOT NULL,
  body            TEXT    NOT NULL,
  headers         TEXT,
  created_at      INTEGER NOT NULL,
  status          TEXT    NOT NULL DEFAULT 'pending',
  retry_count     INTEGER NOT NULL DEFAULT 0,
  max_retries     INTEGER NOT NULL DEFAULT 3,
  next_retry_at   INTEGER,
  error           TEXT,
  completed_at    INTEGER,
  idempotency_key TEXT    UNIQUE,
  related_entity_id   TEXT,
  related_entity_type TEXT
);
CREATE INDEX IF NOT EXISTS idx_mutation_queue_status ON mutation_queue(status);
CREATE INDEX IF NOT EXISTS idx_mutation_queue_created ON mutation_queue(created_at);
CREATE INDEX IF NOT EXISTS idx_mutation_queue_related
  ON mutation_queue(related_entity_type, related_entity_id);
CREATE INDEX IF NOT EXISTS idx_mutation_queue_next_retry
  ON mutation_queue(next_retry_at) WHERE status = 'pending' OR status = 'failed';

-- Dead Letter Queue
CREATE TABLE IF NOT EXISTS dead_letter_queue (
  id                TEXT    PRIMARY KEY NOT NULL,
  original_mutation TEXT    NOT NULL,
  failure_reason    TEXT    NOT NULL,
  moved_at          INTEGER NOT NULL,
  user_action       TEXT,
  resolved_at       INTEGER
);

-- Image Cache Metadata
CREATE TABLE IF NOT EXISTS image_cache (
  url         TEXT    PRIMARY KEY NOT NULL,
  local_path  TEXT    NOT NULL,
  size_bytes  INTEGER NOT NULL,
  mime_type   TEXT,
  accessed_at INTEGER NOT NULL,
  created_at  INTEGER NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_image_cache_accessed ON image_cache(accessed_at);
CREATE INDEX IF NOT EXISTS idx_image_cache_size ON image_cache(size_bytes);

-- App Metadata
CREATE TABLE IF NOT EXISTS metadata (
  key   TEXT PRIMARY KEY NOT NULL,
  value TEXT NOT NULL
);

INSERT OR IGNORE INTO schema_version (version) VALUES (1);
```

#### 2.3 Migration System

```typescript
// src/lib/migrations.ts
interface Migration {
  version: number;
  up: (db: SQLiteDatabase) => Promise<void>;
  description: string;
}

const migrations: Migration[] = [
  // v1 is the initial schema (above). Future migrations added here.
];

export async function runMigrations(db: SQLiteDatabase): Promise<void> {
  const result = await db.getFirstAsync<{ version: number }>(
    'SELECT MAX(version) as version FROM schema_version'
  ).catch(() => null);
  const currentVersion = result?.version ?? 0;

  for (const migration of migrations) {
    if (migration.version > currentVersion) {
      await db.execAsync('BEGIN TRANSACTION;');
      try {
        await migration.up(db);
        await db.runAsync(
          'INSERT INTO schema_version (version) VALUES (?)',
          [migration.version]
        );
        await db.execAsync('COMMIT;');
      } catch (error) {
        await db.execAsync('ROLLBACK;');
        throw new Error(`Migration v${migration.version} failed: ${error}`);
      }
    }
  }
}
```

---

### 3. Cache Strategy

#### 3.1 Cache-First Pattern

Skye uses a **cache-first** pattern for all data screens. The user always sees data immediately (from cache), and a background refresh updates the display when available. This eliminates loading spinners for repeat visits.

```
User taps tab
  |
  v
TanStack Query checks in-memory cache
  |
  +--> Cache HIT
  |      |
  |      v
  |    Render cached data immediately (0ms perceived latency)
  |      |
  |      v
  |    Stale? --> YES: background refetch + subtle indicator
  |           --> NO: done, no network request
  |
  +--> Cache MISS
         |
         v
       Show skeleton loading --> fetch --> populate cache --> render
```

#### 3.2 Stale Times by Entity

| Entity | `staleTime` | Rationale |
|---|---|---|
| Contact list | 5 minutes | Contacts change infrequently |
| Contact detail | 5 minutes | Same; invalidated immediately on local edit |
| Property list | 15 minutes | Property data changes less frequently (MLS sync cadence) |
| Property detail | 15 minutes | Same as list |
| Chat history | 1 minute | Messages can arrive from AI or other sessions |
| Chat sessions list | 2 minutes | Changes when new chats are started |
| Task list | 3 minutes | Tasks are time-sensitive |
| User profile | 30 minutes | Rarely changes |
| Dashboard stats | 5 minutes | Should feel reasonably current |

#### 3.3 Cache Invalidation Triggers

**Time-based**: Handled automatically by TanStack Query's `staleTime`.

**Event-based**:

```typescript
// src/lib/cacheInvalidation.ts
export const cacheInvalidation = {
  afterContactMutation: (contactId?: string) => {
    queryClient.invalidateQueries({ queryKey: queryKeys.contacts.lists() });
    if (contactId) {
      queryClient.invalidateQueries({
        queryKey: queryKeys.contacts.detail(contactId),
      });
    }
  },

  afterPushNotification: (payload: PushPayload) => {
    switch (payload.type) {
      case 'contact_updated':
        queryClient.invalidateQueries({ queryKey: queryKeys.contacts.detail(payload.entityId) });
        queryClient.invalidateQueries({ queryKey: queryKeys.contacts.lists() });
        break;
      case 'new_message':
        queryClient.invalidateQueries({ queryKey: queryKeys.chat.history(payload.sessionId) });
        break;
      case 'task_assigned':
        queryClient.invalidateQueries({ queryKey: queryKeys.tasks.all });
        break;
      default:
        queryClient.invalidateQueries();
    }
  },

  afterExtendedBackground: () => {
    queryClient.invalidateQueries(); // Invalidate all
  },

  afterQueueDrain: () => {
    queryClient.invalidateQueries({ queryKey: queryKeys.contacts.all });
    queryClient.invalidateQueries({ queryKey: queryKeys.tasks.all });
    queryClient.invalidateQueries({ queryKey: queryKeys.properties.all });
  },
};
```

**App foreground invalidation** (after >5 minutes backgrounded):

```typescript
// src/hooks/useAppForegroundRefresh.ts
const BACKGROUND_THRESHOLD_MS = 5 * 60 * 1000;

export function useAppForegroundRefresh() {
  const backgroundTimestamp = useRef<number | null>(null);

  useEffect(() => {
    const subscription = AppState.addEventListener('change', (nextState) => {
      if (nextState === 'background' || nextState === 'inactive') {
        backgroundTimestamp.current = Date.now();
      }
      if (nextState === 'active') {
        const elapsed = backgroundTimestamp.current
          ? Date.now() - backgroundTimestamp.current : 0;
        if (elapsed > BACKGROUND_THRESHOLD_MS) {
          cacheInvalidation.afterExtendedBackground();
        }
        backgroundTimestamp.current = null;
      }
    });
    return () => subscription.remove();
  }, []);
}
```

**Manual**: Pull-to-refresh calls `refetch()` on the active query.

#### 3.4 Cache Eviction

| Rule | Detail |
|---|---|
| Data cache limit | 50 MB total across contacts, properties, chat sessions, query cache |
| Image cache limit | 100 MB (separate from data cache) |
| Eviction strategy | LRU based on `accessed_at` column |
| Eviction target | When limit exceeded, free down to 80% of limit |
| Protected from eviction | Contacts with pending offline mutations; currently viewed items |
| Eviction schedule | After every successful sync; on app foreground; on storage pressure |

```typescript
// src/lib/cacheEviction.ts
const MAX_DATA_CACHE_BYTES = 50 * 1024 * 1024;
const MAX_IMAGE_CACHE_BYTES = 100 * 1024 * 1024;

export async function evictDataCacheIfNeeded(db: SQLiteDatabase): Promise<void> {
  // Calculate total cache size across all tables.
  const sizeResult = await db.getFirstAsync<{ total: number }>(`
    SELECT
      COALESCE(SUM(LENGTH(data)), 0) +
      COALESCE((SELECT SUM(LENGTH(notes)+LENGTH(deals)+LENGTH(tasks)+LENGTH(timeline))
                FROM contact_details), 0) +
      COALESCE((SELECT SUM(LENGTH(data)) FROM properties), 0) +
      COALESCE((SELECT SUM(LENGTH(messages)) FROM chat_sessions), 0) +
      COALESCE((SELECT SUM(LENGTH(value)) FROM query_cache), 0)
    AS total FROM contacts
  `);
  const totalBytes = sizeResult?.total ?? 0;
  if (totalBytes <= MAX_DATA_CACHE_BYTES) return;

  const bytesToFree = totalBytes - MAX_DATA_CACHE_BYTES * 0.8;

  // Get protected IDs (contacts with pending mutations).
  const protectedIds = await db.getAllAsync<{ related_entity_id: string }>(`
    SELECT DISTINCT related_entity_id FROM mutation_queue
    WHERE related_entity_type = 'contact'
      AND status IN ('pending', 'processing', 'failed')
      AND related_entity_id IS NOT NULL
  `);
  const protectedSet = protectedIds.map((r) => r.related_entity_id);

  // LRU eviction: oldest accessed contacts first, skipping protected.
  const candidates = await db.getAllAsync<{ id: string; size: number }>(`
    SELECT id, LENGTH(data) as size FROM contacts
    WHERE id NOT IN (${protectedSet.map(() => '?').join(',') || "''"})
    ORDER BY accessed_at ASC
  `, protectedSet);

  let freed = 0;
  const toEvict: string[] = [];
  for (const c of candidates) {
    if (freed >= bytesToFree) break;
    toEvict.push(c.id);
    freed += c.size;
  }

  if (toEvict.length > 0) {
    const ph = toEvict.map(() => '?').join(',');
    await db.runAsync(`DELETE FROM contacts WHERE id IN (${ph})`, toEvict);
    await db.runAsync(`DELETE FROM contact_details WHERE contact_id IN (${ph})`, toEvict);
  }

  // Same pattern for properties, chat_sessions, and old query_cache entries.
}
```

Image cache eviction follows the same LRU pattern, also deleting the actual files from disk via `expo-file-system`.

#### 3.5 Cache Warming

On login (or first launch after authentication), Skye proactively prefetches the most-needed data:

```typescript
// src/lib/cacheWarming.ts
export async function warmCache(): Promise<void> {
  await Promise.allSettled([
    queryClient.prefetchQuery({
      queryKey: queryKeys.contacts.list({ status: 'all', search: '' }),
      queryFn: () => api.getContacts({ status: 'all', search: '' }),
      staleTime: 5 * 60 * 1000,
    }),
    queryClient.prefetchQuery({
      queryKey: queryKeys.properties.list(),
      queryFn: () => api.getProperties(),
      staleTime: 15 * 60 * 1000,
    }),
    queryClient.prefetchQuery({
      queryKey: queryKeys.chat.recent(),
      queryFn: () => api.getRecentChats(),
      staleTime: 1 * 60 * 1000,
    }),
    queryClient.prefetchQuery({
      queryKey: queryKeys.tasks.list(),
      queryFn: () => api.getTasks(),
      staleTime: 3 * 60 * 1000,
    }),
    queryClient.prefetchQuery({
      queryKey: queryKeys.user.profile,
      queryFn: () => api.getUserProfile(),
      staleTime: 30 * 60 * 1000,
    }),
  ]);
}
```

---

### 4. Offline Mutation Queue

#### 4.1 Queue Architecture

The offline mutation queue is a durable FIFO queue backed by the `mutation_queue` SQLite table. It guarantees that every user action taken while offline is persisted, survives app kills, and is replayed in correct order when connectivity returns.

```
User performs mutation
  |
  v
Is device online?
  +--> YES: Execute directly via TanStack Query mutationFn.
  |         On network error: fall through to offline path.
  |
  +--> NO (or network error):
         1. Generate UUID + idempotency key.
         2. Apply optimistic update to TanStack Query cache.
         3. INSERT into mutation_queue (status: 'pending').
         4. Update useOfflineStore.pendingMutationCount.
         5. Show "Pending sync" badge on affected entity.
```

#### 4.2 Supported Offline Mutations

| Mutation | Method | Endpoint | Offline Behavior |
|---|---|---|---|
| Create contact | `POST` | `/api/crm/contacts` | Queued. Temp client UUID assigned. Mapped to server ID on sync. |
| Edit contact | `PATCH` | `/api/contacts/[id]` | Queued. Optimistic update shown. |
| Delete contact | `DELETE` | `/api/contacts/[id]` | Queued. Item hidden from list. Restored on sync failure. |
| Add note | `POST` | `/api/contacts/[id]/notes` | Queued. Note appears with "pending" badge. |
| Edit note | `PATCH` | `/api/contacts/[id]/notes/[noteId]` | Queued. Optimistic update. |
| Create task | `POST` | `/api/tasks` | Queued. Task appears with "pending" badge. |
| Complete task | `PATCH` | `/api/tasks/[id]` | Queued. Marked complete immediately. |
| Send chat message | `POST` | `/api/chat` | Queued with warning: "AI responses require connectivity." |

**Not supported offline** (disabled in UI):
- Starting new AI chat sessions (requires server-side session creation).
- Uploading images/documents (too large for reliable queue).
- Bulk import/export operations.
- Account settings changes.

#### 4.3 Enqueue Implementation

```typescript
// src/lib/mutationQueue.ts
import { v4 as uuidv4 } from 'uuid';

export async function enqueueMutation(params: {
  type: MutationType;
  endpoint: string;
  method: 'POST' | 'PATCH' | 'PUT' | 'DELETE';
  body: Record<string, unknown>;
  relatedEntityId?: string;
  relatedEntityType?: EntityType;
}): Promise<string> {
  const db = await getDatabase();
  const id = uuidv4();
  const now = Date.now();
  const idempotencyKey = `${params.type}:${params.relatedEntityId ?? 'new'}:${now}`;

  await db.runAsync(
    `INSERT INTO mutation_queue
       (id, type, endpoint, method, body, created_at, status, retry_count,
        max_retries, idempotency_key, related_entity_id, related_entity_type)
     VALUES (?, ?, ?, ?, ?, ?, 'pending', 0, 3, ?, ?, ?)`,
    [id, params.type, params.endpoint, params.method,
     JSON.stringify(params.body), now, idempotencyKey,
     params.relatedEntityId ?? null, params.relatedEntityType ?? null]
  );

  await refreshPendingCount();
  return id;
}
```

#### 4.4 Queue Processor

```typescript
// src/lib/queueProcessor.ts

// Map of client IDs to server IDs for entities created offline.
const idRemapping = new Map<string, string>();

export async function processQueue(): Promise<void> {
  const offlineStore = useOfflineStore.getState();
  if (offlineStore.syncStatus === 'syncing') return;
  if (offlineStore.networkQuality === 'offline') return;

  const db = await getDatabase();
  offlineStore.setSyncStatus('syncing');

  try {
    // FIFO: process sequentially (order matters).
    const pendingMutations = await db.getAllAsync<QueuedMutation>(`
      SELECT * FROM mutation_queue
      WHERE status IN ('pending', 'failed')
        AND (next_retry_at IS NULL OR next_retry_at <= ?)
      ORDER BY created_at ASC
    `, [Date.now()]);

    for (const mutation of pendingMutations) {
      await processSingleMutation(db, mutation);
    }

    cacheInvalidation.afterQueueDrain();
    offlineStore.setSyncStatus('idle');
    offlineStore.setLastSyncTimestamp(Date.now());
  } catch (error) {
    offlineStore.setSyncStatus('error');
  } finally {
    await refreshPendingCount();
  }
}

async function processSingleMutation(db: SQLiteDatabase, mutation: QueuedMutation) {
  await db.runAsync(`UPDATE mutation_queue SET status = 'processing' WHERE id = ?`, [mutation.id]);

  try {
    const endpoint = remapIds(mutation.endpoint);
    const body = JSON.parse(JSON.stringify(mutation.body));
    remapBodyIds(body);

    const response = await api.request({
      method: mutation.method,
      url: endpoint,
      data: body,
      headers: { ...mutation.headers, 'Idempotency-Key': mutation.idempotencyKey },
      timeout: 30000,
    });

    // Record ID mapping for "create" mutations.
    if (mutation.type.startsWith('create_') && response.data?.id) {
      const clientId = mutation.relatedEntityId;
      if (clientId && clientId !== response.data.id) {
        idRemapping.set(clientId, response.data.id);
      }
    }

    await db.runAsync(
      `UPDATE mutation_queue SET status = 'completed', completed_at = ?, error = NULL WHERE id = ?`,
      [Date.now(), mutation.id]
    );
  } catch (error: any) {
    await handleMutationError(db, mutation, error);
  }
}
```

**Error handling logic**:

```typescript
async function handleMutationError(db: SQLiteDatabase, mutation: QueuedMutation, error: any) {
  const statusCode = error?.response?.status;
  const newRetryCount = mutation.retryCount + 1;
  const permanentFailureCodes = [400, 403, 404, 422];

  // 409 Conflict: special handling (see Section 4.5).
  if (statusCode === 409) {
    await handleConflict(db, mutation, error.response.data);
    return;
  }

  // Other permanent failures: dead letter.
  if (permanentFailureCodes.includes(statusCode)) {
    await moveToDeadLetter(db, mutation, { statusCode, message: error.response?.data?.message });
    return;
  }

  // Transient failures: retry with exponential backoff.
  if (newRetryCount >= mutation.maxRetries) {
    await moveToDeadLetter(db, mutation, {
      statusCode,
      message: `Max retries (${mutation.maxRetries}) exceeded. Last: ${error.message}`,
    });
    return;
  }

  const backoffMs = Math.min(1000 * Math.pow(2, newRetryCount), 60000);
  await db.runAsync(
    `UPDATE mutation_queue
     SET status = 'failed', retry_count = ?, next_retry_at = ?, error = ?
     WHERE id = ?`,
    [newRetryCount, Date.now() + backoffMs, JSON.stringify({ statusCode, message: error.message }), mutation.id]
  );
}
```

#### 4.5 Conflict Resolution

```typescript
// Field-level strategy mapping.
const fieldStrategies: Record<string, 'last_write_wins' | 'append' | 'user_merge'> = {
  name: 'last_write_wins',
  email: 'last_write_wins',
  phone: 'last_write_wins',
  address: 'last_write_wins',
  status: 'last_write_wins',
  company: 'last_write_wins',
  title: 'last_write_wins',
  notes: 'append',    // Both offline note and remote notes are kept.
  tags: 'append',
  deals: 'user_merge', // Complex -- needs user decision.
};
```

When the server returns 409:
1. For each changed field, apply the field-level strategy.
2. `last_write_wins`: client value wins (the agent made the most recent intentional change).
3. `append`: merge both arrays (deduplicate by ID if applicable).
4. `user_merge`: mark the mutation as needing user resolution. The `SyncStatusBadge` shows "Conflict -- Tap to resolve" and opens a merge UI showing both versions side by side.

If all fields are auto-resolved, the mutation is retried with the merged data. If any field requires `user_merge`, the mutation is paused until the user resolves it.

#### 4.6 Queue Processing Triggers

```typescript
// src/hooks/useQueueTriggers.ts
export function useQueueTriggers() {
  useEffect(() => {
    // Trigger 1: Network restoration.
    const netInfoUnsub = NetInfo.addEventListener((state) => {
      const wasOffline = useOfflineStore.getState().networkQuality === 'offline';
      const isNowOnline = state.isConnected && state.isInternetReachable;
      if (wasOffline && isNowOnline) {
        setTimeout(() => processQueue(), 2000); // Stabilize connection first
      }
    });

    // Trigger 2: App foreground.
    const appStateSub = AppState.addEventListener('change', (nextState) => {
      if (nextState === 'active') {
        const { networkQuality, pendingMutationCount } = useOfflineStore.getState();
        if (networkQuality !== 'offline' && pendingMutationCount > 0) {
          processQueue();
        }
      }
    });

    return () => { netInfoUnsub(); appStateSub.remove(); };
  }, []);
}

// Trigger 3: Manual sync button (exposed for UI).
export { processQueue as manualSync };
```

#### 4.7 UI Indicators

```typescript
// Entity sync status hook
export type EntitySyncStatus = 'synced' | 'pending' | 'syncing' | 'failed' | 'conflict';

export function useEntitySyncStatus(entityType: string, entityId: string): EntitySyncStatus {
  // Polls mutation_queue every 2 seconds for this entity.
  // Returns 'synced' if no pending mutations.
  // Returns 'pending' | 'syncing' | 'failed' | 'conflict' based on queue status.
}
```

| Status | Visual Treatment |
|---|---|
| `pending` | Amber badge with clock icon: "Pending sync" |
| `syncing` | Blue badge with spinning icon: "Syncing..." |
| `synced` | Brief green flash, then badge removed |
| `failed` | Red badge with exclamation: "Sync failed -- Tap to retry" |
| `conflict` | Orange badge with merge icon: "Conflict -- Tap to resolve" |

---

### 5. Network State Management

#### 5.1 Network Quality Detection

```typescript
// src/lib/networkManager.ts
export function classifyNetworkQuality(state: NetInfoState): NetworkQuality {
  if (!state.isConnected || !state.isInternetReachable) return 'offline';

  if (state.type === NetInfoStateType.wifi || state.type === NetInfoStateType.ethernet) {
    return 'strong';
  }

  if (state.type === NetInfoStateType.cellular) {
    const gen = state.details?.cellularGeneration;
    if (gen === '4g' || gen === '5g') return 'strong';
    return 'weak'; // 3G, 2G, or unknown
  }

  return 'weak'; // Unknown type
}
```

#### 5.2 Network State Machine

```
  +-----------+     signal degrades     +-----------+
  |  STRONG   | ----------------------> |   WEAK    |
  |  (online) | <---------------------- |  (online) |
  +-----+-----+    signal improves      +-----+-----+
        |                                      |
        | connection lost         connection lost
        v                                      v
  +----------------------------------------------------+
  |                    OFFLINE                           |
  |  - Show offline banner                              |
  |  - Mutations route to offline queue                 |
  |  - Serve all data from cache                        |
  |  - Disable auto-prefetch                            |
  |  - Periodic health check continues (30s interval)   |
  +----------------------------------------------------+
        |                                      |
        | connection restored    connection restored
        v                                      v
  +-----------+                          +-----------+
  |  STRONG   |                          |   WEAK    |
  +-----------+                          +-----------+
```

##### 5.2.1 Transition: Online -> Offline

1. Show non-blocking banner: "You're offline -- showing cached data"
2. Cancel all in-flight queries (they will fail anyway).
3. TanStack Query automatically pauses pending mutations (`networkMode: 'offlineFirst'`).
4. Disable background prefetch timers.
5. Start health check polling (every 30 seconds).

##### 5.2.2 Transition: Offline -> Online

1. Health check confirms API reachability (do not trust NetInfo alone).
2. Hide offline banner.
3. Process offline mutation queue.
4. TanStack Query's `refetchOnReconnect: 'always'` automatically refetches all active queries.
5. Re-enable auto-prefetch.

##### 5.2.3 Transition: Strong -> Weak

1. Reduce image quality in API requests (`image_quality: 'low'`).
2. Increase request timeouts from 15s to 30s.
3. Disable auto-prefetch (save bandwidth).
4. Show subtle network quality indicator (icon change, not a banner).

##### 5.2.4 Transition: Weak -> Strong

1. Restore normal image quality (`image_quality: 'high'`).
2. Restore normal timeouts (15s).
3. Re-enable auto-prefetch.

#### 5.3 Network-Aware UI Component

```typescript
// src/components/NetworkBanner.tsx
export function NetworkBanner() {
  const networkQuality = useOfflineStore((s) => s.networkQuality);
  const pendingCount = useOfflineStore((s) => s.pendingMutationCount);
  const syncStatus = useOfflineStore((s) => s.syncStatus);

  if (networkQuality === 'strong' && syncStatus === 'idle') return null;

  if (networkQuality === 'offline') {
    return (
      <Banner variant="warning" icon="wifi-off">
        You're offline -- showing cached data
        {pendingCount > 0 && ` (${pendingCount} changes pending sync)`}
      </Banner>
    );
  }

  if (syncStatus === 'syncing') {
    return (
      <Banner variant="info" icon="sync" spinning>
        Syncing {pendingCount} pending changes...
      </Banner>
    );
  }

  if (syncStatus === 'error') {
    return (
      <Banner variant="error" icon="alert-circle" onPress={manualSync}>
        Sync failed -- tap to retry
      </Banner>
    );
  }

  return null;
}
```

#### 5.4 Health Check Ping

```typescript
// src/lib/healthCheck.ts
const HEALTH_ENDPOINT = '/api/health';
const HEALTH_CHECK_INTERVAL_MS = 30 * 1000; // 30 seconds when offline
const HEALTH_CHECK_TIMEOUT_MS = 5000;

export async function checkApiHealth(): Promise<boolean> {
  try {
    const response = await fetch(`${API_BASE_URL}${HEALTH_ENDPOINT}`, {
      method: 'HEAD',
      signal: AbortSignal.timeout(HEALTH_CHECK_TIMEOUT_MS),
    });
    return response.ok;
  } catch {
    return false;
  }
}

// Starts polling when offline. Stops when connectivity confirmed.
export function startHealthCheckPolling(): void { /* 30s interval */ }
export function stopHealthCheckPolling(): void { /* clear interval */ }
```

#### 5.5 Special Cases

| Scenario | Behavior |
|---|---|
| **Airplane mode** | Treated as `offline`. NetInfo detects immediately. |
| **VPN changes** | Re-validate via health check. VPN can alter API reachability. |
| **Captive portal** | NetInfo may report connected but not reachable. Health check catches this. |
| **iOS background fetch** | Check connectivity, process mutation queue if online. |

---

### 6. Data Flow Diagrams

#### 6.1 Scenario A: User Opens People Tab (Online)

```
User taps "People" tab
  |
  v
[1] Component mounts, calls useContacts(filters)
  |
  v
[2] TanStack Query checks in-memory cache for ['contacts', 'list', {filters}]
  |
  +--> Cache HIT
  |     [3] Return cached data immediately (0ms latency)
  |     [4] Component renders contact list
  |     [5] Is data stale? (staleTime 5min exceeded?)
  |          +--> YES:
  |          |     [6] Background GET /api/crm/contacts?{filters}
  |          |     [7] Subtle refresh indicator (thin progress bar, NOT skeleton)
  |          |     [8] Response received
  |          |     [9] Cache updated with fresh data
  |          |     [10] SQLite persister writes to query_cache
  |          |     [11] Component re-renders with fresh data (imperceptible)
  |          |     [12] Refresh indicator hidden
  |          +--> NO: Done, no network request
  |
  +--> Cache MISS
        [3] Skeleton loading state
        [4] GET /api/crm/contacts?{filters}
        [5] Response received
        [6] Cache populated
        [7] SQLite persister writes data
        [8] Component renders (skeleton replaced)
```

#### 6.2 Scenario B: User Opens People Tab (Offline)

```
User taps "People" tab
  |
  v
[1] Component mounts, calls useContacts(filters)
  |
  v
[2] TanStack Query checks in-memory cache
  |
  +--> Cache HIT (from previous session, restored by persister)
  |     [3] Return cached data immediately
  |     [4] Component renders contact list
  |     [5] Background refetch attempted
  |     [6] onlineManager reports offline -> query PAUSED (not failed)
  |     [7] NetworkBanner: "You're offline -- showing cached data"
  |     [8] User browses cached data normally
  |     [9] When connectivity returns:
  |         - Paused query resumes automatically
  |         - Banner hides
  |         - Fresh data replaces cached seamlessly
  |
  +--> Cache MISS
        [3] Empty state: "No cached data. Connect to load contacts."
        [4] NetworkBanner: "You're offline"
        [5] When online: query fires automatically
```

#### 6.3 Scenario C: User Edits Contact (Online)

```
User changes phone number, taps "Save"
  |
  v
[1] useEditContact mutation fires
  |
  v
[2] onMutate (optimistic update):
  |   - Cancel in-flight refetches for this contact
  |   - Snapshot current cache value (previousContact)
  |   - Update detail cache: { ...contact, phone: newPhone }
  |   - Update list caches: find contact in all list variants, update phone
  |   - Return { previousContact } for rollback
  |
  v
[3] UI re-renders with new phone (0ms perceived latency)
  |
  v
[4] mutationFn: PATCH /api/contacts/abc { phone: newPhone }
  |
  +--> SUCCESS (200)
  |     [5] onSuccess: invalidate detail + list queries
  |     [6] Background refetch gets canonical server data
  |     [7] SQLite persister updated
  |     Done.
  |
  +--> FAILURE (network/5xx)
        [5] onError: restore previousContact (rollback)
        [6] UI reverts to original phone
        [7] TanStack Query retries (up to 3x, exponential backoff)
        [8] All retries fail: error toast "Failed to save. Try again."
```

#### 6.4 Scenario D: User Edits Contact (Offline)

```
User changes phone number, taps "Save" (offline)
  |
  v
[1] useEditContact mutation fires
  |
  v
[2] onMutate: optimistic update (same as online)
  |
  v
[3] UI shows new phone number immediately
  |
  v
[4] mutationFn attempts PATCH -> onlineManager reports offline -> PAUSED
  |
  v
[5] Offline queue system activates:
  |   - enqueueMutation({ type: 'edit_contact', endpoint, method, body })
  |   - INSERT into mutation_queue (status: 'pending')
  |   - useOfflineStore.pendingMutationCount++
  |
  v
[6] SyncStatusBadge shows "Pending sync" on this contact
  |
  v
[7] User continues offline. Contact shows updated phone everywhere.
  |
  --- CONNECTIVITY RETURNS ---
  |
  v
[8] NetInfo fires, health check confirms API reachable
  |
  v
[9] processQueue() starts
  |
  v
[10] PATCH /api/contacts/abc with Idempotency-Key header
  |
  +--> SUCCESS (200)
  |     [11] mutation_queue status = 'completed'
  |     [12] cacheInvalidation.afterQueueDrain()
  |     [13] Refetch gets canonical server data
  |     [14] SyncStatusBadge -> 'synced' -> green flash -> badge removed
  |
  +--> CONFLICT (409)
  |     [11] Server returns its version (someone else edited this contact)
  |     [12] For 'phone': last_write_wins -> auto-resolve (client wins)
  |     [13] For 'deals' (if changed): user_merge -> show merge UI
  |     [14] Auto-resolved: retry with merged data
  |          User merge needed: badge shows "Conflict -- Tap to resolve"
  |
  +--> PERMANENT FAILURE (404: contact deleted elsewhere)
        [11] moveToDeadLetter()
        [12] Rollback optimistic cache update
        [13] Toast: "Contact deleted by another user. Changes not applied."
        [14] SyncStatusBadge: "Sync failed" with dismiss option
```

#### 6.5 Scenario E: User Sends Chat Message (Offline)

```
User types message, taps Send (offline)
  |
  v
[1] useChatStore.inputDraft consumed and cleared
  |
  v
[2] Message object created locally:
  |   { id: uuid(), role: 'user', content: '...', status: 'queued', createdAt: now }
  |
  v
[3] Optimistic update: message appended to chat history cache
  |
  v
[4] UI renders message with muted styling + clock icon
  |   Label: "Queued -- will send when online"
  |
  v
[5] Warning banner: "AI responses require connectivity"
  |
  v
[6] enqueueMutation({ type: 'send_chat_message', ... })
  |   Persisted to mutation_queue.
  |
  --- CONNECTIVITY RETURNS ---
  |
  v
[7] processQueue() picks up chat mutation
  |
  v
[8] POST /api/chat { sessionId, content }
  |
  +--> SUCCESS
  |     [9] Message status: 'queued' -> 'sent' (styling normalizes)
  |     [10] Invalidate chat history query -> refetch
  |     [11] Refetch returns full history including AI response
  |     [12] AI response appears in chat thread
  |     [13] If user is elsewhere: chat tab badge increments
  |
  +--> FAILURE (session expired, etc.)
        [9] Move to dead letter queue
        [10] Message shows red indicator: "Failed to send -- tap to retry"
```

---

### 7. Real-Time Sync Considerations

#### 7.1 The Two-Frontend Problem

Skye agents may use the web dashboard (Next.js) and the native iOS app simultaneously. Both frontends read from and write to the same Supabase database. Without real-time push synchronization, the two clients can diverge.

**Current state (V2)**: Poll-based synchronization via TanStack Query's `staleTime` and refetch mechanisms. No WebSocket or SSE channel exists.

**Acceptable for V2**: Real estate CRM data is not latency-sensitive at the sub-minute level. A 5-minute delay before seeing another device's edits is acceptable.

#### 7.2 Current Sync Model

```
Native App ----GET /api/contacts (every 5min)----> Supabase
                                                       ^
Web App -------GET /api/contacts (every 5min)----> ---|

Native App ----PATCH /api/contacts/abc-----------> Supabase
  (Web does not see this until its next poll)
```

#### 7.3 Conflict Detection

When both frontends edit the same entity within a short window:

```
t=0s    Contact 'abc' has updated_at = 1000
t=10s   Native PATCH with updated_at = 1000 (matches) -> SUCCESS
        Server sets updated_at = 1010
t=15s   Web PATCH with updated_at = 1000 (stale!) -> 409 CONFLICT
        Server returns { serverVersion, serverUpdatedAt: 1010 }
```

Client-side resolution uses the field-level strategies from Section 4.5.

#### 7.4 Future-Proofing for Real-Time

The state layer is designed to accept real-time updates without architectural changes:

1. **TanStack Query cache as single source of truth**: Any real-time update calls `queryClient.setQueryData()` -- identical to optimistic updates. No component changes needed.

2. **Query key hierarchy supports targeted updates**: A WebSocket event like `{ type: 'contact_updated', id: 'abc', data: {...} }` maps directly to `queryClient.setQueryData(queryKeys.contacts.detail('abc'), data)`.

3. **Invalidation over replacement**: For complex updates, real-time events can invalidate the relevant key, triggering a refetch. Simpler and safer than merging partial updates.

```typescript
// src/lib/realtimeHandler.ts (future -- not implemented in V2)
export function handleRealtimeEvent(event: RealtimeEvent): void {
  switch (event.type) {
    case 'contact_updated':
      queryClient.setQueryData(queryKeys.contacts.detail(event.entityId), event.data);
      queryClient.invalidateQueries({ queryKey: queryKeys.contacts.lists() });
      break;
    case 'contact_deleted':
      queryClient.removeQueries({ queryKey: queryKeys.contacts.detail(event.entityId) });
      queryClient.invalidateQueries({ queryKey: queryKeys.contacts.lists() });
      break;
    case 'chat_message':
      queryClient.setQueryData(queryKeys.chat.history(event.sessionId), (old) => {
        if (!old) return old;
        return { ...old, messages: [...old.messages, event.message] };
      });
      break;
  }
}
```

4. **Zustand stores are event-driven**: `set()` works identically whether called from user action, query callback, or WebSocket handler.

5. **Offline queue coexists**: When real-time is added, the queue remains the fallback on disconnect.

#### 7.5 Sync Latency Budget (V2)

| Data Type | Max Staleness (Same Device) | Max Staleness (Cross-Frontend) |
|---|---|---|
| Contacts | 5 min (staleTime) | 5 min (next poll by other client) |
| Properties | 15 min | 15 min |
| Chat messages | 1 min | 1 min |
| Tasks | 3 min | 3 min |
| After push notification | 0 (immediate invalidation) | N/A (push to native only) |
| After offline queue drain | 0 (immediate on native) | Up to staleTime on web |

#### 7.6 Dependency Summary

| Package | Version | Purpose |
|---|---|---|
| `@tanstack/react-query` | `^5.x` | Server state, caching, background refetch |
| `@tanstack/query-persist-client-core` | `^5.x` | Per-query SQLite persistence |
| `zustand` | `^4.x` | Client/UI state management |
| `react-native-mmkv` | `^3.x` | Fast synchronous key-value storage |
| `expo-secure-store` | `~13.x` | iOS Keychain-backed secure storage |
| `expo-sqlite` | `~14.x` | SQLite database for offline cache and mutation queue |
| `@react-native-community/netinfo` | `^11.x` | Network connectivity detection |
| `expo-file-system` | `~17.x` | Local file operations for image cache |
| `uuid` | `^10.x` | UUID generation for mutation queue IDs |
| `react-native-get-random-values` | `^1.x` | Polyfill for `crypto.getRandomValues()` |

---

### Appendix A: Complete App Provider Tree

```tsx
// src/app/_layout.tsx
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/lib/queryClient';
import { setupOnlineManager } from '@/lib/onlineManager';
import { setupNetworkManager } from '@/lib/networkManager';
import { initDatabase } from '@/lib/database';
import { NetworkBanner } from '@/components/NetworkBanner';
import { useQueueTriggers } from '@/hooks/useQueueTriggers';
import { useAppForegroundRefresh } from '@/hooks/useAppForegroundRefresh';

setupOnlineManager();
setupNetworkManager();
initDatabase();

export default function RootLayout() {
  useQueueTriggers();
  useAppForegroundRefresh();

  return (
    <QueryClientProvider client={queryClient}>
      <NetworkBanner />
      <Slot />
    </QueryClientProvider>
  );
}
```

### Appendix B: Testing Strategy for Offline Flows

| Test Scenario | Method | Validation |
|---|---|---|
| Cache-first rendering | Unit: mock QueryClient with pre-populated cache | Component renders without fetch |
| Stale data refetch | Unit: staleTime=0, observe fetch | Background fetch fires, UI updates |
| Offline mutation queue persistence | Integration: enqueue, kill app, restart, check SQLite | Mutation exists after restart |
| Queue processing order | Integration: enqueue create + edit for same entity | Edit uses server ID from create |
| Conflict resolution (auto) | Integration: simulate 409 for scalar field | Mutation retried with client value |
| Conflict resolution (manual) | E2E: simulate 409 for complex field | Merge UI appears |
| Dead letter handling | Integration: simulate 404 | Mutation in dead_letter_queue, error badge |
| Online -> offline transition | E2E: disable network in simulator | Banner, queue mode, cache serves |
| Offline -> online transition | E2E: re-enable network | Banner hides, queue processes, refresh |
| Cache eviction | Integration: fill >50MB | LRU eviction, protected entities preserved |
| Cold start with persisted cache | E2E: kill app, relaunch offline | Cached data visible immediately |

---

This completes the V2 Offline Architecture, State Management, and Data Sync specification for Skye. Every configuration value, data flow, state transition, and architectural decision has been made explicit with implementation-ready code.

**Key research sources referenced**:
- [TanStack Query v5 persistQueryClient docs](https://tanstack.com/query/v5/docs/react/plugins/persistQueryClient)
- [TanStack Query v5 experimental_createQueryPersister](https://tanstack.com/query/v5/docs/react/plugins/createPersister)
- [TanStack Query React Native guide](https://tanstack.com/query/v5/docs/framework/react/react-native)
- [react-native-mmkv Zustand persist middleware docs](https://github.com/mrousavy/react-native-mmkv/blob/main/docs/WRAPPER_ZUSTAND_PERSIST_MIDDLEWARE.md)
- [Expo SQLite documentation](https://docs.expo.dev/versions/latest/sdk/sqlite/)
- [Expo local-first architecture guide](https://docs.expo.dev/guides/local-first/)
- [Expo SecureStore documentation](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [React Native offline-first with TanStack Query](https://dev.to/fedorish/react-native-offline-first-with-tanstack-query-1pe5)
- [Building offline-first React Native apps with React Query](https://www.whitespectre.com/ideas/how-to-build-offline-first-react-native-apps-with-react-query-and-typescript/)
- [React Native offline sync with SQLite queue](https://dev.to/sathish_daggula/react-native-offline-sync-with-sqlite-queue-4975)

---


## Section 9: Camera, OCR, Media & Studio

---

### 1. Camera System

#### 1.1 Library Selection: `react-native-vision-camera` (Recommended)

**Decision: Use `react-native-vision-camera` v4+.**

| Criterion | `react-native-vision-camera` | `expo-camera` |
|---|---|---|
| Frame Processors (C++/native thread) | Yes | No |
| Manual controls (ISO, shutter, WB, focus) | Full | Limited |
| Real-time card-edge detection potential | Yes (via Frame Processors) | No |
| Capture latency | Low (GPU-backed, no JS bridge hop) | Higher (actions cross JS bridge) |
| Fabric / New Architecture | Supported | Supported |
| Expo Go support | No (requires dev client build) | Yes |
| Setup complexity | Moderate (native config required) | Minimal |
| Maintenance | Actively maintained by mrousavy; frequent releases | Maintained within Expo SDK; 3 releases/year |

**Justification:** Skye's business-card-scanning feature is a core differentiator. The camera is not a casual selfie feature — it must feel instantaneous and professional. `react-native-vision-camera` provides:
- **Frame Processors** for future auto-edge-detection of business cards (detect card boundaries in real time, trigger auto-capture).
- **Lower capture latency** because the camera pipeline never touches the JS thread.
- **Full sensor control** for challenging lighting conditions (dim conference halls, fluorescent-lit offices).
- Since Skye already requires a custom dev client build (not Expo Go), the "works in Expo Go" advantage of `expo-camera` is irrelevant.

**Migration note:** If the team is already using `expo-camera` in V1, migration involves replacing `<Camera>` JSX, updating permission hooks, and adjusting capture calls. The API surface is similar enough that migration is a single-sprint task.

#### 1.2 Permission Flow

All permission logic is encapsulated in a `useCameraPermission()` hook (provided by `react-native-vision-camera`) plus a custom wrapper hook `useCameraAccess()` that manages the full flow.

##### 1.2.1 Permission States

| State | Detection | UX |
|---|---|---|
| `not-determined` | First launch, permission never requested | Show Pre-Permission Screen |
| `granted` | User previously approved | Open camera immediately |
| `denied` | User previously tapped "Don't Allow" | Show Denied Explanation Screen |
| `restricted` | MDM / parental controls / enterprise policy | Show Restricted Screen |

##### 1.2.2 Pre-Permission Screen (state: `not-determined`)

```
+-----------------------------------------------+
|                                                 |
|        [Business Card Illustration]             |
|        (Lottie animation: card scanning)        |
|                                                 |
|     "Scan Business Cards Instantly"             |
|                                                 |
|  Skye uses your camera to read business         |
|  cards and create contacts in seconds.          |
|  No photos are saved to your device.            |
|                                                 |
|     [ Enable Camera Access ]  (primary btn)     |
|     [ Not Now ]               (text link)       |
|                                                 |
+-----------------------------------------------+
```

- **Illustration:** Lottie animation showing a business card being scanned with a subtle sweep effect, 3 seconds looped, 120x120pt.
- **Title:** "Scan Business Cards Instantly" — SF Pro Display Semibold, 22pt, `colors.text.primary`.
- **Body:** SF Pro Text Regular, 15pt, `colors.text.secondary`. Two sentences maximum. The privacy line ("No photos are saved") is intentional — agents handle sensitive contact info.
- **"Enable Camera Access" button:** Full-width, 50pt height, `colors.brand.primary` background, white text, 12pt corner radius. On tap: calls `Camera.requestCameraPermission()` which triggers the iOS system alert.
- **"Not Now" link:** Below the button, `colors.text.tertiary`, 14pt. Dismisses the screen and returns to the previous context. Does NOT trigger the system prompt.
- **Transition:** If permission granted after system prompt, immediately animate-transition to camera viewfinder (shared element transition on the card illustration if feasible, otherwise a 250ms crossfade).

##### 1.2.3 Permission Granted Flow

```
User taps "Scan Card" anywhere in app
  → Check permission status
  → Status is "granted"
  → Present camera viewfinder (modal, full-screen, 300ms slide-up)
```

No intermediate screen. Camera opens immediately. Time from tap to live viewfinder must be under 500ms (measured).

##### 1.2.4 Permission Denied Screen (state: `denied`)

```
+-----------------------------------------------+
|                                                 |
|        [Camera icon, crossed out]               |
|        (SF Symbol: camera.slash)                |
|                                                 |
|     "Camera Access Required"                    |
|                                                 |
|  To scan business cards, Skye needs access      |
|  to your camera. You can enable this in         |
|  your device settings.                          |
|                                                 |
|     [ Open Settings ]         (primary btn)     |
|     [ Enter Manually ]        (secondary btn)   |
|                                                 |
+-----------------------------------------------+
```

- **"Open Settings" button:** Calls `Linking.openSettings()` which deep-links directly to Skye's entry in iOS Settings. Full-width primary button.
- **"Enter Manually" button:** Secondary style (outlined), navigates to blank contact creation form so the user is never blocked from adding a contact.
- **Return handling:** When the user returns from Settings, the app re-checks permission status via `AppState` change listener. If now granted, automatically transition to camera viewfinder. If still denied, remain on this screen.

##### 1.2.5 Permission Restricted Screen (state: `restricted`)

```
+-----------------------------------------------+
|                                                 |
|        [Lock icon]                              |
|        (SF Symbol: lock.shield)                 |
|                                                 |
|     "Camera Access Restricted"                  |
|                                                 |
|  Camera access is restricted on this device.    |
|  This is usually due to parental controls or    |
|  a device management profile.                   |
|                                                 |
|     [ Enter Contact Manually ]  (primary btn)   |
|                                                 |
+-----------------------------------------------+
```

- **No "Open Settings" button.** Restricted state cannot be changed by the user in Settings; showing the button would be misleading.
- **Single action:** "Enter Contact Manually" — navigates to blank contact creation form.

##### 1.2.6 Permission Hook API

```typescript
// hooks/useCameraAccess.ts

type CameraAccessState =
  | 'loading'
  | 'not-determined'
  | 'granted'
  | 'denied'
  | 'restricted';

interface UseCameraAccessReturn {
  status: CameraAccessState;
  requestPermission: () => Promise<boolean>;
  openSettings: () => Promise<void>;
}

function useCameraAccess(): UseCameraAccessReturn;
```

- On mount: reads `Camera.getCameraPermissionStatus()` (synchronous).
- `requestPermission()`: calls `Camera.requestCameraPermission()`, updates state.
- `openSettings()`: calls `Linking.openSettings()`.
- Listens to `AppState` changes to re-check permission when app returns from background.

#### 1.3 Camera UI

The camera screen is presented as a **full-screen modal** (no navigation bar, no status bar — `StatusBar` hidden while camera is active).

##### 1.3.1 Viewfinder Layout

```
+-----------------------------------------------+
| [X Close]                    [Flash: Auto ⚡]  |
|                                                 |
|                                                 |
|    +-----------------------------------+        |
|    |                                   |        |
|    |    (Live camera preview)          |        |
|    |    Business card frame overlay    |        |
|    |    Aspect ratio 3.5:2             |        |
|    |                                   |        |
|    +-----------------------------------+        |
|                                                 |
|    "Position business card within frame"        |
|                                                 |
|                                                 |
|              [ ◯ Capture ]                      |
|                                                 |
|         [ Choose from Library ]                 |
+-----------------------------------------------+
```

##### 1.3.2 Component Specifications

**Camera Preview:**
- Full-screen `<Camera>` component, `device={backCamera}`, `isActive={true}`.
- `photo={true}` to enable photo capture.
- `outputOrientation="device"` to ensure correct EXIF orientation regardless of device rotation.
- `zoom`: default 1x, pinch-to-zoom enabled (min 1x, max 3x), smooth interpolation.

**Overlay Frame (Business Card Guide):**
- Semi-transparent black overlay (`rgba(0, 0, 0, 0.55)`) covering the entire screen EXCEPT the card cutout region.
- Card cutout: centered horizontally, positioned at 35% from top of screen.
- Aspect ratio: **3.5:2** (standard US business card, 3.5 inches x 2 inches).
- Width: 85% of screen width. Height: calculated from aspect ratio.
- Corner brackets: white, 2pt stroke, 20pt length on each corner edge, 8pt corner radius. Animated subtle pulse (opacity 0.7 to 1.0, 2s cycle, `Animated.loop`).
- When card is detected (future auto-detect): corner brackets turn `colors.brand.primary` (green/blue) and guidance text changes.

**Guidance Text:**
- "Position business card within the frame"
- SF Pro Text Regular, 14pt, white, `textShadowColor: rgba(0,0,0,0.5)`, `textShadowRadius: 4`.
- Centered horizontally, positioned 16pt below the card frame overlay.
- **Auto-capture hint (future enhancement):** When frame processor detects a card within the guide region, text changes to "Hold steady..." with a subtle 1.5s progress ring around the capture button. If the card remains stable for 1.5 seconds, auto-capture fires. This feature is gated behind a feature flag (`FEATURE_AUTO_CAPTURE`) and is OFF in initial V2 release.

**Capture Button:**
- Position: center-bottom, 40pt from bottom safe area.
- Outer ring: 70pt diameter, white, 4pt stroke, no fill (transparent center).
- Inner circle: 58pt diameter, white fill.
- **Press state:** On touch-down, inner circle scales to 0.9x (100ms, `Easing.out`). On touch-up, scales back to 1.0x (100ms).
- **Haptic feedback:** `Haptics.impactAsync(ImpactFeedbackStyle.Medium)` on capture (using `expo-haptics`).
- **Capture animation:** On capture, the entire screen flashes white (opacity 0 to 0.4 to 0, 200ms total) to simulate a shutter effect. The overlay briefly freezes the last frame.
- **Disabled state:** While a capture is in progress (typically <100ms), button is disabled and inner circle shows a brief loading spinner. Prevents double-tap captures.

**Flash Toggle:**
- Position: top-right, 16pt from top safe area, 16pt from right edge.
- States: `auto` (default), `on`, `off`.
- Icons: SF Symbols — `bolt.badge.a` (auto), `bolt.fill` (on), `bolt.slash` (off).
- Size: 28pt icon within 44pt tap target (minimum iOS tap target).
- Color: white with text shadow.
- Tap cycles through: auto → on → off → auto.
- Current state label: small text below icon ("Auto", "On", "Off"), 10pt, white, fades in/out on change.
- Persisted: flash preference saved in `AsyncStorage` (`camera.flash_mode`), restored on next camera open.

**Close Button:**
- Position: top-left, 16pt from top safe area, 16pt from left edge.
- Icon: SF Symbol `xmark`, 18pt, white, within a 36pt circular background (`rgba(0, 0, 0, 0.3)`).
- Tap target: 44pt.
- Action: dismisses camera modal with 300ms slide-down animation.
- **Confirmation:** No confirmation dialog needed — no data is lost by closing the camera.

**"Choose from Library" Link:**
- Position: centered, 16pt below the capture button.
- Text: "Choose from Library", SF Pro Text Regular, 14pt, white, underlined.
- Tap target: full text width + 16pt horizontal padding, 44pt height.
- Action: dismisses camera, opens `expo-image-picker` (see Section 3).

#### 1.4 Photo Capture Pipeline

```
User taps Capture
  → Haptic fires (immediate, before capture completes)
  → camera.current.takePhoto({
      flash: currentFlashMode,
      enableShutterSound: false,  // Custom shutter animation instead
      qualityPrioritization: 'quality',
    })
  → Photo returned as { path: string, width: number, height: number, ... }
  → Shutter flash animation plays
  → Resize image (see 1.4.1)
  → Navigate to Preview screen (see 2.1)
  → Camera remains mounted but isActive={false} (instant re-activation on "Retake")
```

##### 1.4.1 Image Processing (Pre-Upload)

All processing occurs in-memory. No files are written to the camera roll or persistent disk.

| Step | Detail |
|---|---|
| **1. Read captured file** | `takePhoto()` returns a temp file path in the app's cache directory. |
| **2. Correct orientation** | VisionCamera writes correct EXIF orientation flags automatically when `outputOrientation="device"` is set. However, some server-side OCR engines ignore EXIF. Therefore, after capture, use `react-native-image-resizer` (or `expo-image-manipulator`) to physically rotate pixels to match EXIF orientation and strip the EXIF orientation tag. This guarantees the server receives a correctly-oriented image regardless of EXIF support. |
| **3. Resize** | Resize so the longest edge is **1920px**. If the captured image's longest edge is already <=1920px, skip resize. Use Lanczos resampling for quality. |
| **4. Compress** | Re-encode as **JPEG at 85% quality**. This balances OCR accuracy (needs sharp text) against upload size (typically 300-600KB after resize). |
| **5. Validate** | Assert file size < 5MB. If somehow larger, re-compress at 75% quality. If still larger, re-compress at 65%. Log a warning if this fallback is needed. |
| **6. Generate metadata** | Create an object: `{ uri: string, width: number, height: number, fileSize: number, mimeType: 'image/jpeg' }`. |

**Temporary file lifecycle:**
- The processed JPEG exists only in the app's temporary cache directory (`NSTemporaryDirectory()` on iOS).
- It is explicitly deleted (`RNFS.unlink()` or `FileSystem.deleteAsync()`) after OCR upload completes (success or final failure).
- It is also deleted if the user navigates away from the OCR flow (retake, close, back).
- **Never** call `CameraRoll.save()` or `MediaLibrary.saveToLibraryAsync()`. The image must never reach the user's photo library (privacy compliance for business contacts).

##### 1.4.2 Capture Error Handling

| Error | Detection | UX |
|---|---|---|
| Camera hardware failure | `takePhoto()` throws | Toast: "Couldn't take photo. Please try again." Camera remains open for retry. |
| Out of disk space | `takePhoto()` throws with storage error | Alert: "Not enough storage space. Free up space and try again." Dismiss camera. |
| Processing failure (resize/compress) | Image manipulation throws | Toast: "Photo processing failed. Please try again." Return to viewfinder. |

---

### 2. OCR Pipeline

#### 2.1 Complete Flow Diagram

```
[Camera Viewfinder]
        │
        ▼ (capture)
[Preview Screen] ──── "Retake" ────→ [Camera Viewfinder]
        │
        ▼ ("Use Photo")
[Upload + Processing] ──── timeout/error ────→ [Error State]
        │                                            │
        ▼ (success)                                  ▼
[Review Screen] ──── "Try Again" ────→ [Camera Viewfinder]
        │           "Enter Manually" → [Blank Contact Form]
        │
        ▼ ("Looks Good")
[Creating Contact...] ──── error ────→ [Creation Error]
        │
        ▼ (success)
[Success Screen] ──── "View Contact" ────→ [Contact Detail]
                 ──── "Scan Another" ────→ [Camera Viewfinder]
                 ──── auto-dismiss (3s) ──→ [Previous Screen]
```

#### 2.2 Preview Screen

Presented immediately after photo capture, before any network request.

```
+-----------------------------------------------+
| [← Back]              Preview                   |
|                                                 |
| +-------------------------------------------+  |
| |                                           |  |
| |                                           |  |
| |      [Captured business card image]       |  |
| |      (full-width, aspect-fit)             |  |
| |                                           |  |
| |                                           |  |
| +-------------------------------------------+  |
|                                                 |
|                                                 |
|   [ Retake ]              [ Use Photo ]         |
|   (secondary)             (primary)             |
+-----------------------------------------------+
```

**Image display:**
- Full width of screen minus 24pt horizontal padding.
- Aspect-fit within a container with max-height of 60% of screen.
- Subtle rounded corners (8pt) and 1pt border in `colors.border.secondary`.
- **Perspective correction (enhancement):** If `react-native-vision-camera` Frame Processors are available, apply lightweight perspective correction using corner detection. This is a V2.1 enhancement and is NOT required for V2 launch. For V2, the image is shown as-captured.

**"Retake" button:**
- Left-aligned in a bottom button bar.
- Secondary style: outlined, `colors.text.primary` text, `colors.border.primary` border.
- Action: navigates back to camera viewfinder. Camera re-activates (`isActive={true}`). The temp file from the previous capture is deleted.

**"Use Photo" button:**
- Right-aligned in the bottom button bar.
- Primary style: `colors.brand.primary` background, white text.
- Action: initiates upload to OCR endpoint. Transitions to Upload/Processing state.

**"Back" button (top-left):**
- Navigates back to camera viewfinder (same as "Retake").
- If the user has not yet tapped "Use Photo", no confirmation needed.

#### 2.3 Upload + Processing State

This is a **single screen** with two visual phases: uploading and processing.

##### 2.3.1 Upload Phase

```
+-----------------------------------------------+
|                                                 |
|                                                 |
|        [Small card image thumbnail]             |
|        (80x50pt, rounded corners)               |
|                                                 |
|        "Uploading..."                           |
|                                                 |
|   [████████░░░░░░░░░░░░]  45%                  |
|                                                 |
|        [ Cancel ]                               |
|                                                 |
+-----------------------------------------------+
```

**Upload request:**
```typescript
// services/ocr.ts

interface OCRUploadRequest {
  image: {
    uri: string;
    type: 'image/jpeg';
    name: 'business_card.jpg';
  };
}

interface OCRUploadResponse {
  success: boolean;
  data: {
    name?: string;
    first_name?: string;
    last_name?: string;
    title?: string;
    company?: string;
    phones?: Array<{ type: string; number: string; confidence: number }>;
    emails?: Array<{ address: string; confidence: number }>;
    address?: {
      street?: string;
      city?: string;
      state?: string;
      zip?: string;
      confidence: number;
    };
    website?: string;
    raw_text?: string;  // Full OCR text for debugging
    confidence_overall: number;  // 0.0 to 1.0
  };
  error?: {
    code: string;
    message: string;
  };
}
```

**Upload mechanics:**
- Method: `POST` to `/api/contacts/ocr`.
- Content-Type: `multipart/form-data`.
- Body: single field `image` containing the JPEG file.
- Authorization: Bearer token in `Authorization` header (same as all API calls).
- **Progress tracking:** Use `XMLHttpRequest` with `upload.onprogress` event to get real-time byte-level progress. Do NOT use `fetch()` (no upload progress support). Alternatively, use `axios` with `onUploadProgress` callback.
- **Progress bar:** Horizontal bar, full width minus 48pt padding. Height 4pt, rounded ends. Background: `colors.bg.tertiary`. Fill: `colors.brand.primary`. Percentage text: right-aligned, 13pt, `colors.text.secondary`.
- **Cancel button:** Text button, `colors.text.secondary`, centered below progress bar. Calls `xhr.abort()` or `cancelToken.cancel()`. On cancel: delete temp file, return to Preview screen. No confirmation dialog.

##### 2.3.2 Processing Phase

Begins when upload completes (progress reaches 100%) and server is performing OCR.

```
+-----------------------------------------------+
|                                                 |
|                                                 |
|        [Skye logo animation]                    |
|        (Lottie, 80x80pt, pulsing/thinking)      |
|                                                 |
|        "Reading business card..."               |
|                                                 |
|        [Subtle shimmer animation]               |
|                                                 |
+-----------------------------------------------+
```

- **Skye logo animation:** Lottie animation of the Skye logo with a subtle thinking/processing motion (e.g., gentle orbit, pulse, or sparkle effect). 80x80pt, looping. Asset: `lottie/skye-thinking.json`.
- **Text:** "Reading business card..." — SF Pro Text Regular, 16pt, `colors.text.secondary`. Ellipsis animates (dots appear one at a time, 500ms interval, looping).
- **No cancel button during processing** — the upload is already complete; we are waiting for the server response. The user can still press the hardware back gesture, which would abandon the result and return to camera.

##### 2.3.3 Timeout Handling

| Condition | Threshold | UX |
|---|---|---|
| Upload taking too long | 60 seconds total upload time | Show "Upload is taking longer than expected. Check your connection." with "Cancel" and "Keep Waiting" buttons. |
| Server processing timeout | 30 seconds after upload completes with no response | Show "This is taking longer than expected" screen (see below). |
| Overall timeout | 90 seconds from "Use Photo" tap to response | Force timeout, show error state. |

**Processing Timeout Screen:**

```
+-----------------------------------------------+
|                                                 |
|        [Hourglass icon]                         |
|                                                 |
|     "This is taking too long"                   |
|                                                 |
|  The server is taking longer than expected       |
|  to read this card.                             |
|                                                 |
|     [ Try Again ]            (primary)          |
|     [ Enter Manually ]       (secondary)        |
|                                                 |
+-----------------------------------------------+
```

- "Try Again": returns to camera viewfinder.
- "Enter Manually": navigates to blank contact creation form.

#### 2.4 Review Screen (Critical UX Moment)

This is the most important screen in the OCR flow. The user must be able to quickly verify and correct OCR results.

```
+-----------------------------------------------+
| [← Back]          Review Contact        [card] |
|                                                 |
| +-------------------------------------------+  |
| | [Thumbnail of business card]   40x25pt    |  |
| | Tap to enlarge                             |  |
| +-------------------------------------------+  |
|                                                 |
| First Name *                                    |
| [ John                                    ] ✓   |
|                                                 |
| Last Name *                                     |
| [ Smith                                   ] ✓   |
|                                                 |
| Title                                           |
| [ Senior VP, Commercial Lending           ] ✓   |
|                                                 |
| Company                                         |
| [ Westfield Realty Group                  ] ✓   |
|                                                 |
| Phone                                           |
| [ (555) 123-4567                          ] ⚠   |
| [ + Add phone number ]                          |
|                                                 |
| Email *                                         |
| [ john.smlth@westfield.com               ] ⚠   |
| [ + Add email ]                                 |
|                                                 |
| Address                                         |
| [ 123 Main St, Suite 400                 ]      |
| [ Anytown, CA 90210                      ]      |
|                                                 |
| Website                                         |
| [ www.westfieldrealty.com                 ] ✓   |
|                                                 |
|                                                 |
| [ Enter Manually ]  [ Try Again ]  [Looks Good] |
+-----------------------------------------------+
```

##### 2.4.1 Field Layout and Behavior

**Field structure:**
Each field is an editable `TextInput` with:
- Label above (SF Pro Text Medium, 13pt, `colors.text.secondary`, uppercase).
- Input field (SF Pro Text Regular, 16pt, `colors.text.primary`, standard text input styling).
- Confidence indicator on the right side of the input.

**Confidence indicators:**

| Confidence Level | Threshold | Visual | Behavior |
|---|---|---|---|
| High | >= 0.85 | Small green checkmark icon (SF Symbol: `checkmark.circle.fill`, 16pt, `colors.status.success`) | Field appears normal. |
| Medium | 0.60 - 0.84 | Yellow warning icon (SF Symbol: `exclamationmark.triangle.fill`, 16pt, `colors.status.warning`) + yellow left border (3pt) on the input | "Please verify" hint text below input in yellow, 12pt. Field is subtly highlighted to draw attention. |
| Low | < 0.60 | Red warning icon (SF Symbol: `exclamationmark.circle.fill`, 16pt, `colors.status.error`) + red left border (3pt) | "Low confidence — please check" hint text below input in red, 12pt. Field has red-tinted background (`rgba(255, 0, 0, 0.03)`). |

**Field ordering:** Fields are displayed in this fixed order: First Name, Last Name, Title, Company, Phone(s), Email(s), Address (street, city/state/zip on separate lines), Website. Empty fields that OCR did not detect are still shown (empty input) so the user can fill them in.

**Phone and Email arrays:**
- OCR may return multiple phones and emails.
- Each is displayed as a separate input row.
- "Remove" button (red minus icon, `SF Symbol: minus.circle.fill`) to the left of each row (except if only one remains).
- "+ Add phone number" / "+ Add email" link below the last entry. Tapping adds a blank input row with focus.
- Phone inputs use `keyboardType="phone-pad"`.
- Email inputs use `keyboardType="email-address"`, `autoCapitalize="none"`.

**Required fields:** First Name and Email are marked with `*` and must be non-empty to enable "Looks Good". If both are empty, "Looks Good" is disabled (grayed out, opacity 0.4) with a subtle hint: "At least a name or email is required."

##### 2.4.2 Business Card Thumbnail

- Position: top of the screen, below the navigation bar.
- Size: full width, max height 80pt, aspect-fit.
- Rounded corners: 6pt. Border: 1pt `colors.border.secondary`.
- **Tap to enlarge:** On tap, opens a full-screen image viewer (modal) with pinch-to-zoom and pan. Dismiss by tapping "X" or swiping down. This allows the user to zoom into the card to verify a hard-to-read phone number or email character.
- **Caption:** "Tap to view original" — 11pt, `colors.text.tertiary`, centered below thumbnail.

##### 2.4.3 Action Buttons

Bottom action bar, pinned above keyboard (if keyboard is visible) or pinned to bottom safe area.

**"Looks Good" button (primary):**
- Right-aligned. Primary style. SF Pro Text Semibold, 16pt.
- Action: Creates the contact via `POST /api/crm/contacts` with the reviewed/edited field data.
- Loading state: button text changes to spinner + "Creating..." while API call is in flight.
- Disabled if: (a) no first name AND no email are filled, or (b) API call is in flight.

**"Try Again" button (secondary):**
- Center position. Outlined style.
- Action: discards OCR results, returns to camera viewfinder. Temp image is deleted.

**"Enter Manually" button (tertiary):**
- Left-aligned. Text-only style, `colors.text.secondary`.
- Action: clears all OCR-populated fields, transitions to the standard blank contact creation form (same form used from the Contacts tab "+ Add Contact" flow). The business card image is discarded.

##### 2.4.4 Keyboard Handling

- Screen content is inside a `KeyboardAvoidingView` (behavior `"padding"` on iOS).
- When a field is focused, the scroll view auto-scrolls to keep the focused field visible above the keyboard.
- "Looks Good" button row floats above the keyboard via `KeyboardAvoidingView`.
- Dismiss keyboard by tapping outside inputs or scrolling.

#### 2.5 Contact Creation (from Review Screen)

```typescript
// Payload sent to POST /api/crm/contacts

interface CreateContactFromOCR {
  first_name: string;
  last_name?: string;
  title?: string;
  company?: string;
  phones: Array<{ number: string; type: 'mobile' | 'work' | 'home' | 'other' }>;
  emails: Array<{ address: string; type: 'work' | 'personal' | 'other' }>;
  address?: {
    street?: string;
    city?: string;
    state?: string;
    zip?: string;
  };
  website?: string;
  source: 'business_card_scan';  // Always this value for OCR-created contacts
  metadata?: {
    ocr_confidence: number;
    scan_timestamp: string;  // ISO 8601
  };
}
```

- **Source tracking:** `source: 'business_card_scan'` allows analytics to track how many contacts came from scanning vs. manual entry vs. import.
- **Duplicate detection:** The API should return a `409 Conflict` if a contact with the same email or phone already exists. The client handles this in the error states below.

#### 2.6 Success State

```
+-----------------------------------------------+
|                                                 |
|        [Checkmark animation]                    |
|        (Lottie: success-check.json, 80x80pt)    |
|                                                 |
|     "Contact Created!"                          |
|                                                 |
| +-------------------------------------------+  |
| | [JD]  John Smith                           |  |
| |       Senior VP, Westfield Realty Group    |  |
| |       john.smith@westfield.com             |  |
| |       (555) 123-4567                       |  |
| +-------------------------------------------+  |
|                                                 |
|     [ View Contact ]          (primary)         |
|     [ Scan Another Card ]     (secondary)       |
|                                                 |
+-----------------------------------------------+
```

- **Checkmark animation:** Lottie animation, plays once (not looping), 1 second duration. Combined with `Haptics.notificationAsync(NotificationFeedbackType.Success)`.
- **Contact card preview:** Compact card showing avatar (initials-based, see Section 4.2), name, title + company, primary email, primary phone. Same component as used in the contact list. Tappable (same as "View Contact").
- **"View Contact":** Navigates to the full Contact Detail screen for the newly created contact. Dismisses the entire OCR modal stack.
- **"Scan Another Card":** Returns to camera viewfinder. Resets entire OCR state.
- **Auto-dismiss:** If the user takes no action for 5 seconds, gently pulse the "View Contact" button to draw attention. No auto-navigation — the user must explicitly choose.

#### 2.7 Error States

##### 2.7.1 OCR Returns No Data

Condition: API returns `success: true` but all fields are empty or `confidence_overall < 0.15`.

```
+-----------------------------------------------+
|                                                 |
|        [Document icon with question mark]       |
|                                                 |
|     "Couldn't Read This Card"                   |
|                                                 |
|  We weren't able to extract contact details     |
|  from this image. Tips:                         |
|  • Make sure the card is well-lit               |
|  • Avoid glare and shadows                      |
|  • Hold the camera steady                       |
|  • Try a flat surface                           |
|                                                 |
|     [ Try Again ]             (primary)         |
|     [ Enter Manually ]        (secondary)       |
|                                                 |
+-----------------------------------------------+
```

##### 2.7.2 Network Error During Upload

Condition: `XMLHttpRequest` fires `onerror` or `ontimeout`, or `HTTP status 0` (no connection).

```
+-----------------------------------------------+
|                                                 |
|        [Wi-Fi icon with exclamation]            |
|                                                 |
|     "Upload Failed"                             |
|                                                 |
|  Check your internet connection and try again.  |
|                                                 |
|     [ Retry Upload ]         (primary)          |
|     [ Enter Manually ]       (secondary)        |
|                                                 |
+-----------------------------------------------+
```

- **Retry Upload:** Re-attempts the upload with the same processed image (still in temp cache). Does NOT require re-capture.
- **Automatic retry:** Before showing this screen, the system automatically retries once with a 2-second delay. Only if the automatic retry also fails does the error screen appear.
- **Temporary image preservation:** If the user navigates away ("Enter Manually") and then returns, the image is gone. But while on this error screen, the image is preserved for retry.

##### 2.7.3 Server Error (HTTP 500)

```
+-----------------------------------------------+
|                                                 |
|        [Server icon with exclamation]           |
|                                                 |
|     "Something Went Wrong"                      |
|                                                 |
|  Our servers are having trouble processing      |
|  this image. Please try again in a moment.      |
|                                                 |
|     [ Try Again ]             (primary)         |
|     [ Enter Manually ]        (secondary)       |
|                                                 |
+-----------------------------------------------+
```

- "Try Again" returns to camera (re-capture, since server-side issues may be image-specific).

##### 2.7.4 Duplicate Contact Detected (HTTP 409)

```
+-----------------------------------------------+
|                                                 |
|        [Person icon with badge]                 |
|                                                 |
|     "Contact May Already Exist"                 |
|                                                 |
| +-------------------------------------------+  |
| | [JS]  John Smith                           |  |
| |       Westfield Realty Group               |  |
| |       john.smith@westfield.com             |  |
| +-------------------------------------------+  |
|                                                 |
|  A contact with this email or phone already     |
|  exists. What would you like to do?             |
|                                                 |
|     [ View Existing Contact ] (primary)         |
|     [ Create Anyway ]         (secondary)       |
|     [ Merge Contacts ]        (tertiary)        |
|                                                 |
+-----------------------------------------------+
```

- **"View Existing Contact":** navigates to the existing contact's detail screen.
- **"Create Anyway":** sends `POST /api/crm/contacts` with `force: true` to bypass duplicate check.
- **"Merge Contacts":** sends `POST /api/crm/contacts/merge` with existing contact ID + new OCR data. Server merges fields intelligently (new data fills in blanks, does not overwrite existing).

---

### 3. Photo Library Access

#### 3.1 Entry Point

The "Choose from Library" link on the Camera Viewfinder screen (Section 1.3.1) triggers the photo library flow.

#### 3.2 Library Selection: `expo-image-picker`

Use `expo-image-picker` for photo library access. It wraps Apple's `PHPicker` (iOS 14+), which:
- Runs in a separate process (the user sees all their photos regardless of "limited access" setting).
- Returns only the selected photo(s) to the app.
- Does NOT require `NSPhotoLibraryUsageDescription` permission for picking (the system picker is privacy-preserving by design).
- Provides a native, familiar UI that users already know.

**Note on iOS 17+ Limited Photo Access:** Apple's `PHPicker` intentionally shows all photos to the user even when "limited access" is granted. This is by design: the picker runs in a separate process and only returns selected items. No custom picker is needed. The `NSPhotoLibraryUsageDescription` key is still included in `Info.plist` as a fallback for edge cases, with the message: "Skye uses your photo library to select business card images for scanning."

#### 3.3 Configuration

```typescript
// Invoked when user taps "Choose from Library"

import * as ImagePicker from 'expo-image-picker';

const result = await ImagePicker.launchImageLibraryAsync({
  mediaTypes: ImagePicker.MediaTypeOptions.Images,  // Photos only, no video
  allowsEditing: false,       // No cropping — OCR needs the full card
  quality: 0.85,              // JPEG quality if conversion needed
  exif: true,                 // Include EXIF data for orientation
  base64: false,              // We use URI, not base64 (memory efficiency)
  allowsMultipleSelection: false,  // Single image only
});

if (result.canceled) {
  // User dismissed picker — return to camera or previous screen
  return;
}

const selectedAsset = result.assets[0];
// selectedAsset: { uri, width, height, fileSize, type, exif }
```

#### 3.4 Post-Selection Processing

The selected image enters the same pipeline as a camera-captured image:

```
User selects photo from library
  → Check file size
  → If > 10MB: resize longest edge to 1920px, compress to JPEG 85%
  → If <= 10MB but longest edge > 1920px: resize to 1920px
  → If <= 10MB and <= 1920px: compress to JPEG 85% only
  → Navigate to Preview Screen (Section 2.2)
  → Same flow: Preview → Upload → Processing → Review → Confirm
```

**File size gate:** Large files (>10MB) from modern iPhone cameras (ProRAW, HEIF at full resolution) must be resized before upload. The 10MB threshold is generous; most business card photos will be well under this.

#### 3.5 Edge Cases

| Scenario | Handling |
|---|---|
| User selects a screenshot | Proceed normally — OCR can handle screenshots of digital business cards. |
| User selects a non-card image (landscape, selfie) | Proceed normally — OCR will likely return low/no confidence. The "Couldn't Read This Card" error (2.7.1) handles this gracefully. |
| User selects a video (shouldn't be possible with `MediaTypeOptions.Images`) | Filter is enforced by the picker. If somehow a video URI is returned, show toast: "Please select a photo, not a video." |
| User cancels picker | Return to camera viewfinder. No error, no toast. |
| HEIC/HEIF format | `expo-image-picker` automatically converts to JPEG when `quality` is specified. No extra handling needed. |

---

### 4. Image Handling Across the App

#### 4.1 Library Selection: `expo-image` (Recommended)

**Decision: Use `expo-image`.**

| Criterion | `expo-image` | `react-native-fast-image` |
|---|---|---|
| Active maintenance | Yes (Expo team, regular releases) | No (original repo frozen; `@d11` community fork exists) |
| New Architecture / Fabric | Fully supported | Original does not support; fork does |
| BlurHash / ThumbHash placeholders | Built-in | Not supported |
| Transition animations (flicker-free) | Built-in | Manual implementation required |
| Priority loading | Supported | Supported |
| Disk + memory caching | Yes (configurable) | Yes (via SDWebImage/Glide) |
| Web support | Yes | No |
| Setup complexity | Zero-config in Expo projects | Manual linking, native config |

**Justification:** `expo-image` is the modern standard. `react-native-fast-image` is effectively abandoned (the original repo has hundreds of unresolved issues and no Fabric support). The community fork `@d11/react-native-fast-image` is viable but adds a maintenance dependency on a community project. Since Skye is an Expo-based project, `expo-image` is the natural, zero-friction choice with superior DX, built-in BlurHash support (important for Street View placeholders), and guaranteed compatibility with future Expo SDK updates.

#### 4.2 Contact Avatars

##### 4.2.1 Avatar Generation

**Default: Initials-based avatar.**

```typescript
// utils/avatar.ts

interface AvatarConfig {
  initials: string;        // "JS" for John Smith
  backgroundColor: string; // Derived from name hash
  textColor: string;       // Always white
}

function getAvatarConfig(contact: Contact): AvatarConfig {
  const initials = getInitials(contact.first_name, contact.last_name); // Max 2 chars
  const backgroundColor = getColorFromName(
    `${contact.first_name} ${contact.last_name}`
  );
  return { initials, backgroundColor, textColor: '#FFFFFF' };
}

// Color derivation: deterministic hash of full name → index into color palette
// Palette: 12 distinct, accessible colors that work with white text
const AVATAR_COLORS = [
  '#E53935', '#D81B60', '#8E24AA', '#5E35B1',
  '#3949AB', '#1E88E5', '#00ACC1', '#00897B',
  '#43A047', '#7CB342', '#F4511E', '#6D4C41',
];

function getColorFromName(name: string): string {
  let hash = 0;
  for (let i = 0; i < name.length; i++) {
    hash = name.charCodeAt(i) + ((hash << 5) - hash);
    hash = hash & hash; // Convert to 32-bit integer
  }
  return AVATAR_COLORS[Math.abs(hash) % AVATAR_COLORS.length];
}
```

**Why initials-based:** Real estate agents may have thousands of contacts. Generating or fetching real photos for all would be expensive and slow. Initials provide instant, distinctive, zero-network-cost avatars. The deterministic color hash means "John Smith" is always the same color — visually consistent across the app.

##### 4.2.2 Avatar Sizes

| Context | Size (pt) | Text Size | Font |
|---|---|---|---|
| Contact list row | 40 | 15pt | SF Pro Display Semibold |
| Chat/interaction cards | 60 | 22pt | SF Pro Display Semibold |
| Contact detail header | 80 | 30pt | SF Pro Display Semibold |
| Mini (tags, pills, mentions) | 24 | 10pt | SF Pro Display Semibold |

##### 4.2.3 Avatar Rendering

```typescript
// components/ContactAvatar.tsx

interface ContactAvatarProps {
  contact: Contact;
  size: 24 | 40 | 60 | 80;
  showBorder?: boolean; // default: true in light mode, false in dark mode
}
```

- **Shape:** Circular (`borderRadius: size / 2`).
- **Border:** 1pt solid `colors.border.secondary` in light mode. No border in dark mode (the dark background provides sufficient contrast).
- **Initials text:** Centered vertically and horizontally within the circle.
- **Profile photo override:** If the contact has a `profile_image_url` (e.g., pulled from Google Profile via People API), display the photo instead of initials.
  - Use `<Image>` from `expo-image`.
  - `placeholder`: the initials avatar (rendered as a component, or a blurhash of the avatar color).
  - `transition`: 200ms crossfade from placeholder to loaded image.
  - `contentFit`: `"cover"`.
  - `cachePolicy`: `"disk"` — profile photos rarely change.
- **Error fallback:** If profile photo URL fails to load (404, timeout), fall back to initials avatar. No broken image icon ever shown.

#### 4.3 Street View Images

##### 4.3.1 URL Construction

```typescript
// utils/streetView.ts

interface StreetViewParams {
  latitude: number;
  longitude: number;
  width: number;   // in pixels (retina-adjusted)
  height: number;  // in pixels (retina-adjusted)
  heading?: number; // 0-360, optional (API auto-selects if omitted)
  pitch?: number;   // default 0 (horizontal)
  fov?: number;     // default 90 (field of view)
}

function getStreetViewUrl(params: StreetViewParams): string {
  const {
    latitude,
    longitude,
    width,
    height,
    heading,
    pitch = 0,
    fov = 90,
  } = params;

  const baseUrl = 'https://maps.googleapis.com/maps/api/streetview';
  const queryParams = new URLSearchParams({
    size: `${width}x${height}`,
    location: `${latitude},${longitude}`,
    fov: fov.toString(),
    pitch: pitch.toString(),
    key: Config.GOOGLE_MAPS_API_KEY,
    // signature: generated server-side for production (URL signing)
  });

  if (heading !== undefined) {
    queryParams.set('heading', heading.toString());
  }

  return `${baseUrl}?${queryParams.toString()}`;
}

// Metadata check: determine if Street View is available at coordinates
function getStreetViewMetadataUrl(lat: number, lng: number): string {
  return `https://maps.googleapis.com/maps/api/streetview/metadata?location=${lat},${lng}&key=${Config.GOOGLE_MAPS_API_KEY}`;
}
```

**Retina sizing:** On a 3x retina device, a 120x80pt thumbnail needs a 360x240px image. Always multiply by `PixelRatio.get()` for the API `size` parameter, up to a maximum of `640x640` (API limit).

##### 4.3.2 Display Sizes

| Context | Point Size | Pixel Size (@3x) | API Size |
|---|---|---|---|
| Property list thumbnail | 120 x 80pt | 360 x 240px | `360x240` |
| Property detail hero | Full width x 220pt | ~1170 x 660px | `640x400` (API max width 640, scale up client-side) |
| Map card popup | 160 x 100pt | 480 x 300px | `480x300` |

##### 4.3.3 Availability Check and Fallback

Before displaying a Street View image, check the metadata endpoint.

```typescript
interface StreetViewMetadata {
  status: 'OK' | 'ZERO_RESULTS' | 'NOT_FOUND' | 'OVER_QUERY_LIMIT' | 'REQUEST_DENIED' | 'INVALID_REQUEST' | 'UNKNOWN_ERROR';
  pano_id?: string;
  date?: string;     // "2023-05" format
  location?: { lat: number; lng: number };
}
```

**Metadata caching:** Cache metadata responses in a persistent key-value store keyed by `"sv_meta_{lat}_{lng}"` with a 30-day TTL. Metadata requests are free (no billing) so we can be aggressive with pre-checking.

**Fallback display when Street View is unavailable (`ZERO_RESULTS`):**

```
+-----------------------------------+
|                                    |
|   [House icon, 32pt, gray]        |
|   "No Street View Available"      |
|   (12pt, colors.text.tertiary)    |
|                                    |
+-----------------------------------+
```

- Background: `colors.bg.secondary` (light gray).
- Rounded corners matching the image slot (8pt for thumbnails, 0 for full-width heroes).
- House icon: SF Symbol `house.fill`, 32pt, `colors.icon.secondary`.

##### 4.3.4 Street View Image Component

```typescript
// components/StreetViewImage.tsx

interface StreetViewImageProps {
  latitude: number;
  longitude: number;
  size: 'thumbnail' | 'hero' | 'card';
  style?: ViewStyle;
}
```

- Uses `expo-image` `<Image>` component.
- `placeholder`: blurhash string (pre-computed neutral gray-blue: `"L6Pj0^jE.mfQ~qj[ayj@_3j[D%fQ"`) providing a smooth blur placeholder while loading.
- `transition`: 300ms crossfade.
- `cachePolicy`: `"disk"` — street addresses do not change; aggressive caching is safe.
- `contentFit`: `"cover"` for thumbnails and cards; `"cover"` for heroes.

#### 4.4 Image Caching Strategy

All image caching is handled by `expo-image`'s built-in caching layer (which uses SDWebImage on iOS under the hood). Additional configuration:

##### 4.4.1 Cache Configuration

```typescript
// config/imageCache.ts
import { Image } from 'expo-image';

// Set cache limits at app initialization
Image.setCachePolicy?.({
  diskCacheLimit: 100 * 1024 * 1024,   // 100MB disk cache (LRU eviction)
  memoryCacheLimit: 50 * 1024 * 1024,  // 50MB memory cache (LRU eviction)
});
```

| Cache Layer | Size Limit | Eviction Policy | Scope |
|---|---|---|---|
| **Memory cache** | 50MB | LRU (Least Recently Used) | Current app session. Cleared on memory warning. |
| **Disk cache** | 100MB | LRU | Persistent across sessions. Separate from data/API cache. |

**Cache key:** URL hash (handled automatically by `expo-image`). For Street View URLs, the API key is included in the URL, so the same coordinates always produce the same cache key.

##### 4.4.2 Preloading Strategy

```typescript
// hooks/useImagePreloading.ts

import { Image } from 'expo-image';

/**
 * Preload images for visible items + next N items in a list.
 * Called from FlatList's onViewableItemsChanged callback.
 */
function preloadPropertyImages(
  properties: Property[],
  visibleIndices: number[],
  lookAhead: number = 5
): void {
  const maxIndex = Math.min(
    Math.max(...visibleIndices) + lookAhead,
    properties.length - 1
  );

  const urlsToPreload: string[] = [];
  for (let i = visibleIndices[0]; i <= maxIndex; i++) {
    const property = properties[i];
    if (property.latitude && property.longitude) {
      urlsToPreload.push(
        getStreetViewUrl({
          latitude: property.latitude,
          longitude: property.longitude,
          width: 360,
          height: 240,
        })
      );
    }
  }

  Image.prefetch(urlsToPreload);
}
```

- **Trigger:** `onViewableItemsChanged` in `FlatList` / `FlashList`.
- **Look-ahead:** Preload images for the next 5 items beyond the current visible range.
- **Throttle:** Debounce preload calls by 200ms to avoid thrashing during fast scrolling.
- **Network-aware:** Skip preloading on cellular if the user has enabled "Low Data Mode" (check via `NetInfo.fetch()` and `details.isConnectionExpensive`).

##### 4.4.3 Cache Clearing

- **Manual clear:** Accessible in Settings > Storage > "Clear Image Cache". Calls `Image.clearDiskCache()` and `Image.clearMemoryCache()`.
- **Automatic:** iOS will automatically purge the disk cache directory under storage pressure. The 100MB limit also ensures the cache self-manages via LRU.
- **On logout:** Clear all caches (images + data) to prevent data leakage between accounts.

---

### 5. Studio Screen

#### 5.1 Purpose and Positioning

The Studio is Skye's AI-powered content generation workspace for real estate marketing. It is a dedicated tab in the app's bottom navigation, positioned as a creative tool distinct from the chat-based AI assistant (which handles conversational queries).

**Key distinction from Chat:** Chat is for questions and tasks ("What's the average price per sqft in zip 90210?"). Studio is for content creation ("Write an Instagram caption for my new listing at 123 Oak Lane").

#### 5.2 Screen Layout

```
+-----------------------------------------------+
| Studio                              [History 🕐]|
|                                                 |
| [Instagram] [Email] [Listing] [Market] [Open H] |
|  (selected)                          ← scroll → |
|                                                 |
| ┌─────────────────────────────────────────────┐ |
| │  Property *                                  │ |
| │  [ 123 Oak Lane, Beverly Hills       ▼ ]    │ |
| │                                              │ |
| │  Tone                                        │ |
| │  [Professional] [Casual] [Luxury] [Playful]  │ |
| │   (selected)                                 │ |
| │                                              │ |
| │  Key Features (select up to 5)               │ |
| │  [Pool] [Ocean View] [New Reno] [Open Floor] │ |
| │  [Smart Home] [Gourmet Kitchen] [+ Custom]   │ |
| │                                              │ |
| │  Additional Notes (optional)                 │ |
| │  [ Mention the school district...        ]   │ |
| └─────────────────────────────────────────────┘ |
|                                                 |
|     [ ✨ Generate with Skye ]                    |
|                                                 |
+-----------------------------------------------+
```

#### 5.3 Content Type Selector

**Horizontal scrollable row** of content type cards/chips.

| Content Type | Icon (SF Symbol) | Description |
|---|---|---|
| Instagram Caption | `camera.fill` | Social media post captions |
| Email Template | `envelope.fill` | Client email drafts |
| Listing Description | `doc.text.fill` | MLS / property listing copy |
| Market Update | `chart.line.uptrend.xyaxis` | Market summary posts |
| Open House Invite | `door.left.hand.open` | Event invitation copy |

**Chip design:**
- Height: 36pt. Horizontal padding: 16pt. Corner radius: 18pt (pill shape).
- Unselected: `colors.bg.secondary` background, `colors.text.secondary` text, icon at 16pt.
- Selected: `colors.brand.primary` background, white text, white icon. Subtle scale animation (1.0 to 1.05 to 1.0, 200ms) on selection.
- Scroll: horizontal `ScrollView` with `showsHorizontalScrollIndicator={false}`. Scroll padding: 16pt leading/trailing.
- Haptic: `Haptics.selectionAsync()` on tap.

#### 5.4 Context Inputs (Per Content Type)

Each content type shows different input fields. These are configured via a `contentTypeConfig` map.

##### 5.4.1 Instagram Caption

| Input | Type | Details |
|---|---|---|
| Property | Dropdown (searchable) | Populated from user's property list (`GET /api/crm/properties`). Required. Shows address as label. Auto-fills latitude/longitude for optional image reference. |
| Tone | Single-select chips | Options: Professional, Casual, Luxury, Playful, Witty. Default: Professional. |
| Key Features | Multi-select chips (max 5) | Pre-populated options: Pool, Ocean View, New Renovation, Open Floor Plan, Smart Home, Gourmet Kitchen, Walk-in Closet, Large Backyard, Mountain View, Near Transit. "+ Custom" chip opens a text input to add a custom tag. |
| Additional Notes | Text input (multiline) | Optional. Placeholder: "Any specific details to include..." Max 200 characters. |
| Include Hashtags | Toggle switch | Default: ON. If on, response includes relevant hashtags. |
| Include Emojis | Toggle switch | Default: ON for Instagram. |

##### 5.4.2 Email Template

| Input | Type | Details |
|---|---|---|
| Recipient Type | Single-select chips | Options: Buyer, Seller, Investor, Past Client, General Lead. Required. |
| Purpose | Dropdown | Options: New Listing Announcement, Price Reduction, Open House Invite, Market Update, Check-In / Follow-Up, Thank You, Referral Request. Required. |
| Property (optional) | Dropdown (searchable) | Same as Instagram. Optional — some emails (check-in, thank you) don't need a property. |
| Recipient Name | Text input | Optional. If provided, AI personalizes greeting ("Dear Sarah" vs "Dear Homeowner"). |
| Key Points | Multiline text | Optional. "Any specific points to cover..." Max 300 characters. |
| Tone | Single-select chips | Options: Professional, Warm, Urgent, Casual. Default: Professional. |

##### 5.4.3 Listing Description

| Input | Type | Details |
|---|---|---|
| Property | Dropdown (searchable) | Required. On selection, auto-fills known details (beds, baths, sqft, lot size, year built) from property data. |
| Highlight Features | Multi-select checklist | Checkboxes for: Recently Renovated, Pool/Spa, Views, Smart Home, Energy Efficient, Outdoor Living, Gated Community, Near Schools, Near Shopping, Quiet Neighborhood. |
| Style | Single-select chips | Options: MLS Standard, Narrative / Story-telling, Luxury / Aspirational, Concise / Bullet Points. Default: MLS Standard. |
| Target Buyer | Single-select chips | Options: First-Time Buyer, Family, Investor, Downsizer, Luxury Buyer. |
| Max Length | Segmented control | Options: Short (150 words), Medium (300 words), Long (500 words). Default: Medium. |

##### 5.4.4 Market Update

| Input | Type | Details |
|---|---|---|
| Area | Text input with autocomplete | Zip code, neighborhood, or city. Autocomplete from previously used areas + Google Places API. |
| Timeframe | Segmented control | This Month, This Quarter, This Year. Default: This Month. |
| Focus | Multi-select chips | Options: Median Price, Inventory Levels, Days on Market, Interest Rates, New Construction, Buyer Demand. |
| Tone | Single-select chips | Options: Professional, Conversational, Data-Driven. Default: Professional. |
| Format | Single-select chips | Options: Social Post, Newsletter Blurb, Full Report. Default: Social Post. |

##### 5.4.5 Open House Invite

| Input | Type | Details |
|---|---|---|
| Property | Dropdown (searchable) | Required. Auto-fills address and basic details. |
| Date & Time | Date/time picker | Required. Uses native iOS date picker. |
| RSVP Method | Multi-select chips | Options: Call, Text, Email, Website Link. |
| Special Features | Multiline text | Optional. "Refreshments, live music, tours..." Max 200 characters. |
| Tone | Single-select chips | Options: Professional, Exciting, Exclusive / VIP, Neighborhood-Friendly. Default: Professional. |
| Format | Single-select chips | Options: Email, Social Post, Flyer Text. Default: Email. |

#### 5.5 Generate Button

```
[ ✨ Generate with Skye ]
```

- Full width minus 32pt horizontal padding. Height: 52pt. Corner radius: 12pt.
- Background: `colors.brand.primary`. Text: white, SF Pro Text Semibold, 17pt.
- Sparkle icon (SF Symbol: `sparkles`) at 18pt, left of text, white.
- **Disabled state:** If required fields are empty, button opacity: 0.4, non-interactive. No toast on tap when disabled — the empty required fields show red borders to indicate what's missing.
- **Tap:** `Haptics.impactAsync(ImpactFeedbackStyle.Medium)`. Button text changes to "Generating..." with a subtle shimmer animation. Button is non-interactive during generation.

#### 5.6 Generation Flow

##### 5.6.1 API Request

```typescript
// services/studio.ts

interface GenerateContentRequest {
  content_type: 'instagram_caption' | 'email_template' | 'listing_description' | 'market_update' | 'open_house_invite';
  property_id?: string;
  parameters: Record<string, any>;  // Content-type-specific inputs
  regenerate_from?: string;  // Previous generation ID, if regenerating with tweaks
  tweak?: string;  // "Make it shorter", "More professional", etc.
}

// Endpoint: POST /api/studio/generate
// Response: Server-Sent Events (SSE) stream
```

##### 5.6.2 Loading State

```
+-----------------------------------------------+
| Studio                                          |
|                                                 |
| [Inputs are still visible but dimmed/disabled]  |
|                                                 |
|     ┌───────────────────────────────────────┐   |
|     │                                       │   |
|     │   [Skye thinking animation]           │   |
|     │   (Lottie, 60x60pt, centered)         │   |
|     │                                       │   |
|     │   "Crafting your Instagram caption..." │   |
|     │                                       │   |
|     │   [ Cancel ]                          │   |
|     │                                       │   |
|     └───────────────────────────────────────┘   |
|                                                 |
+-----------------------------------------------+
```

- The input area dims (opacity 0.3) but remains visible.
- The output area appears below with the loading state.
- Loading text is content-type specific: "Crafting your Instagram caption...", "Drafting your email...", "Writing your listing description...", etc.
- **Cancel:** Text button that closes the SSE connection (`eventSource.close()`), removes the loading state, re-enables inputs. No data is lost.

##### 5.6.3 Streaming Output

Content streams in token-by-token via SSE (same pattern as the chat system).

```
+-----------------------------------------------+
| Studio                                          |
|                                                 |
| [Inputs collapsed to single summary line]       |
| "Instagram Caption • 123 Oak Lane • Luxury"     |
| [ Edit Inputs ▼ ]                               |
|                                                 |
| ┌───────────────────────────────────────────┐   |
| │                                           │   |
| │  ✨ Step into paradise at 123 Oak Lane,    │   |
| │  where luxury meets modern living. This    │   |
| │  stunning 4-bed, 3-bath retreat features   │   |
| │  an infinity pool with breathtaking ocean  │   |
| │  views, a chef's gourmet kitchen, and...   │   |
| │  ▊  (cursor, blinking, streaming)         │   |
| │                                           │   |
| └───────────────────────────────────────────┘   |
|                                                 |
+-----------------------------------------------+
```

- **Input collapse:** When generation starts, the input fields collapse into a single summary line showing the key parameters. An "Edit Inputs" toggle expands them back.
- **Streaming text:** Rendered with markdown support (bold, italic, line breaks, bullet points). Uses a lightweight markdown renderer (e.g., `react-native-markdown-display` or a custom component for the subset of markdown used).
- **Cursor:** Blinking vertical bar (`|`) at the end of the streaming text, same as chat. Removed when streaming completes.
- **Auto-scroll:** The output area auto-scrolls to keep the latest text visible during streaming.

##### 5.6.4 Completed Output

```
+-----------------------------------------------+
| Studio                                          |
|                                                 |
| "Instagram Caption • 123 Oak Lane • Luxury"     |
| [ Edit Inputs ▼ ]                               |
|                                                 |
| ┌───────────────────────────────────────────┐   |
| │                                           │   |
| │  ✨ Step into paradise at 123 Oak Lane,    │   |
| │  where luxury meets modern living. This    │   |
| │  stunning 4-bed, 3-bath retreat features   │   |
| │  an infinity pool with breathtaking ocean  │   |
| │  views, a chef's gourmet kitchen, and      │   |
| │  open floor plan perfect for entertaining. │   |
| │                                           │   |
| │  Your dream home awaits. Schedule your     │   |
| │  private tour today! 🏡                    │   |
| │                                           │   |
| │  #BeverlyHills #LuxuryRealEstate          │   |
| │  #OceanView #DreamHome #NewListing        │   |
| │                                           │   |
| └───────────────────────────────────────────┘   |
|                                                 |
| Quick Tweaks:                                   |
| [Shorter] [Longer] [More Professional]          |
| [Add Emojis] [Remove Hashtags] [More Casual]    |
|                                                 |
| [ Copy ]  [ Share ]  [ Regenerate ↻ ]           |
|                                                 |
+-----------------------------------------------+
```

**Output area:**
- Editable: the user can tap into the output text and edit directly (inline editing). The text area becomes a large `TextInput` with `multiline={true}` on tap.
- **Character count:** Shown below the output, right-aligned, `colors.text.tertiary`, 12pt. For Instagram, also shows a visual indicator if exceeding Instagram's 2200-character caption limit (text turns `colors.status.warning` at >2000, `colors.status.error` at >2200).
- **Word count:** Shown next to character count for listing descriptions: "287 words".

**Quick Tweak chips:**
- Horizontal scrollable row of adjustment options.
- Chip design: same as content type selector chips but smaller (height 30pt, 12pt text).
- Available tweaks (universal): "Make it Shorter", "Make it Longer", "More Professional", "More Casual", "Add Emojis", "Remove Emojis".
- Content-type-specific tweaks:
  - Instagram: "Add Hashtags", "Remove Hashtags", "Add Call-to-Action"
  - Email: "More Formal", "Add Urgency", "Softer Tone"
  - Listing: "More Descriptive", "More Concise", "Add Neighborhood Info"
- **Tweak flow:** Tapping a tweak chip sends a new generation request with `regenerate_from` set to the current generation ID and `tweak` set to the chip label. The output area shows the loading state again, then streams the revised content. Previous output is replaced (not appended).

**Action buttons (bottom bar):**

| Button | Style | Action |
|---|---|---|
| **Copy** | Icon + text, secondary style. Icon: `doc.on.doc` (SF Symbol). | Copies the full generated text (plain text, no markdown formatting) to the system clipboard. Shows toast: "Copied to clipboard!" with checkmark. `Haptics.notificationAsync(Success)`. |
| **Share** | Icon + text, secondary style. Icon: `square.and.arrow.up` (SF Symbol). | Opens the iOS system share sheet (`Share.share({ message: generatedText })`). Allows sharing to Messages, Mail, Instagram, Notes, etc. |
| **Regenerate** | Icon + text, secondary style. Icon: `arrow.clockwise` (SF Symbol). | Re-runs the generation with the same inputs (no tweaks). Shows a brief confirmation: "Generate new version?" with "Yes" and "Cancel". This prevents accidental loss of a good generation. |

#### 5.7 History

##### 5.7.1 History Tab

Accessible via the clock icon in the top-right of the Studio screen. Opens a bottom sheet or navigates to a sub-screen.

```
+-----------------------------------------------+
| History                              [ Clear ]  |
|                                                 |
| Today                                           |
| ┌─────────────────────────────────────────────┐|
| │ 📸 Instagram Caption                        │|
| │ 123 Oak Lane • 2:34 PM                      │|
| │ "Step into paradise at 123 Oak Lane..."     │|
| └─────────────────────────────────────────────┘|
|                                                 |
| ┌─────────────────────────────────────────────┐|
| │ ✉️ Email Template                            │|
| │ New Listing Announcement • 1:12 PM          │|
| │ "Dear Sarah, I'm thrilled to share..."     │|
| └─────────────────────────────────────────────┘|
|                                                 |
| Yesterday                                       |
| ┌─────────────────────────────────────────────┐|
| │ 🏠 Listing Description                      │|
| │ 456 Elm Street • 4:45 PM                    │|
| │ "Welcome to this stunning 3-bedroom..."    │|
| └─────────────────────────────────────────────┘|
|                                                 |
+-----------------------------------------------+
```

**History entry:**
- Icon: content type icon (same as selector chips).
- Title: content type name.
- Subtitle: property address (if applicable) + timestamp.
- Preview: first 60 characters of generated content, truncated with "..."
- Tap: opens the full generated content in a read-only view with "Copy", "Share", "Edit & Regenerate" buttons.
- Swipe-to-delete: individual entries can be deleted with a left swipe → red "Delete" button.

##### 5.7.2 History Storage

- **Storage:** Local-only, stored in SQLite (same database as other app data) or `AsyncStorage` (for simplicity in V2).
- **Schema:**

```typescript
interface GenerationHistoryEntry {
  id: string;                 // UUID
  content_type: string;       // 'instagram_caption', etc.
  parameters: Record<string, any>;  // Input parameters used
  generated_content: string;  // Final output text
  property_id?: string;       // Associated property, if any
  property_address?: string;  // Denormalized for display
  created_at: string;         // ISO 8601
  is_edited: boolean;         // True if user edited the output inline
  edited_content?: string;    // User's edited version (if different)
}
```

- **Retention:** Keep last 100 entries. LRU eviction when limit is reached.
- **No server sync:** History is local-only in V2. Server-side history (for cross-device access) is a V3 consideration.

#### 5.8 Offline Behavior

```
+-----------------------------------------------+
|                                                 |
|        [Cloud icon with slash]                  |
|                                                 |
|     "No Internet Connection"                    |
|                                                 |
|  Content generation requires an internet        |
|  connection. Connect to Wi-Fi or cellular       |
|  to generate content.                           |
|                                                 |
|  You can still view your recent generations     |
|  below.                                         |
|                                                 |
| ┌─ Recent Generations ──────────────────────┐   |
| │ [History entries from local cache]        │   |
| └───────────────────────────────────────────┘   |
|                                                 |
+-----------------------------------------------+
```

- **Detection:** Use `@react-native-community/netinfo` to detect connectivity state.
- **Generate button:** Disabled when offline. Tooltip on tap: "Internet connection required."
- **History:** Fully accessible offline (local storage).
- **Reconnection:** When connectivity is restored, the offline banner dismisses automatically (200ms fade-out) and the generate button re-enables.

---

### 6. Media Upload Architecture

#### 6.1 Upload Manager Service

A centralized `UploadManager` service handles all media uploads in the app. In V2, the primary use case is OCR image uploads. The architecture is designed to support future media uploads (property photos, document attachments, etc.) without refactoring.

```typescript
// services/uploadManager.ts

type UploadStatus = 'pending' | 'uploading' | 'processing' | 'completed' | 'failed' | 'cancelled';

interface UploadTask {
  id: string;               // UUID, generated client-side
  uri: string;              // Local file URI
  endpoint: string;         // API endpoint (e.g., '/api/contacts/ocr')
  mimeType: string;         // 'image/jpeg'
  fieldName: string;        // Form field name (e.g., 'image')
  status: UploadStatus;
  progress: number;         // 0.0 to 1.0
  retryCount: number;       // Number of retries attempted
  maxRetries: number;       // Default: 1
  createdAt: Date;
  completedAt?: Date;
  response?: any;           // Server response on completion
  error?: Error;            // Error details on failure
  onProgress?: (progress: number) => void;
  onComplete?: (response: any) => void;
  onError?: (error: Error) => void;
}

class UploadManager {
  private activeTasks: Map<string, UploadTask> = new Map();
  private xhr: Map<string, XMLHttpRequest> = new Map();

  /**
   * Enqueue a new upload.
   * Returns the upload task ID for tracking.
   */
  async upload(config: {
    uri: string;
    endpoint: string;
    mimeType: string;
    fieldName: string;
    additionalFields?: Record<string, string>;
    onProgress?: (progress: number) => void;
    onComplete?: (response: any) => void;
    onError?: (error: Error) => void;
  }): Promise<string>;

  /**
   * Cancel an in-progress upload.
   */
  cancel(taskId: string): void;

  /**
   * Retry a failed upload.
   */
  retry(taskId: string): Promise<void>;

  /**
   * Get the current status of an upload.
   */
  getStatus(taskId: string): UploadTask | undefined;

  /**
   * Cancel all active uploads (e.g., on logout).
   */
  cancelAll(): void;
}

// Singleton instance
export const uploadManager = new UploadManager();
```

#### 6.2 Upload Flow (Detailed)

```
1. COMPRESS
   └─ Image processing (Section 1.4.1): orient, resize, JPEG encode
   └─ Validate output: assert file size < 5MB

2. GENERATE UPLOAD ID
   └─ Client-side UUID v4
   └─ Register task in UploadManager

3. BUILD MULTIPART REQUEST
   └─ Create FormData:
       formData.append(fieldName, {
         uri: processedImageUri,
         type: mimeType,
         name: `upload_${taskId}.jpg`,
       })
   └─ Add any additional fields (e.g., metadata)

4. EXECUTE UPLOAD
   └─ Create XMLHttpRequest (NOT fetch — need upload progress)
   └─ Set headers: Authorization, Accept: application/json
   └─ xhr.upload.onprogress = (event) => {
        task.progress = event.loaded / event.total;
        task.onProgress?.(task.progress);
      }
   └─ xhr.onload = (event) => { ... handle response ... }
   └─ xhr.onerror = (event) => { ... handle error ... }
   └─ xhr.send(formData)

5. TRACK PROGRESS
   └─ Real-time progress via xhr.upload.onprogress
   └─ Update task.progress (0.0 to 1.0)
   └─ Call onProgress callback for UI updates

6. HANDLE COMPLETION
   └─ Success (HTTP 200-201):
       task.status = 'completed'
       task.response = JSON.parse(xhr.responseText)
       task.onComplete?.(task.response)
   └─ Client error (HTTP 400-499):
       task.status = 'failed'
       task.error = new UploadError(status, body)
       task.onError?.(task.error)
       // No auto-retry for client errors
   └─ Server error (HTTP 500+):
       if (task.retryCount < task.maxRetries) {
         task.retryCount++
         // Wait 2 seconds, then retry
         setTimeout(() => this.executeUpload(taskId), 2000)
       } else {
         task.status = 'failed'
         task.onError?.(new Error('Upload failed after retries'))
       }

7. HANDLE FAILURE
   └─ Network error (no response): auto-retry once after 2s
   └─ Timeout (60s): treat as network error
   └─ After all retries exhausted: surface error to UI
```

#### 6.3 Progress Bar Implementation

```typescript
// components/UploadProgressBar.tsx

interface UploadProgressBarProps {
  progress: number;  // 0.0 to 1.0
  status: UploadStatus;
}
```

- **Visual:** Horizontal bar, full width minus horizontal padding (48pt total).
- Height: 4pt. Corner radius: 2pt.
- Background track: `colors.bg.tertiary`.
- Fill: `colors.brand.primary`. Width animated with `Animated.timing` (or Reanimated `useSharedValue` for 60fps).
- **Smoothing:** Raw progress values can be jerky. Apply a minimum animation duration of 200ms per progress update to smooth visual jumps. Never animate backwards (if progress jumps, go directly to new value).
- **Status text:** Right-aligned, showing percentage: "45%" — SF Pro Text Regular, 13pt, `colors.text.secondary`.
- **Indeterminate state:** If progress events are not firing (e.g., during server processing after upload), switch to an indeterminate animation (a gradient shimmer that slides left-to-right, looping).

#### 6.4 Retry Logic

| Failure Type | Auto-Retry | Delay | Max Retries | User Action |
|---|---|---|---|---|
| Network error (no connectivity) | Yes | 2 seconds | 1 | "Retry Upload" button after auto-retry fails |
| Timeout (60s) | Yes | 2 seconds | 1 | "Retry Upload" button after auto-retry fails |
| HTTP 500+ (server error) | Yes | 2 seconds | 1 | "Try Again" button (returns to camera for re-capture) |
| HTTP 400-499 (client error) | No | N/A | 0 | "Try Again" button or "Enter Manually" |
| HTTP 413 (payload too large) | No | N/A | 0 | Re-compress at lower quality, auto-retry once. If still too large, show error. |

#### 6.5 Cancellation

```typescript
// Cancelling an upload

uploadManager.cancel(taskId);
// Internally:
// 1. xhr.abort() — immediately stops the network request
// 2. task.status = 'cancelled'
// 3. Clean up: delete temp file associated with this upload
// 4. No callbacks fired after cancellation
```

- Cancel is instantaneous — `XMLHttpRequest.abort()` terminates the connection immediately.
- UI: Cancel button is always visible during upload phase. No confirmation dialog (the image can always be re-captured).
- After cancel: return to Preview screen (image is still in memory for "Use Photo" retry) or Camera screen.

#### 6.6 Background Upload

When the user backgrounds the app during an OCR upload, the upload should complete rather than be killed.

##### 6.6.1 Implementation Strategy

Use `react-native-background-upload` (which wraps iOS `NSURLSession` background transfer service).

```typescript
// services/backgroundUpload.ts
import Upload from 'react-native-background-upload';

async function startBackgroundUpload(
  taskId: string,
  fileUri: string,
  endpoint: string,
  authToken: string
): Promise<string> {
  const options = {
    url: `${Config.API_BASE_URL}${endpoint}`,
    path: fileUri,
    method: 'POST',
    type: 'multipart',
    field: 'image',
    headers: {
      Authorization: `Bearer ${authToken}`,
      Accept: 'application/json',
    },
    notification: {
      enabled: false,  // No push notification for OCR upload
    },
  };

  const uploadId = await Upload.startUpload(options);

  Upload.addListener('progress', uploadId, (data) => {
    // data.progress: 0-100
    uploadManager.updateProgress(taskId, data.progress / 100);
  });

  Upload.addListener('error', uploadId, (data) => {
    uploadManager.handleError(taskId, new Error(data.error));
  });

  Upload.addListener('completed', uploadId, (data) => {
    const response = JSON.parse(data.responseBody);
    uploadManager.handleComplete(taskId, response);
  });

  return uploadId;
}
```

##### 6.6.2 Background Upload Behavior

| Scenario | Behavior |
|---|---|
| App backgrounded during upload | `NSURLSession` background session continues the upload in a separate OS process. |
| App terminated by OS during upload | Upload continues. When app is relaunched, call `Upload.getFileInfo(uploadId)` to check status. |
| App returns to foreground mid-upload | Progress events resume. UI updates to reflect current progress. |
| Upload completes while app is backgrounded | Result is stored. When app returns to foreground, the completion handler fires and the OCR result is available. |
| Upload fails while app is backgrounded | Error is stored. When app returns to foreground, the error handler fires and the error screen is shown. |

##### 6.6.3 Foreground vs Background Upload Decision

```typescript
// Decision logic in UploadManager.upload():

const appState = AppState.currentState;

if (appState === 'active') {
  // Use XMLHttpRequest for real-time progress and simpler control
  return this.executeXHRUpload(task);
} else {
  // App is already backgrounded or about to be — use background service
  return this.executeBackgroundUpload(task);
}

// Also: listen for AppState changes. If app transitions to background
// during an XHR upload, the XHR will likely complete (iOS gives ~30s
// of background execution time). If it doesn't complete in that window,
// the upload will fail and the user can retry when they return.
// For V2, this edge case is acceptable. In V2.1, consider starting
// ALL uploads as background uploads for maximum reliability.
```

##### 6.6.4 Cleanup After Background Upload

- When the app returns to foreground after a background upload completed:
  1. Check for pending OCR results in the upload manager.
  2. If a result exists, present a local notification banner (in-app, not push): "Business card scan complete! Tap to review." 
  3. Tapping the banner navigates to the OCR Review screen (Section 2.4) with the pre-populated results.
  4. If the user dismisses the banner, the results are accessible from a "Pending Scans" badge on the Contacts tab (small red dot indicator).

---

### Appendix A: Library Dependencies Summary

| Purpose | Library | Version (minimum) | Notes |
|---|---|---|---|
| Camera | `react-native-vision-camera` | v4.0+ | Core camera for business card scanning |
| Image picker | `expo-image-picker` | SDK 51+ | Photo library selection |
| Image display | `expo-image` | SDK 51+ | All image rendering across the app |
| Image processing | `expo-image-manipulator` | SDK 51+ | Resize, rotate, compress captured photos |
| Haptics | `expo-haptics` | SDK 51+ | Tactile feedback on camera capture |
| File system | `expo-file-system` | SDK 51+ | Temp file management, cleanup |
| Sharing | `react-native-share` or `expo-sharing` | Latest | iOS share sheet integration |
| Background upload | `react-native-background-upload` | v7.0+ | iOS NSURLSession background transfers |
| Lottie animations | `lottie-react-native` | v6.0+ | Skye thinking animation, success checkmark, card scan |
| Markdown rendering | `react-native-markdown-display` | v7.0+ | Studio output rendering |
| Network info | `@react-native-community/netinfo` | v11.0+ | Offline detection for Studio |
| Linking | `expo-linking` (built-in) | N/A | Open Settings for permission denied flow |

### Appendix B: Animation Inventory

| Animation | Type | File | Duration | Loop |
|---|---|---|---|---|
| Business card scan illustration | Lottie | `lottie/card-scan.json` | 3s | Yes |
| Skye thinking / processing | Lottie | `lottie/skye-thinking.json` | 2s | Yes |
| Success checkmark | Lottie | `lottie/success-check.json` | 1s | No |
| Capture button pulse | Animated API | Inline | 100ms | No |
| Card frame corner brackets pulse | Animated API | Inline | 2s | Yes |
| Shutter flash | Animated API | Inline | 200ms | No |
| Progress bar fill | Reanimated | Inline | 200ms per update | No |
| Content type chip selection scale | Animated API | Inline | 200ms | No |
| Streaming cursor blink | Animated API | Inline | 500ms | Yes |
| Upload progress shimmer (indeterminate) | Reanimated | Inline | 1.5s | Yes |

### Appendix C: Accessibility Requirements

| Element | Accessibility Feature |
|---|---|
| Camera capture button | `accessibilityLabel="Take photo"`, `accessibilityRole="button"` |
| Flash toggle | `accessibilityLabel="Flash mode: ${currentMode}"`, `accessibilityRole="button"`, announces state change |
| Close camera button | `accessibilityLabel="Close camera"`, `accessibilityRole="button"` |
| OCR confidence indicators | `accessibilityLabel` includes confidence level: "Email, low confidence, please verify" |
| Preview image | `accessibilityLabel="Captured business card image"`, `accessibilityRole="image"` |
| Street View images | `accessibilityLabel="Street view of ${address}"`, `accessibilityRole="image"` |
| Contact avatars | `accessibilityLabel="${contactName}"`, `accessibilityRole="image"` |
| Studio generate button | `accessibilityLabel="Generate content with Skye"`, `accessibilityRole="button"`, announces disabled state |
| Studio output area | `accessibilityLabel="Generated content"`, `accessibilityRole="text"`, full content read by VoiceOver |
| Upload progress bar | `accessibilityLabel="Upload progress, ${percent} percent"`, `accessibilityRole="progressbar"`, `accessibilityValue` updates |
| Quick tweak chips | Each chip has `accessibilityRole="button"`, `accessibilityLabel` matching visible text |

### Appendix D: Analytics Events

| Event | Trigger | Properties |
|---|---|---|
| `camera.opened` | Camera viewfinder appears | `source: 'contacts' \| 'action_sheet'` |
| `camera.permission.requested` | System permission prompt shown | `previous_status` |
| `camera.permission.result` | User responds to permission prompt | `granted: boolean` |
| `camera.photo.captured` | User taps capture button | `flash_mode`, `capture_duration_ms` |
| `camera.photo.retaken` | User taps "Retake" on preview | - |
| `ocr.upload.started` | Upload begins | `file_size_bytes`, `image_dimensions` |
| `ocr.upload.completed` | Upload finishes | `duration_ms`, `file_size_bytes` |
| `ocr.upload.failed` | Upload fails | `error_type`, `retry_count` |
| `ocr.processing.completed` | OCR results received | `confidence_overall`, `fields_detected_count`, `processing_duration_ms` |
| `ocr.review.field_edited` | User edits a field on review screen | `field_name`, `was_empty: boolean` |
| `ocr.contact.created` | Contact created from OCR | `fields_count`, `edits_count`, `confidence_overall` |
| `ocr.abandoned` | User exits OCR flow without creating contact | `stage: 'preview' \| 'upload' \| 'review'`, `reason: 'manual_entry' \| 'close' \| 'back'` |
| `library.picker.opened` | Photo library picker opened | - |
| `library.photo.selected` | Photo selected from library | `file_size_bytes`, `source_type` |
| `studio.opened` | Studio screen appears | - |
| `studio.content_type.selected` | User selects a content type | `content_type` |
| `studio.generate.started` | Generate button tapped | `content_type`, `parameters_summary` |
| `studio.generate.completed` | Content generation finished | `content_type`, `duration_ms`, `output_length_chars` |
| `studio.generate.failed` | Generation failed | `content_type`, `error_type` |
| `studio.tweak.applied` | Quick tweak chip tapped | `tweak_type`, `content_type` |
| `studio.output.copied` | Copy button tapped | `content_type`, `output_length_chars` |
| `studio.output.shared` | Share button tapped | `content_type`, `share_target` |
| `studio.history.viewed` | History tab opened | `entries_count` |

---

This concludes the V2 specification for Camera, OCR, Media, and Studio systems. Every screen state, transition, animation, error path, and edge case has been documented. The spec is implementation-ready: an engineer can build each screen and flow directly from these wireframes, type definitions, and behavioral descriptions without ambiguity.

---


## Section 10: App Store Compliance & Submission

**Document Version:** 2.0
**App:** Skye -- AI Real Estate CRM
**Platform:** React Native (Expo) -- iOS only
**Target:** First-attempt App Store approval
**Last Updated:** 2026-03-02

---

### Table of Contents

1. [Privacy Manifest -- Exact Specification](#1-privacy-manifest--exact-specification)
2. [Privacy Nutrition Labels -- Exact Entries](#2-privacy-nutrition-labels--exact-entries)
3. [App Tracking Transparency Analysis](#3-app-tracking-transparency-analysis)
4. [Human Interface Guidelines Compliance Checklist](#4-human-interface-guidelines-compliance-checklist)
5. [App Store Submission Package](#5-app-store-submission-package)
6. [App Review Preparation](#6-app-review-preparation)
7. [Post-Launch Operations](#7-post-launch-operations)
8. [Legal Requirements](#8-legal-requirements)

---

### 1. Privacy Manifest -- Exact Specification

#### 1.1 Overview

Apple requires all iOS apps to include a `PrivacyInfo.xcprivacy` file declaring: (a) whether the app engages in tracking, (b) tracking domains, (c) collected data types, and (d) required-reason API usage. This requirement has been enforced since May 1, 2024. Apps submitted without a compliant privacy manifest will be rejected.

Since Skye is built with Expo (CNG workflow), the privacy manifest is configured via the `privacyManifests` field under `expo.ios` in `app.json` / `app.config.ts`. Expo generates the native `PrivacyInfo.xcprivacy` file during `npx expo prebuild`.

**Critical note on static linking:** React Native apps typically use statically linked CocoaPods dependencies. Apple does NOT correctly parse all `PrivacyInfo` files from static CocoaPods dependencies. Therefore, the app-level privacy manifest MUST include all required reason API declarations from every statically linked dependency (Sentry, Expo SDK packages, React Native core, etc.).

#### 1.2 Required Reason API Audit

The following table documents every required-reason API category and whether Skye or any of its dependencies uses it:

| API Category | Category Key | Used By | Reason Code | Justification |
|---|---|---|---|---|
| **File Timestamp APIs** (`NSFileCreationDate`, `NSFileModificationDate`, `NSFileContentModificationDate`) | `NSPrivacyAccessedAPICategoryFileTimestamp` | React Native core (Metro bundler file checks), Expo file system, Sentry (event caching), offline cache module | `C617.1` | Accessing file timestamps for the app's own files within its container. The offline cache checks modification dates to determine cache staleness. Sentry reads file timestamps for its own event files. |
| **System Boot Time APIs** (`systemUptime`, `mach_absolute_time`, `ProcessInfo.systemUptime`) | `NSPrivacyAccessedAPICategorySystemBootTime` | React Native internals (Hermes engine timing), Sentry (session duration tracking), React Native Reanimated (animation timing) | `35F9.1` | Measuring elapsed time between events within the app. Used for performance measurement, animation timing, and crash session duration -- NOT for device fingerprinting. |
| **Disk Space APIs** (`volumeAvailableCapacityKey`, `volumeAvailableCapacityForImportantUsageKey`) | `NSPrivacyAccessedAPICategoryDiskSpace` | Expo FileSystem (cache management), potential use by React Native image caching, offline data storage checks | `E174.1` | Checking available disk space to prevent writes when storage is critically low. The app's offline cache checks disk capacity before persisting contact/property data locally. |
| **User Defaults** (`NSUserDefaults`) | `NSPrivacyAccessedAPICategoryUserDefaults` | React Native (`AsyncStorage` fallback), Expo SecureStore (internal flags), Sentry (SDK configuration state), Google Sign-In SDK (auth state persistence), push notification token storage | `CA92.1` | Accessing user defaults to read and write data that is only accessible to the app itself. Used for persisting user preferences, auth state tokens, onboarding completion flags, and SDK configuration. |
| **Active Keyboard APIs** | `NSPrivacyAccessedAPICategoryActiveKeyboards` | **NOT USED** | N/A | Skye does not query the list of active keyboards. No dependency uses this API. No declaration required. |

#### 1.3 Third-Party SDK Privacy Manifest Audit

Each SDK must either bundle its own privacy manifest (for dynamic frameworks) or have its declarations merged into the app-level manifest (for static libraries):

| SDK | Version Requirement | Bundles Own Manifest? | Static/Dynamic | Action Required |
|---|---|---|---|---|
| `@sentry/react-native` | >= 5.22.1 | Yes (but static linking means Apple may not parse it) | Static | Include Sentry's API declarations in app-level manifest: `UserDefaults` (CA92.1), `SystemBootTime` (35F9.1), `FileTimestamp` (C617.1) |
| `GoogleSignIn-iOS` | >= 7.1.0 | Yes (includes privacy manifest + signatures) | Dynamic (XCFramework) | Apple will auto-process. Verify by generating Xcode Privacy Report. Still declare collected data types in nutrition labels. |
| `expo-camera` | Current Expo SDK | Yes (bundled in package) | Static | Include any required-reason APIs in app-level manifest. Camera access itself is governed by `NSCameraUsageDescription` in Info.plist, not the privacy manifest. |
| `expo-notifications` | Current Expo SDK | Yes (bundled in package) | Static | Include UserDefaults (CA92.1) in app-level manifest. |
| `expo-file-system` | Current Expo SDK | Yes (bundled in package) | Static | Include FileTimestamp (C617.1), DiskSpace (E174.1) in app-level manifest. |
| `expo-secure-store` | Current Expo SDK | Yes (bundled in package) | Static | May use UserDefaults internally. Already declared via CA92.1. |
| `react-native-reanimated` | >= 3.x | Yes (bundled in package) | Static | Uses `mach_absolute_time` for animation timing. Already declared via 35F9.1. |
| `@react-native-async-storage/async-storage` | >= 1.23.1 | Yes (bundled in package) | Static | Uses UserDefaults on iOS. Already declared via CA92.1. |

#### 1.4 Complete `PrivacyInfo.xcprivacy` File

The following is the exact XML content for the generated privacy manifest. In the Expo workflow, this is configured via `app.json` (see Section 1.5), but the resulting native file will contain:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>

    <!-- TRACKING DECLARATION -->
    <key>NSPrivacyTracking</key>
    <false/>

    <!-- TRACKING DOMAINS: None -- Skye does not engage in tracking -->
    <key>NSPrivacyTrackingDomains</key>
    <array/>

    <!-- COLLECTED DATA TYPES -->
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <!-- Contact Info: Name -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeName</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>

        <!-- Contact Info: Email Address -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeEmailAddress</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>

        <!-- Contact Info: Phone Number -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypePhoneNumber</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>

        <!-- Identifiers: User ID (for authentication) -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeUserID</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>

        <!-- Identifiers: Device ID (push notification token) -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeDeviceID</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>

        <!-- Diagnostics: Crash Data (Sentry) -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeCrashData</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <false/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>

        <!-- Diagnostics: Performance Data (Sentry) -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypePerformanceData</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <false/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>

        <!-- Diagnostics: Other Diagnostic Data (Sentry) -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeOtherDiagnosticData</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <false/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>
    </array>

    <!-- REQUIRED REASON API DECLARATIONS -->
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <!-- File Timestamp APIs -->
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string>
            </array>
        </dict>

        <!-- System Boot Time APIs -->
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategorySystemBootTime</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>35F9.1</string>
            </array>
        </dict>

        <!-- Disk Space APIs -->
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryDiskSpace</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>E174.1</string>
            </array>
        </dict>

        <!-- User Defaults APIs -->
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
    </array>

</dict>
</plist>
```

#### 1.5 Expo `app.json` Configuration

The Expo-native way to declare the privacy manifest. This configuration is placed under `expo.ios.privacyManifests`:

```jsonc
{
  "expo": {
    "ios": {
      "privacyManifests": {
        "NSPrivacyTracking": false,
        "NSPrivacyTrackingDomains": [],
        "NSPrivacyCollectedDataTypes": [
          {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypeName",
            "NSPrivacyCollectedDataTypeLinked": true,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
          },
          {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypeEmailAddress",
            "NSPrivacyCollectedDataTypeLinked": true,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
          },
          {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypePhoneNumber",
            "NSPrivacyCollectedDataTypeLinked": true,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
          },
          {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypeUserID",
            "NSPrivacyCollectedDataTypeLinked": true,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
          },
          {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypeDeviceID",
            "NSPrivacyCollectedDataTypeLinked": true,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
          },
          {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypeCrashData",
            "NSPrivacyCollectedDataTypeLinked": false,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
          },
          {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypePerformanceData",
            "NSPrivacyCollectedDataTypeLinked": false,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
          },
          {
            "NSPrivacyCollectedDataType": "NSPrivacyCollectedDataTypeOtherDiagnosticData",
            "NSPrivacyCollectedDataTypeLinked": false,
            "NSPrivacyCollectedDataTypeTracking": false,
            "NSPrivacyCollectedDataTypePurposes": ["NSPrivacyCollectedDataTypePurposeAppFunctionality"]
          }
        ],
        "NSPrivacyAccessedAPITypes": [
          {
            "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategoryFileTimestamp",
            "NSPrivacyAccessedAPITypeReasons": ["C617.1"]
          },
          {
            "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategorySystemBootTime",
            "NSPrivacyAccessedAPITypeReasons": ["35F9.1"]
          },
          {
            "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategoryDiskSpace",
            "NSPrivacyAccessedAPITypeReasons": ["E174.1"]
          },
          {
            "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategoryUserDefaults",
            "NSPrivacyAccessedAPITypeReasons": ["CA92.1"]
          }
        ]
      }
    }
  }
}
```

#### 1.6 Verification Procedure

1. Run `npx expo prebuild --clean` to regenerate the native iOS project.
2. Open the generated Xcode project in `ios/`.
3. Verify `PrivacyInfo.xcprivacy` exists in the bundle root.
4. In Xcode, choose **Product > Archive**, then Control-click the archive and choose **Generate Privacy Report**.
5. Review the generated report. Ensure all four required-reason API categories are listed with their reason codes.
6. Submit a build to TestFlight. If any `ITMS-91053` warnings appear in the email from Apple, add the missing API declarations and rebuild.
7. Re-verify after every dependency update -- new SDK versions may introduce new API usage.

---

### 2. Privacy Nutrition Labels -- Exact Entries

#### 2.1 Overview

Apple's privacy nutrition labels are completed in App Store Connect during app submission. They must accurately reflect all data collection by the app AND all third-party SDKs. Inaccurate labels are grounds for rejection under Guideline 5.1.1. The labels below must match the privacy manifest declarations in Section 1.

#### 2.2 Complete Data Type Declarations

##### 2.2.1 Contact Info -- Name

| Field | Value |
|---|---|
| **Data Type** | Name |
| **Collection** | Yes -- user enters contact names into the CRM |
| **Linked to User Identity** | Yes -- associated with the authenticated user's account |
| **Used for Tracking** | No |
| **Purpose** | App Functionality |
| **Data Retention** | Retained until the user deletes the contact or deletes their account |
| **Access** | Accessible by the authenticated user and authorized backend services |
| **Shared with Third Parties** | No -- contact names are not shared with any third party |

##### 2.2.2 Contact Info -- Email Address

| Field | Value |
|---|---|
| **Data Type** | Email Address |
| **Collection** | Yes -- user's own email via Google OAuth, plus contact emails entered into CRM |
| **Linked to User Identity** | Yes |
| **Used for Tracking** | No |
| **Purpose** | App Functionality |
| **Data Retention** | Retained until account deletion |
| **Access** | Authenticated user, backend services |
| **Shared with Third Parties** | No (Google receives the user's email only as part of the OAuth flow, which is initiated by the user) |

##### 2.2.3 Contact Info -- Phone Number

| Field | Value |
|---|---|
| **Data Type** | Phone Number |
| **Collection** | Yes -- contact phone numbers entered into CRM |
| **Linked to User Identity** | Yes |
| **Used for Tracking** | No |
| **Purpose** | App Functionality |
| **Data Retention** | Retained until contact or account deletion |
| **Access** | Authenticated user, backend services |
| **Shared with Third Parties** | No |

##### 2.2.4 Identifiers -- User ID

| Field | Value |
|---|---|
| **Data Type** | User ID |
| **Collection** | Yes -- Google OAuth user identifier |
| **Linked to User Identity** | Yes |
| **Used for Tracking** | No |
| **Purpose** | App Functionality (authentication) |
| **Data Retention** | Retained for the lifetime of the account |
| **Access** | Backend authentication services |
| **Shared with Third Parties** | No (Google issues the ID; it is not re-shared) |

##### 2.2.5 Identifiers -- Device ID

| Field | Value |
|---|---|
| **Data Type** | Device ID |
| **Collection** | Yes -- APNs push notification device token |
| **Linked to User Identity** | Yes -- associated with user account for push delivery |
| **Used for Tracking** | No |
| **Purpose** | App Functionality (push notifications) |
| **Data Retention** | Retained while user has push notifications enabled; deleted on account deletion |
| **Access** | Backend push notification service |
| **Shared with Third Parties** | No (APNs token is sent only to Apple's push notification service as part of normal push delivery) |

##### 2.2.6 Photos or Videos

| Field | Value |
|---|---|
| **Data Type** | Photos or Videos |
| **Collection** | **No** -- camera frames for OCR are processed ephemerally on-device or via a single API call. Raw images are never persisted to the server or stored beyond the immediate processing request. Per Apple's definition, data processed on-device without transmission is not "collected." |
| **Note** | If OCR sends the image to a backend for processing, declare this as collected. If processing is entirely on-device (e.g., Vision framework), no declaration is needed. **Decision: Verify implementation. If server-side OCR, declare as collected, not linked, app functionality.** |

##### 2.2.7 Usage Data -- Product Interaction

| Field | Value |
|---|---|
| **Data Type** | Product Interaction |
| **Collection** | No -- Skye V1 does not include any analytics SDK that tracks feature usage or screen views. Sentry captures crash/error events only, not general usage patterns. |
| **Note** | If analytics is added in a future version (e.g., Mixpanel, Amplitude, PostHog), this must be updated to "Yes" with appropriate purpose (Analytics). |

##### 2.2.8 Diagnostics -- Crash Data

| Field | Value |
|---|---|
| **Data Type** | Crash Data |
| **Collection** | Yes -- via Sentry |
| **Linked to User Identity** | No -- Sentry is configured without user-identifying PII. Crash reports contain device model, OS version, stack traces, and breadcrumbs but no user email/name. |
| **Used for Tracking** | No |
| **Purpose** | App Functionality (stability monitoring and bug fixing) |
| **Data Retention** | 90 days (Sentry default retention) |
| **Access** | Development team via Sentry dashboard |
| **Shared with Third Parties** | Yes -- data is transmitted to Sentry (functional.software GmbH) as a data processor for crash reporting purposes only |

##### 2.2.9 Diagnostics -- Performance Data

| Field | Value |
|---|---|
| **Data Type** | Performance Data |
| **Collection** | Yes -- via Sentry Performance Monitoring |
| **Linked to User Identity** | No |
| **Used for Tracking** | No |
| **Purpose** | App Functionality |
| **Data Retention** | 90 days |
| **Access** | Development team via Sentry dashboard |
| **Shared with Third Parties** | Yes -- Sentry (data processor) |

##### 2.2.10 Diagnostics -- Other Diagnostic Data

| Field | Value |
|---|---|
| **Data Type** | Other Diagnostic Data |
| **Collection** | Yes -- Sentry breadcrumbs, error context, device state |
| **Linked to User Identity** | No |
| **Used for Tracking** | No |
| **Purpose** | App Functionality |
| **Data Retention** | 90 days |
| **Access** | Development team |
| **Shared with Third Parties** | Yes -- Sentry (data processor) |

#### 2.3 Data Types NOT Collected (Explicit Denials)

The following data types are explicitly NOT collected by Skye and should be answered "No" in the App Store Connect questionnaire:

| Data Category | Data Type | Reason Not Collected |
|---|---|---|
| Health & Fitness | Health, Fitness | Not a health app |
| Financial Info | Payment Info, Credit Info | No in-app purchases in V1 |
| Location | Precise Location, Coarse Location | App does not request location permission |
| Sensitive Info | Sensitive data | Not collected |
| Contacts | Address book/Contact list | App does NOT access the device contacts -- users manually enter CRM contacts |
| User Content | Emails, Text Messages, Audio, Gameplay | Not applicable |
| Browsing History | Browsing History | No web browsing |
| Search History | Search History | Search within CRM is local; not transmitted as "search history" |
| Purchases | Purchase History | No purchases in V1 |

#### 2.4 App Store Connect Questionnaire Summary

When completing the questionnaire in App Store Connect, select:

**"Data Used to Track You":** None

**"Data Linked to You":** Contact Info (Name, Email, Phone), Identifiers (User ID, Device ID)

**"Data Not Linked to You":** Diagnostics (Crash Data, Performance Data, Other Diagnostic Data)

---

### 3. App Tracking Transparency Analysis

#### 3.1 Definition of "Tracking"

Per Apple, "tracking" means linking user or device data collected from your app with user or device data collected from other companies' apps, websites, or offline properties for the purposes of targeted advertising or advertising measurement, OR sharing user or device data with data brokers.

#### 3.2 SDK-by-SDK Tracking Audit

| SDK | Tracking? | Analysis |
|---|---|---|
| **Sentry** (`@sentry/react-native`) | **NO** | Sentry collects crash reports and performance data exclusively for the app developer's own debugging and monitoring. Data is not shared with advertisers, not used for cross-app profiling, and not sold to data brokers. Sentry is a data processor acting on behalf of the app developer. |
| **Google Sign-In** (`GoogleSignIn-iOS`) | **NO** | Used exclusively for authentication. Google receives the user's Google account identifier to complete the OAuth flow. This is a user-initiated authentication action, not cross-app tracking for advertising. Google Sign-In SDK v7.1+ does not engage in tracking as defined by Apple. However: if Google Analytics or Firebase Analytics were also integrated, the combination could constitute tracking. Skye does NOT include these SDKs. |
| **Expo Push Notifications** (`expo-notifications`) | **NO** | Push tokens are generated by APNs and sent to the app's own backend. No third-party advertising or cross-app data linkage occurs. |
| **Expo Camera** (`expo-camera`) | **NO** | Camera frames are used for OCR. No image data is shared with third parties for advertising. |
| **Expo Updates** (`expo-updates`) | **NO** | OTA update mechanism. Contacts Expo's update servers to check for JS bundle updates. No user-identifying data is shared for advertising. |

#### 3.3 ATT Prompt Determination

**Recommendation: ATT prompt is NOT required for Skye V1.**

Justification:
- No SDK in the dependency tree engages in "tracking" as defined by Apple.
- No advertising SDKs are integrated.
- No cross-app data linkage occurs.
- No data is shared with data brokers.
- Sentry data collection is for developer crash reporting only.
- Google Sign-In is for user-initiated authentication only.

#### 3.4 Prepared Justification for Apple Review

If Apple's review team questions the absence of an ATT prompt, provide this written justification in the App Review Notes:

> **ATT Justification:**
> Skye does not engage in tracking as defined by Apple's App Tracking Transparency framework. The app does not include any advertising SDKs, does not link user data with data from other companies' apps or websites for advertising purposes, and does not share data with data brokers. The only third-party data transmission is to Sentry for crash reporting (developer diagnostics only) and to Google for user-initiated OAuth authentication. Neither of these constitutes tracking under Apple's definition. Therefore, the ATT prompt is not displayed to users, and `NSPrivacyTracking` is set to `false` in the privacy manifest.

#### 3.5 Future Considerations

If any of the following are added in future versions, ATT analysis must be re-evaluated:
- Any analytics SDK with cross-app identifiers (e.g., Adjust, AppsFlyer, Branch)
- Any advertising SDK (e.g., AdMob, Meta Audience Network)
- Any integration that sends device identifiers to a third party for advertising measurement
- Firebase Analytics (even without ads, Firebase can contribute to Google's ad ecosystem)

**Action item:** Before integrating ANY new third-party SDK, perform an ATT audit. Check the SDK's `PrivacyInfo.xcprivacy` for `NSPrivacyTracking: true` and review the SDK's documentation for tracking behavior.

---

### 4. Human Interface Guidelines Compliance Checklist

Every item below must be verified before submission. Mark each with PASS/FAIL during QA. A single FAIL blocks submission.

#### 4.1 Visual Design

| # | Requirement | Pass/Fail Criteria | Verification Method |
|---|---|---|---|
| 4.1.1 | **Dark Mode** | Every screen renders correctly in both Light and Dark appearances. No white-on-white or black-on-black text. No hard-coded colors that ignore `colorScheme`. | Toggle Appearance in Settings > Display & Brightness. Walk through every screen in both modes. Use Xcode's Environment Overrides panel for rapid toggling during development. |
| 4.1.2 | **Dynamic Type** | All text scales correctly at all 12 accessibility text sizes (xSmall through AX5). No text truncation that destroys meaning. Layouts remain functional at the largest sizes. | Settings > Accessibility > Display & Text Size > Larger Text. Enable "Larger Accessibility Sizes." Test at minimum: default, largest non-accessibility size, and AX3. Verify no overlapping, no cutoff labels, scrollable containers expand. |
| 4.1.3 | **Safe Area Insets** | No content overlaps the Dynamic Island/notch, home indicator, or status bar. Content respects `SafeAreaView` on all edges. | Test on iPhone 16 Pro Max (Dynamic Island), iPhone SE 3rd gen (home button). Verify no content is clipped behind the sensor housing or hidden under the home indicator. |
| 4.1.4 | **Status Bar** | Correct style (light content on dark backgrounds, dark content on light backgrounds) on every screen. Status bar does not disappear unexpectedly. | Navigate every screen. Verify status bar text is legible against the background on each screen in both Light and Dark mode. |
| 4.1.5 | **Launch Screen** | Uses a native storyboard-based launch screen (SplashScreen.storyboard), NOT a React Native rendered screen. No visible "flash" between launch screen and first React Native render. | Cold-launch the app on a physical device. The launch screen should appear instantly from the native storyboard. Verify with Expo's `expo-splash-screen` configuration. |

#### 4.2 Interaction Design

| # | Requirement | Pass/Fail Criteria | Verification Method |
|---|---|---|---|
| 4.2.1 | **Touch Targets** | All interactive elements (buttons, links, toggles, list items) have a minimum hit area of 44x44 points. | Use Xcode's Accessibility Inspector to measure hit areas. Manually tap elements at their edges. Small icons must have expanded touch padding. |
| 4.2.2 | **System Gestures** | Edge swipe back gesture works on every screen that uses stack navigation. No custom gestures conflict with system gestures (edge swipe, home indicator swipe, notification pull-down). | On every screen with a back button, verify that swiping from the left edge navigates back. Verify bottom-edge swipe always returns to the home screen. |
| 4.2.3 | **Standard UI Components** | Alert dialogs use the system `Alert.alert()` (maps to `UIAlertController`), not custom modals that mimic alerts. Action sheets use system action sheet presentation. | Trigger every alert and confirmation dialog in the app. Verify they use the native iOS appearance (blurred background, standard button layout). |
| 4.2.4 | **Scroll Behavior** | All scrollable content exhibits native-feeling bounce, rubber-band, and momentum scrolling. No janky scroll implementations. | Rapidly scroll through long lists (contacts, properties). Verify overscroll bounce, smooth deceleration, and pull-to-refresh (if implemented). |
| 4.2.5 | **Keyboard Handling** | Keyboard does not obscure text inputs. `KeyboardAvoidingView` or equivalent is used on all screens with text input. Tapping outside the keyboard dismisses it (where appropriate). | Tap into every text field in the app. Verify the input scrolls into view above the keyboard. Verify dismissal behavior. Test with external keyboard connected. |

#### 4.3 Accessibility

| # | Requirement | Pass/Fail Criteria | Verification Method |
|---|---|---|---|
| 4.3.1 | **VoiceOver Navigation** | Every screen is fully navigable with VoiceOver. All interactive elements have meaningful accessibility labels. Decorative images are hidden from VoiceOver. Headers are marked as headers. | Enable VoiceOver (Settings > Accessibility > VoiceOver). Navigate every screen using swipe gestures. Verify every button, input, and list item is announced with a meaningful label. Verify navigation order is logical (top-to-bottom, left-to-right). |
| 4.3.2 | **Accessibility Labels** | All custom components have `accessibilityLabel`, `accessibilityHint`, and `accessibilityRole` set correctly. Icons used as buttons are labeled. | Use Xcode's Accessibility Inspector to audit every element on every screen. Verify no "Button" without a label, no unlabeled icons. |
| 4.3.3 | **Color Contrast** | Text meets minimum contrast ratio: 4.5:1 for normal text, 3:1 for large text (18pt+ or 14pt+ bold). Information is never conveyed by color alone. | Use Xcode's Accessibility Inspector contrast checker. Manually verify critical text elements. Ensure error states, status indicators, etc. have non-color indicators (icons, text). |
| 4.3.4 | **Reduce Motion** | If the app uses animations, respect the `Reduce Motion` setting. Provide alternatives to motion-based animations (e.g., cross-dissolve instead of slide). | Enable Settings > Accessibility > Motion > Reduce Motion. Navigate the app. Verify animations are simplified or removed. |

#### 4.4 Technical Compliance

| # | Requirement | Pass/Fail Criteria | Verification Method |
|---|---|---|---|
| 4.4.1 | **No Private APIs** | Binary scan is clean of private API usage. No undocumented selectors or frameworks. | Apple runs automated binary scans during review. Pre-validate with `nm` and `otool` on the built binary. |
| 4.4.2 | **No Deprecated APIs** | No usage of APIs deprecated before iOS 16 (the minimum deployment target). | Run Xcode build with "Treat Warnings as Errors" for deprecation warnings. Search codebase for known deprecated APIs. |
| 4.4.3 | **iPad Compatibility** | Either: (a) provide proper iPad-optimized layouts, OR (b) declare iPhone-only in `app.json` via `"supportsTablet": false`. Skye V1 recommendation: **iPhone-only**. | Set `"supportsTablet": false` in `app.json`. Verify the app does not appear in the iPad App Store as a native iPad app. |
| 4.4.4 | **No Misleading Features** | The app does exactly what the App Store description says. No hidden functionality, no undisclosed features, no bait-and-switch. | Compare every claim in the App Store description against actual app behavior. Verify all screenshots match real screens. |
| 4.4.5 | **Minimum iOS Version** | App builds and runs on the declared minimum iOS version (recommend iOS 16.0+). | Test on a device or simulator running iOS 16.0. Verify no crashes on launch, all features work. |

#### 4.5 QA Sign-Off Template

Before submission, the QA lead must sign off on the following:

```
HIG COMPLIANCE SIGN-OFF
========================
Date: ____________
Build Version: ____________
Tester: ____________

[ ] 4.1.1 Dark Mode            PASS / FAIL  Notes: ___
[ ] 4.1.2 Dynamic Type         PASS / FAIL  Notes: ___
[ ] 4.1.3 Safe Area Insets     PASS / FAIL  Notes: ___
[ ] 4.1.4 Status Bar           PASS / FAIL  Notes: ___
[ ] 4.1.5 Launch Screen        PASS / FAIL  Notes: ___
[ ] 4.2.1 Touch Targets        PASS / FAIL  Notes: ___
[ ] 4.2.2 System Gestures      PASS / FAIL  Notes: ___
[ ] 4.2.3 Standard UI          PASS / FAIL  Notes: ___
[ ] 4.2.4 Scroll Behavior      PASS / FAIL  Notes: ___
[ ] 4.2.5 Keyboard Handling    PASS / FAIL  Notes: ___
[ ] 4.3.1 VoiceOver            PASS / FAIL  Notes: ___
[ ] 4.3.2 Accessibility Labels PASS / FAIL  Notes: ___
[ ] 4.3.3 Color Contrast       PASS / FAIL  Notes: ___
[ ] 4.3.4 Reduce Motion        PASS / FAIL  Notes: ___
[ ] 4.4.1 No Private APIs      PASS / FAIL  Notes: ___
[ ] 4.4.2 No Deprecated APIs   PASS / FAIL  Notes: ___
[ ] 4.4.3 iPad Compatibility   PASS / FAIL  Notes: ___
[ ] 4.4.4 No Misleading        PASS / FAIL  Notes: ___
[ ] 4.4.5 Minimum iOS Version  PASS / FAIL  Notes: ___

ALL ITEMS PASS: [ ] YES  [ ] NO
Submission Approved: ____________ (signature)
```

---

### 5. App Store Submission Package

#### 5.1 App Store Connect Metadata

##### 5.1.1 App Name

```
Skye - AI Real Estate CRM
```

Character count: 24 (within 30-character limit)

Rationale: Includes the brand name ("Skye"), the primary keyword ("Real Estate"), the category keyword ("CRM"), and the differentiator ("AI"). The dash separator is clean and professional.

##### 5.1.2 Subtitle

```
AI Assistant for Top Agents
```

Character count: 27 (within 30-character limit)

Rationale: Reinforces the AI capability, targets the aspirational "top agents" audience, and avoids repeating keywords from the app name. "Assistant" expands keyword reach beyond "CRM."

##### 5.1.3 Keywords (100-character field)

```
realtor,broker,leads,property,listings,contacts,pipeline,follow-up,client,home,sales,housing,manage
```

Character count: 99

Keyword strategy notes:
- No spaces after commas (saves characters).
- No repetition of words already in the app name ("real", "estate", "crm", "ai") or subtitle ("assistant", "agent") -- Apple automatically indexes those.
- Covers synonyms: "realtor" and "broker" capture alternative user searches.
- Includes action words: "manage", "follow-up".
- Includes industry terms: "listings", "pipeline".
- No plural forms if singular is included (Apple matches both).
- No competitor brand names (grounds for rejection).

##### 5.1.4 Description (up to 4,000 characters)

```
Skye is the AI-powered CRM built exclusively for real estate professionals. Manage your contacts,
track properties, and get intelligent insights -- all from your iPhone.

MEET YOUR AI ASSISTANT
Skye includes a built-in AI assistant that understands real estate. Ask it anything:
- "Who should I follow up with this week?"
- "Draft a message to Sarah about her listing on Oak Street"
- "What properties did I show to the Johnsons?"
- "Summarize my pipeline for this month"

Your AI assistant remembers your contacts, your deals, and your preferences. It gets smarter
the more you use it.

CONTACTS & RELATIONSHIPS
Keep every client, lead, and referral partner organized in one place. Skye stores names, phone
numbers, emails, and notes -- then helps you stay on top of every relationship.
- Add contacts manually or scan business cards with your camera
- Tag and categorize contacts (buyer, seller, investor, lender)
- See full interaction history for every contact
- Never forget a follow-up with smart reminders

PROPERTY TRACKING
Track every listing, showing, and deal in your pipeline.
- Log properties with address, price, status, and notes
- Link properties to contacts
- Monitor your pipeline at a glance

BUSINESS CARD SCANNER
Use your iPhone camera to instantly capture contact information from business cards. Skye uses
OCR technology to extract names, phone numbers, and emails -- then creates a new contact for
you in seconds.

MORNING BRIEFING
Start every day with a personalized briefing from your AI assistant. See who needs attention,
what follow-ups are due, and what is on your schedule -- delivered as a notification each morning.

BUILT FOR REAL ESTATE
Unlike generic CRMs, Skye speaks the language of real estate. It understands listings, showings,
open houses, buyer consultations, and the rhythms of a real estate business.

PRIVACY & SECURITY
Your client data is protected with enterprise-grade security. All data is encrypted in transit
and at rest. We never sell your data. You can export or delete your data at any time.

Download Skye today and close more deals with less effort.
```

Character count: approximately 1,850 (well within 4,000-character limit)

##### 5.1.5 Promotional Text

```
New: AI-powered morning briefings and smart follow-up reminders. Your real estate business, simplified.
```

Character count: 103 (within 170-character limit)

Note: Promotional text can be updated at any time without a new app version submission. Use this for seasonal promotions, feature launches, and event tie-ins.

##### 5.1.6 Category

| Field | Value |
|---|---|
| **Primary Category** | Business |
| **Secondary Category** | Productivity |

##### 5.1.7 Content Rating Questionnaire

Answer the following in App Store Connect:

| Question | Answer |
|---|---|
| Cartoon or Fantasy Violence | None |
| Realistic Violence | None |
| Prolonged Graphic or Sadistic Realistic Violence | None |
| Profanity or Crude Humor | None |
| Mature/Suggestive Themes | None |
| Horror/Fear Themes | None |
| Medical/Treatment Information | None |
| Alcohol, Tobacco, or Drug Use or References | None |
| Simulated Gambling | None |
| Sexual Content or Nudity | None |
| Graphic Sexual Content and Nudity | None |
| Unrestricted Web Access | No |
| Gambling with Real Currency | No |
| Contests | No |

**Resulting Age Rating:** 4+ (Contains no objectionable material)

**AI content consideration:** Skye uses AI to generate text responses. The AI is constrained to real estate CRM functionality and does not generate violent, sexual, or otherwise objectionable content. If Apple's updated (2025) questionnaire asks about AI-generated content, select that AI does not generate objectionable user-facing content.

##### 5.1.8 Price and Availability

| Field | Value |
|---|---|
| **Price** | Free |
| **In-App Purchases** | None in V1. If premium features are planned for V2, declare "In-App Purchases" and implement via StoreKit 2. |
| **Availability** | All territories (or restrict to US, Canada, UK, Australia initially if backend infrastructure is region-limited) |

#### 5.2 Screenshots

##### 5.2.1 Required Sizes

| Display Size | Dimensions (Portrait) | Priority | Notes |
|---|---|---|---|
| **6.9" (iPhone 16 Pro Max)** | 1320 x 2868 px | **REQUIRED** | Primary set. All other sizes auto-scale from this. |
| **6.7" (iPhone 15 Pro Max)** | 1290 x 2796 px | Recommended | Provide if the UI looks significantly different. Otherwise, Apple auto-scales from 6.9". |
| **6.5" (iPhone 14 Plus, etc.)** | 1284 x 2778 px | Optional | Auto-scales from 6.9" if not provided. |
| **5.5" (iPhone 8 Plus)** | 1242 x 2208 px | Conditional | Required ONLY if supporting devices older than iPhone X. |

File requirements: Flattened JPEG or PNG, RGB color space, no transparency, max 10MB per file. Up to 10 screenshots per localization.

##### 5.2.2 Recommended Screenshot Set (8 Screenshots)

Design each screenshot with: device frame mockup, gradient background (brand colors), headline text above the device.

| # | Screen | Headline Text | Content Description |
|---|---|---|---|
| 1 | AI Chat | "Your AI Real Estate Assistant" | Chat interface with a sample conversation: user asks "Who should I follow up with?" and AI responds with a list |
| 2 | People Tab | "Every Contact, Organized" | Contacts list with multiple entries, search bar, category tags (Buyer, Seller, Investor) |
| 3 | Contact Detail | "Complete Client Profiles" | Single contact detail view with name, phone, email, tags, notes, interaction history |
| 4 | Properties | "Track Every Listing" | Property list with addresses, prices, status badges (Active, Pending, Closed) |
| 5 | OCR Scanning | "Scan Business Cards Instantly" | Camera viewfinder with a business card in frame and extracted data overlaid |
| 6 | Morning Briefing | "Start Your Day Informed" | Morning briefing screen with follow-up reminders, pipeline summary |
| 7 | Studio/AI Features | "AI That Knows Real Estate" | Conversation where the AI drafts a follow-up message |
| 8 | Push Notification | "Never Miss a Follow-Up" | Lock screen with a Skye push notification |

##### 5.2.3 Screenshot Design Specifications

- **Background:** Gradient using brand colors (recommend deep blue to teal, or similar professional palette)
- **Device frame:** iPhone 16 Pro Max mockup, angled slightly (5-10 degrees) for visual interest
- **Headline font:** SF Pro Display Bold, 56-64pt, white text
- **Sub-headline (optional):** SF Pro Display Regular, 32-36pt, white with 80% opacity
- **Screenshot content:** Actual app screens with realistic demo data (not lorem ipsum)

#### 5.3 App Preview Video

| Field | Value |
|---|---|
| **Required** | No (optional), but highly recommended for AI-powered apps |
| **Duration** | 15-30 seconds |
| **Resolution** | 1320 x 2868 (6.9" iPhone, portrait) |
| **Format** | H.264, MP4 or MOV |
| **Content** | Show: (1) app launch, (2) typing an AI question and receiving a response, (3) scrolling through contacts, (4) scanning a business card, (5) morning briefing notification |
| **Restrictions** | No iPhone hardware other than screen content. No pricing information. No "Download now" CTAs. |

#### 5.4 App Icon

| Field | Value |
|---|---|
| **Size** | 1024 x 1024 px |
| **Format** | PNG |
| **Alpha channel** | None (no transparency) |
| **Rounded corners** | None (iOS applies the mask automatically) |
| **Design** | Single distinctive "S" or cloud/sky motif. Clean, recognizable at small sizes. Avoid text, photographs, or screenshots within the icon. |

---

### 6. App Review Preparation

#### 6.1 Demo Account

Create and maintain a persistent demo account that the Apple review team can use:

| Field | Value |
|---|---|
| **Email** | `demo@skyeapp.com` (or a dedicated test account) |
| **Password** | `SkyeDemo2026!` |
| **Authentication** | If Google OAuth only: create a Google account `skye.demo.review@gmail.com` with the same password. Alternatively, implement email/password login as a fallback for review. |
| **Pre-populated data** | Minimum 20 contacts (realistic names, phone numbers, emails), 10 properties (real addresses, various statuses), 15+ AI chat messages showing diverse queries, 5+ notes/interactions per contact |
| **Data persistence** | Demo account data must NOT be wiped by automated cleanup jobs. Flag this account in the database as `is_demo_account: true`. |
| **Availability** | Backend services must be live and responsive 24/7 during review. Apple reviews at all hours, including weekends. |

**Google OAuth workaround:** If the app uses Google OAuth exclusively and Apple's reviewer cannot log in with a Google account (common), provide one of these alternatives:
1. A "Sign in with Apple" option (recommended -- Apple requires this per Guideline 4.8)
2. A dedicated bypass: email/password login for the demo account only
3. A pre-authenticated TestFlight build (risky -- session may expire)

**Recommended approach:** Implement "Sign in with Apple" as a primary login option alongside Google OAuth. This satisfies Guideline 4.8, provides a login method the reviewer can easily use, and avoids rejection.

#### 6.2 App Review Notes

Enter the following in the "Notes for Reviewer" field in App Store Connect:

```
DEMO ACCOUNT
=============
To test the app, please use the following credentials:

Option 1 -- Sign in with Apple:
Use any Apple ID to sign in. A new account will be created with sample data.

Option 2 -- Google Sign-In (pre-populated demo account):
Email: skye.demo.review@gmail.com
Password: SkyeDemo2026!

Once signed in, the demo account contains 25 pre-populated contacts,
12 properties, and AI chat history to explore.

WHAT THE APP DOES
==================
Skye is a CRM (Customer Relationship Management) tool built for real
estate agents. Users can:
1. Manage contacts (clients, leads, referral partners)
2. Track properties and deals in their pipeline
3. Chat with an AI assistant that understands their CRM data
4. Scan business cards to create contacts (uses camera)
5. Receive morning briefing notifications with daily priorities

HOW TO TEST THE AI CHAT
========================
Tap the Chat tab and try these example prompts:
- "Who should I follow up with this week?"
- "Tell me about Sarah Johnson" (a pre-populated contact)
- "What properties are currently active in my pipeline?"
- "Draft a follow-up email to the Johnsons about 456 Elm Street"

The AI assistant responds based on the user's actual CRM data.

PUSH NOTIFICATIONS
===================
The app requests push notification permission to deliver:
- Morning briefing summaries (daily digest of priorities)
- Follow-up reminders (when a contact is due for outreach)
- Property status updates

Push notifications are essential to the CRM functionality -- they
ensure agents never miss time-sensitive follow-ups with clients.

CAMERA USAGE
=============
The app uses the camera exclusively for scanning business cards via
OCR (Optical Character Recognition). When a user receives a business
card at an open house or networking event, they can scan it to
instantly create a CRM contact. Camera data is processed in real-time
and is not stored as photographs.

TRACKING & PRIVACY
===================
This app does NOT engage in tracking as defined by Apple's App
Tracking Transparency framework. No advertising SDKs are integrated.
The only third-party data transmission is:
- Sentry: crash reporting (developer diagnostics, not linked to user)
- Google: OAuth authentication (user-initiated sign-in)
No ATT prompt is displayed because no tracking occurs.

ACCOUNT DELETION
=================
Users can delete their account and all associated data via:
Settings tab > Account > Delete Account
This permanently removes all user data from our servers.
```

#### 6.3 Common Rejection Reasons -- Prevention Matrix

| Guideline | Rejection Reason | How Skye Prevents This | Verification |
|---|---|---|---|
| **2.1 App Completeness** | Crashes, placeholder screens, broken features | All features fully implemented before submission. No "Coming Soon" screens. No debug menus accessible to users. | Full regression test on release build. |
| **2.3 Accurate Metadata** | Screenshots don't match app, description promises non-existent features | All screenshots captured from the actual submission build. Every feature mentioned in the description is functional. | Side-by-side comparison. |
| **3.1.1 In-App Purchases** | Digital content unlocked outside Apple IAP | V1 is fully free. No payment processing. | Verify no hidden payment flows. |
| **4.0 Design** | App doesn't provide sufficient value beyond a mobile website | Skye provides native capabilities (push, camera OCR, offline caching, haptics, native nav) that a website cannot. | Document native-exclusive features. |
| **4.2 Minimum Functionality** | App is a thin wrapper around a web view | Full-featured native app with multiple screens, native navigation, camera integration, push, offline, AI. | Review team can navigate 6+ functional areas. |
| **4.8 Sign in with Apple** | App offers third-party sign-in but not Sign in with Apple | Implement Sign in with Apple alongside Google OAuth. **This is a hard requirement.** | Verify button appears on login screen and works end-to-end. |
| **5.1.1(i) Data Collection** | Privacy manifest or nutrition labels inaccurate | Sections 1 and 2 of this document provide exact declarations. | Generate Xcode Privacy Report and cross-reference. |
| **5.1.1(v) Account Deletion** | No in-app account deletion | Settings > Account > Delete Account. Deletes all server-side data. | Test end-to-end. Verify data removed from database. |
| **5.1.2 Data Use and Sharing** | Undisclosed data sharing | Only sharing: Sentry (crashes) and Google (OAuth). Both disclosed. | Audit all network requests with proxy tool. |

#### 6.4 Pre-Submission Checklist

```
PRE-SUBMISSION CHECKLIST
=========================
[ ] App builds and runs on release configuration (not debug)
[ ] App tested on physical device (not just simulator)
[ ] App tested on minimum supported iOS version (16.0)
[ ] App tested on latest iOS version
[ ] Demo account is populated and credentials work
[ ] Backend services are live and stable
[ ] Privacy manifest is configured (Section 1.5)
[ ] Privacy nutrition labels entered in App Store Connect (Section 2)
[ ] All screenshots captured from the submission build
[ ] App description matches actual functionality
[ ] Privacy policy URL is live and accessible
[ ] Terms of service URL is live and accessible
[ ] Support URL is live and accessible
[ ] Contact email is valid and monitored
[ ] Account deletion works end-to-end
[ ] Sign in with Apple works end-to-end
[ ] Google Sign-In works end-to-end
[ ] Push notifications work (test via APNs)
[ ] Camera OCR works on physical device
[ ] Offline mode works (airplane mode test)
[ ] HIG compliance checklist signed off (Section 4.5)
[ ] No ITMS-91053 warnings from TestFlight upload
[ ] Xcode Privacy Report reviewed and clean
[ ] App icon is 1024x1024, PNG, no alpha, no rounded corners
[ ] All URLs in metadata are HTTPS and return 200
[ ] Age rating questionnaire completed accurately
[ ] App Review notes entered (Section 6.2)
```

---

### 7. Post-Launch Operations

#### 7.1 Phased Release

**Note:** Phased release is only available for app UPDATES, not the initial release. The V1.0 launch goes to 100% immediately. Use phased release starting with V1.0.1 and all subsequent updates.

**Recommendation:** Enable 7-day phased release for all update submissions.

| Day | Percentage of Auto-Update Users |
|---|---|
| Day 1 | 1% |
| Day 2 | 2% |
| Day 3 | 5% |
| Day 4 | 10% |
| Day 5 | 20% |
| Day 6 | 50% |
| Day 7 | 100% |

Important behaviors:
- Users who manually check for updates or download the app for the first time always get the latest version regardless of phased release stage.
- Phased release percentages are based on INSTALLED users, not active users. Actual numbers may be higher than expected.
- You can pause the phased release at any time. The pause lasts up to 30 days total across unlimited pause/resume cycles.
- You can release to 100% at any time after the update reaches "Ready for Distribution" status.

Configuration: In App Store Connect, on the version page, under "Phased Release," check "Release update over 7-day period" before submitting for review.

#### 7.2 Monitoring During Rollout

| Metric | Tool | Threshold | Action if Breached |
|---|---|---|---|
| **Crash-free rate** | Sentry | >= 99.0% | Pause phased release. Investigate top crasher. Hotfix via OTA if JS-only, or submit expedited App Store update if native. |
| **Crash-free rate (critical)** | Sentry | < 97.0% | Pause immediately. Halt marketing. Investigate. Consider pulling the update. |
| **API error rate (5xx)** | Backend monitoring | < 1% of requests | Investigate backend. Scale infrastructure. |
| **API latency (p95)** | Backend monitoring | < 2,000ms | Investigate slow queries, AI response times. |
| **App Store rating** | App Store Connect | >= 4.0 stars | Monitor daily. Investigate common complaints if below 4.0. |
| **User feedback** | App Store Connect reviews | No pattern of identical complaints | If 3+ users report the same issue, investigate immediately. |
| **Push notification delivery** | APNs dashboard | >= 95% delivery | Investigate invalid/expired tokens. |
| **AI response quality** | Backend logs | No pattern of failures or hallucinations | Review AI prompt engineering. |

#### 7.3 Pause Decision Framework

```
IF crash-free rate < 99.0% for > 1 hour:
    1. PAUSE phased release in App Store Connect
    2. ALERT on-call engineer
    3. Identify top crasher in Sentry
    4. Determine if fix is JS-only or native:
        - JS-only: deploy OTA update via Expo Updates
        - Native: submit emergency App Store update (request expedited review)
    5. VERIFY fix on staging
    6. DEPLOY fix
    7. MONITOR for 2 hours
    8. RESUME phased release if crash-free >= 99.5%

IF crash-free rate < 97.0%:
    1. PAUSE phased release IMMEDIATELY
    2. ALERT engineering leadership
    3. CONSIDER pulling the update entirely
    4. ROOT CAUSE analysis before any resume
```

#### 7.4 App Store Rating Prompt

Implementation: Use `SKStoreReviewController.requestReview(in:)` (the non-deprecated API).

In React Native/Expo, use the `expo-store-review` package:

```typescript
import * as StoreReview from 'expo-store-review';

async function maybeRequestReview() {
  const isAvailable = await StoreReview.isAvailableAsync();
  if (isAvailable) {
    await StoreReview.requestReview();
  }
}
```

**Trigger conditions (ALL must be true):**
1. User has been using the app for >= 7 days since first install
2. User has completed >= 5 sessions
3. User has just completed a "positive moment" -- one of:
   - Successfully sent an AI chat message and received a helpful response
   - Created a new contact (manually or via business card scan)
   - Completed a follow-up action
4. User has NOT been prompted in the current app version
5. User has NOT been prompted in the last 120 days (conservative -- iOS enforces max 3 per 365 days)

**Things NOT to do:**
- Do NOT trigger from a button labeled "Rate Us" (the system prompt may not appear)
- Do NOT prompt on first launch
- Do NOT prompt after a negative experience
- Do NOT use a custom pre-screening prompt

**In-app "Leave a Review" link:** For a user-facing option, deep link to the App Store:

```typescript
const APP_STORE_ID = 'XXXXXXXXXX'; // Replace with actual ID
Linking.openURL(`https://apps.apple.com/app/id${APP_STORE_ID}?action=write-review`);
```

#### 7.5 Update Strategy

| Update Type | Mechanism | Review Required? | Turnaround | Use Case |
|---|---|---|---|---|
| **JS Bug Fixes** | Expo Updates (OTA) | No | Minutes | Crash fixes in JS, text changes, minor UI adjustments |
| **JS Feature Updates** | Expo Updates (OTA) | No | Minutes | New prompts, UI enhancements, new JS-only screens |
| **Native Bug Fixes** | App Store submission | Yes (request expedited) | 24-48 hours | Native module crashes, dependency updates |
| **Native Feature Updates** | App Store submission | Yes (standard) | 1-3 days | New native module integration, new permissions |
| **SDK Updates** | App Store submission | Yes | 1-3 days | Sentry, Expo SDK, Google Sign-In updates |

**Expo Updates (OTA) compliance:** OTA updates are permitted under Apple's Guidelines Section 3.3.2 as long as they only modify JavaScript bundles and assets, do not fundamentally change the app's purpose, and do not bypass review for significant changes. Keep a changelog of all OTA deployments.

#### 7.6 Version Management

| Field | Convention | Example |
|---|---|---|
| `CFBundleShortVersionString` | Semantic versioning: `MAJOR.MINOR.PATCH` | `1.0.0` |
| `CFBundleVersion` | Monotonically increasing integer | `1`, `2`, `3`, ... |
| App Store display version | Matches `CFBundleShortVersionString` | `1.0.0` |

Version increment rules:
- **MAJOR** (1.x.x to 2.0.0): Breaking changes, major redesign
- **MINOR** (1.0.x to 1.1.0): New features, new screens
- **PATCH** (1.0.0 to 1.0.1): Bug fixes, performance improvements

Build number must always increment for every App Store submission. Apple rejects duplicate build numbers.

#### 7.7 Customer Support

| Channel | Location | Notes |
|---|---|---|
| **Support Email** | App Store Connect "Support URL" + in-app Settings | `support@skyeapp.com`. Monitored daily. |
| **App Store Reviews** | App Store Connect | Respond to every 1-star and 2-star review within 24 hours. |
| **In-App Feedback** | Settings > Send Feedback | Pre-populated with device model, OS version, app version. |
| **Privacy Inquiries** | `privacy@skyeapp.com` | Required for GDPR/CCPA. Linked from privacy policy. |

---

### 8. Legal Requirements

#### 8.1 Terms of Service

| Requirement | Implementation |
|---|---|
| **Availability** | Link displayed: (1) during onboarding, (2) in app Settings, (3) in App Store Connect metadata |
| **URL** | `https://skyeapp.com/terms` |
| **Content** | Standard SaaS terms: acceptable use, intellectual property, limitation of liability, termination, dispute resolution, governing law |
| **AI Disclaimer** | Must include: AI responses may contain errors. Skye is not a substitute for professional advice. Users should verify AI information. |

#### 8.2 Privacy Policy

| Requirement | Implementation |
|---|---|
| **Availability** | Link displayed: (1) during onboarding, (2) in app Settings, (3) in App Store Connect, (4) on website |
| **URL** | `https://skyeapp.com/privacy` |

**Privacy policy must address all of the following:**

1. **What data is collected:** Contact info (name, email, phone), authentication data (Google/Apple ID), device identifiers (APNs token), diagnostic data (crash reports via Sentry), camera data (ephemeral OCR processing)

2. **How data is used:** CRM functionality, authentication, push notifications, crash reporting, business card OCR

3. **Who has access:** The authenticated user, backend infrastructure, Sentry (crash data only), Google/Apple (auth tokens only)

4. **Data retention:** CRM data until user deletes; auth data for account lifetime; push tokens while enabled; diagnostics 90 days; camera images not retained

5. **Third-party sharing:** Sentry receives anonymized crash data. No data sold. No data shared with advertisers or data brokers.

6. **User rights:** Right to access (view in-app), right to delete (account deletion), right to export, right to opt out of notifications

7. **Children's privacy:** Not directed at children under 13. No knowing collection from children.

8. **Contact:** `privacy@skyeapp.com`

#### 8.3 Account Deletion

Apple requirement (Guideline 5.1.1(v)): Any app that supports account creation MUST offer in-app account deletion. Enforced since June 30, 2022.

| Aspect | Specification |
|---|---|
| **Location** | Settings > Account > Delete Account |
| **Visibility** | No more than 2 taps from Settings |
| **Confirmation** | Dialog: "Are you sure? This permanently deletes all contacts, properties, chat history, and account data. This cannot be undone." with Cancel and Delete buttons. |
| **Identity verification** | Require re-authentication before deletion |
| **What is deleted** | ALL user data: contacts, properties, notes, chat history, AI logs, push tokens, preferences, profile |
| **Token revocation** | If using Sign in with Apple: revoke tokens via Apple REST API (`https://appleid.apple.com/auth/revoke`). **Required by Apple.** |
| **Timeline** | Complete within 24 hours. If longer, inform user and confirm when done. |
| **Post-deletion** | User signed out, returned to login. Fresh account if they sign up again. |
| **Server-side** | Soft-delete immediately (stop serving data). Hard-delete within 30 days. |

**Do NOT:** require email to support for deletion, make deletion hard to find, offer only deactivation, or retain data after stated period.

#### 8.4 CCPA Compliance

Applicable if any user is a California resident (likely for real estate agents).

| Requirement | Implementation |
|---|---|
| **Right to Know** | View data in-app; request comprehensive export |
| **Right to Delete** | Account deletion (Section 8.3) |
| **Right to Opt-Out of Sale** | Skye does not sell personal information. State in privacy policy. |
| **Non-Discrimination** | Same service quality regardless of rights exercised |

#### 8.5 GDPR Compliance

Applicable if any user is an EU/EEA resident.

| Requirement | Implementation |
|---|---|
| **Lawful Basis** | Contract performance |
| **Consent** | Explicit consent for push notifications; consent during account creation |
| **Right of Access** | View data in-app; support requests via privacy email |
| **Right to Rectification** | Edit contacts/profile directly in app |
| **Right to Erasure** | Account deletion (Section 8.3) |
| **Right to Data Portability** | Data export (JSON/CSV) in V1.1; via email in V1 |
| **Data Processing Agreements** | Execute DPAs with Sentry, hosting provider, Google |
| **Cross-border transfers** | Standard Contractual Clauses if US-hosted with EU users |

#### 8.6 EULA

**Recommendation for V1:** Use Apple's standard EULA. Include AI content disclaimers in the Terms of Service. A custom EULA is only needed if you have unique licensing terms, specific liability limitations, or custom AI content licensing.

---

### Appendix A: Info.plist Permission Strings

```xml
<!-- Camera Permission (Business Card OCR) -->
<key>NSCameraUsageDescription</key>
<string>Skye uses your camera to scan business cards and extract contact information using OCR.</string>
```

Do NOT request permissions the app does not use. Each permission request must have a corresponding feature.

---

### Appendix B: Apple Guideline 4.8 -- Sign in with Apple (Critical)

Because Skye offers Google Sign-In (a third-party authentication service), Apple Guideline 4.8 **requires** that the app ALSO offer "Sign in with Apple" as an equivalent option.

Implementation requirements:
1. "Sign in with Apple" button must appear alongside Google Sign-In on the login screen.
2. Both buttons must be equally prominent (same size, similar visual weight).
3. Sign in with Apple must use the system-provided `ASAuthorizationAppleIDButton`.
4. The flow must create a full account with the same capabilities.
5. When deleting accounts, revoke Sign in with Apple tokens via Apple's REST API.

**Failure to implement Sign in with Apple when offering Google Sign-In is one of the most common rejection reasons. This is a hard requirement.**

---

### Appendix C: Submission Timeline

| Step | Timeline | Owner |
|---|---|---|
| Complete development | T-14 days | Engineering |
| HIG compliance testing | T-12 days | QA |
| Privacy manifest + nutrition labels | T-10 days | Engineering |
| Screenshots + video | T-10 days | Design |
| Demo account population | T-8 days | Engineering |
| App Store Connect metadata | T-7 days | Product |
| Privacy policy + ToS | T-7 days | Legal |
| Internal TestFlight | T-7 to T-5 | QA + Team |
| External TestFlight beta | T-5 to T-3 | Beta testers |
| Pre-submission checklist | T-2 days | Engineering + QA |
| Submit to App Review | T-0 | Engineering |
| Review period | T+1 to T+3 | Apple |
| Approval + release | T+3 (target) | Release |

Total recommended lead time: 14 days from code complete to submission.

---

*End of V2: App Store Compliance, Submission & Post-Launch Specification*

---

This document was produced using the following authoritative sources:

- [Apple Privacy Manifest Files Documentation](https://developer.apple.com/documentation/bundleresources/privacy-manifest-files)
- [Apple Required Reason API Documentation](https://developer.apple.com/documentation/bundleresources/describing-use-of-required-reason-api)
- [Apple TN3183: Adding Required Reason API Entries](https://developer.apple.com/documentation/technotes/tn3183-adding-required-reason-api-entries-to-your-privacy-manifest)
- [Expo Privacy Manifests Guide](https://docs.expo.dev/guides/apple-privacy/)
- [Sentry React Native Apple Privacy Manifest](https://docs.sentry.io/platforms/react-native/data-management/apple-privacy-manifest/)
- [Apple App Privacy Details](https://developer.apple.com/app-store/app-privacy-details/)
- [Apple User Privacy and Data Use](https://developer.apple.com/app-store/user-privacy-and-data-use/)
- [Apple App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [Apple Screenshot Specifications](https://developer.apple.com/help/app-store-connect/reference/app-information/screenshot-specifications/)
- [Apple Age Rating Values](https://developer.apple.com/help/app-store-connect/reference/app-information/age-ratings-values-and-definitions/)
- [Apple Account Deletion Requirement](https://developer.apple.com/support/offering-account-deletion-in-your-app/)
- [Apple Phased Release Documentation](https://developer.apple.com/help/app-store-connect/update-your-app/release-a-version-update-in-phases/)
- [Apple SKStoreReviewController](https://developer.apple.com/documentation/storekit/skstorereviewcontroller)
- [Google Sign-In iOS App Privacy](https://developers.google.com/identity/sign-in/ios/app-privacy)
- [React Native Privacy Manifest Discussion](https://github.com/react-native-community/discussions-and-proposals/discussions/766)
- [Expo EAS Updates Documentation](https://docs.expo.dev/eas-update/introduction/)

---


## Section 11: Testing Strategy, QA & Acceptance Criteria

---

### 1. Testing Pyramid

The Skye testing strategy follows a strict pyramid allocation: **60% unit tests** (fast, cheap, high coverage), **25% component/integration tests** (screen-level behavior), and **15% E2E tests** (critical user journeys). Every layer has explicit coverage targets, tooling choices, and concrete test descriptions.

---

#### 1.1 Unit Tests (60% of Test Effort)

**Tooling:** Jest 29+ with `ts-jest` preset, `@testing-library/react-native` for hook testing via `renderHook`.

**File convention:** Co-located. `src/lib/apiClient.ts` is tested by `src/lib/apiClient.test.ts`. `src/hooks/useContacts.ts` is tested by `src/hooks/useContacts.test.ts`. No separate `__tests__` directories.

**Coverage enforcement (enforced in CI via `jest --coverage --coverageThreshold`):**

| Layer | Target | Fail CI Below |
|-------|--------|---------------|
| `src/lib/**` (utilities, helpers, transformers) | 90% lines, 85% branches | 88% lines |
| `src/hooks/**` | 80% lines, 75% branches | 78% lines |
| `src/stores/**` (Zustand stores) | 70% lines, 65% branches | 68% lines |
| `src/components/**` | 70% lines | 65% lines |

**Jest configuration excerpt (`jest.config.ts`):**

```typescript
export default {
  preset: 'jest-expo',
  setupFilesAfterSetup: ['./jest.setup.ts'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@sentry/react-native|native-base|react-native-svg|react-native-reanimated)',
  ],
  coverageThreshold: {
    'src/lib/**': { lines: 88, branches: 83 },
    'src/hooks/**': { lines: 78, branches: 73 },
    'src/stores/**': { lines: 68, branches: 63 },
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
    '!src/**/index.ts',
  ],
};
```

---

##### 1.1.1 API Client Unit Tests (`src/lib/apiClient.test.ts`)

```typescript
describe('ApiClient', () => {
  describe('auth header injection', () => {
    it('attaches Bearer token from SecureStore to every request', async () => {
      // GIVEN a stored access token "test-token-abc"
      mockSecureStore.getItemAsync.mockResolvedValue('test-token-abc');
      // WHEN a GET request is made to /api/crm/contacts
      await apiClient.get('/api/crm/contacts');
      // THEN the Authorization header is "Bearer test-token-abc"
      expect(mockFetch).toHaveBeenCalledWith(
        expect.any(String),
        expect.objectContaining({
          headers: expect.objectContaining({
            Authorization: 'Bearer test-token-abc',
          }),
        })
      );
    });

    it('does NOT attach Authorization header when no token exists', async () => {
      mockSecureStore.getItemAsync.mockResolvedValue(null);
      await apiClient.get('/api/public/health');
      const callHeaders = mockFetch.mock.calls[0][1].headers;
      expect(callHeaders.Authorization).toBeUndefined();
    });
  });

  describe('token refresh trigger', () => {
    it('refreshes token on 401 response and retries the original request exactly once', async () => {
      // GIVEN the first call returns 401, the refresh succeeds, and the retry returns 200
      mockFetch
        .mockResolvedValueOnce({ status: 401, json: async () => ({ error: 'Unauthorized' }) })
        .mockResolvedValueOnce({ status: 200, json: async () => ({ accessToken: 'new-token' }) })
        .mockResolvedValueOnce({ status: 200, json: async () => ({ data: [] }) });
      // WHEN a GET request is made
      const result = await apiClient.get('/api/crm/contacts');
      // THEN the token refresh endpoint was called
      expect(mockFetch).toHaveBeenCalledWith(
        expect.stringContaining('/api/auth/refresh'),
        expect.any(Object)
      );
      // AND the original request was retried with the new token
      expect(mockFetch).toHaveBeenCalledTimes(3);
      expect(result.data).toEqual([]);
    });

    it('redirects to login screen when refresh token is also expired (401 on refresh)', async () => {
      mockFetch
        .mockResolvedValueOnce({ status: 401, json: async () => ({ error: 'Unauthorized' }) })
        .mockResolvedValueOnce({ status: 401, json: async () => ({ error: 'Refresh token expired' }) });
      await apiClient.get('/api/crm/contacts');
      expect(mockNavigation.reset).toHaveBeenCalledWith({
        index: 0,
        routes: [{ name: 'Auth' }],
      });
    });

    it('does NOT retry more than once on repeated 401 to prevent infinite loops', async () => {
      mockFetch.mockResolvedValue({ status: 401, json: async () => ({ error: 'Unauthorized' }) });
      await expect(apiClient.get('/api/crm/contacts')).rejects.toThrow('Authentication failed');
      // Original call + refresh + one retry = 3 max
      expect(mockFetch.mock.calls.length).toBeLessThanOrEqual(3);
    });
  });

  describe('error normalization', () => {
    it('normalizes network error to AppError with type "network"', async () => {
      mockFetch.mockRejectedValue(new TypeError('Network request failed'));
      try {
        await apiClient.get('/api/crm/contacts');
      } catch (e) {
        expect(e).toBeInstanceOf(AppError);
        expect(e.type).toBe('network');
        expect(e.userMessage).toBe('No internet connection. Please check your network.');
        expect(e.retryable).toBe(true);
      }
    });

    it('normalizes 500 response to AppError with type "server"', async () => {
      mockFetch.mockResolvedValue({
        status: 500,
        json: async () => ({ error: 'Internal server error' }),
      });
      try {
        await apiClient.get('/api/crm/contacts');
      } catch (e) {
        expect(e).toBeInstanceOf(AppError);
        expect(e.type).toBe('server');
        expect(e.userMessage).toBe('Something went wrong. Please try again.');
        expect(e.retryable).toBe(true);
      }
    });

    it('normalizes 422 validation error to AppError with field-level details', async () => {
      mockFetch.mockResolvedValue({
        status: 422,
        json: async () => ({
          error: 'Validation failed',
          details: [{ field: 'email', message: 'Invalid email format' }],
        }),
      });
      try {
        await apiClient.post('/api/crm/contacts', { email: 'bad' });
      } catch (e) {
        expect(e).toBeInstanceOf(AppError);
        expect(e.type).toBe('validation');
        expect(e.fieldErrors).toEqual({ email: 'Invalid email format' });
        expect(e.retryable).toBe(false);
      }
    });

    it('normalizes 429 rate limit to AppError with retryAfter', async () => {
      mockFetch.mockResolvedValue({
        status: 429,
        headers: { get: (key: string) => key === 'Retry-After' ? '30' : null },
        json: async () => ({ error: 'Rate limited' }),
      });
      try {
        await apiClient.get('/api/chat/send');
      } catch (e) {
        expect(e).toBeInstanceOf(AppError);
        expect(e.type).toBe('rateLimit');
        expect(e.retryAfterMs).toBe(30000);
      }
    });
  });

  describe('timeout handling', () => {
    it('aborts request after 15000ms and throws timeout AppError', async () => {
      mockFetch.mockImplementation(
        () => new Promise((resolve) => setTimeout(resolve, 20000))
      );
      jest.useFakeTimers();
      const promise = apiClient.get('/api/crm/contacts');
      jest.advanceTimersByTime(15001);
      await expect(promise).rejects.toMatchObject({
        type: 'timeout',
        userMessage: 'Request timed out. Please try again.',
        retryable: true,
      });
      jest.useRealTimers();
    });

    it('uses custom timeout of 60000ms for chat send endpoint', async () => {
      mockFetch.mockImplementation(
        () => new Promise((resolve) => setTimeout(resolve, 30000))
      );
      jest.useFakeTimers();
      const promise = apiClient.post('/api/chat/send', { message: 'hello' });
      jest.advanceTimersByTime(30000);
      // Should NOT have timed out yet (custom timeout is 60s)
      await expect(Promise.race([promise, Promise.resolve('still-pending')])).resolves.toBe('still-pending');
      jest.useRealTimers();
    });
  });
});
```

---

##### 1.1.2 Offline Queue Unit Tests (`src/lib/offlineQueue.test.ts`)

```typescript
describe('OfflineQueue', () => {
  let queue: OfflineQueue;

  beforeEach(() => {
    queue = new OfflineQueue({ maxRetries: 3, retryDelayMs: 1000, maxQueueSize: 100 });
    mockAsyncStorage.clear();
  });

  describe('enqueue', () => {
    it('adds an operation to the queue with timestamp and incremented ID', async () => {
      await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '1', name: 'Updated' } });
      const items = await queue.getAll();
      expect(items).toHaveLength(1);
      expect(items[0]).toMatchObject({
        id: expect.any(String),
        type: 'UPDATE_CONTACT',
        payload: { id: '1', name: 'Updated' },
        createdAt: expect.any(Number),
        retryCount: 0,
        status: 'pending',
      });
    });

    it('persists queue to AsyncStorage so it survives app restart', async () => {
      await queue.enqueue({ type: 'CREATE_NOTE', payload: { contactId: '1', text: 'Test' } });
      const stored = JSON.parse(await mockAsyncStorage.getItem('offline_queue'));
      expect(stored).toHaveLength(1);
    });

    it('rejects enqueue when queue is at maxQueueSize (100) to prevent memory exhaustion', async () => {
      for (let i = 0; i < 100; i++) {
        await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: String(i) } });
      }
      await expect(
        queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '101' } })
      ).rejects.toThrow('Queue is full');
    });

    it('maintains FIFO order for operations on the same entity', async () => {
      await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '1', name: 'First' } });
      await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '1', name: 'Second' } });
      const items = await queue.getAll();
      expect(items[0].payload.name).toBe('First');
      expect(items[1].payload.name).toBe('Second');
    });
  });

  describe('dequeue and process', () => {
    it('processes the oldest pending item first (FIFO)', async () => {
      await queue.enqueue({ type: 'A', payload: {} });
      await queue.enqueue({ type: 'B', payload: {} });
      const next = await queue.dequeue();
      expect(next.type).toBe('A');
    });

    it('marks item as "processing" during execution to prevent duplicate processing', async () => {
      await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '1' } });
      const processPromise = queue.processNext(mockApiClient);
      const items = await queue.getAll();
      expect(items[0].status).toBe('processing');
      await processPromise;
    });

    it('removes item from queue on successful API call', async () => {
      mockApiClient.execute.mockResolvedValue({ status: 200 });
      await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '1' } });
      await queue.processNext(mockApiClient);
      expect(await queue.getAll()).toHaveLength(0);
    });
  });

  describe('retry logic', () => {
    it('increments retryCount on failure and returns item to pending status', async () => {
      mockApiClient.execute.mockRejectedValue(new Error('Network error'));
      await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '1' } });
      await queue.processNext(mockApiClient);
      const items = await queue.getAll();
      expect(items[0].retryCount).toBe(1);
      expect(items[0].status).toBe('pending');
    });

    it('applies exponential backoff: delay = baseDelay * 2^retryCount', async () => {
      const item = { type: 'UPDATE_CONTACT', payload: { id: '1' }, retryCount: 2 };
      const delay = queue.calculateRetryDelay(item);
      expect(delay).toBe(1000 * Math.pow(2, 2)); // 4000ms
    });

    it('caps retry delay at 30000ms regardless of retry count', () => {
      const item = { type: 'UPDATE_CONTACT', payload: { id: '1' }, retryCount: 10 };
      const delay = queue.calculateRetryDelay(item);
      expect(delay).toBeLessThanOrEqual(30000);
    });
  });

  describe('dead letter handling', () => {
    it('moves item to dead letter queue after maxRetries (3) exhausted', async () => {
      mockApiClient.execute.mockRejectedValue(new Error('Server error'));
      await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '1' } });
      // Process 3 times (all fail)
      for (let i = 0; i < 3; i++) {
        await queue.processNext(mockApiClient);
      }
      const mainQueue = await queue.getAll();
      const deadLetters = await queue.getDeadLetters();
      expect(mainQueue).toHaveLength(0);
      expect(deadLetters).toHaveLength(1);
      expect(deadLetters[0].error).toBe('Server error');
    });

    it('emits "deadLetter" event when item is moved so UI can show a warning', async () => {
      const listener = jest.fn();
      queue.on('deadLetter', listener);
      mockApiClient.execute.mockRejectedValue(new Error('Permanent failure'));
      await queue.enqueue({ type: 'DELETE_NOTE', payload: { id: '99' } });
      for (let i = 0; i < 3; i++) {
        await queue.processNext(mockApiClient);
      }
      expect(listener).toHaveBeenCalledWith(
        expect.objectContaining({ type: 'DELETE_NOTE', payload: { id: '99' } })
      );
    });

    it('does NOT retry 422 validation errors — moves directly to dead letter', async () => {
      mockApiClient.execute.mockRejectedValue(
        new AppError({ type: 'validation', status: 422 })
      );
      await queue.enqueue({ type: 'UPDATE_CONTACT', payload: { id: '1', email: 'invalid' } });
      await queue.processNext(mockApiClient);
      const mainQueue = await queue.getAll();
      const deadLetters = await queue.getDeadLetters();
      expect(mainQueue).toHaveLength(0);
      expect(deadLetters).toHaveLength(1);
    });
  });
});
```

---

##### 1.1.3 Data Transformer Unit Tests (`src/lib/transformers.test.ts`)

```typescript
describe('transformApiContact → UIContact', () => {
  it('maps flat API fields to nested UI model', () => {
    const apiContact = {
      id: 'c_123',
      first_name: 'Jane',
      last_name: 'Doe',
      email_addresses: [{ value: 'jane@example.com', label: 'work' }],
      phone_numbers: [{ value: '+14155551234', label: 'mobile' }],
      engagement_score: 85,
      last_activity_at: '2025-12-01T10:00:00Z',
      tags: ['buyer', 'hot-lead'],
      custom_fields: { budget: '$500K-$1M' },
    };
    const result = transformApiContact(apiContact);
    expect(result).toEqual({
      id: 'c_123',
      name: { first: 'Jane', last: 'Doe', display: 'Jane Doe', initials: 'JD' },
      emails: [{ value: 'jane@example.com', label: 'work', isPrimary: true }],
      phones: [{ value: '+14155551234', formatted: '(415) 555-1234', label: 'mobile', isPrimary: true }],
      engagement: { score: 85, level: 'hot', dotColor: '#FF3B30' },
      lastActivity: { timestamp: expect.any(Date), relative: expect.any(String) },
      tags: ['buyer', 'hot-lead'],
      customFields: { budget: '$500K-$1M' },
    });
  });

  it('handles missing optional fields without throwing', () => {
    const minimalContact = { id: 'c_456', first_name: 'Bob', last_name: null };
    const result = transformApiContact(minimalContact);
    expect(result.name.display).toBe('Bob');
    expect(result.name.initials).toBe('B');
    expect(result.emails).toEqual([]);
    expect(result.phones).toEqual([]);
    expect(result.engagement.level).toBe('cold');
  });

  it('maps engagement score brackets correctly', () => {
    expect(transformApiContact({ ...base, engagement_score: 95 }).engagement.level).toBe('hot');
    expect(transformApiContact({ ...base, engagement_score: 70 }).engagement.level).toBe('warm');
    expect(transformApiContact({ ...base, engagement_score: 40 }).engagement.level).toBe('cool');
    expect(transformApiContact({ ...base, engagement_score: 10 }).engagement.level).toBe('cold');
    expect(transformApiContact({ ...base, engagement_score: 0 }).engagement.level).toBe('cold');
    expect(transformApiContact({ ...base, engagement_score: null }).engagement.level).toBe('unknown');
  });

  it('formats US phone numbers with correct grouping', () => {
    const contact = { ...base, phone_numbers: [{ value: '+12125551234' }] };
    expect(transformApiContact(contact).phones[0].formatted).toBe('(212) 555-1234');
  });

  it('preserves international phone numbers as-is when non-US', () => {
    const contact = { ...base, phone_numbers: [{ value: '+442071234567' }] };
    expect(transformApiContact(contact).phones[0].formatted).toBe('+44 207 123 4567');
  });
});

describe('transformApiProperty → UIProperty', () => {
  it('maps all property fields including computed metrics', () => {
    const apiProperty = {
      id: 'p_789',
      address: { street: '123 Main St', city: 'San Francisco', state: 'CA', zip: '94105' },
      list_price: 1250000,
      bedrooms: 3,
      bathrooms: 2,
      square_feet: 1800,
      lot_size_sqft: 3200,
      year_built: 1925,
      status: 'active',
      days_on_market: 14,
      mls_id: 'MLS12345',
      photos: ['https://cdn.example.com/p1.jpg'],
      latitude: 37.7849,
      longitude: -122.4094,
    };
    const result = transformApiProperty(apiProperty);
    expect(result.priceFormatted).toBe('$1,250,000');
    expect(result.pricePerSqFt).toBe('$694');
    expect(result.addressOneLine).toBe('123 Main St, San Francisco, CA 94105');
    expect(result.streetViewUrl).toContain('maps.googleapis.com/maps/api/streetview');
    expect(result.streetViewUrl).toContain('37.7849');
  });

  it('handles missing lot_size and year_built gracefully', () => {
    const minimal = { ...base, lot_size_sqft: null, year_built: null };
    const result = transformApiProperty(minimal);
    expect(result.lotSize).toBeNull();
    expect(result.yearBuilt).toBeNull();
  });
});

describe('transformApiDeal → UIDeal', () => {
  it('computes deal stage progress percentage', () => {
    const deal = { ...baseDeal, stage: 'under_contract' };
    const result = transformApiDeal(deal);
    expect(result.stageProgress).toBe(0.75); // under_contract = 75%
  });

  it('formats deal value with comma separators', () => {
    const deal = { ...baseDeal, value: 2500000 };
    expect(transformApiDeal(deal).valueFormatted).toBe('$2,500,000');
  });
});
```

---

##### 1.1.4 Chat/SSE Parser Unit Tests (`src/lib/chatParser.test.ts`)

```typescript
describe('SSE Parser', () => {
  it('parses a single "data:" line into a text token', () => {
    const result = parseSSEChunk('data: {"type":"text","content":"Hello"}\n\n');
    expect(result).toEqual([{ type: 'text', content: 'Hello' }]);
  });

  it('parses multiple events from a single chunk (buffered by network)', () => {
    const chunk = [
      'data: {"type":"text","content":"Hello "}',
      '',
      'data: {"type":"text","content":"world"}',
      '',
    ].join('\n');
    const result = parseSSEChunk(chunk);
    expect(result).toEqual([
      { type: 'text', content: 'Hello ' },
      { type: 'text', content: 'world' },
    ]);
  });

  it('handles partial chunk at buffer boundary without data loss', () => {
    const parser = new IncrementalSSEParser();
    parser.feed('data: {"type":"text","con');
    expect(parser.flush()).toEqual([]);
    parser.feed('tent":"Hello"}\n\n');
    expect(parser.flush()).toEqual([{ type: 'text', content: 'Hello' }]);
  });

  it('parses [DONE] sentinel and emits stream-end event', () => {
    const result = parseSSEChunk('data: [DONE]\n\n');
    expect(result).toEqual([{ type: 'done' }]);
  });

  it('ignores SSE comments (lines starting with ":")', () => {
    const chunk = ':keepalive\n\ndata: {"type":"text","content":"Hi"}\n\n';
    const result = parseSSEChunk(chunk);
    expect(result).toEqual([{ type: 'text', content: 'Hi' }]);
  });

  it('parses tool_result event with structured card data', () => {
    const chunk = 'data: {"type":"tool_result","tool":"property_search","data":{"results":[{"id":"p_1","address":"123 Main St","price":500000}]}}\n\n';
    const result = parseSSEChunk(chunk);
    expect(result[0]).toMatchObject({
      type: 'tool_result',
      tool: 'property_search',
      data: {
        results: [{ id: 'p_1', address: '123 Main St', price: 500000 }],
      },
    });
  });

  it('handles malformed JSON gracefully by emitting error event (no crash)', () => {
    const chunk = 'data: {broken json\n\n';
    const result = parseSSEChunk(chunk);
    expect(result).toEqual([{ type: 'error', message: expect.stringContaining('parse') }]);
  });
});

describe('Markdown Renderer Token Extraction', () => {
  it('converts **bold** to styled text segment', () => {
    const segments = parseMarkdownToSegments('This is **bold** text');
    expect(segments).toContainEqual({ type: 'bold', text: 'bold' });
  });

  it('converts inline `code` to styled text segment', () => {
    const segments = parseMarkdownToSegments('Use `npx expo start`');
    expect(segments).toContainEqual({ type: 'inlineCode', text: 'npx expo start' });
  });

  it('converts ```code blocks``` to code block segment with language', () => {
    const md = '```javascript\nconsole.log("hi");\n```';
    const segments = parseMarkdownToSegments(md);
    expect(segments).toContainEqual({
      type: 'codeBlock',
      language: 'javascript',
      code: 'console.log("hi");',
    });
  });

  it('converts numbered lists to ordered list segments', () => {
    const md = '1. First\n2. Second\n3. Third';
    const segments = parseMarkdownToSegments(md);
    expect(segments[0]).toMatchObject({ type: 'orderedList', items: ['First', 'Second', 'Third'] });
  });

  it('handles mixed markdown elements in a single response', () => {
    const md = '**Title**\n\nHere are properties:\n\n1. 123 Main St\n2. 456 Oak Ave\n\n> Note: prices are estimates';
    const segments = parseMarkdownToSegments(md);
    expect(segments.map((s) => s.type)).toEqual([
      'bold', 'text', 'orderedList', 'blockquote',
    ]);
  });
});
```

---

##### 1.1.5 Cache Manager Unit Tests (`src/lib/cacheManager.test.ts`)

```typescript
describe('CacheManager', () => {
  let cache: CacheManager;

  beforeEach(() => {
    cache = new CacheManager({ maxEntries: 5, defaultTTLMs: 60000 });
    jest.useFakeTimers();
  });

  afterEach(() => jest.useRealTimers());

  describe('LRU eviction', () => {
    it('evicts least recently used entry when maxEntries exceeded', () => {
      cache.set('a', 1);
      cache.set('b', 2);
      cache.set('c', 3);
      cache.set('d', 4);
      cache.set('e', 5);
      // Access 'a' to make it recently used
      cache.get('a');
      // Add one more, should evict 'b' (least recently used after 'a' was accessed)
      cache.set('f', 6);
      expect(cache.has('b')).toBe(false);
      expect(cache.has('a')).toBe(true); // 'a' was accessed, so not evicted
      expect(cache.has('f')).toBe(true);
    });

    it('updates access order on get (not just set)', () => {
      cache.set('a', 1);
      cache.set('b', 2);
      cache.set('c', 3);
      cache.set('d', 4);
      cache.set('e', 5);
      cache.get('a'); // 'a' is now most recently used
      cache.get('b'); // 'b' is now most recently used
      cache.set('f', 6); // Should evict 'c' (oldest untouched)
      expect(cache.has('c')).toBe(false);
      expect(cache.has('a')).toBe(true);
      expect(cache.has('b')).toBe(true);
    });
  });

  describe('staleness checks', () => {
    it('returns null for expired entries (TTL exceeded)', () => {
      cache.set('contacts', [{ id: 1 }], { ttlMs: 5000 });
      jest.advanceTimersByTime(5001);
      expect(cache.get('contacts')).toBeNull();
    });

    it('returns data for non-expired entries', () => {
      cache.set('contacts', [{ id: 1 }], { ttlMs: 5000 });
      jest.advanceTimersByTime(4999);
      expect(cache.get('contacts')).toEqual([{ id: 1 }]);
    });

    it('isStale() returns true when entry is past TTL but before eviction', () => {
      cache.set('contacts', [{ id: 1 }], { ttlMs: 5000 });
      jest.advanceTimersByTime(5001);
      expect(cache.isStale('contacts')).toBe(true);
    });

    it('getStale() returns expired data for stale-while-revalidate pattern', () => {
      cache.set('contacts', [{ id: 1 }], { ttlMs: 5000 });
      jest.advanceTimersByTime(5001);
      expect(cache.get('contacts')).toBeNull();
      expect(cache.getStale('contacts')).toEqual([{ id: 1 }]);
    });
  });

  describe('cache invalidation', () => {
    it('invalidate(key) removes specific entry', () => {
      cache.set('contacts', []);
      cache.set('properties', []);
      cache.invalidate('contacts');
      expect(cache.has('contacts')).toBe(false);
      expect(cache.has('properties')).toBe(true);
    });

    it('invalidatePattern(regex) removes all matching keys', () => {
      cache.set('contacts:list', []);
      cache.set('contacts:detail:1', {});
      cache.set('properties:list', []);
      cache.invalidatePattern(/^contacts:/);
      expect(cache.has('contacts:list')).toBe(false);
      expect(cache.has('contacts:detail:1')).toBe(false);
      expect(cache.has('properties:list')).toBe(true);
    });

    it('invalidateAll() clears entire cache', () => {
      cache.set('a', 1);
      cache.set('b', 2);
      cache.invalidateAll();
      expect(cache.size()).toBe(0);
    });
  });
});
```

---

##### 1.1.6 Navigation Helper Unit Tests (`src/lib/navigation.test.ts`)

```typescript
describe('Deep Link Parser', () => {
  it('parses skye://contact/c_123 to ContactDetail route with id param', () => {
    const route = parseDeepLink('skye://contact/c_123');
    expect(route).toEqual({ screen: 'ContactDetail', params: { contactId: 'c_123' } });
  });

  it('parses skye://property/p_456 to PropertyDetail route', () => {
    const route = parseDeepLink('skye://property/p_456');
    expect(route).toEqual({ screen: 'PropertyDetail', params: { propertyId: 'p_456' } });
  });

  it('parses skye://chat to Chat route with no params', () => {
    const route = parseDeepLink('skye://chat');
    expect(route).toEqual({ screen: 'Chat', params: {} });
  });

  it('parses skye://chat?prefill=Show+listings to Chat with prefill text', () => {
    const route = parseDeepLink('skye://chat?prefill=Show+listings');
    expect(route).toEqual({ screen: 'Chat', params: { prefillText: 'Show listings' } });
  });

  it('returns null for unknown deep link scheme', () => {
    expect(parseDeepLink('https://example.com')).toBeNull();
  });

  it('returns null for malformed deep link (missing entity ID)', () => {
    expect(parseDeepLink('skye://contact/')).toBeNull();
  });

  it('handles deep link with extra path segments gracefully', () => {
    expect(parseDeepLink('skye://contact/c_123/notes')).toEqual({
      screen: 'ContactDetail',
      params: { contactId: 'c_123', section: 'notes' },
    });
  });
});

describe('Route Resolution', () => {
  it('resolves push notification payload to correct navigation action', () => {
    const notificationPayload = {
      type: 'new_message',
      entityType: 'contact',
      entityId: 'c_123',
    };
    const action = resolveNotificationRoute(notificationPayload);
    expect(action).toEqual({
      type: 'NAVIGATE',
      screen: 'ContactDetail',
      params: { contactId: 'c_123', initialTab: 'timeline' },
    });
  });

  it('resolves deal notification to deal detail within contact', () => {
    const payload = { type: 'deal_update', entityType: 'deal', entityId: 'd_789', contactId: 'c_123' };
    const action = resolveNotificationRoute(payload);
    expect(action).toEqual({
      type: 'NAVIGATE',
      screen: 'ContactDetail',
      params: { contactId: 'c_123', initialTab: 'deals', highlightDealId: 'd_789' },
    });
  });

  it('returns fallback to Home for notification with unknown type', () => {
    const payload = { type: 'unknown_type', entityId: 'x_1' };
    const action = resolveNotificationRoute(payload);
    expect(action).toEqual({ type: 'NAVIGATE', screen: 'Home', params: {} });
  });
});
```

---

#### 1.2 Component Tests (25% of Test Effort)

**Tooling:** `@testing-library/react-native` (RNTL) v12+, `jest-expo` preset, `msw` for API mocking.

**Rule: Every screen component must be tested in 4 states.** This is enforced by a custom ESLint rule (`eslint-plugin-skye/require-four-states`) that flags any `*.screen.test.tsx` file missing `loading`, `success`, `empty`, or `error` in its `describe` blocks.

---

##### 1.2.1 ContactListItem Component Tests

```typescript
// src/components/ContactListItem.test.tsx
import { render, fireEvent, within } from '@testing-library/react-native';
import { ContactListItem } from './ContactListItem';
import { createMockContact } from '@test/factories/contact';

describe('ContactListItem', () => {
  const mockOnPress = jest.fn();
  const mockOnCall = jest.fn();
  const mockOnMessage = jest.fn();

  const defaultContact = createMockContact({
    name: { first: 'Jane', last: 'Doe', display: 'Jane Doe', initials: 'JD' },
    engagement: { score: 85, level: 'hot', dotColor: '#FF3B30' },
    lastActivity: { relative: '2 hours ago' },
  });

  afterEach(() => jest.clearAllMocks());

  describe('rendering', () => {
    it('renders contact display name', () => {
      const { getByText } = render(
        <ContactListItem contact={defaultContact} onPress={mockOnPress} />
      );
      expect(getByText('Jane Doe')).toBeTruthy();
    });

    it('renders initials avatar when no photo URL provided', () => {
      const { getByText } = render(
        <ContactListItem contact={{ ...defaultContact, photoUrl: null }} onPress={mockOnPress} />
      );
      expect(getByText('JD')).toBeTruthy();
    });

    it('renders photo avatar when photo URL is provided', () => {
      const contactWithPhoto = { ...defaultContact, photoUrl: 'https://cdn.example.com/photo.jpg' };
      const { getByTestId } = render(
        <ContactListItem contact={contactWithPhoto} onPress={mockOnPress} />
      );
      const avatar = getByTestId('contact-avatar-image');
      expect(avatar.props.source.uri).toBe('https://cdn.example.com/photo.jpg');
    });

    it('renders engagement dot with correct color for "hot" engagement', () => {
      const { getByTestId } = render(
        <ContactListItem contact={defaultContact} onPress={mockOnPress} />
      );
      const dot = getByTestId('engagement-dot');
      expect(dot.props.style).toMatchObject(
        expect.objectContaining({ backgroundColor: '#FF3B30' })
      );
    });

    it('renders engagement dot green (#34C759) for "warm" engagement', () => {
      const warmContact = createMockContact({
        engagement: { score: 70, level: 'warm', dotColor: '#34C759' },
      });
      const { getByTestId } = render(
        <ContactListItem contact={warmContact} onPress={mockOnPress} />
      );
      expect(getByTestId('engagement-dot').props.style).toMatchObject(
        expect.objectContaining({ backgroundColor: '#34C759' })
      );
    });

    it('renders engagement dot gray (#8E8E93) for "cold" engagement', () => {
      const coldContact = createMockContact({
        engagement: { score: 10, level: 'cold', dotColor: '#8E8E93' },
      });
      const { getByTestId } = render(
        <ContactListItem contact={coldContact} onPress={mockOnPress} />
      );
      expect(getByTestId('engagement-dot').props.style).toMatchObject(
        expect.objectContaining({ backgroundColor: '#8E8E93' })
      );
    });

    it('renders last activity time in subtitle', () => {
      const { getByText } = render(
        <ContactListItem contact={defaultContact} onPress={mockOnPress} />
      );
      expect(getByText('2 hours ago')).toBeTruthy();
    });
  });

  describe('interactions', () => {
    it('calls onPress with contact when tapped', () => {
      const { getByTestId } = render(
        <ContactListItem contact={defaultContact} onPress={mockOnPress} />
      );
      fireEvent.press(getByTestId('contact-list-item'));
      expect(mockOnPress).toHaveBeenCalledWith(defaultContact);
    });

    it('swipe left reveals call and message actions', async () => {
      const { getByTestId } = render(
        <ContactListItem
          contact={defaultContact}
          onPress={mockOnPress}
          onCall={mockOnCall}
          onMessage={mockOnMessage}
        />
      );
      // Simulate swipe (implementation-specific: may need to fireEvent on Swipeable)
      fireEvent(getByTestId('contact-list-item-swipeable'), 'onSwipeableOpen', 'right');
      expect(getByTestId('action-call')).toBeTruthy();
      expect(getByTestId('action-message')).toBeTruthy();
    });

    it('call action button triggers onCall with contact phone number', () => {
      const { getByTestId } = render(
        <ContactListItem
          contact={defaultContact}
          onPress={mockOnPress}
          onCall={mockOnCall}
        />
      );
      fireEvent(getByTestId('contact-list-item-swipeable'), 'onSwipeableOpen', 'right');
      fireEvent.press(getByTestId('action-call'));
      expect(mockOnCall).toHaveBeenCalledWith(defaultContact.phones[0].value);
    });
  });

  describe('accessibility', () => {
    it('has accessibilityLabel with full contact name and engagement status', () => {
      const { getByTestId } = render(
        <ContactListItem contact={defaultContact} onPress={mockOnPress} />
      );
      const item = getByTestId('contact-list-item');
      expect(item.props.accessibilityLabel).toBe('Jane Doe, hot lead, last active 2 hours ago');
    });

    it('has accessibilityRole="button"', () => {
      const { getByTestId } = render(
        <ContactListItem contact={defaultContact} onPress={mockOnPress} />
      );
      expect(getByTestId('contact-list-item').props.accessibilityRole).toBe('button');
    });
  });
});
```

---

##### 1.2.2 ChatMessage Component Tests

```typescript
// src/components/ChatMessage.test.tsx
describe('ChatMessage', () => {
  describe('text rendering', () => {
    it('renders plain text message', () => {
      const { getByText } = render(<ChatMessage message={createMockMessage({ text: 'Hello' })} />);
      expect(getByText('Hello')).toBeTruthy();
    });

    it('renders **bold** markdown as bold styled text', () => {
      const { getByText } = render(
        <ChatMessage message={createMockMessage({ text: 'This is **important**' })} />
      );
      const boldText = getByText('important');
      expect(boldText.props.style).toMatchObject(expect.objectContaining({ fontWeight: 'bold' }));
    });

    it('renders inline `code` with monospace font and gray background', () => {
      const { getByText } = render(
        <ChatMessage message={createMockMessage({ text: 'Run `npm start`' })} />
      );
      const codeText = getByText('npm start');
      expect(codeText.props.style).toMatchObject(
        expect.objectContaining({ fontFamily: expect.stringContaining('Mono') })
      );
    });

    it('renders multi-line code block with syntax highlighting container', () => {
      const text = '```javascript\nconst x = 1;\nconsole.log(x);\n```';
      const { getByTestId } = render(
        <ChatMessage message={createMockMessage({ text })} />
      );
      expect(getByTestId('code-block')).toBeTruthy();
      expect(getByTestId('code-block-language').props.children).toBe('javascript');
    });

    it('renders numbered lists with correct numbering', () => {
      const text = '1. First item\n2. Second item\n3. Third item';
      const { getByText } = render(
        <ChatMessage message={createMockMessage({ text })} />
      );
      expect(getByText('First item')).toBeTruthy();
      expect(getByText('Second item')).toBeTruthy();
      expect(getByText('Third item')).toBeTruthy();
    });
  });

  describe('tool result cards', () => {
    it('renders property search results as PropertyCard components', () => {
      const message = createMockMessage({
        toolResults: [{
          tool: 'property_search',
          data: {
            results: [
              { id: 'p_1', address: '123 Main St', price: 500000, beds: 3, baths: 2 },
              { id: 'p_2', address: '456 Oak Ave', price: 750000, beds: 4, baths: 3 },
            ],
          },
        }],
      });
      const { getAllByTestId } = render(<ChatMessage message={message} />);
      const cards = getAllByTestId(/^property-card-/);
      expect(cards).toHaveLength(2);
    });

    it('renders contact lookup result as ContactCard component', () => {
      const message = createMockMessage({
        toolResults: [{
          tool: 'contact_lookup',
          data: { contact: { id: 'c_1', name: 'Jane Doe', phone: '+14155551234' } },
        }],
      });
      const { getByTestId } = render(<ChatMessage message={message} />);
      expect(getByTestId('contact-card-c_1')).toBeTruthy();
    });

    it('renders "View Details" action button on tool result cards', () => {
      const message = createMockMessage({
        toolResults: [{
          tool: 'property_search',
          data: { results: [{ id: 'p_1', address: '123 Main St', price: 500000 }] },
        }],
      });
      const { getByText } = render(<ChatMessage message={message} />);
      expect(getByText('View Details')).toBeTruthy();
    });

    it('"View Details" navigates to correct detail screen', () => {
      const mockNavigate = jest.fn();
      jest.spyOn(Navigation, 'useNavigation').mockReturnValue({ navigate: mockNavigate });
      const message = createMockMessage({
        toolResults: [{
          tool: 'property_search',
          data: { results: [{ id: 'p_1', address: '123 Main St', price: 500000 }] },
        }],
      });
      const { getByText } = render(<ChatMessage message={message} />);
      fireEvent.press(getByText('View Details'));
      expect(mockNavigate).toHaveBeenCalledWith('PropertyDetail', { propertyId: 'p_1' });
    });
  });

  describe('message states', () => {
    it('renders sending indicator when status is "sending"', () => {
      const { getByTestId } = render(
        <ChatMessage message={createMockMessage({ status: 'sending' })} />
      );
      expect(getByTestId('sending-indicator')).toBeTruthy();
    });

    it('renders error state with retry button when status is "error"', () => {
      const mockRetry = jest.fn();
      const { getByTestId, getByText } = render(
        <ChatMessage message={createMockMessage({ status: 'error' })} onRetry={mockRetry} />
      );
      expect(getByText('Failed to send')).toBeTruthy();
      fireEvent.press(getByTestId('retry-button'));
      expect(mockRetry).toHaveBeenCalled();
    });

    it('renders streaming dots animation when message is streaming and text is empty', () => {
      const { getByTestId } = render(
        <ChatMessage message={createMockMessage({ text: '', isStreaming: true })} />
      );
      expect(getByTestId('streaming-indicator')).toBeTruthy();
    });
  });
});
```

---

##### 1.2.3 PropertyCard Component Tests

```typescript
// src/components/PropertyCard.test.tsx
describe('PropertyCard', () => {
  const mockProperty = createMockProperty({
    id: 'p_1',
    addressOneLine: '123 Main St, San Francisco, CA 94105',
    priceFormatted: '$1,250,000',
    beds: 3,
    baths: 2,
    sqft: 1800,
    photos: ['https://cdn.example.com/photo1.jpg'],
    streetViewUrl: 'https://maps.googleapis.com/maps/api/streetview?...',
    status: 'active',
    daysOnMarket: 14,
  });

  it('renders property image from photos array', () => {
    const { getByTestId } = render(<PropertyCard property={mockProperty} />);
    const image = getByTestId('property-image');
    expect(image.props.source.uri).toBe('https://cdn.example.com/photo1.jpg');
  });

  it('falls back to Street View image when photos array is empty', () => {
    const noPhotos = { ...mockProperty, photos: [] };
    const { getByTestId } = render(<PropertyCard property={noPhotos} />);
    expect(getByTestId('property-image').props.source.uri).toContain('streetview');
  });

  it('renders formatted address', () => {
    const { getByText } = render(<PropertyCard property={mockProperty} />);
    expect(getByText('123 Main St, San Francisco, CA 94105')).toBeTruthy();
  });

  it('renders formatted price', () => {
    const { getByText } = render(<PropertyCard property={mockProperty} />);
    expect(getByText('$1,250,000')).toBeTruthy();
  });

  it('renders bed/bath/sqft metrics', () => {
    const { getByText } = render(<PropertyCard property={mockProperty} />);
    expect(getByText('3 bed')).toBeTruthy();
    expect(getByText('2 bath')).toBeTruthy();
    expect(getByText('1,800 sqft')).toBeTruthy();
  });

  it('renders days on market badge', () => {
    const { getByText } = render(<PropertyCard property={mockProperty} />);
    expect(getByText('14 days')).toBeTruthy();
  });

  it('renders status badge with correct color for "active"', () => {
    const { getByTestId } = render(<PropertyCard property={mockProperty} />);
    const badge = getByTestId('status-badge');
    expect(badge.props.children).toBe('Active');
    expect(badge.props.style).toMatchObject(
      expect.objectContaining({ backgroundColor: expect.any(String) })
    );
  });

  it('calls onPress with property ID when tapped', () => {
    const onPress = jest.fn();
    const { getByTestId } = render(<PropertyCard property={mockProperty} onPress={onPress} />);
    fireEvent.press(getByTestId('property-card'));
    expect(onPress).toHaveBeenCalledWith('p_1');
  });
});
```

---

##### 1.2.4 SearchBar Component Tests

```typescript
// src/components/SearchBar.test.tsx
describe('SearchBar', () => {
  beforeEach(() => jest.useFakeTimers());
  afterEach(() => jest.useRealTimers());

  it('calls onSearch after 300ms debounce when user types', () => {
    const onSearch = jest.fn();
    const { getByTestId } = render(<SearchBar onSearch={onSearch} debounceMs={300} />);
    fireEvent.changeText(getByTestId('search-input'), 'Jane');
    expect(onSearch).not.toHaveBeenCalled();
    jest.advanceTimersByTime(300);
    expect(onSearch).toHaveBeenCalledWith('Jane');
    expect(onSearch).toHaveBeenCalledTimes(1);
  });

  it('resets debounce timer on each keystroke (only fires once after last keystroke)', () => {
    const onSearch = jest.fn();
    const { getByTestId } = render(<SearchBar onSearch={onSearch} debounceMs={300} />);
    fireEvent.changeText(getByTestId('search-input'), 'J');
    jest.advanceTimersByTime(200);
    fireEvent.changeText(getByTestId('search-input'), 'Ja');
    jest.advanceTimersByTime(200);
    fireEvent.changeText(getByTestId('search-input'), 'Jan');
    jest.advanceTimersByTime(300);
    expect(onSearch).toHaveBeenCalledTimes(1);
    expect(onSearch).toHaveBeenCalledWith('Jan');
  });

  it('shows clear button when input has text', () => {
    const { getByTestId, queryByTestId } = render(<SearchBar onSearch={jest.fn()} />);
    expect(queryByTestId('search-clear-button')).toBeNull();
    fireEvent.changeText(getByTestId('search-input'), 'test');
    expect(getByTestId('search-clear-button')).toBeTruthy();
  });

  it('clear button resets input text and calls onSearch with empty string', () => {
    const onSearch = jest.fn();
    const { getByTestId } = render(<SearchBar onSearch={onSearch} debounceMs={0} />);
    fireEvent.changeText(getByTestId('search-input'), 'test');
    fireEvent.press(getByTestId('search-clear-button'));
    expect(getByTestId('search-input').props.value).toBe('');
    expect(onSearch).toHaveBeenCalledWith('');
  });

  it('dismisses keyboard when onScrollBegin prop is triggered', () => {
    const dismissSpy = jest.spyOn(Keyboard, 'dismiss');
    const { getByTestId } = render(<SearchBar onSearch={jest.fn()} />);
    // Simulate parent scroll beginning
    fireEvent(getByTestId('search-input'), 'onBlur');
    expect(dismissSpy).toHaveBeenCalled();
  });

  it('has correct accessibility label', () => {
    const { getByTestId } = render(<SearchBar onSearch={jest.fn()} placeholder="Search contacts" />);
    expect(getByTestId('search-input').props.accessibilityLabel).toBe('Search contacts');
  });
});
```

---

##### 1.2.5 FilterChips Component Tests

```typescript
// src/components/FilterChips.test.tsx
describe('FilterChips', () => {
  const filters = [
    { key: 'buyer', label: 'Buyer' },
    { key: 'seller', label: 'Seller' },
    { key: 'hot', label: 'Hot Lead' },
    { key: 'cold', label: 'Cold Lead' },
  ];

  it('renders all filter chips', () => {
    const { getByText } = render(
      <FilterChips filters={filters} selected={[]} onSelectionChange={jest.fn()} />
    );
    filters.forEach((f) => expect(getByText(f.label)).toBeTruthy());
  });

  it('applies active visual style to selected chips', () => {
    const { getByTestId } = render(
      <FilterChips filters={filters} selected={['buyer']} onSelectionChange={jest.fn()} />
    );
    const buyerChip = getByTestId('filter-chip-buyer');
    expect(buyerChip.props.style).toMatchObject(
      expect.objectContaining({ backgroundColor: expect.stringMatching(/#|rgb/) })
    );
  });

  it('supports multi-select: tapping unselected chip adds it to selection', () => {
    const onChange = jest.fn();
    const { getByTestId } = render(
      <FilterChips filters={filters} selected={['buyer']} onSelectionChange={onChange} />
    );
    fireEvent.press(getByTestId('filter-chip-seller'));
    expect(onChange).toHaveBeenCalledWith(['buyer', 'seller']);
  });

  it('tapping selected chip removes it from selection', () => {
    const onChange = jest.fn();
    const { getByTestId } = render(
      <FilterChips filters={filters} selected={['buyer', 'seller']} onSelectionChange={onChange} />
    );
    fireEvent.press(getByTestId('filter-chip-buyer'));
    expect(onChange).toHaveBeenCalledWith(['seller']);
  });

  it('renders "Clear All" button when 1+ filters are selected', () => {
    const { getByText, queryByText } = render(
      <FilterChips filters={filters} selected={[]} onSelectionChange={jest.fn()} />
    );
    expect(queryByText('Clear All')).toBeNull();

    const { getByText: getByText2 } = render(
      <FilterChips filters={filters} selected={['buyer']} onSelectionChange={jest.fn()} />
    );
    expect(getByText2('Clear All')).toBeTruthy();
  });

  it('"Clear All" resets selection to empty array', () => {
    const onChange = jest.fn();
    const { getByText } = render(
      <FilterChips filters={filters} selected={['buyer', 'hot']} onSelectionChange={onChange} />
    );
    fireEvent.press(getByText('Clear All'));
    expect(onChange).toHaveBeenCalledWith([]);
  });
});
```

---

##### 1.2.6 InputBar Component Tests

```typescript
// src/components/InputBar.test.tsx
describe('InputBar', () => {
  it('renders in collapsed (single-line) state by default', () => {
    const { getByTestId } = render(<InputBar onSend={jest.fn()} />);
    const input = getByTestId('input-bar-text');
    expect(input.props.multiline).toBe(true);
    expect(input.props.style.maxHeight).toBeLessThanOrEqual(44); // Single line height
  });

  it('expands to multi-line when text exceeds one line', () => {
    const { getByTestId } = render(<InputBar onSend={jest.fn()} />);
    const input = getByTestId('input-bar-text');
    fireEvent.changeText(input, 'This is a very long message that should cause the input to expand to multiple lines for better readability');
    fireEvent(input, 'onContentSizeChange', {
      nativeEvent: { contentSize: { height: 88 } },
    });
    // After expansion, maxHeight should allow multi-line
    expect(input.props.style.maxHeight).toBeGreaterThan(44);
  });

  it('collapses back to single line when text is cleared', () => {
    const { getByTestId } = render(<InputBar onSend={jest.fn()} />);
    const input = getByTestId('input-bar-text');
    fireEvent.changeText(input, 'Some text');
    fireEvent.changeText(input, '');
    fireEvent(input, 'onContentSizeChange', {
      nativeEvent: { contentSize: { height: 22 } },
    });
    expect(input.props.style.maxHeight).toBeLessThanOrEqual(44);
  });

  it('send button is hidden when input is empty', () => {
    const { queryByTestId } = render(<InputBar onSend={jest.fn()} />);
    expect(queryByTestId('send-button')).toBeNull();
  });

  it('send button appears when input has non-whitespace text', () => {
    const { getByTestId, queryByTestId } = render(<InputBar onSend={jest.fn()} />);
    fireEvent.changeText(getByTestId('input-bar-text'), '   ');
    expect(queryByTestId('send-button')).toBeNull();
    fireEvent.changeText(getByTestId('input-bar-text'), 'Hello');
    expect(getByTestId('send-button')).toBeTruthy();
  });

  it('send button calls onSend with trimmed text and clears input', () => {
    const onSend = jest.fn();
    const { getByTestId } = render(<InputBar onSend={onSend} />);
    fireEvent.changeText(getByTestId('input-bar-text'), '  Hello world  ');
    fireEvent.press(getByTestId('send-button'));
    expect(onSend).toHaveBeenCalledWith('Hello world');
    expect(getByTestId('input-bar-text').props.value).toBe('');
  });

  it('attachment button triggers onAttachment callback', () => {
    const onAttachment = jest.fn();
    const { getByTestId } = render(<InputBar onSend={jest.fn()} onAttachment={onAttachment} />);
    fireEvent.press(getByTestId('attachment-button'));
    expect(onAttachment).toHaveBeenCalled();
  });

  it('disables send button and input while sending prop is true', () => {
    const { getByTestId } = render(<InputBar onSend={jest.fn()} isSending={true} />);
    fireEvent.changeText(getByTestId('input-bar-text'), 'Hello');
    expect(getByTestId('send-button').props.disabled).toBe(true);
    expect(getByTestId('input-bar-text').props.editable).toBe(false);
  });
});
```

---

##### 1.2.7 Screen-Level Component Tests (4-State Pattern)

Every screen follows this template. Here is the People screen as the canonical example:

```typescript
// src/screens/PeopleScreen.test.tsx
describe('PeopleScreen', () => {
  describe('loading state', () => {
    it('renders skeleton placeholders while contacts are loading', () => {
      mockUseContacts.mockReturnValue({ status: 'loading', data: null, error: null });
      const { getAllByTestId } = render(<PeopleScreen />);
      expect(getAllByTestId(/^skeleton-contact-item-/)).toHaveLength(10);
    });

    it('does not render search bar during initial load', () => {
      mockUseContacts.mockReturnValue({ status: 'loading', data: null, error: null });
      const { queryByTestId } = render(<PeopleScreen />);
      expect(queryByTestId('search-bar')).toBeNull();
    });
  });

  describe('success state', () => {
    it('renders FlashList with all contacts', () => {
      mockUseContacts.mockReturnValue({
        status: 'success',
        data: createMockContacts(50),
        error: null,
      });
      const { getByTestId } = render(<PeopleScreen />);
      expect(getByTestId('contacts-flash-list')).toBeTruthy();
    });

    it('renders search bar above the list', () => {
      mockUseContacts.mockReturnValue({
        status: 'success',
        data: createMockContacts(5),
        error: null,
      });
      const { getByTestId } = render(<PeopleScreen />);
      expect(getByTestId('search-bar')).toBeTruthy();
    });

    it('renders filter chips below search bar', () => {
      mockUseContacts.mockReturnValue({
        status: 'success',
        data: createMockContacts(5),
        error: null,
      });
      const { getByTestId } = render(<PeopleScreen />);
      expect(getByTestId('filter-chips')).toBeTruthy();
    });

    it('renders section headers with alphabetical grouping', () => {
      const contacts = [
        createMockContact({ name: { display: 'Alice Adams' } }),
        createMockContact({ name: { display: 'Bob Brown' } }),
      ];
      mockUseContacts.mockReturnValue({ status: 'success', data: contacts, error: null });
      const { getByText } = render(<PeopleScreen />);
      expect(getByText('A')).toBeTruthy();
      expect(getByText('B')).toBeTruthy();
    });
  });

  describe('empty state', () => {
    it('renders empty state illustration and message when no contacts exist', () => {
      mockUseContacts.mockReturnValue({ status: 'success', data: [], error: null });
      const { getByText, getByTestId } = render(<PeopleScreen />);
      expect(getByTestId('empty-state-illustration')).toBeTruthy();
      expect(getByText('No contacts yet')).toBeTruthy();
    });

    it('renders "Add Contact" CTA button in empty state', () => {
      mockUseContacts.mockReturnValue({ status: 'success', data: [], error: null });
      const { getByText } = render(<PeopleScreen />);
      expect(getByText('Add Contact')).toBeTruthy();
    });

    it('renders empty search results state when search returns no matches', () => {
      mockUseContacts.mockReturnValue({ status: 'success', data: [], error: null });
      const { getByText, getByTestId } = render(<PeopleScreen searchQuery="xyznotfound" />);
      expect(getByText('No contacts match "xyznotfound"')).toBeTruthy();
    });
  });

  describe('error state', () => {
    it('renders error message and retry button on API failure', () => {
      mockUseContacts.mockReturnValue({
        status: 'error',
        data: null,
        error: new AppError({ type: 'server', userMessage: 'Something went wrong' }),
      });
      const { getByText, getByTestId } = render(<PeopleScreen />);
      expect(getByText('Something went wrong')).toBeTruthy();
      expect(getByTestId('retry-button')).toBeTruthy();
    });

    it('retry button triggers data refetch', () => {
      const refetch = jest.fn();
      mockUseContacts.mockReturnValue({
        status: 'error',
        data: null,
        error: new AppError({ type: 'network' }),
        refetch,
      });
      const { getByTestId } = render(<PeopleScreen />);
      fireEvent.press(getByTestId('retry-button'));
      expect(refetch).toHaveBeenCalledTimes(1);
    });

    it('shows offline banner with cached data when network is unavailable', () => {
      mockUseContacts.mockReturnValue({
        status: 'error',
        data: createMockContacts(10), // Stale cached data
        error: new AppError({ type: 'network' }),
      });
      const { getByTestId, getByText } = render(<PeopleScreen />);
      expect(getByTestId('offline-banner')).toBeTruthy();
      expect(getByText("You're offline. Showing cached data.")).toBeTruthy();
      // List should still render with stale data
      expect(getByTestId('contacts-flash-list')).toBeTruthy();
    });

    it('NEVER renders a blank screen — always shows either content, error, or skeleton', () => {
      // Test with every possible state combination
      const states = [
        { status: 'loading', data: null, error: null },
        { status: 'success', data: [], error: null },
        { status: 'success', data: createMockContacts(1), error: null },
        { status: 'error', data: null, error: new AppError({ type: 'server' }) },
        { status: 'error', data: createMockContacts(5), error: new AppError({ type: 'network' }) },
      ];
      states.forEach((state) => {
        mockUseContacts.mockReturnValue(state);
        const { container } = render(<PeopleScreen />);
        // Container should have at least one visible child
        expect(container.children.length).toBeGreaterThan(0);
      });
    });
  });
});
```

The same 4-state pattern applies to: `ChatScreen.test.tsx`, `PropertiesScreen.test.tsx`, `ContactDetailScreen.test.tsx`, `StudioScreen.test.tsx`, `HomeScreen.test.tsx`.

---

#### 1.3 End-to-End Tests (15% of Test Effort)

**Tooling:** Maestro (version 1.36+). YAML-based, reliable on React Native, supports visual assertions.

**CI execution:** Run against staging backend (`staging.skye-app.com`) with seeded test data. Triggered on every PR merge to `main` and nightly at 02:00 UTC.

---

##### 1.3.1 E2E: Auth Flow

```yaml
# e2e/auth-flow.yaml
appId: com.skye.crm
name: "Auth Flow - Sign in with Google and verify home screen"
tags:
  - critical
  - auth
---
- launchApp:
    clearState: true
    clearKeychain: true
- assertVisible: "Sign in with Google"
- tapOn: "Sign in with Google"
# For CI: use test Google account via Maestro's pre-configured auth
- waitForAnimationToEnd
- runFlow:
    when:
      visible: "Choose an account"
    commands:
      - tapOn: "skyetest@gmail.com"
- waitForAnimationToEnd
- assertVisible: "Good morning"  # Home screen greeting
- assertVisible:
    id: "user-avatar"
- assertVisible:
    id: "bottom-tab-bar"
- assertVisible: "People"
- assertVisible: "Chat"
# Verify token persistence: kill and relaunch
- stopApp
- launchApp:
    clearState: false
- assertNotVisible: "Sign in with Google"
- assertVisible: "Good morning"  # Should auto-login

# Edge case: Cancel OAuth
- launchApp:
    clearState: true
    clearKeychain: true
- tapOn: "Sign in with Google"
- waitForAnimationToEnd
- runFlow:
    when:
      visible: "Choose an account"
    commands:
      - pressKey: back
- assertVisible: "Sign in with Google"  # Returns to auth screen cleanly
```

---

##### 1.3.2 E2E: Chat Flow

```yaml
# e2e/chat-flow.yaml
appId: com.skye.crm
name: "Chat Flow - Send message, receive streaming response with tool result"
tags:
  - critical
  - chat
---
- launchApp
- tapOn:
    id: "bottom-tab-chat"
- assertVisible:
    id: "chat-screen"
- tapOn:
    id: "input-bar-text"
- inputText: "Show me properties in San Francisco under $1M"
- assertVisible:
    id: "send-button"
- tapOn:
    id: "send-button"
# Verify user message appears
- assertVisible: "Show me properties in San Francisco under $1M"
# Wait for streaming response (first token within 2s)
- waitForAnimationToEnd:
    timeout: 5000
- assertVisible:
    id: "assistant-message"
    timeout: 5000
# Verify tool result card appears
- assertVisible:
    id: "tool-result-property-search"
    timeout: 10000
# Verify property card data
- assertVisible:
    text: ".*San Francisco.*"
    regex: true
- assertVisible:
    text: ".*\\$.*"
    regex: true
# Tap "View Details" on first property result
- tapOn:
    text: "View Details"
    index: 0
- assertVisible:
    id: "property-detail-screen"
- pressKey: back
- assertVisible:
    id: "chat-screen"
```

---

##### 1.3.3 E2E: Contact CRUD

```yaml
# e2e/contact-crud.yaml
appId: com.skye.crm
name: "Contact CRUD - Search, view, edit phone, verify persistence"
tags:
  - critical
  - crm
---
- launchApp
- tapOn:
    id: "bottom-tab-people"
- assertVisible:
    id: "contacts-flash-list"
# Search for test contact
- tapOn:
    id: "search-bar"
- inputText: "Jane Testuser"
- waitForAnimationToEnd
- assertVisible: "Jane Testuser"
- tapOn: "Jane Testuser"
# Verify contact detail loads
- assertVisible:
    id: "contact-detail-screen"
- assertVisible: "Jane Testuser"
# Edit phone number
- scrollUntilVisible:
    element:
      id: "edit-button"
    direction: DOWN
- tapOn:
    id: "edit-button"
- tapOn:
    id: "phone-field-0"
- clearText
- inputText: "+14155559999"
- tapOn: "Save"
- waitForAnimationToEnd
# Verify change persisted
- assertVisible: "(415) 555-9999"
# Navigate away and back to verify persistence
- pressKey: back
- tapOn:
    id: "search-bar"
- clearText
- inputText: "Jane Testuser"
- tapOn: "Jane Testuser"
- assertVisible: "(415) 555-9999"
# Reset test data (update phone back to original)
- tapOn:
    id: "edit-button"
- tapOn:
    id: "phone-field-0"
- clearText
- inputText: "+14155551234"
- tapOn: "Save"
```

---

##### 1.3.4 E2E: Camera OCR Flow

```yaml
# e2e/ocr-flow.yaml
appId: com.skye.crm
name: "OCR Flow - Scan business card, review, create contact"
tags:
  - critical
  - ocr
---
- launchApp
- tapOn:
    id: "bottom-tab-people"
- tapOn:
    id: "add-contact-button"
- tapOn: "Scan Business Card"
# On CI: use test image injection via Maestro's camera mocking
- runFlow:
    env:
      MAESTRO_MOCK_CAMERA: "fixtures/business-card-sample.jpg"
    commands:
      - assertVisible:
          id: "camera-viewfinder"
      - tapOn:
          id: "capture-button"
- waitForAnimationToEnd:
    timeout: 10000
# Review extracted data
- assertVisible:
    id: "ocr-review-screen"
- assertVisible: "John Smith"     # Extracted name
- assertVisible: "Acme Realty"    # Extracted company
- assertVisible:
    text: ".*555.*"               # Extracted phone (partial match)
    regex: true
# Confirm contact creation
- tapOn: "Confirm"
- waitForAnimationToEnd
- assertVisible:
    id: "contact-detail-screen"
- assertVisible: "John Smith"
# Clean up: delete test contact
- scrollUntilVisible:
    element:
      text: "Delete Contact"
    direction: DOWN
- tapOn: "Delete Contact"
- tapOn: "Delete"  # Confirm dialog
```

---

##### 1.3.5 E2E: Push Notification Deep Link

```yaml
# e2e/push-deep-link.yaml
appId: com.skye.crm
name: "Push Deep Link - Tap notification, navigate to correct screen"
tags:
  - critical
  - notifications
---
- launchApp
# Send test notification via staging API
- runScript:
    file: "scripts/send-test-notification.js"
    env:
      NOTIFICATION_TYPE: "new_message"
      ENTITY_TYPE: "contact"
      ENTITY_ID: "c_test_123"
      TITLE: "New message from Jane Testuser"
      BODY: "Hey, I'm interested in the listing on Main St"
# Pull down notification shade
- swipe:
    direction: DOWN
    from:
      x: 50%
      y: 0%
    to:
      x: 50%
      y: 50%
- assertVisible: "New message from Jane Testuser"
- tapOn: "New message from Jane Testuser"
# Verify correct screen opens
- assertVisible:
    id: "contact-detail-screen"
- assertVisible: "Jane Testuser"
```

---

##### 1.3.6 E2E: Offline Flow

```yaml
# e2e/offline-flow.yaml
appId: com.skye.crm
name: "Offline Flow - Browse cached data offline, refresh on reconnect"
tags:
  - critical
  - offline
---
- launchApp
# Load contacts while online to populate cache
- tapOn:
    id: "bottom-tab-people"
- assertVisible:
    id: "contacts-flash-list"
- scroll:
    direction: DOWN
    distance: 200
# Enable airplane mode
- toggleAirplaneMode
- wait: 2000
# Verify offline banner appears
- assertVisible:
    id: "offline-banner"
# Verify cached contacts are still visible
- assertVisible:
    id: "contacts-flash-list"
- scroll:
    direction: UP
    distance: 200
# Search should work against cached data
- tapOn:
    id: "search-bar"
- inputText: "Jane"
- assertVisible: "Jane Testuser"
# Disable airplane mode
- toggleAirplaneMode
- wait: 3000
# Verify offline banner disappears
- assertNotVisible:
    id: "offline-banner"
    timeout: 10000
# Verify data refreshes (pull to refresh)
- scroll:
    direction: DOWN
    from:
      x: 50%
      y: 20%
    to:
      x: 50%
      y: 80%
- waitForAnimationToEnd
```

---

### 2. Acceptance Test Suite

Each acceptance criterion is mapped to concrete, executable test cases with pass/fail criteria, edge cases, and automation designation.

---

#### AC1: App Builds and Runs on Simulator

| Dimension | Specification |
|-----------|--------------|
| **Test ID** | AC1-BUILD |
| **Method** | CI automated |
| **Command** | `npx expo run:ios --configuration Release --device "iPhone 15 Pro"` |
| **Pass Criteria** | Exit code 0, no TypeScript errors, no build warnings treated as errors |
| **Timing** | Build completes in <5 minutes on CI (M1 Mac runner) |

| Dimension | Specification |
|-----------|--------------|
| **Test ID** | AC1-RENDER |
| **Method** | Maestro E2E |
| **Script** | `e2e/ac1-first-render.yaml` |
| **Pass Criteria** | App renders first interactive screen within 5000ms of launch (measured via `launchApp` + `assertVisible` with timeout) |

| Dimension | Specification |
|-----------|--------------|
| **Test ID** | AC1-NO-CRASH |
| **Method** | Maestro E2E |
| **Script** | `e2e/ac1-stability.yaml` — Navigate through every tab (Home, People, Chat, Properties, Studio), no crash within 60 seconds of navigation |
| **Pass Criteria** | App remains responsive, no `ANR` or crash log generated |

---

#### AC2: Google OAuth Authentication

| Test ID | Scenario | Steps | Pass Criteria | Automation |
|---------|----------|-------|---------------|------------|
| AC2-HAPPY | Successful sign-in | Tap "Sign in with Google" > complete Google OAuth with test account > verify home screen | Home screen visible with user name, SecureStore contains valid token, `/api/crm/contacts` returns 200 | E2E: `e2e/auth-flow.yaml` |
| AC2-CANCEL | User cancels OAuth | Tap "Sign in with Google" > tap back/cancel on Google screen | App returns to auth screen, no error shown, no token in SecureStore | E2E: `e2e/auth-flow.yaml` (cancel branch) |
| AC2-NETWORK | Network error during OAuth | Disconnect network before completing OAuth | Error message: "Unable to connect. Check your network and try again." Retry button visible. | Component test: `AuthScreen.test.tsx` (error state) |
| AC2-TOKEN-PERSIST | Token survives app restart | Sign in > kill app > relaunch | App skips auth, shows home screen directly | E2E: `e2e/auth-flow.yaml` (persistence check) |
| AC2-TOKEN-REFRESH | Token refresh on 401 | Wait for token expiry (or mock) > make API call | Token is silently refreshed, API call succeeds, user sees no interruption | Unit test: `apiClient.test.ts` (token refresh) |
| AC2-SIGNOUT | Sign out clears all data | Tap profile > sign out > confirm | SecureStore cleared, cache cleared, app returns to auth screen, back button does not return to authenticated screens | E2E: `e2e/signout-flow.yaml` |

---

#### AC3: Chat Streaming Quality

| Test ID | Scenario | Steps | Pass Criteria | Automation |
|---------|----------|-------|---------------|------------|
| AC3-FIRST-TOKEN | Response latency | Send "hello" | First token appears within 2000ms of send (measured from send button tap to first assistant text visible) | E2E + performance measurement |
| AC3-COMPLETE | Full response integrity | Send "List 5 things to do before a home inspection" | Response contains numbered list, no dropped characters, complete sentences, matches web app output for same prompt | Manual comparison + E2E |
| AC3-MARKDOWN | Markdown rendering | Send "Show me a code example for calculating mortgage payments" | Bold text renders bold, code blocks have syntax highlighting, lists are properly numbered, links are tappable | Component test: `ChatMessage.test.tsx` |
| AC3-TOOL-CALL | Tool result card | Send "Find properties in Austin under $500k" | Tool result card appears with property data, "View Details" button navigates to property detail | E2E: `e2e/chat-flow.yaml` |
| AC3-LONG | Long response (2000+ tokens) | Send "Write a comprehensive market analysis for the San Francisco Bay Area real estate market" | Full response renders, scrolling is smooth (60fps), no memory spike >50MB above baseline | E2E + Instruments profiling |
| AC3-FPS | Rendering performance | Stream 100+ tokens | FPS during streaming >=55fps average measured via React Native Performance Monitor or Xcode Instruments Time Profiler | Manual: Instruments trace |
| AC3-ERROR | Stream interruption | Kill network mid-stream | Partial response preserved, error message appended: "Response interrupted. Tap to retry.", retry resends from original message | Component test + E2E |
| AC3-MULTI | Conversation continuity | Send 5 messages in sequence | All messages visible in scroll, context maintained between messages, assistant references earlier messages correctly | E2E |

---

#### AC4: People Screen (CRM Contact List)

| Test ID | Scenario | Steps | Pass Criteria | Automation |
|---------|----------|-------|---------------|------------|
| AC4-LOAD-500 | Large dataset performance | Load account with

---


## Section 12: Build Roadmap & Phased Implementation

**Document Version:** 2.0
**Last Updated:** 2026-03-02
**Status:** LOCKED — governs all development decisions

---

### 1. Refined Phase Structure

---

#### Phase 0: Pre-Development Setup

**Purpose:** Eliminate every "we should have done this first" moment. Zero code is written until Phase 0 passes its exit criteria.

**Entry Criteria:**
- Project has been greenlit by stakeholders
- At least one developer is allocated full-time
- Access to all required accounts (Apple Developer, Google Cloud, Supabase, Sentry) is available or in progress

**Task List:**

| # | Task | Depends On | Owner |
|---|------|-----------|-------|
| 0.1 | Activate Apple Developer Program membership (paid), create App ID with bundle identifier `com.skye.crm`, enable Push Notifications and Associated Domains capabilities | None | Lead |
| 0.2 | Google Cloud Console: create OAuth 2.0 iOS client ID, register redirect URI `com.googleusercontent.apps.{CLIENT_ID}:/oauthredirect`, create web client ID (for token exchange on backend) | None | Lead |
| 0.3 | Supabase: audit all RLS policies with a mobile-issued JWT (not a web session cookie); confirm `auth.uid()` resolves correctly from a bearer token; document any policies that assume web-only session format | None | Backend |
| 0.4 | Design system finalized: color palette (light + dark), typography scale (SF Pro or custom), icon set (SF Symbols or bundled), spacing scale (4px base), component inventory (buttons, cards, inputs, avatars, badges, chips, empty states, skeleton loaders) — delivered as a Figma library or equivalent, not loose mockups | None | Design |
| 0.5 | Lock all Architecture Decision Records (ADR-1 through ADR-5, see Section 3) — no revisiting during build | 0.4 | Lead + Team |
| 0.6 | Dev environment provisioned: Xcode 16+ installed, CocoaPods installed, Expo CLI + EAS CLI installed globally, at least one physical iOS device registered for development, Homebrew dependencies current | None | All Devs |
| 0.7 | Repository initialized: monorepo or standalone, `.gitignore`, `eslint` + `prettier` config, `tsconfig.json`, `app.json` skeleton, CI pipeline stub (GitHub Actions or equivalent), branch protection rules on `main` | 0.6 | Lead |
| 0.8 | Sentry project created, DSN obtained, invite team members | None | Lead |
| 0.9 | Create `DECISIONS.md` in repo root documenting every ADR with date, participants, rationale, and "reversibility cost" rating | 0.5 | Lead |
| 0.10 | Backend: implement `/api/health` endpoint (returns `{ "status": "ok", "timestamp": <ISO> }`, no auth required) | None | Backend |
| 0.11 | Backend: implement `/api/auth/native` endpoint (accepts Google auth code + code verifier from native PKCE flow, exchanges for Google tokens, creates/updates Supabase user, returns access + refresh tokens) | 0.2 | Backend |
| 0.12 | Backend: implement `/api/user/delete` endpoint (Apple requires in-app account deletion; cascades through all user data, returns confirmation) | None | Backend |
| 0.13 | Backend: audit all 129 API routes for JSON-only responses (no HTML error pages), correct `Content-Type` headers, and absence of `Origin` header requirements (native apps do not send Origin) | None | Backend |

**Exit Criteria:**
- Apple Developer portal shows App ID with Push Notifications enabled
- Google Cloud Console shows iOS OAuth client ID with correct redirect URI
- A test JWT issued by Supabase (simulating mobile auth) passes RLS checks against every table the app will query
- Design system Figma (or equivalent) is marked "ready for dev" with no open comments
- All 5 ADRs are documented in `DECISIONS.md` and signed off
- `npx expo doctor` passes on every developer's machine
- A physical iOS device can run `npx expo start --dev-client` successfully
- `/api/health`, `/api/auth/native`, and `/api/user/delete` endpoints are deployed and pass integration tests
- Backend audit report confirms all routes return JSON with correct headers

**Decision Gate:**
All ADR decisions are irrevocable after Phase 0 sign-off. Changing auth library, state management, or navigation approach after this point requires a formal change request with impact analysis.

**Deliverables:**
- Configured Apple Developer portal
- Configured Google Cloud Console
- Supabase RLS audit report
- Figma design system library
- `DECISIONS.md` with 5 locked ADRs
- Repository with CI pipeline, linting, TypeScript config
- Three new backend endpoints deployed
- Backend JSON audit report

**Verification Checklist:**
- [ ] `curl https://api.skye.app/api/health` returns 200 with JSON body
- [ ] Google OAuth test flow completes on iOS device using registered redirect URI
- [ ] Supabase RLS test script passes with mobile-format JWT
- [ ] `npx expo doctor` shows no errors
- [ ] All team members can build to a physical device
- [ ] `DECISIONS.md` exists and contains 5 ADR entries

---

#### Phase 1: Foundation — Scaffold, Theming, Auth, API Client, Offline Storage, State Management

**Purpose:** Build the invisible infrastructure that every subsequent phase depends on. At the end of Phase 1, the app authenticates, fetches data, stores it offline, and renders a themed shell with tab navigation — but no real screens.

**Entry Criteria:**
- Phase 0 exit criteria fully met
- Design system tokens (colors, typography, spacing) exported and available
- `/api/auth/native` endpoint deployed and tested

**Task List:**

| # | Task | Depends On | Est. Hours |
|---|------|-----------|-----------|
| 1.1 | Initialize Expo project with TypeScript template, configure `app.json` (name, slug, bundle ID, scheme, version, icon placeholder, splash placeholder), install Hermes (default) | None | 2 |
| 1.2 | Install and configure React Navigation: create `RootNavigator` (stack), `AuthNavigator` (stack: Welcome, SignIn), `MainTabNavigator` (bottom tabs: Chat, People, Properties, Studio), and stub placeholder screens for each tab | 1.1 | 4 |
| 1.3 | Build theming system: create `theme.ts` with light and dark color tokens, typography scale, spacing scale, border radii; create `ThemeProvider` using React context; create `useTheme()` hook; wire `useColorScheme()` to auto-detect system preference; store user override in MMKV | 1.1 | 6 |
| 1.4 | Install `react-native-mmkv`, create `mmkvStorage.ts` utility with typed get/set helpers for: auth tokens, theme preference, onboarding state, last sync timestamps | 1.1 | 2 |
| 1.5 | Install `expo-secure-store`, create `secureStorage.ts` utility wrapping `setItemAsync` / `getItemAsync` / `deleteItemAsync`; store refresh tokens here (NOT in MMKV) | 1.1 | 2 |
| 1.6 | Install `expo-sqlite`, create database initialization module: `db.ts` opens database, runs migration scripts on app start; create migration system (version table, sequential migration files); create initial schema: `contacts`, `properties`, `conversations`, `messages`, `sync_meta` tables | 1.1 | 8 |
| 1.7 | Install and configure `@tanstack/react-query`: create `queryClient.ts` with defaults (staleTime: 5 min, gcTime: 30 min, retry: 2, refetchOnWindowFocus: false, refetchOnReconnect: true); create `QueryProvider` wrapper; install `@tanstack/react-query-persist-client` with MMKV adapter for query cache persistence | 1.1, 1.4 | 4 |
| 1.8 | Install and configure `zustand`: create stores — `useAuthStore` (user, tokens, isAuthenticated, login/logout actions), `useUIStore` (theme, bottomSheetState, toastQueue), `useNetworkStore` (isConnected, lastConnectedAt) | 1.1 | 4 |
| 1.9 | Build API client: create `apiClient.ts` using `fetch` (not axios — smaller bundle, native to Hermes); implement request interceptor that attaches bearer token from `useAuthStore`; implement response interceptor that catches 401, attempts token refresh via `/api/auth/refresh`, retries original request, or forces logout on failure; implement request queuing to prevent concurrent refresh races (mutex pattern) | 1.8, 1.5 | 8 |
| 1.10 | Build auth flow: implement Google Sign-In using `expo-auth-session` with PKCE; on successful Google auth, send auth code to `/api/auth/native`; receive Supabase tokens; store access token in Zustand (memory), refresh token in `expo-secure-store`; navigate to `MainTabNavigator`; implement logout (clear all stores, clear secure store, navigate to `AuthNavigator`) | 1.2, 1.8, 1.9, 1.5 | 10 |
| 1.11 | Build network connectivity monitor: use `@react-native-community/netinfo` to detect connectivity changes; update `useNetworkStore`; show a non-intrusive banner when offline; queue mutations when offline | 1.8 | 3 |
| 1.12 | Build error boundary: create `ErrorBoundary` component that catches render errors, logs to Sentry, shows "Something went wrong" screen with retry button; wrap each tab navigator independently (one tab crashing does not kill others) | 1.1 | 3 |
| 1.13 | Install and configure `@sentry/react-native`: wrap app with `Sentry.wrap()`, configure DSN, environment, release tracking; add Sentry navigation integration for screen tracking | 1.1, 1.2 | 3 |
| 1.14 | Build splash/loading screen: animated Skye logo during app initialization (auth check, database migration, cache hydration); transition to auth screen or main tabs based on stored auth state | 1.3, 1.10 | 4 |
| 1.15 | Create shared UI primitives: `<Text>` (themed, variant-based), `<Button>` (primary, secondary, ghost, destructive, loading state), `<Card>`, `<Avatar>`, `<Badge>`, `<SkeletonLoader>`, `<EmptyState>`, `<ErrorState>` — all with dark mode support, accessibility labels, and testIDs | 1.3 | 10 |
| 1.16 | Create 4-state screen wrapper: `<ScreenContainer>` component that accepts `isLoading`, `isError`, `isEmpty`, `data` props and renders the appropriate state (skeleton, error with retry, empty with CTA, or children); enforces the "every screen has 4 states" principle | 1.15, 1.12 | 4 |
| 1.17 | Configure EAS Build: create `eas.json` with `development`, `preview`, and `production` profiles; run first development build to physical device; confirm Hermes is active (check `global.HermesInternal`) | 1.1 | 3 |
| 1.18 | Write unit tests for: API client (token attachment, refresh flow, 401 handling), auth store (login, logout, token rotation), MMKV utilities, secure storage utilities, theme provider (light/dark switching) | 1.9, 1.10, 1.4, 1.5, 1.3 | 6 |
| 1.19 | Write first Maestro E2E flow: app launches, splash screen appears, transitions to sign-in screen, Google auth button is visible and tappable | 1.14, 1.17 | 3 |

**Exit Criteria:**
- App launches on a physical iOS device within 2 seconds
- User can sign in with Google, see the tab bar, and sign out
- Token refresh works when access token expires (tested with a short-lived token)
- Offline banner appears when device enters airplane mode
- Theme switches between light and dark (follows system and manual toggle)
- SQLite database is created with correct schema on first launch
- Sentry receives a test error event
- All unit tests pass
- Maestro E2E flow passes

**Decision Gate:**
Confirm that the auth flow works end-to-end with the production Supabase instance (not just local). Confirm API client handles all error codes (400, 401, 403, 404, 500) gracefully.

**Deliverables:**
- Functioning app shell with tab navigation
- Complete auth flow (sign in, sign out, token refresh)
- Theming system (light + dark)
- API client with interceptors
- Offline storage layer (SQLite + MMKV + SecureStore)
- State management (TanStack Query + Zustand)
- Error boundary + Sentry integration
- Shared UI component library
- Unit + E2E test suites

**Verification Checklist:**
- [ ] Cold start to interactive < 2 seconds (measured with Xcode Instruments)
- [ ] Sign in with Google completes without errors
- [ ] Manually expire access token; next API call triggers refresh and succeeds
- [ ] Sign out clears all persisted state (verify SecureStore, MMKV, SQLite)
- [ ] Toggle system appearance; app theme updates immediately
- [ ] Enable airplane mode; offline banner appears within 3 seconds
- [ ] Trigger a JS error; verify it appears in Sentry dashboard
- [ ] `npm test` passes with >80% coverage on foundation modules
- [ ] Maestro flow runs green on physical device

---

#### Phase 2: Chat Screen — SSE Streaming, Markdown, Tool Cards, Input Bar

**Purpose:** Build the most important and technically complex screen in the app. The chat screen is the core product experience — it must feel native, fast, and reliable.

**Entry Criteria:**
- Phase 1 exit criteria fully met
- API client with auth interceptors is working
- Backend chat/streaming endpoint is documented and accessible

**Task List:**

| # | Task | Depends On | Est. Hours |
|---|------|-----------|-----------|
| 2.1 | Create `ChatScreen` with `<ScreenContainer>` wrapper; implement 4 states: loading (skeleton chat bubbles), empty (welcome message + suggested prompts), error (retry button), success (message list) | None | 4 |
| 2.2 | Build `<MessageList>` using `FlashList` (from `@shopify/flash-list`) for virtualized performance; messages render from bottom; implement inverted list pattern; handle scroll-to-bottom on new messages; preserve scroll position when loading older messages | 2.1 | 8 |
| 2.3 | Build `<UserBubble>` component: right-aligned, themed background, text content, timestamp, sent/delivered/error status indicator | 2.1 | 3 |
| 2.4 | Build `<AssistantBubble>` component: left-aligned, themed background, supports streaming text (character-by-character append without re-render of entire bubble), typing indicator animation during initial response delay | 2.1 | 6 |
| 2.5 | Implement SSE streaming client: create `useStreamingChat()` hook using `react-native-sse`; connect to backend streaming endpoint with auth header; handle events: `message_start`, `content_delta`, `content_stop`, `tool_use`, `error`; implement reconnection with exponential backoff (1s, 2s, 4s, 8s, max 30s); handle mid-stream disconnection (preserve partial response, resume or restart); track connection state in Zustand | 1.9, 2.4 | 12 |
| 2.6 | Build markdown renderer for assistant messages: use `react-native-markdown-display` or custom renderer; support: bold, italic, code inline, code blocks (with syntax highlighting via `react-syntax-highlighter` or simpler approach), bulleted lists, numbered lists, headers, links (open in in-app browser), tables (basic); ensure all markdown elements are themed (light/dark) | 2.4 | 8 |
| 2.7 | Build `<ToolCard>` component system: when the assistant invokes a tool (contact lookup, property search, schedule event), render a structured card inside the chat; card types: `ContactCard` (avatar, name, phone, tap to navigate), `PropertyCard` (thumbnail, address, price, tap to navigate), `ActionCard` (generic action confirmation); cards are rendered inline in the message flow, not as separate messages | 2.4 | 8 |
| 2.8 | Build `<ChatInputBar>`: multi-line `TextInput` with auto-grow (max 4 lines then scroll), send button (disabled while streaming), attachment button (placeholder for Phase 6), haptic feedback on send; keyboard-aware positioning using `KeyboardAvoidingView` + `react-native-keyboard-controller` for smooth keyboard animation | 2.1 | 6 |
| 2.9 | Implement conversation persistence: save messages to SQLite `messages` table as they arrive (both user and assistant); on app relaunch, load conversation from SQLite and hydrate `FlashList`; implement pagination (load 50 messages at a time, load more on scroll-up) | 1.6, 2.2 | 6 |
| 2.10 | Implement conversation list/management: create `ConversationListScreen` (accessible from chat header); list all conversations with title, last message preview, timestamp; swipe to delete; tap to switch conversation; "New Chat" button | 2.9 | 6 |
| 2.11 | Implement suggested prompts: on empty chat, show 4-6 contextual prompt suggestions (e.g., "Draft a follow-up email", "Find contacts in 90210", "What's my schedule today?"); tapping a suggestion sends it as a user message | 2.1, 2.8 | 3 |
| 2.12 | Implement chat memory pressure management: for conversations exceeding 200 messages, only keep the most recent 200 in memory; older messages are in SQLite only, loaded on demand when scrolling up; implement `FlashList` estimated item size for smooth scroll performance | 2.2, 2.9 | 4 |
| 2.13 | Implement streaming error recovery: if SSE connection drops mid-response, show "Connection lost" inline with retry button; if retry succeeds, append new content after the partial response; if retry fails 3 times, show persistent error with "Copy partial response" option | 2.5 | 4 |
| 2.14 | Add haptic feedback: light impact on message send, medium impact on tool card appearance, selection feedback on prompt tap | 2.8, 2.7, 2.11 | 2 |
| 2.15 | Write unit tests: SSE event parsing, markdown rendering edge cases, message persistence CRUD, conversation management, token attachment on SSE connection | 2.5, 2.6, 2.9, 2.10 | 6 |
| 2.16 | Write Maestro E2E flows: send a message and receive a streamed response; scroll through long conversation; switch conversations; test offline behavior (send while offline, message queues) | 2.15 | 4 |

**Exit Criteria:**
- User can send a message and see a streamed response appear token by token
- Markdown renders correctly (bold, italic, code, lists, links)
- Tool cards render inline and are tappable (navigation wired in Phase 4)
- Conversations persist across app restarts
- Long conversations (200+ messages) scroll smoothly at 60 FPS
- SSE reconnects automatically after network interruption
- Empty, loading, and error states all render correctly
- Dark mode renders all chat elements correctly

**Decision Gate:**
Performance benchmark: measure FPS during rapid streaming on oldest supported device (iPhone 8 / iOS 16). If below 50 FPS, investigate and resolve before proceeding. Determine if FlashList `estimatedItemSize` needs per-message-type calibration.

**Deliverables:**
- Complete chat screen with streaming
- Conversation persistence layer
- Conversation list/management
- Markdown renderer
- Tool card system
- Chat input bar
- Test suites

**Verification Checklist:**
- [ ] Send 10 messages in rapid succession; all responses stream correctly
- [ ] Paste a response with markdown (bold, code block, list, link) — all render correctly
- [ ] Tool card appears when assistant invokes a tool; card is tappable
- [ ] Kill app during active stream; relaunch; partial response is preserved
- [ ] Scroll through 500-message conversation at 60 FPS (measure with Perf Monitor)
- [ ] Enable airplane mode mid-stream; "Connection lost" appears; disable airplane mode; retry succeeds
- [ ] Switch to dark mode; all chat elements are legible and correctly themed
- [ ] Maestro flows pass on physical device

---

#### Phase 3: People Tab — Contact List, Search, Filters, Swipe Actions, Offline Cache

**Purpose:** Build the CRM backbone. Real estate agents live in their contacts — this screen must be fast, searchable, and work offline.

**Entry Criteria:**
- Phase 2 exit criteria fully met (or Phase 1 exit criteria met — Phase 3 can begin after Phase 1 if resources allow parallel work, but the developer must not be the same person building Phase 2)
- Backend `/api/contacts` endpoint is documented and accessible

**Task List:**

| # | Task | Depends On | Est. Hours |
|---|------|-----------|-----------|
| 3.1 | Create `PeopleScreen` with `<ScreenContainer>` wrapper; implement 4 states | None | 3 |
| 3.2 | Build `<ContactList>` using `FlashList`; render `<ContactRow>` components: avatar (initials fallback), full name, company/role, last interaction date, lead status badge; section headers by alphabetical grouping (A, B, C...); fast scroll index strip on right edge | 3.1 | 8 |
| 3.3 | Implement contact data fetching with TanStack Query: `useContacts()` hook calls `/api/contacts` with pagination (cursor-based, 50 per page); infinite scroll via `useInfiniteQuery`; data is cached in TanStack Query and persisted to SQLite for offline access | 1.7, 1.6 | 6 |
| 3.4 | Implement offline-first sync: on app launch or pull-to-refresh, fetch contacts from API and upsert into SQLite; when offline, query SQLite directly; track `last_synced_at` per entity in `sync_meta` table; show "Last synced: X minutes ago" in header when offline | 3.3, 1.11 | 8 |
| 3.5 | Build search: `<SearchBar>` with debounced input (300ms); search executes against local SQLite (LIKE query on name, email, phone, company) for instant results; simultaneously hits `/api/contacts/search` for server results if online; merge and deduplicate results | 3.2, 3.3 | 6 |
| 3.6 | Build filter system: filter chips below search bar — by lead status (Lead, Client, Past Client, Vendor), by tag, by last interaction date range; filters apply to both local SQLite query and API query; multiple filters combine with AND logic; active filter count shown on chip | 3.5 | 6 |
| 3.7 | Implement swipe actions using `react-native-gesture-handler` + `react-native-reanimated`: swipe right reveals "Call" (green) and "Message" (blue) actions; swipe left reveals "Archive" (yellow) and "Delete" (red) actions; actions trigger immediately on full swipe or on button tap for partial swipe; haptic feedback on action trigger | 3.2 | 6 |
| 3.8 | Implement pull-to-refresh: `RefreshControl` on the `FlashList`; triggers full sync with server; shows success/failure toast on completion | 3.3 | 2 |
| 3.9 | Build `<AddContactFAB>`: floating action button (bottom-right, above tab bar); tapping opens `AddContactSheet` (bottom sheet via `@gorhom/bottom-sheet`): name, email, phone, company, lead status; validates required fields; submits to `/api/contacts` and inserts into SQLite | 3.1 | 5 |
| 3.10 | Implement "quick actions" from contact row: long-press reveals context menu (call, text, email, copy phone, copy email) using `react-native-context-menu-view` or equivalent | 3.2 | 4 |
| 3.11 | Write unit tests: contact sync logic, search ranking, filter combination, swipe action handlers, add contact validation | 3.4, 3.5, 3.6, 3.7, 3.9 | 5 |
| 3.12 | Write Maestro E2E flows: scroll contact list, search for contact, apply filter, swipe to call, add new contact, pull to refresh, test offline mode (airplane mode shows cached contacts) | 3.11 | 4 |

**Exit Criteria:**
- Contact list loads and displays 500+ contacts smoothly (60 FPS scroll)
- Search returns results within 200ms for local queries
- Filters narrow the list correctly (verified with known test data)
- Swipe actions work reliably on first attempt
- App shows cached contacts when offline
- Pull-to-refresh syncs new contacts from server
- Add contact form validates and submits correctly
- All 4 screen states render correctly

**Decision Gate:**
Evaluate whether SQLite FTS (Full-Text Search) is needed for search performance at scale (1000+ contacts). If search latency exceeds 200ms with LIKE queries, implement FTS5 virtual table.

**Deliverables:**
- Complete People tab with list, search, filters
- Offline sync system
- Swipe actions
- Add contact flow
- Test suites

**Verification Checklist:**
- [ ] Load 500 contacts; scroll at 60 FPS (Perf Monitor)
- [ ] Search "john" returns results in <200ms
- [ ] Apply "Lead" filter; only leads are shown
- [ ] Apply "Lead" + tag "Buyer"; intersection is shown
- [ ] Swipe right on contact; "Call" action opens phone dialer with correct number
- [ ] Enable airplane mode; contacts still display with "offline" indicator
- [ ] Pull to refresh while online; new contacts appear
- [ ] Add a contact; it appears in the list immediately
- [ ] Long-press contact; context menu appears with correct actions
- [ ] Maestro flows pass on physical device

---

#### Phase 4: Detail Screens — Contact Detail, Properties List, Property Detail

**Purpose:** Build the detail/drill-down layer. Users tap into contacts and properties from lists and from chat tool cards.

**Entry Criteria:**
- Phase 3 exit criteria met (contact list is operational)
- Backend `/api/contacts/:id`, `/api/properties`, `/api/properties/:id` endpoints are documented

**Task List:**

| # | Task | Depends On | Est. Hours |
|---|------|-----------|-----------|
| 4.1 | Build `ContactDetailScreen`: header with avatar (large), name, role, company; action row (call, text, email, share); detail sections: contact info, address, tags, notes, lead status, source, created/updated dates; all data from SQLite (offline) or API (online); edit mode toggle | None | 8 |
| 4.2 | Build contact edit mode: inline editing of all fields; save button validates and PATCHes to `/api/contacts/:id`; optimistic update in SQLite and TanStack Query cache; revert on failure with toast notification | 4.1 | 6 |
| 4.3 | Build contact activity timeline: `<ActivityTimeline>` component below contact details; shows chronological list of interactions (calls, emails, meetings, notes, AI conversations); each entry: icon, title, timestamp, preview; data from `/api/contacts/:id/activity`; paginated (load 20 at a time) | 4.1 | 6 |
| 4.4 | Build contact notes section: list of notes with add/edit/delete; notes stored in SQLite and synced to API; markdown support in note body; timestamps and "by" attribution | 4.1 | 5 |
| 4.5 | Wire chat tool cards to contact detail: `ContactCard` in chat navigates to `ContactDetailScreen` with `contactId` param; back button returns to chat | 2.7, 4.1 | 2 |
| 4.6 | Build `PropertiesScreen` (Properties tab): 4-state screen; `FlashList` of property cards; each card: hero image (using `expo-image` for caching + progressive loading), address, price, beds/baths/sqft, status badge (Active, Pending, Sold, Off-Market); sort options (price, date listed, status); pull-to-refresh | None | 8 |
| 4.7 | Implement property data fetching and offline cache: `useProperties()` hook with TanStack Query + SQLite sync pattern (same as contacts) | 1.7, 1.6 | 4 |
| 4.8 | Build property search and filters: search by address/MLS number; filters: status, price range (slider), beds (min), baths (min), sqft range; same local + server search pattern as contacts | 4.6 | 6 |
| 4.9 | Build `PropertyDetailScreen`: image gallery (horizontal pager with page dots, pinch-to-zoom via `react-native-gesture-handler`), property info section (address, price, beds/baths/sqft, lot size, year built, MLS number), description (collapsible if long), listing agent info, status history, associated contacts (linked buyers/sellers with tap to navigate) | 4.6 | 10 |
| 4.10 | Wire chat tool cards to property detail: `PropertyCard` in chat navigates to `PropertyDetailScreen` with `propertyId` param | 2.7, 4.9 | 2 |
| 4.11 | Build share functionality: share contact (vCard format) or property (formatted text with link) via `Share` API from React Native | 4.1, 4.9 | 3 |
| 4.12 | Write unit tests: contact detail rendering, edit/save flow, activity timeline pagination, property list rendering, property filters, image gallery | All above | 6 |
| 4.13 | Write Maestro E2E flows: navigate to contact detail from People list, edit a field and save, navigate to property detail from Properties tab, swipe through property images, navigate to contact detail from chat tool card | 4.12 | 4 |

**Exit Criteria:**
- Contact detail screen loads in <500ms from tap
- Contact edit saves correctly (optimistic update + server sync)
- Activity timeline loads and paginates
- Property list renders with images loading progressively (blur-up)
- Property detail image gallery supports swipe and pinch-to-zoom
- All navigation paths work (list to detail, chat to detail, back)
- Offline: detail screens show cached data with "offline" indicator

**Decision Gate:**
Decide whether property images need aggressive pre-caching (disk cache budget) or lazy loading is sufficient. Measure image memory pressure on iPhone SE (2 GB RAM).

**Deliverables:**
- Contact detail screen with edit, activity timeline, notes
- Properties list screen with search and filters
- Property detail screen with image gallery
- Navigation wiring from chat tool cards
- Test suites

**Verification Checklist:**
- [ ] Tap contact in People list; detail screen appears in <500ms
- [ ] Edit contact phone number; save; verify change persists after app restart
- [ ] Activity timeline shows 20 items; scroll down triggers load of next 20
- [ ] Properties list loads with images; no blank white squares during load
- [ ] Property detail: swipe through 5 images; pinch-to-zoom on one
- [ ] Tap `ContactCard` in chat; navigates to contact detail; press back; returns to chat
- [ ] Tap `PropertyCard` in chat; navigates to property detail; press back; returns to chat
- [ ] Share contact; share sheet opens with formatted vCard
- [ ] Airplane mode: all detail screens show cached data
- [ ] Maestro flows pass on physical device

---

#### Phase 5: Push Notifications & Deep Links

**Purpose:** Make Skye proactive. Notifications bring users back into the app; deep links take them to the right screen.

**Entry Criteria:**
- Phase 1 exit criteria met (auth and navigation working)
- Phase 2 and Phase 3 exit criteria met (screens exist to deep link into)
- Backend push subscribe and send endpoints are ready (see Section 5)
- APNs key (.p8 file) is uploaded to Expo push notification credentials or backend push service

**Task List:**

| # | Task | Depends On | Est. Hours |
|---|------|-----------|-----------|
| 5.1 | Install and configure `expo-notifications`: request permission on first launch (with pre-permission prompt explaining value), register for push notifications, obtain Expo push token or native APNs token | None | 4 |
| 5.2 | Backend: modify `/api/push/subscribe` to accept `{ token, platform: "ios", tokenType: "apns" | "expo" }` alongside existing web push subscriptions; store with platform discriminator; deduplicate tokens per user-device pair | None (backend) | 4 |
| 5.3 | Backend: modify `/api/push/send` to route to APNs (via Expo Push API or directly) for iOS tokens, Web Push for browser tokens; implement provider abstraction | 5.2 | 4 |
| 5.4 | Implement token registration flow: after auth, call `/api/push/subscribe` with device token; re-register on token change; unregister on logout | 5.1, 5.2 | 3 |
| 5.5 | Implement notification categories: define notification types (new_message, contact_update, property_alert, reminder, system); each type has: title template, body template, category identifier, action buttons (if applicable) | 5.1 | 4 |
| 5.6 | Implement foreground notification handling: `setNotificationHandler` to show/hide notifications based on current screen (don't show "new message" notification if user is already on chat screen); play custom sound for high-priority notifications | 5.1, 5.5 | 4 |
| 5.7 | Implement notification tap handling: `addNotificationResponseReceivedListener` extracts `data` payload (type, entityId); routes to correct screen (chat, contact detail, property detail); handle cold start (app was killed, user taps notification, app launches and navigates directly) | 5.6, 1.2 | 6 |
| 5.8 | Implement deep linking: configure URL scheme (`skye://`) and universal links (`https://app.skye.com/...`) in `app.json`; create `linkingConfig` for React Navigation mapping URL paths to screens (`/chat/:conversationId`, `/contacts/:contactId`, `/properties/:propertyId`); handle auth-gated deep links (if not authenticated, store deep link target, authenticate, then navigate) | 1.2, 5.7 | 8 |
| 5.9 | Implement notification preferences screen: allow users to toggle notification categories on/off; store preferences in API and locally; respect preferences in foreground handler | 5.5 | 4 |
| 5.10 | Implement badge count management: update app badge count on notification receipt; clear badge when user opens app; keep badge in sync with unread message count | 5.1 | 2 |
| 5.11 | Write unit tests: token registration, notification routing logic, deep link parsing, preference management | All above | 4 |
| 5.12 | Write Maestro E2E flows: receive notification (simulated), tap notification, verify correct screen opens; test deep link from Safari | 5.11 | 3 |

**Exit Criteria:**
- Permission prompt shows clear value proposition, not raw system dialog
- Push notifications are received when app is in foreground, background, and killed states
- Tapping a notification navigates to the correct screen
- Deep links from Safari or other apps open the correct screen
- Auth-gated deep links work (unauthenticated user taps link, signs in, then sees target screen)
- Badge count reflects unread state
- Notification preferences are respected

**Decision Gate:**
Choose between Expo Push API (simpler, Expo manages APNs communication) vs. direct APNs integration (more control, no Expo dependency). Recommendation: Expo Push API for V1, with architecture that allows swap to direct APNs later.

**Deliverables:**
- Push notification system (registration, receipt, handling)
- Deep link system (URL scheme + universal links)
- Notification preferences screen
- Backend push infrastructure
- Test suites

**Verification Checklist:**
- [ ] Fresh install: permission prompt shows custom pre-permission screen, then system dialog
- [ ] Send test push via Expo Push Tool; notification appears on device
- [ ] App in background: notification appears; tap opens correct screen
- [ ] App killed: notification appears; tap launches app and opens correct screen
- [ ] Open `skye://contacts/123` in Safari; app opens to contact detail for ID 123
- [ ] Open deep link while signed out; sign-in screen appears; after sign-in, navigates to target
- [ ] Toggle off "Contact Updates" in preferences; send a contact update push; notification does NOT appear
- [ ] Verify badge count clears when app is opened
- [ ] Maestro flows pass on physical device

---

#### Phase 6: Camera & Studio — OCR Flow, Content Generation

**Purpose:** Build the camera/OCR utility and the content generation studio. These are differentiating features that leverage AI.

**Entry Criteria:**
- Phase 1 exit criteria met (foundation)
- Phase 2 exit criteria met (streaming/AI infrastructure can be reused)
- Backend OCR and studio endpoints are documented and accessible

**Task List:**

| # | Task | Depends On | Est. Hours |
|---|------|-----------|-----------|
| 6.1 | Install and configure `react-native-vision-camera`: request camera permission (with pre-permission prompt); create `<CameraView>` component with viewfinder overlay, capture button, flash toggle, front/back toggle | None | 6 |
| 6.2 | Implement photo capture: capture high-resolution image, save to temporary directory; show preview with "Retake" and "Use Photo" options; implement basic image cropping/adjustment if needed | 6.1 | 4 |
| 6.3 | Implement OCR flow: after capture, send image to backend OCR endpoint (`/api/ocr/extract`); show processing indicator; receive structured data (name, phone, email, company, address from business card; or property details from flyer/sign); display extracted fields in editable form; user confirms or corrects; save as new contact or property | 6.2, 1.9 | 10 |
| 6.4 | Implement gallery picker: use `expo-image-picker` to select from camera roll; same OCR flow applies to gallery images | 6.3 | 3 |
| 6.5 | Build Studio tab screen with `<ScreenContainer>`: list of content generation templates (Property Listing Description, Social Media Post, Email Follow-Up, Open House Invite, Market Update, Neighborhood Guide); each template: icon, title, description | None | 4 |
| 6.6 | Build template execution flow: user selects template; form collects required inputs (property address, target audience, tone, key features, etc.); submit to backend `/api/studio/generate`; show streaming response (reuse SSE infrastructure from Phase 2); result screen with formatted output | 6.5, 2.5 | 8 |
| 6.7 | Build result actions: copy to clipboard, share via share sheet, save as draft (stored in SQLite), regenerate (re-submit with same inputs), edit and refine (send edited version back for AI improvement) | 6.6 | 4 |
| 6.8 | Build Studio history: list of previously generated content; stored in SQLite with template type, inputs, output, timestamp; search and filter by template type; tap to view/re-use | 6.7 | 4 |
| 6.9 | Implement camera permission recovery: if user denied camera permission, show instructions to enable in Settings with a button that opens iOS Settings directly via `Linking.openSettings()` | 6.1 | 2 |
| 6.10 | Write unit tests: OCR data extraction parsing, template form validation, studio generation request formatting, result actions | 6.3, 6.6, 6.7 | 4 |
| 6.11 | Write Maestro E2E flows: open camera, capture image, see OCR results, save as contact; open Studio, select template, fill form, see generated content, copy to clipboard | 6.10 | 4 |

**Exit Criteria:**
- Camera opens with viewfinder overlay within 1 second
- Photo capture produces a usable image
- OCR extracts text from a business card with >80% field accuracy
- Extracted data can be saved as a contact
- Studio templates produce streaming AI content
- Generated content can be copied, shared, and saved
- Studio history persists across sessions
- All permission flows handle denial gracefully

**Decision Gate:**
Evaluate OCR accuracy with real-world business cards and property flyers. If backend OCR accuracy is below 80% on a test set of 20 images, consider supplementing with on-device ML Kit pre-processing or a different OCR provider.

**Deliverables:**
- Camera capture screen
- OCR extraction and editing flow
- Studio template list screen
- Content generation with streaming
- Result actions (copy, share, save, regenerate)
- Studio history
- Test suites

**Verification Checklist:**
- [ ] Camera permission prompt shows custom pre-permission screen
- [ ] Capture a business card photo; OCR extracts name and phone correctly
- [ ] Correct an extracted field; save as contact; contact appears in People tab
- [ ] Select "Property Listing Description" template; fill form; see streaming output
- [ ] Copy generated content; paste in Notes app; content matches
- [ ] Share generated content; share sheet opens with formatted text
- [ ] View Studio history; previously generated content is listed
- [ ] Deny camera permission; recovery screen appears with Settings button
- [ ] Maestro flows pass on physical device

---

#### Phase 7: Polish & Compliance — Dark Mode Audit, Accessibility, Privacy Manifest, Performance

**Purpose:** Transform the functional app into a submission-quality product. This phase is not "make it look nice" — it is a systematic audit with concrete pass/fail criteria.

**Entry Criteria:**
- Phases 1-6 exit criteria all met
- All features are functionally complete
- No known crash bugs in Sentry

**Task List:**

| # | Task | Depends On | Est. Hours |
|---|------|-----------|-----------|
| 7.1 | Dark mode audit: walk through every screen in dark mode; verify: no white flashes during transitions, no unreadable text on dark backgrounds, no invisible icons, no hardcoded colors bypassing theme, status bar style correct on all screens; fix all issues | None | 6 |
| 7.2 | Accessibility audit: every interactive element has an `accessibilityLabel`; every image has `accessibilityRole="image"` and descriptive label; all buttons have minimum 44x44pt touch target; screen reader (VoiceOver) can navigate every screen meaningfully; focus order is logical; no information conveyed only by color; dynamic type scaling works (up to accessibility large sizes) | None | 8 |
| 7.3 | Performance audit: measure and optimize cold start time (target: <2s), screen transition time (target: <300ms), list scroll FPS (target: 60 FPS), memory usage (target: <150 MB at rest, <300 MB under heavy use), bundle size (target: <50 MB IPA); use Xcode Instruments (Time Profiler, Allocations, Core Animation) and Flipper | None | 8 |
| 7.4 | Privacy manifest: create/update `PrivacyInfo.xcprivacy` with all required reason APIs — `NSPrivacyAccessedAPICategoryFileTimestamp` (C617.1), `NSPrivacyAccessedAPICategorySystemBootTime` (35F9.1), `NSPrivacyAccessedAPICategoryDiskSpace` (E174.1), `NSPrivacyAccessedAPICategoryUserDefaults` (CA92.1); add any additional categories used by third-party SDKs; also configure in `app.json` under `expo.ios.privacyManifests` | None | 3 |
| 7.5 | Privacy nutrition labels: prepare accurate data collection declarations for App Store Connect — what data is collected, linked to identity, used for tracking; this must match actual app behavior exactly | 7.4 | 3 |
| 7.6 | Third-party SDK audit: verify every dependency's privacy manifest status; check for private API usage that could cause rejection; verify no SDK downloads executable code at runtime; document audit results | None | 4 |
| 7.7 | Animation polish: ensure all transitions use native driver (`useNativeDriver: true` or Reanimated worklets); add micro-interactions — tab switch animations, button press scale, list item appearance, pull-to-refresh custom animation; verify no janky animations on oldest supported device | None | 6 |
| 7.8 | Error message review: audit every user-facing error message for clarity and actionability; replace technical messages ("Error 500") with human messages ("Something went wrong. Please try again."); ensure all error states have retry actions | None | 3 |
| 7.9 | Offline experience review: walk through every screen while offline; verify: appropriate offline indicators, cached data displays correctly, mutations queue and sync when back online, no blank screens, no spinners that never resolve | None | 4 |
| 7.10 | Memory leak audit: navigate through all screens repeatedly (50 times); monitor memory with Xcode Instruments; identify and fix any components that retain memory after unmount; particular focus on SSE connections, event listeners, and timers | None | 6 |
| 7.11 | Haptic feedback audit: verify haptic feedback is consistent and appropriate across the app; not too frequent (annoying) or too sparse (missing affordance) | None | 2 |
| 7.12 | Splash screen and app icon: finalize app icon (1024x1024 + all sizes), splash screen animation (matches brand), loading states | None | 4 |
| 7.13 | Implement account deletion flow (Apple requirement): Settings > Delete Account > confirmation dialog > call `/api/user/delete` > clear all local data > navigate to sign-in screen | 0.12 | 4 |
| 7.14 | Run full Maestro E2E test suite: all flows from all phases; fix any regressions | All above | 4 |
| 7.15 | Run `npx expo-doctor`; resolve all warnings | All above | 2 |

**Exit Criteria:**
- Zero dark mode visual bugs across all screens
- VoiceOver can navigate the entire app without getting stuck
- Cold start < 2 seconds on iPhone SE (3rd gen)
- 60 FPS scroll on all lists (measured, not estimated)
- Privacy manifest is complete and validated
- No third-party SDK uses private APIs
- No memory leaks detected after 50-screen navigation cycle
- Account deletion works end-to-end
- Full Maestro suite passes

**Decision Gate:**
If performance targets are missed on oldest supported device, decide whether to raise the minimum iOS version (dropping device support) or invest additional optimization time. This decision has App Store reach implications.

**Deliverables:**
- Polished, accessible, performant app
- Privacy manifest
- SDK audit report
- Performance benchmark report
- Accessibility audit report
- Account deletion flow
- Complete Maestro test suite (green)

**Verification Checklist:**
- [ ] Screenshot every screen in dark mode; no visual issues
- [ ] VoiceOver reads every screen correctly (manual walkthrough)
- [ ] Cold start measured at <2s on physical iPhone SE
- [ ] Scroll People list (500 contacts) at 60 FPS (Instruments)
- [ ] Scroll Chat (200 messages) at 60 FPS (Instruments)
- [ ] Memory after 50-screen cycle is within 10% of initial measurement
- [ ] Privacy manifest passes Xcode validation
- [ ] All Maestro E2E flows pass
- [ ] `npx expo-doctor` shows zero warnings
- [ ] Delete account; relaunch; sign-in screen appears; no residual data

---

#### Phase 8: Submission — TestFlight, App Store Metadata, Review Prep

**Purpose:** Get the app through Apple's review and into users' hands. This is a separate phase because submission is not trivial — it requires specific artifacts, compliance checks, and a review strategy.

**Entry Criteria:**
- Phase 7 exit criteria fully met
- App is feature-complete and passes all tests
- Legal has reviewed privacy policy and terms of service
- Marketing has approved app name, description, and screenshots

**Task List:**

| # | Task | Depends On | Est. Hours |
|---|------|-----------|-----------|
| 8.1 | Create App Store Connect listing: app name, subtitle, category (Business), content rating, pricing (free), availability (US initially or global) | None | 2 |
| 8.2 | Prepare app metadata: description (4000 char max, keyword-optimized), keywords (100 char max, comma-separated), promotional text (170 char max), support URL, privacy policy URL, marketing URL | 8.1 | 3 |
| 8.3 | Capture App Store screenshots: required sizes (6.7", 6.5", 5.5" — at minimum 6.7" and 5.5"); capture actual app screens (not mockups); include: chat, people, property detail, studio, camera; dark and light variants if desired; add marketing frames/titles if desired | 8.1 | 6 |
| 8.4 | Build production binary: `eas build --platform ios --profile production`; verify bundle identifier, version, build number; verify app icon and splash screen are correct in the built IPA | None | 2 |
| 8.5 | Deploy to TestFlight: submit production build to TestFlight via EAS Submit or Transporter; add internal testers; verify app installs and runs on TestFlight; run full Maestro suite against TestFlight build | 8.4 | 3 |
| 8.6 | Internal beta testing: 5-10 testers use the app for 3-7 days; collect feedback via structured form (bugs, UX issues, performance complaints); triage and fix critical issues; rebuild and re-deploy if needed | 8.5 | 16 |
| 8.7 | Prepare App Review notes: explain features that require special configuration (e.g., "To test push notifications, send a test push from our admin panel"); provide demo credentials if login requires invitation; explain any features that may seem incomplete but are intentional | 8.6 | 2 |
| 8.8 | Compliance checklist before submission: (1) Account deletion works, (2) Privacy manifest complete, (3) Privacy nutrition labels accurate, (4) No IDFA usage (or ATT prompt if used), (5) No third-party payment for digital goods, (6) Export compliance answered correctly (`ITSAppUsesNonExemptEncryption` set), (7) All permission purpose strings are specific and clear, (8) App does not crash on any screen, (9) All links work (privacy policy, support URL), (10) Content moderation is in place for any UGC | 8.6 | 3 |
| 8.9 | Submit for App Review: submit via App Store Connect; select "Manual release" (not auto-release) to control launch timing; monitor review status | 8.7, 8.8 | 1 |
| 8.10 | Handle review feedback: if rejected, analyze rejection reason, fix the specific issue, re-submit with detailed notes explaining the fix; common RN rejections: crash on specific device (test on that device), permission usage description too vague (rewrite), HIG violation (adjust UI) | 8.9 | 8 (estimate) |
| 8.11 | Release: once approved, trigger release; monitor Sentry for crash-free rate; monitor App Store Connect for user reviews | 8.10 | 1 |

**Exit Criteria:**
- App is live on the App Store
- Sentry crash-free rate is >99.5% in first 48 hours
- No 1-star reviews citing bugs in first week
- TestFlight remains available for continued internal testing

**Decision Gate:**
After TestFlight beta, decide if app quality is sufficient for public release or if an additional polish sprint is needed. This is the last off-ramp before the app is public.

**Deliverables:**
- App Store listing with metadata and screenshots
- Production binary on TestFlight
- Beta test report with resolutions
- App Review submission
- Live app on App Store

**Verification Checklist:**
- [ ] TestFlight build installs and runs on 3+ different iPhone models
- [ ] 5+ internal testers have used the app for 3+ days
- [ ] All critical beta feedback is resolved
- [ ] Compliance checklist has 10/10 checks passing
- [ ] App Review submission is accepted (not rejected)
- [ ] App is live and downloadable from App Store
- [ ] Sentry shows >99.5% crash-free sessions

---

### 2. Dependency Graph

#### Phase-Level Dependencies

```
Phase 0 ────────→ Phase 1 ────────→ Phase 2 ────────→ Phase 4 ──┐
                     │                                             │
                     │                  Phase 3 ──────────────────→├──→ Phase 7 ──→ Phase 8
                     │                    ↑                        │
                     └────────────────────┘                        │
                     │                                             │
                     └──────────→ Phase 5 ────────────────────────→┘
                     │                                             │
                     └──────────→ Phase 6 ────────────────────────→┘
```

**Detailed dependency rules:**

| Phase | Hard Dependencies | Can Start When | Can Overlap With |
|-------|------------------|----------------|-----------------|
| 0 | None | Immediately | Nothing (must complete first) |
| 1 | Phase 0 complete | Phase 0 exits | Nothing (critical path) |
| 2 | Phase 1 complete | Phase 1 exits | Phase 3 (different developer) |
| 3 | Phase 1 complete | Phase 1 exits | Phase 2 (different developer), Phase 5 partially |
| 4 | Phase 2 + Phase 3 complete | Phase 3 exits (needs contact list + chat tool cards) | Phase 5, Phase 6 |
| 5 | Phase 1 complete (auth + nav); Phase 2 + 3 recommended (screens to navigate to) | Phase 3 exits | Phase 4, Phase 6 |
| 6 | Phase 1 complete (foundation); Phase 2 recommended (SSE infrastructure reuse) | Phase 2 exits | Phase 4, Phase 5 |
| 7 | Phases 1-6 all complete | Phase 6 exits (last feature phase) | Nothing (full-app audit) |
| 8 | Phase 7 complete | Phase 7 exits | Nothing (submission requires stability) |

#### Critical Path (Minimum Calendar Time)

```
Phase 0 → Phase 1 → Phase 2 → Phase 4 → Phase 7 → Phase 8
  (2w)      (3w)      (3w)      (2.5w)    (2w)      (2w)
                                                    = ~14.5 weeks
```

With parallelization (2 developers):

```
                               Developer A              Developer B
Phase 0 (2 weeks)             [Both developers]
Phase 1 (3 weeks)             [Both developers]
Phase 2 (3 weeks)             [Chat screen]             [People tab - Phase 3]
Phase 4 (2.5 weeks)           [Detail screens]          [Push & Deep Links - Phase 5]
Phase 6 (2 weeks)             [Camera & Studio]          ←── (joins Phase 6)
Phase 7 (2 weeks)             [Both developers]
Phase 8 (2 weeks)             [Both developers]
                                                        = ~11.5 weeks with 2 devs
```

#### Task-Level Parallelization Within Phases

**Phase 1** (with 2 developers):
- Dev A: Tasks 1.1-1.2-1.10-1.14 (project scaffold, navigation, auth, splash)
- Dev B: Tasks 1.3-1.4-1.5-1.6-1.7-1.8 (theming, storage layers, state management)
- Converge: Tasks 1.9 (API client needs auth store), 1.11-1.16 (shared utilities)

**Phase 2** (mostly sequential due to tight coupling):
- Parallel: 2.1/2.8 (screen shell + input bar) can be built simultaneously
- Parallel: 2.6 (markdown) and 2.7 (tool cards) are independent after 2.4
- Sequential: 2.5 (SSE) must precede 2.13 (error recovery); 2.9 (persistence) must precede 2.12 (memory management)

**Phase 4** (with 2 developers):
- Dev A: Tasks 4.1-4.4 (contact detail)
- Dev B: Tasks 4.6-4.9 (properties)
- Converge: Tasks 4.5, 4.10 (wiring tool cards)

#### Backend Change Timeline

| When | Endpoint | Change Type |
|------|----------|------------|
| Phase 0 | `/api/health` | New |
| Phase 0 | `/api/auth/native` | New |
| Phase 0 | `/api/user/delete` | New |
| Phase 0 | All 129 routes | Audit (JSON responses, CORS) |
| Phase 5 | `/api/push/subscribe` | Modify |
| Phase 5 | `/api/push/send` | Modify |

---

### 3. Architecture Decision Records — Final Recommendations

---

#### ADR-1: Auth Library

**Decision:** `expo-auth-session` with PKCE flow

**Justification:**
- `expo-auth-session` provides a universal OAuth implementation that works within the Expo managed workflow without ejecting or requiring native module linking beyond what Expo provides
- Supports PKCE (Proof Key for Code Exchange), which is the current security best practice and avoids the deprecated Implicit flow
- The alternative, `@react-native-google-signin/google-signin`, provides a more native UI but puts web authentication behind a paywall and introduces a heavier native dependency. Since Skye is iOS-only and the web app already handles web auth separately, this paywall is less relevant — however, `expo-auth-session` is still preferred because it keeps the auth logic in JavaScript (easier to debug), does not require linking native Google Sign-In SDK, and works seamlessly with development builds
- Supabase integration is straightforward: the auth code from Google is sent to our own `/api/auth/native` backend endpoint, which performs the server-side token exchange (keeping the Google client secret on the server) and returns Supabase-compatible tokens

**Implementation Sketch:**

```typescript
// 1. Configuration
const discovery = useAutoDiscovery('https://accounts.google.com');
const [request, response, promptAsync] = Google.useAuthRequest({
  iosClientId: process.env.EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID,
  webClientId: process.env.EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID,
  scopes: ['openid', 'profile', 'email'],
  usePKCE: true,
  redirectUri: makeRedirectUri({ scheme: 'com.skye.crm' }),
});

// 2. Trigger auth
const handleSignIn = () => promptAsync();

// 3. Handle response
useEffect(() => {
  if (response?.type === 'success') {
    const { code } = response.params;
    // Send to our backend — NOT directly to Google token endpoint
    apiClient.post('/api/auth/native', {
      code,
      codeVerifier: request?.codeVerifier,
      redirectUri: makeRedirectUri({ scheme: 'com.skye.crm' }),
    }).then(({ accessToken, refreshToken, user }) => {
      authStore.login({ accessToken, refreshToken, user });
    });
  }
}, [response]);
```

**Redirect URI Format:** `com.googleusercontent.apps.{IOS_CLIENT_ID}:/oauthredirect`

**Token Storage:** Access token in Zustand (memory) + MMKV (for persistence across restarts). Refresh token in `expo-secure-store` (iOS Keychain). Refresh token is NEVER stored in MMKV or AsyncStorage.

---

#### ADR-2: SSE/Streaming

**Decision:** `react-native-sse`

**Justification:**
- `react-native-sse` is the most widely adopted SSE library for React Native with TypeScript support and no native module dependencies (uses `XMLHttpRequest` internally, which is polyfilled in React Native)
- It works on Hermes engine without issues since it avoids native module bridges
- The alternative, `react-native-fetch-event-source` (fetch-based), supports POST requests and custom headers, which would be useful if the streaming endpoint required POST. However, since the Skye backend streaming endpoint uses GET with query parameters and auth header, `react-native-sse` is sufficient and has a simpler API
- The library handles automatic reconnection natively, which is critical for mobile (users move between Wi-Fi and cellular)

**Implementation Sketch:**

```typescript
const useStreamingChat = (conversationId: string) => {
  const { accessToken } = useAuthStore();
  const eventSourceRef = useRef<EventSource | null>(null);
  const [streamState, setStreamState] = useState<'idle'|'connecting'|'streaming'|'error'>('idle');

  const sendMessage = useCallback((content: string) => {
    const es = new EventSource(
      `${API_BASE}/api/chat/stream?conversationId=${conversationId}&message=${encodeURIComponent(content)}`,
      {
        headers: { Authorization: `Bearer ${accessToken}` },
        method: 'GET',
        timeout: 0, // No timeout for streaming
        pollingInterval: 5000, // Reconnect after 5s if server closes
      }
    );

    es.addEventListener('content_delta', (event) => {
      const { text } = JSON.parse(event.data);
      appendToCurrentMessage(text);
    });

    es.addEventListener('content_stop', () => {
      finalizeMessage();
      es.close();
    });

    es.addEventListener('error', (event) => {
      handleStreamError(event);
    });

    eventSourceRef.current = es;
  }, [conversationId, accessToken]);

  // Cleanup on unmount
  useEffect(() => () => eventSourceRef.current?.close(), []);

  return { sendMessage, streamState };
};
```

**Reconnection Strategy:**
- On disconnect: exponential backoff (1s, 2s, 4s, 8s, 16s, max 30s)
- After 3 consecutive failures: stop auto-reconnect, show manual retry button
- On reconnect success: if mid-message, re-request from the last received event ID (if backend supports `Last-Event-ID`) or restart the message
- Partial response preservation: store streamed tokens in a buffer; on disconnect, the buffer content is displayed as-is with a "Connection lost" indicator

---

#### ADR-3: Offline Storage

**Decision:** Dual-layer — `expo-sqlite` for structured data + `react-native-mmkv` for key-value data

**Justification:**
- **expo-sqlite** is the right choice for contacts (500+), properties, conversations, and messages because this data has relational structure (contacts have activities, properties have images, conversations have messages) and requires complex queries (search, filter, sort, paginate). SQLite handles this natively with full SQL support. expo-sqlite is a core Expo SDK package, meaning zero native configuration and automatic compatibility with Expo updates
- **react-native-mmkv** is the right choice for fast key-value access: theme preference, onboarding state, last sync timestamps, feature flags, TanStack Query cache persistence. MMKV uses memory-mapped files and is up to 30x faster than AsyncStorage — relevant for data read on every app launch (theme, auth state)
- `expo-secure-store` (iOS Keychain) is used exclusively for refresh tokens — this is not general-purpose storage, it is a security-specific choice
- The alternative of using only SQLite for everything (including key-value) adds unnecessary overhead for simple reads. The alternative of only MMKV for everything cannot handle relational queries on 500+ contacts

**Schema Overview (SQLite):**

```sql
-- Core tables
CREATE TABLE contacts (
  id TEXT PRIMARY KEY,
  first_name TEXT NOT NULL,
  last_name TEXT NOT NULL,
  email TEXT,
  phone TEXT,
  company TEXT,
  role TEXT,
  lead_status TEXT DEFAULT 'lead',
  tags TEXT, -- JSON array
  address TEXT, -- JSON object
  notes TEXT,
  source TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  synced_at TEXT
);

CREATE TABLE properties (
  id TEXT PRIMARY KEY,
  address TEXT NOT NULL,
  city TEXT,
  state TEXT,
  zip TEXT,
  price REAL,
  beds INTEGER,
  baths REAL,
  sqft INTEGER,
  lot_size REAL,
  year_built INTEGER,
  mls_number TEXT,
  status TEXT DEFAULT 'active',
  description TEXT,
  images TEXT, -- JSON array of URLs
  listing_agent_id TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  synced_at TEXT
);

CREATE TABLE conversations (
  id TEXT PRIMARY KEY,
  title TEXT,
  last_message_preview TEXT,
  last_message_at TEXT,
  message_count INTEGER DEFAULT 0,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  synced_at TEXT
);

CREATE TABLE messages (
  id TEXT PRIMARY KEY,
  conversation_id TEXT NOT NULL,
  role TEXT NOT NULL, -- 'user' | 'assistant'
  content TEXT NOT NULL,
  tool_calls TEXT, -- JSON array of tool invocations
  created_at TEXT NOT NULL,
  FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE
);

CREATE TABLE studio_drafts (
  id TEXT PRIMARY KEY,
  template_type TEXT NOT NULL,
  inputs TEXT NOT NULL, -- JSON
  output TEXT NOT NULL,
  created_at TEXT NOT NULL
);

CREATE TABLE sync_meta (
  entity_type TEXT PRIMARY KEY, -- 'contacts' | 'properties' | 'conversations'
  last_synced_at TEXT NOT NULL,
  last_cursor TEXT
);

-- Indexes for search performance
CREATE INDEX idx_contacts_name ON contacts(first_name, last_name);
CREATE INDEX idx_contacts_email ON contacts(email);
CREATE INDEX idx_contacts_phone ON contacts(phone);
CREATE INDEX idx_contacts_lead_status ON contacts(lead_status);
CREATE INDEX idx_properties_status ON properties(status);
CREATE INDEX idx_properties_price ON properties(price);
CREATE INDEX idx_messages_conversation ON messages(conversation_id, created_at);
```

**Migration System:**
- `sync_meta` tracks schema version
- Migrations are numbered sequentially (`001_initial.sql`, `002_add_fts.sql`, etc.)
- On app launch, `db.ts` checks current version and runs any pending migrations
- Migrations are irreversible (no down migrations — not needed for mobile)

---

#### ADR-4: State Management

**Decision:** TanStack Query (server state) + Zustand (client state) + MMKV (persistence)

**Justification:**
- This combination is the established standard for React Native apps in 2025-2026, documented extensively by the TanStack and Zustand communities as complementary (not competing) tools
- **TanStack Query** handles all server data: contacts, properties, conversations, studio output. It provides automatic caching, background refetching, optimistic updates, infinite scroll pagination, and retry logic — eliminating the need to build any of these manually
- **Zustand** handles client-only state: auth state (user, tokens), UI state (active bottom sheet, toast queue, theme preference), network state (connectivity), and streaming state (current SSE connection status). Zustand is chosen over Redux because the client state in Skye is small and simple — there is no need for Redux's middleware, devtools ecosystem, or boilerplate
- **MMKV** persists Zustand stores and TanStack Query cache across app restarts. For TanStack Query, a custom MMKV persistor serializes the query cache; for Zustand, the `persist` middleware writes to MMKV
- The separation is clean: if it comes from the server, it lives in TanStack Query. If it is generated client-side and never sent to a server, it lives in Zustand

**Store Separation:**

```
zustand/
  useAuthStore.ts       — user, tokens, isAuthenticated, login(), logout()
  useUIStore.ts         — theme, activeModal, toastQueue
  useNetworkStore.ts    — isConnected, lastConnectedAt
  useStreamStore.ts     — activeStreamId, streamState, partialContent

queries/
  useContacts.ts        — useQuery/useInfiniteQuery for /api/contacts
  useContact.ts         — useQuery for /api/contacts/:id
  useProperties.ts      — useQuery/useInfiniteQuery for /api/properties
  useProperty.ts        — useQuery for /api/properties/:id
  useConversations.ts   — useQuery for /api/conversations
  useMessages.ts        — useInfiniteQuery for /api/conversations/:id/messages
  useActivity.ts        — useInfiniteQuery for /api/contacts/:id/activity

mutations/
  useCreateContact.ts   — useMutation with optimistic update
  useUpdateContact.ts   — useMutation with optimistic update
  useDeleteContact.ts   — useMutation with optimistic update
  useGenerateContent.ts — useMutation for /api/studio/generate
```

**TanStack Query Configuration:**

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // 5 minutes
      gcTime: 30 * 60 * 1000,         // 30 minutes garbage collection
      retry: 2,
      refetchOnWindowFocus: false,     // Not useful on mobile
      refetchOnReconnect: true,        // Critical for mobile
      networkMode: 'offlineFirst',     // Return cache while offline
    },
    mutations: {
      retry: 1,
      networkMode: 'offlineFirst',
    },
  },
});
```

---

#### ADR-5: Navigation

**Decision:** React Navigation v7 with static configuration API

**Justification:**
- React Navigation is the de facto navigation library for React Native, and v7 (shipped with Expo SDK 52) introduces the static configuration API that automatically generates TypeScript types and simplifies deep link configuration
- Expo Router was considered as an alternative (it is built on React Navigation and provides file-based routing). However, Expo Router is optimized for universal (web + native) apps, and Skye is iOS-only. The file-based routing convention adds indirection that is unnecessary for a single-platform app with a known, static navigation hierarchy. React Navigation v7's static API provides the same TypeScript benefits without the file-system convention
- The navigation hierarchy is simple enough that React Navigation v7's explicit configuration is clearer than file-based routing

**Navigator Hierarchy:**

```
RootNavigator (Stack)
├── SplashScreen
├── AuthNavigator (Stack)
│   ├── WelcomeScreen
│   └── SignInScreen
├── MainTabNavigator (Bottom Tabs)
│   ├── ChatTab (Stack)
│   │   ├── ChatScreen
│   │   └── ConversationListScreen
│   ├── PeopleTab (Stack)
│   │   ├── PeopleScreen
│   │   └── ContactDetailScreen
│   ├── PropertiesTab (Stack)
│   │   ├── PropertiesScreen
│   │   └── PropertyDetailScreen
│   └── StudioTab (Stack)
│       ├── StudioScreen
│       ├── StudioTemplateScreen
│       └── StudioResultScreen
├── CameraScreen (Modal)
├── NotificationPreferencesScreen (Modal)
└── SettingsScreen (Modal)
```

**Deep Link Configuration:**

```typescript
const linking: LinkingOptions = {
  prefixes: ['skye://', 'https://app.skye.com'],
  config: {
    screens: {
      MainTabNavigator: {
        screens: {
          ChatTab: {
            screens: {
              ChatScreen: 'chat/:conversationId?',
            },
          },
          PeopleTab: {
            screens: {
              ContactDetailScreen: 'contacts/:contactId',
            },
          },
          PropertiesTab: {
            screens: {
              PropertyDetailScreen: 'properties/:propertyId',
            },
          },
        },
      },
    },
  },
};
```

**Auth-Gated Deep Links:** When a deep link arrives and the user is not authenticated, the target route is stored in Zustand (`pendingDeepLink`). After successful authentication, the navigation system checks for a pending deep link and navigates to it, then clears the stored value.

---

### 4. Risk Register

| ID | Risk | Likelihood (1-5) | Impact (1-5) | Score (LxI) | Mitigation | Contingency |
|----|------|:--:|:--:|:--:|------------|-------------|
| R-01 | **SSE streaming performance on React Native** — RN's bridge (even with Hermes) may introduce latency during high-throughput streaming, causing UI jank when rendering tokens at 20+ per second | 3 | 4 | **12** | Use `react-native-sse` (XMLHttpRequest-based, no native bridge for each event). Batch UI updates: accumulate tokens in a ref for 50ms before triggering a state update. Use FlashList with estimated item size. Profile with Instruments early in Phase 2. | If streaming causes >16ms frame drops: switch to polling with 200ms interval during streaming (degraded but smooth experience). Alternatively, buffer entire response and display at once with a typing animation effect. |
| R-02 | **Google OAuth redirect URI conflicts with web app** — iOS app and web app may register conflicting redirect URIs in Google Cloud Console, causing one to break | 2 | 4 | **8** | Create a separate iOS OAuth client ID in Google Cloud Console (not reuse the web client ID). iOS client ID uses a reversed client ID scheme (`com.googleusercontent.apps.{ID}`), which cannot conflict with web redirect URIs. Document both client IDs clearly in `.env` files. | If conflict occurs: create an entirely new Google Cloud project for the mobile app. This is an organizational overhead, not a technical blocker. |
| R-03 | **Apple App Store rejection** — Privacy violations, HIG non-compliance, crashes, missing account deletion, private API usage by third-party SDKs | 4 | 5 | **20** | Dedicated Phase 7 audit (accessibility, privacy manifest, SDK audit, performance). Phase 8 has explicit compliance checklist with 10 items. Account deletion implemented as a requirement (task 7.13). Pre-submission review using Apple's common rejection checklist. Test on minimum 3 different iPhone models. | Budget 1-2 weeks for rejection response. First rejection is informational (they tell you exactly what to fix). Have a "rejection response" playbook: fix within 48 hours, re-submit with detailed notes. Most RN apps pass on second attempt. |
| R-04 | **Offline sync conflicts** — User edits a contact on mobile while offline, same contact is edited on web. When mobile comes online, conflicts arise | 3 | 3 | **9** | Implement "last-write-wins" strategy with timestamps. Each entity has `updated_at` field. On sync, compare mobile's `updated_at` with server's `updated_at`. If server is newer, server wins (mobile changes are lost). If mobile is newer, mobile wins. This is simple and predictable. | If users report data loss: implement field-level merge (compare individual fields, not entire records). If that is insufficient: implement conflict resolution UI ("Your version vs. server version — choose one"). This adds complexity and should only be built if users actually encounter this issue. |
| R-05 | **Push notification delivery reliability** — APNs is best-effort; notifications may be delayed, deduplicated, or dropped by iOS power management | 3 | 3 | **9** | Use Expo Push API which handles APNs receipts and retry logic. Implement notification receipt tracking: when backend sends push, log the push ID; when app receives push, confirm receipt via API; monitor delivery rate in analytics. Set notification priority to "high" for time-sensitive notifications. | If delivery rate falls below 90%: investigate whether iOS is throttling (often due to too many silent pushes). Implement in-app notification polling as backup: on app foreground, fetch unread notifications from server. This ensures no notification is permanently lost. |
| R-06 | **Chat screen memory pressure with long conversations** — 500+ messages with markdown, tool cards, and images could exhaust memory on older devices (2-3 GB RAM) | 3 | 4 | **12** | Task 2.12 limits in-memory messages to 200. FlashList's virtualization ensures only visible items are rendered. Images use `expo-image` with disk caching (not memory caching) for inline images. Measure memory with Xcode Instruments during Phase 2 on iPhone SE. | If memory still exceeds 300 MB: reduce in-memory limit to 100 messages. Implement message "windowing" where only the viewport +/- 50 messages are in JS memory. Move markdown parsing to a pre-computed step (parse on insert, store rendered tree in SQLite) to reduce per-render computation. |
| R-07 | **Dependency compatibility with Hermes engine** — Some npm packages use APIs not supported by Hermes (e.g., `Intl`, `Proxy` edge cases), causing runtime crashes | 2 | 4 | **8** | All dependencies are vetted for Hermes compatibility before inclusion in the bill of materials (Section 6). Expo SDK 52 packages are tested against Hermes by the Expo team. For non-Expo packages, check GitHub issues for Hermes-related bug reports. Run `global.HermesInternal` check on launch to confirm Hermes is active. | If a dependency fails on Hermes: first check for polyfills (e.g., `intl` polyfill for Intl API). If no polyfill exists: replace the dependency with a Hermes-compatible alternative (there is always one for mainstream packages). As a last resort: file an issue on the dependency's GitHub and implement a workaround. |
| R-08 | **Token refresh race conditions** — Multiple concurrent API calls all detect a 401; each independently tries to refresh the token; only the first succeeds, others fail | 4 | 3 | **12** | Task 1.9 implements a mutex pattern in the API client: the first 401 triggers a refresh; all subsequent requests during refresh are queued in a promise array; when refresh completes, all queued requests are retried with the new token. This is a well-known pattern (sometimes called a "token refresh lock"). | If the mutex pattern has edge cases: implement a simpler approach — proactively refresh the token 60 seconds before expiry (using the `exp` claim from the JWT). This eliminates 401s entirely for normal usage. The 401 handler becomes a safety net, not the primary mechanism. |
| R-09 | **Image caching memory pressure on older devices** — Property images (10+ per listing, high resolution) could cause memory warnings and OOM kills | 3 | 4 | **12** | Use `expo-image` which provides automatic memory-aware caching (it evicts images under memory pressure). Configure max memory cache to 100 MB and max disk cache to 500 MB. Load thumbnails in list views, full resolution only in detail view gallery. Use progressive JPEG loading (blur-up effect) via `expo-image`'s `placeholder` prop. | If OOM kills occur: reduce memory cache to 50 MB. Implement explicit image recycling: when navigating away from a gallery, call `Image.clearMemoryCache()`. If still insufficient: serve WebP format from backend (40% smaller than JPEG at equivalent quality). |
| R-10 | **App startup time exceeding 2s budget** — Database migrations, cache hydration, auth check, Sentry init, and navigation render may exceed 2 seconds total | 3 | 3 | **9** | Defer non-critical initialization: Sentry can init async after first render. Database migrations run lazily (check schema version synchronously, run migrations in background if needed). Cache hydration uses MMKV (synchronous, <5ms). Auth check is a single MMKV read (not a network call). Splash screen covers the initialization period gracefully. | If startup exceeds 2s: profile with Xcode Instruments Time Profiler to identify the bottleneck. Common fixes: lazy-load tab screens (only import when first visited), reduce initial query prefetching, defer font loading (use system fonts initially, swap to custom fonts after load). |
| R-11 | **Privacy Manifest completeness** — Apple rejects the app because a third-party SDK uses a Required Reason API that is not declared in the app's privacy manifest | 4 | 4 | **16** | Task 7.4 creates the privacy manifest. Task 7.6 audits every third-party SDK for required reason API usage. Use Xcode's "Generate Privacy Report" feature (Product > Generate Privacy Report) to automatically detect missing declarations. Cross-reference with Apple's list of required reason APIs. | If rejected for privacy manifest: Apple's rejection message will specify exactly which API category is missing. Add the declaration, rebuild, and re-submit. Turnaround: 24-48 hours. This is one of the most fixable rejection reasons. |
| R-12 | **Third-party SDK introducing private API usage** — A dependency (or one of its transitive dependencies) calls a private Apple API, which Apple's automated scanner detects during review | 2 | 5 | **10** | Task 7.6 audits SDKs. Before submission: run `nm` and `otool` on the built binary to scan for private API symbols. Use third-party tools like App Store Compliance Scanner. Prefer Expo SDK packages (audited by Expo) over random npm packages. Pin all dependency versions (no `^` or `~` in `package.json` for native dependencies). | If rejected: the rejection message names the specific API. Identify which dependency uses it. Either update the dependency (if the maintainer has fixed it), replace it with an alternative, or fork and patch. Typically resolvable within 1-3 days. |

**Risk Heat Map:**

```
Impact  5│    R-03
        4│ R-07  R-11  R-01,R-06,R-08,R-09   R-12
        3│       R-10  R-04,R-05
        2│
        1│
         └─────────────────────────────────
           1     2     3     4     5
                    Likelihood
```

**Top 3 Risks by Score:**
1. **R-03** (App Store Rejection) — Score 20 — Mitigated by dedicated Phase 7 + Phase 8 compliance process
2. **R-11** (Privacy Manifest) — Score 16 — Mitigated by automated detection tools + explicit audit task
3. **R-01, R-06, R-08, R-09** (tied at Score 12) — All performance/resource risks, mitigated by early measurement and conservative defaults

---

### 5. Backend Changes Required

Despite the principle that "the backend is largely unchanged," the following changes are required. These are minimal, targeted modifications — not a backend rewrite.

| Priority | Endpoint | Change Type | Phase Needed | Effort Est. | Description |
|----------|----------|-------------|:---:|:-----------:|-------------|
| **P0** | `GET /api/health` | New | 0 | 1h | Returns `{ "status": "ok", "timestamp": "..." }`. No auth required. Used by native app to verify connectivity before auth flow. Also useful for uptime monitoring. |
| **P0** | `POST /api/auth/native` | New | 0 | 8h | Accepts `{ code, codeVerifier, redirectUri }` from native Google PKCE flow. Server-side: exchanges code with Google (using client secret stored server-side), obtains Google tokens, creates or updates Supabase user via `supabase.auth.admin.createUser()` or `signInWithIdToken()`, returns `{ accessToken, refreshToken, user }`. Must handle: new user creation, existing user login, invalid code, expired code. |
| **P0** | `DELETE /api/user/delete` | New | 0 | 6h | Apple App Store requirement. Accepts authenticated request. Cascades deletion: user data, contacts, properties, conversations, messages, push subscriptions, studio drafts. Returns confirmation. Must be idempotent. Should send confirmation email before deletion (with 24h grace period if possible). |
| **P1** | `POST /api/push/subscribe` | Modify | 5 | 4h | Currently accepts web push subscription object. Modify to accept: `{ token, platform: "ios" | "web", tokenType: "apns" | "expo" | "web-push" }`. Store platform discriminator in push_subscriptions table. Deduplicate by (user_id, device_token) pair. Remove stale tokens (not seen in 30 days). |
| **P1** | `POST /api/push/send` | Modify | 5 | 4h | Currently sends via Web Push. Add provider routing: if `platform === "ios"`, send via Expo Push API (or direct APNs). If `platform === "web"`, send via existing Web Push. Abstract into a `PushProvider` interface. |
| **P2** | All 129 routes | Audit | 0 | 4h | (1) Verify every error response is JSON, not HTML (e.g., Next.js default error pages return HTML). Add global error handler that wraps all responses in `{ error: { code, message } }` format. (2) Verify no route checks `Origin` or `Referer` header (native apps don't send these). (3) Verify `Content-Type: application/json` header is set on all responses. |

**Total Backend Effort:** ~27 hours (~3.5 developer-days)

**CORS Note:** React Native apps make HTTP requests via the native networking stack, not a browser. This means there is no CORS preflight — the `Origin` header is not sent. If any backend middleware rejects requests without an `Origin` header, it will break the native app. The audit must verify that CORS middleware either (a) allows requests without `Origin` or (b) is bypassed for API routes.

---

### 6. Technology Stack — Final Bill of Materials

#### Core Framework

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `expo` | ~52.x | App framework, managed workflow, build tooling | MIT | Yes | Included |
| `react-native` | ~0.76.x | Native runtime | MIT | Yes (default) | N/A |
| `react` | ~18.3.x | UI rendering library | MIT | Yes | N/A |
| `typescript` | ~5.3.x | Static type checking | Apache-2.0 | N/A (build-time) | N/A |

#### Navigation

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `@react-navigation/native` | ^7.x | Navigation framework core | MIT | Yes | N/A |
| `@react-navigation/native-stack` | ^7.x | Native stack navigator (uses UINavigationController on iOS) | MIT | Yes | N/A |
| `@react-navigation/bottom-tabs` | ^7.x | Bottom tab navigator | MIT | Yes | N/A |
| `react-native-screens` | ~4.8.x | Native screen containers (required by native-stack) | MIT | Yes | N/A |
| `react-native-safe-area-context` | ~5.1.x | Safe area insets | MIT | Yes | N/A |

#### State Management

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `@tanstack/react-query` | ^5.x | Server state management (caching, fetching, sync) | MIT | Yes | N/A |
| `@tanstack/query-async-storage-persister` | ^5.x | Persist query cache to MMKV | MIT | Yes | N/A |
| `zustand` | ^5.x | Client state management | MIT | Yes | N/A |
| `react-native-mmkv` | ^3.x | Fast key-value storage (memory-mapped) | MIT | Yes | UserDefaults (CA92.1) |

#### Auth

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `expo-auth-session` | ~6.x | OAuth 2.0 with PKCE flow | MIT | Yes | N/A |
| `expo-crypto` | ~14.x | PKCE code verifier generation | MIT | Yes | N/A |
| `expo-web-browser` | ~14.x | OAuth browser redirect handling | MIT | Yes | N/A |
| `expo-secure-store` | ~14.x | Keychain storage for refresh tokens | MIT | Yes | N/A |

#### Networking & Streaming

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `react-native-sse` | ^1.2.x | Server-Sent Events for AI streaming | MIT | Yes (XMLHttpRequest) | N/A |
| `@react-native-community/netinfo` | ^11.x | Network connectivity detection | MIT | Yes | N/A |

#### Storage

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `expo-sqlite` | ~15.x | Relational database for offline data | MIT | Yes | FileTimestamp (C617.1) |

#### UI & Animation

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `react-native-reanimated` | ~3.16.x | High-performance animations (worklet-based) | MIT | Yes | N/A |
| `react-native-gesture-handler` | ~2.22.x | Native gesture recognition (swipe, pinch, pan) | MIT | Yes | N/A |
| `expo-image` | ~2.x | Image loading with caching, blur placeholder, progressive loading | MIT | Yes | DiskSpace (E174.1) |
| `expo-haptics` | ~14.x | Haptic feedback (impact, notification, selection) | MIT | Yes | N/A |
| `@gorhom/bottom-sheet` | ^5.x | Bottom sheet modal component | MIT | Yes | N/A |
| `@shopify/flash-list` | ^2.x | Virtualized list (replaces FlatList, 5-10x faster) | MIT | Yes | N/A |
| `react-native-markdown-display` | ^7.x | Markdown rendering in chat bubbles | MIT | Yes | N/A |
| `react-native-keyboard-controller` | ^1.x | Smooth keyboard animation handling | MIT | Yes | N/A |

#### Push Notifications

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `expo-notifications` | ~0.29.x | Push notification registration, handling, display | MIT | Yes | N/A |
| `expo-device` | ~7.x | Device info for push token registration | MIT | Yes | N/A |

#### Camera & Media

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `react-native-vision-camera` | ^4.x | Camera capture with frame processor support | MIT | Yes | Camera usage |
| `expo-image-picker` | ~16.x | Gallery image selection | MIT | Yes | Photo library usage |

#### Monitoring & Error Tracking

| Package | Version | Purpose | License | Hermes | Privacy Manifest |
|---------|---------|---------|---------|:------:|:----------------:|
| `@sentry/react-native` | ^8.x | Crash reporting, performance monitoring, error tracking | MIT | Yes | Included |

#### Testing

| Package | Version | Purpose | License | Notes |
|---------|---------|---------|---------|-------|
| `jest` | ^29.x | Unit test runner | MIT | Dev dependency |
| `@testing-library/react-native` | ^12.x | Component testing utilities | MIT | Dev dependency |
| `maestro` | latest | E2E testing framework (YAML-based flows) | Apache-2.0 | CLI tool, not npm dependency |

#### Build & Deployment

| Package | Version | Purpose | License | Notes |
|---------|---------|---------|---------|-------|
| `eas-cli` | latest | EAS Build + Submit CLI | MIT | Global CLI tool |
| `expo-updates` | ~0.27.x | OTA update delivery (post-launch) | MIT | Optional, for post-v1 updates |

**Total production dependencies:** ~35 packages
**Estimated IPA size:** 35-45 MB (pre-App Thinning)

**Version Pinning Policy:** All dependencies that include native modules (anything with iOS/Android code) are pinned to exact minor versions using `~` (tilde, patch-only updates). Pure JS dependencies use `^` (caret, minor updates). This prevents native module incompatibilities from unexpected updates.

---

### 7. Execution Principles

These are not aspirational guidelines. They are contractual obligations that every developer on the project agrees to follow.

**7.1 — One Phase at a Time**
Never start the next phase until the current phase passes every item on its verification checklist. "Mostly done" is not done. If a phase is blocked, the blockage is resolved — it is not worked around by jumping ahead. The only exception is explicit parallelization noted in the dependency graph (Section 2), and only with separate developers on separate branches.

**7.2 — Four States From Day One**
Every screen renders four states: loading (skeleton), success (data), empty (no data + CTA), and error (message + retry). These are not added in Phase 7. They are built as the screen is built, enforced by the `<ScreenContainer>` component from task 1.16. A PR that shows a screen with only a success state will be rejected.

**7.3 — Dark Mode Is Not a Phase 7 Task**
Every component is built with both light and dark variants from the moment it is created. The theming system from task 1.3 makes this automatic if developers use theme tokens instead of hardcoded colors. Phase 7's dark mode audit is a verification step, not a creation step. If the audit finds 50 issues, something went wrong in Phases 2-6.

**7.4 — Accessibility Labels Are Built In, Not Bolted On**
Every interactive element receives an `accessibilityLabel` and appropriate `accessibilityRole` when it is created. Every image receives a descriptive label. This is enforced by code review — a PR without accessibility labels on new interactive elements will be rejected. Phase 7's accessibility audit verifies correctness and completeness, but the labels themselves are already there.

**7.5 — Performance Is Measured Continuously**
After Phase 1, every phase includes a performance check: cold start time, list scroll FPS, screen transition time, and memory usage. These are recorded in a shared spreadsheet or tracking document. If any metric regresses by more than 10% from the previous phase, the regression is investigated and fixed before the phase is closed. Performance is not "optimized at the end" — it is maintained throughout.

**7.6 — Every PR Is Reviewed Against the Spec**
The reviewer's first action is to find the relevant task in this document and verify that the PR satisfies the task's requirements. PRs that add unrequested features, skip error handling, omit tests, or deviate from the ADRs are rejected. The spec is the source of truth, not the developer's interpretation.

**7.7 — Demo at Every Phase Gate**
At the end of each phase, the app is demoed to stakeholders (product owner, designer, or other team members) on a physical device. Feedback is collected in writing. Critical feedback (bugs, UX failures) is resolved before the phase is marked complete. Non-critical feedback (preferences, nice-to-haves) is logged for consideration but does not block the phase.

**7.8 — No Undocumented Dependencies**
Every npm package added to the project must appear in the Bill of Materials (Section 6) or be explicitly justified in the PR that adds it. Random packages discovered via Stack Overflow are not added without team discussion. This prevents dependency bloat, license conflicts, and privacy manifest gaps.

**7.9 — Offline Is Not an Afterthought**
Every screen that displays server data must work offline from the moment it is built. This means: data is written to SQLite during the TanStack Query fetch cycle, queries fall back to SQLite when offline, and an offline indicator is visible to the user. This is part of the screen's definition of done, not a future enhancement.

**7.10 — Test Coverage Is a Phase Exit Criterion**
Each phase's task list includes unit test and E2E test tasks. The phase cannot exit until these tests pass. The minimum bar: every API interaction has a unit test, every critical user flow has a Maestro E2E test, and code coverage on new code is above 70% (measured per phase, not globally).

---

*This document governs the entire Skye iOS build. Changes require a formal change request reviewed by the project lead, with impact analysis on the dependency graph, risk register, and timeline.*

---



---

## Document Lineage

| Version | Author | Description |
|---------|--------|-------------|
| V1 | Steve | Initial rough spec — raw decomposition of intent into screens, acceptance criteria, constraints, and phases |
| V1.5 | Robert (review) | Added: Apple Compliance, Error Handling Contract, App Lifecycle Handling, Architecture Decision Records, Infrastructure Requirements. All V1 content preserved. |
| V2 | 12-Specialist Agent System | Full production spec: every screen, every token, every animation, every test, every compliance item specified to implementation-ready detail. 12 independent specialist agents each researched and authored their domain against current documentation. |

---

*This document governs the entire Skye iOS build. Every numeric target is a CI-enforced gate. Every design token is the authoritative value. Every screen state is a build requirement. When in doubt, this document is the source of truth. When this document is silent, refer to the Apple Human Interface Guidelines.*
