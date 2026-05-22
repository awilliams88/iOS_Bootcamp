# `some`, `any`, and Type Erasure

## `some`
Opaque return types preserve concrete type identity while hiding implementation details.

```swift
func makeView() -> some View
```

## `any`
Existential types can store values of different concrete conformers.

```swift
let service: any NetworkService
```

Tradeoff: dynamic dispatch + existential overhead.

## Type Erasure
Used when protocols with associated types cannot be stored directly.

Interview answer: use `some` when returning one hidden concrete type, `any` for heterogeneous abstraction.
