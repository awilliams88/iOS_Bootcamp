# 13 — Modern Swift

Modern Swift interviews reward awareness of where the language is going.

## Swift 5.9/5.10 direction
- More expressive generics.
- More compile-time tooling via macros.
- More explicit ownership.
- Stricter concurrency correctness heading into Swift 6.

## Parameter packs and variadic generics
Parameter packs reduce overload explosion for APIs that work over arbitrary numbers of type parameters.

> 💡 **Tip:** Even if you have not used the syntax heavily in production, explain the pain point it solves.

## Macros overview
Macros provide structured compile-time code generation.

### Interview framing
- Safer and more structured than textual C-style macros.
- Useful for reducing repetitive declarations and conformance boilerplate.
- Should be adopted thoughtfully because code generation affects readability.

## Ownership keywords and noncopyable types
`borrowing`, `consuming`, and `~Copyable` are part of Swift's move toward more explicit ownership modeling.

### Why it matters
- Resource handles and single-owner state can be modeled more precisely.
- Performance-sensitive libraries gain more control over copies.
- Swift is broadening from app ergonomics into stronger systems-level capabilities.

## Typed throws, Swift 6 concurrency, expression forms
### Example
```swift
let status = if isEnabled {
    "enabled"
} else {
    "disabled"
}
```

### Senior-level discussion
- Typed throws aim to make error contracts more explicit.
- Swift 6 concurrency checks tighten Sendable and isolation enforcement.
- Modern Swift is becoming more explicit about correctness while reducing boilerplate.

## Topic-specific oral drills
Use these prompts to turn **modern swift** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — Swift 5.9/5.10 direction
- **Definition prompt:** Explain what **Swift 5.9/5.10 direction** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **Swift 5.9/5.10 direction**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **Swift 5.9/5.10 direction**, not just the spelling.
- **Production example to mention:** Connect **Swift 5.9/5.10 direction** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **Swift 5.9/5.10 direction** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **Swift 5.9/5.10 direction**.
- **One-line close:** End with a concise summary that tells the interviewer why **Swift 5.9/5.10 direction** matters in production code.

### Follow-up 1 — Swift 5.9/5.10 direction
- **Compare prompt:** Contrast **Swift 5.9/5.10 direction** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **Swift 5.9/5.10 direction** is misunderstood.
- **Code review angle:** Describe what misuse of **Swift 5.9/5.10 direction** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **Swift 5.9/5.10 direction** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **Swift 5.9/5.10 direction**.

### Drill 2 — modern Swift themes
- **Definition prompt:** Explain what **modern Swift themes** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **modern Swift themes**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **modern Swift themes**, not just the spelling.
- **Production example to mention:** Connect **modern Swift themes** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **modern Swift themes** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **modern Swift themes**.
- **One-line close:** End with a concise summary that tells the interviewer why **modern Swift themes** matters in production code.

### Follow-up 2 — modern Swift themes
- **Compare prompt:** Contrast **modern Swift themes** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **modern Swift themes** is misunderstood.
- **Code review angle:** Describe what misuse of **modern Swift themes** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **modern Swift themes** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **modern Swift themes**.

### Drill 3 — parameter packs
- **Definition prompt:** Explain what **parameter packs** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **parameter packs**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **parameter packs**, not just the spelling.
- **Production example to mention:** Connect **parameter packs** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **parameter packs** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **parameter packs**.
- **One-line close:** End with a concise summary that tells the interviewer why **parameter packs** matters in production code.

### Follow-up 3 — parameter packs
- **Compare prompt:** Contrast **parameter packs** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **parameter packs** is misunderstood.
- **Code review angle:** Describe what misuse of **parameter packs** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **parameter packs** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **parameter packs**.

### Drill 4 — variadic generics
- **Definition prompt:** Explain what **variadic generics** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **variadic generics**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **variadic generics**, not just the spelling.
- **Production example to mention:** Connect **variadic generics** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **variadic generics** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **variadic generics**.
- **One-line close:** End with a concise summary that tells the interviewer why **variadic generics** matters in production code.

### Follow-up 4 — variadic generics
- **Compare prompt:** Contrast **variadic generics** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **variadic generics** is misunderstood.
- **Code review angle:** Describe what misuse of **variadic generics** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **variadic generics** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **variadic generics**.

### Drill 5 — overload explosion reduction
- **Definition prompt:** Explain what **overload explosion reduction** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **overload explosion reduction**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **overload explosion reduction**, not just the spelling.
- **Production example to mention:** Connect **overload explosion reduction** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **overload explosion reduction** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **overload explosion reduction**.
- **One-line close:** End with a concise summary that tells the interviewer why **overload explosion reduction** matters in production code.

### Follow-up 5 — overload explosion reduction
- **Compare prompt:** Contrast **overload explosion reduction** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **overload explosion reduction** is misunderstood.
- **Code review angle:** Describe what misuse of **overload explosion reduction** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **overload explosion reduction** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **overload explosion reduction**.

### Drill 6 — macro overview
- **Definition prompt:** Explain what **macro overview** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **macro overview**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **macro overview**, not just the spelling.
- **Production example to mention:** Connect **macro overview** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **macro overview** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **macro overview**.
- **One-line close:** End with a concise summary that tells the interviewer why **macro overview** matters in production code.

### Follow-up 6 — macro overview
- **Compare prompt:** Contrast **macro overview** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **macro overview** is misunderstood.
- **Code review angle:** Describe what misuse of **macro overview** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **macro overview** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **macro overview**.

### Drill 7 — macro safety vs textual macros
- **Definition prompt:** Explain what **macro safety vs textual macros** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **macro safety vs textual macros**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **macro safety vs textual macros**, not just the spelling.
- **Production example to mention:** Connect **macro safety vs textual macros** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **macro safety vs textual macros** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **macro safety vs textual macros**.
- **One-line close:** End with a concise summary that tells the interviewer why **macro safety vs textual macros** matters in production code.

### Follow-up 7 — macro safety vs textual macros
- **Compare prompt:** Contrast **macro safety vs textual macros** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **macro safety vs textual macros** is misunderstood.
- **Code review angle:** Describe what misuse of **macro safety vs textual macros** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **macro safety vs textual macros** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **macro safety vs textual macros**.

### Drill 8 — compile-time code generation
- **Definition prompt:** Explain what **compile-time code generation** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **compile-time code generation**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **compile-time code generation**, not just the spelling.
- **Production example to mention:** Connect **compile-time code generation** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **compile-time code generation** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **compile-time code generation**.
- **One-line close:** End with a concise summary that tells the interviewer why **compile-time code generation** matters in production code.

### Follow-up 8 — compile-time code generation
- **Compare prompt:** Contrast **compile-time code generation** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **compile-time code generation** is misunderstood.
- **Code review angle:** Describe what misuse of **compile-time code generation** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **compile-time code generation** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **compile-time code generation**.

### Drill 9 — observation-style macros
- **Definition prompt:** Explain what **observation-style macros** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **observation-style macros**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **observation-style macros**, not just the spelling.
- **Production example to mention:** Connect **observation-style macros** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **observation-style macros** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **observation-style macros**.
- **One-line close:** End with a concise summary that tells the interviewer why **observation-style macros** matters in production code.

### Follow-up 9 — observation-style macros
- **Compare prompt:** Contrast **observation-style macros** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **observation-style macros** is misunderstood.
- **Code review angle:** Describe what misuse of **observation-style macros** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **observation-style macros** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **observation-style macros**.

### Drill 10 — ownership keywords
- **Definition prompt:** Explain what **ownership keywords** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **ownership keywords**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **ownership keywords**, not just the spelling.
- **Production example to mention:** Connect **ownership keywords** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **ownership keywords** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **ownership keywords**.
- **One-line close:** End with a concise summary that tells the interviewer why **ownership keywords** matters in production code.

### Follow-up 10 — ownership keywords
- **Compare prompt:** Contrast **ownership keywords** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **ownership keywords** is misunderstood.
- **Code review angle:** Describe what misuse of **ownership keywords** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **ownership keywords** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **ownership keywords**.

### Drill 11 — `borrowing`
- **Definition prompt:** Explain what **`borrowing`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`borrowing`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`borrowing`**, not just the spelling.
- **Production example to mention:** Connect **`borrowing`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`borrowing`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`borrowing`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`borrowing`** matters in production code.

### Follow-up 11 — `borrowing`
- **Compare prompt:** Contrast **`borrowing`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`borrowing`** is misunderstood.
- **Code review angle:** Describe what misuse of **`borrowing`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`borrowing`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`borrowing`**.

### Drill 12 — `consuming`
- **Definition prompt:** Explain what **`consuming`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`consuming`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`consuming`**, not just the spelling.
- **Production example to mention:** Connect **`consuming`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`consuming`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`consuming`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`consuming`** matters in production code.

### Follow-up 12 — `consuming`
- **Compare prompt:** Contrast **`consuming`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`consuming`** is misunderstood.
- **Code review angle:** Describe what misuse of **`consuming`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`consuming`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`consuming`**.

### Drill 13 — `~Copyable`
- **Definition prompt:** Explain what **`~Copyable`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`~Copyable`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`~Copyable`**, not just the spelling.
- **Production example to mention:** Connect **`~Copyable`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`~Copyable`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`~Copyable`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`~Copyable`** matters in production code.

### Follow-up 13 — `~Copyable`
- **Compare prompt:** Contrast **`~Copyable`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`~Copyable`** is misunderstood.
- **Code review angle:** Describe what misuse of **`~Copyable`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`~Copyable`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`~Copyable`**.

### Drill 14 — noncopyable types
- **Definition prompt:** Explain what **noncopyable types** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **noncopyable types**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **noncopyable types**, not just the spelling.
- **Production example to mention:** Connect **noncopyable types** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **noncopyable types** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **noncopyable types**.
- **One-line close:** End with a concise summary that tells the interviewer why **noncopyable types** matters in production code.

### Follow-up 14 — noncopyable types
- **Compare prompt:** Contrast **noncopyable types** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **noncopyable types** is misunderstood.
- **Code review angle:** Describe what misuse of **noncopyable types** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **noncopyable types** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **noncopyable types**.

### Drill 15 — resource-oriented APIs
- **Definition prompt:** Explain what **resource-oriented APIs** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **resource-oriented APIs**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **resource-oriented APIs**, not just the spelling.
- **Production example to mention:** Connect **resource-oriented APIs** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **resource-oriented APIs** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **resource-oriented APIs**.
- **One-line close:** End with a concise summary that tells the interviewer why **resource-oriented APIs** matters in production code.

### Follow-up 15 — resource-oriented APIs
- **Compare prompt:** Contrast **resource-oriented APIs** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **resource-oriented APIs** is misunderstood.
- **Code review angle:** Describe what misuse of **resource-oriented APIs** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **resource-oriented APIs** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **resource-oriented APIs**.

### Drill 16 — typed throws awareness
- **Definition prompt:** Explain what **typed throws awareness** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **typed throws awareness**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **typed throws awareness**, not just the spelling.
- **Production example to mention:** Connect **typed throws awareness** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **typed throws awareness** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **typed throws awareness**.
- **One-line close:** End with a concise summary that tells the interviewer why **typed throws awareness** matters in production code.

### Follow-up 16 — typed throws awareness
- **Compare prompt:** Contrast **typed throws awareness** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **typed throws awareness** is misunderstood.
- **Code review angle:** Describe what misuse of **typed throws awareness** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **typed throws awareness** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **typed throws awareness**.

### Drill 17 — Swift 6 concurrency changes
- **Definition prompt:** Explain what **Swift 6 concurrency changes** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **Swift 6 concurrency changes**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **Swift 6 concurrency changes**, not just the spelling.
- **Production example to mention:** Connect **Swift 6 concurrency changes** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **Swift 6 concurrency changes** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **Swift 6 concurrency changes**.
- **One-line close:** End with a concise summary that tells the interviewer why **Swift 6 concurrency changes** matters in production code.

### Follow-up 17 — Swift 6 concurrency changes
- **Compare prompt:** Contrast **Swift 6 concurrency changes** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **Swift 6 concurrency changes** is misunderstood.
- **Code review angle:** Describe what misuse of **Swift 6 concurrency changes** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **Swift 6 concurrency changes** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **Swift 6 concurrency changes**.

### Drill 18 — stricter Sendable checking
- **Definition prompt:** Explain what **stricter Sendable checking** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **stricter Sendable checking**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **stricter Sendable checking**, not just the spelling.
- **Production example to mention:** Connect **stricter Sendable checking** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **stricter Sendable checking** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **stricter Sendable checking**.
- **One-line close:** End with a concise summary that tells the interviewer why **stricter Sendable checking** matters in production code.

### Follow-up 18 — stricter Sendable checking
- **Compare prompt:** Contrast **stricter Sendable checking** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **stricter Sendable checking** is misunderstood.
- **Code review angle:** Describe what misuse of **stricter Sendable checking** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **stricter Sendable checking** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **stricter Sendable checking**.

### Drill 19 — `if` expressions
- **Definition prompt:** Explain what **`if` expressions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`if` expressions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`if` expressions**, not just the spelling.
- **Production example to mention:** Connect **`if` expressions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`if` expressions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`if` expressions**.
- **One-line close:** End with a concise summary that tells the interviewer why **`if` expressions** matters in production code.

### Follow-up 19 — `if` expressions
- **Compare prompt:** Contrast **`if` expressions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`if` expressions** is misunderstood.
- **Code review angle:** Describe what misuse of **`if` expressions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`if` expressions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`if` expressions**.

### Drill 20 — `switch` expressions
- **Definition prompt:** Explain what **`switch` expressions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`switch` expressions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`switch` expressions**, not just the spelling.
- **Production example to mention:** Connect **`switch` expressions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`switch` expressions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`switch` expressions**.
- **One-line close:** End with a concise summary that tells the interviewer why **`switch` expressions** matters in production code.

### Follow-up 20 — `switch` expressions
- **Compare prompt:** Contrast **`switch` expressions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`switch` expressions** is misunderstood.
- **Code review angle:** Describe what misuse of **`switch` expressions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`switch` expressions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`switch` expressions**.

### Drill 21 — string interpolation improvements
- **Definition prompt:** Explain what **string interpolation improvements** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **string interpolation improvements**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **string interpolation improvements**, not just the spelling.
- **Production example to mention:** Connect **string interpolation improvements** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **string interpolation improvements** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **string interpolation improvements**.
- **One-line close:** End with a concise summary that tells the interviewer why **string interpolation improvements** matters in production code.

### Follow-up 21 — string interpolation improvements
- **Compare prompt:** Contrast **string interpolation improvements** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **string interpolation improvements** is misunderstood.
- **Code review angle:** Describe what misuse of **string interpolation improvements** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **string interpolation improvements** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **string interpolation improvements**.

### Drill 22 — how to discuss modern Swift honestly
- **Definition prompt:** Explain what **how to discuss modern Swift honestly** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **how to discuss modern Swift honestly**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **how to discuss modern Swift honestly**, not just the spelling.
- **Production example to mention:** Connect **how to discuss modern Swift honestly** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **how to discuss modern Swift honestly** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **how to discuss modern Swift honestly**.
- **One-line close:** End with a concise summary that tells the interviewer why **how to discuss modern Swift honestly** matters in production code.

### Follow-up 22 — how to discuss modern Swift honestly
- **Compare prompt:** Contrast **how to discuss modern Swift honestly** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **how to discuss modern Swift honestly** is misunderstood.
- **Code review angle:** Describe what misuse of **how to discuss modern Swift honestly** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **how to discuss modern Swift honestly** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **how to discuss modern Swift honestly**.

