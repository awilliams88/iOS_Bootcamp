# Architecture — VIPER & Clean Architecture

---

## VIPER

VIPER is an acronym for View, Interactor, Presenter, Entity, Router. It strictly applies the Single Responsibility Principle.

```
View ←→ Presenter ←→ Interactor ←→ Entity (Model)
              ↓
           Router (Navigation)
```

### Layer Responsibilities

| Layer | Responsibility | UIKit? | Network? |
|-------|---------------|--------|----------|
| View | Display, user input | ✅ | ❌ |
| Presenter | Formatting, logic bridge | ❌ | ❌ |
| Interactor | Business rules, data fetching | ❌ | ✅ |
| Entity | Data models | ❌ | ❌ |
| Router | Navigation, module creation | ✅ | ❌ |

```swift
// VIPER — User List Module

// MARK: — Protocols
protocol UserListViewProtocol: AnyObject {
    func displayUsers(_ users: [UserViewModel])
    func displayError(_ message: String)
    func setLoading(_ loading: Bool)
}

protocol UserListPresenterProtocol: AnyObject {
    func viewDidLoad()
    func didSelectUser(at index: Int)
}

protocol UserListInteractorProtocol: AnyObject {
    func fetchUsers() async throws -> [User]
}

protocol UserListRouterProtocol: AnyObject {
    func navigateToUserDetail(_ user: User)
}

// MARK: — View
class UserListViewController: UIViewController, UserListViewProtocol {
    var presenter: UserListPresenterProtocol!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        presenter.viewDidLoad()
    }
    
    func displayUsers(_ users: [UserViewModel]) { /* reload table */ }
    func displayError(_ message: String) { /* show alert */ }
    func setLoading(_ loading: Bool) { /* toggle spinner */ }
}

// MARK: — Presenter
final class UserListPresenter: UserListPresenterProtocol {
    weak var view: UserListViewProtocol?
    var interactor: UserListInteractorProtocol!
    var router: UserListRouterProtocol!
    
    private var users: [User] = []
    
    func viewDidLoad() {
        view?.setLoading(true)
        Task {
            do {
                users = try await interactor.fetchUsers()
                let vms = users.map { UserViewModel(user: $0) }
                await MainActor.run {
                    view?.setLoading(false)
                    view?.displayUsers(vms)
                }
            } catch {
                await MainActor.run {
                    view?.setLoading(false)
                    view?.displayError(error.localizedDescription)
                }
            }
        }
    }
    
    func didSelectUser(at index: Int) {
        router.navigateToUserDetail(users[index])
    }
}

// MARK: — Interactor
final class UserListInteractor: UserListInteractorProtocol {
    private let userService: UserServiceProtocol
    
    init(userService: UserServiceProtocol) {
        self.userService = userService
    }
    
    func fetchUsers() async throws -> [User] {
        try await userService.fetchUsers()
    }
}

// MARK: — Router
final class UserListRouter: UserListRouterProtocol {
    weak var viewController: UIViewController?
    
    static func createModule(userService: UserServiceProtocol) -> UIViewController {
        let view = UserListViewController()
        let interactor = UserListInteractor(userService: userService)
        let router = UserListRouter()
        let presenter = UserListPresenter()
        
        view.presenter = presenter
        presenter.view = view
        presenter.interactor = interactor
        presenter.router = router
        router.viewController = view
        
        return view
    }
    
    func navigateToUserDetail(_ user: User) {
        let detailVC = UserDetailRouter.createModule(user: user)
        viewController?.navigationController?.pushViewController(detailVC, animated: true)
    }
}
```

### VIPER Tradeoffs

| Pro | Con |
|-----|-----|
| Strict separation of concerns | High boilerplate (5+ files per screen) |
| Excellent testability | Steep learning curve |
| Clear module boundaries | Overkill for small apps |
| Large team friendly | Lots of protocols |

---

## Clean Architecture (Uncle Bob)

Clean Architecture organizes code in concentric layers with a **dependency rule**: outer layers depend on inner layers, never the reverse.

```
       ┌─────────────────────────────┐
       │   Frameworks & Drivers      │  (UIKit, URLSession, CoreData)
       │  ┌───────────────────────┐  │
       │  │   Interface Adapters  │  │  (ViewModels, Presenters, Gateways)
       │  │  ┌─────────────────┐  │  │
       │  │  │  Application    │  │  │  (Use Cases / Interactors)
       │  │  │  ┌───────────┐  │  │  │
       │  │  │  │  Entities │  │  │  │  (Business rules, Domain Models)
       │  │  │  └───────────┘  │  │  │
       │  │  └─────────────────┘  │  │
       │  └───────────────────────┘  │
       └─────────────────────────────┘
```

### iOS Clean Architecture Layers

```swift
// DOMAIN LAYER — pure Swift, no frameworks

// Entity (business rule model)
struct User {
    let id: UUID
    let name: String
    let email: String
}

// Use Case
protocol FetchUsersUseCaseProtocol {
    func execute() async throws -> [User]
}

final class FetchUsersUseCase: FetchUsersUseCaseProtocol {
    private let repository: UserRepositoryProtocol
    
    init(repository: UserRepositoryProtocol) {
        self.repository = repository
    }
    
    func execute() async throws -> [User] {
        let users = try await repository.fetchUsers()
        return users.filter { $0.isActive }  // Business rule: only active users
    }
}

// Repository interface (defined in domain, implemented in data layer)
protocol UserRepositoryProtocol {
    func fetchUsers() async throws -> [User]
    func saveUser(_ user: User) async throws
}


// DATA LAYER — implements repository using network/database

final class UserRepository: UserRepositoryProtocol {
    private let remoteDataSource: UserRemoteDataSourceProtocol
    private let localDataSource: UserLocalDataSourceProtocol
    
    func fetchUsers() async throws -> [User] {
        // Try cache first
        if let cached = try? await localDataSource.getUsers(), !cached.isEmpty {
            // Refresh in background
            Task { try? await refreshFromNetwork() }
            return cached
        }
        return try await refreshFromNetwork()
    }
    
    @discardableResult
    private func refreshFromNetwork() async throws -> [User] {
        let dtos = try await remoteDataSource.fetchUsers()
        let users = dtos.map { User(dto: $0) }  // Map DTO → Domain model
        try await localDataSource.saveUsers(users)
        return users
    }
    
    func saveUser(_ user: User) async throws {
        try await localDataSource.saveUser(user)
    }
}


// PRESENTATION LAYER — ViewModels using Use Cases

@MainActor
final class UserListViewModel: ObservableObject {
    @Published private(set) var users: [User] = []
    @Published private(set) var isLoading = false
    
    private let fetchUsersUseCase: FetchUsersUseCaseProtocol
    
    init(fetchUsersUseCase: FetchUsersUseCaseProtocol) {
        self.fetchUsersUseCase = fetchUsersUseCase
    }
    
    func loadUsers() async {
        isLoading = true
        defer { isLoading = false }
        users = (try? await fetchUsersUseCase.execute()) ?? []
    }
}
```

### Dependency Rule in Practice

```
ViewModel → UseCase → Repository Protocol (← Repository Implementation ← DataSource)
                                                         ↑
                                               (Network / Database)
All arrows point inward. Network/DB depend on domain, not vice versa.
```

---

## Interview Q&A

**Q: What problem does VIPER solve that MVVM doesn't?**  
A: VIPER draws even stronger boundaries — specifically, it separates navigation (Router), business logic (Interactor), and presentation formatting (Presenter) into distinct objects. This is valuable in large teams where different engineers own different layers. MVVM can still end up with fat ViewModels that mix business logic and presentation logic.

**Q: Explain the Dependency Rule in Clean Architecture.**  
A: Source code dependencies must only point inward — from outer layers (frameworks, UI) toward inner layers (use cases, entities). The inner domain layer has no knowledge of UIKit, databases, or network. This makes the domain logic independently testable and swappable.

**Q: What is a Use Case in Clean Architecture?**  
A: A Use Case (also called Interactor) encapsulates a single business operation — like "fetch active users" or "place order." It orchestrates data flow between repositories, applies business rules, and returns results. It has no knowledge of how data is displayed or stored.

**Q: What is the Repository pattern?**  
A: A Repository provides a clean abstraction over data access. The domain layer defines a repository protocol; the data layer implements it with network and/or local storage. The domain never knows whether data comes from an API or a local database — it just calls the repository.

---

## Quick Revision

- VIPER: View, Interactor, Presenter, Entity, Router — strict SRP
- Router creates modules and handles navigation
- Clean Architecture: concentric rings, dependency rule (inward only)
- Domain layer: Entities + Use Cases — no frameworks
- Data layer: Repository implementations — network + cache
- Repository pattern: abstracts data source behind a protocol
- Use Case: single business operation, orchestrates repositories
