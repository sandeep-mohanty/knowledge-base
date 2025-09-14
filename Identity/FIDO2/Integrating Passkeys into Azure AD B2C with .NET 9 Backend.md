# Integrating Passkeys into Azure AD B2C with .NET 9 Backend

This guide shows how to add **passkey‑based registration and login** into your Azure AD B2C custom policies, using:

- **Vanilla JavaScript** on the B2C login page  
- **Passwordless.dev** as the passkey provider  
- **.NET 9 Minimal API backend** to handle secure Passwordless.dev API calls with the secret key  

---

## 1. Architecture Overview

```
+--------------------------------------------------+
|           Azure AD B2C Login Page                |
|  - Domain: https://tenant.b2clogin.com           |
|  - Custom HTML + Vanilla JS                      |
|  - Uses Passwordless.dev PUBLIC API key          |
|  - Shows: Email, Password, Continue,             |
|           Sign in with Passkey, Register buttons |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|              Your Backend (.NET 9)               |
|  - Minimal API                                   |
|  - Endpoints:                                    |
|      /api/register-token                         |
|      /api/signin-token                           |
|      /api/has-passkeys                           |
|      (optional /api/unregister,                  |
|       /api/b2c-passkey-login for REST TP)        |
|  - Uses Passwordless.dev SECRET API key          |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|                Passwordless.dev                  |
|  - Issues one-time tokens for registration/login |
|  - Manages credentials (stores public keys)      |
|  - Validates authenticator signatures            |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|              OS Authenticator (via browser)      |
|  - Windows Hello (PIN/Face/Fingerprint)          |
|  - macOS/iOS Touch ID / Face ID                  |
|  - YubiKeys, Android passkeys, etc.              |
|  - Holds private key (never leaves device)       |
+--------------------------------------------------+
```

---

## 2. Passwordless.dev Setup

1. Create an account at [passwordless.dev](https://passwordless.dev).
2. Register a new Application.  
   - Add **allowed origins**:  
     - Your B2C login domain (`https://tenant.b2clogin.com` or `https://login.company.com`)  
     - localhost for testing, if desired
3. Copy down:  
   - **Public API key** → will be placed in B2C login page JavaScript  
   - **Secret API key** → used only in your .NET backend  

---

## 3. .NET 9 Backend

We’ll build a minimal API backend that creates **register/signin tokens**, checks **if passkeys exist**, and optionally allows cleanup.

### Project Setup
```bash
dotnet new web -n PasskeyBackend
cd PasskeyBackend
```

### `Program.cs`
```csharp
using System.Net.Http.Headers;
using System.Text;
using System.Text.Json;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseCors(policy => 
    policy.AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader());

const string PASSWORDLESS_BASE = "https://v4.passwordless.dev";
const string SECRET_API_KEY = "YOUR_SECRET_API_KEY";

var httpClient = new HttpClient();

async Task<IResult> PostPasswordless(string url, object payload)
{
    var json = JsonSerializer.Serialize(payload);
    var request = new HttpRequestMessage(HttpMethod.Post, url)
    {
        Content = new StringContent(json, Encoding.UTF8, "application/json")
    };
    request.Headers.Add("ApiSecret", SECRET_API_KEY);

    var resp = await httpClient.SendAsync(request);
    var respStr = await resp.Content.ReadAsStringAsync();
    if (!resp.IsSuccessStatusCode)
    {
        return Results.Problem(respStr, statusCode: (int)resp.StatusCode);
    }
    return Results.Content(respStr, "application/json");
}

// 1. Registration token endpoint
app.MapPost("/api/register-token", async (HttpContext ctx) =>
{
    var body = await JsonSerializer.DeserializeAsync<Dictionary<string,string>>(ctx.Request.Body);
    var email = body?["email"] ?? "";
    return await PostPasswordless($"{PASSWORDLESS_BASE}/register/token",
        new { userId = email, username = email });
});

// 2. Signin token endpoint
app.MapPost("/api/signin-token", async (HttpContext ctx) =>
{
    var body = await JsonSerializer.DeserializeAsync<Dictionary<string,string>>(ctx.Request.Body);
    var email = body?["email"] ?? "";
    return await PostPasswordless($"{PASSWORDLESS_BASE}/signin/token",
        new { userId = email });
});

// 3. Lookup credentials
app.MapPost("/api/has-passkeys", async (HttpContext ctx) =>
{
    var body = await JsonSerializer.DeserializeAsync<Dictionary<string,string>>(ctx.Request.Body);
    var email = body?["email"] ?? "";

    var request = new HttpRequestMessage(HttpMethod.Get,
        $"{PASSWORDLESS_BASE}/credentials?userId={Uri.EscapeDataString(email)}");
    request.Headers.Add("ApiSecret", SECRET_API_KEY);

    var resp = await httpClient.SendAsync(request);
    var respStr = await resp.Content.ReadAsStringAsync();

    if (!resp.IsSuccessStatusCode)
    {
        return Results.Problem(respStr, statusCode: (int)resp.StatusCode);
    }

    var credentials = JsonSerializer.Deserialize<List<object>>(respStr);
    bool hasPasskeys = credentials?.Count > 0;
    return Results.Json(new { hasPasskeys });
});

app.Run();
```

### Running the backend
```bash
dotnet run
```

Backend runs on `https://localhost:5001`.

---

## 4. Azure AD B2C Custom Policy Updates

### Add ClaimsProvider
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

### Update Orchestration Step 1
```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.selfasserted">
  <ClaimsProviderSelections>
    <ClaimsProviderSelection TargetClaimsExchangeId="LocalAccountSignin"/>
    <ClaimsProviderSelection TargetClaimsExchangeId="PasskeyLoginExchange"/>
  </ClaimsProviderSelections>

  <ClaimsExchanges>
    <ClaimsExchange Id="LocalAccountSignin" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email"/>
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

## 5. Custom B2C Login Page (HTML + JS)

### Include SDK
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
      const resp = await fetch("https://yourbackend.com/api/has-passkeys", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email })
      });
      const { hasPasskeys } = await resp.json();
      if (!hasPasskeys) {
        document.getElementById("passkeyError").innerText = 
          "No passkeys registered for this account.";
        document.getElementById("passkeyBtn").disabled = true;
      } else {
        document.getElementById("passkeyError").innerText = "";
      }
    }
  }

  function loginWithPassword() {
    document.getElementById("authMethod").value = "password";
    document.forms[0].submit();
  }

  async function loginWithPasskey() {
    const email = document.getElementById("email").value;
    document.getElementById("authMethod").value = "passkey";

    const tokenResp = await fetch("https://yourbackend.com/api/signin-token", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email })
    });
    const { token } = await tokenResp.json();

    try {
      await client.signin({ token });
      document.forms[0].submit();
    } catch {
      document.getElementById("passkeyError").innerText = "Failed passkey login.";
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
    } catch {
      alert("Registration failed.");
    }
  }
</script>
```

### HTML Controls
```html
<input id="email" type="email" name="email" oninput="enableButtons()" />
<input id="password" type="password" name="password" oninput="enableButtons()" />
<input type="hidden" id="authMethod" name="authMethod" value="password" />

<button id="continueBtn" onclick="loginWithPassword()">Continue</button>
<button id="passkeyBtn" onclick="loginWithPasskey()">Sign in with Passkey</button>
<button onclick="registerPasskey()">Register Passkey</button>

<div id="passkeyError" style="color:red"></div>
```

---

## 6. Flow Summary

- **Registration:**  
  User enters email → clicks *Register Passkey* → Backend issues token → Passkey registered at Passwordless.dev bound to B2C domain.  

- **Login with password:**  
  email+password entered → LocalAccount TP executes → B2C issues tokens.  

- **Login with passkey:**  
  email only → Fetch signin token → `client.signin(token)` runs → Device Authenticator verifies → B2C claims step continues → user logged in via passkey.  

- If **no passkeys registered** → “No passkeys registered” message shown next to button.  

---

# ✅ Conclusion

- Passkeys seamlessly integrated into B2C login flow alongside passwords.  
- Public API key = in page JS, Secret API key = stored only in .NET 9 backend.  
- B2C orchestrates which path executes (`authMethod=password` or `authMethod=passkey`).  
- Users can self‑register passkeys on the login page, and the button reflects whether passkeys already exist.  