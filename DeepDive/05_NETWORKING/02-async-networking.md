# Networking — Async Networking & Retry

---

## async/await with URLSession

iOS 15 introduced native async support in URLSession:

```swift
// data(for:) — returns (Data, URLResponse)
func fetchUser(id: Int) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.badStatusCode((response as? HTTPURLResponse)?.statusCode ?? -1)
    }
    return try JSONDecoder().decode(User.self, from: data)
}

// Parallel requests with async let
func fetchDashboard() async throws -> Dashboard {
    async let user    = fetchUser(id: currentUserID)
    async let posts   = fetchPosts(page: 1)
    async let metrics = fetchMetrics()
    
    // All three fire concurrently, we await all three
    return Dashboard(
        user: try await user,
        posts: try await posts,
        metrics: try await metrics
    )
}
```

---

## Bytes Streaming

For large responses, stream bytes instead of loading all data into memory:

```swift
func streamLargeFile(url: URL) async throws {
    let (bytes, response) = try await URLSession.shared.bytes(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else { return }
    
    var buffer = Data()
    for try await byte in bytes {
        buffer.append(byte)
        if buffer.count >= 4096 {
            process(buffer)
            buffer.removeAll(keepingCapacity: true)
        }
    }
    if !buffer.isEmpty { process(buffer) }
}
```

---

## Task Cancellation

```swift
// Store task reference to cancel later
var currentTask: Task<User, Error>?

func loadUser(id: Int) {
    currentTask?.cancel()  // Cancel previous request
    currentTask = Task {
        try await fetchUser(id: id)
    }
}

// In network layer — check for cancellation
func fetchWithCancellationCheck(url: URL) async throws -> Data {
    let (data, _) = try await URLSession.shared.data(from: url)
    try Task.checkCancellation()  // Throw if cancelled after request completed
    return data
}

// URLSession task is automatically cancelled when the Task is cancelled
```

> 💡 **Tip:** When a Swift `Task` is cancelled, in-flight `URLSession` async requests are automatically cancelled — they throw `URLError.cancelled`.

---

## Retry with Exponential Backoff

```swift
func fetchWithRetry<T: Decodable>(
    request: URLRequest,
    maxAttempts: Int = 3,
    baseDelay: TimeInterval = 1.0
) async throws -> T {
    var lastError: Error?
    
    for attempt in 0..<maxAttempts {
        do {
            let (data, response) = try await URLSession.shared.data(for: request)
            try validateResponse(response)
            return try JSONDecoder().decode(T.self, from: data)
        } catch {
            lastError = error
            
            // Don't retry on client errors (4xx) or cancellation
            if let urlError = error as? URLError, urlError.code == .cancelled {
                throw error
            }
            if let networkError = error as? NetworkError,
               case .badStatusCode(let code) = networkError,
               (400...499).contains(code) {
                throw error  // Client error — no point retrying
            }
            
            if attempt < maxAttempts - 1 {
                // Exponential backoff with jitter
                let delay = baseDelay * pow(2.0, Double(attempt))
                let jitter = Double.random(in: 0...0.5)
                try await Task.sleep(for: .seconds(delay + jitter))
            }
        }
    }
    throw lastError ?? NetworkError.noData
}
```

---

## AsyncStream for Real-time Data

Bridge callback-based APIs (WebSocket, SSE) to AsyncSequence:

```swift
// Server-Sent Events streaming
func streamEvents(url: URL) -> AsyncStream<ServerEvent> {
    AsyncStream { continuation in
        let task = Task {
            let (bytes, _) = try await URLSession.shared.bytes(from: url)
            for try await line in bytes.lines {
                if line.hasPrefix("data: ") {
                    let json = String(line.dropFirst(6))
                    if let event = ServerEvent(json: json) {
                        continuation.yield(event)
                    }
                }
            }
            continuation.finish()
        }
        
        continuation.onTermination = { _ in task.cancel() }
    }
}

// Usage
for await event in streamEvents(url: sseURL) {
    updateUI(with: event)
}
```

---

## Timeout Handling

```swift
// Per-request timeout
var request = URLRequest(url: url)
request.timeoutInterval = 10  // 10 second timeout

// Task-level timeout using withTimeout pattern
func withTimeout<T>(seconds: Double, operation: () async throws -> T) async throws -> T {
    try await withThrowingTaskGroup(of: T.self) { group in
        group.addTask { try await operation() }
        group.addTask {
            try await Task.sleep(for: .seconds(seconds))
            throw NetworkError.timeout
        }
        
        let result = try await group.next()!
        group.cancelAll()
        return result
    }
}

// Usage
let user = try await withTimeout(seconds: 5) {
    try await fetchUser(id: 1)
}
```

---

## Interview Q&A

**Q: How does URLSession integrate with Swift concurrency?**  
A: iOS 15 added `async` variants of URLSession methods: `data(from:)`, `data(for:)`, `bytes(from:)`, `download(from:)`. These suspend the current Task rather than blocking a thread. Cancelling the Swift Task automatically cancels the underlying URLSessionTask.

**Q: How do you run multiple network requests in parallel?**  
A: Use `async let` for a fixed set of concurrent requests, or `TaskGroup` for dynamic sets. `async let` fires all requests simultaneously and you `await` them together, getting structured concurrency's automatic cancellation on error.

**Q: Walk me through implementing retry with exponential backoff.**  
A: Loop for `maxAttempts`, catch errors, skip retry for non-retryable errors (4xx client errors, cancellation), and use `Task.sleep` with exponentially increasing delay (1s, 2s, 4s, ...) plus random jitter to avoid thundering herd problems.

**Q: What is `Task.checkCancellation()` and when do you use it?**  
A: It throws `CancellationError` if the current task has been cancelled. Use it after long-running operations (like a completed network request) to propagate cancellation before doing expensive post-processing (decoding, writing to disk).

---

## Quick Revision

- `URLSession.data(for:)` — async/await network request  
- `URLSession.bytes(from:)` — streaming bytes (large files, SSE)  
- `async let` — concurrent parallel requests  
- Task cancellation propagates to URLSession automatically  
- Retry: exponential backoff + jitter + skip 4xx errors  
- `AsyncStream` — bridge callbacks to AsyncSequence  
- `Task.sleep(for:)` — async non-blocking delay
