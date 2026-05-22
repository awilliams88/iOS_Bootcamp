# Testing Fundamentals

## Unit Testing

Tests isolated logic.

## UI Testing

Automates UI workflows.

## Async Testing

Use XCTest expectations or async test support.

## Mocking

Replace dependencies.

```swift
protocol APIService {
    func fetchUsers() async throws -> [User]
}
```

Mock implementation improves deterministic tests.

## Interview Answer

Good architecture improves testability. Protocol abstraction and dependency injection make isolated testing practical.
