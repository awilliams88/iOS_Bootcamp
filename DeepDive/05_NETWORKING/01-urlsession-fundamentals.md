# Networking — URLSession Fundamentals

---

## URLSession Overview

`URLSession` is the foundation of all networking in iOS. It manages network requests, caching, cookies, and authentication.

```swift
// Three built-in configurations
let defaultSession   = URLSession(configuration: .default)    // Persistent cache & cookies
let ephemeralSession = URLSession(configuration: .ephemeral)  // No persistence (good for sensitive data)
let backgroundSession = URLSession(                           // Downloads/uploads in background
    configuration: .background(withIdentifier: "com.app.background")
)

// Shared singleton — fine for simple tasks
URLSession.shared
```

### URLSessionConfiguration

```swift
let config = URLSessionConfiguration.default
config.timeoutIntervalForRequest  = 30   // Per-request timeout
config.timeoutIntervalForResource = 300  // Total resource timeout
config.requestCachePolicy = .returnCacheDataElseLoad
config.httpAdditionalHeaders = ["Accept": "application/json"]
config.urlCache = URLCache(
    memoryCapacity: 20 * 1024 * 1024,   // 20 MB
    diskCapacity: 100 * 1024 * 1024,    // 100 MB
    directory: nil
)
config.waitsForConnectivity = true  // Wait for network instead of failing immediately

let session = URLSession(configuration: config)
```

> 🎯 **Interview Answer:** "URLSession has three configuration types: `default` (persistent disk cache + cookies), `ephemeral` (in-memory only — good for private browsing or sensitive operations), and `background` (continues transfers when the app is suspended or terminated)."

---

## Building URLRequests

```swift
var request = URLRequest(url: URL(string: "https://api.example.com/users")!)
request.httpMethod = "POST"
request.setValue("application/json", forHTTPHeaderField: "Content-Type")
request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
request.timeoutInterval = 30

// JSON body
let body = ["name": "Alice", "email": "alice@example.com"]
request.httpBody = try? JSONEncoder().encode(body)
```

### HTTP Methods

```swift
enum HTTPMethod: String {
    case get    = "GET"
    case post   = "POST"
    case put    = "PUT"
    case patch  = "PATCH"
    case delete = "DELETE"
}
```

---

## Data Tasks

```swift
// Completion handler (old style)
let task = URLSession.shared.dataTask(with: request) { data, response, error in
    if let error = error {
        print("Error: \(error)")
        return
    }
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        print("Bad status")
        return
    }
    if let data = data {
        let user = try? JSONDecoder().decode(User.self, from: data)
    }
}
task.resume()

// Modern async/await
let (data, response) = try await URLSession.shared.data(for: request)
guard let httpResponse = response as? HTTPURLResponse,
      (200...299).contains(httpResponse.statusCode) else {
    throw NetworkError.badStatusCode((response as? HTTPURLResponse)?.statusCode ?? -1)
}
let user = try JSONDecoder().decode(User.self, from: data)
```

---

## Download Tasks

Used for large files — saves directly to disk, supports background downloads:

```swift
// Basic download
let downloadTask = URLSession.shared.downloadTask(with: url) { localURL, response, error in
    guard let localURL = localURL, error == nil else { return }
    
    let destination = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        .appendingPathComponent("downloaded_file.pdf")
    
    try? FileManager.default.moveItem(at: localURL, to: destination)
}
downloadTask.resume()

// With progress tracking
class DownloadDelegate: NSObject, URLSessionDownloadDelegate {
    func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask,
                    didWriteData bytesWritten: Int64, totalBytesWritten: Int64,
                    totalBytesExpectedToWrite: Int64) {
        let progress = Double(totalBytesWritten) / Double(totalBytesExpectedToWrite)
        print("Progress: \(progress * 100)%")
    }
    
    func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask,
                    didFinishDownloadingTo location: URL) {
        // Move file from temporary location
    }
}
```

---

## Upload Tasks

```swift
// Upload Data
let uploadTask = URLSession.shared.uploadTask(with: request, from: jsonData) { data, response, error in
    // Handle response
}
uploadTask.resume()

// Multipart Form Upload
func createMultipartRequest(imageData: Data, fileName: String) -> URLRequest {
    let boundary = UUID().uuidString
    var request = URLRequest(url: URL(string: "https://api.example.com/upload")!)
    request.httpMethod = "POST"
    request.setValue("multipart/form-data; boundary=\(boundary)", 
                     forHTTPHeaderField: "Content-Type")
    
    var body = Data()
    let boundaryPrefix = "--\(boundary)\r\n"
    
    // Add image field
    body.append(boundaryPrefix.data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"image\"; filename=\"\(fileName)\"\r\n".data(using: .utf8)!)
    body.append("Content-Type: image/jpeg\r\n\r\n".data(using: .utf8)!)
    body.append(imageData)
    body.append("\r\n".data(using: .utf8)!)
    
    // Close boundary
    body.append("--\(boundary)--\r\n".data(using: .utf8)!)
    
    request.httpBody = body
    return request
}
```

---

## Background Sessions

For downloads/uploads that must complete even if the app is suspended:

```swift
// AppDelegate
func application(_ application: UIApplication,
                 handleEventsForBackgroundURLSession identifier: String,
                 completionHandler: @escaping () -> Void) {
    // Store completion handler — call it after session finishes
    BackgroundSessionManager.shared.completionHandler = completionHandler
}

// Session delegate
func urlSessionDidFinishEvents(forBackgroundURLSession session: URLSession) {
    DispatchQueue.main.async {
        BackgroundSessionManager.shared.completionHandler?()
        BackgroundSessionManager.shared.completionHandler = nil
    }
}
```

> ⚠️ **Pitfall:** Background URLSession must be identified with a string. The same identifier must be used to reconnect to the session when the app is relaunched in the background.

---

## Response Handling

```swift
enum NetworkError: Error, LocalizedError {
    case invalidURL
    case badStatusCode(Int)
    case decodingFailed(Error)
    case noData
    case unauthorized
    case serverError(Int)
    
    var errorDescription: String? {
        switch self {
        case .invalidURL:         return "Invalid URL"
        case .badStatusCode(let code): return "HTTP \(code)"
        case .decodingFailed(let e):   return "Decoding: \(e.localizedDescription)"
        case .noData:             return "No data received"
        case .unauthorized:       return "Authentication required"
        case .serverError(let code):   return "Server error \(code)"
        }
    }
}

func validateResponse(_ response: URLResponse) throws -> HTTPURLResponse {
    guard let httpResponse = response as? HTTPURLResponse else {
        throw NetworkError.noData
    }
    switch httpResponse.statusCode {
    case 200...299: return httpResponse
    case 401:       throw NetworkError.unauthorized
    case 500...599: throw NetworkError.serverError(httpResponse.statusCode)
    default:        throw NetworkError.badStatusCode(httpResponse.statusCode)
    }
}
```

---

## Interview Q&A

**Q: What's the difference between URLSession.shared and a custom session?**  
A: `URLSession.shared` is a singleton with default configuration — fine for simple requests. A custom session lets you configure timeout, cache policy, additional headers, authentication delegates, and use background transfers. You also need a custom session to set yourself as a delegate.

**Q: When would you use a background URLSession?**  
A: When transfers must complete even if the app is suspended or terminated — large file downloads, photo uploads. The system keeps the transfer alive and relaunch the app when done, calling `handleEventsForBackgroundURLSession`.

**Q: What is `waitsForConnectivity` and when should you enable it?**  
A: When `true`, URLSession waits for a network connection rather than failing immediately with a "not connected" error. Useful for offline resilience — the request queues and fires when connectivity is restored. Pair with a timeout to avoid waiting indefinitely.

**Q: How does multipart/form-data work?**  
A: It's a MIME encoding that allows sending multiple parts in one request body, each separated by a boundary string. Each part has headers (Content-Disposition, Content-Type) and a body. Used for file uploads alongside metadata.

---

## Quick Revision

- `URLSession.shared` — singleton, default config
- `.default` — disk cache + cookies; `.ephemeral` — memory only; `.background(id:)` — background transfers
- `URLRequest` — URL + method + headers + body
- `dataTask` → `data(for:)` in async/await
- Download tasks save to temp file — move to permanent location in delegate
- Background session requires `handleEventsForBackgroundURLSession` in AppDelegate
- Always validate `HTTPURLResponse.statusCode`
