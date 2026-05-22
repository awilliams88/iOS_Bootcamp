# GCD Fundamentals

## What is GCD?

Grand Central Dispatch is Apple's low-level concurrency framework for scheduling work.

## Dispatch Queues

### Serial Queue
Runs one task at a time.

### Concurrent Queue
Can run multiple tasks simultaneously.

## sync vs async

### sync
Caller waits.

### async
Caller continues immediately.

## Deadlock Example

```swift
DispatchQueue.main.sync {
    print("deadlock")
}
```

Calling sync on the current serial queue blocks forever.

## Interview Answer

GCD provides queue-based concurrency scheduling. async does not block the caller, while sync waits for execution completion.
