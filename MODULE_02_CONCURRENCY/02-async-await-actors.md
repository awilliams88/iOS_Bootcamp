# async/await, Tasks, and Actors

## async/await

Modern structured concurrency.

```swift
func fetchUser() async throws -> User
```

Usage:

```swift
let user = try await fetchUser()
```

## Task

Creates asynchronous units of work.

```swift
Task {
    await load()
}
```

## Actors

Actors protect mutable shared state.

```swift
actor UserStore {
    var users: [User] = []
}
```

## MainActor

Used for UI isolation.

```swift
@MainActor
class ViewModel {}
```

## Sendable

Marks types safe to transfer across concurrency boundaries.

## Interview Answer

Modern Swift concurrency uses structured async tasks and actors to improve safety compared to manual thread coordination.
