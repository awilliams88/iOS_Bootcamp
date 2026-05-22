# Swift Foundations Exercises

## Exercise 1
Explain why this causes a retain cycle and fix it.

```swift
class A {
    var closure: (() -> Void)?
    func setup() {
        closure = {
            print(self)
        }
    }
}
```

## Exercise 2
Design a protocol with associated type for caching.

## Exercise 3
Model API state using enum with associated values.

## Exercise 4
Write custom Codable decoding for mismatched JSON keys.

## Exercise 5
Explain some vs any with practical examples.
