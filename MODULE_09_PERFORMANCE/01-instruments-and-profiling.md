# Performance — Instruments & Profiling

---

## Instruments Overview

Instruments is Xcode's performance analysis tool. Always profile on a **physical device** (not simulator).

```
Xcode → Product → Profile (⌘I) → Choose Template
```

### Key Templates

| Template | Use For |
|----------|---------|
| **Time Profiler** | CPU bottlenecks, slow functions |
| **Allocations** | Memory usage, object lifetimes |
| **Leaks** | Memory leaks |
| **SwiftUI** | View invalidation count, body calls |
| **Core Data** | Fetch performance, save frequency |
| **Network** | Request timing, payload size |
| **Energy Log** | Battery impact |

---

## Time Profiler

Shows where CPU time is spent during a time window.

### How to Use

1. Launch with Instruments → Time Profiler
2. Perform the slow action in your app
3. Select the time range
4. Click "Heaviest Stack Trace" or browse call tree
5. Check "Hide System Libraries" to focus on your code
6. Check "Invert Call Tree" to see hottest leaf functions

### Interpreting Results

```
Self %  — time spent in this function alone
Total % — time including all callees

Look for:
- Large Self% → this function is the bottleneck
- Unexpected framework calls (JSON parsing on main thread, CoreData on main thread)
- Your code appearing high in the inverted tree
```

### Common Bottlenecks Found in Time Profiler

```swift
// ❌ Decoding on main thread
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let item = JSONDecoder().decode(Item.self, from: data[indexPath.row])  // ❌ On main thread
    ...
}

// ✅ Decode in background, display on main
Task.detached(priority: .userInitiated) {
    let items = try JSONDecoder().decode([Item].self, from: data)
    await MainActor.run { self.items = items; self.tableView.reloadData() }
}

// ❌ Expensive layout calculation in layoutSubviews (called many times)
override func layoutSubviews() {
    super.layoutSubviews()
    expensiveTextLayout()  // Called on every frame change
}

// ✅ Cache the result
override func layoutSubviews() {
    super.layoutSubviews()
    if needsTextLayoutUpdate {
        expensiveTextLayout()
        needsTextLayoutUpdate = false
    }
}
```

---

## Allocations Instrument

Shows all object allocations and their memory impact.

### Tracking Object Lifetime

1. Launch with Allocations instrument
2. Perform actions (navigate to screen, scroll, navigate back)
3. Check if object count returns to baseline after navigation
4. If objects don't deallocate → leak or unintended retention

### Identifying Memory Issues

```
Allocations → Statistics → Persistent Bytes (filter by your module name)

Signs of issues:
- Persistent count keeps growing → leak
- Large allocation spikes → loading too much at once
- Same object type allocated thousands of times → reuse issue
```

---

## Startup Time Optimization

```
Total launch time = Pre-main time + Post-main time

Pre-main (dyld):
- Static initializers
- +load methods (Obj-C)
- Dynamic library linking

Post-main (your code):
- AppDelegate.didFinishLaunching
- Root ViewController init + viewDidLoad
- First render
```

### Measuring Launch Time

```bash
# Add environment variable in scheme
DYLD_PRINT_STATISTICS = 1

# Output:
# Total pre-main time: 850.44 milliseconds (100.0%)
#          dylib loading time: 100ms
#    rebase/binding time: 50ms
#        ObjC setup time: 200ms
#       initializer time: 500ms
```

### Optimization Techniques

```swift
// ❌ Heavy work in AppDelegate.didFinishLaunching
func application(_ application: UIApplication, 
                 didFinishLaunchingWithOptions...) -> Bool {
    Database.shared.setup()         // Slow
    Analytics.shared.initialize()   // Slow
    CacheManager.shared.preload()   // Slow
    return true
}

// ✅ Defer non-critical initialization
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions...) -> Bool {
    // Only critical setup here
    setupCrashReporting()
    
    // Defer the rest
    DispatchQueue.global().async {
        Analytics.shared.initialize()
    }
    
    Task.detached(priority: .background) {
        await CacheManager.shared.preload()
    }
    return true
}

// ✅ Lazy initialization for heavy singletons
class UserService {
    // Initialized on first access, not at app launch
    static let shared = UserService()
    private init() { /* potentially slow setup */ }
}
```

---

## Interview Q&A

**Q: How do you investigate slow app startup?**  
A: Set `DYLD_PRINT_STATISTICS=1` to measure pre-main time. Profile with Time Profiler to find slow code in `application(_:didFinishLaunchingWithOptions:)`. Common fixes: lazy-initialize singletons, defer non-critical setup to background queues, reduce +load methods, minimize dynamic framework count.

**Q: Walk me through using Time Profiler to find a CPU bottleneck.**  
A: Launch with Instruments → Time Profiler, reproduce the slow action, select the time range, invert the call tree and hide system libraries to surface your code. High "Self%" indicates the function itself is slow; high "Total%" with low "Self%" means its callees are slow. Look for main-thread work that should be off-thread.

**Q: How much of startup time is pre-main vs post-main?**  
A: Pre-main includes dyld loading frameworks, ObjC runtime setup, and static initializers — often 200-600ms on a cold launch. Post-main is your `didFinishLaunching` code. Pre-main is reduced by reducing dynamic framework count and eliminating `+load` methods. Post-main by deferring slow initialization.

---

## Quick Revision

- Instruments: profile on device, not simulator
- Time Profiler: CPU bottlenecks; invert call tree + hide system libs
- Allocations: object counts; persistent count growing = leak
- Startup: pre-main (dyld) + post-main (your code); minimize both
- Defer non-critical init to background; lazy-init heavy singletons
- Never do JSON decoding, database reads, or heavy computation on main thread
