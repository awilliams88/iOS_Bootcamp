# Protocol Extensions and Existentials

## Protocol Extensions

Protocol extensions provide shared default implementations.

```swift
protocol Drawable {
    func draw()
}

extension Drawable {
    func draw() {
        print("default draw")
    }
}
```

Benefits:
- protocol-oriented design
- code reuse
- cleaner abstractions

Pitfall: protocol extension dispatch can surprise developers due to static dispatch behavior.

## Existentials

Existentials allow storing values conforming to a protocol.

```swift
let service: any NetworkService
```

Tradeoffs:
- dynamic dispatch
- existential boxing overhead
- reduced compile-time specialization

Interview angle: use generics when concrete specialization matters, existentials when abstraction storage matters.
