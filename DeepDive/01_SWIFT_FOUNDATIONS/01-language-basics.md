# 01 — Language Basics

Swift interviews often begin with deceptively simple language questions. The trick is not syntax recall; it is being able to explain how Swift nudges you toward safer, more explicit code.

## Why this topic matters

- It sets the tone for every later answer around API design, correctness, and readability.
- Many coding screens expose fundamentals through loops, branching, and tuple destructuring.
- Senior candidates should demonstrate taste: choosing constructs that make invalid states harder to represent.
## Constants and variables (`let` vs `var`)
Swift makes immutability the easy default. Use `let` for bindings that should never be rebound and `var` when mutation is a real requirement.
### Mental model
- A binding is a name attached to a value or reference.
- `let` freezes the binding; it does not magically deep-freeze the internals of referenced objects.
- Prefer the narrowest mutability surface possible because it simplifies reasoning and concurrency safety.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let appName = "Bootcamp"
var attemptCount = 0
attemptCount += 1

final class Session {
    var token: String
    init(token: String) { self.token = token }
}

let session = Session(token: "abc")
session.token = "refreshed"   // allowed: object mutates
// session = Session(token: "new") // not allowed: binding is constant
```
### Common pitfalls
- ⚠️ Using `var` everywhere signals weak ownership and makes code review harder.
- ⚠️ Assuming `let` on a class instance makes the instance immutable; only the reference is immutable.
- ⚠️ Mutating large scopes when a smaller local `var` would isolate change more clearly.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- Why does Swift encourage `let` by default?
- What is the difference between a constant binding to a struct and a constant binding to a class instance?
- How does immutability help with thread-safety and reasoning?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- In production code, defaulting to `let` reduces accidental state changes and shrinks the bug surface.
- In concurrent code, immutable data is dramatically easier to send across isolation boundaries.
- When mutation is needed, scope it tightly so the next engineer can see where state changes occur.
### Quick recall
- Use `let` unless mutation is semantically required.
- `let` on a class instance protects the reference, not the object.
- Prefer designing APIs so callers do not need to mutate internal state directly.

## Type inference and explicit typing
Swift infers types aggressively, but explicit annotations are still valuable when they improve clarity, stabilize APIs, or avoid surprising inference results.
### Mental model
- Inference is local reasoning performed by the compiler from surrounding expressions.
- Annotations are documentation plus a constraint on future edits.
- In interviews, explain when inference helps readability and when it hides intent.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let retries = 3              // inferred Int
let timeout: TimeInterval = 2.5
let headers: [String: String] = [
    "Accept": "application/json",
    "Authorization": "Bearer token"
]

let pair = (status: 200, body: Data())
let codes = [200, 201, 204]
```
### Common pitfalls
- ⚠️ Using explicit types everywhere creates noise.
- ⚠️ Relying on inference when the inferred type is broader or narrower than intended.
- ⚠️ For literals, forgetting that context changes the inferred type, especially with numeric values.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- When should you prefer explicit types over inference?
- Can type inference ever hurt API evolution or readability?
- How does compiler context influence inferred numeric types?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Use explicit typing at module boundaries, for public APIs, and when a literal's meaning is not obvious.
- For complex closures or generic code, annotations reduce debugging time by making compiler assumptions visible.
- Senior engineers optimize for long-term readability rather than saving three characters today.
### Quick recall
- Inference is great for obvious locals.
- Annotations help at boundaries and in generic-heavy code.
- Make intent obvious before making syntax short.

## Tuples
Tuples are lightweight groupings of values. They are excellent for local, temporary structure but usually not a substitute for named domain models.
### Mental model
- A tuple is structural, not nominal.
- Named tuple elements improve readability but do not create a first-class type with methods and semantics.
- Promote tuples to a struct when the data has behavior, invariants, or reuse value.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let response = (code: 200, message: "OK")
if response.code == 200 {
    print(response.message)
}

let point = (x: 12, y: 8)
let (x, y) = point
print(x + y)
```
### Common pitfalls
- ⚠️ Returning giant tuples from APIs instead of defining a real type.
- ⚠️ Overusing positional access like `.0` and `.1`.
- ⚠️ Using tuples in places where invariants should be enforced by a dedicated model.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- When is a tuple appropriate versus a struct?
- What does it mean that tuples are structural types?
- Why can tuple-heavy APIs be a maintainability smell?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Tuples shine inside a function where data is transient and meaning is obvious from context.
- They become fragile at API boundaries because adding or reordering elements is awkward and semantically muddy.
- A struct is a better home once names, invariants, testability, or protocol conformance matter.

## Control flow with `if`, `else`, and conditions
Swift conditionals are explicit and type-safe. Conditions must be `Bool`; there is no accidental truthiness like in some other languages.
### Mental model
- The compiler prevents ambiguous conditions by requiring a real Boolean expression.
- Conditionals are a readability tool; flattening logic matters as much as correctness.
- Think about the happy path first, then isolate exceptional branches.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let isLoggedIn = true
let hasSubscription = false

if isLoggedIn && hasSubscription {
    print("Show premium content")
} else if isLoggedIn {
    print("Show paywall")
} else {
    print("Show login")
}
```
### Common pitfalls
- ⚠️ Writing deeply nested `if` trees instead of exiting early with `guard`.
- ⚠️ Packing too much work into condition expressions.
- ⚠️ Forgetting that readability matters more than being clever with operators.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- Why does Swift require conditions to be `Bool`?
- How would you refactor a deeply nested conditional pyramid?
- What makes a conditional branch easy to maintain?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Senior engineers treat control flow as API design for future readers.
- Early exits, small helpers, and clear condition names are worth more than syntactic compression.
- You should be able to explain every condition in domain language, not just operator language.

## `switch` and pattern matching
Swift's `switch` is powerful because it is exhaustive and pattern-driven, not just value-comparison sugar.
### Mental model
- Think of a case as a pattern that either matches or does not match a value.
- Exhaustiveness pushes you to consider every possible state.
- Pattern matching becomes especially important with enums and tuples.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let event = (type: "tap", location: (x: 20, y: 30))

switch event {
case ("tap", let location) where location.x > 10:
    print("Tap on the right side")
case ("tap", _):
    print("Tap elsewhere")
default:
    print("Unhandled event type")
}

let statusCode = 204
switch statusCode {
case 200...299:
    print("Success")
case 400...499:
    print("Client error")
case 500...599:
    print("Server error")
default:
    print("Unexpected")
}
```
### Common pitfalls
- ⚠️ Using `default` too early and losing exhaustiveness on enums.
- ⚠️ Forgetting `where` can refine a pattern cleanly.
- ⚠️ Treating `switch` as a giant replacement for polymorphism when behavior should live on the type.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- What makes Swift's `switch` more powerful than in many languages?
- Why is exhaustiveness valuable?
- When would you avoid a `default` case?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- For enums representing app state, exhaustive `switch` statements act like compile-time checklists during refactors.
- Pattern matching is one of the reasons enums are so expressive in Swift.
- A clean answer often mentions `where`, tuple matching, ranges, and binding with `let`.
### Quick recall
- `switch` matches patterns, not only exact values.
- Avoid `default` on enums when you want compiler help during evolution.
- Use `where` to keep matching logic precise without nested `if` statements.

## `guard` and early exit
`guard` expresses preconditions for continuing. It keeps the success path unindented and makes failure behavior explicit.
### Mental model
- Read `guard` as: beyond this line, these conditions are guaranteed to hold.
- Bindings created by `guard let` stay available in the outer scope after the statement.
- Use `guard` for prerequisites, not as a stylistic replacement for every `if`.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
func makeAuthorizedRequest(token: String?) -> URLRequest? {
    guard let token, !token.isEmpty else {
        return nil
    }

    var request = URLRequest(url: URL(string: "https://example.com")!)
    request.addValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    return request
}
```
### Common pitfalls
- ⚠️ Using `guard` for branches that are not truly preconditions.
- ⚠️ Putting too much unrelated work inside the `else` block.
- ⚠️ Forgetting that the `else` branch must exit the current scope.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- When is `guard` preferable to `if`?
- Why do many Swift style guides prefer early returns?
- What scope rules make `guard let` convenient?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- `guard` is especially useful in lifecycle methods, async functions, and validation-heavy code.
- Senior-level code often reads as a series of explicit requirements followed by a focused happy path.
- If the failure path is the exceptional path, `guard` is usually the clearer spelling.

## Ranges
Ranges model sequences or boundaries of comparable values and are frequently used in slicing, loops, and `switch` patterns.
### Mental model
- Closed range `a...b` includes both ends.
- Half-open range `a..<b` includes the lower bound and excludes the upper bound.
- One-sided ranges are useful for slicing and boundary checks.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let closed = 1...3
let halfOpen = 1..<3

let scores = [90, 82, 77, 98]
let topTwo = scores[..<2]

let progress = 0.72
switch progress {
case 0..<0.5:
    print("Getting started")
case 0.5..<1.0:
    print("In progress")
default:
    print("Complete")
}
```
### Common pitfalls
- ⚠️ Off-by-one errors when choosing closed versus half-open ranges.
- ⚠️ Assuming integer-like behavior in APIs that operate on collection indices or non-integer comparables.
- ⚠️ Forgetting that collection slices preserve original indexing behavior.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- When do you use `...` vs `..<`?
- Why are half-open ranges common in indexing?
- How do ranges integrate with pattern matching?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Half-open ranges compose naturally with zero-based indexing because the upper bound can be the collection count.
- A strong candidate connects ranges to APIs like `prefix`, slicing, and `switch` case matching.
- In performance-sensitive code, correct bounds handling avoids both crashes and subtle logic bugs.

## Loops: `for`, `while`, and `repeat`
Swift gives you three main loop shapes. Choose based on whether you are iterating a sequence, looping until a condition changes, or guaranteeing at least one pass.
### Mental model
- `for-in` is the default for sequences and collections.
- `while` checks before entering the body.
- `repeat-while` executes the body once before checking the condition.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let ids = [101, 102, 103]
for id in ids {
    print("Syncing \(id)")
}

var attempts = 0
while attempts < 3 {
    attempts += 1
}

var page = 0
repeat {
    page += 1
    print("Fetched page \(page)")
} while page < 1
```
### Common pitfalls
- ⚠️ Using index-based loops when direct element iteration is clearer.
- ⚠️ Creating accidental infinite `while` loops by forgetting state mutation.
- ⚠️ Using `repeat-while` without thinking through side effects on the first guaranteed iteration.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- Why is `for-in` usually preferred over manual indexing?
- What is the difference between `while` and `repeat-while`?
- When might manual indexing still be justified?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Prefer expressive sequence APIs or `for-in` unless you genuinely need index coordination or mutation patterns.
- When performance matters, know whether an index-based loop changes complexity or just readability.
- For collections like `String`, manual integer indexing is wrong because indices are not trivial offsets.
### Quick recall
- Default to `for-in`.
- Use `while` for condition-driven repetition.
- Use `repeat-while` only when the first iteration must always happen.

## Labeled statements, `break`, and `continue`
Labels let you target control-flow changes at a specific loop or switch, which can be useful in nested algorithms.
### Mental model
- A label names a loop or switch statement.
- `break labelName` exits that statement.
- `continue labelName` moves to the next iteration of the labeled loop.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let grid = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
var found: (row: Int, col: Int)?

search: for (rowIndex, row) in grid.enumerated() {
    for (colIndex, value) in row.enumerated() {
        if value == 5 {
            found = (rowIndex, colIndex)
            break search
        }
    }
}

print(found as Any)
```
### Common pitfalls
- ⚠️ Using labels to compensate for overly complex logic that should be extracted into a function.
- ⚠️ Forgetting that `break` inside a `switch` behaves differently from exiting surrounding loops.
- ⚠️ Making nested loops so large that labels become hard to track.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- When are labeled statements appropriate?
- What is `break search` doing in the example?
- When would you refactor labels away?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Labels are fine in localized algorithms like search, parsing, or matrix traversal.
- If the code grows beyond a small control-flow aid, extract helper functions and return early instead.
- The senior answer balances pragmatic use with readability discipline.

## Interview drill
### 1. Why does Swift forbid non-Boolean conditions?
Because the language optimizes for explicitness and safety. There is no implicit truthiness conversion from integers, optionals, or collections, so code is easier to review and harder to misread.

### 2. What is the practical difference between `let` and `var` in day-to-day engineering?
`let` communicates that a binding should not change and helps the compiler and human readers reason about state. `var` should be introduced only where mutation is part of the design, not out of habit.

### 3. How do you sound senior when discussing basic control flow?
Talk about intention-revealing code, early exits, exhaustiveness, and reducing invalid states. The syntax is easy; the design judgment is what differentiates levels.


## Extended interview practice
The following prompts are designed to help you turn **language basics** from reading knowledge into speaking knowledge. Use them for mock interviews, notebook recall, and timed whiteboard rehearsals.

### Flashcard 1 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **language basics** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **language basics**.
- **Senior angle:** Connect **language basics** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 1A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **language basics**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 1B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **language basics** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 1C — Senior signal
- **Prompt:** How do you make an answer about **language basics** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 1
- **Situation:** A teammate misuses **language basics** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

### Flashcard 2 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **language basics** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **language basics**.
- **Senior angle:** Connect **language basics** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 2A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **language basics**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 2B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **language basics** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 2C — Senior signal
- **Prompt:** How do you make an answer about **language basics** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 2
- **Situation:** A teammate misuses **language basics** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

