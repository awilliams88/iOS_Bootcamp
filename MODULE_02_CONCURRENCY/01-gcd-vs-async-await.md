# GCD vs async/await

## Grand Central Dispatch

Legacy concurrency primitives:
- DispatchQueue.main
- global queues
- sync
- async
- DispatchGroup
- DispatchSemaphore

Pitfalls:
- callback pyramids
- deadlocks
- thread reasoning complexity

## async/await
Structured concurrency simplifies async flows.

```swift
let users = try await api.fetchUsers()
```

Benefits:
- readability
- cancellation
- structured task hierarchy
- safer async code
