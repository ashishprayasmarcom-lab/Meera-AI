# Jarvis Assistant - Security & Privacy Architecture Guide

## Threat Model & Core Guardrails

When granting an application the Android `AccessibilityService` permission, it inherently receives the ability to observe screen content and dispatch touch gestures. To ensure **absolute personal privacy and zero data leakage**, this codebase implements the following four non-negotiable security layers:

---

### Layer 1: Financial & Credential Package Blacklist
- The `SecurityPolicyManager` maintains a hardcoded set of protected packages:
  - Google Pay (`com.google.android.apps.nbu.paisa.user`)
  - PhonePe (`com.phonepe.app`)
  - Paytm (`net.one97.paytm`)
  - Banking apps (SBI, HDFC, ICICI, Chase, Bank of America, PayPal)
  - 2FA & Password Managers (Bitwarden, 1Password, Google Authenticator)
  - Crypto wallets (Binance, MetaMask, Trust Wallet)
- **Behavior:** Whenever the user opens or commands one of these packages, the service immediately refuses to read screen nodes or dispatch gestures.

---

### Layer 2: Password & PIN Field Shielding
- Android OS marks all credential inputs with `node.isPassword = true`.
- `VoiceAutomationService.injectTextSecure()` inspects this attribute prior to executing any action. If a password or PIN field is detected, the operation is immediately aborted with a critical security alert.

---

### Layer 3: 100% Local Air-Gap Processing Mode
- In **Air-Gap Mode**, zero outbound internet connections are made.
- Voice intents are parsed purely on-device via local heuristic state machines and regex engines.
- No audio recordings, transcripts, or screen content are transmitted to any external server.

---

### Layer 4: Emergency Hardware / Software Kill-Switch
- If the assistant ever misbehaves or if you want immediate privacy, the **Kill-Switch** triggers:
  ```kotlin
  VoiceAutomationService.instance?.disableSelf()
  ```
- This invokes Android's native OS API that tears down the service from the system framework immediately.

---

## ADB Permission & Verification Commands

```bash
# 1. Verify that no telemetry or unauthorized background services exist
adb shell dumpsys activity services com.jarvis.assistant

# 2. Grant Write Secure Settings (Optional, eliminates confirmation dialogs)
adb shell pm grant com.jarvis.assistant android.permission.WRITE_SECURE_SETTINGS

# 3. Whitelist from OS Doze Mode (Optional)
adb shell dumpsys deviceidle whitelist +com.jarvis.assistant
```
