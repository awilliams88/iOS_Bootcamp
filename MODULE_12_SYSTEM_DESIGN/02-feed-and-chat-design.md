# System Design — Feed & Chat App Design

---

## Social Feed Design

### Requirements (clarify first in interview)
- Show posts from followed users, most-recent first
- Infinite scroll with 20 items per page
- Like, comment, share actions
- Offline support: show cached feed
- Real-time like count updates

### Component Architecture

```
┌─────────────────────────────────────────────┐
│              FeedCoordinator                │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │           FeedViewModel              │   │
│  │  @Published items: [FeedItem]        │   │
│  │  @Published isLoadingMore: Bool      │   │
│  └──────────────┬──────────────────────┘   │
│                 │                           │
│  ┌──────────────▼──────────────────────┐   │
│  │           FeedRepository             │   │
│  │  memory cache + CoreData cache       │   │
│  │  remote: FeedAPIDataSource           │   │
│  │  local: FeedLocalDataSource          │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### Feed Data Flow

```swift
// FeedItem — keep flat for efficient diffing
struct FeedItem: Identifiable, Hashable {
    let id: String
    let authorID: String
    let authorName: String
    let authorAvatarURL: URL?
    let content: String
    let mediaURLs: [URL]
    var likeCount: Int
    var isLikedByCurrentUser: Bool
    let createdAt: Date
    
    // Precomputed for display
    let formattedTimestamp: String
    let formattedLikeCount: String
}

// Diffable data source key
extension FeedItem: Hashable {
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
        hasher.combine(likeCount)  // Include mutable state in hash for diffing
        hasher.combine(isLikedByCurrentUser)
    }
}
```

### Optimistic Like

```swift
func toggleLike(item: FeedItem) async {
    // 1. Optimistic update
    if let index = items.firstIndex(where: { $0.id == item.id }) {
        let wasLiked = items[index].isLikedByCurrentUser
        items[index].isLikedByCurrentUser = !wasLiked
        items[index].likeCount += wasLiked ? -1 : 1
    }
    
    // 2. Sync to server
    do {
        if item.isLikedByCurrentUser {
            try await feedService.unlike(itemID: item.id)
        } else {
            try await feedService.like(itemID: item.id)
        }
    } catch {
        // 3. Rollback on failure
        if let index = items.firstIndex(where: { $0.id == item.id }) {
            items[index].isLikedByCurrentUser = item.isLikedByCurrentUser
            items[index].likeCount = item.likeCount
        }
    }
}
```

---

## Chat App Design

### Requirements
- One-on-one and group messaging
- Real-time delivery
- Message states: sent, delivered, read
- Offline: queue messages, send when online
- Media: images, videos
- Push notifications for new messages

### Architecture

```
WebSocket Manager (real-time)
    ↓
Message Queue (actor-based)
    ↓
Local Store (Core Data / SwiftData)
    ↓
Message Repository
    ↓
Chat ViewModel
    ↓
Chat View (SwiftUI List)
```

### Message Model

```swift
struct ChatMessage: Identifiable {
    enum Status { case sending, sent, delivered, read, failed }
    enum Content {
        case text(String)
        case image(URL, thumbnail: UIImage?)
        case video(URL, duration: TimeInterval)
    }
    
    let id: UUID
    let conversationID: String
    let senderID: String
    let content: Content
    var status: Status
    let sentAt: Date
    var deliveredAt: Date?
    var readAt: Date?
    
    // For optimistic sending
    var isLocal: Bool  // True if not yet confirmed by server
}
```

### Real-time WebSocket Architecture

```swift
actor WebSocketManager {
    private var webSocket: URLSessionWebSocketTask?
    private var continuations: [AsyncStream<WebSocketEvent>.Continuation] = []
    
    func connect(to url: URL) {
        let session = URLSession(configuration: .default)
        webSocket = session.webSocketTask(with: url)
        webSocket?.resume()
        startReceiving()
    }
    
    private func startReceiving() {
        Task {
            guard let ws = webSocket else { return }
            do {
                while true {
                    let message = try await ws.receive()
                    let event = parseEvent(from: message)
                    continuations.forEach { $0.yield(event) }
                }
            } catch {
                // Reconnect logic
                await reconnect()
            }
        }
    }
    
    func events() -> AsyncStream<WebSocketEvent> {
        AsyncStream { continuation in
            continuations.append(continuation)
            continuation.onTermination = { [weak self] _ in
                Task { await self?.removeContinuation(continuation) }
            }
        }
    }
    
    func send(event: WebSocketEvent) async throws {
        let data = try JSONEncoder().encode(event)
        try await webSocket?.send(.data(data))
    }
    
    private func reconnect() async {
        try? await Task.sleep(for: .seconds(2))
        // Reconnect with exponential backoff
    }
}
```

### Message Delivery (Optimistic + Queue)

```swift
@MainActor
final class ChatViewModel: ObservableObject {
    @Published private(set) var messages: [ChatMessage] = []
    private let wsManager: WebSocketManager
    private let localStore: MessageStore
    private var pendingQueue: [ChatMessage] = []
    
    func send(text: String) async {
        let message = ChatMessage(
            id: UUID(),
            content: .text(text),
            status: .sending,
            isLocal: true
        )
        
        // Immediately show in UI
        messages.append(message)
        
        do {
            let confirmed = try await wsManager.send(
                event: .newMessage(conversationID: conversationID, content: text)
            )
            // Update with server-confirmed ID and status
            if let idx = messages.firstIndex(where: { $0.id == message.id }) {
                messages[idx] = confirmed
            }
        } catch {
            // Mark as failed
            if let idx = messages.firstIndex(where: { $0.id == message.id }) {
                messages[idx].status = .failed
            }
            pendingQueue.append(message)
        }
    }
    
    func listenForMessages() async {
        for await event in await wsManager.events() {
            switch event {
            case .newMessage(let message):
                messages.append(message)
            case .messageDelivered(let id):
                updateStatus(id: id, to: .delivered)
            case .messageRead(let id):
                updateStatus(id: id, to: .read)
            }
        }
    }
}
```

---

## Interview Q&A

**Q: How would you design a social feed for an iOS app?**  
A: Repository pattern with multi-tier caching (memory → Core Data → network). Cursor-based pagination to avoid gaps on new posts. Diffable data source for efficient updates. Optimistic like/unlike. Real-time count updates via WebSocket or silent push. Pre-computed display strings (timestamps, counts) to keep cell rendering fast.

**Q: How do you handle message ordering in a chat app?**  
A: Use server-generated timestamps and sequence numbers. Store messages sorted by timestamp locally. For optimistic messages (sent but not server-confirmed), append to the end with a "sending" status. On server confirmation, replace the optimistic message with the server message using its authoritative ID and timestamp.

**Q: How do you handle WebSocket reconnection?**  
A: Implement exponential backoff with jitter for reconnection attempts. Keep a pending message queue during disconnection. On reconnect, first fetch missed messages from REST API (for the gap period), then switch back to WebSocket for real-time. Show connection status in UI.

---

## Quick Revision

- Feed: flat FeedItem, optimistic likes, cursor pagination, diffable data source
- Chat: WebSocket for real-time, optimistic send, status progression (sending→sent→delivered→read)
- WebSocket: actor-based manager, AsyncStream for events, reconnect with backoff
- Message queue: store failed messages, retry on network restore
- Real-time + REST: WebSocket for live updates, REST for historical/on-load
