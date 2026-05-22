# Module 11 — App Lifecycle

Understanding the app lifecycle is fundamental for building apps that handle backgrounding, notifications, and state restoration correctly.

---

## Topics

| # | File | Description |
|---|------|-------------|
| 01 | [App & Scene Lifecycle](01-app-and-scene-lifecycle.md) | UIApplication, AppDelegate, SceneDelegate, state transitions |
| 02 | [Background Execution](02-background-execution.md) | Background modes, BGTaskScheduler, silent push |
| 03 | [Push Notifications & Deep Links](03-push-and-deep-links.md) | APNs, UNUserNotificationCenter, universal links, URL schemes |

---

## Key Interview Themes

- **AppDelegate vs SceneDelegate** — when each was introduced, what each handles
- **App state machine** — Not Running → Inactive → Active → Background → Suspended
- **Background fetch** — how it works, limitations
- **Deep linking** — URL scheme vs universal links tradeoffs
- **Push notification handling** — foreground vs background
