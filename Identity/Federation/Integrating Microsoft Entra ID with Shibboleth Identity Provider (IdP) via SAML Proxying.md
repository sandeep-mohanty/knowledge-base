# 🧩 Tutorial: Integrating Microsoft Entra ID with Shibboleth Identity Provider (IdP) via SAML Proxying

This guide extends your Docker-based Shibboleth IdP setup to **leverage Microsoft Entra ID (formerly Azure AD)** as the user store. Instead of using LDAP or local databases, Shibboleth will act as a **SAML proxy**, delegating authentication to Entra ID and consuming user attributes from it.

---

## ✅ Overview

| Feature | Supported |
|--------|-----------|
| Use Entra ID tenant for authentication | ✅ |
| Shibboleth as SAML proxy | ✅ |
| Attribute pass-through from Entra ID | ✅ |
| Docker-based local testing | ✅ (with cloud Entra ID) |
| Metadata URL for SP configuration | ✅ |

---

## 🛠️ Step 1: Prepare Entra ID for Federation

### 1.1 Register Shibboleth IdP as an Enterprise Application

- Go to **Microsoft Entra Admin Center** → *Enterprise Applications* → *New Application*
- Choose **Non-gallery application**
- Name it (e.g., `ShibbolethProxy`)
- Configure **Single Sign-On** → *SAML*

### 1.2 Upload Shibboleth IdP Metadata

- Export your IdP metadata from:
  ```
  https://<your-idp-host>/idp/shibboleth
  ```
  > ✅ This is the official metadata endpoint exposed by Shibboleth IdP. It returns XML-formatted metadata used by SPs to establish trust.

- Upload it to Entra ID under *SAML Settings*

### 1.3 Configure Claim Mapping

- Map Entra ID attributes to SAML claims:
  - `user.mail` → `mail`
  - `user.userPrincipalName` → `uid` or `NameID`
  - Add custom claims if needed

---

## 🛠️ Step 2: Configure Shibboleth IdP for SAML Proxying

### 2.1 Enable SAML Authentication Flow

Edit `idp.properties`:

```properties
idp.authn.flows = SAML
```

### 2.2 Add Entra ID Metadata to Shibboleth

Download Entra ID metadata from:
```
https://login.microsoftonline.com/<tenant-id>/federationmetadata/2007-06/federationmetadata.xml
```

Place it in `idp/metadata/entra-id.xml`

Update `metadata-providers.xml`:

```xml
<MetadataProvider id="EntraID" xsi:type="FilesystemMetadataProvider"
                  metadataFile="/opt/shibboleth-idp/metadata/entra-id.xml"
                  requireValidMetadata="true">
    <MetadataFilter xsi:type="RequiredValidUntil" maxValidityInterval="P30D"/>
</MetadataProvider>
```

### 2.3 Configure SAML Proxy Authentication

Edit `saml-authn-config.xml` to define the upstream IdP (Entra ID):

```xml
<bean id="shibboleth.authn.SAML.SAML2AuthnConfiguration"
      class="net.shibboleth.idp.authn.saml.SAML2AuthnConfiguration">
    <property name="idpEntityID" value="https://login.microsoftonline.com/<tenant-id>/v2.0"/>
    <property name="metadataProviderRef" value="EntraID"/>
    <property name="NameIDFormatPrecedence">
        <list>
            <value>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</value>
        </list>
    </property>
</bean>
```

---

## 🛠️ Step 3: Attribute Resolution and Filtering

### 3.1 Pass Through Entra ID Attributes

Edit `attribute-resolver.xml` to define attributes sourced from Entra ID assertions:

```xml
<AttributeDefinition xsi:type="Script" id="mail">
    <Script><![CDATA[
        mail = subject.getPrincipal().getAttribute("mail");
    ]]></Script>
</AttributeDefinition>
```

### 3.2 Filter Attributes for SPs

Edit `attribute-filter.xml` to release attributes to downstream SPs:

```xml
<AttributeFilterPolicy id="ReleaseMail">
    <PolicyRequirementRule xsi:type="ANY"/>
    <AttributeRule attributeID="mail">
        <PermitValueRule xsi:type="ANY"/>
    </AttributeRule>
</AttributeFilterPolicy>
```

---

## 🧪 Step 4: Test the Flow

- Initiate SAML login from a test SP
- Shibboleth redirects to Entra ID
- Entra ID authenticates the user
- Shibboleth receives assertion and passes attributes to SP

---

## 🌐 Metadata URL for SP Configuration

To configure your SP to trust the Shibboleth IdP, use the following metadata URL:

```
https://<your-idp-host>/idp/shibboleth
```

> ✅ This endpoint serves the IdP's metadata in XML format. It includes entity ID, supported bindings, NameID formats, and signing certificates. No `.xml` extension is required.

For local Docker setups using port `8443`, the URL becomes:

```
https://localhost:8443/idp/shibboleth
```

You can verify it with:

```bash
curl -k https://localhost:8443/idp/shibboleth
```

---

## 🔐 Notes on Security and UX

- Entra ID handles MFA, Conditional Access, and user lifecycle
- Shibboleth acts as a federation bridge, enabling multilateral trust
- You can customize the login experience using Entra branding

---

## 📘 References

- [Shibboleth KB: SAML Proxying with Entra ID](https://shibboleth.atlassian.net/wiki/spaces/KB/pages/2783936889/SAML+Proxying+EntraID+Azure+with+the+Shibboleth+IdP)
- [Microsoft Learn: Entra ID with Shibboleth Proxy](https://learn.microsoft.com/en-us/entra/architecture/multilateral-federation-solution-two)

---
