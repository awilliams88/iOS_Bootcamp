# Testing — Unit Testing Fundamentals

---

## XCTest Basics

```swift
import XCTest
@testable import MyApp  // Access internal types

final class UserViewModelTests: XCTestCase {
    // setUp — runs before each test
    override func setUp() {
        super.setUp()
        // Initialize fresh objects
    }
    
    // tearDown — runs after each test
    override func tearDown() {
        super.tearDown()
        // Clean up
    }
    
    // Test methods must start with "test"
    func testUserCountAfterLoad() {
        // Arrange
        let service = MockUserService()
        service.stubbedUsers = [User(id: UUID(), name: "Alice")]
        let sut = UserListViewModel(service: service)  // sut = System Under Test
        
        // Act
        // (synchronous — see async tests for async)
        
        // Assert
        XCTAssertEqual(sut.users.count, 0)  // Before load
    }
}
```

### Assertion Methods

```swift
XCTAssertEqual(a, b)               // a == b
XCTAssertNotEqual(a, b)            // a != b
XCTAssertTrue(condition)
XCTAssertFalse(condition)
XCTAssertNil(optional)
XCTAssertNotNil(optional)
XCTAssertGreaterThan(a, b)
XCTAssertLessThanOrEqual(a, b)

// Throwing
XCTAssertThrowsError(try riskyCall()) { error in
    XCTAssertEqual(error as? NetworkError, .unauthorized)
}
XCTAssertNoThrow(try safeCall())

// Unwrap optional safely (Swift 5.3+)
let user = try XCTUnwrap(viewModel.currentUser)
XCTAssertEqual(user.name, "Alice")
```

---

## Test Naming Convention

```swift
// Format: test_[condition]_[expectedOutcome]
// or: test[MethodName]_[state]_[expectation]

func test_loadUsers_whenNetworkFails_showsError() { ... }
func test_deleteUser_withValidId_removesFromList() { ... }
func testLoginViewModel_withEmptyEmail_disablesLoginButton() { ... }
```

---

## Testing ViewModels

```swift
@MainActor
final class LoginViewModelTests: XCTestCase {
    var sut: LoginViewModel!
    var mockAuth: MockAuthService!
    
    override func setUp() {
        super.setUp()
        mockAuth = MockAuthService()
        sut = LoginViewModel(authService: mockAuth)
    }
    
    override func tearDown() {
        sut = nil
        mockAuth = nil
        super.tearDown()
    }
    
    func test_isLoginEnabled_withEmptyEmail_returnsFalse() {
        sut.email = ""
        sut.password = "password123"
        XCTAssertFalse(sut.isLoginEnabled)
    }
    
    func test_isLoginEnabled_withValidCredentials_returnsTrue() {
        sut.email = "test@example.com"
        sut.password = "password123"
        XCTAssertTrue(sut.isLoginEnabled)
    }
    
    func test_login_withSuccess_updatesIsLoggedIn() async {
        sut.email = "test@example.com"
        sut.password = "password123"
        mockAuth.shouldSucceed = true
        
        await sut.login()
        
        XCTAssertTrue(sut.isLoggedIn)
        XCTAssertNil(sut.errorMessage)
    }
    
    func test_login_withFailure_showsError() async {
        mockAuth.shouldSucceed = false
        mockAuth.error = AuthError.invalidCredentials
        
        await sut.login()
        
        XCTAssertFalse(sut.isLoggedIn)
        XCTAssertNotNil(sut.errorMessage)
    }
}
```

---

## TDD — Test-Driven Development

Red → Green → Refactor cycle:

1. **Red**: Write a failing test for new behavior
2. **Green**: Write minimum code to make it pass
3. **Refactor**: Clean up while keeping tests green

```swift
// TDD example: implementing a CurrencyFormatter

// Step 1 — RED: Write failing test first
func test_formatAmount_withUSD_returnsFormattedString() {
    let formatter = CurrencyFormatter()
    let result = formatter.format(1234.5, currency: "USD")
    XCTAssertEqual(result, "$1,234.50")  // Fails — CurrencyFormatter doesn't exist yet
}

// Step 2 — GREEN: Minimal implementation
struct CurrencyFormatter {
    func format(_ amount: Double, currency: String) -> String {
        let f = NumberFormatter()
        f.numberStyle = .currency
        f.currencyCode = currency
        return f.string(from: NSNumber(value: amount)) ?? ""
    }
}

// Step 3 — REFACTOR: Improve without breaking tests
// e.g., cache formatter, support locale
```

> 🎯 **Interview Answer:** "TDD drives better design because you write the API before the implementation — forcing you to think about how the code will be used. Tests become the specification. The red-green-refactor cycle ensures you write minimal code and have tests for every behavior before moving on."

---

## What to Test (and What Not To)

```
✅ Test:
- Business logic (ViewModel methods, Use Cases, calculations)
- Error paths and edge cases
- State transitions
- Data transformations
- Protocol/interface contracts

❌ Don't test:
- Private implementation details (test behavior, not implementation)
- Third-party libraries
- Simple data models (struct with no logic)
- UI layout (use snapshot tests for that)
- Framework code (XCTest, URLSession internals)
```

---

## Code Coverage

```bash
# Enable in scheme: Edit Scheme → Test → Code Coverage → check "Gather coverage"
# Target: 70-80% for business logic, not 100% (not worth it)
```

---

## Interview Q&A

**Q: Describe the Arrange-Act-Assert pattern.**  
A: It's the standard structure for test methods. Arrange: set up the system under test and its dependencies (mocks, initial state). Act: call the method being tested. Assert: verify the outcome matches expectation. This keeps tests readable and focused on a single behavior.

**Q: What should you test in a ViewModel?**  
A: State transitions (loading → success/failure), computed properties (isLoginEnabled), side effects via mock verification (did the service get called?), error handling, and edge cases. Don't test private helpers — test the public API.

**Q: What's `@testable import`?**  
A: Adding `@testable` to an import statement gives the test target access to `internal` declarations in the module — you don't need to make everything `public` just for tests. It only works with debug builds.

**Q: Should you aim for 100% code coverage?**  
A: No. 100% coverage is neither achievable nor desirable — it's expensive, brittle, and often tests implementation details. Aim for ~70-80% on business logic. Focus on covering all business rules, error paths, and state transitions. Don't count test quality by coverage percentage alone.

---

## Quick Revision

- XCTest: `setUp` (before each), `tearDown` (after each), `test*` methods
- AAA: Arrange (setup) → Act (call method) → Assert (verify)
- `try XCTUnwrap()`: safely unwrap optional, fail test if nil
- `@testable import`: access internal declarations in test target
- TDD: Red (failing test) → Green (minimal pass) → Refactor
- sut = "System Under Test" — conventional naming
- Test behavior, not implementation; mock dependencies via protocols
