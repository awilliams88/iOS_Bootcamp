# Storage — File System

---

## App Sandbox Directories

iOS apps live in a sandboxed container with specific directories for different data:

```swift
let fm = FileManager.default

// Documents — user-created content, backed up by iCloud/iTunes
let documents = fm.urls(for: .documentDirectory, in: .userDomainMask)[0]

// Library/Application Support — app data, backed up, NOT shown to user
let appSupport = fm.urls(for: .applicationSupportDirectory, in: .userDomainMask)[0]

// Library/Caches — cache data, NOT backed up, system may purge when disk is low
let caches = fm.urls(for: .cachesDirectory, in: .userDomainMask)[0]

// tmp — temporary files, NOT backed up, purged after each session
let temp = URL(fileURLWithPath: NSTemporaryDirectory())
```

> 🎯 **Interview Answer:** "iOS has four key app directories. Documents is for user files backed up to iCloud. Library/Application Support is for app data that should be backed up but not shown in Files app. Library/Caches is for regenerable cached data that the system can delete when storage is low. tmp is for ephemeral scratch space."

---

## Reading & Writing Files

```swift
// Write Data
let url = documents.appendingPathComponent("profile.json")
let data = try JSONEncoder().encode(profile)
try data.write(to: url, options: .atomic)  // .atomic = write to temp then rename (safe)

// Read Data
let loaded = try Data(contentsOf: url)
let profile = try JSONDecoder().decode(Profile.self, from: loaded)

// Write String
let logURL = caches.appendingPathComponent("app.log")
try logText.write(to: logURL, atomically: true, encoding: .utf8)

// Read String
let log = try String(contentsOf: logURL, encoding: .utf8)
```

---

## FileManager Operations

```swift
// Create directory
try fm.createDirectory(at: appSupport.appendingPathComponent("UserData"),
                       withIntermediateDirectories: true)

// Check existence
fm.fileExists(atPath: url.path)

// Copy
try fm.copyItem(at: sourceURL, to: destinationURL)

// Move
try fm.moveItem(at: sourceURL, to: destinationURL)

// Delete
try fm.removeItem(at: url)

// List directory contents
let contents = try fm.contentsOfDirectory(at: documents,
                                          includingPropertiesForKeys: [.fileSizeKey, .creationDateKey])

// Get file size
let resourceValues = try url.resourceValues(forKeys: [.fileSizeKey])
let size = resourceValues.fileSize
```

---

## File Protection

```swift
// Set file protection level (equivalent to Keychain accessibility)
try data.write(to: url, options: .completeFileProtection)
// .completeFileProtection      — encrypted until device unlocked
// .completeFileProtectionUnlessOpen — encrypted when file closed + device locked
// .completeFileProtectionUntilFirstUserAuthentication — encrypted until first unlock after reboot
// .noFileProtection            — no encryption

// Set on existing file
try fm.setAttributes(
    [.protectionKey: FileProtectionType.complete],
    ofItemAtPath: url.path
)
```

---

## iCloud Document Storage

```swift
// Check iCloud availability
if fm.ubiquityIdentityToken != nil {
    // iCloud available
    let cloudURL = fm.url(forUbiquityContainerIdentifier: nil)?
        .appendingPathComponent("Documents")
        .appendingPathComponent("myfile.json")
}

// Use UIDocument for conflict resolution and state tracking
class ProfileDocument: UIDocument {
    var profile: Profile?
    
    override func contents(forType typeName: String) throws -> Any {
        return try JSONEncoder().encode(profile)
    }
    
    override func load(fromContents contents: Any, ofType typeName: String?) throws {
        guard let data = contents as? Data else { return }
        profile = try JSONDecoder().decode(Profile.self, from: data)
    }
}
```

---

## Image Caching to Disk

```swift
final class ImageDiskCache {
    private let cacheDir: URL
    
    init() {
        cacheDir = FileManager.default
            .urls(for: .cachesDirectory, in: .userDomainMask)[0]
            .appendingPathComponent("ImageCache")
        try? FileManager.default.createDirectory(at: cacheDir, withIntermediateDirectories: true)
    }
    
    func store(_ data: Data, forKey key: String) {
        let url = cacheDir.appendingPathComponent(key.hashValue.description)
        try? data.write(to: url, options: .atomic)
    }
    
    func retrieve(forKey key: String) -> Data? {
        let url = cacheDir.appendingPathComponent(key.hashValue.description)
        return try? Data(contentsOf: url)
    }
    
    func clearAll() {
        try? FileManager.default.removeItem(at: cacheDir)
        try? FileManager.default.createDirectory(at: cacheDir, withIntermediateDirectories: true)
    }
}
```

---

## Interview Q&A

**Q: Where should you store user-created documents vs app cache data?**  
A: User-created documents go in `Documents` — they're backed up and shown in Files app if you set `UIFileSharingEnabled`. App cache (downloaded images, API response cache) goes in `Library/Caches` — not backed up, and the system purges it when storage is low, so your app must handle cache misses gracefully.

**Q: What is `.atomic` write option and why use it?**  
A: Atomic writes first write to a temporary file then rename it to the destination. This ensures the file is never in a partially-written state — if the app crashes mid-write, the original file is intact. Always use `.atomic` when writing important data.

**Q: What directories are included in iCloud backup?**  
A: `Documents` and `Library` (except `Caches`) are backed up. `tmp` and `Library/Caches` are NOT backed up. You can also exclude specific files with `URLResourceValues.isExcludedFromBackup = true`.

---

## Quick Revision

- `Documents`: user files, backed up, shown in Files app if enabled
- `Library/Application Support`: app data, backed up, not user-visible
- `Library/Caches`: regenerable data, NOT backed up, system may purge
- `tmp`: ephemeral scratch space
- `.atomic` write: temp-then-rename, prevents partial writes
- File protection: `.completeFileProtection` encrypts until unlock
- `UIDocument` for iCloud with automatic conflict resolution
