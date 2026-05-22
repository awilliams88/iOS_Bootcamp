# System Design — Upload & Analytics Architecture

---

## Photo/Video Upload Architecture

### Requirements
- Upload photos/videos in background
- Resume interrupted uploads (large files)
- Show progress to user
- Retry on failure

### Architecture

```swift
// Upload manager
actor UploadManager {
    private var activeUploads: [UUID: UploadTask] = [:]
    private let session: URLSession
    
    func upload(
        media: MediaAsset,
        progressHandler: @escaping (Double) -> Void
    ) async throws -> UploadedMedia {
        
        // 1. Compress/resize if needed (background task)
        let prepared = try await prepareMedia(media)
        
        // 2. Request upload URL from server (pre-signed S3 URL)
        let uploadURL = try await apiClient.requestUploadURL(
            contentType: prepared.mimeType,
            size: prepared.data.count
        )
        
        // 3. Upload to CDN with progress
        let task = UploadTask(
            id: UUID(),
            data: prepared.data,
            url: uploadURL.uploadURL,
            progressHandler: progressHandler
        )
        activeUploads[task.id] = task
        defer { activeUploads.removeValue(forKey: task.id) }
        
        try await task.upload(session: session)
        
        // 4. Notify server upload is complete
        return try await apiClient.confirmUpload(key: uploadURL.key)
    }
    
    private func prepareMedia(_ asset: MediaAsset) async throws -> PreparedMedia {
        return try await Task.detached(priority: .userInitiated) {
            // Compress on background thread
            switch asset {
            case .photo(let image):
                let compressed = image.jpegData(compressionQuality: 0.8)!
                return PreparedMedia(data: compressed, mimeType: "image/jpeg")
            case .video(let url):
                return try await VideoCompressor.compress(url: url)
            }
        }.value
    }
}

// Resumable upload (multipart)
final class ResumableUpload {
    private let chunkSize = 5 * 1024 * 1024  // 5MB chunks (S3 minimum)
    
    func upload(data: Data, to url: URL) async throws {
        let chunks = stride(from: 0, to: data.count, by: chunkSize).map {
            data[$0..<min($0 + chunkSize, data.count)]
        }
        
        // Initiate multipart upload
        let uploadID = try await initiateMultipartUpload(url: url)
        var uploadedParts: [(Int, String)] = []  // (partNumber, eTag)
        
        // Upload each chunk with retry
        for (index, chunk) in chunks.enumerated() {
            let eTag = try await uploadPart(
                chunk: chunk,
                partNumber: index + 1,
                uploadID: uploadID,
                url: url
            )
            uploadedParts.append((index + 1, eTag))
        }
        
        // Complete multipart upload
        try await completeMultipartUpload(uploadID: uploadID, parts: uploadedParts, url: url)
    }
}
```

---

## Analytics Architecture

### Requirements
- Track user events (tap, view, purchase)
- Batch events to reduce network calls
- Persist events so none are lost during app crash
- Support multiple analytics providers (Firebase, Mixpanel, custom)

### Event Model

```swift
struct AnalyticsEvent: Codable {
    let id: UUID
    let name: String
    let properties: [String: AnyCodable]
    let timestamp: Date
    let sessionID: String
    let userID: String?
    let deviceInfo: DeviceInfo
}

// DeviceInfo captured once
struct DeviceInfo: Codable {
    let platform: String   // "iOS"
    let osVersion: String  // "17.0"
    let appVersion: String
    let deviceModel: String
}
```

### Batching Pipeline

```swift
actor AnalyticsPipeline {
    private var buffer: [AnalyticsEvent] = []
    private var flushTimer: Task<Void, Never>?
    private let batchSize = 50
    private let flushInterval: TimeInterval = 30
    
    private let store: AnalyticsStore  // Persistent (Core Data/SQLite)
    private let sender: AnalyticsSender
    
    func track(_ event: AnalyticsEvent) async {
        buffer.append(event)
        await store.save(event)  // Persist first
        
        if buffer.count >= batchSize {
            await flush()
        } else {
            scheduleFlush()
        }
    }
    
    private func flush() async {
        guard !buffer.isEmpty else { return }
        let batch = buffer
        buffer.removeAll()
        
        do {
            try await sender.send(events: batch)
            await store.delete(events: batch)  // Remove after successful send
        } catch {
            // Keep in store — will retry on next app launch
            buffer.insert(contentsOf: batch, at: 0)
        }
    }
    
    private func scheduleFlush() {
        flushTimer?.cancel()
        flushTimer = Task {
            try? await Task.sleep(for: .seconds(flushInterval))
            await flush()
        }
    }
    
    // Called on app background
    func flushOnBackground() async {
        await flush()
    }
}

// Multiple providers via protocol
protocol AnalyticsProvider {
    func send(events: [AnalyticsEvent]) async throws
}

final class MultiProviderSender: AnalyticsSender {
    private let providers: [AnalyticsProvider]
    
    func send(events: [AnalyticsEvent]) async throws {
        // Send to all providers concurrently
        try await withThrowingTaskGroup(of: Void.self) { group in
            for provider in providers {
                group.addTask { try await provider.send(events: events) }
            }
            try await group.waitForAll()
        }
    }
}
```

---

## Analytics Interview Discussion

### Privacy Considerations

```swift
// PII-safe analytics
func trackSearch(query: String, resultCount: Int) {
    analyticsService.track(AnalyticsEvent(
        name: "search_performed",
        properties: [
            // ❌ Don't include raw query (PII risk)
            // "query": query,
            
            // ✅ Track metadata, not content
            "query_length": query.count,
            "query_has_special_chars": query.unicodeScalars.contains { !$0.isASCII },
            "result_count": resultCount
        ]
    ))
}
```

---

## Interview Q&A

**Q: How do you design photo upload for a social app?**  
A: Pre-signed URL flow: client requests an upload URL from the server (which returns a pre-signed S3 URL), uploads directly to S3 (bypassing your server), then notifies your server with the S3 key. For large files, use multipart upload with 5MB+ chunks. Show progress via URLSession download delegate. Retry failed chunks independently.

**Q: How do you ensure analytics events aren't lost if the app crashes?**  
A: Persist events to a local store (Core Data, SQLite) before trying to send. After successful batch send, delete the stored events. On next launch, check for unsent events and include them in the next batch. This guarantees at-least-once delivery.

**Q: What's the tradeoff between event volume and batching interval?**  
A: More frequent sends: lower latency for real-time dashboards, but more battery/network usage. Less frequent (larger batches): fewer network calls, better battery life, but analytics lag. Common production balance: flush on batch size (50-100 events) OR time interval (30-60 seconds), whichever comes first. Always flush when app goes to background.

---

## Quick Revision

- Upload: pre-signed URL → direct to CDN → notify server with key
- Large files: multipart upload with 5MB+ chunks, retry failed chunks
- Analytics: persist first, send second → guaranteed delivery
- Batching: flush on size threshold OR time interval
- Multiple providers: send to all concurrently via TaskGroup
- PII: never log user-identifiable content; log metadata/counts instead
- Background flush: always flush analytics when app enters background
