# Security — Network Security

---

## App Transport Security (ATS)

ATS enforces HTTPS for all network connections since iOS 9.

### What ATS Enforces

- TLS 1.2 minimum (TLS 1.3 for new connections)
- Certificate signed by trusted CA
- Perfect Forward Secrecy (ECDHE cipher suites)
- SHA-256+ certificate signature

### Info.plist Configuration

```xml
<!-- ❌ NEVER do this — disables ATS globally -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>

<!-- ✅ Domain-specific exception (requires justification in App Review) -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy-api.internal.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
            <key>NSIncludesSubdomains</key>
            <false/>
        </dict>
    </dict>
</dict>
```

> ⚠️ **Pitfall:** `NSAllowsArbitraryLoads` requires App Review justification. Apple rejects apps using it without a valid reason. Use domain-specific exceptions or fix the server.

---

## Certificate Pinning

```swift
class PinningURLSessionDelegate: NSObject, URLSessionDelegate {
    // SHA-256 hash of the expected public key (Base64)
    private let pinnedKeyHashes: Set<String> = [
        "primaryCertificatePublicKeyHashBase64",
        "backupCertificatePublicKeyHashBase64"   // Always pin multiple!
    ]
    
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
        
        // Evaluate server trust
        var error: CFError?
        guard SecTrustEvaluateWithError(serverTrust, &error) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Check each certificate in chain
        let certificateCount = SecTrustGetCertificateCount(serverTrust)
        for i in 0..<certificateCount {
            guard let cert = SecTrustGetCertificateAtIndex(serverTrust, i),
                  let publicKey = SecCertificateCopyKey(cert),
                  let keyData = SecKeyCopyExternalRepresentation(publicKey, nil) as Data?,
                  let hash = sha256Hash(of: keyData) else { continue }
            
            if pinnedKeyHashes.contains(hash) {
                completionHandler(.useCredential, URLCredential(trust: serverTrust))
                return
            }
        }
        
        // No match — reject connection
        completionHandler(.cancelAuthenticationChallenge, nil)
    }
    
    private func sha256Hash(of data: Data) -> String? {
        var digest = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
        data.withUnsafeBytes {
            _ = CC_SHA256($0.baseAddress, CC_LONG(data.count), &digest)
        }
        return Data(digest).base64EncodedString()
    }
}
```

### Pinning Strategies

| Strategy | Pin | Rotation Risk | Flexibility |
|----------|-----|---------------|-------------|
| Leaf cert | Exact cert | High (cert expires) | Low |
| Intermediate CA | CA cert | Medium | Medium |
| Public key | SPKI hash | Low (same key, new cert) | High ✅ |

> 🎯 **Interview Answer:** "Certificate pinning validates that the server presents a known certificate or public key. Without it, a compromised Certificate Authority could issue a fraudulent cert for your domain, enabling a MITM attack. Pin the public key hash (SPKI) rather than the full certificate — the same key can be reused across cert renewals. Always pin at least two keys (primary + backup) and have a server-side override to disable pinning in emergencies."

---

## Preventing MITM Attacks

```swift
// Beyond pinning — defense in depth:

// 1. Don't ignore SSL errors
// urlSession(_:didReceive:completionHandler:) with `.useCredential` only after validation

// 2. Validate response headers
func validateSecurityHeaders(_ response: HTTPURLResponse) throws {
    let headers = response.allHeaderFields
    
    // Strict Transport Security
    guard headers["Strict-Transport-Security"] != nil else {
        throw SecurityError.missingHSTSHeader
    }
}

// 3. Encrypt payload in addition to TLS (defense in depth for sensitive operations)
func encryptPayload(_ data: Data, publicKey: SecKey) throws -> Data {
    var error: Unmanaged<CFError>?
    guard let encrypted = SecKeyCreateEncryptedData(
        publicKey, .eciesEncryptionCofactorVariableIVX963SHA256AESGCM,
        data as CFData, &error
    ) as Data? else {
        throw error!.takeRetainedValue() as Error
    }
    return encrypted
}
```

---

## Interview Q&A

**Q: What is App Transport Security and what does it do by default?**  
A: ATS forces all HTTP connections to use HTTPS with TLS 1.2+, perfect forward secrecy, and SHA-256 certificate signatures. By default since iOS 9, any `http://` URL fails with an ATS error unless you add exceptions in Info.plist.

**Q: Certificate pinning vs certificate authority trust — tradeoffs?**  
A: Default TLS trusts any cert signed by a CA in the system trust store. Pinning adds a layer: even with a valid CA-signed cert, we reject it if it's not our expected cert/key. Tradeoff: operational complexity — cert rotation requires coordinated app update unless you pin public keys (which can outlive individual certs). Pin multiple keys (primary + backup) to maintain connectivity during rotation.

**Q: How would you disable certificate pinning for testing?**  
A: Environment detection: check for `DEBUG` build flag or a test environment variable, and skip pinning in development. For staging environments, pin staging certs. Never disable pinning in production builds based on a runtime flag — that creates a bypass vulnerability.

---

## Quick Revision

- ATS: enforces HTTPS/TLS 1.2+; avoid `NSAllowsArbitraryLoads`
- Use `NSExceptionDomains` for specific legacy endpoints
- Certificate pinning: validate server cert/key against known hash
- Pin public key hash (SPKI) not full cert — survives cert renewal
- Always pin 2+ keys (primary + backup)
- `URLSessionDelegate.didReceive(challenge:)` for pinning implementation
- MITM defense: HSTS headers, payload encryption for highly sensitive data
