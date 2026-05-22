# 12 — Codable and Extensions

Codable is powerful on the happy path, but interviews often focus on what happens once the payload gets messy.

## Synthesized Codable
### Example
```swift
struct UserDTO: Codable {
    let id: Int
    let fullName: String
    let isPremium: Bool
}
```

## CodingKeys, strategies, and API issues
### Example
```swift
struct ArticleDTO: Decodable {
    let title: String
    let publishedAt: Date

    enum CodingKeys: String, CodingKey {
        case title
        case publishedAt = "published_at"
    }
}

let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
decoder.dateDecodingStrategy = .iso8601
```

## Custom decoding and nested containers
### Example
```swift
struct UserSummary: Decodable {
    let id: Int
    let city: String

    enum CodingKeys: String, CodingKey {
        case id
        case address
    }

    enum AddressKeys: String, CodingKey {
        case city
    }

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        id = try container.decode(Int.self, forKey: .id)
        let address = try container.nestedContainer(keyedBy: AddressKeys.self, forKey: .address)
        city = try address.decode(String.self, forKey: .city)
    }
}
```

## Extensions and subscripts
### Example
```swift
struct ScoreBoard {
    private var scores: [String: Int] = [:]

    subscript(player name: String) -> Int {
        get { scores[name, default: 0] }
        set { scores[name] = newValue }
    }
}

extension Array where Element: Hashable {
    func duplicatesRemoved() -> [Element] {
        var seen = Set<Element>()
        return filter { seen.insert($0).inserted }
    }
}
```

### Quick recall
- Use DTOs at the boundary if the wire format is messy.
- Extensions add behavior and conformance, not stored properties.
- Subscripts should feel naturally index-like.

## Topic-specific oral drills
Use these prompts to turn **codable and extensions** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — `Codable` synthesis
- **Definition prompt:** Explain what **`Codable` synthesis** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`Codable` synthesis**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`Codable` synthesis**, not just the spelling.
- **Production example to mention:** Connect **`Codable` synthesis** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`Codable` synthesis** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`Codable` synthesis**.
- **One-line close:** End with a concise summary that tells the interviewer why **`Codable` synthesis** matters in production code.

### Follow-up 1 — `Codable` synthesis
- **Compare prompt:** Contrast **`Codable` synthesis** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`Codable` synthesis** is misunderstood.
- **Code review angle:** Describe what misuse of **`Codable` synthesis** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`Codable` synthesis** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`Codable` synthesis**.

### Drill 2 — `Encodable` vs `Decodable`
- **Definition prompt:** Explain what **`Encodable` vs `Decodable`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`Encodable` vs `Decodable`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`Encodable` vs `Decodable`**, not just the spelling.
- **Production example to mention:** Connect **`Encodable` vs `Decodable`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`Encodable` vs `Decodable`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`Encodable` vs `Decodable`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`Encodable` vs `Decodable`** matters in production code.

### Follow-up 2 — `Encodable` vs `Decodable`
- **Compare prompt:** Contrast **`Encodable` vs `Decodable`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`Encodable` vs `Decodable`** is misunderstood.
- **Code review angle:** Describe what misuse of **`Encodable` vs `Decodable`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`Encodable` vs `Decodable`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`Encodable` vs `Decodable`**.

### Drill 3 — DTO vs domain model
- **Definition prompt:** Explain what **DTO vs domain model** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **DTO vs domain model**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **DTO vs domain model**, not just the spelling.
- **Production example to mention:** Connect **DTO vs domain model** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **DTO vs domain model** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **DTO vs domain model**.
- **One-line close:** End with a concise summary that tells the interviewer why **DTO vs domain model** matters in production code.

### Follow-up 3 — DTO vs domain model
- **Compare prompt:** Contrast **DTO vs domain model** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **DTO vs domain model** is misunderstood.
- **Code review angle:** Describe what misuse of **DTO vs domain model** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **DTO vs domain model** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **DTO vs domain model**.

### Drill 4 — `CodingKeys` mapping
- **Definition prompt:** Explain what **`CodingKeys` mapping** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`CodingKeys` mapping**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`CodingKeys` mapping**, not just the spelling.
- **Production example to mention:** Connect **`CodingKeys` mapping** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`CodingKeys` mapping** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`CodingKeys` mapping**.
- **One-line close:** End with a concise summary that tells the interviewer why **`CodingKeys` mapping** matters in production code.

### Follow-up 4 — `CodingKeys` mapping
- **Compare prompt:** Contrast **`CodingKeys` mapping** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`CodingKeys` mapping** is misunderstood.
- **Code review angle:** Describe what misuse of **`CodingKeys` mapping** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`CodingKeys` mapping** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`CodingKeys` mapping**.

### Drill 5 — snake_case decoding
- **Definition prompt:** Explain what **snake_case decoding** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **snake_case decoding**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **snake_case decoding**, not just the spelling.
- **Production example to mention:** Connect **snake_case decoding** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **snake_case decoding** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **snake_case decoding**.
- **One-line close:** End with a concise summary that tells the interviewer why **snake_case decoding** matters in production code.

### Follow-up 5 — snake_case decoding
- **Compare prompt:** Contrast **snake_case decoding** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **snake_case decoding** is misunderstood.
- **Code review angle:** Describe what misuse of **snake_case decoding** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **snake_case decoding** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **snake_case decoding**.

### Drill 6 — date decoding strategies
- **Definition prompt:** Explain what **date decoding strategies** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **date decoding strategies**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **date decoding strategies**, not just the spelling.
- **Production example to mention:** Connect **date decoding strategies** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **date decoding strategies** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **date decoding strategies**.
- **One-line close:** End with a concise summary that tells the interviewer why **date decoding strategies** matters in production code.

### Follow-up 6 — date decoding strategies
- **Compare prompt:** Contrast **date decoding strategies** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **date decoding strategies** is misunderstood.
- **Code review angle:** Describe what misuse of **date decoding strategies** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **date decoding strategies** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **date decoding strategies**.

### Drill 7 — custom `init(from:)`
- **Definition prompt:** Explain what **custom `init(from:)`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **custom `init(from:)`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **custom `init(from:)`**, not just the spelling.
- **Production example to mention:** Connect **custom `init(from:)`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **custom `init(from:)`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **custom `init(from:)`**.
- **One-line close:** End with a concise summary that tells the interviewer why **custom `init(from:)`** matters in production code.

### Follow-up 7 — custom `init(from:)`
- **Compare prompt:** Contrast **custom `init(from:)`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **custom `init(from:)`** is misunderstood.
- **Code review angle:** Describe what misuse of **custom `init(from:)`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **custom `init(from:)`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **custom `init(from:)`**.

### Drill 8 — custom `encode(to:)`
- **Definition prompt:** Explain what **custom `encode(to:)`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **custom `encode(to:)`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **custom `encode(to:)`**, not just the spelling.
- **Production example to mention:** Connect **custom `encode(to:)`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **custom `encode(to:)`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **custom `encode(to:)`**.
- **One-line close:** End with a concise summary that tells the interviewer why **custom `encode(to:)`** matters in production code.

### Follow-up 8 — custom `encode(to:)`
- **Compare prompt:** Contrast **custom `encode(to:)`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **custom `encode(to:)`** is misunderstood.
- **Code review angle:** Describe what misuse of **custom `encode(to:)`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **custom `encode(to:)`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **custom `encode(to:)`**.

### Drill 9 — nested containers
- **Definition prompt:** Explain what **nested containers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **nested containers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **nested containers**, not just the spelling.
- **Production example to mention:** Connect **nested containers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **nested containers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **nested containers**.
- **One-line close:** End with a concise summary that tells the interviewer why **nested containers** matters in production code.

### Follow-up 9 — nested containers
- **Compare prompt:** Contrast **nested containers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **nested containers** is misunderstood.
- **Code review angle:** Describe what misuse of **nested containers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **nested containers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **nested containers**.

### Drill 10 — keyed vs unkeyed containers
- **Definition prompt:** Explain what **keyed vs unkeyed containers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **keyed vs unkeyed containers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **keyed vs unkeyed containers**, not just the spelling.
- **Production example to mention:** Connect **keyed vs unkeyed containers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **keyed vs unkeyed containers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **keyed vs unkeyed containers**.
- **One-line close:** End with a concise summary that tells the interviewer why **keyed vs unkeyed containers** matters in production code.

### Follow-up 10 — keyed vs unkeyed containers
- **Compare prompt:** Contrast **keyed vs unkeyed containers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **keyed vs unkeyed containers** is misunderstood.
- **Code review angle:** Describe what misuse of **keyed vs unkeyed containers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **keyed vs unkeyed containers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **keyed vs unkeyed containers**.

### Drill 11 — partial backend data
- **Definition prompt:** Explain what **partial backend data** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **partial backend data**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **partial backend data**, not just the spelling.
- **Production example to mention:** Connect **partial backend data** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **partial backend data** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **partial backend data**.
- **One-line close:** End with a concise summary that tells the interviewer why **partial backend data** matters in production code.

### Follow-up 11 — partial backend data
- **Compare prompt:** Contrast **partial backend data** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **partial backend data** is misunderstood.
- **Code review angle:** Describe what misuse of **partial backend data** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **partial backend data** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **partial backend data**.

### Drill 12 — normalization during decoding
- **Definition prompt:** Explain what **normalization during decoding** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **normalization during decoding**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **normalization during decoding**, not just the spelling.
- **Production example to mention:** Connect **normalization during decoding** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **normalization during decoding** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **normalization during decoding**.
- **One-line close:** End with a concise summary that tells the interviewer why **normalization during decoding** matters in production code.

### Follow-up 12 — normalization during decoding
- **Compare prompt:** Contrast **normalization during decoding** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **normalization during decoding** is misunderstood.
- **Code review angle:** Describe what misuse of **normalization during decoding** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **normalization during decoding** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **normalization during decoding**.

### Drill 13 — failure debugging for payloads
- **Definition prompt:** Explain what **failure debugging for payloads** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **failure debugging for payloads**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **failure debugging for payloads**, not just the spelling.
- **Production example to mention:** Connect **failure debugging for payloads** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **failure debugging for payloads** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **failure debugging for payloads**.
- **One-line close:** End with a concise summary that tells the interviewer why **failure debugging for payloads** matters in production code.

### Follow-up 13 — failure debugging for payloads
- **Compare prompt:** Contrast **failure debugging for payloads** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **failure debugging for payloads** is misunderstood.
- **Code review angle:** Describe what misuse of **failure debugging for payloads** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **failure debugging for payloads** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **failure debugging for payloads**.

### Drill 14 — extension methods
- **Definition prompt:** Explain what **extension methods** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **extension methods**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **extension methods**, not just the spelling.
- **Production example to mention:** Connect **extension methods** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **extension methods** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **extension methods**.
- **One-line close:** End with a concise summary that tells the interviewer why **extension methods** matters in production code.

### Follow-up 14 — extension methods
- **Compare prompt:** Contrast **extension methods** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **extension methods** is misunderstood.
- **Code review angle:** Describe what misuse of **extension methods** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **extension methods** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **extension methods**.

### Drill 15 — extension-based conformance
- **Definition prompt:** Explain what **extension-based conformance** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **extension-based conformance**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **extension-based conformance**, not just the spelling.
- **Production example to mention:** Connect **extension-based conformance** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **extension-based conformance** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **extension-based conformance**.
- **One-line close:** End with a concise summary that tells the interviewer why **extension-based conformance** matters in production code.

### Follow-up 15 — extension-based conformance
- **Compare prompt:** Contrast **extension-based conformance** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **extension-based conformance** is misunderstood.
- **Code review angle:** Describe what misuse of **extension-based conformance** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **extension-based conformance** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **extension-based conformance**.

### Drill 16 — computed properties in extensions
- **Definition prompt:** Explain what **computed properties in extensions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **computed properties in extensions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **computed properties in extensions**, not just the spelling.
- **Production example to mention:** Connect **computed properties in extensions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **computed properties in extensions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **computed properties in extensions**.
- **One-line close:** End with a concise summary that tells the interviewer why **computed properties in extensions** matters in production code.

### Follow-up 16 — computed properties in extensions
- **Compare prompt:** Contrast **computed properties in extensions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **computed properties in extensions** is misunderstood.
- **Code review angle:** Describe what misuse of **computed properties in extensions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **computed properties in extensions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **computed properties in extensions**.

### Drill 17 — limitations of extensions
- **Definition prompt:** Explain what **limitations of extensions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **limitations of extensions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **limitations of extensions**, not just the spelling.
- **Production example to mention:** Connect **limitations of extensions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **limitations of extensions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **limitations of extensions**.
- **One-line close:** End with a concise summary that tells the interviewer why **limitations of extensions** matters in production code.

### Follow-up 17 — limitations of extensions
- **Compare prompt:** Contrast **limitations of extensions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **limitations of extensions** is misunderstood.
- **Code review angle:** Describe what misuse of **limitations of extensions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **limitations of extensions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **limitations of extensions**.

### Drill 18 — no stored properties in extensions
- **Definition prompt:** Explain what **no stored properties in extensions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **no stored properties in extensions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **no stored properties in extensions**, not just the spelling.
- **Production example to mention:** Connect **no stored properties in extensions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **no stored properties in extensions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **no stored properties in extensions**.
- **One-line close:** End with a concise summary that tells the interviewer why **no stored properties in extensions** matters in production code.

### Follow-up 18 — no stored properties in extensions
- **Compare prompt:** Contrast **no stored properties in extensions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **no stored properties in extensions** is misunderstood.
- **Code review angle:** Describe what misuse of **no stored properties in extensions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **no stored properties in extensions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **no stored properties in extensions**.

### Drill 19 — retroactive modeling
- **Definition prompt:** Explain what **retroactive modeling** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **retroactive modeling**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **retroactive modeling**, not just the spelling.
- **Production example to mention:** Connect **retroactive modeling** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **retroactive modeling** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **retroactive modeling**.
- **One-line close:** End with a concise summary that tells the interviewer why **retroactive modeling** matters in production code.

### Follow-up 19 — retroactive modeling
- **Compare prompt:** Contrast **retroactive modeling** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **retroactive modeling** is misunderstood.
- **Code review angle:** Describe what misuse of **retroactive modeling** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **retroactive modeling** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **retroactive modeling**.

### Drill 20 — subscripts
- **Definition prompt:** Explain what **subscripts** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **subscripts**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **subscripts**, not just the spelling.
- **Production example to mention:** Connect **subscripts** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **subscripts** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **subscripts**.
- **One-line close:** End with a concise summary that tells the interviewer why **subscripts** matters in production code.

### Follow-up 20 — subscripts
- **Compare prompt:** Contrast **subscripts** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **subscripts** is misunderstood.
- **Code review angle:** Describe what misuse of **subscripts** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **subscripts** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **subscripts**.

### Drill 21 — index-like API design
- **Definition prompt:** Explain what **index-like API design** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **index-like API design**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **index-like API design**, not just the spelling.
- **Production example to mention:** Connect **index-like API design** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **index-like API design** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **index-like API design**.
- **One-line close:** End with a concise summary that tells the interviewer why **index-like API design** matters in production code.

### Follow-up 21 — index-like API design
- **Compare prompt:** Contrast **index-like API design** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **index-like API design** is misunderstood.
- **Code review angle:** Describe what misuse of **index-like API design** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **index-like API design** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **index-like API design**.

### Drill 22 — clean file organization with extensions
- **Definition prompt:** Explain what **clean file organization with extensions** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **clean file organization with extensions**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **clean file organization with extensions**, not just the spelling.
- **Production example to mention:** Connect **clean file organization with extensions** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **clean file organization with extensions** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **clean file organization with extensions**.
- **One-line close:** End with a concise summary that tells the interviewer why **clean file organization with extensions** matters in production code.

### Follow-up 22 — clean file organization with extensions
- **Compare prompt:** Contrast **clean file organization with extensions** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **clean file organization with extensions** is misunderstood.
- **Code review angle:** Describe what misuse of **clean file organization with extensions** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **clean file organization with extensions** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **clean file organization with extensions**.

