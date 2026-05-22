# Advanced — Objective-C Interoperability

Swift and Objective-C can coexist in the same project. Understanding the bridge is essential for working with legacy codebases.

---

## Exposing Swift to Objective-C

```swift
// Classes must inherit from NSObject to be visible to ObjC
@objc class UserManager: NSObject {
    
    // Properties exposed to ObjC
    @objc var currentUser: User?
    @objc private(set) var isLoggedIn: Bool = false
    
    // Methods exposed to ObjC
    @objc func login(username: String, password: String) {
        // ...
    }
    
    // Custom ObjC selector name
    @objc(logoutWithCompletion:)
    func logout(completion: @escaping () -> Void) {
        // ...
    }
    
    // ObjC-compatible enum (must be Int)
    @objc enum LoginState: Int {
        case loggedOut = 0
        case loggingIn = 1
        case loggedIn = 2
    }
}

// @objcMembers — expose all members without individual @objc
@objcMembers
class AnalyticsManager: NSObject {
    var userID: String?
    func track(event: String) { }
}
```

---

## Calling Objective-C from Swift

ObjC files are automatically available via the bridging header:

```objc
// MyApp-Bridging-Header.h
#import "LegacyUserManager.h"
#import "ObjCNetworkManager.h"
```

```swift
// Use ObjC types directly in Swift
let manager = LegacyUserManager()
manager.fetchUsers { users, error in
    guard let users = users as? [LegacyUser] else { return }
    self.users = users.map { User(legacy: $0) }
}
```

### Nullability Annotations

ObjC code without nullability annotations is imported as implicitly unwrapped optionals:

```objc
// ObjC without nullability — dangerous in Swift
- (NSString *)userName;  // Imported as String! (implicitly unwrapped)

// ObjC with nullability — safe
- (nullable NSString *)userName;   // String?
- (nonnull NSString *)displayName; // String (non-optional)

// Audit region — mark entire file
NS_ASSUME_NONNULL_BEGIN
- (NSString *)firstName;  // String
- (nullable NSString *)middleName;  // String?
NS_ASSUME_NONNULL_END
```

---

## KVO in Swift

Key-Value Observing — ObjC runtime mechanism for property observation:

```swift
class Sensor: NSObject {
    // Must be @objc dynamic for KVO
    @objc dynamic var temperature: Double = 0.0
}

class Dashboard {
    private var sensor = Sensor()
    private var observation: NSKeyValueObservation?
    
    func startMonitoring() {
        observation = sensor.observe(\.temperature, options: [.old, .new]) { sensor, change in
            guard let newTemp = change.newValue else { return }
            print("Temperature changed to \(newTemp)")
        }
    }
    
    deinit {
        observation?.invalidate()  // Required for legacy KVO; not needed for new block-based KVO
    }
}
```

> 💡 **Tip:** Prefer Combine or `@Observable` over KVO in new code. KVO is type-unsafe and string-key based in its legacy form.

---

## Selector Pattern

```swift
// Selectors used for target-action, timers, NotificationCenter
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let button = UIButton()
        button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)
        
        let timer = Timer.scheduledTimer(
            target: self,
            selector: #selector(timerFired),
            userInfo: nil,
            repeats: true
        )
    }
    
    @objc func buttonTapped() { ... }  // Must be @objc
    @objc func timerFired() { ... }
}
```

---

## ObjC Runtime

```swift
import ObjectiveC

// Dynamic method resolution
let methods = class_copyMethodList(SomeClass.self, nil)

// Method swizzling (use with extreme caution)
extension UIViewController {
    static func swizzleViewDidAppear() {
        let original = #selector(viewDidAppear(_:))
        let swizzled = #selector(swizzled_viewDidAppear(_:))
        
        guard let originalMethod = class_getInstanceMethod(Self.self, original),
              let swizzledMethod = class_getInstanceMethod(Self.self, swizzled) else { return }
        
        method_exchangeImplementations(originalMethod, swizzledMethod)
    }
    
    @objc func swizzled_viewDidAppear(_ animated: Bool) {
        swizzled_viewDidAppear(animated)  // Calls original (names are swapped)
        // Extra behavior here
    }
}
```

> ⚠️ **Pitfall:** Method swizzling relies on the ObjC runtime and can break unexpectedly across iOS updates. Only use it when absolutely necessary (analytics, debugging). Modern alternatives: subclassing, protocol extensions, aspect-oriented libraries.

---

## Interview Q&A

**Q: How do you expose a Swift class to Objective-C?**  
A: The class must inherit from `NSObject` and be marked `@objc` (or `@objcMembers` for all members). Properties and methods use ObjC-compatible types. Swift features without ObjC equivalents (generics, tuples, enums with associated values) can't be directly bridged.

**Q: What is the bridging header?**  
A: A header file (`AppName-Bridging-Header.h`) that imports ObjC headers, making those classes and functions available in Swift without explicit imports. Created automatically by Xcode when you add an ObjC file to a Swift project.

**Q: What is method swizzling and when would you use it?**  
A: Swizzling exchanges two method implementations at runtime using the ObjC runtime. It's used for: analytics (intercept all button taps), debugging (track all VC appearances), and legacy behavior injection. It's fragile — use only as a last resort, always in `+load` or `initialize`, and document it clearly.

**Q: What does `@objc dynamic` mean?**  
A: `@objc` exposes the declaration to ObjC. `dynamic` tells Swift to always use ObjC dynamic dispatch (message sending) for this member, enabling KVO and swizzling to work. Without `dynamic`, Swift might use static dispatch, bypassing the ObjC runtime.

---

## Quick Revision

- `NSObject` subclass + `@objc` to expose Swift to ObjC
- `@objcMembers`: expose all members without individual `@objc`
- Bridging header: import ObjC into Swift
- Nullability: `nullable`, `nonnull`, `NS_ASSUME_NONNULL_BEGIN` for safe bridging
- KVO: `@objc dynamic` + `observe(\.property, options:)` + invalidate on deinit
- `#selector`: reference ObjC method for target-action, timers
- Method swizzling: runtime method exchange — fragile, use sparingly
- `@objc dynamic`: ObjC dynamic dispatch — required for KVO and swizzling
