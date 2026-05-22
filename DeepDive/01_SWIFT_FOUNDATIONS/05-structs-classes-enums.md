# 05 — Structs, Classes, and Enums

This topic ties together modeling, identity, testing, memory management, and architecture.

## Value semantics vs reference semantics
Structs and enums are value types; classes are reference types.

### Mental model
- Value types reduce aliasing.
- Reference types provide shared identity.
- The real tradeoff is independent state versus shared mutable state.

### Example
```swift
struct Counter { var value = 0 }
final class CounterBox { var value = 0 }

var a = Counter()
var b = a
b.value = 10

let x = CounterBox()
let y = x
y.value = 10
```

### Common pitfalls
- ⚠️ Saying only 'structs copy, classes don't' without discussing aliasing.
- ⚠️ Choosing classes by habit.
- ⚠️ Ignoring architectural impact of shared mutable state.

### Interview questions
- Why do value semantics reduce bugs?
- What is aliasing?
- When is shared mutable identity desirable?

### Senior-level discussion
- Use value types for snapshots, DTOs, config, and state.
- Use classes when identity, lifecycle, or shared coordination really matters.
- Explain the choice in terms of correctness, not trend.

## Struct features and when to use them
Structs support methods, computed properties, protocol conformance, and initializers.

### Example
```swift
struct SearchQuery: Equatable, Hashable {
    var term: String
    var page: Int

    mutating func advance() {
        page += 1
    }
}
```

> 💡 **Tip:** Structs pair naturally with testing, diffing, and concurrent data flow.

### Common pitfalls
- ⚠️ Forgetting `mutating`.
- ⚠️ Hiding reference-typed shared state inside a struct without acknowledging it.
- ⚠️ Using structs where identity actually matters.

## Class features, inheritance, and identity
Classes support inheritance, `deinit`, ARC, and identity checks.

### Example
```swift
class ViewModel {
    func load() {}
}

class ProfileViewModel: ViewModel {
    override func load() {
        print("Load profile")
    }
}

let first = ProfileViewModel()
let second = first
print(first === second)
```

### Common pitfalls
- ⚠️ Confusing `===` with `==`.
- ⚠️ Overusing inheritance.
- ⚠️ Building deep hierarchies that become brittle.

### Interview questions
- What does `===` test?
- When is inheritance appropriate?
- Why can class hierarchies become fragile?

### Senior-level discussion
- Keep inheritance shallow unless an override-oriented design is genuinely needed.
- Favor composition and protocols for many variation points.
- Use classes for controllers, managers, caches, and lifecycle-driven objects.

## Copy-on-write
Swift standard collections show that value semantics and performance can coexist.

### Mental model
- COW is an implementation optimization.
- Semantics remain value-based.
- Mutation is the key moment to reason about copying.

### Example
```swift
var original = [1, 2, 3]
var copy = original
copy.append(4)

print(original)
print(copy)
```

### Interview questions
- Why is copy-on-write important?
- How does it preserve value semantics?
- Can custom types implement COW?

## Enums for state modeling
Swift enums are full modeling tools.

### Example
```swift
enum LoadState {
    case idle
    case loading
    case loaded(items: [String])
    case failed(message: String)
}
```

### Common pitfalls
- ⚠️ Using multiple booleans instead of a single enum state.
- ⚠️ Adding `default` and losing exhaustiveness.
- ⚠️ Treating enums as constant bags instead of domain models.

> 🎯 **Interview Answer:** Enums are great because they make invalid combinations harder to represent and force exhaustive handling.

## Associated values, raw values, methods, and indirect enums
Associated values store per-instance data; raw values provide a simple backing representation.

### Example
```swift
enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
}

indirect enum FileNode {
    case file(name: String)
    case folder(name: String, children: [FileNode])
}
```

### Quick recall
- Raw values are fixed representatives.
- Associated values vary per case instance.
- `indirect` enables recursion.
- Enums can have methods and computed properties.

## Topic-specific oral drills
Use these prompts to turn **structs classes enums** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — value semantics
- **Definition prompt:** Explain what **value semantics** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **value semantics**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **value semantics**, not just the spelling.
- **Production example to mention:** Connect **value semantics** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **value semantics** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **value semantics**.
- **One-line close:** End with a concise summary that tells the interviewer why **value semantics** matters in production code.

### Follow-up 1 — value semantics
- **Compare prompt:** Contrast **value semantics** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **value semantics** is misunderstood.
- **Code review angle:** Describe what misuse of **value semantics** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **value semantics** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **value semantics**.

### Drill 2 — reference semantics
- **Definition prompt:** Explain what **reference semantics** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **reference semantics**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **reference semantics**, not just the spelling.
- **Production example to mention:** Connect **reference semantics** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **reference semantics** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **reference semantics**.
- **One-line close:** End with a concise summary that tells the interviewer why **reference semantics** matters in production code.

### Follow-up 2 — reference semantics
- **Compare prompt:** Contrast **reference semantics** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **reference semantics** is misunderstood.
- **Code review angle:** Describe what misuse of **reference semantics** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **reference semantics** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **reference semantics**.

### Drill 3 — aliasing
- **Definition prompt:** Explain what **aliasing** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **aliasing**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **aliasing**, not just the spelling.
- **Production example to mention:** Connect **aliasing** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **aliasing** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **aliasing**.
- **One-line close:** End with a concise summary that tells the interviewer why **aliasing** matters in production code.

### Follow-up 3 — aliasing
- **Compare prompt:** Contrast **aliasing** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **aliasing** is misunderstood.
- **Code review angle:** Describe what misuse of **aliasing** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **aliasing** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **aliasing**.

### Drill 4 — identity and `===`
- **Definition prompt:** Explain what **identity and `===`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **identity and `===`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **identity and `===`**, not just the spelling.
- **Production example to mention:** Connect **identity and `===`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **identity and `===`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **identity and `===`**.
- **One-line close:** End with a concise summary that tells the interviewer why **identity and `===`** matters in production code.

### Follow-up 4 — identity and `===`
- **Compare prompt:** Contrast **identity and `===`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **identity and `===`** is misunderstood.
- **Code review angle:** Describe what misuse of **identity and `===`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **identity and `===`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **identity and `===`**.

### Drill 5 — struct mutation
- **Definition prompt:** Explain what **struct mutation** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **struct mutation**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **struct mutation**, not just the spelling.
- **Production example to mention:** Connect **struct mutation** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **struct mutation** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **struct mutation**.
- **One-line close:** End with a concise summary that tells the interviewer why **struct mutation** matters in production code.

### Follow-up 5 — struct mutation
- **Compare prompt:** Contrast **struct mutation** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **struct mutation** is misunderstood.
- **Code review angle:** Describe what misuse of **struct mutation** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **struct mutation** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **struct mutation**.

### Drill 6 — class lifecycle
- **Definition prompt:** Explain what **class lifecycle** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **class lifecycle**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **class lifecycle**, not just the spelling.
- **Production example to mention:** Connect **class lifecycle** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **class lifecycle** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **class lifecycle**.
- **One-line close:** End with a concise summary that tells the interviewer why **class lifecycle** matters in production code.

### Follow-up 6 — class lifecycle
- **Compare prompt:** Contrast **class lifecycle** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **class lifecycle** is misunderstood.
- **Code review angle:** Describe what misuse of **class lifecycle** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **class lifecycle** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **class lifecycle**.

### Drill 7 — inheritance tradeoffs
- **Definition prompt:** Explain what **inheritance tradeoffs** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **inheritance tradeoffs**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **inheritance tradeoffs**, not just the spelling.
- **Production example to mention:** Connect **inheritance tradeoffs** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **inheritance tradeoffs** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **inheritance tradeoffs**.
- **One-line close:** End with a concise summary that tells the interviewer why **inheritance tradeoffs** matters in production code.

### Follow-up 7 — inheritance tradeoffs
- **Compare prompt:** Contrast **inheritance tradeoffs** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **inheritance tradeoffs** is misunderstood.
- **Code review angle:** Describe what misuse of **inheritance tradeoffs** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **inheritance tradeoffs** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **inheritance tradeoffs**.

### Drill 8 — composition over inheritance
- **Definition prompt:** Explain what **composition over inheritance** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **composition over inheritance**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **composition over inheritance**, not just the spelling.
- **Production example to mention:** Connect **composition over inheritance** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **composition over inheritance** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **composition over inheritance**.
- **One-line close:** End with a concise summary that tells the interviewer why **composition over inheritance** matters in production code.

### Follow-up 8 — composition over inheritance
- **Compare prompt:** Contrast **composition over inheritance** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **composition over inheritance** is misunderstood.
- **Code review angle:** Describe what misuse of **composition over inheritance** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **composition over inheritance** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **composition over inheritance**.

### Drill 9 — copy-on-write
- **Definition prompt:** Explain what **copy-on-write** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **copy-on-write**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **copy-on-write**, not just the spelling.
- **Production example to mention:** Connect **copy-on-write** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **copy-on-write** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **copy-on-write**.
- **One-line close:** End with a concise summary that tells the interviewer why **copy-on-write** matters in production code.

### Follow-up 9 — copy-on-write
- **Compare prompt:** Contrast **copy-on-write** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **copy-on-write** is misunderstood.
- **Code review angle:** Describe what misuse of **copy-on-write** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **copy-on-write** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **copy-on-write**.

### Drill 10 — structs with reference-typed fields
- **Definition prompt:** Explain what **structs with reference-typed fields** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **structs with reference-typed fields**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **structs with reference-typed fields**, not just the spelling.
- **Production example to mention:** Connect **structs with reference-typed fields** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **structs with reference-typed fields** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **structs with reference-typed fields**.
- **One-line close:** End with a concise summary that tells the interviewer why **structs with reference-typed fields** matters in production code.

### Follow-up 10 — structs with reference-typed fields
- **Compare prompt:** Contrast **structs with reference-typed fields** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **structs with reference-typed fields** is misunderstood.
- **Code review angle:** Describe what misuse of **structs with reference-typed fields** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **structs with reference-typed fields** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **structs with reference-typed fields**.

### Drill 11 — when to choose a struct
- **Definition prompt:** Explain what **when to choose a struct** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **when to choose a struct**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **when to choose a struct**, not just the spelling.
- **Production example to mention:** Connect **when to choose a struct** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **when to choose a struct** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **when to choose a struct**.
- **One-line close:** End with a concise summary that tells the interviewer why **when to choose a struct** matters in production code.

### Follow-up 11 — when to choose a struct
- **Compare prompt:** Contrast **when to choose a struct** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **when to choose a struct** is misunderstood.
- **Code review angle:** Describe what misuse of **when to choose a struct** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **when to choose a struct** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **when to choose a struct**.

### Drill 12 — when to choose a class
- **Definition prompt:** Explain what **when to choose a class** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **when to choose a class**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **when to choose a class**, not just the spelling.
- **Production example to mention:** Connect **when to choose a class** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **when to choose a class** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **when to choose a class**.
- **One-line close:** End with a concise summary that tells the interviewer why **when to choose a class** matters in production code.

### Follow-up 12 — when to choose a class
- **Compare prompt:** Contrast **when to choose a class** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **when to choose a class** is misunderstood.
- **Code review angle:** Describe what misuse of **when to choose a class** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **when to choose a class** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **when to choose a class**.

### Drill 13 — enum state modeling
- **Definition prompt:** Explain what **enum state modeling** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **enum state modeling**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **enum state modeling**, not just the spelling.
- **Production example to mention:** Connect **enum state modeling** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **enum state modeling** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **enum state modeling**.
- **One-line close:** End with a concise summary that tells the interviewer why **enum state modeling** matters in production code.

### Follow-up 13 — enum state modeling
- **Compare prompt:** Contrast **enum state modeling** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **enum state modeling** is misunderstood.
- **Code review angle:** Describe what misuse of **enum state modeling** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **enum state modeling** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **enum state modeling**.

### Drill 14 — associated values
- **Definition prompt:** Explain what **associated values** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **associated values**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **associated values**, not just the spelling.
- **Production example to mention:** Connect **associated values** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **associated values** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **associated values**.
- **One-line close:** End with a concise summary that tells the interviewer why **associated values** matters in production code.

### Follow-up 14 — associated values
- **Compare prompt:** Contrast **associated values** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **associated values** is misunderstood.
- **Code review angle:** Describe what misuse of **associated values** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **associated values** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **associated values**.

### Drill 15 — raw values
- **Definition prompt:** Explain what **raw values** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **raw values**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **raw values**, not just the spelling.
- **Production example to mention:** Connect **raw values** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **raw values** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **raw values**.
- **One-line close:** End with a concise summary that tells the interviewer why **raw values** matters in production code.

### Follow-up 15 — raw values
- **Compare prompt:** Contrast **raw values** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **raw values** is misunderstood.
- **Code review angle:** Describe what misuse of **raw values** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **raw values** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **raw values**.

### Drill 16 — enum methods
- **Definition prompt:** Explain what **enum methods** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **enum methods**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **enum methods**, not just the spelling.
- **Production example to mention:** Connect **enum methods** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **enum methods** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **enum methods**.
- **One-line close:** End with a concise summary that tells the interviewer why **enum methods** matters in production code.

### Follow-up 16 — enum methods
- **Compare prompt:** Contrast **enum methods** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **enum methods** is misunderstood.
- **Code review angle:** Describe what misuse of **enum methods** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **enum methods** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **enum methods**.

### Drill 17 — exhaustive switching
- **Definition prompt:** Explain what **exhaustive switching** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **exhaustive switching**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **exhaustive switching**, not just the spelling.
- **Production example to mention:** Connect **exhaustive switching** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **exhaustive switching** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **exhaustive switching**.
- **One-line close:** End with a concise summary that tells the interviewer why **exhaustive switching** matters in production code.

### Follow-up 17 — exhaustive switching
- **Compare prompt:** Contrast **exhaustive switching** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **exhaustive switching** is misunderstood.
- **Code review angle:** Describe what misuse of **exhaustive switching** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **exhaustive switching** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **exhaustive switching**.

### Drill 18 — invalid state prevention
- **Definition prompt:** Explain what **invalid state prevention** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **invalid state prevention**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **invalid state prevention**, not just the spelling.
- **Production example to mention:** Connect **invalid state prevention** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **invalid state prevention** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **invalid state prevention**.
- **One-line close:** End with a concise summary that tells the interviewer why **invalid state prevention** matters in production code.

### Follow-up 18 — invalid state prevention
- **Compare prompt:** Contrast **invalid state prevention** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **invalid state prevention** is misunderstood.
- **Code review angle:** Describe what misuse of **invalid state prevention** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **invalid state prevention** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **invalid state prevention**.

### Drill 19 — recursive enums
- **Definition prompt:** Explain what **recursive enums** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **recursive enums**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **recursive enums**, not just the spelling.
- **Production example to mention:** Connect **recursive enums** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **recursive enums** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **recursive enums**.
- **One-line close:** End with a concise summary that tells the interviewer why **recursive enums** matters in production code.

### Follow-up 19 — recursive enums
- **Compare prompt:** Contrast **recursive enums** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **recursive enums** is misunderstood.
- **Code review angle:** Describe what misuse of **recursive enums** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **recursive enums** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **recursive enums**.

### Drill 20 — `indirect` enums
- **Definition prompt:** Explain what **`indirect` enums** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`indirect` enums**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`indirect` enums**, not just the spelling.
- **Production example to mention:** Connect **`indirect` enums** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`indirect` enums** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`indirect` enums**.
- **One-line close:** End with a concise summary that tells the interviewer why **`indirect` enums** matters in production code.

### Follow-up 20 — `indirect` enums
- **Compare prompt:** Contrast **`indirect` enums** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`indirect` enums** is misunderstood.
- **Code review angle:** Describe what misuse of **`indirect` enums** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`indirect` enums** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`indirect` enums**.

### Drill 21 — testing value vs reference models
- **Definition prompt:** Explain what **testing value vs reference models** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **testing value vs reference models**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **testing value vs reference models**, not just the spelling.
- **Production example to mention:** Connect **testing value vs reference models** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **testing value vs reference models** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **testing value vs reference models**.
- **One-line close:** End with a concise summary that tells the interviewer why **testing value vs reference models** matters in production code.

### Follow-up 21 — testing value vs reference models
- **Compare prompt:** Contrast **testing value vs reference models** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **testing value vs reference models** is misunderstood.
- **Code review angle:** Describe what misuse of **testing value vs reference models** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **testing value vs reference models** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **testing value vs reference models**.

### Drill 22 — architecture consequences of type choice
- **Definition prompt:** Explain what **architecture consequences of type choice** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **architecture consequences of type choice**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **architecture consequences of type choice**, not just the spelling.
- **Production example to mention:** Connect **architecture consequences of type choice** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **architecture consequences of type choice** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **architecture consequences of type choice**.
- **One-line close:** End with a concise summary that tells the interviewer why **architecture consequences of type choice** matters in production code.

### Follow-up 22 — architecture consequences of type choice
- **Compare prompt:** Contrast **architecture consequences of type choice** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **architecture consequences of type choice** is misunderstood.
- **Code review angle:** Describe what misuse of **architecture consequences of type choice** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **architecture consequences of type choice** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **architecture consequences of type choice**.

