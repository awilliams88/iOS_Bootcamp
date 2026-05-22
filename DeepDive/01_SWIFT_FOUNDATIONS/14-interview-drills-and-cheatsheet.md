# 14 — Interview Drills and Cheatsheet

Use this file in the final stretch before interviews.

## Rapid-fire Q&A
### 1. What is an optional?
A type-safe representation of presence or absence.

### 2. Weak vs unowned?
Weak becomes nil; unowned traps if accessed after deallocation.

### 3. `some` vs `any`?
`some` hides one concrete type; `any` stores an existential conformer.

### 4. Why prefer structs for many models?
They reduce aliasing and shared mutable state.

### 5. Why are Swift strings not integer indexed?
Because strings are Unicode-correct collections of extended grapheme clusters.

### 6. What is copy-on-write?
A storage optimization that preserves value semantics.

### 7. Why use `guard`?
To express preconditions and keep the happy path flat.

### 8. When use `Result`?
When failure must be stored or passed as data.

### 9. Why are enums powerful?
They model finite states with associated data and exhaustive switching.

### 10. What is the protocol extension dispatch pitfall?
Extension-only methods are not dynamically dispatched through protocol-typed values.

## Coding challenge — Reverse a linked list
```swift
final class ListNode {
    var value: Int
    var next: ListNode?

    init(value: Int, next: ListNode? = nil) {
        self.value = value
        self.next = next
    }
}

func reverse(_ head: ListNode?) -> ListNode? {
    var previous: ListNode?
    var current = head

    while let node = current {
        let next = node.next
        node.next = previous
        previous = node
        current = next
    }

    return previous
}
```

## Coding challenge — Implement a stack
```swift
struct Stack<Element> {
    private var storage: [Element] = []

    mutating func push(_ value: Element) { storage.append(value) }
    mutating func pop() -> Element? { storage.popLast() }
    func peek() -> Element? { storage.last }
}
```

## Coding challenge — LRU cache
```swift
final class LRUCache<Key: Hashable, Value> {
    private final class Node {
        let key: Key
        var value: Value
        var prev: Node?
        var next: Node?

        init(key: Key, value: Value) {
            self.key = key
            self.value = value
        }
    }

    private let capacity: Int
    private var storage: [Key: Node] = [:]
    private var head: Node?
    private var tail: Node?

    init(capacity: Int) { self.capacity = max(1, capacity) }
}
```

## Scenario prompts
- A view model stores a closure and starts leaking. Explain the ownership graph.
- A queue uses `removeFirst()` and gets slow. Explain the complexity issue.
- A protocol extension method is not dynamically dispatched. Explain why.
- A decoded payload is full of optionals. Explain boundary mapping with DTOs.
- A teammate wants to replace a class with a struct. Explain the identity tradeoff.

## Quick revision cheatsheet
- Default to `let`.
- Strings are Unicode-correct.
- Arrays, dictionaries, and sets are value types with copy-on-write.
- Optionals model absence explicitly.
- `guard` expresses preconditions.
- Structs and enums are value types; classes are reference types.
- Protocol requirements dispatch differently from extension-only methods.
- `throws` is control flow; `Result` is data.
- Weak becomes nil; unowned can crash.
- Modern Swift emphasizes macros, ownership, generics, and stricter concurrency.

## Expanded rapid-fire Q&A
### 11. Why is `guard` often preferred in validation-heavy code?
Because it states preconditions early, keeps the success path flat, and makes failure behavior explicit.

### 12. Why can `Array.removeFirst()` be a performance smell?
Because it is O(n) due to element shifting, which makes it poor for queue-like workloads.

### 13. Why does `default` sometimes weaken enum-based state modeling?
Because it hides newly added cases from the compiler and removes exhaustiveness as a refactor aid.

### 14. Why are DTOs useful with Codable?
Because they let you mirror the wire format and then map into stronger domain models with better invariants.

### 15. What is the fastest way to explain copy-on-write?
Value semantics stay intact while storage is shared until a mutation requires a unique copy.

### 16. How do you justify `try!` if you ever use it?
By naming the invariant that makes failure impossible or indicates a programmer error rather than a recoverable condition.

### 17. What does a good answer about `weak` include?
It should mention non-owning optional references, automatic niling, and why it fits breakable ownership relationships.

### 18. What does a good answer about `unowned` include?
It should mention non-owning non-nil references plus the lifetime guarantee required to avoid a crash.

### 19. Why are protocol abstractions sometimes overused?
Because abstraction adds indirection cost, discoverability cost, and maintenance cost when the system does not actually need variation.

### 20. What is a senior answer on structs vs classes?
Describe the choice in terms of identity, shared mutable state, lifetime, and architectural fit.

### 21. Why is `String.Index` a high-signal interview topic?
Because it shows whether you understand Swift’s Unicode correctness model rather than assuming strings are byte arrays.

### 22. Why is `Result` useful in callback APIs?
Because success or failure must be delivered as data after the original call stack has already returned.

### 23. How do you explain witness tables simply?
They are runtime structures Swift uses to look up protocol requirement implementations for a conforming type.

### 24. Why are enums strong for app state?
Because they encode a finite set of valid states and force exhaustive handling of each case.

### 25. How do you sound current when talking about modern Swift?
Mention macros, ownership, stricter concurrency, and more expressive generics, then explain why those changes exist.

### 26. Why is `some` not the same as `any`?
Because `some` preserves one hidden concrete type chosen by the implementation, while `any` stores an unknown conforming value behind an existential container.

### 27. Why can long optional chains be a smell?
Because they may hide meaningful missing-data cases that should be modeled or handled explicitly.

### 28. What is a strong explanation of property wrappers?
They package reusable storage or access policy behind an attribute, but should not hide core business logic.

### 29. Why do argument labels matter?
They make the call site intention-revealing and are part of Swift API design, not ornamental syntax.

### 30. How do you explain two-phase initialization?
Swift ensures stored properties are initialized before unsafe use or subclass customization, protecting invariants during construction.

### 31. Why can IUOs still be dangerous?
They are still optionals underneath, and a broken lifecycle invariant turns use into a runtime crash.

### 32. When is inheritance justified?
When identity and shared override-based behavior are central to the design rather than incidental.

### 33. What is the risk of extension-only protocol methods?
They are statically dispatched through protocol-typed values, which can surprise teams expecting polymorphism.

### 34. Why do interviewers ask about access control?
Because access modifiers reflect API boundary judgment and long-term maintainability choices.

### 35. How do you explain `defer` well?
It guarantees cleanup at scope exit, which is especially useful for locks, temporary state, and resource handling.

### 36. Why can generics outperform existential-heavy designs?
Because preserving concrete type relationships can enable better static checking and compiler optimization opportunities.

### 37. Why is type erasure useful?
Because some abstractions need a stable storable surface even when concrete underlying types differ or protocols have associated types.

### 38. What is a clean answer for nil coalescing?
Use it when all missing-value cases legitimately share the same fallback meaning.

### 39. Why is `mutating` useful on structs?
It makes state change explicit and reinforces value-type semantics.

### 40. How do you explain ARC without jargon overload?
Say ARC keeps class instances alive while strong references exist and leaks happen when strong ownership cycles never release.

### 41. What is a good fallback if you forget syntax in a live round?
Describe the invariant, the control flow, and the data structure clearly, then reconstruct the syntax step by step.

## Scenario follow-ups
### Scenario 1
- **Prompt:** You are asked to refactor a view model to stop leaking after adding stored callbacks.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 2
- **Prompt:** A reviewer says your protocol abstraction feels premature. Defend or change it.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 3
- **Prompt:** A coding round solution uses a class where a struct may be safer. Explain the tradeoff.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 4
- **Prompt:** A backend payload is inconsistent across endpoints. Explain your Codable strategy.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 5
- **Prompt:** A teammate defaults everything to `public`. Explain why that is risky.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 6
- **Prompt:** A performance issue appears after repeated queue operations. Identify the data structure mismatch.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 7
- **Prompt:** A crash happens after changing `weak` to `unowned`. Explain the likely lifetime bug.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 8
- **Prompt:** A SwiftUI-heavy team ignores UIKit fundamentals. Explain why that is still risky in interviews.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 9
- **Prompt:** A protocol extension helper is accidentally treated like an override point. Explain the dispatch bug.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 10
- **Prompt:** A manager asks whether modern Swift knowledge matters if the product codebase is older. Explain why it still matters in interviews.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 11
- **Prompt:** You are asked to explain typed throws even though your app has not adopted Swift 6 mode yet.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 12
- **Prompt:** A cache implementation uses only a dictionary and cannot maintain recency ordering. Explain the missing piece.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 13
- **Prompt:** An engineer uses `try?` everywhere in networking code. Explain the debugging downside.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 14
- **Prompt:** A string-processing solution uses integer offsets directly. Explain why that is unsafe in Swift.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

### Scenario 15
- **Prompt:** A candidate answer sounds textbook but not senior. Explain how you would level it up.
- **What a strong candidate says:** Define the relevant Swift rule, explain the runtime or architectural consequence, then propose a safer design or debugging approach.

## What interviewers listen for
- Clear definitions before details.
- A mental model rather than memorized trivia.
- Production consequences, not just syntax trivia.
- Correct tradeoffs between ergonomics and performance.
- Ownership reasoning when memory is discussed.
- Boundary reasoning when Codable or API design is discussed.
- Precise language around value vs reference semantics.
- Confidence around optionals without overusing force unwraps.
- Current awareness of modern Swift evolution.
- Ability to recover when exact syntax is forgotten.
- Recognition of invalid-state prevention as a design goal.
- Awareness that abstraction has runtime and maintenance cost.
- Explicit discussion of invariants and lifetime guarantees.
- Comfort moving from simple examples to real app scenarios.
- Clear communication under time pressure.
- Reasoning about algorithms and complexity when data structures are discussed.
- Awareness that some convenience features can hide important failure modes.
- Use of examples that sound like real product engineering, not coding kata only.
- Balanced, non-dogmatic recommendations.
- Ability to explain why Swift makes certain safety tradeoffs.

## Final pre-interview reset
- Rehearse 10 rapid-fire Swift questions without notes.
- Reverse a linked list once from memory.
- Explain the LRU cache design in plain English.
- Draw one retain cycle and show how to break it.
- Explain `some` vs `any` in under 45 seconds.
- Summarize modern Swift in one minute.
- Name one UIKit and one SwiftUI tradeoff confidently.
- Explain why DTO mapping matters for messy APIs.
- Practice one “why” answer for `guard`, `enum`, and value semantics.
- End with calm, concise answers rather than maximal detail.

## Extra rapid-fire bank
### 42. Why is `Set` often the right choice for membership checks?
Because average O(1) membership is the real workload need, and uniqueness is built into the abstraction.

### 43. Why can protocol-heavy designs become harder to navigate?
Because behavior is spread across contracts, conformers, and extensions, which raises discoverability cost.

### 44. What is a quick explanation of witness-table dispatch?
Swift uses protocol conformance lookup data to call the correct requirement implementation for a conforming type.

### 45. Why can `lazy` properties be tricky?
Because initialization timing shifts to first access and thread-safety is not automatic.

### 46. Why do classes participate in ARC while structs do not in the same way?
Classes are reference types with shared identity, while structs and enums are value types.

### 47. Why is `defer` valuable in interview code?
It shows disciplined cleanup thinking, especially around locks and temporary state.

### 48. What is a good explanation of `fileprivate` vs `private`?
Both narrow visibility heavily, but `fileprivate` opens access to the entire file while `private` is scoped to the enclosing declaration context.

### 49. Why are enums especially good for navigation or loading state?
Because they make legal states explicit and illegal combinations harder to represent.

### 50. Why does `map` on `Optional` exist?
Because an optional can be transformed like a container of zero or one value.

### 51. Why can `flatMap` avoid nested optionals?
Because it flattens one level when the transform already returns an optional.

### 52. Why are associated types hard to store directly?
Because the protocol leaves some related type choice to each conformer, so the concrete relationship is not fully known at simple existential use sites.

### 53. Why might a senior engineer reject a giant protocol?
Because capability boundaries become muddy and the abstraction becomes harder to satisfy, mock, and evolve.

### 54. What is a compact explanation of two-phase initialization?
Stored properties are initialized safely before later customization can use `self` more freely.

### 55. Why can `default` in a `switch` be harmful during refactors?
Because it hides newly added enum cases from the compiler.

### 56. Why is `public` safer than `open` by default for frameworks?
Because it exposes use without also promising subclassing and override points outside the module.

### 57. Why is DTO mapping a senior signal?
Because it shows you protect domain models from unstable transport details.

### 58. What is a short definition of value semantics?
Each value behaves as an independent value after assignment, even when storage optimizations exist.

### 59. Why can excessive `[weak self]` be noisy?
Because not every closure forms a cycle, and unnecessary weak handling can obscure logic.

### 60. Why are typed errors interesting even if not fully adopted everywhere yet?
They point toward more explicit failure contracts and stronger compile-time modeling.

### 61. What makes a Swift answer sound current?
Awareness of modern concurrency checking, macros, ownership, and generics evolution.

## Whiteboard explanation prompts
- Draw the ownership graph for a leaking view controller and view model.
- Explain how a dictionary plus doubly linked list powers an LRU cache.
- Show how a protocol extension bug appears when the compile-time type changes.
- Explain how you would redesign a payload with too many optionals.
- Walk through a switch-based state machine for loading, loaded, and error states.
- Explain why `String.Index` exists with an emoji example.
- Compare a concrete type dependency with an existential dependency.
- Describe how copy-on-write changes the practical behavior of arrays.
- Explain why `guard` often produces a cleaner function body.
- Describe when a class is the better answer over a struct.

## Final confidence checklist
- I can explain all 14 module topics without reading from the page.
- I can answer the same question in both short and long formats.
- I can name one pitfall per topic from memory.
- I can give at least one production example per topic.
- I can explain one tradeoff per topic instead of giving absolute rules.
- I can keep calm if an interviewer asks for more detail on memory or generics.
- I can write a linked-list reversal without freezing.
- I can explain LRU cache design even if I do not finish every line of code.
- I can explain what interviewers care about in Swift answers: correctness, judgment, and communication.
- I can close by summarizing the most important design tradeoff clearly.

## Extra scenario bank
### Extra scenario 1
- **Prompt:** A teammate force-unwraps network data because the backend contract is “stable.”
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 2
- **Prompt:** A class-based state model keeps causing accidental shared mutation.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 3
- **Prompt:** A protocol abstraction was added before a second implementation existed.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 4
- **Prompt:** A long optional chain hides why critical data is missing.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 5
- **Prompt:** An API client swallows all errors with `try?` and now debugging is painful.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 6
- **Prompt:** An engineer uses `unowned` inside an async callback.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 7
- **Prompt:** A senior interviewer asks whether `some` or `any` is the right return type for a factory.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 8
- **Prompt:** A coding round solution for a queue uses an array inefficiently.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 9
- **Prompt:** A codebase uses extensions heavily and discoverability is suffering.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

### Extra scenario 10
- **Prompt:** A mock interviewer says your answer is technically right but not senior enough.
- **Model answer:** Identify the semantic rule, explain the failure mode, then recommend the safer or clearer design choice.

## Last-minute verbal reps
- Explain `guard` in one sentence.
- Explain copy-on-write in one sentence.
- Explain `weak` vs `unowned` in one sentence.
- Explain `some` vs `any` in one sentence.
- Explain why enums help state modeling in one sentence.
- Explain why DTOs help Codable in one sentence.
- Explain why value semantics matter in one sentence.
- Explain the protocol extension dispatch trap in one sentence.
- Explain why Swift strings need special indexing in one sentence.
- Explain what modern Swift is optimizing for in one sentence.
