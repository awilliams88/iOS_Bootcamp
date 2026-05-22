# Module 01 — Swift Foundations

Master the language layer that sits underneath every strong iOS interview answer. This module is written for experienced engineers who want to sound current, precise, and senior when discussing Swift.

## How to use this module

- Read the files in order on the first pass; later passes can jump directly to weak areas.
- Treat code snippets as speaking prompts: explain what the compiler does, what the runtime does, and why it matters in an app.
- After each file, close the tab and summarize the topic aloud in two minutes.
- Use the final cheatsheet file before mock interviews or live screens.
> 💡 **Tip:** Senior candidates are not expected to memorize syntax perfectly. They are expected to reason from Swift's semantics and tradeoffs.

## Topic map

- [01-language-basics.md](./01-language-basics.md) — Variables, control flow, switching, loops, and labeled statements.
- [02-strings-and-collections.md](./02-strings-and-collections.md) — Unicode-correct strings, indexing, arrays, dictionaries, and sets.
- [03-optionals.md](./03-optionals.md) — Safe absence, unwrapping patterns, chaining, and optional transformations.
- [04-functions-and-closures.md](./04-functions-and-closures.md) — Function design, closures, capture semantics, and escaping behavior.
- [05-structs-classes-enums.md](./05-structs-classes-enums.md) — Value vs reference semantics, identity, inheritance, and enum modeling.
- [06-properties-and-initialization.md](./06-properties-and-initialization.md) — Stored/computed/lazy properties, wrappers, and initialization rules.
- [07-protocols-and-pop.md](./07-protocols-and-pop.md) — Protocols, composition, default implementations, and POP tradeoffs.
- [08-generics-and-associated-types.md](./08-generics-and-associated-types.md) — Generics, associated types, existentials, opaque types, and erasure.
- [09-arc-memory-management.md](./09-arc-memory-management.md) — ARC ownership, retain cycles, weak/unowned, and closure captures.
- [10-error-handling-and-result.md](./10-error-handling-and-result.md) — throws, Result, defer, rethrows, and async error handling.
- [11-access-control-and-dispatch.md](./11-access-control-and-dispatch.md) — Visibility, dispatch mechanisms, witness tables, vtables, and traps.
- [12-codable-and-extensions.md](./12-codable-and-extensions.md) — Codable strategies, custom decoding, extensions, and subscripts.
- [13-modern-swift.md](./13-modern-swift.md) — Swift 5.9/5.10 features plus the Swift 6 direction of travel.
- [14-interview-drills-and-cheatsheet.md](./14-interview-drills-and-cheatsheet.md) — Rapid-fire Q&A, coding drills, scenarios, and a revision sheet.

## Recommended study sequence

- Days 1-2: Language basics, strings/collections, and optionals.
- Days 3-4: Functions, closures, structs/classes/enums, and properties.
- Days 5-6: Protocols, generics, ARC, and error handling.
- Days 7+: Access control, Codable, modern Swift, then drills and timed speaking practice.
## Interview outcomes for this module

- Explain value semantics versus reference semantics with production examples.
- Describe ARC, retain cycles, and closure capture decisions without hand-waving.
- Discuss generics, protocols, existentials, and dispatch with enough depth for senior screens.
- Confidently answer practical Swift questions around optionals, collections, errors, and Codable.
## What modern means here

This module assumes Swift 5.9 and 5.10 syntax and language behavior, and it also calls out important Swift 6 direction-of-travel topics where interviewers may probe for awareness.
## Module mastery checklist

Use this checklist as you progress through the module.

### 01-language-basics.md
- Explain `let` vs `var` without reducing the answer to syntax only.
- Describe why exhaustive `switch` improves refactor safety.
- Give one production example where `guard` improves control flow.
- Explain why labeled statements are useful but rare.

### 02-strings-and-collections.md
- Explain why Swift `String` cannot use integer indexing safely.
- Compare `Array`, `Dictionary`, and `Set` by workload shape.
- State the real meaning of copy-on-write.
- Describe one performance pitfall for each major collection.

### 03-optionals.md
- Explain optionals as explicit absence in the type system.
- Compare `if let`, `guard let`, and nil coalescing.
- Justify or reject a force unwrap using an invariant.
- Explain `map` vs `flatMap` on `Optional`.

### 04-functions-and-closures.md
- Explain why argument labels matter in Swift API design.
- Describe `inout` in terms of semantics, not folklore.
- Explain closure capture using an ownership graph.
- Compare escaping and nonescaping closures.

### 05-structs-classes-enums.md
- Contrast value semantics with reference semantics.
- Explain what `===` tests and when it matters.
- Model UI state with an enum and explain why it helps.
- Explain raw values vs associated values clearly.

### 06-properties-and-initialization.md
- Compare stored, computed, lazy, static, and class properties.
- Explain when property wrappers add value.
- Walk through designated vs convenience initializers.
- Describe two-phase initialization confidently.

### 07-protocols-and-pop.md
- Define protocol requirements beyond methods only.
- Explain protocol composition with a real example.
- Describe POP as a design philosophy with tradeoffs.
- Explain the protocol extension dispatch trap accurately.

### 08-generics-and-associated-types.md
- Explain what generic parameters represent.
- Use a `where` clause in a clear example.
- Explain why associated types exist.
- Compare `any`, `some`, and type erasure.

### 09-arc-memory-management.md
- Draw a retain cycle as a reference graph.
- Compare strong, weak, and unowned references.
- Explain why closures often create cycles.
- Describe when `weak` is safer than `unowned`.

### 10-error-handling-and-result.md
- Explain when a function should `throw`.
- Compare `try`, `try?`, and `try!`.
- Explain what `defer` guarantees.
- Compare `throws` with `Result` cleanly.

### 11-access-control-and-dispatch.md
- Define all five access levels accurately.
- Explain the difference between `public` and `open`.
- Describe static dispatch vs dynamic dispatch.
- Explain witness tables and extension dispatch traps.

### 12-codable-and-extensions.md
- Describe when Codable synthesis is enough.
- Explain how `CodingKeys` and strategies help.
- Walk through a nested container example.
- Explain what extensions can and cannot add.

### 13-modern-swift.md
- Summarize what modern Swift is optimizing for.
- Explain parameter packs at a conceptual level.
- Describe macros without comparing them incorrectly to textual macros.
- Explain why ownership and concurrency rules are tightening.

### 14-interview-drills-and-cheatsheet.md
- Answer the rapid-fire questions aloud without notes.
- Walk through the linked-list reversal solution step by step.
- Explain the LRU cache data structure tradeoffs.
- Use the cheat sheet as a final-pass revision tool.

## Speaking drills for mock interviews

Use these prompts in 60-second, 2-minute, and 5-minute formats.

1. Explain why Swift favors value semantics for many models.
2. Explain why `String.Index` exists and what problem it prevents.
3. Explain the difference between `weak` and `unowned` like you would in a panel interview.
4. Explain why protocol extension methods can surprise engineers.
5. Explain `some` vs `any` without using hand-wavy language.
6. Explain when `Result` is better than `throws`.
7. Explain why enums are stronger than loose booleans for app state.
8. Explain how property wrappers help without overusing them.
9. Explain why copy-on-write does not break value semantics.
10. Explain what modern Swift is trying to improve in ownership and concurrency.
11. Explain why `guard` is more than style preference.
12. Explain why access control is an API design tool.
13. Explain what interviewers mean when they ask for a mental model.
14. Explain how to turn a syntax answer into a senior answer.
15. Explain how to recover when you forget exact syntax in a live interview.
16. Explain why correctness and tradeoffs matter more than trivia recall.
17. Explain the difference between a good abstraction and premature abstraction.
18. Explain how to decide between a struct and a class in app architecture.
19. Explain how to reason about reference graphs during leak debugging.
20. Explain how to separate transport models from domain models during decoding.

## What strong answers sound like

- They define the concept in one sentence.
- They give the interviewer a clear mental model.
- They mention at least one runtime or architectural consequence.
- They include one real app example.
- They acknowledge tradeoffs instead of pretending every tool is universally correct.
- They use precise Swift terminology.
- They avoid cargo-cult advice.
- They do not confuse abstraction mechanism with implementation detail.
- They distinguish semantics from optimization.
- They are concise before they are clever.
- They treat memory management as ownership, not superstition.
- They connect language features to maintainability.
- They show awareness of modern Swift evolution.
- They demonstrate judgment, not just recall.

## Final-week review flow for this module

- Day 1: Re-read files 01-03 and answer five optional-related questions aloud.
- Day 2: Re-read files 04-06 and whiteboard one API-design example.
- Day 3: Re-read files 07-09 and draw one retain cycle and one dispatch example.
- Day 4: Re-read files 10-12 and explain one decoding strategy and one error boundary.
- Day 5: Re-read files 13-14 and run rapid-fire questions under a timer.
- Day 6: Do a full mock Swift fundamentals interview.
- Day 7: Review only misses, hesitations, and unclear explanations.

## Senior signal checklist

- I can define value semantics without saying only “it copies.”
- I can explain optionals as modeling absence, not as annoying syntax.
- I can describe ARC using reference graphs.
- I can explain protocol extension dispatch without guessing.
- I can compare `some`, `any`, and type erasure clearly.
- I can explain modern Swift direction-of-travel honestly.
- I can connect language features to app architecture decisions.
- I can answer follow-up questions without contradicting my first answer.
- I can give at least one concrete production example per topic.
- I can describe tradeoffs instead of pretending there is one perfect pattern.

## Module review prompts by theme
### Language and control flow
- Explain why Swift requires Boolean conditions.
- Compare `switch` pattern matching with if/else chains.
- Describe when a tuple should become a struct.
- Explain why ranges matter for safe iteration and slicing.
- Describe a bug that early-exit `guard` can prevent.
- Explain labeled loops without sounding like they are a primary design tool.
- Give a clean example of using `where` in pattern matching.
- Explain why readability matters more than clever branching.

### Collections and optionals
- Explain why `String.Index` exists.
- Compare `Array`, `Dictionary`, and `Set` with complexity expectations.
- Explain copy-on-write using `Array`.
- Describe when nil coalescing helps and when it hides a bug.
- Explain optional chaining and when long chains become suspicious.
- Compare `if let` and `guard let` with a practical example.
- Explain why nested optionals are a modeling smell.
- Justify one acceptable use of force unwrap in production.

### Abstraction and type system
- Explain the difference between a protocol and a base class.
- Describe when protocol composition is stronger than a large umbrella protocol.
- Explain the protocol extension dispatch trap with a concrete example.
- Compare generic constraints and associated types.
- Explain why `any` has runtime cost compared with preserving a concrete type.
- Describe when opaque types (`some`) improve API design.
- Explain why type erasure exists in frameworks like Combine.
- Summarize the mental model for value semantics versus reference semantics.

### Memory, errors, and boundaries
- Draw a retain cycle involving a stored closure.
- Explain `weak` vs `unowned` with lifetime guarantees.
- Describe what belongs in `deinit` and what does not.
- Explain when a function should throw instead of returning an optional.
- Compare `throws` and `Result` in callback-oriented APIs.
- Explain why DTO mapping helps with Codable-heavy codebases.
- Describe how access control supports API evolution.
- Explain the runtime difference between static dispatch, vtables, and witness tables.

### Modern Swift and interview polish
- Explain parameter packs conceptually.
- Describe Swift macros in a safe and accurate way.
- Explain why ownership modifiers are appearing in modern Swift.
- Summarize the impact of Swift 6 concurrency tightening.
- Describe typed throws at an interview-awareness level.
- Explain how to turn a syntax answer into a senior answer.
- Describe how to recover when you forget exact syntax in a live round.
- Explain why modern Swift knowledge matters even on older production codebases.

## Self-assessment rubric
- I can answer a rapid-fire Swift question in 30 seconds without rambling.
- I can expand the same answer to two minutes with examples and tradeoffs.
- I can explain one performance consequence for each major collection.
- I can explain one ownership consequence for each ARC keyword.
- I can name one modeling benefit of enums over multiple booleans.
- I can explain why protocols and generics solve different kinds of abstraction problems.
- I can distinguish API ergonomics from runtime cost.
- I can explain at least one dispatch rule accurately.
- I can explain modern Swift features honestly without bluffing production experience.
- I can turn a code snippet into an architecture discussion when asked a follow-up.
- I can identify when a question is really testing semantics rather than syntax.
- I can speak clearly about boundaries: network boundary, domain boundary, ownership boundary, module boundary.
- I can explain why invalid-state prevention is a recurring Swift theme.
- I can identify when a simple concrete type is better than abstraction.
- I can identify when abstraction is necessary for reuse or testability.
- I can explain why strong interview answers include tradeoffs.
- I can explain why Swift strings are designed the way they are.
- I can explain when COW matters and when it usually does not.
- I can compare `some`, `any`, and type erasure without contradiction.
- I can answer a retain-cycle question using a diagram in words.

## Practice cadence template
- Session 1: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 2: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 3: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 4: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 5: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 6: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 7: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 8: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 9: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 10: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 11: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 12: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 13: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.
- Session 14: read one lesson, extract five bullets, answer two follow-up questions aloud, and capture one lingering weak spot for the next review.

## Module-specific mock interview loop
### Mock prompt 1
- **Prompt:** Explain value semantics and reference semantics with one app-layer example each.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 2
- **Prompt:** Explain why Swift strings are Unicode-correct and why that changes indexing.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 3
- **Prompt:** Explain optionals as a modeling tool, not just a syntax feature.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 4
- **Prompt:** Explain how closure capture can create a retain cycle.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 5
- **Prompt:** Explain when a struct should become a class.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 6
- **Prompt:** Explain designated vs convenience initialization.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 7
- **Prompt:** Explain why protocol composition can be better than inheritance.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 8
- **Prompt:** Explain why associated types make protocols more expressive.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 9
- **Prompt:** Explain weak vs unowned references with lifetime guarantees.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 10
- **Prompt:** Explain when `Result` is preferable to `throws`.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 11
- **Prompt:** Explain the difference between `public` and `open`.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 12
- **Prompt:** Explain a real-world Codable customization example.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 13
- **Prompt:** Explain one modern Swift feature you would mention in a current interview.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 14
- **Prompt:** Explain how you would use the drills file the night before an interview.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 15
- **Prompt:** Explain one common mistake experienced iOS engineers make after time away from Swift.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 16
- **Prompt:** Explain how to answer follow-up questions when the interviewer pushes deeper on semantics.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 17
- **Prompt:** Explain how to connect a language-level answer to architecture-level reasoning.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 18
- **Prompt:** Explain which parts of this module are most likely to appear in a senior screen.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 19
- **Prompt:** Explain which parts are most likely to appear in a live coding round.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

### Mock prompt 20
- **Prompt:** Explain how to recover if you realize mid-answer that you started with the wrong abstraction.
- **What a strong candidate does:** define the rule, give a mental model, show one production consequence, and close with a tradeoff or pitfall.

## Common returner mistakes after a 1-2 year gap
- Remembering syntax patterns but not modern concurrency terminology.
- Speaking confidently about older habits that newer Swift style now avoids.
- Over-indexing on framework trivia while under-explaining language semantics.
- Using stale opinions on weak/unowned, optionals, or protocol-oriented programming without nuance.
- Forgetting how much interviewers now care about ownership, Sendable, and abstraction cost.
- Answering with implementation detail before giving the mental model.
- Giving absolute rules instead of tradeoff-based recommendations.
- Skipping mock speaking practice because the material feels familiar on paper.
- Underestimating how much clarity and calm communication matter in senior loops.
- Neglecting system-design framing even during language-level questions.

## Final module outcomes
- You can explain Swift fundamentals with current terminology.
- You can connect language features to real iOS production tradeoffs.
- You can answer follow-up questions on memory, generics, and dispatch without guessing.
- You can explain why modern Swift has evolved in the directions it has.
- You can move from short answers to deeper architectural answers smoothly.
- You can identify where your remaining weak spots are before live interviews.
- You can use Module 14 as a compressed revision pass.
- You can sound like an engineer returning sharp—not rusty.

## One-hour final review plan
- 10 minutes: scan language basics, strings, and optionals.
- 10 minutes: scan functions, closures, structs, classes, and enums.
- 10 minutes: scan properties, initialization, and protocols.
- 10 minutes: scan generics, ARC, and error handling.
- 10 minutes: scan access control, Codable, and modern Swift.
- 10 minutes: answer rapid-fire questions from the drills file aloud.

- Final check: explain one tradeoff from each major lesson family.
- Final check: draw one ownership graph and one dispatch example from memory.
- Final check: summarize modern Swift in under 60 seconds.
- Final check: close with calm, concise answers rather than maximum detail.
