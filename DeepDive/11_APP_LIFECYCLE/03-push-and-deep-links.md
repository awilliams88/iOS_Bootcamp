# App Lifecycle — Push Notifications & Deep Links

---

## Push Notifications (APNs)

### Setup

```swift
// 1. Request permission
UNUserNotificationCenter.current().requestAuthorization(
    options: [.alert, .badge, .sound]
) { granted, error in
    guard granted else { return }
    DispatchQueue.main.async {
        UIApplication.shared.registerForRemoteNotifications()
    }
}

// 2. Get device token (AppDelegate)
func application(_ application: UIApplication,
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let tokenString = deviceToken.map { String(format: "%02x", $0) }.joined()
    // Send tokenString to your server
}
```

### Handling Notifications

```swift
class NotificationDelegate: NSObject, UNUserNotificationCenterDelegate {
    
    // Notification received while app is FOREGROUND
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        // Show banner even while app is open
        completionHandler([.banner, .badge, .sound])
    }
    
    // User tapped a notification (foreground or background)
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        didReceive response: UNNotificationResponse,
        withCompletionHandler completionHandler: @escaping () -> Void
    ) {
        let userInfo = response.notification.request.content.userInfo
        
        if let type = userInfo["type"] as? String {
            switch type {
            case "message": navigateToMessages(userInfo: userInfo)
            case "order":   navigateToOrder(userInfo: userInfo)
            default: break
            }
        }
        
        completionHandler()
    }
}

// Register in AppDelegate
UNUserNotificationCenter.current().delegate = NotificationDelegate.shared
```

### Local Notifications

```swift
func scheduleLocalNotification(title: String, body: String, after seconds: TimeInterval) {
    let content = UNMutableNotificationContent()
    content.title = title
    content.body = body
    content.sound = .default
    content.badge = 1
    
    let trigger = UNTimeIntervalNotificationTrigger(timeInterval: seconds, repeats: false)
    
    // Calendar trigger
    var dateComponents = DateComponents()
    dateComponents.hour = 9
    dateComponents.minute = 0
    let calendarTrigger = UNCalendarNotificationTrigger(dateMatching: dateComponents, repeats: true)
    
    let request = UNNotificationRequest(
        identifier: UUID().uuidString,
        content: content,
        trigger: trigger
    )
    
    UNUserNotificationCenter.current().add(request)
}
```

---

## Deep Links — URL Schemes

Custom URL scheme (`myapp://`):

```xml
<!-- Info.plist -->
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

```swift
// SceneDelegate handles while app running
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard let url = URLContexts.first?.url else { return }
    // myapp://profile/123 → navigate to profile 123
    DeepLinkHandler.shared.handle(url: url)
}

// Parser
struct DeepLinkHandler {
    static let shared = DeepLinkHandler()
    
    func handle(url: URL) {
        guard url.scheme == "myapp" else { return }
        
        switch url.host {
        case "profile":
            let userId = url.pathComponents.dropFirst().first
            navigateToProfile(id: userId)
        case "article":
            let articleId = url.pathComponents.dropFirst().first
            navigateToArticle(id: articleId)
        case "settings":
            navigateToSettings()
        default:
            break
        }
    }
}
```

---

## Universal Links

Universal links use HTTPS URLs — no custom scheme needed. Requires server-side `apple-app-site-association` file.

```json
// https://example.com/.well-known/apple-app-site-association
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appIDs": ["TEAMID.com.example.app"],
                "components": [
                    { "/": "/profile/*" },
                    { "/": "/articles/*" },
                    { "/": "/settings", "comment": "Settings page" }
                ]
            }
        ]
    }
}
```

```swift
// Handle in SceneDelegate
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else { return }
    
    // https://example.com/profile/123
    DeepLinkHandler.shared.handleUniversalLink(url: url)
}
```

### URL Scheme vs Universal Links

| Feature | URL Scheme | Universal Links |
|---------|-----------|-----------------|
| Format | `myapp://path` | `https://example.com/path` |
| Server file needed | No | Yes (AASA) |
| Opens in browser if app not installed | No (fails) | Yes (falls back to web) |
| Security | Any app can register same scheme | Only your app (server verified) |
| Recommended | Legacy | ✅ Preferred |

---

## Interview Q&A

**Q: What's the difference between URL schemes and Universal Links?**  
A: URL schemes use a custom `myapp://` protocol registered in Info.plist. Any app could theoretically register the same scheme, making them a security concern. Universal Links use regular HTTPS URLs validated against an `apple-app-site-association` file on your server — only your app can handle those URLs, and they fall back to the website if the app isn't installed.

**Q: How do you handle a push notification when the app is in the foreground?**  
A: Implement `UNUserNotificationCenterDelegate.userNotificationCenter(_:willPresent:withCompletionHandler:)` and call the completion handler with presentation options (`.banner`, `.sound`, etc.). By default, iOS suppresses alerts for foreground apps — you must explicitly opt in.

**Q: A user taps a push notification when the app is closed. How do you handle navigation?**  
A: The notification response is available in `SceneDelegate.scene(_:willConnectTo:options:)` via `connectionOptions.notificationResponse`. Parse the userInfo and navigate to the appropriate screen after the app's UI is set up.

---

## Quick Revision

- APNs: register → device token → send to server
- `willPresent`: notification received while foreground — opt in with completion handler options
- `didReceive(response:)`: user tapped notification — handle navigation
- URL scheme: `myapp://host/path` — no server required, any app can register same scheme
- Universal Links: `https://example.com/path` — requires AASA file, secure, falls back to web
- `scene(_:openURLContexts:)`: URL scheme while app running
- `scene(_:continue:)`: universal link while app running
- Launch handling: check `connectionOptions` in `scene(_:willConnectTo:options:)`
