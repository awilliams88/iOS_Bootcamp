# Advanced — Combine Framework

Combine is Apple's reactive programming framework. While async/await handles much of what Combine did, Combine remains widely used in production codebases and is a common interview topic.

---

## Core Concepts

```
Publisher → (operators) → Subscriber
```

- **Publisher**: emits values over time (`Output`, `Failure`)
- **Operator**: transforms publisher output (`map`, `filter`, `flatMap`)
- **Subscriber**: receives values (`sink`, `assign`)
- **Cancellable**: token to cancel a subscription

---

## Creating Publishers

```swift
// Just — single value, then completes
let publisher = Just(42)

// Fail — immediately fails
let failing = Fail<Int, Error>(error: NetworkError.unauthorized)

// Empty — completes immediately without value
let empty = Empty<Int, Never>()

// Future — async operation, single value or error
let future = Future<String, Error> { promise in
    URLSession.shared.dataTask(with: url) { data, _, error in
        if let error = error { promise(.failure(error)) }
        else if let data = data { promise(.success(String(data: data, encoding: .utf8)!)) }
    }.resume()
}

// Subject — manually push values
let passthrough = PassthroughSubject<Int, Never>()  // No initial value
let current = CurrentValueSubject<Int, Never>(0)     // Holds current value

passthrough.send(1)
passthrough.send(2)
passthrough.send(completion: .finished)
```

---

## @Published

```swift
class ViewModel: ObservableObject {
    @Published var query = ""        // PassthroughSubject under the hood
    @Published var results: [String] = []
    
    init() {
        // React to query changes
        $query  // Publisher<String, Never>
            .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .filter { !$0.isEmpty }
            .flatMap { query in
                self.search(query: query)
                    .replaceError(with: [])  // Handle errors
            }
            .assign(to: &$results)  // Assign back to @Published (no cancellable needed)
    }
}
```

---

## Key Operators

```swift
// Transformation
publisher.map { $0 * 2 }           // Transform each value
publisher.compactMap { $0 }        // Filter nil values
publisher.flatMap { fetchData($0) } // Map to publisher, flatten

// Filtering  
publisher.filter { $0 > 0 }
publisher.removeDuplicates()
publisher.first()
publisher.debounce(for: .seconds(0.3), scheduler: RunLoop.main)
publisher.throttle(for: .seconds(1), scheduler: RunLoop.main, latest: true)

// Combining
Publishers.CombineLatest(publisher1, publisher2)  // Emit when either emits
Publishers.Merge(publisher1, publisher2)           // Merge two publishers
Publishers.Zip(publisher1, publisher2)             // Pair values in order
publisher.combineLatest(publisher2) { a, b in a + b }

// Error handling
publisher.catch { error in Just(fallbackValue) }  // Replace error with value
publisher.retry(3)                                  // Retry on error
publisher.replaceError(with: defaultValue)

// Timing
publisher.delay(for: .seconds(1), scheduler: DispatchQueue.main)
publisher.timeout(.seconds(5), scheduler: DispatchQueue.main)

// Scheduling
publisher.receive(on: DispatchQueue.main)           // Switch to main for UI
publisher.subscribe(on: DispatchQueue.global())     // Start work on background
```

---

## Sinking and Storing Subscriptions

```swift
var cancellables = Set<AnyCancellable>()

publisher
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished: print("Done")
            case .failure(let error): print("Error: \(error)")
            }
        },
        receiveValue: { value in
            print("Value: \(value)")
        }
    )
    .store(in: &cancellables)  // CRITICAL — must store or subscription is cancelled immediately

// assign (no error handling)
publisher
    .receive(on: DispatchQueue.main)
    .assign(to: \.text, on: label)  // Assign to property
    .store(in: &cancellables)
```

> ⚠️ **Critical Pitfall:** If you don't store the `AnyCancellable` from `.sink` or `.assign`, the subscription is immediately cancelled and you receive nothing. Always store in `Set<AnyCancellable>`.

---

## Real-World Combine Pattern — Network with Retry

```swift
func fetchUser(id: Int) -> AnyPublisher<User, Error> {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    
    return URLSession.shared.dataTaskPublisher(for: url)
        .mapError { $0 as Error }
        .retry(3)
        .map(\.data)
        .decode(type: User.self, decoder: JSONDecoder())
        .receive(on: DispatchQueue.main)
        .eraseToAnyPublisher()  // Hide implementation type
}

// Usage in ViewModel
func loadUser(id: Int) {
    fetchUser(id: id)
        .sink(
            receiveCompletion: { [weak self] completion in
                if case .failure(let error) = completion {
                    self?.errorMessage = error.localizedDescription
                }
            },
            receiveValue: { [weak self] user in
                self?.user = user
            }
        )
        .store(in: &cancellables)
}
```

---

## Combine vs async/await

| Aspect | Combine | async/await |
|--------|---------|-------------|
| Style | Declarative/functional | Imperative |
| Error handling | `.catch`, `.replaceError` | `try/catch` |
| Cancellation | Store `AnyCancellable` | `Task.cancel()` |
| Operators | Rich (100+ operators) | TaskGroup, AsyncSequence |
| Learning curve | Steep | Gradual |
| Min iOS | 13 | 15 (full support) |
| Use today | Legacy code, `@Published` | New code |

---

## Interview Q&A

**Q: What is the difference between PassthroughSubject and CurrentValueSubject?**  
A: `PassthroughSubject` has no initial value — it only emits values after subscription. `CurrentValueSubject` holds and emits the current value immediately to new subscribers, then continues emitting future values. Use `CurrentValueSubject` when the subscriber needs the current state immediately (like a ViewModel's state).

**Q: Why do you need to store Cancellables?**  
A: A Combine subscription lives only as long as its `AnyCancellable`. If you don't store it, ARC releases it immediately, cancelling the subscription. `Set<AnyCancellable>` is the standard container — when the ViewModel is deallocated, the set is too, automatically cancelling all subscriptions.

**Q: What is `eraseToAnyPublisher()` and why is it used?**  
A: It wraps any publisher in `AnyPublisher`, hiding the concrete (often complex) generic type. Without it, return types like `Publishers.Catch<Publishers.Retry<URLSession.DataTaskPublisher>, Just<Data>>` are impossible to write. `AnyPublisher` is the standard return type for public API.

---

## Quick Revision

- Publisher → emits Output/Failure, Subscriber receives
- `@Published`: property wrapper that creates a publisher, used with `ObservableObject`
- `PassthroughSubject`: no initial value; `CurrentValueSubject`: holds current value
- Key operators: `map`, `flatMap`, `debounce`, `combineLatest`, `catch`, `retry`
- `.receive(on: DispatchQueue.main)`: switch to main thread for UI updates
- Store cancellables: `anyCancellable.store(in: &cancellables)` — else subscription is lost
- `eraseToAnyPublisher()`: hide complex publisher type behind `AnyPublisher<Output, Failure>`
