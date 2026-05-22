# Storage — UserDefaults & Keychain

---

## UserDefaults

`UserDefaults` is a key-value store backed by a plist file. Ideal for small, non-sensitive preferences.

```swift
// Reading and writing
UserDefaults.standard.set(true, forKey: "onboardingComplete")
UserDefaults.standard.set("Alice", forKey: "username")
UserDefaults.standard.set(42, forKey: "launchCount")

let isComplete = UserDefaults.standard.bool(forKey: "onboardingComplete")
let username   = UserDefaults.standard.string(forKey: "username") // Optional<String>
let count      = UserDefaults.standard.integer(forKey: "launchCount")

// Remove
UserDefaults.standard.removeObject(forKey: "username")

// Supported types
// Bool, Int, Float, Double, String, Data, Date, Array, Dictionary, URL
```

### Custom Types via Codable

```swift
struct AppSettings: Codable {
    var theme: Theme
    var notificationsEnabled: Bool
    var fontSize: Double
}

extension UserDefaults {
    func setEncodable<T: Encodable>(_ value: T, forKey key: String) {
        guard let data = try? JSONEncoder().encode(value) else { return }
        set(data, forKey: key)
    }
    
    func decodable<T: Decodable>(_ type: T.Type, forKey key: String) -> T? {
        guard let data = data(forKey: key) else { return nil }
        return try? JSONDecoder().decode(type, from: data)
    }
}

// Usage
UserDefaults.standard.setEncodable(settings, forKey: "appSettings")
let settings = UserDefaults.standard.decodable(AppSettings.self, forKey: "appSettings")
```

### App Groups (Sharing with Extensions)

```swift
// Share data between app and widget/extension
let sharedDefaults = UserDefaults(suiteName: "group.com.example.myapp")!
sharedDefaults.set("shared value", forKey: "key")
```

> ⚠️ **Pitfall:** UserDefaults stores data in plaintext on disk. **Never store passwords, tokens, or sensitive data in UserDefaults.** Use Keychain.

> ⚠️ **Pitfall:** UserDefaults is not designed for large amounts of data. Storing large arrays or blobs slows app launch (the whole plist is loaded into memory). Use FileManager or CoreData for substantial data.

> 🎯 **Interview Answer:** "UserDefaults is appropriate for small, non-sensitive user preferences: boolean flags, theme selection, onboarding state, numeric settings. It's backed by a plist file loaded into memory at launch — so keep it small. For sensitive data, use Keychain; for structured data, use Core Data or SwiftData."

---

## Keychain

The Keychain is an encrypted, hardware-backed (Secure Enclave where available) database for sensitive information.

```swift
import Security

enum KeychainError: Error {
    case saveFailed(OSStatus)
    case notFound
    case readFailed(OSStatus)
    case deleteFailed(OSStatus)
}

struct Keychain {
    // Save
    static func save(_ data: Data, service: String, account: String) throws {
        let query: [String: Any] = [
            kSecClass as String:            kSecClassGenericPassword,
            kSecAttrService as String:      service,
            kSecAttrAccount as String:      account,
            kSecValueData as String:        data,
            kSecAttrAccessible as String:   kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        SecItemDelete(query as CFDictionary)
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else { throw KeychainError.saveFailed(status) }
    }
    
    // Read
    static func read(service: String, account: String) throws -> Data {
        let query: [String: Any] = [
            kSecClass as String:       kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecReturnData as String:  true,
            kSecMatchLimit as String:  kSecMatchLimitOne
        ]
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        guard status == errSecSuccess, let data = result as? Data else {
            throw KeychainError.notFound
        }
        return data
    }
    
    // Delete
    static func delete(service: String, account: String) throws {
        let query: [String: Any] = [
            kSecClass as String:       kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account
        ]
        let status = SecItemDelete(query as CFDictionary)
        guard status == errSecSuccess || status == errSecItemNotFound else {
            throw KeychainError.deleteFailed(status)
        }
    }
}

// Convenience for String tokens
extension Keychain {
    static func saveToken(_ token: String, for key: String) throws {
        guard let data = token.data(using: .utf8) else { return }
        try save(data, service: "com.example.app", account: key)
    }
    
    static func readToken(for key: String) throws -> String {
        let data = try read(service: "com.example.app", account: key)
        guard let token = String(data: data, encoding: .utf8) else {
            throw KeychainError.notFound
        }
        return token
    }
}
```

### Keychain Accessibility Options

```swift
// kSecAttrAccessibleWhenUnlocked         — default, accessible when device is unlocked
// kSecAttrAccessibleAfterFirstUnlock     — accessible after first unlock (good for background tasks)
// kSecAttrAccessibleWhenUnlockedThisDeviceOnly  — NOT backed up to iCloud ✅ recommended for tokens
// kSecAttrAccessibleAlways               — accessible always (poor security, avoid)
```

> 💡 **Tip:** Use `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` for auth tokens — not backed up, not transferred to new device, only readable when unlocked.

### Keychain with Biometric Authentication

```swift
import LocalAuthentication

let context = LAContext()
let accessControl = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    .biometryCurrentSet,  // Requires Face ID / Touch ID
    nil
)!

let query: [String: Any] = [
    kSecClass as String:              kSecClassGenericPassword,
    kSecAttrAccount as String:        "sensitiveKey",
    kSecAttrAccessControl as String:  accessControl,
    kSecUseAuthenticationContext as String: context,
    kSecValueData as String:          sensitiveData
]
```

---

## Comparison Table

| Storage | Encrypted | iCloud Backup | Max Size | Use For |
|---------|-----------|---------------|----------|---------|
| UserDefaults | No | Yes | Small | Non-sensitive preferences |
| Keychain | Yes | Configurable | Small | Tokens, passwords, keys |
| FileManager | No* | Configurable | Large | Documents, blobs, cache |
| Core Data | No* | No (by default) | Large | Structured relational data |
| SwiftData | No* | No (by default) | Large | Structured data (iOS 17+) |

*Can add file-level encryption with `NSFileProtection`

---

## Interview Q&A

**Q: Why shouldn't you store auth tokens in UserDefaults?**  
A: UserDefaults is stored as a plaintext plist file on disk. On a jailbroken device or via iTunes backup, this file is readable. The Keychain is encrypted using keys derived from the device passcode and Secure Enclave, making it significantly harder to extract.

**Q: What's the difference between `kSecAttrAccessibleAlways` and `kSecAttrAccessibleWhenUnlocked`?**  
A: `Always` means the item can be read even when the device is locked — poor security for tokens. `WhenUnlocked` means the item is only accessible while the device is unlocked, which is the right choice for auth tokens used during active user sessions.

**Q: How do you share Keychain data between an app and its extensions?**  
A: Use `kSecAttrAccessGroup` with a shared App Group identifier (e.g., `group.com.example.app`). Both the main app and extension must have the same app group entitlement.

---

## Quick Revision

- UserDefaults: plist, non-sensitive, small preferences, shared via suite name
- Keychain: encrypted, hardware-backed, tokens/passwords, use `ThisDeviceOnly` for tokens
- Never store tokens in UserDefaults
- Keychain accessibility: `WhenUnlockedThisDeviceOnly` is the recommended option
- Biometric Keychain: `SecAccessControlCreateWithFlags` with `.biometryCurrentSet`
- App groups: share UserDefaults and Keychain with extensions
