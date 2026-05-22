# 07 — Protocols and Protocol-Oriented Programming

Protocols are a core Swift abstraction tool for capability modeling, testing, and composition.

## Protocol requirements
Protocols can require properties, methods, static members, initializers, and subscripts.

### Example
```swift
protocol Cacheable {
    static var namespace: String { get }
    var cacheKey: String { get }
    init(identifier: String)
    func serialize() -> Data?
}
```

### Common pitfalls
- ⚠️ Designing protocols that lack a coherent abstraction.
- ⚠️ Forgetting protocols cannot require stored properties.
- ⚠️ Leaking implementation details into the contract.

## Protocol inheritance and composition
Use small capabilities and combine them when needed.

### Example
```swift
protocol Readable { func read() -> Data }
protocol Writable { func write(_ data: Data) }

func syncFile(_ file: Readable & Writable) {
    let data = file.read()
    file.write(data)
}
```

> 💡 **Tip:** Composition often produces cleaner dependencies than one giant protocol.

## POP philosophy
Protocol-oriented programming favors capability composition over deep inheritance.

### Example
```swift
protocol Retryable {
    var maxRetries: Int { get }
    func shouldRetry(statusCode: Int) -> Bool
}

extension Retryable {
    func shouldRetry(statusCode: Int) -> Bool {
        [500, 502, 503, 504].contains(statusCode)
    }
}
```

### Senior-level discussion
- POP is a design bias, not a religion.
- Concrete types are still valuable.
- Protocol extensions reduce boilerplate, but too much abstraction hurts discoverability.

## Default implementations and extension dispatch pitfalls
This is the classic interview trap.

### Example
```swift
protocol Greeter {
    func greet()
}

extension Greeter {
    func greet() { print("Hello from default") }
    func wave() { print("Default wave") }
}

struct Person: Greeter {
    func greet() { print("Hello from Person") }
    func wave() { print("Person wave") }
}

let value: Greeter = Person()
value.greet() // Person
value.wave()  // Default wave
```

### Why it happens
- `greet` is a protocol requirement, so witness-table dispatch can reach the conformer.
- `wave` exists only in the extension, so the call is statically dispatched from the compile-time type.

> 🎯 **Interview Answer:** If you want polymorphism through protocol-typed values, declare the member in the protocol itself.

## Topic-specific oral drills
Use these prompts to turn **protocols and pop** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — protocol method requirements
- **Definition prompt:** Explain what **protocol method requirements** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol method requirements**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol method requirements**, not just the spelling.
- **Production example to mention:** Connect **protocol method requirements** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol method requirements** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol method requirements**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol method requirements** matters in production code.

### Follow-up 1 — protocol method requirements
- **Compare prompt:** Contrast **protocol method requirements** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol method requirements** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol method requirements** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol method requirements** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol method requirements**.

### Drill 2 — protocol property requirements
- **Definition prompt:** Explain what **protocol property requirements** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol property requirements**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol property requirements**, not just the spelling.
- **Production example to mention:** Connect **protocol property requirements** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol property requirements** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol property requirements**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol property requirements** matters in production code.

### Follow-up 2 — protocol property requirements
- **Compare prompt:** Contrast **protocol property requirements** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol property requirements** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol property requirements** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol property requirements** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol property requirements**.

### Drill 3 — static requirements
- **Definition prompt:** Explain what **static requirements** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **static requirements**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **static requirements**, not just the spelling.
- **Production example to mention:** Connect **static requirements** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **static requirements** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **static requirements**.
- **One-line close:** End with a concise summary that tells the interviewer why **static requirements** matters in production code.

### Follow-up 3 — static requirements
- **Compare prompt:** Contrast **static requirements** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **static requirements** is misunderstood.
- **Code review angle:** Describe what misuse of **static requirements** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **static requirements** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **static requirements**.

### Drill 4 — initializer requirements
- **Definition prompt:** Explain what **initializer requirements** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **initializer requirements**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **initializer requirements**, not just the spelling.
- **Production example to mention:** Connect **initializer requirements** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **initializer requirements** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **initializer requirements**.
- **One-line close:** End with a concise summary that tells the interviewer why **initializer requirements** matters in production code.

### Follow-up 4 — initializer requirements
- **Compare prompt:** Contrast **initializer requirements** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **initializer requirements** is misunderstood.
- **Code review angle:** Describe what misuse of **initializer requirements** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **initializer requirements** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **initializer requirements**.

### Drill 5 — protocol inheritance
- **Definition prompt:** Explain what **protocol inheritance** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol inheritance**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol inheritance**, not just the spelling.
- **Production example to mention:** Connect **protocol inheritance** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol inheritance** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol inheritance**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol inheritance** matters in production code.

### Follow-up 5 — protocol inheritance
- **Compare prompt:** Contrast **protocol inheritance** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol inheritance** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol inheritance** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol inheritance** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol inheritance**.

### Drill 6 — protocol composition
- **Definition prompt:** Explain what **protocol composition** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol composition**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol composition**, not just the spelling.
- **Production example to mention:** Connect **protocol composition** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol composition** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol composition**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol composition** matters in production code.

### Follow-up 6 — protocol composition
- **Compare prompt:** Contrast **protocol composition** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol composition** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol composition** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol composition** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol composition**.

### Drill 7 — small protocols vs giant protocols
- **Definition prompt:** Explain what **small protocols vs giant protocols** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **small protocols vs giant protocols**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **small protocols vs giant protocols**, not just the spelling.
- **Production example to mention:** Connect **small protocols vs giant protocols** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **small protocols vs giant protocols** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **small protocols vs giant protocols**.
- **One-line close:** End with a concise summary that tells the interviewer why **small protocols vs giant protocols** matters in production code.

### Follow-up 7 — small protocols vs giant protocols
- **Compare prompt:** Contrast **small protocols vs giant protocols** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **small protocols vs giant protocols** is misunderstood.
- **Code review angle:** Describe what misuse of **small protocols vs giant protocols** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **small protocols vs giant protocols** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **small protocols vs giant protocols**.

### Drill 8 — protocol-oriented programming philosophy
- **Definition prompt:** Explain what **protocol-oriented programming philosophy** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol-oriented programming philosophy**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol-oriented programming philosophy**, not just the spelling.
- **Production example to mention:** Connect **protocol-oriented programming philosophy** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol-oriented programming philosophy** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol-oriented programming philosophy**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol-oriented programming philosophy** matters in production code.

### Follow-up 8 — protocol-oriented programming philosophy
- **Compare prompt:** Contrast **protocol-oriented programming philosophy** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol-oriented programming philosophy** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol-oriented programming philosophy** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol-oriented programming philosophy** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol-oriented programming philosophy**.

### Drill 9 — default implementations
- **Definition prompt:** Explain what **default implementations** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **default implementations**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **default implementations**, not just the spelling.
- **Production example to mention:** Connect **default implementations** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **default implementations** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **default implementations**.
- **One-line close:** End with a concise summary that tells the interviewer why **default implementations** matters in production code.

### Follow-up 9 — default implementations
- **Compare prompt:** Contrast **default implementations** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **default implementations** is misunderstood.
- **Code review angle:** Describe what misuse of **default implementations** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **default implementations** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **default implementations**.

### Drill 10 — protocol extensions as helpers
- **Definition prompt:** Explain what **protocol extensions as helpers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol extensions as helpers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol extensions as helpers**, not just the spelling.
- **Production example to mention:** Connect **protocol extensions as helpers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol extensions as helpers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol extensions as helpers**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol extensions as helpers** matters in production code.

### Follow-up 10 — protocol extensions as helpers
- **Compare prompt:** Contrast **protocol extensions as helpers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol extensions as helpers** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol extensions as helpers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol extensions as helpers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol extensions as helpers**.

### Drill 11 — cohesive protocol design
- **Definition prompt:** Explain what **cohesive protocol design** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **cohesive protocol design**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **cohesive protocol design**, not just the spelling.
- **Production example to mention:** Connect **cohesive protocol design** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **cohesive protocol design** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **cohesive protocol design**.
- **One-line close:** End with a concise summary that tells the interviewer why **cohesive protocol design** matters in production code.

### Follow-up 11 — cohesive protocol design
- **Compare prompt:** Contrast **cohesive protocol design** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **cohesive protocol design** is misunderstood.
- **Code review angle:** Describe what misuse of **cohesive protocol design** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **cohesive protocol design** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **cohesive protocol design**.

### Drill 12 — testing with protocols
- **Definition prompt:** Explain what **testing with protocols** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **testing with protocols**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **testing with protocols**, not just the spelling.
- **Production example to mention:** Connect **testing with protocols** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **testing with protocols** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **testing with protocols**.
- **One-line close:** End with a concise summary that tells the interviewer why **testing with protocols** matters in production code.

### Follow-up 12 — testing with protocols
- **Compare prompt:** Contrast **testing with protocols** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **testing with protocols** is misunderstood.
- **Code review angle:** Describe what misuse of **testing with protocols** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **testing with protocols** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **testing with protocols**.

### Drill 13 — abstraction cost
- **Definition prompt:** Explain what **abstraction cost** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **abstraction cost**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **abstraction cost**, not just the spelling.
- **Production example to mention:** Connect **abstraction cost** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **abstraction cost** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **abstraction cost**.
- **One-line close:** End with a concise summary that tells the interviewer why **abstraction cost** matters in production code.

### Follow-up 13 — abstraction cost
- **Compare prompt:** Contrast **abstraction cost** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **abstraction cost** is misunderstood.
- **Code review angle:** Describe what misuse of **abstraction cost** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **abstraction cost** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **abstraction cost**.

### Drill 14 — witness-table dispatch
- **Definition prompt:** Explain what **witness-table dispatch** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **witness-table dispatch**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **witness-table dispatch**, not just the spelling.
- **Production example to mention:** Connect **witness-table dispatch** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **witness-table dispatch** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **witness-table dispatch**.
- **One-line close:** End with a concise summary that tells the interviewer why **witness-table dispatch** matters in production code.

### Follow-up 14 — witness-table dispatch
- **Compare prompt:** Contrast **witness-table dispatch** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **witness-table dispatch** is misunderstood.
- **Code review angle:** Describe what misuse of **witness-table dispatch** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **witness-table dispatch** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **witness-table dispatch**.

### Drill 15 — extension-only methods
- **Definition prompt:** Explain what **extension-only methods** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **extension-only methods**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **extension-only methods**, not just the spelling.
- **Production example to mention:** Connect **extension-only methods** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **extension-only methods** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **extension-only methods**.
- **One-line close:** End with a concise summary that tells the interviewer why **extension-only methods** matters in production code.

### Follow-up 15 — extension-only methods
- **Compare prompt:** Contrast **extension-only methods** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **extension-only methods** is misunderstood.
- **Code review angle:** Describe what misuse of **extension-only methods** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **extension-only methods** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **extension-only methods**.

### Drill 16 — protocol existential call sites
- **Definition prompt:** Explain what **protocol existential call sites** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol existential call sites**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol existential call sites**, not just the spelling.
- **Production example to mention:** Connect **protocol existential call sites** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol existential call sites** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol existential call sites**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol existential call sites** matters in production code.

### Follow-up 16 — protocol existential call sites
- **Compare prompt:** Contrast **protocol existential call sites** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol existential call sites** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol existential call sites** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol existential call sites** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol existential call sites**.

### Drill 17 — compile-time type vs runtime type
- **Definition prompt:** Explain what **compile-time type vs runtime type** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **compile-time type vs runtime type**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **compile-time type vs runtime type**, not just the spelling.
- **Production example to mention:** Connect **compile-time type vs runtime type** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **compile-time type vs runtime type** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **compile-time type vs runtime type**.
- **One-line close:** End with a concise summary that tells the interviewer why **compile-time type vs runtime type** matters in production code.

### Follow-up 17 — compile-time type vs runtime type
- **Compare prompt:** Contrast **compile-time type vs runtime type** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **compile-time type vs runtime type** is misunderstood.
- **Code review angle:** Describe what misuse of **compile-time type vs runtime type** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **compile-time type vs runtime type** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **compile-time type vs runtime type**.

### Drill 18 — retroactive modeling
- **Definition prompt:** Explain what **retroactive modeling** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **retroactive modeling**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **retroactive modeling**, not just the spelling.
- **Production example to mention:** Connect **retroactive modeling** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **retroactive modeling** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **retroactive modeling**.
- **One-line close:** End with a concise summary that tells the interviewer why **retroactive modeling** matters in production code.

### Follow-up 18 — retroactive modeling
- **Compare prompt:** Contrast **retroactive modeling** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **retroactive modeling** is misunderstood.
- **Code review angle:** Describe what misuse of **retroactive modeling** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **retroactive modeling** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **retroactive modeling**.

### Drill 19 — mixing concrete and protocol APIs
- **Definition prompt:** Explain what **mixing concrete and protocol APIs** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **mixing concrete and protocol APIs**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **mixing concrete and protocol APIs**, not just the spelling.
- **Production example to mention:** Connect **mixing concrete and protocol APIs** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **mixing concrete and protocol APIs** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **mixing concrete and protocol APIs**.
- **One-line close:** End with a concise summary that tells the interviewer why **mixing concrete and protocol APIs** matters in production code.

### Follow-up 19 — mixing concrete and protocol APIs
- **Compare prompt:** Contrast **mixing concrete and protocol APIs** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **mixing concrete and protocol APIs** is misunderstood.
- **Code review angle:** Describe what misuse of **mixing concrete and protocol APIs** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **mixing concrete and protocol APIs** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **mixing concrete and protocol APIs**.

### Drill 20 — when classes are simpler than protocols
- **Definition prompt:** Explain what **when classes are simpler than protocols** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **when classes are simpler than protocols**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **when classes are simpler than protocols**, not just the spelling.
- **Production example to mention:** Connect **when classes are simpler than protocols** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **when classes are simpler than protocols** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **when classes are simpler than protocols**.
- **One-line close:** End with a concise summary that tells the interviewer why **when classes are simpler than protocols** matters in production code.

### Follow-up 20 — when classes are simpler than protocols
- **Compare prompt:** Contrast **when classes are simpler than protocols** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **when classes are simpler than protocols** is misunderstood.
- **Code review angle:** Describe what misuse of **when classes are simpler than protocols** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **when classes are simpler than protocols** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **when classes are simpler than protocols**.

### Drill 21 — naming protocols well
- **Definition prompt:** Explain what **naming protocols well** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **naming protocols well**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **naming protocols well**, not just the spelling.
- **Production example to mention:** Connect **naming protocols well** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **naming protocols well** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **naming protocols well**.
- **One-line close:** End with a concise summary that tells the interviewer why **naming protocols well** matters in production code.

### Follow-up 21 — naming protocols well
- **Compare prompt:** Contrast **naming protocols well** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **naming protocols well** is misunderstood.
- **Code review angle:** Describe what misuse of **naming protocols well** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **naming protocols well** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **naming protocols well**.

### Drill 22 — POP tradeoffs in app architecture
- **Definition prompt:** Explain what **POP tradeoffs in app architecture** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **POP tradeoffs in app architecture**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **POP tradeoffs in app architecture**, not just the spelling.
- **Production example to mention:** Connect **POP tradeoffs in app architecture** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **POP tradeoffs in app architecture** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **POP tradeoffs in app architecture**.
- **One-line close:** End with a concise summary that tells the interviewer why **POP tradeoffs in app architecture** matters in production code.

### Follow-up 22 — POP tradeoffs in app architecture
- **Compare prompt:** Contrast **POP tradeoffs in app architecture** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **POP tradeoffs in app architecture** is misunderstood.
- **Code review angle:** Describe what misuse of **POP tradeoffs in app architecture** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **POP tradeoffs in app architecture** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **POP tradeoffs in app architecture**.

