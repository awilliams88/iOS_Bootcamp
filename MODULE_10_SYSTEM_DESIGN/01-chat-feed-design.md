# Chat App and Feed System Design

## Chat App
Architecture:

```text
View -> ViewModel -> Repository -> WebSocket/API -> Local Cache
```

Concerns:
- message ordering
- retries
- offline queue
- optimistic UI
- pagination history
- typing indicators

## Social Feed
Concerns:
- pagination
- image caching
- prefetching
- diffable rendering
- background refresh

## Interview Answer
Design around responsiveness, offline resilience, and efficient rendering.