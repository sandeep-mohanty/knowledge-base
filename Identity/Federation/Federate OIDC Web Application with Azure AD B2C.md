# Step-by-Step Guide: Federate an Application with Azure AD B2C Using Custom Policies (OIDC)

This guide explains how to federate an application with an Azure AD B2C tenant using **custom policies** and the **OpenID Connect (OIDC)** protocol. The application and custom policies reside in the **same Azure AD B2C tenant**.

---
## Prerequisites
1. **Azure AD B2C Tenant**: Ensure you have an Azure AD B2C tenant created.
2. **Application Registration**:
   - Register your application in Azure AD B2C:
     - Go to **Azure AD B2C** > **App registrations** > **New registration**.
     - Set a name (e.g., `MyApp`).
     - Add a **Redirect URI** (e.g., `https://your-app.com/auth/callback`).
     - Note the **Application (client) ID**.
3. **Client Secret**:
   - Under your app registration, go to **Certificates & secrets** > **New client secret**.
   - Save the secret value (it will not be shown again).

---

## Step 1: Prepare Custom Policy Files
Custom policies are XML files that define the authentication flow. You need three files:
1. **TrustFrameworkBase.xml**: Base configuration.
2. **TrustFrameworkExtensions.xml**: Extensions for your tenant.
3. **SignUpSignIn.xml**: Relying Party (RP) policy for your application.

### Download Starter Pack
1. Clone or download the [Azure AD B2C Custom Policy Starter Pack](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack).
2. Use the **LocalAccounts** starter pack for OIDC federation.

---
## Step 2: Configure OIDC Technical Profile
In the custom policy files, define an OIDC technical profile to federate with Azure AD B2C itself.

### Update `TrustFrameworkExtensions.xml`
Add the following XML snippet to configure the OIDC provider (replace placeholders with your values):

```xml
<ClaimsProvider>
  <Domain>your-tenant-name.onmicrosoft.com</Domain>
  <DisplayName>Azure AD B2C OIDC</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="OIDC-AzureADB2C">
      <DisplayName>Azure AD B2C</DisplayName>
      <Description>Login with your Azure AD B2C account</Description>
      <Protocol Name="OpenIdConnect" />
      <Metadata>
        <Item Key="METADATA">https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=B2C_1A_signup_signin</Item>
        <Item Key="client_id">Your-Application-Client-ID</Item>
        <Item Key="response_types">code</Item>
        <Item Key="scope">openid profile api://your-api-client-id/api.read</Item>
        <Item Key="response_mode">form_post</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_ClientSecret" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
      </OutputClaims>
      <OutputClaimsTransformations>
        <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
        <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
        <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromObjectID" />
      </OutputClaimsTransformations>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

### Replace Placeholders :
- your-tenant-name: Your Azure AD B2C tenant name.
- Your-Application-Client-ID: Client ID of your app registration.
- B2C_1A_signup_signin: Name of the policy you want to use.

---
## Step 3: Add Client Secret to Policy Keys

1. In Azure AD B2C, go to Identity Experience Framework > Policy keys.
2. Create a new key:
  * **Name** : `B2C_1A_ClientSecret`
  * **Secret** : Paste the client secret from your app registration.
  * **Key usage** : `Encryption`.
---

## Step 4: Configure Relying Party (RP) Policy
Update the SignUpSignIn.xml file to reference your OIDC technical profile.

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```
---
## Step 5: Upload Custom Policies
1. In Azure AD B2C, go to Identity Experience Framework > Upload Custom Policy.
2. Upload the files in this order:
    - `TrustFrameworkBase.xml`
    - `TrustFrameworkExtensions.xml`
    - `SignUpSignIn.xml`

---
## Step 6: Configure Application to Use Custom Policy
1. In your app registration, go to Authentication .
2. Under Implicit grant and hybrid flows , enable:
    - ID tokens (for implicit flow).
    - Access tokens (for implicit flow).
4. Expose an API scope :
    - Under Expose an API , add a scope (e.g., api.read).
5. Grant API permissions to your application:
    - In your application’s registration, go to API permissions .
    - Add the API scope you created (e.g., api://your-api-client-id/api.read).

---
## Step 7: Test the Configuration
1. Go to Azure AD B2C > User flows .
2. Select your custom policy (e.g., B2C_1A_signup_signin).
3. Click Run user flow and test the login experience.