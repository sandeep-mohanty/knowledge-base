# 🎤 Passkey Presentation: Audience Q&A Guide

## 📋 QUICK REFERENCE CHEAT SHEET (Print This Page)

### **Top 10 Questions & 30-Second Answers**

| # | Question | Quick Answer |
|---|----------|--------------|
| 1 | **What are passkeys?** | Cryptographic key pairs replacing passwords. Private key stays on your device; only public key on server. 6x faster, 100% phishing-resistant. |
| 2 | **Why more secure than passwords?** | No shared secret, domain-bound (won't work on fake sites), server breach safe (only public keys exposed). |
| 3 | **Phishing protection?** | Passkey for `github.com` ONLY works on `github.com`. Fake sites automatically rejected. Invisible protection. |
| 4 | **Biometric data sent to server?** | **Never.** Processed locally on device (Secure Enclave/TPM). Server only gets cryptographic signature. |
| 5 | **Lost all devices?** | Backup codes, hardware security keys, service recovery flows. Credential Exchange Protocol (2025-26) will enable cross-platform transfers. |
| 6 | **Ubuntu/Linux support?** | Yes! Chrome (synced via Google), YubiKey (device-bound), SSH/sudo/disk encryption all supported on Ubuntu 24.04. |
| 7 | **Speed comparison?** | Passkey: 3-4 seconds, 98% success. Password: ~20 seconds, 70% success. Amazon: 6x faster with passkeys. |
| 8 | **Is it MFA?** | Yes! Something you HAVE (device) + Something you ARE/KNOW (biometric/PIN). One tap replaces password + OTP. |
| 9 | **Cloud sync safe?** | End-to-end encrypted. iCloud/Google can't decrypt—only your trusted devices can. |
| 10 | **Future of passwords?** | 100+ major services already support passkeys. Gradual transition, but passwords becoming obsolete. |

---

## ⚡ ELEVATOR PITCH VERSION (30-Second Answers)

Use these for quick, confident responses during the presentation:

**Q: What are passkeys in one sentence?**
> "Passkeys are cryptographic keys that replace passwords—your private key never leaves your device, making them 6x faster and completely phishing-resistant."

**Q: Why should my company adopt passkeys?**
> "Passkeys eliminate password-related breaches (77% of hacks), reduce helpdesk costs for password resets, and improve user experience with 3-second logins instead of 20-second password+OTP combos."

**Q: Are they really unhackable?**
> "Not unhackable, but they eliminate entire attack categories: no phishing, no credential stuffing, no database breach risks. The attack surface shrinks dramatically."

**Q: What if users resist change?**
> "Users love passkeys—98% success rate vs 70% for passwords. Amazon saw 6x faster sign-ins. Once people use them, they never want to go back to passwords."

**Q: Implementation cost?**
> "Minimal. Use open-source libraries like SimpleWebAuthn. Major platforms (Apple, Google, Microsoft) already support them. Your users already have the hardware (phones, laptops)."

**Q: Regulatory compliance?**
> "Yes. FIDO2 hardware keys provide audit trails, attestation data, and FIPS 140-2 compliance. Perfect for GDPR, NIS2, healthcare, and government requirements."

---

## 📚 COMPREHENSIVE Q&A BY TOPIC

### **TOPIC 1: The Password Problem**

**Q: What's wrong with passwords?**
Passwords are shared secrets existing in two places (user device + server), making them vulnerable to breaches, phishing, and reuse. Statistics: 77% of breaches involve stolen credentials, 48% abandon purchases due to forgotten passwords, 3000% increase in AI phishing attacks.

**Q: Why doesn't 2FA (SMS OTP) solve this?**
SMS OTPs are still phishable via real-time relay attacks. A fake site can capture both your password AND the OTP simultaneously. Passkeys eliminate the password entirely—no secret to steal.

**Q: What about password managers?**
Password managers help but are themselves targets requiring a master password. Passkeys eliminate the need for a master password entirely—authentication is a cryptographic operation, not a typed secret.

---

### **TOPIC 2: FIDO History & Standards**

**Q: Who created FIDO and when?**
FIDO Alliance founded in 2012 by PayPal, Lenovo, Google, Apple, Microsoft, Amazon. Three generations: UAF (2014, mobile biometrics), U2F (2014, security keys as 2FA), FIDO2 (2018, full passwordless web login).

**Q: What's the difference between UAF, U2F, and FIDO2?**
- **UAF**: Biometric passwordless for mobile apps only (not web)
- **U2F**: Physical security keys as 2FA (still needed passwords)
- **FIDO2**: Full passwordless web login (current standard, what passkeys use)

**Q: Is FIDO owned by one company?**
No, it's an open standard backed by 300+ companies (Apple, Google, Microsoft, Amazon, Meta, Yubico). Not owned by any single entity—anyone can implement it.

---

### **TOPIC 3: Cryptography Basics**

**Q: How does public/private key cryptography work in passkeys?**
During registration: Device generates key pair → private key stays on device (in Secure Enclave/TPM), public key goes to server. During login: Server sends challenge → device signs with private key → server verifies with stored public key. Private key never leaves device.

**Q: What is the challenge-response pattern?**
Server sends random challenge (nonce) → device signs it with private key → server verifies signature. Prevents replay attacks because each challenge is unique and one-time use.

**Q: Can you derive the private key from the public key?**
No. Mathematically impossible with current computing power. This is the foundation of asymmetric cryptography.

---

### **TOPIC 4: Passkey Mechanics**

**Q: What gets stored where?**
**Your device**: Private key (never leaves) + credential metadata (Credential ID, RP ID, user handle)
**Server**: Public key (useless for authentication) + user record with Credential ID

**Q: What is RP ID and why does it matter?**
RP ID = the domain (e.g., `github.com`). Passkeys are cryptographically bound to this domain. If you visit `github-login.com` (phishing site), the authenticator refuses to sign. This is the core phishing protection.

**Q: What is a "discoverable credential"?**
A passkey the authenticator can find automatically based on the website's domain—no username needed. This is why you can sign in with just a fingerprint tap; the authenticator already knows which key to use for `github.com`.

**Q: What is attestation?**
Cryptographic proof that your authenticator is a trusted device (e.g., FIDO Certified Level 3). Used in enterprise/government to verify specific hardware models. Most consumer apps use `none` attestation for privacy.

---

### **TOPIC 5: Security Architecture**

**Q: How do passkeys defeat common attacks?**
- **Phishing**: Domain binding prevents use on fake sites
- **Database breach**: Only public keys stored—useless to attackers
- **Credential stuffing**: Each passkey unique per site—can't reuse across sites
- **Replay attacks**: Fresh challenge each login—captured responses expire immediately
- **Keyloggers**: Nothing typed—biometric/PIN stays on device

**Q: What if the server storing my public key gets hacked?**
Nothing. Public keys are useless for authentication—they can only verify signatures, not create them. The attacker would need your private key, which never left your device and lives in secure hardware.

**Q: Can passkeys be stolen via keyloggers or screen sharing?**
No. Nothing is typed (no keystrokes to record), and the private key operation happens inside secure hardware (Secure Enclave/TPM), invisible to screen sharing. Each challenge is unique, so captured responses can't be replayed.

**Q: What if someone watches me authenticate?**
They see you touch your fingerprint sensor. But the signature is unique to that specific challenge and can't be reused. Watching doesn't help them authenticate as you.

---

### **TOPIC 6: Passkey Types & Storage**

**Q: Synced vs Device-Bound passkeys—which should I use?**
- **Synced** (iCloud/Google/1Password): Best UX, works across devices, survives device loss, end-to-end encrypted
- **Device-bound** (YubiKey/TPM): Highest security, no cloud dependency, but lost device = lost passkey
**Recommendation**: Most users should use synced for convenience. Enterprise/government should use device-bound for maximum security.

**Q: Where are passkeys stored on different platforms?**
| Platform | Storage | Synced? |
|----------|---------|---------|
| iOS/macOS | iCloud Keychain | ✅ Yes (Apple devices) |
| Android | Google Password Manager | ✅ Yes (Android devices) |
| Windows 11 | Windows Hello/TPM | ⚠️ Partial (Microsoft Account) |
| YubiKey | Hardware chip | ❌ No |
| Ubuntu 24.04 | GNOME Keyring/TPM/security keys | Varies |

**Q: Can I use the same passkey on multiple devices?**
With synced passkeys (iCloud/Google Password Manager), yes—they automatically sync across your devices. With device-bound passkeys (YubiKey), you must register separately on each device, but the same hardware key works everywhere.

---

### **TOPIC 7: Cross-Device Authentication**

**Q: How does Cross-Device Authentication (CDA) work?**
You can log into a laptop using a passkey stored on your phone. The laptop shows a QR code, your phone scans it (BLE verifies physical proximity), you confirm with biometric on phone, and authentication completes via secure tunnel. Your phone's private key never leaves the phone.

**Q: Why is BLE (Bluetooth) used in CDA?**
BLE verifies physical proximity—prevents an attacker in another country from using your phone's passkey to log into your laptop. The actual cryptographic communication happens over a secure internet tunnel, not Bluetooth.

**Q: Can I use CDA on Ubuntu?**
Yes, if using Chrome with a phone that has passkeys (iPhone/Android). The QR code flow works cross-platform.

---

### **TOPIC 8: Ubuntu 24.04 Implementation**

**Q: Can I use passkeys on Ubuntu 24.04?**
Yes. Chrome supports synced passkeys via Google Password Manager. Firefox supports WebAuthn but lacks built-in sync. Hardware security keys (YubiKey) work fully for device-bound passkeys.

**Q: How do I set up FIDO2 tools on Ubuntu?**
```bash
sudo apt install libfido2-1 fido2-tools libpam-u2f
fido2-token -L  # List connected devices
```

**Q: Can I use my YubiKey for SSH on Ubuntu?**
Yes! Generate a FIDO2-backed SSH key:
```bash
ssh-keygen -t ed25519-sk -f ~/.ssh/id_ed25519_sk
```
The private key lives in your YubiKey hardware—you just touch it when connecting. Even if your laptop is stolen, the attacker can't use your SSH key.

**Q: How do I make sudo require a YubiKey touch?**
1. Install: `sudo apt install libpam-u2f`
2. Register: `pamu2fcfg -o pam://hostname > ~/.config/Yubico/u2f_keys`
3. Add to `/etc/pam.d/sudo`: `auth required pam_u2f.so`
Now sudo requires both password AND YubiKey touch (true 2FA).

**Q: Can I unlock disk encryption with a YubiKey?**
Yes:
```bash
sudo systemd-cryptenroll --fido2-device=auto /dev/sda3
```
On boot: plug in YubiKey → touch → disk unlocks automatically.

**Q: What if my YubiKey isn't detected on Ubuntu?**
Install udev rules: `sudo apt install libu2f-udev`. Then run `sudo udevadm trigger`. Verify with `fido2-token -L`.

---

### **TOPIC 9: Comparison & Migration**

**Q: Passkeys vs Passwords vs MFA—quick comparison?**
| Method | Security | Speed | Phishing Resistant |
|--------|----------|-------|-------------------|
| Password | ⭐☆☆☆☆ | ~20s, 70% success | ❌ |
| Password + SMS OTP | ⭐⭐⭐☆☆ | ~45s, 60% success | ❌ (relay attacks) |
| Passkey | ⭐⭐⭐⭐⭐ | ~3-4s, 98% success | ✅ |

Amazon: 6x faster sign-in. Google: 4x better success rate. CVS Health: 98% reduction in mobile account takeover fraud.

**Q: How do I start using passkeys?**
1. Enable on your Google/Apple account (synced passkeys)
2. Add to GitHub, Amazon, PayPal (consumer services)
3. For enterprise: Get YubiKey, enable on corporate SSO
4. On Ubuntu: Install Chrome, use Google Password Manager sync

**Q: What if I lose all my devices with synced passkeys?**
Options: Use backup codes/recovery email, use a hardware security key as backup, or go through identity verification with the service. The Credential Exchange Protocol (in development) will make cross-ecosystem transfers easier.

---

### **TOPIC 10: Future & Trends**

**Q: Will passwords disappear completely?**
Likely yes, but gradually. Services will maintain backup methods during transition. 100+ major services already support passkeys. The FIDO Alliance's goal is universal passwordless authentication.

**Q: What is the Credential Exchange Protocol (CXP)?**
A FIDO Alliance working draft enabling secure transfer of passkeys between ecosystems (iPhone → Android, iCloud → 1Password). Solves the "platform lock-in" problem. Expected 2025-2026.

**Q: What's on the passkey roadmap?**
- **2025**: 100+ major services, Credential Exchange Protocol draft
- **2025-2026**: Linux native UI (GNOME/KDE), IoT integration
- **Long-term**: Passwords obsolete, passkeys as digital ID, AI agent authentication

---

## 🔒 ADVANCED QUESTIONS (Azure AD B2C, Spring Boot, Implementation)

### **Azure AD B2C Integration**

**Q: Can I use passkeys with Azure AD B2C?**
Yes. Azure AD B2C supports passkeys through custom policies or the built-in passkey authentication method. You can configure passkeys as a passwordless sign-in method alongside or instead of passwords.

**Q: How do passkeys integrate with Azure AD B2C user accounts?**
Passkeys are stored as credentials in Azure AD B2C. During sign-in, B2C validates the passkey assertion and issues tokens (JWT/SAML) just like any other authentication method. You can map passkey output claims to user attributes.

**Q: What about server-verified ceremony in Azure AD B2C?**
Azure AD B2C can perform server-side validation of passkey registration and authentication ceremonies, ensuring the relying party (RP) validates origin, challenge, and signature before trusting the credential.

**Q: Can I use passkeys with .NET 9 backend and Azure AD B2C?**
Yes. Azure AD B2C with .NET 9 supports passkey authentication through MSAL.NET libraries. You can build a server-verified ceremony where your backend validates WebAuthn assertions before accepting them.

---

### **Spring Boot Implementation**

**Q: How do I implement passkeys in Spring Boot?**
Use the `webauthn` library or Spring Security's WebAuthn support:
```java
// Dependencies
implementation 'org.springframework.security:spring-security-web'
implementation 'com.webauthn:webauthn-server:1.4.0'
```

**Q: What is the open-passkey library for Spring Boot?**
`open-passkey` is a Spring Boot starter that simplifies passkey integration. It handles registration, authentication, and JWT token handoff. Supports Spring Boot 3.x with JWT token issuance after passkey sign-in.

**Q: How do I handle JWT token handoff after passkey authentication in Spring Boot?**
After successful passkey authentication:
1. Verify WebAuthn assertion
2. Load/create user in your database
3. Generate JWT token with user claims
4. Return token to client
5. Client uses token for subsequent API calls

**Q: Can I override the login-finish flow in open-passkey?**
Yes. Spring Boot 3.x with open-passkey allows you to customize the post-authentication flow. You can issue your own authorization tokens, set session attributes, or redirect to custom pages after successful passkey sign-in.

---

### **General Implementation Questions**

**Q: What libraries should I use to implement passkeys?**
- **Node.js**: `@simplewebauthn/server` + `@simplewebauthn/browser`
- **Python**: `webauthn` package
- **Go**: `github.com/go-webauthn/webauthn`
- **Java/Spring**: `webauthn-server` or `open-passkey`
- **.NET**: `Fido2NetLib` or `Microsoft.AspNetCore.WebUtilities`

**Q: What algorithms do passkeys use?**
- **ES256** (ECDSA P-256): Most widely supported
- **RS256** (RSA 2048+): Legacy support
- **Ed25519**: Modern, faster, smaller keys (growing adoption)

**Q: What is a minimal passkey registration flow?**
1. Server generates registration options (challenge, RP ID, user info)
2. Browser calls `navigator.credentials.create(options)`
3. User verifies with biometric/PIN
4. Authenticator returns credential ID + public key
5. Server verifies and stores public key linked to user

**Q: What is a minimal passkey authentication flow?**
1. Server generates authentication options (challenge, RP ID)
2. Browser calls `navigator.credentials.get(options)`
3. Authenticator finds passkey by domain
4. User verifies with biometric/PIN
5. Authenticator signs challenge with private key
6. Server verifies signature with stored public key

**Q: How do I test passkeys locally during development?**
Use `localhost` as RP ID. Most browsers support passkeys on localhost for testing. Use Chrome DevTools → Application → Passkeys to inspect stored credentials.

**Q: What is resident key vs non-resident key?**
- **Resident key**: Stored on authenticator (discoverable credential, no username needed)
- **Non-resident**: Stored on server, requires username + credential ID
Passkeys are typically resident keys for better UX.

---

## 🆘 TROUBLESHOOTING QUESTIONS

**Q: Passkey not working in Chrome on Ubuntu?**
Ensure `libsecret` is installed: `sudo apt install libsecret-1-0 libsecret-tools gnome-keyring`. Start GNOME keyring: `/usr/bin/gnome-keyring-daemon --start --components=secrets`. Check `chrome://flags` ensures Web Authentication API is enabled.

**Q: YubiKey not detected?**
Install udev rules: `sudo apt install libu2f-udev`. Verify with `fido2-token -L`. Check USB connection and try different port.

**Q: SSH FIDO2 key not working?**
Check OpenSSH version (needs 8.2+): `ssh -V`. Verify server's `sshd_config` allows `sk-ed25519@openssh.com`. Use verbose mode: `ssh -vvv -i ~/.ssh/id_ed25519_sk user@server`.

**Q: Browser says "Passkey not available"?**
Ensure you're on HTTPS (or localhost). Check that the RP ID matches the domain. Try creating a new passkey instead of using existing one.

---

## 📦 DELIVERY FORMAT

### **For Presenter Reference Card (1 Page)**
Print the "Quick Reference Cheat Sheet" section at the top. Keep it visible during presentation for quick answers.

### **For Detailed Preparation**
Review the "Comprehensive Q&A by Topic" section. Focus on High Priority questions first, then Medium Priority.

### **For Follow-Up Email**
Share the entire document with attendees. Highlight the "Elevator Pitch" section for quick consumption.

### **For Demo Preparation**
Have these commands ready:
```bash
# Ubuntu FIDO2 setup
sudo apt install libfido2-1 fido2-tools libpam-u2f
fido2-token -L

# SSH FIDO2 key generation
ssh-keygen -t ed25519-sk -f ~/.ssh/id_ed25519_sk

# sudo with YubiKey
pamu2fcfg -o pam://hostname > ~/.config/Yubico/u2f_keys
```

---

## 🎯 HANDLING DIFFICULT QUESTIONS

**If you don't know the answer:**
> "That's an excellent question that I don't have the exact answer to. Let me connect you with our security team / I'll follow up with you after this session with detailed information."

**If it's outside scope:**
> "That's beyond what I'm covering today, but I'd be happy to discuss it offline. The short answer is [brief response]."

**If it's controversial:**
> "That's a valid concern. The industry is actively working on [solution]. For now, [current best practice]."

**If it's too technical:**
> "Great technical question! The high-level answer is [simple version]. For the deep dive, I recommend checking out [resource]."

---

## 📊 SUCCESS METRICS TO MENTION

- **Amazon**: 6x faster sign-in times
- **Google**: 4x improvement in sign-in success rates
- **CVS Health**: 98% reduction in mobile account takeover fraud
- **Air New Zealand**: 50% reduction in login abandonment rates
- **General**: 3-4 second login vs 20+ seconds for passwords

---

## 🔗 RESOURCES TO SHARE

- **FIDO Alliance**: fidoalliance.org
- **Passkeys.com**: passkeys.com
- **SimpleWebAuthn**: simplewebauthn.com
- **Ubuntu FIDO2 Docs**: Official Ubuntu 24.04 package documentation
- **YubiKey Guide**: yubico.com/support

---

*Document prepared for Passkey Authentication POC Demo presentation. Based on comprehensive FIDO & Passkeys tutorial covering Ubuntu 24.04 implementation, WebAuthn, CTAP2, and enterprise deployment patterns.*