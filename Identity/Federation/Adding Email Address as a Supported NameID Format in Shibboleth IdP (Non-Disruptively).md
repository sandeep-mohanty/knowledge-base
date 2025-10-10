# ✅ Step-by-Step Guide: Adding Email Address as a Supported NameID Format in Shibboleth IdP (Non-Disruptively)

This guide walks you through how to **add support for `emailAddress` as a NameID format** in Shibboleth Identity Provider (IdP), **without affecting existing Service Providers (SPs)**. The process is designed to be **seamless and non-disruptive**, ensuring backward compatibility.

---

## 🔍 Target Version: Shibboleth IdP v5.x (Most Widely Used as of 2025)

> The steps for IdP v4.x are nearly identical, with minor differences in bean syntax and file structure. Notes for v4.x are included where relevant.

---

## 🛠️ Step 1: Ensure `mail` Attribute Is Available

Edit `attribute-resolver.xml` to confirm or define the `mail` attribute.

```xml
<!-- attribute-resolver.xml -->
<AttributeDefinition xsi:type="Simple" id="mail" xmlns="urn:mace:shibboleth:2.0:resolver:ad">
    <InputDataConnector ref="LDAP" attributeNames="mail"/>
    <AttributeEncoder xsi:type="SAML2String"
                      name="urn:oid:0.9.2342.19200300.100.1.3"
                      friendlyName="email"
                      encodeType="false"/>
</AttributeDefinition>
```

> ✅ This ensures the email address is available for use in assertions.

---

## 🛠️ Step 2: Add Email-Based NameID Generator

Edit `saml-nameid.xml` to append a new generator for the email format.

```xml
<!-- saml-nameid.xml -->
<bean id="shibboleth.SAML2NameIDGenerators" class="java.util.ArrayList">
  <constructor-arg>
    <list>
      <!-- Existing generators (leave untouched) -->
      <bean parent="shibboleth.SAML2PersistentGenerator"/>
      <bean parent="shibboleth.SAML2TransientGenerator"/>

      <!-- New generator for email-based NameID -->
      <bean parent="shibboleth.SAML2AttributeSourcedGenerator">
        <property name="format" value="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"/>
        <property name="attributeSourceIds">
          <list>
            <value>mail</value>
          </list>
        </property>
      </bean>
    </list>
  </constructor-arg>
</bean>
```

> ✅ This adds support for email-based NameID **only when requested** by the SP.

---

## 🛠️ Step 3: Leave Relying Party Config Unchanged

No changes are needed in `relying-party.xml` unless you want to **force email format** for specific SPs. To keep things non-disruptive:

```xml
<!-- relying-party.xml -->
<ProfileConfiguration xsi:type="saml:SAML2SSOProfile" includeAttributeStatement="true">
  <!-- Do NOT specify NameIDFormat here -->
</ProfileConfiguration>
```

> ✅ This ensures Shibboleth uses the format requested by the SP, preserving existing behavior.

---

## 🛠️ Step 4: Confirm SP Metadata Supports Email Format

For SPs that want to use email as NameID, ensure their metadata includes:

```xml
<NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</NameIDFormat>
```

> ✅ Existing SPs that don’t include this will continue using their current formats.

---

## 🛠️ Step 5: Restart Shibboleth IdP

After making these changes, restart the IdP service:

```bash
# On Linux systems
sudo systemctl restart shibboleth-idp
```

> ✅ This reloads the configuration and activates the new generator.

---

## 📘 Notes for Shibboleth IdP v4.x

- The bean syntax and file names are largely the same.
- Ensure you’re editing the correct `saml-nameid.xml` under `conf/`.
- Attribute definitions and encoders follow the same structure.

---

## ✅ Summary

| Goal | Achieved |
|------|----------|
| Add support for email-based NameID | ✅ |
| Preserve existing SP behavior | ✅ |
| Avoid default overrides or format forcing | ✅ |
| Seamless and non-disruptive | ✅ |

---

## 🔗 Reference

- [Shibboleth IdP v5 NameIDGenerationConfiguration](https://shibboleth.atlassian.net/wiki/spaces/IDP5/pages/3199507810/NameIDGenerationConfiguration)
