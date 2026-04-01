# 🚨 Incident Response Playbook — Mobile App Data Leakage

> **Severity:** High–Critical
> **CWE:** CWE-312, CWE-359, CWE-532
> **OWASP MASVS:** MASVS-STORAGE, MASVS-NETWORK
> **Playbook Version:** 1.0

---

## Overview

Mobile data leakage occurs when sensitive data stored on a mobile device or transmitted by a mobile app is accessible to unauthorized parties — via insecure storage, unencrypted transmission, logs, or inter-process communication.

---

## Common Leakage Scenarios

| Scenario | Risk Level |
|---|---|
| PII/tokens in SharedPreferences / NSUserDefaults | High |
| Sensitive data in application logs (Logcat/Console) | High |
| Unencrypted SQLite database | High |
| Cleartext traffic (HTTP) | Critical |
| Sensitive data in clipboard | Medium |
| Data exposed via screenshots (task switcher) | Medium |
| Sensitive data in crash reports sent to analytics | High |
| Backup files containing sensitive data | High |

---

## Phase 1 — Detection & Triage

**Detection Sources:**
- Security researcher / bug bounty disclosure
- Internal mobile security assessment
- Privacy audit / GDPR review
- Store review rejection citing privacy violation
- Third-party SDK privacy scan (Exodus Privacy)

**Triage Questions:**
1. What data is leaking? (PII, tokens, credentials, health data)
2. How is it leaking? (Storage, network, logs, IPC)
3. Who can access it? (Other apps, backup extract, network attacker)
4. What platform? (Android, iOS, both)
5. Is it in production builds or debug only?

---

## Phase 2 — Containment

### Cleartext Network Traffic
- [ ] Issue app update disabling HTTP immediately
- [ ] Server-side: enforce HTTPS, reject HTTP connections
- [ ] Push emergency update via store expedited review

### Sensitive Data in Logs
- [ ] Deploy app update removing log statements
- [ ] Backend: purge log aggregation for affected time period

### Third-Party SDK Leaking Data
- [ ] Disable SDK data collection feature if toggle available
- [ ] Coordinate hotfix removing or updating SDK
- [ ] Review SDK privacy policy and DPA

---

## Phase 3 — Investigation

### Android Investigation
```bash
# Check SharedPreferences
adb shell run-as com.example.app cat /data/data/com.example.app/shared_prefs/prefs.xml

# Dump SQLite database
adb shell run-as com.example.app sqlite3 /data/data/com.example.app/databases/app.db .dump

# Check Logcat for sensitive data (requires device)
adb logcat | grep -i "password\|token\|email\|credit\|ssn"

# Check for backup
adb backup -noapk -f backup.ab com.example.app
java -jar android-backup-extractor.jar backup.ab backup.tar
tar xf backup.tar
```

### iOS Investigation
```bash
# On jailbroken test device — check NSUserDefaults plist
cat /var/mobile/Containers/Data/Application/<UUID>/Library/Preferences/com.example.app.plist

# Check SQLite
sqlite3 /var/mobile/Containers/Data/Application/<UUID>/Documents/app.db .dump

# Check Keychain
keychain-dumper -a

# Check app logs (on simulator)
xcrun simctl spawn booted log stream --predicate 'subsystem == "com.example.app"'
```

---

## Phase 4 — Remediation

### Secure Storage Fix (Android)
```kotlin
// VULNERABLE: SharedPreferences plaintext
val prefs = getSharedPreferences("app_prefs", MODE_PRIVATE)
prefs.edit().putString("auth_token", token).apply()

// SECURE: EncryptedSharedPreferences
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedPrefs = EncryptedSharedPreferences.create(
    context, "secret_prefs", masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
encryptedPrefs.edit().putString("auth_token", token).apply()
```

### Secure Storage Fix (iOS)
```swift
// VULNERABLE: UserDefaults
UserDefaults.standard.set(authToken, forKey: "auth_token")

// SECURE: Keychain
import Security

func saveToKeychain(key: String, value: String) {
    let data = value.data(using: .utf8)!
    let query: [CFString: Any] = [
        kSecClass: kSecClassGenericPassword,
        kSecAttrAccount: key,
        kSecValueData: data,
        kSecAttrAccessible: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
    ]
    SecItemDelete(query as CFDictionary)
    SecItemAdd(query as CFDictionary, nil)
}
```

### Remove Sensitive Logs
```kotlin
// VULNERABLE
Log.d("Auth", "Token: $authToken")
Log.e("DB", "Password: $dbPassword")

// SECURE: Remove entirely in release, or use:
if (BuildConfig.DEBUG) {
    Log.d("Auth", "Login successful")  // Log success only, never sensitive values
}
```

---

## Phase 5 — Recovery & Release

- [ ] Hotfix prepared, reviewed, tested on device
- [ ] Expedited app store review requested (24hr for critical security)
- [ ] Force-update mechanism triggered (if implemented)
- [ ] Push notification sent to users to update immediately (if breach)
- [ ] Token rotation / password reset if tokens/credentials were exposed

---

## Phase 6 — Post-Incident

- [ ] DAST scan of new build before release
- [ ] Privacy impact assessment updated
- [ ] MASVS checklist re-evaluated for full app
- [ ] Mobile security testing added to release gate
- [ ] Developer training on mobile data security
- [ ] Third-party SDKs audited for data collection

---

*Playbook Version: 1.0 | Reference: OWASP MASTG, OWASP MASVS v2.0*
