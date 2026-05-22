# 10 — Error Handling and `Result`

Swift gives you multiple ways to model failure. Choose based on how the caller will use the outcome.

## `Error`, throwing functions, and `do/catch`
### Example
```swift
enum APIError: Error {
    case invalidResponse
    case unauthorized
    case decodingFailed
}

func parseStatus(_ code: Int) throws {
    guard code == 200 else { throw APIError.invalidResponse }
}

do {
    try parseStatus(500)
} catch {
    print(error)
}
```

### Interview questions
- When should a function throw?
- Why is `throws` different from returning an optional?
- How do you design useful error types?

## `try`, `try?`, `try!`, and `defer`
### Example
```swift
func readConfig() {
    let lock = NSLock()
    lock.lock()
    defer { lock.unlock() }

    let url = URL(fileURLWithPath: "config.json")
    let data = try? Data(contentsOf: url)
    print(data as Any)
}
```

> 💡 **Tip:** Use `try?` only when all failures collapse to the same meaning.

## `rethrows` and `async throws`
### Example
```swift
func performTwice(_ work: () throws -> Void) rethrows {
    try work()
    try work()
}

func fetchUser() async throws -> String {
    "Taylor"
}
```

## `Result` and when to use it
### Example
```swift
enum UploadError: Error {
    case offline
    case server
}

func completeUpload(_ completion: (Result<URL, UploadError>) -> Void) {
    completion(.success(URL(string: "https://example.com")!))
}
```

### Senior-level discussion
- Use `throws` for procedural control-flow propagation.
- Use `Result` when the outcome must be stored, passed, or transformed as data.
- Both coexist naturally in modern async code.

## Topic-specific oral drills
Use these prompts to turn **error handling and result** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — `Error` protocol
- **Definition prompt:** Explain what **`Error` protocol** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`Error` protocol**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`Error` protocol**, not just the spelling.
- **Production example to mention:** Connect **`Error` protocol** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`Error` protocol** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`Error` protocol**.
- **One-line close:** End with a concise summary that tells the interviewer why **`Error` protocol** matters in production code.

### Follow-up 1 — `Error` protocol
- **Compare prompt:** Contrast **`Error` protocol** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`Error` protocol** is misunderstood.
- **Code review angle:** Describe what misuse of **`Error` protocol** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`Error` protocol** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`Error` protocol**.

### Drill 2 — throwing functions
- **Definition prompt:** Explain what **throwing functions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **throwing functions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **throwing functions**, not just the spelling.
- **Production example to mention:** Connect **throwing functions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **throwing functions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **throwing functions**.
- **One-line close:** End with a concise summary that tells the interviewer why **throwing functions** matters in production code.

### Follow-up 2 — throwing functions
- **Compare prompt:** Contrast **throwing functions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **throwing functions** is misunderstood.
- **Code review angle:** Describe what misuse of **throwing functions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **throwing functions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **throwing functions**.

### Drill 3 — recoverable vs unrecoverable failure
- **Definition prompt:** Explain what **recoverable vs unrecoverable failure** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **recoverable vs unrecoverable failure**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **recoverable vs unrecoverable failure**, not just the spelling.
- **Production example to mention:** Connect **recoverable vs unrecoverable failure** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **recoverable vs unrecoverable failure** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **recoverable vs unrecoverable failure**.
- **One-line close:** End with a concise summary that tells the interviewer why **recoverable vs unrecoverable failure** matters in production code.

### Follow-up 3 — recoverable vs unrecoverable failure
- **Compare prompt:** Contrast **recoverable vs unrecoverable failure** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **recoverable vs unrecoverable failure** is misunderstood.
- **Code review angle:** Describe what misuse of **recoverable vs unrecoverable failure** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **recoverable vs unrecoverable failure** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **recoverable vs unrecoverable failure**.

### Drill 4 — `do/catch`
- **Definition prompt:** Explain what **`do/catch`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`do/catch`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`do/catch`**, not just the spelling.
- **Production example to mention:** Connect **`do/catch`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`do/catch`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`do/catch`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`do/catch`** matters in production code.

### Follow-up 4 — `do/catch`
- **Compare prompt:** Contrast **`do/catch`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`do/catch`** is misunderstood.
- **Code review angle:** Describe what misuse of **`do/catch`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`do/catch`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`do/catch`**.

### Drill 5 — `try` propagation
- **Definition prompt:** Explain what **`try` propagation** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`try` propagation**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`try` propagation**, not just the spelling.
- **Production example to mention:** Connect **`try` propagation** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`try` propagation** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`try` propagation**.
- **One-line close:** End with a concise summary that tells the interviewer why **`try` propagation** matters in production code.

### Follow-up 5 — `try` propagation
- **Compare prompt:** Contrast **`try` propagation** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`try` propagation** is misunderstood.
- **Code review angle:** Describe what misuse of **`try` propagation** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`try` propagation** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`try` propagation**.

### Drill 6 — `try?` optional conversion
- **Definition prompt:** Explain what **`try?` optional conversion** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`try?` optional conversion**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`try?` optional conversion**, not just the spelling.
- **Production example to mention:** Connect **`try?` optional conversion** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`try?` optional conversion** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`try?` optional conversion**.
- **One-line close:** End with a concise summary that tells the interviewer why **`try?` optional conversion** matters in production code.

### Follow-up 6 — `try?` optional conversion
- **Compare prompt:** Contrast **`try?` optional conversion** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`try?` optional conversion** is misunderstood.
- **Code review angle:** Describe what misuse of **`try?` optional conversion** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`try?` optional conversion** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`try?` optional conversion**.

### Drill 7 — `try!` as assertion
- **Definition prompt:** Explain what **`try!` as assertion** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`try!` as assertion**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`try!` as assertion**, not just the spelling.
- **Production example to mention:** Connect **`try!` as assertion** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`try!` as assertion** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`try!` as assertion**.
- **One-line close:** End with a concise summary that tells the interviewer why **`try!` as assertion** matters in production code.

### Follow-up 7 — `try!` as assertion
- **Compare prompt:** Contrast **`try!` as assertion** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`try!` as assertion** is misunderstood.
- **Code review angle:** Describe what misuse of **`try!` as assertion** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`try!` as assertion** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`try!` as assertion**.

### Drill 8 — `defer` cleanup
- **Definition prompt:** Explain what **`defer` cleanup** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`defer` cleanup**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`defer` cleanup**, not just the spelling.
- **Production example to mention:** Connect **`defer` cleanup** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`defer` cleanup** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`defer` cleanup**.
- **One-line close:** End with a concise summary that tells the interviewer why **`defer` cleanup** matters in production code.

### Follow-up 8 — `defer` cleanup
- **Compare prompt:** Contrast **`defer` cleanup** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`defer` cleanup** is misunderstood.
- **Code review angle:** Describe what misuse of **`defer` cleanup** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`defer` cleanup** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`defer` cleanup**.

### Drill 9 — failable init vs throws
- **Definition prompt:** Explain what **failable init vs throws** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **failable init vs throws**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **failable init vs throws**, not just the spelling.
- **Production example to mention:** Connect **failable init vs throws** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **failable init vs throws** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **failable init vs throws**.
- **One-line close:** End with a concise summary that tells the interviewer why **failable init vs throws** matters in production code.

### Follow-up 9 — failable init vs throws
- **Compare prompt:** Contrast **failable init vs throws** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **failable init vs throws** is misunderstood.
- **Code review angle:** Describe what misuse of **failable init vs throws** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **failable init vs throws** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **failable init vs throws**.

### Drill 10 — domain-specific error types
- **Definition prompt:** Explain what **domain-specific error types** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **domain-specific error types**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **domain-specific error types**, not just the spelling.
- **Production example to mention:** Connect **domain-specific error types** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **domain-specific error types** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **domain-specific error types**.
- **One-line close:** End with a concise summary that tells the interviewer why **domain-specific error types** matters in production code.

### Follow-up 10 — domain-specific error types
- **Compare prompt:** Contrast **domain-specific error types** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **domain-specific error types** is misunderstood.
- **Code review angle:** Describe what misuse of **domain-specific error types** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **domain-specific error types** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **domain-specific error types**.

### Drill 11 — error context quality
- **Definition prompt:** Explain what **error context quality** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **error context quality**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **error context quality**, not just the spelling.
- **Production example to mention:** Connect **error context quality** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **error context quality** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **error context quality**.
- **One-line close:** End with a concise summary that tells the interviewer why **error context quality** matters in production code.

### Follow-up 11 — error context quality
- **Compare prompt:** Contrast **error context quality** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **error context quality** is misunderstood.
- **Code review angle:** Describe what misuse of **error context quality** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **error context quality** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **error context quality**.

### Drill 12 — `rethrows`
- **Definition prompt:** Explain what **`rethrows`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`rethrows`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`rethrows`**, not just the spelling.
- **Production example to mention:** Connect **`rethrows`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`rethrows`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`rethrows`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`rethrows`** matters in production code.

### Follow-up 12 — `rethrows`
- **Compare prompt:** Contrast **`rethrows`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`rethrows`** is misunderstood.
- **Code review angle:** Describe what misuse of **`rethrows`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`rethrows`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`rethrows`**.

### Drill 13 — `async throws`
- **Definition prompt:** Explain what **`async throws`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`async throws`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`async throws`**, not just the spelling.
- **Production example to mention:** Connect **`async throws`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`async throws`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`async throws`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`async throws`** matters in production code.

### Follow-up 13 — `async throws`
- **Compare prompt:** Contrast **`async throws`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`async throws`** is misunderstood.
- **Code review angle:** Describe what misuse of **`async throws`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`async throws`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`async throws`**.

### Drill 14 — `Result` as data
- **Definition prompt:** Explain what **`Result` as data** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`Result` as data**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`Result` as data**, not just the spelling.
- **Production example to mention:** Connect **`Result` as data** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`Result` as data** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`Result` as data**.
- **One-line close:** End with a concise summary that tells the interviewer why **`Result` as data** matters in production code.

### Follow-up 14 — `Result` as data
- **Compare prompt:** Contrast **`Result` as data** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`Result` as data** is misunderstood.
- **Code review angle:** Describe what misuse of **`Result` as data** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`Result` as data** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`Result` as data**.

### Drill 15 — `throws` vs `Result`
- **Definition prompt:** Explain what **`throws` vs `Result`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`throws` vs `Result`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`throws` vs `Result`**, not just the spelling.
- **Production example to mention:** Connect **`throws` vs `Result`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`throws` vs `Result`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`throws` vs `Result`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`throws` vs `Result`** matters in production code.

### Follow-up 15 — `throws` vs `Result`
- **Compare prompt:** Contrast **`throws` vs `Result`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`throws` vs `Result`** is misunderstood.
- **Code review angle:** Describe what misuse of **`throws` vs `Result`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`throws` vs `Result`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`throws` vs `Result`**.

### Drill 16 — callbacks with `Result`
- **Definition prompt:** Explain what **callbacks with `Result`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **callbacks with `Result`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **callbacks with `Result`**, not just the spelling.
- **Production example to mention:** Connect **callbacks with `Result`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **callbacks with `Result`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **callbacks with `Result`**.
- **One-line close:** End with a concise summary that tells the interviewer why **callbacks with `Result`** matters in production code.

### Follow-up 16 — callbacks with `Result`
- **Compare prompt:** Contrast **callbacks with `Result`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **callbacks with `Result`** is misunderstood.
- **Code review angle:** Describe what misuse of **callbacks with `Result`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **callbacks with `Result`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **callbacks with `Result`**.

### Drill 17 — testing errors
- **Definition prompt:** Explain what **testing errors** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **testing errors**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **testing errors**, not just the spelling.
- **Production example to mention:** Connect **testing errors** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **testing errors** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **testing errors**.
- **One-line close:** End with a concise summary that tells the interviewer why **testing errors** matters in production code.

### Follow-up 17 — testing errors
- **Compare prompt:** Contrast **testing errors** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **testing errors** is misunderstood.
- **Code review angle:** Describe what misuse of **testing errors** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **testing errors** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **testing errors**.

### Drill 18 — propagating versus handling
- **Definition prompt:** Explain what **propagating versus handling** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **propagating versus handling**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **propagating versus handling**, not just the spelling.
- **Production example to mention:** Connect **propagating versus handling** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **propagating versus handling** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **propagating versus handling**.
- **One-line close:** End with a concise summary that tells the interviewer why **propagating versus handling** matters in production code.

### Follow-up 18 — propagating versus handling
- **Compare prompt:** Contrast **propagating versus handling** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **propagating versus handling** is misunderstood.
- **Code review angle:** Describe what misuse of **propagating versus handling** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **propagating versus handling** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **propagating versus handling**.

### Drill 19 — user-facing versus logging errors
- **Definition prompt:** Explain what **user-facing versus logging errors** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **user-facing versus logging errors**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **user-facing versus logging errors**, not just the spelling.
- **Production example to mention:** Connect **user-facing versus logging errors** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **user-facing versus logging errors** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **user-facing versus logging errors**.
- **One-line close:** End with a concise summary that tells the interviewer why **user-facing versus logging errors** matters in production code.

### Follow-up 19 — user-facing versus logging errors
- **Compare prompt:** Contrast **user-facing versus logging errors** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **user-facing versus logging errors** is misunderstood.
- **Code review angle:** Describe what misuse of **user-facing versus logging errors** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **user-facing versus logging errors** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **user-facing versus logging errors**.

### Drill 20 — boundary error mapping
- **Definition prompt:** Explain what **boundary error mapping** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **boundary error mapping**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **boundary error mapping**, not just the spelling.
- **Production example to mention:** Connect **boundary error mapping** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **boundary error mapping** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **boundary error mapping**.
- **One-line close:** End with a concise summary that tells the interviewer why **boundary error mapping** matters in production code.

### Follow-up 20 — boundary error mapping
- **Compare prompt:** Contrast **boundary error mapping** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **boundary error mapping** is misunderstood.
- **Code review angle:** Describe what misuse of **boundary error mapping** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **boundary error mapping** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **boundary error mapping**.

### Drill 21 — senior error-design tradeoffs
- **Definition prompt:** Explain what **senior error-design tradeoffs** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **senior error-design tradeoffs**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **senior error-design tradeoffs**, not just the spelling.
- **Production example to mention:** Connect **senior error-design tradeoffs** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **senior error-design tradeoffs** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **senior error-design tradeoffs**.
- **One-line close:** End with a concise summary that tells the interviewer why **senior error-design tradeoffs** matters in production code.

### Follow-up 21 — senior error-design tradeoffs
- **Compare prompt:** Contrast **senior error-design tradeoffs** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **senior error-design tradeoffs** is misunderstood.
- **Code review angle:** Describe what misuse of **senior error-design tradeoffs** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **senior error-design tradeoffs** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **senior error-design tradeoffs**.

### Drill 22 — failure modeling in APIs
- **Definition prompt:** Explain what **failure modeling in APIs** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **failure modeling in APIs**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **failure modeling in APIs**, not just the spelling.
- **Production example to mention:** Connect **failure modeling in APIs** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **failure modeling in APIs** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **failure modeling in APIs**.
- **One-line close:** End with a concise summary that tells the interviewer why **failure modeling in APIs** matters in production code.

### Follow-up 22 — failure modeling in APIs
- **Compare prompt:** Contrast **failure modeling in APIs** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **failure modeling in APIs** is misunderstood.
- **Code review angle:** Describe what misuse of **failure modeling in APIs** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **failure modeling in APIs** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **failure modeling in APIs**.

