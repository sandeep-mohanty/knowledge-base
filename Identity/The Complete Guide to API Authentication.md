# The Complete Guide to API Authentication
### From Basic Auth to SSO — How Every Method Works, Why It Exists, and When to Use It

---

> **"Authentication is not a feature. It's the foundation of trust in your system."**  
> This tutorial walks through every major authentication method — not as a checklist, but as an *evolution story* where each method solves the problems of the last.

---

## 📋 Table of Contents

1. [Why Authentication Is Harder Than It Looks](#1-why-authentication-is-harder-than-it-looks)
2. [Basic Authentication — The Grandparent](#2-basic-authentication)
3. [Session-Based Auth — Stateful Trust](#3-session-based-auth)
4. [API Keys — Machine Credentials](#4-api-keys)
5. [Bearer Tokens — The Header Standard](#5-bearer-tokens)
6. [JWT — Self-Contained Proof](#6-jwt-json-web-tokens)
7. [OAuth 2.0 — Delegated Authorization](#7-oauth-20)
8. [OpenID Connect — Identity on Top of OAuth](#8-openid-connect-oidc)
9. [SSO — One Login to Rule Them All](#9-sso-single-sign-on)
10. [How Everything Fits Together](#10-how-everything-fits-together)
11. [Security Non-Negotiables](#11-security-non-negotiables)
12. [Decision Framework](#12-decision-framework)

---

## 1. Why Authentication Is Harder Than It Looks

Every API call answers one fundamental question:

> **"Who are you, and why should I trust you?"**

That question sounds simple. The answers, over decades of engineering, have spawned a rich ecosystem of protocols, token formats, flows, and standards — each solving real failures of the previous approach.

### The Evolution Arc

```mermaid
timeline
    title Authentication Evolution
    section Era 1 — Simplicity
        1990s : Basic Auth
              : Username + Password on every request
              : Simple but costly and insecure
    section Era 2 — State
        Early 2000s : Sessions + Cookies
                    : Prove once, get a ticket
                    : Stateful — hard to scale
    section Era 3 — Scale
        2007-2012 : API Keys
                  : Machine-to-machine credentials
                  : JWTs appear (stateless tokens)
    section Era 4 — Delegation
        2012-2015 : OAuth 2.0
                  : Delegated authorization
                  : OpenID Connect adds identity
    section Era 5 — Enterprise
        2015-Now : SSO + Federated Identity
                 : One login, everything unlocked
                 : PKCE, Zero Trust, MFA
```

### The Core Concepts

Before diving in, three concepts underlie everything:

| Concept | Question Answered | Example |
|---------|------------------|---------|
| **Authentication (AuthN)** | *Who are you?* | Logging into Gmail |
| **Authorization (AuthZ)** | *What can you do?* | Accessing an admin panel |
| **Identity** | *What do we know about you?* | Your email, name, roles |

> **Critical distinction:** OAuth 2.0 is **authorization** — it does NOT tell you who the user is. OpenID Connect adds the identity layer. Many developers confuse these.

---

## 2. Basic Authentication

### How It Works

Basic Auth is the simplest possible scheme: send credentials with every request.

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant DB as Database

    Client->>Client: Encode "username:password" in Base64
    Note over Client: "alice:secret123" → "YWxpY2U6c2VjcmV0MTIz"
    Client->>Server: GET /api/resource<br/>Authorization: Basic YWxpY2U6c2VjcmV0MTIz
    Server->>Server: Decode Base64
    Server->>DB: SELECT * FROM users WHERE username='alice'
    DB-->>Server: {password_hash: "..."}
    Server->>Server: bcrypt.compare("secret123", hash)
    alt Valid credentials
        Server-->>Client: 200 OK + data
    else Invalid
        Server-->>Client: 401 Unauthorized
    end
    Note over DB: ⚠️ This DB hit happens on EVERY request
```

### The Header Format

```http
GET /api/resource HTTP/1.1
Host: api.example.com
Authorization: Basic YWxpY2U6c2VjcmV0MTIz
```

**Decoding it:**
```
YWxpY2U6c2VjcmV0MTIz → alice:secret123
```

> ⚠️ **Base64 is encoding, NOT encryption.** Anyone who intercepts the header can decode it in 2 seconds. HTTPS is mandatory.

### Implementation

```python
# Server-side: Flask example
import base64
from flask import request, jsonify

def check_basic_auth(f):
    def decorated(*args, **kwargs):
        auth_header = request.headers.get('Authorization', '')
        
        if not auth_header.startswith('Basic '):
            return jsonify({'error': 'Missing credentials'}), 401
        
        # Decode
        encoded = auth_header[6:]  # Remove "Basic "
        decoded = base64.b64decode(encoded).decode('utf-8')
        username, password = decoded.split(':', 1)
        
        # Validate (always use hashed passwords!)
        user = db.get_user(username)
        if not user or not bcrypt.checkpw(password.encode(), user.password_hash):
            return jsonify({'error': 'Invalid credentials'}), 401
        
        return f(*args, **kwargs)
    return decorated
```

### Why It Falls Short at Scale

```mermaid
graph LR
    subgraph Problem["⚠️ Basic Auth Problems"]
        A["DB query per request<br/>→ Bottleneck at 10k req/s"]
        B["Credentials travel everywhere<br/>→ More exposure = more risk"]
        C["No session control<br/>→ Can't 'log out' a specific device"]
        D["Can't rotate credentials<br/>→ Changing password logs everyone out"]
    end

    style Problem fill:#ffebee,stroke:#c62828
```

### When to Use It

✅ Internal tooling and admin panels over HTTPS  
✅ Server-to-server in fully trusted, isolated networks  
✅ Quick prototyping and local development  
✅ Legacy systems where simplicity is a hard constraint  
❌ Never for user-facing production systems  
❌ Never over HTTP (plaintext)  

---

## 3. Session-Based Auth

Sessions solved Basic Auth's "database hit on every request" problem with a clever insight: **prove once, remember forever (or until logout).**

### The Session Flow

```mermaid
sequenceDiagram
    participant Browser
    participant Server
    participant SessionStore as Session Store (Redis)
    participant DB

    Browser->>Server: POST /login { username, password }
    Server->>DB: Validate credentials
    DB-->>Server: ✅ Valid — user ID: 42
    Server->>SessionStore: SET session:abc123xyz { userId: 42, role: "admin", createdAt: "..." }
    Server-->>Browser: Set-Cookie: sessionId=abc123xyz; HttpOnly; Secure; SameSite=Strict

    Note over Browser: Cookie stored automatically by browser

    Browser->>Server: GET /dashboard<br/>Cookie: sessionId=abc123xyz
    Server->>SessionStore: GET session:abc123xyz
    SessionStore-->>Server: { userId: 42, role: "admin" }
    Server-->>Browser: 200 OK — here's your dashboard

    Browser->>Server: POST /logout
    Server->>SessionStore: DEL session:abc123xyz
    Server-->>Browser: 200 OK — cookie cleared
    Note over SessionStore: Session gone — even if someone has<br/>the old cookie, it's now worthless
```

### The Cookie Flags That Matter

```javascript
// Express.js — production session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,  // 32+ random bytes
  resave: false,
  saveUninitialized: false,
  store: new RedisStore({ client: redisClient }),
  cookie: {
    httpOnly: true,     // ← JavaScript cannot access this cookie (XSS protection)
    secure: true,       // ← HTTPS only; never sent over HTTP
    sameSite: 'strict', // ← Never sent in cross-site requests (CSRF protection)
    maxAge: 1000 * 60 * 60 * 24  // 24 hours
  }
}));
```

### Cookie Security Flags Explained

| Flag | What It Prevents | Without It |
|------|-----------------|------------|
| `HttpOnly` | XSS — malicious JS reads the cookie | `document.cookie` can steal sessions |
| `Secure` | Network sniffing | Cookie sent over HTTP in clear text |
| `SameSite=Strict` | CSRF attacks | Other sites can trigger requests with your cookie |

### The Scaling Problem

```mermaid
graph TB
    subgraph Monolith["✅ Works Great: Monolith"]
        U1[User] --> S1[Single Server]
        S1 --> SS1[(Session Store)]
    end

    subgraph Distributed["⚠️ Problem: Distributed"]
        U2[User] --> LB[Load Balancer]
        LB -->|"Request 1"| S2[Server A<br/>Session: abc123]
        LB -->|"Request 2"| S3[Server B<br/>❌ No session abc123!]
        LB -->|"Request 3"| S4[Server C<br/>❌ No session abc123!]
    end

    subgraph Solution["✅ Solution: Centralized Store"]
        U3[User] --> LB2[Load Balancer]
        LB2 --> S5[Server A]
        LB2 --> S6[Server B]
        LB2 --> S7[Server C]
        S5 & S6 & S7 --> RS[(Redis Session Store<br/>All sessions here)]
    end

    style Monolith fill:#e8f5e9,stroke:#388e3c
    style Distributed fill:#ffebee,stroke:#c62828
    style Solution fill:#e3f2fd,stroke:#1976d2
```

### When to Use Sessions

✅ Traditional web apps with server-rendered HTML  
✅ Admin dashboards and internal tools  
✅ Anywhere instant revocation is critical (banking, healthcare)  
❌ Mobile apps (can't easily manage cookies)  
❌ Pure API backends serving SPAs or third-party clients  
❌ Microservices with many independent services  

---

## 4. API Keys

API Keys are permanent credentials for **machines, not humans**. They identify an application or service, not a person.

### The Anatomy of an API Key

```
sk_live_a8f3b2c1d4e5f6789012345678901234
│  │    └─────────────────────────────── Random 32 bytes
│  └──────────────────────────────────── Environment: live/test
└─────────────────────────────────────── Type prefix (secret key)
```

### How APIs Use Them

```http
# Option 1: Authorization header (preferred)
GET /v1/charges HTTP/1.1
Host: api.stripe.com
Authorization: Bearer sk_live_abc123def456

# Option 2: Custom header
GET /api/data HTTP/1.1
X-API-Key: your_api_key_here

# Option 3: Query parameter — AVOID THIS
GET /api/data?api_key=abc123   ← appears in server logs, browser history!
```

### Generating and Storing API Keys Securely

```javascript
const crypto = require('crypto');
const argon2 = require('argon2');

// 1. Generate
function generateApiKey(prefix = 'sk') {
  const rawKey = `${prefix}_${crypto.randomBytes(32).toString('hex')}`;
  return rawKey;
}

// 2. Hash before storing (NEVER store the raw key)
async function storeApiKey(userId, scopes) {
  const rawKey = generateApiKey();
  const hash = await argon2.hash(rawKey);
  
  await db.apiKeys.create({
    userId,
    keyHash: hash,
    // Store only a prefix for identification (e.g., "sk_a8f3b2")
    keyPrefix: rawKey.substring(0, 10),
    scopes,    // ['read:orders', 'write:invoices']
    createdAt: new Date()
  });
  
  // Return raw key ONCE — never again
  return rawKey;
}

// 3. Validate incoming key
async function validateApiKey(incomingKey) {
  // Find candidate by prefix (avoids full-table scan)
  const prefix = incomingKey.substring(0, 10);
  const candidates = await db.apiKeys.findByPrefix(prefix);
  
  for (const candidate of candidates) {
    if (await argon2.verify(candidate.keyHash, incomingKey)) {
      return candidate; // Found!
    }
  }
  return null;
}
```

### The Power Features of API Keys

```mermaid
mindmap
  root((API Key Features))
    Scoping
      Read-only keys for read-only apps
      Write keys scoped to specific resources
      Per-environment keys (test/live)
    Rotation
      Compromised? Revoke and reissue
      No user password change required
      Zero user friction
    Rate Limiting
      Limit per key not per IP
      IP spoofing is trivial; key spoofing is not
      Different limits per tier
    Analytics
      Usage per key
      Which endpoints per client
      Anomaly detection per key
```

### Real-World Examples

```python
# FastAPI: API key middleware
from fastapi import Security, HTTPException
from fastapi.security.api_key import APIKeyHeader

api_key_header = APIKeyHeader(name="X-API-Key")

async def get_api_key(api_key: str = Security(api_key_header)):
    key_record = await validate_api_key(api_key)
    if not key_record:
        raise HTTPException(status_code=403, detail="Invalid API key")
    
    # Scope check
    if "read:orders" not in key_record.scopes:
        raise HTTPException(status_code=403, detail="Insufficient scope")
    
    return key_record

@app.get("/orders")
async def list_orders(key=Depends(get_api_key)):
    return await orders_service.list(key.userId)
```

### When to Use API Keys

✅ Third-party developer APIs (Stripe, Twilio, SendGrid pattern)  
✅ Internal microservice-to-microservice calls  
✅ CI/CD pipelines and automation scripts  
✅ IoT devices calling home  
❌ User authentication (keys don't carry user identity)  
❌ Where per-request granular authorization is needed  

---

## 5. Bearer Tokens

Bearer Token is **not** an authentication mechanism — it's a **header format** standardized in RFC 6750.

```http
Authorization: Bearer <token>
```

> **"Bearer"** means: *whoever presents this token gets access.* Possession is proof. The server doesn't verify who you are beyond the token itself.

The token itself can be **anything:**

```mermaid
graph LR
    BT["Authorization: Bearer <token>"]
    BT --> A["Random opaque string<br/>(like an API key)"]
    BT --> B["OAuth 2.0 access token<br/>(opaque to the client)"]
    BT --> C["JWT<br/>(self-contained, verifiable)"]
    BT --> D["Any format the server<br/>understands"]

    style BT fill:#e3f2fd,stroke:#1976d2
```

### Why Bearer Tokens Exist

Before RFC 6750, every API invented its own header format:

```http
# The wild west before Bearer standardization
X-Auth-Token: abc123          # Twitter (old)
Authorization: GoogleLogin ... # Google (old)
X-API-Token: abc123           # Random APIs
```

RFC 6750 standardized this for OAuth 2.0, and the ecosystem converged. Now every library, proxy, and API gateway knows what `Authorization: Bearer` means.

### The Security Implication

Since possession is proof:

```mermaid
flowchart LR
    A["Bearer Token Leaked"] --> B["Attacker has full access"]
    B --> C["Until: token expires<br/>or is manually revoked"]
    
    D["Mitigation Strategies"]
    D --> E["Short expiry (15 min)"]
    D --> F["Refresh token pattern"]
    D --> G["Store in HttpOnly cookies"]
    D --> H["HTTPS always"]

    style A fill:#ffebee,stroke:#c62828
    style D fill:#e8f5e9,stroke:#388e3c
```

---

## 6. JWT (JSON Web Tokens)

JWT is a **token format** — not a protocol. Its superpower: the token *is* the proof. No database lookup required.

### Anatomy of a JWT

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyXzEyMyIsInJvbGUiOiJhZG1pbiIsImV4cCI6MTcwMDAwMDAwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
│─────────────────────────────────────│─────────────────────────────────────────────────────────────│─────────────────────────────────────────────────────────────────│
                Header                                          Payload                                                   Signature
```

**Part 1 — Header:**
```json
{
  "alg": "RS256",   // Signing algorithm (prefer RS256 over HS256 in production)
  "typ": "JWT"
}
```

**Part 2 — Payload (Claims):**
```json
{
  "sub": "user_123",                    // Subject (user ID)
  "iss": "https://auth.yourapp.com",   // Issuer
  "aud": "https://api.yourapp.com",    // Audience
  "role": "admin",
  "email": "alice@example.com",
  "iat": 1700000000,                   // Issued at
  "exp": 1700003600                    // Expires at (1 hour later)
}
```

> ⚠️ **The payload is Base64-encoded, NOT encrypted.** Anyone can decode and read it. Never put passwords, credit card numbers, or sensitive PII in a JWT payload.

**Part 3 — Signature:**
```
HMACSHA256(
  base64url(header) + "." + base64url(payload),
  secret_key            // HS256: shared secret
                        // RS256: private key (server signs, anyone verifies with public key)
)
```

### The JWT Verification Flow

```mermaid
sequenceDiagram
    participant Client
    participant API as API Server
    participant AuthServer as Auth Server

    Note over AuthServer: Has the signing key (or key pair)

    Client->>AuthServer: POST /login { username, password }
    AuthServer->>AuthServer: Validate credentials
    AuthServer->>AuthServer: Sign JWT with private key
    AuthServer-->>Client: { access_token: "eyJ...", expires_in: 900 }

    Client->>API: GET /orders<br/>Authorization: Bearer eyJ...

    API->>API: Split token into header.payload.signature
    API->>API: Recompute expected signature using public key
    API->>API: Compare: computed == received?
    API->>API: Check exp claim (not expired?)
    API->>API: Check iss claim (trusted issuer?)
    API->>API: Check aud claim (intended for me?)

    alt All checks pass
        API-->>Client: 200 OK + orders data
        Note over API: No database lookup needed!
    else Any check fails
        API-->>Client: 401 Unauthorized
    end
```

### Implementation

```javascript
const jwt = require('jsonwebtoken');

// ─── Issue on login ──────────────────────────────────────────────
function issueTokens(user) {
  const accessToken = jwt.sign(
    {
      sub: user.id,
      role: user.role,
      email: user.email
      // Don't include: password, SSN, payment info
    },
    process.env.JWT_PRIVATE_KEY,  // RS256 private key
    {
      algorithm: 'RS256',
      expiresIn: '15m',           // Short-lived!
      issuer: 'https://auth.yourapp.com',
      audience: 'https://api.yourapp.com'
    }
  );

  const refreshToken = crypto.randomBytes(64).toString('hex');
  // Store refresh token in DB (allows revocation)
  await db.refreshTokens.create({ token: hash(refreshToken), userId: user.id });

  return { accessToken, refreshToken };
}

// ─── Verify on each request ───────────────────────────────────────
const authMiddleware = (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'No token' });

    const decoded = jwt.verify(token, process.env.JWT_PUBLIC_KEY, {
      algorithms: ['RS256'],        // Never allow 'none'!
      issuer: 'https://auth.yourapp.com',
      audience: 'https://api.yourapp.com'
    });

    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid or expired token' });
  }
};

// ─── Refresh flow ────────────────────────────────────────────────
async function refreshAccessToken(refreshToken) {
  const record = await db.refreshTokens.findOne({ token: hash(refreshToken) });
  if (!record) throw new Error('Invalid refresh token');
  
  const user = await db.users.findById(record.userId);
  const { accessToken } = issueTokens(user);
  return accessToken;
}
```

### The Access + Refresh Token Pattern

```mermaid
sequenceDiagram
    participant Client
    participant API

    Client->>API: POST /login
    API-->>Client: access_token (15min) + refresh_token (7days)

    loop Normal usage
        Client->>API: GET /data<br/>Bearer: access_token
        API-->>Client: 200 OK
    end

    Note over Client: access_token expires...

    Client->>API: GET /data<br/>Bearer: expired_access_token
    API-->>Client: 401 Unauthorized

    Client->>API: POST /refresh<br/>{ refresh_token: "..." }
    API->>API: Look up refresh_token in DB
    API->>API: Valid? Issue new access_token
    API-->>Client: new access_token (15min)

    Client->>API: POST /logout
    API->>API: DELETE refresh_token from DB
    API-->>Client: 200 OK
    Note over API: Even if old access_token is held,<br/>it'll expire in ≤15 min
```

### JWT Pitfalls — The Hall of Shame

```mermaid
graph TD
    subgraph Pitfalls["⚠️ JWT Pitfalls"]
        P1["alg: none<br/>Never accept unsigned tokens<br/>Some libraries did this"]
        P2["Long expiry (days/weeks)<br/>Impossible to revoke<br/>if compromised"]
        P3["Storing in localStorage<br/>Vulnerable to XSS<br/>JS can read it"]
        P4["Sensitive data in payload<br/>It's just Base64!<br/>Anyone can decode"]
        P5["Not validating iss/aud<br/>Token from another app<br/>could work on yours"]
        P6["HS256 with weak secret<br/>Brute-forceable<br/>Use RS256 in production"]
    end

    style Pitfalls fill:#ffebee,stroke:#c62828
```

### When to Use JWTs

✅ Microservices — services validate the token locally, no centralized session store  
✅ APIs consumed by mobile apps and SPAs  
✅ Cross-domain authentication  
❌ When instant revocation is critical (bank fraud, admin removal)  
❌ When token size is a constraint (JWTs are bigger than session IDs)  

---

## 7. OAuth 2.0

### The Problem OAuth Solves

Before OAuth, "connect your app with Google" meant:

```
User → App: "Please give me your Google password"
User → App: 😬 hands over Google password
App → Google: Here's their password, let me in!
```

**Problems:** The app sees your Google password. You can't revoke access without changing your password. The app can do anything — forever.

OAuth 2.0 answer: **delegated authorization without password sharing.**

### The Four Roles

```mermaid
graph TB
    RO["👤 Resource Owner<br/>The User<br/>(Alice)"]
    C["📱 Client<br/>The App wanting access<br/>(Your Todo App)"]
    AS["🔑 Authorization Server<br/>The Trust Broker<br/>(Google's Auth Server)"]
    RS["📦 Resource Server<br/>The API with the data<br/>(Google Drive API)"]

    RO -->|"Grants permission to"| C
    C -->|"Redirects to"| AS
    AS -->|"Authenticates"| RO
    AS -->|"Issues tokens to"| C
    C -->|"Accesses with token"| RS

    style RO fill:#e3f2fd,stroke:#1976d2
    style C fill:#f3e5f5,stroke:#7b1fa2
    style AS fill:#e8f5e9,stroke:#388e3c
    style RS fill:#fff3e0,stroke:#e65100
```

### The Authorization Code Flow (Full Detail)

```mermaid
sequenceDiagram
    participant User
    participant App as Your App (Client)
    participant AS as Google Auth Server
    participant RS as Google APIs

    User->>App: Click "Connect with Google"
    
    App->>App: Generate state = random_token (CSRF protection)
    App->>User: Redirect to Google:
    Note over App,AS: GET accounts.google.com/o/oauth2/auth<br/>?client_id=YOUR_APP_ID<br/>&redirect_uri=https://yourapp.com/callback<br/>&response_type=code<br/>&scope=email profile drive.readonly<br/>&state=abc123

    User->>AS: Log into Google (if not already)
    AS->>User: "YourApp wants access to: Email, Profile, Drive (read)"
    User->>AS: ✅ Allow

    AS->>App: Redirect to yourapp.com/callback<br/>?code=4/P7q7W91a-oMsCeLvIaQm6bTrgtp7<br/>&state=abc123

    App->>App: Verify state matches (CSRF check)
    Note over App: Code is single-use and expires in ~10 minutes

    App->>AS: POST /token (server-to-server — never in browser!)<br/>{ client_id, client_secret, code, redirect_uri, grant_type: "authorization_code" }
    AS->>AS: Validate code + client_secret
    AS-->>App: { access_token, refresh_token, expires_in: 3600, scope }

    Note over App: Access token stored securely on server
    Note over App: Refresh token stored in DB

    App->>RS: GET /drive/v3/files<br/>Authorization: Bearer access_token
    RS-->>App: { files: [...] }

    App->>User: Here are your Drive files!
```

### Why the Code Exchange Step?

```mermaid
graph LR
    A["Why not return access_token directly<br/>in the redirect URL?"] --> B["Browser history stores it"]
    A --> C["Server logs record it"]
    A --> D["Referrer headers leak it"]
    
    E["The code is single-use and useless<br/>without the client_secret"] --> F["Exchange happens server-to-server"]
    F --> G["Access token never touches the browser"]

    style A fill:#ffebee,stroke:#c62828
    style E fill:#e8f5e9,stroke:#388e3c
```

### Other OAuth 2.0 Grant Types

**Client Credentials (Machine-to-Machine):**

```javascript
// Your backend calling a payment API — no user involved
const response = await fetch('https://api.payment.com/oauth/token', {
  method: 'POST',
  body: new URLSearchParams({
    grant_type: 'client_credentials',
    client_id: process.env.PAYMENT_CLIENT_ID,
    client_secret: process.env.PAYMENT_CLIENT_SECRET,
    scope: 'payments:write invoices:read'
  })
});
const { access_token } = await response.json();
// Use access_token to call payment APIs
```

**PKCE (for Mobile Apps and SPAs):**

```mermaid
sequenceDiagram
    participant App as Mobile/SPA App
    participant AS as Auth Server

    App->>App: Generate code_verifier = random 128 chars
    App->>App: code_challenge = SHA256(code_verifier) → base64url

    App->>AS: Auth request with code_challenge<br/>(no client_secret — apps can't keep secrets!)

    AS->>AS: Store code_challenge for this auth request

    App->>AS: Token request with code_verifier
    AS->>AS: SHA256(code_verifier) == stored code_challenge?
    AS-->>App: ✅ access_token (if match)

    Note over App,AS: Even if auth code is intercepted,<br/>attacker doesn't have code_verifier
```

```javascript
// PKCE implementation
function generateCodeVerifier() {
  return crypto.randomBytes(64).toString('base64url');
}

function generateCodeChallenge(verifier) {
  return crypto.createHash('sha256').update(verifier).digest('base64url');
}

const verifier = generateCodeVerifier();
const challenge = generateCodeChallenge(verifier);

// In auth URL
const authUrl = new URL('https://auth.example.com/authorize');
authUrl.searchParams.set('code_challenge', challenge);
authUrl.searchParams.set('code_challenge_method', 'S256');
// No client_secret in URL or body

// In token exchange
body.append('code_verifier', verifier);
// No client_secret
```

### Grant Type Decision Tree

```mermaid
flowchart TD
    A{Who is authenticating?}
    A -->|"A human user"| B{Can you store a client_secret?}
    A -->|"Machine/service"| C[Client Credentials Flow]
    B -->|"Yes (server-side app)"| D[Authorization Code Flow]
    B -->|"No (SPA or mobile)"| E[Authorization Code + PKCE]
    
    style C fill:#e8f5e9,stroke:#388e3c
    style D fill:#e8f5e9,stroke:#388e3c
    style E fill:#e8f5e9,stroke:#388e3c
```

---

## 8. OpenID Connect (OIDC)

### The Identity Gap in OAuth

OAuth 2.0 gives you an access token. But it doesn't tell you **who the user is.** You know the user authorized your app to access their data — but what's their name? Their email? Their user ID?

```mermaid
graph LR
    subgraph OAuth["OAuth 2.0 Alone"]
        A["access_token"] --> B["Can call APIs"]
        A -.->|"❌ Doesn't tell you"| C["Who the user is"]
    end

    subgraph OIDC["OAuth 2.0 + OpenID Connect"]
        D["access_token"] --> E["Can call APIs"]
        F["id_token (JWT)"] --> G["Who the user is"]
        D & F -->|"Both issued together"| H["Complete solution"]
    end

    style OAuth fill:#fff3e0,stroke:#e65100
    style OIDC fill:#e8f5e9,stroke:#388e3c
```

### One Scope Change Unlocks Identity

```
scope=email profile           ← OAuth only: access to email/profile resources
scope=openid email profile    ← OIDC: same + an ID Token identifying the user
```

### The ID Token

```json
{
  "iss": "https://accounts.google.com",
  "sub": "118374829374829374",       // Stable, unique user identifier
  "aud": "your_client_id",
  "exp": 1700003600,
  "iat": 1700000000,
  "email": "alice@gmail.com",
  "email_verified": true,
  "name": "Alice Smith",
  "picture": "https://lh3.googleusercontent.com/...",
  "locale": "en",
  "nonce": "random_value"            // Prevents replay attacks
}
```

### Verifying the ID Token

```javascript
const { OAuth2Client } = require('google-auth-library');
const client = new OAuth2Client(CLIENT_ID);

async function verifyGoogleIdToken(idToken) {
  const ticket = await client.verifyIdToken({
    idToken,
    audience: CLIENT_ID,
  });
  
  const payload = ticket.getPayload();
  // payload.sub — stable unique Google user ID
  // payload.email — user's email
  // payload.email_verified — Google verified this email
  
  return payload;
}
```

The verification process:

```mermaid
flowchart LR
    A["Receive id_token from auth server"] --> B["Split into header.payload.signature"]
    B --> C["Fetch public keys from\n/.well-known/openid-configuration"]
    C --> D["Verify signature using public key"]
    D --> E{Checks}
    E -->|"iss == expected issuer"| F["✅ Trusted source"]
    E -->|"aud == your client_id"| G["✅ Intended for you"]
    E -->|"exp > now()"| H["✅ Not expired"]
    E -->|"nonce matches"| I["✅ Not replayed"]
    F & G & H & I --> J["✅ User identity confirmed — no extra API call!"]

    style J fill:#e8f5e9,stroke:#388e3c
```

### OIDC Standard Endpoints

```
GET /.well-known/openid-configuration
```

This well-known URL returns everything you need to integrate:

```json
{
  "issuer": "https://accounts.google.com",
  "authorization_endpoint": "https://accounts.google.com/o/oauth2/v2/auth",
  "token_endpoint": "https://oauth2.googleapis.com/token",
  "userinfo_endpoint": "https://openid.googleapis.com/v1/userinfo",
  "jwks_uri": "https://www.googleapis.com/oauth2/v3/certs",
  "scopes_supported": ["openid", "email", "profile"],
  "response_types_supported": ["code", "token", "id_token"],
  ...
}
```

---

## 9. SSO (Single Sign-On)

SSO is the **experience**, not a protocol. It's built on top of OAuth/OIDC or SAML.

> **The magic:** You log in once to your company's Identity Provider (IdP). Every other app trusts that IdP. No more 40 passwords.

### How the SSO Session Works

```mermaid
sequenceDiagram
    participant User
    participant AppA as App A (Jira)
    participant IdP as Company IdP (Okta)
    participant AppB as App B (Slack)

    User->>AppA: Visit Jira
    AppA->>IdP: Redirect (no SSO session yet)
    IdP->>User: Login page
    User->>IdP: Username + Password + MFA
    IdP->>IdP: Create SSO session (cookie on okta.company.com)
    IdP->>AppA: ID token for Jira
    AppA->>User: ✅ Welcome to Jira!

    Note over User: Later, User visits Slack...
    User->>AppB: Visit Slack
    AppB->>IdP: Redirect (check for SSO session)
    IdP->>IdP: SSO session exists! ✅
    IdP->>AppB: ID token for Slack (no login prompt!)
    AppB->>User: ✅ Welcome to Slack!

    Note over User: One login unlocked both apps
```

### SAML vs OIDC-Based SSO

```mermaid
graph TB
    subgraph SAML["SAML 2.0 (Enterprise Veteran)"]
        S1["XML-based assertions"]
        S2["20 years of enterprise adoption"]
        S3["Works with ADFS, Okta, Ping Identity"]
        S4["Heavy, complex, but battle-tested"]
        S5["Common for: Salesforce, ServiceNow, legacy apps"]
    end

    subgraph OIDC2["OIDC-Based SSO (Modern)"]
        O1["JSON + JWT based"]
        O2["Built on OAuth 2.0"]
        O3["Simpler to implement"]
        O4["Better for mobile and SPAs"]
        O5["Common for: modern SaaS, cloud-native apps"]
    end

    style SAML fill:#e3f2fd,stroke:#1976d2
    style OIDC2 fill:#e8f5e9,stroke:#388e3c
```

### Implementing SSO in Your App (Passport.js + OIDC)

```javascript
const { Strategy } = require('passport-openidconnect');

passport.use('sso', new Strategy({
  issuer: 'https://your-company.okta.com',
  authorizationURL: 'https://your-company.okta.com/oauth2/v1/authorize',
  tokenURL: 'https://your-company.okta.com/oauth2/v1/token',
  userInfoURL: 'https://your-company.okta.com/oauth2/v1/userinfo',
  clientID: process.env.OKTA_CLIENT_ID,
  clientSecret: process.env.OKTA_CLIENT_SECRET,
  callbackURL: 'https://yourapp.com/auth/callback',
  scope: 'openid email profile groups'
}, async (tokenSet, userInfo, done) => {
  // Find or create user in your DB
  const user = await db.users.findOrCreate({
    where: { ssoId: userInfo.sub },
    defaults: {
      email: userInfo.email,
      name: userInfo.name,
      groups: userInfo.groups
    }
  });
  return done(null, user);
}));

// Routes
app.get('/auth/login', passport.authenticate('sso'));
app.get('/auth/callback', 
  passport.authenticate('sso', { failureRedirect: '/login' }),
  (req, res) => res.redirect('/dashboard')
);
```

### SSO for Microservices — The API Gateway Pattern

```mermaid
graph LR
    Client["Client<br/>(Browser/Mobile)"] --> GW["API Gateway<br/>1. Validate JWT<br/>2. Check scopes<br/>3. Inject X-User-Id header"]
    GW -->|"X-User-Id: 42<br/>X-User-Role: admin"| SA["Service A<br/>(Orders)"]
    GW -->|"X-User-Id: 42<br/>X-User-Role: admin"| SB["Service B<br/>(Inventory)"]
    GW -->|"X-User-Id: 42<br/>X-User-Role: admin"| SC["Service C<br/>(Payments)"]
    GW -->|"401 Unauthorized"| Blocked["❌ Invalid token<br/>never reaches services"]

    style GW fill:#e3f2fd,stroke:#1976d2
    style Blocked fill:#ffebee,stroke:#c62828
```

Each downstream service trusts the gateway's headers — no repeated token validation, no DB calls per service.

---

## 10. How Everything Fits Together

Real production systems use **multiple auth methods simultaneously** at different layers.

```mermaid
graph TB
    subgraph UserFacing["🌐 User-Facing Web App"]
        U1["Login: OIDC/OAuth 2.0 (SSO via Okta/Auth0)"]
        U2["Session: JWT access token in HttpOnly cookie"]
        U3["Internal API calls: Bearer JWT"]
    end

    subgraph PublicAPI["📡 Public Developer API"]
        P1["Authentication: API Keys (like Stripe)"]
        P2["Rate limiting: Per API key"]
        P3["Scoped access: OAuth 2.0 scopes"]
    end

    subgraph ServiceToService["⚙️ Service-to-Service (Internal)"]
        S1["Auth: OAuth 2.0 Client Credentials"]
        S2["Token: Short-lived JWT (15-minute)"]
        S3["No user context — machine identity only"]
    end

    UserFacing --> GW["API Gateway / Load Balancer"]
    PublicAPI --> GW
    ServiceToService --> GW
    GW --> Svcs["Internal Microservices"]

    style UserFacing fill:#e3f2fd,stroke:#1976d2
    style PublicAPI fill:#f3e5f5,stroke:#7b1fa2
    style ServiceToService fill:#e8f5e9,stroke:#388e3c
```

---

## 11. Security Non-Negotiables

These are table stakes — ship nothing without them.

```mermaid
mindmap
  root((Security<br/>Non-Negotiables))
    Transport
      Always HTTPS/TLS
      HSTS header
      Certificate pinning for mobile
    Cookies
      HttpOnly prevents XSS theft
      Secure prevents HTTP leakage
      SameSite prevents CSRF
    Tokens
      Short expiry: access tokens 15 min
      Refresh tokens stored in DB
      Hash API keys at rest
    JWT
      Never accept alg:none
      Validate iss, aud, exp
      Use RS256 not HS256
    OAuth
      Always use state parameter
      PKCE for public clients
      Request minimum scopes
    Secrets
      Secrets in environment variables
      Rotate regularly
      Never in source code or logs
```

### The Checklist

```
□ HTTPS on every endpoint — no exceptions
□ HttpOnly + Secure + SameSite=Strict on session cookies
□ Access tokens: 15 minutes max
□ Refresh tokens: stored in DB, revocable
□ JWT: validate iss, aud, exp on every request
□ JWT: never accept { "alg": "none" }
□ JWT: use RS256 (asymmetric) in production
□ OAuth: always validate state parameter (CSRF)
□ OAuth: use PKCE for mobile and SPA clients
□ API Keys: hash at rest (never store raw key)
□ API Keys: log prefix only, never full key
□ No credentials in query params (appear in logs)
□ No sensitive data in JWT payload (it's readable!)
□ Rate limiting on all auth endpoints
□ Account lockout after N failed attempts
□ MFA for admin and sensitive operations
```

---

## 12. Decision Framework

### Which Auth Method for Which Scenario?

```mermaid
flowchart TD
    A{What are you authenticating?}
    
    A -->|"A human user"| B{What kind of app?}
    A -->|"A machine/service"| C{External or internal?}
    A -->|"Third-party developer"| D["API Keys<br/>with OAuth scopes"]
    
    B -->|"Web app, enterprise/company"| E["SSO via OIDC<br/>(Okta, Auth0, Azure AD)"]
    B -->|"Consumer app"| F["OAuth 2.0 + OIDC<br/>Login with Google/Apple"]
    B -->|"Simple internal tool"| G["Session-based auth<br/>or Basic Auth over HTTPS"]
    
    C -->|"External (calling third-party API)"| H["OAuth 2.0 Client Credentials<br/>or API Keys provided by them"]
    C -->|"Internal microservices"| I["OAuth 2.0 Client Credentials<br/>Short-lived JWT passed between services"]
    
    E & F --> J["Token format: JWT<br/>Storage: HttpOnly cookie<br/>Refresh: DB-stored refresh token"]
    
    style D fill:#f3e5f5,stroke:#7b1fa2
    style E fill:#e8f5e9,stroke:#388e3c
    style F fill:#e8f5e9,stroke:#388e3c
    style G fill:#e3f2fd,stroke:#1976d2
    style H fill:#fff3e0,stroke:#e65100
    style I fill:#fff3e0,stroke:#e65100
    style J fill:#e8f5e9,stroke:#388e3c
```

### The Mental Model

| Method | The Analogy | Use When |
|--------|-------------|----------|
| **Basic Auth** | Showing ID every time you enter a building | Simple internal tools, legacy systems |
| **Sessions** | Getting a wristband at the door | Monolith web apps, admin panels |
| **API Keys** | A master key for a specific tenant | Developer APIs, machine-to-machine |
| **Bearer Token** | A concert ticket (possesion = entry) | Any modern API |
| **JWT** | A concert ticket with your name printed on it (self-verifying) | Microservices, stateless APIs |
| **OAuth 2.0** | Giving a valet partial access to your car (just drive, not search glove box) | Delegated access to user resources |
| **OIDC** | Valet access + a name tag saying who owns the car | Login with Google/GitHub/Apple |
| **SSO** | A company badge that opens all doors | Enterprise apps, multi-app organizations |

### The Evolution Summary

```mermaid
graph LR
    BA["Basic Auth<br/>Here's my credential<br/>trust me"] -->|"Problem: DB hit<br/>every request"| S["Sessions<br/>I proved myself once<br/>here's my ticket"]
    S -->|"Problem: Hard to<br/>scale statefully"| JWT["JWT<br/>My ticket contains<br/>all the proof"]
    JWT -->|"Problem: Can't access<br/>other services safely"| OAuth["OAuth 2.0<br/>Let me access<br/>their stuff for you"]
    OAuth -->|"Problem: Doesn't tell<br/>me who the user IS"| OIDC["OIDC<br/>And tell me who<br/>they are too"]
    OIDC -->|"Problem: Many apps,<br/>many logins"| SSO["SSO<br/>One login unlocks<br/>everything"]

    style BA fill:#ffcdd2,stroke:#c62828
    style S fill:#fff9c4,stroke:#f57f17
    style JWT fill:#fff9c4,stroke:#f57f17
    style OAuth fill:#c8e6c9,stroke:#388e3c
    style OIDC fill:#c8e6c9,stroke:#388e3c
    style SSO fill:#e8f5e9,stroke:#2e7d32
```

---

## 🏁 Conclusion

Authentication is a series of very deliberate engineering decisions — each one balancing **security, usability, and scale.**

The key insight from every method:

- **Basic Auth & API Keys** — "Here's my credential, trust me" *(simple, but all trust up front)*
- **Sessions** — "I proved myself once, here's my ticket" *(stateful, easy to revoke)*
- **JWT** — "My ticket contains all the proof, check the signature" *(stateless, hard to revoke)*
- **OAuth 2.0** — "Let me access their stuff on their behalf" *(delegation, not authentication)*
- **OIDC** — "Let me know who they are, too" *(identity on top of delegation)*
- **SSO** — "Let one login unlock everything" *(the experience that ties it all together)*

Each evolved to solve a real failure of the previous. Understanding that evolution is understanding when to use each today — and how to combine them into a production system that is secure, scalable, and maintainable.

---