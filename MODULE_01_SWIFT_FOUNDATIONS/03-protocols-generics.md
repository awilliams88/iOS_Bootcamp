# Protocols, Associated Types, and Generics

## Protocols

Protocols define behavioral contracts.

```swift
protocol NetworkService {
    func fetch()
}
```

Capabilities:
- property requirements
- method requirements
- initializer requirements
- static requirements
- inheritance
- composition

## Associated Types

Protocols sometimes need placeholder types.

```swift
protocol Repository {
    associatedtype Model
    func save(_ item: Model)
}
```

This allows conformers to choose concrete type later.

## Why PATs Matter
Protocols with associated types cannot be used directly without abstraction help because required concrete type information is incomplete.

## Generics

```swift
func compare<T: Equatable>(_ a: T, _ b: T) -> Bool {
    a == b
}
```

Advanced topics:
- constraints
- where clauses
- generic types
- specialization

## Interview Guidance
Use generics for compile-time specialization, existentials for runtime abstraction storage.
