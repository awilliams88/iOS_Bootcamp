# Optionals

## Mental Model

An optional explicitly models absence.

```swift
String?
```
means:
- a String value
- or nil

This avoids hidden null crashes.

## Unwrapping Techniques

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

Preferred for early exits.

### nil coalescing
```swift
let display = name ?? "Guest"
```

### optional chaining
```swift
user.profile?.avatarURL
```

## Implicitly Unwrapped Optionals

```swift
@IBOutlet weak var label: UILabel!
```

Use sparingly.

## map / flatMap

Transform optional values safely.

## Interview Pitfalls

- force unwrap crashes
- nested optionals
- IUO misuse
