# Security — Secure Storage

---

## What Belongs in the Keychain

```
✅ Keychain:
- Auth tokens (access token, refresh token)
- Passwords
- Cryptographic keys
- Certificates
- Biometric-protected secrets

❌ Never in UserDefaults or plain files:
- Auth tokens
- Passwords
- API keys
- Personal data
```

---

## Keychain Best Practices

```swift
import Security

struct SecureStorage {
    private static let service = Bundle.main.bundleIdentifier ?? "com.app"
    
    // Save — always use ThisDeviceOnly to prevent iCloud sync of tokens
    static func save(data: Data, account: String, 
                     accessibility: CFString = kSecAttrAccessibleWhenUnlockedThisDeviceOnly) throws {
        let query: [String: Any] = [
            kSecClass as String:             kSecClassGenericPassword,
            kSecAttrService as String:       service,
            kSecAttrAccount as String:       account,
            kSecValueData as String:         data,
            kSecAttrAccessible as String:    accessibility
        ]
        
        // Delete existing item first
        SecItemDelete(query as CFDictionary)
        
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw SecurityError.keychainError(status)
        }
    }
    
    // Load
    static func load(account: String) throws -> Data {
        let query: [String: Any] = [
            kSecClass as String:       kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecReturnData as String:  true,
            kSecMatchLimit as String:  kSecMatchLimitOne
        ]
        
        var item: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &item)
        
        guard status == errSecSuccess, let data = item as? Data else {
            throw SecurityError.keychainNotFound
        }
        return data
    }
    
    // Delete
    static func delete(account: String) {
        let query: [String: Any] = [
            kSecClass as String:       kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account
        ]
        SecItemDelete(query as CFDictionary)
    }
    
    // Delete all app keychain items (on logout)
    static func clearAll() {
        let query: [String: Any] = [kSecClass as String: kSecClassGenericPassword]
        SecItemDelete(query as CFDictionary)
    }
}
```

---

## File Protection

```swift
// Writing sensitive data to file with encryption
let sensitiveData = try JSONEncoder().encode(userData)
let url = FileManager.default
    .urls(for: .applicationSupportDirectory, in: .userDomainMask)[0]
    .appendingPathComponent("userData.json")

// Complete protection — file only accessible when device unlocked
try sensitiveData.write(to: url, options: .completeFileProtection)

// Set protection on existing file
try FileManager.default.setAttributes(
    [.protectionKey: FileProtectionType.complete],
    ofItemAtPath: url.path
)
```

---

## Excluding from Backup

```swift
// Mark file to exclude from iCloud/iTunes backup
var resourceValues = URLResourceValues()
resourceValues.isExcludedFromBackup = true
try url.setResourceValues(resourceValues)

// Use this for:
// - Downloaded media (movies, music) — Apple policy requires this
// - Regenerable cache data
// - Sensitive data that should stay on-device
```

---

## Secure Data in Memory

```swift
// Clear sensitive data after use
func processPassword(_ password: String) {
    var mutablePassword = Array(password.utf8)
    defer {
        // Zero out memory (best effort in Swift)
        for i in mutablePassword.indices {
            mutablePassword[i] = 0
        }
    }
    // Use password...
}

// Don't log sensitive data
// ❌
print("Token: \(authToken)")

// ✅ 
#if DEBUG
print("Token present: \(authToken != nil)")
#endif
```

---

## Interview Q&A

**Q: A junior developer stored auth tokens in UserDefaults. What are the risks?**  
A: UserDefaults is a plaintext plist file stored at a known path in the app container. On a jailbroken device, any app with root access can read it. On a non-jailbroken device, iTunes backup and iCloud backup can include it (unless excluded). If a user's iCloud account is compromised, tokens could be extracted. Move all tokens to Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`.

**Q: What does `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` mean?**  
A: The item is only accessible when the device is unlocked (after the user authenticates). `ThisDeviceOnly` means it's not included in iCloud Keychain sync or backups — it stays on the device. This is the right choice for auth tokens since you don't want tokens from one device usable on another.

**Q: How do you handle sensitive data in memory?**  
A: Minimize time sensitive data exists in memory. Don't log it. Don't store it in strings longer than necessary (strings are immutable and hard to clear). For passwords, use `SecureField` in SwiftUI / `isSecureTextEntry` in UIKit. After use, attempt to zero out the memory buffer.

---

## Quick Revision

- Tokens → Keychain (`WhenUnlockedThisDeviceOnly`)
- Never → UserDefaults, NSUserDefaults, plain files
- File protection: `.completeFileProtection` on sensitive files
- Exclude from backup: `isExcludedFromBackup = true` for sensitive/downloadable content
- Clear keychain on logout: `SecItemDelete` for all app items
- Don't log tokens; use DEBUG guards for diagnostic output
