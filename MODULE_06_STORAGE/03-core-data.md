# Storage — Core Data

---

## Core Data Overview

Core Data is Apple's object graph and persistence framework. It maps Swift objects to SQLite (default), binary, or in-memory stores.

```
NSManagedObject  ←→  NSManagedObjectContext  ←→  NSPersistentContainer  ←→  SQLite
(your model)          (scratch pad / session)      (stack manager)
```

---

## Setting Up NSPersistentContainer

```swift
class PersistenceController {
    static let shared = PersistenceController()
    
    let container: NSPersistentContainer
    
    // Main context for UI
    var viewContext: NSManagedObjectContext { container.viewContext }
    
    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "MyApp")  // matches .xcdatamodeld name
        
        if inMemory {
            container.persistentStoreDescriptions.first?.url = URL(fileURLWithPath: "/dev/null")
        }
        
        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("Core Data failed to load: \(error)")
            }
        }
        
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
    }
    
    // Background context for write operations
    func newBackgroundContext() -> NSManagedObjectContext {
        container.newBackgroundContext()
    }
    
    func save() {
        let context = container.viewContext
        guard context.hasChanges else { return }
        do {
            try context.save()
        } catch {
            print("Core Data save error: \(error)")
        }
    }
}
```

---

## NSManagedObject

```swift
// In .xcdatamodeld, create Entity "User" with attributes: id (UUID), name (String), email (String)

// Generated or manual NSManagedObject subclass
@objc(User)
class User: NSManagedObject {
    @NSManaged var id: UUID
    @NSManaged var name: String
    @NSManaged var email: String
    @NSManaged var createdAt: Date
    @NSManaged var posts: NSSet?  // To-many relationship
}

extension User {
    @nonobjc class func fetchRequest() -> NSFetchRequest<User> {
        NSFetchRequest<User>(entityName: "User")
    }
    
    var postsArray: [Post] {
        (posts as? Set<Post>)?.sorted { $0.createdAt < $1.createdAt } ?? []
    }
}
```

---

## CRUD Operations

```swift
// CREATE
func createUser(name: String, email: String) -> User {
    let context = PersistenceController.shared.viewContext
    let user = User(context: context)
    user.id = UUID()
    user.name = name
    user.email = email
    user.createdAt = Date()
    try? context.save()
    return user
}

// READ with NSFetchRequest
func fetchUsers(matching name: String? = nil) throws -> [User] {
    let context = PersistenceController.shared.viewContext
    let request: NSFetchRequest<User> = User.fetchRequest()
    
    if let name = name {
        request.predicate = NSPredicate(format: "name CONTAINS[cd] %@", name)
    }
    request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
    request.fetchLimit = 50
    
    return try context.fetch(request)
}

// UPDATE
func updateUserName(user: User, newName: String) {
    user.name = newName
    try? PersistenceController.shared.viewContext.save()
}

// DELETE
func deleteUser(_ user: User) {
    let context = PersistenceController.shared.viewContext
    context.delete(user)
    try? context.save()
}
```

---

## Threading — The Critical Rule

> ⚠️ **Critical:** Core Data contexts and managed objects are **NOT thread-safe**. Always access them on the queue they were created on.

```swift
// viewContext is main-queue context
// Access on main thread only

// ❌ WRONG — accessing viewContext on background thread
DispatchQueue.global().async {
    let users = try? PersistenceController.shared.viewContext.fetch(request) // CRASH
}

// ✅ CORRECT — use perform/performAndWait
let context = PersistenceController.shared.viewContext
context.perform {
    let users = try? context.fetch(request)
    // Use users here (still on main queue internally)
}

// ✅ Background write — use background context
func importUsers(_ users: [UserDTO]) async {
    let context = PersistenceController.shared.container.newBackgroundContext()
    await context.perform {
        for dto in users {
            let user = User(context: context)
            user.id = dto.id
            user.name = dto.name
        }
        try? context.save()
    }
}
```

### NSFetchedResultsController (UIKit)

```swift
class UserListViewController: UITableViewController, NSFetchedResultsControllerDelegate {
    lazy var fetchedResultsController: NSFetchedResultsController<User> = {
        let request: NSFetchRequest<User> = User.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
        
        let frc = NSFetchedResultsController(
            fetchRequest: request,
            managedObjectContext: PersistenceController.shared.viewContext,
            sectionNameKeyPath: nil,
            cacheName: nil
        )
        frc.delegate = self
        return frc
    }()
    
    // Auto-update table when data changes
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        tableView.reloadData()
    }
}
```

### @FetchRequest in SwiftUI

```swift
struct UserListView: View {
    @FetchRequest(
        sortDescriptors: [SortDescriptor(\.name)],
        predicate: NSPredicate(format: "isActive == YES"),
        animation: .default
    )
    private var users: FetchedResults<User>
    
    var body: some View {
        List(users) { user in
            Text(user.name)
        }
    }
}
```

---

## Migration

### Lightweight Migration (most common)

Works for simple changes: adding optional attributes, adding relationships, renaming:

```swift
let description = NSPersistentStoreDescription()
description.shouldMigrateStoreAutomatically = true    // default: true
description.shouldInferMappingModelAutomatically = true  // default: true
container.persistentStoreDescriptions = [description]
```

### Heavyweight Migration

Required for complex changes (merging entities, complex data transforms):

```swift
// 1. Create a new model version in .xcdatamodeld
// 2. Create NSMappingModel describing the transformation
// 3. Use NSMigrationManager

let migrationManager = NSMigrationManager(sourceModel: sourceModel, destinationModel: destModel)
try migrationManager.migrateStore(
    from: sourceURL, sourceType: NSSQLiteStoreType,
    options: nil, with: mappingModel,
    toDestinationURL: destURL, destinationType: NSSQLiteStoreType,
    destinationOptions: nil
)
```

---

## Interview Q&A

**Q: Explain the Core Data stack.**  
A: `NSPersistentContainer` manages the whole stack. It contains the data model (`NSManagedObjectModel`), the persistent store coordinator (`NSPersistentStoreCoordinator`) which talks to SQLite, and the `viewContext` (`NSManagedObjectContext`) for UI work. You create additional background contexts for write operations.

**Q: Why can't you use Core Data objects across threads?**  
A: `NSManagedObjectContext` has thread affinity — it must only be accessed on the queue it was created on. Managed objects belong to a context. Crossing threads causes data corruption or crashes. Use `perform`/`performAndWait` to dispatch work to a context's queue, and use `objectID` to safely pass references between contexts.

**Q: What's the difference between lightweight and heavyweight migration?**  
A: Lightweight migration works for simple model changes (adding optional attributes, renaming with a mapping) and happens automatically. Heavyweight migration requires an explicit `NSMappingModel` for complex transformations like splitting entities, merging data, or complex data type changes.

**Q: How does `automaticallyMergesChangesFromParent` work?**  
A: When enabled on `viewContext`, it automatically merges saved changes from child contexts or other contexts. Essential when you do background writes — without it, the view context doesn't see the changes until you manually call `mergeChanges`.

---

## Quick Revision

- `NSPersistentContainer`: manages whole Core Data stack
- `viewContext`: main queue context for UI reads
- Background context: use for imports/writes (`container.newBackgroundContext()`)
- Thread rule: context access only on its queue (`perform`/`performAndWait`)
- `NSFetchRequest`: query with predicates, sort descriptors, fetch limits
- `NSFetchedResultsController`: live-updating UIKit data source
- `@FetchRequest`: SwiftUI equivalent, auto-updating
- Lightweight migration: automatic for simple changes; heavyweight for complex transforms
