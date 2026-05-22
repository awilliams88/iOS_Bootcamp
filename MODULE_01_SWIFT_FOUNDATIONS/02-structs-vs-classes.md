# Structs vs Classes

## Core Model

Structs are value types.
Classes are reference types.
Actors are reference types with isolation.

## Value Semantics

```swift
struct User {
    var name: String
}

var a = User(name: "Arpit")
var b = a
b.name = "Alex"
```

`a` remains unchanged.

## Reference Semantics

```swift
class User {
    var name: String
    init(name: String) { self.name = name }
}
```

Copies reference same object.

## Class Features
- inheritance
- identity (`===`)
- deinit
- ARC
- shared mutable state

## Struct Features
- safer defaults
- thread-friendlier semantics
- predictable copying
- copy-on-write with stdlib collections

## Interview Guidance
Prefer structs by default.
Use classes when identity/shared mutation is required.
