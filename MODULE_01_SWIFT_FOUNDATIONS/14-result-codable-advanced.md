# Result and Advanced Codable

## Result

```swift
Result<Success, Failure>
```

Represents explicit success/failure outcomes.

```swift
func load(completion: (Result<User, Error>) -> Void)
```

Compare:
- `throws` for synchronous/async structured flows
- `Result` for explicit value transport and callback APIs

## Codable

Codable = Encodable + Decodable.

## Common Features
- CodingKeys
- custom init(from:)
- custom encode(to:)
- nested containers
- keyed/unkeyed containers
- date strategies
- snake_case conversion

## Real API Challenges
- mismatched schemas
- polymorphic payloads
- optional fields
- inconsistent server data

## Interview Guidance
Codable is productive for standard schemas, but real production apps often require custom decoding logic.
