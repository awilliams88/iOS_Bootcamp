# Testing — Mocks, Stubs & Fakes

---

## Test Doubles

A "test double" is any object that replaces a real dependency in tests:

| Type | Description |
|------|-------------|
| **Mock** | Verifies interactions — did method X get called with Y? |
| **Stub** | Returns pre-configured values — no verification |
| **Spy** | Records calls for later verification |
| **Fake** | Real implementation but lightweight (in-memory DB) |
| **Dummy** | Placeholder — never actually used |

---

## Stub Pattern

```swift
// Protocol to mock
protocol UserServiceProtocol {
    func fetchUsers() async throws -> [User]
    func deleteUser(id: UUID) async throws
}

// Stub — returns configured values
final class StubUserService: UserServiceProtocol {
    var stubbedUsers: [User] = []
    var shouldThrowError: Error?
    
    func fetchUsers() async throws -> [User] {
        if let error = shouldThrowError { throw error }
        return stubbedUsers
    }
    
    func deleteUser(id: UUID) async throws {
        if let error = shouldThrowError { throw error }
    }
}
```

---

## Mock Pattern (Interaction Verification)

```swift
// Mock — records calls AND allows verification
final class MockUserService: UserServiceProtocol {
    // Return stubs
    var stubbedUsers: [User] = []
    var shouldThrowError: Error?
    
    // Interaction tracking (spy behavior)
    private(set) var fetchUsersCallCount = 0
    private(set) var deleteUserCallHistory: [UUID] = []
    private(set) var lastDeletedUserId: UUID?
    
    func fetchUsers() async throws -> [User] {
        fetchUsersCallCount += 1
        if let error = shouldThrowError { throw error }
        return stubbedUsers
    }
    
    func deleteUser(id: UUID) async throws {
        deleteUserCallHistory.append(id)
        lastDeletedUserId = id
        if let error = shouldThrowError { throw error }
    }
    
    // Convenience verify methods
    var didFetchUsers: Bool { fetchUsersCallCount > 0 }
    func didDeleteUser(id: UUID) -> Bool { deleteUserCallHistory.contains(id) }
}

// Test using mock
func test_deleteUser_callsServiceWithCorrectId() async {
    let mock = MockUserService()
    mock.stubbedUsers = [User(id: targetId, name: "Alice")]
    let sut = UserListViewModel(service: mock)
    await sut.loadUsers()
    await sut.deleteUser(id: targetId)
    
    XCTAssertTrue(mock.didDeleteUser(id: targetId))
    XCTAssertEqual(mock.deleteUserCallHistory.count, 1)
}
```

---

## Fake Pattern

```swift
// Fake — real logic, in-memory storage
final class FakeUserRepository: UserRepositoryProtocol {
    private var storage: [UUID: User] = [:]
    
    func fetchUsers() async throws -> [User] {
        Array(storage.values).sorted { $0.name < $1.name }
    }
    
    func saveUser(_ user: User) async throws {
        storage[user.id] = user
    }
    
    func deleteUser(id: UUID) async throws {
        storage.removeValue(forKey: id)
    }
    
    func fetchUser(id: UUID) async throws -> User {
        guard let user = storage[id] else { throw RepositoryError.notFound }
        return user
    }
}
```

Fakes are useful when you need real behavioral logic (e.g., a save followed by a fetch should return the saved item).

---

## Dependency Injection for Testing

```swift
// ViewModel with injectable dependencies
final class CheckoutViewModel {
    private let paymentService: PaymentServiceProtocol
    private let analyticsService: AnalyticsServiceProtocol
    private let inventoryService: InventoryServiceProtocol
    
    init(
        paymentService: PaymentServiceProtocol,
        analyticsService: AnalyticsServiceProtocol = NoOpAnalyticsService(),
        inventoryService: InventoryServiceProtocol
    ) {
        self.paymentService = paymentService
        self.analyticsService = analyticsService
        self.inventoryService = inventoryService
    }
}

// NoOp for analytics — do nothing in tests
final class NoOpAnalyticsService: AnalyticsServiceProtocol {
    func track(_ event: AnalyticsEvent) { /* no-op */ }
}

// Test setup — clean injection
func makeCheckoutViewModel(
    payment: PaymentServiceProtocol = MockPaymentService(),
    inventory: InventoryServiceProtocol = StubInventoryService()
) -> CheckoutViewModel {
    CheckoutViewModel(paymentService: payment, inventoryService: inventory)
}
```

---

## Testing Combine Publishers

```swift
import Combine

final class SearchViewModelTests: XCTestCase {
    var sut: SearchViewModel!
    var cancellables = Set<AnyCancellable>()
    
    func test_searchResults_updatesWhenQueryChanges() {
        let expectation = expectation(description: "Results updated")
        let mockService = MockSearchService()
        mockService.stubbedResults = [SearchResult(title: "Swift")]
        sut = SearchViewModel(searchService: mockService)
        
        sut.$results
            .dropFirst()  // Skip initial empty value
            .sink { results in
                XCTAssertEqual(results.count, 1)
                XCTAssertEqual(results[0].title, "Swift")
                expectation.fulfill()
            }
            .store(in: &cancellables)
        
        sut.query = "Swift"
        wait(for: [expectation], timeout: 2.0)
    }
}
```

---

## Interview Q&A

**Q: What's the difference between a mock and a stub?**  
A: A stub returns pre-configured values when called — it doesn't verify how it was called. A mock both returns values AND verifies interactions (method call count, parameters used). In practice, many "mock" objects combine both behaviors.

**Q: Why use protocols to enable mocking?**  
A: Protocols define a contract without specifying implementation. Tests can provide a lightweight mock implementation instead of the real one. Without protocols, you'd need the actual concrete type (URLSession, database, etc.) which makes tests slow, brittle, and non-deterministic.

**Q: When would you use a Fake over a Stub?**  
A: When you need the test to exercise real behavioral logic — for example, an in-memory repository where save → fetch correctly returns the saved item. A stub can only return one fixed value; a Fake has real (but simplified) logic.

**Q: What's a Spy?**  
A: A Spy records information about how it was called (call count, arguments) for later verification, but otherwise delegates to the real implementation or returns a stub value. Spies are useful when you want to verify side effects without completely replacing behavior.

---

## Quick Revision

- Stub: returns pre-configured values, no verification
- Mock: records calls AND verifies interactions
- Fake: real logic, simplified implementation (in-memory)
- Spy: records calls, may delegate to real implementation
- Dummy: placeholder, never used
- Use protocols to enable injection of test doubles
- `NoOp` implementations: conform to protocol, do nothing (good for analytics in tests)
