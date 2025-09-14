# Mapping Passkey Output Claims to Azure AD B2C User Accounts

This section extends the **Passkey-Login Technical Profile** integration to ensure B2C directory lookups succeed.

---

## 1. Why Transformation is Needed

- The Passkey REST API returns `{ "email": "alice@example.com" }`.  
- B2C must know which user in its directory corresponds to `alice@example.com`.  
- The standard attribute B2C uses for local accounts is `signInNames.emailAddress`.  
- So we must transform/copy the `email` claim into `signInNames.emailAddress`.

---

## 2. Define a ClaimsTransformation

Add this to your **TrustFrameworkExtensions.xml**, inside `<ClaimsTransformations>`:

```xml
<ClaimsTransformations>
  <ClaimsTransformation Id="CopyEmailToSignInName" TransformationMethod="AssertBooleanClaimIsEqualToValue">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim1" />
    </InputClaims>
    <OutputClaims>
      <!-- Map to the actual directory signInNames.emailAddress -->
      <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" TransformationClaimType="outputClaim" />
    </OutputClaims>
  </ClaimsTransformation>
</ClaimsTransformations>
```

⚠️ If you already have other `ClaimsTransformations`, just add this one alongside them.

---

## 3. Reference It in Passkey-Login TP

Update the **Passkey-Login Technical Profile** to use this transformation:

```xml
<TechnicalProfile Id="Passkey-Login">
  <DisplayName>Passkey Authentication</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider">
    <Metadata>
      <Item Key="ServiceUrl">https://your-backend.com/api/b2c-passkey-login</Item>
      <Item Key="AuthenticationType">None</Item>
      <Item Key="SendClaimsIn">Body</Item>
    </Metadata>
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
      <InputClaim ClaimTypeReferenceId="assertion" PartnerClaimType="assertion"/>
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
      <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
    </OutputClaims>
    <OutputClaimsTransformations>
      <OutputClaimsTransformation ReferenceId="CopyEmailToSignInName" />
    </OutputClaimsTransformations>
  </TechnicalProfile>
```

---

## 4. Orchestration Step Impact

In your UserJourney, when the **Passkey-Login** TP runs:  
1. It calls your backend `/api/b2c-passkey-login`.  
2. Backend returns verified `email`.  
3. The `CopyEmailToSignInName` transformation copies it → `signInNames.emailAddress`.  
4. Now B2C looks up the directory user by `signInNames.emailAddress`.  
5. If user exists → session continues → JWT issued.  
6. If user does not exist → journey shows “User not found” (or branches to a sign‑up step if configured).

---

## 5. Example Response Flow in Action

- User: `alice@example.com`  
- Passkey validation result from your backend to B2C:  
  ```json
  {
    "version": "1.0.0",
    "status": 200,
    "email": "alice@example.com"
  }
  ```  
- Transformation:  
  - `email` → assigned into `signInNames.emailAddress`  
- B2C Directory Lookup:  
  - Finds Alice’s account with `signInNames.emailAddress = alice@example.com`  

✅ Login completes smoothly without requiring a password.

---

# ✅ Conclusion

By adding an **OutputClaimsTransformation**, you glue the verified passkey login claims into the **B2C directory’s expected attributes**.  

- Passkey login returns `email`.  
- Transformation maps it to `signInNames.emailAddress`.  
- B2C can now locate the correct user and issue tokens.  

This ensures **server‑verified passkeys** fully interoperate with **Azure AD B2C’s user directory** just like passwords or federated identities.