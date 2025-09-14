# 🔐 Federating Azure AD B2C with Okta using OIDC and SAML (Custom Policies)

This guide explains how to integrate **Okta** with **Azure AD B2C** using **custom policies (Identity Experience Framework)**.

You’ll learn how to set up federation in two ways:

- ✅ Using **OpenID Connect (OIDC)**
- ✅ Using **SAML 2.0**

---

## 📌 Table of Contents

- [OIDC Integration](#1-okta-as-oidc-provider)
  - [Configure Okta](#a-configure-okta-oidc-app)
  - [Configure Azure AD B2C](#b-configure-azure-ad-b2c-custom-policy-for-oidc)
- [SAML Integration](#2-okta-as-saml-identity-provider)
  - [Configure Okta](#a-configure-okta-saml-app)
  - [Configure Azure AD B2C](#b-configure-azure-ad-b2c-custom-policy-for-saml)
- [Testing and Debugging](#3-testing-the-integration)
- [Resources](#resources)

---

## 1. ✅ Okta as OIDC Provider

### A. Configure Okta OIDC App

1. Go to **Okta Developer Console**.
2. Navigate to **Applications** > **Create App Integration**.
3. Choose:
   - **Sign-in method**: OIDC - OpenID Connect
   - **Application type**: Web App
4. Provide:
   - **Name**: `Azure AD B2C OIDC`
   - **Sign-in redirect URIs**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/oauth2/authresp
     ```
   - **Sign-out redirect URI** (optional)
5. Save and **note down**:
   - Client ID
   - Client Secret
   - Issuer URL (e.g., `https://dev-123456.okta.com/oauth2/default`)

---

### B. Configure Azure AD B2C Custom Policy for OIDC

#### Add Technical Profile in `TrustFrameworkExtensions.xml`

```xml
<ClaimsProvider>
  <Domain>Okta</Domain>
  <DisplayName>Okta OIDC</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Okta-OIDC">
      <DisplayName>Login with Okta</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <Metadata>
        <Item Key="client_id">your-okta-client-id</Item>
        <Item Key="response_types">code</Item>
        <Item Key="scope">openid profile email</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">false</Item>
        <Item Key="issuer">https://dev-123456.okta.com/oauth2/default</Item>
        <Item Key="metadata_url">https://dev-123456.okta.com/oauth2/default/.well-known/openid-configuration</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_OktaClientSecret" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="Okta" />
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
  --name B2C_1A_OktaClientSecret \
  --policy-name TrustFrameworkExtensions \
  --key-type Secret \
  --key-usage Signature \
  --value "<your-okta-client-secret>"
```

---

## 2. ✅ Okta as SAML Identity Provider

### A. Configure Okta SAML App

1. In Okta, go to **Applications** > **Create App Integration**.
2. Choose **SAML 2.0**.
3. Configure:
   - **Single Sign-On URL**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>/samlp/sso/login
     ```
   - **Audience URI (SP Entity ID)**:
     ```
     https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>
     ```
4. Configure attributes (email, given_name, family_name).
5. Save and download the **metadata XML**.

---

### B. Configure Azure AD B2C Custom Policy for SAML

#### Add Technical Profile in `TrustFrameworkExtensions.xml`

```xml
<ClaimsProvider>
  <Domain>Okta</Domain>
  <DisplayName>Okta SAML</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Okta-SAML">
      <DisplayName>Login with Okta (SAML)</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
        <Item Key="PartnerEntity">http://www.okta.com/exk123456789</Item>
        <Item Key="SingleSignOnServiceUrl">https://dev-123456.okta.com/app/okta_id/sso/saml</Item>
        <Item Key="WantsEncryptedAssertions">false</Item>
        <Item Key="AllowUnsolicitedAuthResponses">true</Item>
        <Item Key="IdpMetadata">[Paste Okta metadata XML here]</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="SamlIdpSigningCert" StorageReferenceId="B2C_1A_OktaSamlCert" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="Okta" />
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

#### Upload Okta Signing Certificate

```bash
az ad b2c policy key set \
  --name B2C_1A_OktaSamlCert \
  --policy-name TrustFrameworkExtensions \
  --key-type X509Certificate \
  --key-usage Verify \
  --value "<okta-certificate-base64>"
```

---

## 3. 🔄 Update User Journey for Either Flow

In your relying party (e.g., `SignUpOrSignin.xml`), update orchestration steps like below:

```xml
<OrchestrationStep Order="1" Type="ClaimsExchange">
  <ClaimsExchanges>
    <ClaimsExchange Id="OktaOIDCExchange" TechnicalProfileReferenceId="Okta-OIDC" />
    <!-- or -->
    <ClaimsExchange Id="OktaSAMLExchange" TechnicalProfileReferenceId="Okta-SAML" />
  </ClaimsExchanges>
</OrchestrationStep>
```

---

## 4. 🧪 Testing the Integration

- Use the **“Run now”** feature in Azure AD B2C to test your custom policy.
- You should be redirected to Okta (OIDC or SAML) for login.
- Upon successful authentication, Azure AD B2C issues a token to your application.

---  