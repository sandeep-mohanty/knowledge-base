# 🧭 Tutorial: Federated Identity Architecture with Shibboleth, Keycloak, and Azure Entra ID

This tutorial walks you through designing a **federated identity mesh** using three powerful identity platforms: **Shibboleth IdP**, **Keycloak**, and **Azure Entra ID**. It’s tailored for CIAM architects who want to support diverse protocols, user stores, and trust relationships across enterprise and cloud-native environments.

---

## 🧩 Architecture Overview

### 🎯 Goal

- Use **Shibboleth** as a central SAML IdP
- Proxy authentication to **Azure Entra ID** for enterprise users
- Integrate **Keycloak** for developer apps or open-source stacks
- Enable seamless federation across all three

---

## 🗺️ Diagram: Federated Identity Flow

```plaintext
+------------------+       SAML AuthnRequest       +------------------+
|     SP (App)     | ----------------------------> |   Shibboleth IdP |
|  (SAML-enabled)  |                                |   (Docker-based) |
+------------------+                                +------------------+
                                                           |
                                                           | SAML Proxy
                                                           v
                                                  +------------------+
                                                  | Azure Entra ID   |
                                                  | (Enterprise Auth)|
                                                  +------------------+

                                                           |
                                                           | Federation / Brokering
                                                           v
                                                  +------------------+
                                                  |    Keycloak      |
                                                  | (Open-source CIAM)|
                                                  +------------------+
```

---

## 🛠️ Step-by-Step Setup

### 🔹 Step 1: Deploy Shibboleth IdP (Docker)

Follow the earlier Docker tutorial to set up Shibboleth IdP locally. Ensure it exposes metadata at:

```
https://localhost:8443/idp/shibboleth
```

> This URL will be used by both Entra ID and Keycloak to establish trust.

---

### 🔹 Step 2: Configure Azure Entra ID Federation

1. Register Shibboleth IdP as an Enterprise Application
2. Upload metadata from `https://localhost:8443/idp/shibboleth`
3. Configure claim mappings (e.g., `mail`, `userPrincipalName`)
4. Download Entra metadata and add to Shibboleth:
   ```xml
   <MetadataProvider id="EntraID" xsi:type="FilesystemMetadataProvider"
                     metadataFile="/opt/shibboleth-idp/metadata/entra-id.xml"
                     requireValidMetadata="true"/>
   ```

5. Enable SAML proxying in `saml-authn-config.xml`

---

### 🔹 Step 3: Integrate Keycloak as a Federated IdP

1. In Keycloak Admin Console:
   - Create a new **Identity Provider** of type **SAML**
   - Set `Entity ID` to Shibboleth's entity ID
   - Set `Single Sign-On Service URL` to:
     ```
     https://localhost:8443/idp/profile/SAML2/Redirect/SSO
     ```

2. In Shibboleth:
   - Add Keycloak metadata to `metadata-providers.xml`
   - Configure attribute release policies for Keycloak SP

---

### 🔹 Step 4: Configure Attribute Resolution

In `attribute-resolver.xml`, define attributes sourced from Entra ID or Keycloak:

```xml
<AttributeDefinition xsi:type="Script" id="mail">
    <Script><![CDATA[
        mail = subject.getPrincipal().getAttribute("mail");
    ]]></Script>
</AttributeDefinition>
```

---

### 🔹 Step 5: Test End-to-End Flow

- Initiate login from a SAML SP
- Shibboleth receives AuthnRequest
- Based on SP metadata or flow config:
  - Redirects to Entra ID for enterprise users
  - Redirects to Keycloak for developer apps
- Assertion flows back to SP with attributes

---

## 🧠 Mental Model Summary

| Component | Role |
|----------|------|
| **Shibboleth IdP** | Central SAML broker and metadata authority |
| **Azure Entra ID** | Enterprise authentication source |
| **Keycloak** | Developer-friendly federated IdP |
| **SPs** | Rely on Shibboleth for SAML assertions |

---

## 🔐 Security Notes

- Use signed metadata and HTTPS for all endpoints
- Configure logout flows carefully across IdPs
- Monitor assertion lifetimes and replay protection

---

## 📘 References

- [Shibboleth SAML Proxying Guide](https://shibboleth.atlassian.net/wiki/spaces/KB/pages/2783936889/SAML+Proxying+EntraID+Azure+with+the+Shibboleth+IdP)
- [Keycloak Identity Brokering](https://www.keycloak.org/docs/latest/server_admin/#identity-broker)
- [Azure AD B2C Federation](https://learn.microsoft.com/en-us/azure/active-directory-b2c/federation-overview)

---
