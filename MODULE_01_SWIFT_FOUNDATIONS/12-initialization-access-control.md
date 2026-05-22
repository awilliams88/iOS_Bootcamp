# Initialization, Deinitialization, and Access Control

## Initialization
Swift uses designated and convenience initializers for classes.
Structs get memberwise initializers by default.

Topics:
- designated initializers
- convenience initializers
- failable initializers
- required initializers

```swift
init?(raw: String) {
    guard !raw.isEmpty else { return nil }
}
```

## Deinitialization
Classes can define `deinit` for cleanup.

```swift
deinit {
    print("cleanup")
}
```

## Access Control
Levels:
- open
- public
- internal
- fileprivate
- private

Interview tip: default access is internal.