# Module 02 — Concurrency

Modern iOS interviews increasingly expect engineers to explain both legacy and modern concurrency tools.
This module helps you build that full stack mental model.
You should be comfortable moving from threads and queues to tasks and actors.
You should also be able to explain tradeoffs, migration paths, and debugging strategy.

## Why this module matters

Concurrency is where many senior-level interview loops separate surface knowledge from production judgment.
It is easy to memorize syntax.
It is harder to explain why a deadlock happened, why a task should be structured, or why actor isolation changes API design.
This module is built for that second skill.

> 💡 **Tip:** In interviews, define the primitive, give the mental model, explain the production consequence, then describe the tradeoff.

## How to use this module

Read the files in order the first time.
Do not skip the GCD material just because you use async/await today.
Most real iOS codebases still contain queues, operations, delegates, and callback APIs.
Strong candidates can bridge old and new models clearly.

Recommended approach:

- Read one file at a time.
- Rewrite the key ideas in your own words.
- Run the Swift examples locally if you want to test intuition.
- Practice the interview questions aloud.
- Revisit the senior-level discussion sections before mock interviews.

## Topic map

- [01-threads-gcd.md](./01-threads-gcd.md) — Threads, tasks, GCD fundamentals, queues, QoS, groups, semaphores, operations, and deadlocks.
- [02-async-await-tasks.md](./02-async-await-tasks.md) — Structured concurrency, async/await, tasks, task groups, async let, cancellation, continuations, and bridging callbacks.
- [03-actors-and-isolation.md](./03-actors-and-isolation.md) — Actors, isolation, MainActor, Sendable, global actors, reentrancy, nonisolated, and cross-actor communication.
- [04-concurrency-patterns-and-pitfalls.md](./04-concurrency-patterns-and-pitfalls.md) — Race conditions, data races, deadlocks, priority inversion, thread explosion, AsyncStream, migration scenarios, and senior tradeoffs.

## Learning outcomes

By the end of this module you should be able to:

- Explain the difference between a thread, a queue, and a task.
- Describe how Grand Central Dispatch schedules work.
- Explain serial versus concurrent queues without hand-waving.
- Walk through a deadlock step by step.
- Use DispatchGroup and OperationQueue appropriately.
- Explain what structured concurrency guarantees.
- Distinguish child tasks from detached tasks.
- Describe cancellation as cooperative rather than magical.
- Explain actor isolation and why it exists.
- Describe actor reentrancy and the bugs it can introduce.
- Use Sendable terminology correctly.
- Compare actors, queues, and locks at a senior level.
- Diagnose common concurrency bugs in interviews and in real apps.

## Suggested reading sequence

### Pass 1 — Core mental models

1. Read file 01 and focus on execution model vocabulary.
2. Read file 02 and connect that vocabulary to modern Swift concurrency.
3. Read file 03 and make sure isolation and Sendable concepts feel precise.
4. Read file 04 and rehearse debugging, migration, and tradeoff discussions.

### Pass 2 — Interview rehearsal

1. Re-read the callouts only.
2. Rehearse the Mermaid diagrams from memory on a whiteboard.
3. Explain one code sample from each file out loud.
4. Answer five Q&A prompts without looking.

### Pass 3 — Senior polish

1. Compare old and new APIs.
2. Practice migration stories from callback code to async code.
3. Practice explaining when *not* to introduce more concurrency.
4. Practice answering “what bug would you expect here?” follow-ups.

## Module mindset

A good concurrency answer is rarely about syntax first.
It is about execution boundaries, ownership of work, cancellation rules, and correctness under load.
Interviewers want to know whether you can reason about concurrent systems, not just whether you can remember `await`.

> 🎯 **Interview Answer:** “I think about concurrency in layers: execution resources like threads, scheduling constructs like queues and operations, language-level tasks, and isolation boundaries like actors.”

## Fast glossary

### Thread

A thread is an operating system execution resource.
It has its own stack and can run instructions independently.
Too many threads create overhead.

### Queue

A queue is a scheduling abstraction.
Work items are submitted to it.
The system decides when and where they run.
A queue is not the same thing as a thread.

### Task

A Swift task is a unit of asynchronous work managed by the concurrency runtime.
Tasks compose with cancellation, priority, and structured lifetimes.
Tasks are not one-to-one with threads.

### Actor

An actor is a reference type that protects mutable state with isolation.
Only one actor-isolated execution context can mutate that state at a time.
Actors help prevent many data races.

### Structured concurrency

Structured concurrency means child tasks are scoped to the parent.
Their lifetimes are explicit.
Cancellation and errors propagate in predictable ways.
This improves correctness and debuggability.

### Unstructured concurrency

Unstructured concurrency means work is launched without a strong parent-child lifetime relationship.
This can be useful.
It is also easier to leak work, lose cancellation, or hide ownership boundaries.

## What interviewers usually probe

- Do you confuse threads with queues?
- Do you know why `DispatchQueue.main.sync` can deadlock?
- Can you compare `Task {}` and `Task.detached {}`?
- Do you know what actor reentrancy is?
- Can you explain why Sendable exists?
- Can you bridge a completion handler API to async/await?
- Can you talk about thread sanitizer and concurrency debugging?
- Can you describe tradeoffs instead of saying “always use actors now”?

## Common misconceptions to remove early

### “Async means background thread.”

Not necessarily.
`async` means the function can suspend.
The resumed work might continue on a different thread.
The core idea is suspension, not thread hopping.

### “Concurrent means parallel.”

Not always.
Concurrency means multiple units of work can make progress over time.
Parallelism means work is actually executing simultaneously.
A single-core device can still be concurrent.

### “Actors eliminate all concurrency bugs.”

No.
Actors eliminate many data races around isolated mutable state.
They do not eliminate logic bugs, reentrancy bugs, deadlocks caused by waiting patterns, or misuse of nonisolated/shared state.

### “Cancellation kills the task immediately.”

No.
Swift task cancellation is cooperative.
The task must check for cancellation or hit an API that observes it.

### “DispatchSemaphore is just a lock.”

Not exactly.
A semaphore controls access based on a count.
It can be used like a gate, a backpressure tool, or a limited resource controller.
It is powerful and easy to misuse.

## Study checklist by file

### 01-threads-gcd.md

- I can explain threads, queues, and tasks separately.
- I can compare serial and concurrent queues clearly.
- I can explain sync versus async with a deadlock example.
- I can justify when to use DispatchGroup.
- I can explain why semaphores are risky if overused.
- I can compare OperationQueue to bare GCD.

### 02-async-await-tasks.md

- I can define suspension points accurately.
- I can describe task hierarchy in structured concurrency.
- I can compare `async let` and `TaskGroup`.
- I can explain cancellation behavior.
- I can bridge callbacks with checked continuations.
- I can say when `Task.detached` is appropriate.

### 03-actors-and-isolation.md

- I can explain actor isolation in one sentence.
- I can describe what `@MainActor` actually guarantees.
- I can explain actor reentrancy with a realistic bug.
- I can define Sendable and `@Sendable` precisely.
- I can explain `nonisolated` without making it sound unsafe by default.
- I can compare global actors and instance actors.

### 04-concurrency-patterns-and-pitfalls.md

- I can distinguish a data race from a race condition.
- I can describe priority inversion.
- I can explain thread explosion.
- I can describe migration from completion handlers to async APIs.
- I can explain AsyncStream.
- I can choose between actors and queues with tradeoff language.

## Whiteboard drills

Use these for mock interviews.

1. Draw a serial queue and a concurrent queue.
2. Draw a parent task with two child tasks.
3. Draw an actor boundary and one cross-actor call.
4. Draw a deadlock involving the main queue.
5. Draw a callback API wrapped by a continuation.
6. Draw an AsyncStream producer and consumer.
7. Draw priority inversion at a high level.
8. Draw a table comparing queue-based and actor-based isolation.

## Speaking drills

Practice the following in 60-second and 3-minute forms.

1. Explain the difference between a thread and a queue.
2. Explain why sync dispatch can deadlock.
3. Explain why async/await does not mean multithreading by itself.
4. Explain task cancellation.
5. Explain actor reentrancy.
6. Explain Sendable.
7. Explain how you would modernize a callback-heavy codebase.
8. Explain when you would still use OperationQueue.
9. Explain why structured concurrency is easier to reason about.
10. Explain how you would investigate a thread sanitizer warning.

## Production scenarios this module prepares you for

- Refreshing a feed while limiting duplicate work.
- Updating UI safely from async operations.
- Protecting shared caches or stores.
- Coordinating multiple network calls before rendering a screen.
- Cancelling in-flight work when a screen disappears.
- Migrating old service layers from callbacks to async APIs.
- Mixing Combine and async/await during gradual adoption.
- Avoiding subtle bugs caused by reentrancy or unstructured tasks.

## Senior-level discussion

Senior engineers usually stand out in concurrency interviews by doing three things.
First, they name the boundary correctly.
Second, they explain the failure mode before the fix.
Third, they discuss tradeoffs rather than presenting one tool as universally correct.

For example:

- A queue is good when you want explicit scheduling and compatibility with older APIs.
- An actor is good when state isolation is the primary concern.
- A detached task is acceptable when you truly want independent work, but it deserves scrutiny.
- A semaphore may solve a problem quickly, but often points to a better structured design.

> ⚠️ **Pitfall:** If your answer jumps straight to “I would just use async/await for everything,” expect follow-up questions about legacy APIs, third-party SDKs, and mixed codebases.

## Last-week revision plan

### Day 1

- Re-read threads, queues, and deadlock examples.
- Draw queue diagrams from memory.
- Explain QoS aloud.

### Day 2

- Re-read task hierarchy and cancellation.
- Implement one small `async let` example.
- Explain checked continuations aloud.

### Day 3

- Re-read actors, MainActor, and Sendable.
- Write one small actor-based store.
- Explain reentrancy aloud.

### Day 4

- Re-read race conditions, priority inversion, and AsyncStream.
- Practice migration answers.
- Review thread sanitizer workflow.

### Day 5

- Run rapid-fire Q&A from all files.
- Whiteboard two diagrams without notes.
- Answer “queues vs actors” and “structured vs unstructured” in senior language.

## Senior signal checklist

- I use precise vocabulary.
- I do not confuse suspension with blocking.
- I can explain a deadlock step by step.
- I can explain cancellation as cooperative.
- I can explain actor reentrancy without guessing.
- I can compare `TaskGroup` and `async let`.
- I can describe why Sendable matters for safe transfers.
- I can discuss migration, not just greenfield code.
- I can talk about debugging tools and symptoms.
- I can explain tradeoffs under legacy constraints.

## Rapid-fire review prompts

- What is the main queue actually used for?
- Why can a concurrent queue still preserve ordering for submission?
- Why is `DispatchQueue.global()` not a replacement for API design?
- What does `await` mean operationally?
- Why does `Task.detached` lose context?
- What bug does `withCheckedContinuation` help catch?
- Why is `MainActor` not literally the same as “main thread forever” in phrasing?
- What is reentrancy at an `await` boundary?
- Why can race conditions exist without low-level data races?
- What is the danger of blocking inside async code?

## Prep prompts for staff-level interviews

- How would you migrate a mixed UIKit app from callback APIs to structured concurrency incrementally?
- Where would you place actor boundaries in a feature-oriented architecture?
- How would you prevent duplicate work across screens while preserving cancellation?
- How would you handle expensive CPU work triggered from async UI flows?
- How would you make concurrency decisions visible in API signatures and code review guidelines?

## Common interview follow-up traps

- “Can you explain the bug without changing the code yet?”
- “What would this look like in a legacy callback codebase?”
- “What would you cancel when the user leaves the screen?”
- “How would you test this behavior?”
- “Where would you put the actor boundary?”
- “Would you use `Task.detached` here?”
- “What would happen under slow network conditions?”
- “What would happen if this API resumed twice?”
- “How would you debug a hang versus a race?”
- “What part of this is about correctness versus throughput?”

## Final rehearsal prompts

1. Explain a main queue deadlock in 30 seconds.
2. Explain `DispatchGroup` versus `TaskGroup` in 60 seconds.
3. Explain `async let` versus `TaskGroup` in 60 seconds.
4. Explain actor reentrancy with one realistic bug.
5. Explain `Sendable` without sounding compiler-driven only.
6. Explain migration from completion handlers with cancellation preserved.
7. Explain why blocking is often the wrong bridge to async work.
8. Explain when a queue is still more practical than an actor.
9. Explain why structured concurrency improves readability and cleanup.
10. Explain how you would investigate a thread sanitizer report.

## What strong concurrency answers sound like

- They distinguish blocking from suspension.
- They distinguish ownership from scheduling.
- They distinguish data races from logic races.
- They use cancellation as part of design, not as an afterthought.
- They acknowledge legacy code instead of pretending every codebase is greenfield.
- They explain why a specific abstraction fits the problem.
- They recognize when more concurrency would make the system worse.
- They connect language rules to production failure modes.
- They describe migration in slices rather than rewrites.
- They sound precise under follow-up pressure.

## Bonus review checklist

- I can draw the main queue deadlock from memory.
- I can explain why semaphores are sharp tools.
- I can compare checked and unsafe continuations.
- I can explain why reentrancy changes invariants.
- I can explain why `@MainActor` is an isolation guarantee.
- I can choose between queue, actor, and task group with clear reasoning.
- I can describe one real production incident this module helps prevent.
- I can answer follow-up questions without switching terminology mid-answer.
- I can explain one debugging tool I would use first.
- I can explain one migration step I would avoid rushing.
- I can explain one place where blocking is unacceptable.
- I can explain one case where a semaphore is reasonable.
- I can explain one case where an actor is a poor fit.
- I can explain one case where a queue is still the right tool.

## Final advice

Concurrency answers become strong when they sound operational.
Describe what runs where.
Describe when execution suspends versus blocks.
Describe who owns the work.
Describe how cancellation and failure flow.
Describe how state is protected.
Then describe why that choice fits the app.

Good interviewers can tell when the mental model is real.
This module is designed to help you make it real.

## Interview Q&A

### 1. What is the fastest way to improve on concurrency interview answers?

Use precise vocabulary, draw the execution boundary, explain failure modes, and connect every abstraction to one real production use case.

### 2. Why should I still study GCD if I use async/await?

Because many real codebases still use queues, operations, and callbacks, and strong candidates can explain how modern Swift concurrency relates to those foundations.

### 3. What topics usually separate senior candidates?

Deadlock reasoning, cancellation design, actor reentrancy, Sendable correctness, migration strategy, and clear tradeoff language between actors, queues, and unstructured work.

### 4. What should I practice before a mock interview?

Explain one deadlock, one task hierarchy, one actor boundary, one migration story, and one sanitizer debugging workflow without notes.
