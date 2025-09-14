# Tutorial: Azure AD B2C Passkeys with Node.js Backend — Server‑Verified Ceremony (Option 2)

This tutorial covers the **secure server‑verified passkey flow** with **Node.js Express backend**.  

---

## 1. Architecture Overview

```
+--------------------------------------------------+
|           Azure AD B2C Login Page                |
|  - Runs at https://tenant.b2clogin.com           |
|  - Custom HTML + Vanilla JS + Passwordless SDK   |
|  - User enters email + clicks "Sign in with PK"  |
|  - JS runs ceremony → collects WebAuthn assertion|
|  - Form submits with email + assertion           |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|              Your Backend (Node.js)              |
|   Endpoints:                                     |
|     /api/get-signin-token                        |
|     /api/b2c-passkey-login                       |
|   - Uses Passwordless.dev SECRET API key         |
|   - Verifies assertion using passwordless.dev    |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|                Passwordless.dev                  |
|   - Issues signin token                          |
|   - Verifies signed assertion                    |
|   - Stores public keys, private key never leaves |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|          Authenticator (Windows Hello, etc.)     |
|   - Biometric, PIN, or Security Key              |
|   - Device-local storage of private key          |
+--------------------------------------------------+
```

---

## 2. Node.js Backend Implementation

### Setup
```bash
mkdir b2c-passkey-backend
cd b2c-passkey-backend
npm init -y
npm install express axios cors body-parser
```

### `server.js`

```js
const express = require("express");
const axios = require("axios");
const cors = require("cors");
const bodyParser = require("body-parser");

const app = express();
app.use(cors());
app.use(bodyParser.json());

const PASSWORDLESS_BASE = "https://v4.passwordless.dev";
const SECRET_API_KEY = "YOUR_SECRET_API_KEY";

// 1. Helper function
async function callPasswordless(endpoint, payload) {
  try {
    const resp = await axios.post(`${PASSWORDLESS_BASE}${endpoint}`, payload, {
      headers: { "ApiSecret": SECRET_API_KEY }
    });
    return resp.data;
  } catch (err) {
    console.error("Passwordless API error:", err.response?.data || err.message);
    throw err;
  }
}

// 2. Get signin token (called by B2C login page JS)
app.post("/api/get-signin-token", async (req, res) => {
  const { email } = req.body;
  if (!email) return res.status(400).json({ error: "Missing email" });

  try {
    const result = await callPasswordless("/signin/token", { userId: email });
    res.json(result);
  } catch {
    res.status(500).json({ error: "Failed to get signin token" });
  }
});

// 3. Verify assertion — main B2C REST TP endpoint
app.post("/api/b2c-passkey-login", async (req, res) => {
  const { email, assertion } = req.body;

  if (!email || !assertion) {
    return res.status(400).json({
      version: "1.0.0",
      status: 400,
      userMessage: "Missing email or assertion"
    });
  }

  try {
    // Verify with Passwordless.dev
    const result = await callPasswordless("/signin/verify", { token: assertion });

    if (!result || !result.success) {
      return res.status(400).json({
        version: "1.0.0",
        status: 400,
        userMessage: "Passkey validation failed"
      });
    }

    // On success, return claims for B2C
    res.json({
      version: "1.0.0",
      status: 200,
      email: email
    });
  } catch {
    res.status(400).json({
      version: "1.0.0",
      status: 400,
      userMessage: "Assertion verification failed"
    });
  }
});

app.listen(3000, () =>
  console.log("Passkey backend listening at http://localhost:3000")
);
```

---

## 3. B2C Custom Policy Technical Profile

In **TrustFrameworkExtensions.xml**:

```xml
<TechnicalProfile Id="Passkey-Login">
  <DisplayName>Passkey Authentication</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider">
    <Metadata>
      <Item Key="ServiceUrl">https://your-backend.com/api/b2c-passkey-login</Item>
      <Item Key="AuthenticationType">None</Item>
      <Item Key="SendClaimsIn">Body</Item>
    </Metadata>
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
      <InputClaim ClaimTypeReferenceId="assertion" PartnerClaimType="assertion"/>
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
    </OutputClaims>
  </TechnicalProfile>
</ClaimsProvider>
```

Now, if `authMethod=passkey`, step 1’s `ClaimsExchange` calls this TP. B2C sends `{ email, assertion }` → backend verifies with Passwordless.dev → returns `email`.

---

## 4. B2C Login Page (Vanilla JS)

```html
<script src="https://cdn.jsdelivr.net/npm/@passwordlessdev/passwordless-client@latest/dist/passwordless.min.js"></script>
<script>
  const client = new Passwordless.Client({ apiKey: "YOUR_PUBLIC_API_KEY" });

  async function loginWithPasskey() {
    const email = document.getElementById("email").value;
    document.getElementById("authMethod").value = "passkey";

    // 1. Ask backend for signin token
    const resp = await fetch("https://your-backend.com/api/get-signin-token", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email })
    });
    const { token } = await resp.json();

    try {
      // 2. Perform ceremony (navigator.credentials.get())
      const assertion = await client.signin({ token });

      // 3. Store assertion JSON in a hidden field
      document.getElementById("assertion").value = JSON.stringify(assertion);

      // 4. Submit form (B2C engine will POST to /api/b2c-passkey-login)
      document.forms[0].submit();
    } catch (e) {
      document.getElementById("passkeyError").innerText =
        "Passkey login failed: " + e;
    }
  }
</script>

<input type="hidden" id="authMethod" name="authMethod" value="password">
<input type="hidden" id="assertion" name="assertion" value="">
<button onclick="loginWithPasskey()">Sign in with Passkey</button>
<div id="passkeyError" style="color:red"></div>
```

---

## 5. REST TP Response Format

Your backend should return JSON response in this format (B2C expects it):

- Success
```json
{
  "version": "1.0.0",
  "status": 200,
  "email": "alice@example.com"
}
```

- Failure  
```json
{
  "version": "1.0.0",
  "status": 400,
  "userMessage": "Passkey validation failed"
}
```

---

## 6. Flow Summary

1. User enters email, clicks "Sign in with Passkey".  
2. Frontend JS calls `/api/get-signin-token` (Node.js backend).  
3. JS runs `client.signin({ token })`; browser authenticator prompts.  
4. Assertion JSON returned → placed into hidden field → submitted to B2C form.  
5. B2C TP calls `/api/b2c-passkey-login` with `email` + `assertion`.  
6. Backend posts assertion to Passwordless.dev `/signin/verify`.  
7. If verified → return claim `email` → B2C continues and issues tokens.  

---

# ✅ Conclusion

With this **Node.js server‑verified ceremony**:

- Passwordless.dev challenge + assertion are **only trusted after backend verification**.  
- Azure AD B2C gets verified claims via REST TP.  
- Entire flow remains phishing‑resistant, device‑bound, and standards‑based.  

This matches the .NET implementation, so you can confidently choose either stack.  