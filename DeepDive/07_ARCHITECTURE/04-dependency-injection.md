# Architecture — Dependency Injection

Dependency Injection (DI) makes code testable, modular, and flexible by providing dependencies from outside rather than creating them internally.

---

## Constructor Injection (Preferred)

```swift
// ❌ Hard dependency — impossible to test without hitting real network
final class UserViewModel {
    private let service = UserService()  // Created internally
    
    func loadUsers() async { ... }
}

// ✅ Constructor injection — dependency provided externally
final class UserViewModel {
    private let service: UserServiceProtocol  // Protocol, not concrete type
    
    init(service: UserServiceProtocol = UserService()) {  // Default for convenience
        self.service = service
    }
}

// In production
let vm = UserViewModel(service: UserService())

// In tests
let vm = UserViewModel(service: MockUserService())
```

---

## Property Injection

```swift
// When you can't use constructor (e.g., storyboard-instantiated VCs)
class UserViewController: UIViewController {
    var viewModel: UserViewModelProtocol!  // Injected after instantiation
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // viewModel is expected to be set before viewDidLoad
    }
}

// Usage
let vc = storyboard.instantiate(UserViewController.self)
vc.viewModel = UserViewModel(service: UserService())
```

---

## Protocol Abstractions

Define dependencies as protocols so they can be mocked:

```swift
// Protocol
protocol UserServiceProtocol {
    func fetchUsers() async throws -> [User]
    func deleteUser(id: UUID) async throws
}

// Production implementation
final class UserService: UserServiceProtocol {
    private let client: APIClientProtocol
    
    init(client: APIClientProtocol) { self.client = client }
    
    func fetchUsers() async throws -> [User] {
        try await client.fetch(.users(page: 1))
    }
    
    func deleteUser(id: UUID) async throws {
        try await client.send(.deleteUser(id: id))
    }
}

// Mock for tests
final class MockUserService: UserServiceProtocol {
    var stubbedUsers: [User] = []
    var deleteCalledWith: UUID?
    var shouldThrow: Error?
    
    func fetchUsers() async throws -> [User] {
        if let error = shouldThrow { throw error }
        return stubbedUsers
    }
    
    func deleteUser(id: UUID) async throws {
        deleteCalledWith = id
        if let error = shouldThrow { throw error }
    }
}
```

---

## DI Container

For large apps, a container manages object creation and lifetime:

```swift
// Simple service locator / DI container
final class DependencyContainer {
    static let shared = DependencyContainer()
    
    // Singleton lifetime
    private lazy var apiClient: APIClientProtocol = {
        APIClient(baseURL: APIEnvironment.current.baseURL, tokenProvider: tokenProvider)
    }()
    
    private lazy var tokenProvider: TokenProvider = {
        KeychainTokenProvider()
    }()
    
    // Factory — creates new instance each time
    func makeUserService() -> UserServiceProtocol {
        UserService(client: apiClient)
    }
    
    func makeUserViewModel() -> UserListViewModel {
        UserListViewModel(service: makeUserService())
    }
    
    // Feature factory
    func makeUserListViewController() -> UserListViewController {
        let vm = makeUserViewModel()
        return UserListViewController(viewModel: vm)
    }
}

// Usage
let vc = DependencyContainer.shared.makeUserListViewController()
```

> ⚠️ **Pitfall:** The Service Locator pattern (calling a global container from inside a class) is an anti-pattern — it hides dependencies. Prefer true constructor injection where the container is only used at composition root (AppDelegate/SceneDelegate).

---

## Composition Root

The composition root is the single place where the entire object graph is assembled:

```swift
// SceneDelegate — composition root for UIKit
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?
    private var appCoordinator: AppCoordinator!
    
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
               options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = scene as? UIWindowScene else { return }
        
        // Build the entire dependency graph here
        let container = DependencyContainer()
        let navController = UINavigationController()
        appCoordinator = AppCoordinator(
            navigationController: navController,
            userService: container.makeUserService(),
            authService: container.makeAuthService()
        )
        
        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = navController
        window?.makeKeyAndVisible()
        
        appCoordinator.start()
    }
}
```

---

## Testing with DI

```swift
class UserListViewModelTests: XCTestCase {
    var sut: UserListViewModel!
    var mockService: MockUserService!
    
    override func setUp() {
        mockService = MockUserService()
        sut = UserListViewModel(service: mockService)
    }
    
    func testLoadUsersSuccess() async {
        // Arrange
        mockService.stubbedUsers = [User(id: UUID(), name: "Alice")]
        
        // Act
        await sut.loadUsers()
        
        // Assert
        XCTAssertEqual(sut.users.count, 1)
        XCTAssertEqual(sut.users[0].name, "Alice")
        XCTAssertFalse(sut.isLoading)
    }
    
    func testLoadUsersFailure() async {
        mockService.shouldThrow = NetworkError.unauthorized
        await sut.loadUsers()
        XCTAssertNil(sut.users.first)
        XCTAssertNotNil(sut.errorMessage)
    }
}
```

---

## Interview Q&A

**Q: What is Dependency Injection and why does it matter?**  
A: DI means providing an object's dependencies from outside rather than creating them internally. It matters for testability (inject mocks), flexibility (swap implementations), and clarity (dependencies are explicit in the initializer). An object that creates its own dependencies is tightly coupled and hard to test.

**Q: What's the difference between constructor injection and property injection?**  
A: Constructor injection provides dependencies via `init` — they're guaranteed to exist from the moment the object is created, making them immutable (`let`). Property injection uses `var` properties set after creation — useful when constructor injection isn't possible (storyboard VCs) but riskier since properties might be nil.

**Q: What's the Service Locator anti-pattern?**  
A: Service Locator is when a class calls a global registry to get its dependencies: `DependencyContainer.shared.makeService()`. This hides dependencies (they're not visible in the init signature), makes code harder to test (requires configuring the global before tests), and creates implicit coupling. Prefer explicit constructor injection.

**Q: How do you handle dependency injection in SwiftUI?**  
A: Using `@EnvironmentObject` for app-wide dependencies, and constructor injection in `@StateObject` initializers. For large apps, a composition root in the `App` struct builds the initial dependency graph and injects it via `environmentObject`.

---

## Quick Revision

- Constructor injection: best practice, dependencies visible in `init`, use `let`
- Property injection: fallback for storyboard VCs, use `var`, may be nil
- Protocol abstractions: depend on protocol, not concrete type
- Composition root: single place where entire graph assembled (SceneDelegate/App)
- Service Locator: anti-pattern — hides dependencies, avoid
- DI container: builds object graph, manages lifetimes (singleton vs factory)
- Mock injection: replace real dependencies with mocks in tests
