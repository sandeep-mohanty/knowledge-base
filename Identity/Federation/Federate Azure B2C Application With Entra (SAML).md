# Federate Azure AD B2C with Entra ID (Azure AD) using Custom Policies (SAML Protocol)

## Prerequisites
1. **Azure AD B2C Tenant**: An active Azure AD B2C tenant.
2. **Entra ID Tenant**: Access to the Entra ID tenant to federate with.
3. **Custom Policy Starter Pack**: Download from [Microsoft's GitHub](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack).
4. **Text Editor**: For modifying XML files (e.g., VS Code).

---

## Step 1: Register an Application in Entra ID

### Navigate to Entra ID
1. Go to the [Azure Portal](https://portal.azure.com) and switch to your Entra ID tenant.

### Register a New App
1. **App registrations** > **New registration**.
2. **Name**: e.g., `B2C-Entra-SAML`.
3. **Supported account types**: Select Accounts in this organizational directory only .
4. **Redirect URI**: 
   - Select **Web**.
   - Enter:  
     `https://<your-b2c-tenant-name>.b2clogin.com/<your-b2c-tenant-id>/samlp/sso/assertionconsumer`  
     *(Replace `<your-b2c-tenant-name>` and `<your-b2c-tenant-id>` with your B2C tenant details).*

### Note Key Values
- Copy the **Application (Client) ID** and **Directory (Tenant) ID**.

### Configure SAML Settings
1. **App registrations** > **Select your app** > **Expose an API** .
2. **Add a scope**: Create a placeholder scope (not used but required).
3. **Manifest** :
    * Set `"samlMetadataUrl"` to: <br>
`https://<your-b2c-tenant>.b2clogin.com/<your-b2c-tenant>.onmicrosoft.com/<policy-name>/Samlp/metadata`
    * Set `"identifierUris"` to: <br>
`["https://<your-b2c-tenant>.onmicrosoft.com/<your-b2c-tenant>.onmicrosoft.com"]`
---

## Step 2: Configure SAML Signing Certificate in Entra ID

1. **App registrations** > **Select your app** > **Certificates & secrets** .
2. **Federation metadata document**: <br>
    * Download the XML metadata from: <br>
`https://login.microsoftonline.com/<Entra-Tenant-ID>/federationmetadata/2007-06/federationmetadata.xml`
    * Extract the X.509 certificate (used later for B2C policy).
---

## Step 3: Set Up Policy Keys in Azure AD B2C

1. **Upload Entra ID Certificate** :
    * In B2C, go to Identity Experience Framework > Policy keys .
    * Add > Upload: <br>
        * **Name** : EntraSamlCert (automatically prefixed with B2C_1A_).
        * **Upload** : The X.509 certificate from Step 2.
    * Create .
---

## Step 4: Modify Custom Policy Files

### Update `TrustFrameworkExtensions.xml`
Add the following `<ClaimsProvider>` under `<ClaimsProviders>`:

```xml
<ClaimsProvider>
  <Domain>entra.com</Domain>
  <DisplayName>Entra ID (SAML)</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Entra-SAML">
      <DisplayName>Entra ID</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
        <Item Key="PartnerEntity">https://login.microsoftonline.com/<Entra-Tenant-ID>/federationmetadata/2007-06/federationmetadata.xml</Item>
        <Item Key="RequestsSigned">false</Item>
        <Item Key="WantsEncryptedAssertions">false</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_EntraSamlCert" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="assertion.subject.nameid" />
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname" />
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname" />
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="entra.com" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Saml" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```
### Update User Journey
Add the SAML technical profile to the sign-in journey in <UserJourneys>:

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
    <ClaimsExchange Id="EntraSamlExchange" TechnicalProfileReferenceId="Entra-SAML" />
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
    * Use the `Run now` endpoint to test the sign-in flow.  You should see an option to sign in with Entra ID via SAML.

