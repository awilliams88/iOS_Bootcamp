# SwiftUI — Performance & Observation

---

## How SwiftUI Diffing Works

SwiftUI re-renders views by calling `body` and comparing the result. It uses **structural identity** and **value equality** to decide what changed.

### Structural Identity

SwiftUI identifies views by their **type and position** in the view hierarchy:

```swift
// These are always the same "view" — structural identity
if condition {
    Text("A")  // Always a Text at position 0
} else {
    Text("B")  // Same structural position, different value
}

// These are DIFFERENT views — SwiftUI destroys and recreates
if condition {
    Text("A")
} else {
    Image(systemName: "star")  // Different type at same position
}
```

### Explicit Identity with `id`

```swift
// Force SwiftUI to treat this as a new view when id changes
Text("Hello")
    .id(refreshToken)  // When refreshToken changes, view is destroyed + recreated

// Useful in ScrollView to scroll to a position
ScrollViewReader { proxy in
    ScrollView {
        ForEach(messages) { msg in
            MessageRow(msg: msg).id(msg.id)
        }
    }
    .onChange(of: messages.count) { _ in
        proxy.scrollTo(messages.last?.id, anchor: .bottom)
    }
}
```

> ⚠️ **Pitfall:** Using `.id()` unnecessarily causes the view to be destroyed and recreated (losing any internal state). Only use it when you explicitly want view identity to change.

---

## Avoiding Unnecessary Redraws

### 1. Use Equatable Views

```swift
struct UserRow: View, Equatable {
    let user: User
    
    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.user.id == rhs.user.id && lhs.user.name == rhs.user.name
    }
    
    var body: some View { ... }
}

// Tell SwiftUI to skip re-render if view is equal
EquatableView(content: UserRow(user: user))

// Or via modifier
UserRow(user: user).equatable()
```

### 2. Keep Body Computation Cheap

```swift
// ❌ Expensive work in body
var body: some View {
    let processed = heavyDataProcessing(items) // Called every render
    List(processed) { item in ItemRow(item: item) }
}

// ✅ Compute in ViewModel, only recalculate when needed
@StateObject private var vm = ItemViewModel()
var body: some View {
    List(vm.processedItems) { item in ItemRow(item: item) } // Pre-computed
}
```

### 3. Isolate State

```swift
// ❌ One big view with all state — any state change re-renders everything
struct BigView: View {
    @State private var searchText = ""
    @State private var isLoading = false
    // ... many more state vars
    
    var body: some View {
        VStack {
            SearchBar(text: $searchText) // Changes here re-render all below
            LoadingIndicator(isLoading: isLoading)
            ContentList(...)
        }
    }
}

// ✅ Extract subviews to isolate re-render scope
struct SmallSearchBar: View {
    @State private var text = ""  // Local — only this view re-renders on change
    var body: some View { TextField("Search", text: $text) }
}
```

### 4. Lazy Stacks for Large Lists

```swift
// ❌ VStack renders ALL views immediately
VStack {
    ForEach(0..<10_000) { i in ExpensiveRow(index: i) }
}

// ✅ LazyVStack renders only visible views
ScrollView {
    LazyVStack(pinnedViews: .sectionHeaders) {
        ForEach(0..<10_000) { i in ExpensiveRow(index: i) }
    }
}

// Same for horizontal
ScrollView(.horizontal) {
    LazyHStack { ... }
}
```

> 💡 **Tip:** `List` is lazy by default. `ScrollView + LazyVStack` gives you more layout control while still being performant.

---

## The @Observable Macro (iOS 17+)

`@Observable` from the `Observation` framework replaces `ObservableObject` with fine-grained, per-property tracking:

```swift
import Observation

@Observable
class UserStore {
    var users: [User] = []
    var isLoading = false
    var searchQuery = ""
    
    // Computed properties are also tracked
    var filteredUsers: [User] {
        users.filter { $0.name.contains(searchQuery) }
    }
    
    // Not observed (opt out)
    @ObservationIgnored
    var cache: [String: User] = [:]
    
    func loadUsers() async {
        isLoading = true
        users = await UserService.fetch()
        isLoading = false
    }
}
```

### Using @Observable in Views

```swift
// No @StateObject/@ObservedObject needed
struct UserListView: View {
    @State private var store = UserStore()  // Owned by this view
    
    var body: some View {
        List(store.filteredUsers) { user in
            UserRow(user: user)
        }
        .searchable(text: $store.searchQuery)
        .task { await store.loadUsers() }
    }
}

// Injected from outside — just use it directly (no wrapper needed)
struct ChildView: View {
    var store: UserStore  // Plain property, still observed
    
    var body: some View {
        Text(store.users.count.description)
    }
}
```

### Fine-Grained Tracking

With `ObservableObject`, any `@Published` change causes all subscribing views to re-render. With `@Observable`, only views that **read** a changed property re-render:

```swift
@Observable class Store {
    var name = ""
    var count = 0
}

// Only re-renders when 'name' changes, NOT when 'count' changes
struct NameView: View {
    var store: Store
    var body: some View { Text(store.name) }
}

// Only re-renders when 'count' changes
struct CountView: View {
    var store: Store
    var body: some View { Text("\(store.count)") }
}
```

> 🎯 **Interview Answer:** "@Observable provides automatic, fine-grained dependency tracking. SwiftUI tracks exactly which properties each view reads during `body`. When only `name` changes, only `NameView` re-renders — `CountView` is untouched. With `ObservableObject + @Published`, any property change fires `objectWillChange`, causing all observers to re-render regardless of which property changed."

### @Observable vs ObservableObject Comparison

| Feature | ObservableObject | @Observable |
|---------|-----------------|-------------|
| Min iOS | 13 | 17 |
| Tracking | Whole object | Per-property |
| Boilerplate | `@Published` everywhere | Nothing extra |
| In views | `@StateObject`/`@ObservedObject` | `@State` or plain property |
| Environment | `.environmentObject()` | `.environment()` |
| Protocol | `ObservableObject` | None (macro) |

```swift
// @Observable with environment
@Observable class ThemeStore { var isDark = false }

// Inject
ContentView().environment(ThemeStore())

// Consume (iOS 17)
@Environment(ThemeStore.self) private var theme
```

---

## Image Performance

```swift
// ❌ Loading large images without downsampling
AsyncImage(url: imageURL) { image in
    image.resizable().scaledToFill()
}

// ✅ Specify target size to avoid unnecessary memory
AsyncImage(url: imageURL) { phase in
    switch phase {
    case .success(let image):
        image
            .resizable()
            .scaledToFill()
            .frame(width: 100, height: 100)
            .clipped()
    case .failure:
        Image(systemName: "photo")
    case .empty:
        ProgressView()
    @unknown default:
        EmptyView()
    }
}
```

For production, use a caching library (Kingfisher, Nuke) or implement URLCache-backed image loading.

---

## Instruments for SwiftUI

Key Instruments templates:
- **SwiftUI** — shows view body call counts, invalidations
- **Time Profiler** — CPU usage during rendering
- **Allocations** — memory usage

```
Xcode → Product → Profile → SwiftUI template

Key metrics:
- "View Body" duration — should be fast (<1ms)
- "View Count" — how many views created  
- Excessive invalidations — state changing too often
```

> 💡 **Tip:** Use `Self._printChanges()` in `body` during debugging to print what caused a re-render:

```swift
var body: some View {
    let _ = Self._printChanges()  // Debug only — prints which properties changed
    Text("Hello")
}
```

---

## Interview Q&A

**Q: How does SwiftUI know which views to re-render?**  
A: SwiftUI tracks view identity (type + position in hierarchy) and calls `body` when subscribed state changes. It then diffs the new view description against the previous one and applies minimal changes to the render tree. With `@Observable`, it also tracks fine-grained property dependencies per view.

**Q: What is view identity and why does it matter?**  
A: SwiftUI assigns identity to views based on their type and structural position in the hierarchy (structural identity), or explicitly via the `.id()` modifier. Identity determines whether SwiftUI updates an existing view or destroys it and creates a new one. Changing identity resets all view state.

**Q: How do you prevent a child view from re-rendering when parent state changes?**  
A: Use `EquatableView` or `.equatable()` modifier with `Equatable` conformance. Move state down into child views where possible so parent state changes don't trigger child re-renders. With `@Observable`, views only re-render when the properties they specifically read change.

**Q: What's wrong with putting expensive computation directly in `body`?**  
A: `body` can be called many times as SwiftUI re-renders. Expensive work (filtering, sorting, parsing) should be moved into a ViewModel or computed once and cached, not recomputed on every render pass.

**Q: LazyVStack vs List — when do you use each?**  
A: `List` has built-in features (swipe actions, editing, selection, section headers) and is always lazy. `LazyVStack` inside `ScrollView` gives more layout flexibility (custom spacing, mixed content, no separators) while still being lazy. Use `List` for data lists with standard interactions; `LazyVStack` for custom scrollable layouts.

---

## Quick Revision

- SwiftUI uses structural identity (type + position) to track views
- `.id()` forces identity change → view destroyed and recreated
- `Equatable` views + `.equatable()` skip re-render if value unchanged
- Keep `body` computation fast — move expensive work to ViewModel
- `LazyVStack`/`LazyHStack` render only visible views
- `@Observable` (iOS 17): fine-grained per-property tracking, no `@Published`
- `Self._printChanges()` — debug tool to see what triggered re-render
- `AsyncImage` for remote images; use caching library for production
