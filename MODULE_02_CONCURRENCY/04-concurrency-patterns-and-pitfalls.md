# 04 — Concurrency Patterns, Pitfalls, and Interview Scenarios

Knowing the APIs is not enough.
Strong iOS candidates can diagnose what goes wrong under concurrency pressure and can explain how they would migrate older patterns safely.
This file focuses on the bugs, smells, and decision points that show up in senior interviews and production incidents.

## Learning goals

By the end of this file you should be able to:

- Distinguish race conditions from data races.
- Explain common deadlocks in both GCD and actor-based systems.
- Describe priority inversion and thread explosion.
- Bridge completion handlers to async APIs thoughtfully.
- Explain AsyncStream and common producer-consumer patterns.
- Describe how Combine and async/await can coexist.
- Answer practical interview scenarios with debugging steps.
- Compare actors versus queues and structured versus unstructured concurrency.

## Race conditions versus data races

These terms are related but not identical.
Interviewers often use them loosely.
Senior candidates do not.

### Race condition

A race condition happens when correctness depends on timing or ordering between events.
The program may have no low-level memory safety violation, but the result is still wrong because operations happened in an unexpected order.

### Data race

A data race is a specific low-level condition where two concurrent accesses touch the same memory location, at least one access is a write, and synchronization is missing.
Data races are undefined behavior territory in many systems and can lead to crashes or corrupted state.

> 🎯 **Interview Answer:** “A data race is a specific unsafe concurrent memory access bug. A race condition is broader: any correctness bug that depends on timing or ordering, even if memory access itself is synchronized.”

## Example — race condition without a classic data race

Imagine a search screen.
The user types “sw” and then quickly types “swift”.
Request A for “sw” starts first.
Request B for “swift” starts second.
Request B returns first and updates the UI correctly.
Then Request A returns late and overwrites the UI with stale results.

That is a race condition.
You might still be on the main actor the whole time.
There may be no low-level data race.
The ordering bug is the problem.

### Example implementation

```swift
import Foundation

@MainActor
final class SearchViewModel {
    private let service: SearchService
    private(set) var results: [SearchResult] = []
    private var latestQuery: String = ""

    init(service: SearchService) {
        self.service = service
    }

    func search(query: String) {
        latestQuery = query

        Task {
            let fetched = try? await service.search(query: query)
            guard query == latestQuery else { return }
            results = fetched ?? []
        }
    }
}
```

The fix is not about locks.
It is about ownership and result relevance.
Cancellation can also help here.

## Example — classic data race

```swift
import Foundation

final class UnsafeCounter {
    var value = 0
}

let counter = UnsafeCounter()

DispatchQueue.global().async {
    for _ in 0..<10_000 {
        counter.value += 1
    }
}

DispatchQueue.global().async {
    for _ in 0..<10_000 {
        counter.value += 1
    }
}
```

Two concurrent writers mutate the same memory without synchronization.
That is a data race.
The fix could be a serial queue, a lock, an actor, or a redesign around immutability.

## Deadlocks revisited

You already saw GCD deadlocks.
At a broader level, deadlock means progress stops because everyone waits on everyone else.
The modern version of this conversation includes actor-based waiting patterns too.

### GCD deadlock recap

- Sync dispatch to the same serial queue.
- Main thread waiting with a semaphore for work that needs the main queue.
- Cyclic waits across queues.

### Actor-flavored deadlock patterns

Actors themselves avoid many direct data races.
But you can still create deadlock-like or starvation-heavy systems if you mix blocking waits, semaphores, or badly designed cycles around async work.

#### Example — blocking around actor work

```swift
import Foundation

actor SettingsStore {
    private var flag = false

    func update(flag: Bool) {
        self.flag = flag
    }
}

let store = SettingsStore()
let semaphore = DispatchSemaphore(value: 0)

Task {
    await store.update(flag: true)
    semaphore.signal()
}

semaphore.wait()
```

If you do this on the main thread or inside a fragile execution context, you risk UI hangs and architectural confusion.
The deeper point is that blocking is usually the wrong way to consume actor-based async work.

#### Example — cyclic async design smell

Actor A awaits work from service B.
Service B tries to synchronously ask actor A for more state through a blocked bridge.
No one makes progress.
The exact syntax may vary, but the smell is mixing async isolation with blocking dependencies.

> ⚠️ **Pitfall:** In modern Swift systems, many “actor deadlocks” are really blocking-bridge deadlocks or cyclic design bugs around actor boundaries.

## Priority inversion

Priority inversion happens when high-priority work is effectively delayed by lower-priority work holding a needed resource.
For example:

- A low-priority task holds a lock.
- A high-priority task needs that lock.
- A medium-priority task keeps running and prevents the low-priority task from finishing and releasing it.

The high-priority task is now indirectly blocked by lower-priority activity.

### Why iOS engineers should care

It can show up as UI stutter, delayed interactions, or strange latency spikes.
It is especially relevant when heavy background tasks contend with user-facing work over shared resources.

### Mitigation ideas

- Reduce shared mutable state.
- Keep critical sections small.
- Avoid unnecessary locks around slow work.
- Use appropriate priorities and actor/queue boundaries.
- Avoid broad serialization bottlenecks.

## Thread explosion

Thread explosion happens when too much work causes too many threads to be created or kept active.
This wastes memory and scheduling time.
The app may become slower even though you tried to “make it concurrent.”

### Common causes

- Spawning many independent blocking operations.
- Using global concurrent queues for every tiny piece of work.
- Mixing async APIs with blocking wrappers.
- Creating too many `Operation`s or detached tasks without control.

### Symptoms

- High context-switch overhead.
- CPU wasted on scheduling instead of useful work.
- Memory pressure from many thread stacks.
- Unpredictable latency.

### Better patterns

- Bound concurrency.
- Preserve structure.
- Avoid blocking waits.
- Batch work when appropriate.
- Prefer cooperative async workflows over thread-per-task thinking.

> 💡 **Tip:** More concurrency is not the same as more throughput. Past a point, it becomes scheduler overhead and contention.

## Bridging completion handlers to async/await

Migration is not just about wrapping a callback in a continuation.
You must preserve:

- Error semantics.
- Cancellation semantics.
- Queue or actor expectations.
- Lifetime ownership.
- Backpressure or result ordering expectations.

### Mechanical wrapper

```swift
import Foundation

extension LegacyUserService {
    func fetchUser(id: String) async throws -> User {
        try await withCheckedThrowingContinuation { continuation in
            fetchUser(id: id) { result in
                continuation.resume(with: result)
            }
        }
    }
}
```

This is a good first step.
But it may be incomplete.
If the underlying request is cancellable, the wrapper should probably forward cancellation.
If the completion always arrives on a background queue but the caller assumed main-thread callbacks, your migration must make that explicit too.

### Better migration framing

When discussing migration in interviews, explain the layers:

1. Add async wrappers to preserve behavior.
2. Move call sites gradually.
3. Propagate cancellation intentionally.
4. Revisit API shape and ownership once the app compiles.
5. Remove old callback APIs only after usage stabilizes.

## Async sequences and `AsyncStream`

`AsyncSequence` is to asynchronous streams what `Sequence` is to synchronous collections.
It lets you iterate over values that arrive over time.
`AsyncStream` is a convenient way to bridge callback- or delegate-based event sources into async iteration.

### Why `AsyncStream` matters

Many app workflows are not single request-response calls.
They are event streams.
Examples include:

- Location updates.
- WebSocket messages.
- Notification callbacks.
- File or progress events.
- Delegate-driven subsystem events.

### Basic `AsyncStream` example

```swift
import Foundation

final class DownloadProgressEmitter {
    private var continuation: AsyncStream<Double>.Continuation?

    lazy var progressStream: AsyncStream<Double> = {
        AsyncStream { continuation in
            self.continuation = continuation
        }
    }()

    func emit(_ progress: Double) {
        continuation?.yield(progress)
    }

    func finish() {
        continuation?.finish()
    }
}
```

Consuming it:

```swift
import Foundation

func observeProgress(emitter: DownloadProgressEmitter) {
    Task {
        for await progress in emitter.progressStream {
            print("Progress: \(progress)")
        }
    }
}
```

### A delegate-to-AsyncStream bridge

```swift
import Foundation
import CoreLocation

final class LocationStreamAdapter: NSObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    private var continuation: AsyncStream<CLLocation>.Continuation?

    lazy var locations: AsyncStream<CLLocation> = {
        AsyncStream { continuation in
            self.continuation = continuation
            manager.delegate = self
            manager.startUpdatingLocation()
            continuation.onTermination = { [weak self] _ in
                self?.manager.stopUpdatingLocation()
            }
        }
    }()

    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        for location in locations {
            continuation?.yield(location)
        }
    }
}
```

This is a great interview example because it shows modern bridging skill.
It also demonstrates resource cleanup on termination.

## Combining Combine with async/await

Real codebases often use both.
You do not need a dramatic big-bang rewrite.
The pragmatic goal is interop.

### Use Combine when

- You already have stream-heavy pipelines.
- Operators like debounce, merge, combineLatest, or retry are already central.
- A subsystem is mature and stable in Combine.

### Use async/await when

- The workflow is request-response shaped.
- Structured cancellation and readability matter.
- You want clearer imperative-style async control flow.

### Interop pattern — awaiting publisher values

```swift
import Combine
import Foundation

extension Publisher where Failure == Error {
    func firstValue() async throws -> Output {
        try await withCheckedThrowingContinuation { continuation in
            var cancellable: AnyCancellable?
            cancellable = first()
                .sink(
                    receiveCompletion: { completion in
                        if case .failure(let error) = completion {
                            continuation.resume(throwing: error)
                        }
                        cancellable?.cancel()
                    },
                    receiveValue: { value in
                        continuation.resume(returning: value)
                        cancellable?.cancel()
                    }
                )
        }
    }
}
```

### Interop pattern — publishing async work into Combine

You might also wrap async work inside a `Future` or another publisher when Combine is the consumer-facing API.
That can be appropriate during incremental migration.

### Senior migration stance

Use the model that best fits the workflow.
Do not force everything into Combine or everything into async/await if the shape does not fit.
Interoperability is often the most practical path.

## Interview scenario — “You see a purple thread sanitizer warning. What do you do?”

A strong answer should be systematic.

### Step 1 — treat it as a real bug until proven otherwise

Do not dismiss it as noise.
Thread sanitizer warnings often point to genuine unsafe shared state access.

### Step 2 — identify the shared memory

Ask:

- What exact object or property is involved?
- Which code paths are reading and writing it?
- Are these accesses meant to be isolated somehow?

### Step 3 — map the concurrency model

Determine whether the code is using:

- Raw threads.
- GCD queues.
- Operations.
- Tasks.
- Actors.
- Locks.
- MainActor isolation.

Often the bug comes from mixing models inconsistently.

### Step 4 — reproduce with a smaller mental model

Simplify the race.
Identify two conflicting accesses.
Ask whether there is a missing synchronization boundary or whether the type itself should be redesigned.

### Step 5 — choose the right fix

Possible fixes:

- Move the state into an actor.
- Protect legacy state with a queue or lock.
- Remove mutable sharing and return immutable snapshots.
- Enforce `@MainActor` for UI-facing state.
- Make a value type Sendable rather than sharing a mutable class.

### Step 6 — re-run sanitizer and add regression tests where possible

For low-level data races, rerun Thread Sanitizer.
For logical races, add async tests or scenario tests that validate ordering and cancellation expectations.

> 🎯 **Interview Answer:** “I start by identifying the shared mutable state and the conflicting accesses. Then I verify the intended isolation boundary and either enforce it with an actor, queue, or main-actor annotation, or redesign the data flow to avoid shared mutation entirely.”

## Interview scenario — “How do you migrate completion handlers to async?”

A strong answer is incremental.
It is not “search and replace everything.”

### Recommended answer structure

1. Inventory the high-traffic APIs first.
2. Add async wrappers using checked continuations.
3. Preserve cancellation and queue semantics.
4. Migrate call sites from leaf layers upward.
5. Prefer structured concurrency in new callers.
6. Remove callback overloads only after adoption stabilizes.
7. Use tests and instrumentation to verify behavior parity.

### What senior interviewers like to hear

- “I preserve semantics, not just signatures.”
- “I keep unstructured tasks at the edges.”
- “I define ownership and cancellation per feature.”
- “I modernize in slices, not by destabilizing the whole app.”

## When to use actors versus queues

This is a classic senior discussion.
There is no one-line rule.

### Prefer actors when

- Shared mutable state is the main problem.
- You want language-level isolation.
- The subsystem is new or already async-aware.
- You want compiler help around Sendable and isolation.

### Prefer queues when

- You are working inside a legacy GCD-heavy subsystem.
- Explicit scheduling behavior matters.
- You need compatibility with older callback-based internals.
- You are incrementally wrapping rather than redesigning.

### Hybrid reality

Many codebases use actors at service boundaries and still use queues inside lower-level libraries or older integration layers.
That is fine if the seams are clear.

## Structured versus unstructured concurrency

### Prefer structured when

- Child work belongs to a parent operation.
- You want cancellation and failure to flow predictably.
- Readability and ownership matter.
- The async workflow is part of a request-response or fan-out/fan-in shape.

### Use unstructured when

- You are launching work from a synchronous API boundary.
- You intentionally need independent lifetime.
- You will store and manage the task handle explicitly.

### Warning signs of too much unstructured work

- `Task {}` sprinkled through many view methods.
- No stored task handles.
- No cancellation on deinit or screen disappearance.
- Hidden dependencies between unrelated tasks.
- UI state updated from whichever task finishes last.

## Common production smells

- Fire-and-forget tasks that outlive the screen.
- Blocking waits inside async code.
- Task.detached used as a shortcut around isolation.
- Mutable reference types passed across concurrency boundaries.
- One global actor used as a bottleneck for unrelated work.
- Async wrappers that forget to propagate cancellation.
- Async sequences that never finish or never clean up.

## A migration example — completion handlers to async with modern call site

```swift
import Foundation

final class LegacyCatalogService {
    func fetchCatalog(completion: @escaping (Result<[Product], Error>) -> Void) {
        completion(.success([]))
    }
}

extension LegacyCatalogService {
    func fetchCatalog() async throws -> [Product] {
        try await withCheckedThrowingContinuation { continuation in
            fetchCatalog { result in
                continuation.resume(with: result)
            }
        }
    }
}

@MainActor
final class CatalogViewModel {
    private let service: LegacyCatalogService
    private(set) var products: [Product] = []
    private var refreshTask: Task<Void, Never>?

    init(service: LegacyCatalogService) {
        self.service = service
    }

    func refresh() {
        refreshTask?.cancel()
        refreshTask = Task {
            do {
                let fetched = try await service.fetchCatalog()
                try Task.checkCancellation()
                products = fetched
            } catch is CancellationError {
                // Ignore cancellation.
            } catch {
                products = []
            }
        }
    }
}
```

This answer is strong because it shows a wrapper and an owned task at the call site.
It also shows cancellation.

## Senior-level discussion

The senior difference in concurrency is not knowing more APIs.
It is choosing the right boundary and failure mode.
If your first instinct is “make it concurrent,” you may create more problems.
If your first instinct is “what owns this work, what state is shared, what must stay ordered, and what can be cancelled,” you are thinking like a senior engineer.

A good senior answer often sounds like this:

- I avoid shared mutable state when possible.
- If state must be shared, I choose an isolation boundary deliberately.
- I preserve structured lifetimes whenever possible.
- I do not block async code unless I absolutely have to bridge into a synchronous world.
- I treat sanitizer warnings and timing bugs as design feedback.
- I modernize incrementally so the codebase stays stable while becoming clearer.

## Interview Q&A

### 1. What is the difference between a race condition and a data race?

A data race is a specific unsafe concurrent memory access bug involving overlapping unsynchronized access where at least one access is a write.
A race condition is broader and includes any correctness issue caused by timing or ordering.

### 2. Can you have a race condition without a data race?

Yes.
Stale search results overwriting fresh results on the main actor is a classic example.
It is an ordering bug, not necessarily a low-level shared-memory bug.

### 3. What is priority inversion?

It is when higher-priority work is effectively delayed because a lower-priority task holds a needed resource.
Other work can worsen the delay.

### 4. What is thread explosion?

It is excessive thread creation or activity caused by over-concurrency, especially blocking work.
It wastes memory and CPU on scheduling overhead.

### 5. How do you bridge a completion handler to async/await?

Usually with `withCheckedContinuation` or `withCheckedThrowingContinuation`, while preserving semantics like errors, cancellation, and callback expectations.

### 6. Why is a naive continuation wrapper sometimes incomplete?

Because it may ignore cancellation, queue expectations, multi-callback behavior, or resource cleanup.
A correct migration preserves behavior, not just API shape.

### 7. What is `AsyncStream` used for?

It bridges event-driven or callback-driven streams into async iteration, such as delegate events, progress updates, or messages over time.

### 8. When would you keep Combine instead of rewriting everything to async/await?

When the workload is truly stream-oriented and existing Combine pipelines are valuable or deeply integrated.
Async/await is often better for request-response flows.

### 9. What do you do when Thread Sanitizer reports a warning?

Identify the shared mutable state, map the conflicting accesses, verify the intended isolation model, fix the ownership or synchronization boundary, and rerun the sanitizer.

### 10. When would you use actors over queues?

When shared mutable state is the central problem and you want language-level isolation with compiler help.
Queues may still fit legacy scheduling-heavy or GCD-centric subsystems.

### 11. When would you choose structured concurrency over unstructured tasks?

When the child work clearly belongs to a parent operation and you want cancellation, errors, and lifetime to be explicit.

### 12. What is a common mistake with unstructured tasks in UI code?

Launching them everywhere without storing task handles or cancelling when the screen changes, which causes stale updates and wasted work.

### 13. Are actor deadlocks common in the same way as GCD deadlocks?

The patterns are different.
More often the issue is blocking around actor work or creating cyclic dependencies between async boundaries, rather than direct self-sync deadlocks.

### 14. What is the senior-level answer on concurrency modernization?

Modernize incrementally, preserve semantics, use structured ownership, isolate mutable state clearly, and avoid using concurrency features as mere syntax upgrades.
