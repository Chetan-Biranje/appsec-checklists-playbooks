# 🤖 Android Application Security Checklist

> **Level:** Professional / Senior AppSec Engineer
> **Use Case:** Android APK Security Review, Pentest, Pre-release Audit

---

## Static Analysis (APK Review)

- [ ] APK decompiled and reviewed with jadx / apktool
- [ ] `AndroidManifest.xml` reviewed for:
  - [ ] `android:debuggable="false"` in production
  - [ ] `android:allowBackup="false"` (or selective backup rules)
  - [ ] No exported components without intent filters/permissions
  - [ ] `android:usesCleartextTraffic="false"`
  - [ ] Minimum SDK version appropriate (API 28+)
- [ ] Hardcoded secrets searched in: strings.xml, raw resources, BuildConfig, source code
- [ ] `google-services.json` reviewed — no production API keys with overly broad permissions
- [ ] Proguard/R8 mapping applied — sensitive class names obfuscated

---

## Data Storage

- [ ] SharedPreferences not used for sensitive data (use EncryptedSharedPreferences)
- [ ] SQLite databases encrypted (SQLCipher) for sensitive data
- [ ] World-readable or world-writable file permissions not set
- [ ] Internal storage used for sensitive files (not external)
- [ ] No sensitive data written to `/sdcard/` or `Environment.getExternalStorageDirectory()`
- [ ] Room database with encryption for sensitive local storage
- [ ] Android Keystore used for cryptographic keys
- [ ] `FLAG_SECURE` set on Activities displaying sensitive info

---

## Network Security

- [ ] Network Security Config defined and restricts cleartext
- [ ] Certificate pinning implemented for all API endpoints
- [ ] Pin backup (backup pins) implemented to prevent lockout
- [ ] Custom `TrustManager` not disabled (`X509TrustManager` overrides reviewed)
- [ ] Custom `HostnameVerifier` not accepting all hosts
- [ ] OkHttp / Retrofit configured with certificate pinner

---

## IPC & Component Security

- [ ] Exported `Activity` components require permissions where needed
- [ ] Exported `Service` components protected
- [ ] `BroadcastReceiver` not exporting sensitive data via implicit broadcasts
- [ ] `ContentProvider` exported only if required; permissions enforced
- [ ] `PendingIntent` created with `FLAG_IMMUTABLE` (API 31+)
- [ ] Deep links validate the calling app and data
- [ ] Intent data validated before processing

---

## WebView Security

- [ ] `setJavaScriptEnabled(false)` unless JavaScript is required
- [ ] `addJavascriptInterface()` not used with untrusted content
- [ ] `setAllowFileAccess(false)` — prevent local file access
- [ ] `setAllowFileAccessFromFileURLs(false)`
- [ ] `setAllowUniversalAccessFromFileURLs(false)`
- [ ] WebView URL validated before loading
- [ ] WebView loaded over HTTPS only

---

## Authentication & Biometrics

- [ ] Android BiometricPrompt API used (not deprecated FingerprintManager)
- [ ] Biometric auth backed by `CryptoObject` (hardware-backed)
- [ ] `setInvalidatedByBiometricEnrollment(true)` set for keys
- [ ] PIN/password fallback secured
- [ ] Session token stored in EncryptedSharedPreferences or Keystore

---

## Permissions

- [ ] Dangerous permissions requested at runtime with rationale shown
- [ ] Permissions not requested unnecessarily
- [ ] Permission results handled gracefully (app functions without optional permissions)
- [ ] `READ_PHONE_STATE`, `ACCESS_FINE_LOCATION`, `READ_CONTACTS` justified
- [ ] Custom permissions defined with `protectionLevel="signature"` for IPC

---

## Anti-Tamper & Root Detection

- [ ] Root detection implemented (SafetyNet/Play Integrity API)
- [ ] Play Integrity API used (replaced SafetyNet in 2024)
- [ ] Debug detection in place
- [ ] Emulator detection implemented
- [ ] App signature verified at runtime
- [ ] Frida/Magisk detection for high-risk apps (banking, fintech)

---

## Dynamic Testing Checklist

- [ ] Traffic intercepted via Burp Suite proxy
- [ ] Certificate pinning bypass attempted (Frida, apk-mitm)
- [ ] Root bypass tested
- [ ] Backup extraction attempted (`adb backup`)
- [ ] Logcat reviewed for sensitive data leaks (`adb logcat`)
- [ ] SQLite databases extracted and reviewed
- [ ] SharedPreferences files reviewed
- [ ] Runtime method hooking via Frida tested
- [ ] Memory dump reviewed for sensitive data at rest in RAM

---

## Tools Reference

| Task | Tool |
|---|---|
| Decompile APK | jadx-gui, apktool |
| Static Analysis | MobSF, semgrep-rules-android |
| Dynamic Analysis | objection, Frida |
| Traffic Interception | Burp Suite + Android proxy |
| Backup Extraction | adb backup, Android Backup Extractor |
| PIN/Biometric bypass | Frida scripts (objection) |
| Root Detection bypass | Magisk + LSPosed (for testing only) |

---

*Last Updated: 2024 | Reference: OWASP MASTG (Android)*
