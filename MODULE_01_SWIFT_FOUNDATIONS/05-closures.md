# Closures

## Definition

A closure is executable behavior plus captured surrounding context.

## Example

```swift
let greet = {
    print("Hello")
}
```

## Capture

Closures can retain surrounding values.

```swift
var x = 10
let closure = {
    print(x)
}
```

## Capture List

```swift
{ [weak self] in }
```

Used to control ownership.

## Escaping

```swift
@escaping
```

Used when closure outlives function scope.

## Autoclosure

Automatically wraps expressions into closures.

## Interview Answer

Closures are reusable executable code blocks that can capture surrounding context and are widely used in callbacks and async programming.
