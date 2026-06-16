# Overriding `login/finish` in open-passkey (Spring Boot 3.x): Issuing Your Own Authorization Tokens After Passkey Sign-In

## Table of Contents

1. [Introduction](#introduction)
2. [How open-passkey Works Under the Hood](#how-open-passkey-works-under-the-hood)
3. [The Problem: The Default `/login/finish` Response](#the-problem-the-default-loginfinish-response)
4. [The Goal Architecture](#the-goal-architecture)
5. [Step 1 — Add the Dependency](#step-1--add-the-dependency)
6. [Step 2 — Reconfigure the Auto-Configured Controller](#step-2--reconfigure-the-auto-configured-controller)
7. [Step 3 — Define the "Black Box" Authorization Service](#step-3--define-the-black-box-authorization-service)
8. [Step 4 — Build the Custom Passkey Controller](#step-4--build-the-custom-passkey-controller)
9. [Step 5 — Wire It All Together](#step-5--wire-it-all-together)
10. [Full Sequence Diagram](#full-sequence-diagram)
11. [Testing the Flow End-to-End](#testing-the-flow-end-to-end)
12. [Real-World Use Cases](#real-world-use-cases)
13. [Security Considerations](#security-considerations)
14. [Extending Further](#extending-further)
15. [Summary](#summary)

---

## Introduction

[open-passkey](https://github.com/locke-inc/open-passkey) is an open-source library that adds **WebAuthn/FIDO2 passkey authentication** to applications, with bonus support for post-quantum signature algorithms (ML-DSA-65 + ES256 hybrid). It ships core cryptographic verification libraries in 8 languages and framework bindings for over 20 server frameworks — including a **Spring Boot 3.x starter** (`open-passkey-spring`).

Out of the box, the Spring starter gives you four REST endpoints:

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/passkey/register/begin` | Start passkey registration |
| `POST` | `/passkey/register/finish` | Complete registration, store credential |
| `POST` | `/passkey/login/begin` | Start authentication (sign-in) |
| `POST` | `/passkey/login/finish` | Complete authentication, verify signature |

This is fantastic for "passkeys replace passwords" use cases. But many real systems don't stop at "the user proved who they are." They need to **bridge that proof into their existing identity/authorization stack** — issuing an OAuth2 access token, a JWT from an internal Authorization Server, or a session token from a downstream identity service.

That's exactly the scenario this tutorial solves:

> **After a user successfully completes passkey sign-in (`/login/finish`), call an internal Authorization Service to mint an access token *on behalf of that user*, and return that token to the client instead of (or in addition to) open-passkey's own response.**

We'll treat the "call the Authorization Service and get a token" step as a **black box** — your consuming application already knows how to do this (e.g., via a Feign client, WebClient call to an OAuth2 token endpoint, or an internal gRPC call). We'll just define the *seam* where that black box plugs in.

---

## How open-passkey Works Under the Hood

Before overriding anything, it helps to see the layering. The Spring module (`packages/server-spring`) is structured like this:

```mermaid
flowchart TB
    subgraph Client["Browser / Mobile Client"]
        SDK["@open-passkey/sdk\n(PasskeyClient)"]
    end

    subgraph SpringApp["Your Spring Boot 3.x Application"]
        direction TB
        Controller["PasskeyController\n(@RestController)\n/passkey/* endpoints"]
        Service["PasskeyService\n(orchestration: challenges,\nverification, sessions)"]
        Stores["Stores\nChallengeStore / CredentialStore\n(in-memory by default,\npluggable for DB/Redis)"]
        Core["core-java\n(WebAuthn / FIDO2\nsignature verification,\nML-DSA-65 + ES256)"]
        AutoConfig["PasskeyAutoConfiguration\n(@EnableConfigurationProperties,\nwires beans together)"]
    end

    SDK -- "navigator.credentials\nWebAuthn ceremony" --> Controller
    Controller --> Service
    Service --> Stores
    Service --> Core
    AutoConfig -.creates.-> Controller
    AutoConfig -.creates.-> Service
    AutoConfig -.creates.-> Stores

    style Controller fill:#4f46e5,color:#fff
    style Service fill:#0891b2,color:#fff
    style Stores fill:#16a34a,color:#fff
    style Core fill:#9333ea,color:#fff
    style AutoConfig fill:#6b7280,color:#fff
```

Key facts that make the override possible:

- **`PasskeyService`** is a plain Spring `@Bean` (created via `@ConditionalOnMissingBean`) that exposes high-level methods: `beginRegistration`, `finishRegistration`, `beginAuthentication`, `finishAuthentication`, `isSessionEnabled`, `getSessionConfig`, `getSessionTokenData`. It has **no HTTP concerns** — it's pure orchestration logic.
- **`PasskeyController`** is a thin `@RestController` that calls `PasskeyService` and translates results to JSON / cookies.
- Because `PasskeyService` is independently injectable, **you can write your own controller that calls `PasskeyService.finishAuthentication(...)` directly** and then layer your own logic (like calling an Authorization Service) on top of the result — without touching the library's internals at all.

This is the seam we'll use.

---

## The Problem: The Default `/login/finish` Response

When the WebAuthn ceremony completes, the client `POST`s to `/passkey/login/finish` with the signed assertion. `PasskeyService.finishAuthentication(...)` returns a `Map<String, Object>` shaped like this:

```json
{
  "userId": "user-123",
  "credentialId": "Q3JlZGVudGlhbElk...",
  "authenticated": true,
  "prfSupported": true,
  "sessionToken": "user-123:1750000000000:abc123signature..."
}
```

The built-in `PasskeyController.finishAuthentication` does one more thing: if sessions are enabled, it **pulls `sessionToken` out of the body** and sets it as an `HttpOnly` cookie instead, returning:

```json
{
  "userId": "user-123",
  "credentialId": "Q3JlZGVudGlhbElk...",
  "authenticated": true,
  "prfSupported": true
}
```

This is *correct* for "I just need a session cookie for my Spring app." But it doesn't help if:

- Your real source of truth for tokens is an external **Authorization Server** (e.g., an OAuth2/OIDC provider, an internal "Auth Service" microservice).
- The client (a SPA, mobile app, or another backend service) needs an **access token** (JWT/opaque bearer token) to call your other APIs — not just a passkey-server session cookie.

So we need `authenticated: true` to trigger a **second step**: "go get a real token for this `userId`."

---

## The Goal Architecture

Here's the target flow, with the Authorization Service represented as a black box:

```mermaid
flowchart LR
    A[Browser] -- "1: POST /passkey/login/begin" --> B[Custom PasskeyController]
    B -- "delegates" --> C[PasskeyService.beginAuthentication]
    C -- "WebAuthn challenge" --> A
    A -- "2: navigator.credentials.get()\n(user taps fingerprint/FaceID)" --> A
    A -- "3: POST /passkey/login/finish\n(signed assertion)" --> B
    B -- "4: delegate" --> C
    C -- "5: verify signature,\nreturn {userId, authenticated:true, ...}" --> B
    B -- "6: userId" --> D[("Authorization Service\n(BLACK BOX)\nissues token for userId")]
    D -- "7: access token\n(JWT / opaque)" --> B
    B -- "8: { authenticated, userId,\n  accessToken, expiresIn }" --> A

    style D fill:#f59e0b,color:#000,stroke-dasharray: 5 5
    style B fill:#4f46e5,color:#fff
    style C fill:#0891b2,color:#fff
```

The contract for the black box is intentionally minimal:

```java
public interface AuthorizationService {
    AuthorizationToken issueTokenFor(String userId);
}
```

Everything from "call the OAuth2 token endpoint" to "look up the user, build claims, sign a JWT" lives **inside** that implementation — our passkey integration code never needs to know.

---

## Step 1 — Add the Dependency

Add the Spring starter to your `pom.xml`. (Check [the GitHub Releases page](https://github.com/locke-inc/open-passkey/releases) or Maven Central for the latest version.)

```xml
<dependency>
    <groupId>com.openpasskey</groupId>
    <artifactId>open-passkey-spring</artifactId>
    <version>0.1.4</version>
</dependency>
```

Configure the base properties in `application.yml`. Note the `base-path` — we're going to point the **auto-configured** controller at an *internal* path so it doesn't collide with our custom controller (more on this in Step 2):

```yaml
open-passkey:
  rp-id: example.com
  rp-display-name: "My App"
  origin: "https://example.com"
  base-path: /internal/op          # <-- auto-configured controller lives here
  # Optional: enable open-passkey's own session cookie too, if you still want it
  # session-secret: "at-least-32-character-hmac-signing-secret!!"
```

> 💡 **Why `base-path`?** `PasskeyController` is annotated `@RequestMapping("${open-passkey.base-path:/passkey}")`. By moving it off `/passkey`, we free up `/passkey/*` for our own controller — which is the *public-facing* API — while still letting Spring auto-create the `PasskeyService` bean we depend on. The auto-configured controller becomes "dead code" mounted at a path nothing calls; if you prefer, you can also just leave both controllers mounted at different prefixes (e.g., expose your custom one at `/api/auth/passkey`).

---

## Step 2 — Reconfigure the Auto-Configured Controller

To recap what `PasskeyAutoConfiguration` gives you for free (these beans are exactly what we'll reuse):

```java
@Configuration
@EnableConfigurationProperties(PasskeyProperties.class)
public class PasskeyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(Stores.ChallengeStore.class)
    public Stores.ChallengeStore challengeStore() {
        return new Stores.MemoryChallengeStore();
    }

    @Bean
    @ConditionalOnMissingBean(Stores.CredentialStore.class)
    public Stores.CredentialStore credentialStore() {
        return new Stores.MemoryCredentialStore();
    }

    @Bean
    @ConditionalOnMissingBean
    public PasskeyService passkeyService(PasskeyProperties props,
                                          Stores.ChallengeStore challengeStore,
                                          Stores.CredentialStore credentialStore) {
        return new PasskeyService(props, challengeStore, credentialStore);
    }

    @Bean
    @ConditionalOnMissingBean
    public PasskeyController passkeyController(PasskeyService passkeyService) {
        return new PasskeyController(passkeyService);
    }
}
```

Two important takeaways:

1. **`PasskeyService` is reusable as-is.** It already knows how to talk to your `CredentialStore` (in-memory by default, but you'll typically swap in a JPA/Redis-backed implementation — see the README's "Pluggable Stores" section).
2. **You don't need to fight `@ConditionalOnMissingBean`.** Just let both controllers exist — the library's one quietly sits at `/internal/op/*`, and yours owns `/passkey/*`.

If you'd rather not have the internal controller mounted at all, you can exclude it explicitly:

```java
@SpringBootApplication(exclude = {}) // base app config — leave as-is
```

```java
// Optional: prevent the library's controller bean from being created.
// Define a no-op bean of the same type so @ConditionalOnMissingBean is satisfied,
// but never register it as a managed @RestController by using a non-component scan path.
// In practice, mounting it on an internal path (Step 1) is simpler and has zero downside.
```

We'll go with the simple `base-path` approach.

---

## Step 3 — Define the "Black Box" Authorization Service

This is the seam your team controls. Define a small interface in your application:

```java
package com.example.auth;

/**
 * Black box: given an authenticated userId (proven via passkey),
 * produce an access token from the organization's Authorization Service.
 *
 * The implementation is entirely up to the consuming application —
 * it might call an OAuth2 token endpoint, an internal STS, mint a JWT
 * directly, etc. This integration only depends on this contract.
 */
public interface AuthorizationService {

    AuthorizationToken issueTokenFor(String userId);
}
```

```java
package com.example.auth;

import java.time.Instant;

/**
 * Minimal token envelope returned to the passkey controller.
 * Extend with refreshToken, scope, etc. as needed.
 */
public record AuthorizationToken(
    String accessToken,
    String tokenType,     // e.g. "Bearer"
    long expiresInSeconds,
    Instant issuedAt
) {}
```

### Example implementations (for illustration only)

These are **not** part of the open-passkey library — they're examples of what your team might put behind the interface. The passkey integration code never sees this detail.

**Example A — Calling an internal OAuth2 Authorization Server via `client_credentials` + token exchange:**

```java
@Service
public class OAuth2AuthorizationService implements AuthorizationService {

    private final WebClient webClient;

    public OAuth2AuthorizationService(WebClient.Builder builder,
                                       @Value("${auth-service.base-url}") String baseUrl) {
        this.webClient = builder.baseUrl(baseUrl).build();
    }

    @Override
    public AuthorizationToken issueTokenFor(String userId) {
        var response = webClient.post()
            .uri("/oauth2/token")
            .contentType(MediaType.APPLICATION_FORM_URLENCODED)
            .bodyValue("grant_type=urn:ietf:params:oauth:grant-type:token-exchange"
                + "&subject_token=" + userId
                + "&subject_token_type=urn:internal:user-id")
            .retrieve()
            .bodyToMono(TokenResponse.class)
            .block();

        return new AuthorizationToken(
            response.accessToken(),
            response.tokenType(),
            response.expiresIn(),
            Instant.now()
        );
    }
}
```

**Example B — Minting a signed JWT directly:**

```java
@Service
public class JwtAuthorizationService implements AuthorizationService {

    private final JwtEncoder jwtEncoder;

    public JwtAuthorizationService(JwtEncoder jwtEncoder) {
        this.jwtEncoder = jwtEncoder;
    }

    @Override
    public AuthorizationToken issueTokenFor(String userId) {
        Instant now = Instant.now();
        Instant expiry = now.plusSeconds(3600);

        JwtClaimsSet claims = JwtClaimsSet.builder()
            .issuer("https://auth.example.com")
            .subject(userId)
            .issuedAt(now)
            .expiresAt(expiry)
            .claim("scope", "api.read api.write")
            .build();

        Jwt jwt = jwtEncoder.encode(JwtEncoderParameters.from(claims));

        return new AuthorizationToken(jwt.getTokenValue(), "Bearer", 3600, now);
    }
}
```

Either way, our controller only ever calls `authorizationService.issueTokenFor(userId)`.

---

## Step 4 — Build the Custom Passkey Controller

Now the centerpiece: a controller that exposes `/passkey/*` for the client, delegating the WebAuthn ceremony to `PasskeyService`, and adding the authorization step on `/login/finish`.

```java
package com.example.passkey;

import com.example.auth.AuthorizationService;
import com.example.auth.AuthorizationToken;
import com.openpasskey.core.WebAuthnException;
import com.openpasskey.spring.PasskeyService;
import com.openpasskey.spring.Stores;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.LinkedHashMap;
import java.util.Map;

@RestController
@RequestMapping("/passkey")
public class CustomPasskeyController {

    private final PasskeyService passkeyService;
    private final AuthorizationService authorizationService;

    public CustomPasskeyController(PasskeyService passkeyService,
                                    AuthorizationService authorizationService) {
        this.passkeyService = passkeyService;
        this.authorizationService = authorizationService;
    }

    // ------------------------------------------------------------
    // Registration — pass-through to PasskeyService (unchanged)
    // ------------------------------------------------------------

    @PostMapping("/register/begin")
    public ResponseEntity<?> beginRegistration(@RequestBody Map<String, String> body) {
        try {
            var result = passkeyService.beginRegistration(body.get("userId"), body.get("username"));
            return ResponseEntity.ok(result);
        } catch (Stores.PasskeyException e) {
            return ResponseEntity.status(e.getStatusCode()).body(Map.of("error", e.getMessage()));
        }
    }

    @PostMapping("/register/finish")
    @SuppressWarnings("unchecked")
    public ResponseEntity<?> finishRegistration(@RequestBody Map<String, Object> body) {
        try {
            var result = passkeyService.finishRegistration(
                (String) body.get("userId"),
                (Map<String, Object>) body.get("credential"),
                (Boolean) body.get("prfSupported")
            );
            return ResponseEntity.ok(result);
        } catch (Stores.PasskeyException e) {
            return ResponseEntity.status(e.getStatusCode()).body(Map.of("error", e.getMessage()));
        } catch (WebAuthnException e) {
            return ResponseEntity.badRequest().body(Map.of("error", e.getMessage()));
        }
    }

    // ------------------------------------------------------------
    // Authentication: begin — pass-through
    // ------------------------------------------------------------

    @PostMapping("/login/begin")
    public ResponseEntity<?> beginAuthentication(@RequestBody(required = false) Map<String, String> body) {
        try {
            String userId = body != null ? body.get("userId") : null;
            var result = passkeyService.beginAuthentication(userId);
            return ResponseEntity.ok(result);
        } catch (Stores.PasskeyException e) {
            return ResponseEntity.status(e.getStatusCode()).body(Map.of("error", e.getMessage()));
        }
    }

    // ------------------------------------------------------------
    // Authentication: finish — THE OVERRIDE
    // ------------------------------------------------------------

    @PostMapping("/login/finish")
    @SuppressWarnings("unchecked")
    public ResponseEntity<?> finishAuthentication(@RequestBody Map<String, Object> body) {
        try {
            // 1. Run the normal WebAuthn verification via PasskeyService.
            //    This validates the signed assertion, checks sign counters,
            //    verifies userHandle, etc. Throws if anything is invalid.
            Map<String, Object> result = passkeyService.finishAuthentication(
                (String) body.get("userId"),
                (Map<String, Object>) body.get("credential")
            );

            // 2. `result.get("authenticated")` is always `true` here —
            //    finishAuthentication() throws on failure, so reaching this
            //    line means the passkey signature was valid.
            String userId = (String) result.get("userId");

            // 3. BLACK BOX: ask the Authorization Service for a token
            //    on behalf of this authenticated user.
            AuthorizationToken token = authorizationService.issueTokenFor(userId);

            // 4. Build the response the client actually needs.
            Map<String, Object> response = new LinkedHashMap<>();
            response.put("authenticated", true);
            response.put("userId", userId);
            response.put("credentialId", result.get("credentialId"));
            if (result.containsKey("prfSupported")) {
                response.put("prfSupported", result.get("prfSupported"));
            }
            response.put("accessToken", token.accessToken());
            response.put("tokenType", token.tokenType());
            response.put("expiresIn", token.expiresInSeconds());

            return ResponseEntity.ok(response);

        } catch (Stores.PasskeyException e) {
            // Challenge missing/expired, credential not found, userHandle mismatch, etc.
            return ResponseEntity.status(e.getStatusCode()).body(Map.of("error", e.getMessage()));
        } catch (WebAuthnException e) {
            // Signature verification failed, replay detected, origin mismatch, etc.
            return ResponseEntity.badRequest().body(Map.of("error", e.getMessage()));
        } catch (AuthorizationException e) {
            // The Authorization Service black box failed — the user IS authenticated
            // via passkey, but we couldn't mint a token. Surface a 502 so the client
            // knows authentication succeeded but token issuance did not.
            return ResponseEntity.status(502).body(Map.of(
                "authenticated", true,
                "error", "token_issuance_failed",
                "message", e.getMessage()
            ));
        }
    }
}
```

```java
package com.example.auth;

/** Thrown by AuthorizationService implementations on failure. */
public class AuthorizationException extends RuntimeException {
    public AuthorizationException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### A note on response shape — `authenticated: true` vs. `authenticated: true` + `accessToken`

This is the crux of the override:

| Field | Source | Meaning |
|-------|--------|---------|
| `authenticated` | `PasskeyService.finishAuthentication()` | "The cryptographic passkey ceremony succeeded." |
| `userId`, `credentialId`, `prfSupported` | `PasskeyService.finishAuthentication()` | Identity + credential metadata from the verified assertion. |
| `accessToken`, `tokenType`, `expiresIn` | **Your `AuthorizationService` black box** | "Here's a token your client can now use to call protected APIs." |

The passkey layer answers *"who is this, cryptographically?"* The authorization layer answers *"what can they do, and how do they prove it to other services?"* Keeping these concerns separated (as the interface boundary does) means you can swap your token issuer later without touching any WebAuthn code.

---

## Step 5 — Wire It All Together

```yaml
# application.yml
open-passkey:
  rp-id: example.com
  rp-display-name: "My App"
  origin: "https://example.com"
  base-path: /internal/op

auth-service:
  base-url: https://auth.example.com
```

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

```java
@Configuration
public class AppConfig {

    @Bean
    public AuthorizationService authorizationService(WebClient.Builder builder,
                                                       @Value("${auth-service.base-url}") String baseUrl) {
        return new OAuth2AuthorizationService(builder, baseUrl);
    }
}
```

Spring will now:

1. Auto-configure `PasskeyService`, `ChallengeStore`, `CredentialStore` (per `PasskeyAutoConfiguration`).
2. Auto-configure the library's `PasskeyController` at `/internal/op/*` (unused by your clients, harmless).
3. Register `CustomPasskeyController` at `/passkey/*` — **this is what your client talks to.**
4. Inject your `AuthorizationService` bean into `CustomPasskeyController`.

---

## Full Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    participant U as User's Device
    participant C as CustomPasskeyController
    participant P as PasskeyService
    participant CS as CredentialStore
    participant AZ as AuthorizationService<br/>(black box)

    U->>C: POST /passkey/login/begin {userId?}
    C->>P: beginAuthentication(userId)
    P->>CS: getByUser(userId) / store challenge
    P-->>C: WebAuthn options (challenge, allowCredentials)
    C-->>U: 200 OK (options)

    Note over U: navigator.credentials.get()<br/>User authenticates with biometric/PIN

    U->>C: POST /passkey/login/finish {userId, credential}
    C->>P: finishAuthentication(userId, credential)
    P->>CS: get(credentialId)
    P->>P: verify signature, sign count,<br/>userHandle, origin (core-java)
    P->>CS: update(signCount)
    P-->>C: {userId, credentialId, authenticated:true, prfSupported}

    rect rgb(255, 247, 224)
    Note over C,AZ: --- THE OVERRIDE ---
    C->>AZ: issueTokenFor(userId)
    AZ-->>C: AuthorizationToken{accessToken, tokenType, expiresIn}
    end

    C-->>U: 200 OK<br/>{authenticated:true, userId,<br/>accessToken, tokenType, expiresIn}

    Note over U: Client stores accessToken<br/>and uses it as Bearer token<br/>for subsequent API calls
```

---

## Testing the Flow End-to-End

### 1. Begin authentication

```bash
curl -X POST https://example.com/passkey/login/begin \
  -H "Content-Type: application/json" \
  -d '{"userId": "user-123"}'
```

Response (WebAuthn options — pass straight to `navigator.credentials.get()` on the client):

```json
{
  "challenge": "weMLPOSx1VfSnMV6uPwDKbjGdKRMaUDGxeDEUTT5VN8",
  "rpId": "example.com",
  "timeout": 300000,
  "userVerification": "preferred",
  "allowCredentials": [
    { "type": "public-key", "id": "Q3JlZGVudGlhbElk..." }
  ]
}
```

### 2. Client completes the WebAuthn ceremony

Using the open-passkey browser SDK (or vanilla `navigator.credentials.get()`), the user authenticates with their fingerprint/Face ID/PIN, producing a signed assertion.

### 3. Finish authentication — the overridden endpoint

```bash
curl -X POST https://example.com/passkey/login/finish \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "user-123",
    "credential": {
      "id": "Q3JlZGVudGlhbElk...",
      "rawId": "Q3JlZGVudGlhbElk...",
      "type": "public-key",
      "response": {
        "clientDataJSON": "...",
        "authenticatorData": "...",
        "signature": "...",
        "userHandle": "dXNlci0xMjM="
      }
    }
  }'
```

Response — now includes the access token issued by your Authorization Service:

```json
{
  "authenticated": true,
  "userId": "user-123",
  "credentialId": "Q3JlZGVudGlhbElk...",
  "prfSupported": true,
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

The client now uses `accessToken` exactly like any OAuth2 bearer token:

```bash
curl https://api.example.com/v1/profile \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIs..."
```

---

## Real-World Use Cases

```mermaid
flowchart TD
    A[Passkey-Authenticated User] --> B{What does the client need next?}
    B -->|"Call your own APIs"| C["BFF Pattern:\nIssue first-party JWT\nfrom your Auth Service"]
    B -->|"Call a 3rd-party API on user's behalf"| D["Token Exchange:\nExchange passkey identity\nfor OAuth2 token via\ntoken-exchange grant"]
    B -->|"Microservices architecture"| E["Service Mesh:\nMint short-lived service token\nscoped to user's permissions"]
    B -->|"Legacy system bridge"| F["Federation:\nMap passkey userId to\nlegacy SSO session token"]

    style A fill:#4f46e5,color:#fff
    style C fill:#16a34a,color:#fff
    style D fill:#16a34a,color:#fff
    style E fill:#16a34a,color:#fff
    style F fill:#16a34a,color:#fff
```

### Use Case 1 — Backend-for-Frontend (BFF) Issuing First-Party JWTs

Your SPA never talks to the Authorization Server directly. After passkey login, your Spring Boot BFF mints a short-lived JWT scoped to that user's permissions and hands it to the SPA for calling your microservices. This is exactly the `JwtAuthorizationService` example above.

### Use Case 2 — Token Exchange for Third-Party APIs

Suppose your app needs to call a partner API (e.g., a payments provider) **as the authenticated user**. After `/login/finish` confirms the passkey assertion, your `AuthorizationService.issueTokenFor(userId)` calls the partner's OAuth2 `token-exchange` endpoint, trading your internal `userId` for a partner-scoped access token — which is then handed back to the client.

### Use Case 3 — Microservices: Short-Lived Service Tokens

In a microservices setup, the passkey server might be a dedicated "Identity Edge" service. Once it confirms `authenticated: true`, it calls an internal Authorization microservice (gRPC/REST) to mint a **scoped, short-TTL token** (e.g., 5 minutes) that downstream services validate via a shared JWKS endpoint — keeping the blast radius of any leaked token small.

### Use Case 4 — Legacy SSO Bridge

If you're migrating from a legacy password-based SSO system to passkeys, `AuthorizationService.issueTokenFor(userId)` can call the **legacy SSO's session-creation API**, returning a legacy session token. The client (which may still expect the old token format) keeps working unmodified while your authentication layer is fully modernized.

---

## Security Considerations

- **Order of operations matters.** Always call `passkeyService.finishAuthentication(...)` *first* and let it throw on any verification failure (`WebAuthnException`, `PasskeyException`) **before** calling the Authorization Service. Never issue a token based on an unverified assertion.
- **Don't log the access token.** Treat the `AuthorizationToken` like any other secret — avoid `System.out.println`/`log.info` on the full response object.
- **Fail closed on authorization errors.** If `AuthorizationService.issueTokenFor(userId)` throws, the user *is* authenticated but has no usable token. Return a distinct error (e.g., HTTP 502 with `"authenticated": true, "error": "token_issuance_failed"`) so the client can retry token issuance without re-running the WebAuthn ceremony.
- **Consider whether you still need open-passkey's session cookie.** If your Authorization Service's token *is* your session mechanism, you can leave `open-passkey.session-secret` unset entirely — there's no need to maintain two parallel session systems.
- **Short-lived tokens + refresh.** Since the passkey ceremony itself is fast and low-friction, consider issuing short-lived access tokens (e.g., 5–15 minutes) and letting the client re-authenticate (often silently, via platform passkey autofill) rather than relying on long-lived refresh tokens.

---

## Extending Further

```mermaid
flowchart LR
    A["/passkey/login/finish"] --> B[PasskeyService.finishAuthentication]
    B --> C{authenticated?}
    C -->|"No (exception)"| D[Return 400/error]
    C -->|"Yes"| E[AuthorizationService.issueTokenFor]
    E --> F{Need refresh token too?}
    F -->|Yes| G["Add refreshToken to response\n(store hashed in DB)"]
    F -->|No| H[Return accessToken only]
    E --> I{Audit logging?}
    I -->|Yes| J["Emit AuthEvent\n(userId, credentialId, timestamp)\nto audit topic"]

    style E fill:#f59e0b,color:#000
```

A few natural extensions to the pattern shown above:

- **Audit events**: emit a domain event (`UserAuthenticatedEvent`) right after step 5 of the sequence diagram, so security teams can correlate passkey logins with token issuance — independent of whether the Authorization Service call succeeds.
- **Refresh tokens**: extend `AuthorizationToken` with a `refreshToken` field and have your `AuthorizationService` implementation persist a hashed version for later rotation.
- **Per-credential scopes**: if `result.get("credentialId")` corresponds to a device with restricted permissions (e.g., a "trusted device" vs. a "new device"), pass that into `issueTokenFor(userId, credentialId)` so your Authorization Service can scope the token accordingly.
- **Same pattern for `/register/finish`**: if you also want to issue a token immediately after a brand-new passkey registration (auto-login after sign-up), apply the identical override to the `finishRegistration` branch of `CustomPasskeyController`.

---

## Summary

1. **`PasskeyService`** (auto-configured by `open-passkey-spring`) is a plain, injectable Spring bean with no HTTP concerns — `finishAuthentication(userId, credential)` does all the WebAuthn signature verification and returns a result map with `userId` once the passkey is verified.
2. Move the library's auto-configured `PasskeyController` off to an internal `base-path` (or simply ignore it), and write your own `@RestController` at `/passkey/*` that delegates the begin/registration endpoints unchanged.
3. In your **custom `/login/finish`**, call `passkeyService.finishAuthentication(...)` first — if it doesn't throw, the user is cryptographically verified.
4. Take `userId` from the result and call your **`AuthorizationService` black box** (`issueTokenFor(userId)`) — entirely decoupled from the passkey logic.
5. Merge the passkey result and the authorization token into a single response and return it to the client.

This gives you the security and UX benefits of passkeys (`open-passkey`) while keeping your existing authorization/token infrastructure as the single source of truth for what an authenticated user is allowed to do.