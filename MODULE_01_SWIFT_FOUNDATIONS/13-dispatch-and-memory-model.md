# Dispatch, Memory Model, and Copy Semantics

## Static vs Dynamic Dispatch

Static dispatch resolves method calls at compile time.
Dynamic dispatch resolves at runtime.

Swift commonly uses static dispatch for value types and protocol extensions, dynamic dispatch for class inheritance/objective-c runtime cases.

## Copy-on-Write

Swift collections optimize value semantics.

```swift
var a = [1,2,3]
var b = a
b.append(4)
```

Physical copy happens only when mutation occurs.

## Mutating Methods

```swift
struct Counter {
    var value = 0
    mutating func increment() { value += 1 }
}
```

## ARC Scope
ARC manages reference types only.
