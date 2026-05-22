# Advanced Swift Concurrency

## Race Conditions

Happen when multiple tasks access shared mutable state unsafely.

```swift
counter += 1
```

Unsafe across threads.

## Cancellation

Swift tasks support cooperative cancellation.

```swift
Task.isCancelled
```

## TaskGroup

Structured parallel execution.

```swift
await withTaskGroup(of: User.self) { group in
}
```

## async let

Parallel child tasks.

```swift
async let user = fetchUser()
async let posts = fetchPosts()
```

## Continuations

Bridge callback APIs to async/await.

## Detached Tasks

Independent tasks outside parent structure.

Use cautiously.

## Interview Answer

Prefer structured concurrency, actor isolation, and cancellation-aware async workflows over manual thread management.
