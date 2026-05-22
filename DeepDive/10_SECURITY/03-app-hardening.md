# Security — App Hardening

---

## Jailbreak Detection

Jailbroken devices bypass iOS sandboxing, giving attackers root access. Detection is a defense layer — not a silver bullet.

```swift
struct JailbreakDetector {
    static var isJailbroken: Bool {
        #if targetEnvironment(simulator)
        return false  // Simulator is not jailbroken
        #else
        return checkSuspiciousPaths()
            || checkSandboxViolation()
            || checkDynamicLibraries()
            || checkSymbolicLinks()
        #endif
    }
    
    // Check for common jailbreak files
    private static func checkSuspiciousPaths() -> Bool {
        let paths = [
            "/Applications/Cydia.app",
            "/Library/MobileSubstrate/MobileSubstrate.dylib",
            "/bin/bash",
            "/usr/sbin/sshd",
            "/etc/apt",
            "/private/var/lib/apt/",
            "/usr/bin/ssh"
        ]
        return paths.contains { FileManager.default.fileExists(atPath: $0) }
    }
    
    // Try to write outside sandbox — should fail on non-jailbroken device
    private static func checkSandboxViolation() -> Bool {
        let path = "/private/jailbreak_test_\(UUID().uuidString)"
        do {
            try "test".write(toFile: path, atomically: true, encoding: .utf8)
            try? FileManager.default.removeItem(atPath: path)
            return true  // Succeeded — likely jailbroken
        } catch {
            return false  // Failed — good, sandboxed
        }
    }
    
    // Check for injected dynamic libraries
    private static func checkDynamicLibraries() -> Bool {
        let suspiciousLibs = ["SubstrateLoader", "CydiaSubstrate", "FridaGadget"]
        for i in 0..<_dyld_image_count() {
            if let name = _dyld_get_image_name(i) {
                let imageName = String(cString: name)
                if suspiciousLibs.contains(where: { imageName.contains($0) }) {
                    return true
                }
            }
        }
        return false
    }
    
    private static func checkSymbolicLinks() -> Bool {
        // /Applications is a symlink on jailbroken devices
        return (try? FileManager.default.destinationOfSymbolicLink(atPath: "/Applications")) != nil
    }
}

// Usage
if JailbreakDetector.isJailbroken {
    // Log to analytics, show warning, or restrict functionality
    // Don't just crash — determined attackers will patch that check
}
```

> ⚠️ **Important:** Jailbreak detection is a speed bump, not a wall. Sophisticated attackers (Frida, Substrate hooking) can bypass these checks. Use it as one layer of a defense-in-depth strategy.

> 🎯 **Interview Answer:** "Jailbreak detection adds friction for attackers but isn't foolproof. Common checks: suspicious file paths (Cydia.app, MobileSubstrate), writing outside sandbox, injected dylibs, symlink inspection. The right response depends on the app — financial apps might block access, others might just log the event. Since checks can be bypassed, combine with server-side validation and behavioral anomaly detection."

---

## Anti-Debugging & Anti-Tampering

```swift
// Detect if debugger is attached
func isDebuggerAttached() -> Bool {
    var info = kinfo_proc()
    var mib: [Int32] = [CTL_KERN, KERN_PROC, KERN_PROC_PID, getpid()]
    var size = MemoryLayout<kinfo_proc>.stride
    sysctl(&mib, UInt32(mib.count), &info, &size, nil, 0)
    return (info.kp_proc.p_flag & P_TRACED) != 0
}

// Note: In production, use this judiciously
// Legitimate use: banking/payment apps, DRM
// Don't break debugger access during development
```

---

## Secure Coding Practices

```swift
// 1. Input validation — never trust user input
func validateEmail(_ input: String) -> Bool {
    let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
    return NSPredicate(format: "SELF MATCHES %@", emailRegex).evaluate(with: input)
}

// 2. Avoid logging sensitive data
func processPayment(cardNumber: String) {
    // ❌
    // print("Processing card: \(cardNumber)")
    
    // ✅
    let masked = String(repeating: "*", count: cardNumber.count - 4) 
        + cardNumber.suffix(4)
    os_log("Processing card ending: %@", masked)
}

// 3. Secure random number generation
import CryptoKit
func generateSecureToken() -> String {
    let key = SymmetricKey(size: .bits256)
    return key.withUnsafeBytes { Data($0).base64EncodedString() }
}

// 4. Use CryptoKit for cryptography — never roll your own
func encryptData(_ data: Data, key: SymmetricKey) throws -> Data {
    let sealed = try AES.GCM.seal(data, using: key)
    return sealed.combined!
}

func decryptData(_ encrypted: Data, key: SymmetricKey) throws -> Data {
    let box = try AES.GCM.SealedBox(combined: encrypted)
    return try AES.GCM.open(box, using: key)
}

// 5. Biometric authentication
import LocalAuthentication
func authenticateWithBiometrics() async throws {
    let context = LAContext()
    var error: NSError?
    
    guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
        throw AuthError.biometricsUnavailable
    }
    
    try await context.evaluatePolicy(
        .deviceOwnerAuthenticationWithBiometrics,
        localizedReason: "Authenticate to access your account"
    )
}
```

---

## Preventing Sensitive Data in Screenshots

```swift
// Hide sensitive content in app switcher screenshot
class SecureViewController: UIViewController {
    private var blurView: UIVisualEffectView?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupObservers()
    }
    
    private func setupObservers() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(appWillResignActive),
            name: UIApplication.willResignActiveNotification,
            object: nil
        )
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(appDidBecomeActive),
            name: UIApplication.didBecomeActiveNotification,
            object: nil
        )
    }
    
    @objc private func appWillResignActive() {
        let blur = UIVisualEffectView(effect: UIBlurEffect(style: .systemMaterial))
        blur.frame = view.bounds
        blur.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(blur)
        blurView = blur
    }
    
    @objc private func appDidBecomeActive() {
        blurView?.removeFromSuperview()
        blurView = nil
    }
}
```

---

## Interview Q&A

**Q: How would you implement jailbreak detection and what are its limitations?**  
A: Check for suspicious files (Cydia.app, MobileSubstrate), try to write outside the sandbox (should fail), inspect loaded dylibs for injection frameworks (Frida, Substrate), and check for symlinks. Limitations: tools like Frida and Liberty Lite can hook these checks and return false results. Use detection as one layer — combine with server-side validation, behavioral anomaly detection, and risk-based authentication rather than relying on it alone.

**Q: What is the right response when jailbreak is detected?**  
A: Depends on the app's risk profile. Banking/payment apps may block all access with a clear message. Most apps should log the event to analytics and optionally restrict sensitive features (biometric data access, payment flows). Hard-crashing the app is easily bypassed. A layered response with server awareness is more effective.

**Q: How do you prevent sensitive data appearing in screenshots (app switcher)?**  
A: Listen for `UIApplication.willResignActiveNotification` (fires before screenshot is taken for app switcher) and overlay a blur or placeholder view. Remove it on `didBecomeActiveNotification`.

**Q: Why should you use CryptoKit instead of rolling your own cryptography?**  
A: Cryptography is notoriously easy to get wrong. CryptoKit provides battle-tested implementations of AES-GCM, ChaCha20-Poly1305, ECDSA, and Curve25519. It uses the Secure Enclave on supported devices for key storage. Rolling your own crypto risks subtle implementation errors that create vulnerabilities.

---

## Quick Revision

- Jailbreak detection: file paths, sandbox escape, dylib inspection, symlinks
- Detection is not foolproof — use as one layer of defense
- Anti-debugging: `sysctl` + `P_TRACED` flag
- Secure coding: validate input, don't log sensitive data, use `os_log`
- Cryptography: use CryptoKit — never roll your own
- `SymmetricKey(size: .bits256)` for secure random key generation
- App switcher screenshot: blur on `willResignActive`, remove on `didBecomeActive`
- Biometrics: `LAContext.evaluatePolicy`
