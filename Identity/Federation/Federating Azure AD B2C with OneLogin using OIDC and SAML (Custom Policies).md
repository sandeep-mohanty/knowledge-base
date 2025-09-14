# 🔐 Federating Azure AD B2C with OneLogin using OIDC and SAML (Custom Policies)

This guide demonstrates how to integrate **OneLogin** with **Azure AD B2C** using **custom policies** in two different federation protocols:

- ✅ **OpenID Connect (OIDC)**
- ✅ **SAML 2.0**

---

## 📌 Table of Contents

- [OIDC Integration](#1-onelogin-as-oidc-provider)
  - [Configure OneLogin](#a-configure-onelogin-oidc-app)
  - [Configure Azure AD B2C](#b-configure-azure-ad-b2c-custom-policy-for-oidc)
- [SAML Integration](#2-onelogin-as-saml-identity-provider)
  - [Configure OneLogin](#a-configure-onelogin-saml-app)
  - [Configure Azure AD B2C](#b-configure-azure-ad-b2c-custom-policy-for-saml)
- [Testing and Debugging](#3-testing-the-integration)
- [Resources](#resources)

---

## 1. ✅ OneLogin as OIDC Provider

### A. Configure OneLogin OIDC App

1. Log into your **OneLogin Admin Portal**.
2. Go to **Applications** > **Add App**.
3. Search for **"OpenID Connect"** and choose **"OpenID Connect (OIDC)"**.
4. Set:
   - **Display Name**: `Azure AD B2C OIDC`
   - **Redirect URI**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/oauth2/authresp
     ```
5. Go to the **SSO tab** and note:
   - **Client ID**
   - **Client Secret**
   - **Issuer URL**:
     ```
     https://<your-subdomain>.onelogin.com/oidc/2
     ```
   - **Well-known configuration URL**:
     ```
     https://<your-subdomain>.onelogin.com/oidc/2/.well-known/openid-configuration
     ```

---

### B. Configure Azure AD B2C Custom Policy for OIDC

#### Add Technical Profile in `TrustFrameworkExtensions.xml`

```xml
<ClaimsProvider>
  <Domain>OneLogin</Domain>
  <DisplayName>OneLogin OIDC</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="OneLogin-OIDC">
      <DisplayName>Login with OneLogin</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <Metadata>
        <Item Key="client_id">your-onelogin-client-id</Item>
        <Item Key="response_types">code</Item>
        <Item Key="scope">openid profile email</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">false</Item>
        <Item Key="issuer">https://<your-subdomain>.onelogin.com/oidc/2</Item>
        <Item Key="metadata_url">https://<your-subdomain>.onelogin.com/oidc/2/.well-known/openid-configuration</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_OneLoginClientSecret" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="OneLogin" />
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
  --name B2C_1A_OneLoginClientSecret \
  --policy-name TrustFrameworkExtensions \
  --key-type Secret \
  --key-usage Signature \
  --value "<your-onelogin-client-secret>"
```

---

## 2. ✅ OneLogin as SAML Identity Provider

### A. Configure OneLogin SAML App

1. In OneLogin, go to **Applications** > **Add App**.
2. Search for and select **SAML Custom Connector (Advanced)**.
3. Under **Configuration**, set:
   - **ACS (Consumer) URL**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>/samlp/sso/login
     ```
   - **Audience (Entity ID)**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>
     ```
4. Under **Parameters**, configure attributes:
   - `email`
   - `first_name` → `givenName`
   - `last_name` → `surname`
5. Under **SSO tab**, download the **X.509 certificate** and **metadata XML**.

---

### B. Configure Azure AD B2C Custom Policy for SAML

#### Add Technical Profile in `TrustFrameworkExtensions.xml`

```xml
<ClaimsProvider>
  <Domain>OneLogin</Domain>
  <DisplayName>OneLogin SAML</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="OneLogin-SAML">
      <DisplayName>Login with OneLogin (SAML)</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
        <Item Key="PartnerEntity">https://app.onelogin.com/saml/metadata/<app-id></Item>
        <Item Key="SingleSignOnServiceUrl">https://<your-subdomain>.onelogin.com/trust/saml2/http-post/sso/<app-id></Item>
        <Item Key="WantsEncryptedAssertions">false</Item>
        <Item Key="AllowUnsolicitedAuthResponses">true</Item>
        <Item Key="IdpMetadata">[Paste OneLogin metadata XML here]</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="SamlIdpSigningCert" StorageReferenceId="B2C_1A_OneLoginSamlCert" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="first_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="last_name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="OneLogin" />
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

#### Upload OneLogin Signing Certificate

```bash
az ad b2c policy key set \
  --name B2C_1A_OneLoginSamlCert \
  --policy-name TrustFrameworkExtensions \
  --key-type X509Certificate \
  --key-usage Verify \
  --value "<onelogin-certificate-base64>"
```

---

## 3. 🔄 Update User Journey for Either Flow

In your relying party policy (e.g., `SignUpOrSignin.xml`), update orchestration steps:

```xml
<OrchestrationStep Order="1" Type="ClaimsExchange">
  <ClaimsExchanges>
    <ClaimsExchange Id="OneLoginOIDCExchange" TechnicalProfileReferenceId="OneLogin-OIDC" />
    <!-- or -->
    <ClaimsExchange Id="OneLoginSAMLExchange" TechnicalProfileReferenceId="OneLogin-SAML" />
  </ClaimsExchanges>
</OrchestrationStep>
```

---

## 4. 🧪 Testing the Integration

- Upload your policies to Azure AD B2C.
- Use the **“Run now”** feature

---