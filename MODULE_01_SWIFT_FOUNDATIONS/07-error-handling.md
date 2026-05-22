# Error Handling

## Swift Errors

Swift uses explicit error propagation.

```swift
func fetchUser() throws -> User
```

## try / try? / try!

### try
Requires explicit handling.

### try?
Converts failure to nil.

### try!
Crashes on failure.

## Result

```swift
Result<User, Error>
```

Represents either success or failure.

## Custom Errors

```swift
enum APIError: Error {
    case invalidURL
    case timeout
    case decoding(Error)
}
```

## Interview Answer

Use domain-specific error enums, throws for modern APIs, and Result for callback-based APIs.
