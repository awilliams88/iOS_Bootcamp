# Protocols, Associated Types, and Generics

## Protocols

Protocols define contracts.

```swift
protocol NetworkService {
    func fetch()
}
```

Any conforming type must implement required behavior.

## Protocol Extensions

Protocols can provide default implementations.

## Associated Types

Used when protocol needs a placeholder type.

```swift
protocol Repository {
    associatedtype Model
    func save(_ item: Model)
}
```

Why not generic properties? Protocols describe requirements, not concrete storage.

## Generics

Reusable type-safe abstraction.

```swift
func compare<T: Equatable>(_ a: T, _ b: T) -> Bool {
    a == b
}
```

## Interview Answer

Protocols define behavior contracts. Generics provide reusable type-safe abstraction. Associated types allow protocols to defer concrete type definition to conformers.
