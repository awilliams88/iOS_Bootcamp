# System Design — Scalable Architecture Patterns

---

## Modular Architecture

For large teams, split the app into frameworks that can be developed and compiled independently.

```
App Target
├── Features/
│   ├── FeedFeature (framework)
│   ├── MessagingFeature (framework)
│   ├── ProfileFeature (framework)
│   └── SearchFeature (framework)
├── Core/
│   ├── NetworkKit (framework)    — URLSession, API client
│   ├── StorageKit (framework)    — CoreData, Keychain
│   ├── DesignSystem (framework)  — UI components, colors, fonts
│   └── AnalyticsKit (framework)  — event tracking
└── Shared/
    ├── Models (framework)        — shared domain models
    └── Utilities (framework)     — extensions, helpers
```

### Benefits of Modularization

- **Faster compilation**: only rebuild changed modules
- **Team isolation**: different teams own different modules
- **Reusability**: DesignSystem shared across app targets (main app, widget, extension)
- **Testability**: each module tested independently
- **Dynamic feature delivery**: download features on-demand

---

## Pagination Architecture

```swift
// Generic pagination state
enum PaginationState<T> {
    case idle
    case loading(page: Int)
    case loaded(items: [T], hasMore: Bool, nextCursor: String?)
    case error(Error, canRetry: Bool)
}

// Cursor-based pagination (preferred over offset)
struct PaginationRequest {
    let limit: Int
    let cursor: String?  // nil = first page
}

struct PaginatedResponse<T: Decodable>: Decodable {
    let items: [T]
    let nextCursor: String?  // nil = last page
    let totalCount: Int?
}

// ViewModel with pagination
@MainActor
final class FeedViewModel: ObservableObject {
    @Published private(set) var items: [FeedItem] = []
    @Published private(set) var isLoadingMore = false
    @Published private(set) var hasMore = true
    
    private var nextCursor: String?
    private var loadTask: Task<Void, Never>?
    
    func loadInitial() async {
        items = []
        nextCursor = nil
        hasMore = true
        await loadNextPage()
    }
    
    func loadNextPage() async {
        guard hasMore, !isLoadingMore else { return }
        isLoadingMore = true
        defer { isLoadingMore = false }
        
        do {
            let response: PaginatedResponse<FeedItem> = try await apiClient.fetch(
                .feed(cursor: nextCursor, limit: 20)
            )
            items.append(contentsOf: response.items)
            nextCursor = response.nextCursor
            hasMore = response.nextCursor != nil
        } catch {
            // Handle error
        }
    }
    
    // Called when user scrolls near bottom
    func onItemAppear(item: FeedItem) {
        if items.suffix(5).contains(where: { $0.id == item.id }) {
            Task { await loadNextPage() }
        }
    }
}
```

---

## Caching Architecture

```swift
// Multi-tier cache: Memory (NSCache) → Disk → Network
final class CacheManager {
    private let memoryCache = NSCache<NSString, AnyObject>()
    private let diskCache: DiskCache
    
    func object<T: Codable>(for key: String, type: T.Type) async -> T? {
        let cacheKey = NSString(string: key)
        
        // L1: Memory
        if let cached = memoryCache.object(forKey: cacheKey) as? T {
            return cached
        }
        
        // L2: Disk
        if let cached = await diskCache.object(for: key, type: type) {
            memoryCache.setObject(cached as AnyObject, forKey: cacheKey)
            return cached
        }
        
        return nil  // L3: caller fetches from network
    }
    
    func store<T: Codable>(_ object: T, for key: String, expiresIn: TimeInterval = 300) async {
        let cacheKey = NSString(string: key)
        memoryCache.setObject(object as AnyObject, forKey: cacheKey)
        await diskCache.store(object, for: key, expiresIn: expiresIn)
    }
    
    func invalidate(key: String) async {
        memoryCache.removeObject(forKey: NSString(string: key))
        await diskCache.remove(key: key)
    }
}

// Cache expiry envelope
struct CachedValue<T: Codable>: Codable {
    let value: T
    let expiresAt: Date
    var isExpired: Bool { Date() > expiresAt }
}
```

---

## Offline-First Architecture

```swift
// Strategy: return cached data immediately, sync in background

final class UserRepository {
    private let remoteDS: UserRemoteDataSource
    private let localDS: UserLocalDataSource
    
    // Return stale data immediately, refresh in background
    func getUsers() -> AsyncStream<[User]> {
        AsyncStream { continuation in
            Task {
                // 1. Immediately emit cached data
                if let cached = try? await localDS.getUsers() {
                    continuation.yield(cached)
                }
                
                // 2. Fetch fresh data
                do {
                    let fresh = try await remoteDS.fetchUsers()
                    try await localDS.saveUsers(fresh)
                    continuation.yield(fresh)
                } catch {
                    // Already emitted cached — acceptable degraded state
                    continuation.yield(with: .success([]))
                }
                
                continuation.finish()
            }
        }
    }
    
    // Optimistic update — update local immediately, sync to server
    func updateUser(_ user: User) async throws {
        // Immediately update local (optimistic)
        try await localDS.updateUser(user)
        
        do {
            // Sync to server
            try await remoteDS.updateUser(user)
        } catch {
            // Rollback on failure
            try? await localDS.revertUser(user.id)
            throw error
        }
    }
}

// Sync queue for offline operations
actor OfflineSyncQueue {
    private var pendingOperations: [SyncOperation] = []
    
    func enqueue(_ operation: SyncOperation) {
        pendingOperations.append(operation)
    }
    
    func processAll(using service: RemoteService) async {
        var remaining: [SyncOperation] = []
        for operation in pendingOperations {
            do {
                try await operation.execute(using: service)
            } catch {
                if operation.canRetry { remaining.append(operation) }
            }
        }
        pendingOperations = remaining
    }
}
```

---

## Interview Q&A

**Q: How do you approach system design for an iOS app?**  
A: Clarify requirements first (offline support? real-time? scale?), then define the data model and API contract, design the layered architecture (presentation → domain → data), define caching strategy (memory/disk/TTL), handle offline (optimistic updates, sync queue), and define error/retry behavior. Talk about testability throughout.

**Q: Cursor vs offset pagination — which do you prefer and why?**  
A: Cursor-based. Offset pagination has a fundamental flaw: if new items are inserted while the user is paginating, items shift, causing duplicates or gaps. Cursor-based uses a pointer to a specific item, so the feed is consistent. It's more complex server-side but produces a better user experience.

**Q: How would you implement optimistic updates?**  
A: Apply the change to local state immediately (so UI feels instant), then make the network call. If the network call fails, roll back the local state and show an error. This requires keeping the previous state around for rollback. Popular in messaging and social apps where latency is critical.

---

## Quick Revision

- Modularization: Features, Core, Shared — independent compilation units
- Pagination: cursor-based over offset (no duplicate/missing items on insert)
- Cache tiers: Memory (NSCache) → Disk → Network
- Cache envelope: store with TTL, check expiry on read
- Offline-first: emit cached immediately, refresh in background
- Optimistic update: apply locally → sync remotely → rollback on failure
- Sync queue: persist pending operations, process when online
