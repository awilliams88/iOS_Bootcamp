# Storage — SwiftData

SwiftData is Apple's modern persistence framework introduced in iOS 17, built on top of Core Data. It uses macros and Swift-native APIs.

---

## @Model Macro

```swift
import SwiftData

@Model
final class User {
    // All stored properties are automatically persisted
    var id: UUID
    var name: String
    var email: String
    var createdAt: Date
    
    // Relationships
    @Relationship(deleteRule: .cascade, inverse: \Post.author)
    var posts: [Post]
    
    // Transient (not persisted)
    @Transient
    var displayName: String { "\(name) <\(email)>" }
    
    // Custom attribute options
    @Attribute(.unique) var username: String
    @Attribute(.externalStorage) var avatarData: Data?
    
    init(name: String, email: String, username: String) {
        self.id = UUID()
        self.name = name
        self.email = email
        self.username = username
        self.createdAt = Date()
        self.posts = []
    }
}

@Model
final class Post {
    var title: String
    var content: String
    var publishedAt: Date?
    var author: User?  // Back-reference
    
    init(title: String, content: String) {
        self.title = title
        self.content = content
    }
}
```

---

## ModelContainer & ModelContext

```swift
// Set up container (usually in App entry point)
@main
struct MyApp: App {
    let container: ModelContainer
    
    init() {
        do {
            container = try ModelContainer(
                for: User.self, Post.self,
                configurations: ModelConfiguration(isStoredInMemoryOnly: false)
            )
        } catch {
            fatalError("Failed to create ModelContainer: \(error)")
        }
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

---

## CRUD with ModelContext

```swift
struct UserService {
    private let context: ModelContext
    
    init(context: ModelContext) {
        self.context = context
    }
    
    // CREATE
    func createUser(name: String, email: String, username: String) {
        let user = User(name: name, email: email, username: username)
        context.insert(user)
        try? context.save()
    }
    
    // READ — using #Predicate macro
    func fetchUsers(matching name: String) throws -> [User] {
        let predicate = #Predicate<User> { user in
            user.name.contains(name)
        }
        let descriptor = FetchDescriptor<User>(
            predicate: predicate,
            sortBy: [SortDescriptor(\.name)]
        )
        return try context.fetch(descriptor)
    }
    
    // UPDATE — just mutate the model
    func updateUserName(user: User, newName: String) {
        user.name = newName
        try? context.save()  // Or let autosave handle it
    }
    
    // DELETE
    func deleteUser(_ user: User) {
        context.delete(user)
        try? context.save()
    }
}
```

---

## @Query in SwiftUI

```swift
struct UserListView: View {
    // Automatically live-updating
    @Query(sort: \User.name, order: .forward)
    private var users: [User]
    
    // With predicate
    @Query(filter: #Predicate<User> { $0.posts.count > 0 })
    private var activeUsers: [User]
    
    @Environment(\.modelContext) private var context
    
    var body: some View {
        List(users) { user in
            VStack(alignment: .leading) {
                Text(user.name)
                Text(user.email).font(.caption).foregroundStyle(.secondary)
            }
        }
        .toolbar {
            Button("Add") {
                let user = User(name: "New User", email: "new@example.com", username: "newuser")
                context.insert(user)
            }
        }
    }
}
```

---

## SwiftData vs Core Data

| Feature | Core Data | SwiftData |
|---------|-----------|-----------|
| Min iOS | iOS 3 | iOS 17 |
| Model definition | .xcdatamodeld GUI | @Model macro in Swift |
| Thread safety | Manual (perform/performAndWait) | Actor-based (ModelActor) |
| SwiftUI integration | @FetchRequest | @Query |
| Relationships | NSSet / NSOrderedSet | Native Swift arrays |
| Migration | Lightweight / Heavyweight | Stage-based (SchemaMigrationPlan) |
| Testing | In-memory store | `isStoredInMemoryOnly: true` |

---

## Migration in SwiftData

```swift
// Version 1
typealias CurrentSchema = SchemaV2

enum SchemaV1: VersionedSchema {
    static var versionIdentifier = Schema.Version(1, 0, 0)
    static var models: [any PersistentModel.Type] { [User.self] }
    
    @Model final class User {
        var name: String
        init(name: String) { self.name = name }
    }
}

// Version 2 — added email
enum SchemaV2: VersionedSchema {
    static var versionIdentifier = Schema.Version(2, 0, 0)
    static var models: [any PersistentModel.Type] { [User.self] }
    
    @Model final class User {
        var name: String
        var email: String
        init(name: String, email: String) { self.name = name; self.email = email }
    }
}

// Migration plan
enum UserMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] { [SchemaV1.self, SchemaV2.self] }
    
    static var stages: [MigrationStage] {
        [MigrationStage.lightweight(fromVersion: SchemaV1.self, toVersion: SchemaV2.self)]
    }
}

// Container with migration
container = try ModelContainer(
    for: User.self,
    migrationPlan: UserMigrationPlan.self
)
```

---

## Interview Q&A

**Q: What are the key differences between SwiftData and Core Data?**  
A: SwiftData uses the `@Model` macro to define persisted types in pure Swift code — no `.xcdatamodeld` file needed. `@Query` replaces `@FetchRequest` in SwiftUI. Threading uses Swift's actor model rather than `perform`/`performAndWait`. SwiftData is built on Core Data under the hood and they can interoperate.

**Q: How does @Query differ from @FetchRequest?**  
A: Both provide live-updating results in SwiftUI, but `@Query` uses Swift-native predicates (`#Predicate` macro) with compile-time checking, while `@FetchRequest` uses `NSPredicate` strings checked at runtime. `@Query` also integrates with SwiftData's `ModelContext` directly.

**Q: Can SwiftData and Core Data interoperate?**  
A: Yes. You can use `NSPersistentContainer` from Core Data alongside SwiftData's `ModelContainer` pointed at the same store. Apple provides migration guidance for incrementally adopting SwiftData in existing Core Data apps.

---

## Quick Revision

- `@Model` macro: marks class as SwiftData model, all stored properties persisted
- `@Attribute(.unique)`: unique constraint; `.externalStorage`: large binary data
- `@Relationship(deleteRule:)`: cascade, nullify, deny
- `@Transient`: not persisted
- `ModelContainer`: stack manager (equivalent to `NSPersistentContainer`)
- `ModelContext`: operation context (equivalent to `NSManagedObjectContext`)
- `@Query`: live-updating SwiftUI results with `#Predicate`
- Migration: `VersionedSchema` + `SchemaMigrationPlan`
