# 🔐 Federating Azure AD B2C with Auth0 using OIDC and SAML (Custom Policies)

This guide shows how to integrate **Auth0** with **Azure AD B2C** using custom policies, supporting both:

- ✅ **OpenID Connect (OIDC)**
- ✅ **SAML 2.0**

---

## 📌 Table of Contents

- [OIDC Integration](#1-auth0-as-oidc-provider)
  - [Configure Auth0](#a-configure-auth0-oidc-app)
  - [Configure Azure AD B2C](#b-configure-azure-ad-b2c-custom-policy-for-oidc)
- [SAML Integration](#2-auth0-as-saml-identity-provider)
  - [Configure Auth0](#a-configure-auth0-saml-app)
  - [Configure Azure AD B2C](#b-configure-azure-ad-b2c-custom-policy-for-saml)
- [Testing and Debugging](#3-testing-the-integration)
- [Resources](#resources)

---

## 1. ✅ Auth0 as OIDC Provider

### A. Configure Auth0 OIDC App

1. Log into your **Auth0 dashboard**.
2. Go to **Applications** > **Applications** > **Create Application**.
3. Choose Application Type: **Regular Web Application**.
4. Set:
   - **Name**: Azure AD B2C OIDC
   - **Callback URL**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/oauth2/authresp
     ```
5. Save and note the:
   - **Client ID**
   - **Client Secret**
   - **Issuer URL** (e.g., `https://<your-auth0-domain>/`)

6. Enable **OIDC Conformant** in the app settings if not already enabled.

---

### B. Configure Azure AD B2C Custom Policy for OIDC

#### Add Technical Profile in `TrustFrameworkExtensions.xml`

```xml
<ClaimsProvider>
  <Domain>Auth0</Domain>
  <DisplayName>Auth0 OIDC</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Auth0-OIDC">
      <DisplayName>Login with Auth0</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <Metadata>
        <Item Key="client_id">your-auth0-client-id</Item>
        <Item Key="response_types">code</Item>
        <Item Key="scope">openid profile email</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">false</Item>
        <Item Key="issuer">https://<your-auth0-domain>/</Item>
        <Item Key="metadata_url">https://<your-auth0-domain>/.well-known/openid-configuration</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_Auth0ClientSecret" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="Auth0" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-OIDC" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### Session Management Profile

```xml
<TechnicalProfile Id="SM-OIDC">
  <DisplayName>OIDC Session</DisplayName>
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

#### Upload the Client Secret

```bash
az ad b2c policy key set \
  --name B2C_1A_Auth0ClientSecret \
  --policy-name TrustFrameworkExtensions \
  --key-type Secret \
  --key-usage Signature \
  --value "<your-auth0-client-secret>"
```

---

## 2. ✅ Auth0 as SAML Identity Provider

### A. Configure Auth0 SAML App

1. In the **Auth0 dashboard**, go to **Applications** > **Enterprise** > **SAMLP Identity Provider**.
2. Create a new SAML connection.
3. Configure:
   - **Callback URL (ACS)**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>/samlp/sso/login
     ```
   - **Audience URI (Entity ID)**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>
     ```
4. Under **Settings**, configure attribute mappings:
   ```json
   {
     "email": "email",
     "given_name": "given_name",
     "family_name": "family_name"
   }
   ```
5. Activate the connection and download the **metadata XML** and **X.509 certificate**.

---

### B. Configure Azure AD B2C Custom Policy for SAML

#### Add Technical Profile in `TrustFrameworkExtensions.xml`

```xml
<ClaimsProvider>
  <Domain>Auth0</Domain>
  <DisplayName>Auth0 SAML</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Auth0-SAML">
      <DisplayName>Login with Auth0 (SAML)</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
        <Item Key="PartnerEntity">urn:auth0:saml:provider</Item>
        <Item Key="SingleSignOnServiceUrl">https://<your-auth0-domain>/login/callback?connection=<saml-connection-name></Item>
        <Item Key="WantsEncryptedAssertions">false</Item>
        <Item Key="AllowUnsolicitedAuthResponses">true</Item>
        <Item Key="IdpMetadata">[Paste Auth0 metadata XML here]</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="SamlIdpSigningCert" StorageReferenceId="B2C_1A_Auth0SamlCert" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="Auth0" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-SAML" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### Session Management Profile

```xml
<TechnicalProfile Id="SM-SAML">
  <DisplayName>Session Management</DisplayName>
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

#### Upload Auth0 Signing Certificate

```bash
az ad b2c policy key set \
  --name B2C_1A_Auth0SamlCert \
  --policy-name TrustFrameworkExtensions \
  --key-type X509Certificate \
  --key-usage Verify \
  --value "<auth0-certificate-base64>"
```

---

## 3. 🔄 Update User Journey for Either Flow

In your relying party policy (e.g., `SignUpOrSignin.xml`):

```xml
<OrchestrationStep Order="1" Type="ClaimsExchange">
  <ClaimsExchanges>
    <ClaimsExchange Id="Auth0OIDCExchange" TechnicalProfileReferenceId="Auth0-OIDC" />
    <!-- or -->
    <ClaimsExchange Id="Auth0SAMLExchange" TechnicalProfileReferenceId="Auth0-SAML" />
  </ClaimsExchanges>
</OrchestrationStep>
```

---

## 4. 🧪 Testing the Integration

- Upload your custom policies to Azure AD B2C.
- Use the **"Run now"** feature or integrate with your app.
- You should be redirected to **Auth0** (OIDC or SAML).
- After login, the user is directed back to Azure AD B2C and then to your app.

---

## 📚 Resources

- 🔗 [Auth0 Docs - OIDC](https://auth0.com/docs/protocols/

---
