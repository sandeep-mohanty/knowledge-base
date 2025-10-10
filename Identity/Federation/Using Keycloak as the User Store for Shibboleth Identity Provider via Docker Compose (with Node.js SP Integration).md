# 🧪 Tutorial: Using Keycloak as the User Store for Shibboleth Identity Provider via Docker Compose (with Node.js SP Integration and Self-Signed Certificates)

This tutorial sets up a **local federated identity testbed** using Docker Compose, where:

- **Keycloak** acts as the upstream identity provider and user store
- **Shibboleth IdP** acts as a SAML proxy, delegating authentication to Keycloak
- **Node.js app** acts as a Service Provider (SP) using `samlify` to consume SAML assertions
- **Self-signed certificates** are used for signing and securing SAML assertions

---

## ✅ Overview

| Component | Role |
|----------|------|
| **Keycloak** | Identity provider and user store |
| **Shibboleth IdP** | SAML broker and assertion issuer |
| **Node.js SP** | Consumes SAML assertions via `samlify` |
| **Self-signed certs** | Used for signing SAML metadata and assertions |

---

## 🛠️ Step 1: Generate Self-Signed Certificates

Use OpenSSL to generate a key pair and certificate for your SP:

```bash
openssl genrsa -out sp-key.pem 2048
openssl req -new -key sp-key.pem -out sp-csr.pem \
  -subj "/C=IN/ST=Odisha/L=Bhubaneswar/O=TestSP/CN=localhost"
openssl x509 -req -days 365 -in sp-csr.pem -signkey sp-key.pem -out sp-cert.pem
```

> ✅ These files will be used by the Node.js SP to sign requests and validate responses.

Place them in your `node-sp/` directory:
```
node-sp/
├── sp-key.pem
├── sp-cert.pem
```

---

## 🛠️ Step 2: Create `docker-compose.yml`

```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:24.0.1
    command: start-dev
    environment:
      KC_DB: dev-mem
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
    ports:
      - "8080:8080"

  shibboleth-idp:
    build: ./shibboleth-idp
    ports:
      - "8443:8443"
    volumes:
      - ./shibboleth-idp/idp:/opt/shibboleth-idp

  node-sp:
    build: ./node-sp
    ports:
      - "3000:3000"
    volumes:
      - ./node-sp:/usr/src/app
```

---

## 🛠️ Step 3: Configure Keycloak

1. Access Keycloak Admin Console:
   ```
   http://localhost:8080
   ```
   Login with `admin/admin`.

2. Create a new **Realm** (e.g., `TestRealm`)

3. Add a few **Users** with email and username

4. Create a new **Client**:
   - Client ID: `shibboleth-idp`
   - Protocol: `saml`
   - Root URL: `https://localhost:8443/idp/profile/SAML2/Redirect/SSO`
   - Enable `Sign Assertions`, `Sign Documents`, and `Force POST Binding`

5. Export Keycloak metadata:
   ```
   http://localhost:8080/realms/TestRealm/protocol/saml/descriptor
   ```

---

## 🛠️ Step 4: Configure Shibboleth IdP to Use Keycloak

### 4.1 Add Keycloak Metadata

Save Keycloak metadata as:

```
shibboleth-idp/idp/metadata/keycloak.xml
```

Update `metadata-providers.xml`:

```xml
<MetadataProvider id="Keycloak" xsi:type="FilesystemMetadataProvider"
                  metadataFile="/opt/shibboleth-idp/metadata/keycloak.xml"
                  requireValidMetadata="true"/>
```

---

### 4.2 Enable SAML Proxying

Edit `idp.properties`:

```properties
idp.authn.flows = SAML
```

Edit `saml-authn-config.xml`:

```xml
<bean id="shibboleth.authn.SAML.SAML2AuthnConfiguration"
      class="net.shibboleth.idp.authn.saml.SAML2AuthnConfiguration">
    <property name="idpEntityID" value="http://localhost:8080/realms/TestRealm"/>
    <property name="metadataProviderRef" value="Keycloak"/>
    <property name="NameIDFormatPrecedence">
        <list>
            <value>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</value>
        </list>
    </property>
</bean>
```

---

### 4.3 Attribute Resolution

Edit `attribute-resolver.xml`:

```xml
<AttributeDefinition xsi:type="Script" id="mail">
    <Script><![CDATA[
        mail = subject.getPrincipal().getAttribute("email");
    ]]></Script>
</AttributeDefinition>
```

---

## 🛠️ Step 5: Configure Node.js SP Using `samlify`

### 5.1 Create `node-sp` Directory Structure

```
node-sp/
├── Dockerfile
├── index.js
├── idp-metadata.xml
├── sp-cert.pem
├── sp-key.pem
```

### 5.2 Sample `Dockerfile`

```Dockerfile
FROM node:18
WORKDIR /usr/src/app
COPY . .
RUN npm install samlify express body-parser
EXPOSE 3000
CMD ["node", "index.js"]
```

### 5.3 Sample `index.js`

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const saml = require('samlify');
const fs = require('fs');

const app = express();
app.use(bodyParser.urlencoded({ extended: true }));

const sp = saml.ServiceProvider({
  entityID: 'http://localhost:3000/metadata',
  assertionConsumerService: [{
    Binding: saml.Constants.namespace.post,
    Location: 'http://localhost:3000/assert'
  }],
  privateKey: fs.readFileSync('./sp-key.pem'),
  privateKeyPass: '',
  signingCert: fs.readFileSync('./sp-cert.pem'),
});

const idp = saml.IdentityProvider({
  metadata: fs.readFileSync('./idp-metadata.xml'),
});

app.get('/metadata', (req, res) => {
  res.type('application/xml');
  res.send(sp.getMetadata());
});

app.get('/login', async (req, res) => {
  const { context } = await sp.createLoginRequest(idp, 'redirect');
  res.redirect(context);
});

app.post('/assert', async (req, res) => {
  const { extract } = await sp.parseLoginResponse(idp, 'post', req);
  res.send(`Hello ${extract.attributes.email}`);
});

app.listen(3000, () => console.log('SP running on http://localhost:3000'));
```

---

## 🧪 Step 6: Test the Full SAML Flow

1. Start the stack:
   ```bash
   docker-compose up --build
   ```

2. Access Node.js SP:
   ```
   http://localhost:3000/login
   ```

3. Flow:
   - SP sends AuthnRequest to Shibboleth
   - Shibboleth proxies to Keycloak
   - Keycloak authenticates user
   - Shibboleth issues SAML assertion
   - Node.js SP receives and parses assertion

---

## 🔐 Notes on Self-Signed Certificates

- Self-signed certs are fine for local testing and development
- Ensure the cert is included in your SP metadata
- For production, use certificates from a trusted CA
- Track expiration dates to avoid silent failures

---

## 📘 References

- [Keycloak Docker Docs](https://www.keycloak.org/server/docker)
- [Shibboleth SAML Proxying Guide](https://shibboleth.atlassian.net/wiki/spaces/KB/pages/2783936889/SAML+Proxying+EntraID+Azure+with+the+Shibboleth+IdP)
- [samlify Documentation](https://samlify.js.org/)
- [OpenSSL Certificate Guide](https://security.stackexchange.com/questions/146132/self-signed-certificate-for-a-idp-initiated-saml-sso)

---
