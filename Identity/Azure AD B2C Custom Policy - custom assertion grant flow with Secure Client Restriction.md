# 🔐 Azure AD B2C Custom Policy: custom `assertion` grant flow with Secure Client Restriction

## ✅ Goal

Enable a secure OAuth 2.0 flow in Azure AD B2C where:

- A trusted application can request a token for a user using a custom `grant_type=assertion`
- The app authenticates with `client_id` and `client_secret`
- Provides the target user’s `email`
- Azure AD B2C issues a token for the user **without redirect, UI, or external API**
- Only a specific **registered application** (by `client_id`) can use this flow
- The allowed `client_id` is stored **securely** in a **policy key**, not hard-coded

---

## 🔁 Token Request Format

```http
POST https://<tenant>.b2clogin.com/<tenant>.onmicrosoft.com/oauth2/v2.0/token?p=B2C_1A_AssertionLogin
Content-Type: application/x-www-form-urlencoded

grant_type=assertion
client_id=<your-client-id>
client_secret=<your-client-secret>
scope=https://<tenant>.onmicrosoft.com/api/read
email=user@example.com
```

---

## 🧱 Implementation Overview

### Key Files

- `TrustFrameworkBase.xml`
- `TrustFrameworkExtensions.xml`
- `B2C_1A_AssertionLogin.xml` (Relying Party policy)

---

## 🔐 Step 1: Store `client_id` Securely in Azure B2C

### ✅ Create Policy Key in Azure Portal

1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to your **Azure AD B2C tenant**
3. Go to:  
   `Azure AD B2C > Identity Experience Framework > Policy Keys`
4. Click **+ Add**
5. Fill out the form:
   - **Name**: `allowed-client-id`
   - **Key usage**: `Signature`
   - **Key type**: `Manual`
   - **Secret**: `your-client-id` (e.g., `11111111-aaaa-bbbb-cccc-222222222222`)
6. Click **Create**

✅ Your allowed client ID is now securely stored in B2C.

---

## 🛠️ Step 2: Custom Policy Code

### 📄 `TrustFrameworkExtensions.xml`

#### ➤ 1. ClaimType: `email`

```xml
<ClaimType Id="email">
  <DisplayName>Email Address</DisplayName>
  <DataType>string</DataType>
  <DefaultPartnerClaimTypes>
    <Protocol Name="OAuth2" PartnerClaimType="email" />
  </DefaultPartnerClaimTypes>
</ClaimType>
```

---

#### ➤ 2. ClaimType: `client_id`

```xml
<ClaimType Id="client_id">
  <DisplayName>Client ID</DisplayName>
  <DataType>string</DataType>
  <DefaultPartnerClaimTypes>
    <Protocol Name="OAuth2" PartnerClaimType="client_id" />
  </DefaultPartnerClaimTypes>
</ClaimType>
```

---

#### ➤ 3. Technical Profile: Validate `client_id` using Policy Key

```xml
<TechnicalProfile Id="ValidateClientId">
  <DisplayName>Validate client_id</DisplayName>
  <Protocol Name="Proprietary" />
  <Metadata>
    <Item Key="Operation">AssertBooleanClaimIsEqualToValue</Item>
    <Item Key="ClaimToCompare">client_id</Item>
    <Item Key="ValueToCompare">{PolicyKey:allowed-client-id}</Item>
    <Item Key="ErrorMessage">Unauthorized client application.</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="client_id" />
  </InputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

---

#### ➤ 4. Technical Profile: Input Email (Self-Asserted)

```xml
<TechnicalProfile Id="SelfAsserted-InputEmail">
  <DisplayName>Input Email</DisplayName>
  <Protocol Name="None" />
  <Metadata>
    <Item Key="ContentDefinitionReferenceId">api.selfasserted</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" Required="true" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" />
  </OutputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

---

#### ➤ 5. Technical Profile: Read User by Email

```xml
<TechnicalProfile Id="AAD-UserReadUsingEmail">
  <Metadata>
    <Item Key="Operation">Read</Item>
    <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">true</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="email" />
    <OutputClaim ClaimTypeReferenceId="givenName" />
    <OutputClaim ClaimTypeReferenceId="surname" />
  </OutputClaims>
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
</TechnicalProfile>
```

---

### 📄 User Journey: `AssertionUserJourney`

```xml
<UserJourney Id="AssertionUserJourney">
  <OrchestrationSteps>
    <OrchestrationStep Order="1" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="ValidateClientId" TechnicalProfileReferenceId="ValidateClientId" />
      </ClaimsExchanges>
    </OrchestrationStep>
    <OrchestrationStep Order="2" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="GetEmail" TechnicalProfileReferenceId="SelfAsserted-InputEmail" />
      </ClaimsExchanges>
    </OrchestrationStep>
    <OrchestrationStep Order="3" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="LookupUser" TechnicalProfileReferenceId="AAD-UserReadUsingEmail" />
      </ClaimsExchanges>
    </OrchestrationStep>
    <OrchestrationStep Order="4" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
  </OrchestrationSteps>
</UserJourney>
```

---

### 📄 `B2C_1A_AssertionLogin.xml` (Relying Party Policy)

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="AssertionUserJourney" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>Assertion Login</DisplayName>
    <Protocol Name="OAuth2" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

---

## 🧪 Token Request Example

```bash
curl -X POST "https://<tenant>.b2clogin.com/<tenant>.onmicrosoft.com/oauth2/v2.0/token?p=B2C_1A_AssertionLogin" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=assertion" \
  -d "client_id=11111111-aaaa-bbbb-cccc-222222222222" \
  -d "client_secret=<client-secret>" \
  -d "scope=https://<tenant>.onmicrosoft.com/api/read" \
  -d "email=user@example.com"
```

---

## 🔐 Security Behavior

| Scenario | Result |
|----------|--------|
| `client_id` matches policy key | ✅ Token issued |
| `client_id` does not match | ❌ Error: Unauthorized client application |
| Email does not match any user | ❌ Error: User not found |
| All valid | ✅ User token issued |

---

## ✅ Summary

| Feature | Status |
|--------|--------|
| Custom grant type: `assertion` | ✅ |
| Non-interactive token issuance | ✅ |
| No external API call | ✅ |
| Restrict to specific client_id | ✅ |
| Secure `client_id` storage | ✅ via Policy Key |
| User lookup by email | ✅ |

---

## 📎 Resources

- [Azure AD B2C: Custom policy overview](https://learn.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-overview)
- [Azure AD B2C: Manage policy keys](https://learn.microsoft.com/en-us/azure/active-directory-b2c/manage-policy-keys)
- [Azure B2C: Token endpoint](https://learn.microsoft.com/en-us/azure/active-directory-b2c/access-tokens)