# 🍎 iOS Application Security Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Use Case:** iOS App Security Review, Pentest, Pre-release Audit

---

## Static Analysis (IPA Review)

- [ ] IPA extracted and binary analyzed (class-dump, Hopper, Ghidra)
- [ ] `Info.plist` reviewed for:
  - [ ] `NSAllowsArbitraryLoads: false` (ATS enforced)
  - [ ] No overly broad URL schemes
  - [ ] Privacy usage descriptions (`NSCameraUsageDescription`, etc.) meaningful
  - [ ] Minimum iOS version appropriate (iOS 15+)
- [ ] Hardcoded secrets, API keys searched in binary strings
- [ ] Third-party SDK inventory reviewed for privacy risk
- [ ] Bitcode / symbols stripped from release builds

---

## Data Storage

- [ ] No sensitive data in `NSUserDefaults`
- [ ] No sensitive data in `plist` files
- [ ] Keychain used for credentials, tokens, cryptographic keys
- [ ] Keychain items set with appropriate accessibility:
  - [ ] `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` for most sensitive data
  - [ ] `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly` for background-needed data
- [ ] iCloud sync disabled for sensitive Keychain items (`kSecAttrSynchronizable: false`)
- [ ] File Protection class set on sensitive files (`NSFileProtectionComplete`)
- [ ] Sensitive files excluded from iCloud backup (`NSURLIsExcludedFromBackupKey`)
- [ ] No sensitive data in `tmp/` directory
- [ ] Core Data / SQLite sensitive fields encrypted

---

## Network Security

- [ ] App Transport Security (ATS) enforced — no exceptions in Info.plist
- [ ] TLS 1.2+ used for all connections
- [ ] Certificate pinning implemented (`NSURLSessionDelegate` / TrustKit)
- [ ] Certificate pinning validates both leaf and intermediate certificates
- [ ] Backup pins configured to prevent lockout during certificate rotation
- [ ] Custom URLSession delegates don't bypass certificate validation

---

## Authentication & Biometrics

- [ ] LocalAuthentication framework used correctly
- [ ] `LAContext.evaluatePolicy` used with `deviceOwnerAuthenticationWithBiometrics`
- [ ] Fallback to device passcode (not custom password) where appropriate
- [ ] Biometric key backed by Secure Enclave (`kSecAttrTokenIDSecureEnclave`)
- [ ] `SecAccessControl` created with `biometryCurrentSet` to invalidate on biometric change
- [ ] Jailbreak check performed before sensitive operations

---

## Jailbreak Detection

- [ ] Common jailbreak paths checked (`/Applications/Cydia.app`, `/bin/bash`, `/etc/apt`)
- [ ] Cydia URL scheme check (`cydia://`)
- [ ] Write to system directories check
- [ ] Dylib injection check
- [ ] Symbolic link detection
- [ ] Anti-bypass hardening (obfuscated detection logic)

> ⚠️ Note: Jailbreak detection is an obstacle, not a guarantee. Use server-side controls as primary protection.

---

## IPC & URL Schemes

- [ ] Custom URL schemes validate calling app using `openURL` source app
- [ ] Universal Links (HTTPS-based) used instead of custom URL schemes for deep linking
- [ ] `apple-app-site-association` file reviewed and correctly configured
- [ ] `UIApplication.openURL` handles untrusted input safely
- [ ] Clipboard access minimized; sensitive data not read from clipboard unnecessarily

---

## WebView Security

- [ ] WKWebView used (UIWebView deprecated and removed)
- [ ] JavaScript disabled where not needed (`javaScriptEnabled: false`)
- [ ] `WKScriptMessageHandler` exposed to JS reviewed for injection
- [ ] Loaded URLs validated before navigation
- [ ] File URL access restricted
- [ ] Navigation delegate validates redirects

---

## Binary Protections

- [ ] PIE (Position Independent Executable) enabled
- [ ] Stack canaries enabled
- [ ] ARC (Automatic Reference Counting) enabled
- [ ] Objective-C method swizzling protections in security-critical paths
- [ ] Anti-debugging protections implemented (ptrace, sysctl)

**Verify binary protections:**
```bash
otool -hv <binary> | grep PIE
otool -Iv <binary> | grep stack_chk
```

---

## Privacy & Data Minimization

- [ ] Only necessary permissions declared in Info.plist
- [ ] Permissions requested at point of need with clear rationale
- [ ] Analytics/crash reporting SDKs reviewed for data collection
- [ ] User data deletable (GDPR/CCPA compliance)
- [ ] Sensitive screen content excluded from App Switcher screenshots (background state)

---

## Dynamic Testing Checklist

- [ ] Traffic intercepted via Burp Suite (iOS proxy configured)
- [ ] SSL Kill Switch / TrustKit bypass attempted
- [ ] Jailbreak bypass tested
- [ ] Keychain items dumped (Keychain-Dumper) on jailbroken test device
- [ ] File system reviewed for sensitive data (iExplorer, iFunBox)
- [ ] Memory inspected for sensitive data (Fridump, r2frida)
- [ ] Runtime manipulation via Frida / objection
- [ ] Screenshots of sensitive screens captured (flag bypass tested)

---

## Tools Reference

| Task | Tool |
|---|---|
| Static Analysis | MobSF, Hopper, class-dump, nm |
| Binary Checks | otool, checksec |
| Dynamic Analysis | objection, Frida, lldb |
| Traffic Interception | Burp Suite + iOS proxy |
| Keychain Dump | Keychain-Dumper (jailbroken device) |
| File System | iMazing, iFunBox (non-jailbroken) |
| SSL Pinning Bypass | SSL Kill Switch 2, Frida TrustKit bypass |

---

*Last Updated: 2024 | Reference: OWASP MASTG (iOS)*
