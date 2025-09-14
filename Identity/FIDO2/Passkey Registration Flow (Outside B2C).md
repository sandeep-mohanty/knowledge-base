# Passkey Registration Flow (Outside B2C)

This document describes how to implement **passkey registration** using [passwordless.dev](https://www.passwordless.dev/) and Azure AD B2C.  
Registration happens **outside of B2C** in your RP (Relying Party) backend and SPA (frontend).  
B2C is not involved in registration. It is only used later for authentication and issuing tokens.

---

## 🔑 High-Level Flow

1. User signs in to your SPA via a **non-passkey method** (e.g., email+password, social login).  
2. SPA shows **"Register Passkey"** option.  
3. SPA calls your **backend API** to start passkey registration.  
4. Backend requests registration token from `passwordless.dev` and passes it to SPA.  
5. SPA calls `navigator.credentials.create(...)` (WebAuthn) with the token.  
6. SPA sends the WebAuthn credential back to the backend.  
7. Backend finalizes registration with `passwordless.dev` → receives **`sub`**.  
8. Backend updates the user’s B2C profile in Microsoft Graph with this `sub` (stored in an extension attribute).  
9. Next time the user logs in, B2C custom policy uses this stored `sub` to validate passkey authentication.

---

## 📦 Backend Implementation (.NET Core / C#)

### 1. Configure Graph Client

```csharp
using Microsoft.Graph;
using Azure.Identity;

/// <summary>
/// Service to interact with Azure AD B2C user objects via Microsoft Graph.
/// Provides methods to store the passwordless.dev "sub" identifier
/// in a custom extension attribute.
/// </summary>
public class B2CUserService
{
    private readonly GraphServiceClient _graph;

    /// <summary>
    /// Initializes a new instance of <see cref="B2CUserService"/> with
    /// Graph client credentials.
    /// </summary>
    /// <param name="config">Configuration containing tenantId, clientId, and clientSecret.</param>
    public B2CUserService(IConfiguration config)
    {
        var tenantId = config["Graph:TenantId"];
        var clientId = config["Graph:ClientId"];
        var clientSecret = config["Graph:ClientSecret"];

        var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
        _graph = new GraphServiceClient(credential);
    }

    /// <summary>
    /// Saves the passkey subject identifier ("sub") from passwordless.dev
    /// to the given user's extension attribute in Azure AD B2C.
    /// </summary>
    /// <param name="objectId">The B2C user's objectId.</param>
    /// <param name="passkeySub">The passkey subject identifier from passwordless.dev.</param>
    /// <param name="appIdWithoutHyphens">App ID (without hyphens) of the Graph API app registration.</param>
    /// <returns>True if update succeeds; otherwise false.</returns>
    public async Task<bool> SavePasskeySubAsync(string objectId, string passkeySub, string appIdWithoutHyphens)
    {
        var extensionName = $"extension_{appIdWithoutHyphens}_passwordlessSub";

        var userUpdate = new User
        {
            AdditionalData = new Dictionary<string, object>
            {
                { extensionName, passkeySub }
            }
        };

        try
        {
            await _graph.Users[objectId].Request().UpdateAsync(userUpdate);
            return true;
        }
        catch (ServiceException ex)
        {
            Console.WriteLine($"Error updating user: {ex.Message}");
            return false;
        }
    }
}
```

---

### 2. Controller Endpoint

```csharp
/// <summary>
/// API controller for managing passkey registration in Azure AD B2C.
/// </summary>
[ApiController]
[Route("api/[controller]")]
public class PasskeyController : ControllerBase
{
    private readonly B2CUserService _b2cService;

    public PasskeyController(B2CUserService b2cService)
    {
        _b2cService = b2cService;
    }

    /// <summary>
    /// Saves the passwordless.dev "sub" into the user's B2C profile.
    /// </summary>
    /// <param name="dto">The registration DTO containing the passwordless "sub".</param>
    /// <returns>200 OK if saved successfully, error otherwise.</returns>
    [HttpPost("register")]
    public async Task<IActionResult> RegisterPasskey([FromBody] PasskeyRegisterDto dto)
    {
        var oid = User.FindFirst("oid")?.Value;
        if (string.IsNullOrEmpty(oid))
            return Unauthorized("Missing oid in token");

        var success = await _b2cService.SavePasskeySubAsync(
            oid,
            dto.PasswordlessSub,
            "<AppIdWithoutHyphens>"
        );

        if (!success)
            return StatusCode(500, "Failed to save passkey");

        return Ok(new { message = "Passkey registered successfully" });
    }
}

/// <summary>
/// DTO used for passkey registration. Holds the passwordless.dev subject identifier.
/// </summary>
public class PasskeyRegisterDto
{
    public string PasswordlessSub { get; set; }
}
```

---

## 🌐 Frontend (Angular 18 SPA, TypeScript)

### 1. Interfaces

```typescript
/**
 * Response from backend for start-registration endpoint.
 */
export interface StartRegistrationResponse {
  token: string;
}

/**
 * WebAuthn credential payload returned from navigator.credentials.create.
 */
export interface FinishRegistrationRequest {
  id: string;
  rawId: string;
  type: string;
  response: {
    attestationObject: string;
    clientDataJSON: string;
  };
}

/**
 * DTO for saving passkey sub into B2C via backend.
 */
export interface RegisterPasskeyDto {
  passwordlessSub: string;
}
```

---

### 2. Angular Service (Refined)

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, switchMap, map } from 'rxjs';
import { StartRegistrationResponse, FinishRegistrationRequest, RegisterPasskeyDto } from './models';

@Injectable({
  providedIn: 'root'
})
export class PasskeyService {
  private apiBase = '/api/passkey';

  constructor(private http: HttpClient) {}

  /**
   * Initiates the full passkey registration flow.
   * - Calls backend to start registration
   * - Creates WebAuthn credential
   * - Sends credential to backend to finish registration
   * - Saves the "sub" to B2C
   */
  public startRegistration(): Observable<void> {
    return this.http.post<StartRegistrationResponse>(`${this.apiBase}/start-registration`, {})
      .pipe(
        switchMap(this.createCredential.bind(this)),
        switchMap(this.finishRegistration.bind(this)),
        switchMap(this.savePasskeySub.bind(this))
      );
  }

  /**
   * Converts backend start-registration response into a WebAuthn credential.
   */
  private createCredential(response: StartRegistrationResponse): Observable<FinishRegistrationRequest> {
    return new Observable<FinishRegistrationRequest>((observer) => {
      navigator.credentials.create({
        publicKey: JSON.parse(response.token)
      }).then((cred: any) => {
        const publicKeyCred: FinishRegistrationRequest = {
          id: cred.id,
          rawId: this.arrayBufferToBase64(cred.rawId),
          type: cred.type,
          response: {
            attestationObject: this.arrayBufferToBase64(cred.response.attestationObject),
            clientDataJSON: this.arrayBufferToBase64(cred.response.clientDataJSON)
          }
        };
        observer.next(publicKeyCred);
        observer.complete();
      }).catch(err => observer.error(err));
    });
  }

  /**
   * Finishes passkey registration with backend.
   * @returns Observable of passwordless.dev "sub".
   */
  private finishRegistration(credential: FinishRegistrationRequest): Observable<string> {
    return this.http.post<{ sub: string }>(`${this.apiBase}/finish-registration`, credential)
      .pipe(map(this.extractSub));
  }

  /**
   * Extracts the sub field from finish-registration response.
   */
  private extractSub(response: { sub: string }): string {
    return response.sub;
  }

  /**
   * Saves passkey sub to B2C via backend.
   */
  private savePasskeySub(sub: string): Observable<void> {
    const dto: RegisterPasskeyDto = { passwordlessSub: sub };
    return this.http.post<void>(`${this.apiBase}/register`, dto);
  }

  /**
   * Utility: converts ArrayBuffer to base64 string.
   */
  private arrayBufferToBase64(buffer: ArrayBuffer): string {
    let binary = '';
    const bytes = new Uint8Array(buffer);
    const chunkSize = 0x8000;
    for (let i = 0; i < bytes.length; i += chunkSize) {
      const chunk = bytes.subarray(i, i + chunkSize);
      binary += String.fromCharCode.apply(null, Array.from(chunk));
    }
    return btoa(binary);
  }
}
```

---

### 3. Component Usage

```typescript
import { Component } from '@angular/core';
import { PasskeyService } from './passkey.service';

@Component({
  selector: 'app-passkey-register',
  template: `
    <button (click)="registerPasskey()">Register Passkey</button>
  `
})
export class PasskeyRegisterComponent {
  constructor(private passkeyService: PasskeyService) {}

  /**
   * Trigger registration flow when user clicks button.
   */
  public registerPasskey(): void {
    this.passkeyService.startRegistration()
      .subscribe({
        next: () => alert('Passkey registered successfully'),
        error: (err) => alert('Passkey registration failed: ' + err)
      });
  }
}
```

---

## ✅ Outcome

- Each user now has their **unique passkey `sub`** stored in B2C.  
- During authentication, B2C custom policies will verify that the `sub` returned by passwordless.dev matches this stored value.  
- If they don’t match, login fails.

---

## 🔒 Security Notes

- Backend APIs must require a valid **B2C access token**.  
- Never trust `sub` from client → always verify with `passwordless.dev` before saving.  
- Store secrets (Graph, passwordless.dev API key) in **Key Vault**.  
- Consider rate limiting registration attempts.

---
