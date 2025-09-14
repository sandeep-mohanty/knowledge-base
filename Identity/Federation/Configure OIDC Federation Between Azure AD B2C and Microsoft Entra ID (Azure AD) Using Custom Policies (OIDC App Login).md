# 🛡️ Guide: Configure OIDC Federation Between Azure AD B2C and Microsoft Entra ID (Azure AD) Using Custom Policies (OIDC App Login)

## 📘 Scenario Overview

This guide configures:

- **Application ↔ Azure AD B2C**: Communication via **OIDC (OpenID Connect)**
- **Azure AD B2C ↔ Microsoft Entra ID (Azure AD)**: Communication also via **OIDC**
- The login journey and federation logic is handled using **Azure AD B2C Custom Policies**

---

## ✅ Prerequisites

- Azure AD B2C tenant with **custom policies** enabled
- Microsoft Entra ID (Azure AD) tenant with users
- Application registered in Azure AD B2C using **OIDC**
- Admin access to both Azure AD B2C and Microsoft Entra ID
- A test user in Microsoft Entra ID

---

## 1️⃣ Set Up Azure AD B2C Custom Policies

### 1.1 Upload Base IEF Policy Files

Use the Microsoft starter pack to upload:

- `TrustFrameworkBase.xml`
- `TrustFrameworkExtensions.xml`
- `SignUpOrSignin.xml`

📖 [Start with custom policies](https://learn.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-get-started)

---

## 2️⃣ Register Azure AD B2C as an App in Microsoft Entra ID

This allows B2C to authenticate users via Entra ID using **OIDC**.

### 2.1 Register Application in Entra ID

In Microsoft Entra ID:

1. Go to **App registrations** > **+ New registration**
2. Name: `AzureB2C-OIDC-Federation`
3. Supported account types: `Accounts in this organizational directory only`
4. Redirect URI (Web):
   ```
   https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/oauth2/authresp
   ```

5. Click Register

---

### 2.2 Configure API Permissions

1. Go to the registered app
2. Under **API permissions**, add:
   - Microsoft Graph > `openid`, `email`, `profile`
3. Click **Grant admin consent**

---

### 2.3 Create a Client Secret

1. Go to **Certificates & secrets**
2. Click **+ New client secret**
3. Copy the generated secret — you’ll use it in B2C

---

## 3️⃣ Configure Microsoft Entra ID as an OIDC IdP in B2C

### 3.1 Add Claims Provider in `TrustFrameworkExtensions.xml`

Under `<ClaimsProviders>`, add:

```xml
<ClaimsProvider>
  <Domain>entra</Domain>
  <DisplayName>Microsoft Entra ID</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="OIDC-EntraID">
      <DisplayName>Sign in with Microsoft Entra ID</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <Metadata>
        <Item Key="METADATA">https://login.microsoftonline.com/<entra-tenant-id>/v2.0/.well-known/openid-configuration</Item>
        <Item Key="client_id">[Enter the client_id from Entra App]</Item>
        <Item Key="response_types">code</Item>
        <Item Key="scope">openid profile email</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_EntraClientSecret" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="EntraID" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-OIDC" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

Replace:
- `<entra-tenant-id>` with your Entra tenant ID
- `client_id` with the application ID from Entra
- Store the client secret in B2C as a key (see below)

---

### 3.2 Create a Policy Key for Client Secret

In Azure AD B2C:

- Go to **Identity Experience Framework** > **Policy Keys**
- Click **+ Add**
- Type: **Manual**
- Name: `B2C_1A_EntraClientSecret`
- Secret: Paste the client secret from Entra

---

### 3.3 Add Session Management Profile

```xml
<TechnicalProfile Id="SM-OIDC">
  <DisplayName>Session Management OIDC</DisplayName>
  <Protocol Name="OpenIdConnect" />
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
    <OutputClaim ClaimTypeReferenceId="identityProvider" />
  </OutputClaims>
</TechnicalProfile>
```

---

## 4️⃣ Update User Journey to Include OIDC Federation

In your `SignUpOrSignin.xml`, update the user journey section:

### 4.1 Step: ClaimsProvider Selection

```xml
<OrchestrationStep Order="1" Type="ClaimsProviderSelection" ContentDefinitionReferenceId="api.idpselections">
  <ClaimsProviderSelections>
    <ClaimsProviderSelection TargetClaimsExchangeId="OIDC-Exchange" />
  </ClaimsProviderSelections>
</OrchestrationStep>
```

### 4.2 Step: Claims Exchange

```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
      <Value>authenticationSource</Value>
      <Value>localAccountAuthentication</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="OIDC-Exchange" TechnicalProfileReferenceId="OIDC-EntraID" />
  </ClaimsExchanges>
</OrchestrationStep>
```

---

## 5️⃣ Register OIDC Application in Azure B2C

Your app will communicate with Azure B2C using **OIDC**.

1. Go to **Azure AD B2C > App registrations**
2. Register your application
3. Configure:
   - **Redirect URI**: e.g., `https://yourapp.com/auth/callback`
   - **Supported account types**: Accounts in this org only
   - **Token configuration**: Add claims like `email`, `given_name`, `surname`
4. Under **Authentication**, enable **ID tokens**

---

## 6️⃣ Upload and Test Custom Policies

1. Upload the updated policy files:
   - `TrustFrameworkBase.xml`
   - `TrustFrameworkExtensions.xml`
   - `SignUpOrSignin.xml`
2. Go to **Identity Experience Framework**
3. Run the policy using **Run now**
4. Choose the OIDC app you registered
5. You’ll be redirected to Microsoft Entra ID to sign in
6. Upon success, B2C issues an **OIDC token** to your application

---

## 🔍 Troubleshooting Tips

- Use **Application Insights** to debug errors
- Use **browser dev tools** or **network trace** to inspect OIDC redirects
- Confirm that the redirect URI and client secret are correct
- Make sure you’ve granted **admin consent** in Entra ID

---

## 🔐 Security Best Practices

- Store client secrets in **Key Vault** or secure policy keys
- Enable **token lifetime policies** in Entra ID if needed
- Use **PKCE** for mobile/native apps
- Rotate secrets regularly

---

## 📚 References

- [Azure AD B2C custom policy overview](https://learn.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-overview)
- [Set up OpenID Connect identity provider in B2C](https://learn.microsoft.com/en-us/azure/active-directory-b2c/identity-provider-openid-connect)
- [Azure AD OIDC metadata endpoint](https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration)