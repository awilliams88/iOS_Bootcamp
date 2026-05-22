# App Lifecycle — Background Execution

---

## Background Execution Modes

Declared in `Info.plist` under `UIBackgroundModes`:

| Mode | Key | Use Case |
|------|-----|----------|
| Audio | `audio` | Music, VoIP, navigation |
| Location | `location` | Always-on location tracking |
| VoIP | `voip` | VoIP apps (PushKit) |
| Background Fetch | `fetch` | Periodic content refresh |
| Remote Notifications | `remote-notification` | Silent push |
| Background Processing | `processing` | Long-running background tasks |
| Bluetooth | `bluetooth-central`/`peripheral` | BT accessories |

---

## Background Task (Short)

When the app enters background, it gets ~30 seconds to finish work:

```swift
class DataSyncService {
    private var backgroundTaskID: UIBackgroundTaskIdentifier = .invalid
    
    func syncOnBackground() {
        // Request background execution time
        backgroundTaskID = UIApplication.shared.beginBackgroundTask(withName: "DataSync") {
            [weak self] in
            // Expiration handler — called with ~5 seconds left
            self?.cancelSync()
            UIApplication.shared.endBackgroundTask(self?.backgroundTaskID ?? .invalid)
            self?.backgroundTaskID = .invalid
        }
        
        Task {
            defer {
                UIApplication.shared.endBackgroundTask(backgroundTaskID)
                backgroundTaskID = .invalid
            }
            
            do {
                try await performSync()
            } catch {
                print("Sync failed: \(error)")
            }
        }
    }
}
```

> ⚠️ **Pitfall:** Always call `endBackgroundTask` — even in the expiration handler. Failing to do so causes the system to terminate the app.

---

## BGTaskScheduler (iOS 13+)

For periodic background work managed by the system:

```swift
import BackgroundTasks

// Register in AppDelegate.didFinishLaunching
BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "com.example.app.refresh",
    using: nil
) { task in
    handleAppRefresh(task: task as! BGAppRefreshTask)
}

BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "com.example.app.cleanup",
    using: nil
) { task in
    handleCleanup(task: task as! BGProcessingTask)
}

// Schedule refresh
func scheduleAppRefresh() {
    let request = BGAppRefreshTaskRequest(identifier: "com.example.app.refresh")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)  // 15 minutes
    
    do {
        try BGTaskScheduler.shared.submit(request)
    } catch {
        print("Could not schedule refresh: \(error)")
    }
}

// Schedule processing
func scheduleCleanup() {
    let request = BGProcessingTaskRequest(identifier: "com.example.app.cleanup")
    request.requiresNetworkConnectivity = false
    request.requiresExternalPower = true  // Only when charging
    request.earliestBeginDate = Date(timeIntervalSinceNow: 60 * 60)  // 1 hour
    
    try? BGTaskScheduler.shared.submit(request)
}

// Handle refresh task
func handleAppRefresh(task: BGAppRefreshTask) {
    scheduleAppRefresh()  // Schedule next refresh
    
    let refreshOperation = Task {
        do {
            try await DataService.shared.refresh()
            task.setTaskCompleted(success: true)
        } catch {
            task.setTaskCompleted(success: false)
        }
    }
    
    task.expirationHandler = {
        refreshOperation.cancel()
    }
}
```

---

## Silent Push Notifications

Silent push triggers background execution when received:

```swift
// Info.plist: UIBackgroundModes → remote-notification

// AppDelegate
func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable: Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    // Called for silent push (content-available: 1)
    Task {
        do {
            try await DataService.shared.refresh()
            completionHandler(.newData)
        } catch {
            completionHandler(.failed)
        }
    }
}

// Push payload for silent push:
// {
//     "aps": {
//         "content-available": 1
//     },
//     "type": "data_refresh",
//     "resource": "messages"
// }
```

---

## Interview Q&A

**Q: How long does an app have to run in the background after the user presses Home?**  
A: Apps get approximately 30 seconds of background execution time by default. Use `beginBackgroundTask(withName:expirationHandler:)` to request this window. The expiration handler fires with ~5 seconds remaining. For longer tasks, use `BGTaskScheduler`.

**Q: What's the difference between BGAppRefreshTask and BGProcessingTask?**  
A: `BGAppRefreshTask` is for short, time-sensitive work like fetching new content — the system runs it opportunistically when conditions allow (device unlocked, network available). `BGProcessingTask` is for longer work (database cleanup, ML training) — can specify `requiresExternalPower` and `requiresNetworkConnectivity`.

**Q: How does silent push work?**  
A: A push notification with `"content-available": 1` in the APS payload (no alert/sound/badge) wakes the app into background state and triggers `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)`. The app has about 30 seconds to perform work and must call the completion handler.

---

## Quick Revision

- `beginBackgroundTask`: ~30 second execution window after backgrounding
- Always call `endBackgroundTask` — including in expiration handler
- `BGAppRefreshTask`: periodic short refresh (15-min minimum interval)
- `BGProcessingTask`: longer background processing, can require charging
- Silent push: `content-available: 1`, triggers `didReceiveRemoteNotification`
- Register BGTask identifiers in Info.plist `BGTaskSchedulerPermittedIdentifiers`
