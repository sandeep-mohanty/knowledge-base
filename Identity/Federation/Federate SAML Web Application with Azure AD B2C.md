# Federate a SAML-Compliant Application with Azure AD B2C

## Prerequisites
1. **Azure AD B2C Tenant**: An active Azure AD B2C tenant.
2. **SAML Web Application** : A SAML-enabled application that will federate with Azure AD B2C tenant.
3. **Custom Policy Starter Pack**: Download from [Microsoft's GitHub](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack).
4. **Text Editor**: For modifying XML files (e.g., VS Code).
5. **SAML Metadata (Optional)**: Obtain the SP’s metadata URL or XML file (if available).

---

## Step 1: Register the SAML Application in Entra ID

### Navigate to Entra ID
1. Go to the [Azure Portal](https://portal.azure.com) and switch to your Entra ID tenant.

### Register a New App
1. **App registrations** > **New registration**.
2. **Name**: e.g., `SAML-WebApp`.
3. **Supported account types**: Select Accounts in this organizational directory only .
4. **Redirect URI**: 
   - Select **Web**.
   - Enter the **Assertion Consumer Service (ACS)** URL from the SP’s metadata (e.g., https://your-sp-app.com/saml/acs).

### Note Key Values
- Copy the **Application (Client) ID** and **Directory (Tenant) ID**.
---

## Step 2: Configure Custom Policies
### A. Modify `TrustFrameworkExtensions.xml`
Add a SAML technical profile for the SP. Two scenarios are covered below: <br>
**Scenario 1: SP Provides Metadata URL**
```xml
<ClaimsProvider>
  <DisplayName>SAML Web App (Metadata URL)</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SAML-WebApp-TechnicalProfile">
      <DisplayName>SAML Web App</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
        <!-- SP's metadata URL -->
        <Item Key="PartnerEntity">https://your-sp-app.com/saml/metadata</Item>
        <Item Key="XmlSignatureAlgorithm">Sha256</Item>
      </Metadata>
      <CryptographicKeys>
        <!-- Certificate to sign SAML assertions -->
        <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_SamlCert" />
      </CryptographicKeys>
      <OutputClaims>
        <!-- Map claims to the SP -->
        <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="nameid" />
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```
**Scenario 2: SP Does Not Provide the Metadata URL**
```xml
<ClaimsProvider>
  <DisplayName>SAML Web App (Manual Config)</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SAML-WebApp-TechnicalProfile">
      <DisplayName>SAML Web App</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
        <!-- SP's ACS URL -->
        <Item Key="AssertionConsumerServiceUrl">https://your-sp-app.com/saml/acs</Item>
        <!-- SP's entity ID -->
        <Item Key="IssuerUri">https://your-sp-app.com</Item>
        <Item Key="XmlSignatureAlgorithm">Sha256</Item>
      </Metadata>
      <CryptographicKeys>
        <!-- Certificate to sign SAML assertions -->
        <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_SamlCert" />
        <!-- SP's signing certificate (if required) -->
        <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SPCert" />
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="nameid" />
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```
### B. Upload Certificates to B2C
1. **SAML Assertion Signing Certificate** :
    * In B2C, go to **Policy keys** > **Add** > **Upload** :
        * **Name** : SamlCert.
        * **Upload** : Certificate (.pfx) used to sign SAML assertions.
2. **SP’s Signing Certificate (Scenario 2 Only)** :
    * If the SP requires B2C to validate signed requests:
        * **Name** : SPCert.
        * **Upload** : The SP’s public certificate (.cer).

### C. Create the Relying Party (RP) Policy File
Create a new file (e.g., `SignUpOrSignInSAML.xml`) to define the SAML flow :
```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2" />
    <Metadata>
      <!-- SP's entity ID (from metadata or manual config) -->
      <Item Key="IssuerUri">https://your-sp-app.com</Item>
    </Metadata>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="issuerUserId" />
      <OutputClaim ClaimTypeReferenceId="email" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="issuerUserId" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient" />
    <!-- Reference the SAML technical profile -->
    <TechnicalProfileReference Id="SAML-WebApp-TechnicalProfile" />
  </TechnicalProfile>
</RelyingParty>
```
### D. Update the User Journey
Ensure the user journey (in `TrustFrameworkBase.xml` or `TrustFrameworkExtensions.xml`) includes a SendClaims step to trigger the SAML response:
```xml
<UserJourney Id="SignUpOrSignIn">
  <OrchestrationSteps>
    <!-- Step 1: Authenticate the user (e.g., local account) -->
    <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
      <!-- ... existing configuration ... -->
    </OrchestrationStep>

    <!-- Step 2: Send SAML assertion to SP -->
    <OrchestrationStep Order="2" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="SAML-WebApp-TechnicalProfile" />
  </OrchestrationSteps>
</UserJourney>
```
---

## Step 3: Configure the Service Provider (SP)
**Scenario 1: SP Supports Metadata URL**
1. **Provide the SP with B2C’s metadata URL:**
```
https://<Your-B2C-Tenant>.b2clogin.com/<Your-B2C-Tenant>.onmicrosoft.com/<Policy-Name>/Samlp/metadata
```
Replace `<Your-B2C-Tenant>` and `<Policy-Name>` with your B2C tenant and RP policy name.

**Scenario 2: Manual SP Configuration**
1. **Provide the SP with:**
    * **B2C’s Entity ID** : It should be whatever you configure as the issuerUri in the policy file e.g., `https://your-sp-app.com`
    * **B2C’s Signing Certificate** : Download from B2C’s metadata URL or policy keys.

## Step 4: Upload and Test Policies
1. Upload Policies :
    * In B2C’s Identity Experience Framework , upload:
        * `TrustFrameworkBase.xml` <br>
        * `TrustFrameworkExtensions.xml` <br>
        * `SignUpOrSignInSAML.xml (RP policy)` <br>
2. Test the Flow :
    * Use the Run now endpoint:
    ```
    https://<Your-B2C-Tenant>.b2clogin.com/<Your-B2C-Tenant>.onmicrosoft.com/oauth2/samlp/v2.0/authorize?
    p=B2C_1A_SAML2_signup_signin
    &client_id=<SAML-WebApp-Client-ID>
    &nonce=defaultNonce
    &redirect_uri=https%3A%2F%2Fyour-sp-app.com%2Fsaml%2Facs
    &response_mode=form_post

    ```