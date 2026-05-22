# Extensions and Property Wrappers

## Extensions

Extensions add functionality to existing types.

```swift
extension String {
    var isBlank: Bool {
        trimmingCharacters(in: .whitespacesAndNewlines).isEmpty
    }
}
```

Can add:
- methods
- computed properties
- protocol conformance
- subscripts
- nested types

Cannot add stored properties.

## Property Wrappers

Encapsulate reusable property behavior.

```swift
@propertyWrapper
struct Clamped {
    private var value: Int
    init(wrappedValue: Int) {
        value = min(max(wrappedValue, 0), 100)
    }
    var wrappedValue: Int {
        get { value }
        set { value = min(max(newValue, 0), 100) }
    }
}
```
