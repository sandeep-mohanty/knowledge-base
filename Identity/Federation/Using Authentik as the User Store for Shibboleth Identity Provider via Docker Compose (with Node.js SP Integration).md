# 🧪 Tutorial: Using Authentik as the User Store for Shibboleth Identity Provider via Docker Compose (with Node.js SP Integration and PostgreSQL 17)

This tutorial sets up a **local federated identity testbed** using Docker Compose, where:

- **Authentik** acts as the upstream identity provider and user store
- **Shibboleth IdP** acts as a SAML proxy, delegating authentication to Authentik
- **Node.js app** acts as a Service Provider (SP) using `samlify` to consume SAML assertions
- **PostgreSQL 17** and **Redis 7** are used for Authentik’s backend
- **Self-signed certificates** are used for signing and securing SAML assertions

---

## ✅ Overview

| Component        | Role                                 |
|------------------|--------------------------------------|
| **Authentik**     | Identity provider and user store     |
| **Shibboleth IdP**| SAML broker and assertion issuer     |
| **Node.js SP**    | Consumes SAML assertions via `samlify` |
| **PostgreSQL 17** | Authentik database backend           |
| **Redis 7**       | Authentik cache and queue backend    |

---

## 🗺️ Architecture Diagram

```plaintext
+------------------+       SAML AuthnRequest       +------------------+
| Node.js SP App   | ----------------------------> |   Shibboleth IdP |
| (samlify-based)  |                                |   (Docker-based) |
+------------------+                                +------------------+
                                                           |
                                                           | SAML Proxy
                                                           v
                                                  +------------------+
                                                  |    Authentik     |
                                                  | (User Store + IdP)|
                                                  +------------------+
```

---

## 🛠️ Step 1: Generate Self-Signed Certificates

```bash
openssl genrsa -out sp-key.pem 2048
openssl req -new -key sp-key.pem -out sp-csr.pem \
  -subj "/C=IN/ST=Odisha/L=Bhubaneswar/O=TestSP/CN=localhost"
openssl x509 -req -days 365 -in sp-csr.pem -signkey sp-key.pem -out sp-cert.pem
```

Place these files in your `node-sp/` directory:
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
  authentik:
    image: ghcr.io/goauthentik/server:latest
    environment:
      AUTHENTIK_SECRET_KEY: "supersecretkey"
      AUTHENTIK_POSTGRESQL__HOST: db
      AUTHENTIK_POSTGRESQL__USER: authentik
      AUTHENTIK_POSTGRESQL__PASSWORD: authentik
      AUTHENTIK_POSTGRESQL__NAME: authentik
      AUTHENTIK_REDIS__HOST: redis
    ports:
      - "9000:9000"

  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_DB: authentik
      POSTGRES_USER: authentik
      POSTGRES_PASSWORD: authentik

  redis:
    image: redis:7

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

## 🛠️ Step 3: Configure Authentik

1. Access Authentik Admin Console:
   ```
   http://localhost:9000
   ```

2. Create a new **Application** (e.g., `ShibbolethProxy`)

3. Add a **SAML Provider**:
   - SSO URL: `/application/saml/shibbolethproxy/sso/binding/redirect/`
   - Metadata URL: `/application/saml/shibbolethproxy/metadata/`
   - NameID Format: `emailAddress`
   - Enable signed assertions

4. Add **Users** and assign them to the application

5. Export Authentik metadata:
   ```
   http://localhost:9000/application/saml/shibbolethproxy/metadata/
   ```

---

## 🛠️ Step 4: Configure Shibboleth IdP to Use Authentik

### 4.1 Add Authentik Metadata

Save Authentik metadata as:

```
shibboleth-idp/idp/metadata/authentik.xml
```

Update `metadata-providers.xml`:

```xml
<MetadataProvider id="Authentik" xsi:type="FilesystemMetadataProvider"
                  metadataFile="/opt/shibboleth-idp/metadata/authentik.xml"
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
    <property name="idpEntityID" value="http://localhost:9000/application/saml/shibbolethproxy/metadata/"/>
    <property name="metadataProviderRef" value="Authentik"/>
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
   - Shibboleth proxies to Authentik
   - Authentik authenticates user
   - Shibboleth issues SAML assertion
   - Node.js SP receives and parses assertion

---

## 🔐 Notes

- PostgreSQL 17 ensures future-proof compatibility and performance
- Redis 7 is optimal for Authentik’s caching and queuing
- Self-signed certs are fine for local testing; use trusted certs in production
- You can extend Authentik with LDAP, OAuth2, or social login

---

## 📘 References

- [Authentik SAML Provider Docs](https://docs.goauthentik.io/add-secure-apps/providers/saml)
- [Shibboleth SAML Proxying Guide](https://shibboleth.atlassian.net/wiki/spaces/IDP5/pages/3199505973/SAMLAuthnConfiguration)
- [samlify Documentation](https://samlify.js.org/)
- [PostgreSQL 17 Release Notes](https://www.postgresql.org/docs/17/release-17.html)

---
