# 04 — Functions and Closures

Functions define API shape; closures define behavior you pass around. In Swift interviews, this topic connects API design, readability, memory management, and concurrency.

## Function parameters, argument labels, defaults, and variadics
Swift separates external labels from internal parameter names so call sites can read naturally.

### Mental model
- Argument labels are part of API design.
- Default values reduce overload noise when the default is stable and obvious.
- Variadics are ergonomic, but collections may be clearer when data shape matters.

> 💡 **Tip:** Great Swift APIs read like English at the call site.

### Example
```swift
func log(event name: String, level: String = "info", tags: String...) {
    print("event=\(name), level=\(level), tags=\(tags)")
}

log(event: "launch")
log(event: "purchase", level: "debug", tags: "checkout", "premium")
```

### Common pitfalls
- ⚠️ Overusing `_` and producing ambiguous call sites.
- ⚠️ Choosing defaults that hide important behavior.
- ⚠️ Using variadics where an array would be clearer and easier to test.

### Interview questions
- Why are argument labels important in Swift?
- When do default parameters improve API design?
- What are the tradeoffs of variadic parameters?

> 🎯 **Interview Answer:** Talk about call-site clarity, API evolution, and how labels communicate intent better than comments.

### Senior-level discussion
- Labels are especially important on public and team-shared APIs.
- Defaults should reinforce the common case without making uncommon cases confusing.
- Strong engineers optimize for long-term readability, not character count.

## `inout`, function types, and returning functions
Functions are first-class values in Swift.

### Mental model
- `inout` gives temporary writable access to a caller's variable.
- Function types describe the shape of behavior.
- Returning functions is a clean way to package strategy without conditionals everywhere.

### Example
```swift
func increment(_ value: inout Int) {
    value += 1
}

func makeFormatter(prefix: String) -> (Int) -> String {
    { number in "\(prefix)-\(number)" }
}

var count = 0
increment(&count)
let formatter = makeFormatter(prefix: "ORD")
print(formatter(42))
```

### Common pitfalls
- ⚠️ Using `inout` as a casual performance trick instead of an API choice.
- ⚠️ Returning closures that capture state opaquely.
- ⚠️ Forgetting that `inout` communicates mutation responsibility at the call site.

### Interview questions
- What does `inout` actually mean?
- Why can returning functions be cleaner than branching?
- How do function types show up in iOS code?

### Senior-level discussion
- Returned closures appear in formatters, dependency injection, reducers, and retry logic.
- Prefer semantic clarity over clever functional abstractions.
- `inout` is best for small, local mutation APIs.

## Higher-order functions
Higher-order functions either take functions or return functions.

### Mental model
- `map` transforms.
- `filter` selects.
- `reduce` accumulates.
- They are tools, not a personality trait.

### Example
```swift
struct Product {
    let name: String
    let price: Double
    let isAvailable: Bool
}

let products = [
    Product(name: "A", price: 10, isAvailable: true),
    Product(name: "B", price: 30, isAvailable: false),
    Product(name: "C", price: 20, isAvailable: true)
]

let total = products
    .filter(\.isAvailable)
    .map(\.price)
    .reduce(0, +)
```

### Common pitfalls
- ⚠️ Building unreadable one-liners.
- ⚠️ Ignoring intermediate allocations.
- ⚠️ Using chains where a loop would be clearer.

> 💡 **Tip:** Mention `lazy` when discussing performance of transformation pipelines.

### Interview questions
- When are higher-order functions the right choice?
- What are the tradeoffs versus imperative loops?
- How can `lazy` change the performance story?

### Senior-level discussion
- Favor the clearest representation for the workload.
- Understand data size, intermediate collections, and debugging tradeoffs.
- Expressiveness is valuable only if maintenance stays easy.

## Closure syntax, shorthand, and trailing closures
Swift closure syntax scales from explicit to concise.

### Mental model
- Start explicit, then remove ceremony only if readability improves.
- Trailing closure syntax works well when the closure is the main point of the call.
- `$0` is fine for tiny closures but becomes opaque quickly.

### Example
```swift
let names = ["Chris", "Alex", "Ewa", "Barry"]

let sorted = names.sorted { lhs, rhs in
    lhs.localizedCaseInsensitiveCompare(rhs) == .orderedAscending
}

let lengths = names.map { $0.count }
```

### Common pitfalls
- ⚠️ Overusing shorthand arguments in non-trivial closures.
- ⚠️ Making multi-trailing-closure call sites hard to scan.
- ⚠️ Confusing terseness with clarity.

### Interview questions
- How explicit should a closure be?
- When is trailing closure syntax beneficial?
- Why can `$0` hurt readability?

### Senior-level discussion
- Name parameters when domain meaning matters.
- Use concise syntax only when the closure is obvious at a glance.
- Readability is especially important in SwiftUI-heavy code.

## Capture semantics and capture lists
Closures capture surrounding values.

### Mental model
- Value semantics and reference semantics still matter inside captures.
- Capture lists let you control ownership and snapshot values.
- Ask: who owns the closure, and what does the closure own back?

### Example
```swift
final class ImageLoader {
    var onComplete: (() -> Void)?

    func start() {
        onComplete = { [weak self] in
            guard let self else { return }
            print("Finished: \(self)")
        }
    }
}
```

### Common pitfalls
- ⚠️ Writing `[weak self]` everywhere without understanding the graph.
- ⚠️ Using `unowned` when lifetime is not guaranteed.
- ⚠️ Forgetting capture lists can snapshot values too.

### Interview questions
- What does a closure capture?
- When should you use `weak` vs `unowned`?
- Can a capture list store value copies?

> 🎯 **Interview Answer:** Start with the ownership graph, not the syntax. Explain who stores the closure and whether the closure retains that owner back.

### Senior-level discussion
- Not every closure needs weak capture.
- Nonescaping closures usually do not create retain cycles.
- Long-lived callbacks in view models, coordinators, and managers deserve explicit ownership reasoning.

## Escaping, nonescaping, and `@autoclosure`
A closure is escaping if it can outlive the call.

### Mental model
- Stored callbacks and async callbacks are escaping.
- Nonescaping is the default because it is easier to reason about.
- `@autoclosure` delays evaluation while keeping the call site clean.

### Example
```swift
final class RetryController {
    private var pending: (() -> Void)?

    func schedule(_ work: @escaping () -> Void) {
        pending = work
    }
}

func logIfNeeded(_ message: @autoclosure () -> String, enabled: Bool) {
    guard enabled else { return }
    print(message())
}
```

### Common pitfalls
- ⚠️ Marking closures `@escaping` without a reason.
- ⚠️ Using `@autoclosure` to hide surprising side effects.
- ⚠️ Forgetting that `async` and callback storage usually imply escaping behavior.

### Interview questions
- What makes a closure escaping?
- Why is nonescaping the default?
- When is `@autoclosure` appropriate?

### Senior-level discussion
- Escaping vs nonescaping is a lifetime conversation.
- `@autoclosure` is great for assertion-style and logging APIs.
- Modern Swift code increasingly combines escaping closures with async bridges and task-based execution.

## Topic-specific oral drills
Use these prompts to turn **functions and closures** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — argument labels
- **Definition prompt:** Explain what **argument labels** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **argument labels**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **argument labels**, not just the spelling.
- **Production example to mention:** Connect **argument labels** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **argument labels** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **argument labels**.
- **One-line close:** End with a concise summary that tells the interviewer why **argument labels** matters in production code.

### Follow-up 1 — argument labels
- **Compare prompt:** Contrast **argument labels** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **argument labels** is misunderstood.
- **Code review angle:** Describe what misuse of **argument labels** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **argument labels** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **argument labels**.

### Drill 2 — internal vs external parameter names
- **Definition prompt:** Explain what **internal vs external parameter names** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **internal vs external parameter names**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **internal vs external parameter names**, not just the spelling.
- **Production example to mention:** Connect **internal vs external parameter names** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **internal vs external parameter names** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **internal vs external parameter names**.
- **One-line close:** End with a concise summary that tells the interviewer why **internal vs external parameter names** matters in production code.

### Follow-up 2 — internal vs external parameter names
- **Compare prompt:** Contrast **internal vs external parameter names** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **internal vs external parameter names** is misunderstood.
- **Code review angle:** Describe what misuse of **internal vs external parameter names** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **internal vs external parameter names** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **internal vs external parameter names**.

### Drill 3 — default parameters
- **Definition prompt:** Explain what **default parameters** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **default parameters**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **default parameters**, not just the spelling.
- **Production example to mention:** Connect **default parameters** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **default parameters** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **default parameters**.
- **One-line close:** End with a concise summary that tells the interviewer why **default parameters** matters in production code.

### Follow-up 3 — default parameters
- **Compare prompt:** Contrast **default parameters** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **default parameters** is misunderstood.
- **Code review angle:** Describe what misuse of **default parameters** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **default parameters** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **default parameters**.

### Drill 4 — variadic parameters
- **Definition prompt:** Explain what **variadic parameters** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **variadic parameters**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **variadic parameters**, not just the spelling.
- **Production example to mention:** Connect **variadic parameters** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **variadic parameters** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **variadic parameters**.
- **One-line close:** End with a concise summary that tells the interviewer why **variadic parameters** matters in production code.

### Follow-up 4 — variadic parameters
- **Compare prompt:** Contrast **variadic parameters** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **variadic parameters** is misunderstood.
- **Code review angle:** Describe what misuse of **variadic parameters** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **variadic parameters** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **variadic parameters**.

### Drill 5 — `inout` semantics
- **Definition prompt:** Explain what **`inout` semantics** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`inout` semantics**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`inout` semantics**, not just the spelling.
- **Production example to mention:** Connect **`inout` semantics** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`inout` semantics** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`inout` semantics**.
- **One-line close:** End with a concise summary that tells the interviewer why **`inout` semantics** matters in production code.

### Follow-up 5 — `inout` semantics
- **Compare prompt:** Contrast **`inout` semantics** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`inout` semantics** is misunderstood.
- **Code review angle:** Describe what misuse of **`inout` semantics** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`inout` semantics** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`inout` semantics**.

### Drill 6 — function types
- **Definition prompt:** Explain what **function types** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **function types**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **function types**, not just the spelling.
- **Production example to mention:** Connect **function types** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **function types** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **function types**.
- **One-line close:** End with a concise summary that tells the interviewer why **function types** matters in production code.

### Follow-up 6 — function types
- **Compare prompt:** Contrast **function types** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **function types** is misunderstood.
- **Code review angle:** Describe what misuse of **function types** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **function types** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **function types**.

### Drill 7 — returning functions
- **Definition prompt:** Explain what **returning functions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **returning functions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **returning functions**, not just the spelling.
- **Production example to mention:** Connect **returning functions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **returning functions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **returning functions**.
- **One-line close:** End with a concise summary that tells the interviewer why **returning functions** matters in production code.

### Follow-up 7 — returning functions
- **Compare prompt:** Contrast **returning functions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **returning functions** is misunderstood.
- **Code review angle:** Describe what misuse of **returning functions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **returning functions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **returning functions**.

### Drill 8 — higher-order functions
- **Definition prompt:** Explain what **higher-order functions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **higher-order functions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **higher-order functions**, not just the spelling.
- **Production example to mention:** Connect **higher-order functions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **higher-order functions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **higher-order functions**.
- **One-line close:** End with a concise summary that tells the interviewer why **higher-order functions** matters in production code.

### Follow-up 8 — higher-order functions
- **Compare prompt:** Contrast **higher-order functions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **higher-order functions** is misunderstood.
- **Code review angle:** Describe what misuse of **higher-order functions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **higher-order functions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **higher-order functions**.

### Drill 9 — readability of `map`/`filter`/`reduce`
- **Definition prompt:** Explain what **readability of `map`/`filter`/`reduce`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **readability of `map`/`filter`/`reduce`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **readability of `map`/`filter`/`reduce`**, not just the spelling.
- **Production example to mention:** Connect **readability of `map`/`filter`/`reduce`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **readability of `map`/`filter`/`reduce`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **readability of `map`/`filter`/`reduce`**.
- **One-line close:** End with a concise summary that tells the interviewer why **readability of `map`/`filter`/`reduce`** matters in production code.

### Follow-up 9 — readability of `map`/`filter`/`reduce`
- **Compare prompt:** Contrast **readability of `map`/`filter`/`reduce`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **readability of `map`/`filter`/`reduce`** is misunderstood.
- **Code review angle:** Describe what misuse of **readability of `map`/`filter`/`reduce`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **readability of `map`/`filter`/`reduce`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **readability of `map`/`filter`/`reduce`**.

### Drill 10 — closure syntax levels
- **Definition prompt:** Explain what **closure syntax levels** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **closure syntax levels**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **closure syntax levels**, not just the spelling.
- **Production example to mention:** Connect **closure syntax levels** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **closure syntax levels** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **closure syntax levels**.
- **One-line close:** End with a concise summary that tells the interviewer why **closure syntax levels** matters in production code.

### Follow-up 10 — closure syntax levels
- **Compare prompt:** Contrast **closure syntax levels** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **closure syntax levels** is misunderstood.
- **Code review angle:** Describe what misuse of **closure syntax levels** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **closure syntax levels** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **closure syntax levels**.

### Drill 11 — shorthand arguments
- **Definition prompt:** Explain what **shorthand arguments** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **shorthand arguments**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **shorthand arguments**, not just the spelling.
- **Production example to mention:** Connect **shorthand arguments** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **shorthand arguments** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **shorthand arguments**.
- **One-line close:** End with a concise summary that tells the interviewer why **shorthand arguments** matters in production code.

### Follow-up 11 — shorthand arguments
- **Compare prompt:** Contrast **shorthand arguments** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **shorthand arguments** is misunderstood.
- **Code review angle:** Describe what misuse of **shorthand arguments** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **shorthand arguments** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **shorthand arguments**.

### Drill 12 — trailing closures
- **Definition prompt:** Explain what **trailing closures** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **trailing closures**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **trailing closures**, not just the spelling.
- **Production example to mention:** Connect **trailing closures** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **trailing closures** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **trailing closures**.
- **One-line close:** End with a concise summary that tells the interviewer why **trailing closures** matters in production code.

### Follow-up 12 — trailing closures
- **Compare prompt:** Contrast **trailing closures** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **trailing closures** is misunderstood.
- **Code review angle:** Describe what misuse of **trailing closures** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **trailing closures** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **trailing closures**.

### Drill 13 — capture semantics
- **Definition prompt:** Explain what **capture semantics** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **capture semantics**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **capture semantics**, not just the spelling.
- **Production example to mention:** Connect **capture semantics** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **capture semantics** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **capture semantics**.
- **One-line close:** End with a concise summary that tells the interviewer why **capture semantics** matters in production code.

### Follow-up 13 — capture semantics
- **Compare prompt:** Contrast **capture semantics** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **capture semantics** is misunderstood.
- **Code review angle:** Describe what misuse of **capture semantics** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **capture semantics** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **capture semantics**.

### Drill 14 — value capture in lists
- **Definition prompt:** Explain what **value capture in lists** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **value capture in lists**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **value capture in lists**, not just the spelling.
- **Production example to mention:** Connect **value capture in lists** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **value capture in lists** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **value capture in lists**.
- **One-line close:** End with a concise summary that tells the interviewer why **value capture in lists** matters in production code.

### Follow-up 14 — value capture in lists
- **Compare prompt:** Contrast **value capture in lists** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **value capture in lists** is misunderstood.
- **Code review angle:** Describe what misuse of **value capture in lists** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **value capture in lists** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **value capture in lists**.

### Drill 15 — `weak self` capture
- **Definition prompt:** Explain what **`weak self` capture** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`weak self` capture**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`weak self` capture**, not just the spelling.
- **Production example to mention:** Connect **`weak self` capture** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`weak self` capture** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`weak self` capture**.
- **One-line close:** End with a concise summary that tells the interviewer why **`weak self` capture** matters in production code.

### Follow-up 15 — `weak self` capture
- **Compare prompt:** Contrast **`weak self` capture** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`weak self` capture** is misunderstood.
- **Code review angle:** Describe what misuse of **`weak self` capture** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`weak self` capture** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`weak self` capture**.

### Drill 16 — `unowned self` capture
- **Definition prompt:** Explain what **`unowned self` capture** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`unowned self` capture**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`unowned self` capture**, not just the spelling.
- **Production example to mention:** Connect **`unowned self` capture** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`unowned self` capture** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`unowned self` capture**.
- **One-line close:** End with a concise summary that tells the interviewer why **`unowned self` capture** matters in production code.

### Follow-up 16 — `unowned self` capture
- **Compare prompt:** Contrast **`unowned self` capture** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`unowned self` capture** is misunderstood.
- **Code review angle:** Describe what misuse of **`unowned self` capture** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`unowned self` capture** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`unowned self` capture**.

### Drill 17 — escaping closures
- **Definition prompt:** Explain what **escaping closures** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **escaping closures**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **escaping closures**, not just the spelling.
- **Production example to mention:** Connect **escaping closures** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **escaping closures** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **escaping closures**.
- **One-line close:** End with a concise summary that tells the interviewer why **escaping closures** matters in production code.

### Follow-up 17 — escaping closures
- **Compare prompt:** Contrast **escaping closures** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **escaping closures** is misunderstood.
- **Code review angle:** Describe what misuse of **escaping closures** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **escaping closures** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **escaping closures**.

### Drill 18 — nonescaping closures
- **Definition prompt:** Explain what **nonescaping closures** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **nonescaping closures**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **nonescaping closures**, not just the spelling.
- **Production example to mention:** Connect **nonescaping closures** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **nonescaping closures** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **nonescaping closures**.
- **One-line close:** End with a concise summary that tells the interviewer why **nonescaping closures** matters in production code.

### Follow-up 18 — nonescaping closures
- **Compare prompt:** Contrast **nonescaping closures** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **nonescaping closures** is misunderstood.
- **Code review angle:** Describe what misuse of **nonescaping closures** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **nonescaping closures** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **nonescaping closures**.

### Drill 19 — `@autoclosure`
- **Definition prompt:** Explain what **`@autoclosure`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`@autoclosure`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`@autoclosure`**, not just the spelling.
- **Production example to mention:** Connect **`@autoclosure`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`@autoclosure`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`@autoclosure`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`@autoclosure`** matters in production code.

### Follow-up 19 — `@autoclosure`
- **Compare prompt:** Contrast **`@autoclosure`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`@autoclosure`** is misunderstood.
- **Code review angle:** Describe what misuse of **`@autoclosure`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`@autoclosure`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`@autoclosure`**.

### Drill 20 — retain cycles via closures
- **Definition prompt:** Explain what **retain cycles via closures** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **retain cycles via closures**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **retain cycles via closures**, not just the spelling.
- **Production example to mention:** Connect **retain cycles via closures** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **retain cycles via closures** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **retain cycles via closures**.
- **One-line close:** End with a concise summary that tells the interviewer why **retain cycles via closures** matters in production code.

### Follow-up 20 — retain cycles via closures
- **Compare prompt:** Contrast **retain cycles via closures** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **retain cycles via closures** is misunderstood.
- **Code review angle:** Describe what misuse of **retain cycles via closures** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **retain cycles via closures** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **retain cycles via closures**.

### Drill 21 — closure lifetime debugging
- **Definition prompt:** Explain what **closure lifetime debugging** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **closure lifetime debugging**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **closure lifetime debugging**, not just the spelling.
- **Production example to mention:** Connect **closure lifetime debugging** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **closure lifetime debugging** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **closure lifetime debugging**.
- **One-line close:** End with a concise summary that tells the interviewer why **closure lifetime debugging** matters in production code.

### Follow-up 21 — closure lifetime debugging
- **Compare prompt:** Contrast **closure lifetime debugging** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **closure lifetime debugging** is misunderstood.
- **Code review angle:** Describe what misuse of **closure lifetime debugging** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **closure lifetime debugging** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **closure lifetime debugging**.

### Drill 22 — API design through functions
- **Definition prompt:** Explain what **API design through functions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **API design through functions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **API design through functions**, not just the spelling.
- **Production example to mention:** Connect **API design through functions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **API design through functions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **API design through functions**.
- **One-line close:** End with a concise summary that tells the interviewer why **API design through functions** matters in production code.

### Follow-up 22 — API design through functions
- **Compare prompt:** Contrast **API design through functions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **API design through functions** is misunderstood.
- **Code review angle:** Describe what misuse of **API design through functions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **API design through functions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **API design through functions**.

