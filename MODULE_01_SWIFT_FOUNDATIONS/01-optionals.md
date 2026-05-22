# Optionals

## What are Optionals?

Optionals represent a value that may or may not exist.

```swift
var name: String?
```

Meaning:
- contains String
- or nil

## Why Needed

Avoid null pointer style crashes by making absence explicit.

## Safe Unwrapping

### if let
```swift
if let name = name {
    print(name)
}
```

### guard let
```swift
guard let user = user else { return }
```

## Force Unwrap
```swift
name!
```
Unsafe if nil.

## Interview Answer

Optionals make absence explicit and force safer handling compared to implicit null usage.
