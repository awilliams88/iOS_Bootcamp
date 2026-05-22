# Structs vs Classes

## Core Difference

Structs are value types.
Classes are reference types.

## Value Semantics

```swift
struct User {
    var name: String
}

var a = User(name: "Arpit")
var b = a
b.name = "Alex"
```

a remains unchanged.

## Reference Semantics

```swift
class User {
    var name: String
    init(name: String) { self.name = name }
}
```

Copies share same object.

## Class-only Features
- inheritance
- deinit
- identity checks
- ARC

## Interview Answer

Use structs by default for predictable value semantics. Use classes when shared mutable state or identity is required.
