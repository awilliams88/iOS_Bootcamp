# Testing — Async Testing

---

## async/await in Test Functions

Swift 5.5+ supports `async` test functions natively in XCTest:

```swift
final class UserServiceTests: XCTestCase {
    // async throws test — no expectations needed!
    func test_fetchUsers_returnsUsers() async throws {
        let mockClient = MockAPIClient()
        mockClient.responses["/users"] = .success(try JSONEncoder().encode([User.mock]))
        
        let service = UserService(client: mockClient)
        let users = try await service.fetchUsers()
        
        XCTAssertEqual(users.count, 1)
        XCTAssertEqual(users[0].name, "Alice")
    }
    
    func test_fetchUsers_whenUnauthorized_throwsError() async {
        let mockClient = MockAPIClient()
        mockClient.responses["/users"] = .failure(NetworkError.unauthorized)
        
        let service = UserService(client: mockClient)
        
        do {
            _ = try await service.fetchUsers()
            XCTFail("Expected throw but didn't")
        } catch NetworkError.unauthorized {
            // Expected
        } catch {
            XCTFail("Wrong error: \(error)")
        }
    }
}
```

> 💡 **Tip:** Use `async throws` test functions — they're cleaner and don't need `XCTestExpectation`. XCTest handles the async lifecycle automatically.

---

## XCTestExpectation (Legacy / Callback-based)

Still needed for completion-handler APIs:

```swift
func test_fetchUsers_callsCompletion() {
    let expectation = expectation(description: "Fetch users completes")
    let service = UserService()
    
    service.fetchUsers { result in
        switch result {
        case .success(let users):
            XCTAssertFalse(users.isEmpty)
            expectation.fulfill()
        case .failure(let error):
            XCTFail("Unexpected error: \(error)")
        }
    }
    
    wait(for: [expectation], timeout: 5.0)
}

// Multiple expectations
func test_multipleAsyncOperations() {
    let exp1 = expectation(description: "First operation")
    let exp2 = expectation(description: "Second operation")
    
    performOperation1 { exp1.fulfill() }
    performOperation2 { exp2.fulfill() }
    
    wait(for: [exp1, exp2], timeout: 10.0, enforceOrder: true)
}

// Inverted expectation — assert something does NOT happen
func test_noNetworkCall_whenCached() {
    let notCalledExpectation = expectation(description: "Network not called")
    notCalledExpectation.isInverted = true
    
    mockService.onFetch = { notCalledExpectation.fulfill() }
    viewModel.loadWithCache()  // Should use cache, not network
    
    wait(for: [notCalledExpectation], timeout: 0.5)
}
```

---

## Testing @MainActor Code

```swift
// Mark test class or individual tests with @MainActor
@MainActor
final class MainActorViewModelTests: XCTestCase {
    var sut: UserListViewModel!
    
    override func setUp() async throws {
        // setUp can also be async for @MainActor classes
        sut = UserListViewModel(service: MockUserService())
    }
    
    func test_isLoading_isTrueDuringFetch() async {
        XCTAssertFalse(sut.isLoading)
        
        let fetchTask = Task { await sut.loadUsers() }
        // isLoading should be true now — but this is tricky to test
        // Better: test final state
        await fetchTask.value
        XCTAssertFalse(sut.isLoading)
    }
}
```

---

## Testing Actors

```swift
actor CounterActor {
    private(set) var count = 0
    func increment() { count += 1 }
    func reset() { count = 0 }
}

final class CounterActorTests: XCTestCase {
    func test_increment_increasesCount() async {
        let counter = CounterActor()
        await counter.increment()
        await counter.increment()
        let count = await counter.count
        XCTAssertEqual(count, 2)
    }
    
    func test_concurrentIncrements_areThreadSafe() async {
        let counter = CounterActor()
        
        await withTaskGroup(of: Void.self) { group in
            for _ in 0..<100 {
                group.addTask { await counter.increment() }
            }
        }
        
        let finalCount = await counter.count
        XCTAssertEqual(finalCount, 100)  // Guaranteed — actor serializes access
    }
}
```

---

## Testing Task Cancellation

```swift
func test_loadCancelled_doesNotUpdateState() async {
    let sut = UserListViewModel(service: SlowMockService())
    
    let task = Task { await sut.loadUsers() }
    task.cancel()
    
    await task.value
    XCTAssertTrue(sut.users.isEmpty)  // Cancelled before completion
}
```

---

## setUp / tearDown async Support

```swift
final class AsyncSetupTests: XCTestCase {
    var database: TestDatabase!
    
    override func setUp() async throws {
        database = try await TestDatabase.create()
        try await database.seed(with: TestFixtures.users)
    }
    
    override func tearDown() async throws {
        try await database.clear()
    }
    
    func test_fetchUsers_returnsSeededUsers() async throws {
        let users = try await database.fetchUsers()
        XCTAssertEqual(users.count, TestFixtures.users.count)
    }
}
```

---

## Interview Q&A

**Q: How do you test async/await code in XCTest?**  
A: Mark the test function as `async throws`. XCTest handles the async execution automatically. You can `await` async methods directly and use normal `XCTAssert*` assertions after. No `XCTestExpectation` needed for async functions.

**Q: When would you still use XCTestExpectation?**  
A: For legacy completion-handler APIs that haven't been converted to async, for Combine publishers, for testing that something does NOT happen (inverted expectation), or for verifying callback ordering with `enforceOrder: true`.

**Q: How do you test that an actor is thread-safe?**  
A: Use `TaskGroup` to launch many concurrent tasks that mutate the actor, then verify the final state is correct. Because actors serialize access, the result is always deterministic — unlike testing a regular class with a queue where you'd need locks.

**Q: How do you handle `@MainActor` in tests?**  
A: Mark the test class or method with `@MainActor`. You can also use `await MainActor.run {}` to execute specific assertions on the main actor. `setUp` and `tearDown` can also be marked `async throws` for `@MainActor` test classes.

---

## Quick Revision

- `async throws` test function: XCTest handles it natively, no expectations needed
- `XCTestExpectation`: for callbacks, Combine, or "not called" assertions
- `isInverted = true`: expectation that fires means test FAILS
- `wait(for:timeout:enforceOrder:)`: wait with optional ordering requirement
- Actor tests: use `TaskGroup` for concurrency stress tests
- `@MainActor` class or method annotation for MainActor ViewModel tests
- `setUp() async throws`: async setup supported in XCTest
