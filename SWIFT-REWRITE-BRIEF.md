# Swift Rewrite Brief — Context Engineering Document

> **Purpose:** This document is the single reference for rewriting the Skye V2 spec from React Native to Swift native. Every rewrite agent reads this before touching a section. It captures all decisions, the complete technology mapping, rewrite rules, and what to preserve vs. replace.

---

## 1. Project Context

**Skye** is an AI-powered CRM for real estate agents. It exists today as a Next.js web app with 129 API routes, backed by Supabase (Postgres + Auth + RLS) and Claude AI (Haiku/Sonnet/Opus). The V2 spec defines a native iOS companion app that connects to this same backend.

The original V2 spec was written for **React Native + Expo**. We are rewriting it for **Swift native**.

### 1.1 Decisions Made (Locked)

| Decision | Value | Rationale |
|---|---|---|
| Language | Swift 5.9+ | Native iOS development language |
| UI Framework | SwiftUI-first, UIKit for performance-critical views | SwiftUI is mature at iOS 17. UIKit for `UICollectionView` with 500+ item lists where SwiftUI `List`/`LazyVStack` may not hit 60fps. |
| Minimum iOS | **iOS 17.0** | Unlocks `@Observable`, improved `NavigationStack`, better `ScrollView`, `SheetPresentationController` detents. Covers iPhone XS (2018) and newer — 7+ years of hardware. |
| Architecture | MVVM with Repository pattern | Standard Swift/SwiftUI architecture. ViewModels are `@Observable` classes. Repositories abstract data sources (network + local cache). |
| Android strategy | Kotlin native, built later as separate codebase | iOS is primary platform (real estate agents overwhelmingly use iPhone). Android will be built against the same spec and API, reimplemented in Kotlin. Not React Native cross-platform. |
| Backend | Unchanged — same Next.js API routes, same Supabase, same Claude | The native app is a client. It does not modify the backend. |
| Build system | Xcode 16+ with Swift Package Manager (SPM) | No CocoaPods unless a critical dependency requires it. SPM is the standard. |
| CI/CD | Xcode Cloud or GitHub Actions with fastlane | Replaces Expo EAS Build entirely. |
| Distribution | TestFlight → App Store | Standard Apple distribution. |

### 1.2 What the Original Spec Covers

12 sections, 20,000 lines, 964KB:

| # | Section | Original File |
|---|---|---|
| 1 | UX Architecture & Interaction Design | `01_ux_architecture.md` (94KB) |
| 2 | Visual Design System | `02_visual_design.md` (62KB) |
| 3 | Chat Engine & AI Interface | `03_chat_engine.md` (80KB) |
| 4 | CRM Screens — People, Contact Detail, Properties | `04_crm_screens.md` (70KB) |
| 5 | Authentication, Security & Data Protection | `05_auth_security.md` (91KB) |
| 6 | Push Notifications | `06_push_notifications.md` (84KB) |
| 7 | Performance, Infrastructure & Build Pipeline | `07_performance.md` (100KB) |
| 8 | Offline Architecture & State Management | `08_offline_state.md` (66KB) |
| 9 | Camera, OCR, Media & Studio | `09_camera_ocr.md` (88KB) |
| 10 | App Store Compliance & Submission | `10_compliance.md` (68KB) |
| 11 | Testing Strategy, QA & Acceptance Criteria | `11_testing_qa.md` (68KB) |
| 12 | Build Roadmap & Phased Implementation | `12_build_roadmap.md` (90KB) |

All original sections are at: `/Users/nikositaras/Downloads/skye_v2_sections/`

---

## 2. Complete Technology Mapping

### 2.1 Libraries: React Native → Swift Native

| Concern | React Native (Original) | Swift Native (Rewrite) | Notes |
|---|---|---|---|
| **UI Framework** | React Native JSX + StyleSheet | SwiftUI `View` protocol | SwiftUI-first. Drop to UIKit via `UIViewRepresentable` / `UIViewControllerRepresentable` only when SwiftUI can't hit performance targets. |
| **State (server/async)** | TanStack Query v5 | Custom `Repository` + `@Observable` ViewModels + `URLSession` | No direct equivalent. Build a lightweight caching/fetching layer with `actor`-based repositories. Use `AsyncStream` for reactive updates. |
| **State (client/UI)** | Zustand v4 with persist middleware | `@Observable` classes (iOS 17) + `@AppStorage` / custom persistence | `@Observable` replaces Zustand entirely. Persist via `@AppStorage` (UserDefaults), Keychain, or GRDB. |
| **Navigation** | React Navigation 6 (native stack, bottom tabs) | `NavigationStack` + `TabView` (SwiftUI) | SwiftUI native navigation. Path-based navigation with `NavigationPath` for programmatic control. |
| **Local database** | expo-sqlite | **GRDB.swift** | GRDB is the standard Swift SQLite library. WAL mode, migrations, type-safe queries, Combine/async support. |
| **Secure storage** | expo-secure-store | **KeychainAccess** (or raw Keychain Services) | `KeychainAccess` is a thin, well-maintained wrapper. Alternatively use Security framework directly. |
| **Fast KV storage** | react-native-mmkv | **UserDefaults** (simple) or **MMKV** (Swift wrapper exists) | For the use cases in the spec (UI preferences, badge counts), `UserDefaults` is sufficient. MMKV if sub-millisecond reads matter. |
| **HTTP client** | fetch / axios | **URLSession** (built-in) | Native `URLSession` with `async/await`. No third-party HTTP library needed. |
| **SSE streaming** | Custom EventSource polyfill | **URLSession bytes** (`URLSession.AsyncBytes`) or **LDSwiftEventSource** | `URLSession`'s `bytes` property on `URLSessionDataTask` gives an `AsyncSequence` of bytes — parse SSE frames from that. Or use LDSwiftEventSource for a clean abstraction. |
| **Image loading/caching** | expo-image | **Nuke** (recommended) or **Kingfisher** | Nuke is the modern standard: async/await native, SwiftUI `LazyImage` view, pipeline-based, blurhash support via plugin, disk + memory caching. |
| **Camera** | react-native-vision-camera v4 | **AVFoundation** (`AVCaptureSession`, `AVCapturePhotoOutput`) | Direct framework access. No wrapper library needed. Full control over capture pipeline. |
| **Image processing** | expo-image-manipulator | **Core Image** (`CIImage`, `CIFilter`) + **UIImage** | Resize, rotate, compress — all built into the platform. `UIImage.jpegData(compressionQuality:)` for JPEG encoding. |
| **Animations** | react-native-reanimated v3 | **SwiftUI `.animation()`** / **`withAnimation()`** | SwiftUI's animation system is declarative and runs on the main thread. For complex gesture-driven animations, use `GestureState` + `Animation.spring()`. UIKit: `UIViewPropertyAnimator`. |
| **Spring curves** | Reanimated `withSpring()` with custom configs | **`Animation.spring(duration:bounce:)`** (iOS 17) | iOS 17's new spring API uses `duration` and `bounce` instead of damping/stiffness/mass. More intuitive. Map the original spec's spring presets to equivalent `duration`/`bounce` values. |
| **Gesture handling** | react-native-gesture-handler v2 | **SwiftUI `.gesture()` modifiers** / **UIGestureRecognizer** | `DragGesture`, `LongPressGesture`, `MagnifyGesture`, `TapGesture` — all built into SwiftUI. Compose with `.simultaneously()`, `.exclusively()`. |
| **Lists (high performance)** | @shopify/flash-list | **`UICollectionView`** with `UICollectionViewCompositionalLayout` + `UICollectionViewDiffableDataSource` wrapped in `UIViewControllerRepresentable` | For 500+ contacts at 60fps, `UICollectionView` is the proven path. SwiftUI `List` may work but has less control over cell recycling. Profile first, then decide. |
| **Lists (simple)** | React Native FlatList | **SwiftUI `List`** or **`LazyVStack` inside `ScrollView`** | For lists under 50-100 items, SwiftUI is fine. |
| **Bottom sheets** | @gorhom/bottom-sheet v5 | **`.sheet()` with `presentationDetents`** (iOS 16+) | Native SwiftUI sheet presentation with detents (`.medium`, `.large`, custom heights). No third-party library needed. |
| **Pull-to-refresh** | Custom PanGestureHandler implementation | **`.refreshable()`** modifier (SwiftUI) | Built-in. Custom animations can be applied via `RefreshControl` in UIKit if the branded animation is needed. |
| **Haptics** | expo-haptics | **`UIImpactFeedbackGenerator`**, **`UISelectionFeedbackGenerator`**, **`UINotificationFeedbackGenerator`** | Direct UIKit haptic API. Prepare generators on view appear for zero-latency feedback. |
| **Push notifications** | @notifee/react-native + PushNotificationIOS | **`UNUserNotificationCenter`** (built-in) | Direct framework access. No wrapper needed. Register categories, handle foreground/background, manage badge — all native API. |
| **Deep linking** | React Navigation linking config | **`onOpenURL()` modifier** + **`NSUserActivity`** | SwiftUI handles URL schemes and universal links natively. |
| **Network reachability** | @react-native-community/netinfo | **`NWPathMonitor`** (Network framework) | Built-in. `NWPathMonitor` provides real-time network status with path type (wifi, cellular, etc.) and constrained/expensive flags. |
| **Error monitoring** | @sentry/react-native | **sentry-cocoa** (`Sentry` SPM package) | Sentry's native iOS SDK. Same DSN, same project. |
| **Lottie animations** | lottie-react-native | **lottie-ios** (`Lottie` SPM package) | Same Lottie files, native Swift player. `LottieAnimationView` or SwiftUI `LottieView`. |
| **Markdown rendering** | react-native-markdown-display | **`AttributedString` with Markdown init** (iOS 15+) or **swift-markdown-ui** | `AttributedString(markdown:)` handles basic markdown. For richer rendering, `MarkdownUI` SPM package. |
| **Keyboard management** | react-native-keyboard-controller | **`ScrollViewReader` + `.scrollDismissesKeyboard()`** | SwiftUI handles keyboard avoidance automatically in most cases. `scrollDismissesKeyboard(.interactively)` for chat-style dismiss. |
| **Safe areas** | react-native-safe-area-context | **Built-in** (`.safeAreaInset()`, `GeometryReader`) | SwiftUI respects safe areas by default. No library needed. |
| **Testing** | Jest + @testing-library/react-native + Maestro | **XCTest** (unit/integration) + **XCUITest** (UI tests) + **Maestro** (E2E) | Maestro works with native iOS apps — keep it for E2E. XCTest replaces Jest. |
| **Build/CI** | Expo EAS Build + GitHub Actions | **Xcode Cloud** or **GitHub Actions + fastlane** | fastlane for signing, building, uploading to TestFlight. |
| **Bundle analysis** | react-native-bundle-visualizer | **Xcode Build Report** + **Bloaty** (binary size analysis) | Monitor app binary size (not JS bundle — there is none). Target: < 30MB total app download size. |
| **JS engine** | Hermes (bytecode compilation) | **N/A** — Swift compiles to native machine code | No JS engine. No bridge. No serialization overhead. This is a pure win. |

### 2.2 Spring Animation Mapping

The original spec defines named spring presets with `damping`, `stiffness`, `mass` values for Reanimated. These must be converted to SwiftUI's `Animation.spring(duration:bounce:)` API (iOS 17+).

| Original Preset | Reanimated Config | SwiftUI Equivalent | Usage |
|---|---|---|---|
| `snappy` | damping: 20, stiffness: 300, mass: 0.8 | `.spring(duration: 0.3, bounce: 0.15)` | Button presses, toggles, small UI responses |
| `gentle` | damping: 20, stiffness: 120, mass: 1.0 | `.spring(duration: 0.5, bounce: 0.1)` | Content transitions, page pushes, card reveals |
| `bouncy` | damping: 12, stiffness: 150, mass: 0.9 | `.spring(duration: 0.4, bounce: 0.3)` | Playful elements, FAB press, tab icon, empty states |
| `heavy` | damping: 28, stiffness: 200, mass: 1.2 | `.spring(duration: 0.45, bounce: 0.05)` | Large elements, bottom sheet, modal, full-screen |
| `sheet` | damping: 30, stiffness: 250, mass: 0.9, overshootClamping: true | `.spring(duration: 0.35, bounce: 0.0)` | Bottom sheet snap animations |
| `navigation` | damping: 25, stiffness: 180, mass: 1.0, overshootClamping: true | `.spring(duration: 0.4, bounce: 0.0)` | Screen push/pop transitions |
| `keyboard` | damping: 30, stiffness: 250, mass: 0.8, overshootClamping: true | `.spring(duration: 0.3, bounce: 0.0)` | Keyboard avoidance translations |
| `micro` | damping: 15, stiffness: 400, mass: 0.5 | `.spring(duration: 0.2, bounce: 0.2)` | Icon state changes, indicator dots |

### 2.3 Architecture Pattern

```
┌─────────────────────────────────────────────────┐
│                   SwiftUI Views                  │
│  (View structs, @State, @Binding, @Environment) │
└──────────────────────┬──────────────────────────┘
                       │ observes
                       ▼
┌─────────────────────────────────────────────────┐
│              @Observable ViewModels              │
│  (ContactsViewModel, ChatViewModel, etc.)        │
│  Owns UI state, calls repository methods,        │
│  exposes published properties for views.         │
└──────────────────────┬──────────────────────────┘
                       │ calls
                       ▼
┌─────────────────────────────────────────────────┐
│                  Repositories                    │
│  (ContactRepository, PropertyRepository, etc.)   │
│  Abstract data source: network + local cache.    │
│  Cache-first: return local → fetch remote →      │
│  update local → notify observers.                │
│  Implemented as Swift actors for thread safety.  │
└────────┬────────────────────────┬───────────────┘
         │                        │
         ▼                        ▼
┌─────────────────┐    ┌──────────────────┐
│  NetworkService  │    │  LocalDatabase   │
│  (URLSession)    │    │  (GRDB / SQLite) │
│  API calls,      │    │  Offline cache,  │
│  SSE streaming   │    │  mutation queue  │
└─────────────────┘    └──────────────────┘
```

### 2.4 State Management Pattern (replaces TanStack Query + Zustand)

```swift
// EXAMPLE PATTERN — ContactsViewModel

import SwiftUI
import Observation

@Observable
final class ContactsViewModel {
    // Published state
    var contacts: [Contact] = []
    var isLoading = false
    var error: AppError?
    var searchQuery = ""
    var filterState = ContactFilterState()

    // Dependencies (injected)
    private let repository: ContactRepository

    init(repository: ContactRepository) {
        self.repository = repository
    }

    // Cache-first fetch
    func loadContacts() async {
        isLoading = contacts.isEmpty // Only show loading if no cached data
        do {
            // 1. Serve from cache first (instant)
            contacts = try await repository.getCachedContacts(filters: filterState)

            // 2. Fetch fresh from network (background)
            let fresh = try await repository.fetchContacts(filters: filterState)
            contacts = fresh
            error = nil
        } catch {
            // If we have cached data, keep showing it
            if contacts.isEmpty {
                self.error = AppError(error)
            }
        }
        isLoading = false
    }

    // Optimistic mutation
    func updateContact(_ contact: Contact, with changes: ContactUpdate) async {
        // 1. Snapshot for rollback
        let snapshot = contacts

        // 2. Optimistic update
        if let index = contacts.firstIndex(where: { $0.id == contact.id }) {
            contacts[index].apply(changes)
        }

        // 3. Persist to server (or queue offline)
        do {
            try await repository.updateContact(contact.id, changes: changes)
        } catch {
            // 4. Rollback on failure
            contacts = snapshot
            self.error = AppError(error)
        }
    }
}
```

```swift
// EXAMPLE PATTERN — ContactRepository (actor for thread safety)

actor ContactRepository {
    private let networkService: NetworkService
    private let database: AppDatabase  // GRDB

    // Cache-first
    func getCachedContacts(filters: ContactFilterState) async throws -> [Contact] {
        try await database.fetchContacts(filters: filters)
    }

    func fetchContacts(filters: ContactFilterState) async throws -> [Contact] {
        let remote = try await networkService.get("/api/crm/contacts", params: filters.queryParams)
        let contacts: [Contact] = try JSONDecoder().decode([Contact].self, from: remote)
        // Persist to local cache
        try await database.upsertContacts(contacts)
        return contacts
    }

    func updateContact(_ id: String, changes: ContactUpdate) async throws {
        let networkAvailable = NetworkMonitor.shared.isConnected
        if networkAvailable {
            try await networkService.patch("/api/contacts/\(id)", body: changes)
            // Invalidate cache
            try await database.updateContact(id, with: changes)
        } else {
            // Queue for offline sync
            try await database.enqueueMutation(.updateContact(id: id, changes: changes))
        }
    }
}
```

### 2.5 Offline Queue Pattern (replaces SQLite mutation_queue)

Same concept as original spec, reimplemented in Swift with GRDB:

```swift
// Mutation queue record (GRDB)
struct QueuedMutation: Codable, FetchableRecord, PersistableRecord {
    static let databaseTableName = "mutation_queue"

    var id: String  // UUID
    var type: String
    var endpoint: String
    var method: String  // POST, PATCH, DELETE
    var body: Data  // JSON-encoded
    var createdAt: Date
    var status: MutationStatus  // pending, processing, completed, failed
    var retryCount: Int
    var maxRetries: Int
    var nextRetryAt: Date?
    var error: String?
    var idempotencyKey: String
    var relatedEntityId: String?
    var relatedEntityType: String?
}

enum MutationStatus: String, Codable, DatabaseValueConvertible {
    case pending, processing, completed, failed
}
```

### 2.6 SSE Streaming Pattern (replaces EventSource polyfill)

```swift
// Chat streaming with URLSession AsyncBytes

func streamChat(message: String, sessionId: String) -> AsyncThrowingStream<ChatStreamEvent, Error> {
    AsyncThrowingStream { continuation in
        Task {
            var request = URLRequest(url: URL(string: "\(baseURL)/api/chat")!)
            request.httpMethod = "POST"
            request.setValue("text/event-stream", forHTTPHeaderField: "Accept")
            request.setValue("application/json", forHTTPHeaderField: "Content-Type")
            request.setValue("Bearer \(accessToken)", forHTTPHeaderField: "Authorization")
            request.httpBody = try JSONEncoder().encode(ChatRequest(message: message, sessionId: sessionId))

            let (bytes, response) = try await URLSession.shared.bytes(for: request)

            guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                continuation.finish(throwing: ChatError.serverError)
                return
            }

            var currentEvent = ""
            var currentData = ""

            for try await line in bytes.lines {
                if line.hasPrefix("event: ") {
                    currentEvent = String(line.dropFirst(7))
                } else if line.hasPrefix("data: ") {
                    currentData = String(line.dropFirst(6))
                } else if line.isEmpty {
                    // Empty line = end of SSE event
                    if let event = parseSSEEvent(event: currentEvent, data: currentData) {
                        continuation.yield(event)
                    }
                    currentEvent = ""
                    currentData = ""
                }
            }

            continuation.finish()
        }
    }
}
```

---

## 3. Rewrite Rules

### 3.1 What to PRESERVE (carry over unchanged)

- All **screen names, screen flows, and navigation graphs**
- All **UX behaviors** (tap interactions, swipe actions, long-press menus, pull-to-refresh behavior)
- All **design tokens** (color hex values, typography scale, spacing values, shadow definitions)
- All **API endpoint references** and request/response shapes
- All **acceptance criteria behaviors** (the "what," not the "how")
- All **haptic feedback mappings** (which haptic type fires on which interaction)
- All **accessibility requirements** (VoiceOver order, touch targets, Dynamic Type rules, Reduce Motion)
- All **offline behaviors** (cache-first, mutation queue, conflict resolution, network state machine)
- All **push notification payloads** and deep link resolution logic
- All **App Store compliance requirements** (privacy manifest, nutrition labels, review preparation)
- All **wireframe layouts** and screen state machines (IDLE → LOADING → LOADED → EMPTY → ERROR → OFFLINE)

### 3.2 What to REPLACE

- All **TypeScript/JSX code samples** → Swift code samples
- All **React Native library references** → Swift native equivalents (per mapping in Section 2.1)
- All **npm package names** → SPM package names or framework references
- All **build pipeline references** (Expo, EAS, Metro) → Xcode, SPM, fastlane, Xcode Cloud
- All **testing references** (Jest, @testing-library) → XCTest, XCUITest
- All **state management patterns** (TanStack Query hooks, Zustand stores) → @Observable ViewModels, Repository actors
- All **animation configurations** (Reanimated withSpring/withTiming) → SwiftUI Animation (per mapping in Section 2.2)
- All **component patterns** (React.memo, useCallback, useMemo) → SwiftUI `@State`, `@Binding`, `Equatable` conformance
- All **StyleSheet** definitions → SwiftUI modifiers and `ViewModifier` structs
- All **gesture handler** references → SwiftUI `.gesture()` modifiers or UIKit gesture recognizers
- All **FlatList/FlashList** references → SwiftUI `List` / `LazyVStack` or `UICollectionView`
- **Hermes/JS engine** references → remove entirely (native compilation, no JS engine)
- **Bridge/serialization** references → remove entirely (no bridge in native Swift)
- **Bundle size** concerns → replace with **app binary size** and **download size** concerns

### 3.3 What to REMOVE (not applicable in Swift native)

- All references to the **JS thread vs UI thread** distinction (Swift runs on main thread + structured concurrency with `Task` and actors)
- All references to **Hermes bytecode compilation**
- All references to **Metro bundler**
- All references to **Expo Go** or **development client builds**
- All references to **React Native bridge overhead**
- All references to **`useRef` vs `useState`** performance distinctions
- All references to **`requestAnimationFrame`** for render batching (SwiftUI handles this)
- All references to **Flipper** (use Xcode Instruments instead)

### 3.4 Section-Specific Rewrite Guidance

| Section | Rewrite Intensity | Key Changes |
|---|---|---|
| **1. UX Architecture** | Medium | Replace Reanimated spring configs with SwiftUI Animation equivalents. Replace gesture handler refs with SwiftUI gestures. Replace haptic library with UIKit feedback generators. Keep all UX behaviors, touch targets, interaction tables. |
| **2. Visual Design** | Low | Token values stay identical. Replace StyleSheet/JS token format with Swift structs/enums. Replace component token consumption patterns with SwiftUI `ViewModifier` and `EnvironmentValues`. |
| **3. Chat Engine** | High | Replace entire SSE implementation with URLSession AsyncBytes. Replace token buffer/RAF pattern with `AsyncStream` + SwiftUI `@Observable` updates. Replace message list FlashList with SwiftUI `ScrollView` + `LazyVStack` (inverted). |
| **4. CRM Screens** | High | Replace FlashList with `UICollectionView` (wrapped) or SwiftUI `List`. Replace mutation hooks with ViewModel methods. Replace all component code. Keep wireframes, layouts, engagement health dot logic. |
| **5. Auth & Security** | High | Replace `@react-native-google-signin` with `GoogleSignIn-iOS` SPM package (same underlying Google SDK, different API surface). Replace expo-secure-store with KeychainAccess. Replace all token management code. |
| **6. Push Notifications** | Medium | Simpler in native — replace Notifee with direct `UNUserNotificationCenter`. Replace deep link handling with SwiftUI `.onOpenURL()`. Keep all notification types, payloads, and badge logic. |
| **7. Performance** | High | Completely different build pipeline. Remove JS bundle concerns. Add app binary size concerns. Replace Flipper with Instruments. Replace EAS Build with Xcode Cloud/fastlane. Cold launch budget drops (native is faster). |
| **8. Offline & State** | High | Replace TanStack Query with Repository + @Observable pattern. Replace Zustand with @Observable. Replace expo-sqlite with GRDB. Keep all offline queue logic, conflict resolution, cache eviction rules. |
| **9. Camera & OCR** | Medium | Replace VisionCamera with AVFoundation. Replace expo-image-picker with PHPickerViewController. Replace expo-image with Nuke. Keep all UX flows, permission screens, OCR pipeline. |
| **10. Compliance** | Low | Nearly identical. Update SDK names in privacy manifest audit. Privacy manifest XML stays the same. App Store submission process is identical. |
| **11. Testing** | High | Replace Jest with XCTest. Replace all test code samples with Swift. Replace coverage tooling with Xcode coverage reports. Keep test descriptions and coverage thresholds. Maestro E2E tests may survive with minor changes. |
| **12. Build Roadmap** | High | Replace all dependency references. Rewrite Phase 0 tasks (Xcode setup, SPM, no Expo). Rewrite Phase 1 tasks (Swift scaffold, not Expo scaffold). Task decomposition structure survives but file names and tooling change. |

---

## 4. File Structure (Swift Project)

```
Skye/
├── Skye.xcodeproj
├── Package.swift (or use Xcode project-level SPM)
├── Skye/
│   ├── SkyeApp.swift                    # @main entry point
│   ├── ContentView.swift                # Root TabView
│   ├── Info.plist
│   ├── PrivacyInfo.xcprivacy
│   │
│   ├── App/
│   │   ├── AppState.swift               # App-wide observable state
│   │   ├── AppDelegate.swift            # Push notification registration
│   │   └── SceneDelegate.swift          # Deep link handling (if needed)
│   │
│   ├── Theme/
│   │   ├── Colors.swift                 # Semantic color tokens
│   │   ├── Typography.swift             # Font/type scale
│   │   ├── Spacing.swift                # Spacing scale
│   │   ├── Shadows.swift                # Shadow definitions
│   │   └── Springs.swift                # Named spring animation presets
│   │
│   ├── Models/
│   │   ├── Contact.swift
│   │   ├── Property.swift
│   │   ├── ChatMessage.swift
│   │   ├── ChatSession.swift
│   │   ├── Task.swift
│   │   └── StudioContent.swift
│   │
│   ├── Services/
│   │   ├── NetworkService.swift         # URLSession wrapper, auth headers
│   │   ├── AuthService.swift            # Google OAuth, token management
│   │   ├── ChatStreamService.swift      # SSE streaming
│   │   ├── UploadService.swift          # Multipart upload with progress
│   │   ├── PushNotificationService.swift
│   │   ├── NetworkMonitor.swift         # NWPathMonitor wrapper
│   │   └── KeychainService.swift        # Keychain wrapper
│   │
│   ├── Database/
│   │   ├── AppDatabase.swift            # GRDB setup, migrations
│   │   ├── MutationQueue.swift          # Offline mutation queue
│   │   ├── QueueProcessor.swift         # FIFO queue drain on reconnect
│   │   └── CacheManager.swift           # Eviction, size tracking
│   │
│   ├── Repositories/
│   │   ├── ContactRepository.swift      # Network + cache for contacts
│   │   ├── PropertyRepository.swift
│   │   ├── ChatRepository.swift
│   │   ├── TaskRepository.swift
│   │   └── StudioRepository.swift
│   │
│   ├── ViewModels/
│   │   ├── ContactsViewModel.swift
│   │   ├── ContactDetailViewModel.swift
│   │   ├── PropertiesViewModel.swift
│   │   ├── PropertyDetailViewModel.swift
│   │   ├── ChatViewModel.swift
│   │   ├── ChatListViewModel.swift
│   │   ├── StudioViewModel.swift
│   │   ├── AuthViewModel.swift
│   │   └── SettingsViewModel.swift
│   │
│   ├── Views/
│   │   ├── People/
│   │   │   ├── PeopleListView.swift
│   │   │   ├── ContactDetailView.swift
│   │   │   ├── AddContactView.swift
│   │   │   ├── EditContactSheet.swift
│   │   │   └── ContactFilterSheet.swift
│   │   ├── Properties/
│   │   │   ├── PropertyListView.swift
│   │   │   ├── PropertyDetailView.swift
│   │   │   └── PropertyFilterSheet.swift
│   │   ├── Chat/
│   │   │   ├── ChatListView.swift
│   │   │   ├── ChatConversationView.swift
│   │   │   ├── ChatBubbleView.swift
│   │   │   ├── ChatInputBar.swift
│   │   │   └── TypingIndicatorView.swift
│   │   ├── Studio/
│   │   │   ├── StudioHomeView.swift
│   │   │   ├── ContentInputForm.swift
│   │   │   ├── GenerationOutputView.swift
│   │   │   └── StudioHistoryView.swift
│   │   ├── Camera/
│   │   │   ├── CameraView.swift
│   │   │   ├── CameraPermissionView.swift
│   │   │   ├── PhotoPreviewView.swift
│   │   │   ├── OCRReviewView.swift
│   │   │   └── OCRResultView.swift
│   │   ├── Settings/
│   │   │   ├── SettingsView.swift
│   │   │   ├── ProfileSettingsView.swift
│   │   │   ├── NotificationSettingsView.swift
│   │   │   └── DeleteAccountView.swift
│   │   ├── Auth/
│   │   │   ├── LoginView.swift
│   │   │   ├── WelcomeView.swift
│   │   │   └── BiometricUnlockView.swift
│   │   └── Components/
│   │       ├── ContactAvatar.swift
│   │       ├── EngagementDot.swift
│   │       ├── StreetViewImage.swift
│   │       ├── NetworkBanner.swift
│   │       ├── SkeletonView.swift
│   │       ├── BrandedRefreshView.swift
│   │       ├── SyncStatusBadge.swift
│   │       └── EmptyStateView.swift
│   │
│   ├── Utilities/
│   │   ├── DeepLinkHandler.swift
│   │   ├── HapticManager.swift
│   │   ├── ImageProcessor.swift
│   │   └── DateFormatters.swift
│   │
│   └── Resources/
│       ├── Assets.xcassets
│       ├── Lottie/
│       │   ├── skye-thinking.json
│       │   ├── success-check.json
│       │   └── card-scan.json
│       └── Localizable.strings
│
├── SkyeTests/
│   ├── Services/
│   ├── Repositories/
│   ├── ViewModels/
│   └── Database/
│
└── SkyeUITests/
    ├── AuthFlowTests.swift
    ├── ContactFlowTests.swift
    ├── ChatFlowTests.swift
    └── CameraFlowTests.swift
```

---

## 5. Dependencies (SPM Packages)

| Package | SPM URL | Purpose | Required? |
|---|---|---|---|
| **GRDB.swift** | `https://github.com/groue/GRDB.swift` | SQLite database (offline cache, mutation queue) | Yes |
| **KeychainAccess** | `https://github.com/kishikawakatsumi/KeychainAccess` | Keychain wrapper (secure token storage) | Yes |
| **Nuke** | `https://github.com/kean/Nuke` | Image loading, caching, blurhash | Yes |
| **NukeUI** | `https://github.com/kean/NukeUI` | SwiftUI `LazyImage` view | Yes |
| **GoogleSignIn-iOS** | `https://github.com/google/GoogleSignIn-iOS` | Google OAuth native flow | Yes |
| **Sentry** | `https://github.com/getsentry/sentry-cocoa` | Error monitoring and crash reporting | Yes |
| **Lottie** | `https://github.com/airbnb/lottie-ios` | Lottie animations (branded refresh, thinking, success) | Yes |
| **LDSwiftEventSource** | `https://github.com/launchdarkly/swift-eventsource` | SSE client (alternative to raw URLSession bytes) | Optional |
| **MarkdownUI** | `https://github.com/gonzalezreal/swift-markdown-ui` | Rich markdown rendering (Studio output, chat) | Yes |
| **swift-collections** | `https://github.com/apple/swift-collections` | OrderedDictionary, Deque (for efficient data structures) | Optional |

Total third-party dependencies: **8-10** (vs. 25+ npm packages in the React Native version).

---

## 6. Performance Budget Adjustments

Native Swift eliminates JS engine overhead entirely. Adjusted budgets:

| Metric | React Native Budget | Swift Native Budget | Rationale |
|---|---|---|---|
| Cold launch to interactive | < 2,000ms | **< 1,000ms** | No Hermes load, no JS bundle parse, no bridge init. Native compilation starts executing immediately. |
| App download size | < 15MB (JS bundle only) | **< 30MB** (total app) | No JS bundle. Binary is compiled machine code + assets. Target competitive with native CRM apps. |
| App memory (500 contacts) | < 150MB | **< 100MB** | No JS runtime memory overhead (~30-50MB). No bridge serialization buffers. |
| Scroll FPS | >= 58fps JS thread | **>= 60fps** (no JS thread) | No JS thread bottleneck. All rendering on main thread with Swift structured concurrency. |
| Time to first meaningful paint | < 500ms | **< 300ms** | Cached data loads from GRDB synchronously on main thread. |

---

*This document is the complete context for the Swift rewrite. Every agent reads this, then reads the original section it's rewriting, then produces the Swift-native version following the rules above.*
