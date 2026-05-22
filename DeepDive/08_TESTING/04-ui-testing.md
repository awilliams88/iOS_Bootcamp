# Testing — UI Testing & Snapshot Testing

---

## XCUITest Basics

XCUITest drives the app as a user would — tapping, typing, swiping.

```swift
import XCTest

final class LoginUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        continueAfterFailure = false  // Stop on first failure
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]  // Signal to app to use test mode
        app.launchEnvironment = ["API_BASE_URL": "http://localhost:8080"]
        app.launch()
    }
    
    func test_login_withValidCredentials_showsHomeScreen() {
        // Find elements
        let emailField = app.textFields["emailTextField"]
        let passwordField = app.secureTextFields["passwordTextField"]
        let loginButton = app.buttons["loginButton"]
        
        // Wait for element to exist
        XCTAssertTrue(emailField.waitForExistence(timeout: 5))
        
        // Interact
        emailField.tap()
        emailField.typeText("test@example.com")
        
        passwordField.tap()
        passwordField.typeText("password123")
        
        loginButton.tap()
        
        // Assert result
        let homeTitle = app.navigationBars["Home"]
        XCTAssertTrue(homeTitle.waitForExistence(timeout: 5))
    }
    
    func test_login_withInvalidCredentials_showsError() {
        app.textFields["emailTextField"].tap()
        app.textFields["emailTextField"].typeText("wrong@example.com")
        app.secureTextFields["passwordTextField"].tap()
        app.secureTextFields["passwordTextField"].typeText("wrongpass")
        app.buttons["loginButton"].tap()
        
        let errorAlert = app.alerts["Login Failed"]
        XCTAssertTrue(errorAlert.waitForExistence(timeout: 5))
    }
}
```

---

## Accessibility Identifiers

Set identifiers in your views to make UI tests reliable:

```swift
// UIKit
emailTextField.accessibilityIdentifier = "emailTextField"
loginButton.accessibilityIdentifier = "loginButton"

// SwiftUI
TextField("Email", text: $email)
    .accessibilityIdentifier("emailTextField")

Button("Login", action: login)
    .accessibilityIdentifier("loginButton")
```

> ⚠️ **Pitfall:** Never use display text (`app.buttons["Sign In"]`) in UI tests — text changes break tests. Always use `accessibilityIdentifier`.

---

## Launch Arguments for Test Mode

```swift
// In the app — detect test mode and skip animation, use mock data
override func viewDidLoad() {
    super.viewDidLoad()
    if CommandLine.arguments.contains("--uitesting") {
        UIView.setAnimationsEnabled(false)  // Faster, more reliable tests
    }
}

// Or in AppDelegate
if ProcessInfo.processInfo.arguments.contains("--uitesting") {
    // Use in-memory/mock backend
    container.register(UserServiceProtocol.self) { _ in MockUserService() }
}
```

---

## Snapshot Testing

Snapshot tests render a view and compare it against a saved reference image:

```swift
// Using swift-snapshot-testing (pointfreeco/swift-snapshot-testing)
import SnapshotTesting

final class UserRowSnapshotTests: XCTestCase {
    func test_userRow_defaultState() {
        let view = UserRow(user: .mock)
        let vc = UIHostingController(rootView: view)
        vc.view.frame = CGRect(x: 0, y: 0, width: 375, height: 80)
        
        assertSnapshot(of: vc, as: .image(on: .iPhone13))
    }
    
    func test_userRow_premiumBadge() {
        let user = User.mock(isPremium: true)
        let view = UserRow(user: user)
        
        assertSnapshot(of: UIHostingController(rootView: view), as: .image(on: .iPhone13))
    }
    
    // Test dark mode
    func test_userRow_darkMode() {
        let view = UserRow(user: .mock)
        let vc = UIHostingController(rootView: view)
        vc.overrideUserInterfaceStyle = .dark
        
        assertSnapshot(of: vc, as: .image(on: .iPhone13))
    }
}
```

### Snapshot Test Workflow

1. First run: **record mode** — creates reference images in `__Snapshots__` folder
2. Subsequent runs: **compare mode** — compares against reference, fails on pixel difference
3. Update references: set `record: true` or delete snapshot files

```swift
// Force record mode (update snapshots)
assertSnapshot(of: vc, as: .image, record: true)
```

---

## Page Object Pattern for UI Tests

Encapsulate UI test logic for reuse:

```swift
// Page Object
struct LoginPage {
    let app: XCUIApplication
    
    var emailField:    XCUIElement { app.textFields["emailTextField"] }
    var passwordField: XCUIElement { app.secureTextFields["passwordTextField"] }
    var loginButton:   XCUIElement { app.buttons["loginButton"] }
    var errorAlert:    XCUIElement { app.alerts.firstMatch }
    
    @discardableResult
    func enterEmail(_ email: String) -> Self {
        emailField.tap()
        emailField.typeText(email)
        return self
    }
    
    @discardableResult
    func enterPassword(_ password: String) -> Self {
        passwordField.tap()
        passwordField.typeText(password)
        return self
    }
    
    @discardableResult
    func tapLogin() -> HomePage {
        loginButton.tap()
        return HomePage(app: app)
    }
}

// Clean test using page object
func test_validLogin_navigatesToHome() {
    let home = LoginPage(app: app)
        .enterEmail("test@example.com")
        .enterPassword("password123")
        .tapLogin()
    
    XCTAssertTrue(home.navigationTitle.waitForExistence(timeout: 5))
}
```

---

## Interview Q&A

**Q: What's the biggest challenge with UI tests?**  
A: Flakiness — UI tests are inherently slower and timing-dependent. Common fixes: disable animations in test mode (`UIView.setAnimationsEnabled(false)`), use `waitForExistence(timeout:)` instead of sleep, use accessibility identifiers instead of text, and run with a mock backend to avoid network flakiness.

**Q: When would you use snapshot tests instead of UI tests?**  
A: Snapshot tests are for visual regression — catching unintended UI changes. UI tests are for interaction flows. Snapshots run in unit test targets (fast, no simulator), while UI tests run in a separate target (slow, requires simulator). Use snapshots for component libraries and design-critical screens; UI tests for critical user flows.

**Q: What is the Page Object pattern in UI testing?**  
A: It encapsulates UI element access and interactions in a dedicated object per screen. Tests compose page objects rather than directly accessing `app.buttons["x"]`. This makes tests readable, reduces duplication, and isolates element changes to one place.

---

## Quick Revision

- `XCUIApplication`: test entry point; `.launch()` to start
- `accessibilityIdentifier`: reliable element lookup (not display text)
- `waitForExistence(timeout:)`: wait for element to appear (better than sleep)
- `continueAfterFailure = false`: stop test on first failure
- Launch arguments: `--uitesting` to signal test mode (disable animations, mock backend)
- Snapshot testing: pixel-level visual regression, record then compare
- Page Object: encapsulate screen interactions, improves readability & reuse
