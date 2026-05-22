# 11 — Access Control and Dispatch

This topic reveals whether you understand both API design and runtime behavior.

## Access control levels
- `open`: visible and subclassable/overridable outside the module.
- `public`: visible outside the module but not open for outside overriding.
- `internal`: default, module-only.
- `fileprivate`: file-only.
- `private`: enclosing declaration-focused privacy.

### Example
```swift
public struct APIClient {
    public init() {}
    public func send() {}
}

open class BaseViewController: UIViewController {
    open func trackScreen() {}
}
```

## Static dispatch, dynamic dispatch, witness tables, and vtables
### Mental model
- Static dispatch resolves at compile time.
- Class overrides use dynamic dispatch.
- Protocol requirement calls through protocol-typed values use witness tables.

### Example
```swift
protocol Renderer {
    func render()
}

struct TextRenderer: Renderer {
    func render() { print("text") }
}
```

## Protocol dispatch rules and extension traps
### Example
```swift
protocol Worker {
    func work()
}

extension Worker {
    func work() { print("default work") }
    func debugInfo() { print("default debug") }
}

struct Engineer: Worker {
    func work() { print("engineer work") }
    func debugInfo() { print("engineer debug") }
}

let abstract: Worker = Engineer()
abstract.work()      // engineer work
abstract.debugInfo() // default debug
```

> 🎯 **Interview Answer:** Only protocol requirements participate in witness-table polymorphism through protocol-typed values. Extension-only methods are statically dispatched.

## Topic-specific oral drills
Use these prompts to turn **access control and dispatch** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — `open`
- **Definition prompt:** Explain what **`open`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`open`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`open`**, not just the spelling.
- **Production example to mention:** Connect **`open`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`open`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`open`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`open`** matters in production code.

### Follow-up 1 — `open`
- **Compare prompt:** Contrast **`open`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`open`** is misunderstood.
- **Code review angle:** Describe what misuse of **`open`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`open`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`open`**.

### Drill 2 — `public`
- **Definition prompt:** Explain what **`public`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`public`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`public`**, not just the spelling.
- **Production example to mention:** Connect **`public`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`public`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`public`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`public`** matters in production code.

### Follow-up 2 — `public`
- **Compare prompt:** Contrast **`public`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`public`** is misunderstood.
- **Code review angle:** Describe what misuse of **`public`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`public`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`public`**.

### Drill 3 — `internal`
- **Definition prompt:** Explain what **`internal`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`internal`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`internal`**, not just the spelling.
- **Production example to mention:** Connect **`internal`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`internal`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`internal`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`internal`** matters in production code.

### Follow-up 3 — `internal`
- **Compare prompt:** Contrast **`internal`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`internal`** is misunderstood.
- **Code review angle:** Describe what misuse of **`internal`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`internal`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`internal`**.

### Drill 4 — `fileprivate`
- **Definition prompt:** Explain what **`fileprivate`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`fileprivate`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`fileprivate`**, not just the spelling.
- **Production example to mention:** Connect **`fileprivate`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`fileprivate`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`fileprivate`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`fileprivate`** matters in production code.

### Follow-up 4 — `fileprivate`
- **Compare prompt:** Contrast **`fileprivate`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`fileprivate`** is misunderstood.
- **Code review angle:** Describe what misuse of **`fileprivate`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`fileprivate`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`fileprivate`**.

### Drill 5 — `private`
- **Definition prompt:** Explain what **`private`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`private`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`private`**, not just the spelling.
- **Production example to mention:** Connect **`private`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`private`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`private`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`private`** matters in production code.

### Follow-up 5 — `private`
- **Compare prompt:** Contrast **`private`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`private`** is misunderstood.
- **Code review angle:** Describe what misuse of **`private`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`private`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`private`**.

### Drill 6 — encapsulation strategy
- **Definition prompt:** Explain what **encapsulation strategy** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **encapsulation strategy**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **encapsulation strategy**, not just the spelling.
- **Production example to mention:** Connect **encapsulation strategy** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **encapsulation strategy** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **encapsulation strategy**.
- **One-line close:** End with a concise summary that tells the interviewer why **encapsulation strategy** matters in production code.

### Follow-up 6 — encapsulation strategy
- **Compare prompt:** Contrast **encapsulation strategy** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **encapsulation strategy** is misunderstood.
- **Code review angle:** Describe what misuse of **encapsulation strategy** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **encapsulation strategy** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **encapsulation strategy**.

### Drill 7 — framework API design
- **Definition prompt:** Explain what **framework API design** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **framework API design**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **framework API design**, not just the spelling.
- **Production example to mention:** Connect **framework API design** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **framework API design** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **framework API design**.
- **One-line close:** End with a concise summary that tells the interviewer why **framework API design** matters in production code.

### Follow-up 7 — framework API design
- **Compare prompt:** Contrast **framework API design** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **framework API design** is misunderstood.
- **Code review angle:** Describe what misuse of **framework API design** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **framework API design** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **framework API design**.

### Drill 8 — `public` vs `open`
- **Definition prompt:** Explain what **`public` vs `open`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`public` vs `open`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`public` vs `open`**, not just the spelling.
- **Production example to mention:** Connect **`public` vs `open`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`public` vs `open`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`public` vs `open`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`public` vs `open`** matters in production code.

### Follow-up 8 — `public` vs `open`
- **Compare prompt:** Contrast **`public` vs `open`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`public` vs `open`** is misunderstood.
- **Code review angle:** Describe what misuse of **`public` vs `open`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`public` vs `open`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`public` vs `open`**.

### Drill 9 — static dispatch
- **Definition prompt:** Explain what **static dispatch** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **static dispatch**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **static dispatch**, not just the spelling.
- **Production example to mention:** Connect **static dispatch** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **static dispatch** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **static dispatch**.
- **One-line close:** End with a concise summary that tells the interviewer why **static dispatch** matters in production code.

### Follow-up 9 — static dispatch
- **Compare prompt:** Contrast **static dispatch** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **static dispatch** is misunderstood.
- **Code review angle:** Describe what misuse of **static dispatch** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **static dispatch** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **static dispatch**.

### Drill 10 — dynamic dispatch
- **Definition prompt:** Explain what **dynamic dispatch** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **dynamic dispatch**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **dynamic dispatch**, not just the spelling.
- **Production example to mention:** Connect **dynamic dispatch** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **dynamic dispatch** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **dynamic dispatch**.
- **One-line close:** End with a concise summary that tells the interviewer why **dynamic dispatch** matters in production code.

### Follow-up 10 — dynamic dispatch
- **Compare prompt:** Contrast **dynamic dispatch** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **dynamic dispatch** is misunderstood.
- **Code review angle:** Describe what misuse of **dynamic dispatch** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **dynamic dispatch** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **dynamic dispatch**.

### Drill 11 — class vtables
- **Definition prompt:** Explain what **class vtables** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **class vtables**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **class vtables**, not just the spelling.
- **Production example to mention:** Connect **class vtables** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **class vtables** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **class vtables**.
- **One-line close:** End with a concise summary that tells the interviewer why **class vtables** matters in production code.

### Follow-up 11 — class vtables
- **Compare prompt:** Contrast **class vtables** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **class vtables** is misunderstood.
- **Code review angle:** Describe what misuse of **class vtables** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **class vtables** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **class vtables**.

### Drill 12 — protocol witness tables
- **Definition prompt:** Explain what **protocol witness tables** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol witness tables**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol witness tables**, not just the spelling.
- **Production example to mention:** Connect **protocol witness tables** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol witness tables** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol witness tables**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol witness tables** matters in production code.

### Follow-up 12 — protocol witness tables
- **Compare prompt:** Contrast **protocol witness tables** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol witness tables** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol witness tables** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol witness tables** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol witness tables**.

### Drill 13 — final methods
- **Definition prompt:** Explain what **final methods** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **final methods**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **final methods**, not just the spelling.
- **Production example to mention:** Connect **final methods** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **final methods** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **final methods**.
- **One-line close:** End with a concise summary that tells the interviewer why **final methods** matters in production code.

### Follow-up 13 — final methods
- **Compare prompt:** Contrast **final methods** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **final methods** is misunderstood.
- **Code review angle:** Describe what misuse of **final methods** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **final methods** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **final methods**.

### Drill 14 — protocol requirement dispatch
- **Definition prompt:** Explain what **protocol requirement dispatch** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **protocol requirement dispatch**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **protocol requirement dispatch**, not just the spelling.
- **Production example to mention:** Connect **protocol requirement dispatch** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **protocol requirement dispatch** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **protocol requirement dispatch**.
- **One-line close:** End with a concise summary that tells the interviewer why **protocol requirement dispatch** matters in production code.

### Follow-up 14 — protocol requirement dispatch
- **Compare prompt:** Contrast **protocol requirement dispatch** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **protocol requirement dispatch** is misunderstood.
- **Code review angle:** Describe what misuse of **protocol requirement dispatch** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **protocol requirement dispatch** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **protocol requirement dispatch**.

### Drill 15 — extension-only dispatch
- **Definition prompt:** Explain what **extension-only dispatch** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **extension-only dispatch**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **extension-only dispatch**, not just the spelling.
- **Production example to mention:** Connect **extension-only dispatch** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **extension-only dispatch** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **extension-only dispatch**.
- **One-line close:** End with a concise summary that tells the interviewer why **extension-only dispatch** matters in production code.

### Follow-up 15 — extension-only dispatch
- **Compare prompt:** Contrast **extension-only dispatch** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **extension-only dispatch** is misunderstood.
- **Code review angle:** Describe what misuse of **extension-only dispatch** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **extension-only dispatch** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **extension-only dispatch**.

### Drill 16 — compile-time type effects
- **Definition prompt:** Explain what **compile-time type effects** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **compile-time type effects**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **compile-time type effects**, not just the spelling.
- **Production example to mention:** Connect **compile-time type effects** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **compile-time type effects** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **compile-time type effects**.
- **One-line close:** End with a concise summary that tells the interviewer why **compile-time type effects** matters in production code.

### Follow-up 16 — compile-time type effects
- **Compare prompt:** Contrast **compile-time type effects** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **compile-time type effects** is misunderstood.
- **Code review angle:** Describe what misuse of **compile-time type effects** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **compile-time type effects** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **compile-time type effects**.

### Drill 17 — runtime polymorphism
- **Definition prompt:** Explain what **runtime polymorphism** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **runtime polymorphism**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **runtime polymorphism**, not just the spelling.
- **Production example to mention:** Connect **runtime polymorphism** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **runtime polymorphism** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **runtime polymorphism**.
- **One-line close:** End with a concise summary that tells the interviewer why **runtime polymorphism** matters in production code.

### Follow-up 17 — runtime polymorphism
- **Compare prompt:** Contrast **runtime polymorphism** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **runtime polymorphism** is misunderstood.
- **Code review angle:** Describe what misuse of **runtime polymorphism** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **runtime polymorphism** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **runtime polymorphism**.

### Drill 18 — performance implications of dispatch
- **Definition prompt:** Explain what **performance implications of dispatch** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **performance implications of dispatch**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **performance implications of dispatch**, not just the spelling.
- **Production example to mention:** Connect **performance implications of dispatch** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **performance implications of dispatch** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **performance implications of dispatch**.
- **One-line close:** End with a concise summary that tells the interviewer why **performance implications of dispatch** matters in production code.

### Follow-up 18 — performance implications of dispatch
- **Compare prompt:** Contrast **performance implications of dispatch** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **performance implications of dispatch** is misunderstood.
- **Code review angle:** Describe what misuse of **performance implications of dispatch** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **performance implications of dispatch** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **performance implications of dispatch**.

### Drill 19 — access control and testing
- **Definition prompt:** Explain what **access control and testing** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **access control and testing**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **access control and testing**, not just the spelling.
- **Production example to mention:** Connect **access control and testing** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **access control and testing** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **access control and testing**.
- **One-line close:** End with a concise summary that tells the interviewer why **access control and testing** matters in production code.

### Follow-up 19 — access control and testing
- **Compare prompt:** Contrast **access control and testing** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **access control and testing** is misunderstood.
- **Code review angle:** Describe what misuse of **access control and testing** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **access control and testing** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **access control and testing**.

### Drill 20 — module boundaries
- **Definition prompt:** Explain what **module boundaries** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **module boundaries**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **module boundaries**, not just the spelling.
- **Production example to mention:** Connect **module boundaries** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **module boundaries** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **module boundaries**.
- **One-line close:** End with a concise summary that tells the interviewer why **module boundaries** matters in production code.

### Follow-up 20 — module boundaries
- **Compare prompt:** Contrast **module boundaries** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **module boundaries** is misunderstood.
- **Code review angle:** Describe what misuse of **module boundaries** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **module boundaries** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **module boundaries**.

### Drill 21 — extension dispatch traps
- **Definition prompt:** Explain what **extension dispatch traps** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **extension dispatch traps**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **extension dispatch traps**, not just the spelling.
- **Production example to mention:** Connect **extension dispatch traps** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **extension dispatch traps** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **extension dispatch traps**.
- **One-line close:** End with a concise summary that tells the interviewer why **extension dispatch traps** matters in production code.

### Follow-up 21 — extension dispatch traps
- **Compare prompt:** Contrast **extension dispatch traps** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **extension dispatch traps** is misunderstood.
- **Code review angle:** Describe what misuse of **extension dispatch traps** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **extension dispatch traps** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **extension dispatch traps**.

### Drill 22 — how to explain dispatch simply
- **Definition prompt:** Explain what **how to explain dispatch simply** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **how to explain dispatch simply**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **how to explain dispatch simply**, not just the spelling.
- **Production example to mention:** Connect **how to explain dispatch simply** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **how to explain dispatch simply** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **how to explain dispatch simply**.
- **One-line close:** End with a concise summary that tells the interviewer why **how to explain dispatch simply** matters in production code.

### Follow-up 22 — how to explain dispatch simply
- **Compare prompt:** Contrast **how to explain dispatch simply** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **how to explain dispatch simply** is misunderstood.
- **Code review angle:** Describe what misuse of **how to explain dispatch simply** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **how to explain dispatch simply** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **how to explain dispatch simply**.

