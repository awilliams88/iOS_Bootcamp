# Swift Concurrency & Threading — Interview Revision Guide

### Contents

- [Why Concurrency Matters](#why-concurrency-matters)
- [Processes vs Threads](#processes-vs-threads)
- [Main Thread](#main-thread)
- [Race Conditions](#race-conditions)
- [Deadlocks](#deadlocks)
- [Thread Explosion](#thread-explosion)
- [Grand Central Dispatch (GCD)](#grand-central-dispatch-gcd)
- [Dispatch Queues](#dispatch-queues)
- [Serial vs Concurrent Queues](#serial-vs-concurrent-queues)
- [Sync vs Async](#sync-vs-async)
- [QoS (Quality of Service)](#qos-quality-of-service)
- [OperationQueue](#operationqueue)
- [Swift Concurrency](#swift-concurrency)
- [async / await](#async--await)
- [Structured Concurrency](#structured-concurrency)
- [Task](#task)
- [async let](#async-let)
- [Task Groups](#task-groups)
- [Detached Tasks](#detached-tasks)
- [Task Priorities](#task-priorities)
- [Task Cancellation](#task-cancellation)
- [Actors](#actors)
- [Actor Isolation](#actor-isolation)
- [MainActor](#mainactor)
- [Sendable](#sendable)
- [Data Races](#data-races)
- [Continuations](#continuations)
- [AsyncSequence](#asyncsequence)
- [AsyncStream](#asyncstream)
- [Thread Safety](#thread-safety)
- [Locks vs Actors](#locks-vs-actors)
- [Performance Considerations](#performance-considerations)
- [Production Concurrency Pitfalls](#production-concurrency-pitfalls)

---

# Why Concurrency Matters

Modern applications perform many tasks simultaneously:
- networking
- rendering
- image processing
- database operations
- animations
- file I/O
- async workflows

Without concurrency:
applications would:
- freeze
- block UI
- become unresponsive

---

# Why Blocking Is Dangerous

Imagine:

```swift id="t8j8sm"
let data = try Data(contentsOf: url)
```

on the main thread.

The UI freezes until the operation finishes.

This creates:
- lag
- dropped frames
- poor UX
- ANRs

---

# Production Perspective

Concurrency exists to:
- keep applications responsive
- maximize hardware utilization
- avoid blocking expensive work
- improve scalability

Modern iOS systems heavily rely on async execution.

---

# Common Production Mistake

Running:
- networking
- JSON parsing
- image decoding
- database work

directly on the main thread.

---

# Interview Questions

### Why is concurrency important in modern apps?

Concurrency keeps applications responsive by allowing expensive work to execute without blocking the UI.

---

### Why is blocking the main thread dangerous?

Because UI rendering and user interaction occur on the main thread.

---

# Processes vs Threads

---

# Process

A process is an isolated running application instance.

Each process has:
- its own memory
- resources
- execution environment

---

# Thread

Threads are execution units inside a process.

A single app process may contain many threads.

---

# Why Threads Matter

Threads allow work to happen concurrently.

Examples:
- rendering thread
- networking thread
- background processing

---

# Production Perspective

Threads are expensive system resources.

Too many threads create:
- scheduling overhead
- memory pressure
- context switching cost

Modern Swift Concurrency tries to reduce unnecessary thread explosion.

---

# Interview Questions

### Difference between processes and threads?

Processes are isolated application environments while threads are execution units inside processes.

---

### Why are threads expensive?

Because thread scheduling and context switching introduce system overhead.

---

# Main Thread

The main thread handles:
- UI rendering
- animations
- touch events
- user interaction

---

# Why Main Thread Safety Matters

UIKit and SwiftUI UI updates must occur on the main thread.

---

# Bad Example

```swift id="jlwmc2"
label.text = "Updated"
```

from a background thread.

This may create:
- crashes
- rendering bugs
- undefined behavior

---

# Correct Main Thread Usage

```swift id="2jlwm0"
DispatchQueue.main.async {
    label.text = "Updated"
}
```

---

# Swift Concurrency Equivalent

```swift id="jlwmt8"
await MainActor.run {
    label.text = "Updated"
}
```

---

# Production Perspective

Main-thread violations are extremely common interview discussion topics.

---

# Common Mistake

Doing heavy work on MainActor accidentally.

Example:
- image processing
- JSON decoding
- database parsing

inside MainActor contexts.

---

# Interview Questions

### Why must UI updates occur on the main thread?

Because UIKit and SwiftUI rendering systems are not thread-safe.

---

### Why is doing heavy work on the main thread dangerous?

Because it blocks rendering and creates lag.

---

# Race Conditions

Race conditions occur when:
multiple execution paths access shared mutable state simultaneously.

---

# Example

```swift id="jlwmm3"
var counter = 0

DispatchQueue.global().async {
    counter += 1
}

DispatchQueue.global().async {
    counter += 1
}
```

This may produce unpredictable results.

---

# Why Race Conditions Are Dangerous

Race conditions create:
- inconsistent state
- intermittent bugs
- difficult debugging
- nondeterministic behavior

---

# Production Perspective

Race conditions are among the MOST difficult production bugs to debug.

Because:
- they may occur rarely
- timing affects behavior
- logs often look normal

---

# Common Sources

- shared mutable state
- caches
- async callbacks
- global singletons
- mutable collections

---

# Better Approach

Prefer:
- immutable state
- actor isolation
- isolated ownership
- value semantics

---

# Interview Questions

### What is a race condition?

A race condition occurs when multiple execution paths access shared mutable state simultaneously without synchronization.

---

### Why are race conditions difficult to debug?

Because timing-dependent behavior makes them nondeterministic.

---

# Deadlocks

Deadlocks happen when execution waits forever.

Usually because multiple locks or queues wait on each other.

---

# Example

```swift id="jlwmh9"
DispatchQueue.main.sync {
    print("Deadlock")
}
```

from the main thread itself.

The queue waits for itself indefinitely.

---

# Why Deadlocks Are Dangerous

Deadlocks freeze execution completely.

Apps may:
- freeze
- hang
- stop responding

---

# Production Perspective

Deadlocks commonly appear with:
- nested sync calls
- lock ordering problems
- queue misuse
- blocking APIs

---

# Better Approach

Prefer:
- async workflows
- structured concurrency
- actor isolation

over:
- heavy locking systems

---

# Interview Questions

### What is a deadlock?

A deadlock occurs when execution waits indefinitely because resources block each other cyclically.

---

### Why is DispatchQueue.main.sync dangerous?

Because synchronously dispatching onto the same queue causes self-waiting deadlock.

---

# Thread Explosion

Too many threads create performance problems.

---

# Why Excessive Threads Are Dangerous

Each thread requires:
- memory
- scheduling
- context switching
- system coordination

---

# Symptoms

- CPU spikes
- lag
- poor battery life
- memory pressure
- reduced scalability

---

# Production Perspective

Older callback-heavy systems often created excessive thread usage.

Modern Swift Concurrency attempts to manage execution more efficiently using cooperative scheduling.

---

# Interview Questions

### Why is creating too many threads dangerous?

Because thread scheduling and context switching introduce significant overhead.

---

### How does Swift Concurrency help reduce thread explosion?

It uses cooperative task scheduling instead of uncontrolled thread creation.

---

# Grand Central Dispatch (GCD)

Before Swift Concurrency,
Apple heavily relied on:
- Grand Central Dispatch (GCD)
- Dispatch queues
- manual queue management

GCD is still extremely important because:
- legacy codebases use it heavily
- UIKit APIs still integrate with it
- interviewers frequently ask about it

---

# What Is GCD?

GCD is Apple’s low-level concurrency framework.

It manages:
- task scheduling
- thread pools
- queue execution
- async execution

---

# Why GCD Exists

Instead of developers manually creating threads,
GCD abstracts:
- thread creation
- scheduling
- resource management

This improves:
- scalability
- performance
- system efficiency

---

# Dispatch Queues

Work is submitted onto dispatch queues.

```swift id="jlwmx2"
DispatchQueue.global().async {
    print("Background work")
}
```

---

# Main Queue

```swift id="jlwmv1"
DispatchQueue.main.async {
    label.text = "Updated"
}
```

Used for:
- UI updates
- rendering
- animations

---

# Global Queues

Global queues are shared concurrent system queues.

```swift id="jlwmo8"
DispatchQueue.global(qos: .background)
```

---

# Serial Queues

Serial queues execute one task at a time.

```swift id="jlwmy3"
let queue = DispatchQueue(
    label: "com.app.serial"
)
```

---

# Why Serial Queues Matter

Serial queues help avoid race conditions because:
only one task executes at a time.

---

# Concurrent Queues

Concurrent queues may execute multiple tasks simultaneously.

```swift id="jlwmu9"
let queue = DispatchQueue(
    label: "com.app.concurrent",
    attributes: .concurrent
)
```

---

# Production Perspective

Serial queues simplify synchronization.

Concurrent queues improve throughput but increase complexity.

---

# Sync vs Async

This is VERY interview relevant.

---

# sync

Blocks current execution until completion.

```swift id="jlwmz7"
queue.sync {
    print("Done")
}
```

---

# async

Returns immediately.

```swift id="8jlwmm"
queue.async {
    print("Done")
}
```

---

# Why sync Can Be Dangerous

Synchronous execution may:
- block threads
- freeze UI
- create deadlocks

---

# Production Perspective

Modern Swift code strongly prefers:
- async workflows
- structured concurrency

over:
- excessive sync blocking

---

# QoS (Quality of Service)

QoS prioritizes system work.

Levels include:
- userInteractive
- userInitiated
- utility
- background

---

# Example

```swift id="jlwmh5"
DispatchQueue.global(
    qos: .userInitiated
)
```

---

# QoS Mental Model

## userInteractive
Critical UI work.

## userInitiated
User-requested operations.

## utility
Long-running work.

## background
Low-priority background work.

---

# Production Perspective

Incorrect QoS usage may:
- waste battery
- block important tasks
- reduce responsiveness

---

# Common Production Mistake

Running heavy background processing using:
- high-priority QoS

This starves important UI work.

---

# Interview Questions

### What problem does GCD solve?

GCD abstracts thread management and task scheduling efficiently.

---

### Difference between sync and async?

`sync` blocks execution until completion while `async` returns immediately.

---

### Why are serial queues useful?

They simplify synchronization and help avoid race conditions.

---

### Why can synchronous dispatch become dangerous?

Because blocking execution may freeze threads or create deadlocks.

---

### Why is QoS important?

QoS helps the system prioritize work appropriately for responsiveness and battery efficiency.

---

# OperationQueue

OperationQueue is a higher-level abstraction over GCD.

It provides:
- dependencies
- cancellation
- priorities
- operation management

---

# Basic Example

```swift id="jlwmr8"
let queue = OperationQueue()

queue.addOperation {
    print("Work")
}
```

---

# Operation Dependencies

Operations may depend on each other.

```swift id="jlwmt2"
operationB.addDependency(operationA)
```

---

# Why Dependencies Matter

Dependencies improve:
- workflow coordination
- pipeline management
- async orchestration

---

# Cancellation

Operations support cancellation.

```swift id="jlwml8"
operation.cancel()
```

---

# Production Perspective

OperationQueue was heavily used before Swift Concurrency.

Many legacy systems still rely on it.

---

# Why Swift Concurrency Reduced OperationQueue Usage

Swift Concurrency provides:
- cleaner async syntax
- structured concurrency
- task cancellation
- actor isolation

with less complexity.

---

# Interview Questions

### Why was OperationQueue useful?

It provided higher-level orchestration features like dependencies and cancellation.

---

### Why has Swift Concurrency reduced OperationQueue usage?

Because async/await and structured concurrency simplify async architecture significantly.

---

# Swift Concurrency

Swift Concurrency is Apple’s modern concurrency system.

Introduced to improve:
- readability
- safety
- structured async execution
- thread safety

---

# Problems With Older Concurrency Systems

Older callback-heavy systems often created:
- callback pyramids
- lifecycle complexity
- race conditions
- retain cycles
- difficult debugging

---

# Example Callback Hell

```swift id="jlwmh3"
fetchUser {
    fetchPosts {
        fetchComments {
        }
    }
}
```

---

# Why This Becomes Problematic

Nested callbacks create:
- unreadable code
- lifecycle complexity
- weak error handling
- difficult cancellation

---

# Swift Concurrency Goals

Swift Concurrency aims to improve:
- readability
- structured execution
- cancellation
- actor isolation
- concurrency safety

---

# async / await

Swift introduced async/await to simplify asynchronous code.

---

# async Function

```swift id="jlwmj1"
func fetchUsers() async
```

---

# await

```swift id="8jlwmw"
let users = await service.fetchUsers()
```

---

# Why async/await Matters

async/await makes async code:
- linear
- readable
- maintainable

instead of deeply nested callbacks.

---

# Error Handling With async

```swift id="jlwmv7"
let users = try await service.fetchUsers()
```

---

# Production Perspective

Modern iOS architecture heavily relies on:
- async/await
- structured concurrency
- actor isolation

---

# Important Mental Model

`await` does NOT necessarily block a thread.

This is VERY important.

Swift Concurrency suspends tasks efficiently.

---

# Why Suspension Is Better Than Blocking

Blocking:
- wastes threads

Suspension:
- frees execution resources

This improves scalability significantly.

---

# Interview Questions

### Why is async/await better than callback-heavy systems?

It improves readability, maintainability, and structured async flow.

---

### Does await block threads?

No. Await suspends tasks instead of blocking threads directly.

---

### Why is task suspension valuable?

Because execution resources can be reused efficiently while waiting.

---

# Structured Concurrency

Structured concurrency organizes async work into predictable lifecycles.

This is one of Swift Concurrency’s MOST important concepts.

---

# Why Structured Concurrency Matters

Older async systems often created:
- orphaned tasks
- lifecycle leaks
- cancellation complexity
- difficult ownership tracking

Structured concurrency improves:
- ownership
- cancellation propagation
- lifecycle predictability

---

# Parent-Child Task Relationships

Tasks may own child tasks.

When parents cancel:
children cancel automatically.

---

# Production Perspective

Structured concurrency significantly improves:
- cancellation handling
- task ownership
- async architecture safety

---

# Common Production Mistake

Creating detached async work without ownership tracking.

This often causes:
- leaked work
- retained tasks
- stale requests
- invalid UI updates

---

# Interview Questions

### What is structured concurrency?

Structured concurrency organizes async work into predictable ownership and lifecycle hierarchies.

---

### Why is structured concurrency valuable?

It improves cancellation propagation, lifecycle safety, and ownership management.

---

# Task

Tasks represent units of asynchronous work.

---

# Basic Task

```swift id="jlwmc6"
Task {
    await loadData()
}
```

---

# Why Tasks Matter

Tasks allow async work to execute concurrently.

---

# Production Perspective

Tasks are heavily used in:
- SwiftUI
- networking
- rendering workflows
- async orchestration

---

# Task Lifecycle

Tasks may:
- suspend
- resume
- cancel
- throw errors

---

# Common Production Mistake

Launching uncontrolled tasks everywhere.

```swift id="jlwmy5"
Task {
}
```

without ownership reasoning.

This creates:
- lifecycle complexity
- retained work
- cancellation problems

---

# Better Approach

Think carefully about:
- who owns the task
- when it should cancel
- whether the task outlives the screen

---

# Interview Questions

### What is a Task in Swift Concurrency?

A Task represents a unit of asynchronous work managed by the concurrency runtime.

---

### Why can uncontrolled tasks become dangerous?

Because ownership and cancellation become difficult to manage.

---

# async let

`async let` allows lightweight concurrent child tasks.

---

# Example

```swift id="jlwmu1"
async let users = fetchUsers()
async let posts = fetchPosts()

let result = await (users, posts)
```

Both operations execute concurrently.

---

# Why async let Matters

Without concurrency:

```swift id="jlwmg1"
let users = await fetchUsers()
let posts = await fetchPosts()
```

This executes sequentially.

---

# Benefits of async let

`async let` improves:
- performance
- throughput
- responsiveness

when independent work can run simultaneously.

---

# Important Mental Model

`async let` creates structured child tasks automatically.

These child tasks:
- inherit cancellation
- inherit priority
- belong to parent scope

---

# Why Structured Ownership Matters

When the parent task exits:
child tasks are automatically cleaned up.

This dramatically reduces:
- orphaned work
- lifecycle leaks
- stale async operations

---

# Production Perspective

`async let` is excellent for:
- parallel networking
- image loading
- independent API requests
- concurrent transformations

---

# Common Production Mistake

Using `async let` for dependent work.

Bad example:

```swift id="jlwmf0"
async let profile = fetchProfile()
async let posts = fetchPosts(profile.id)
```

`posts` depends on `profile`,
so concurrency here is invalid.

---

# Better Approach

Use `async let` only for:
- independent work units

---

# Interview Questions

### What problem does async let solve?

It enables lightweight structured concurrent execution for independent async work.

---

### Why is async let safer than detached concurrency?

Because child tasks remain tied to parent task ownership and cancellation.

---

### When should async let be avoided?

When tasks depend on each other sequentially.

---

# Task Groups

Task groups allow dynamic concurrent task creation.

---

# Why Task Groups Matter

`async let` works well for:
- fixed task counts

Task groups work better for:
- dynamic workloads

---

# Basic Example

```swift id="jlwm84"
await withTaskGroup(
    of: Int.self
) { group in

    for value in values {
        group.addTask {
            value * 2
        }
    }

    for await result in group {
        print(result)
    }
}
```

---

# Why Task Groups Are Powerful

Task groups enable:
- scalable concurrency
- dynamic fan-out
- concurrent aggregation
- structured ownership

---

# Production Use Cases

Task groups are excellent for:
- image processing
- batch networking
- concurrent parsing
- parallel transformations

---

# Cancellation Behavior

If parent tasks cancel:
child tasks cancel automatically.

This is one of structured concurrency’s biggest advantages.

---

# Error Handling

Task groups also support throwing variants.

```swift id="jlwmp9"
withThrowingTaskGroup
```

---

# Production Perspective

Task groups help build scalable async systems while preserving lifecycle safety.

---

# Common Production Mistake

Creating too many child tasks unnecessarily.

Tiny tasks may:
- increase scheduling overhead
- reduce performance
- create unnecessary complexity

---

# Better Approach

Use concurrency thoughtfully.

Concurrency is not automatically faster.

---

# Interview Questions

### Why are task groups useful?

They allow dynamic structured concurrent workloads.

---

### Difference between async let and task groups?

`async let` works well for fixed concurrent tasks while task groups support dynamic workloads.

---

### Why are task groups safer than manual thread management?

Because structured concurrency automatically manages ownership and cancellation.

---

# Detached Tasks

Detached tasks are independent tasks outside structured concurrency.

---

# Example

```swift id="jlwmg9"
Task.detached {
    await performWork()
}
```

---

# Why Detached Tasks Are Dangerous

Detached tasks:
- do NOT inherit cancellation
- do NOT inherit actor context
- do NOT inherit structured ownership

This creates:
- lifecycle complexity
- stale work
- retained tasks
- cancellation bugs

---

# Production Perspective

Detached tasks should be relatively rare.

Most async workflows should remain structured.

---

# When Detached Tasks Make Sense

Rare cases:
- background analytics
- independent cleanup
- fire-and-forget work
- isolated background systems

---

# Common Production Mistake

Using detached tasks for UI-related workflows.

This may:
- outlive screens
- update stale state
- leak work

---

# Better Approach

Prefer:
- structured tasks
- task groups
- async let

unless independence is truly required.

---

# Interview Questions

### Why are detached tasks dangerous?

Because they escape structured ownership and cancellation propagation.

---

### When should detached tasks be used?

Only for truly independent async work.

---

### Why are structured tasks generally preferred?

Because lifecycle management and cancellation become safer and more predictable.

---

# Task Priorities

Swift tasks support priorities.

Examples:
- high
- medium
- low
- background

---

# Example

```swift id="jlwmj0"
Task(priority: .background) {
    await cleanup()
}
```

---

# Why Priorities Matter

Priorities help the runtime schedule work appropriately.

Critical UI tasks should not compete equally with:
- analytics
- cleanup
- prefetching

---

# Important Mental Model

Priorities are hints,
not strict guarantees.

---

# Production Perspective

Incorrect priority usage may:
- reduce responsiveness
- waste battery
- starve important tasks

---

# Common Mistake

Running heavy background processing at high priority.

---

# Better Approach

Use priorities intentionally based on:
- UX importance
- responsiveness requirements
- system impact

---

# Interview Questions

### Why do task priorities matter?

They help the runtime prioritize important work appropriately.

---

### Are task priorities strict guarantees?

No. They are scheduling hints.

---

# Task Cancellation

Swift Concurrency supports cooperative cancellation.

This is EXTREMELY important in production systems.

---

# Why Cancellation Matters

Users constantly:
- navigate away
- cancel actions
- refresh screens
- close workflows

Without cancellation:
- stale work continues
- resources waste
- invalid UI updates occur

---

# Example

```swift id="jlwmf6"
let task = Task {
    await fetchData()
}

task.cancel()
```

---

# Important Mental Model

Cancellation is cooperative.

Tasks must check cancellation explicitly.

---

# Cancellation Checks

```swift id="jlwme7"
try Task.checkCancellation()
```

---

# isCancelled

```swift id="0jlwmp"
if Task.isCancelled {
    return
}
```

---

# Why Cooperative Cancellation Exists

Forcefully killing tasks is unsafe.

Swift instead allows:
- graceful cleanup
- safe termination
- predictable ownership

---

# Production Perspective

Cancellation handling is critical for:
- networking
- search systems
- image loading
- SwiftUI screens
- scrolling feeds

---

# Common Production Mistake

Ignoring cancellation entirely.

This often creates:
- wasted networking
- stale rendering
- invalid state updates
- unnecessary battery usage

---

# Better Approach

Check cancellation during:
- loops
- retries
- long-running work
- async pipelines

---

# Interview Questions

### Why is cancellation important?

Cancellation prevents stale work and improves responsiveness and resource efficiency.

---

### Why is Swift cancellation cooperative?

Because forcefully terminating tasks unsafely could corrupt state and resources.

---

### Why should long-running tasks check cancellation?

To stop unnecessary work promptly and safely.

---

# Actors

Actors are Swift’s concurrency-safe reference types.

They protect mutable state automatically.

Actors are one of the MOST important modern Swift interview topics.

---

# Why Actors Exist

Shared mutable state creates:
- race conditions
- synchronization complexity
- thread-safety bugs

Actors solve this by isolating mutable state.

---

# Basic Actor Example

```swift id="jlwmv2"
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }
}
```

---

# Why Actors Are Powerful

Only one execution path may access actor-isolated mutable state at a time.

This dramatically simplifies thread safety.

---

# Actor Isolation

Actor state is isolated automatically.

External access requires:

```swift id="jlwm83"
await counter.increment()
```

---

# Why await Is Required

Crossing actor boundaries may suspend execution safely.

---

# Production Perspective

Actors dramatically reduce:
- manual locking
- synchronization complexity
- thread-safety bugs

---

# Common Production Mistake

Putting huge amounts of unrelated state inside one actor.

This creates:
- bottlenecks
- unnecessary serialization
- reduced concurrency

---

# Better Approach

Use focused actors with clear ownership boundaries.

---

# Actor Reentrancy

Actors are reentrant.

This means:
during suspension,
other work may enter the actor.

This is VERY interview relevant.

---

# Why Reentrancy Matters

Developers may incorrectly assume actor methods are fully isolated throughout the entire async function.

Suspension points change this.

---

# Interview Questions

### What problem do actors solve?

Actors protect shared mutable state safely and reduce race conditions.

---

### Why is await required for actor access?

Because crossing actor boundaries may suspend execution safely.

---

### Why can large actors become bottlenecks?

Because actor isolation serializes access to actor state.

---

### What is actor reentrancy?

Actors may process other work during suspension points.

# Actor Isolation

Actor isolation is one of the MOST important concepts in Swift Concurrency.

Actors protect mutable state by isolating access.

---

# Why Isolation Matters

Without isolation:

```swift id="jlwmx5"
var users: [User] = []
```

Multiple threads may mutate this simultaneously.

This creates:
- race conditions
- corrupted state
- nondeterministic bugs

---

# Actor-Isolated State

```swift id="jlwmo1"
actor UserStore {
    private var users: [User] = []
}
```

Now mutation becomes isolated automatically.

---

# Cross-Actor Access

External access requires `await`.

```swift id="jlwmh7"
await store.add(user)
```

---

# Why await Is Necessary

Actor access may:
- suspend execution
- serialize safely
- coordinate ownership

---

# Internal Access

Inside the actor,
access is synchronous.

```swift id="jlwmp4"
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }
}
```

No `await` needed internally.

---

# Why This Is Powerful

Actors eliminate large categories of:
- manual locking
- synchronization bugs
- race conditions

---

# Production Perspective

Actor isolation dramatically simplifies:
- state containers
- caches
- synchronization systems
- async architecture

---

# Common Production Mistake

Assuming actors magically make ALL code safe.

Actors only protect:
- actor-isolated state

They do NOT automatically protect:
- external mutable references
- unsafe shared objects
- nonisolated state

---

# nonisolated

Actors may expose nonisolated members.

```swift id="jlwmj2"
nonisolated let id: UUID
```

---

# Why nonisolated Exists

Some values are:
- immutable
- thread-safe
- safe to access directly

without actor hopping.

---

# Production Perspective

Overusing actor isolation unnecessarily may:
- reduce performance
- create serialization bottlenecks
- increase actor hopping overhead

---

# Better Approach

Isolate:
- mutable shared state

Avoid isolating:
- immutable lightweight values

---

# Interview Questions

### What is actor isolation?

Actor isolation protects mutable state by ensuring controlled serialized access.

---

### Why is await required for actor access?

Because crossing actor boundaries may suspend execution safely.

---

### What problem does actor isolation solve?

It reduces race conditions and synchronization complexity.

---

### Why can excessive actor isolation hurt performance?

Because excessive serialization and actor hopping introduce overhead.

---

# MainActor

`MainActor` is a special global actor representing the main thread.

---

# Why MainActor Matters

UI frameworks require:
- main-thread execution
- rendering safety
- synchronized UI access

---

# MainActor Example

```swift id="jlwmd1"
@MainActor
final class HomeViewModel {
}
```

All isolated members now execute on the main actor.

---

# MainActor Functions

```swift id="jlwmg3"
@MainActor
func updateUI() {
}
```

---

# MainActor.run

```swift id="jlwmf4"
await MainActor.run {
    label.text = "Updated"
}
```

---

# Production Perspective

MainActor is heavily used in:
- SwiftUI
- UIKit state updates
- observable systems
- rendering workflows

---

# Common Production Mistake

Putting heavy work inside MainActor.

Bad example:
- JSON parsing
- image processing
- database decoding

inside MainActor contexts.

This blocks rendering.

---

# Better Approach

Do expensive work:
- off MainActor

Then return to MainActor only for:
- UI updates
- rendering state changes

---

# Example

```swift id="jlwmq5"
let users = try await service.fetch()

await MainActor.run {
    self.users = users
}
```

---

# Production Mental Model

MainActor should mostly coordinate:
- UI state
- rendering updates

NOT heavy processing.

---

# Interview Questions

### What is MainActor?

MainActor is a global actor representing the main thread.

---

### Why is MainActor important?

Because UI frameworks require thread-safe main-thread rendering.

---

### Why is heavy work on MainActor dangerous?

Because it blocks rendering and reduces responsiveness.

---

# Sendable

`Sendable` is one of the MOST important modern Swift concurrency topics.

---

# Why Sendable Exists

Concurrency becomes dangerous when mutable state crosses execution boundaries unsafely.

`Sendable` helps guarantee safe value transfer across concurrency domains.

---

# Example

```swift id="jlwmy8"
struct User: Sendable {
    let id: UUID
}
```

---

# Why Value Types Work Well

Immutable value types are naturally safer for concurrent transfer.

This strongly connects to:
- value semantics
- ownership isolation
- concurrency safety

---

# Why Classes Become Dangerous

Reference types may contain:
- shared mutable state
- unsafe mutation
- race-condition risk

---

# Sendable Restrictions

Swift may warn when unsafe types cross concurrency boundaries.

---

# Example

```swift id="jlwms5"
final class Cache {
    var values: [String] = []
}
```

This may NOT be safely Sendable.

---

# Why Sendable Matters

Sendable improves:
- thread safety
- compiler validation
- concurrency correctness

---

# @unchecked Sendable

Developers may manually bypass compiler checks.

```swift id="jlwmv4"
final class Cache: @unchecked Sendable {
}
```

---

# Why unchecked Is Dangerous

You are manually promising:
- thread safety
- synchronization correctness

Incorrect promises may create:
- race conditions
- memory corruption
- nondeterministic bugs

---

# Production Perspective

Modern Swift increasingly relies on:
- Sendable checking
- actor isolation
- value semantics

to improve concurrency safety.

---

# Common Production Mistake

Blindly adding:
- `@unchecked Sendable`

without proper synchronization reasoning.

---

# Better Approach

Prefer:
- immutable values
- actors
- isolated ownership
- thread-safe architecture

---

# Interview Questions

### What problem does Sendable solve?

Sendable helps guarantee safe transfer of values across concurrency boundaries.

---

### Why are immutable value types naturally safer for concurrency?

Because they avoid shared mutable state.

---

### Why is @unchecked Sendable dangerous?

Because developers manually bypass compiler safety guarantees.

---

### Why are classes harder to make Sendable safely?

Because shared mutable state creates race-condition risk.

---

# Data Races

Data races occur when:
multiple execution contexts access shared mutable state simultaneously,
with at least one mutation.

---

# Example

```swift id="jlwme2"
var count = 0

Task {
    count += 1
}

Task {
    count += 1
}
```

---

# Why Data Races Are Dangerous

Data races create:
- corrupted state
- intermittent bugs
- nondeterministic behavior
- impossible-to-reproduce crashes

---

# Why Swift Concurrency Helps

Swift Concurrency introduces:
- actor isolation
- Sendable validation
- structured ownership

to reduce data races.

---

# Production Perspective

Data races are among the MOST severe concurrency bugs.

Because:
- they may occur rarely
- timing affects behavior
- reproduction becomes difficult

---

# Common Production Sources

- mutable globals
- shared caches
- singleton mutation
- callback-heavy systems
- unsynchronized collections

---

# Better Approach

Prefer:
- immutable state
- actor ownership
- isolated mutation
- value semantics

---

# Interview Questions

### What is a data race?

A data race occurs when multiple execution contexts access shared mutable state simultaneously without proper synchronization.

---

### Why are data races dangerous?

Because they create nondeterministic and difficult-to-debug behavior.

---

### How does Swift Concurrency help reduce data races?

Through actor isolation, Sendable validation, and structured concurrency.

---

# Continuations

Continuations bridge old callback APIs into async/await systems.

---

# Why Continuations Matter

Many legacy APIs still use callbacks.

Continuations allow migration into modern async systems.

---

# Example

```swift id="jlwma9"
func fetch() async throws -> Data {
    try await withCheckedThrowingContinuation {
        continuation in

        api.fetch { result in
            continuation.resume(with: result)
        }
    }
}
```

---

# Why Checked Continuations Exist

Checked continuations validate:
- double resume bugs
- missing resume bugs

This improves safety significantly.

---

# Common Mistake

Resuming continuations multiple times.

This creates:
- crashes
- undefined async behavior

---

# Production Perspective

Continuations are extremely important for:
- legacy migration
- SDK integration
- callback interoperability

---

# Interview Questions

### What problem do continuations solve?

They bridge callback-based APIs into async/await systems.

---

### Why are checked continuations safer?

Because they validate incorrect continuation usage.

---

### Why can double-resume bugs become dangerous?

Because continuations expect exactly one completion signal.

---

# AsyncSequence

`AsyncSequence` represents asynchronously produced values over time.

---

# Why AsyncSequence Matters

Not all async systems return:
- one value

Some produce:
- streams
- events
- progressive updates

---

# Examples

- notifications
- sockets
- streaming APIs
- progress updates
- event systems

---

# Basic Example

```swift id="jlwmm8"
for await value in stream {
    print(value)
}
```

---

# Why AsyncSequence Is Powerful

It combines:
- asynchronous execution
with:
- sequence iteration

in a clean declarative model.

---

# Production Perspective

AsyncSequence is heavily used in:
- streaming systems
- event pipelines
- async state systems
- reactive workflows

---

# Interview Questions

### What problem does AsyncSequence solve?

It models asynchronously produced streams of values over time.

---

### Why is AsyncSequence useful for streaming systems?

Because values may arrive progressively instead of all at once.

---

# AsyncStream

`AsyncStream` creates custom AsyncSequence streams.

---

# Example

```swift id="jlwmf9"
let stream = AsyncStream<Int> {
    continuation in

    continuation.yield(1)
    continuation.finish()
}
```

---

# Why AsyncStream Matters

It helps bridge:
- delegates
- callbacks
- event systems

into async streams.

---

# Production Perspective

AsyncStream is useful for:
- WebSocket systems
- delegates
- notifications
- event pipelines

---

# Common Production Mistake

Never finishing streams properly.

This may:
- retain resources
- leak tasks
- keep consumers suspended forever

---

# Interview Questions

### Why is AsyncStream useful?

It helps create custom asynchronous event streams.

---

### Why must streams eventually finish?

To avoid leaked resources and suspended consumers.

# Thread Safety

Thread safety means:
multiple execution contexts can access code safely without:
- corruption
- race conditions
- inconsistent state

This is one of the MOST important production engineering concepts.

---

# Why Thread Safety Matters

Modern apps constantly execute:
- networking
- rendering
- caching
- parsing
- async workflows

simultaneously.

Without thread safety:
shared state becomes dangerous.

---

# Unsafe Example

```swift id="jlwmt5"
final class Counter {
    var value = 0
}
```

Multiple threads mutating `value` simultaneously may corrupt state.

---

# Why Shared Mutable State Is Dangerous

Shared mutable state creates:
- race conditions
- inconsistent reads
- synchronization complexity
- nondeterministic bugs

---

# Production Perspective

Most severe concurrency bugs come from:
- unsafe shared mutation
- hidden ownership
- unsynchronized access

---

# Safer Approaches

Prefer:
- immutable state
- actor isolation
- isolated ownership
- value semantics

over:
- shared mutable globals

---

# Immutable State

Immutable state is naturally thread-safe.

```swift id="jlwmy0"
struct User {
    let id: UUID
}
```

No mutation means:
no synchronization problem.

---

# Why Swift Strongly Encourages Value Semantics

Value semantics dramatically reduce:
- synchronization complexity
- shared mutation bugs
- ownership ambiguity

---

# Production Mental Model

The safest shared state is:
- state that cannot mutate

---

# Synchronization

Sometimes shared mutable state is unavoidable.

Synchronization mechanisms include:
- locks
- queues
- actors
- atomics

---

# Why Synchronization Exists

Synchronization coordinates access safely between execution contexts.

---

# Common Production Mistake

Synchronizing EVERYTHING unnecessarily.

Over-synchronization creates:
- bottlenecks
- serialization
- performance loss

---

# Better Approach

Minimize shared mutable state first.

Synchronization should be the LAST solution,
not the FIRST.

---

# Interview Questions

### What is thread safety?

Thread safety means code can execute concurrently without corrupting shared state.

---

### Why is immutable state naturally thread-safe?

Because immutable values cannot change concurrently.

---

### Why is shared mutable state dangerous?

Because simultaneous mutation creates race conditions and inconsistent behavior.

---

# Locks vs Actors

This is a VERY important modern interview topic.

---

# Locks

Traditional synchronization uses locks.

Examples:
- NSLock
- pthread_mutex
- os_unfair_lock

---

# Example

```swift id="jlwmw4"
let lock = NSLock()

lock.lock()
counter += 1
lock.unlock()
```

---

# Why Locks Are Dangerous

Locks may create:
- deadlocks
- priority inversion
- difficult debugging
- ownership complexity

---

# Production Problems With Locks

Large lock-heavy systems become:
- fragile
- difficult to reason about
- error-prone

---

# Actors

Actors provide:
- higher-level synchronization
- automatic isolation
- safer ownership

---

# Actor Example

```swift id="jlwmn1"
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }
}
```

---

# Why Actors Are Safer

Actors:
- isolate state automatically
- reduce manual synchronization
- avoid many race conditions

---

# Locks vs Actors Mental Model

## Locks
Manual synchronization.

## Actors
Ownership isolation model.

---

# Why Actors Improve Readability

Actor-based systems usually express:
- ownership
- mutation boundaries
- concurrency rules

more clearly.

---

# Performance Tradeoffs

Locks may sometimes be:
- lower overhead
- faster in tiny critical sections

Actors may introduce:
- suspension overhead
- actor hopping
- serialization

---

# Production Perspective

Modern Swift strongly encourages:
- actors
- structured concurrency

over:
- heavy manual locking systems

---

# When Locks Still Make Sense

Rare low-level systems:
- extremely performance-sensitive code
- interoperability layers
- tiny synchronization primitives

---

# Common Production Mistake

Using actors for huge unrelated systems.

This creates:
- bottlenecks
- serialized throughput
- poor concurrency scaling

---

# Better Approach

Use:
- focused actors
- isolated ownership domains
- small synchronization boundaries

---

# Interview Questions

### Why are actors generally safer than locks?

Actors automatically isolate mutable state and reduce synchronization mistakes.

---

### Why can locks become dangerous?

Because they may create deadlocks and ownership complexity.

---

### Why can actors still become bottlenecks?

Because actor isolation serializes access to actor state.

---

### When might locks still be appropriate?

In low-level highly performance-sensitive synchronization scenarios.

---

# Reentrancy & Suspension Points

This is one of the MOST misunderstood Swift Concurrency topics.

---

# Actor Reentrancy

Actors are reentrant.

This means:
during suspension,
other work may enter the actor.

---

# Example

```swift id="jlwma5"
actor BankAccount {
    var balance = 100

    func withdraw() async {
        let current = balance

        await networkCall()

        balance = current - 50
    }
}
```

---

# Why This Is Dangerous

During:
```swift id="jlwmg6"
await networkCall()
```

other tasks may mutate:
```swift id="jlwmh8"
balance
```

This creates logical race conditions.

---

# Important Mental Model

Actors prevent:
- simultaneous memory corruption

But NOT:
- logical state invalidation across suspension points

---

# Better Approach

Revalidate state after suspension.

---

# Production Perspective

Many actor bugs come from misunderstanding:
- reentrancy
- suspension boundaries
- state assumptions

---

# Interview Questions

### What is actor reentrancy?

Actors may process other work during suspension points.

---

### Why can suspension points become dangerous?

Because actor state may change while execution is suspended.

---

# Cooperative Thread Pool

Swift Concurrency uses cooperative scheduling.

This is VERY important.

---

# Older Threading Systems

Older systems often mapped:
- tasks
directly to:
- threads

This caused:
- thread explosion
- scheduling overhead
- memory pressure

---

# Swift Concurrency Approach

Swift tasks are lightweight.

The runtime cooperatively schedules tasks onto a smaller thread pool.

---

# Why This Matters

Suspended tasks do NOT necessarily consume active threads.

This dramatically improves scalability.

---

# Production Perspective

This is one reason Swift Concurrency scales much better than callback-heavy thread-spawning systems.

---

# Important Mental Model

Tasks are NOT threads.

This is EXTREMELY interview relevant.

---

# Interview Questions

### Are Swift tasks the same as threads?

No. Tasks are lightweight scheduled units managed cooperatively by the runtime.

---

### Why is cooperative scheduling valuable?

Because it reduces unnecessary thread creation and improves scalability.

---

# Task Local Values

Swift supports task-local context values.

---

# Example

```swift id="jlwmv3"
enum RequestID {
    @TaskLocal static var current: UUID?
}
```

---

# Why Task Locals Matter

Useful for:
- tracing
- logging
- analytics
- request context propagation

---

# Production Perspective

Task locals help propagate metadata across async systems cleanly.

---

# Interview Questions

### What are task-local values useful for?

They help propagate contextual async metadata like tracing or request IDs.

---

# Async Architecture Best Practices

Modern async systems should emphasize:
- ownership clarity
- cancellation
- isolation
- predictable lifecycles

---

# Good Async Architecture

Prefer:
- structured concurrency
- isolated ownership
- explicit cancellation
- actor boundaries

---

# Dangerous Async Architecture

Avoid:
- uncontrolled detached tasks
- hidden async work
- infinite retries
- retained tasks
- global mutable state

---

# Retry Mechanisms

VERY common interview topic.

---

# Why Retry Exists

Networking may fail because of:
- timeouts
- temporary outages
- rate limits
- connectivity issues

Retries improve resilience.

---

# Important Production Rule

NOT all failures should retry.

---

# Retryable Errors

Usually retry:
- 5xx server errors
- temporary connectivity failures
- timeouts

---

# Non-Retryable Errors

Usually DO NOT retry:
- 400 bad requests
- authentication failures
- validation failures

---

# Exponential Backoff

Retries should usually delay progressively.

---

# Example Mental Model

```text
1s
2s
4s
8s
```

---

# Why Exponential Backoff Matters

Without backoff:
clients may:
- spam servers
- worsen outages
- create cascading failures

---

# Jitter

Jitter adds randomness to retry delays.

---

# Why Jitter Matters

Without jitter:
many clients retry simultaneously.

This creates:
- retry storms
- traffic spikes
- server overload

---

# Production Perspective

Strong retry systems include:
- retry limits
- backoff
- jitter
- cancellation awareness
- idempotency reasoning

---

# Idempotency

This is VERY interview relevant.

---

# Why Idempotency Matters

Some operations are safe to retry repeatedly.

Examples:
- GET requests

Others may duplicate effects:
- payments
- purchases
- mutations

---

# Production Perspective

Retry systems must understand:
- whether operations are safe to repeat

---

# Interview Questions

### Why should retries use exponential backoff?

Because immediate retries may overload struggling systems.

---

### What is jitter?

Jitter adds randomness to retry delays to avoid synchronized retry storms.

---

### Why is idempotency important for retries?

Because some operations are unsafe to repeat multiple times.

---

### Which HTTP errors are commonly retryable?

Usually temporary failures like 5xx errors and connectivity timeouts.

---

# Performance Considerations

Concurrency does NOT automatically improve performance.

Incorrect concurrency may:
- reduce throughput
- increase synchronization overhead
- create contention
- increase memory pressure

---

# Common Performance Problems

- excessive task creation
- unnecessary actor hopping
- over-synchronization
- thread explosion
- blocking async execution
- huge actor bottlenecks

---

# Actor Hopping

Crossing actor boundaries repeatedly introduces overhead.

---

# Example

```swift id="jlwmt0"
await actorA.load()
await actorB.save()
await actorA.load()
```

Repeated actor hopping may reduce efficiency.

---

# Better Approach

Group related work together when possible.

---

# Production Perspective

Performance optimization is usually about:
- reducing unnecessary work
- reducing synchronization
- minimizing ownership complexity

not:
- blindly adding concurrency

---

# Common Production Mistake

Assuming:
“more async”
automatically means:
“faster”

This is false.

---

# Interview Questions

### Why does concurrency not automatically improve performance?

Because synchronization and scheduling overhead may outweigh benefits.

---

### What is actor hopping?

Repeatedly crossing actor boundaries during execution.

---

### Why can excessive actor hopping hurt performance?

Because actor transitions introduce suspension and scheduling overhead.

---

# Production Concurrency Pitfalls

These are VERY important for medium/senior interviews.

---

# Common Pitfalls

- retain cycles in tasks
- detached task misuse
- stale async updates
- cancellation ignored
- actor bottlenecks
- race conditions
- excessive synchronization
- blocking MainActor
- retry storms
- infinite async loops

---

# Example: Retained ViewModel

```swift id="jlwmz6"
Task {
    self.loadData()
}
```

The task strongly captures:
```swift id="jlwmk5"
self
```

---

# Safer Approach

```swift id="jlwmr2"
Task { [weak self] in
    await self?.loadData()
}
```

---

# Infinite Retry Pitfall

```swift id="jlwmo6"
while true {
    retry()
}
```

Without:
- cancellation
- backoff
- retry limits

this becomes dangerous.

---

# Stale UI Updates

Users may navigate away while async work continues.

Without cancellation:
old tasks may update invalid UI state.

---

# Production Perspective

Most real-world concurrency bugs come from:
- ownership
- lifecycle
- cancellation
- synchronization reasoning

NOT:
- syntax mistakes

---

# Final Concurrency Mental Model

Strong concurrency engineers think carefully about:
- ownership
- cancellation
- lifecycle
- isolation
- synchronization
- scalability
- performance tradeoffs

---

# Final Interview Advice

For medium/senior concurrency interviews:
focus on explaining:
- WHY
- TRADEOFFS
- PRODUCTION IMPACT

not just:
- syntax

---

# Example

Weak answer:

> “Actors prevent race conditions.”

Strong answer:

> “Actors isolate mutable state and reduce synchronization complexity by serializing access safely. However, they can still introduce bottlenecks and developers must still understand reentrancy and suspension behavior.”

That depth difference is what interviewers usually look for.

---

# Final Concurrency Summary

The MOST important recurring concurrency interview themes are:

- async/await
- structured concurrency
- cancellation
- actor isolation
- Sendable
- race conditions
- ownership
- MainActor
- task lifecycle
- retry mechanisms
- backoff and jitter
- thread safety
- actor reentrancy
- performance tradeoffs
- synchronization reasoning

Understanding:
- not only HOW concurrency syntax works
but:
- WHY concurrency systems exist
- HOW ownership behaves
- WHAT tradeoffs exist
- HOW production async systems fail