# Shibboleth IdP with LDAP Integration and Azure AD B2C Federation

## Part 1: Architecture Overview and Prerequisites

### Architecture

This test harness consists of the following components:

```
┌─────────────────┐
│   End User      │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│   Azure AD B2C (SAML SP)    │
│   (Identity Broker)          │
└────────────┬────────────────┘
             │ SAML AuthN Request
             ▼
┌──────────────────────────────────┐
│  Shibboleth IdP (localhost)      │
│  - SAML 2.0 Support              │
│  - NameID: Email & Transient     │
└────────────┬─────────────────────┘
             │ LDAP Authentication
             ▼
┌──────────────────────────────────┐
│      OpenLDAP Server             │
│      (User Store)                 │
└──────────────────────────────────┘
             ▲
             │
┌────────────┴─────────────────────┐
│  User Management Web Interface   │
│  (phpLDAPadmin)                   │
└──────────────────────────────────┘
```

### Prerequisites

- Docker and Docker Compose installed[1][2]
- Basic understanding of SAML 2.0 protocol[3][4]
- Azure AD B2C tenant with custom policy capability[5][3]
- 8GB RAM minimum for running all containers[6]
- OpenSSL for certificate generation[1]

***

## Part 2: Project Structure and Docker Compose Setup

### Directory Structure

Create the following directory structure:

```bash
shibboleth-test-harness/
├── docker-compose.yml
├── shibboleth-idp/
│   ├── config/
│   │   ├── ldap.properties
│   │   ├── saml-nameid.properties
│   │   ├── saml-nameid.xml
│   │   ├── attribute-resolver.xml
│   │   ├── attribute-filter.xml
│   │   └── metadata-providers.xml
│   ├── credentials/
│   │   └── secrets.properties
│   └── metadata/
├── openldap/
│   ├── ldif/
│   │   ├── base.ldif
│   │   └── users.ldif
│   └── config/
├── nginx/
│   ├── nginx.conf
│   └── ssl/
└── azure-b2c/
    └── policies/
```

### Docker Compose Configuration

Create `docker-compose.yml`:[2][7]

```yaml
version: '3.8'

services:
  # OpenLDAP Server
  openldap:
    image: osixia/openldap:1.5.0
    container_name: shibboleth-openldap
    hostname: ldap.example.local
    environment:
      LDAP_ORGANISATION: "Example Organization"
      LDAP_DOMAIN: "example.local"
      LDAP_ADMIN_PASSWORD: "AdminPassword123"
      LDAP_CONFIG_PASSWORD: "ConfigPassword123"
      LDAP_BASE_DN: "dc=example,dc=local"
      LDAP_TLS: "false"
    volumes:
      - ./openldap/ldif:/container/service/slapd/assets/config/bootstrap/ldif/custom
      - openldap-data:/var/lib/ldap
      - openldap-config:/etc/ldap/slapd.d
    ports:
      - "389:389"
      - "636:636"
    networks:
      - shibboleth-network
    restart: unless-stopped

  # phpLDAPadmin - User Management Interface
  phpldapadmin:
    image: osixia/phpldapadmin:0.9.0
    container_name: shibboleth-phpldapadmin
    hostname: phpldapadmin.example.local
    environment:
      PHPLDAPADMIN_LDAP_HOSTS: "openldap"
      PHPLDAPADMIN_HTTPS: "false"
    ports:
      - "8080:80"
    depends_on:
      - openldap
    networks:
      - shibboleth-network
    restart: unless-stopped

  # Shibboleth IdP
  shibboleth-idp:
    image: unicon/shibboleth-idp:4.3.1
    container_name: shibboleth-idp
    hostname: idp.example.local
    environment:
      JETTY_MAX_HEAP: "2048m"
      JETTY_BROWSER_SSL_KEYSTORE_PASSWORD: "password"
      JETTY_BACKCHANNEL_SSL_KEYSTORE_PASSWORD: "password"
    volumes:
      - ./shibboleth-idp/config:/ext-mount/customized-shibboleth-idp/conf
      - ./shibboleth-idp/credentials:/ext-mount/customized-shibboleth-idp/credentials
      - ./shibboleth-idp/metadata:/ext-mount/customized-shibboleth-idp/metadata
      - shibboleth-logs:/opt/shibboleth-idp/logs
    ports:
      - "4443:4443"
      - "8443:8443"
    depends_on:
      - openldap
    networks:
      - shibboleth-network
    restart: unless-stopped

  # Nginx Reverse Proxy (for easier local access)
  nginx:
    image: nginx:alpine
    container_name: shibboleth-nginx
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    ports:
      - "443:443"
      - "80:80"
    depends_on:
      - shibboleth-idp
    networks:
      - shibboleth-network
    restart: unless-stopped

volumes:
  openldap-data:
  openldap-config:
  shibboleth-logs:

networks:
  shibboleth-network:
    driver: bridge
```

### Environment Variables File

Create `.env` file for sensitive data:

```bash
# OpenLDAP Configuration
LDAP_ADMIN_PASSWORD=AdminPassword123
LDAP_CONFIG_PASSWORD=ConfigPassword123

# Shibboleth IdP Configuration
IDP_ENTITY_ID=https://idp.example.local/idp/shibboleth
IDP_SCOPE=example.local
IDP_SEALER_PASSWORD=SuperSecretSealerPassword123
IDP_KEYSTORE_PASSWORD=KeystorePassword123

# Azure AD B2C Configuration
AZURE_B2C_TENANT_NAME=yourtenant
AZURE_B2C_POLICY_NAME=B2C_1A_SAML_SHIBBOLETH
```

***

## Part 3: OpenLDAP Configuration and User Setup

### Base LDIF Configuration

Create `openldap/ldif/base.ldif` to set up organizational structure:[8][2]

```ldif
# Root Organization
dn: dc=example,dc=local
objectClass: top
objectClass: dcObject
objectClass: organization
o: Example Organization
dc: example

# Organizational Unit for People
dn: ou=people,dc=example,dc=local
objectClass: organizationalUnit
objectClass: top
ou: people

# Organizational Unit for Groups
dn: ou=groups,dc=example,dc=local
objectClass: organizationalUnit
objectClass: top
ou: groups

# Admin Group
dn: cn=admins,ou=groups,dc=example,dc=local
objectClass: groupOfNames
cn: admins
member: cn=admin,dc=example,dc=local
```

### Sample Users LDIF

Create `openldap/ldif/users.ldif` with test users:[9][8]

```ldif
# Test User 1
dn: uid=john.doe,ou=people,dc=example,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: john.doe
cn: John Doe
givenName: John
sn: Doe
displayName: John Doe
mail: john.doe@example.local
userPassword: {SSHA}DkMTwBl+a/3DQTxCYEApdUtNXGgdUac3
uidNumber: 10001
gidNumber: 10001
homeDirectory: /home/john.doe
loginShell: /bin/bash

# Test User 2
dn: uid=jane.smith,ou=people,dc=example,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: jane.smith
cn: Jane Smith
givenName: Jane
sn: Smith
displayName: Jane Smith
mail: jane.smith@example.local
userPassword: {SSHA}hPRwnsqNt4lbtJzTOqpWmAH4jTmKEMaV
uidNumber: 10002
gidNumber: 10002
homeDirectory: /home/jane.smith
loginShell: /bin/bash

# Test User 3 - No email for testing
dn: uid=test.user,ou=people,dc=example,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: test.user
cn: Test User
givenName: Test
sn: User
displayName: Test User
userPassword: {SSHA}XvQE/hy0SY0M6Y3h8rDTL8pLOKkYbHPQ
uidNumber: 10003
gidNumber: 10003
homeDirectory: /home/test.user
loginShell: /bin/bash
```

**Note:** The userPassword values are SSHA hashed. For all test users, the password is `Password123`. To generate your own SSHA passwords, use:[8]

```bash
slappasswd -s YourPassword
```

### Initialize OpenLDAP Container

Start only the OpenLDAP container first to ensure it initializes properly:[2]

```bash
# Start OpenLDAP
docker-compose up -d openldap

# Wait for initialization (about 10 seconds)
sleep 10

# Verify LDAP is running
docker-compose logs openldap

# Test LDAP connection
docker exec shibboleth-openldap ldapsearch -x -H ldap://localhost -b dc=example,dc=local -D "cn=admin,dc=example,dc=local" -w AdminPassword123
```

### Verify User Data

Test that users were created successfully:[8]

```bash
# Search for all users
docker exec shibboleth-openldap ldapsearch -x -H ldap://localhost -b "ou=people,dc=example,dc=local" -D "cn=admin,dc=example,dc=local" -w AdminPassword123

# Test user authentication
docker exec shibboleth-openldap ldapwhoami -x -H ldap://localhost -D "uid=john.doe,ou=people,dc=example,dc=local" -w Password123
```

Expected output:
```
dn:uid=john.doe,ou=people,dc=example,dc=local
```

***

## Part 4: Shibboleth IdP Configuration

### Generate IdP Configuration Files

First, generate the base Shibboleth configuration:[1]

```bash
# Create directory for config generation
mkdir -p ./shibboleth-idp-init

# Generate IdP configuration
docker run -it \
  --volume $(pwd)/shibboleth-idp-init:/ext-mount \
  --rm unicon/shibboleth-idp:4.3.1 \
  init-idp.sh

# When prompted, enter:
# Hostname: idp.example.local
# SAML EntityID: https://idp.example.local/idp/shibboleth
# Attribute Scope: example.local
# Backchannel PKCS12 Password: password
# Cookie Encryption Key Password: password

# Copy generated files to proper location
cp -r ./shibboleth-idp-init/customized-shibboleth-idp/conf/* ./shibboleth-idp/config/
cp -r ./shibboleth-idp-init/customized-shibboleth-idp/credentials/* ./shibboleth-idp/credentials/
```

### Configure LDAP Authentication

Edit `shibboleth-idp/config/ldap.properties`:[10][11]

```properties
# LDAP connection configuration
idp.authn.LDAP.authenticator = bindSearchAuthenticator
idp.authn.LDAP.ldapURL = ldap://openldap:389
idp.authn.LDAP.useStartTLS = false
idp.authn.LDAP.useSSL = false
idp.authn.LDAP.connectTimeout = PT3S
idp.authn.LDAP.responseTimeout = PT3S

# SSL/TLS configuration (disabled for local testing)
idp.authn.LDAP.sslConfig = disabled

# Search configuration for finding user DN
idp.authn.LDAP.baseDN = ou=people,dc=example,dc=local
idp.authn.LDAP.subtreeSearch = true
idp.authn.LDAP.userFilter = (uid={user})

# Bind DN for search operation
idp.authn.LDAP.bindDN = cn=admin,dc=example,dc=local
idp.authn.LDAP.bindDNCredential = AdminPassword123

# Return attributes from LDAP
idp.authn.LDAP.returnAttributes = mail,displayName,givenName,sn,uid

# Connection pool configuration
idp.pool.LDAP.minSize = 3
idp.pool.LDAP.maxSize = 10
idp.pool.LDAP.validateOnCheckout = false
idp.pool.LDAP.validatePeriodically = true
idp.pool.LDAP.validatePeriod = PT5M
idp.pool.LDAP.prunePeriod = PT5M
idp.pool.LDAP.idleTime = PT10M
idp.pool.LDAP.blockWaitTime = PT3S
```

### Enable LDAP Authentication

Edit `shibboleth-idp/config/authn/password-authn-config.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
       default-init-method="initialize"
       default-destroy-method="destroy">

    <!-- Import LDAP authentication configuration -->
    <import resource="ldap-authn-config.xml" />

    <!-- Define LDAP validator -->
    <util:list id="shibboleth.authn.Password.Validators">
        <ref bean="shibboleth.LDAPValidator" />
    </util:list>

</beans>
```

### Configure Attribute Resolution

Edit `shibboleth-idp/config/attribute-resolver.xml` to retrieve attributes from LDAP:[12][13]

```xml
<?xml version="1.0" encoding="UTF-8"?>
<AttributeResolver
        xmlns="urn:mace:shibboleth:2.0:resolver"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="urn:mace:shibboleth:2.0:resolver http://shibboleth.net/schema/idp/shibboleth-attribute-resolver.xsd">

    <!-- LDAP Data Connector -->
    <DataConnector id="myLDAP" xsi:type="LDAPDirectory"
        ldapURL="ldap://openldap:389"
        baseDN="ou=people,dc=example,dc=local"
        principal="cn=admin,dc=example,dc=local"
        principalCredential="AdminPassword123"
        useStartTLS="false">
        <FilterTemplate>
            <![CDATA[
                (uid=$resolutionContext.principal)
            ]]>
        </FilterTemplate>
        <ReturnAttributes>uid mail displayName givenName sn</ReturnAttributes>
    </DataConnector>

    <!-- Email Address Attribute -->
    <AttributeDefinition xsi:type="Simple" id="mail">
        <InputDataConnector ref="myLDAP" attributeNames="mail"/>
        <AttributeEncoder xsi:type="SAML2String" 
            xmlns="urn:mace:shibboleth:2.0:attribute:encoder"
            name="urn:oid:0.9.2342.19200300.100.1.3" 
            friendlyName="mail"/>
        <!-- Encoder for NameID format email -->
        <AttributeEncoder xsi:type="SAML2StringNameID"
            xmlns="urn:mace:shibboleth:2.0:attribute:encoder"
            nameFormat="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"/>
    </AttributeDefinition>

    <!-- Display Name Attribute -->
    <AttributeDefinition xsi:type="Simple" id="displayName">
        <InputDataConnector ref="myLDAP" attributeNames="displayName"/>
        <AttributeEncoder xsi:type="SAML2String"
            xmlns="urn:mace:shibboleth:2.0:attribute:encoder"
            name="urn:oid:2.16.840.1.113730.3.1.241"
            friendlyName="displayName"/>
    </AttributeDefinition>

    <!-- Given Name Attribute -->
    <AttributeDefinition xsi:type="Simple" id="givenName">
        <InputDataConnector ref="myLDAP" attributeNames="givenName"/>
        <AttributeEncoder xsi:type="SAML2String"
            xmlns="urn:mace:shibboleth:2.0:attribute:encoder"
            name="urn:oid:2.5.4.42"
            friendlyName="givenName"/>
    </AttributeDefinition>

    <!-- Surname Attribute -->
    <AttributeDefinition xsi:type="Simple" id="sn">
        <InputDataConnector ref="myLDAP" attributeNames="sn"/>
        <AttributeEncoder xsi:type="SAML2String"
            xmlns="urn:mace:shibboleth:2.0:attribute:encoder"
            name="urn:oid:2.5.4.4"
            friendlyName="sn"/>
    </AttributeDefinition>

    <!-- UID Attribute -->
    <AttributeDefinition xsi:type="Simple" id="uid">
        <InputDataConnector ref="myLDAP" attributeNames="uid"/>
        <AttributeEncoder xsi:type="SAML2String"
            xmlns="urn:mace:shibboleth:2.0:attribute:encoder"
            name="urn:oid:0.9.2342.19200300.100.1.1"
            friendlyName="uid"/>
    </AttributeDefinition>

</AttributeResolver>
```

***

## Part 5: NameID Configuration (Email and Transient)

### Configure NameID Properties

Edit `shibboleth-idp/config/saml-nameid.properties`:[14][12]

```properties
# Default NameID format for SAML 2.0
# Set to transient by default, but will respect SP requests
idp.nameid.saml2.default = urn:oasis:names:tc:SAML:2.0:nameid-format:transient

# Default NameID format for SAML 1.0
idp.nameid.saml1.default = urn:mace:shibboleth:1.0:nameIdentifier

# Transient ID generation strategy
# Use crypto-based transient IDs (no storage needed)
idp.transientId.generator = shibboleth.CryptoTransientIdGenerator

# Source attribute for persistent IDs (if enabled)
idp.persistentId.sourceAttribute = uid

# Computed ID generation strategy
idp.persistentId.generator = shibboleth.StoredPersistentIdGenerator
idp.persistentId.dataSource = shibboleth.JPAStorageService
idp.persistentId.computed = shibboleth.ComputedPersistentIdGenerator

# Salt for computed persistent ID (change this!)
idp.persistentId.salt = changeMeToSomethingSecureAndRandom12345
```

### Configure NameID Generators

Edit `shibboleth-idp/config/saml-nameid.xml`:[13][12]

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
       default-init-method="initialize"
       default-destroy-method="destroy">

    <!-- SAML 2.0 NameID Generators -->
    <util:list id="shibboleth.SAML2NameIDGenerators">
    
        <!-- Transient NameID Generator -->
        <ref bean="shibboleth.SAML2TransientGenerator" />
        
        <!-- Email Address NameID Generator -->
        <bean parent="shibboleth.SAML2AttributeSourcedGenerator"
            p:format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"
            p:attributeSourceIds="#{ {'mail'} }">
            <property name="activationCondition">
                <bean parent="shibboleth.Conditions.RelyingPartyId">
                    <constructor-arg>
                        <list>
                            <!-- Allow email NameID for all SPs -->
                            <value>.*</value>
                        </list>
                    </constructor-arg>
                </bean>
            </property>
        </bean>

        <!-- Unspecified NameID - fallback to uid -->
        <bean parent="shibboleth.SAML2AttributeSourcedGenerator"
            p:format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified"
            p:attributeSourceIds="#{ {'uid'} }"/>
        
    </util:list>

    <!-- SAML 1.0 NameIdentifier Generators -->
    <util:list id="shibboleth.SAML1NameIdentifierGenerators">
        
        <!-- Transient NameIdentifier Generator -->
        <ref bean="shibboleth.SAML1TransientGenerator" />
        
        <!-- Email Address NameIdentifier Generator -->
        <bean parent="shibboleth.SAML1AttributeSourcedGenerator"
            p:format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"
            p:attributeSourceIds="#{ {'mail'} }"/>
            
    </util:list>

</beans>
```

### Configure Attribute Filter for NameID

Edit `shibboleth-idp/config/attribute-filter.xml` to release attributes for NameID generation:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<AttributeFilterPolicyGroup id="ShibbolethFilterPolicy"
        xmlns="urn:mace:shibboleth:2.0:afp"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="urn:mace:shibboleth:2.0:afp http://shibboleth.net/schema/idp/shibboleth-afp.xsd">

    <!-- Release attributes to all SPs -->
    <AttributeFilterPolicy id="releaseToAll">
        <PolicyRequirementRule xsi:type="ANY"/>

        <!-- Release email for NameID -->
        <AttributeRule attributeID="mail">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <!-- Release display name -->
        <AttributeRule attributeID="displayName">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <!-- Release given name -->
        <AttributeRule attributeID="givenName">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <!-- Release surname -->
        <AttributeRule attributeID="sn">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <!-- Release UID -->
        <AttributeRule attributeID="uid">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <!-- Deny transient IDs for Azure B2C when email format is requested -->
        <!-- This ensures email NameID is prioritized -->
        <AttributeRule attributeID="transientId">
            <PermitValueRule xsi:type="NOT">
                <Rule xsi:type="AttributeRequesterRegex" regex=".*b2clogin\.com.*"/>
            </PermitValueRule>
        </AttributeRule>

    </AttributeFilterPolicy>

    <!-- Azure AD B2C Specific Policy -->
    <AttributeFilterPolicy id="releaseToAzureB2C">
        <PolicyRequirementRule xsi:type="AttributeRequesterRegex" 
            regex="https://.*\.b2clogin\.com/.*"/>

        <!-- Release all attributes to Azure B2C -->
        <AttributeRule attributeID="mail">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <AttributeRule attributeID="displayName">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <AttributeRule attributeID="givenName">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <AttributeRule attributeID="sn">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>

        <AttributeRule attributeID="uid">
            <PermitValueRule xsi:type="ANY"/>
        </AttributeRule>
    </AttributeFilterPolicy>

</AttributeFilterPolicyGroup>
```

***

## Part 6: Azure AD B2C Integration Configuration

### Important Note on Localhost Connectivity

Azure AD B2C cannot directly reach `localhost` or private IP addresses. To work around this limitation for local development, you have three options:[4][3]

**Option 1: Use ngrok or similar tunneling service (Recommended for testing)**
```bash
# Install ngrok from https://ngrok.com/
ngrok http 443

# Use the provided HTTPS URL in your B2C configuration
# Example: https://abcd1234.ngrok.io
```

**Option 2: Use hosts file trick (localhost only)**
Configure your local machine's hosts file to resolve a custom domain to 127.0.0.1, then use that domain in B2C

**Option 3: Deploy to cloud temporarily**
Deploy Shibboleth to a cloud VM with a public IP for testing

For this tutorial, we'll assume you're using **ngrok** with the domain `https://your-ngrok-domain.ngrok.io`

### Configure Nginx for External Access

Update `nginx/nginx.conf` to support external access:

```nginx
events {
    worker_connections 1024;
}

http {
    upstream shibboleth {
        server shibboleth-idp:8443;
    }

    # HTTP to HTTPS redirect
    server {
        listen 80;
        server_name _;
        return 301 https://$host$request_uri;
    }

    # HTTPS Server
    server {
        listen 443 ssl;
        server_name _;

        # SSL Configuration (self-signed for local testing)
        ssl_certificate /etc/nginx/ssl/server.crt;
        ssl_certificate_key /etc/nginx/ssl/server.key;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;

        # Proxy settings
        location /idp/ {
            proxy_pass https://shibboleth/idp/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_ssl_verify off;
        }

        # Health check
        location /health {
            return 200 "OK\n";
            add_header Content-Type text/plain;
        }
    }
}
```

### Generate Self-Signed SSL Certificates

Generate SSL certificates for nginx:

```bash
mkdir -p nginx/ssl
cd nginx/ssl

# Generate private key
openssl genrsa -out server.key 2048

# Generate certificate signing request
openssl req -new -key server.key -out server.csr \
  -subj "/C=US/ST=State/L=City/O=Example/OU=IT/CN=idp.example.local"

# Generate self-signed certificate
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

cd ../..
```

### Register Shibboleth IdP Metadata with Azure B2C

First, start your Shibboleth IdP to generate metadata:

```bash
# Start all services
docker-compose up -d

# Wait for Shibboleth to initialize
sleep 30

# Check if Shibboleth is running
curl -k https://localhost:443/idp/status
```

Download the IdP metadata (replace with your ngrok URL when using external tunnel):

```bash
curl -k https://localhost:443/idp/shibboleth > shibboleth-metadata.xml
```

### Create Azure AD B2C Custom Policy

Create the directory `azure-b2c/policies/` and add the following files:[3][4]

**1. TrustFrameworkBase_SAML.xml** (Base policy with SAML support)

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" 
  PolicySchemaVersion="0.3.0.0" 
  TenantId="yourtenant.onmicrosoft.com" 
  PolicyId="B2C_1A_TrustFrameworkBase_SAML" 
  PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase_SAML">
  
  <BuildingBlocks>
    <ClaimsSchema>
      <ClaimType Id="issuerUserId">
        <DisplayName>Username</DisplayName>
        <DataType>string</DataType>
        <UserHelpText/>
        <UserInputType>TextBox</UserInputType>
      </ClaimType>
      <ClaimType Id="email">
        <DisplayName>Email Address</DisplayName>
        <DataType>string</DataType>
        <UserHelpText>Email address</UserHelpText>
        <UserInputType>TextBox</UserInputType>
      </ClaimType>
      <ClaimType Id="displayName">
        <DisplayName>Display Name</DisplayName>
        <DataType>string</DataType>
        <UserHelpText>Your display name.</UserHelpText>
        <UserInputType>TextBox</UserInputType>
      </ClaimType>
      <ClaimType Id="givenName">
        <DisplayName>Given Name</DisplayName>
        <DataType>string</DataType>
        <UserHelpText>Your given name (also known as first name).</UserHelpText>
        <UserInputType>TextBox</UserInputType>
      </ClaimType>
      <ClaimType Id="surname">
        <DisplayName>Surname</DisplayName>
        <DataType>string</DataType>
        <UserHelpText>Your surname (also known as family name or last name).</UserHelpText>
        <UserInputType>TextBox</UserInputType>
      </ClaimType>
    </ClaimsSchema>
    
    <ClaimsTransformations>
      <ClaimsTransformation Id="CreateRandomUPNUserName" TransformationMethod="CreateRandomString">
        <InputParameters>
          <InputParameter Id="randomGeneratorType" DataType="string" Value="GUID"/>
        </InputParameters>
        <OutputClaims>
          <OutputClaim ClaimTypeReferenceId="issuerUserId" TransformationClaimType="outputClaim"/>
        </OutputClaims>
      </ClaimsTransformation>
    </ClaimsTransformations>
  </BuildingBlocks>

  <ClaimsProviders>
    <ClaimsProvider>
      <DisplayName>Shibboleth IdP</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Shibboleth-SAML">
          <DisplayName>Shibboleth SAML IdP</DisplayName>
          <Protocol Name="SAML2"/>
          <Metadata>
            <!-- Replace with your ngrok URL -->
            <Item Key="PartnerEntity">https://your-ngrok-domain.ngrok.io/idp/shibboleth</Item>
            
            <!-- Request email NameID format -->
            <Item Key="WantsSignedRequests">false</Item>
            <Item Key="NameIdPolicyFormat">urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</Item>
            
            <!-- Alternative: Use transient NameID -->
            <!-- <Item Key="NameIdPolicyFormat">urn:oasis:names:tc:SAML:2.0:nameid-format:transient</Item> -->
            
            <!-- SSO Service URL - Replace with your ngrok URL -->
            <Item Key="SingleSignOnServiceUrl">https://your-ngrok-domain.ngrok.io/idp/profile/SAML2/Redirect/SSO</Item>
            
            <!-- Logout Service URL -->
            <Item Key="SingleLogoutServiceUrl">https://your-ngrok-domain.ngrok.io/idp/profile/SAML2/Redirect/SLO</Item>
            
            <!-- Response signing -->
            <Item Key="WantsSignedAssertions">true</Item>
          </Metadata>
          <CryptographicKeys>
            <!-- Upload Shibboleth's signing certificate to B2C policy keys -->
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_ShibbolethSamlCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="NameId"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="mail"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="displayName"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="givenName"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="sn"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="ShibbolethIdP"/>
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication"/>
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Saml-idp"/>
        </TechnicalProfile>

        <!-- Session management -->
        <TechnicalProfile Id="SM-Saml-idp">
          <DisplayName>Session Management Provider</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
  </ClaimsProviders>

</TrustFrameworkPolicy>
```

**2. TrustFrameworkExtensions_SAML.xml** (Extension policy)

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" 
  PolicySchemaVersion="0.3.0.0" 
  TenantId="yourtenant.onmicrosoft.com" 
  PolicyId="B2C_1A_TrustFrameworkExtensions_SAML" 
  PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_TrustFrameworkExtensions_SAML">
  
  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkBase_SAML</PolicyId>
  </BasePolicy>

  <BuildingBlocks></BuildingBlocks>

  <ClaimsProviders></ClaimsProviders>

  <UserJourneys>
    <UserJourney Id="SignInWithShibboleth">
      <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="ShibbolethExchange" TechnicalProfileReferenceId="Shibboleth-SAML"/>
          </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="2" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer"/>
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb"/>
    </UserJourney>
  </UserJourneys>

</TrustFrameworkPolicy>
```

**3. SignIn_SAML_Shibboleth.xml** (Relying party policy)

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" 
  PolicySchemaVersion="0.3.0.0" 
  TenantId="yourtenant.onmicrosoft.com" 
  PolicyId="B2C_1A_SignIn_SAML_Shibboleth" 
  PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_SignIn_SAML_Shibboleth">
  
  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions_SAML</PolicyId>
  </BasePolicy>

  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignInWithShibboleth"/>
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect"/>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName"/>
        <OutputClaim ClaimTypeReferenceId="givenName"/>
        <OutputClaim ClaimTypeReferenceId="surname"/>
        <OutputClaim ClaimTypeReferenceId="email"/>
        <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="identityProvider"/>
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub"/>
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

### Upload Shibboleth Certificate to Azure AD B2C

Extract the signing certificate from Shibboleth metadata:[3]

```bash
# Extract certificate from metadata
xmllint --xpath "//ds:X509Certificate[1]/text()" shibboleth-metadata.xml > shibboleth-cert.txt

# Or manually open shibboleth-metadata.xml and copy the certificate content
```

Upload to Azure AD B2C:

1. Navigate to Azure Portal → Azure AD B2C → Identity Experience Framework
2. Click on **Policy Keys**
3. Click **Add**
   - Options: Upload
   - Name: ShibbolethSamlCert
   - File upload: Upload the certificate file (you may need to convert to .cer format)
   - Key usage: Signature

### Upload Custom Policies to Azure AD B2C

Upload policies in this order:[3]

```bash
1. B2C_1A_TrustFrameworkBase_SAML
2. B2C_1A_TrustFrameworkExtensions_SAML
3. B2C_1A_SignIn_SAML_Shibboleth
```

In Azure Portal:
1. Navigate to Identity Experience Framework
2. Click **Upload custom policy**
3. Select each XML file in order above

***

## Part 7: User Management Interface Setup

The phpLDAPadmin container is already included in the docker-compose file. Access it at:[9]

```
http://localhost:8080
```

Login credentials:
- Login DN: `cn=admin,dc=example,dc=local`
- Password: `AdminPassword123`

### Creating Users via phpLDAPadmin

1. Navigate to `ou=people,dc=example,dc=local`
2. Click **Create new entry here**
3. Select **Generic: User Account**
4. Fill in the form:
   - Common Name: User's full name
   - User ID: username
   - Email: user@example.local
   - Password: User's password
5. Click **Create Object**

### Creating Users via Command Line

```bash
# Create a new user LDIF file
cat > newuser.ldif << 'EOF'
dn: uid=alice.wonder,ou=people,dc=example,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: alice.wonder
cn: Alice Wonder
givenName: Alice
sn: Wonder
displayName: Alice Wonder
mail: alice.wonder@example.local
userPassword: {SSHA}encryptedPasswordHere
uidNumber: 10004
gidNumber: 10004
homeDirectory: /home/alice.wonder
loginShell: /bin/bash
EOF

# Add the user
docker exec shibboleth-openldap ldapadd -x -H ldap://localhost \
  -D "cn=admin,dc=example,dc=local" -w AdminPassword123 -f /tmp/newuser.ldif
```

### Deleting Users

```bash
# Delete a user
docker exec shibboleth-openldap ldapdelete -x -H ldap://localhost \
  -D "cn=admin,dc=example,dc=local" -w AdminPassword123 \
  "uid=alice.wonder,ou=people,dc=example,dc=local"
```

***

## Part 8: Testing the Complete Setup

### Step 1: Verify All Services are Running

```bash
# Check all containers
docker-compose ps

# Should show:
# - shibboleth-openldap (running)
# - shibboleth-phpldapadmin (running)
# - shibboleth-idp (running)
# - shibboleth-nginx (running)

# Check Shibboleth status
curl -k https://localhost:443/idp/status
```

### Step 2: Test LDAP Authentication

```bash
# Test user authentication
docker exec shibboleth-openldap ldapwhoami -x -H ldap://localhost \
  -D "uid=john.doe,ou=people,dc=example,dc=local" -w Password123

# Expected: dn:uid=john.doe,ou=people,dc=example,dc=local
```

### Step 3: Test Shibboleth IdP Login

1. Navigate to: `https://localhost:443/idp/profile/admin/hello`
2. You should see Shibboleth status page

### Step 4: View Shibboleth Metadata

```bash
# View metadata
curl -k https://localhost:443/idp/shibboleth

# You should see XML metadata with:
# - EntityDescriptor element
# - IDPSSODescriptor element
# - SingleSignOnService endpoints
# - NameIDFormat elements (transient and emailAddress)
```

### Step 5: Configure Azure B2C Metadata in Shibboleth

Create `shibboleth-idp/metadata/azure-b2c-metadata.xml` with Azure B2C metadata:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<EntityDescriptor 
    entityID="https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/B2C_1A_SignIn_SAML_Shibboleth"
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata">
    <SPSSODescriptor 
        protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol"
        AuthnRequestsSigned="false"
        WantAssertionsSigned="true">
        
        <!-- NameID formats supported -->
        <NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</NameIDFormat>
        <NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:transient</NameIDFormat>
        
        <!-- Assertion Consumer Service -->
        <AssertionConsumerService 
            Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
            Location="https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/B2C_1A_SignIn_SAML_Shibboleth/samlp/sso/assertionconsumer"
            index="0" isDefault="true"/>
    </SPSSODescriptor>
</EntityDescriptor>
```

Update `shibboleth-idp/config/metadata-providers.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<MetadataProvider id="ShibbolethMetadata"
    xmlns="urn:mace:shibboleth:2.0:metadata"
    xmlns:resource="urn:mace:shibboleth:2.0:resource"
    xmlns:security="urn:mace:shibboleth:2.0:security"
    xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="urn:mace:shibboleth:2.0:metadata http://shibboleth.net/schema/idp/shibboleth-metadata.xsd
                        urn:mace:shibboleth:2.0:resource http://shibboleth.net/schema/idp/shibboleth-resource.xsd 
                        urn:mace:shibboleth:2.0:security http://shibboleth.net/schema/idp/shibboleth-security.xsd
                        urn:oasis:names:tc:SAML:2.0:metadata http://docs.oasis-open.org/security/saml/v2.0/saml-schema-metadata-2.0.xsd">

    <!-- Azure AD B2C Metadata -->
    <MetadataProvider id="AzureB2C" xsi:type="FilesystemMetadataProvider">
        <MetadataResource xsi:type="resource:FilesystemResource" 
            file="/opt/shibboleth-idp/metadata/azure-b2c-metadata.xml"/>
        <MetadataFilter xsi:type="RequiredValidUntil" maxValidityInterval="P30D"/>
    </MetadataProvider>

</MetadataProvider>
```

### Step 6: Test End-to-End Flow

1. **Setup ngrok tunnel** (if not already running):
```bash
ngrok http 443
# Note the HTTPS URL provided (e.g., https://abc123.ngrok.io)
```

2. **Update all configurations with ngrok URL**:
   - Azure B2C custom policy: Replace `your-ngrok-domain.ngrok.io` with actual ngrok domain
   - Re-upload the custom policies to Azure B2C

3. **Test authentication flow**:
   - Navigate to: `https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/B2C_1A_SignIn_SAML_Shibboleth/oauth2/v2.0/authorize?client_id=YOUR_APP_ID&redirect_uri=https://jwt.ms&response_type=id_token&scope=openid`
   - You should be redirected to Shibboleth login page
   - Login with: john.doe / Password123
   - After successful authentication, you'll be redirected back to jwt.ms with a JWT token

4. **Verify NameID in SAML Response**:
   - Use browser developer tools (F12)
   - Look at network tab for SAML POST response
   - The NameID element should contain john.doe@example.local if email format was requested

### Step 7: Testing Different NameID Formats

**Test Email NameID Format:**

Modify Azure B2C technical profile to request email:
```xml
<Item Key="NameIdPolicyFormat">urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</Item>
```

Re-upload policy and test. Check Shibboleth logs:

```bash
docker logs shibboleth-idp | grep -i nameid
```

**Test Transient NameID Format:**

Modify Azure B2C technical profile:
```xml
<Item Key="NameIdPolicyFormat">urn:oasis:names:tc:SAML:2.0:nameid-format:transient</Item>
```

Re-upload policy and test. The NameID should be a random string.

***

## Part 9: Troubleshooting and Logs

### Viewing Logs

**Shibboleth IdP Logs:**
```bash
# Real-time logs
docker logs -f shibboleth-idp

# Authentication flow logs
docker exec shibboleth-idp tail -f /opt/shibboleth-idp/logs/idp-process.log

# Audit logs
docker exec shibboleth-idp tail -f /opt/shibboleth-idp/logs/idp-audit.log
```

**OpenLDAP Logs:**
```bash
docker logs -f shibboleth-openldap
```

### Common Issues

**Issue 1: LDAP Connection Failed**

```bash
# Test LDAP connectivity from Shibboleth container
docker exec shibboleth-idp ldapsearch -x -H ldap://openldap:389 \
  -b "dc=example,dc=local" -D "cn=admin,dc=example,dc=local" -w AdminPassword123
```

Solution: Verify LDAP service is running and network connectivity exists.

**Issue 2: NameID Not Generated**

Check Shibboleth logs for attribute resolution:
```bash
docker exec shibboleth-idp grep -i "attribute resolution" /opt/shibboleth-idp/logs/idp-process.log
```

Solution: Ensure `mail` attribute is returned from LDAP and properly configured in attribute-resolver.xml[12][13]

**Issue 3: Azure B2C Cannot Reach Localhost**

Solution: Use ngrok or deploy to public server[3]

**Issue 4: Certificate Validation Failed**

```bash
# Check certificate in metadata
xmllint --xpath "//ds:X509Certificate[1]/text()" shibboleth-metadata.xml
```

Solution: Ensure certificate is correctly uploaded to Azure B2C policy keys

### Debugging Tips

1. **Enable TRACE logging** in Shibboleth:

Edit `shibboleth-idp/config/logback.xml`:
```xml
<logger name="net.shibboleth.idp" level="TRACE"/>
<logger name="org.ldaptive" level="TRACE"/>
```

2. **Monitor SAML messages**:

Use browser extension like "SAML tracer" to inspect SAML requests/responses

3. **Validate SAML metadata**:

```bash
# Validate Shibboleth metadata
xmllint --schema saml-schema-metadata-2.0.xsd shibboleth-metadata.xml
```

***

## Part 10: Production Considerations

This setup is designed for testing. For production use, consider:

1. **Security Hardening**:
   - Use proper SSL/TLS certificates from a trusted CA
   - Enable LDAPS (LDAP over SSL)[11]
   - Implement proper firewall rules
   - Use secrets management (Azure Key Vault, HashiCorp Vault)

2. **High Availability**:
   - Deploy multiple Shibboleth IdP instances behind a load balancer
   - Use replicated LDAP setup (master-slave or multi-master)
   - Implement session clustering for Shibboleth

3. **Monitoring**:
   - Set up log aggregation (ELK stack, Splunk)
   - Configure health checks and alerts
   - Monitor LDAP performance and connection pool

4. **Backup and Recovery**:
   - Regular backups of LDAP data
   - Backup Shibboleth configuration and credentials
   - Document recovery procedures

5. **Performance Tuning**:
   - Optimize LDAP queries and indexing
   - Tune connection pool sizes[11]
   - Enable caching where appropriate

***

## Summary

You now have a complete test harness with:

✅ OpenLDAP server running in Docker[7][2]
✅ Shibboleth IdP configured for SAML federation[11][1]
✅ Support for both Email and Transient NameID formats[14][12]
✅ Azure AD B2C integration as identity broker[4][3]
✅ User management interface (phpLDAPadmin)[9]
✅ Docker Compose setup for easy deployment

This architecture allows you to test SAML federation flows locally before deploying to production environments[15][6]

