# Enums, Functions, and Properties

## Enums
Swift enums are first-class types with associated values and methods.

```swift
enum APIState {
    case idle
    case loading
    case success(Data)
    case failure(Error)
}
```

Why interviewers care:
- safer state modeling
- exhaustive switch handling
- expressive domain design

## Functions
Topics:
- argument labels
- default parameters
- variadics
- inout
- higher-order functions
- returning functions

## Properties
Kinds:
- stored
- computed
- lazy
- static
- observers (willSet/didSet)

Interview tip: property wrappers are separate advanced abstraction.