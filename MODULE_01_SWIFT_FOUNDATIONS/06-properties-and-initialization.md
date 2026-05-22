# 06 — Properties and Initialization

Property design and initialization rules are where Swift's safety guarantees become concrete.

## Stored, computed, lazy, static, and class properties
Each property flavor communicates a different storage and lifetime story.

### Example
```swift
struct Config {
    static let sharedTimeout = 30

    var firstName: String
    var lastName: String
    lazy var formatter = PersonNameComponentsFormatter()

    var fullName: String {
        "\(firstName) \(lastName)"
    }
}
```

### Common pitfalls
- ⚠️ Hiding expensive work in computed properties.
- ⚠️ Assuming `lazy` is automatically thread-safe.
- ⚠️ Treating `static` mutable state like harmless global data.

### Interview questions
- When is a computed property better than a method?
- What are the risks of `lazy`?
- What is the difference between `static` and `class`?

## Property observers and property wrappers
Observers react to changes; wrappers package storage behavior behind a reusable attribute.

### Example
```swift
@propertyWrapper
struct Clamped {
    private var value: Int
    let range: ClosedRange<Int>

    init(wrappedValue: Int, _ range: ClosedRange<Int>) {
        self.range = range
        self.value = min(max(wrappedValue, range.lowerBound), range.upperBound)
    }

    var wrappedValue: Int {
        get { value }
        set { value = min(max(newValue, range.lowerBound), range.upperBound) }
    }
}
```

> 💡 **Tip:** Use wrappers for reusable policy, not for hiding essential domain behavior.

## Struct initialization and memberwise init
Structs get great initializer ergonomics.

### Example
```swift
struct UserProfile {
    let id: UUID
    var displayName: String
    var age: Int

    init(id: UUID = UUID(), displayName: String, age: Int) {
        self.id = id
        self.displayName = displayName
        self.age = max(age, 0)
    }
}
```

### Senior-level discussion
- Initializers should enforce invariants as early as possible.
- Default values improve ergonomics when they encode a real default.
- Strong boundary models remove optionality from later code.

## Class initialization: designated, convenience, required, and failable init
Class initialization is more complex because inheritance is involved.

### Example
```swift
class Vehicle {
    let wheels: Int
    init(wheels: Int) { self.wheels = wheels }
}

class Car: Vehicle {
    let brand: String

    init?(brand: String, wheels: Int) {
        guard !brand.isEmpty else { return nil }
        self.brand = brand
        super.init(wheels: wheels)
    }

    convenience init?(brand: String) {
        self.init(brand: brand, wheels: 4)
    }
}
```

### Common pitfalls
- ⚠️ Mixing up designated and convenience delegation rules.
- ⚠️ Using `init?` when detailed errors would be better.
- ⚠️ Treating required initializers as meaningless boilerplate.

## Two-phase initialization and `deinit`
Swift guarantees fully initialized state before unsafe use.

### Example
```swift
class FileHandleWrapper {
    private let path: String

    init(path: String) {
        self.path = path
        print("Open file at \(path)")
    }

    deinit {
        print("Close file at \(path)")
    }
}
```

> 🎯 **Interview Answer:** Two-phase initialization exists so subclass customization cannot touch incompletely initialized state.

## Topic-specific oral drills
Use these prompts to turn **properties and initialization** into interview-ready speaking material. Each drill is intentionally specific so you can rehearse definitions, tradeoffs, and production consequences without relying on generic filler.

### Drill 1 — stored properties
- **Definition prompt:** Explain what **stored properties** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **stored properties**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **stored properties**, not just the spelling.
- **Production example to mention:** Connect **stored properties** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **stored properties** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **stored properties**.
- **One-line close:** End with a concise summary that tells the interviewer why **stored properties** matters in production code.

### Follow-up 1 — stored properties
- **Compare prompt:** Contrast **stored properties** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **stored properties** is misunderstood.
- **Code review angle:** Describe what misuse of **stored properties** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **stored properties** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **stored properties**.

### Drill 2 — computed properties
- **Definition prompt:** Explain what **computed properties** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **computed properties**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **computed properties**, not just the spelling.
- **Production example to mention:** Connect **computed properties** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **computed properties** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **computed properties**.
- **One-line close:** End with a concise summary that tells the interviewer why **computed properties** matters in production code.

### Follow-up 2 — computed properties
- **Compare prompt:** Contrast **computed properties** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **computed properties** is misunderstood.
- **Code review angle:** Describe what misuse of **computed properties** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **computed properties** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **computed properties**.

### Drill 3 — cost visibility of computed values
- **Definition prompt:** Explain what **cost visibility of computed values** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **cost visibility of computed values**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **cost visibility of computed values**, not just the spelling.
- **Production example to mention:** Connect **cost visibility of computed values** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **cost visibility of computed values** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **cost visibility of computed values**.
- **One-line close:** End with a concise summary that tells the interviewer why **cost visibility of computed values** matters in production code.

### Follow-up 3 — cost visibility of computed values
- **Compare prompt:** Contrast **cost visibility of computed values** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **cost visibility of computed values** is misunderstood.
- **Code review angle:** Describe what misuse of **cost visibility of computed values** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **cost visibility of computed values** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **cost visibility of computed values**.

### Drill 4 — lazy properties
- **Definition prompt:** Explain what **lazy properties** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **lazy properties**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **lazy properties**, not just the spelling.
- **Production example to mention:** Connect **lazy properties** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **lazy properties** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **lazy properties**.
- **One-line close:** End with a concise summary that tells the interviewer why **lazy properties** matters in production code.

### Follow-up 4 — lazy properties
- **Compare prompt:** Contrast **lazy properties** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **lazy properties** is misunderstood.
- **Code review angle:** Describe what misuse of **lazy properties** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **lazy properties** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **lazy properties**.

### Drill 5 — static properties
- **Definition prompt:** Explain what **static properties** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **static properties**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **static properties**, not just the spelling.
- **Production example to mention:** Connect **static properties** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **static properties** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **static properties**.
- **One-line close:** End with a concise summary that tells the interviewer why **static properties** matters in production code.

### Follow-up 5 — static properties
- **Compare prompt:** Contrast **static properties** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **static properties** is misunderstood.
- **Code review angle:** Describe what misuse of **static properties** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **static properties** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **static properties**.

### Drill 6 — class properties
- **Definition prompt:** Explain what **class properties** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **class properties**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **class properties**, not just the spelling.
- **Production example to mention:** Connect **class properties** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **class properties** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **class properties**.
- **One-line close:** End with a concise summary that tells the interviewer why **class properties** matters in production code.

### Follow-up 6 — class properties
- **Compare prompt:** Contrast **class properties** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **class properties** is misunderstood.
- **Code review angle:** Describe what misuse of **class properties** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **class properties** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **class properties**.

### Drill 7 — property observers
- **Definition prompt:** Explain what **property observers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **property observers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **property observers**, not just the spelling.
- **Production example to mention:** Connect **property observers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **property observers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **property observers**.
- **One-line close:** End with a concise summary that tells the interviewer why **property observers** matters in production code.

### Follow-up 7 — property observers
- **Compare prompt:** Contrast **property observers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **property observers** is misunderstood.
- **Code review angle:** Describe what misuse of **property observers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **property observers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **property observers**.

### Drill 8 — `willSet` vs `didSet`
- **Definition prompt:** Explain what **`willSet` vs `didSet`** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **`willSet` vs `didSet`**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **`willSet` vs `didSet`**, not just the spelling.
- **Production example to mention:** Connect **`willSet` vs `didSet`** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **`willSet` vs `didSet`** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **`willSet` vs `didSet`**.
- **One-line close:** End with a concise summary that tells the interviewer why **`willSet` vs `didSet`** matters in production code.

### Follow-up 8 — `willSet` vs `didSet`
- **Compare prompt:** Contrast **`willSet` vs `didSet`** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **`willSet` vs `didSet`** is misunderstood.
- **Code review angle:** Describe what misuse of **`willSet` vs `didSet`** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **`willSet` vs `didSet`** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **`willSet` vs `didSet`**.

### Drill 9 — property wrappers
- **Definition prompt:** Explain what **property wrappers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **property wrappers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **property wrappers**, not just the spelling.
- **Production example to mention:** Connect **property wrappers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **property wrappers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **property wrappers**.
- **One-line close:** End with a concise summary that tells the interviewer why **property wrappers** matters in production code.

### Follow-up 9 — property wrappers
- **Compare prompt:** Contrast **property wrappers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **property wrappers** is misunderstood.
- **Code review angle:** Describe what misuse of **property wrappers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **property wrappers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **property wrappers**.

### Drill 10 — wrapper readability tradeoffs
- **Definition prompt:** Explain what **wrapper readability tradeoffs** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **wrapper readability tradeoffs**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **wrapper readability tradeoffs**, not just the spelling.
- **Production example to mention:** Connect **wrapper readability tradeoffs** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **wrapper readability tradeoffs** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **wrapper readability tradeoffs**.
- **One-line close:** End with a concise summary that tells the interviewer why **wrapper readability tradeoffs** matters in production code.

### Follow-up 10 — wrapper readability tradeoffs
- **Compare prompt:** Contrast **wrapper readability tradeoffs** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **wrapper readability tradeoffs** is misunderstood.
- **Code review angle:** Describe what misuse of **wrapper readability tradeoffs** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **wrapper readability tradeoffs** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **wrapper readability tradeoffs**.

### Drill 11 — memberwise initialization
- **Definition prompt:** Explain what **memberwise initialization** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **memberwise initialization**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **memberwise initialization**, not just the spelling.
- **Production example to mention:** Connect **memberwise initialization** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **memberwise initialization** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **memberwise initialization**.
- **One-line close:** End with a concise summary that tells the interviewer why **memberwise initialization** matters in production code.

### Follow-up 11 — memberwise initialization
- **Compare prompt:** Contrast **memberwise initialization** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **memberwise initialization** is misunderstood.
- **Code review angle:** Describe what misuse of **memberwise initialization** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **memberwise initialization** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **memberwise initialization**.

### Drill 12 — default values in initializers
- **Definition prompt:** Explain what **default values in initializers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **default values in initializers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **default values in initializers**, not just the spelling.
- **Production example to mention:** Connect **default values in initializers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **default values in initializers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **default values in initializers**.
- **One-line close:** End with a concise summary that tells the interviewer why **default values in initializers** matters in production code.

### Follow-up 12 — default values in initializers
- **Compare prompt:** Contrast **default values in initializers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **default values in initializers** is misunderstood.
- **Code review angle:** Describe what misuse of **default values in initializers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **default values in initializers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **default values in initializers**.

### Drill 13 — custom struct initializers
- **Definition prompt:** Explain what **custom struct initializers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **custom struct initializers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **custom struct initializers**, not just the spelling.
- **Production example to mention:** Connect **custom struct initializers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **custom struct initializers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **custom struct initializers**.
- **One-line close:** End with a concise summary that tells the interviewer why **custom struct initializers** matters in production code.

### Follow-up 13 — custom struct initializers
- **Compare prompt:** Contrast **custom struct initializers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **custom struct initializers** is misunderstood.
- **Code review angle:** Describe what misuse of **custom struct initializers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **custom struct initializers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **custom struct initializers**.

### Drill 14 — designated initializers
- **Definition prompt:** Explain what **designated initializers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **designated initializers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **designated initializers**, not just the spelling.
- **Production example to mention:** Connect **designated initializers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **designated initializers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **designated initializers**.
- **One-line close:** End with a concise summary that tells the interviewer why **designated initializers** matters in production code.

### Follow-up 14 — designated initializers
- **Compare prompt:** Contrast **designated initializers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **designated initializers** is misunderstood.
- **Code review angle:** Describe what misuse of **designated initializers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **designated initializers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **designated initializers**.

### Drill 15 — convenience initializers
- **Definition prompt:** Explain what **convenience initializers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **convenience initializers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **convenience initializers**, not just the spelling.
- **Production example to mention:** Connect **convenience initializers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **convenience initializers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **convenience initializers**.
- **One-line close:** End with a concise summary that tells the interviewer why **convenience initializers** matters in production code.

### Follow-up 15 — convenience initializers
- **Compare prompt:** Contrast **convenience initializers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **convenience initializers** is misunderstood.
- **Code review angle:** Describe what misuse of **convenience initializers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **convenience initializers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **convenience initializers**.

### Drill 16 — required initializers
- **Definition prompt:** Explain what **required initializers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **required initializers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **required initializers**, not just the spelling.
- **Production example to mention:** Connect **required initializers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **required initializers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **required initializers**.
- **One-line close:** End with a concise summary that tells the interviewer why **required initializers** matters in production code.

### Follow-up 16 — required initializers
- **Compare prompt:** Contrast **required initializers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **required initializers** is misunderstood.
- **Code review angle:** Describe what misuse of **required initializers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **required initializers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **required initializers**.

### Drill 17 — failable initializers
- **Definition prompt:** Explain what **failable initializers** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **failable initializers**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **failable initializers**, not just the spelling.
- **Production example to mention:** Connect **failable initializers** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **failable initializers** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **failable initializers**.
- **One-line close:** End with a concise summary that tells the interviewer why **failable initializers** matters in production code.

### Follow-up 17 — failable initializers
- **Compare prompt:** Contrast **failable initializers** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **failable initializers** is misunderstood.
- **Code review angle:** Describe what misuse of **failable initializers** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **failable initializers** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **failable initializers**.

### Drill 18 — initializer inheritance
- **Definition prompt:** Explain what **initializer inheritance** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **initializer inheritance**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **initializer inheritance**, not just the spelling.
- **Production example to mention:** Connect **initializer inheritance** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **initializer inheritance** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **initializer inheritance**.
- **One-line close:** End with a concise summary that tells the interviewer why **initializer inheritance** matters in production code.

### Follow-up 18 — initializer inheritance
- **Compare prompt:** Contrast **initializer inheritance** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **initializer inheritance** is misunderstood.
- **Code review angle:** Describe what misuse of **initializer inheritance** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **initializer inheritance** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **initializer inheritance**.

### Drill 19 — two-phase initialization
- **Definition prompt:** Explain what **two-phase initialization** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **two-phase initialization**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **two-phase initialization**, not just the spelling.
- **Production example to mention:** Connect **two-phase initialization** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **two-phase initialization** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **two-phase initialization**.
- **One-line close:** End with a concise summary that tells the interviewer why **two-phase initialization** matters in production code.

### Follow-up 19 — two-phase initialization
- **Compare prompt:** Contrast **two-phase initialization** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **two-phase initialization** is misunderstood.
- **Code review angle:** Describe what misuse of **two-phase initialization** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **two-phase initialization** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **two-phase initialization**.

### Drill 20 — self-use during initialization
- **Definition prompt:** Explain what **self-use during initialization** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **self-use during initialization**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **self-use during initialization**, not just the spelling.
- **Production example to mention:** Connect **self-use during initialization** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **self-use during initialization** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **self-use during initialization**.
- **One-line close:** End with a concise summary that tells the interviewer why **self-use during initialization** matters in production code.

### Follow-up 20 — self-use during initialization
- **Compare prompt:** Contrast **self-use during initialization** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **self-use during initialization** is misunderstood.
- **Code review angle:** Describe what misuse of **self-use during initialization** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **self-use during initialization** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **self-use during initialization**.

### Drill 21 — deinitialization
- **Definition prompt:** Explain what **deinitialization** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **deinitialization**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **deinitialization**, not just the spelling.
- **Production example to mention:** Connect **deinitialization** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **deinitialization** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **deinitialization**.
- **One-line close:** End with a concise summary that tells the interviewer why **deinitialization** matters in production code.

### Follow-up 21 — deinitialization
- **Compare prompt:** Contrast **deinitialization** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **deinitialization** is misunderstood.
- **Code review angle:** Describe what misuse of **deinitialization** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **deinitialization** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **deinitialization**.

### Drill 22 — resource cleanup patterns
- **Definition prompt:** Explain what **resource cleanup patterns** means in Swift without starting from syntax alone.
- **Mental model prompt:** Describe the simplest accurate mental model for **resource cleanup patterns**.
- **Why interviewers ask:** They want to see whether you understand the semantics behind **resource cleanup patterns**, not just the spelling.
- **Production example to mention:** Connect **resource cleanup patterns** to a real iOS code path such as view state, networking, decoding, ownership, or architecture.
- **Common mistake:** Do not answer **resource cleanup patterns** as a memorized rule if there is a tradeoff or boundary decision involved.
- **Senior signal:** Mention the invariant, ownership rule, abstraction cost, or maintainability consequence behind **resource cleanup patterns**.
- **One-line close:** End with a concise summary that tells the interviewer why **resource cleanup patterns** matters in production code.

### Follow-up 22 — resource cleanup patterns
- **Compare prompt:** Contrast **resource cleanup patterns** with the nearby concept candidates most often confuse it with.
- **Pitfall prompt:** Name a bug, crash, leak, or maintainability issue that appears when **resource cleanup patterns** is misunderstood.
- **Code review angle:** Describe what misuse of **resource cleanup patterns** would look like in a pull request.
- **Refactor angle:** Explain how you would redesign the code so the correct use of **resource cleanup patterns** becomes more obvious.
- **Interview follow-up:** Be ready to give one short example and one tradeoff sentence about **resource cleanup patterns**.

