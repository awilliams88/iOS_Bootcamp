# Advanced — AsyncSequence & Streams

`AsyncSequence` is the async/await alternative to Combine for streaming data.

---

## AsyncSequence Protocol

```swift
// Protocol definition
protocol AsyncSequence {
    associatedtype Element
    associatedtype AsyncIterator: AsyncIteratorProtocol
    func makeAsyncIterator() -> AsyncIterator
}

// Usage — just like regular for-in, but with await
for try await line in url.lines {
    process(line)
}

// Built-in AsyncSequences in Foundation
url.lines                          // Async line-by-line file reading
URLSession.shared.bytes(from: url) // Async byte streaming
NotificationCenter.default.notifications(named: .didUpdate) // Notification stream
```

---

## AsyncStream

`AsyncStream` bridges callback-based APIs to `AsyncSequence`:

```swift
// Basic AsyncStream
func countdown(from n: Int) -> AsyncStream<Int> {
    AsyncStream { continuation in
        Task {
            for i in stride(from: n, through: 0, by: -1) {
                continuation.yield(i)
                try await Task.sleep(for: .seconds(1))
            }
            continuation.finish()
        }
    }
}

// Usage
for await count in countdown(from: 5) {
    print(count)
}
```

### Bridging Delegate Callback to AsyncStream

```swift
class BluetoothScannerStream: NSObject, CBCentralManagerDelegate {
    private var continuation: AsyncStream<CBPeripheral>.Continuation?
    
    func peripherals() -> AsyncStream<CBPeripheral> {
        AsyncStream { continuation in
            self.continuation = continuation
            
            continuation.onTermination = { [weak self] _ in
                self?.centralManager.stopScan()
                self?.continuation = nil
            }
        }
    }
    
    func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral,
                        advertisementData: [String: Any], rssi RSSI: NSNumber) {
        continuation?.yield(peripheral)
    }
}

// Usage
let scanner = BluetoothScannerStream()
for await peripheral in scanner.peripherals() {
    print("Found: \(peripheral.name ?? "Unknown")")
}
```

---

## AsyncThrowingStream

For streams that can fail:

```swift
func streamData(from url: URL) -> AsyncThrowingStream<Data, Error> {
    AsyncThrowingStream { continuation in
        Task {
            do {
                let (bytes, _) = try await URLSession.shared.bytes(from: url)
                var buffer = Data()
                for try await byte in bytes {
                    buffer.append(byte)
                    if buffer.count >= 1024 {
                        continuation.yield(buffer)
                        buffer.removeAll()
                    }
                }
                if !buffer.isEmpty { continuation.yield(buffer) }
                continuation.finish()
            } catch {
                continuation.finish(throwing: error)
            }
        }
    }
}
```

---

## Custom AsyncSequence

```swift
struct TimerSequence: AsyncSequence {
    typealias Element = Date
    let interval: TimeInterval
    
    struct AsyncIterator: AsyncIteratorProtocol {
        let interval: TimeInterval
        
        mutating func next() async -> Date? {
            try? await Task.sleep(for: .seconds(interval))
            return Task.isCancelled ? nil : Date()
        }
    }
    
    func makeAsyncIterator() -> AsyncIterator {
        AsyncIterator(interval: interval)
    }
}

// Usage
for await timestamp in TimerSequence(interval: 1.0) {
    print(timestamp)
}
```

---

## Combining AsyncSequences

```swift
// Map
let doubled = countdown(from: 5).map { $0 * 2 }

// Filter  
let evens = countdown(from: 10).filter { $0 % 2 == 0 }

// First
let first = await countdown(from: 5).first(where: { $0 < 3 })

// Collect into array
let all = await countdown(from: 5).reduce(into: []) { $0.append($1) }

// AsyncStream merging (manual)
func merge<T>(_ streams: [AsyncStream<T>]) -> AsyncStream<T> {
    AsyncStream { continuation in
        let group = Task {
            await withTaskGroup(of: Void.self) { group in
                for stream in streams {
                    group.addTask {
                        for await value in stream {
                            continuation.yield(value)
                        }
                    }
                }
            }
            continuation.finish()
        }
        continuation.onTermination = { _ in group.cancel() }
    }
}
```

---

## Combine vs AsyncSequence

| Feature | Combine | AsyncSequence/AsyncStream |
|---------|---------|--------------------------|
| Error handling | `Failure` generic type | `AsyncThrowingStream` |
| Operators | Rich built-in operators | Manual via `map`, `filter` |
| Backpressure | Built-in | Manual |
| Min iOS | 13 | 15 |
| Cancellation | `AnyCancellable` | Task cancellation |
| Learning curve | Steep | Gradual |
| UIKit integration | Good (Combine bindings) | Requires adaptation |
| SwiftUI | `objectWillChange` | Works with `.task` |

---

## Interview Q&A

**Q: What is AsyncSequence and how does it differ from Combine?**  
A: `AsyncSequence` is the async/await-native way to iterate over a stream of values using `for await in`. Unlike Combine, there's no separate subscriber type — you just use `for await`. It's simpler for one-to-one linear flows but lacks Combine's rich operator library. For complex multi-publisher compositions, Combine still has an advantage.

**Q: How do you bridge a callback API to AsyncSequence?**  
A: Use `AsyncStream` with a `continuation`. Pass the continuation to your callback handler; call `continuation.yield(value)` in callbacks and `continuation.finish()` when done. Set `onTermination` to clean up (cancel the underlying operation) when the consumer stops iterating.

**Q: How does cancellation work with AsyncSequence?**  
A: Since iteration happens inside a `Task`, cancelling the `Task` cancels the iteration. The `next()` method returns `nil` when cancelled (or throws `CancellationError` for throwing sequences). The `onTermination` handler of `AsyncStream` is called, allowing cleanup.

---

## Quick Revision

- `AsyncSequence`: iterate async values with `for await in`
- Built-in: `url.lines`, `URLSession.bytes(from:)`, `NotificationCenter.notifications`
- `AsyncStream`: bridge callbacks to AsyncSequence (one producer, one consumer)
- `AsyncThrowingStream`: stream that can fail
- `continuation.yield()`: emit value; `continuation.finish()`: complete
- `continuation.onTermination`: cleanup when consumer stops
- Cancellation: Task cancellation propagates to `AsyncStream` automatically
