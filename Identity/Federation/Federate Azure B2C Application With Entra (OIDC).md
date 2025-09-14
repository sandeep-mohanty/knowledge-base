# Federate Azure AD B2C with Entra ID (Azure AD) using Custom Policies (OIDC Protocol)

## Prerequisites
1. **Azure AD B2C Tenant**: An active Azure AD B2C tenant.
2. **Entra ID Tenant**: Access to the Entra ID tenant to federate with.
3. **Custom Policy Starter Pack**: Download from [Microsoft's GitHub](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack).

---

## Step 1: Register an Application in Entra ID

### Navigate to Entra ID
1. Go to the [Azure Portal](https://portal.azure.com) and switch to your Entra ID tenant.

### Register a New App
1. **App registrations** > **New registration**.
2. **Name**: e.g., `B2C-Entra-OIDC`.
3. **Redirect URI**: 
   - Select **Web**.
   - Enter:  
     `https://<your-b2c-tenant-name>.b2clogin.com/<your-b2c-tenant-id>/oauth2/authresp`  
     *(Replace `<your-b2c-tenant-name>` and `<your-b2c-tenant-id>` with your B2C tenant details).*

### Note Key Values
- Copy the **Application (Client) ID** and **Directory (Tenant) ID**.

### Create Client Secret
1. **Certificates & secrets** > **New client secret**.
2. Save the secret value (it will not be shown again).

---

## Step 2: Configure API Permissions in Entra ID

1. **API permissions** > **Add a permission** > **Microsoft Graph**.
2. Select **Delegated permissions**:
   - `openid`
   - `profile`
   - `email`
   - `User.Read`
3. **Grant admin consent** for the Entra tenant.

---

## Step 3: Set Up Policy Keys in Azure AD B2C

1. In the B2C tenant, go to **Identity Experience Framework** > **Policy keys**.
2. **Add** > **Manual**:
   - **Name**: `EntraSecret` (automatically prefixed with `B2C_1A_`).
   - **Secret**: Paste the Entra client secret from Step 1.
   - **Create**.

---

## Step 4: Modify Custom Policy Files

### Update `TrustFrameworkExtensions.xml`
Add the following `<ClaimsProvider>` under `<ClaimsProviders>`:

```xml
<ClaimsProvider>
  <Domain>entra.com</Domain>
  <DisplayName>Entra ID</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Entra-OIDC">
      <DisplayName>Entra ID</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <Metadata>
        <Item Key="METADATA">https://login.microsoftonline.com/{Entra-Tenant-ID}/v2.0/.well-known/openid-configuration</Item>
        <Item Key="client_id">{Entra-Client-ID}</Item>
        <Item Key="response_types">code</Item>
        <Item Key="scope">openid profile email</Item>
        <Item Key="response_mode">form_post</Item>
        <Item Key="HttpBinding">POST</Item>
        <Item Key="UsePolicyInRedirectUri">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="client_secret" StorageReferenceId="B2C_1A_EntraSecret" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid" />
        <OutputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="tid" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
        <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
        <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="entra.com" />
      </OutputClaims>
      <OutputClaimsTransformations>
        <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
        <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
        <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
        <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId" />
      </OutputClaimsTransformations>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```
### Update User Journey
Add the Entra technical profile to the sign-in journey in <UserJourneys>:

 ```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimEquals" ExecuteActionsIf="false">
      <Value>identityProvider</Value>
      <Value>entra.com</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="EntraExchange" TechnicalProfileReferenceId="Entra-OIDC" />
  </ClaimsExchanges>
</OrchestrationStep>
 ```
### Update Relying Party File (e.g., SignUpOrSignIn.xml)
Ensure the <DefaultUserJourney> references the updated journey.

## Step 5: Upload and Test Policies
### 1. Upload Policies :
* In B2C’s Identity Experience Framework , upload: <br>
    * TrustFrameworkBase.xml
    * TrustFrameworkExtensions.xml
    * Relying party file (e.g., SignUpOrSignIn.xml). <br>
* Test the Policy :
    * Use the `Run now` endpoint to test the sign-in flow. You should see an option to sign in with Entra ID.

