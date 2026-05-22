# 02 — Strings and Collections

Swift's collections look familiar, but the interview bar is understanding their semantics rather than quoting the API. Strings are Unicode-correct, collections are value types, and performance depends on the concrete operation rather than wishful thinking.

## `String` internals and why Unicode correctness matters
Swift `String` is a collection of extended grapheme clusters, not a bag of UTF-8 bytes or UTF-16 code units. That design preserves human-visible text correctness across languages and emoji.
### Mental model
- A single user-perceived character can contain multiple Unicode scalars.
- Operations that appear O(1) in ASCII-focused languages may need to walk variable-width boundaries in Swift.
- Correctness for names, accents, flags, and emoji matters in production apps, not just in theory.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let cafe1 = "café"
let cafe2 = "cafe\u{301}"

print(cafe1 == cafe2)
print(cafe1.count)
print(cafe2.count)
```
### Common pitfalls
- ⚠️ Assuming character count equals byte count.
- ⚠️ Treating text slicing as cheap integer-offset arithmetic.
- ⚠️ Ignoring normalization and user-visible equality rules when comparing text from external systems.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- Why can't Swift strings be indexed by integer offsets?
- What is an extended grapheme cluster?
- Why is Unicode correctness worth the extra complexity?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- The senior answer ties Unicode correctness to user trust, localization, and avoiding subtle data bugs.
- It's especially relevant for messaging, social, commerce, and international apps.
- You do not need implementation trivia, but you should explain the design tradeoff confidently.

## `String.Index`
Swift uses `String.Index` because valid positions depend on grapheme boundaries, not raw integer offsets.
### Mental model
- An index is a position that is guaranteed to be meaningful for the collection's representation.
- You derive indices from the string itself using APIs like `startIndex`, `endIndex`, and `index(after:)`.
- Think of indices as opaque tokens owned by a collection, not universal integers.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
let title = "👩🏽‍💻 Swift"
let first = title.startIndex
let second = title.index(after: first)
let prefix = title[..<second]
print(prefix)

if let space = title.firstIndex(of: " ") {
    let language = title[title.index(after: space)...]
    print(language)
}
```
### Common pitfalls
- ⚠️ Converting a string to an array of characters just to get integer indexing.
- ⚠️ Reusing indices from one string on another string.
- ⚠️ Assuming index movement by N steps is always O(1).
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- Why is `String.Index` not an `Int`?
- What can make string indexing non-constant time?
- How would you safely slice a string around a delimiter?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Opaque indices protect correctness while allowing representation flexibility.
- If you truly need byte-level work, use a lower-level view instead of fighting `String` semantics.
- This is a high-signal interview topic because it exposes whether you understand Swift beyond surface syntax.
### Quick recall
- Indices belong to a specific collection instance.
- String traversal may walk variable-width boundaries.
- Avoid assuming `index(_:offsetBy:)` is always cheap.

## Arrays
`Array` is an ordered, random-access collection with value semantics and copy-on-write storage. It is one of the most used Swift types, so complexity expectations matter.
### Mental model
- Appending at the end is amortized O(1).
- Indexed lookup is O(1).
- Insertion or removal near the front or middle is O(n) because elements shift.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
var queue = ["A", "B", "C"]
queue.append("D")
print(queue[1])
queue.insert("Start", at: 0)
queue.removeFirst()
```
### Common pitfalls
- ⚠️ Using `removeFirst()` repeatedly for queue-like workloads without considering O(n) shifting cost.
- ⚠️ Assuming value types imply full copying on every assignment or pass.
- ⚠️ Mutating arrays from many places instead of maintaining clear ownership boundaries.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- What is the complexity of append, lookup, insert, and remove for `Array`?
- How does copy-on-write change the practical cost model?
- When would `Array` be a poor queue implementation?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- A senior answer acknowledges both algorithmic complexity and data-size reality.
- `Array` is perfect until usage patterns say otherwise; then choose a more suitable data structure.
- You should know both the average fast path and the pathological mutation path.

## Dictionaries
`Dictionary` stores key-value pairs with hash-based lookup. It is ideal for membership, indexing by identifier, and aggregating data by key.
### Mental model
- Lookup, insert, and update are average O(1), assuming good hash behavior.
- Order is not the semantic contract.
- Key design matters: keys must be `Hashable` and should represent stable identity.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
var usersByID: [UUID: String] = [:]
let id = UUID()
usersByID[id] = "Taylor"

if let name = usersByID[id] {
    print(name)
}

let previous = usersByID.updateValue("Chris", forKey: id)
print(previous as Any)
```
### Common pitfalls
- ⚠️ Relying on dictionary iteration order in app logic.
- ⚠️ Choosing unstable or overly large keys without thinking about hashing and equality.
- ⚠️ Confusing missing keys with keys whose associated value is itself an optional.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- Why is dictionary lookup average O(1) rather than guaranteed O(1)?
- What makes a good dictionary key?
- Why should you avoid relying on ordering?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- In production, dictionaries often back caches, ID maps, diffing inputs, and request coalescing.
- Strong answers mention `Hashable`, collisions, and average-case assumptions.
- If ordering matters, use a separate ordered structure instead of hoping dictionary behavior never changes.

## Sets
`Set` stores unique elements with hash-based membership checks. Reach for it when uniqueness and fast membership are the core needs.
### Mental model
- A set answers 'have I seen this?' efficiently.
- Insert, remove, and contains are average O(1).
- Sets are value types and use copy-on-write.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
var selectedTags: Set<String> = ["swift", "ios"]
selectedTags.insert("performance")
print(selectedTags.contains("swift"))
selectedTags.remove("ios")
```
### Common pitfalls
- ⚠️ Using an array for uniqueness checks and paying O(n) membership cost repeatedly.
- ⚠️ Expecting deterministic order from a set.
- ⚠️ Forgetting that custom element types need correct `Hashable` behavior.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- When do you choose `Set` over `Array`?
- What are common production uses of sets?
- What goes wrong if `Hashable` is implemented incorrectly?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Sets are excellent for deduplication, feature flags, selected IDs, and graph visitation.
- Incorrect equality/hash definitions can create impossible-to-debug lookup inconsistencies.
- Senior engineers pair data structure choice with workload shape rather than personal preference.

## Mutation, value semantics, and copy-on-write
Swift standard collections are value types. Assigning or passing them copies the value semantics, while actual storage is often shared until mutation via copy-on-write.
### Mental model
- Value semantics means each variable behaves as if it owns its own value.
- Copy-on-write is an optimization that preserves value semantics while avoiding unnecessary copies.
- Mutation on one logically separate copy triggers unique storage before writing if needed.
> 💡 **Tip:** Frame this concept in terms of predictability, safety, and compiler guarantees.

### Example
```swift
var a = [1, 2, 3]
var b = a
b.append(4)

print(a)
print(b)
```
### Common pitfalls
- ⚠️ Saying 'arrays are reference types under the hood' as if that invalidates value semantics.
- ⚠️ Assuming every assignment eagerly duplicates all elements.
- ⚠️ Passing giant collections around carelessly without thinking about mutation hotspots and memory pressure.
> ⚠️ **Pitfall:** If you cannot explain *why* Swift behaves this way, interviewers assume the behavior is memorized rather than understood.

### Interview questions
- What is copy-on-write?
- How can `Array` be a value type without copying constantly?
- Why does value semantics help in real applications?
> 🎯 **Interview Answer:** Start with the language guarantee, then explain runtime behavior, then mention a real production consequence.

### Senior-level discussion
- Value semantics reduces aliasing bugs because mutation does not silently affect distant owners.
- Copy-on-write is the performance bridge that makes this practical for large collections.
- A strong senior answer contrasts semantic guarantees with storage implementation details clearly.
### Quick recall
- Semantics first, optimization second.
- Collections behave like independent values even if storage is temporarily shared.
- Mutation is the trigger point to reason about COW cost.

## Complexity cheat sheet
### 1. `Array` append vs insert at front?
Appending to the end is amortized O(1). Inserting at the front is O(n) because every existing element shifts.

### 2. `Dictionary` lookup complexity?
Average O(1), because the structure is hash-based. It is not a mathematical worst-case guarantee.

### 3. `Set.contains` complexity?
Average O(1). That is why `Set` is usually the right choice for repeated membership checks.

### 4. Why are collections value types in Swift?
Because value semantics improve predictability and reduce accidental shared mutable state, while copy-on-write keeps the common case fast.


## Extended interview practice
The following prompts are designed to help you turn **strings and collections** from reading knowledge into speaking knowledge. Use them for mock interviews, notebook recall, and timed whiteboard rehearsals.

### Flashcard 1 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **strings and collections** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **strings and collections**.
- **Senior angle:** Connect **strings and collections** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 1A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **strings and collections**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 1B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **strings and collections** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 1C — Senior signal
- **Prompt:** How do you make an answer about **strings and collections** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 1
- **Situation:** A teammate misuses **strings and collections** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

### Flashcard 2 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **strings and collections** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **strings and collections**.
- **Senior angle:** Connect **strings and collections** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 2A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **strings and collections**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 2B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **strings and collections** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 2C — Senior signal
- **Prompt:** How do you make an answer about **strings and collections** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 2
- **Situation:** A teammate misuses **strings and collections** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

### Flashcard 3 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **strings and collections** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **strings and collections**.
- **Senior angle:** Connect **strings and collections** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 3A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **strings and collections**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 3B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **strings and collections** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 3C — Senior signal
- **Prompt:** How do you make an answer about **strings and collections** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 3
- **Situation:** A teammate misuses **strings and collections** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

### Flashcard 4 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **strings and collections** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **strings and collections**.
- **Senior angle:** Connect **strings and collections** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 4A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **strings and collections**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 4B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **strings and collections** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 4C — Senior signal
- **Prompt:** How do you make an answer about **strings and collections** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 4
- **Situation:** A teammate misuses **strings and collections** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

### Flashcard 5 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **strings and collections** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **strings and collections**.
- **Senior angle:** Connect **strings and collections** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 5A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **strings and collections**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 5B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **strings and collections** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 5C — Senior signal
- **Prompt:** How do you make an answer about **strings and collections** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 5
- **Situation:** A teammate misuses **strings and collections** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

### Flashcard 6 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **strings and collections** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **strings and collections**.
- **Senior angle:** Connect **strings and collections** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 6A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **strings and collections**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 6B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **strings and collections** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 6C — Senior signal
- **Prompt:** How do you make an answer about **strings and collections** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 6
- **Situation:** A teammate misuses **strings and collections** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

### Flashcard 7 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **strings and collections** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **strings and collections**.
- **Senior angle:** Connect **strings and collections** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 7A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **strings and collections**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 7B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **strings and collections** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 7C — Senior signal
- **Prompt:** How do you make an answer about **strings and collections** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 7
- **Situation:** A teammate misuses **strings and collections** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

### Flashcard 8 — Explain the core idea
- **Prompt:** Explain the most important mental model behind **strings and collections** in under 30 seconds.
- **Strong answer:** Start with the language guarantee or abstraction boundary, then describe the runtime consequence, then give one production example.
- **Follow-up:** Name one mistake mid-level engineers often make when discussing **strings and collections**.
- **Senior angle:** Connect **strings and collections** to readability, correctness, performance, or ownership rather than only syntax.

### Flashcard 8A — Compare and contrast
- **Prompt:** Compare the most commonly confused concepts inside **strings and collections**.
- **Strong answer:** Define each concept separately, identify the shared surface area, then explain the one decisive difference interviewers care about.
- **Production note:** Mention how the wrong choice changes API design, debugging difficulty, or runtime behavior.

### Flashcard 8B — Pitfall rehearsal
- **Prompt:** What is a subtle pitfall in **strings and collections** that can survive code review?
- **Strong answer:** Name the pitfall, explain why it looks harmless at first, and describe the bug or maintenance cost it creates later.
- **Recovery move:** Say how you would refactor the code to make the correct behavior more obvious.

### Flashcard 8C — Senior signal
- **Prompt:** How do you make an answer about **strings and collections** sound senior rather than textbook?
- **Strong answer:** Tie the concept to invariants, boundaries, lifetimes, data-shape decisions, or tradeoffs between abstraction and cost.
- **Warning:** Avoid buzzwords unless you can connect them to code behavior.

### Scenario 8
- **Situation:** A teammate misuses **strings and collections** in a production feature and the bug only appears under refactoring or scale.
- **What to say:** Explain the semantic mismatch first, then show the safer redesign, then mention how tests or architecture would keep the mistake from returning.

