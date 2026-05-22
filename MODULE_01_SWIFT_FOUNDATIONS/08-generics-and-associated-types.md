# 08 — Generics and Associated Types

Generics are where many mid-level Swift conversations turn senior-level.

## Generic functions and types
Generics let you write reusable abstractions while preserving type information.

### Example
```swift
func swapValues<T>(_ lhs: inout T, _ rhs: inout T) {
    let temp = lhs
    lhs = rhs
    rhs = temp
}

struct Stack<Element> {
    private var storage: [Element] = []
    mutating func push(_ value: Element) { storage.append(value) }
    mutating func pop() -> Element? { storage.popLast() }
}
```

### Interview questions
- Why are generics better than `Any` here?
- What does a generic parameter represent?
- Where do generic types appear in the standard library?

## Constraints and `where` clauses
Constraints express capability requirements precisely.

### Example
```swift
func uniqueElements<S: Sequence>(_ sequence: S) -> [S.Element]
where S.Element: Hashable {
    var seen = Set<S.Element>()
    return sequence.filter { seen.insert($0).inserted }
}
```

> 💡 **Tip:** Constraints are about semantic capability, not type-system decoration.

## Associated types
Associated types let a protocol describe related placeholder types chosen by each conformer.

### Example
```swift
protocol Repository {
    associatedtype Entity
    func loadAll() async throws -> [Entity]
    func save(_ entity: Entity) async throws
}
```

### Common pitfalls
- ⚠️ Explaining associated types without naming the problem they solve.
- ⚠️ Ignoring type relationships within the protocol.
- ⚠️ Getting stuck on syntax rather than abstraction.

## Existentials (`any`)
An existential stores some conforming value behind a protocol-shaped interface.

### Example
```swift
protocol Logger {
    func log(_ message: String)
}

struct ConsoleLogger: Logger {
    func log(_ message: String) { print(message) }
}

let logger: any Logger = ConsoleLogger()
logger.log("Started")
```

### Senior-level discussion
- Existentials are great for heterogeneity and abstraction boundaries.
- They may introduce boxing and reduce optimization opportunities.
- Use them when ergonomics or API boundaries matter more than preserving concrete type information.

## Opaque types (`some`) and type erasure
`some` hides one concrete type; type erasure hides arbitrary concrete types behind a wrapper or existential.

### Example
```swift
protocol Screen {
    func render() -> String
}

struct HomeScreen: Screen {
    func render() -> String { "home" }
}

func makeScreen() -> some Screen {
    HomeScreen()
}
```

### Production example
- `AnyPublisher` exists because concrete Combine publisher chains are complex and protocols with associated types are hard to store directly.

> 🎯 **Interview Answer:** `some` preserves one hidden concrete type chosen by the implementation; `any` boxes an unknown conforming value; type erasure is the practical tool when protocols alone are not enough.

## Topic-specific oral drills
Use these prompts to turn **generics and associated types** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — generic placeholders
- **Definition prompt:** Explain what **generic placeholders** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **generic placeholders**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **generic placeholders**, not just the spelling.
- **Production example to mention:** Connect **generic placeholders** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **generic placeholders** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **generic placeholders**.
- **One-line close:** End with a concise summary that tells the interviewer why **generic placeholders** matters in production code.

### Follow-up 1 — generic placeholders
- **Compare prompt:** Contrast **generic placeholders** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **generic placeholders** is misunderstood.
- **Code review angle:** Describe what misuse of **generic placeholders** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **generic placeholders** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **generic placeholders**.

### Drill 2 — generic functions
- **Definition prompt:** Explain what **generic functions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **generic functions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **generic functions**, not just the spelling.
- **Production example to mention:** Connect **generic functions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **generic functions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **generic functions**.
- **One-line close:** End with a concise summary that tells the interviewer why **generic functions** matters in production code.

### Follow-up 2 — generic functions
- **Compare prompt:** Contrast **generic functions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **generic functions** is misunderstood.
- **Code review angle:** Describe what misuse of **generic functions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **generic functions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **generic functions**.

### Drill 3 — generic types
- **Definition prompt:** Explain what **generic types** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **generic types**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **generic types**, not just the spelling.
- **Production example to mention:** Connect **generic types** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **generic types** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **generic types**.
- **One-line close:** End with a concise summary that tells the interviewer why **generic types** matters in production code.

### Follow-up 3 — generic types
- **Compare prompt:** Contrast **generic types** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **generic types** is misunderstood.
- **Code review angle:** Describe what misuse of **generic types** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **generic types** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **generic types**.

### Drill 4 — generic constraints
- **Definition prompt:** Explain what **generic constraints** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **generic constraints**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **generic constraints**, not just the spelling.
- **Production example to mention:** Connect **generic constraints** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **generic constraints** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **generic constraints**.
- **One-line close:** End with a concise summary that tells the interviewer why **generic constraints** matters in production code.

### Follow-up 4 — generic constraints
- **Compare prompt:** Contrast **generic constraints** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **generic constraints** is misunderstood.
- **Code review angle:** Describe what misuse of **generic constraints** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **generic constraints** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **generic constraints**.

### Drill 5 — `where` clauses
- **Definition prompt:** Explain what **`where` clauses** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`where` clauses**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`where` clauses**, not just the spelling.
- **Production example to mention:** Connect **`where` clauses** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`where` clauses** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`where` clauses**.
- **One-line close:** End with a concise summary that tells the interviewer why **`where` clauses** matters in production code.

### Follow-up 5 — `where` clauses
- **Compare prompt:** Contrast **`where` clauses** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`where` clauses** is misunderstood.
- **Code review angle:** Describe what misuse of **`where` clauses** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`where` clauses** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`where` clauses**.

### Drill 6 — specialization awareness
- **Definition prompt:** Explain what **specialization awareness** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **specialization awareness**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **specialization awareness**, not just the spelling.
- **Production example to mention:** Connect **specialization awareness** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **specialization awareness** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **specialization awareness**.
- **One-line close:** End with a concise summary that tells the interviewer why **specialization awareness** matters in production code.

### Follow-up 6 — specialization awareness
- **Compare prompt:** Contrast **specialization awareness** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **specialization awareness** is misunderstood.
- **Code review angle:** Describe what misuse of **specialization awareness** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **specialization awareness** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **specialization awareness**.

### Drill 7 — preserving type relationships
- **Definition prompt:** Explain what **preserving type relationships** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **preserving type relationships**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **preserving type relationships**, not just the spelling.
- **Production example to mention:** Connect **preserving type relationships** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **preserving type relationships** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **preserving type relationships**.
- **One-line close:** End with a concise summary that tells the interviewer why **preserving type relationships** matters in production code.

### Follow-up 7 — preserving type relationships
- **Compare prompt:** Contrast **preserving type relationships** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **preserving type relationships** is misunderstood.
- **Code review angle:** Describe what misuse of **preserving type relationships** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **preserving type relationships** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **preserving type relationships**.

### Drill 8 — `Any` vs generics
- **Definition prompt:** Explain what **`Any` vs generics** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`Any` vs generics**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`Any` vs generics**, not just the spelling.
- **Production example to mention:** Connect **`Any` vs generics** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`Any` vs generics** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`Any` vs generics**.
- **One-line close:** End with a concise summary that tells the interviewer why **`Any` vs generics** matters in production code.

### Follow-up 8 — `Any` vs generics
- **Compare prompt:** Contrast **`Any` vs generics** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`Any` vs generics** is misunderstood.
- **Code review angle:** Describe what misuse of **`Any` vs generics** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`Any` vs generics** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`Any` vs generics**.

### Drill 9 — associated types
- **Definition prompt:** Explain what **associated types** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **associated types**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **associated types**, not just the spelling.
- **Production example to mention:** Connect **associated types** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **associated types** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **associated types**.
- **One-line close:** End with a concise summary that tells the interviewer why **associated types** matters in production code.

### Follow-up 9 — associated types
- **Compare prompt:** Contrast **associated types** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **associated types** is misunderstood.
- **Code review angle:** Describe what misuse of **associated types** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **associated types** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **associated types**.

### Drill 10 — why associated types exist
- **Definition prompt:** Explain what **why associated types exist** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **why associated types exist**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **why associated types exist**, not just the spelling.
- **Production example to mention:** Connect **why associated types exist** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **why associated types exist** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **why associated types exist**.
- **One-line close:** End with a concise summary that tells the interviewer why **why associated types exist** matters in production code.

### Follow-up 10 — why associated types exist
- **Compare prompt:** Contrast **why associated types exist** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **why associated types exist** is misunderstood.
- **Code review angle:** Describe what misuse of **why associated types exist** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **why associated types exist** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **why associated types exist**.

### Drill 11 — protocols with associated types
- **Definition prompt:** Explain what **protocols with associated types** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocols with associated types**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocols with associated types**, not just the spelling.
- **Production example to mention:** Connect **protocols with associated types** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocols with associated types** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocols with associated types**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocols with associated types** matters in production code.

### Follow-up 11 — protocols with associated types
- **Compare prompt:** Contrast **protocols with associated types** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocols with associated types** is misunderstood.
- **Code review angle:** Describe what misuse of **protocols with associated types** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocols with associated types** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocols with associated types**.

### Drill 12 — existentials with `any`
- **Definition prompt:** Explain what **existentials with `any`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **existentials with `any`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **existentials with `any`**, not just the spelling.
- **Production example to mention:** Connect **existentials with `any`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **existentials with `any`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **existentials with `any`**.
- **One-line close:** End with a concise summary that tells the interviewer why **existentials with `any`** matters in production code.

### Follow-up 12 — existentials with `any`
- **Compare prompt:** Contrast **existentials with `any`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **existentials with `any`** is misunderstood.
- **Code review angle:** Describe what misuse of **existentials with `any`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **existentials with `any`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **existentials with `any`**.

### Drill 13 — existential boxing
- **Definition prompt:** Explain what **existential boxing** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **existential boxing**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **existential boxing**, not just the spelling.
- **Production example to mention:** Connect **existential boxing** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **existential boxing** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **existential boxing**.
- **One-line close:** End with a concise summary that tells the interviewer why **existential boxing** matters in production code.

### Follow-up 13 — existential boxing
- **Compare prompt:** Contrast **existential boxing** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **existential boxing** is misunderstood.
- **Code review angle:** Describe what misuse of **existential boxing** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **existential boxing** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **existential boxing**.

### Drill 14 — runtime cost of existentials
- **Definition prompt:** Explain what **runtime cost of existentials** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **runtime cost of existentials**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **runtime cost of existentials**, not just the spelling.
- **Production example to mention:** Connect **runtime cost of existentials** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **runtime cost of existentials** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **runtime cost of existentials**.
- **One-line close:** End with a concise summary that tells the interviewer why **runtime cost of existentials** matters in production code.

### Follow-up 14 — runtime cost of existentials
- **Compare prompt:** Contrast **runtime cost of existentials** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **runtime cost of existentials** is misunderstood.
- **Code review angle:** Describe what misuse of **runtime cost of existentials** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **runtime cost of existentials** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **runtime cost of existentials**.

### Drill 15 — opaque types with `some`
- **Definition prompt:** Explain what **opaque types with `some`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **opaque types with `some`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **opaque types with `some`**, not just the spelling.
- **Production example to mention:** Connect **opaque types with `some`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **opaque types with `some`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **opaque types with `some`**.
- **One-line close:** End with a concise summary that tells the interviewer why **opaque types with `some`** matters in production code.

### Follow-up 15 — opaque types with `some`
- **Compare prompt:** Contrast **opaque types with `some`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **opaque types with `some`** is misunderstood.
- **Code review angle:** Describe what misuse of **opaque types with `some`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **opaque types with `some`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **opaque types with `some`**.

### Drill 16 — `some` vs `any`
- **Definition prompt:** Explain what **`some` vs `any`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`some` vs `any`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`some` vs `any`**, not just the spelling.
- **Production example to mention:** Connect **`some` vs `any`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`some` vs `any`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`some` vs `any`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`some` vs `any`** matters in production code.

### Follow-up 16 — `some` vs `any`
- **Compare prompt:** Contrast **`some` vs `any`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`some` vs `any`** is misunderstood.
- **Code review angle:** Describe what misuse of **`some` vs `any`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`some` vs `any`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`some` vs `any`**.

### Drill 17 — type erasure
- **Definition prompt:** Explain what **type erasure** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **type erasure**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **type erasure**, not just the spelling.
- **Production example to mention:** Connect **type erasure** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **type erasure** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **type erasure**.
- **One-line close:** End with a concise summary that tells the interviewer why **type erasure** matters in production code.

### Follow-up 17 — type erasure
- **Compare prompt:** Contrast **type erasure** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **type erasure** is misunderstood.
- **Code review angle:** Describe what misuse of **type erasure** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **type erasure** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **type erasure**.

### Drill 18 — `AnyPublisher`
- **Definition prompt:** Explain what **`AnyPublisher`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`AnyPublisher`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`AnyPublisher`**, not just the spelling.
- **Production example to mention:** Connect **`AnyPublisher`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`AnyPublisher`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`AnyPublisher`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`AnyPublisher`** matters in production code.

### Follow-up 18 — `AnyPublisher`
- **Compare prompt:** Contrast **`AnyPublisher`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`AnyPublisher`** is misunderstood.
- **Code review angle:** Describe what misuse of **`AnyPublisher`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`AnyPublisher`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`AnyPublisher`**.

### Drill 19 — storing heterogeneous conformers
- **Definition prompt:** Explain what **storing heterogeneous conformers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **storing heterogeneous conformers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **storing heterogeneous conformers**, not just the spelling.
- **Production example to mention:** Connect **storing heterogeneous conformers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **storing heterogeneous conformers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **storing heterogeneous conformers**.
- **One-line close:** End with a concise summary that tells the interviewer why **storing heterogeneous conformers** matters in production code.

### Follow-up 19 — storing heterogeneous conformers
- **Compare prompt:** Contrast **storing heterogeneous conformers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **storing heterogeneous conformers** is misunderstood.
- **Code review angle:** Describe what misuse of **storing heterogeneous conformers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **storing heterogeneous conformers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **storing heterogeneous conformers**.

### Drill 20 — API boundary design
- **Definition prompt:** Explain what **API boundary design** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **API boundary design**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **API boundary design**, not just the spelling.
- **Production example to mention:** Connect **API boundary design** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **API boundary design** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **API boundary design**.
- **One-line close:** End with a concise summary that tells the interviewer why **API boundary design** matters in production code.

### Follow-up 20 — API boundary design
- **Compare prompt:** Contrast **API boundary design** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **API boundary design** is misunderstood.
- **Code review angle:** Describe what misuse of **API boundary design** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **API boundary design** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **API boundary design**.

### Drill 21 — performance tradeoffs of abstraction
- **Definition prompt:** Explain what **performance tradeoffs of abstraction** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **performance tradeoffs of abstraction**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **performance tradeoffs of abstraction**, not just the spelling.
- **Production example to mention:** Connect **performance tradeoffs of abstraction** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **performance tradeoffs of abstraction** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **performance tradeoffs of abstraction**.
- **One-line close:** End with a concise summary that tells the interviewer why **performance tradeoffs of abstraction** matters in production code.

### Follow-up 21 — performance tradeoffs of abstraction
- **Compare prompt:** Contrast **performance tradeoffs of abstraction** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **performance tradeoffs of abstraction** is misunderstood.
- **Code review angle:** Describe what misuse of **performance tradeoffs of abstraction** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **performance tradeoffs of abstraction** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **performance tradeoffs of abstraction**.

### Drill 22 — senior explanations of generics
- **Definition prompt:** Explain what **senior explanations of generics** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **senior explanations of generics**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **senior explanations of generics**, not just the spelling.
- **Production example to mention:** Connect **senior explanations of generics** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **senior explanations of generics** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **senior explanations of generics**.
- **One-line close:** End with a concise summary that tells the interviewer why **senior explanations of generics** matters in production code.

### Follow-up 22 — senior explanations of generics
- **Compare prompt:** Contrast **senior explanations of generics** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **senior explanations of generics** is misunderstood.
- **Code review angle:** Describe what misuse of **senior explanations of generics** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **senior explanations of generics** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **senior explanations of generics**.

