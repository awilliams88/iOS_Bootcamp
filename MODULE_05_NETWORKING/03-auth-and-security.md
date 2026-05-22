# Networking — Auth & Security

---

## Bearer Token Authentication

```swift
class AuthenticatedSession {
    private let session: URLSession
    private var accessToken: String?
    
    func request(_ url: URL) async throws -> Data {
        var request = URLRequest(url: url)
        if let token = accessToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        let (data, response) = try await session.data(for: request)
        try validateResponse(response)
        return data
    }
}
```

---

## The Token Refresh Race Condition

This is one of the most common senior iOS interview scenarios.

**The Problem:** Multiple concurrent requests fail with 401. Each one tries to refresh the token simultaneously, causing multiple refresh calls, token invalidation, and chaos.

```swift
// ❌ BROKEN — race condition
func fetchWithAuth<T: Decodable>(_ request: URLRequest) async throws -> T {
    do {
        return try await performRequest(request)
    } catch NetworkError.unauthorized {
        let newToken = try await refreshToken()  // Called N times concurrently!
        storeToken(newToken)
        return try await performRequest(withToken: newToken, request: request)
    }
}
```

**The Solution — Actor-based token refresh serialization:**

```swift
actor TokenRefresher {
    private var currentRefreshTask: Task<String, Error>?
    
    func validToken(current: String, refresher: () async throws -> String) async throws -> String {
        // If a refresh is already in-flight, wait for it instead of starting a new one
        if let ongoing = currentRefreshTask {
            return try await ongoing.value
        }
        
        let task = Task<String, Error> {
            defer { currentRefreshTask = nil }
            return try await refresher()
        }
        currentRefreshTask = task
        return try await task.value
    }
}

class NetworkClient {
    private let tokenRefresher = TokenRefresher()
    private var token: String = ""
    
    func fetch<T: Decodable>(_ request: URLRequest) async throws -> T {
        var req = authenticated(request)
        
        do {
            return try await performAndDecode(req)
        } catch NetworkError.unauthorized {
            // All concurrent callers hitting here will wait for the SAME refresh
            let newToken = try await tokenRefresher.validToken(current: token) {
                try await self.callRefreshEndpoint()
            }
            self.token = newToken
            req = authenticated(request, token: newToken)
            return try await performAndDecode(req)
        }
    }
}
```

> 🎯 **Interview Answer:** "The token refresh race condition occurs when multiple concurrent requests get 401 responses and all try to refresh the token simultaneously. The solution is to serialize the refresh using an actor. When a refresh is in-flight, subsequent callers wait for the same Task rather than starting a new refresh. Only the first caller triggers the actual refresh — all others get the result."

---

## Certificate Pinning

Certificate pinning rejects TLS connections that don't present an expected certificate/public key, protecting against man-in-the-middle attacks.

```swift
class PinningURLSessionDelegate: NSObject, URLSessionDelegate {
    // Store SHA-256 hash of the server's public key
    private let pinnedPublicKeyHash = "abc123...base64hash..."
    
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Evaluate the server's certificate chain
        var error: CFError?
        guard SecTrustEvaluateWithError(serverTrust, &error) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Get the server's public key
        guard let serverCertificate = SecTrustGetCertificateAtIndex(serverTrust, 0),
              let publicKey = SecCertificateCopyKey(serverCertificate) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Hash the public key and compare
        if let publicKeyHash = hash(publicKey), publicKeyHash == pinnedPublicKeyHash {
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
    
    private func hash(_ publicKey: SecKey) -> String? {
        guard let data = SecKeyCopyExternalRepresentation(publicKey, nil) as Data? else { return nil }
        var digest = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
        data.withUnsafeBytes { CC_SHA256($0.baseAddress, CC_LONG(data.count), &digest) }
        return Data(digest).base64EncodedString()
    }
}

// Usage
let config = URLSessionConfiguration.default
let session = URLSession(
    configuration: config,
    delegate: PinningURLSessionDelegate(),
    delegateQueue: nil
)
```

> ⚠️ **Pitfall:** Hard-coded certificate hashes mean you must ship an app update when certificates rotate. Always pin multiple certificates (primary + backup) and have a server-side configuration endpoint to emergency-disable pinning.

---

## App Transport Security (ATS)

ATS enforces HTTPS for all connections by default since iOS 9.

```xml
<!-- Info.plist — only add exceptions when absolutely required -->
<key>NSAppTransportSecurity</key>
<dict>
    <!-- AVOID: Global exception disables ATS entirely -->
    <!-- <key>NSAllowsArbitraryLoads</key><true/> -->
    
    <!-- BETTER: Domain-specific exception -->
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy.internal.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
        </dict>
    </dict>
</dict>
```

> ⚠️ **Pitfall:** `NSAllowsArbitraryLoads = true` disables ATS globally and will cause App Store review scrutiny. Always use domain-specific exceptions.

---

## OAuth 2.0 Overview

```
1. App opens browser/SFSafariViewController with authorization URL
2. User authenticates, grants permission
3. Auth server redirects to app URL scheme with auth code
4. App exchanges auth code for access token + refresh token (server-to-server)
5. App stores tokens securely in Keychain
6. Access token used for API calls
7. Refresh token used to get new access token when expired

Key URLs:
- Authorization URL: https://auth.server/oauth/authorize
- Token URL: https://auth.server/oauth/token (server-side only)
- Redirect URI: myapp://oauth/callback
```

```swift
// ASWebAuthenticationSession for OAuth
import AuthenticationServices

func authenticateWithOAuth() async throws -> OAuthTokens {
    let authURL = buildAuthorizationURL()
    let callbackScheme = "myapp"
    
    return try await withCheckedThrowingContinuation { continuation in
        let session = ASWebAuthenticationSession(
            url: authURL,
            callbackURLScheme: callbackScheme
        ) { callbackURL, error in
            if let error = error {
                continuation.resume(throwing: error)
                return
            }
            guard let url = callbackURL,
                  let code = URLComponents(url: url, resolvingAgainstBaseURL: false)?
                      .queryItems?.first(where: { $0.name == "code" })?.value else {
                continuation.resume(throwing: AuthError.missingCode)
                return
            }
            Task {
                do {
                    let tokens = try await self.exchangeCodeForTokens(code: code)
                    continuation.resume(returning: tokens)
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }
        session.presentationContextProvider = self
        session.prefersEphemeralWebBrowserSession = true
        session.start()
    }
}
```

---

## Secure Token Storage

```swift
// Store tokens in Keychain, NOT UserDefaults
struct TokenStore {
    static func save(token: String, for key: String) throws {
        let data = token.data(using: .utf8)!
        let query: [String: Any] = [
            kSecClass as String:       kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String:   data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        SecItemDelete(query as CFDictionary)  // Remove existing
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else { throw KeychainError.saveFailed(status) }
    }
    
    static func retrieve(for key: String) throws -> String {
        let query: [String: Any] = [
            kSecClass as String:       kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String:  true,
            kSecMatchLimit as String:  kSecMatchLimitOne
        ]
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        guard status == errSecSuccess, let data = result as? Data,
              let token = String(data: data, encoding: .utf8) else {
            throw KeychainError.notFound
        }
        return token
    }
}
```

---

## Interview Q&A

**Q: Describe the token refresh race condition and your solution.**  
A: Multiple concurrent requests get 401 responses and each tries to refresh the token independently. This can trigger multiple refresh calls, which may invalidate each other (especially with rotation). Solution: use an actor with a shared `Task` for the refresh. First caller starts the Task; concurrent callers `await` the same Task value instead of starting new refreshes.

**Q: What is certificate pinning and what are its tradeoffs?**  
A: Certificate pinning validates that the server presents a known certificate or public key hash, preventing MITM attacks even with a compromised CA. Tradeoff: operational risk — if the pinned cert expires or changes without an app update, all users lose connectivity. Mitigate by pinning multiple certs, implementing a grace period, and having server-side pin override capability.

**Q: Why should auth tokens be stored in Keychain rather than UserDefaults?**  
A: Keychain data is encrypted and protected by the Secure Enclave. UserDefaults is stored in plaintext in an unprotected plist file that can be read from a backup or jailbroken device. Keychain also supports access controls like biometric protection and device-only restrictions.

**Q: What does `NSAllowsArbitraryLoads` do and when is it acceptable?**  
A: It disables App Transport Security globally, allowing HTTP connections. It's almost never acceptable in production — App Store review may reject the app without justification. Use domain-specific exceptions (`NSExceptionDomains`) for specific legacy endpoints instead.

---

## Quick Revision

- Bearer token: `Authorization: Bearer <token>` header
- Token refresh race condition: multiple 401s → multiple refreshes → chaos
- Solution: Actor + shared Task — only one refresh runs, others wait
- Certificate pinning: compare server public key hash to pinned value in `URLSessionDelegate`
- ATS: enforces HTTPS, use domain exceptions not global disable
- OAuth: auth code flow, use `ASWebAuthenticationSession`
- Token storage: **Keychain only**, never UserDefaults
