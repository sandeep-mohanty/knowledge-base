# Passwordless.dev + Angular + .NET Backend Tutorial  
*A minimal demo app for registering, signing in with, and unregistering passkeys.*

This tutorial shows how to integrate [Passwordless.dev](https://passwordless.dev) into an **Angular frontend** with a **C# .NET backend**.  

---

## 1. Prerequisites

- Angular CLI installed.  
- .NET 7 or 8 SDK.  
- Passwordless.dev account to get API keys.  

---

## 2. Get Your Passwordless.dev API Keys

1. Sign up at [passwordless.dev](https://passwordless.dev).  
2. Create an application in the dashboard.  
3. You’ll get:  
   - **Public API Key** → used in frontend.  
   - **Secret API Key** → used only in backend.  

⚠️ Never expose your **secret key** in the frontend. Backend only.  

---

## 3. Angular Frontend

We’ll reuse the same Angular frontend.  

### Install Passwordless.dev SDK

```bash
npm install @passwordlessdev/passwordless-client
```

### `app.component.html`

```html
<div style="margin: 2rem;">
  <h1>Passwordless.dev Demo</h1>

  <mat-form-field appearance="outline">
    <mat-label>Email</mat-label>
    <input matInput [(ngModel)]="email" />
  </mat-form-field>

  <div style="margin-top: 1rem;">
    <button mat-raised-button color="primary" (click)="registerPasskey()">
      Register Passkey
    </button>

    <button mat-raised-button color="accent" (click)="signInPasskey()">
      Sign In with Passkey
    </button>

    <button mat-raised-button color="warn" (click)="unregisterPasskey()">
      Unregister Existing Passkey
    </button>
  </div>
</div>
```

### `app.component.ts`

```ts
import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import * as Passwordless from '@passwordlessdev/passwordless-client';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
  email: string = '';
  client: Passwordless.Client;

  constructor(private http: HttpClient) {
    this.client = new Passwordless.Client({ apiKey: "YOUR_PUBLIC_API_KEY" });
  }

  async registerPasskey() {
    try {
      const { token } = await this.http.post<{ token: string }>(
        'https://localhost:5001/api/register-token',
        { email: this.email }
      ).toPromise();

      await this.client.register(token!);
      alert('Passkey registered successfully');
    } catch (err) {
      console.error(err);
      alert('Registration failed');
    }
  }

  async signInPasskey() {
    try {
      const { token } = await this.http.post<{ token: string }>(
        'https://localhost:5001/api/signin-token',
        { email: this.email }
      ).toPromise();

      const result = await this.client.signin({ token: token! });
      console.log(result);
      alert('Sign in successful for ' + this.email);
    } catch (err) {
      console.error(err);
      alert('Sign in failed');
    }
  }

  async unregisterPasskey() {
    try {
      await this.http.post(
        'https://localhost:5001/api/unregister-passkeys',
        { email: this.email }
      ).toPromise();

      alert('Removed passkeys for this email');
    } catch (err) {
      console.error(err);
      alert('Unregister failed');
    }
  }
}
```

---

## 4. Backend in .NET (Minimal API)

Let’s build a **.NET 7/8 Minimal API** backend.

### Create Project

```bash
dotnet new webapi -n PasskeyBackend
cd PasskeyBackend
```

Edit `Program.cs`:

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using System.Net.Http.Headers;
using System.Text.Json;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseCors(policy => policy
    .AllowAnyOrigin()
    .AllowAnyHeader()
    .AllowAnyMethod()
);

const string PASSWORDLESS_BASE = "https://v4.passwordless.dev";
string SECRET_API_KEY = "YOUR_SECRET_API_KEY";

// HttpClient shared
HttpClient httpClient = new HttpClient();

// 1. Registration Token
app.MapPost("/api/register-token", async (HttpContext context) =>
{
    var body = await JsonSerializer.DeserializeAsync<Dictionary<string,string>>(context.Request.Body);
    var email = body?["email"] ?? "";

    var payload = new Dictionary<string,string>
    {
        ["userId"] = email,
        ["username"] = email
    };

    var request = new HttpRequestMessage(HttpMethod.Post, $"{PASSWORDLESS_BASE}/register/token")
    {
        Content = new StringContent(JsonSerializer.Serialize(payload))
        {
            Headers = { ContentType = new MediaTypeHeaderValue("application/json") }
        }
    };
    request.Headers.Add("ApiSecret", SECRET_API_KEY);

    var response = await httpClient.SendAsync(request);
    var result = await response.Content.ReadAsStringAsync();

    return Results.Content(result, "application/json");
});

// 2. Signin Token
app.MapPost("/api/signin-token", async (HttpContext context) =>
{
    var body = await JsonSerializer.DeserializeAsync<Dictionary<string,string>>(context.Request.Body);
    var email = body?["email"] ?? "";

    var payload = new Dictionary<string,string>
    {
        ["userId"] = email
    };

    var request = new HttpRequestMessage(HttpMethod.Post, $"{PASSWORDLESS_BASE}/signin/token")
    {
        Content = new StringContent(JsonSerializer.Serialize(payload))
        {
            Headers = { ContentType = new MediaTypeHeaderValue("application/json") }
        }
    };
    request.Headers.Add("ApiSecret", SECRET_API_KEY);

    var response = await httpClient.SendAsync(request);
    var result = await response.Content.ReadAsStringAsync();

    return Results.Content(result, "application/json");
});

// 3. Unregister Credentials
app.MapPost("/api/unregister-passkeys", async (HttpContext context) =>
{
    var body = await JsonSerializer.DeserializeAsync<Dictionary<string,string>>(context.Request.Body);
    var email = body?["email"] ?? "";

    // List credentials
    var listReq = new HttpRequestMessage(HttpMethod.Get, $"{PASSWORDLESS_BASE}/credentials?userId={Uri.EscapeDataString(email)}");
    listReq.Headers.Add("ApiSecret", SECRET_API_KEY);

    var listResp = await httpClient.SendAsync(listReq);
    var credentialsStr = await listResp.Content.ReadAsStringAsync();

    var credentials = JsonSerializer.Deserialize<List<Dictionary<string, object>>>(credentialsStr);

    foreach (var cred in credentials ?? new())
    {
        if (cred.TryGetValue("credentialId", out var idObj) && idObj is JsonElement je && je.ValueKind == JsonValueKind.String)
        {
            string credentialId = je.GetString()!;
            var delReq = new HttpRequestMessage(HttpMethod.Delete, $"{PASSWORDLESS_BASE}/credentials/{credentialId}");
            delReq.Headers.Add("ApiSecret", SECRET_API_KEY);
            await httpClient.SendAsync(delReq);
        }
    }

    return Results.Json(new { success = true });
});

app.Run();
```

---

## 5. Run Everything

- Run backend:

```bash
dotnet run --project PasskeyBackend
```

Backend runs at `https://localhost:5001`  

- Run Angular frontend:

```bash
ng serve
```

Frontend runs at `http://localhost:4200`

---

## 6. Test the Flows

1. **Register Passkey**  
   - Enter your email.  
   - Click *Register*.  
   - Windows Hello / Face ID / PIN prompt appears.  
   - Passkey stored on device + passwordless.dev.  

2. **Sign In with Passkey**  
   - Enter the same email.  
   - Click *Sign In with Passkey*.  
   - OS prompt appears.  
   - Authentication succeeds.  

3. **Unregister Passkey**  
   - Click *Unregister Existing Passkey*.  
   - All passkeys tied to that user ID are deleted.  

---

## 7. Key Points

- The **Angular frontend** only ever uses the **public API key**.  
- The **.NET backend** uses the **secret API key** with passwordless.dev endpoints.  
- The OS (Windows Hello, macOS Touch ID, Android, iOS) manages the actual private key.  

---

# 🎉 Conclusion

You now have a working **Angular + .NET backend** prototype using passwordless.dev to:  
- Register passkeys.  
- Sign in users via passkeys.  
- Unregister user credentials.  

This demonstrates how **frontend + OS authenticator + backend + passwordless.dev** work together for a full passwordless solution.