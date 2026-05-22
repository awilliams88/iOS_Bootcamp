# Performance — Memory Leaks & Retain Cycles

---

## Finding Leaks

### Instruments → Leaks

1. Profile → Leaks template
2. Perform actions (navigate to screen, interact, navigate back)
3. Press the "Mark Generation" button (red M)
4. Perform more actions
5. Leaks instrument shows objects allocated but not deallocated since last generation mark

### Debug Memory Graph

In Xcode during a debug session:
```
Debug → Memory Graph (⌃⌘M) 
or click the "memory graph" button in debug bar
```

Shows all living objects and their reference graph. Look for:
- Your ViewModel/ViewController being referenced when it shouldn't be
- Unexpected retain chains

---

## Common Retain Cycle Patterns

### 1. Closure Capturing self Strongly

```swift
// ❌ Retain cycle — closure captures self strongly
class UserViewModel {
    var onDataLoaded: (() -> Void)?
    var timer: Timer?
    
    func startPolling() {
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { timer in
            self.fetchData()  // self is retained by the Timer closure
        }
    }
    // UserViewModel owns timer, timer closure owns UserViewModel = cycle
    
    deinit {
        print("UserViewModel deinit")  // Never called!
    }
}

// ✅ Fix with [weak self]
func startPolling() {
    timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
        self?.fetchData()
    }
}
```

### 2. Delegate Retain Cycle

```swift
// ❌ Strong delegate creates cycle
class UserListViewController: UIViewController {
    var dataSource: UserDataSource?
}

class UserDataSource: NSObject, UITableViewDataSource {
    var delegate: UserListViewController  // Strong reference back to VC!
}

// ✅ Weak delegate
class UserDataSource: NSObject, UITableViewDataSource {
    weak var delegate: UserListViewController?  // Weak — breaks cycle
}
```

### 3. Escaping Closure in Class

```swift
class NetworkManager {
    var completionHandlers: [() -> Void] = []
    
    func store(handler: @escaping () -> Void) {
        completionHandlers.append(handler)
    }
}

class ViewController: UIViewController {
    let networkManager = NetworkManager()
    
    override func viewDidLoad() {
        // ❌ ViewController captured by closure stored in NetworkManager
        networkManager.store {
            self.updateUI()  // NetworkManager → closure → ViewController → NetworkManager? No, but VC won't deinit
        }
        
        // ✅ Break the cycle
        networkManager.store { [weak self] in
            self?.updateUI()
        }
    }
}
```

### 4. NotificationCenter Retain

```swift
class ViewController: UIViewController {
    // ❌ Old-style observer — must be removed manually or leaks
    override func viewDidLoad() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleNotification),
            name: .didReceiveData,
            object: nil
        )
    }
    
    deinit {
        NotificationCenter.default.removeObserver(self)  // Required!
    }
    
    // ✅ Modern closure-based — store token and remove on deinit
    var notificationToken: Any?
    
    override func viewDidLoad() {
        notificationToken = NotificationCenter.default.addObserver(
            forName: .didReceiveData,
            object: nil,
            queue: .main
        ) { [weak self] _ in
            self?.handleNotification()
        }
    }
    
    deinit {
        if let token = notificationToken {
            NotificationCenter.default.removeObserver(token)
        }
    }
}
```

---

## weak vs unowned — Choosing Correctly

```swift
// weak: when the referenced object CAN become nil during the closure/relationship lifetime
// - Use for delegates, view controllers referenced by view models
// - Access is optional: self?.doSomething()

// unowned: when the referenced object CANNOT become nil (its lifetime >= the closure's)
// - Use when closure and referenced object have same lifetime
// - Access is non-optional: self.doSomething() — CRASHES if self IS nil

class UserService {
    func fetchUser(id: UUID, completion: @escaping (User) -> Void) {
        Task {
            let user = try await apiClient.fetchUser(id: id)
            completion(user)
        }
    }
}

class UserViewModel {
    let service: UserService
    
    // unowned safe here — completion closure doesn't outlive UserViewModel
    // because UserViewModel owns the service
    func loadUser(id: UUID) {
        service.fetchUser(id: id) { [unowned self] user in
            self.currentUser = user  // Crashes if self is deallocated first
        }
    }
    
    // weak is always safer
    func loadUserSafe(id: UUID) {
        service.fetchUser(id: id) { [weak self] user in
            self?.currentUser = user  // Safe — does nothing if self is nil
        }
    }
}
```

> 🎯 **Interview Answer:** "Use `weak` when the referenced object can be deallocated before the closure/relationship is done — it becomes nil safely. Use `unowned` when you're certain the object will outlive the closure — it doesn't have the overhead of optional checking but crashes if the assumption is wrong. In practice, `weak` is almost always the safer choice. Only use `unowned` when you have a guaranteed lifetime relationship, like `[unowned self]` in `init`-time closures."

---

## Detecting deinit

Add `print` or breakpoint in `deinit` to verify objects are released:

```swift
class UserViewModel {
    deinit {
        print("✅ UserViewModel released")
        // If this never prints after navigating away → leak
    }
}
```

---

## Combine Retain Cycles

```swift
class FeedViewModel {
    private var cancellables = Set<AnyCancellable>()
    
    func bind() {
        // ❌ Self retained by sink
        timer.sink { _ in
            self.refresh()  // FeedViewModel retained by cancellable stored in self
        }
        .store(in: &cancellables)
        // This is actually OK in most cases since cancellables are cancelled on deinit
        
        // ❌ Problematic: storing cancellable in external object
        someExternalObject.cancellables.insert(
            myPublisher.sink { [strong self!] _ in  // External strong reference
                self.handle()
            }
        )
        
        // ✅ Always use [weak self] in sink closures
        timer.sink { [weak self] _ in
            self?.refresh()
        }
        .store(in: &cancellables)
    }
}
```

---

## Interview Q&A

**Q: How do you find a memory leak in an iOS app?**  
A: Three approaches: (1) Instruments → Leaks template — run the app, navigate around, check for unreleased objects. (2) Debug Memory Graph in Xcode — shows the live object graph and reference chains. (3) Add `deinit { print("released") }` to key objects and verify they print when you navigate away.

**Q: What is a retain cycle and how do you fix it?**  
A: A retain cycle is when two or more objects hold strong references to each other, preventing both from being deallocated. Fix: break the cycle by making one reference `weak` (can become nil) or `unowned` (guaranteed non-nil). Common fix: `[weak self]` in closures, `weak var delegate` in delegates.

**Q: When is `unowned` dangerous?**  
A: When the referenced object is deallocated before the closure is called. `unowned` doesn't nil out the reference — accessing a deallocated unowned reference is a crash. Only use `unowned` when you can guarantee the referenced object outlives the closure.

---

## Quick Revision

- Retain cycle: mutual strong references prevent deallocation
- Fix: `[weak self]` in closures, `weak var` for delegates
- `weak`: optional, safe (nil on deallocation)
- `unowned`: non-optional, crashes if reference outlives object
- `deinit { print(...) }`: cheap leak detector
- Instruments Leaks + Debug Memory Graph: professional leak detection
- NotificationCenter: always remove observer (or store token + remove in deinit)
- Combine: always `[weak self]` in sink closures
