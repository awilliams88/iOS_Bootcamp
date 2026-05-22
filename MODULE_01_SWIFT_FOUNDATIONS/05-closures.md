# Closures

## Mental Model

Closures are executable code blocks that can capture surrounding context.

## Syntax

```swift
let greet = {
    print("Hello")
}
```

## Capture Semantics

```swift
var x = 10
let closure = {
    print(x)
}
```

Closures capture referenced values.

## Capture Lists

```swift
{ [weak self] in }
```

Also:
```swift
{ [x] in }
```

Useful for explicit value capture.

## Escaping

```swift
@escaping
```

Closure survives function lifetime.

## Nonescaping
Default behavior.

Compiler can optimize more aggressively.

## Autoclosure

Automatically wraps expressions lazily.

## Retain Cycles
Closures can strongly capture objects.
Use weak/unowned carefully.

## Interview Guidance
Closures power callbacks, async APIs, functional transformations, and event handling.
