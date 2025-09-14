# Configuring SAML Trust Relationship Between IdP and SP  


To establish a trust relationship, both the **Identity Provider (IdP)** and the **Service Provider (SP)** must exchange metadata and configure endpoints.


## Table of Contents
1. [Overview of Trust Configuration](#overview-of-trust-configuration)
2. [Configuration on the IdP Side](#configuration-on-the-idp-side)
3. [Configuration on the SP Side](#configuration-on-the-sp-side)
4. [Testing the Trust Relationship](#testing-the-trust-relationship)

---

## Overview of Trust Configuration

The trust relationship is established by exchanging metadata between the IdP and SP. The metadata includes:
- **Entity ID**: A unique identifier for the IdP or SP.
- **Endpoints**: URLs for SAML operations like Single Sign-On (SSO), Single Logout (SLO), and Assertion Consumer Service (ACS).
- **Certificates**: Public keys used for signing and encrypting SAML messages.

Once the metadata is exchanged and configured, the IdP and SP can communicate securely using SAML assertions.

---

## Configuration on the IDP Side

### Step 1: Generate Metadata
The IdP generates metadata that describes its capabilities and endpoints. This metadata typically includes:
- Entity ID of the IdP.
- Single Sign-On (SSO) URL.
- Single Logout (SLO) URL.
- Public certificate for signing assertions.

Example of IdP Metadata:
```xml
<EntityDescriptor 
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
    entityID="https://idp.example.com">
    <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
        <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://idp.example.com/sso" />
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://idp.example.com/logout" />
        <KeyDescriptor use="signing">
            <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
                <X509Data>
                    <X509Certificate>MIIC... (Base64-encoded certificate)</X509Certificate>
                </X509Data>
            </KeyInfo>
        </KeyDescriptor>
    </IDPSSODescriptor>
</EntityDescriptor>
```
### Step 2: Configure SP Information
The IdP administrator must configure the SP's details, which are typically provided in the SP's metadata. Key configurations include:  

`Entity ID` : Unique identifier for the SP.  
`Assertion Consumer Service (ACS) URL` : Endpoint where the SP receives SAML responses.  
`NameID Format` : Format of the user identifier (e.g., email address).

**Example Configuration**:  

`Entity ID`: https://sp.example.com  
`ACS URL`: https://sp.example.com/acs  
`NameID Format`: urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress

### Step 3: Enable Trust
Once the SP's details are configured, the IdP administrator enables trust by associating the SP with the IdP. This ensures that the IdP recognizes the SP as a trusted partner.

## Configuration on the SP Side

### Step 1: Generate Metadata
The SP generates metadata that describes its capabilities and endpoints. This metadata typically includes:

- Entity ID of the SP.
- Assertion Consumer Service (ACS) URL.
- Single Logout Service (SLO) URL.
- Public certificate for signing requests.

**Example of SP Metadata**:
```xml
<EntityDescriptor 
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
    entityID="https://sp.example.com">
    <SPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
        <AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://sp.example.com/acs" index="0" isDefault="true" />
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://sp.example.com/logout" />
        <KeyDescriptor use="signing">
            <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
                <X509Data>
                    <X509Certificate>MIIC... (Base64-encoded certificate)</X509Certificate>
                </X509Data>
            </KeyInfo>
        </KeyDescriptor>
    </SPSSODescriptor>
</EntityDescriptor>
```
### Step 2: Configure IdP Information
The SP administrator must configure the IdP's details, which are typically provided in the IdP's metadata. Key configurations include:  

- `Entity ID` : Unique identifier for the IdP.
- `Single Sign-On (SSO) URL` : Endpoint where the SP redirects users for authentication.
- `Single Logout (SLO) URL`: Endpoint where SP sends the SAML logout request.
- `Public Certificate` : Used to verify the signature of SAML assertions.

Example Configuration:

- `Entity ID`: https://idp.example.com
- `SSO URL`: https://idp.example.com/sso
- `Single Logout (SLO) URL`: https://idp.example.com/logout
- `Public Certificate`: MIIC... (Base64-encoded certificate)

### Step 3: Enable Trust
Once the IdP's details are configured, the SP administrator enables trust by associating the IdP with the SP. This ensures that the SP recognizes the IdP as a trusted partner.

## Testing the trust relationship

After configuring both sides, test the trust relationship to ensure that SAML-based SSO works as expected. Follow these steps:

### Step 1: Initiate SSO
- Access a protected resource on the SP.
- The SP should redirect you to the IdP for authentication.
### Step 2: Authenticate at the IdP
- Log in using valid credentials at the IdP.
- The IdP should generate a SAML assertion and redirect you back to the SP.
### Step 3: Verify Access
- The SP should validate the SAML assertion and grant access to the protected resource.
### Troubleshooting
- `Invalid Signature` : Ensure that the public certificate matches the private key used for signing.
- `Endpoint Mismatch` : Verify that the ACS URL and SSO URL are correctly configured.
- `Metadata Issues` : Re-exchange metadata if there are discrepancies.

## Conclusion

Configuring the trust relationship between the IdP and SP involves exchanging metadata, configuring endpoints, and enabling trust on both sides. By following the steps outlined above, you can establish a secure and functional SAML-based SSO system.
