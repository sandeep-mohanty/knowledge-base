# Understanding Passkeys, Browsers, OS, and Domains  
*A conceptual guide to where passkeys live and how they relate to Passwordless.dev applications.*

---

## 1. Where Do Passkeys Live?

When you create a passkey, three main players are involved:
- **Browser** (Chrome, Edge, Firefox, Safari): provides a WebAuthn interface.  
- **OS Authenticator** (Windows Hello, macOS Touch ID, Android, iOS): manages the actual keys and prompts.  
- **Cloud Sync Services** (iCloud Keychain, Google Password Manager, Microsoft Account sync): optionally sync the keys across devices.

So the general truth is:

- **Not browser‑specific**:  
  Chrome, Edge, and Firefox on the *same machine* all rely on the OS authenticator.  
  Register a passkey in Chrome → it’s available to Edge/Firefox, because they all talk to Windows Hello.

- **Mostly OS‑specific**:  
  Passkey’s private key is stored locally in OS secure hardware (TPM, Secure Enclave).  
  That makes it “device‑bound” unless key sync is enabled (like iCloud across iPhone/Mac).

- **Sync might make it cross‑device**:  
  If you use iCloud Keychain or Google Password Manager, the passkey travels with you.  
  → Register once on your phone, it may also work on your laptop.

✅ **Rule of thumb**: Passkeys follow the **authenticator** (OS / tied account), not the browser itself.

---

## 2. Passkeys Are Always Domain‑Bound

Every passkey is registered for exactly one **Relying Party ID (rpId)**:  
- That’s essentially the domain (`example.com`).  
- It’s baked into the WebAuthn ceremony.  
- The OS authenticator enforces: *“I will not let a credential for app1.com be used on app2.com.”*

This means:
- A passkey created on `myapp.com` **cannot** be reused on `anotherapp.com`.  
- Even if both apps use the same backend or same passwordless.dev account, authenticator will block mismatch.

That’s why passkeys are resistant to phishing:  
- A fake site at `g00gle.com` cannot use a credential meant for `google.com`.

---

## 3. Relation to Passwordless.dev Applications

In Passwordless.dev account management:
- Each **Application** has a **public key** (frontend) and **secret key** (backend).  
- Each Application also defines its **Allowed Origins (Authorized Domains)**.

### How this ties to domains:
- If you add both `https://app.com` and `http://localhost:4200` as origins, both can use the same Application keys.  
- But even then, when registering a passkey, WebAuthn uses the actual origin (`rpId = app.com` or `localhost`).  
- So a passkey created at `localhost` is tied to that rpId. If you switch to production at `app.com`, you’ll be asked to register again.

### Recommended setup:
- **One Application per product/domain** in passwordless.dev.  
- Add multiple origins for that product (localhost, staging, production).  
- Don’t reuse the same API app keys for totally different domains (`app1.com` vs `app2.com`).

---

## 4. Visual Model

```
                +-----------------+
                | Passwordless.dev|
                |   Application A |
                |  - Public Key   |
                |  - Secret Key   |
                |  - Allowed      |
                |    Origins:     |
                |    • app.com    |
                |    • staging... |
                |    • localhost  |
                +-----------------+
                          |
                          v
                  WebAuthn / OS Authenticator
                          |
                          v
                  Passkey stored at OS level
                  • Bound to rpId (domain)
                  • Not bound to browser
                  • Tied to secure enclave or TPM
```

---

## 5. Key Takeaways

1. **Browser independence**: Passkeys are tied to OS authenticators — works in Chrome, Firefox, Edge on same machine.  
2. **OS/account dependence**: Without sync, a passkey is device‑local. With sync (iCloud, Google), it travels across devices (but still domain‑specific).  
3. **Domain binding**: You cannot share passkeys across different domains, even if both apps use Passwordless.dev. rpId check forbids it.  
4. **Passwordless.dev Applications**:  
   - Think of them as containers for your API keys + allowed origins.  
   - Use 1 Application per real product/domain.  
   - Add localhost/staging/prod as origins under the same Application.

---

## 6. Why This Matters

- Stops phishing: “same API key, different domain” won’t trick the authenticator.  
- Clarifies testing: registering at `localhost:4200` doesn’t automatically work for `myapp.com` → you need real production registration.  
- Ensures scaling: multiple products? Give each its own passwordless.dev Application & API keys.

---

# 🎉 Conclusion

Passkeys are **authenticator‑centric** (OS level) and **domain‑bound** (rpId locked).  
Browsers are just messengers.  
Passwordless.dev Applications are your way of mapping **API keys → allowed domains**, which makes integrating multiple environments painless.

This mental model prevents confusion between **browser** vs **OS** vs **domain** vs **application keys** — and helps you design clean, secure setups.