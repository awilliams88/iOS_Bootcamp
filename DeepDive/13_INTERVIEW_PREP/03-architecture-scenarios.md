# Architecture Scenarios

Practice articulating architectural decisions with confidence. For each scenario, structure your answer: problem → options → decision → tradeoffs.

---

## Scenario 1: Migrating from MVC to MVVM

**Interviewer:** "Our codebase has 2000-line ViewControllers. How would you migrate to MVVM without breaking the app?"

**Your Answer:**

"I'd approach this incrementally — never a big bang rewrite.

**Step 1: Identify extraction targets**
- Start with the most-tested, most-used screens
- Pick screens with the most business logic (network calls, data transformation)
- Avoid screens with complex UIKit-specific behavior (custom animations, gesture recognizers)

**Step 2: Extract ViewModel alongside existing ViewController**
- Create a `UserListViewModel` next to the existing `UserListViewController`
- Move business logic (service calls, data processing) into the ViewModel
- ViewController subscribes to ViewModel's published state via Combine
- No behavior change to the user — same screen, cleaner code

**Step 3: Establish protocol boundaries**
- Define `UserServiceProtocol` so ViewModel can be unit tested
- Add unit tests before refactoring (tests as safety net)

**Step 4: Gradual migration**
- Set a team rule: any file you touch gets migrated
- Track percentage in sprint retrospectives

**Tradeoffs:** Takes longer but much safer. Avoids the 'we rewrote it and it's broken' problem."

---

## Scenario 2: Designing Offline Support

**Interviewer:** "Users complain the app is unusable without network. How would you add offline support?"

**Your Answer:**

"Offline support needs to be designed, not bolted on. Here's my approach:

**Define the offline contract:**
- Which features must work offline? (read-only feed? posting new content? searching?)
- What's the staleness tolerance? (show 1-hour-old data? 1-day-old?)

**Storage layer:**
- Core Data or SwiftData as the local source of truth
- Repository pattern: ViewModel talks to Repository, not directly to network
- Repository decides: return cache immediately, then refresh in background

**Write operations:**
- Optimistic updates: apply locally immediately, sync to server
- Pending operations queue (actor-based) for mutations made while offline
- Process queue when connectivity returns (monitor via `NWPathMonitor`)

**Sync strategy:**
```swift
// Listen for connectivity
let monitor = NWPathMonitor()
monitor.pathUpdateHandler = { path in
    if path.status == .satisfied {
        await syncQueue.processAll()
    }
}
```

**Conflict resolution:**
- Simple: last-write-wins with server timestamp
- Complex: operational transforms or CRDTs for collaborative data
- Show UI for unresolvable conflicts (rare)

**Tradeoffs:** Adds significant complexity to the data layer. Team must be disciplined about always writing through the repository, not directly to the network."

---

## Scenario 3: Real-time Chat Architecture

**Interviewer:** "Design the iOS side of a real-time chat feature."

**Your Answer:**

"I'll design bottom-up: data → networking → storage → presentation.

**Data Model:**
```
Message: id, conversationID, senderID, content, status (sending/sent/delivered/read), timestamp
Conversation: id, participants, lastMessage, unreadCount
```

**Networking — Dual approach:**
- WebSocket for real-time: new messages, typing indicators, delivery status
- REST API for: history loading, image uploads, pagination, initial load
- Reconnection logic with exponential backoff in actor

**Storage — Core Data:**
- Messages table: indexed on conversationID + timestamp
- Fetch batches (NSFetchedResultsController) so old messages don't load into memory
- FetchLimit + batch-size for efficient memory use

**Message lifecycle:**
1. User sends → optimistic append locally with `status: .sending`
2. WebSocket send → server confirms → update to `status: .sent` with server ID
3. Remote push/WebSocket event → `status: .delivered`, then `.read`
4. Failure → `status: .failed`, offer retry

**Image upload:**
- Pre-signed S3 URL pattern
- Upload in background URLSession task
- Show progress in message bubble
- Send message only after upload confirmed

**Pagination:**
- Cursor-based, load 50 messages, scroll-up triggers previous page
- Use NSFetchedResultsController with batchSize for efficient memory

**Tradeoffs:** WebSocket + REST dual-track adds complexity but provides resilience (REST for history, WS for live). Full offline support needs local queue for sent messages."

---

## Scenario 4: Performance Investigation

**Interviewer:** "Users report the app feels slow. How do you investigate and fix it?"

**Your Answer:**

"I'd approach this systematically, not by guessing.

**Step 1: Reproduce and categorize**
- Slow startup? Slow screen transition? Janky scrolling? Slow network?
- User reports often conflate these — reproduce it first

**Step 2: Instrument, don't guess**
- Xcode Instruments on a physical device (simulator is inaccurate for performance)
- Time Profiler for CPU: where is the main thread spending time?
- Core Animation for scrolling: enable 'Color Off-screen Rendered', watch FPS
- Allocations for memory: growing persistent count = leak

**Step 3: Common findings and fixes**

*Slow startup:*
- Heavy work in `didFinishLaunchingWithOptions` → defer to background
- Too many dynamic frameworks → evaluate if all are needed

*Janky scrolling:*
- Image loading on main thread → async with cancellation in `prepareForReuse`
- Off-screen rendering (cornerRadius + clips) → set shadowPath, rasterize static views
- Complex Auto Layout in cells → use `UIView.layoutIfNeeded()` with pre-calculated frames

*Slow networking:*
- Large response payloads → pagination, field filtering
- No caching → URLCache, or app-level cache
- Not using background thread for decoding → `Task.detached(priority: .userInitiated)`

**Step 4: Measure after fixing**
- Benchmark before and after, track in CI if possible

**Tradeoffs:** Premature optimization is dangerous. Always measure first — most performance issues are in unexpected places."

---

## Scenario 5: App Security Review

**Interviewer:** "A security audit is pending. What areas do you review for iOS app security?"

**Your Answer:**

"I'd audit across four areas:

**1. Data at rest:**
- Auth tokens and passwords → Keychain only (`WhenUnlockedThisDeviceOnly`)
- Sensitive user data → encrypted files (`.completeFileProtection`)
- No sensitive data in UserDefaults or plaintext files
- Database encryption (Core Data stores are not encrypted by default)

**2. Data in transit:**
- ATS enabled, no `NSAllowsArbitraryLoads`
- Certificate pinning for sensitive endpoints (payment, auth)
- Token rotation policy
- Sensitive operations: additional payload encryption beyond TLS

**3. Runtime:**
- No sensitive data in logs (especially tokens, PII)
- Screen content hidden on `willResignActive` (app switcher)
- Biometric authentication for sensitive features
- Jailbreak detection (layered with server-side validation)

**4. Code:**
- Hardcoded secrets audit → none should exist (use environment config or remote config)
- Input validation (especially URL parsing for deep links)
- Third-party SDK security review (what data does each SDK collect?)

**Tradeoffs:** Perfect security adds friction. Balance based on data sensitivity — a to-do list app doesn't need cert pinning. A banking app does."

---

## Quick Tips for Architecture Questions

1. **Ask clarifying questions first** — scale, team size, timeline
2. **State your assumptions** — "assuming this is a solo developer app..."
3. **Draw the architecture** — even on a whiteboard mentally, talk through layers
4. **Mention tradeoffs** — every choice has pros/cons; naming them shows depth
5. **Avoid over-engineering** — show you know when NOT to add complexity
6. **Reference past experience** — "In a previous project, we faced a similar problem and..."
