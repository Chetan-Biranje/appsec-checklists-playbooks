# 📱 OWASP MASVS Mobile Application Security Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Standard:** OWASP MASVS v2.0 + MASTG
> **Applies To:** Android & iOS Applications

---

## MASVS-STORAGE — Data Storage Security

- [ ] No sensitive data (PII, tokens, credentials) stored in plaintext
- [ ] No sensitive data in SharedPreferences (Android) / NSUserDefaults (iOS) without encryption
- [ ] No credentials or tokens stored in external storage (SD card)
- [ ] Sensitive data excluded from iOS iCloud backups (`NSURLIsExcludedFromBackupKey`)
- [ ] Android auto-backup excludes sensitive files
- [ ] No secrets in SQLite databases without encryption (SQLCipher recommended)
- [ ] Keychain (iOS) / Android Keystore used for cryptographic keys
- [ ] Screenshots of sensitive screens disabled (`FLAG_SECURE` / `.preventScreenCapture`)
- [ ] Pasteboard cleared after use for sensitive fields (iOS)
- [ ] No sensitive data in application logs

---

## MASVS-CRYPTO — Cryptography

- [ ] No custom cryptographic implementations
- [ ] AES-256-GCM or ChaCha20-Poly1305 used for symmetric encryption
- [ ] RSA-2048+ or EC P-256+ used for asymmetric operations
- [ ] Hardcoded cryptographic keys NOT present in code or resources
- [ ] IV/Nonce is randomly generated per encryption operation
- [ ] Keys stored in Android Keystore / iOS Secure Enclave
- [ ] Deprecated algorithms not used: DES, 3DES, RC4, MD5, SHA1 for security

---

## MASVS-AUTH — Authentication & Session Management

- [ ] Biometric authentication implemented correctly (Android BiometricPrompt / iOS LocalAuthentication)
- [ ] Biometric authentication backed by hardware (StrongBox / Secure Enclave)
- [ ] Re-authentication required for sensitive operations
- [ ] Session token stored in Keychain/Keystore (not SharedPreferences/NSUserDefaults)
- [ ] Token expiry enforced client and server side
- [ ] Token invalidated on logout (server-side revocation)
- [ ] PIN/passcode lockout after failed attempts
- [ ] Screen lock timeout configured

---

## MASVS-NETWORK — Network Communication

- [ ] All network traffic uses TLS 1.2+
- [ ] Certificate pinning implemented for sensitive API communication
- [ ] Pin validation correct: pins on leaf AND intermediate certs
- [ ] Certificate pinning bypass via custom CA not possible on production builds
- [ ] ATS (App Transport Security) enforced on iOS — no `NSAllowsArbitraryLoads`
- [ ] Android Network Security Config restricts cleartext traffic
- [ ] Certificate validation not disabled in custom TrustManagers
- [ ] No sensitive data transmitted via HTTP (even on local/debug builds)

**Android Network Security Config (network_security_config.xml):**
```xml
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set>
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

---

## MASVS-PLATFORM — Platform Interaction

- [ ] Only necessary Android permissions declared (minimal permission model)
- [ ] Dangerous permissions requested at runtime with justification
- [ ] Exported components (Activities, Services, Receivers) reviewed and secured
- [ ] Implicit intents not used to receive sensitive data
- [ ] Deep links / URL schemes validated before processing
- [ ] WebViews: JavaScript disabled unless required
- [ ] WebViews: `setAllowFileAccess(false)`, `setAllowContentAccess(false)`
- [ ] WebViews: No `addJavascriptInterface()` with untrusted input
- [ ] iOS URL scheme handling validates calling app
- [ ] Pasteboard usage restricted to necessary operations

---

## MASVS-CODE — Code Quality

- [ ] Debug features and test code removed from release builds
- [ ] `android:debuggable="false"` in production manifest
- [ ] Log statements with sensitive data removed from release builds
- [ ] App not allowing backup of sensitive data on Android (`android:allowBackup="false"`)
- [ ] Proguard / R8 obfuscation enabled for Android release
- [ ] Code signing enforced (valid certificates, not debug keystores)
- [ ] Third-party SDKs reviewed for data collection / privacy policies
- [ ] No hardcoded secrets, API keys, or credentials in source code or resources

---

## MASVS-RESILIENCE — Anti-Tampering & Anti-Reverse Engineering

*(Required for high-risk apps: banking, healthcare, DRM)*

- [ ] Root/jailbreak detection implemented
- [ ] Debugger detection implemented
- [ ] Emulator detection implemented
- [ ] App integrity / signature verification on startup
- [ ] Anti-tampering protection (hash verification of critical code)
- [ ] Obfuscation applied to security-critical code paths
- [ ] Dynamic analysis protections (anti-hooking, Frida detection)
- [ ] Key protection via white-box cryptography (if required)

---

## Testing Notes

| Attack Vector | Android Tool | iOS Tool |
|---|---|---|
| Static Analysis | jadx, apktool, MobSF | class-dump, MobSF, Hopper |
| Dynamic Analysis | Frida, objection | Frida, objection |
| Network Traffic | Burp + proxy setup | Burp + proxy setup |
| Certificate Pinning Bypass | Frida scripts, apk-mitm | SSL Kill Switch, Frida |
| Runtime Analysis | Drozer | idb, needle |

---

*Last Updated: 2024 | Standard: OWASP MASVS v2.0 | Guide: OWASP MASTG*
