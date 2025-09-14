# Tutorial: Integrating Passkeys into Azure AD B2C Custom Policies

This guide shows how to integrate **passkeys (via Passwordless.dev)** into **Azure AD B2C custom policies** for login and registration, alongside password login.  

## 1. Architecture Overview

```
+-----------------+
| Azure AD B2C    |
| Custom Policy   |
| Custom HTML     | <-- User enters email, chooses password OR passkey
+-----------------+
          |
   +-------------------+
   | Your Backend      |  (Node.js)
   | /api/register     |  handles registering passkeys
   | /api/signin       |  handles login challenges
   | /api/lookup       |  check registered passkeys
   +-------------------+
          |
   +-------------------+
   | Passwordless.dev  |
   | (Passkey Service) |
   +-------------------+
          |
   +-------------------+
   | Authenticator     |
   | (Windows Hello,   |
   | Touch ID, PIN)    |
   +-------------------+
```

Key Points:
- Passkeys are **bound to the B2C login domain** (e.g. `login.company.com`), not your app domain.
- Custom HTML page runs on the B2C login domain, so it can register and use passkeys properly.
- Backend mediates calls to Passwordless.dev with the secret API key.

---

## 2. Passwordless.dev Setup

- Create a new **application** in passwordless.dev.
- Allowed origins:  
  - `https://{yourtenant}.b2clogin.com` OR your B2C custom domain (`https://login.company.com`)  
  - `http://localhost:3000` (for local backend testing)
- Note the **Public API Key** (used in B2C HTML page) and **Secret API Key** (used in backend).

---

## 3. Node.js Backend

### Install prerequisites
```bash
mkdir b2c-passkey-backend
cd b2c-passkey-backend
npm init -y
npm install express axios cors
```

### `server.js`

```js
const express = require("express");
const axios = require("axios");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());

const PASSWORDLESS_BASE = "https://v4.passwordless.dev";
const SECRET_API_KEY = "YOUR_SECRET_API_KEY";

// Issue a register token
app.post("/api/register-token", async (req, res) => {
  const { email } = req.body;
  try {
    const response = await axios.post(
      `${PASSWORDLESS_BASE}/register/token`,
      { userId: email, username: email },
      { headers: { ApiSecret: SECRET_API_KEY } }
    );
    res.json({ token: response.data.token });
  } catch (err) {
    console.error(err.response?.data || err);
    res.status(500).json({ error: "Failed to get registration token" });
  }
});

// Issue a signin token
app.post("/api/signin-token", async (req, res) => {
  const { email } = req.body;
  try {
    const response = await axios.post(
      `${PASSWORDLESS_BASE}/signin/token`,
      { userId: email },
      { headers: { ApiSecret: SECRET_API_KEY } }
    );
    res.json({ token: response.data.token });
  } catch (err) {
    console.error(err.response?.data || err);
    res.status(500).json({ error: "Failed to get signin token" });
  }
});

// Check if passkeys exist
app.post("/api/has-passkeys", async (req, res) => {
  const { email } = req.body;
  try {
    const response = await axios.get(
      `${PASSWORDLESS_BASE}/credentials?userId=${encodeURIComponent(email)}`,
      { headers: { ApiSecret: SECRET_API_KEY } }
    );
    res.json({ hasPasskeys: response.data.length > 0 });
  } catch (err) {
    console.error(err.response?.data || err);
    res.status(500).json({ error: "Failed to check passkeys" });
  }
});

app.listen(3000, () => console.log("Backend running on http://localhost:3000"));
```

---

## 4. Azure AD B2C Policy Updates

### Add Passkey ClaimsProvider
In **TrustFrameworkExtensions.xml**:

```xml
<ClaimsProvider>
  <DisplayName>Passkeys</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Passkey-Login">
      <DisplayName>Passkey Authentication</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider">
        <Metadata>
          <Item Key="ServiceUrl">https://yourbackend.com/api/b2c-passkey-login</Item>
          <Item Key="AuthenticationType">None</Item>
          <Item Key="SendClaimsIn">Body</Item>
        </Metadata>
        <InputClaims>
          <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
        </InputClaims>
        <OutputClaims>
          <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
        </OutputClaims>
      </TechnicalProfile>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

### Orchestration Step
UserJourney step 1 will allow both local account and passkey:

```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.selfasserted">
  <ClaimsProviderSelections>
    <ClaimsProviderSelection TargetClaimsExchangeId="LocalAccountSignin"/>
    <ClaimsProviderSelection TargetClaimsExchangeId="PasskeyLoginExchange"/>
  </ClaimsProviderSelections>
  <ClaimsExchanges>
    <ClaimsExchange Id="LocalAccountSignin" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
    <ClaimsExchange Id="PasskeyLoginExchange" TechnicalProfileReferenceId="Passkey-Login">
      <Preconditions>
        <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
          <Value>authMethod</Value>
          <Value>passkey</Value>
          <Action>Continue</Action>
        </Precondition>
        <Precondition Type="ClaimEquals" ExecuteActionsIf="false">
          <Value>authMethod</Value>
          <Value>passkey</Value>
          <Action>SkipThisValidationTechnicalProfile</Action>
        </Precondition>
      </Preconditions>
    </ClaimsExchange>
  </ClaimsExchanges>
</OrchestrationStep>
```

---

## 5. Custom B2C Login Page HTML + JS

### Include Passwordless.dev JS
```html
<script src="https://cdn.jsdelivr.net/npm/@passwordlessdev/passwordless-client@latest/dist/passwordless.min.js"></script>
<script>
  const client = new Passwordless.Client({ apiKey: "YOUR_PUBLIC_API_KEY" });

  async function enableButtons() {
    const email = document.getElementById("email").value;
    const password = document.getElementById("password").value;

    document.getElementById("continueBtn").disabled = !password;
    document.getElementById("passkeyBtn").disabled = !email;

    if (email) {
      // Ask backend if passkeys exist
      const resp = await fetch("https://yourbackend.com/api/has-passkeys", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email })
      });
      const { hasPasskeys } = await resp.json();
      if (!hasPasskeys) {
        document.getElementById("passkeyError").innerText =
          "No passkeys registered. Please register a passkey first.";
        document.getElementById("passkeyBtn").disabled = true;
      } else {
        document.getElementById("passkeyError").innerText = "";
      }
    }
  }

  async function loginWithPassword() {
    document.getElementById("authMethod").value = "password";
    document.forms[0].submit();
  }

  async function loginWithPasskey() {
    const email = document.getElementById("email").value;
    document.getElementById("authMethod").value = "passkey";

    // Request signin token
    const tokenResp = await fetch("https://yourbackend.com/api/signin-token", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email })
    });
    const { token } = await tokenResp.json();

    try {
      await client.signin({ token });
      document.forms[0].submit();
    } catch (e) {
      document.getElementById("passkeyError").innerText = "Failed login with passkey.";
    }
  }

  async function registerPasskey() {
    const email = document.getElementById("email").value;
    const tokenResp = await fetch("https://yourbackend.com/api/register-token", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email })
    });
    const { token } = await tokenResp.json();

    try {
      await client.register(token);
      alert("Passkey registered successfully! You can now log in with it.");
    } catch (e) {
      alert("Registration failed.");
    }
  }
</script>
```

### HTML Markup
```html
<input id="email" type="email" name="email" placeholder="Email" oninput="enableButtons()" />
<input id="password" type="password" name="password" placeholder="Password" oninput="enableButtons()" />
<input type="hidden" id="authMethod" name="authMethod" value="password" />

<button id="continueBtn" onclick="loginWithPassword()">Continue</button>
<button id="passkeyBtn" onclick="loginWithPasskey()">Sign in with Passkey</button>
<button onclick="registerPasskey()">Register a Passkey</button>

<div id="passkeyError" style="color:red;"></div>
```

---

## 6. UX Behavior

- As soon as an email is entered:  
  → Page checks if passkeys registered for that user via `/api/has-passkeys`.  
  → If yes: enable “Sign in with Passkey”.  
  → If no: disable “Sign in with Passkey” and show error.  

- If user clicks **Register Passkey**:  
  → Starts WebAuthn ceremony.  
  → Registers credential with Passwordless.dev.  
  → On success → inform user to retry login.  

- If user clicks **Continue** with password:  
  → `authMethod=password` set → B2C executes local account flow.  

- If user clicks **Sign in with Passkey**:  
  → `authMethod=passkey` set → B2C executes Passkey REST profile, which maps to user account.  

---

# ✅ Conclusion

- Passkeys and password logins are combined on **one B2C login page**.  
- Registration is possible from the same page (only once).  
- Passkey button only enabled if user already registered; otherwise shows helpful error.  
- Orchestration step ensures the correct technical profile runs, based on hidden `authMethod` claim.  

This gives a **seamless dual login** experience: Username+Password or Passkeys, directly on Azure AD B2C’s login domain.