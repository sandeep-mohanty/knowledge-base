# Passkeys with Azure AD B2C and Passwordless.dev (SPA + Backend)

This tutorial explains how to implement **passkeys (FIDO2/WebAuthn)** in an **Azure AD B2C** environment when B2C itself does not natively store passkeys. We integrate with **Passwordless.dev** using a **SPA frontend** and a **C# backend**.

---

## 1. High-Level Architecture

1. **Azure AD B2C**

   * Manages users and federation.
   * Provides custom policies.
   * Integrates with your backend via **RESTful Technical Profiles (TPs)**.

2. **SPA (Frontend)**

   * React / Angular / Vue app.
   * Handles the browser-side WebAuthn calls (`navigator.credentials.create()` and `navigator.credentials.get()`).
   * Calls backend API endpoints for registration and authentication.

3. **Backend (C# .NET 9)**

   * Talks securely to **Passwordless.dev API** using your API key (server-side only).
   * Issues short-lived WebAuthn challenges to SPA.
   * Verifies responses with Passwordless.dev.
   * Notifies B2C via RESTful TP after successful authentication.

---

## 2. Registration Flow

### Sequence

1. User clicks **"Register Passkey"** in SPA.
2. SPA calls backend `/webauthn/register/start`.
3. Backend calls Passwordless.dev `/register/start` with API key.
4. Backend returns challenge to SPA.
5. SPA runs `navigator.credentials.create({ publicKey: challenge })`.
6. SPA sends attestation result to backend `/webauthn/register/finish`.
7. Backend calls Passwordless.dev `/register/finish`.
8. Backend stores mapping (`userId ↔ credentialId`).

### SPA JavaScript Example

```javascript
// Start registration
async function registerPasskey(userId) {
  const startResp = await fetch("/webauthn/register/start", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ userId })
  });
  const options = await startResp.json();

  // Create credential in browser
  const cred = await navigator.credentials.create({ publicKey: options });

  // Send to backend for finish
  await fetch("/webauthn/register/finish", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(cred)
  });
}
```

### Backend C# Example (Registration)

```csharp
[ApiController]
[Route("webauthn/register")]
public class RegistrationController : ControllerBase
{
    private readonly HttpClient _http;
    private readonly string _apiKey = Environment.GetEnvironmentVariable("PASSWORDLESS_API_KEY");

    public RegistrationController(HttpClient http)
    {
        _http = http;
        _http.DefaultRequestHeaders.Add("ApiSecret", _apiKey);
    }

    [HttpPost("start")]
    public async Task<IActionResult> Start([FromBody] dynamic body)
    {
        string userId = body.userId;
        var response = await _http.PostAsJsonAsync("https://v4.passwordless.dev/register/start", new
        {
            userId = userId,
            username = userId
        });
        var challenge = await response.Content.ReadFromJsonAsync<object>();
        return Ok(challenge);
    }

    [HttpPost("finish")]
    public async Task<IActionResult> Finish([FromBody] object credential)
    {
        var response = await _http.PostAsJsonAsync("https://v4.passwordless.dev/register/finish", credential);
        var result = await response.Content.ReadAsStringAsync();
        return Ok(result);
    }
}
```

---

## 3. Authentication Flow

### Sequence

1. User clicks **"Sign in with Passkey"** in SPA.
2. SPA calls backend `/webauthn/auth/start`.
3. Backend calls Passwordless.dev `/signin/start`.
4. Backend returns challenge to SPA.
5. SPA runs `navigator.credentials.get({ publicKey: challenge })`.
6. SPA sends assertion result to backend `/webauthn/auth/finish`.
7. Backend calls Passwordless.dev `/signin/finish`.
8. Backend validates and issues session/JWT.
9. Optionally: backend notifies B2C via RESTful TP to complete federation.

### SPA JavaScript Example

```javascript
// Start authentication
async function signInPasskey(usernameOrEmail) {
  const startResp = await fetch("/webauthn/auth/start", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ usernameOrEmail })
  });
  const options = await startResp.json();

  // Request credential
  const assertion = await navigator.credentials.get({ publicKey: options });

  // Send to backend for verification
  const finishResp = await fetch("/webauthn/auth/finish", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(assertion)
  });

  if (finishResp.ok) {
    console.log("Passkey login successful!");
  }
}
```

### Backend C# Example (Authentication)

```csharp
[ApiController]
[Route("webauthn/auth")]
public class AuthenticationController : ControllerBase
{
    private readonly HttpClient _http;
    private readonly string _apiKey = Environment.GetEnvironmentVariable("PASSWORDLESS_API_KEY");

    public AuthenticationController(HttpClient http)
    {
        _http = http;
        _http.DefaultRequestHeaders.Add("ApiSecret", _apiKey);
    }

    [HttpPost("start")]
    public async Task<IActionResult> Start([FromBody] dynamic body)
    {
        string usernameOrEmail = body.usernameOrEmail;
        var response = await _http.PostAsJsonAsync("https://v4.passwordless.dev/signin/start", new
        {
            username = usernameOrEmail
        });
        var challenge = await response.Content.ReadFromJsonAsync<object>();
        return Ok(challenge);
    }

    [HttpPost("finish")]
    public async Task<IActionResult> Finish([FromBody] object assertion)
    {
        var response = await _http.PostAsJsonAsync("https://v4.passwordless.dev/signin/finish", assertion);
        var result = await response.Content.ReadAsStringAsync();

        // Issue your own JWT or set B2C federation result here
        return Ok(result);
    }
}
```

---

## 4. Integrating with Azure AD B2C

* B2C cannot store passkeys, but you can use a **custom policy** to:

  * Call your backend via **RESTful Technical Profile** after successful authentication.
  * Receive claims (userId, email, etc.).
  * Continue the user journey with those claims.

### Example RESTful TP

```xml
<TechnicalProfile Id="REST-VerifyPasskey">
  <DisplayName>Verify Passkey</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider.RestfulProvider, Web.TPEngine">
    <Metadata>
      <Item Key="ServiceUrl">https://your-backend.com/webauthn/auth/finish</Item>
      <Item Key="AuthenticationType">None</Item>
      <Item Key="SendClaimsIn">Body</Item>
    </Metadata>
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="assertion" PartnerClaimType="assertion" />
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="userId" />
      <OutputClaim ClaimTypeReferenceId="email" />
    </OutputClaims>
  </Protocol>
</TechnicalProfile>
```

---

## 5. Security Notes

* **API key stays server-side** in backend only.
* SPA communicates only with backend.
* Use **HTTPS** everywhere.
* Map passkey credentials to B2C userId for consistency.
* Handle fallback for users without passkeys (e.g., email OTP).

---

## 6. Summary

* SPA handles WebAuthn UI (`navigator.credentials.*`).
* Backend (C#) talks to Passwordless.dev with API key.
* B2C integrates with backend via REST TP.
* Users can register and authenticate using passkeys while remaining under B2C identity management.

---

✅ With this design:

* Your API key is **never exposed to the frontend**.
* Users get **seamless passkey registration & login**.
* Azure AD B2C remains the central identity provider.
