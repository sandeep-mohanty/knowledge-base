# 🛡️ Guide: Setup SAML-based Federation Between Azure AD B2C and Microsoft Entra ID (Azure AD) with OIDC Application

## 📘 Scenario Overview

This guide configures:

- **Application ↔ Azure AD B2C**: Communication via **OIDC (OpenID Connect)**
- **Azure AD B2C ↔ Microsoft Entra ID (Azure AD)**: Communication via **SAML**
- The login journey and federation logic is handled using **Azure AD B2C Custom Policies**

---

## ✅ Prerequisites

- Azure AD B2C tenant with **custom policies** enabled
- Microsoft Entra ID (Azure AD) tenant with users
- Application registered in Azure AD B2C using **OIDC**
- Certificate for **SAML signing**
- Admin access to both Azure B2C and Azure AD tenants

---

## 1️⃣ Set Up Azure AD B2C Custom Policies

### 1.1 Upload Base IEF Policy Files

Upload the starter pack from Microsoft:

- `TrustFrameworkBase.xml`
- `TrustFrameworkExtensions.xml`
- `SignUpOrSignin.xml`

Follow this guide:

📖 https://learn.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-get-started

---

## 2️⃣ Generate or Upload SAML Signing Certificate

Create a certificate (PFX file) or use a Key Vault reference.

### 2.1 Upload Certificate as Policy Key

In Azure AD B2C:

- Go to **Identity Experience Framework** > **Policy Keys**
- Click **+ Add**
- Choose **Upload**
- Name it `SamlSigningCert`
- Select **Use in SAML Token Signing**

---

## 3️⃣ Configure Microsoft Entra ID (Azure AD) as a SAML IdP

### 3.1 Register B2C as a SAML-based Enterprise Application

In Microsoft Entra ID (Azure AD):

1. Go to **Enterprise Applications**
2. Click **+ New Application**
3. Click **Create your own application**
4. Name: `AzureB2C-SAML-Federation`
5. Choose **Integrate any other application (non-gallery)**

---

### 3.2a Configure SAML SSO Settings (Manually)

In the new Enterprise App:

1. Go to **Single sign-on** > **SAML**
2. Set the following:

| Setting                  | Value |
|--------------------------|-------|
| **Identifier (Entity ID)** | `https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>` |
| **Reply URL (ACS URL)**    | `https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>/samlp/sso/assertionconsumer` |

3. Upload the **B2C SAML signing certificate**
4. Save the configuration

---

### 3.2b Configure SAML SSO Using Metadata URL

1. In the new app, go to **Single sign-on** > **SAML**
2. Select **Upload metadata file** or **Enter metadata URL**

3. Paste this metadata URL:

```plaintext
https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>/samlp/metadata
```

> 🔁 Replace `<your-b2c-tenant>` and `<policy-name>` accordingly.

--- 

### 3.3 Download Federation Metadata XML

From the SAML SSO configuration page, download the **Federation Metadata XML** file. You’ll use this in the next step.

---

## 4️⃣ Configure the SAML IdP in B2C Custom Policies

### 4.1 Extract Settings from Federation Metadata

From the downloaded metadata, extract:

- **Entity ID** (e.g. `https://sts.windows.net/{tenant-id}/`)
- **SSO URL** (e.g. `https://login.microsoftonline.com/{tenant-id}/saml2`)
- **X509 Certificate** (Base64)

---

### 4.2 Add Claims Provider in `TrustFrameworkExtensions.xml`

Inside the `<ClaimsProviders>` section:

```xml
<ClaimsProvider>
  <Domain>aadfederated</Domain>
  <DisplayName>Contoso Azure AD</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SAML-AAD-Federation">
      <DisplayName>Sign in with Azure AD</DisplayName>
      <Protocol Name="SAML2"/>
      <Metadata>
        <Item Key="PartnerEntity">https://sts.windows.net/{your-tenant-id}/</Item>
        <Item Key="WantsEncryptedAssertions">false</Item>
        <Item Key="SingleSignOnServiceUrl">https://login.microsoftonline.com/{your-tenant-id}/saml2</Item>
        <Item Key="IdpInitiatedProfileEnabled">true</Item>
        <Item Key="AllowUnsolicitedAuthResponses">true</Item>
        <Item Key="AuthenticationRequestSigningBehavior">Always</Item>
        <Item Key="IssuerUri">https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SamlSigningCert" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="ContosoAAD" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-SAML"/>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

---

### 4.3 Add Session Management Profile

Still under `<TechnicalProfiles>`:

```xml
<TechnicalProfile Id="SM-SAML">
  <DisplayName>Session Management SAML</DisplayName>
  <Protocol Name="SAML2"/>
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="samlToken" />
  </PersistedClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="samlToken" />
  </OutputClaims>
</TechnicalProfile>
```

---

## 5️⃣ Update User Journey to Include SAML Federation

In your `SignUpOrSignin.xml` or similar file:

### 5.1 Add ClaimsProviderSelection Step

```xml
<OrchestrationStep Order="1" Type="ClaimsProviderSelection" ContentDefinitionReferenceId="api.idpselections">
  <ClaimsProviderSelections>
    <ClaimsProviderSelection TargetClaimsExchangeId="SAML-AAD-Exchange" />
  </ClaimsProviderSelections>
</OrchestrationStep>
```

### 5.2 Add Claims Exchange Step for SAML

```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
      <Value>authenticationSource</Value>
      <Value>socialIdpAuthentication</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="SAML-AAD-Exchange" TechnicalProfileReferenceId="SAML-AAD-Federation" />
  </ClaimsExchanges>
</OrchestrationStep>
```

---

## 6️⃣ Register OIDC Application in Azure B2C

1. Go to **Azure AD B2C > App registrations**
2. Register your application
3. Configure:
   - **Redirect URI**: e.g., `https://yourapp.com/auth/callback`
   - **Supported account types**: Accounts in this org only
   - **Token configuration**: Add `email`, `given_name`, `surname`
4. Under **Authentication**, enable **ID tokens** (v1 or v2)

---

## 7️⃣ Upload and Test Custom Policies

1. Upload the updated policy files:
   - `TrustFrameworkBase.xml`
   - `TrustFrameworkExtensions.xml`
   - `SignUpOrSignin.xml`
2. Go to **Identity Experience Framework**
3. Run the policy using **Run user flow**
4. Choose the B2C app (OIDC relying party)
5. You’ll be redirected to **Microsoft Entra ID login**
6. Upon success, B2C issues an **OIDC token** to your application

---

## 🔍 Troubleshooting

- Use **Application Insights** logs in B2C for debugging
- Use browser tools like **SAML-tracer** or **Fiddler** to inspect SAML requests/responses
- Ensure time sync between B2C and Entra ID
- Verify that the correct **ACS URL** is configured in Entra ID

---

## 🔐 Security Recommendations

- Use a **valid certificate** for signing assertions
- Rotate certificates periodically
- Enable **SAML request signing** in Entra ID
- Restrict the SAML app to known users/groups in Azure AD

---