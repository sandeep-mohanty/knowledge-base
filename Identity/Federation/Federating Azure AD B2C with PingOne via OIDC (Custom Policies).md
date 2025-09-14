# 🔐 Federating Azure AD B2C with PingOne via OIDC (Custom Policies)

This guide shows you how to configure **Azure AD B2C custom policies** to delegate user authentication to **PingOne** using **OpenID Connect (OIDC)** as the federation protocol.

---

## 🔁 Federation Flow Overview

1. User accesses your application → redirected to Azure AD B2C.
2. Azure AD B2C custom policy redirects to PingOne (OIDC IdP).
3. User authenticates on PingOne.
4. PingOne returns an ID token to Azure AD B2C.
5. Azure AD B2C issues a token to your application.

---

## 🧰 Prerequisites

- Azure AD B2C tenant with Identity Experience Framework (custom policies) enabled.
- PingOne environment with an **OIDC application** configured.
- Client ID, client secret, and discovery URL from PingOne.
- A valid redirect URI pointing to your Azure AD B2C policy.

---

## 🔧 Step-by-Step Implementation

### 1. ✅ Set Up PingOne OIDC Application

1. In **PingOne Admin Portal**:
   - Go to **Connections** > **Applications**.
   - Create a new **OIDC Web App**.

2. Configure the **Redirect URI** as:

   ```
   https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/oauth2/authresp
   ```

3. Note down:
   - **Client ID**
   - **Client Secret**
   - **Issuer URL / Discovery URL** (usually in the format: `https://auth.pingone.com/<env-id>/as/.well-known/openid-configuration`)

---

### 2. 🏗️ Create or Update B2C Custom Policies

You will configure the PingOne OIDC provider in your `TrustFrameworkExtensions.xml`.

#### a. Add the OIDC Technical Profile

```xml
<ClaimsProvider>
  <Domain>PingOne</Domain>
  <DisplayName>PingOne</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="PingOne-OIDC">
      <DisplayName>PingOne Login</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <Metadata>
        <Item Key="client_id">your-client-id</Item>
        <Item Key="response_types">code</Item>
        <Item Key="scope">openid profile email</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">false</Item>
        <Item Key="issuer">https://auth.pingone.com</Item>
        <Item Key="metadata_url">https://auth.pingone.com/<env-id>/as/.well-known/openid-configuration</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_PingOneClientSecret" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="PingOne" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-OIDC" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### b. Add the Session Management Profile

```xml
<TechnicalProfile Id="SM-OIDC">
  <DisplayName>OIDC Session Management</DisplayName>
  <Protocol Name="None" />
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="email" />
  </PersistedClaims>
  <SessionManagement>
    <SessionExpiryType>Rolling</SessionExpiryType>
    <SessionExpiryInSeconds>3600</SessionExpiryInSeconds>
  </SessionManagement>
</TechnicalProfile>
```

---

### 3. 🔐 Upload the Client Secret to Azure AD B2C

Use Azure CLI or PowerShell to store the PingOne client secret securely:

```bash
az ad b2c policy key set \
  --name B2C_1A_PingOneClientSecret \
  --policy-name TrustFrameworkExtensions \
  --key-type Secret \
  --key-usage Signature \
  --value "<your-client-secret>"
```

---

### 4. 🔄 Update the User Journey to Use PingOne

In your relying party policy or `TrustFrameworkExtensions.xml`, add or update an orchestration step:

```xml
<OrchestrationStep Order="1" Type="ClaimsExchange">
  <ClaimsExchanges>
    <ClaimsExchange Id="PingOneExchange" TechnicalProfileReferenceId="PingOne-OIDC" />
  </ClaimsExchanges>
</OrchestrationStep>
```

---

### 5. 🧪 Test the Login Flow

1. Upload the updated policies (`TrustFrameworkBase.xml`, `TrustFrameworkExtensions.xml`, etc.) to Azure AD B2C.
2. Test the custom policy via the **Run Now** feature or application integration.
3. You should be redirected to **PingOne** for authentication.
4. Upon successful login, B2C will issue a token to your app.

---

## 🧠 Tips & Considerations

- **Scopes**: Use `openid profile email` to get standard claims.
- **Discovery URL**: Make sure your PingOne discovery document is publicly accessible.
- **Mobile Apps**: OIDC is mobile-friendly and preferred over SAML.
- **Logout**: For federated logout support, implement OpenID Connect `end_session_endpoint`.

---

## ✅ Summary

| Component             | Role                                 |
|-----------------------|--------------------------------------|
| Azure AD B2C          | Acts as the **OIDC Relying Party**   |
| PingOne               | Acts as the **OIDC Identity Provider** |
| Federation Protocol   | OpenID Connect (OIDC)                |
| Policy Type           | Custom (Identity Experience Framework) |

---

## 🛠️ Optional: Need Help?

You can ask for:

- ✅ A full sample `TrustFrameworkExtensions.xml` file with PingOne OIDC preconfigured.
- ✅ A downloadable `.md` or `.xml` template.
- ✅ Help parsing PingOne metadata or setting up claims.

Let me know what you need!