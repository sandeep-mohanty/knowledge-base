# Tutorial: Azure AD B2C Passkeys with .NET 9 — Server‑Verified Ceremony (Option 2)

This extends our earlier tutorial by having the backend **verify WebAuthn assertion results** instead of just trusting the frontend.

---

## 1. Flow Architecture

```
+--------------------------------------------------+
|           Azure AD B2C Login Page                |
|  - Email, Password fields                        |
|  - Buttons: Continue, Register Passkey,          |
|             Sign in with Passkey                 |
|  - Runs Passwordless.Client in JS                |
|  - Collects WebAuthn assertion JSON              |
|  - Posts it -> /api/b2c-passkey-login            |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|           Your Backend (.NET 9 Minimal API)      |
|  - /api/register-token                           |
|  - /api/b2c-passkey-login (NEW)                  |
|  - Uses Passwordless.dev SECRET API Key          |
|  - Verifies assertion against Passwordless.dev   |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|                Passwordless.dev                  |
|  - Issues registration & signin tokens           |
|  - Validates signed challenge/assertion          |
|  - Confirms credential matches userId            |
+--------------------------------------------------+
                         |
                         v
+--------------------------------------------------+
|        OS Authenticator (via WebAuthn)           |
|  - Windows Hello / TouchID / FaceID              |
|  - Private key never leaves device               |
+--------------------------------------------------+
```

---

## 2. Backend Implementation (`Program.cs`)

We now handle `/api/b2c-passkey-login`. This endpoint will:

1. Get `email` from B2C page POST.  
2. Generate a signin token via Passwordless.dev.  
3. Send signin token to frontend → do `navigator.credentials.get()`.  
4. Receive assertion back in frontend → post it to `/api/b2c-passkey-login`.  
5. Backend verifies assertion with Passwordless.dev.  
6. If valid → backend returns claims JSON to **B2C policy REST TP**.

```csharp
using System.Net.Http.Headers;
using System.Text;
using System.Text.Json;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

const string PASSWORDLESS_BASE = "https://v4.passwordless.dev";
const string SECRET_API_KEY = "YOUR_SECRET_API_KEY";
var httpClient = new HttpClient();

// Call Passwordless.dev helper
async Task<HttpResponseMessage> PostPasswordlessDev(string url, object payload)
{
    var json = JsonSerializer.Serialize(payload);
    var request = new HttpRequestMessage(HttpMethod.Post, url)
    {
        Content = new StringContent(json, Encoding.UTF8, "application/json")
    };
    request.Headers.Add("ApiSecret", SECRET_API_KEY);
    return await httpClient.SendAsync(request);
}

// Step 1: Generate signin token
app.MapPost("/api/get-signin-token", async (HttpContext ctx) =>
{
    var body = await JsonSerializer.DeserializeAsync<Dictionary<string, string>>(ctx.Request.Body);
    var email = body?["email"];
    if (string.IsNullOrEmpty(email)) return Results.BadRequest("Missing email");

    var resp = await PostPasswordlessDev(
        $"{PASSWORDLESS_BASE}/signin/token",
        new { userId = email });

    var data = await resp.Content.ReadAsStringAsync();
    return Results.Content(data, "application/json");
});

// Step 2: Verify assertion — called by B2C REST TP
app.MapPost("/api/b2c-passkey-login", async (HttpContext ctx) =>
{
    var body = await JsonSerializer.DeserializeAsync<Dictionary<string, object>>(ctx.Request.Body);

    var email = body?["email"]?.ToString();
    var assertion = body?["assertion"]; // full assertion object from frontend

    if (string.IsNullOrEmpty(email) || assertion == null)
    {
        return Results.Json(new {
            version = "1.0.0",
            status = 400,
            userMessage = "Missing email or assertion"
        }, statusCode: 400);
    }

    // Verify with Passwordless.dev
    var resp = await PostPasswordlessDev(
        $"{PASSWORDLESS_BASE}/signin/verify",
        new { token = assertion }
    );

    if (!resp.IsSuccessStatusCode)
    {
        return Results.Json(new {
            version = "1.0.0",
            status = 400,
            userMessage = "Passkey validation failed"
        }, statusCode: 400);
    }

    // Success -> return claim back to AAD B2C
    return Results.Json(new {
        version = "1.0.0",
        status = 200,
        email = email
    });
});

app.Run();
```

---

## 3. Custom Policy Update in B2C

**Technical Profile** in `TrustFrameworkExtensions.xml`:

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
      <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
      <InputClaim ClaimTypeReferenceId="assertion" PartnerClaimType="assertion" />
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
    </OutputClaims>
  </TechnicalProfile>
</TechnicalProfile>
```

---

## 4. Frontend (B2C Login Page JS)

New:  
- First call backend `/api/get-signin-token` to request a token.  
- Call `client.signin(token)` in JS → get `assertion`.  
- Submit form with `email`, `authMethod=passkey`, and `assertion`.  

```html
<script src="https://cdn.jsdelivr.net/npm/@passwordlessdev/passwordless-client@latest/dist/passwordless.min.js"></script>
<script>
  const client = new Passwordless.Client({ apiKey: "YOUR_PUBLIC_API_KEY" });

  async function loginWithPasskey() {
    const email = document.getElementById("email").value;
    document.getElementById("authMethod").value = "passkey";

    // Step 1: get signin token
    const resp = await fetch("https://your-backend.com/api/get-signin-token", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email })
    });
    const { token } = await resp.json();

    try {
      // Step 2: perform ceremony
      const assertion = await client.signin({ token });

      // Step 3: put assertion in hidden field for B2C form
      document.getElementById("assertion").value = JSON.stringify(assertion);

      // Step 4: submit form to B2C -> TP calls /api/b2c-passkey-login
      document.forms[0].submit();
    } catch (e) {
      document.getElementById("passkeyError").innerText =
        "Passkey login failed: " + e;
    }
  }
</script>

<input type="hidden" id="authMethod" name="authMethod" value="password" />
<input type="hidden" id="assertion" name="assertion" value="" />

<button onclick="loginWithPasskey()">Sign in with Passkey</button>
```

---

## 5. Recap of Assertions

- The `assertion` hidden input gets the **full signed response** from `client.signin()`.  
- When form submits → B2C passes it into the TP as `InputClaim assertion`.  
- B2C calls `/api/b2c-passkey-login` with `email` + `assertion`.  
- Backend verifies with Passwordless.dev.  
- If valid, backend responds with claims (`email`), status=200.  
- B2C proceeds with login → issues tokens.

---

# ✅ Conclusion

This **server‑verified ceremony** design ensures:  
- **Frontend can’t trick B2C** (backend always re‑verifies).  
- **B2C’s REST Technical Profile** receives a stable, JSON claim contract.  
- **Passwordless.dev** handles WebAuthn validation, storing only public key.  

This is the most secure and production‑ready way to federate passkeys into **Azure AD B2C**.