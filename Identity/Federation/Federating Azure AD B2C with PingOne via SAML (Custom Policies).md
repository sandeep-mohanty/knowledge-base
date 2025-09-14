# 🔐 Federating Azure AD B2C with PingOne via SAML (Custom Policies)

This guide walks you through how to federate your application using **Azure AD B2C custom policies**, where **Azure AD B2C delegates authentication to PingOne** using **SAML 2.0**.

---

## 🔁 Federation Flow Overview

1. User accesses your application → redirected to Azure AD B2C.
2. Azure AD B2C executes a custom policy that redirects to PingOne (SAML IdP).
3. User authenticates on PingOne.
4. PingOne sends a SAML response to Azure AD B2C.
5. Azure AD B2C processes the response and returns a token to your application.

---

## 🧰 Prerequisites

- Azure AD B2C tenant with custom policies (Identity Experience Framework) enabled.
- PingOne account with a SAML 2.0 IdP application configured.
- Access to metadata (XML or URLs) for both PingOne and Azure AD B2C.
- Basic understanding of SAML and Azure AD B2C custom policies.

---

## 🏗️ Step-by-Step Implementation

### 1. ✅ Set Up PingOne as a SAML IdP

1. In **PingOne**, create a new **SAML Identity Provider**.
2. Configure the **ACS (Assertion Consumer Service) URL** as:

   ```
   https://<your-b2c-tenant-name>.b2clogin.com/<your-b2c-tenant-name>.onmicrosoft.com/<custom-policy-name>/samlp/sso/login
   ```

3. Set the **Entity ID** as:

   ```
   https://<your-b2c-tenant-name>.b2clogin.com/<your-b2c-tenant-name>.onmicrosoft.com/<custom-policy-name>
   ```

4. Define the attribute mappings (e.g., email, givenName, surname).
5. Download the **IdP metadata XML** from PingOne – needed for B2C configuration.

---

### 2. 🧾 Create or Update B2C Custom Policies

You'll define the PingOne SAML IdP in your custom policy files.

#### a. `TrustFrameworkExtensions.xml`

Add the PingOne SAML identity provider as a `ClaimsProvider`.

```xml
<ClaimsProvider>
  <Domain>PingOne</Domain>
  <DisplayName>PingOne</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="PingOne-SAML">
      <DisplayName>PingOne SAML</DisplayName>
      <Protocol Name="SAML2"/>
      <Metadata>
        <Item Key="PartnerEntity">https://pingone.com/idp/</Item>
        <Item Key="WantsEncryptedAssertions">false</Item>
        <Item Key="SingleSignOnServiceUrl">https://sso.pingidentity.com/sso/idp/SSO.saml2</Item>
        <Item Key="IdpMetadata">[Insert PingOne metadata XML here]</Item>
        <Item Key="AllowUnsolicitedAuthResponses">true</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="SamlIdpSigningCert" StorageReferenceId="B2C_1A_PingOneCert"/>
      </CryptographicKeys>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
        <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
        <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-SAML"/>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

#### b. Define Session Management Technical Profile

```xml
<TechnicalProfile Id="SM-SAML">
  <DisplayName>Session Management</DisplayName>
  <Protocol Name="None"/>
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="email"/>
  </PersistedClaims>
  <SessionManagement>
    <SessionExpiryType>Rolling</SessionExpiryType>
    <SessionExpiryInSeconds>3600</SessionExpiryInSeconds>
  </SessionManagement>
</TechnicalProfile>
```

---

### 3. 🔐 Upload SAML Signing Certificate to Azure AD B2C

1. Convert the PingOne signing certificate to Base64 if necessary.
2. Upload the certificate using Azure CLI:

```sh
az ad b2c policy key set \
  --name B2C_1A_PingOneCert \
  --policy-name TrustFrameworkExtensions \
  --key-type X509Certificate \
  --key-usage Verify \
  --value <certificate_base64>
```

---

### 4. 🔄 Modify the User Journey to Include PingOne

Update your orchestration steps in the relying party file or `TrustFrameworkExtensions.xml`:

```xml
<OrchestrationStep Order="1" Type="ClaimsExchange">
  <ClaimsExchanges>
    <ClaimsExchange Id="PingOneExchange" TechnicalProfileReferenceId="PingOne-SAML"/>
  </ClaimsExchanges>
</OrchestrationStep>
```

---

### 5. 🧪 Test the Federation

1. Upload the `TrustFrameworkBase.xml`, `TrustFrameworkExtensions.xml`, and `TrustFrameworkLocalization.xml` to your Azure AD B2C tenant.
2. Use the **"Run now"** feature in Azure B2C to test the policy.
3. You should be redirected to PingOne for authentication.
4. After successful login, you will be redirected back to your application with a token.

---

## 🧠 Additional Considerations

- **SAML Attributes**: Ensure PingOne sends the correct attributes mapped in your B2C policy.
- **Time Synchronization**: Ensure both PingOne and Azure AD B2C servers have accurate clocks.
- **Logout Support**: Implement SAML Single Logout (SLO) if needed.
- **Token Claims**: Map attributes from PingOne to claims expected by your application.

---

## ✅ Summary

- Azure AD B2C acts as the SAML SP.
- PingOne acts as the SAML IdP.
- Custom policies in Azure AD B2C handle redirection and federation.
- After successful login, Azure AD B2C issues a token to your application.

---