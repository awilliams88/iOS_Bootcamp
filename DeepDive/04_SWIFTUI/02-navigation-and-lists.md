# SwiftUI — Navigation & Lists

---

## NavigationStack (iOS 16+)

`NavigationStack` replaces the deprecated `NavigationView`. It uses a type-safe, value-driven navigation model.

```swift
struct ContentView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            List(Item.samples) { item in
                NavigationLink(value: item) {
                    Text(item.name)
                }
            }
            .navigationTitle("Items")
            .navigationDestination(for: Item.self) { item in
                ItemDetailView(item: item)
            }
            .navigationDestination(for: String.self) { route in
                Text("Route: \(route)")
            }
        }
    }
}
```

### Programmatic Navigation

```swift
// Push programmatically
path.append(someItem)

// Pop to root
path.removeLast(path.count)

// Pop one level
path.removeLast()

// Navigate via string route
path.append("settings")
```

### NavigationPath for Mixed Types

`NavigationPath` is a type-erased stack that can hold any `Hashable` value:

```swift
@State private var path = NavigationPath()

// Push different types
path.append(UserProfile())
path.append("onboarding")
path.append(42)  // Int
```

> 💡 **Tip:** Encode `NavigationPath` with `Codable` to restore navigation state after app relaunch (state restoration).

> ⚠️ **Pitfall:** `NavigationView` is deprecated in iOS 16. Migration to `NavigationStack` is required for new features and correct behavior on iPad/Mac.

> 🎯 **Interview Answer:** "NavigationStack uses value-based navigation — you push Hashable values onto a path, and declare destinations with `.navigationDestination(for:)`. This enables programmatic navigation, deep linking, and state restoration without needing to track view references."

---

## List

`List` is SwiftUI's primary scrollable collection view.

```swift
struct UserListView: View {
    @State private var users = User.samples
    @State private var selection: Set<User.ID> = []

    var body: some View {
        List(selection: $selection) {
            Section("Active") {
                ForEach(users.filter(\.isActive)) { user in
                    UserRow(user: user)
                        .swipeActions(edge: .trailing) {
                            Button("Delete", role: .destructive) {
                                deleteUser(user)
                            }
                            Button("Archive") {
                                archiveUser(user)
                            }.tint(.orange)
                        }
                        .swipeActions(edge: .leading) {
                            Button("Pin") { pin(user) }
                                .tint(.yellow)
                        }
                }
                .onMove { source, destination in
                    users.move(fromOffsets: source, toOffset: destination)
                }
                .onDelete { offsets in
                    users.remove(atOffsets: offsets)
                }
            }
        }
        .listStyle(.insetGrouped)
        .toolbar { EditButton() }
    }
}
```

### List Styles

```swift
.listStyle(.plain)           // UITableView plain
.listStyle(.grouped)         // UITableView grouped
.listStyle(.insetGrouped)    // Modern iOS style
.listStyle(.sidebar)         // iPad sidebar style
```

### Dynamic Content with ForEach

```swift
ForEach(items) { item in ... }                        // Identifiable
ForEach(items, id: \.self) { item in ... }            // Hashable
ForEach(0..<10) { index in ... }                      // Range
ForEach(Array(items.enumerated()), id: \.offset) { index, item in ... }
```

---

## Searchable

```swift
struct SearchableList: View {
    @State private var searchText = ""
    let allItems = Item.samples

    var filtered: [Item] {
        searchText.isEmpty ? allItems : allItems.filter {
            $0.name.localizedCaseInsensitiveContains(searchText)
        }
    }

    var body: some View {
        NavigationStack {
            List(filtered) { item in
                Text(item.name)
            }
            .searchable(text: $searchText, placement: .navigationBarDrawer(displayMode: .always))
            .navigationTitle("Search")
        }
    }
}
```

---

## Refreshable (Pull to Refresh)

```swift
List(items) { item in ItemRow(item: item) }
    .refreshable {
        await viewModel.refresh() // async context provided
    }
```

---

## Sheets, Alerts, and Dialogs

```swift
struct PresentationView: View {
    @State private var showSheet = false
    @State private var showAlert = false
    @State private var showConfirmation = false
    @State private var selectedItem: Item?

    var body: some View {
        VStack {
            Button("Show Sheet") { showSheet = true }
            Button("Show Alert") { showAlert = true }
            Button("Confirm") { showConfirmation = true }
        }
        // Sheet
        .sheet(isPresented: $showSheet) {
            DetailSheet()
                .presentationDetents([.medium, .large])    // iOS 16
                .presentationDragIndicator(.visible)
        }
        // Item-driven sheet (preferred pattern)
        .sheet(item: $selectedItem) { item in
            ItemSheet(item: item)
        }
        // Alert
        .alert("Confirm Delete", isPresented: $showAlert) {
            Button("Delete", role: .destructive) { }
            Button("Cancel", role: .cancel) { }
        } message: {
            Text("This cannot be undone.")
        }
        // Confirmation Dialog (action sheet)
        .confirmationDialog("Options", isPresented: $showConfirmation, titleVisibility: .visible) {
            Button("Save") { }
            Button("Share") { }
            Button("Delete", role: .destructive) { }
            Button("Cancel", role: .cancel) { }
        }
        // Full Screen
        .fullScreenCover(isPresented: $showSheet) {
            FullScreenView()
        }
    }
}
```

> ⚠️ **Pitfall:** Use `.sheet(item:)` over `.sheet(isPresented:)` when presenting item-specific content. The item-driven variant automatically dismisses when the item becomes nil and re-presents with new items.

---

## Forms

```swift
struct SettingsForm: View {
    @AppStorage("notifications") private var notificationsEnabled = true
    @State private var selectedFont = "System"
    @State private var textSize = 16.0

    var body: some View {
        NavigationStack {
            Form {
                Section("Display") {
                    Picker("Font", selection: $selectedFont) {
                        Text("System").tag("System")
                        Text("Serif").tag("Serif")
                    }
                    Slider(value: $textSize, in: 12...24, step: 1) {
                        Text("Size: \(Int(textSize))")
                    }
                }
                Section("Notifications") {
                    Toggle("Enable", isOn: $notificationsEnabled)
                }
                Section {
                    Button("Reset", role: .destructive) { }
                }
            }
            .navigationTitle("Settings")
        }
    }
}
```

---

## Interview Q&A

**Q: What is the difference between NavigationView and NavigationStack?**  
A: `NavigationView` is deprecated in iOS 16. `NavigationStack` provides value-based, type-safe navigation using a path. It supports programmatic navigation, deep linking, and state restoration. `NavigationView` had confusing multi-column behavior on iPad; `NavigationStack` has predictable single-column behavior.

**Q: How do you deep link into a specific screen using NavigationStack?**  
A: Populate the `NavigationPath` with the values representing the desired route. For URL-based deep links, parse the URL in `.onOpenURL`, convert path components to your navigation values, and append them to the path.

**Q: When should you use `.sheet(item:)` vs `.sheet(isPresented:)`?**  
A: Use `.sheet(item:)` when the sheet displays content for a specific data item. It's cleaner — when item is nil the sheet dismisses, when set it presents, and SwiftUI handles transitions automatically. `.sheet(isPresented:)` is for sheets with no associated data.

**Q: How does `.refreshable` work under the hood?**  
A: It provides an `async` context via a continuation. SwiftUI shows the refresh indicator and awaits your async task. The indicator disappears when the task completes. This integrates with Swift's structured concurrency.
