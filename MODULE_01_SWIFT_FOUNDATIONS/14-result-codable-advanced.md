# Result and Advanced Codable

## Result

```swift
Result<Success, Failure>
```

Encapsulates success/failure explicitly.

```swift
func load(completion: (Result<User, Error>) -> Void)
```

Useful in callback APIs and composable error flows.

## Advanced Codable

Topics:
- CodingKeys
- custom init(from:)
- custom encode(to:)
- nested containers
- date decoding strategies
- snake_case conversion
- decoding polymorphic payloads

Interview tip: Codable works well for standard schemas, but custom decoding is often needed in real APIs.
