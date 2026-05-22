# Rapid-Fire Q&A — 100+ Questions

Use this for self-testing. Cover each question mentally before reading the answer.

---

## Swift Foundations

**1. What is the difference between `let` and `var`?**  
`let` declares a constant (immutable after initialization). `var` declares a variable (mutable). For value types, `let` prevents reassignment and mutation. For reference types, `let` prevents reassignment but still allows mutation of the object's properties.

**2. What is type inference?**  
Swift infers the type from the assigned value. `let x = 42` → `Int`. `let s = "hello"` → `String`. Explicit typing: `let x: Double = 42`.

**3. What is an optional in Swift?**  
An optional `T?` represents either a value of type `T` or `nil`. It's an enum: `Optional<T> { case some(T); case none }`. Forces you to explicitly handle the absence of a value.

**4. What is optional binding?**  
`if let value = optional { }` — safely unwraps optional. `guard let value = optional else { return }` — early exit if nil, value available after.

**5. Force unwrap — when is it acceptable?**  
`value!` crashes if nil. Acceptable only when nil is a programming error (should never happen in correct code), such as outlets after viewDidLoad, or resources guaranteed to exist at compile time. Prefer `guard let` or `XCTUnwrap` in tests.

**6. What is `nil` coalescing operator?**  
`value ?? default` — returns `value` if non-nil, else `default`. Shorthand for `value != nil ? value! : default`.

**7. Struct vs Class — 3 key differences.**  
(1) Value vs reference semantics. (2) Structs don't support inheritance (except protocol conformance). (3) Structs have automatic memberwise initializer. Both support protocols, properties, methods, and extensions.

**8. When do you choose a struct over a class?**  
Prefer structs when: the data is simple and should be copied (not shared), you don't need inheritance, value semantics are appropriate. Use classes for identity (same object referenced in multiple places), reference semantics, or when inheriting from UIKit/AppKit.

**9. What is Copy-on-Write?**  
CoW is an optimization for value types. Copying is deferred until mutation. Multiple variables can share the same underlying storage until one tries to mutate — at that point, a copy is made. Swift's Array, Dictionary, Set use CoW.

**10. What is ARC?**  
Automatic Reference Counting. Swift automatically tracks strong references to class instances and deallocates when count reaches 0. Not a garbage collector — deterministic, happens at compile time.

**11. What causes a retain cycle?**  
When two or more objects hold strong references to each other, preventing either from being deallocated. Fix: use `weak` or `unowned` for one direction of the reference.

**12. weak vs unowned — difference?**  
`weak`: optional, becomes nil when referenced object is deallocated (safe). `unowned`: non-optional, crashes if accessed after deallocation (use only when you're certain of lifetime). `weak` is safer; `unowned` has slightly less overhead.

**13. What is a protocol in Swift?**  
A blueprint of methods, properties, and requirements that a type can adopt. Like an interface in Java/C#. Supports default implementations via protocol extensions.

**14. What are associated types?**  
Placeholder types in protocols that are resolved when a type conforms. E.g., `protocol Container { associatedtype Element; var first: Element? { get } }`. Makes protocols generic without declaring the generic parameter at the protocol level.

**15. What is `some` vs `any`?**  
`some Protocol` (opaque type): hides the concrete type but the compiler knows it — enables static dispatch, usable in return position for generics. `any Protocol` (existential): type-erased, stored as a box with runtime overhead, concrete type unknown to compiler. Prefer `some` where possible.

**16. What is type erasure?**  
Wrapping a generic/protocol type to hide its specific type, allowing heterogeneous collections or return types. `AnyPublisher`, `AnyView`, `AnyHashable` are examples. Custom type erasure: `struct AnyFoo<T>` that wraps `FooProtocol` implementations.

**17. What is `@discardableResult`?**  
Silences the compiler warning when a function returns a value that the caller ignores. Use when it's valid to call the function for side effects without using the return value.

**18. What is `defer`?**  
`defer { }` schedules code to run when the current scope exits, regardless of how (normal return, throw, early return). Useful for cleanup: closing files, releasing locks.

**19. What is `rethrows`?**  
A function marked `rethrows` only throws if one of its closure arguments throws. If you pass a non-throwing closure, the function is considered non-throwing.

**20. What is a closure?**  
A self-contained block of code that can capture and store references to variables from the context in which it's defined. First-class value — can be passed as arguments, returned from functions, stored in variables.

---

## Concurrency

**21. What is a race condition?**  
When two or more threads access shared mutable state concurrently without synchronization, producing non-deterministic results. Fix: serial queue, actor, or mutex.

**22. What is a deadlock?**  
Two or more threads permanently blocked, each waiting for a resource held by the other. Classic example: calling `sync` on the current queue.

**23. What is GCD?**  
Grand Central Dispatch. Apple's C-level threading API. You submit tasks (blocks) to dispatch queues; GCD manages a thread pool. Main queue is serial and runs on the main thread. Custom queues can be serial or concurrent.

**24. Sync vs async dispatch — difference?**  
`sync`: submits work and blocks the calling thread until it completes. `async`: submits work and returns immediately.

**25. What is QoS (Quality of Service)?**  
Priority hint for GCD and Operation queues. Levels: `.userInteractive` (main thread/animations), `.userInitiated` (user-triggered, immediate), `.utility` (progress visible), `.background` (not time-critical).

**26. What is async/await?**  
Swift's structured concurrency. `async` marks a function that can suspend. `await` suspends the current task until the awaited value is ready without blocking the thread. Cleaner than callbacks, compiler-enforced structure.

**27. What is an actor?**  
A reference type with built-in mutual exclusion. Only one piece of code can access an actor's mutable state at a time. Accessed via `await` since crossing actor isolation is asynchronous.

**28. What is `@MainActor`?**  
Declares that a type or function must execute on the main thread. The compiler enforces this — calling a `@MainActor` function from a non-main-actor context requires `await`.

**29. What is `Sendable`?**  
A protocol marking types safe to share across concurrency domains (actors, tasks). Value types are implicitly Sendable. Classes must be explicitly marked and ensure thread safety.

**30. What is structured concurrency?**  
Tasks created within a scope (TaskGroup, async let) are automatically cancelled and awaited when the scope exits. Prevents "fire and forget" memory leaks and provides automatic error propagation.

---

## UIKit

**31. UIView lifecycle — key methods?**  
`init(frame:)` or `init(coder:)` → `awakeFromNib()` (if IB) → `didMoveToSuperview()` → `layoutSubviews()` → `draw(_:)`. `layoutSubviews()` called during layout pass; call `setNeedsLayout()` to trigger.

**32. UIViewController lifecycle order?**  
`init` → `loadView` → `viewDidLoad` → `viewWillAppear` → `viewWillLayoutSubviews` → `viewDidLayoutSubviews` → `viewDidAppear` → `viewWillDisappear` → `viewDidDisappear` → `deinit`

**33. What is Auto Layout?**  
Constraint-based layout system. Views define their position/size relative to other views or the safe area. The layout engine solves the constraint system at runtime. NSLayoutConstraint or NSLayoutAnchor API.

**34. What causes Auto Layout ambiguity?**  
Not enough constraints to determine a view's frame unambiguously. Debug with `UIView.hasAmbiguousLayout`. Fix by adding constraints until the system is fully specified.

**35. UITableView cell reuse — how does it work?**  
`dequeueReusableCell(withIdentifier:for:)` returns a recycled cell from a pool when it scrolls off screen, avoiding expensive allocation. Must `prepareForReuse()` to reset cell state before reconfiguration.

**36. What is a diffable data source?**  
`UITableViewDiffableDataSource` / `UICollectionViewDiffableDataSource`: data source driven by snapshots of identifiable items. Compute diffs automatically and animate changes. No more "invalid number of rows" crashes.

**37. What is the responder chain?**  
A chain of UIResponder objects (views → view controllers → application → app delegate). Events propagate up from the first responder until handled. First responder: the view/VC that has input focus.

**38. What are the three types of UIViewController presentation?**  
Push (navigation stack), modal (sheet/full screen), popover (iPad). Full-screen is deprecated for most uses; `.automatic` uses the current context.

**39. What is `viewSafeAreaInsetsDidChange`?**  
Called when the safe area insets change (device rotation, keyboard appearance). Use safe area layout guides to keep content away from notch, home indicator, status bar.

**40. programmatic UI vs storyboard vs XIB?**  
Programmatic: full code control, no merge conflicts, but verbose. Storyboard: visual, auto-generates connections, bad for large teams (merge hell). XIB: good for reusable components. Modern teams typically prefer programmatic or SwiftUI.

---

## SwiftUI

**41. What is the `body` property in SwiftUI?**  
The required computed property of `View` that describes the view's content and layout. It returns `some View`. Re-evaluated whenever state changes.

**42. What is the difference between `@State` and `@Binding`?**  
`@State`: source of truth, owned by the view, persists across re-renders. `@Binding`: reference to external state — changes propagate back to the owner. Use `$` prefix to pass binding.

**43. `@StateObject` vs `@ObservedObject`?**  
`@StateObject`: creates and owns the ViewModel — not recreated on parent re-renders. `@ObservedObject`: observes an externally-owned ViewModel — could be recreated. Rule: use `@StateObject` when the view creates the ViewModel; `@ObservedObject` when it's passed in.

**44. What is `@EnvironmentObject`?**  
Injects a shared `ObservableObject` from a parent view using `.environmentObject()`. Any descendant can access it with `@EnvironmentObject var store: MyStore`. Fails at runtime if not injected.

**45. What is `@Observable` (iOS 17)?**  
A macro from the `Observation` framework. Replaces `ObservableObject + @Published` with fine-grained per-property tracking. Views only re-render when the properties they specifically read change.

**46. How does SwiftUI identify views?**  
By type and structural position in the view hierarchy (structural identity). `.id()` modifier can assign explicit identity. Changing identity destroys and recreates the view.

**47. Why does modifier order matter in SwiftUI?**  
Each modifier wraps the previous view. `.padding().background()` → background covers padded area. `.background().padding()` → background only behind content's intrinsic size.

**48. What is `PreferenceKey` in SwiftUI?**  
Mechanism for child views to communicate data up the hierarchy (opposite of environment). Child sets preference, parent reads via `onPreferenceChange`. Used for measuring child sizes, scroll offsets.

**49. `LazyVStack` vs `VStack`?**  
`VStack` renders all views at once. `LazyVStack` renders only visible views, creating/destroying as needed during scrolling. Use `LazyVStack` in `ScrollView` for large lists.

**50. How do you bridge UIKit and SwiftUI?**  
`UIViewRepresentable` wraps UIView for SwiftUI. `UIViewControllerRepresentable` wraps UIViewController. `UIHostingController` wraps SwiftUI View for UIKit. Coordinator pattern bridges UIKit delegates to SwiftUI bindings.

---

## Networking

**51. What are the three URLSession configuration types?**  
`default` (disk cache + cookies), `ephemeral` (memory only, no persistence), `background(withIdentifier:)` (continues transfers when app is suspended).

**52. How do you handle 401 Unauthorized responses?**  
Refresh the token. To avoid race conditions with multiple concurrent 401s, serialize the refresh using an actor — the first caller starts the refresh Task; subsequent callers await the same Task instead of starting new refreshes.

**53. What is certificate pinning?**  
Validating that the server's certificate or public key matches a known value stored in the app. Prevents MITM even if a CA is compromised. Implemented in `URLSessionDelegate.urlSession(_:didReceive:completionHandler:)`.

**54. How do you cancel a network request in async/await?**  
Cancel the `Task` that contains the network request. Swift automatically propagates cancellation to in-flight URLSession requests, which throw `URLError.cancelled`.

**55. What is `URLSession.data(for:)` vs `URLSession.bytes(from:)`?**  
`data(for:)` loads the full response into memory. `bytes(from:)` returns an `AsyncBytes` stream — process bytes incrementally for large files or streaming responses.

---

## Storage

**56. Keychain vs UserDefaults — when to use each?**  
Keychain: sensitive data (tokens, passwords) — encrypted, hardware-backed. UserDefaults: small, non-sensitive preferences (settings, flags) — plaintext plist.

**57. What is Core Data's threading rule?**  
Contexts must be accessed on their queue. `viewContext` → main queue only. Background writes → use `container.newBackgroundContext()` and call `perform`/`performAndWait`. NSManagedObjects are not thread-safe.

**58. Lightweight vs heavyweight migration in Core Data?**  
Lightweight: automatic for simple changes (add optional attribute, rename with mapping) — set `shouldMigrateStoreAutomatically`. Heavyweight: requires explicit `NSMappingModel` for complex transformations.

**59. What is SwiftData?**  
Apple's Swift-native persistence framework (iOS 17+). Uses `@Model` macro, `@Query` in SwiftUI. Built on Core Data but with Swift-friendly API and per-property observation.

**60. What directories should you use for different data?**  
Documents: user files (backed up). Library/Application Support: app data (backed up). Library/Caches: regenerable cache (not backed up, system may purge). tmp: ephemeral scratch space.

---

## Architecture

**61. What is Massive View Controller?**  
The anti-pattern in MVC where the ViewController accumulates networking, data transformation, state management, navigation, and delegate implementations — becoming large and untestable.

**62. What belongs in the ViewModel?**  
Business logic, data fetching, state management, data transformation (Model → display values). NOT UIKit imports, NOT view layout.

**63. What is the Coordinator pattern?**  
Removes navigation logic from ViewControllers. VCs signal intent via delegation/closure; Coordinator decides the next screen. Enables reusable VCs and centralized deep link handling.

**64. Dependency injection — three types?**  
Constructor injection (via `init` — preferred), property injection (via `var` property — for storyboard VCs), method injection (via function parameter — for one-off dependencies).

**65. What is the Repository pattern?**  
Abstracts data access behind a protocol. Domain layer calls the repository protocol; data layer implements it (network + cache). Domain never knows if data comes from API or local DB.

---

## Testing

**66. What is Arrange-Act-Assert?**  
Structure for tests: Arrange (setup SUT and dependencies), Act (call the method under test), Assert (verify outcome). Keeps tests focused and readable.

**67. Mock vs Stub?**  
Stub returns pre-configured values (no verification). Mock records calls AND verifies interactions (was method called with correct arguments?). In practice, most "mocks" combine both.

**68. How do you test async code in XCTest?**  
Mark test as `async throws`. XCTest handles the lifecycle. Use `await` directly in the test. For callback-based APIs, use `XCTestExpectation` + `waitForExpectations`.

**69. What is `XCTUnwrap`?**  
Safely unwraps an optional and fails the test (with file/line info) if nil. Better than force-unwrapping in tests.

**70. What should you NOT test?**  
Private implementation details, third-party library code, pure UI layout (use snapshot tests), simple data models with no logic, framework behavior.

---

## Performance

**71. What causes janky scrolling?**  
Main thread blocking (image loading, decoding, computation), off-screen rendering (cornerRadius + masksToBounds, shadows), excessive view creation, non-reused cells.

**72. What is off-screen rendering?**  
GPU renders a layer in a separate offscreen buffer before compositing. Expensive. Triggered by: `cornerRadius + masksToBounds`, shadows without `shadowPath`, complex masks. Fix: set `shadowPath`, rasterize static rounded views.

**73. How do you optimize app launch time?**  
Minimize +load methods, reduce dynamic framework count, defer non-critical initialization to background, use lazy singletons, reduce work in `didFinishLaunching`.

**74. How do you find memory leaks?**  
Instruments → Leaks template, Xcode Debug Memory Graph, `deinit` print statements, watching Allocations for growing persistent counts.

**75. What is `shouldRasterize` and when should you use it?**  
Caches a layer as a bitmap on the GPU. Useful for complex static layers (rounded cards that don't change). Wasteful for frequently-changing content. Must set `rasterizationScale = UIScreen.main.scale`.

---

## Security

**76. Why not store tokens in UserDefaults?**  
Plaintext plist file. Readable from jailbroken devices and backups. Always use Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`.

**77. What does `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` mean?**  
Item only readable when device is unlocked. Not synced to iCloud. Not backed up. Best option for auth tokens.

**78. What is ATS?**  
App Transport Security. Enforces HTTPS with TLS 1.2+, perfect forward secrecy, SHA-256. Enabled by default since iOS 9. Don't use `NSAllowsArbitraryLoads`.

**79. Certificate pinning — operational risk?**  
Hard-coded cert hashes require app update when cert rotates. Mitigate: pin multiple certs, pin public key hash (survives cert renewal with same key), have server-side override mechanism.

**80. How do you prevent sensitive data in screenshots?**  
Listen for `UIApplication.willResignActiveNotification`, overlay blur view. Remove on `didBecomeActiveNotification`.

---

## App Lifecycle

**81. Five app states?**  
Not Running, Inactive (foreground, no events), Active, Background (brief execution), Suspended (memory only, no CPU).

**82. When is `willTerminate` called?**  
Only when app is in background and system needs to terminate it specifically, or when user force-quits. Not called during normal suspension. Don't rely on it for saves.

**83. AppDelegate vs SceneDelegate?**  
AppDelegate: app-level lifecycle (one-time setup, push tokens, background scheduling). SceneDelegate: per-scene (window, deep links, scene lifecycle). Introduced in iOS 13 for multi-window iPad support.

**84. How does background fetch work?**  
Register `BGAppRefreshTask`, schedule with `BGTaskScheduler`, system invokes at optimal times. Must call `setTaskCompleted` when done. Schedule next fetch in the handler.

**85. Universal links vs URL schemes?**  
URL schemes: custom protocol, no server needed, any app can register same scheme. Universal Links: HTTPS URLs validated via AASA file on server, secure, falls back to website. Prefer universal links.

---

## System Design

**86. How do you approach a system design interview?**  
Clarify requirements → define data model → design components/layers → caching strategy → offline strategy → error/retry → performance → testing.

**87. Cursor vs offset pagination?**  
Cursor-based is preferred: not affected by insertions between pages, no duplicate/missing items. Offset pagination drifts when items are inserted.

**88. How do you design for offline support?**  
Local first: show cached data immediately, sync in background. Optimistic updates for write operations. Sync queue for offline mutations. Handle conflicts on sync.

**89. What is optimistic update?**  
Apply change to local state immediately (instant UI), make network call, rollback on failure. Used in social apps for likes, follows, message sending.

**90. How do you ensure analytics aren't lost on crash?**  
Persist events to local store before sending. Delete only after successful send. Process unsent events on next app launch.

---

## Advanced Topics

**91. What is Combine?**  
Apple's reactive programming framework. Publishers emit values over time; subscribers receive them. Combines well with URLSession and NotificationCenter. Superseded in many cases by async/await but still widely used.

**92. What is `@Published`?**  
Property wrapper in Combine. Creates a publisher for the property. Changes emit to subscribers. Used with `ObservableObject` to notify views of state changes.

**93. What is Swift Package Manager?**  
Apple's first-party dependency manager. Manages dependencies via `Package.swift`. Supports binary dependencies (XCFramework), local packages for modularization, and resource bundles.

**94. What are Swift macros?**  
Swift 5.9+ feature. Compile-time code generation using `@attached` or `freestanding` macros. Examples: `@Observable` generates observation tracking code, `@Model` in SwiftData generates persistence code.

**95. What is Objective-C interoperability?**  
Swift can call ObjC and vice versa via `@objc` annotation and the ObjC runtime. `@objc class`, `@objc func`. Required for delegate patterns, KVO, Selector-based APIs. `NSObject` subclass for full ObjC compatibility.

**96. What is KVO?**  
Key-Value Observing. Obj-C mechanism for observing property changes. Available in Swift for `NSObject` subclasses. Replaced by Combine `@Published` and `@Observable` in modern code.

**97. What is `AsyncSequence`?**  
Protocol for asynchronous sequences iterated with `for await`. Alternative to Combine for async data streams. URLSession bytes, FileHandle lines conform to it.

**98. What are XCFramework?**  
Binary framework format that bundles multiple architectures (arm64, x86_64) and platforms (iOS device, simulator, macOS). Created with `xcodebuild -create-xcframework`. Used for closed-source binary distribution.

**99. What is Sendable in practice?**  
Types conforming to `Sendable` can safely cross actor isolation boundaries. Value types are implicitly Sendable. Reference types must be explicitly declared `@unchecked Sendable` (manual guarantee) or truly immutable. The compiler warns when non-Sendable values cross concurrency boundaries.

**100. What is the `Observation` framework?**  
iOS 17+. Provides `@Observable` macro for fine-grained, per-property observation. No manual `@Published` needed. SwiftUI automatically tracks which properties are read per view and only re-renders when those specific properties change.

---

## Bonus Questions

**101. What is `lazy var` vs computed property?**  
`lazy var` is computed once on first access and stored. Subsequent accesses return the stored value. Computed property is evaluated every time it's accessed — no storage.

**102. What is `@autoclosure`?**  
Wraps an expression in a closure automatically. The closure is lazily evaluated only when called. Used in `assert`, `&&`, `||` (short-circuit evaluation).

**103. What is property observer?**  
`willSet` (called before value changes, `newValue` available) and `didSet` (called after value changes, `oldValue` available). Don't trigger during initialization.

**104. What is a convenience initializer?**  
`convenience init` — must call a designated `init` of the same class (via `self.init()`). Provides alternative ways to create instances with default/derived parameters.

**105. What is `@escaping` closure?**  
A closure that outlives the function that received it — stored, used after the function returns. Must be marked `@escaping`. Non-escaping closures (default) are destroyed when the function returns — safer, no capture list needed.

---

## Quick Self-Test Strategy

1. Read each question header
2. Answer mentally (or verbally out loud — simulates real interview)
3. Check against the answer
4. Flag questions you struggled with
5. Revisit flagged questions the next day
6. Prioritize questions from modules matching the job description
