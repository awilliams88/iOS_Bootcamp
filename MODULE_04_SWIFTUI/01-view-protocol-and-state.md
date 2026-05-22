# SwiftUI — View Protocol & State Management

Understanding how SwiftUI manages state and renders views is the foundation of everything. Get this wrong and your app will have subtle bugs, performance issues, and confusing behavior.

---

## The View Protocol

```swift
public protocol View {
    associatedtype Body: View
    @ViewBuilder var body: Self.Body { get }
}
```

### Mental Model

SwiftUI views are **value types** (structs) that describe the UI — they are **not** the UI itself. SwiftUI takes your description and manages the actual rendering. When state changes, SwiftUI calls `body` again and diffs the result against the previous description to compute minimal updates.

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") { count += 1 }
        }
    }
}
```

> 💡 **Mental Model:** A SwiftUI `View` is like a function from state → UI description. The framework calls this function whenever state changes and figures out what changed.

### Key Properties of Views

- **Struct-based** — cheap to create, immutable
- **Value semantics** — no shared mutable state in the view itself
- **Composable** — small views combine into larger ones
- **Declarative** — describe what, not how

> ⚠️ **Pitfall:** Never store mutable state in a plain `var` property in a view — it won't cause redraws and the value resets on every body call.

```swift
// ❌ WRONG — mutating this won't redraw the view
struct BadView: View {
    var count = 0 // Not a source of truth
    var body: some View {
        Button("Tap") { count += 1 } // Won't compile (mutating struct)
    }
}

// ✅ CORRECT
struct GoodView: View {
    @State private var count = 0
    var body: some View {
        Button("Tap") { count += 1 }
    }
}
```

---

## @State

`@State` is for **private, local, value-type** state owned by a view.

```swift
@State private var isExpanded = false
@State private var searchText = ""
@State private var selectedTab = 0
```

### How It Works

SwiftUI allocates storage for `@State` values **outside** the view struct (in the framework's managed storage). When the value changes, SwiftUI invalidates the view and re-renders `body`.

```swift
struct ToggleView: View {
    @State private var isOn = false

    var body: some View {
        VStack {
            Toggle("Feature", isOn: $isOn)
            if isOn {
                Text("Feature is enabled")
                    .transition(.opacity)
            }
        }
        .animation(.default, value: isOn)
    }
}
```

### Rules for @State

1. Always mark `private` — it's local to this view
2. Use for simple value types (Bool, Int, String, structs)
3. Initialize with a default value
4. The `$` prefix gives you a `Binding<T>` to pass to child views

> 🎯 **Interview Answer:** "@State is for view-local state. SwiftUI stores the actual value outside the struct so it persists across re-renders. The view struct is recreated on each render, but the @State storage persists. Use it for ephemeral UI state like toggle positions, text field input, or selection."

---

## @Binding

`@Binding` creates a two-way connection to state owned by a parent view. It has **no storage** — it's just a reference to existing state.

```swift
struct ParentView: View {
    @State private var isOn = false

    var body: some View {
        ChildView(isOn: $isOn) // Pass binding with $ prefix
    }
}

struct ChildView: View {
    @Binding var isOn: Bool

    var body: some View {
        Toggle("Toggle", isOn: $isOn)
    }
}
```

### When to Use @Binding

- Child view needs to **read and write** parent state
- Components like TextField, Toggle, Picker that need two-way data flow

> ⚠️ **Pitfall:** Never pass `@Binding` to views that don't need to write. If the child only reads, use a plain `let` property. This reduces unnecessary dependency.

```swift
// Only reading — use let, not @Binding
struct DisplayView: View {
    let isOn: Bool // ✅ Not @Binding
    var body: some View {
        Text(isOn ? "On" : "Off")
    }
}
```

### Creating Bindings Manually

```swift
// Constant binding for previews or fixed values
Toggle("Preview", isOn: .constant(true))

// Custom binding
var customBinding: Binding<Bool> {
    Binding(
        get: { UserDefaults.standard.bool(forKey: "key") },
        set: { UserDefaults.standard.set($0, forKey: "key") }
    )
}
```

---

## @ObservedObject

`@ObservedObject` subscribes a view to an external **reference type** that conforms to `ObservableObject`.

```swift
class UserViewModel: ObservableObject {
    @Published var name = ""
    @Published var age = 0
    
    func loadUser() {
        // network call
        name = "Alice"
        age = 30
    }
}

struct UserView: View {
    @ObservedObject var viewModel: UserViewModel

    var body: some View {
        VStack {
            Text(viewModel.name)
            Text("\(viewModel.age)")
        }
        .onAppear { viewModel.loadUser() }
    }
}
```

### How It Works

`ObservableObject` uses `objectWillChange` publisher. `@Published` properties automatically call `objectWillChange.send()` before changing. `@ObservedObject` subscribes to this publisher and triggers a redraw when it fires.

> ⚠️ **Critical Pitfall:** `@ObservedObject` does **NOT** own the object. If the parent view is recreated, the object could be recreated too, resetting all state. This is the most common SwiftUI bug.

```swift
// ❌ DANGEROUS — creates new ViewModel on every parent render
struct ParentView: View {
    var body: some View {
        ChildView(viewModel: UserViewModel()) // Recreated each time!
    }
}

struct ChildView: View {
    @ObservedObject var viewModel: UserViewModel // Doesn't own it
    ...
}
```

---

## @StateObject ← The Most Important Distinction

`@StateObject` is like `@ObservedObject` but it **owns** the object. SwiftUI creates it once and keeps it alive for the view's lifetime.

```swift
struct UserView: View {
    @StateObject private var viewModel = UserViewModel() // Owned here
    
    var body: some View {
        Text(viewModel.name)
    }
}
```

> 🎯 **Interview Answer — The Classic Question:**
>
> **"What's the difference between @StateObject and @ObservedObject?"**
>
> `@StateObject` **owns** the object — SwiftUI creates it once and it lives as long as the view is in the hierarchy. `@ObservedObject` **does not own** it — the object is provided externally. If you use `@ObservedObject var vm = MyVM()` inside a view, it will be recreated every time the parent re-renders, losing all state. Use `@StateObject` when the view should own the object's lifetime, and `@ObservedObject` when you're injecting an already-created object from outside.

```swift
// ✅ CORRECT PATTERNS

// View owns the VM — use @StateObject
struct RootView: View {
    @StateObject private var vm = UserViewModel()
    var body: some View {
        ChildView(vm: vm)
    }
}

// View receives external VM — use @ObservedObject
struct ChildView: View {
    @ObservedObject var vm: UserViewModel
    var body: some View { ... }
}
```

### When to Use Each

| Wrapper | Ownership | Use When |
|---------|-----------|----------|
| `@StateObject` | View owns | First point of creation |
| `@ObservedObject` | External | Injected from parent or environment |

---

## @EnvironmentObject

`@EnvironmentObject` allows passing objects through the view hierarchy without explicit injection at every level.

```swift
class AppState: ObservableObject {
    @Published var isLoggedIn = false
    @Published var currentUser: User?
}

// At root
@main
struct MyApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
}

// Deep in the hierarchy — no need to pass through every view
struct ProfileView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        Text(appState.currentUser?.name ?? "Not logged in")
    }
}
```

> ⚠️ **Pitfall:** If a view uses `@EnvironmentObject` but none of its ancestors injected it with `.environmentObject()`, the app **crashes at runtime** — not compile time. Always inject in previews too.

```swift
struct ProfileView_Previews: PreviewProvider {
    static var previews: some View {
        ProfileView()
            .environmentObject(AppState()) // Required for preview
    }
}
```

> ⚠️ **Pitfall:** `@EnvironmentObject` triggers a redraw of every view that accesses it when **any** `@Published` property changes. For large apps, consider breaking state into smaller, focused objects.

---

## @Environment

`@Environment` reads system-provided (or custom) environment values — not objects, but individual values.

```swift
struct AdaptiveView: View {
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.dynamicTypeSize) var typeSize
    @Environment(\.dismiss) var dismiss
    @Environment(\.openURL) var openURL

    var body: some View {
        VStack {
            Text("Theme: \(colorScheme == .dark ? "Dark" : "Light")")
            Button("Dismiss") { dismiss() }
            Button("Open") { openURL(URL(string: "https://apple.com")!) }
        }
    }
}
```

### Custom Environment Values

```swift
// Define the key
struct ThemeKey: EnvironmentKey {
    static let defaultValue: Theme = .default
}

extension EnvironmentValues {
    var theme: Theme {
        get { self[ThemeKey.self] }
        set { self[ThemeKey.self] = newValue }
    }
}

// Inject
ContentView().environment(\.theme, .dark)

// Consume
@Environment(\.theme) var theme
```

---

## @AppStorage

`@AppStorage` is a property wrapper that reads/writes UserDefaults, causing view updates when the value changes.

```swift
struct SettingsView: View {
    @AppStorage("isDarkMode") private var isDarkMode = false
    @AppStorage("fontSize") private var fontSize = 16.0
    @AppStorage("username") private var username = ""

    var body: some View {
        Form {
            Toggle("Dark Mode", isOn: $isDarkMode)
            Slider(value: $fontSize, in: 12...24)
            TextField("Username", text: $username)
        }
    }
}
```

> ⚠️ **Pitfall:** `@AppStorage` only works with types that conform to `RawRepresentable` (String, Int, Double, Bool, Data, URL). For custom types, encode to Data first.

```swift
// Custom type
@AppStorage("theme") private var themeData: Data = Data()

var theme: AppTheme {
    get { (try? JSONDecoder().decode(AppTheme.self, from: themeData)) ?? .default }
    set { themeData = (try? JSONEncoder().encode(newValue)) ?? Data() }
}
```

---

## State Management Decision Tree

```
Do you need mutable state?
├── No → Use `let` (constant)
└── Yes
    ├── Is it local to this view?
    │   ├── Yes → @State (value type) or @StateObject (reference type)
    │   └── No
    │       ├── Passed from parent?
    │       │   ├── Two-way? → @Binding
    │       │   └── Read-only? → `let`
    │       ├── Shared across many views?
    │       │   └── @EnvironmentObject (or @Environment for single values)
    │       └── Persisted to disk?
    │           ├── UserDefaults? → @AppStorage
    │           └── Keychain/CoreData → Custom + @StateObject
    └── External reference type injected?
        └── @ObservedObject
```

---

## The @Observable Macro (iOS 17+)

Swift 5.9 / iOS 17 introduced the `@Observable` macro from the `Observation` framework, replacing `ObservableObject`:

```swift
import Observation

@Observable
class UserViewModel {
    var name = ""
    var age = 0
    // No need for @Published — all stored properties are observed
    
    func load() { name = "Alice"; age = 30 }
}

struct UserView: View {
    // No need for @StateObject or @ObservedObject
    @State private var viewModel = UserViewModel()
    // OR just var viewModel = UserViewModel() if passed in

    var body: some View {
        Text(viewModel.name) // Automatically tracked
    }
}
```

### @Observable vs ObservableObject

| Feature | ObservableObject + @Published | @Observable |
|---------|-------------------------------|-------------|
| Granularity | Whole-object invalidation | Per-property tracking |
| Boilerplate | `@Published` everywhere | Nothing extra |
| Performance | All views using the object re-render | Only views using changed properties |
| iOS version | iOS 13+ | iOS 17+ |
| @StateObject | Required | Use @State |

> 💡 **Tip:** `@Observable` provides **fine-grained dependency tracking** — only views that read a changed property are re-rendered. This is a significant performance improvement.

---

## Senior-Level Discussion

### State Ownership Hierarchy

A well-designed SwiftUI app has a clear state ownership hierarchy:
- `@StateObject` / `@State` at the appropriate level (as low as possible)
- `@EnvironmentObject` for app-wide state (auth, theme, settings)
- `@ObservedObject` for injected dependencies

### Testing State

```swift
// ViewModels should be testable independent of SwiftUI
class UserViewModelTests: XCTestCase {
    func testLoadUser() async {
        let mockService = MockUserService()
        let vm = UserViewModel(service: mockService)
        await vm.loadUser()
        XCTAssertEqual(vm.name, "Alice")
    }
}
```

### Avoiding @EnvironmentObject Overuse

Large apps often make everything `@EnvironmentObject`. This leads to:
- Unclear dependencies
- Performance issues (all consumers re-render)
- Testing difficulty

Better: Use focused, small view models with `@StateObject` / `@ObservedObject` + dependency injection.

---

## Interview Q&A

**Q: What happens if you use `@ObservedObject var vm = MyVM()` in a view?**  
A: The `MyVM` instance is recreated every time the parent view re-renders. Any state in `vm` is lost. You should use `@StateObject` when the view should own the object's lifetime.

**Q: How does @State persist across re-renders if the view struct is recreated?**  
A: SwiftUI allocates persistent storage outside the view struct, keyed by the view's identity in the hierarchy. The struct is recreated but the storage location remains stable as long as the view is in the hierarchy.

**Q: When would you choose @EnvironmentObject over @ObservedObject?**  
A: `@EnvironmentObject` when the object needs to be accessible deep in the hierarchy without explicitly passing it through every intermediate view (prop drilling avoidance). `@ObservedObject` for direct parent-child injection where the relationship is explicit.

**Q: What's the difference between @Environment and @EnvironmentObject?**  
A: `@Environment` reads system or custom **value** types from the environment (colorScheme, dismiss, etc.). `@EnvironmentObject` reads **reference types** that conform to `ObservableObject`. Environment values are value-typed and don't trigger observation.

**Q: How does the @Observable macro improve performance over ObservableObject?**  
A: `ObservableObject` triggers `objectWillChange` for any `@Published` property change, causing all subscribing views to re-render. `@Observable` uses fine-grained per-property tracking — only views that **actually read** a changed property are re-rendered.

**Q: When would you use a custom Binding?**  
A: When you need to derive state from a non-@State source — like UserDefaults, a Core Data attribute, or transforming a value before reading/writing (e.g., converting an Int to a String for a TextField).

---

## Quick Revision

- `@State` — local value-type state, view owns it, private
- `@Binding` — two-way reference to state owned elsewhere, no storage
- `@ObservedObject` — subscribes to ObservableObject, does NOT own
- `@StateObject` — subscribes to ObservableObject, DOES own (use for creation point)
- `@EnvironmentObject` — implicit injection through hierarchy, crashes if missing
- `@Environment` — reads environment values (system or custom)
- `@AppStorage` — UserDefaults-backed, triggers redraws
- `@Observable` (iOS 17) — replaces ObservableObject, fine-grained tracking, use with `@State`
