# Keycloak Complete Tutorial: Enterprise Identity and Access Management

## Table of Contents

1. Introduction to Keycloak
2. Core Concepts and Terminology
3. Installation and Setup
4. Realm Configuration
5. User and Role Management
6. Client Configuration (OIDC & SAML)
7. Authentication Flows and Customization
8. Multi-Factor Authentication
9. Identity Brokering and Social Login
10. User Federation (LDAP/AD)
11. Authorization Services
12. OpenFGA Integration for ReBAC
13. Session Management
14. Events and Logging
15. Security Best Practices
16. Monitoring and Performance
17. Real-World Use Cases
18. Troubleshooting

***

## 1. Introduction to Keycloak

### What is Keycloak?

Keycloak is an open-source Identity and Access Management (IAM) solution developed by Red Hat that provides enterprise-grade authentication and authorization capabilities. It eliminates the need for developers to build login, registration, and access control mechanisms from scratch.[1]

**Core Value Proposition**:[1]
- **Single Sign-On (SSO)**: Users authenticate once and access multiple applications without re-entering credentials
- **Identity Brokering**: Connect to external identity providers (Google, Facebook, LDAP, Active Directory, SAML providers)
- **User Federation**: Integrate existing user directories without data migration
- **Standard Protocol Support**: Out-of-the-box support for OpenID Connect, OAuth 2.0, and SAML 2.0
- **Social Login**: Enable login via social media platforms
- **Admin Console**: Centralized web-based management interface
- **Account Management Console**: Self-service portal for users to manage their profiles
- **Authorization Services**: Fine-grained access control based on policies

### Why Keycloak for CIAM?

In a **Customer Identity and Access Management (CIAM)** context, Keycloak provides:[2][1]

**For Customers**:
- Seamless registration and login experiences
- Social login integration (Google, Facebook, LinkedIn, GitHub)
- Self-service account management (profile updates, password resets)
- Privacy controls and consent management
- Multi-factor authentication for enhanced security

**For Organizations**:
- Centralized customer identity management across multiple applications
- Scalability to handle millions of customer accounts
- Compliance with regulations (GDPR, CCPA) through audit logging
- Customizable branding to match corporate identity
- Analytics and insights through event tracking

**Real-World CIAM Scenario**:
Imagine an e-commerce company with a web store, mobile app, and partner portal. Without Keycloak, each application would need separate authentication logic, leading to:
- Inconsistent user experiences
- Multiple passwords for customers to remember
- Duplicated user data across systems
- Complex password reset workflows
- Security vulnerabilities from inconsistent implementations

With Keycloak, customers authenticate once and access all applications seamlessly. The organization manages all identities from a single control plane.

### Keycloak Architecture Overview

```text
┌─────────────────────────────────────────────────────────────────┐
│                        User/Application                          │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         │ HTTPS/TLS
                         ▼
         ┌───────────────────────────────────────┐
         │         Keycloak Server               │
         │  ┌─────────────────────────────────┐  │
         │  │      Admin Console              │  │ (Port 8080)
         │  │  - Realm Management             │  │
         │  │  - User Management              │  │
         │  │  - Client Configuration         │  │
         │  └─────────────────────────────────┘  │
         │  ┌─────────────────────────────────┐  │
         │  │   Account Management Console    │  │
         │  │  - User Self-Service            │  │
         │  └─────────────────────────────────┘  │
         │  ┌─────────────────────────────────┐  │
         │  │    Authentication Endpoints     │  │
         │  │  - /auth, /token, /userinfo     │  │
         │  │  - /logout, /register           │  │
         │  └─────────────────────────────────┘  │
         └────────────┬──────────────────────────┘
                      │
         ┌────────────┼────────────────────────────┐
         │            │                            │
         ▼            ▼                            ▼
   ┌─────────┐  ┌──────────┐            ┌─────────────────┐
   │Database │  │  Cache   │            │External Identity│
   │(Postgres│  │(Infinispan)          │   Providers     │
   │ MySQL)  │  │- Sessions│            │- LDAP/AD        │
   │         │  │- Tokens  │            │- Google/GitHub  │
   └─────────┘  └──────────┘            │- SAML IdPs      │
                                        └─────────────────┘
```

**Key Components**:

1. **Admin Console**: Web interface for administrators to configure realms, users, clients, and policies
2. **Account Console**: Self-service portal for end-users to manage their profiles and security settings
3. **Authentication Server**: Issues tokens and manages authentication flows
4. **Database**: Stores realms, users, clients, roles, and configuration (supports PostgreSQL, MySQL, Oracle, MS SQL)
5. **Cache Layer**: Infinispan for session caching and performance optimization
6. **Event System**: Captures authentication events, admin actions, and audit trails

---

## 2. Core Concepts and Terminology

Understanding Keycloak's terminology is crucial for effective implementation. Let's explore each concept with real-world analogies.

### Realms

A **realm** is a complete isolated namespace that manages a set of users, credentials, roles, and groups. Think of a realm as a tenant in a multi-tenant SaaS application.[1]

**Analogy**: If Keycloak is an apartment building, each realm is a separate apartment with its own residents (users), access rules (roles), and keys (credentials). Residents of one apartment cannot access another apartment.

**Real-World Use Cases**:

**Use Case 1: Multi-Brand Organization**
```text
Company: Global Retail Corp
├── Realm: brand-a-customers (Brand A e-commerce)
├── Realm: brand-b-customers (Brand B e-commerce)
├── Realm: internal-employees (Corporate intranet)
└── Realm: partner-portal (B2B partners)
```

Each realm has completely separate:
- User bases (customers vs. employees vs. partners)
- Authentication policies (MFA required for employees, optional for customers)
- Branding (logos, color schemes, email templates)
- Integration points (Brand A uses Stripe, Brand B uses PayPal)

**Use Case 2: Environment Isolation**
```text
├── Realm: production (Live customer accounts)
├── Realm: staging (QA testing with test users)
└── Realm: development (Developer sandboxes)
```

**Master Realm**: Special administrative realm that manages Keycloak itself and can create/manage other realms. **Best Practice**: Never use the master realm for application users—always create dedicated realms.[1]

### Users

**Users** are entities that can authenticate to your system. Each user has:[1]

**Core Attributes**:
- **Username**: Unique identifier (can be email or custom format)
- **Email**: Email address (optionally verified)
- **First Name / Last Name**: Display names
- **Enabled**: Whether the account is active
- **Email Verified**: Whether email has been confirmed

**Custom Attributes**:
In CIAM scenarios, you often need custom attributes:
- `customer_tier`: "gold", "silver", "bronze"
- `account_type`: "individual", "business"
- `subscription_id`: Link to billing system
- `marketing_consent`: GDPR consent tracking
- `preferred_language`: Localization preference
- `phone_number`: For SMS MFA
- `date_of_birth`: Age verification
- `loyalty_points`: Rewards program integration

**Example User Object**:
```json
{
  "id": "f0a7c2d3-1234-5678-90ab-cdef12345678",
  "username": "john.customer@example.com",
  "email": "john.customer@example.com",
  "emailVerified": true,
  "enabled": true,
  "firstName": "John",
  "lastName": "Customer",
  "attributes": {
    "customer_tier": ["gold"],
    "subscription_id": ["sub_1234567890"],
    "marketing_consent": ["true"],
    "preferred_language": ["en-US"],
    "phone_number": ["+1-555-123-4567"],
    "loyalty_points": ["15000"]
  },
  "createdTimestamp": 1729000000000,
  "requiredActions": [],
  "access": {
    "manageGroupMembership": true,
    "view": true,
    "mapRoles": true,
    "impersonate": false,
    "manage": true
  }
}
```

### Roles

**Roles** define what users can do in your applications. They represent job functions or access levels.[1]

**Two Types of Roles**:

**1. Realm Roles**: Global across all clients in the realm[1]
```text
Examples:
- admin: Full access to everything
- user: Standard user access
- premium_member: Access to premium features
- content_creator: Can create/edit content
- moderator: Can moderate user content
- analytics_viewer: Can view reports
```

**2. Client Roles**: Specific to individual applications[1]
```text
Client: e-commerce-web
├── customer: Can browse and purchase
├── vendor: Can list products
└── support: Can view orders and issue refunds

Client: mobile-app
├── mobile_user: Can use mobile features
└── beta_tester: Can access beta features
```

**Real-World CIAM Example**:

For a streaming service:
```text
Realm Roles (Global):
├── free_user: Watch free content with ads
├── premium_user: Ad-free watching, HD quality
├── family_plan: Multiple profiles, 4K quality
└── content_admin: Upload and manage content

Client Roles (Smart TV App):
├── tv_viewer: Can watch on TV
└── offline_download: Can download for offline

Client Roles (Mobile App):
├── mobile_viewer: Can watch on mobile
└── chromecast_user: Can cast to TV
```

**Token Claim Example**:
```json
{
  "realm_access": {
    "roles": ["premium_user", "default-roles-streaming"]
  },
  "resource_access": {
    "smart-tv-app": {
      "roles": ["tv_viewer", "offline_download"]
    },
    "mobile-app": {
      "roles": ["mobile_viewer", "chromecast_user"]
    }
  }
}
```

### Composite Roles

A **composite role** is a role that automatically includes other roles. This simplifies role management by creating hierarchies.[1]

**Real-World Example**: Customer Support Organization

```text
support_agent (composite role)
├── Includes: view_customer_profiles
├── Includes: view_orders
└── Includes: create_support_tickets

senior_support_agent (composite role)
├── Includes: support_agent (all of the above)
├── Includes: process_refunds
└── Includes: escalate_issues

support_manager (composite role)
├── Includes: senior_support_agent (all of the above)
├── Includes: view_analytics
├── Includes: manage_team
└── Includes: access_audit_logs
```

When you assign `support_manager` to a user, they automatically get all 9 permissions without individually assigning each role.[1]

**Configuration**:
```json
{
  "name": "support_manager",
  "composite": true,
  "composites": {
    "realm": ["view_analytics", "manage_team", "access_audit_logs"],
    "client": {
      "support-portal": ["senior_support_agent"]
    }
  }
}
```

### Groups

**Groups** organize users and manage role assignments collectively. Groups can be hierarchical.[1]

**Difference Between Groups and Roles**:
- **Roles**: Define WHAT users can do (permissions)
- **Groups**: Define WHO users are (organizational structure)

**Real-World CIAM Example**: SaaS B2B Platform

```text
Organization: Acme Corporation (Customer)
├── Department: Sales
│   ├── Team: North America Sales
│   │   ├── User: alice@acme.com
│   │   └── User: bob@acme.com
│   └── Team: Europe Sales
│       ├── User: charlie@acme.com
│       └── User: diana@acme.com
├── Department: Marketing
│   ├── User: eve@acme.com
│   └── User: frank@acme.com
└── Department: IT
    └── User: grace@acme.com (Admin)
```

**Group Attributes**:
Groups can carry attributes inherited by all members:
```json
{
  "name": "Sales",
  "path": "/Acme Corporation/Sales",
  "attributes": {
    "cost_center": ["CC-1001"],
    "region": ["North America"],
    "manager_email": ["sales-manager@acme.com"],
    "access_level": ["department"]
  }
}
```

**Role Mapping to Groups**:
```text
Group: /Acme Corporation
└── Assigned Roles: [basic_user]

Group: /Acme Corporation/Sales
└── Assigned Roles: [crm_access, sales_dashboard]

Group: /Acme Corporation/IT
└── Assigned Roles: [admin_portal, system_settings]
```

Users inherit all roles from their group hierarchy. Alice in "North America Sales" gets: `basic_user`, `crm_access`, `sales_dashboard`.

### Clients

**Clients** are applications or services that delegate authentication to Keycloak. Each client represents an integration point.[1]

**Client Types**:

**1. Confidential Clients**: Can securely store credentials (backend applications)[1]
```text
Examples:
- Web application with backend (Spring Boot, Django, Node.js)
- API gateway
- Backend service (microservice)
```

**2. Public Clients**: Cannot securely store credentials (frontend applications)[1]
```text
Examples:
- Single Page Applications (React, Angular, Vue)
- Mobile applications (iOS, Android)
- Desktop applications
```

**3. Bearer-Only Clients**: Only validate tokens, don't initiate authentication[1]
```text
Examples:
- REST APIs
- Microservices
- Resource servers
```

**Real-World CIAM Client Architecture**:

```text
Realm: retail-customers

Clients:
├── web-app (Confidential)
│   ├── Type: OpenID Connect
│   ├── Access Type: Confidential
│   ├── Valid Redirect URIs: https://shop.example.com/*
│   ├── Web Origins: https://shop.example.com
│   └── Purpose: Main e-commerce website
│
├── mobile-app-ios (Public)
│   ├── Type: OpenID Connect
│   ├── Access Type: Public
│   ├── Valid Redirect URIs: myapp://callback
│   └── Purpose: iOS mobile application
│
├── mobile-app-android (Public)
│   ├── Type: OpenID Connect
│   ├── Access Type: Public
│   ├── Valid Redirect URIs: com.example.app://callback
│   └── Purpose: Android mobile application
│
├── api-gateway (Bearer-Only)
│   ├── Type: OpenID Connect
│   ├── Access Type: Bearer-Only
│   └── Purpose: Validates tokens for API requests
│
├── partner-integration (Confidential)
│   ├── Type: OpenID Connect
│   ├── Access Type: Confidential
│   ├── Service Accounts Enabled: true
│   └── Purpose: B2B partner API integration (client credentials flow)
│
└── legacy-erp (SAML)
    ├── Type: SAML 2.0
    ├── Client ID: https://erp.example.com/saml
    └── Purpose: Legacy enterprise system integration
```

### Client Scopes

**Client scopes** are templates for claims and roles that can be shared across multiple clients. They enable:[1]
- **Reusability**: Define once, use in many clients
- **Conditional Claims**: Include claims based on requested scope
- **Consent Management**: Users can consent to specific scopes

**Default Client Scopes**: Automatically included in tokens[1]
**Optional Client Scopes**: Included only when explicitly requested[1]

**Real-World Example**: Multi-Application Suite

```text
Client Scope: profile (Default)
├── Mappers:
│   ├── username → preferred_username
│   ├── email → email
│   ├── firstName → given_name
│   ├── lastName → family_name
│   └── fullName → name

Client Scope: email (Default)
├── Mappers:
│   ├── email → email
│   └── emailVerified → email_verified

Client Scope: address (Optional)
├── Mappers:
│   ├── street → address.street_address
│   ├── city → address.locality
│   ├── state → address.region
│   ├── postalCode → address.postal_code
│   └── country → address.country

Client Scope: phone (Optional)
├── Mappers:
│   ├── phoneNumber → phone_number
│   └── phoneVerified → phone_number_verified

Client Scope: orders (Optional)
├── Mappers:
│   ├── orderHistory → order_ids
│   └── loyaltyPoints → loyalty_points

Client Scope: offline_access (Optional)
└── Purpose: Request refresh tokens with long expiration
```

**OAuth 2.0 Scope Request**:
```text
Authorization Request:
https://auth.example.com/realms/retail/protocol/openid-connect/auth?
  client_id=web-app&
  response_type=code&
  scope=openid profile email address orders&
  redirect_uri=https://shop.example.com/callback
```

The resulting token includes claims from: `openid`, `profile`, `email`, `address`, and `orders` scopes.

**Consent Screen**:
When optional scopes are requested, Keycloak can show a consent screen:
```text
┌────────────────────────────────────────┐
│  MyShop would like to:                 │
│                                        │
│  ✓ View your profile and email         │
│  ✓ View your address                   │
│  ✓ Access your order history           │
│                                        │
│  [ Allow ]  [ Deny ]                   │
└────────────────────────────────────────┘
```

***

## 3. Installation and Setup

### Prerequisites

**System Requirements**:
- **Operating System**: Linux (Ubuntu 20.04+, RHEL 8+), Windows Server, macOS
- **Java**: OpenJDK 17 or 21 (required)[3]
- **Memory**: Minimum 2GB RAM (4GB+ recommended for production)
- **Database**: PostgreSQL 12+, MySQL 8+, MariaDB 10.3+, Oracle 12c+, MS SQL Server 2016+
- **Disk Space**: 1GB minimum (more for logs and database)

**Network Requirements**:
- **Ports**: 8080 (HTTP), 8443 (HTTPS), 9990 (Management)
- **Firewall**: Allow inbound on ports 8080/8443
- **DNS**: Proper hostname resolution for production

### Installation Methods

#### Method 1: Docker (Quickest for Development)

**Single Container Setup**:

```bash
# Pull latest Keycloak image
docker pull quay.io/keycloak/keycloak:25.0.1

# Run in development mode
docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin123 \
  quay.io/keycloak/keycloak:25.0.1 \
  start-dev
```

**Access**: Open http://localhost:8080 in your browser.

**With PostgreSQL (Production-Like)**:

```bash
# Create network
docker network create keycloak-network

# Start PostgreSQL
docker run -d \
  --name postgres \
  --network keycloak-network \
  -e POSTGRES_DB=keycloak \
  -e POSTGRES_USER=keycloak \
  -e POSTGRES_PASSWORD=password \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:14

# Start Keycloak with PostgreSQL
docker run -d \
  --name keycloak \
  --network keycloak-network \
  -p 8080:8080 \
  -e KC_DB=postgres \
  -e KC_DB_URL=jdbc:postgresql://postgres:5432/keycloak \
  -e KC_DB_USERNAME=keycloak \
  -e KC_DB_PASSWORD=password \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin123 \
  quay.io/keycloak/keycloak:25.0.1 \
  start-dev
```

#### Method 2: Docker Compose (Recommended for Development)

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14
    container_name: keycloak-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-changeMe123!}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - keycloak-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U keycloak"]
      interval: 10s
      timeout: 5s
      retries: 5

  keycloak:
    image: quay.io/keycloak/keycloak:25.0.1
    container_name: keycloak
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      # Database configuration
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: ${POSTGRES_PASSWORD:-changeMe123!}
      
      # Admin user
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: ${KEYCLOAK_ADMIN_PASSWORD:-admin}
      
      # Development settings
      KC_HEALTH_ENABLED: true
      KC_METRICS_ENABLED: true
      KC_LOG_LEVEL: INFO
    ports:
      - "8080:8080"
    networks:
      - keycloak-network
    command: start-dev
    healthcheck:
      test: ["CMD-SHELL", "exec 3<>/dev/tcp/localhost/8080 && echo -e 'GET /health/ready HTTP/1.1\\r\\nHost: localhost\\r\\nConnection: close\\r\\n\\r\\n' >&3 && cat <&3 | grep -q '200 OK'"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

volumes:
  postgres-data:
    driver: local

networks:
  keycloak-network:
    driver: bridge
```

**Start the stack**:
```bash
docker-compose up -d
```

**View logs**:
```bash
docker-compose logs -f keycloak
```

**Stop the stack**:
```bash
docker-compose down
```

#### Method 3: Bare Metal Installation (Ubuntu/Linux)

**Step-by-Step Installation**:[3]

```bash
# 1. Update system packages
sudo apt update && sudo apt upgrade -y

# 2. Install OpenJDK 17
sudo apt install -y openjdk-17-jdk

# Verify Java installation
java -version
# Output should show: openjdk version "17.x.x"

# 3. Create Keycloak user (security best practice)
sudo useradd -r -s /bin/false keycloak

# 4. Download Keycloak
cd /opt
sudo wget https://github.com/keycloak/keycloak/releases/download/25.0.1/keycloak-25.0.1.tar.gz

# 5. Extract archive
sudo tar -xzf keycloak-25.0.1.tar.gz
sudo mv keycloak-25.0.1 keycloak

# 6. Set ownership
sudo chown -R keycloak:keycloak /opt/keycloak

# 7. Install PostgreSQL (recommended for production)
sudo apt install -y postgresql postgresql-contrib

# 8. Create Keycloak database
sudo -u postgres psql <<EOF
CREATE DATABASE keycloak;
CREATE USER keycloak WITH ENCRYPTED PASSWORD 'your-secure-password-here';
GRANT ALL PRIVILEGES ON DATABASE keycloak TO keycloak;
\q
EOF

# 9. Configure Keycloak
sudo nano /opt/keycloak/conf/keycloak.conf
```

**Configuration File** (`/opt/keycloak/conf/keycloak.conf`):

```properties
# Database configuration
db=postgres
db-url=jdbc:postgresql://localhost:5432/keycloak
db-username=keycloak
db-password=your-secure-password-here

# Hostname configuration (important for production)
hostname=auth.yourdomain.com
hostname-strict=false
hostname-strict-https=false

# HTTP configuration (development)
http-enabled=true
http-host=0.0.0.0
http-port=8080

# HTTPS configuration (production - uncomment when ready)
# https-certificate-file=/etc/ssl/certs/keycloak.crt
# https-certificate-key-file=/etc/ssl/private/keycloak.key
# https-port=8443

# Performance tuning
http-pool-max-threads=200

# Logging
log-level=INFO
log-console-output=default
```

**Create systemd service** (`/etc/systemd/system/keycloak.service`):

```ini
[Unit]
Description=Keycloak Application Server
After=network.target postgresql.service

[Service]
Type=simple
User=keycloak
Group=keycloak
Environment="KEYCLOAK_ADMIN=admin"
Environment="KEYCLOAK_ADMIN_PASSWORD=YourStrongPassword123!"
ExecStart=/opt/keycloak/bin/kc.sh start --optimized
Restart=on-failure
RestartSec=10s

# Security hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/keycloak/data

[Install]
WantedBy=multi-user.target
```

**Build and start Keycloak**:

```bash
# Build optimized Keycloak (compiles configuration)
sudo -u keycloak /opt/keycloak/bin/kc.sh build

# Reload systemd
sudo systemctl daemon-reload

# Enable and start Keycloak
sudo systemctl enable keycloak
sudo systemctl start keycloak

# Check status
sudo systemctl status keycloak

# View logs
sudo journalctl -u keycloak -f
```

### Initial Configuration

#### Accessing Admin Console

1. **Open browser**: Navigate to `http://localhost:8080` or `http://your-server-ip:8080`

2. **Welcome Screen** (first time only):
   - You'll see the Keycloak welcome page
   - Enter admin username: `admin`
   - Enter admin password: Choose a strong password
   - Click "Create"

3. **Login to Admin Console**:
   - Click "Administration Console"
   - Login with credentials you just created
   - You're now in the Keycloak Admin Console

#### Understanding the Admin Console Layout

```text
┌────────────────────────────────────────────────────────────┐
│ Keycloak                    [Master ▼]      Admin ▼       │  ← Top Bar
├────────────────────────────────────────────────────────────┤
│ ┌────────────┐  ┌──────────────────────────────────────┐  │
│ │            │  │                                      │  │
│ │  Realm     │  │                                      │  │
│ │  Settings  │  │         Dashboard / Content          │  │
│ │            │  │                                      │  │
│ │  Clients   │  │                                      │  │
│ │            │  │                                      │  │
│ │  Client    │  │                                      │  │
│ │  Scopes    │  │                                      │  │
│ │            │  │                                      │  │
│ │  Roles     │  │                                      │  │
│ │            │  │                                      │  │
│ │  Users     │  │                                      │  │
│ │            │  │                                      │  │
│ │  Groups    │  │                                      │  │
│ │            │  │                                      │  │
│ │  Sessions  │  │                                      │  │
│ │            │  │                                      │  │
│ │  Events    │  │                                      │  │
│ │            │  │                                      │  │
│ └────────────┘  └──────────────────────────────────────┘  │
│     Left Menu            Main Content Area                 │
└────────────────────────────────────────────────────────────┘
```

**Key Navigation Elements**:
- **Realm Selector** (top left): Switch between realms
- **User Menu** (top right): Admin user settings, logout
- **Left Menu**: Main navigation for realm management
- **Breadcrumbs**: Show current location in hierarchy
- **Tabs**: Context-specific options within each section

---

## 4. Realm Configuration

### Creating Your First Realm

**Important**: The `master` realm is for Keycloak administration only. Always create separate realms for your applications.[1]

**Creating a Realm via Admin Console**:

1. **Click realm selector** (top left, shows "Master")
2. **Click "Create Realm"**
3. **Enter Realm Configuration**:
   - **Realm name**: `customer-portal` (lowercase, no spaces)
   - **Enabled**: Toggle ON (default)
   - **Display name**: "Customer Portal" (user-friendly name)
   - **HTML Display name**: `<strong>Customer</strong> Portal` (optional HTML formatting)
4. **Click "Create"**

**Creating via REST API**:

```bash
# Get admin access token
ACCESS_TOKEN=$(curl -X POST http://localhost:8080/realms/master/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=admin-cli" \
  | jq -r '.access_token')

# Create realm
curl -X POST http://localhost:8080/admin/realms \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "realm": "customer-portal",
    "enabled": true,
    "displayName": "Customer Portal",
    "sslRequired": "external",
    "registrationAllowed": true,
    "registrationEmailAsUsername": true,
    "rememberMe": true,
    "verifyEmail": true,
    "loginWithEmailAllowed": true,
    "duplicateEmailsAllowed": false,
    "resetPasswordAllowed": true,
    "editUsernameAllowed": false,
    "bruteForceProtected": true
  }'
```

### General Realm Settings

**Navigate to**: Realm Settings → General

**Key Configuration Options**:

#### User Profile

**User-Facing Settings**:
- **User registration**: Allow new users to self-register[1]
  - Enable for B2C (e-commerce, SaaS free trials)
  - Disable for B2B (invite-only, enterprise)
  
- **Edit username**: Allow users to change their username[1]
  - Best Practice: Disable (use email as username)
  
- **Forgot password**: Enable password reset flow[1]
  - Requires email configuration
  
- **Remember me**: Keep users logged in across browser sessions[1]
  - Useful for consumer applications
  
- **Verify email**: Require email verification on registration[1]
  - Recommended for all CIAM scenarios
  
- **Login with email**: Use email as username[1]
  - Simplifies user experience

**Example Configuration** (B2C E-commerce):
```json
{
  "registrationAllowed": true,
  "registrationEmailAsUsername": true,
  "rememberMe": true,
  "verifyEmail": true,
  "loginWithEmailAllowed": true,
  "duplicateEmailsAllowed": false,
  "resetPasswordAllowed": true,
  "editUsernameAllowed": false
}
```

#### Email Settings

**Crucial for CIAM**: Email is the primary communication channel with customers.[1]

**Navigate to**: Realm Settings → Email

**Configuration**:

**Option 1: Standard SMTP** (Gmail Example):

```yaml
From: noreply@yourcompany.com
From Display Name: YourCompany Support
Reply To: support@yourcompany.com
Reply To Display Name: Customer Support
Envelope From: bounce@yourcompany.com

Host: smtp.gmail.com
Port: 587
Encryption: Enable StartTLS
Authentication: ON

Username: noreply@yourcompany.com
Password: [App-Specific Password]
```

**Gmail App-Specific Password Setup**:
1. Go to Google Account → Security
2. Enable 2-Step Verification
3. Generate App Password
4. Use generated password in Keycloak

**Option 2: SendGrid** (Recommended for Production):

```yaml
From: noreply@yourcompany.com
From Display Name: YourCompany
Host: smtp.sendgrid.net
Port: 587
Encryption: Enable StartTLS
Authentication: ON
Username: apikey
Password: [Your SendGrid API Key]
```

**Option 3: Amazon SES**:

```yaml
From: noreply@yourcompany.com
Host: email-smtp.us-east-1.amazonaws.com
Port: 587
Encryption: Enable StartTLS
Authentication: ON
Username: [SMTP Username from SES]
Password: [SMTP Password from SES]
```

**Option 4: Microsoft 365 with OAuth2**:[1]

```yaml
From: auth@yourcompany.com
Host: smtp.office365.com
Port: 587
Encryption: Start TLS

Authentication Type: token
Auth Token URL: https://login.microsoftonline.com/{TenantID}/oauth2/v2.0/token
Auth Token Scope: https://outlook.office.com/.default
Auth Token ClientId: {ApplicationId}
Auth Token Client Secret: {ClientSecret}
Username: auth@yourcompany.com
```

**Testing Email Configuration**:
After configuration, scroll down and click **"Test connection"** to verify SMTP settings.

**Email Templates**:
Keycloak sends various emails:
- Email verification
- Password reset
- Account update notifications
- Admin notifications

Customize templates in: Realm Settings → Themes → Email Theme

#### Login Settings

**Navigate to**: Realm Settings → Login

**User Registration Settings**:

```text
User registration: ON
  ├── Registration form action: Default (built-in form)
  ├── Registration flow: browser (can customize)
  └── Terms and conditions: Optional (link to T&C page)

Email as username: ON
  └── Benefit: Users remember email easier than custom usernames

Duplicate emails allowed: OFF
  └── Ensures one account per email address

Verify email: ON
  └── Sends verification email on registration

Login with email: ON
  └── Users can login with email

Edit username: OFF
  └── Prevents username changes (stability)

Forgot password: ON
  └── Enables self-service password reset

Remember me: ON
  └── Checkbox on login to stay logged in
```

**Real-World Example**: SaaS Application Settings

```json
{
  "registrationAllowed": true,
  "registrationEmailAsUsername": true,
  "rememberMe": true,
  "verifyEmail": true,
  "loginWithEmailAllowed": true,
  "duplicateEmailsAllowed": false,
  "resetPasswordAllowed": true,
  "editUsernameAllowed": false,
  "loginTheme": "custom-theme",
  "accountTheme": "custom-theme",
  "emailTheme": "custom-theme",
  "internationalizationEnabled": true,
  "supportedLocales": ["en", "es", "fr", "de", "ja"],
  "defaultLocale": "en"
}
```

#### SSL/HTTPS Requirements

**Navigate to**: Realm Settings → General → Require SSL

**Options**:[1]

1. **All requests** (Production):
   - SSL required for ALL connections
   - Use in production environments
   - Requires valid SSL certificate

2. **External requests** (Default):
   - SSL required for external IP addresses
   - Allows HTTP for localhost and private IPs (10.x.x.x, 192.168.x.x, 127.0.0.1)
   - Good for development/staging

3. **None** (Development Only):
   - No SSL required
   - **Never use in production**
   - Only for local development

**Production SSL Setup**:

```bash
# Using Let's Encrypt with Certbot
sudo apt install certbot

# Obtain certificate
sudo certbot certonly --standalone -d auth.yourcompany.com

# Certificates will be at:
# /etc/letsencrypt/live/auth.yourcompany.com/fullchain.pem
# /etc/letsencrypt/live/auth.yourcompany.com/privkey.pem

# Update keycloak.conf
https-certificate-file=/etc/letsencrypt/live/auth.yourcompany.com/fullchain.pem
https-certificate-key-file=/etc/letsencrypt/live/auth.yourcompany.com/privkey.pem
https-port=8443
http-enabled=false
```

**Auto-renewal** (Let's Encrypt certificates expire every 90 days):

```bash
# Add cron job
sudo crontab -e

# Add line (renew at 2am daily):
0 2 * * * certbot renew --quiet && systemctl restart keycloak
```

### Theme Customization

Keycloak has four theme types:[1]

1. **Login Theme**: Login, registration, password reset pages
2. **Account Theme**: User account management console
3. **Admin Console Theme**: Admin interface (rarely customized)
4. **Email Theme**: Email templates

**Navigate to**: Realm Settings → Themes

**Applying Themes**:

```text
Login Theme: [custom-login]
Account Theme: [custom-account]
Admin Console Theme: [keycloak.v2] (default)
Email Theme: [custom-email]
```

**Creating Custom Theme**:

```bash
# Theme directory structure
/opt/keycloak/themes/
└── custom-brand/
    ├── login/
    │   ├── theme.properties
    │   ├── messages/
    │   │   ├── messages_en.properties
    │   │   └── messages_es.properties
    │   ├── resources/
    │   │   ├── css/
    │   │   │   └── login.css
    │   │   └── img/
    │   │       └── logo.png
    │   └── template.ftl
    ├── account/
    │   └── ... (similar structure)
    └── email/
        └── ... (similar structure)
```

**Example**: `theme.properties`

```properties
parent=keycloak
import=common/keycloak

styles=css/login.css

locales=en,es,fr,de

# Social providers
social.displayName=Sign in with
```

**Custom CSS** (`resources/css/login.css`):

```css
/* Brand colors */
:root {
    --brand-primary: #2563eb;
    --brand-secondary: #1e40af;
    --brand-text: #1f2937;
}

/* Login card */
#kc-header {
    background-color: var(--brand-primary);
}

#kc-header-wrapper {
    font-family: 'Inter', sans-serif;
    color: white;
    font-size: 28px;
    font-weight: 600;
}

/* Logo */
#kc-logo-wrapper img {
    max-height: 80px;
    width: auto;
}

/* Form styling */
.login-pf-page .card-pf {
    border-radius: 12px;
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}

/* Primary button */
#kc-form-buttons .btn-primary {
    background-color: var(--brand-primary);
    border-color: var(--brand-primary);
    border-radius: 6px;
    padding: 12px 24px;
    font-weight: 500;
}

#kc-form-buttons .btn-primary:hover {
    background-color: var(--brand-secondary);
    border-color: var(--brand-secondary);
}

/* Social buttons */
.kc-social-links button {
    border-radius: 6px;
    margin: 4px;
}
```

**Custom Messages** (`messages/messages_en.properties`):

```properties
loginTitle=Welcome Back!
loginWelcomeMessage=Sign in to your account
usernameOrEmail=Email Address
password=Password
doLogIn=Sign In
registerTitle=Create Account
registerWelcomeMessage=Join thousands of happy customers
doRegister=Create Account
backToLogin=Already have an account? Sign in

# Custom error messages
invalidUserMessage=Oops! We couldn't find an account with that email.
invalidPasswordMessage=That password doesn't match our records. Try again?
emailVerifyInstruction=We've sent a verification email to {0}. Please check your inbox!
```

**Reload Theme** (development mode):

```bash
# Clear theme cache
rm -rf /opt/keycloak/data/tmp/*

# Restart Keycloak
sudo systemctl restart keycloak
```

### Internationalization (i18n)

**Navigate to**: Realm Settings → Localization

**Enable Internationalization**:

```text
Internationalization: ON
Supported Locales: 
  ☑ English (en)
  ☑ Spanish (es)
  ☑ French (fr)
  ☑ German (de)
  ☑ Japanese (ja)
  ☑ Chinese Simplified (zh-CN)
Default Locale: en
```

**Locale Selection Priority**:[1]

```text
1. User-selected locale (dropdown on login page)
   ↓ If not selected
2. User profile preferred locale (account setting)
   ↓ If not set
3. Client-specified locale (ui_locales parameter in OAuth request)
   ↓ If not provided
4. Browser cookie (kc-locale)
   ↓ If not present
5. Accept-Language header from browser
   ↓ If not supported
6. Realm default locale
   ↓ Final fallback
7. English (en)
```

**OAuth Request with Locale**:

```javascript
const authUrl = `https://auth.example.com/realms/customer-portal/protocol/openid-connect/auth?` +
  `client_id=web-app&` +
  `response_type=code&` +
  `scope=openid profile email&` +
  `redirect_uri=${encodeURIComponent('https://app.example.com/callback')}&` +
  `ui_locales=es`; // Request Spanish UI

window.location.href = authUrl;
```

**Custom Translations** (`messages/messages_es.properties`):

```properties
loginTitle=¡Bienvenido de nuevo!
loginWelcomeMessage=Inicia sesión en tu cuenta
usernameOrEmail=Correo electrónico
password=Contraseña
doLogIn=Iniciar sesión
registerTitle=Crear cuenta
registerWelcomeMessage=Únete a miles de clientes satisfechos
doRegister=Crear cuenta
backToLogin=¿Ya tienes una cuenta? Inicia sesión
```

### Token Settings

**Navigate to**: Realm Settings → Tokens

**Token Lifespan Configuration**:

```text
SSO Session Idle: 30 minutes
  └── How long until inactive session expires

SSO Session Max: 10 hours
  └── Maximum session duration regardless of activity

SSO Session Idle Remember Me: 7 days
  └── Extended timeout when "Remember Me" checked

SSO Session Max Remember Me: 30 days
  └── Max duration for "Remember Me" sessions

Client Session Idle: 30 minutes
  └── Per-client session timeout (0 = use SSO settings)

Client Session Max: 10 hours
  └── Per-client max duration (0 = use SSO settings)

Access Token Lifespan: 5 minutes
  └── How long access tokens are valid (short = more secure)

Access Token Lifespan For Implicit Flow: 15 minutes
  └── Implicit flow tokens (deprecated, use Authorization Code)

Client login timeout: 1 minute
  └── How long OAuth authorization code is valid

Login timeout: 5 minutes
  └── How long user has to complete login

Login action timeout: 5 minutes
  └── How long to complete required actions (OTP setup, etc.)

Offline Session Idle: 30 days
  └── Timeout for offline tokens when not used

Offline Session Max: 60 days
  └── Maximum duration for offline tokens
```

**Real-World Token Strategy**:

**Scenario 1: Banking Application** (High Security):
```properties
access-token-lifespan=300          # 5 minutes
sso-session-idle=900               # 15 minutes
sso-session-max=3600               # 1 hour
remember-me=false                  # Disabled
```

**Scenario 2: E-commerce Site** (Balanced):
```properties
access-token-lifespan=300          # 5 minutes
sso-session-idle=1800              # 30 minutes
sso-session-max=36000              # 10 hours
sso-session-idle-remember-me=604800 # 7 days
sso-session-max-remember-me=2592000 # 30 days
```

**Scenario 3: Social Media Platform** (Convenience):
```properties
access-token-lifespan=600          # 10 minutes
sso-session-idle=86400             # 24 hours
sso-session-max=604800             # 7 days
sso-session-idle-remember-me=2592000 # 30 days
sso-session-max-remember-me=7776000  # 90 days
```

**Why Short Access Tokens?**:
- Access tokens are bearer tokens (whoever has it can use it)
- Short lifespan limits damage if token is compromised
- Refresh tokens are used to get new access tokens
- Refresh tokens can be revoked immediately

***

## 5. User and Role Management

### Creating and Managing Users

In a CIAM system, users are your customers, and managing them effectively is crucial for business success.

#### Creating Users via Admin Console

**Navigate to**: Users → Add user

**Step-by-Step Process**:

1. **Click "Add user"** button
2. **Fill in user details**:

```text
General Information:
┌─────────────────────────────────────────────┐
│ Username*: john.customer@example.com        │
│ Email: john.customer@example.com            │
│ Email verified: ☑ ON                        │
│ First name: John                            │
│ Last name: Customer                         │
│ Enabled: ☑ ON                               │
└─────────────────────────────────────────────┘

Required Actions: (Actions user must complete on first login)
☐ Verify Email
☐ Update Profile
☐ Update Password
☐ Configure OTP

Groups: (Select groups)
☐ Premium Customers
☐ Newsletter Subscribers

(*Required fields)
```

3. **Click "Create"**
4. **Set Password**:
   - Navigate to **Credentials** tab
   - Click **Set password**
   - Enter password
   - Toggle **Temporary** OFF (permanent password)
   - Click **Save**

**Setting Custom Attributes**:

After creating user, navigate to **Attributes** tab:

```text
Add custom attributes:
┌────────────────────────────────────────────────┐
│ Key              │ Value                       │
├────────────────────────────────────────────────┤
│ customer_tier    │ gold                        │
│ subscription_id  │ sub_ABC123XYZ               │
│ account_type     │ individual                  │
│ phone_number     │ +1-555-123-4567             │
│ date_of_birth    │ 1990-05-15                  │
│ loyalty_points   │ 15000                       │
│ marketing_consent│ true                        │
│ preferred_lang   │ en-US                       │
│ account_manager  │ alice@company.com           │
└────────────────────────────────────────────────┘
```

These attributes appear in access tokens and can be used for personalization and business logic.

#### Creating Users via REST API

**Authentication First**:

```bash
# Get admin access token
ACCESS_TOKEN=$(curl -s -X POST http://localhost:8080/realms/master/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=admin-cli" \
  | jq -r '.access_token')
```

**Create User**:

```bash
curl -X POST http://localhost:8080/admin/realms/customer-portal/users \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "jane.customer@example.com",
    "email": "jane.customer@example.com",
    "emailVerified": true,
    "enabled": true,
    "firstName": "Jane",
    "lastName": "Customer",
    "attributes": {
      "customer_tier": ["platinum"],
      "subscription_id": ["sub_DEF456UVW"],
      "account_type": ["business"],
      "phone_number": ["+1-555-987-6543"],
      "loyalty_points": ["25000"],
      "marketing_consent": ["true"],
      "preferred_language": ["en-US"]
    },
    "groups": ["/Premium Customers"],
    "credentials": [{
      "type": "password",
      "value": "SecurePassword123!",
      "temporary": false
    }]
  }'
```

**Response**:
```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "createdTimestamp": 1729642800000,
  "username": "jane.customer@example.com",
  "enabled": true,
  "emailVerified": true
}
```

**Get User ID from Response**:

```bash
USER_ID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/users?username=jane.customer@example.com" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[0].id')

echo "User ID: $USER_ID"
```

#### Bulk User Import

For migrating existing customers or loading test data:

**Create JSON File** (`users-import.json`):

```json
{
  "realm": "customer-portal",
  "users": [
    {
      "username": "customer1@example.com",
      "email": "customer1@example.com",
      "emailVerified": true,
      "enabled": true,
      "firstName": "Alice",
      "lastName": "Smith",
      "attributes": {
        "customer_tier": ["silver"],
        "loyalty_points": ["5000"]
      },
      "credentials": [{
        "type": "password",
        "value": "TempPassword123!",
        "temporary": true
      }]
    },
    {
      "username": "customer2@example.com",
      "email": "customer2@example.com",
      "emailVerified": true,
      "enabled": true,
      "firstName": "Bob",
      "lastName": "Johnson",
      "attributes": {
        "customer_tier": ["gold"],
        "loyalty_points": ["12000"]
      },
      "credentials": [{
        "type": "password",
        "value": "TempPassword123!",
        "temporary": true
      }]
    }
  ]
}
```

**Import via CLI**:

```bash
/opt/keycloak/bin/kc.sh import --file users-import.json
```

**Import via Script** (for large datasets):

```bash
#!/bin/bash
# bulk-import.sh

REALM="customer-portal"
CSV_FILE="customers.csv"

# Get admin token
ACCESS_TOKEN=$(curl -s -X POST http://localhost:8080/realms/master/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin" \
  -d "password=admin" \
  -d "grant_type=password" \
  -d "client_id=admin-cli" \
  | jq -r '.access_token')

# Read CSV and create users
while IFS=',' read -r email firstName lastName tier points
do
  echo "Creating user: $email"
  
  curl -s -X POST "http://localhost:8080/admin/realms/$REALM/users" \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{
      \"username\": \"$email\",
      \"email\": \"$email\",
      \"emailVerified\": true,
      \"enabled\": true,
      \"firstName\": \"$firstName\",
      \"lastName\": \"$lastName\",
      \"attributes\": {
        \"customer_tier\": [\"$tier\"],
        \"loyalty_points\": [\"$points\"]
      },
      \"credentials\": [{
        \"type\": \"password\",
        \"value\": \"TempPass123!\",
        \"temporary\": true
      }]
    }"
    
  sleep 0.1 # Rate limiting
done < "$CSV_FILE"

echo "Import complete!"
```

**CSV File Format** (`customers.csv`):

```csv
email,firstName,lastName,tier,points
alice@example.com,Alice,Smith,gold,12000
bob@example.com,Bob,Johnson,silver,6000
charlie@example.com,Charlie,Williams,platinum,25000
```

**Run import**:
```bash
chmod +x bulk-import.sh
./bulk-import.sh
```

#### User Search and Filtering

**Admin Console Search**:

```text
Users → Search bar

Search by:
- Username: "john"
- Email: "@example.com"
- First/Last name: "Smith"
- Attribute: "customer_tier:gold"

Filters:
☐ Show only enabled users
☐ Show only email verified
☐ Show users without email
```

**REST API Search**:

```bash
# Search by email
curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/users?email=@example.com" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq

# Search by username
curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/users?username=john" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq

# Search with pagination
curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/users?first=0&max=20" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq

# Get user count
curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/users/count" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

#### User Lifecycle Management

**Account Activation/Deactivation**:

```bash
# Disable user account (soft delete)
curl -X PUT "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"enabled": false}'

# Re-enable account
curl -X PUT "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"enabled": true}'
```

**Force Password Reset**:

```bash
curl -X PUT "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"requiredActions": ["UPDATE_PASSWORD"]}'
```

**Send Verification Email**:

```bash
curl -X PUT "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/send-verify-email" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -d "client_id=web-app"
```

**Delete User** (Hard delete - irreversible):

```bash
curl -X DELETE "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

### Creating and Managing Roles

Roles define what users can do. In CIAM, roles often correspond to subscription tiers, feature access, or administrative privileges.

#### Realm Roles (Global)

**Creating Realm Roles via Admin Console**:

**Navigate to**: Realm roles → Create role

**Example: E-commerce Platform Roles**

```text
Role 1: Basic Customer
┌─────────────────────────────────────────┐
│ Role name*: basic_customer              │
│ Description: Free tier customer with    │
│              basic access                │
└─────────────────────────────────────────┘

Role 2: Premium Customer
┌─────────────────────────────────────────┐
│ Role name*: premium_customer            │
│ Description: Paid subscription with     │
│              premium features            │
└─────────────────────────────────────────┘

Role 3: VIP Customer
┌─────────────────────────────────────────┐
│ Role name*: vip_customer                │
│ Description: VIP tier with exclusive    │
│              benefits                    │
└─────────────────────────────────────────┘

Role 4: Support Agent
┌─────────────────────────────────────────┐
│ Role name*: support_agent               │
│ Description: Customer support staff     │
└─────────────────────────────────────────┘
```

**Creating Roles via REST API**:

```bash
# Create basic_customer role
curl -X POST "http://localhost:8080/admin/realms/customer-portal/roles" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "basic_customer",
    "description": "Free tier customer with basic access",
    "composite": false,
    "clientRole": false
  }'

# Create premium_customer role
curl -X POST "http://localhost:8080/admin/realms/customer-portal/roles" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "premium_customer",
    "description": "Paid subscription with premium features",
    "composite": false,
    "clientRole": false
  }'
```

#### Client Roles (Application-Specific)

Client roles are specific to individual applications, allowing fine-grained access control per application.

**Real-World Scenario**: Multi-App Ecosystem

```text
Client: web-store
├── customer: Can browse and purchase
├── vendor: Can list and manage products
├── affiliate: Can create referral links
└── store_admin: Can manage store settings

Client: mobile-app
├── mobile_user: Can use mobile features
├── offline_access: Can download content offline
└── beta_tester: Can access beta features

Client: admin-portal
├── content_manager: Can manage content
├── user_manager: Can manage users
├── billing_admin: Can manage billing
└── system_admin: Full administrative access
```

**Creating Client Roles**:

**Navigate to**: Clients → Select client (e.g., "web-store") → Roles → Create role

```text
┌─────────────────────────────────────────┐
│ Role name*: vendor                      │
│ Description: Can list and manage        │
│              products in store          │
└─────────────────────────────────────────┘
```

**Via REST API**:

```bash
# Get client UUID (needed for API calls)
CLIENT_UUID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/clients?clientId=web-store" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[0].id')

# Create client role
curl -X POST "http://localhost:8080/admin/realms/customer-portal/clients/${CLIENT_UUID}/roles" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "vendor",
    "description": "Can list and manage products in store",
    "composite": false,
    "clientRole": true
  }'
```

#### Composite Roles (Role Hierarchies)

Composite roles simplify management by creating role hierarchies. When you assign a composite role, users automatically get all included roles.

**Real-World Example**: Customer Support Tiers

```text
Tier 1: support_agent_tier1
├── view_customer_profiles
├── view_orders
└── create_support_tickets

Tier 2: support_agent_tier2 (composite)
├── Includes: support_agent_tier1 (inherits all above)
├── process_refunds
├── modify_orders
└── escalate_issues

Tier 3: support_manager (composite)
├── Includes: support_agent_tier2 (inherits all above)
├── view_analytics
├── manage_team
├── access_audit_logs
└── override_policies
```

**Creating Composite Roles**:

**Navigate to**: Realm roles → Create role → Toggle "Composite role" ON

```text
┌─────────────────────────────────────────────────┐
│ Role name*: support_manager                     │
│ Description: Support team manager with full     │
│              access                              │
│ Composite role: ☑ ON                            │
└─────────────────────────────────────────────────┘

Associated Roles:
┌─────────────────────────────────────────────────┐
│ Realm Roles:                                    │
│ ☑ support_agent_tier2                           │
│ ☑ view_analytics                                │
│ ☑ manage_team                                   │
│ ☑ access_audit_logs                             │
│                                                 │
│ Client Roles (admin-portal):                    │
│ ☑ user_manager                                  │
│ ☑ billing_admin                                 │
└─────────────────────────────────────────────────┘
```

**Via REST API**:

```bash
# Create composite role
curl -X POST "http://localhost:8080/admin/realms/customer-portal/roles" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "support_manager",
    "description": "Support team manager with full access",
    "composite": true,
    "clientRole": false
  }'

# Get role representations for composites
TIER2_ROLE=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/roles/support_agent_tier2" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}")

ANALYTICS_ROLE=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/roles/view_analytics" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}")

# Add composite mappings
curl -X POST "http://localhost:8080/admin/realms/customer-portal/roles/support_manager/composites" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "[${TIER2_ROLE}, ${ANALYTICS_ROLE}]"
```

#### Assigning Roles to Users

**Via Admin Console**:

**Navigate to**: Users → Select user → Role mapping → Assign role

```text
┌─────────────────────────────────────────────────┐
│ Filter by realm roles or clients:               │
│ [Realm roles ▼]                                 │
│                                                 │
│ Available Roles:                                │
│ ☐ basic_customer                                │
│ ☑ premium_customer      [Assign]               │
│ ☐ vip_customer                                  │
│ ☐ support_agent                                 │
│                                                 │
│ Assigned Roles:                                 │
│ • default-roles-customer-portal  [Remove]       │
│ • premium_customer               [Remove]       │
└─────────────────────────────────────────────────┘
```

**Via REST API**:

```bash
# Get user ID
USER_ID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/users?username=john.customer@example.com" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[0].id')

# Get role representation
ROLE=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/roles/premium_customer" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}")

# Assign role to user
curl -X POST "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/role-mappings/realm" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "[${ROLE}]"

# Remove role from user
curl -X DELETE "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/role-mappings/realm" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "[${ROLE}]"
```

**Assigning Client Roles**:

```bash
# Get client UUID
CLIENT_UUID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/clients?clientId=web-store" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[0].id')

# Get client role
CLIENT_ROLE=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/clients/${CLIENT_UUID}/roles/vendor" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}")

# Assign client role to user
curl -X POST "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/role-mappings/clients/${CLIENT_UUID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "[${CLIENT_ROLE}]"
```

**Viewing User's Effective Roles** (including composite):

```bash
curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/role-mappings" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq
```

**Response**:
```json
{
  "realmMappings": [
    {
      "id": "role-id-1",
      "name": "support_manager",
      "composite": true,
      "clientRole": false
    }
  ],
  "clientMappings": {
    "web-store": [
      {
        "id": "role-id-2",
        "name": "vendor",
        "composite": false,
        "clientRole": true
      }
    ]
  }
}
```

### Creating and Managing Groups

Groups organize users hierarchically and simplify role assignment and attribute management.

#### Creating Group Hierarchies

**Real-World Scenario**: B2B SaaS Platform with Multiple Organizations

```text
Root
├── Organization: Acme Corp
│   ├── Department: Sales
│   │   ├── Team: Enterprise Sales
│   │   │   ├── User: alice@acme.com
│   │   │   └── User: bob@acme.com
│   │   └── Team: SMB Sales
│   │       ├── User: charlie@acme.com
│   │       └── User: diana@acme.com
│   ├── Department: Engineering
│   │   ├── Team: Backend
│   │   │   ├── User: eve@acme.com
│   │   │   └── User: frank@acme.com
│   │   └── Team: Frontend
│   │       └── User: grace@acme.com
│   └── Department: Support
│       └── User: henry@acme.com (Support Manager)
│
└── Organization: Beta Industries
    ├── Department: Operations
    │   └── User: iris@beta.com
    └── Department: Finance
        └── User: jack@beta.com
```

**Creating Groups via Admin Console**:

**Navigate to**: Groups → Create group

```text
Step 1: Create root organization
┌─────────────────────────────────────────┐
│ Name*: Acme Corp                        │
└─────────────────────────────────────────┘
[Create]

Step 2: Create child groups
Select "Acme Corp" → Create child group
┌─────────────────────────────────────────┐
│ Name*: Sales                            │
└─────────────────────────────────────────┘
[Create]

Step 3: Add more nested levels
Select "Sales" → Create child group
┌─────────────────────────────────────────┐
│ Name*: Enterprise Sales                 │
└─────────────────────────────────────────┘
[Create]
```

**Via REST API**:

```bash
# Create parent group (Organization)
ORG_RESPONSE=$(curl -s -X POST "http://localhost:8080/admin/realms/customer-portal/groups" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme Corp",
    "attributes": {
      "organization_id": ["org_acme_001"],
      "subscription_tier": ["enterprise"],
      "max_users": ["500"],
      "billing_contact": ["billing@acme.com"]
    }
  }')

# Get the created group's ID from Location header
ORG_ID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/groups?search=Acme Corp" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[0].id')

# Create child group (Department)
DEPT_RESPONSE=$(curl -s -X POST "http://localhost:8080/admin/realms/customer-portal/groups/${ORG_ID}/children" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sales",
    "attributes": {
      "cost_center": ["CC-SALES-001"],
      "manager": ["sales-manager@acme.com"],
      "region": ["North America"]
    }
  }')

# Get Sales group ID
SALES_ID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/groups/${ORG_ID}/children" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[] | select(.name=="Sales") | .id')

# Create sub-group (Team)
curl -X POST "http://localhost:8080/admin/realms/customer-portal/groups/${SALES_ID}/children" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Enterprise Sales",
    "attributes": {
      "team_lead": ["alice@acme.com"],
      "quota": ["5000000"],
      "territory": ["Fortune 500"]
    }
  }'
```

#### Group Attributes

Groups can carry attributes that are inherited by all members. This is powerful for multi-tenancy and organizational context.

**Navigate to**: Groups → Select group → Attributes

```text
Group: Acme Corp
┌────────────────────────────────────────────────┐
│ Key                    │ Value                 │
├────────────────────────────────────────────────┤
│ organization_id        │ org_acme_001          │
│ subscription_tier      │ enterprise            │
│ max_users              │ 500                   │
│ billing_contact        │ billing@acme.com      │
│ support_level          │ premium               │
│ custom_domain          │ acme.myapp.com        │
│ api_rate_limit         │ 10000                 │
│ storage_quota_gb       │ 1000                  │
└────────────────────────────────────────────────┘
```

**Token Claim Example**:

When a user from "Acme Corp → Sales → Enterprise Sales" logs in, their token includes:

```json
{
  "sub": "alice-user-id",
  "email": "alice@acme.com",
  "name": "Alice Smith",
  "groups": [
    "/Acme Corp",
    "/Acme Corp/Sales",
    "/Acme Corp/Sales/Enterprise Sales"
  ],
  "organization_id": "org_acme_001",
  "subscription_tier": "enterprise",
  "support_level": "premium",
  "cost_center": "CC-SALES-001",
  "team_lead": "alice@acme.com",
  "quota": "5000000"
}
```

Your application can use these claims for:
- Multi-tenant data isolation
- Feature flags based on subscription tier
- Rate limiting based on organization
- Custom branding per organization
- Usage tracking and billing

#### Assigning Roles to Groups

Instead of assigning roles to individual users, assign them to groups. All group members inherit the roles.

**Navigate to**: Groups → Select group → Role mapping → Assign role

**Example: Department-Level Permissions**

```text
Group: Acme Corp
└── Assigned Roles: [basic_access]

Group: Acme Corp/Sales
└── Assigned Roles: [crm_access, sales_dashboard, customer_data_view]

Group: Acme Corp/Engineering
└── Assigned Roles: [code_repository_access, deployment_access, staging_access]

Group: Acme Corp/Support
└── Assigned Roles: [support_portal_access, ticket_management, customer_data_view]
```

**Via REST API**:

```bash
# Get group ID
GROUP_ID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/groups?search=Sales" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[0].id')

# Get role representation
CRM_ROLE=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/roles/crm_access" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}")

DASHBOARD_ROLE=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/roles/sales_dashboard" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}")

# Assign roles to group
curl -X POST "http://localhost:8080/admin/realms/customer-portal/groups/${GROUP_ID}/role-mappings/realm" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "[${CRM_ROLE}, ${DASHBOARD_ROLE}]"
```

#### Adding Users to Groups

**Via Admin Console**:

**Navigate to**: Users → Select user → Groups → Join group

```text
Available Groups:
┌─────────────────────────────────────────────────┐
│ Search: acme                                    │
├─────────────────────────────────────────────────┤
│ ☐ /Acme Corp                                    │
│   ☐ /Acme Corp/Sales                            │
│     ☑ /Acme Corp/Sales/Enterprise Sales [Join]  │
│   ☐ /Acme Corp/Engineering                      │
│   ☐ /Acme Corp/Support                          │
└─────────────────────────────────────────────────┘

User's Groups:
• /Acme Corp/Sales/Enterprise Sales [Leave]
```

**Via REST API**:

```bash
# Get user ID
USER_ID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/users?username=alice@acme.com" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[0].id')

# Get group ID
GROUP_ID=$(curl -s -X GET "http://localhost:8080/admin/realms/customer-portal/groups?search=Enterprise Sales" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  | jq -r '.[0].id')

# Add user to group
curl -X PUT "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/groups/${GROUP_ID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"

# Remove user from group
curl -X DELETE "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/groups/${GROUP_ID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

#### Group Membership Inheritance

Users inherit attributes and roles from ALL parent groups in the hierarchy.

**Example Inheritance Chain**:

```text
User: alice@acme.com
Member of: /Acme Corp/Sales/Enterprise Sales

Inherited from /Acme Corp:
├── Attributes: organization_id, subscription_tier, max_users
└── Roles: basic_access

Inherited from /Acme Corp/Sales:
├── Attributes: cost_center, manager, region
└── Roles: crm_access, sales_dashboard

Inherited from /Acme Corp/Sales/Enterprise Sales:
├── Attributes: team_lead, quota, territory
└── Roles: enterprise_deals_access

Effective Permissions for alice@acme.com:
├── All attributes from 3 levels
└── Roles: basic_access, crm_access, sales_dashboard, enterprise_deals_access
```

***

## 6. Client Configuration (OpenID Connect & SAML)

Clients represent applications that delegate authentication to Keycloak. Proper client configuration is crucial for security and functionality.

### Understanding Client Types

**1. Confidential Clients** (Can store secrets securely):[1]
- Backend web applications (Node.js, Spring Boot, Django)
- Server-side applications
- API gateways
- Microservices with secure storage

**2. Public Clients** (Cannot store secrets):[1]
- Single Page Applications (React, Angular, Vue)
- Mobile apps (iOS, Android, React Native)
- Desktop applications
- Command-line tools

**3. Bearer-Only Clients** (Only validate tokens):[1]
- REST APIs
- Resource servers
- Microservices

### Creating an OpenID Connect Client

OpenID Connect (OIDC) is the recommended modern authentication protocol.[2][1]

#### Confidential Web Application

**Real-World Scenario**: E-commerce Backend Application

**Navigate to**: Clients → Create client

**Step 1: General Settings**

```text
Client type: OpenID Connect
Client ID*: web-store-backend

[Next]
```

**Step 2: Capability config**

```text
Client authentication: ☑ ON
  └── This is a confidential client

Authorization: ☐ OFF
  └── Enable for fine-grained authorization (optional)

Authentication flow:
☑ Standard flow
  └── Authorization Code Flow (recommended)
☑ Direct access grants
  └── Resource Owner Password Credentials (use sparingly)
☐ Implicit flow
  └── Deprecated, don't use
☑ Service accounts roles
  └── Enable for machine-to-machine (client credentials flow)

[Next]
```

**Step 3: Login settings**

```text
Root URL: https://shop.example.com
Home URL: /
Valid redirect URIs*: 
  https://shop.example.com/auth/callback
  https://shop.example.com/auth/silent-refresh
  http://localhost:3000/* (development only)

Valid post logout redirect URIs:
  https://shop.example.com
  https://shop.example.com/goodbye

Web origins: 
  https://shop.example.com
  http://localhost:3000 (development only)

Admin URL: https://shop.example.com/admin

[Save]
```

**Step 4: Get Client Secret**

After creating, navigate to **Credentials** tab:

```text
Client Authenticator: Client Id and Secret
┌─────────────────────────────────────────┐
│ Client Secret: [Regenerate]              │
│ XbQ3vK9L...2mP8nY (hidden)              │
│ [Copy] [Show]                           │
└─────────────────────────────────────────┘
```

**Copy this secret** - you'll need it in your application configuration.

**Complete Configuration JSON**:

```json
{
  "clientId": "web-store-backend",
  "name": "Web Store Backend",
  "description": "Main e-commerce application backend",
  "rootUrl": "https://shop.example.com",
  "adminUrl": "https://shop.example.com/admin",
  "baseUrl": "/",
  "surrogateAuthRequired": false,
  "enabled": true,
  "alwaysDisplayInConsole": false,
  "clientAuthenticatorType": "client-secret",
  "secret": "**********",
  "redirectUris": [
    "https://shop.example.com/auth/callback",
    "https://shop.example.com/auth/silent-refresh",
    "http://localhost:3000/*"
  ],
  "webOrigins": [
    "https://shop.example.com",
    "http://localhost:3000"
  ],
  "notBefore": 0,
  "bearerOnly": false,
  "consentRequired": false,
  "standardFlowEnabled": true,
  "implicitFlowEnabled": false,
  "directAccessGrantsEnabled": true,
  "serviceAccountsEnabled": true,
  "publicClient": false,
  "frontchannelLogout": true,
  "protocol": "openid-connect",
  "attributes": {
    "oidc.ciba.grant.enabled": "false",
    "backchannel.logout.session.required": "true",
    "oauth2.device.authorization.grant.enabled": "false",
    "display.on.consent.screen": "false",
    "backchannel.logout.revoke.offline.tokens": "false"
  },
  "authenticationFlowBindingOverrides": {},
  "fullScopeAllowed": true,
  "nodeReRegistrationTimeout": -1,
  "defaultClientScopes": [
    "web",
    "acr",
    "profile",
    "roles",
    "email"
  ],
  "optionalClientScopes": [
    "address",
    "phone",
    "offline_access",
    "microprofile-jwt"
  ]
}
```

#### Public Single Page Application (SPA)

**Real-World Scenario**: React E-commerce Frontend

**Create Client**:

```text
Client type: OpenID Connect
Client ID*: web-store-spa

Capability config:
Client authentication: ☐ OFF
  └── This is a public client (no secret)
Authorization: ☐ OFF

Authentication flow:
☑ Standard flow (with PKCE)
☐ Direct access grants
☐ Implicit flow (deprecated)
☐ Service accounts roles

Login settings:
Root URL: https://shop.example.com
Valid redirect URIs: 
  https://shop.example.com/callback
  http://localhost:3000/callback
Valid post logout redirect URIs:
  https://shop.example.com
  +
Web origins:
  https://shop.example.com
  http://localhost:3000
```

**Advanced Settings** (Important for SPAs):

Navigate to: Client → Advanced

```text
Proof Key for Code Exchange Code Challenge Method: S256
  └── PKCE protects public clients from authorization code interception

Access Token Lifespan: 5 minutes
  └── Short-lived for security

OAuth 2.0 Mutual TLS Certificate Bound Access Tokens Enabled: OFF
  └── Enable for extra security if using mTLS
```

**React Integration Example**:

```javascript
// src/keycloak.js
import Keycloak from 'keycloak-js';

const keycloak = new Keycloak({
  url: 'https://auth.example.com',
  realm: 'customer-portal',
  clientId: 'web-store-spa'
});

export default keycloak;
```

```javascript
// src/App.js
import React from 'react';
import { ReactKeycloakProvider } from '@react-keycloak/web';
import keycloak from './keycloak';
import MainApp from './MainApp';

function App() {
  return (
    <ReactKeycloakProvider
      authClient={keycloak}
      initOptions={{
        onLoad: 'check-sso',
        silentCheckSsoRedirectUri: window.location.origin + '/silent-check-sso.html',
        pkceMethod: 'S256', // Enable PKCE
        checkLoginIframe: false // Disable for better performance
      }}
      onTokens={(tokens) => {
        console.log('Tokens received:', tokens);
        // Store tokens in state management (Redux, Context, etc.)
      }}
    >
      <MainApp />
    </ReactKeycloakProvider>
  );
}

export default App;
```

```html
<!-- public/silent-check-sso.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Silent SSO Check</title>
</head>
<body>
    <script>
        parent.postMessage(location.href, location.origin);
    </script>
</body>
</html>
```

```javascript
// src/components/ProtectedRoute.js
import { useKeycloak } from '@react-keycloak/web';
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ children, roles }) {
  const { keycloak, initialized } = useKeycloak();

  if (!initialized) {
    return <div>Loading...</div>;
  }

  if (!keycloak.authenticated) {
    keycloak.login();
    return null;
  }

  // Check roles if required
  if (roles) {
    const hasRole = roles.some(role => keycloak.hasRealmRole(role));
    if (!hasRole) {
      return <Navigate to="/forbidden" />;
    }
  }

  return children;
}

export default ProtectedRoute;
```

```javascript
// src/api/apiClient.js
import axios from 'axios';
import keycloak from '../keycloak';

const apiClient = axios.create({
  baseURL: 'https://api.example.com'
});

// Add token to all requests
apiClient.interceptors.request.use(
  async (config) => {
    if (keycloak.authenticated) {
      // Refresh token if expiring soon (30 seconds)
      await keycloak.updateToken(30);
      config.headers.Authorization = `Bearer ${keycloak.token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Handle 401 unauthorized
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      keycloak.login();
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

#### Mobile Application (iOS/Android)

**Real-World Scenario**: React Native E-commerce App

**Create Client**:

```text
Client ID: mobile-app-ios

Capability config:
Client authentication: ☐ OFF (public client)
Authentication flow:
☑ Standard flow (with PKCE)

Login settings:
Valid redirect URIs:
  myapp://callback
  com.example.myapp://callback

Valid post logout redirect URIs:
  myapp://logout
```

**React Native Integration** (using `react-native-app-auth`):

```bash
npm install react-native-app-auth
```

```javascript
// src/services/authService.js
import { authorize, refresh, revoke } from 'react-native-app-auth';
import { Platform } from 'react-native';

const config = {
  issuer: 'https://auth.example.com/realms/customer-portal',
  clientId: Platform.OS === 'ios' ? 'mobile-app-ios' : 'mobile-app-android',
  redirectUrl: Platform.OS === 'ios' 
    ? 'com.example.myapp://callback'
    : 'com.example.myapp://callback',
  scopes: ['openid', 'profile', 'email', 'offline_access'],
  serviceConfiguration: {
    authorizationEndpoint: 'https://auth.example.com/realms/customer-portal/protocol/openid-connect/auth',
    tokenEndpoint: 'https://auth.example.com/realms/customer-portal/protocol/openid-connect/token',
    revocationEndpoint: 'https://auth.example.com/realms/customer-portal/protocol/openid-connect/revoke',
    endSessionEndpoint: 'https://auth.example.com/realms/customer-portal/protocol/openid-connect/logout'
  },
  usePKCE: true, // Always use PKCE for mobile
  dangerouslyAllowInsecureHttpRequests: __DEV__, // Only in development
};

export const login = async () => {
  try {
    const result = await authorize(config);
    return {
      accessToken: result.accessToken,
      refreshToken: result.refreshToken,
      idToken: result.idToken,
      accessTokenExpirationDate: result.accessTokenExpirationDate
    };
  } catch (error) {
    console.error('Login error:', error);
    throw error;
  }
};

export const refreshTokens = async (refreshToken) => {
  try {
    const result = await refresh(config, {
      refreshToken
    });
    return {
      accessToken: result.accessToken,
      refreshToken: result.refreshToken,
      accessTokenExpirationDate: result.accessTokenExpirationDate
    };
  } catch (error) {
    console.error('Refresh error:', error);
    throw error;
  }
};

export const logout = async (tokens) => {
  try {
    await revoke(config, {
      tokenToRevoke: tokens.refreshToken,
      sendClientId: true
    });
  } catch (error) {
    console.error('Logout error:', error);
  }
};
```

```javascript
// src/App.js
import React, { useState, useEffect } from 'react';
import { View, Button, Text } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { login, refreshTokens, logout } from './services/authService';

function App() {
  const [tokens, setTokens] = useState(null);
  const [user, setUser] = useState(null);

  useEffect(() => {
    loadTokens();
  }, []);

  const loadTokens = async () => {
    const storedTokens = await AsyncStorage.getItem('tokens');
    if (storedTokens) {
      const parsedTokens = JSON.parse(storedTokens);
      // Check if token is expired
      const now = new Date().getTime();
      const expiration = new Date(parsedTokens.accessTokenExpirationDate).getTime();
      
      if (now < expiration) {
        setTokens(parsedTokens);
        decodeToken(parsedTokens.idToken);
      } else {
        // Try to refresh
        try {
          const refreshed = await refreshTokens(parsedTokens.refreshToken);
          setTokens(refreshed);
          await AsyncStorage.setItem('tokens', JSON.stringify(refreshed));
          decodeToken(refreshed.idToken);
        } catch (error) {
          // Refresh failed, need to login again
          await AsyncStorage.removeItem('tokens');
        }
      }
    }
  };

  const decodeToken = (idToken) => {
    const base64Url = idToken.split('.')[1];
    const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    const jsonPayload = decodeURIComponent(
      atob(base64)
        .split('')
        .map((c) => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2))
        .join('')
    );
    setUser(JSON.parse(jsonPayload));
  };

  const handleLogin = async () => {
    try {
      const result = await login();
      setTokens(result);
      await AsyncStorage.setItem('tokens', JSON.stringify(result));
      decodeToken(result.idToken);
    } catch (error) {
      console.error('Login failed:', error);
    }
  };

  const handleLogout = async () => {
    await logout(tokens);
    await AsyncStorage.removeItem('tokens');
    setTokens(null);
    setUser(null);
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      {!tokens ? (
        <Button title="Login" onPress={handleLogin} />
      ) : (
        <>
          <Text>Welcome, {user?.name}!</Text>
          <Text>Email: {user?.email}</Text>
          <Button title="Logout" onPress={handleLogout} />
        </>
      )}
    </View>
  );
}

export default App;
```

**iOS Configuration** (`ios/MyApp/Info.plist`):

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>com.example.myapp</string>
    </array>
  </dict>
</array>
```

**Android Configuration** (`android/app/src/main/AndroidManifest.xml`):

```xml
<activity android:name="net.openid.appauth.RedirectUriReceiverActivity">
  <intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:scheme="com.example.myapp"/>
  </intent-filter>
</activity>
```

#### Bearer-Only Client (REST API)

**Real-World Scenario**: Microservice API

**Create Client**:

```text
Client ID: product-api

Capability config:
Client authentication: ☑ ON
Authorization: ☐ OFF

Authentication flow:
☐ Standard flow
☐ Direct access grants
☐ Implicit flow
☐ Service accounts roles

Additional settings:
Bearer only: ☑ ON
  └── This client only validates tokens, doesn't initiate login
```

**Spring Boot Resource Server Example**:

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://auth.example.com/realms/customer-portal
          jwk-set-uri: https://auth.example.com/realms/customer-portal/protocol/openid-connect/certs

server:
  port: 8081

logging:
  level:
    org.springframework.security: DEBUG
```

```java
// SecurityConfig.java
package com.example.productapi.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;
import org.springframework.security.web.SecurityFilterChain;

import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/**").authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );

        return http.build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(this::extractAuthorities);
        return converter;
    }

    private Collection<GrantedAuthority> extractAuthorities(Jwt jwt) {
        // Extract realm roles
        Map<String, Object> realmAccess = jwt.getClaim("realm_access");
        List<String> realmRoles = realmAccess != null 
            ? (List<String>) realmAccess.get("roles") 
            : List.of();

        // Extract client roles
        Map<String, Object> resourceAccess = jwt.getClaim("resource_access");
        List<String> clientRoles = List.of();
        if (resourceAccess != null) {
            Map<String, Object> productApi = (Map<String, Object>) resourceAccess.get("product-api");
            if (productApi != null) {
                clientRoles = (List<String>) productApi.get("roles");
            }
        }

        // Combine and convert to authorities
        return Stream.concat(
                realmRoles.stream().map(role -> "ROLE_" + role),
                clientRoles.stream().map(role -> "ROLE_" + role)
            )
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());
    }
}
```

```java
// ProductController.java
package com.example.productapi.controller;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping
    public List<Product> getAllProducts(@AuthenticationPrincipal Jwt jwt) {
        String userId = jwt.getSubject();
        String email = jwt.getClaimAsString("email");
        
        // Use user info for business logic
        return productService.getProducts();
    }

    @GetMapping("/{id}")
    @PreAuthorize("hasAnyRole('USER', 'PREMIUM_USER')")
    public Product getProduct(@PathVariable Long id) {
        return productService.getProduct(id);
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public Product createProduct(@RequestBody Product product, @AuthenticationPrincipal Jwt jwt) {
        String email = jwt.getClaimAsString("email");
        product.setCreatedBy(email);
        return productService.create(product);
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteProduct(@PathVariable Long id) {
        productService.delete(id);
    }
}
```
#### Service-to-Service Communication (Client Credentials Flow)

**Real-World Scenario**: Backend service needs to call another API without user context (scheduled jobs, integrations, webhooks).

**Create Client**:

```text
Client ID: integration-service

Capability config:
Client authentication: ☑ ON
Authorization: ☐ OFF

Authentication flow:
☐ Standard flow
☐ Direct access grants
☐ Implicit flow
☑ Service accounts roles
  └── Enable for client credentials flow

Login settings:
(Not applicable for service accounts)
```

**Assign Service Account Roles**:

After creating the client, navigate to **Service account roles** tab:

```text
Service Account Roles for: integration-service

Assign realm roles to service account:
☑ api_client
☑ integration_access
☑ read_only

Assign client roles:
Client: product-api
☑ read_products
☑ write_products
```

**Get Access Token** (Client Credentials Flow):

```bash
# Request token using client credentials
curl -X POST https://auth.example.com/realms/customer-portal/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=integration-service" \
  -d "client_secret=XbQ3vK9L...2mP8nY" \
  -d "scope=api:read api:write"
```

**Response**:
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 300,
  "token_type": "Bearer",
  "not-before-policy": 0,
  "scope": "api:read api:write"
}
```

**Token Contents**:
```json
{
  "exp": 1729643100,
  "iat": 1729642800,
  "jti": "token-uuid",
  "iss": "https://auth.example.com/realms/customer-portal",
  "sub": "service-account-integration-service",
  "typ": "Bearer",
  "azp": "integration-service",
  "scope": "api:read api:write",
  "resource_access": {
    "product-api": {
      "roles": ["read_products", "write_products"]
    }
  },
  "realm_access": {
    "roles": ["api_client", "integration_access", "read_only"]
  },
  "clientId": "integration-service",
  "clientHost": "192.168.1.10",
  "clientAddress": "192.168.1.10"
}
```

**Node.js Integration Example**:

```javascript
// services/keycloakService.js
const axios = require('axios');

class KeycloakService {
  constructor() {
    this.tokenEndpoint = 'https://auth.example.com/realms/customer-portal/protocol/openid-connect/token';
    this.clientId = process.env.KEYCLOAK_CLIENT_ID;
    this.clientSecret = process.env.KEYCLOAK_CLIENT_SECRET;
    this.accessToken = null;
    this.tokenExpiration = null;
  }

  async getAccessToken() {
    // Return cached token if still valid
    const now = Date.now();
    if (this.accessToken && this.tokenExpiration && now < this.tokenExpiration) {
      return this.accessToken;
    }

    // Request new token
    try {
      const response = await axios.post(this.tokenEndpoint, 
        new URLSearchParams({
          grant_type: 'client_credentials',
          client_id: this.clientId,
          client_secret: this.clientSecret,
          scope: 'api:read api:write'
        }),
        {
          headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
        }
      );

      this.accessToken = response.data.access_token;
      // Set expiration with 30 second buffer
      this.tokenExpiration = now + (response.data.expires_in - 30) * 1000;

      return this.accessToken;
    } catch (error) {
      console.error('Failed to get access token:', error);
      throw error;
    }
  }

  async callProtectedAPI(url, method = 'GET', data = null) {
    const token = await this.getAccessToken();

    try {
      const config = {
        method,
        url,
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json'
        }
      };

      if (data) {
        config.data = data;
      }

      const response = await axios(config);
      return response.data;
    } catch (error) {
      if (error.response?.status === 401) {
        // Token might be expired, clear cache and retry once
        this.accessToken = null;
        this.tokenExpiration = null;
        const newToken = await this.getAccessToken();
        
        const retryConfig = {
          method,
          url,
          headers: {
            'Authorization': `Bearer ${newToken}`,
            'Content-Type': 'application/json'
          }
        };
        if (data) retryConfig.data = data;
        
        const retryResponse = await axios(retryConfig);
        return retryResponse.data;
      }
      throw error;
    }
  }
}

module.exports = new KeycloakService();
```

**Usage in Scheduled Job**:

```javascript
// jobs/syncCustomers.js
const keycloakService = require('../services/keycloakService');

async function syncCustomers() {
  try {
    console.log('Starting customer sync...');

    // Get customers from external system
    const externalCustomers = await fetchExternalCustomers();

    // Call internal API with service account token
    const internalCustomers = await keycloakService.callProtectedAPI(
      'https://api.example.com/v1/customers',
      'GET'
    );

    // Sync logic
    for (const customer of externalCustomers) {
      const exists = internalCustomers.find(c => c.email === customer.email);
      
      if (!exists) {
        // Create new customer
        await keycloakService.callProtectedAPI(
          'https://api.example.com/v1/customers',
          'POST',
          customer
        );
        console.log(`Created customer: ${customer.email}`);
      }
    }

    console.log('Customer sync completed');
  } catch (error) {
    console.error('Customer sync failed:', error);
    throw error;
  }
}

module.exports = syncCustomers;
```

**Python Example**:

```python
# services/keycloak_service.py
import requests
from datetime import datetime, timedelta
import os

class KeycloakService:
    def __init__(self):
        self.token_endpoint = 'https://auth.example.com/realms/customer-portal/protocol/openid-connect/token'
        self.client_id = os.getenv('KEYCLOAK_CLIENT_ID')
        self.client_secret = os.getenv('KEYCLOAK_CLIENT_SECRET')
        self.access_token = None
        self.token_expiration = None

    def get_access_token(self):
        # Return cached token if still valid
        now = datetime.now()
        if self.access_token and self.token_expiration and now < self.token_expiration:
            return self.access_token

        # Request new token
        data = {
            'grant_type': 'client_credentials',
            'client_id': self.client_id,
            'client_secret': self.client_secret,
            'scope': 'api:read api:write'
        }

        response = requests.post(self.token_endpoint, data=data)
        response.raise_for_status()
        
        token_data = response.json()
        self.access_token = token_data['access_token']
        # Set expiration with 30 second buffer
        self.token_expiration = now + timedelta(seconds=token_data['expires_in'] - 30)

        return self.access_token

    def call_protected_api(self, url, method='GET', json_data=None):
        token = self.get_access_token()
        
        headers = {
            'Authorization': f'Bearer {token}',
            'Content-Type': 'application/json'
        }

        try:
            if method == 'GET':
                response = requests.get(url, headers=headers)
            elif method == 'POST':
                response = requests.post(url, headers=headers, json=json_data)
            elif method == 'PUT':
                response = requests.put(url, headers=headers, json=json_data)
            elif method == 'DELETE':
                response = requests.delete(url, headers=headers)
            
            response.raise_for_status()
            return response.json() if response.content else None
            
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 401:
                # Token expired, clear cache and retry once
                self.access_token = None
                self.token_expiration = None
                return self.call_protected_api(url, method, json_data)
            raise

# Singleton instance
keycloak_service = KeycloakService()
```

```python
# jobs/sync_customers.py
from services.keycloak_service import keycloak_service
import logging

logger = logging.getLogger(__name__)

def sync_customers():
    try:
        logger.info('Starting customer sync...')

        # Get customers from external system
        external_customers = fetch_external_customers()

        # Call internal API with service account token
        internal_customers = keycloak_service.call_protected_api(
            'https://api.example.com/v1/customers'
        )

        # Sync logic
        for customer in external_customers:
            exists = any(c['email'] == customer['email'] for c in internal_customers)
            
            if not exists:
                # Create new customer
                keycloak_service.call_protected_api(
                    'https://api.example.com/v1/customers',
                    method='POST',
                    json_data=customer
                )
                logger.info(f"Created customer: {customer['email']}")

        logger.info('Customer sync completed')
    except Exception as e:
        logger.error(f'Customer sync failed: {str(e)}')
        raise
```

### Creating a SAML 2.0 Client

SAML is used for enterprise integrations, legacy systems, and SSO with partners.[1]

**Real-World Scenario**: Integration with Enterprise ERP System

**Navigate to**: Clients → Create client

**Step 1: General Settings**

```text
Client type: SAML
Client ID*: https://erp.company.com/saml
  └── This is the Entity ID (usually a URL)

[Next]
```

**Step 2: SAML Settings**

```text
Name: Enterprise ERP System
Description: Legacy ERP system integration

Master SAML Processing URL: https://erp.company.com/saml/acs

Name ID format: email
  └── Other options: persistent, transient, username

Force POST binding: ☑ ON
  └── Use HTTP-POST instead of HTTP-Redirect

Force name ID format: ☑ ON
  └── Always use the specified format

Include AuthnStatement: ☑ ON
  └── Required by most SAML SPs

Sign documents: ☑ ON
  └── Sign the SAML response

Sign assertions: ☑ ON
  └── Sign the SAML assertion

Signature algorithm: RSA_SHA256
  └── Modern secure algorithm

SAML signature key name: CERT_SUBJECT
  └── How to reference the signing key

Canonicalization method: Exclusive
  └── XML canonicalization method

Encrypt assertions: ☐ OFF
  └── Enable if SP requires encryption

Client signature required: ☐ OFF
  └── Enable if SP signs AuthnRequests

Force artifact binding: ☐ OFF
  └── Use artifact binding instead of POST

[Save]
```

**Step 3: Configure Fine-Grained Settings**

Navigate to: Client → Settings → Fine grain SAML endpoint configuration

```text
Assertion Consumer Service POST Binding URL:
  https://erp.company.com/saml/acs

Assertion Consumer Service Redirect Binding URL:
  https://erp.company.com/saml/acs/redirect

Logout Service POST Binding URL:
  https://erp.company.com/saml/logout

Logout Service Redirect Binding URL:
  https://erp.company.com/saml/logout/redirect

Logout Service SOAP Binding URL:
  https://erp.company.com/saml/logout/soap
```

**Step 4: Configure SAML Keys**

Navigate to: Client → Keys

```text
Client Signature Required: OFF
  └── If ON, upload SP's public certificate here

Signing Key:
  ☑ Use realm certificate
  └── Or generate/import client-specific certificate
```

**Step 5: SAML Attribute Mappers**

Navigate to: Client → Client scopes → [client-name]-dedicated → Mappers

**Add Attribute Mappers**:

```text
Mapper 1: Email
┌─────────────────────────────────────────┐
│ Name: email                             │
│ Mapper Type: User Property             │
│ Property: email                         │
│ SAML Attribute Name: email              │
│ SAML Attribute NameFormat: Basic        │
└─────────────────────────────────────────┘

Mapper 2: First Name
┌─────────────────────────────────────────┐
│ Name: first-name                        │
│ Mapper Type: User Property             │
│ Property: firstName                     │
│ SAML Attribute Name: first_name         │
│ SAML Attribute NameFormat: Basic        │
└─────────────────────────────────────────┘

Mapper 3: Last Name
┌─────────────────────────────────────────┐
│ Name: last-name                         │
│ Mapper Type: User Property             │
│ Property: lastName                      │
│ SAML Attribute Name: last_name          │
│ SAML Attribute NameFormat: Basic        │
└─────────────────────────────────────────┘

Mapper 4: Roles
┌─────────────────────────────────────────┐
│ Name: role-list                         │
│ Mapper Type: Role list                  │
│ Role attribute name: roles              │
│ SAML Attribute NameFormat: Basic        │
│ Single Role Attribute: ON               │
└─────────────────────────────────────────┘

Mapper 5: Custom Attribute (Employee ID)
┌─────────────────────────────────────────┐
│ Name: employee-id                       │
│ Mapper Type: User Attribute            │
│ User Attribute: employee_id             │
│ SAML Attribute Name: employeeID         │
│ SAML Attribute NameFormat: URI          │
└─────────────────────────────────────────┘
```

**Step 6: Download SAML Metadata**

Navigate to: Client → Settings → SAML Metadata

```text
Click "Download" to get SAML metadata XML

Or access via URL:
https://auth.example.com/realms/customer-portal/protocol/saml/descriptor
```

**SAML Metadata Example**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<EntityDescriptor 
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
    entityID="https://auth.example.com/realms/customer-portal">
    
    <IDPSSODescriptor 
        WantAuthnRequestsSigned="false"
        protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
        
        <KeyDescriptor use="signing">
            <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
                <X509Data>
                    <X509Certificate>MIICnzCCAYcCBgF...</X509Certificate>
                </X509Data>
            </KeyInfo>
        </KeyDescriptor>
        
        <SingleLogoutService 
            Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
            Location="https://auth.example.com/realms/customer-portal/protocol/saml"/>
        
        <NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</NameIDFormat>
        
        <SingleSignOnService 
            Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
            Location="https://auth.example.com/realms/customer-portal/protocol/saml"/>
        
        <SingleSignOnService 
            Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
            Location="https://auth.example.com/realms/customer-portal/protocol/saml"/>
    </IDPSSODescriptor>
</EntityDescriptor>
```

**SAML Response Example**:

```xml
<samlp:Response 
    xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
    xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
    ID="_response_uuid"
    Version="2.0"
    IssueInstant="2025-10-23T01:00:00Z"
    Destination="https://erp.company.com/saml/acs">
    
    <saml:Issuer>https://auth.example.com/realms/customer-portal</saml:Issuer>
    
    <samlp:Status>
        <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
    </samlp:Status>
    
    <saml:Assertion 
        ID="_assertion_uuid"
        Version="2.0"
        IssueInstant="2025-10-23T01:00:00Z">
        
        <saml:Issuer>https://auth.example.com/realms/customer-portal</saml:Issuer>
        
        <saml:Subject>
            <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
                john.doe@company.com
            </saml:NameID>
            <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
                <saml:SubjectConfirmationData 
                    NotOnOrAfter="2025-10-23T01:05:00Z"
                    Recipient="https://erp.company.com/saml/acs"/>
            </saml:SubjectConfirmation>
        </saml:Subject>
        
        <saml:Conditions 
            NotBefore="2025-10-23T01:00:00Z"
            NotOnOrAfter="2025-10-23T01:05:00Z">
            <saml:AudienceRestriction>
                <saml:Audience>https://erp.company.com/saml</saml:Audience>
            </saml:AudienceRestriction>
        </saml:Conditions>
        
        <saml:AuthnStatement 
            AuthnInstant="2025-10-23T01:00:00Z"
            SessionIndex="_session_uuid">
            <saml:AuthnContext>
                <saml:AuthnContextClassRef>
                    urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified
                </saml:AuthnContextClassRef>
            </saml:AuthnContext>
        </saml:AuthnStatement>
        
        <saml:AttributeStatement>
            <saml:Attribute Name="email" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
                <saml:AttributeValue>john.doe@company.com</saml:AttributeValue>
            </saml:Attribute>
            <saml:Attribute Name="first_name" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
                <saml:AttributeValue>John</saml:AttributeValue>
            </saml:Attribute>
            <saml:Attribute Name="last_name" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
                <saml:AttributeValue>Doe</saml:AttributeValue>
            </saml:Attribute>
            <saml:Attribute Name="employeeID" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
                <saml:AttributeValue>EMP12345</saml:AttributeValue>
            </saml:Attribute>
            <saml:Attribute Name="roles" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
                <saml:AttributeValue>employee</saml:AttributeValue>
                <saml:AttributeValue>manager</saml:AttributeValue>
            </saml:Attribute>
        </saml:AttributeStatement>
    </saml:Assertion>
</samlp:Response>
```

**Testing SAML Integration**:

1. **SP-Initiated Login**: User visits `https://erp.company.com/login`, gets redirected to Keycloak
2. **User authenticates** at Keycloak
3. **Keycloak sends SAML response** back to SP
4. **SP validates response** and creates session

**Troubleshooting SAML Issues**:

```bash
# View SAML events
Navigate to: Events → Login events
Filter by: SAML events

Common issues:
- Invalid signature: Check certificate configuration
- Wrong NameID format: Verify Name ID format settings
- Missing attributes: Check mapper configuration
- ACS URL mismatch: Verify endpoint URLs
- Expired assertion: Check time sync between servers
```

**SAML Tracer Tool**: Use browser extension "SAML-tracer" to debug SAML flows in real-time.

### Client Scopes and Protocol Mappers

Client scopes define what information (claims) is included in tokens.[2]

#### Understanding Client Scopes

**Default Client Scopes**: Automatically included in all tokens[2]
**Optional Client Scopes**: Only included when explicitly requested via `scope` parameter[2]

**Built-in Client Scopes**:[2]

```text
Default Scopes:
- web: Basic web application claims
- acr: Authentication Context Class Reference
- profile: User profile information (name, username, etc.)
- roles: User roles
- email: Email address and verification status

Optional Scopes:
- address: Postal address
- phone: Phone number
- offline_access: Refresh token with extended lifetime
- microprofile-jwt: MicroProfile JWT claims
```

**Viewing Client Scopes**:

Navigate to: Client scopes

```text
┌────────────────────────────────────────────────┐
│ Name           │ Type     │ Protocol          │
├────────────────────────────────────────────────┤
│ profile        │ Default  │ openid-connect    │
│ email          │ Default  │ openid-connect    │
│ address        │ Optional │ openid-connect    │
│ phone          │ Optional │ openid-connect    │
│ offline_access │ Optional │ openid-connect    │
│ roles          │ Default  │ openid-connect    │
│ web            │ Default  │ openid-connect    │
└────────────────────────────────────────────────┘
```

#### Creating Custom Client Scopes

**Real-World Scenario**: E-commerce application needs order history and loyalty points in tokens

**Navigate to**: Client scopes → Create client scope

**Step 1: Create Scope**

```text
┌─────────────────────────────────────────┐
│ Name*: customer-data                    │
│ Description: Customer-specific claims   │
│                for e-commerce           │
│ Type: Default                           │
│ Display on consent screen: ON           │
│ Consent screen text: Access customer   │
│                      data               │
│ Include in token scope: ON              │
│ Protocol: openid-connect                │
└─────────────────────────────────────────┘
[Save]
```

**Step 2: Add Protocol Mappers**

Navigate to: Client scopes → customer-data → Mappers → Add mapper → By configuration

**Mapper 1: Customer Tier**

```text
Mapper Type: User Attribute
┌─────────────────────────────────────────┐
│ Name*: customer-tier                    │
│ User Attribute: customer_tier           │
│ Token Claim Name: customer_tier         │
│ Claim JSON Type: String                 │
│ Add to ID token: ON                     │
│ Add to access token: ON                 │
│ Add to userinfo: ON                     │
│ Multivalued: OFF                        │
└─────────────────────────────────────────┘
```

**Mapper 2: Loyalty Points**

```text
Mapper Type: User Attribute
┌─────────────────────────────────────────┐
│ Name*: loyalty-points                   │
│ User Attribute: loyalty_points          │
│ Token Claim Name: loyalty_points        │
│ Claim JSON Type: int                    │
│ Add to ID token: ON                     │
│ Add to access token: ON                 │
│ Add to userinfo: ON                     │
└─────────────────────────────────────────┘
```

**Mapper 3: Subscription ID**

```text
Mapper Type: User Attribute
┌─────────────────────────────────────────┐
│ Name*: subscription-id                  │
│ User Attribute: subscription_id         │
│ Token Claim Name: subscription_id       │
│ Claim JSON Type: String                 │
│ Add to ID token: ON                     │
│ Add to access token: ON                 │
│ Add to userinfo: OFF                    │
└─────────────────────────────────────────┘
```

**Mapper 4: Account Created Date**

```text
Mapper Type: User Property
┌─────────────────────────────────────────┐
│ Name*: account-created                  │
│ Property: createdTimestamp              │
│ Token Claim Name: account_created_at    │
│ Claim JSON Type: long                   │
│ Add to ID token: OFF                    │
│ Add to access token: ON                 │
│ Add to userinfo: ON                     │
└─────────────────────────────────────────┘
```

**Mapper 5: Full Name**

```text
Mapper Type: User's full name
┌─────────────────────────────────────────┐
│ Name*: full-name                        │
│ Token Claim Name: name                  │
│ Add to ID token: ON                     │
│ Add to access token: ON                 │
│ Add to userinfo: ON                     │
└─────────────────────────────────────────┘
```

**Mapper 6: Hardcoded Claim (API Version)**

```text
Mapper Type: Hardcoded claim
┌─────────────────────────────────────────┐
│ Name*: api-version                      │
│ Token Claim Name: api_version           │
│ Claim value: v2                         │
│ Claim JSON Type: String                 │
│ Add to ID token: OFF                    │
│ Add to access token: ON                 │
│ Add to userinfo: OFF                    │
└─────────────────────────────────────────┘
```

**Mapper 7: Audience**

```text
Mapper Type: Audience
┌─────────────────────────────────────────┐
│ Name*: audience-mapper                  │
│ Included Client Audience: product-api   │
│ Included Custom Audience:               │
│   https://api.example.com               │
│ Add to ID token: OFF                    │
│ Add to access token: ON                 │
└─────────────────────────────────────────┘
```

**Mapper 8: Group Membership**

```text
Mapper Type: Group Membership
┌─────────────────────────────────────────┐
│ Name*: groups                           │
│ Token Claim Name: groups                │
│ Full group path: ON                     │
│ Add to ID token: ON                     │
│ Add to access token: ON                 │
│ Add to userinfo: ON                     │
└─────────────────────────────────────────┘
```

**Step 3: Assign Scope to Clients**

Navigate to: Clients → Select client → Client scopes → Add client scope

```text
Assign customer-data scope:
☑ Default (automatically included)
☐ Optional (only when explicitly requested)
```

**Resulting Token with Custom Scope**:

```json
{
  "exp": 1729643400,
  "iat": 1729643100,
  "jti": "token-uuid",
  "iss": "https://auth.example.com/realms/customer-portal",
  "aud": [
    "product-api",
    "https://api.example.com",
    "account"
  ],
  "sub": "user-uuid",
  "typ": "Bearer",
  "azp": "web-store-backend",
  "session_state": "session-uuid",
  "scope": "openid profile email customer-data",
  "email_verified": true,
  "name": "John Doe",
  "preferred_username": "john.doe@example.com",
  "given_name": "John",
  "family_name": "Doe",
  "email": "john.doe@example.com",
  "customer_tier": "gold",
  "loyalty_points": 15000,
  "subscription_id": "sub_ABC123XYZ",
  "account_created_at": 1625097600000,
  "api_version": "v2",
  "groups": [
    "/Premium Customers",
    "/Premium Customers/Gold Tier"
  ],
  "realm_access": {
    "roles": ["premium_customer", "user"]
  },
  "resource_access": {
    "product-api": {
      "roles": ["read_products", "write_reviews"]
    }
  }
}
```

**Using Custom Claims in Application**:

```javascript
// React component
import { useKeycloak } from '@react-keycloak/web';

function CustomerDashboard() {
  const { keycloak } = useKeycloak();
  
  const customerTier = keycloak.tokenParsed.customer_tier;
  const loyaltyPoints = keycloak.tokenParsed.loyalty_points;
  const groups = keycloak.tokenParsed.groups || [];

  return (
    <div className="dashboard">
      <h1>Welcome, {keycloak.tokenParsed.name}!</h1>
      
      <div className="tier-badge">
        {customerTier === 'gold' && <GoldBadge />}
        {customerTier === 'platinum' && <PlatinumBadge />}
      </div>
      
      <div className="loyalty">
        <h2>Loyalty Points: {loyaltyPoints.toLocaleString()}</h2>
        <ProgressBar value={loyaltyPoints} max={25000} />
      </div>
      
      {groups.includes('/Premium Customers') && (
        <div className="premium-features">
          <h3>Premium Features</h3>
          <PremiumFeaturesList />
        </div>
      )}
    </div>
  );
}
```

```java
// Spring Boot controller
@RestController
@RequestMapping("/api/customer")
public class CustomerController {
    
    @GetMapping("/dashboard")
    public CustomerDashboard getDashboard(@AuthenticationPrincipal Jwt jwt) {
        String customerTier = jwt.getClaimAsString("customer_tier");
        Integer loyaltyPoints = jwt.getClaim("loyalty_points");
        List<String> groups = jwt.getClaim("groups");
        String subscriptionId = jwt.getClaimAsString("subscription_id");
        
        // Business logic based on custom claims
        boolean isPremium = "gold".equals(customerTier) || "platinum".equals(customerTier);
        boolean hasFreeShipping = loyaltyPoints > 10000;
        
        return CustomerDashboard.builder()
            .customerTier(customerTier)
            .loyaltyPoints(loyaltyPoints)
            .isPremium(isPremium)
            .hasFreeShipping(hasFreeShipping)
            .subscriptionId(subscriptionId)
            .groups(groups)
            .build();
    }
}
```

#### Advanced Protocol Mappers

**Conditional Claim Mapper** (based on group membership):

```text
Mapper Type: Script Mapper
┌─────────────────────────────────────────┐
│ Name*: discount-percentage              │
│ Token Claim Name: discount_pct          │
│ Claim JSON Type: int                    │
│ Script:                                 │
│   var groups = user.getGroups();        │
│   var discount = 0;                     │
│   if (groups.contains('/VIP')) {        │
│     discount = 20;                      │
│   } else if (groups.contains('/Gold')){ │
│     discount = 15;                      │
│   } else if (groups.contains('/Silver'))│
│     discount = 10;                      │
│   }                                     │
│   discount;                             │
│ Add to ID token: OFF                    │
│ Add to access token: ON                 │
└─────────────────────────────────────────┘
```

**Mapper for User Session Info**:

```text
Mapper Type: User Session Note
┌─────────────────────────────────────────┐
│ Name*: login-ip                         │
│ User Session Note: clientAddress        │
│ Token Claim Name: login_ip              │
│ Claim JSON Type: String                 │
│ Add to ID token: OFF                    │
│ Add to access token: ON                 │
└─────────────────────────────────────────┘
```

**Role Name Mapper** (customize role claim structure):

```text
Mapper Type: User Realm Role
┌─────────────────────────────────────────┐
│ Name*: custom-roles                     │
│ Token Claim Name: user_roles            │
│ Claim JSON Type: String                 │
│ Multivalued: ON                         │
│ Add to ID token: ON                     │
│ Add to access token: ON                 │
│ Add to userinfo: ON                     │
└─────────────────────────────────────────┘
```

**Allowed Web Origins Mapper** (for CORS):

```text
Mapper Type: Allowed Web Origins
┌─────────────────────────────────────────┐
│ Name*: web-origins                      │
│ (Automatically adds valid web origins   │
│  to Access-Control-Allow-Origin)        │
└─────────────────────────────────────────┘
```

#### Requesting Optional Scopes

**OAuth Authorization Request with Optional Scopes**:

```javascript
// Request specific scopes
const authUrl = `https://auth.example.com/realms/customer-portal/protocol/openid-connect/auth?` +
  `client_id=web-app&` +
  `response_type=code&` +
  `redirect_uri=${encodeURIComponent('https://app.example.com/callback')}&` +
  `scope=openid profile email customer-data offline_access`;
  // ↑ openid (required), profile/email (default), customer-data (custom), offline_access (optional)

window.location.href = authUrl;
```

**With `offline_access` scope**, you get a refresh token that can be used even when the user is offline:

```javascript
// Refresh token request
const refreshResponse = await fetch(
  'https://auth.example.com/realms/customer-portal/protocol/openid-connect/token',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'refresh_token',
      refresh_token: storedRefreshToken,
      client_id: 'web-app',
      client_secret: 'client-secret'
    })
  }
);
```

**Consent Screen** (when optional scopes are requested):

```text
┌────────────────────────────────────────┐
│  MyApp would like to:                  │
│                                        │
│  ✓ View your basic profile (name,     │
│    username)                           │
│  ✓ View your email address             │
│  ✓ Access your customer data (tier,   │
│    loyalty points)                     │
│  ✓ Keep you signed in                  │
│                                        │
│  This application will be able to:     │
│  • See your purchase history           │
│  • View your loyalty points balance    │
│                                        │
│  [ Cancel ]  [ Allow ]                 │
└────────────────────────────────────────┘
```

***

## 7. Authentication Flows and Customization

Authentication flows define the sequence of steps users go through during login, registration, and credential recovery.[2]

### Understanding Built-in Flows

**Navigate to**: Authentication → Flows

**Default Flows**:[2]

```text
1. Browser Flow
   └── Standard login for web browsers

2. Direct Grant Flow
   └── Username/password direct authentication (REST API)

3. Registration Flow
   └── New user self-registration process

4. Reset Credentials Flow
   └── Password recovery/reset process

5. Client Authentication Flow
   └── How clients authenticate to Keycloak

6. Docker Authentication Flow
   └── Docker registry authentication

7. First Broker Login Flow
   └── First-time login via external identity provider
```

### Browser Flow Deep Dive

**Navigate to**: Authentication → Flows → browser

**Default Browser Flow Structure**:[2]

```text
browser (FLOW)
│
├── Cookie (ALTERNATIVE)
│   └── If valid SSO cookie exists, skip login
│
├── Kerberos (DISABLED)
│   └── Windows integrated authentication (optional)
│
└── browser forms (ALTERNATIVE) (SUBFLOW)
    │
    ├── Username Password Form (REQUIRED)
    │   └── Standard username/password entry
    │
    └── browser - Conditional OTP (CONDITIONAL) (SUBFLOW)
        │
        ├── Condition - User Configured (REQUIRED)
        │   └── Check if user has OTP configured
        │
        └── OTP Form (REQUIRED)
            └── Enter OTP code if configured
```

**Execution Requirements**:
- **REQUIRED**: Must successfully complete
- **ALTERNATIVE**: One execution in a group of alternatives must succeed
- **DISABLED**: Ignored during execution
- **CONDITIONAL**: Executes based on evaluator logic

**How It Works**:

```text
User visits protected application
    ↓
Redirected to Keycloak login
    ↓
Flow execution begins
    ↓
Check Cookie (ALTERNATIVE)
    ├─ Valid cookie? → SSO success, skip login
    └─ No cookie? → Continue
    ↓
Check Kerberos (DISABLED)
    └─ Skipped
    ↓
Forms (ALTERNATIVE) - must succeed since Cookie failed
    ↓
Username Password Form (REQUIRED)
    ├─ User enters credentials
    └─ Validation success
    ↓
Conditional OTP (CONDITIONAL)
    ↓
Condition - User Configured (REQUIRED)
    ├─ User has OTP? → Show OTP form
    └─ No OTP? → Skip
    ↓
Authentication complete
    ↓
Redirect back to application with code
```

### Creating Custom Authentication Flow

**Real-World Scenario**: Add reCAPTCHA and Terms acceptance to registration

#### Step 1: Create Custom Flow

**Navigate to**: Authentication → Flows → Create flow

```text
┌─────────────────────────────────────────┐
│ Alias*: custom-browser-flow             │
│ Description: Browser login with         │
│              reCAPTCHA protection        │
│ Top level flow type: generic            │
└─────────────────────────────────────────┘
[Create]
```

#### Step 2: Add Executions

**Add Cookie Check**:
```text
custom-browser-flow → Add execution
Select: Cookie
Requirement: ALTERNATIVE
```

**Add Identity Provider Redirector**:
```text
custom-browser-flow → Add execution
Select: Identity Provider Redirector
Requirement: ALTERNATIVE
```

**Add Forms Subflow**:
```text
custom-browser-flow → Add flow
Alias: custom-forms
Description: Custom forms with reCAPTCHA
Requirement: ALTERNATIVE
```

**Add Username Password Form**:
```text
custom-forms → Add execution
Select: Username Password Form
Requirement: REQUIRED
```

**Add reCAPTCHA**:
```text
custom-forms → Add execution
Select: Recaptcha
Requirement: REQUIRED
```

**Configure reCAPTCHA**:
Click the gear icon ⚙️ next to Recaptcha:

```text
┌─────────────────────────────────────────┐
│ Recaptcha Site Key: [Your site key]    │
│ Recaptcha Secret: [Your secret key]    │
└─────────────────────────────────────────┘
[Save]
```

**Get reCAPTCHA keys**: Visit https://www.google.com/recaptcha/admin and create a site.

**Add Conditional OTP**:
```text
custom-forms → Add flow
Alias: custom-otp
Description: OTP if configured
Requirement: CONDITIONAL
```

```text
custom-otp → Add execution
Select: Condition - User Configured
Requirement: REQUIRED
```

```text
custom-otp → Add execution
Select: OTP Form
Requirement: REQUIRED
```

**Final Flow Structure**:

```text
custom-browser-flow (FLOW)
│
├── Cookie (ALTERNATIVE)
├── Identity Provider Redirector (ALTERNATIVE)
└── custom-forms (ALTERNATIVE) (SUBFLOW)
    ├── Username Password Form (REQUIRED)
    ├── Recaptcha (REQUIRED)
    └── custom-otp (CONDITIONAL) (SUBFLOW)
        ├── Condition - User Configured (REQUIRED)
        └── OTP Form (REQUIRED)
```

#### Step 3: Bind Custom Flow

**Navigate to**: Authentication → Bindings

```text
┌─────────────────────────────────────────┐
│ Browser Flow: custom-browser-flow   [▼]│
│ Registration Flow: registration      [▼]│
│ Direct Grant Flow: direct grant      [▼]│
│ Reset Credentials: reset credentials [▼]│
│ Client Authentication: clients       [▼]│
│ Docker Authentication: docker        [▼]│
└─────────────────────────────────────────┘
[Save]
```

### Custom Registration Flow

**Real-World Scenario**: Collect additional user information during registration

#### Create Registration Flow

**Navigate to**: Authentication → Flows → Create flow

```text
┌─────────────────────────────────────────┐
│ Alias*: custom-registration             │
│ Description: Registration with extra    │
│              fields and terms           │
│ Top level flow type: generic            │
└─────────────────────────────────────────┘
```

**Add Executions**:

```text
custom-registration → Add execution
Select: Registration Page Form
Requirement: REQUIRED
```

**Add Form Fields Subflow**:

```text
Registration Page Form → Add execution (sub-flow)
Select: Registration User Creation
Requirement: REQUIRED
```

```text
Registration Page Form → Add execution
Select: Profile Validation
Requirement: REQUIRED
```

```text
Registration Page Form → Add execution
Select: Password Validation
Requirement: REQUIRED
```

```text
Registration Page Form → Add execution
Select: Recaptcha
Requirement: REQUIRED
```

```text
Registration Page Form → Add execution
Select: Terms and Conditions
Requirement: REQUIRED
```

**Configure Terms and Conditions**:

Click gear icon ⚙️:

```text
┌─────────────────────────────────────────┐
│ Alias: terms-and-conditions             │
│ Terms text: I accept the Terms of       │
│             Service and Privacy Policy  │
└─────────────────────────────────────────┘
[Save]
```

**Bind Registration Flow**:

```text
Authentication → Bindings
Registration Flow: custom-registration
```

**Resulting Registration Page includes**:
- Username/Email
- First Name, Last Name
- Password, Confirm Password
- reCAPTCHA challenge
- Terms and Conditions checkbox

### Custom Authenticators (SPI)

For advanced use cases, create custom authenticator SPIs.

**Real-World Example**: SMS OTP Authenticator

**Project Structure**:

```text
keycloak-sms-otp/
├── pom.xml
└── src/main/java/com/company/keycloak/
    ├── SMSOTPAuthenticator.java
    ├── SMSOTPAuthenticatorFactory.java
    ├── SMSOTPAuthenticatorFormAction.java
    └── SMSService.java
```

**SMSOTPAuthenticator.java**:

```java
package com.company.keycloak;

import org.keycloak.authentication.AuthenticationFlowContext;
import org.keycloak.authentication.Authenticator;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.RealmModel;
import org.keycloak.models.UserModel;

import javax.ws.rs.core.MultivaluedMap;
import javax.ws.rs.core.Response;

public class SMSOTPAuthenticator implements Authenticator {

    private static final String ATTR_PHONE_NUMBER = "phone_number";
    private static final String ATTR_SMS_CODE = "sms_code";
    private static final String FORM_SMS_CODE = "sms_code";

    @Override
    public void authenticate(AuthenticationFlowContext context) {
        UserModel user = context.getUser();
        String phoneNumber = user.getFirstAttribute(ATTR_PHONE_NUMBER);

        if (phoneNumber == null || phoneNumber.isEmpty()) {
            context.failure(AuthenticationFlowError.INVALID_USER);
            return;
        }

        // Generate 6-digit code
        String code = String.format("%06d", (int)(Math.random() * 1000000));
        
        // Store code in user session (expires in 5 minutes)
        context.getAuthenticationSession().setAuthNote(ATTR_SMS_CODE, code);
        
        // Send SMS
        SMSService.sendSMS(phoneNumber, "Your verification code is: " + code);
        
        // Show OTP input form
        Response challenge = context.form()
            .createForm("sms-otp-form.ftl");
        context.challenge(challenge);
    }

    @Override
    public void action(AuthenticationFlowContext context) {
        MultivaluedMap<String, String> formData = context.getHttpRequest().getDecodedFormParameters();
        String enteredCode = formData.getFirst(FORM_SMS_CODE);
        String expectedCode = context.getAuthenticationSession().getAuthNote(ATTR_SMS_CODE);

        if (expectedCode != null && expectedCode.equals(enteredCode)) {
            context.success();
        } else {
            Response challenge = context.form()
                .setError("Invalid verification code")
                .createForm("sms-otp-form.ftl");
            context.challenge(challenge);
        }
    }

    @Override
    public boolean requiresUser() {
        return true;
    }

    @Override
    public boolean configuredFor(KeycloakSession session, RealmModel realm, UserModel user) {
        return user.getFirstAttribute(ATTR_PHONE_NUMBER) != null;
    }

    @Override
    public void setRequiredActions(KeycloakSession session, RealmModel realm, UserModel user) {
        // Optionally require phone number setup
    }

    @Override
    public void close() {
    }
}
```

**SMSOTPAuthenticatorFactory.java**:

```java
package com.company.keycloak;

import org.keycloak.Config;
import org.keycloak.authentication.Authenticator;
import org.keycloak.authentication.AuthenticatorFactory;
import org.keycloak.models.AuthenticationExecutionModel;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.KeycloakSessionFactory;
import org.keycloak.provider.ProviderConfigProperty;

import java.util.List;

public class SMSOTPAuthenticatorFactory implements AuthenticatorFactory {

    public static final String PROVIDER_ID = "sms-otp-authenticator";

    @Override
    public String getDisplayType() {
        return "SMS OTP";
    }

    @Override
    public String getReferenceCategory() {
        return "otp";
    }

    @Override
    public boolean isConfigurable() {
        return true;
    }

    @Override
    public AuthenticationExecutionModel.Requirement[] getRequirementChoices() {
        return new AuthenticationExecutionModel.Requirement[]{
            AuthenticationExecutionModel.Requirement.REQUIRED,
            AuthenticationExecutionModel.Requirement.ALTERNATIVE,
            AuthenticationExecutionModel.Requirement.DISABLED
        };
    }

    @Override
    public boolean isUserSetupAllowed() {
        return true;
    }

    @Override
    public String getHelpText() {
        return "Validates OTP sent via SMS to user's phone number.";
    }

    @Override
    public List<ProviderConfigProperty> getConfigProperties() {
        return List.of();
    }

    @Override
    public Authenticator create(KeycloakSession session) {
        return new SMSOTPAuthenticator();
    }

    @Override
    public void init(Config.Scope config) {
    }

    @Override
    public void postInit(KeycloakSessionFactory factory) {
    }

    @Override
    public void close() {
    }

    @Override
    public String getId() {
        return PROVIDER_ID;
    }
}
```

**Register SPI** in `META-INF/services/org.keycloak.authentication.AuthenticatorFactory`:

```text
com.company.keycloak.SMSOTPAuthenticatorFactory
```

**Build and Deploy**:

```bash
# Build JAR
mvn clean package

# Copy to Keycloak
cp target/keycloak-sms-otp.jar /opt/keycloak/providers/

# Rebuild Keycloak
/opt/keycloak/bin/kc.sh build

# Restart
systemctl restart keycloak
```

**Use in Flow**:

```text
Authentication → Flows → browser → custom-forms
Add execution → SMS OTP
Requirement: REQUIRED
```

***

## 8. Multi-Factor Authentication (MFA)

Multi-Factor Authentication adds an additional layer of security beyond username and password. Keycloak supports multiple MFA methods.[1]

### OTP (One-Time Password) - TOTP and HOTP

**TOTP** (Time-based OTP): Codes change every 30 seconds (Google Authenticator, Authy)[1]
**HOTP** (HMAC-based OTP): Counter-based codes (less common)[1]

#### Configuring OTP Policy

**Navigate to**: Realm Settings → OTP Policy

```text
OTP Policy Configuration:
┌─────────────────────────────────────────┐
│ OTP Type: Time Based ⦿                  │
│          Counter Based ○                │
│                                         │
│ OTP Hash Algorithm: SHA1      [▼]      │
│   Options: SHA1, SHA256, SHA512         │
│   Note: SHA1 most compatible           │
│                                         │
│ Number of Digits: 6          [▼]       │
│   Options: 6, 8                         │
│   Note: 6 is standard                   │
│                                         │
│ Look Ahead Window: 1                    │
│   (Allows slight time drift)            │
│                                         │
│ OTP Token Period: 30 seconds            │
│   (How often code changes)              │
│                                         │
│ Supported Applications:                 │
│ ☑ totpAppGoogleName (Google Auth)      │
│ ☑ totpAppFreeOTPName (FreeOTP)         │
│ ☑ totpAppMicrosoftAuthenticatorName    │
└─────────────────────────────────────────┘
[Save]
```

**Production Recommendations**:
```text
OTP Type: Time Based
Algorithm: SHA256 (better security) or SHA1 (better compatibility)
Digits: 6 (standard)
Period: 30 seconds (standard)
Look Ahead Window: 1 (tolerates 30s clock drift)
```

#### Enabling OTP for Users

**Method 1: Required Action** (Force all users):

**Navigate to**: Authentication → Required actions

```text
Required Actions:
┌─────────────────────────────────────────┐
│ Configure OTP              ☑ Enabled    │
│                           ☑ Default     │
└─────────────────────────────────────────┘
```

With "Default" enabled, all new users must configure OTP on first login.

**Method 2: Per-User** (Selective):

**Navigate to**: Users → Select user → Required user actions

```text
☑ Configure OTP
```

**Method 3: Via Authentication Flow** (Conditional):

**Navigate to**: Authentication → Flows → browser

Ensure **Conditional OTP** is configured:

```text
browser - Conditional OTP (CONDITIONAL)
├── Condition - User Configured (REQUIRED)
│   └── Checks if user has OTP set up
└── OTP Form (REQUIRED)
    └── Prompts for OTP code
```

#### User OTP Setup Flow

**Step 1**: User logs in and sees:

```text
┌────────────────────────────────────────┐
│     Configure Mobile Authenticator     │
│                                        │
│  Scan this QR code with your           │
│  authenticator app:                    │
│                                        │
│   ████ ▄▄▄▄ █▀▄█▀ ▄▄▄▄ ████          │
│   ████ █ █ █▄ █▄▀ █ █ ████          │
│   ████▄▄▄▄▄▄▀█▄█ ▄▄▄▄▄▄████          │
│   ██▀ ▀▄▀▄▄ ▄▀█▄█ ▄▀ ▀▄██          │
│                                        │
│  Or enter this key manually:           │
│  JBSWY3DPEHPK3PXP                      │
│                                        │
│  Enter code from app:                  │
│  [______]                              │
│                                        │
│  [ Submit ]                            │
└────────────────────────────────────────┘
```

**Step 2**: User scans QR with authenticator app (Google Authenticator, Authy, Microsoft Authenticator)

**Step 3**: User enters 6-digit code to verify setup

**Step 4**: OTP is now configured - user will be prompted for code on subsequent logins

#### Login with OTP

After username/password, user sees:

```text
┌────────────────────────────────────────┐
│     Two-Factor Authentication          │
│                                        │
│  Open your authenticator app and       │
│  enter the 6-digit code:              │
│                                        │
│  [______]                              │
│                                        │
│  [ Verify ]                            │
│                                        │
│  Lost your authenticator?              │
│  [ Contact Support ]                   │
└────────────────────────────────────────┘
```

#### Backup Codes (Recovery Codes)

**Real-World Problem**: User loses phone with authenticator app.

**Solution**: Generate backup codes during OTP setup.

**Custom Authenticator for Backup Codes**:

```java
// BackupCodesAuthenticator.java
public class BackupCodesAuthenticator implements Authenticator {
    
    @Override
    public void authenticate(AuthenticationFlowContext context) {
        UserModel user = context.getUser();
        List<String> backupCodes = user.getAttribute("backup_codes");
        
        if (backupCodes == null || backupCodes.isEmpty()) {
            // Generate 10 backup codes
            List<String> codes = new ArrayList<>();
            for (int i = 0; i < 10; i++) {
                codes.add(generateBackupCode());
            }
            user.setAttribute("backup_codes", codes);
            
            // Show codes to user (one-time display)
            Response response = context.form()
                .setAttribute("backup_codes", codes)
                .createForm("backup-codes-display.ftl");
            context.challenge(response);
        } else {
            context.success();
        }
    }
    
    private String generateBackupCode() {
        // Generate 8-character alphanumeric code
        String chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
        StringBuilder code = new StringBuilder();
        SecureRandom random = new SecureRandom();
        for (int i = 0; i < 8; i++) {
            code.append(chars.charAt(random.nextInt(chars.length())));
        }
        return code.toString();
    }
}
```

**Template for displaying codes** (`backup-codes-display.ftl`):

```html
<div class="backup-codes">
    <h2>Save Your Backup Codes</h2>
    <p>Store these codes securely. Each can be used once if you lose your authenticator device.</p>
    
    <div class="codes-grid">
        <#list backup_codes as code>
            <div class="code">${code}</div>
        </#list>
    </div>
    
    <button onclick="printCodes()">Print Codes</button>
    <button onclick="downloadCodes()">Download Codes</button>
    
    <form action="${url.loginAction}" method="post">
        <input type="checkbox" name="codes_saved" required>
        <label>I have saved these codes securely</label>
        <button type="submit">Continue</button>
    </form>
</div>
```

#### OTP via SMS (Alternative to Authenticator App)

For users who prefer SMS-based OTP:

**SMS OTP Configuration**:

```text
Authentication → Flows → Create flow
Alias: sms-otp-flow

Add executions:
├── Cookie (ALTERNATIVE)
├── Username Password Form (REQUIRED)
└── SMS OTP (REQUIRED)
    └── Custom authenticator (see Section 7)
```

**SMS Service Integration** (Twilio example):

```java
// SMSService.java
import com.twilio.Twilio;
import com.twilio.rest.api.v2010.account.Message;
import com.twilio.type.PhoneNumber;

public class SMSService {
    private static final String ACCOUNT_SID = System.getenv("TWILIO_ACCOUNT_SID");
    private static final String AUTH_TOKEN = System.getenv("TWILIO_AUTH_TOKEN");
    private static final String FROM_NUMBER = System.getenv("TWILIO_PHONE_NUMBER");

    static {
        Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
    }

    public static void sendSMS(String toNumber, String messageBody) {
        try {
            Message message = Message.creator(
                new PhoneNumber(toNumber),
                new PhoneNumber(FROM_NUMBER),
                messageBody
            ).create();
            
            System.out.println("SMS sent: " + message.getSid());
        } catch (Exception e) {
            System.err.println("Failed to send SMS: " + e.getMessage());
            throw new RuntimeException("SMS delivery failed", e);
        }
    }

    public static String generateOTP() {
        return String.format("%06d", (int)(Math.random() * 1000000));
    }
}
```

### WebAuthn (Passwordless Authentication)

WebAuthn enables authentication using security keys (YubiKey) or biometrics (fingerprint, face recognition).[1]

#### Configuring WebAuthn

**Navigate to**: Realm Settings → WebAuthn Policy

```text
WebAuthn Policy:
┌─────────────────────────────────────────┐
│ Relying Party Entity Name*:             │
│   MyCompany                             │
│   (Displayed to users)                  │
│                                         │
│ Signature Algorithms:                   │
│   ☑ ES256 (Recommended)                 │
│   ☑ RS256                               │
│   ☐ ES384                               │
│   ☐ ES512                               │
│                                         │
│ Relying Party ID:                       │
│   example.com                           │
│   (Your domain)                         │
│                                         │
│ Attestation Conveyance Preference:      │
│   none ⦿ (Recommended for privacy)      │
│   indirect ○                            │
│   direct ○                              │
│                                         │
│ Authenticator Attachment:               │
│   not specified ⦿ (Allow all types)     │
│   platform ○ (Built-in only)            │
│   cross-platform ○ (External keys)      │
│                                         │
│ Require Resident Key:                   │
│   No ⦿ (Recommended)                    │
│   Yes ○                                 │
│                                         │
│ User Verification Requirement:          │
│   preferred ⦿ (Recommended)             │
│   required ○                            │
│   discouraged ○                         │
│                                         │
│ Timeout: 300 seconds                    │
│   (Time to complete WebAuthn)           │
│                                         │
│ Avoid Same Authenticator Registration:  │
│   ☐ OFF (Allow same key multiple times)│
└─────────────────────────────────────────┘
[Save]
```

**Production Recommendations**:
```text
Signature Algorithms: ES256, RS256
Attestation: none (privacy)
Authenticator Attachment: not specified (flexibility)
Resident Key: No (better compatibility)
User Verification: preferred (balance security/UX)
Timeout: 300 seconds
```

#### WebAuthn Passwordless Flow

**Create Passwordless Flow**:

**Navigate to**: Authentication → Flows → Create flow

```text
┌─────────────────────────────────────────┐
│ Alias*: webauthn-passwordless           │
│ Description: Passwordless login with    │
│              WebAuthn                    │
└─────────────────────────────────────────┘
```

**Add Executions**:

```text
webauthn-passwordless
├── Cookie (ALTERNATIVE)
├── Username Form (REQUIRED)
│   └── Only asks for username, not password
└── WebAuthn Passwordless (ALTERNATIVE)
    └── Prompts for security key or biometric
```

**Bind Flow**:

```text
Authentication → Bindings
Browser Flow: webauthn-passwordless
```

#### User WebAuthn Registration

**Enable Required Action**:

**Navigate to**: Authentication → Required actions

```text
☑ Webauthn Register
```

**User Flow**:

**Step 1**: User logs in with password (or during registration)

**Step 2**: Prompted to register security key:

```text
┌────────────────────────────────────────┐
│     Register Security Key              │
│                                        │
│  Enhance your account security by      │
│  registering a security key or         │
│  biometric authenticator.              │
│                                        │
│  [ Register Security Key ]             │
│                                        │
│  [ Skip for now ]                      │
└────────────────────────────────────────┘
```

**Step 3**: Browser prompts for security action:

```text
┌────────────────────────────────────────┐
│  🔐 example.com wants to create a      │
│     public key credential              │
│                                        │
│  Touch your security key or use        │
│  your biometric sensor                 │
│                                        │
│  [Cancel] [Allow]                      │
└────────────────────────────────────────┘
```

**Step 4**: User touches YubiKey or uses fingerprint

**Step 5**: Security key registered

#### Passwordless Login Flow

**Step 1**: User enters username (no password):

```text
┌────────────────────────────────────────┐
│          Sign In                       │
│                                        │
│  Email or Username:                    │
│  [john.doe@example.com]                │
│                                        │
│  [ Continue ]                          │
└────────────────────────────────────────┘
```

**Step 2**: Browser prompts for security key:

```text
┌────────────────────────────────────────┐
│  🔐 Use your security key to sign in   │
│                                        │
│  Touch your security key or use        │
│  your biometric sensor                 │
│                                        │
│  Registered keys:                      │
│  • YubiKey 5C                          │
│  • Touch ID                            │
│                                        │
│  [Cancel]                              │
└────────────────────────────────────────┘
```

**Step 3**: User authenticates with key/biometric

**Step 4**: Logged in (no password needed)

### Risk-Based Authentication

**Real-World Scenario**: Require MFA only for suspicious login attempts.

**Conditional Flow Based on Location/Device**:

```text
Create flow: risk-based-auth

├── Cookie (ALTERNATIVE)
├── Username Password Form (REQUIRED)
└── Conditional MFA (CONDITIONAL)
    ├── Condition - Risk Assessment (REQUIRED)
    │   └── Custom condition evaluates risk
    └── OTP Form (REQUIRED)
        └── Only if high risk
```

**Custom Risk Assessment Condition**:

```java
// RiskAssessmentCondition.java
public class RiskAssessmentCondition implements ConditionalAuthenticator {
    
    @Override
    public boolean matchCondition(AuthenticationFlowContext context) {
        UserModel user = context.getUser();
        HttpRequest request = context.getHttpRequest();
        
        String currentIP = request.getHttpHeaders().getHeaderString("X-Forwarded-For");
        if (currentIP == null) {
            currentIP = request.getRemoteAddr();
        }
        
        String userAgent = request.getHttpHeaders().getHeaderString("User-Agent");
        
        // Check known IPs
        List<String> knownIPs = user.getAttribute("known_ips");
        if (knownIPs != null && knownIPs.contains(currentIP)) {
            return false; // Low risk, skip MFA
        }
        
        // Check device fingerprint
        String currentFingerprint = generateDeviceFingerprint(userAgent);
        List<String> knownDevices = user.getAttribute("known_devices");
        if (knownDevices != null && knownDevices.contains(currentFingerprint)) {
            return false; // Known device, skip MFA
        }
        
        // Check geolocation
        String country = getCountryFromIP(currentIP);
        List<String> knownCountries = user.getAttribute("known_countries");
        if (knownCountries != null && !knownCountries.contains(country)) {
            // New country - high risk
            sendSecurityAlert(user, currentIP, country);
            return true; // Require MFA
        }
        
        // Check time-based patterns
        if (isUnusualLoginTime(user)) {
            return true; // Require MFA
        }
        
        // Default: require MFA for unknown scenarios
        return true;
    }
    
    private String generateDeviceFingerprint(String userAgent) {
        // Simplified fingerprinting
        return DigestUtils.sha256Hex(userAgent);
    }
    
    private String getCountryFromIP(String ip) {
        // Use IP geolocation service (MaxMind, IPStack, etc.)
        try {
            GeoIP2DatabaseReader reader = new GeoIP2DatabaseReader.Builder(database).build();
            InetAddress ipAddress = InetAddress.getByName(ip);
            CityResponse response = reader.city(ipAddress);
            return response.getCountry().getIsoCode();
        } catch (Exception e) {
            return "UNKNOWN";
        }
    }
    
    private boolean isUnusualLoginTime(UserModel user) {
        // Check if login is outside user's typical hours
        LocalTime now = LocalTime.now();
        int hour = now.getHour();
        
        // Unusual if between 2 AM and 6 AM
        return hour >= 2 && hour < 6;
    }
    
    private void sendSecurityAlert(UserModel user, String ip, String country) {
        String email = user.getEmail();
        String message = String.format(
            "New login from %s (%s). If this wasn't you, secure your account immediately.",
            country, ip
        );
        // Send email via SMTP
        EmailService.send(email, "Security Alert: New Login Location", message);
    }
}
```

**Update Known IPs/Devices After Successful Authentication**:

```java
// Post-authentication action
public class UpdateKnownDevices implements RequiredActionProvider {
    
    @Override
    public void requiredActionChallenge(RequiredActionContext context) {
        // Not needed - this is a silent action
        context.success();
    }
    
    @Override
    public void processAction(RequiredActionContext context) {
        UserModel user = context.getUser();
        HttpRequest request = context.getHttpRequest();
        
        String currentIP = request.getRemoteAddr();
        String userAgent = request.getHttpHeaders().getHeaderString("User-Agent");
        String fingerprint = DigestUtils.sha256Hex(userAgent);
        
        // Add to known IPs
        List<String> knownIPs = user.getAttribute("known_ips");
        if (knownIPs == null) knownIPs = new ArrayList<>();
        if (!knownIPs.contains(currentIP)) {
            knownIPs.add(currentIP);
            user.setAttribute("known_ips", knownIPs);
        }
        
        // Add to known devices
        List<String> knownDevices = user.getAttribute("known_devices");
        if (knownDevices == null) knownDevices = new ArrayList<>();
        if (!knownDevices.contains(fingerprint)) {
            knownDevices.add(fingerprint);
            user.setAttribute("known_devices", knownDevices);
        }
        
        context.success();
    }
}
```

***

## 9. Identity Brokering and Social Login

Identity brokering allows users to authenticate using external identity providers (social media, enterprise IdPs).[1]

### Understanding Identity Brokering

**Benefits**:
- **Reduced friction**: Users don't need to create new accounts
- **Simplified onboarding**: Faster registration
- **Trust**: Users trust existing providers
- **Social integration**: Access social graph data

**Architecture**:

```text
User
  ↓
  Clicks "Sign in with Google"
  ↓
Application redirects to Keycloak
  ↓
Keycloak redirects to Google
  ↓
User authenticates at Google
  ↓
Google returns user info to Keycloak
  ↓
Keycloak creates/updates local user
  ↓
Keycloak issues tokens to Application
  ↓
User logged in
```

### Configuring Google Identity Provider

**Prerequisites**: Google OAuth 2.0 Client ID and Secret

#### Create Google OAuth Credentials

**Step 1**: Go to https://console.cloud.google.com

**Step 2**: Create or select project

**Step 3**: Navigate to "APIs & Services" → "Credentials"

**Step 4**: Click "Create Credentials" → "OAuth client ID"

```text
Application type: Web application
Name: MyApp Keycloak Integration

Authorized JavaScript origins:
  https://auth.example.com

Authorized redirect URIs:
  https://auth.example.com/realms/customer-portal/broker/google/endpoint

[Create]
```

**Step 5**: Copy Client ID and Client Secret

#### Configure in Keycloak

**Navigate to**: Identity providers → Add provider → Google

```text
Google Identity Provider Configuration:
┌─────────────────────────────────────────┐
│ Alias*: google                          │
│   (URL-friendly identifier)             │
│                                         │
│ Display name: Google                    │
│   (Shown on login button)               │
│                                         │
│ Enabled: ☑ ON                           │
│                                         │
│ Store tokens: ☑ ON                      │
│   (Save Google tokens to access APIs)   │
│                                         │
│ Stored tokens readable: ☐ OFF           │
│   (Security: don't expose tokens)       │
│                                         │
│ Trust email: ☑ ON                       │
│   (Auto-verify email from Google)       │
│                                         │
│ Account linking only: ☐ OFF             │
│   (Allow new account creation)          │
│                                         │
│ Hide on login page: ☐ OFF               │
│   (Show "Sign in with Google" button)   │
│                                         │
│ First login flow: first broker login    │
│   (Flow to execute on first login)      │
│                                         │
│ Sync mode: import                       │
│   (How to handle user data updates)     │
│                                         │
│ Client ID*: [Your Google Client ID]    │
│                                         │
│ Client Secret*: [Your Google Secret]   │
│                                         │
│ Default scopes: openid profile email   │
│   (What data to request from Google)    │
│                                         │
│ Accepts prompt=none forward from client:│
│   ☐ OFF                                 │
│                                         │
│ Disable user info: ☐ OFF                │
│   (Get user info from /userinfo)        │
│                                         │
│ Hosted domain:                          │
│   (Optional: restrict to G Suite domain)│
│                                         │
│ Use userinfo to get user profile: ☑ ON  │
└─────────────────────────────────────────┘
[Save]
```

**Redirect URI** (shown after saving): Copy this and add to Google Console:
```text
https://auth.example.com/realms/customer-portal/broker/google/endpoint
```

#### Identity Provider Mappers

Map Google claims to Keycloak user attributes.

**Navigate to**: Identity providers → google → Mappers → Create

**Mapper 1: Email**

```text
Name: email
Sync mode override: inherit
Mapper type: Attribute Importer

Social Profile JSON Field Path: email
User Attribute Name: email
```

**Mapper 2: First Name**

```text
Name: first-name
Mapper type: Attribute Importer

Social Profile JSON Field Path: given_name
User Attribute Name: firstName
```

**Mapper 3: Last Name**

```text
Name: last-name
Mapper type: Attribute Importer

Social Profile JSON Field Path: family_name
User Attribute Name: lastName
```

**Mapper 4: Profile Picture**

```text
Name: profile-picture
Mapper type: Attribute Importer

Social Profile JSON Field Path: picture
User Attribute Name: profile_picture_url
```

**Mapper 5: Google ID**

```text
Name: google-user-id
Mapper type: Attribute Importer

Social Profile JSON Field Path: sub
User Attribute Name: google_user_id
```

**Mapper 6: Hardcoded Role**

```text
Name: social-user-role
Mapper type: Hardcoded Role

Role: social_login_user
```

**Mapper 7: Advanced Username**

```text
Name: username-mapper
Mapper type: Username Template Importer

Template: ${CLAIM.email}
Target: LOCAL
```

#### Login Page with Google

After configuration, the login page automatically shows:

```text
┌────────────────────────────────────────┐
│          Sign In                       │
│                                        │
│  Email or Username:                    │
│  [____________________]                │
│                                        │
│  Password:                             │
│  [____________________]                │
│                                        │
│  [ ] Remember me                       │
│                                        │
│  [ Sign In ]                           │
│                                        │
│  ────────── Or ──────────              │
│                                        │
│  [ Sign in with Google ]               │
│                                        │
│  Don't have an account?                │
│  [ Register ]                          │
└────────────────────────────────────────┘
```

#### First Login Flow

When user logs in via Google for the first time, Keycloak executes the "First broker login" flow.[1]

**Default Flow**:

```text
first broker login
├── Review Profile (REQUIRED)
│   └── Shows user their profile data from Google
│   └── Allows editing before account creation
└── Create User If Unique (ALTERNATIVE)
    └── Creates account if email doesn't exist
    └── Links to existing account if email exists
```

**Customizing First Login Flow**:

```text
Create flow: custom-first-broker-login

├── Detect Existing Broker User (REQUIRED)
│   └── Check if user already linked
│
├── Unique Email Check (ALTERNATIVE)
│   ├── Create User If Unique (ALTERNATIVE)
│   └── Handle Existing Account (ALTERNATIVE) (SUBFLOW)
│       ├── Confirm Link Existing Account (REQUIRED)
│       └── Account verification (CONDITIONAL)
│           ├── Condition - User Has Email (REQUIRED)
│           └── Verify Existing Account By Email (REQUIRED)
│
├── Require Email Verification (REQUIRED)
│   └── Send verification email to new users
│
└── Automatically Set Email Verified (REQUIRED)
    └── Trust email from IdP
```

**Real-World Customization**: Require terms acceptance on first social login:

```text
custom-first-broker-login
├── ... (existing steps)
├── Review Profile (REQUIRED)
├── Accept Terms and Conditions (REQUIRED)
│   └── Custom authenticator
└── Create User If Unique (ALTERNATIVE)
```

### Configuring GitHub Identity Provider

**Navigate to**: Identity providers → Add provider → GitHub

**Step 1**: Create OAuth App in GitHub

Go to: https://github.com/settings/developers

```text
Application name: MyApp
Homepage URL: https://example.com
Authorization callback URL:
  https://auth.example.com/realms/customer-portal/broker/github/endpoint

[Register application]
```

Copy Client ID and generate Client Secret.

**Step 2**: Configure in Keycloak

```text
GitHub Identity Provider:
┌─────────────────────────────────────────┐
│ Alias: github                           │
│ Enabled: ☑ ON                           │
│ Store tokens: ☑ ON                      │
│ Trust email: ☑ ON                       │
│ Client ID: [GitHub Client ID]          │
│ Client Secret: [GitHub Client Secret]  │
│ Default scopes: user:email              │
└─────────────────────────────────────────┘
```

**GitHub Mappers**:

```text
Mapper: username
Social Profile JSON Field Path: login
User Attribute Name: github_username

Mapper: avatar
Social Profile JSON Field Path: avatar_url
User Attribute Name: avatar_url

Mapper: profile-url
Social Profile JSON Field Path: html_url
User Attribute Name: github_profile_url
```

### Configuring Facebook Identity Provider

**Navigate to**: Identity providers → Add provider → Facebook

**Step 1**: Create Facebook App

Go to: https://developers.facebook.com/apps

```text
Create App → Consumer → Continue

App Display Name: MyApp
App Contact Email: support@example.com

[Create App]
```

**Step 2**: Add Facebook Login Product

```text
Products → Add Product → Facebook Login → Set Up

Valid OAuth Redirect URIs:
  https://auth.example.com/realms/customer-portal/broker/facebook/endpoint

[Save Changes]
```

**Step 3**: Configure in Keycloak

```text
Facebook Identity Provider:
┌─────────────────────────────────────────┐
│ Alias: facebook                         │
│ Client ID: [Facebook App ID]           │
│ Client Secret: [Facebook App Secret]   │
│ Default scopes: email public_profile   │
│                                         │
│ Advanced:                               │
│ Fetch user info from Graph API: ☑ ON   │
│ Fields to fetch: email,name,picture    │
└─────────────────────────────────────────┘
```

### Configuring LinkedIn Identity Provider

```text
LinkedIn Identity Provider:
┌─────────────────────────────────────────┐
│ Alias: linkedin                         │
│ Client ID: [LinkedIn Client ID]        │
│ Client Secret: [LinkedIn Client Secret]│
│ Default scopes: r_liteprofile r_emailaddress │
└─────────────────────────────────────────┘
```

### Configuring Microsoft Account

```text
Microsoft Identity Provider:
┌─────────────────────────────────────────┐
│ Alias: microsoft                        │
│ Client ID: [Azure AD Application ID]   │
│ Client Secret: [Azure AD Secret]       │
│ Default scopes: openid profile email   │
└─────────────────────────────────────────┘
```

### SAML Identity Provider (Enterprise SSO)

**Real-World Scenario**: Corporate customers use their own identity providers.

**Navigate to**: Identity providers → Add provider → SAML v2.0

```text
SAML Identity Provider:
┌─────────────────────────────────────────┐
│ Alias*: corporate-saml                  │
│                                         │
│ Display name: Corporate SSO             │
│                                         │
│ Enabled: ☑ ON                           │
│                                         │
│ Store tokens: ☐ OFF                     │
│   (SAML doesn't return refresh tokens)  │
│                                         │
│ Trust email: ☑ ON                       │
│                                         │
│ SAML configuration:                     │
│                                         │
│ Service Provider Entity ID:             │
│   https://auth.example.com/realms/      │
│   customer-portal                       │
│                                         │
│ Single Sign-On Service URL*:            │
│   https://idp.corporate.com/sso         │
│                                         │
│ Single Logout Service URL:              │
│   https://idp.corporate.com/logout      │
│                                         │
│ Backchannel Logout: ☑ ON                │
│                                         │
│ NameID Policy Format: Email             │
│   urn:oasis:names:tc:SAML:1.1:nameid-   │
│   format:emailAddress                   │
│                                         │
│ Principal Type: Subject NameID          │
│                                         │
│ HTTP-POST Binding Response: ☑ ON        │
│ HTTP-POST Binding for AuthnRequest: ☑ON │
│                                         │
│ Want AuthnRequests Signed: ☑ ON         │
│ Want Assertions Signed: ☑ ON            │
│ Want Assertions Encrypted: ☐ OFF        │
│                                         │
│ Signature Algorithm: RSA_SHA256         │
│                                         │
│ SAML Signature Key Name: CERT_SUBJECT   │
│                                         │
│ Force Authentication: ☐ OFF             │
│                                         │
│ Validate Signature: ☑ ON                │
│                                         │
│ Validating X509 Certificates:           │
│   [Upload IdP certificate]              │
│                                         │
│ Sign Service Provider Metadata: ☐ OFF   │
└─────────────────────────────────────────┘
[Save]
```

**Import IdP Metadata** (easier):

```text
Instead of manual config, click "Import from URL" or "Import from file"

Metadata URL: https://idp.corporate.com/metadata
Or upload XML file

[Import]
```

**SAML Attribute Mappers**:

```text
Mapper: email
Attribute Name: urn:oid:0.9.2342.19200300.100.1.3
User Attribute: email

Mapper: first-name
Attribute Name: urn:oid:2.5.4.42
User Attribute: firstName

Mapper: last-name
Attribute Name: urn:oid:2.5.4.4
User Attribute: lastName

Mapper: employee-id
Attribute Name: employeeID
User Attribute: employee_id
```

### OpenID Connect Identity Provider (Generic)

For any OIDC-compliant provider not explicitly supported:

**Navigate to**: Identity providers → Add provider → OpenID Connect v1.0

```text
OpenID Connect Identity Provider:
┌─────────────────────────────────────────┐
│ Alias*: custom-oidc                     │
│                                         │
│ Authorization URL*:                     │
│   https://idp.example.com/authorize     │
│                                         │
│ Token URL*:                             │
│   https://idp.example.com/token         │
│                                         │
│ Logout URL:                             │
│   https://idp.example.com/logout        │
│                                         │
│ User Info URL:                          │
│   https://idp.example.com/userinfo      │
│                                         │
│ Client Authentication: Client secret    │
│                        sent as post     │
│                                         │
│ Client ID*: [Client ID]                 │
│ Client Secret*: [Client Secret]         │
│                                         │
│ Default Scopes: openid profile email   │
│                                         │
│ Issuer:                                 │
│   https://idp.example.com               │
│                                         │
│ Validate Signatures: ☑ ON               │
│                                         │
│ JWKS URL:                               │
│   https://idp.example.com/.well-known/  │
│   jwks.json                             │
│                                         │
│ Use JWKS URL: ☑ ON                      │
└─────────────────────────────────────────┘
```

**Auto-discovery** (easier):

If provider supports OpenID Connect Discovery:

```text
Discovery endpoint URL:
  https://idp.example.com/.well-known/openid-configuration

[Import from URL]
```

### Account Linking

**Scenario**: User has existing account, later tries to login with Google using same email.

**Options**:

**1. Automatic Linking** (if email matches and is verified):

```text
Identity providers → google → Settings
Account linking only: ☐ OFF

First broker login flow: first broker login
  └── Includes "Create User If Unique"
      └── Links if email exists and verified
```

**2. Manual Confirmation**:

```text
Custom first broker login flow:

Handle Existing Account (ALTERNATIVE)
├── Confirm Link Existing Account (REQUIRED)
│   └── Shows: "Account with this email exists. Link accounts?"
│   └── [Yes, link] [No, different account]
└── Verify Existing Account By Email (REQUIRED)
    └── Sends verification email
    └── User clicks link to confirm
    └── Accounts linked
```

**3. Re-authentication Required**:

```text
Handle Existing Account (ALTERNATIVE)
├── Confirm Link Existing Account (REQUIRED)
└── Verify Existing Account By Re-authentication (REQUIRED)
    └── User must enter password of existing account
    └── If correct, accounts linked
```

### Token Exchange

**Scenario**: Application has Google token, wants Keycloak token.

**Enable Token Exchange**:

```text
Clients → Select client → Permissions → Enabled
Grant: token-exchange
```

**Exchange Google Token for Keycloak Token**:

```bash
curl -X POST https://auth.example.com/realms/customer-portal/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=urn:ietf:params:oauth:grant-type:token-exchange" \
  -d "client_id=web-app" \
  -d "client_secret=client-secret" \
  -d "subject_token=[Google Access Token]" \
  -d "subject_issuer=google" \
  -d "subject_token_type=urn:ietf:params:oauth:token-type:access_token"
```

**Response**: Keycloak access token for the user.

***

## 10. User Federation (LDAP/Active Directory)

User federation allows Keycloak to authenticate users from existing directories without migrating data. This is critical for enterprise integrations.[1]

### Understanding User Federation

**Benefits**:
- **No data migration**: Users stay in existing directory
- **Single source of truth**: Directory remains authoritative
- **Gradual migration**: Can sync incrementally
- **Hybrid architecture**: Mix local and federated users

**Sync Modes**:[1]
- **Import**: Copy users to Keycloak database (one-time or periodic)
- **Proxy**: Query LDAP in real-time (no local copy)
- **Hybrid**: Import but periodically refresh

### LDAP vs Active Directory

**LDAP (Lightweight Directory Access Protocol)**:
- OpenLDAP, Red Hat Directory Server, Apache Directory
- Standard protocol for directory services
- Used in Linux/Unix environments

**Active Directory (Microsoft)**:
- LDAP-compatible with Microsoft extensions
- Includes Kerberos authentication
- Used in Windows environments
- Additional attributes and schema

### Configuring LDAP User Federation

**Real-World Scenario**: Enterprise with existing OpenLDAP directory containing 10,000 employees.

**Navigate to**: User federation → Add provider → ldap

#### Basic Settings

```text
LDAP User Federation Configuration:
┌─────────────────────────────────────────┐
│ Console Display Name*: Company LDAP     │
│   (Name shown in admin console)         │
│                                         │
│ Enabled: ☑ ON                           │
│                                         │
│ Priority: 0                             │
│   (Lower = higher priority)             │
│                                         │
│ Edit Mode: READ_ONLY                [▼]│
│   READ_ONLY: Can't modify LDAP          │
│   WRITABLE: Can modify LDAP             │
│   UNSYNCED: Import but don't sync back │
│                                         │
│ Sync Registrations: ☐ OFF              │
│   (Allow new user registration to LDAP) │
│                                         │
│ Import Users: ☑ ON                      │
│   (Import users into Keycloak DB)       │
│                                         │
│ Batch Size: 1000                        │
│   (Users imported per batch)            │
│                                         │
│ Periodic Full Sync: ☑ Enabled           │
│   Sync Period: 604800 (weekly)          │
│                                         │
│ Periodic Changed Users Sync: ☑ Enabled  │
│   Sync Period: 86400 (daily)            │
│                                         │
│ Cache Policy: DEFAULT                   │
│   (How long to cache LDAP data)         │
└─────────────────────────────────────────┘
```

#### Connection Settings

```text
Connection and Authentication Settings:
┌─────────────────────────────────────────┐
│ Vendor*: Other                      [▼]│
│   Options:                              │
│   - Active Directory                    │
│   - Red Hat Directory Server            │
│   - Tivoli                              │
│   - Novell eDirectory                   │
│   - Other                               │
│                                         │
│ Connection URL*:                        │
│   ldap://ldap.company.com:389           │
│   Or: ldaps://ldap.company.com:636      │
│   Or: ldap://ldap1.company.com:389      │
│        ldap://ldap2.company.com:389     │
│   (Multiple for failover)               │
│                                         │
│ Enable StartTLS: ☐ OFF                  │
│   (Use for ldap:// with encryption)     │
│                                         │
│ Use Truststore SPI: only for ldaps  [▼]│
│   (SSL certificate validation)          │
│                                         │
│ Connection Pooling: ☑ ON                │
│   (Reuse connections for performance)   │
│                                         │
│ Connection Timeout: 10000ms             │
│                                         │
│ Read Timeout: 10000ms                   │
│                                         │
│ Bind Type: simple                   [▼]│
│   Options: simple, none                 │
│                                         │
│ Bind DN*:                               │
│   cn=keycloak-service,ou=service-       │
│   accounts,dc=company,dc=com            │
│   (Service account for Keycloak)        │
│                                         │
│ Bind Credential*:                       │
│   [Service account password]            │
│   [Show password] [Verify credentials]  │
└─────────────────────────────────────────┘
[Test connection] [Test authentication]
```

**Test Connection**: Always click to verify LDAP connectivity before proceeding.

#### LDAP Searching and Updating

```text
LDAP Searching and Updating Settings:
┌─────────────────────────────────────────┐
│ Users DN*:                              │
│   ou=employees,dc=company,dc=com        │
│   (Where users are located)             │
│                                         │
│ Username LDAP attribute*: uid           │
│   (LDAP attr containing username)       │
│   Common values:                        │
│   - uid (OpenLDAP)                      │
│   - sAMAccountName (Active Directory)   │
│   - cn (some LDAP servers)              │
│                                         │
│ RDN LDAP attribute*: uid                │
│   (Relative DN attribute)               │
│   Usually same as username attribute    │
│                                         │
│ UUID LDAP attribute*: entryUUID         │
│   (Unique identifier)                   │
│   Common values:                        │
│   - entryUUID (OpenLDAP)                │
│   - objectGUID (Active Directory)       │
│   - nsuniqueid (389 Directory)          │
│                                         │
│ User Object Classes*:                   │
│   inetOrgPerson, organizationalPerson   │
│   (Classes that define a user)          │
│   AD: user, person, organizationalPerson│
│                                         │
│ User LDAP Filter:                       │
│   (memberOf=cn=keycloak-users,          │
│    ou=groups,dc=company,dc=com)         │
│   (Optional: Filter which users to sync)│
│                                         │
│ Search Scope: Subtree              [▼] │
│   One Level: Only direct children       │
│   Subtree: Recursive (recommended)      │
│                                         │
│ Read Timeout: 10000ms                   │
│                                         │
│ Pagination: ☑ ON                        │
│   (For large directories)               │
└─────────────────────────────────────────┘
```

**User LDAP Filter Examples**:

```text
# Only active employees
(!(employeeStatus=terminated))

# Only specific department
(department=Engineering)

# Combination
(&(!(employeeStatus=terminated))(department=Engineering))

# Members of specific group
(memberOf=cn=keycloak-users,ou=groups,dc=company,dc=com)

# Exclude service accounts
(!(objectClass=serviceAccount))
```

#### Active Directory Specific Configuration

```text
Active Directory Settings:
┌─────────────────────────────────────────┐
│ Vendor: Active Directory            [▼]│
│                                         │
│ Connection URL:                         │
│   ldap://ad.company.com:389             │
│   Or: ldaps://ad.company.com:636        │
│                                         │
│ Bind DN:                                │
│   CN=Keycloak Service,OU=Service        │
│   Accounts,DC=company,DC=com            │
│                                         │
│ Users DN:                               │
│   CN=Users,DC=company,DC=com            │
│                                         │
│ Username LDAP attribute:                │
│   sAMAccountName                        │
│                                         │
│ RDN LDAP attribute: cn                  │
│                                         │
│ UUID LDAP attribute: objectGUID         │
│                                         │
│ User Object Classes:                    │
│   person, organizationalPerson, user    │
│                                         │
│ User LDAP Filter:                       │
│   (&(objectClass=user)                  │
│     (!(userAccountControl:1.2.840.      │
│       113556.1.4.803:=2)))              │
│   (Excludes disabled accounts)          │
└─────────────────────────────────────────┘
```

**Active Directory User Account Control Filter**:

```text
# Enabled accounts only
(!(userAccountControl:1.2.840.113556.1.4.803:=2))

# Not expired
(!(userAccountControl:1.2.840.113556.1.4.803:=8388608))

# Both conditions
(&(!(userAccountControl:1.2.840.113556.1.4.803:=2))
  (!(userAccountControl:1.2.840.113556.1.4.803:=8388608)))
```

### LDAP Attribute Mappers

Mappers synchronize LDAP attributes to Keycloak user attributes.[1]

**Navigate to**: User federation → company-ldap → Mappers

**Default Mappers** (created automatically):

```text
┌────────────────────────────────────────────────┐
│ Name           │ Type            │ LDAP Attr  │
├────────────────────────────────────────────────┤
│ username       │ user-attribute  │ uid        │
│ email          │ user-attribute  │ mail       │
│ first name     │ user-attribute  │ givenName  │
│ last name      │ user-attribute  │ sn         │
│ modify date    │ user-attribute  │ modifyDate │
│ create date    │ user-attribute  │ createDate │
└────────────────────────────────────────────────┘
```

#### Creating Custom Attribute Mappers

**Example 1: Employee ID**

**Navigate to**: Mappers → Create

```text
┌─────────────────────────────────────────┐
│ Name*: employee-id                      │
│                                         │
│ Mapper Type: user-attribute-ldap-      │
│              mapper                     │
│                                         │
│ User Model Attribute: employee_id       │
│   (Keycloak attribute name)             │
│                                         │
│ LDAP Attribute: employeeNumber          │
│   (LDAP attribute name)                 │
│                                         │
│ Read Only: ☑ ON                         │
│   (Don't sync changes back to LDAP)     │
│                                         │
│ Always Read Value From LDAP: ☐ OFF      │
│   (Use cached value)                    │
│                                         │
│ Is Mandatory In LDAP: ☐ OFF             │
│   (Allow users without this attribute)  │
│                                         │
│ Attribute Default Value:                │
│   (Optional default if LDAP value empty)│
│                                         │
│ Is Binary Attribute: ☐ OFF              │
└─────────────────────────────────────────┘
[Save]
```

**Example 2: Department**

```text
Name: department
Mapper Type: user-attribute-ldap-mapper
User Model Attribute: department
LDAP Attribute: department
Read Only: ON
```

**Example 3: Manager**

```text
Name: manager
Mapper Type: user-attribute-ldap-mapper
User Model Attribute: manager_dn
LDAP Attribute: manager
Read Only: ON
```

**Example 4: Phone Number**

```text
Name: phone-number
Mapper Type: user-attribute-ldap-mapper
User Model Attribute: phone_number
LDAP Attribute: telephoneNumber
Read Only: ON
```

**Example 5: Office Location**

```text
Name: office-location
Mapper Type: user-attribute-ldap-mapper
User Model Attribute: office_location
LDAP Attribute: physicalDeliveryOfficeName
Read Only: ON
```

### LDAP Group Mapping

Map LDAP groups to Keycloak groups or roles.

**Navigate to**: User federation → company-ldap → Mappers → Create

#### Group Mapper (Groups → Groups)

```text
┌─────────────────────────────────────────┐
│ Name*: ldap-groups                      │
│                                         │
│ Mapper Type: group-ldap-mapper          │
│                                         │
│ LDAP Groups DN*:                        │
│   ou=groups,dc=company,dc=com           │
│   (Where groups are located in LDAP)    │
│                                         │
│ Group Name LDAP Attribute*: cn          │
│   (Attribute containing group name)     │
│                                         │
│ Group Object Classes*:                  │
│   groupOfNames                          │
│   (AD: group)                           │
│                                         │
│ Preserve Group Inheritance: ☑ ON        │
│   (Maintain nested group structure)     │
│                                         │
│ Ignore Missing Groups: ☐ OFF            │
│   (Error if group doesn't exist)        │
│                                         │
│ Membership LDAP Attribute*: member      │
│   (Attribute listing group members)     │
│   Common values:                        │
│   - member (most LDAP)                  │
│   - uniqueMember (some LDAP)            │
│   - memberUid (POSIX groups)            │
│                                         │
│ Membership Attribute Type:              │
│   DN                                [▼] │
│   Options: DN, UID                      │
│                                         │
│ Membership User LDAP Attribute: uid     │
│   (User attr matched against membership)│
│                                         │
│ LDAP Filter:                            │
│   (Optional: filter which groups)       │
│                                         │
│ Mode: READ_ONLY                     [▼] │
│   READ_ONLY: Import only                │
│   LDAP_ONLY: Write to LDAP only         │
│   IMPORT: Import and don't sync         │
│                                         │
│ User Groups Retrieve Strategy:          │
│   LOAD_GROUPS_BY_MEMBER_ATTRIBUTE   [▼]│
│   Options:                              │
│   - LOAD_GROUPS_BY_MEMBER_ATTRIBUTE     │
│   - GET_GROUPS_FROM_USER_MEMBEROF_ATTR  │
│   - LOAD_GROUPS_BY_MEMBER_ATTRIBUTE_    │
│     RECURSIVELY                         │
│                                         │
│ Member-Of LDAP Attribute: memberOf      │
│   (If using GET_GROUPS_FROM_USER_...)   │
│                                         │
│ Mapped Group Attributes:                │
│   (Additional attributes to sync)       │
│                                         │
│ Drop non-existing groups during sync:   │
│   ☐ OFF                                 │
│   (Remove Keycloak groups not in LDAP)  │
│                                         │
│ Groups Path: /                          │
│   (Root path for imported groups)       │
└─────────────────────────────────────────┘
[Save]
```

**Example LDAP Group Structure**:

```text
LDAP Groups:
ou=groups,dc=company,dc=com
├── cn=Engineering
│   ├── member: uid=john,ou=employees,...
│   └── member: uid=jane,ou=employees,...
├── cn=Sales
│   ├── member: uid=bob,ou=employees,...
│   └── member: uid=alice,ou=employees,...
└── cn=Managers
    └── member: uid=jane,ou=employees,...

Keycloak Groups (after sync):
├── Engineering
│   ├── john
│   └── jane
├── Sales
│   ├── bob
│   └── alice
└── Managers
    └── jane
```

#### Role Mapper (Groups → Roles)

Map LDAP groups to Keycloak roles instead of groups:

```text
┌─────────────────────────────────────────┐
│ Name*: ldap-role-mapper                 │
│                                         │
│ Mapper Type: role-ldap-mapper           │
│                                         │
│ LDAP Roles DN*:                         │
│   ou=roles,dc=company,dc=com            │
│                                         │
│ Role Name LDAP Attribute*: cn           │
│                                         │
│ Role Object Classes*: groupOfNames      │
│                                         │
│ Membership LDAP Attribute*: member      │
│                                         │
│ Membership Attribute Type: DN       [▼] │
│                                         │
│ LDAP Filter:                            │
│   (Optional)                            │
│                                         │
│ Mode: READ_ONLY                     [▼] │
│                                         │
│ User Roles Retrieve Strategy:           │
│   LOAD_ROLES_BY_MEMBER_ATTRIBUTE    [▼]│
│                                         │
│ Use Realm Roles Mapping: ☑ ON          │
│   (Map to realm roles, not client roles)│
│                                         │
│ Client ID:                              │
│   (If mapping to client roles)          │
└─────────────────────────────────────────┘
```

### LDAP Password Policy

**Navigate to**: User federation → company-ldap → Settings

```text
Kerberos Integration:
┌─────────────────────────────────────────┐
│ Allow Kerberos authentication: ☐ OFF    │
│   (Enable for Windows integrated auth)  │
│                                         │
│ Kerberos Realm:                         │
│   COMPANY.COM                           │
│                                         │
│ Server Principal:                       │
│   HTTP/auth.company.com@COMPANY.COM     │
│                                         │
│ KeyTab:                                 │
│   /etc/keycloak/krb5.keytab            │
│                                         │
│ Debug: ☐ OFF                            │
│   (Enable for Kerberos troubleshooting) │
│                                         │
│ Update First Login: ☑ ON                │
│   (Update Keycloak profile on login)    │
└─────────────────────────────────────────┘
```

### Synchronization

#### Manual Synchronization

**Navigate to**: User federation → company-ldap → Settings

```text
Synchronization Actions:
┌─────────────────────────────────────────┐
│ [ Synchronize all users ]               │
│   Full sync: Import all users from LDAP │
│                                         │
│ [ Synchronize changed users ]           │
│   Incremental: Only changed users       │
│                                         │
│ [ Unlink users ]                        │
│   Remove LDAP link (make local)         │
│                                         │
│ [ Remove imported users ]               │
│   Delete all imported LDAP users        │
└─────────────────────────────────────────┘
```

**Click "Synchronize all users"** to start initial import.

**Sync Results**:

```text
Sync Result:
┌─────────────────────────────────────────┐
│ Status: Success                         │
│ Added: 1,247 users                      │
│ Updated: 0 users                        │
│ Removed: 0 users                        │
│ Failed: 0 users                         │
│ Duration: 15.3 seconds                  │
└─────────────────────────────────────────┘
```

#### Automatic Synchronization

Configured in provider settings:

```text
Periodic Full Sync: ☑ Enabled
Period: 604800 seconds (7 days)

Periodic Changed Users Sync: ☑ Enabled
Period: 86400 seconds (1 day)
```

**Sync runs automatically** based on configured periods.

### User Storage SPI (Custom Federation)

For non-LDAP directories, create custom User Storage Provider.

**Example: REST API User Storage**

```java
// RestUserStorageProvider.java
package com.company.keycloak.storage;

import org.keycloak.component.ComponentModel;
import org.keycloak.credential.CredentialInput;
import org.keycloak.credential.CredentialInputValidator;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.RealmModel;
import org.keycloak.models.UserModel;
import org.keycloak.storage.UserStorageProvider;
import org.keycloak.storage.user.UserLookupProvider;

public class RestUserStorageProvider implements 
        UserStorageProvider,
        UserLookupProvider,
        CredentialInputValidator {

    private KeycloakSession session;
    private ComponentModel model;
    private RestApiClient apiClient;

    public RestUserStorageProvider(KeycloakSession session, ComponentModel model) {
        this.session = session;
        this.model = model;
        this.apiClient = new RestApiClient(
            model.get("api.url"),
            model.get("api.key")
        );
    }

    @Override
    public UserModel getUserById(RealmModel realm, String id) {
        // Not implemented for this example
        return null;
    }

    @Override
    public UserModel getUserByUsername(RealmModel realm, String username) {
        try {
            // Call external API
            UserDTO userDto = apiClient.getUserByUsername(username);
            
            if (userDto == null) {
                return null;
            }
            
            // Create UserModel adapter
            return new RestUserAdapter(session, realm, model, userDto);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    @Override
    public UserModel getUserByEmail(RealmModel realm, String email) {
        try {
            UserDTO userDto = apiClient.getUserByEmail(email);
            if (userDto == null) return null;
            return new RestUserAdapter(session, realm, model, userDto);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    @Override
    public boolean supportsCredentialType(String credentialType) {
        return PasswordCredentialModel.TYPE.equals(credentialType);
    }

    @Override
    public boolean isConfiguredFor(RealmModel realm, UserModel user, String credentialType) {
        return supportsCredentialType(credentialType);
    }

    @Override
    public boolean isValid(RealmModel realm, UserModel user, CredentialInput input) {
        if (!supportsCredentialType(input.getType())) {
            return false;
        }

        String username = user.getUsername();
        String password = input.getChallengeResponse();

        try {
            // Validate credentials against external API
            return apiClient.validateCredentials(username, password);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    @Override
    public void close() {
        // Cleanup if needed
    }
}
```

```java
// RestApiClient.java
public class RestApiClient {
    private String apiUrl;
    private String apiKey;
    private HttpClient httpClient;

    public RestApiClient(String apiUrl, String apiKey) {
        this.apiUrl = apiUrl;
        this.apiKey = apiKey;
        this.httpClient = HttpClient.newHttpClient();
    }

    public UserDTO getUserByUsername(String username) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(apiUrl + "/users?username=" + username))
            .header("Authorization", "Bearer " + apiKey)
            .GET()
            .build();

        HttpResponse<String> response = httpClient.send(
            request, 
            HttpResponse.BodyHandlers.ofString()
        );

        if (response.statusCode() == 200) {
            ObjectMapper mapper = new ObjectMapper();
            return mapper.readValue(response.body(), UserDTO.class);
        }
        return null;
    }

    public boolean validateCredentials(String username, String password) throws Exception {
        String json = String.format(
            "{\"username\":\"%s\",\"password\":\"%s\"}", 
            username, 
            password
        );

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(apiUrl + "/auth/validate"))
            .header("Authorization", "Bearer " + apiKey)
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(json))
            .build();

        HttpResponse<String> response = httpClient.send(
            request,
            HttpResponse.BodyHandlers.ofString()
        );

        return response.statusCode() == 200;
    }
}
```

***

## 11. Authorization Services

Keycloak's Authorization Services provide fine-grained access control using User-Managed Access (UMA) 2.0.[1]

### Understanding Authorization Concepts

**Key Components**:
1. **Resources**: Protected objects (documents, APIs, data)
2. **Scopes**: Actions on resources (view, edit, delete)
3. **Policies**: Rules that grant/deny access
4. **Permissions**: Link resources, scopes, and policies

**Authorization Flow**:

```text
User requests resource
    ↓
Application requests authorization decision
    ↓
Keycloak evaluates policies
    ↓
Returns permission decision (grant/deny)
    ↓
Application enforces decision
```

### Enabling Authorization

**Navigate to**: Clients → Select client → Settings

```text
Capability config:
☑ Client authentication: ON
☑ Authorization: ON

[Save]
```

New **Authorization** tab appears.

### Defining Resources

**Navigate to**: Clients → client-name → Authorization → Resources

**Example 1: Document Resource**

```text
[ Create resource ]

┌─────────────────────────────────────────┐
│ Name*: document                         │
│                                         │
│ Display name: Document                  │
│                                         │
│ Type: urn:myapp:resources:document      │
│   (URN for resource type)               │
│                                         │
│ URIs:                                   │
│   /api/documents/*                      │
│   /api/documents/{id}                   │
│   (Resource location patterns)          │
│                                         │
│ Scopes:                                 │
│   ☑ document:view                       │
│   ☑ document:edit                       │
│   ☑ document:delete                     │
│   ☑ document:share                      │
│                                         │
│ Owner:                                  │
│   (Leave empty for application-owned)   │
│                                         │
│ Owner Managed Access: ☐ OFF             │
│   (Enable for user-owned resources)     │
│                                         │
│ Attributes:                             │
│   Key: classification                   │
│   Value: confidential                   │
└─────────────────────────────────────────┘
[Save]
```

**Example 2: User-Owned Document**

```text
Name: user-document
Type: urn:myapp:resources:user-document
URIs: /api/users/{userId}/documents/*
Scopes: view, edit, delete, share
Owner: ${user.id}
Owner Managed Access: ☑ ON
  └── Users can grant access to their documents
```

**Example 3: API Endpoint**

```text
Name: analytics-api
Type: urn:myapp:resources:api
URIs: 
  /api/analytics/*
  /api/reports/*
Scopes: read, write
```

**Example 4: Feature Toggle**

```text
Name: premium-features
Type: urn:myapp:resources:feature
URIs: (none - logical resource)
Scopes: access
Attributes:
  tier: premium
```

### Defining Scopes

**Navigate to**: Clients → client-name → Authorization → Authorization scopes

```text
[ Create authorization scope ]

Common Scopes:
┌─────────────────────────────────────────┐
│ document:view                           │
│ document:edit                           │
│ document:delete                         │
│ document:share                          │
│ api:read                                │
│ api:write                               │
│ feature:access                          │
│ admin:manage                            │
└─────────────────────────────────────────┘
```

**Creating Scope**:

```text
┌─────────────────────────────────────────┐
│ Name*: document:edit                    │
│                                         │
│ Display name: Edit Document             │
│                                         │
│ Icon URI:                               │
│   https://app.example.com/icons/edit.png│
└─────────────────────────────────────────┘
[Save]
```

### Creating Policies

Policies define authorization rules.[1]

**Navigate to**: Clients → client-name → Authorization → Policies

#### Role Policy

Grant access based on user roles.

```text
[ Create policy ] → Role

┌─────────────────────────────────────────┐
│ Name*: Admin Role Policy                │
│                                         │
│ Description: Grants access to admins    │
│                                         │
│ Realm Roles:                            │
│   ☑ admin                               │
│   ☑ super-admin                         │
│                                         │
│ Client Roles:                           │
│   Client: document-app                  │
│   ☑ document-admin                      │
│                                         │
│ Logic: Positive                     [▼] │
│   Positive: Grant if role exists        │
│   Negative: Deny if role exists         │
└─────────────────────────────────────────┘
[Save]
```

#### User Policy

Grant access to specific users.

```text
[ Create policy ] → User

┌─────────────────────────────────────────┐
│ Name*: CEO Policy                       │
│                                         │
│ Users:                                  │
│   [Search users...]                     │
│   Selected:                             │
│   • ceo@company.com                     │
│   • cto@company.com                     │
│                                         │
│ Logic: Positive                         │
└─────────────────────────────────────────┘
```

#### Group Policy

Grant access based on group membership.

```text
[ Create policy ] → Group

┌─────────────────────────────────────────┐
│ Name*: Engineering Group Policy         │
│                                         │
│ Groups:                                 │
│   ☑ /Engineering                        │
│   ☑ /Engineering/Backend                │
│   ☑ /Engineering/Frontend               │
│                                         │
│ Extend to Children: ☑ ON                │
│   (Include nested groups)               │
│                                         │
│ Logic: Positive                         │
└─────────────────────────────────────────┘
```

#### Time Policy

Grant access during specific time periods.

```text
[ Create policy ] → Time

┌─────────────────────────────────────────┐
│ Name*: Business Hours Policy            │
│                                         │
│ Not Before: 2025-01-01 00:00:00         │
│   (Policy active after this date)       │
│                                         │
│ Not On or After: 2025-12-31 23:59:59    │
│   (Policy expires on this date)         │
│                                         │
│ Day of Month: 1-31                      │
│   (All days of month)                   │
│                                         │
│ Month: 1-12                             │
│   (All months)                          │
│                                         │
│ Year: 2025                              │
│                                         │
│ Hour: 9-17                              │
│   (9 AM to 5 PM)                        │
│                                         │
│ Minute: *                               │
│   (All minutes)                         │
│                                         │
│ Logic: Positive                         │
└─────────────────────────────────────────┘
```

**Example: Weekend Policy**

```text
Name: Weekend Policy
Day of Month: *
Month: *
Year: *
Day of Week: 1,7 (Sunday=1, Saturday=7)
Logic: Negative (Deny on weekends)
```

#### Client Policy

Grant access based on which client is making the request.

```text
[ Create policy ] → Client

┌─────────────────────────────────────────┐
│ Name*: Mobile App Policy                │
│                                         │
│ Clients:                                │
│   ☑ mobile-app-ios                      │
│   ☑ mobile-app-android                  │
│                                         │
│ Logic: Positive                         │
└─────────────────────────────────────────┘
```

#### JavaScript Policy

Custom logic using JavaScript.

```text
[ Create policy ] → JavaScript

┌─────────────────────────────────────────┐
│ Name*: Custom Business Logic            │
│                                         │
│ Code*:                                  │
│                                         │
│ var context = $evaluation.getContext(); │
│ var identity = context.getIdentity();   │
│ var attributes = identity.getAttributes();│
│ var permission = $evaluation            │
│     .getPermission();                   │
│ var resource = permission.getResource();│
│                                         │
│ // Get custom attributes                │
│ var tier = attributes                   │
│     .getValue('customer_tier');         │
│ var points = attributes                 │
│     .getValue('loyalty_points');        │
│                                         │
│ // Business logic                       │
│ if (tier.asString(0) === 'platinum') {  │
│     $evaluation.grant();                │
│ } else if (tier.asString(0) === 'gold'  │
│            && points.asLong(0) > 10000){│
│     $evaluation.grant();                │
│ } else {                                │
│     $evaluation.deny();                 │
│ }                                       │
│                                         │
│ Logic: Positive                         │
└─────────────────────────────────────────┘
```

**Advanced JavaScript Example** (Resource attributes):

```javascript
var context = $evaluation.getContext();
var resource = $evaluation.getPermission().getResource();
var user = context.getIdentity().getAttributes();

// Get resource classification
var classification = resource.getAttribute('classification');

// Get user clearance level
var clearance = user.getValue('security_clearance');

// Grant if clearance >= classification
var clearanceLevels = {
    'public': 0,
    'internal': 1,
    'confidential': 2,
    'secret': 3,
    'top-secret': 4
};

var requiredLevel = clearanceLevels[classification[0]] || 0;
var userLevel = clearanceLevels[clearance.asString(0)] || 0;

if (userLevel >= requiredLevel) {
    $evaluation.grant();
} else {
    $evaluation.deny();
}
```

#### Aggregate Policy

Combine multiple policies with AND/OR logic.

```text
[ Create policy ] → Aggregated

┌─────────────────────────────────────────┐
│ Name*: Manager Access                   │
│                                         │
│ Description: Manager during business hrs│
│                                         │
│ Apply Policy:                           │
│   ☑ Manager Role Policy                 │
│   ☑ Business Hours Policy               │
│   ☐ Weekend Policy                      │
│                                         │
│ Decision Strategy: Unanimous        [▼] │
│   Unanimous: ALL policies must grant    │
│   Affirmative: ONE policy must grant    │
│   Consensus: MAJORITY must grant        │
│                                         │
│ Logic: Positive                         │
└─────────────────────────────────────────┘
```

### Creating Permissions

Permissions link resources, scopes, and policies.

**Navigate to**: Clients → client-name → Authorization → Permissions

#### Resource-Based Permission

```text
[ Create permission ] → Create resource-based permission

┌─────────────────────────────────────────┐
│ Name*: Document View Permission         │
│                                         │
│ Description: Who can view documents     │
│                                         │
│ Resources:                              │
│   ☑ document                            │
│   ☐ user-document                       │
│   ☐ analytics-api                       │
│                                         │
│ Apply Policy:                           │
│   ☑ Admin Role Policy                   │
│   ☑ User Policy                         │
│   ☑ Engineering Group Policy            │
│                                         │
│ Decision Strategy: Affirmative      [▼] │
│   (At least one policy must grant)      │
│                                         │
│ Description:                            │
│   Admins, specific users, or            │
│   engineering team can view documents   │
└─────────────────────────────────────────┘
[Save]
```

#### Scope-Based Permission

```text
[ Create permission ] → Create scope-based permission

┌─────────────────────────────────────────┐
│ Name*: Document Edit Permission         │
│                                         │
│ Resources:                              │
│   ☑ document                            │
│                                         │
│ Scopes:                                 │
│   ☐ document:view                       │
│   ☑ document:edit                       │
│   ☐ document:delete                     │
│                                         │
│ Apply Policy:                           │
│   ☑ Admin Role Policy                   │
│   ☑ Manager Access (Aggregate)          │
│                                         │
│ Decision Strategy: Affirmative          │
└─────────────────────────────────────────┘
```

#### Complex Permission Example

```text
Name: Delete Document Permission

Resources: document
Scopes: document:delete

Policies:
☑ Admin Role Policy (Must be admin)
☑ Business Hours Policy (During work hours)
☑ Document Owner Policy (Custom JS checking ownership)

Decision Strategy: Unanimous
  └── ALL conditions must be met

JavaScript Policy (Document Owner):
───────────────────────────────────────────
var context = $evaluation.getContext();
var identity = context.getIdentity();
var resource = $evaluation.getPermission().getResource();

// Get resource owner from attributes
var owner = resource.getOwner();

// Check if current user is owner
if (owner && owner.getId() === identity.getId()) {
    $evaluation.grant();
} else {
    $evaluation.deny();
}
───────────────────────────────────────────
```

### Requesting Authorization

#### Obtaining RPT (Requesting Party Token)

**Authorization Code Flow** (User Context):

```bash
# Get RPT with permissions
curl -X POST https://auth.example.com/realms/customer-portal/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \
  -d "audience=document-app" \
  -d "access_token=${USER_ACCESS_TOKEN}"
```

**Response**:
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 300,
  "authorization": {
    "permissions": [
      {
        "rsid": "resource-id-1",
        "rsname": "document",
        "scopes": ["document:view", "document:edit"]
      },
      {
        "rsid": "resource-id-2",
        "rsname": "analytics-api",
        "scopes": ["read"]
      }
    ]
  }
}
```

**Requesting Specific Resources**:

```bash
curl -X POST https://auth.example.com/realms/customer-portal/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \
  -d "audience=document-app" \
  -d "permission=document#document:view" \
  -d "permission=document#document:edit" \
  -d "access_token=${USER_ACCESS_TOKEN}"
```

#### Enforcing Authorization in Backend

**Java Spring Boot**:

```java
@RestController
@RequestMapping("/api/documents")
public class DocumentController {
    
    @Autowired
    private KeycloakAuthorizationService authzService;
    
    @GetMapping("/{id}")
    public ResponseEntity<Document> getDocument(
            @PathVariable String id,
            @AuthenticationPrincipal Jwt jwt) {
        
        // Check permission
        boolean hasPermission = authzService.checkPermission(
            jwt.getTokenValue(),
            "document",
            "document:view"
        );
        
        if (!hasPermission) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
        }
        
        Document doc = documentService.findById(id);
        return ResponseEntity.ok(doc);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Document> updateDocument(
            @PathVariable String id,
            @RequestBody Document document,
            @AuthenticationPrincipal Jwt jwt) {
        
        // Check edit permission
        boolean canEdit = authzService.checkPermission(
            jwt.getTokenValue(),
            "document",
            "document:edit"
        );
        
        if (!canEdit) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
        }
        
        Document updated = documentService.update(id, document);
        return ResponseEntity.ok(updated);
    }
}
```

**Authorization Service**:

```java
@Service
public class KeycloakAuthorizationService {
    
    @Value("${keycloak.auth-server-url}")
    private String authServerUrl;
    
    @Value("${keycloak.realm}")
    private String realm;
    
    public boolean checkPermission(String accessToken, String resource, String scope) {
        try {
            // Request RPT
            String rptUrl = String.format(
                "%s/realms/%s/protocol/openid-connect/token",
                authServerUrl,
                realm
            );
            
            MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
            body.add("grant_type", "urn:ietf:params:oauth:grant-type:uma-ticket");
            body.add("audience", "document-app");
            body.add("permission", resource + "#" + scope);
            body.add("access_token", accessToken);
            
            RestTemplate restTemplate = new RestTemplate();
            ResponseEntity<Map> response = restTemplate.postForEntity(
                rptUrl,
                new HttpEntity<>(body),
                Map.class
            );
            
            // If we get RPT, permission granted
            return response.getStatusCode() == HttpStatus.OK;
            
        } catch (HttpClientErrorException e) {
            // 403 = permission denied
            return false;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
}
```

**Node.js Express**:

```javascript
const axios = require('axios');

async function checkPermission(accessToken, resource, scope) {
    try {
        const response = await axios.post(
            'https://auth.example.com/realms/customer-portal/protocol/openid-connect/token',
            new URLSearchParams({
                grant_type: 'urn:ietf:params:oauth:grant-type:uma-ticket',
                audience: 'document-app',
                permission: `${resource}#${scope}`,
                access_token: accessToken
            }),
            {
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
            }
        );
        
        // Got RPT = permission granted
        return true;
    } catch (error) {
        if (error.response?.status === 403) {
            // Permission denied
            return false;
        }
        throw error;
    }
}

// Middleware
async function requirePermission(resource, scope) {
    return async (req, res, next) => {
        const accessToken = req.headers.authorization?.split(' ')[1];
        
        if (!accessToken) {
            return res.status(401).json({ error: 'No token provided' });
        }
        
        const hasPermission = await checkPermission(accessToken, resource, scope);
        
        if (hasPermission) {
            next();
        } else {
            res.status(403).json({ error: 'Forbidden' });
        }
    };
}

// Routes
app.get('/api/documents/:id', 
    requirePermission('document', 'document:view'),
    (req, res) => {
        const document = getDocument(req.params.id);
        res.json(document);
    }
);

app.put('/api/documents/:id',
    requirePermission('document', 'document:edit'),
    (req, res) => {
        const updated = updateDocument(req.params.id, req.body);
        res.json(updated);
    }
);
```

***

## 12. Integrating OpenFGA with Keycloak for Relationship-Based Access Control (ReBAC)

### Why OpenFGA with Keycloak?

While Keycloak excels at authentication and role-based access control (RBAC), it has limitations for complex relationship-based authorization scenarios:[1][2]

**Keycloak's Limitations**:
- Role explosions in multi-tenant scenarios
- Difficulty modeling resource ownership
- Limited support for hierarchical relationships
- Complex delegation and sharing patterns
- Performance issues with fine-grained policies

**OpenFGA's Strengths**:[3][4]
- Google Zanzibar-inspired authorization
- Relationship-based access control (ReBAC)
- Scales to billions of checks per day
- Natural modeling of real-world relationships
- Efficient graph-based evaluation

**Combined Architecture**:
- **Keycloak**: Authentication, identity, basic roles
- **OpenFGA**: Fine-grained authorization, relationships, sharing

### Real-World Use Cases for OpenFGA

**Use Case 1: Document Management System**

Traditional RBAC:
```text
❌ Problem: Need roles for every permission combination
- document-viewer
- document-editor
- document-owner
- document-shared-viewer
- document-shared-editor
... (exponential growth)
```

With OpenFGA:
```text
✅ Solution: Model relationships naturally
- alice is owner of document:readme
- bob is editor of document:readme
- charlie can view document:readme
- document:readme is in folder:engineering
- Anyone in organization:acme can view folder:engineering
```

**Use Case 2: Multi-Tenant SaaS**

```text
Organization: Acme Corp
├── alice (owner)
├── bob (admin)
├── charlie (member)
│
└── Workspace: Product Development
    ├── bob (owner)
    ├── charlie (editor)
    │
    └── Document: Roadmap 2025
        ├── charlie (owner)
        └── diana (viewer) [external user]

Authorization Questions:
- Can alice access Roadmap 2025? 
  → Yes (organization owner → workspace → document)
- Can diana access other documents in workspace?
  → No (only has access to specific document)
- Can bob delete workspace?
  → Yes (workspace owner)
```

**Use Case 3: Collaborative Features**

```text
Users can:
- Share documents with specific users
- Grant edit/view permissions
- Share with entire teams/organizations
- Inherit permissions from parent folders
- Delegate administrative rights
```

### OpenFGA Concepts Deep Dive

#### Relation Tuples

The fundamental unit in OpenFGA is a relation tuple:[5]

```text
Format: <object>#<relation>@<user>

Components:
- object: <type>:<id> (what resource)
- relation: relationship name (what connection)
- user: <type>:<id> or <type>:<id>#<relation> (who/what has access)
```

**Examples**:

```text
Direct relationship:
document:readme#owner@user:alice
  └── Alice is owner of document:readme

Indirect relationship (via group):
document:readme#viewer@organization:acme#member
  └── All members of organization:acme can view document:readme

Hierarchical relationship:
document:readme#parent@folder:engineering
  └── document:readme is in folder:engineering

Computed relationship:
document:readme can be viewed by anyone who is:
  - Direct viewer
  - Direct editor  
  - Direct owner
  - Viewer/editor/owner of parent folder
```

### Setting Up Complete Integration

#### Architecture Overview

```text
┌──────────────────────────────────────────────────────────┐
│                    Frontend Application                   │
│              (React, Vue, Angular, Mobile)                │
└────────────┬──────────────────┬──────────────────────────┘
             │                  │
             │ 1. Authenticate  │ 4. Access Resources
             ▼                  ▼
┌────────────────────┐    ┌─────────────────────────────┐
│     Keycloak       │    │     Backend API             │
│  Authentication    │    │                             │
│  - Login           │    │  ┌──────────────────────┐   │
│  - Token issuance  │    │  │ Token Validation     │   │
│  - User management │    │  │ (Keycloak)           │   │
│  - Roles           │    │  └──────────┬───────────┘   │
└──────┬─────────────┘    │             │               │
       │                  │             ▼               │
       │ 2. Sync Events   │  ┌──────────────────────┐   │
       │    (User/Role    │  │ Authorization Check  │   │
       │     changes)     │  │ (OpenFGA)           │   │
       ▼                  │  └──────────┬───────────┘   │
┌────────────────────┐    │             │               │
│  Event Publisher   │    │             ▼               │
│   Extension        │    │  ┌──────────────────────┐   │
│  - Listens events  ├────┼─>│ Business Logic       │   │
│  - Converts tuples │    │  └──────────────────────┘   │
│  - Writes OpenFGA  │    └─────────────────────────────┘
└──────┬─────────────┘                  │
       │ 3. Write Tuples                │
       ▼                                │
┌────────────────────┐                  │
│     OpenFGA        │<─────────────────┘
│  Authorization     │   5. Check Permissions
│  - Store tuples    │
│  - Evaluate graph  │
│  - Return decision │
└────────────────────┘
```

#### Installing OpenFGA

**Docker Compose Setup** (Production-ready):

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14-alpine
    container_name: openfga-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-changeMe}
      POSTGRES_DB: openfga
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - keycloak-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  openfga-migrate:
    image: openfga/openfga:v1.5.5
    container_name: openfga-migrate
    depends_on:
      postgres:
        condition: service_healthy
    command: migrate
    environment:
      OPENFGA_DATASTORE_ENGINE: postgres
      OPENFGA_DATASTORE_URI: postgres://postgres:${POSTGRES_PASSWORD:-changeMe}@postgres:5432/openfga?sslmode=disable
    networks:
      - keycloak-network

  openfga:
    image: openfga/openfga:v1.5.5
    container_name: openfga
    restart: unless-stopped
    depends_on:
      openfga-migrate:
        condition: service_completed_successfully
    ports:
      - "3000:3000"  # Playground UI
      - "8080:8080"  # HTTP API
      - "8081:8081"  # gRPC API
    command: run
    environment:
      OPENFGA_DATASTORE_ENGINE: postgres
      OPENFGA_DATASTORE_URI: postgres://postgres:${POSTGRES_PASSWORD:-changeMe}@postgres:5432/openfga?sslmode=disable
      OPENFGA_LOG_LEVEL: info
      OPENFGA_PLAYGROUND_ENABLED: true
      OPENFGA_PLAYGROUND_PORT: 3000
      OPENFGA_HTTP_ADDR: 0.0.0.0:8080
      OPENFGA_GRPC_ADDR: 0.0.0.0:8081
      OPENFGA_AUTHN_METHOD: none  # Or configure with preshared keys
      OPENFGA_METRICS_ENABLED: true
      OPENFGA_METRICS_ADDR: 0.0.0.0:2112
    networks:
      - keycloak-network
    healthcheck:
      test: ["CMD", "grpc_health_probe", "-addr=:8081"]
      interval: 30s
      timeout: 10s
      retries: 3

  keycloak:
    image: quay.io/keycloak/keycloak:25.0.1
    container_name: keycloak
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: postgres
      KC_DB_PASSWORD: ${POSTGRES_PASSWORD:-changeMe}
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: ${KEYCLOAK_ADMIN_PASSWORD:-admin}
      KC_HOSTNAME: localhost
      KC_HTTP_ENABLED: true
      KC_HEALTH_ENABLED: true
      KC_METRICS_ENABLED: true
      # OpenFGA extension configuration
      KC_SPI_EVENTS_LISTENER_OPENFGA_EVENTS_PUBLISHER_API_URL: http://openfga:8080
      KC_SPI_EVENTS_LISTENER_OPENFGA_EVENTS_PUBLISHER_STORE_ID: ${OPENFGA_STORE_ID}
      KC_SPI_EVENTS_LISTENER_OPENFGA_EVENTS_PUBLISHER_MODEL_ID: ${OPENFGA_MODEL_ID}
      KC_SPI_EVENTS_LISTENER_OPENFGA_EVENTS_PUBLISHER_ENABLED: true
    ports:
      - "8090:8080"
    networks:
      - keycloak-network
    volumes:
      - ./keycloak-openfga-extension.jar:/opt/keycloak/providers/keycloak-openfga-extension.jar
    command: start-dev

volumes:
  postgres-data:
    driver: local

networks:
  keycloak-network:
    driver: bridge
```

**Start the stack**:

```bash
docker-compose up -d
```

**Access points**:
- Keycloak: http://localhost:8090
- OpenFGA API: http://localhost:8080
- OpenFGA Playground: http://localhost:3000

#### Creating OpenFGA Authorization Model

**Comprehensive Model** for document management with multi-tenancy:

Create `authorization-model.fga`:

```dsl
model
  schema 1.1

type user

type organization
  relations
    define owner: [user]
    define admin: [user]
    define member: [user]
    define can_create_workspace: owner or admin
    define can_invite_member: owner or admin or member

type workspace
  relations
    define organization: [organization]
    define owner: [user]
    define admin: [user]
    define member: [user]
    define can_view: owner or admin or member or member from organization
    define can_edit: owner or admin
    define can_delete: owner
    define can_create_folder: owner or admin or member
    define can_invite: owner or admin

type folder
  relations
    define workspace: [workspace]
    define parent: [folder]
    define owner: [user, organization#member]
    define editor: [user, organization#member]
    define viewer: [user, organization#member]
    define can_view: viewer or editor or owner or can_view from parent or can_view from workspace
    define can_edit: editor or owner or owner from parent
    define can_delete: owner
    define can_create_document: owner or editor or editor from parent
    define can_share: owner or editor

type document
  relations
    define workspace: [workspace]
    define folder: [folder]
    define owner: [user, organization#member]
    define editor: [user, organization#member]
    define commenter: [user, organization#member]
    define viewer: [user, organization#member]
    define parent: [folder]
    define can_view: viewer or commenter or editor or owner or can_view from parent or can_view from workspace
    define can_comment: commenter or editor or owner or editor from parent
    define can_edit: editor or owner or can_edit from parent
    define can_delete: owner
    define can_share: owner or editor
    define can_change_owner: owner

type role
  relations
    define assignee: [user, role#assignee]
    define organization: [organization]
    define workspace: [workspace]

type permission
  relations
    define role: [role]
    define granted: assignee from role
```

**Upload Model via CLI**:

```bash
# Install FGA CLI
# macOS
brew install openfga/tap/fga

# Linux
wget https://github.com/openfga/cli/releases/download/v0.2.5/fga_0.2.5_linux_amd64.tar.gz
tar -xzf fga_0.2.5_linux_amd64.tar.gz
sudo mv fga /usr/local/bin/

# Configure API endpoint
export FGA_API_URL=http://localhost:8080

# Create store
fga store create --name "keycloak-authz"

# Output will show store ID
# Store ID: 01HQXYZ123456789ABCDEF

export FGA_STORE_ID="01HQXYZ123456789ABCDEF"

# Upload authorization model
fga model write --file authorization-model.fga

# Output will show model ID
# Model ID: 01HQXYZ987654321FEDCBA

export FGA_MODEL_ID="01HQXYZ987654321FEDCBA"
```

**Verify model**:

```bash
fga model get
```

**Via HTTP API**:

```bash
# Create store
STORE_RESPONSE=$(curl -X POST http://localhost:8080/stores \
  -H "Content-Type: application/json" \
  -d '{"name": "keycloak-authz"}')

STORE_ID=$(echo $STORE_RESPONSE | jq -r '.id')
echo "Store ID: $STORE_ID"

# Write authorization model
MODEL_RESPONSE=$(curl -X POST "http://localhost:8080/stores/${STORE_ID}/authorization-models" \
  -H "Content-Type: application/json" \
  -d @authorization-model.json)

MODEL_ID=$(echo $MODEL_RESPONSE | jq -r '.authorization_model_id')
echo "Model ID: $MODEL_ID"
```

#### Installing Keycloak OpenFGA Event Publisher

**Build from Source**:

```bash
# Clone repository
git clone https://github.com/embesozzi/keycloak-openfga-event-publisher.git
cd keycloak-openfga-event-publisher

# Build with Maven
mvn clean package

# Copy JAR to Keycloak providers
cp target/keycloak-openfga-event-publisher-*.jar /opt/keycloak/providers/

# For Docker, copy to volume
cp target/keycloak-openfga-event-publisher-*.jar ./keycloak-openfga-extension.jar
```

**Configure Keycloak** (`keycloak.conf` or environment variables):

```properties
# OpenFGA Event Publisher Configuration
spi-events-listener-openfga-events-publisher-api-url=http://openfga:8080
spi-events-listener-openfga-events-publisher-store-id=01HQXYZ123456789ABCDEF
spi-events-listener-openfga-events-publisher-model-id=01HQXYZ987654321FEDCBA
spi-events-listener-openfga-events-publisher-enabled=true
spi-events-listener-openfga-events-publisher-authorization-model-name=keycloak-authz

# Optional: Event filtering
spi-events-listener-openfga-events-publisher-include-events=REGISTER,UPDATE_PROFILE,UPDATE_EMAIL
spi-events-listener-openfga-events-publisher-exclude-events=LOGIN,LOGOUT
```

**Enable Event Listener in Realm**:

```text
1. Login to Keycloak Admin Console
2. Select realm (customer-portal)
3. Navigate: Realm Settings → Events → Event listeners
4. Add "openfga-events-publisher" to the list
5. Click "Save"
```

**Verify Installation**:

```bash
# Check Keycloak logs
docker logs keycloak 2>&1 | grep -i openfga

# Should see:
# OpenFGA Events Publisher initialized
# Connected to OpenFGA at http://openfga:8080
# Using store: 01HQXYZ123456789ABCDEF
```

#### Event Mapping and Tuple Creation

**How Extension Works**:[2][6]

**Event 1: User Created**

```text
Keycloak Event:
{
  "type": "REGISTER",
  "realmId": "customer-portal",
  "userId": "user-uuid-123",
  "username": "alice@company.com",
  "details": {...}
}

↓ Extension Converts ↓

OpenFGA Tuple:
user:user-uuid-123
(Creates user object in OpenFGA)
```

**Event 2: User Assigned to Group**

```text
Keycloak Event:
{
  "type": "USER_GROUP_MEMBERSHIP",
  "userId": "user-uuid-123",
  "groupId": "group-uuid-456",
  "groupPath": "/Acme Corp/Engineering"
}

↓ Extension Converts ↓

OpenFGA Tuples:
organization:acme-corp#member@user:user-uuid-123
workspace:engineering#member@user:user-uuid-123
```

**Event 3: Role Assigned to User**

```text
Keycloak Event:
{
  "type": "USER_ROLE_ASSIGNMENT",
  "userId": "user-uuid-123",
  "roleName": "admin",
  "clientId": "document-app"
}

↓ Extension Converts ↓

OpenFGA Tuple:
role:admin#assignee@user:user-uuid-123
```

**Event 4: Composite Role Created**

```text
Keycloak Event:
{
  "type": "ROLE_TO_ROLE_ASSIGNMENT",
  "parentRole": "super-admin",
  "childRole": "admin"
}

↓ Extension Converts ↓

OpenFGA Tuple:
role:super-admin#assignee@role:admin#assignee
(super-admin inherits all admin permissions)
```

#### Application Integration with OpenFGA SDK

**Install SDKs**:[7]

```bash
# Node.js
npm install @openfga/sdk dotenv

# Python
pip install openfga-sdk python-dotenv

# Java
# Add to pom.xml
<dependency>
    <groupId>dev.openfga</groupId>
    <artifactId>openfga-sdk</artifactId>
    <version>0.3.1</version>
</dependency>

# Go
go get github.com/openfga/go-sdk
```

**Node.js Complete Integration**:

```javascript
// config/openfga.js
const { OpenFgaClient } = require('@openfga/sdk');
require('dotenv').config();

const fgaClient = new OpenFgaClient({
  apiUrl: process.env.FGA_API_URL || 'http://localhost:8080',
  storeId: process.env.FGA_STORE_ID,
  authorizationModelId: process.env.FGA_MODEL_ID
});

module.exports = fgaClient;
```

```javascript
// middleware/keycloakAuth.js
const Keycloak = require('keycloak-connect');

const keycloak = new Keycloak({}, {
  realm: process.env.KEYCLOAK_REALM,
  'auth-server-url': process.env.KEYCLOAK_URL,
  'ssl-required': 'external',
  resource: process.env.KEYCLOAK_CLIENT_ID,
  credentials: {
    secret: process.env.KEYCLOAK_CLIENT_SECRET
  }
});

module.exports = keycloak;
```

```javascript
// middleware/openfgaAuth.js
const fgaClient = require('../config/openfga');

/**
 * Middleware to check OpenFGA permissions
 * @param {string} resourceType - Type of resource (document, folder, etc.)
 * @param {string} relation - Required relation (can_view, can_edit, etc.)
 * @param {function} getResourceId - Function to extract resource ID from request
 */
function checkPermission(resourceType, relation, getResourceId) {
  return async (req, res, next) => {
    try {
      // Get user ID from Keycloak token
      const userId = req.kauth.grant.access_token.content.sub;
      
      // Get resource ID from request
      const resourceId = getResourceId(req);
      
      // Check permission in OpenFGA
      const { allowed } = await fgaClient.check({
        user: `user:${userId}`,
        relation: relation,
        object: `${resourceType}:${resourceId}`
      });
      
      if (allowed) {
        // Store permission info for later use
        req.fgaPermission = { resourceType, resourceId, relation, allowed: true };
        next();
      } else {
        res.status(403).json({ 
          error: 'Forbidden', 
          message: `You don't have permission to ${relation} this ${resourceType}`
        });
      }
    } catch (error) {
      console.error('Authorization check failed:', error);
      res.status(500).json({ error: 'Authorization service error' });
    }
  };
}

/**
 * Batch check multiple permissions
 */
async function checkMultiplePermissions(userId, checks) {
  const results = await Promise.all(
    checks.map(check => 
      fgaClient.check({
        user: `user:${userId}`,
        relation: check.relation,
        object: `${check.resourceType}:${check.resourceId}`
      })
    )
  );
  
  return results.map((result, index) => ({
    ...checks[index],
    allowed: result.allowed
  }));
}

/**
 * List objects user can access
 */
async function listAccessibleObjects(userId, resourceType, relation) {
  const { objects } = await fgaClient.listObjects({
    user: `user:${userId}`,
    relation: relation,
    type: resourceType
  });
  
  return objects.map(obj => obj.split(':')[1]); // Extract IDs
}

module.exports = {
  checkPermission,
  checkMultiplePermissions,
  listAccessibleObjects
};
```

```javascript
// routes/documents.js
const express = require('express');
const router = express.Router();
const keycloak = require('../middleware/keycloakAuth');
const { checkPermission, checkMultiplePermissions, listAccessibleObjects } = require('../middleware/openfgaAuth');
const fgaClient = require('../config/openfga');

// List all documents user can view
router.get('/', 
  keycloak.protect(),
  async (req, res) => {
    try {
      const userId = req.kauth.grant.access_token.content.sub;
      
      // Get document IDs user can view
      const documentIds = await listAccessibleObjects(userId, 'document', 'can_view');
      
      // Fetch document data from database
      const documents = await Document.find({ _id: { $in: documentIds } });
      
      // Check additional permissions for each document
      const permissions = await checkMultiplePermissions(
        userId,
        documentIds.map(id => [
          { resourceType: 'document', resourceId: id, relation: 'can_edit' },
          { resourceType: 'document', resourceId: id, relation: 'can_delete' },
          { resourceType: 'document', resourceId: id, relation: 'can_share' }
        ]).flat()
      );
      
      // Attach permissions to documents
      const documentsWithPermissions = documents.map(doc => {
        const docPermissions = permissions.filter(p => p.resourceId === doc._id.toString());
        return {
          ...doc.toObject(),
          permissions: {
            can_view: true, // Already filtered by can_view
            can_edit: docPermissions.find(p => p.relation === 'can_edit')?.allowed || false,
            can_delete: docPermissions.find(p => p.relation === 'can_delete')?.allowed || false,
            can_share: docPermissions.find(p => p.relation === 'can_share')?.allowed || false
          }
        };
      });
      
      res.json(documentsWithPermissions);
    } catch (error) {
      console.error('Error fetching documents:', error);
      res.status(500).json({ error: 'Failed to fetch documents' });
    }
  }
);

// Get specific document
router.get('/:documentId',
  keycloak.protect(),
  checkPermission('document', 'can_view', req => req.params.documentId),
  async (req, res) => {
    try {
      const document = await Document.findById(req.params.documentId);
      
      if (!document) {
        return res.status(404).json({ error: 'Document not found' });
      }
      
      // Get user's permissions for this document
      const userId = req.kauth.grant.access_token.content.sub;
      const permissions = await checkMultiplePermissions(userId, [
        { resourceType: 'document', resourceId: req.params.documentId, relation: 'can_view' },
        { resourceType: 'document', resourceId: req.params.documentId, relation: 'can_edit' },
        { resourceType: 'document', resourceId: req.params.documentId, relation: 'can_delete' },
        { resourceType: 'document', resourceId: req.params.documentId, relation: 'can_share' }
      ]);
      
      res.json({
        ...document.toObject(),
        permissions: {
          can_view: permissions.find(p => p.relation === 'can_view').allowed,
          can_edit: permissions.find(p => p.relation === 'can_edit').allowed,
          can_delete: permissions.find(p => p.relation === 'can_delete').allowed,
          can_share: permissions.find(p => p.relation === 'can_share').allowed
        }
      });
    } catch (error) {
      console.error('Error fetching document:', error);
      res.status(500).json({ error: 'Failed to fetch document' });
    }
  }
);

// Update document
router.put('/:documentId',
  keycloak.protect(),
  checkPermission('document', 'can_edit', req => req.params.documentId),
  async (req, res) => {
    try {
      const document = await Document.findByIdAndUpdate(
        req.params.documentId,
        { $set: req.body },
        { new: true }
      );
      
      res.json(document);
    } catch (error) {
      console.error('Error updating document:', error);
      res.status(500).json({ error: 'Failed to update document' });
    }
  }
);

// Delete document
router.delete('/:documentId',
  keycloak.protect(),
  checkPermission('document', 'can_delete', req => req.params.documentId),
  async (req, res) => {
    try {
      await Document.findByIdAndDelete(req.params.documentId);
      
      // Remove all OpenFGA tuples related to this document
      // This should be done via a background job or webhook
      
      res.json({ message: 'Document deleted successfully' });
    } catch (error) {
      console.error('Error deleting document:', error);
      res.status(500).json({ error: 'Failed to delete document' });
    }
  }
);

// Share document
router.post('/:documentId/share',
  keycloak.protect(),
  checkPermission('document', 'can_share', req => req.params.documentId),
  async (req, res) => {
    try {
      const { userEmail, permission } = req.body; // permission: 'viewer', 'editor', etc.
      
      // Find user by email
      const targetUser = await User.findOne({ email: userEmail });
      if (!targetUser) {
        return res.status(404).json({ error: 'User not found' });
      }
      
      // Write tuple to OpenFGA
      await fgaClient.write({
        writes: [{
          user: `user:${targetUser.keycloakId}`,
          relation: permission, // 'viewer', 'editor', etc.
          object: `document:${req.params.documentId}`
        }]
      });
      
      // Send notification (optional)
      await sendShareNotification(targetUser.email, req.params.documentId);
      
      res.json({ 
        message: 'Document shared successfully',
        sharedWith: userEmail,
        permission: permission
      });
    } catch (error) {
      console.error('Error sharing document:', error);
      res.status(500).json({ error: 'Failed to share document' });
    }
  }
);

// Revoke access
router.delete('/:documentId/share/:userId',
  keycloak.protect(),
  checkPermission('document', 'can_share', req => req.params.documentId),
  async (req, res) => {
    try {
      // Get current permissions for this user
      const { tuples } = await fgaClient.read({
        user: `user:${req.params.userId}`,
        object: `document:${req.params.documentId}`
      });
      
      // Delete all permission tuples
      if (tuples.length > 0) {
        await fgaClient.write({
          deletes: tuples.map(tuple => ({
            user: tuple.key.user,
            relation: tuple.key.relation,
            object: tuple.key.object
          }))
        });
      }
      
      res.json({ message: 'Access revoked successfully' });
    } catch (error) {
      console.error('Error revoking access:', error);
      res.status(500).json({ error: 'Failed to revoke access' });
    }
  }
);

// List who has access to document
router.get('/:documentId/access',
  keycloak.protect(),
  checkPermission('document', 'can_view', req => req.params.documentId),
  async (req, res) => {
    try {
      // Read all tuples for this document
      const { tuples } = await fgaClient.read({
        object: `document:${req.params.documentId}`
      });
      
      // Group by user and collect permissions
      const accessList = {};
      for (const tuple of tuples) {
        const userId = tuple.key.user.split(':')[1];
        if (!accessList[userId]) {
          accessList[userId] = {
            userId,
            permissions: []
          };
        }
        accessList[userId].permissions.push(tuple.key.relation);
      }
      
      // Fetch user details
      const userIds = Object.keys(accessList);
      const users = await User.find({ keycloakId: { $in: userIds } });
      
      const result = users.map(user => ({
        ...accessList[user.keycloakId],
        email: user.email,
        name: user.name
      }));
      
      res.json(result);
    } catch (error) {
      console.error('Error listing access:', error);
      res.status(500).json({ error: 'Failed to list access' });
    }
  }
);

module.exports = router;
```

```javascript
// app.js
const express = require('express');
const session = require('express-session');
const keycloak = require('./middleware/keycloakAuth');
const documentRoutes = require('./routes/documents');

const app = express();

// Session for Keycloak
const memoryStore = new session.MemoryStore();
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: true,
  store: memoryStore
}));

app.use(keycloak.middleware({
  logout: '/logout',
  admin: '/'
}));

app.use(express.json());

// Routes
app.use('/api/documents', documentRoutes);

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**Python Flask Integration**:

```python
# config/openfga.py
import os
from openfga_sdk import OpenFgaClient, ClientConfiguration

configuration = ClientConfiguration(
    api_url=os.getenv('FGA_API_URL', 'http://localhost:8080'),
    store_id=os.getenv('FGA_STORE_ID'),
    authorization_model_id=os.getenv('FGA_MODEL_ID')
)

fga_client = OpenFgaClient(configuration)
```

```python
# middleware/auth.py
from functools import wraps
from flask import request, jsonify
import jwt
import requests
from config.openfga import fga_client

def require_auth(f):
    """Validate Keycloak JWT token"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        token = request.headers.get('Authorization', '').replace('Bearer ', '')
        
        if not token:
            return jsonify({'error': 'No token provided'}), 401
        
        try:
            # Decode without verification (for demo)
            # In production, verify signature with Keycloak public key
            decoded = jwt.decode(token, options={"verify_signature": False})
            request.user_id = decoded['sub']
            request.user_email = decoded.get('email')
            return f(*args, **kwargs)
        except jwt.ExpiredSignatureError:
            return jsonify({'error': 'Token expired'}), 401
        except jwt.InvalidTokenError:
            return jsonify({'error': 'Invalid token'}), 401
    
    return decorated_function

def require_permission(resource_type, relation, get_resource_id):
    """Check OpenFGA permission"""
    def decorator(f):
        @wraps(f)
        async def decorated_function(*args, **kwargs):
            resource_id = get_resource_id(kwargs)
            
            try:
                response = await fga_client.check({
                    "user": f"user:{request.user_id}",
                    "relation": relation,
                    "object": f"{resource_type}:{resource_id}"
                })
                
                if response.allowed:
                    return await f(*args, **kwargs)
                else:
                    return jsonify({'error': 'Forbidden'}), 403
                    
            except Exception as e:
                print(f"Authorization check failed: {e}")
                return jsonify({'error': 'Authorization service error'}), 500
        
        return decorated_function
    return decorator
```

```python
# routes/documents.py
from flask import Blueprint, request, jsonify
from middleware.auth import require_auth, require_permission
from config.openfga import fga_client
from models import Document, User

documents_bp = Blueprint('documents', __name__)

@documents_bp.route('/', methods=['GET'])
@require_auth
async def list_documents():
    """List all documents user can view"""
    try:
        # Get document IDs user can view
        response = await fga_client.list_objects({
            "user": f"user:{request.user_id}",
            "relation": "can_view",
            "type": "document"
        })
        
        document_ids = [obj.split(':')[1] for obj in response.objects]
        
        # Fetch documents from database
        documents = Document.query.filter(Document.id.in_(document_ids)).all()
        
        return jsonify([doc.to_dict() for doc in documents])
        
    except Exception as e:
        print(f"Error fetching documents: {e}")
        return jsonify({'error': 'Failed to fetch documents'}), 500

@documents_bp.route('/<document_id>', methods=['GET'])
@require_auth
@require_permission('document', 'can_view', lambda kwargs: kwargs['document_id'])
async def get_document(document_id):
    """Get specific document"""
    try:
        document = Document.query.get(document_id)
        
        if not document:
            return jsonify({'error': 'Document not found'}), 404
        
        # Check multiple permissions
        permissions = {}
        for relation in ['can_view', 'can_edit', 'can_delete', 'can_share']:
            response = await fga_client.check({
                "user": f"user:{request.user_id}",
                "relation": relation,
                "object": f"document:{document_id}"
            })
            permissions[relation] = response.allowed
        
        return jsonify({
            **document.to_dict(),
            'permissions': permissions
        })
        
    except Exception as e:
        print(f"Error fetching document: {e}")
        return jsonify({'error': 'Failed to fetch document'}), 500

@documents_bp.route('/<document_id>', methods=['PUT'])
@require_auth
@require_permission('document', 'can_edit', lambda kwargs: kwargs['document_id'])
async def update_document(document_id):
    """Update document"""
    try:
        document = Document.query.get(document_id)
        
        if not document:
            return jsonify({'error': 'Document not found'}), 404
        
        data = request.get_json()
        document.update(**data)
        document.save()
        
        return jsonify(document.to_dict())
        
    except Exception as e:
        print(f"Error updating document: {e}")
        return jsonify({'error': 'Failed to update document'}), 500

@documents_bp.route('/<document_id>/share', methods=['POST'])
@require_auth
@require_permission('document', 'can_share', lambda kwargs: kwargs['document_id'])
async def share_document(document_id):
    """Share document with another user"""
    try:
        data = request.get_json()
        user_email = data.get('userEmail')
        permission = data.get('permission')  # 'viewer', 'editor', etc.
        
        # Find target user
        target_user = User.query.filter_by(email=user_email).first()
        
        if not target_user:
            return jsonify({'error': 'User not found'}), 404
        
        # Write tuple to OpenFGA
        await fga_client.write({
            "writes": [{
                "user": f"user:{target_user.keycloak_id}",
                "relation": permission,
                "object": f"document:{document_id}"
            }]
        })
        
        return jsonify({
            'message': 'Document shared successfully',
            'sharedWith': user_email,
            'permission': permission
        })
        
    except Exception as e:
        print(f"Error sharing document: {e}")
        return jsonify({'error': 'Failed to share document'}), 500
```

#### Writing Tuples Programmatically

**Creating Resources**:

```javascript
// When document is created
async function createDocument(userId, documentData) {
  // 1. Create document in database
  const document = await Document.create({
    title: documentData.title,
    content: documentData.content,
    createdBy: userId
  });
  
  // 2. Write OpenFGA tuples
  await fgaClient.write({
    writes: [
      // Creator is owner
      {
        user: `user:${userId}`,
        relation: 'owner',
        object: `document:${document._id}`
      },
      // Set parent folder (if specified)
      ...(documentData.folderId ? [{
        user: `folder:${documentData.folderId}`,
        relation: 'parent',
        object: `document:${document._id}`
      }] : [])
    ]
  });
  
  return document;
}
```

**Moving Documents**:

```javascript
async function moveDocument(documentId, targetFolderId) {
  // Remove old parent relationship
  const { tuples } = await fgaClient.read({
    object: `document:${documentId}`,
    relation: 'parent'
  });
  
  const deletes = tuples.map(tuple => ({
    user: tuple.key.user,
    relation: tuple.key.relation,
    object: tuple.key.object
  }));
  
  // Add new parent relationship
  await fgaClient.write({
    deletes: deletes,
    writes: [{
      user: `folder:${targetFolderId}`,
      relation: 'parent',
      object: `document:${documentId}`
    }]
  });
}
```

**Transferring Ownership**:

```javascript
async function transferOwnership(documentId, newOwnerId) {
  // Check current user can transfer ownership
  const canTransfer = await fgaClient.check({
    user: `user:${currentUserId}`,
    relation: 'can_change_owner',
    object: `document:${documentId}`
  });
  
  if (!canTransfer.allowed) {
    throw new Error('You cannot transfer ownership of this document');
  }
  
  // Get current owner tuple
  const { tuples } = await fgaClient.read({
    object: `document:${documentId}`,
    relation: 'owner'
  });
  
  const currentOwnerTuple = tuples.find(t => t.key.user.startsWith('user:'));
  
  // Transfer ownership
  await fgaClient.write({
    deletes: [currentOwnerTuple.key],
    writes: [{
      user: `user:${newOwnerId}`,
      relation: 'owner',
      object: `document:${documentId}`
    }]
  });
}
```

### Advanced OpenFGA Patterns

#### Contextual Tuples (Temporary Permissions)

```javascript
// Check permission with temporary context
const { allowed } = await fgaClient.check({
  user: `user:${userId}`,
  relation: 'can_view',
  object: `document:${documentId}`,
  contextual_tuples: {
    tuple_keys: [
      // Temporarily grant view access
      {
        user: `user:${userId}`,
        relation: 'viewer',
        object: `document:${documentId}`
      }
    ]
  }
});
```

#### Conditional Relationships

```javascript
// In authorization model
type document
  relations
    define viewer: [user with condition:valid_subscription]
    define can_view: viewer

// Condition definition
condition valid_subscription(subscription_tier: string, subscription_expiry: timestamp) {
  subscription_tier in ["premium", "enterprise"] && 
  subscription_expiry > now()
}

// Check with context
const { allowed } = await fgaClient.check({
  user: `user:${userId}`,
  relation: 'can_view',
  object: `document:${documentId}`,
  context: {
    subscription_tier: 'premium',
    subscription_expiry: '2025-12-31T23:59:59Z'
  }
});
```

#### Wildcard Permissions

```javascript
// Grant access to all documents in a workspace
await fgaClient.write({
  writes: [{
    user: `user:${userId}`,
    relation: 'viewer',
    object: `workspace:${workspaceId}#all_documents`
  }]
});

// In authorization model
type workspace
  relations
    define all_documents: [user]

type document
  relations
    define workspace: [workspace]
    define can_view: viewer or viewer from workspace.all_documents
```

***



## 13. Session Management

Session management controls how long users stay authenticated and how sessions are tracked across applications.

### Understanding Keycloak Sessions

**Session Types**:

1. **User Session**: Active authentication session in Keycloak
2. **Client Session**: Per-application session within user session
3. **Offline Session**: Persistent session for offline access
4. **Remember Me Session**: Extended session when user checks "Remember Me"

**Session Flow**:

```text
User logs in
    ↓
User Session created in Keycloak
    ↓
Access token issued (5 min lifespan)
Refresh token issued (30 min lifespan)
    ↓
User accesses App A
    └── Client Session created for App A
    ↓
User accesses App B
    └── Client Session created for App B
    ↓
User stays inactive for 30 min
    └── Session expires (SSO Session Idle)
    ↓
User must re-authenticate
```

### Configuring Session Timeouts

**Navigate to**: Realm Settings → Sessions

```text
Session Configuration:
┌─────────────────────────────────────────────────┐
│ SSO Session Idle: 1800 seconds (30 min)        │
│   How long until inactive session expires       │
│   Recommended: 15-30 minutes for business apps  │
│                1-4 hours for consumer apps       │
│                                                  │
│ SSO Session Max: 36000 seconds (10 hours)       │
│   Maximum session duration regardless of        │
│   activity                                       │
│   Recommended: 8-12 hours for business          │
│                24-72 hours for consumer          │
│                                                  │
│ SSO Session Idle Remember Me: 604800 (7 days)   │
│   Extended idle timeout when "Remember Me"      │
│   checked                                        │
│   Recommended: 7-30 days                        │
│                                                  │
│ SSO Session Max Remember Me: 2592000 (30 days)  │
│   Maximum duration for "Remember Me"            │
│   Recommended: 30-90 days                       │
│                                                  │
│ Client Session Idle: 0 (use SSO settings)       │
│   Per-client idle timeout override              │
│   0 = inherit from SSO Session Idle             │
│                                                  │
│ Client Session Max: 0 (use SSO settings)        │
│   Per-client max duration override              │
│   0 = inherit from SSO Session Max              │
│                                                  │
│ Offline Session Idle: 2592000 (30 days)         │
│   Offline token idle timeout                    │
│   Recommended: 30-90 days                       │
│                                                  │
│ Offline Session Max: Limited                    │
│   ⦿ Disabled                                    │
│   ○ Enabled: 5184000 (60 days)                  │
│   Maximum offline session duration              │
│                                                  │
│ Login timeout: 1800 seconds (30 min)            │
│   How long user has to complete login           │
│   Recommended: 5-30 minutes                     │
│                                                  │
│ Login action timeout: 300 seconds (5 min)       │
│   How long to complete required actions         │
│   (OTP setup, profile update)                   │
│   Recommended: 5-15 minutes                     │
└─────────────────────────────────────────────────┘
[Save]
```

### Real-World Session Strategies

**Strategy 1: Banking Application** (High Security)

```properties
# Short sessions, no remember me
sso-session-idle=900               # 15 minutes
sso-session-max=3600               # 1 hour
sso-session-idle-remember-me=0     # Disabled
sso-session-max-remember-me=0      # Disabled
access-token-lifespan=300          # 5 minutes
refresh-token-max-reuse=0          # No reuse
```

**Strategy 2: E-commerce Site** (Balanced)

```properties
# Moderate sessions, extended remember me
sso-session-idle=1800              # 30 minutes
sso-session-max=43200              # 12 hours
sso-session-idle-remember-me=604800 # 7 days
sso-session-max-remember-me=2592000 # 30 days
access-token-lifespan=300          # 5 minutes
```

**Strategy 3: Social Media Platform** (Convenience)

```properties
# Long sessions, very long remember me
sso-session-idle=86400             # 24 hours
sso-session-max=604800             # 7 days
sso-session-idle-remember-me=2592000 # 30 days
sso-session-max-remember-me=7776000  # 90 days
access-token-lifespan=600          # 10 minutes
```

**Strategy 4: Internal Enterprise App** (Work Hours)

```properties
# Business hours aligned
sso-session-idle=3600              # 1 hour (lunch break)
sso-session-max=28800              # 8 hours (work day)
sso-session-idle-remember-me=0     # Disabled
sso-session-max-remember-me=0      # Disabled
access-token-lifespan=300          # 5 minutes
```

### Viewing Active Sessions

**Via Admin Console**:

**Navigate to**: Users → Select user → Sessions

```text
User Sessions:
┌────────────────────────────────────────────────────┐
│ Session ID: abc123...def456                        │
│ Started: 2025-10-23 01:00:00                       │
│ Last Access: 2025-10-23 01:25:00                   │
│ IP Address: 192.168.1.100                          │
│ Clients:                                           │
│   • web-app (Last access: 2025-10-23 01:25:00)    │
│   • mobile-app (Last access: 2025-10-23 01:15:00) │
│                                                    │
│ [Sign out]                                         │
├────────────────────────────────────────────────────┤
│ Session ID: xyz789...uvw012                        │
│ Started: 2025-10-22 09:00:00                       │
│ Last Access: 2025-10-22 17:30:00                   │
│ IP Address: 192.168.1.50                           │
│ Clients:                                           │
│   • web-app (Last access: 2025-10-22 17:30:00)    │
│                                                    │
│ [Sign out]                                         │
└────────────────────────────────────────────────────┘
```

**Via REST API**:

```bash
# Get all sessions for a user
curl -X GET "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/sessions" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

**Response**:
```json
[
  {
    "id": "session-uuid-123",
    "username": "john.doe@example.com",
    "userId": "user-uuid",
    "ipAddress": "192.168.1.100",
    "start": 1729642800000,
    "lastAccess": 1729643300000,
    "clients": {
      "client-uuid-1": "web-app",
      "client-uuid-2": "mobile-app"
    }
  }
]
```

### Managing Sessions Programmatically

**Terminate User Sessions**:

```bash
# Logout user (terminate all sessions)
curl -X POST "http://localhost:8080/admin/realms/customer-portal/users/${USER_ID}/logout" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

**Terminate Specific Session**:

```bash
# Delete specific session
curl -X DELETE "http://localhost:8080/admin/realms/customer-portal/sessions/${SESSION_ID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

**Count Active Sessions**:

```bash
# Get session count
curl -X GET "http://localhost:8080/admin/realms/customer-portal/sessions/count" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

**Response**: `{"count": 1247}`

### Session Management in Applications

**Token Refresh Pattern** (JavaScript):

```javascript
// tokenManager.js
class TokenManager {
  constructor(keycloak) {
    this.keycloak = keycloak;
    this.refreshInterval = null;
  }

  /**
   * Start automatic token refresh
   * Refreshes token 30 seconds before expiration
   */
  startAutoRefresh() {
    // Check token expiry every 10 seconds
    this.refreshInterval = setInterval(async () => {
      try {
        const minValidity = 30; // Refresh if expiring in 30s
        const refreshed = await this.keycloak.updateToken(minValidity);
        
        if (refreshed) {
          console.log('Token refreshed successfully');
          // Update stored tokens
          this.saveTokens();
          // Emit event for token update
          window.dispatchEvent(new CustomEvent('tokenRefreshed', {
            detail: {
              token: this.keycloak.token,
              refreshToken: this.keycloak.refreshToken
            }
          }));
        }
      } catch (error) {
        console.error('Token refresh failed:', error);
        // Session expired, redirect to login
        this.keycloak.login();
      }
    }, 10000); // Check every 10 seconds
  }

  stopAutoRefresh() {
    if (this.refreshInterval) {
      clearInterval(this.refreshInterval);
      this.refreshInterval = null;
    }
  }

  saveTokens() {
    sessionStorage.setItem('access_token', this.keycloak.token);
    sessionStorage.setItem('refresh_token', this.keycloak.refreshToken);
    sessionStorage.setItem('id_token', this.keycloak.idToken);
  }

  /**
   * Check if session is still valid
   */
  async isSessionValid() {
    try {
      const minValidity = 0; // Check immediate expiry
      const valid = await this.keycloak.updateToken(minValidity);
      return true;
    } catch (error) {
      return false;
    }
  }

  /**
   * Force session refresh
   */
  async forceRefresh() {
    try {
      const refreshed = await this.keycloak.updateToken(-1); // Force refresh
      if (refreshed) {
        this.saveTokens();
        return true;
      }
      return false;
    } catch (error) {
      console.error('Force refresh failed:', error);
      return false;
    }
  }
}

export default TokenManager;
```

**Session Monitoring** (React Hook):

```javascript
// useSessionMonitor.js
import { useState, useEffect } from 'react';
import { useKeycloak } from '@react-keycloak/web';

export function useSessionMonitor() {
  const { keycloak } = useKeycloak();
  const [sessionInfo, setSessionInfo] = useState({
    isActive: false,
    expiresIn: 0,
    lastActivity: null
  });

  useEffect(() => {
    if (!keycloak.authenticated) {
      return;
    }

    // Calculate time until token expiry
    const updateExpiry = () => {
      const now = Date.now() / 1000;
      const exp = keycloak.tokenParsed?.exp || 0;
      const expiresIn = Math.max(0, exp - now);

      setSessionInfo({
        isActive: expiresIn > 0,
        expiresIn: Math.floor(expiresIn),
        lastActivity: new Date()
      });
    };

    updateExpiry();
    const interval = setInterval(updateExpiry, 1000);

    // Track user activity
    const activityEvents = ['mousedown', 'keydown', 'scroll', 'touchstart'];
    const handleActivity = () => {
      setSessionInfo(prev => ({
        ...prev,
        lastActivity: new Date()
      }));
    };

    activityEvents.forEach(event => {
      document.addEventListener(event, handleActivity);
    });

    return () => {
      clearInterval(interval);
      activityEvents.forEach(event => {
        document.removeEventListener(event, handleActivity);
      });
    };
  }, [keycloak]);

  return sessionInfo;
}
```

**Session Warning Component**:

```javascript
// SessionWarning.js
import React, { useState, useEffect } from 'react';
import { useKeycloak } from '@react-keycloak/web';
import { useSessionMonitor } from './useSessionMonitor';

function SessionWarning() {
  const { keycloak } = useKeycloak();
  const sessionInfo = useSessionMonitor();
  const [showWarning, setShowWarning] = useState(false);

  useEffect(() => {
    // Show warning when 2 minutes left
    if (sessionInfo.expiresIn > 0 && sessionInfo.expiresIn <= 120) {
      setShowWarning(true);
    } else {
      setShowWarning(false);
    }
  }, [sessionInfo.expiresIn]);

  const handleExtendSession = async () => {
    try {
      await keycloak.updateToken(-1); // Force refresh
      setShowWarning(false);
    } catch (error) {
      console.error('Failed to extend session:', error);
      keycloak.login();
    }
  };

  if (!showWarning) {
    return null;
  }

  const minutes = Math.floor(sessionInfo.expiresIn / 60);
  const seconds = sessionInfo.expiresIn % 60;

  return (
    <div className="session-warning-modal">
      <div className="modal-content">
        <h3>⚠️ Session Expiring</h3>
        <p>
          Your session will expire in <strong>{minutes}:{seconds.toString().padStart(2, '0')}</strong>
        </p>
        <p>Do you want to stay logged in?</p>
        <div className="actions">
          <button onClick={handleExtendSession} className="btn-primary">
            Stay Logged In
          </button>
          <button onClick={() => keycloak.logout()} className="btn-secondary">
            Logout
          </button>
        </div>
      </div>
    </div>
  );
}

export default SessionWarning;
```

### Offline Access and Refresh Tokens

**Requesting Offline Access**:

```javascript
// Request offline_access scope
const authUrl = `https://auth.example.com/realms/customer-portal/protocol/openid-connect/auth?` +
  `client_id=web-app&` +
  `response_type=code&` +
  `scope=openid profile email offline_access&` +
  `redirect_uri=${encodeURIComponent('https://app.example.com/callback')}`;

window.location.href = authUrl;
```

**Using Offline Refresh Token**:

```javascript
// Refresh token persists even after browser close
async function refreshWithOfflineToken() {
  const offlineRefreshToken = localStorage.getItem('offline_refresh_token');

  const response = await fetch(
    'https://auth.example.com/realms/customer-portal/protocol/openid-connect/token',
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        refresh_token: offlineRefreshToken,
        client_id: 'web-app',
        client_secret: 'client-secret'
      })
    }
  );

  const tokens = await response.json();
  
  // Store new tokens
  localStorage.setItem('access_token', tokens.access_token);
  localStorage.setItem('offline_refresh_token', tokens.refresh_token);

  return tokens;
}
```

**Mobile App Pattern**:

```javascript
// React Native - Use offline token for persistent login
import AsyncStorage from '@react-native-async-storage/async-storage';

async function initializeAuth() {
  // Try to restore session on app launch
  const offlineToken = await AsyncStorage.getItem('offline_refresh_token');

  if (offlineToken) {
    try {
      // Attempt to refresh
      const tokens = await refreshWithOfflineToken(offlineToken);
      
      // Success - user is logged in
      return tokens;
    } catch (error) {
      // Token expired or invalid - need to login
      await AsyncStorage.removeItem('offline_refresh_token');
      return null;
    }
  }

  return null;
}
```

### Session Persistence Across Devices

**Scenario**: User logs in on desktop, should be logged in on mobile.

**Implementation using Session Sync**:

```javascript
// Backend: Store session mapping
const sessionStore = new Map(); // Or use Redis

async function createSession(userId, deviceId, tokens) {
  const sessionData = {
    userId,
    deviceId,
    accessToken: tokens.access_token,
    refreshToken: tokens.refresh_token,
    createdAt: Date.now()
  };

  // Store with TTL
  await redis.setex(
    `session:${userId}:${deviceId}`,
    3600, // 1 hour TTL
    JSON.stringify(sessionData)
  );

  // Add to user's device list
  await redis.sadd(`user:${userId}:devices`, deviceId);
}

async function getUserSessions(userId) {
  const deviceIds = await redis.smembers(`user:${userId}:devices`);
  
  const sessions = await Promise.all(
    deviceIds.map(async deviceId => {
      const sessionData = await redis.get(`session:${userId}:${deviceId}`);
      return sessionData ? JSON.parse(sessionData) : null;
    })
  );

  return sessions.filter(s => s !== null);
}

async function terminateAllSessions(userId) {
  const deviceIds = await redis.smembers(`user:${userId}:devices`);
  
  // Delete all session keys
  await Promise.all(
    deviceIds.map(deviceId => 
      redis.del(`session:${userId}:${deviceId}`)
    )
  );

  // Clear device list
  await redis.del(`user:${userId}:devices`);
}
```

**Frontend: Device Management UI**:

```javascript
// DeviceManager.js
function DeviceManager() {
  const [devices, setDevices] = useState([]);

  useEffect(() => {
    fetchDevices();
  }, []);

  async function fetchDevices() {
    const response = await fetch('/api/user/sessions', {
      headers: { 'Authorization': `Bearer ${accessToken}` }
    });
    const data = await response.json();
    setDevices(data.sessions);
  }

  async function terminateSession(deviceId) {
    await fetch(`/api/user/sessions/${deviceId}`, {
      method: 'DELETE',
      headers: { 'Authorization': `Bearer ${accessToken}` }
    });
    fetchDevices();
  }

  async function terminateAll() {
    await fetch('/api/user/sessions', {
      method: 'DELETE',
      headers: { 'Authorization': `Bearer ${accessToken}` }
    });
    fetchDevices();
  }

  return (
    <div className="device-manager">
      <h2>Active Sessions</h2>
      
      {devices.map(device => (
        <div key={device.deviceId} className="device-item">
          <div className="device-info">
            <strong>{device.deviceName || 'Unknown Device'}</strong>
            <span className="device-ip">{device.ipAddress}</span>
            <span className="device-time">
              Last active: {new Date(device.lastActivity).toLocaleString()}
            </span>
          </div>
          <button onClick={() => terminateSession(device.deviceId)}>
            Terminate
          </button>
        </div>
      ))}

      <button onClick={terminateAll} className="btn-danger">
        Terminate All Other Sessions
      </button>
    </div>
  );
}
```

***

## 14. Events and Logging

Keycloak's event system provides comprehensive audit trails and monitoring capabilities.[1]

### Understanding Event Types

**Login Events** (User activities):
- LOGIN: Successful login
- LOGIN_ERROR: Failed login attempt
- LOGOUT: User logout
- REGISTER: New user registration
- VERIFY_EMAIL: Email verification
- UPDATE_PASSWORD: Password change
- UPDATE_PROFILE: Profile update
- REFRESH_TOKEN: Token refresh
- CODE_TO_TOKEN: Authorization code exchange
- INTROSPECT_TOKEN: Token introspection

**Admin Events** (Administrative actions):
- CREATE: Resource created
- UPDATE: Resource updated
- DELETE: Resource deleted
- ACTION: Custom action performed

### Configuring Event Logging

**Navigate to**: Realm Settings → Events

#### Login Events Configuration

```text
Event Config:
┌─────────────────────────────────────────────────┐
│ Event Listeners:                                │
│   ☑ jboss-logging                               │
│   ☐ email                                       │
│   ☐ openfga-events-publisher                    │
│                                                 │
│ Save Events: ☑ ON                               │
│   (Store events in database)                    │
│                                                 │
│ Expiration: 7 days                          [▼]│
│   Options: 1 hour, 1 day, 7 days, 30 days,     │
│            90 days, 365 days                    │
│   (Auto-delete old events)                      │
│                                                 │
│ Saved Types:                                    │
│   ☑ Login                                       │
│   ☑ Login Error                                 │
│   ☑ Logout                                      │
│   ☑ Register                                    │
│   ☑ Verify Email                                │
│   ☑ Update Password                             │
│   ☑ Update Profile                              │
│   ☑ Code to Token                               │
│   ☑ Refresh Token                               │
│   ☐ ... (select as needed)                      │
│                                                 │
│ Clear Events: [Clear login events]              │
└─────────────────────────────────────────────────┘
```

#### Admin Events Configuration

```text
Admin Event Config:
┌─────────────────────────────────────────────────┐
│ Save Events: ☑ ON                               │
│                                                 │
│ Include Representation: ☑ ON                    │
│   (Include full resource data in events)        │
│   Warning: May contain sensitive information    │
│                                                 │
│ Clear Events: [Clear admin events]              │
└─────────────────────────────────────────────────┘
```

### Viewing Events

**Navigate to**: Realm Settings → Events

**Login Events Tab**:

```text
Login Events:
┌────────────────────────────────────────────────────────────────┐
│ Search:                                                        │
│ From: [2025-10-23] To: [2025-10-23] Client: [All ▼]          │
│ User: [Search user...] IP: [192.168.1.100]                   │
│                                                                │
│ Type                Time               User          IP        │
├────────────────────────────────────────────────────────────────┤
│ LOGIN               01:30:15          alice         192.1.1.1 │
│ REFRESH_TOKEN       01:35:22          alice         192.1.1.1 │
│ LOGIN_ERROR         01:28:03          unknown       203.0.1.5 │
│ UPDATE_PASSWORD     01:25:47          bob           10.0.0.50 │
│ LOGOUT              01:20:11          charlie       172.16.1.1│
│ REGISTER            01:15:03          diana         192.1.1.50│
│                                                                │
│ [< Previous] [Next >]                                         │
└────────────────────────────────────────────────────────────────┘
```

**Click event for details**:

```text
Event Details:
┌─────────────────────────────────────────────────┐
│ Type: LOGIN                                     │
│ Time: 2025-10-23 01:30:15                       │
│ Realm: customer-portal                          │
│ Client: web-app                                 │
│ User: alice@company.com (user-uuid-123)         │
│ IP Address: 192.168.1.100                       │
│ Session ID: session-abc123                      │
│                                                 │
│ Details:                                        │
│   auth_method: openid-connect                   │
│   auth_type: code                               │
│   redirect_uri: https://app.example.com/callback│
│   consent: no_consent_required                  │
│   code_id: code-xyz789                          │
│   username: alice@company.com                   │
│   response_type: code                           │
│   identity_provider: google                     │
│   identity_provider_identity: alice@gmail.com   │
└─────────────────────────────────────────────────┘
```

### Querying Events via REST API

**Get Login Events**:

```bash
# Get recent login events
curl -X GET "http://localhost:8080/admin/realms/customer-portal/events?type=LOGIN&max=100" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

**Response**:
```json
[
  {
    "time": 1729642815000,
    "type": "LOGIN",
    "realmId": "customer-portal",
    "clientId": "web-app",
    "userId": "user-uuid-123",
    "ipAddress": "192.168.1.100",
    "details": {
      "auth_method": "openid-connect",
      "auth_type": "code",
      "redirect_uri": "https://app.example.com/callback",
      "consent": "no_consent_required",
      "code_id": "code-xyz789",
      "username": "alice@company.com"
    }
  }
]
```

**Filter Events**:

```bash
# Filter by user
curl -X GET "http://localhost:8080/admin/realms/customer-portal/events?user=${USER_ID}" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"

# Filter by date range
curl -X GET "http://localhost:8080/admin/realms/customer-portal/events?dateFrom=2025-10-23T00:00:00Z&dateTo=2025-10-23T23:59:59Z" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"

# Filter by IP address
curl -X GET "http://localhost:8080/admin/realms/customer-portal/events?ipAddress=192.168.1.100" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"

# Multiple filters
curl -X GET "http://localhost:8080/admin/realms/customer-portal/events?type=LOGIN_ERROR&dateFrom=2025-10-23T00:00:00Z&max=50" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

**Get Admin Events**:

```bash
curl -X GET "http://localhost:8080/admin/realms/customer-portal/admin-events?max=100" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

**Response**:
```json
[
  {
    "time": 1729642900000,
    "realmId": "customer-portal",
    "authDetails": {
      "realmId": "master",
      "clientId": "security-admin-console",
      "userId": "admin-uuid",
      "ipAddress": "10.0.0.1"
    },
    "operationType": "UPDATE",
    "resourceType": "USER",
    "resourcePath": "users/user-uuid-123",
    "representation": "{\"id\":\"user-uuid-123\",\"username\":\"alice@company.com\",...}",
    "error": null
  }
]
```

### Custom Event Listeners

Create custom event listeners to integrate with external systems.

**Event Listener SPI**:

```java
// CustomEventListenerProvider.java
package com.company.keycloak.events;

import org.keycloak.events.Event;
import org.keycloak.events.EventListenerProvider;
import org.keycloak.events.EventType;
import org.keycloak.events.admin.AdminEvent;
import org.keycloak.events.admin.OperationType;

public class CustomEventListenerProvider implements EventListenerProvider {

    @Override
    public void onEvent(Event event) {
        // Handle user events
        if (event.getType() == EventType.LOGIN) {
            handleLogin(event);
        } else if (event.getType() == EventType.LOGIN_ERROR) {
            handleLoginError(event);
        } else if (event.getType() == EventType.REGISTER) {
            handleRegistration(event);
        }
    }

    @Override
    public void onEvent(AdminEvent adminEvent, boolean includeRepresentation) {
        // Handle admin events
        if (adminEvent.getOperationType() == OperationType.CREATE) {
            handleResourceCreation(adminEvent);
        } else if (adminEvent.getOperationType() == OperationType.DELETE) {
            handleResourceDeletion(adminEvent);
        }
    }

    private void handleLogin(Event event) {
        String userId = event.getUserId();
        String ipAddress = event.getIpAddress();
        String clientId = event.getClientId();

        // Send to analytics
        sendToAnalytics("user_login", Map.of(
            "user_id", userId,
            "ip_address", ipAddress,
            "client_id", clientId,
            "timestamp", event.getTime()
        ));

        // Check for suspicious activity
        if (isSuspiciousLogin(userId, ipAddress)) {
            sendSecurityAlert(userId, ipAddress);
        }

        // Update last login timestamp in database
        updateLastLogin(userId, event.getTime());
    }

    private void handleLoginError(Event event) {
        String username = event.getDetails().get("username");
        String ipAddress = event.getIpAddress();
        String error = event.getError();

        // Increment failed login counter
        incrementFailedLogins(username, ipAddress);

        // Check if IP should be blocked
        if (shouldBlockIP(ipAddress)) {
            blockIP(ipAddress);
            sendAlert("IP blocked due to multiple failed logins: " + ipAddress);
        }

        // Log to security monitoring system
        logSecurityEvent("failed_login", Map.of(
            "username", username,
            "ip_address", ipAddress,
            "error", error,
            "timestamp", event.getTime()
        ));
    }

    private void handleRegistration(Event event) {
        String userId = event.getUserId();
        String email = event.getDetails().get("email");

        // Send welcome email
        sendWelcomeEmail(email);

        // Track registration source
        String source = event.getDetails().get("registration_source");
        trackConversion("registration", source);

        // Initialize user in external systems
        initializeUserInCRM(userId, email);
    }

    private boolean isSuspiciousLogin(String userId, String ipAddress) {
        // Check for unusual location
        String lastKnownLocation = getLastKnownLocation(userId);
        String currentLocation = getLocationFromIP(ipAddress);
        
        if (!lastKnownLocation.equals(currentLocation)) {
            return true;
        }

        // Check for unusual time
        int hour = LocalTime.now().getHour();
        if (hour >= 2 && hour < 6) {
            return true; // Late night login
        }

        return false;
    }

    @Override
    public void close() {
        // Cleanup if needed
    }
}
```

**Event Listener Factory**:

```java
// CustomEventListenerProviderFactory.java
package com.company.keycloak.events;

import org.keycloak.Config;
import org.keycloak.events.EventListenerProvider;
import org.keycloak.events.EventListenerProviderFactory;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.KeycloakSessionFactory;

public class CustomEventListenerProviderFactory implements EventListenerProviderFactory {

    @Override
    public EventListenerProvider create(KeycloakSession session) {
        return new CustomEventListenerProvider();
    }

    @Override
    public void init(Config.Scope config) {
        // Initialize with configuration
    }

    @Override
    public void postInit(KeycloakSessionFactory factory) {
        // Post-initialization
    }

    @Override
    public void close() {
        // Cleanup
    }

    @Override
    public String getId() {
        return "custom-event-listener";
    }
}
```

**Register SPI** in `META-INF/services/org.keycloak.events.EventListenerProviderFactory`:

```text
com.company.keycloak.events.CustomEventListenerProviderFactory
```

**Deploy and Enable**:

```bash
# Build and deploy
mvn clean package
cp target/custom-event-listener.jar /opt/keycloak/providers/
/opt/keycloak/bin/kc.sh build
systemctl restart keycloak

# Enable in realm
Realm Settings → Events → Event listeners
Add: custom-event-listener
```

### Event-Driven Workflows

**Webhook Integration**:

```java
// WebhookEventListener.java
public class WebhookEventListener implements EventListenerProvider {
    
    private static final String WEBHOOK_URL = System.getenv("WEBHOOK_URL");
    private HttpClient httpClient;

    public WebhookEventListener() {
        this.httpClient = HttpClient.newHttpClient();
    }

    @Override
    public void onEvent(Event event) {
        // Send event to webhook
        sendWebhook(eventToJSON(event));
    }

    private void sendWebhook(String payload) {
        try {
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(WEBHOOK_URL))
                .header("Content-Type", "application/json")
                .header("X-Keycloak-Event", "true")
                .POST(HttpRequest.BodyPublishers.ofString(payload))
                .build();

            httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString())
                .thenApply(HttpResponse::statusCode)
                .thenAccept(statusCode -> {
                    if (statusCode != 200 && statusCode != 204) {
                        System.err.println("Webhook failed with status: " + statusCode);
                    }
                });
        } catch (Exception e) {
            System.err.println("Failed to send webhook: " + e.getMessage());
        }
    }

    private String eventToJSON(Event event) {
        JSONObject json = new JSONObject();
        json.put("type", event.getType().name());
        json.put("time", event.getTime());
        json.put("realmId", event.getRealmId());
        json.put("clientId", event.getClientId());
        json.put("userId", event.getUserId());
        json.put("ipAddress", event.getIpAddress());
        json.put("details", new JSONObject(event.getDetails()));
        return json.toString();
    }
}
```

**Kafka Integration**:

```java
// KafkaEventListener.java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

public class KafkaEventListener implements EventListenerProvider {
    
    private KafkaProducer<String, String> producer;
    private static final String TOPIC = "keycloak-events";

    public KafkaEventListener() {
        Properties props = new Properties();
        props.put("bootstrap.servers", System.getenv("KAFKA_BROKERS"));
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        this.producer = new KafkaProducer<>(props);
    }

    @Override
    public void onEvent(Event event) {
        String key = event.getUserId();
        String value = eventToJSON(event);
        
        ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC, key, value);
        producer.send(record, (metadata, exception) -> {
            if (exception != null) {
                System.err.println("Failed to send to Kafka: " + exception.getMessage());
            }
        });
    }

    @Override
    public void close() {
        if (producer != null) {
            producer.close();
        }
    }
}
```

### Event-Based Analytics Dashboard

**Real-World Example**: Track user engagement

```javascript
// analytics-service.js
const { createClient } = require('@supabase/supabase-js');

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_KEY
);

async function processKeycloakEvent(event) {
  switch (event.type) {
    case 'LOGIN':
      await trackLogin(event);
      break;
    case 'REGISTER':
      await trackRegistration(event);
      break;
    case 'LOGOUT':
      await trackLogout(event);
      break;
    case 'LOGIN_ERROR':
      await trackFailedLogin(event);
      break;
  }
}

async function trackLogin(event) {
  await supabase.from('login_events').insert({
    user_id: event.userId,
    timestamp: new Date(event.time),
    ip_address: event.ipAddress,
    client_id: event.clientId,
    identity_provider: event.details.identity_provider || 'local',
    device_type: detectDeviceType(event.details)
  });

  // Update user last login
  await supabase.from('users').update({
    last_login_at: new Date(event.time),
    login_count: supabase.sql`login_count + 1`
  }).eq('keycloak_id', event.userId);
}

async function trackRegistration(event) {
  await supabase.from('registration_events').insert({
    user_id: event.userId,
    timestamp: new Date(event.time),
    email: event.details.email,
    registration_source: event.details.utm_source || 'direct',
    identity_provider: event.details.identity_provider || 'local'
  });
}

async function getAnalytics(dateFrom, dateTo) {
  // Daily active users
  const { data: dau } = await supabase
    .from('login_events')
    .select('user_id')
    .gte('timestamp', dateFrom)
    .lte('timestamp', dateTo);

  const uniqueDAU = new Set(dau.map(d => d.user_id)).size;

  // New registrations
  const { count: newUsers } = await supabase
    .from('registration_events')
    .select('*', { count: 'exact', head: true })
    .gte('timestamp', dateFrom)
    .lte('timestamp', dateTo);

  // Failed login attempts
  const { count: failedLogins } = await supabase
    .from('failed_login_events')
    .select('*', { count: 'exact', head: true })
    .gte('timestamp', dateFrom)
    .lte('timestamp', dateTo);

  // Login by identity provider
  const { data: idpBreakdown } = await supabase
    .from('login_events')
    .select('identity_provider, count')
    .gte('timestamp', dateFrom)
    .lte('timestamp', dateTo)
    .group('identity_provider');

  return {
    daily_active_users: uniqueDAU,
    new_registrations: newUsers,
    failed_logins: failedLogins,
    identity_provider_breakdown: idpBreakdown
  };
}
```

***

## 15. Security Best Practices

### Production Security Hardening

#### 1. Admin Console Security

**Restrict Admin Access by IP**:

Using Nginx reverse proxy:

```nginx
# /etc/nginx/sites-available/keycloak
upstream keycloak {
    server localhost:8080;
}

server {
    listen 443 ssl http2;
    server_name auth.example.com;

    ssl_certificate /etc/letsencrypt/live/auth.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/auth.example.com/privkey.pem;

    # Admin console - restricted IPs only
    location ~ ^/(admin|realms/master) {
        # Allow office IPs
        allow 203.0.113.0/24;
        allow 198.51.100.50;
        deny all;

        proxy_pass http://keycloak;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Public endpoints
    location / {
        proxy_pass http://keycloak;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Rate limiting
        limit_req zone=general burst=20 nodelay;
    }
}

# Rate limiting zones
limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;
```

**VPN-Only Admin Access**:

```bash
# Using WireGuard VPN
# Only allow admin access from VPN network

# iptables rules
iptables -A INPUT -p tcp --dport 8080 -s 10.8.0.0/24 -j ACCEPT  # VPN subnet
iptables -A INPUT -p tcp --dport 8080 -j DROP  # Block all others
```

**Separate Admin Realm**:

```text
Best Practice: Don't use master realm for application admins

Create dedicated admin realm:
├── master (Keycloak super-admins only)
└── admin-realm (Application administrators)
    ├── Limited permissions
    ├── MFA required
    └── Separate credentials
```

**Admin Account Security**:

```text
1. Strong Password Policy:
   Minimum 16 characters
   Uppercase + lowercase + numbers + special chars
   Not recently used (last 10 passwords)
   Password expiry: 90 days

2. Mandatory MFA:
   OTP or WebAuthn required
   Backup codes generated
   
3. Account Monitoring:
   Alert on admin logins
   Track all admin actions
   Session timeout: 15 minutes

4. Principle of Least Privilege:
   Grant minimal required permissions
   Use role-based delegation
   Regular access reviews
```

#### 2. Password Policies

**Navigate to**: Realm Settings → Authentication → Policies → Password policy

**Comprehensive Password Policy**:

```text
┌─────────────────────────────────────────────────┐
│ Add policy:                                     │
│                                                 │
│ Minimum Length: 12                              │
│   At least 12 characters                        │
│                                                 │
│ Uppercase Characters: 1                         │
│   At least 1 uppercase letter                   │
│                                                 │
│ Lowercase Characters: 1                         │
│   At least 1 lowercase letter                   │
│                                                 │
│ Digits: 1                                       │
│   At least 1 number                             │
│                                                 │
│ Special Characters: 1                           │
│   At least 1 special character (!@#$%^&*)       │
│                                                 │
│ Not Username: ON                                │
│   Password cannot contain username              │
│                                                 │
│ Not Email: ON                                   │
│   Password cannot contain email                 │
│                                                 │
│ Regular Expression:                             │
│   ^(?!.*(password|123456|qwerty)).*$            │
│   Block common weak passwords                   │
│                                                 │
│ Expire Password: 90 days                        │
│   Force password change after 90 days           │
│                                                 │
│ Not Recently Used: 5                            │
│   Cannot reuse last 5 passwords                 │
│                                                 │
│ Password Blacklist:                             │
│   /opt/keycloak/data/password-blacklist.txt     │
│   Check against known leaked passwords          │
└─────────────────────────────────────────────────┘
```

**Password Blacklist** (Common breached passwords):

```bash
# Download Have I Been Pwned password list
wget https://downloads.pwnedpasswords.com/passwords/pwned-passwords-sha1-ordered-by-hash-v8.7z

# Extract top 10,000 most common
7z x pwned-passwords-sha1-ordered-by-hash-v8.7z
head -n 10000 pwned-passwords-sha1-ordered-by-hash-v8.txt > /opt/keycloak/data/password-blacklist.txt

# Configure in Keycloak
Realm Settings → Authentication → Password Policy
Add policy: Password Blacklist
Path: /opt/keycloak/data/password-blacklist.txt
```

**Custom Password Validator**:

```java
// StrongPasswordPolicyProvider.java
package com.company.keycloak.policy;

import org.keycloak.models.RealmModel;
import org.keycloak.models.UserModel;
import org.keycloak.policy.PasswordPolicyProvider;
import org.keycloak.policy.PolicyError;

public class StrongPasswordPolicyProvider implements PasswordPolicyProvider {

    @Override
    public PolicyError validate(RealmModel realm, UserModel user, String password) {
        // Check entropy
        if (calculateEntropy(password) < 50) {
            return new PolicyError("Password entropy too low. Use more varied characters.");
        }

        // Check for keyboard patterns (qwerty, asdfgh, etc.)
        if (containsKeyboardPattern(password)) {
            return new PolicyError("Password contains keyboard pattern.");
        }

        // Check for repeated characters (aaa, 111, etc.)
        if (containsRepeatedCharacters(password, 3)) {
            return new PolicyError("Password contains too many repeated characters.");
        }

        // Check against user attributes
        if (containsUserInfo(password, user)) {
            return new PolicyError("Password contains personal information.");
        }

        return null; // Password is valid
    }

    private double calculateEntropy(String password) {
        int poolSize = 0;
        if (password.matches(".*[a-z].*")) poolSize += 26;
        if (password.matches(".*[A-Z].*")) poolSize += 26;
        if (password.matches(".*[0-9].*")) poolSize += 10;
        if (password.matches(".*[^a-zA-Z0-9].*")) poolSize += 32;
        
        return password.length() * (Math.log(poolSize) / Math.log(2));
    }

    private boolean containsKeyboardPattern(String password) {
        String[] patterns = {
            "qwerty", "asdfgh", "zxcvbn",
            "qwertz", "azerty", "12345"
        };
        
        String lower = password.toLowerCase();
        for (String pattern : patterns) {
            if (lower.contains(pattern)) {
                return true;
            }
        }
        return false;
    }

    private boolean containsRepeatedCharacters(String password, int maxRepeat) {
        for (int i = 0; i < password.length() - maxRepeat; i++) {
            char c = password.charAt(i);
            boolean allSame = true;
            for (int j = 1; j <= maxRepeat; j++) {
                if (password.charAt(i + j) != c) {
                    allSame = false;
                    break;
                }
            }
            if (allSame) return true;
        }
        return false;
    }

    private boolean containsUserInfo(String password, UserModel user) {
        String lower = password.toLowerCase();
        
        if (user.getUsername() != null && 
            lower.contains(user.getUsername().toLowerCase())) {
            return true;
        }
        
        if (user.getEmail() != null) {
            String emailPrefix = user.getEmail().split("@")[0].toLowerCase();
            if (lower.contains(emailPrefix)) {
                return true;
            }
        }
        
        if (user.getFirstName() != null && 
            lower.contains(user.getFirstName().toLowerCase())) {
            return true;
        }
        
        if (user.getLastName() != null && 
            lower.contains(user.getLastName().toLowerCase())) {
            return true;
        }
        
        return false;
    }

    @Override
    public Object parseConfig(String value) {
        return value;
    }

    @Override
    public void close() {
    }
}
```

#### 3. Brute Force Detection

**Navigate to**: Realm Settings → Security defenses → Brute force detection

```text
┌─────────────────────────────────────────────────┐
│ Enabled: ☑ ON                                   │
│                                                 │
│ Permanent Lockout: ☐ OFF                        │
│   (Temporary lockout recommended)               │
│                                                 │
│ Max Login Failures: 5                           │
│   Attempts before lockout                       │
│                                                 │
│ Wait Increment: 60 seconds                      │
│   Added wait time per failed attempt            │
│                                                 │
│ Quick Login Check Milliseconds: 1000            │
│   Minimum time between attempts                 │
│                                                 │
│ Minimum Quick Login Wait: 60 seconds            │
│   Wait time for quick successive attempts       │
│                                                 │
│ Max Wait: 900 seconds (15 minutes)              │
│   Maximum lockout duration                      │
│                                                 │
│ Failure Reset Time: 43200 seconds (12 hours)    │
│   Time until counter resets                     │
│└─────────────────────────────────────────────────┘
```

**Real-World Example**:

```text
Scenario: Attacker tries to brute force account

Attempt 1: Failed → Wait 60 seconds
Attempt 2: Failed → Wait 120 seconds (60 + 60)
Attempt 3: Failed → Wait 180 seconds (60 + 60 + 60)
Attempt 4: Failed → Wait 240 seconds
Attempt 5: Failed → Account locked for 300 seconds (5 minutes)
Attempt 6+: Account locked for 900 seconds (15 minutes - max wait)

After 12 hours of no attempts: Counter resets
```

**Advanced Protection with Fail2Ban**:

```bash
# Install Fail2Ban
apt-get install fail2ban

# Create Keycloak filter
cat > /etc/fail2ban/filter.d/keycloak.conf <<'EOF'
[Definition]
failregex = .*type=LOGIN_ERROR.*username=<F-USER>[^,]+</F-USER>.*clientAddress=<HOST>
ignoreregex =
EOF

# Create jail configuration
cat > /etc/fail2ban/jail.d/keycloak.conf <<'EOF'
[keycloak]
enabled = true
port = http,https
filter = keycloak
logpath = /opt/keycloak/data/log/keycloak.log
maxretry = 5
findtime = 600
bantime = 3600
action = iptables-multiport[name=keycloak, port="http,https", protocol=tcp]
EOF

# Restart Fail2Ban
systemctl restart fail2ban

# Check banned IPs
fail2ban-client status keycloak
```

**IP Blacklisting API**:

```javascript
// Backend service to manage IP blacklist
const express = require('express');
const redis = require('redis');
const app = express();

const redisClient = redis.createClient();

// Track failed login attempts
async function trackFailedLogin(ipAddress, username) {
    const key = `failed_logins:${ipAddress}`;
    const count = await redisClient.incr(key);
    await redisClient.expire(key, 3600); // 1 hour TTL

    if (count >= 10) {
        // Blacklist IP
        await redisClient.setex(`blacklist:${ipAddress}`, 86400, '1'); // 24 hours
        
        // Alert security team
        await sendAlert({
            type: 'IP_BLACKLISTED',
            ip: ipAddress,
            reason: 'Multiple failed login attempts',
            count: count
        });
    }
}

// Middleware to check IP blacklist
async function checkBlacklist(req, res, next) {
    const ipAddress = req.ip;
    const isBlacklisted = await redisClient.get(`blacklist:${ipAddress}`);
    
    if (isBlacklisted) {
        return res.status(403).json({
            error: 'Your IP address has been temporarily blocked due to suspicious activity'
        });
    }
    
    next();
}

app.use(checkBlacklist);
```

#### 4. HTTPS/TLS Configuration

**Production TLS Setup**:

```bash
# Install Certbot
apt-get install certbot

# Obtain Let's Encrypt certificate
certbot certonly --standalone -d auth.example.com

# Certificates location:
# /etc/letsencrypt/live/auth.example.com/fullchain.pem
# /etc/letsencrypt/live/auth.example.com/privkey.pem
```

**Keycloak Configuration** (`conf/keycloak.conf`):

```properties
# Hostname
hostname=auth.example.com
hostname-strict=true
hostname-strict-https=true

# HTTPS
https-certificate-file=/etc/letsencrypt/live/auth.example.com/fullchain.pem
https-certificate-key-file=/etc/letsencrypt/live/auth.example.com/privkey.pem
https-port=8443

# Disable HTTP in production
http-enabled=false

# TLS configuration
https-protocols=TLSv1.3,TLSv1.2
https-cipher-suites=TLS_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384,TLS_CHACHA20_POLY1305_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384

# Security headers
https-client-auth=none
```

**Nginx TLS Configuration** (Reverse Proxy):

```nginx
server {
    listen 443 ssl http2;
    server_name auth.example.com;

    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/auth.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/auth.example.com/privkey.pem;

    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self'" always;

    # Proxy to Keycloak
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name auth.example.com;
    return 301 https://$server_name$request_uri;
}
```

**Test SSL Configuration**:

```bash
# Test SSL with ssllabs.com
curl -s "https://api.ssllabs.com/api/v3/analyze?host=auth.example.com" | jq

# Check TLS version
openssl s_client -connect auth.example.com:443 -tls1_2
openssl s_client -connect auth.example.com:443 -tls1_3

# Check certificate
openssl s_client -connect auth.example.com:443 -showcerts
```

#### 5. Token Security

**Access Token Configuration**:

```properties
# Short-lived access tokens
access-token-lifespan=300  # 5 minutes

# No token reuse
refresh-token-max-reuse=0

# Client-bound tokens
client-auth-method=client-secret-post

# Token revocation
revoke-refresh-token=true
refresh-token-max-reuse=0
```

**Token Validation Best Practices**:

```javascript
// Backend token validation
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const client = jwksClient({
    jwksUri: 'https://auth.example.com/realms/customer-portal/protocol/openid-connect/certs',
    cache: true,
    cacheMaxEntries: 5,
    cacheMaxAge: 600000 // 10 minutes
});

function getKey(header, callback) {
    client.getSigningKey(header.kid, (err, key) => {
        const signingKey = key.publicKey || key.rsaPublicKey;
        callback(null, signingKey);
    });
}

async function validateToken(token) {
    return new Promise((resolve, reject) => {
        jwt.verify(token, getKey, {
            audience: 'web-app',
            issuer: 'https://auth.example.com/realms/customer-portal',
            algorithms: ['RS256']
        }, (err, decoded) => {
            if (err) {
                reject(err);
            } else {
                // Additional validation
                const now = Math.floor(Date.now() / 1000);
                
                // Check expiration
                if (decoded.exp < now) {
                    reject(new Error('Token expired'));
                    return;
                }
                
                // Check not before
                if (decoded.nbf && decoded.nbf > now) {
                    reject(new Error('Token not yet valid'));
                    return;
                }
                
                // Check issued at (reject tokens older than 1 hour)
                if (decoded.iat < now - 3600) {
                    reject(new Error('Token too old'));
                    return;
                }
                
                resolve(decoded);
            }
        });
    });
}

// Express middleware
async function requireAuth(req, res, next) {
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'No token provided' });
    }
    
    const token = authHeader.substring(7);
    
    try {
        const decoded = await validateToken(token);
        req.user = decoded;
        next();
    } catch (error) {
        return res.status(401).json({ error: 'Invalid token', details: error.message });
    }
}
```

**Token Storage Security**:

```javascript
// Frontend - Secure token storage

// ❌ BAD: localStorage (vulnerable to XSS)
localStorage.setItem('access_token', token);

// ✅ GOOD: Memory only (for SPAs)
class TokenStore {
    constructor() {
        this.accessToken = null;
        this.refreshToken = null;
        this.expiresAt = null;
    }

    setTokens(access, refresh, expiresIn) {
        this.accessToken = access;
        this.refreshToken = refresh;
        this.expiresAt = Date.now() + (expiresIn * 1000);
    }

    getAccessToken() {
        if (this.expiresAt && Date.now() >= this.expiresAt) {
            return null; // Expired
        }
        return this.accessToken;
    }

    clear() {
        this.accessToken = null;
        this.refreshToken = null;
        this.expiresAt = null;
    }
}

// ✅ BETTER: HttpOnly cookies (for traditional web apps)
// Server sets cookie:
res.cookie('access_token', token, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 300000 // 5 minutes
});
```

#### 6. Content Security Policy

**Configure CSP Headers**:

```nginx
# Nginx
add_header Content-Security-Policy "
    default-src 'self';
    script-src 'self' 'unsafe-inline' 'unsafe-eval';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    font-src 'self' data:;
    connect-src 'self' https://auth.example.com;
    frame-ancestors 'none';
    base-uri 'self';
    form-action 'self';
" always;
```

**Custom Theme with CSP**:

```html
<!-- login.ftl -->
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Security-Policy" content="
        default-src 'self';
        script-src 'self' 'sha256-abc123...';
        style-src 'self' 'sha256-def456...';
        img-src 'self' data: https:;
        font-src 'self';
        connect-src 'self';
        frame-ancestors 'none';
    ">
    <title>Login</title>
</head>
<body>
    <!-- Login form -->
</body>
</html>
```

#### 7. Database Security

**Encrypt Database Connection**:

```properties
# conf/keycloak.conf
db=postgres
db-url=jdbc:postgresql://localhost:5432/keycloak?ssl=true&sslmode=require
db-username=keycloak
db-password=${vault.db_password}
```

**Use Vault for Secrets**:

```bash
# Install HashiCorp Vault
wget https://releases.hashicorp.com/vault/1.15.0/vault_1.15.0_linux_amd64.zip
unzip vault_1.15.0_linux_amd64.zip
mv vault /usr/local/bin/

# Start Vault
vault server -dev

# Store database password
vault kv put secret/keycloak db_password="SecurePassword123!"

# Configure Keycloak to use Vault
# conf/keycloak.conf
vault-url=http://localhost:8200
vault-token=${VAULT_TOKEN}
```

**Database User Permissions** (Least Privilege):

```sql
-- PostgreSQL

-- Create dedicated user
CREATE USER keycloak WITH PASSWORD 'SecurePassword123!';

-- Create database
CREATE DATABASE keycloak OWNER keycloak;

-- Grant minimal permissions
GRANT CONNECT ON DATABASE keycloak TO keycloak;
GRANT USAGE ON SCHEMA public TO keycloak;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO keycloak;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO keycloak;

-- Revoke unnecessary permissions
REVOKE CREATE ON SCHEMA public FROM keycloak;
REVOKE ALL ON SCHEMA public FROM PUBLIC;

-- Enable row-level security (if needed)
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
```

#### 8. Audit Logging

**Comprehensive Audit Configuration**:

```properties
# conf/keycloak.conf
log-level=INFO
log-console-output=default
log-file=/var/log/keycloak/keycloak.log

# Enable detailed logging for security events
log-level-org.keycloak.events=DEBUG
log-level-org.keycloak.services.managers=DEBUG
```

**Centralized Logging** (ELK Stack):

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/keycloak/*.log
    json.keys_under_root: true
    json.add_error_key: true
    fields:
      service: keycloak
      environment: production

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "keycloak-logs-%{+yyyy.MM.dd}"

setup.kibana:
  host: "kibana:5601"
```

**SIEM Integration**:

```javascript
// Forward events to SIEM
const syslog = require('syslog-client');

const syslogClient = syslog.createClient('siem.example.com', {
    port: 514,
    facility: syslog.Facility.Local0,
    severity: syslog.Severity.Informational
});

function sendToSIEM(event) {
    const message = JSON.stringify({
        timestamp: new Date(event.time).toISOString(),
        type: event.type,
        user: event.userId,
        ip: event.ipAddress,
        client: event.clientId,
        details: event.details
    });

    syslogClient.log(message, {
        severity: getSeverity(event.type)
    });
}

function getSeverity(eventType) {
    const criticalEvents = ['LOGIN_ERROR', 'USER_DISABLED', 'REMOVE_TOTP'];
    return criticalEvents.includes(eventType) 
        ? syslog.Severity.Warning 
        : syslog.Severity.Informational;
}
```

#### 9. Regular Security Maintenance

**Security Checklist**:

```text
Daily:
☐ Monitor failed login attempts
☐ Check for unusual access patterns
☐ Review error logs
☐ Verify backup completion

Weekly:
☐ Review admin access logs
☐ Check for security updates
☐ Verify SSL certificate validity
☐ Review user access permissions

Monthly:
☐ Update Keycloak to latest version
☐ Review and rotate secrets
☐ Audit user accounts (disable inactive)
☐ Review client configurations
☐ Test disaster recovery procedures
☐ Penetration testing
☐ Review and update password policies

Quarterly:
☐ Security audit
☐ Access review (remove unnecessary permissions)
☐ Update threat model
☐ Review and test incident response plan
☐ Compliance audit (GDPR, SOC2, etc.)
```

**Automated Security Scanning**:

```bash
#!/bin/bash
# security-scan.sh

# Check for outdated packages
echo "Checking for security updates..."
apt-get update
apt-get --just-print upgrade | grep -i security

# Scan for vulnerabilities
echo "Scanning for vulnerabilities..."
trivy image quay.io/keycloak/keycloak:25.0.1

# Check SSL certificate expiry
echo "Checking SSL certificate..."
echo | openssl s_client -connect auth.example.com:443 2>/dev/null | \
openssl x509 -noout -dates

# Check for weak ciphers
echo "Testing TLS configuration..."
sslscan auth.example.com

# Check for exposed secrets
echo "Scanning for exposed secrets..."
trufflehog filesystem /opt/keycloak --only-verified

# Generate report
echo "Security scan completed: $(date)" >> /var/log/security-scans.log
```

**Automated Response to Threats**:

```javascript
// threat-response.js
const axios = require('axios');

async function detectAndRespond() {
    // Get recent failed logins
    const events = await getFailedLogins(Date.now() - 3600000); // Last hour

    // Group by IP
    const ipCounts = {};
    events.forEach(event => {
        ipCounts[event.ipAddress] = (ipCounts[event.ipAddress] || 0) + 1;
    });

    // Detect brute force attempts
    for (const [ip, count] of Object.entries(ipCounts)) {
        if (count > 20) {
            // Block IP
            await blockIP(ip);
            
            // Alert security team
            await sendAlert({
                type: 'BRUTE_FORCE_DETECTED',
                ip: ip,
                attempts: count,
                action: 'IP_BLOCKED'
            });
            
            // Log to SIEM
            await logToSIEM({
                severity: 'HIGH',
                event: 'Brute force attack detected',
                ip: ip,
                attempts: count
            });
        }
    }

    // Detect credential stuffing
    const uniqueUsers = new Set(events.map(e => e.details.username));
    if (uniqueUsers.size > 100) {
        await sendAlert({
            type: 'CREDENTIAL_STUFFING_DETECTED',
            uniqueAccounts: uniqueUsers.size,
            totalAttempts: events.length
        });
    }
}

// Run every 5 minutes
setInterval(detectAndRespond, 300000);
```

***

## 16. Monitoring and Performance

### Metrics and Health Checks

**Built-in Health Endpoints**:

```bash
# Liveness probe (is Keycloak running?)
curl http://localhost:8080/health/live

# Response: {"status": "UP"}

# Readiness probe (is Keycloak ready to serve requests?)
curl http://localhost:8080/health/ready

# Response: {"status": "UP", "checks": [...]}

# Started probe (has Keycloak finished startup?)
curl http://localhost:8080/health/started
```

**Enable Metrics**:

```properties
# conf/keycloak.conf
metrics-enabled=true
```

**Prometheus Metrics Endpoint**:

```bash
curl http://localhost:8080/metrics

# Sample output:
# jvm_memory_used_bytes{area="heap"} 524288000
# jvm_threads_current 45
# http_server_requests_seconds_count{method="POST",uri="/realms/{realm}/protocol/openid-connect/token"} 1247
# keycloak_logins_total 5423
# keycloak_failed_login_attempts 127
```

### Prometheus Configuration

**prometheus.yml**:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'keycloak'
    static_configs:
      - targets: ['keycloak:8080']
    metrics_path: '/metrics'
    scrape_interval: 30s
    
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
    
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

### Grafana Dashboards

**Import Keycloak Dashboard**:

```json
{
  "dashboard": {
    "title": "Keycloak Monitoring",
    "panels": [
      {
        "title": "Active Sessions",
        "targets": [{
          "expr": "keycloak_sessions_active"
        }],
        "type": "graph"
      },
      {
        "title": "Login Rate",
        "targets": [{
          "expr": "rate(keycloak_logins_total[5m])"
        }],
        "type": "graph"
      },
      {
        "title": "Failed Login Rate",
        "targets": [{
          "expr": "rate(keycloak_failed_login_attempts[5m])"
        }],
        "type": "graph"
      },
      {
        "title": "Token Generation Time",
        "targets": [{
          "expr": "histogram_quantile(0.95, rate(http_server_requests_seconds_bucket{uri=\"/realms/{realm}/protocol/openid-connect/token\"}[5m]))"
        }],
        "type": "graph"
      },
      {
        "title": "JVM Memory Usage",
        "targets": [{
          "expr": "jvm_memory_used_bytes{area=\"heap\"} / jvm_memory_max_bytes{area=\"heap\"} * 100"
        }],
        "type": "gauge"
      }
    ]
  }
}
```

### Alerting Rules

**alerting-rules.yml**:

```yaml
groups:
  - name: keycloak
    interval: 30s
    rules:
      - alert: KeycloakDown
        expr: up{job="keycloak"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Keycloak is down"
          description: "Keycloak has been down for more than 1 minute"

      - alert: HighFailedLoginRate
        expr: rate(keycloak_failed_login_attempts[5m]) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High failed login rate detected"
          description: "Failed login rate is {{ $value }} per second"

      - alert: HighMemoryUsage
        expr: (jvm_memory_used_bytes{area="heap"} / jvm_memory_max_bytes{area="heap"}) * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "JVM heap usage is {{ $value }}%"

      - alert: SlowTokenGeneration
        expr: histogram_quantile(0.95, rate(http_server_requests_seconds_bucket{uri="/realms/{realm}/protocol/openid-connect/token"}[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Slow token generation"
          description: "95th percentile token generation time is {{ $value }}s"

      - alert: DatabaseConnectionPoolExhausted
        expr: hikaricp_connections_active / hikaricp_connections_max > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Database connection pool nearly exhausted"
          description: "{{ $value }}% of database connections in use"
```

### Performance Tuning

**JVM Configuration**:

```bash
# Set environment variables
export JAVA_OPTS="-Xms2g -Xmx4g \
  -XX:MetaspaceSize=512m \
  -XX:MaxMetaspaceSize=1g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:ParallelGCThreads=8 \
  -XX:ConcGCThreads=2 \
  -XX:InitiatingHeapOccupancyPercent=70 \
  -XX:+DisableExplicitGC \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/keycloak/heap_dump.hprof \
  -XX:+UseStringDeduplication"
```

**Database Connection Pool**:

```properties
# conf/keycloak.conf
db-pool-initial-size=20
db-pool-min-size=10
db-pool-max-size=100
db-pool-max-lifetime=1800000  # 30 minutes
db-pool-idle-timeout=600000    # 10 minutes
```

**Cache Configuration**:

```properties
# conf/keycloak.conf
cache=ispn
cache-stack=kubernetes  # or tcp, udp for clustering

# Cache sizes
cache-realm-max-entries=10000
cache-user-max-entries=10000
cache-authorization-max-entries=10000

# Cache timeouts
cache-realm-max-age=3600000      # 1 hour
cache-user-max-age=3600000       # 1 hour
cache-authorization-max-age=300000  # 5 minutes
```

**HTTP Thread Pool**:

```properties
# conf/keycloak.conf
http-pool-max-threads=200
http-pool-min-threads=10
```

### Load Testing

**K6 Load Test Script**:

```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

const failureRate = new Rate('failed_logins');

export const options = {
    stages: [
        { duration: '2m', target: 100 },  // Ramp up to 100 users
        { duration: '5m', target: 100 },  // Stay at 100 users
        { duration: '2m', target: 200 },  // Ramp up to 200 users
        { duration: '5m', target: 200 },  // Stay at 200 users
        { duration: '2m', target: 0 },    // Ramp down to 0 users
    ],
    thresholds: {
        http_req_duration: ['p(95)<500'],  // 95% of requests must complete below 500ms
        failed_logins: ['rate<0.1'],       // Less than 10% failed logins
    },
};

const BASE_URL = 'https://auth.example.com';
const REALM = 'customer-portal';
const CLIENT_ID = 'web-app';
const CLIENT_SECRET = 'client-secret';

export default function() {
    // 1. Test login
    const loginRes = http.post(
        `${BASE_URL}/realms/${REALM}/protocol/openid-connect/token`,
        {
            grant_type: 'password',
            client_id: CLIENT_ID,
            client_secret: CLIENT_SECRET,
            username: `user${__VU}@example.com`,  // Virtual user email
            password: 'Password123!',
            scope: 'openid profile email'
        },
        {
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
        }
    );

    const loginSuccess = check(loginRes, {
        'login status is 200': (r) => r.status === 200,
        'received access token': (r) => r.json('access_token') !== undefined,
    });

    failureRate.add(!loginSuccess);

    if (loginSuccess) {
        const accessToken = loginRes.json('access_token');
        const refreshToken = loginRes.json('refresh_token');

        // 2. Test userinfo endpoint
        const userinfoRes = http.get(
            `${BASE_URL}/realms/${REALM}/protocol/openid-connect/userinfo`,
            {
                headers: { 'Authorization': `Bearer ${accessToken}` }
            }
        );

        check(userinfoRes, {
            'userinfo status is 200': (r) => r.status === 200,
        });

        // 3. Test token refresh
        const refreshRes = http.post(
            `${BASE_URL}/realms/${REALM}/protocol/openid-connect/token`,
            {
                grant_type: 'refresh_token',
                client_id: CLIENT_ID,
                client_secret: CLIENT_SECRET,
                refresh_token: refreshToken
            },
            {
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
            }
        );

        check(refreshRes, {
            'refresh status is 200': (r) => r.status === 200,
            'received new access token': (r) => r.json('access_token') !== undefined,
        });

        // 4. Test logout
        const logoutRes = http.post(
            `${BASE_URL}/realms/${REALM}/protocol/openid-connect/logout`,
            {
                client_id: CLIENT_ID,
                client_secret: CLIENT_SECRET,
                refresh_token: refreshToken
            },
            {
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
            }
        );

        check(logoutRes, {
            'logout status is 204': (r) => r.status === 204,
        });
    }

    sleep(1);
}
```

**Run Load Test**:

```bash
k6 run --out json=results.json load-test.js

# Analyze results
k6 run --out influxdb=http://localhost:8086/k6 load-test.js
```

### Database Performance

**PostgreSQL Optimization**:

```sql
-- Create indexes for performance
CREATE INDEX idx_user_entity_username ON user_entity(username);
CREATE INDEX idx_user_entity_email ON user_entity(email);
CREATE INDEX idx_user_role_mapping_user ON user_role_mapping(user_id);
CREATE INDEX idx_user_session_user ON user_session(user_id);
CREATE INDEX idx_client_session_session ON client_session(session_id);
CREATE INDEX idx_event_entity_realm_time ON event_entity(realm_id, event_time DESC);

-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM user_entity WHERE username = 'test@example.com';

-- Vacuum and analyze
VACUUM ANALYZE;

-- Monitor slow queries
ALTER DATABASE keycloak SET log_min_duration_statement = 1000;  # Log queries > 1s

-- Check table sizes
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;
```

**Connection Pooling** (PgBouncer):

```ini
# /etc/pgbouncer/pgbouncer.ini
[databases]
keycloak = host=localhost port=5432 dbname=keycloak

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
reserve_pool_size = 5
reserve_pool_timeout = 3
max_db_connections = 100
max_user_connections = 100
```

***

## 17. Real-World Use Cases

### Use Case 1: Multi-Tenant SaaS Platform

**Scenario**: B2B SaaS platform with multiple customer organizations, each with their own users, roles, and permissions.

**Architecture**:

```text
Keycloak Realm: saas-platform
├── Organizations (Groups)
│   ├── /acme-corp
│   │   ├── /acme-corp/admins
│   │   ├── /acme-corp/users
│   │   └── /acme-corp/billing
│   ├── /beta-industries
│   └── /gamma-solutions
│
├── Roles
│   ├── org_owner (realm role)
│   ├── org_admin (realm role)
│   ├── org_user (realm role)
│   └── Client roles (per application)
│
└── OpenFGA Integration
    ├── Organizations
    ├── Workspaces per organization
    └── Resources with fine-grained permissions
```

**Implementation**:

```javascript
// registration-service.js
async function registerOrganization(data) {
    // 1. Create Keycloak group for organization
    const group = await keycloakAdmin.groups.create({
        name: data.organizationName,
        attributes: {
            organization_id: [uuidv4()],
            subscription_tier: [data.tier],
            max_users: [data.maxUsers.toString()],
            created_at: [new Date().toISOString()]
        }
    });

    // 2. Create owner user
    const user = await keycloakAdmin.users.create({
        username: data.ownerEmail,
        email: data.ownerEmail,
        emailVerified: false,
        enabled: true,
        firstName: data.ownerFirstName,
        lastName: data.ownerLastName,
        credentials: [{
            type: 'password',
            value: data.temporaryPassword,
            temporary: true
        }]
    });

    // 3. Add user to organization group
    await keycloakAdmin.users.addToGroup({
        id: user.id,
        groupId: group.id
    });

    // 4. Assign org_owner role
    const ownerRole = await keycloakAdmin.roles.findOneByName({
        name: 'org_owner'
    });
    await keycloakAdmin.users.addRealmRoleMappings({
        id: user.id,
        roles: [ownerRole]
    });

    // 5. Create OpenFGA tuples
    await fgaClient.write({
        writes: [
            {
                user: `user:${user.id}`,
                relation: 'owner',
                object: `organization:${group.attributes.organization_id[0]}`
            }
        ]
    });

    // 6. Send verification email
    await keycloakAdmin.users.sendVerifyEmail({
        id: user.id,
        clientId: 'saas-app'
    });

    // 7. Initialize in billing system
    await initializeBilling(group.attributes.organization_id[0], data.tier);

    return {
        organizationId: group.attributes.organization_id[0],
        userId: user.id,
        message: 'Organization created successfully'
    };
}
```

**Multi-Tenant Isolation**:

```javascript
// Backend middleware
async function tenantIsolation(req, res, next) {
    const userId = req.user.sub;
    const organizationId = req.params.organizationId || req.query.organizationId;

    if (!organizationId) {
        return res.status(400).json({ error: 'Organization ID required' });
    }

    // Check if user belongs to organization
    const { allowed } = await fgaClient.check({
        user: `user:${userId}`,
        relation: 'member',
        object: `organization:${organizationId}`
    });

    if (!allowed) {
        return res.status(403).json({ error: 'Access denied to this organization' });
    }

    // Attach organization to request
    req.organization = { id: organizationId };
    next();
}

// Routes
app.get('/api/organizations/:organizationId/users',
    keycloak.protect(),
    tenantIsolation,
    async (req, res) => {
        // Get users for organization
        const users = await getUsersForOrganization(req.organization.id);
        res.json(users);
    }
);
```

### Use Case 2: Enterprise SSO with LDAP

**Scenario**: Large enterprise with existing Active Directory, wants SSO for all applications.

**Configuration**:

```text
1. Configure LDAP User Federation
   └── Sync 50,000 employees from Active Directory

2. Map LDAP groups to Keycloak groups
   └── Engineering → /Engineering group
   └── Sales → /Sales group
   └── HR → /HR group

3. Configure applications as OIDC clients
   ├── Intranet Portal
   ├── Email (Exchange)
   ├── CRM (Salesforce)
   ├── Collaboration (Slack)
   └── Custom internal apps

4. Enable Kerberos for Windows integrated auth
   └── Seamless SSO for domain-joined machines
```

**Kerberos Configuration**:

```properties
# LDAP provider settings
Allow Kerberos authentication: ON
Kerberos Realm: COMPANY.LOCAL
Server Principal: HTTP/auth.company.com@COMPANY.LOCAL
KeyTab: /etc/keycloak/krb5.keytab
```

**Windows Client Configuration**:

```powershell
# Configure browser for Kerberos (Group Policy)
# Internet Explorer / Edge
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\company.com\auth" /v https /t REG_DWORD /d 1

# Chrome (via Group Policy)
# Add to Local intranet zone
```

### Use Case 3: Customer Identity for E-commerce

**Scenario**: E-commerce platform needs customer authentication with social login, password reset, and loyalty integration.

**Features**:

```text
1. Registration
   ├── Email/password
   ├── Social login (Google, Facebook, Apple)
   ├── Guest checkout
   └── Email verification

2. Login
   ├── Username/password
   ├── Social login
   ├── Remember me (30 days)
   └── Optional MFA for high-value accounts

3. Account Management
   ├── Update profile
   ├── Change password
   ├── Manage addresses
   ├── View order history
   └── Manage payment methods

4. Integration
   ├── Loyalty points in token
   ├── Customer tier (bronze/silver/gold)
   ├── Subscription status
   └── Marketing preferences
```

**Custom Registration Flow**:

```javascript
// registration-handler.js
async function handleRegistration(formData) {
    try {
        // 1. Create user in Keycloak
        const user = await keycloakAdmin.users.create({
            username: formData.email,
            email: formData.email,
            firstName: formData.firstName,
            lastName: formData.lastName,
            enabled: true,
            emailVerified: false,
            attributes: {
                customer_tier: ['bronze'],
                loyalty_points: ['0'],
                marketing_consent: [formData.marketingConsent.toString()],
                registration_source: [formData.utmSource || 'direct'],
                phone_number: [formData.phoneNumber]
            },
            credentials: [{
                type: 'password',
                value: formData.password,
                temporary: false
            }]
        });

        // 2. Send verification email
        await keycloakAdmin.users.sendVerifyEmail({
            id: user.id,
            clientId: 'ecommerce-web'
        });

        // 3. Create customer in CRM
        await createCustomerInCRM({
            keycloakId: user.id,
            email: formData.email,
            firstName: formData.firstName,
            lastName: formData.lastName,
            source: formData.utmSource
        });

        // 4. Initialize loyalty account
        await initializeLoyaltyAccount(user.id);

        // 5. Send welcome email with discount code
        await sendWelcomeEmail(formData.email, {
            firstName: formData.firstName,
            discountCode: 'WELCOME10'
        });

        // 6. Track conversion
        await trackEvent('registration_complete', {
            user_id: user.id,
            source: formData.utmSource,
            timestamp: new Date()
        });

        return {
            success: true,
            message: 'Registration successful. Please check your email to verify your account.',
            userId: user.id
        };
    } catch (error) {
        console.error('Registration failed:', error);
        throw error;
    }
}
```

**Loyalty Points Integration**:

```javascript
// Update loyalty points in token
function addLoyaltyPointsMapper() {
    // Via Admin API or manually in console:
    // Client Scopes → profile → Mappers → Create
    
    // Mapper configuration:
    {
        name: 'loyalty-points',
        protocol: 'openid-connect',
        protocolMapper: 'oidc-usermodel-attribute-mapper',
        config: {
            'user.attribute': 'loyalty_points',
            'claim.name': 'loyalty_points',
            'jsonType.label': 'int',
            'id.token.claim': 'true',
            'access.token.claim': 'true',
            'userinfo.token.claim': 'true'
        }
    }
}

// Backend: Update loyalty points
async function addLoyaltyPoints(userId, points, reason) {
    // Get current points
    const user = await keycloakAdmin.users.findOne({ id: userId });
    const currentPoints = parseInt(user.attributes.loyalty_points[0]) || 0;
    const newPoints = currentPoints + points;

    // Update in Keycloak
    await keycloakAdmin.users.update(
        { id: userId },
        {
            attributes: {
                ...user.attributes,
                loyalty_points: [newPoints.toString()]
            }
        }
    );

    // Log transaction
    await logLoyaltyTransaction({
        userId,
        points,
        reason,
        balanceBefore: currentPoints,
        balanceAfter: newPoints,
        timestamp: new Date()
    });

    // Check for tier upgrade
    await checkTierUpgrade(userId, newPoints);
}
```

### Use Case 4: Mobile App Authentication

**Scenario**: iOS and Android apps need secure authentication with biometric support and offline access.

**Implementation**:

**iOS Example (Swift)**:

```swift
import AppAuth
import LocalAuthentication

class AuthManager {
    private var authState: OIDAuthState?
    
    func login(viewController: UIViewController, completion: @escaping (Bool) -> Void) {
        guard let issuer = URL(string: "https://auth.example.com/realms/mobile-app") else {
            completion(false)
            return
        }
        
        // Discover endpoints
        OIDAuthorizationService.discoverConfiguration(forIssuer: issuer) { configuration, error in
            guard let config = configuration else {
                print("Error discovering configuration: \(error?.localizedDescription ?? "Unknown")")
                completion(false)
                return
            }
            
            // Build authorization request
            let request = OIDAuthorizationRequest(
                configuration: config,
                clientId: "mobile-app-ios",
                clientSecret: nil,
                scopes: ["openid", "profile", "email", "offline_access"],
                redirectURL: URL(string: "com.example.app://callback")!,
                responseType: OIDResponseTypeCode,
                additionalParameters: nil
            )
            
            // Perform authorization
            let appDelegate = UIApplication.shared.delegate as! AppDelegate
            appDelegate.currentAuthorizationFlow = OIDAuthState.authState(
                byPresenting: request,
                presenting: viewController
            ) { authState, error in
                if let authState = authState {
                    self.authState = authState
                    self.saveAuthState(authState)
                    completion(true)
                } else {
                    print("Authorization error: \(error?.localizedDescription ?? "Unknown")")
                    completion(false)
                }
            }
        }
    }
    
    func loginWithBiometrics(completion: @escaping (Bool) -> Void) {
        let context = LAContext()
        var error: NSError?
        
        // Check if biometric auth is available
        guard context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) else {
            completion(false)
            return
        }
        
        // Authenticate with biometrics
        context.evaluatePolicy(
            .deviceOwnerAuthenticationWithBiometrics,
            localizedReason: "Authenticate to access your account"
        ) { success, error in
            if success {
                // Retrieve stored refresh token
                if let refreshToken = self.getStoredRefreshToken() {
                    self.refreshAccessToken(refreshToken: refreshToken, completion: completion)
                } else {
                    completion(false)
                }
            } else {
                completion(false)
            }
        }
    }
    
    func refreshAccessToken(refreshToken: String, completion: @escaping (Bool) -> Void) {
        guard let authState = self.authState else {
            completion(false)
            return
        }
        
        authState.setNeedsTokenRefresh()
        authState.performAction { accessToken, idToken, error in
            if let error = error {
                print("Token refresh failed: \(error.localizedDescription)")
                completion(false)
            } else {
                self.saveAuthState(authState)
                completion(true)
            }
        }
    }
    
    private func saveAuthState(_ authState: OIDAuthState) {
        let encoder = JSONEncoder()
        if let encoded = try? encoder.encode(authState) {
            // Store in Keychain (more secure than UserDefaults)
            KeychainHelper.save(data: encoded, forKey: "authState")
        }
    }
    
    private func getStoredRefreshToken() -> String? {
        if let data = KeychainHelper.load(forKey: "authState") {
            let decoder = JSONDecoder()
            if let authState = try? decoder.decode(OIDAuthState.self, from: data) {
                return authState.refreshToken
            }
        }
        return nil
    }
}
```

**Android Example (Kotlin)**:

```kotlin
import net.openid.appauth.*
import androidx.biometric.BiometricPrompt
import androidx.core.content.ContextCompat

class AuthManager(private val context: Context) {
    private val serviceConfig = AuthorizationServiceConfiguration(
        Uri.parse("https://auth.example.com/realms/mobile-app/protocol/openid-connect/auth"),
        Uri.parse("https://auth.example.com/realms/mobile-app/protocol/openid-connect/token")
    )
    
    fun login(activity: Activity) {
        val authRequest = AuthorizationRequest.Builder(
            serviceConfig,
            "mobile-app-android",
            ResponseTypeValues.CODE,
            Uri.parse("com.example.app://callback")
        )
            .setScopes("openid", "profile", "email", "offline_access")
            .setCodeVerifier(CodeVerifierUtil.generateRandomCodeVerifier()) // PKCE
            .build()
        
        val authService = AuthorizationService(context)
        val authIntent = authService.getAuthorizationRequestIntent(authRequest)
        activity.startActivityForResult(authIntent, AUTH_REQUEST_CODE)
    }
    
    fun loginWithBiometric(activity: FragmentActivity, onSuccess: () -> Unit, onError: () -> Unit) {
        val executor = ContextCompat.getMainExecutor(context)
        val biometricPrompt = BiometricPrompt(
            activity,
            executor,
            object : BiometricPrompt.AuthenticationCallback() {
                override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
                    super.onAuthenticationSucceeded(result)
                    
                    // Retrieve stored refresh token from encrypted storage
                    val refreshToken = getStoredRefreshToken()
                    if (refreshToken != null) {
                        refreshAccessToken(refreshToken, onSuccess, onError)
                    } else {
                        onError()
                    }
                }
                
                override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
                    super.onAuthenticationError(errorCode, errString)
                    onError()
                }
            }
        )
        
        val promptInfo = BiometricPrompt.PromptInfo.Builder()
            .setTitle("Biometric Authentication")
            .setSubtitle("Log in using your biometric credential")
            .setNegativeButtonText("Cancel")
            .build()
        
        biometricPrompt.authenticate(promptInfo)
    }
    
    fun handleAuthorizationResponse(
        data: Intent,
        onSuccess: (tokenResponse: TokenResponse) -> Unit,
        onError: () -> Unit
    ) {
        val authResponse = AuthorizationResponse.fromIntent(data)
        val authException = AuthorizationException.fromIntent(data)
        
        if (authResponse != null) {
            val authService = AuthorizationService(context)
            authService.performTokenRequest(
                authResponse.createTokenExchangeRequest()
            ) { tokenResponse, exception ->
                if (tokenResponse != null) {
                    // Store tokens securely
                    storeTokens(tokenResponse)
                    onSuccess(tokenResponse)
                } else {
                    onError()
                }
                authService.dispose()
            }
        } else {
            onError()
        }
    }
    
    private fun storeTokens(tokenResponse: TokenResponse) {
        // Use EncryptedSharedPreferences for secure storage
        val masterKey = MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .build()
        
        val sharedPreferences = EncryptedSharedPreferences.create(
            context,
            "auth_prefs",
            masterKey,
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        )
        
        with(sharedPreferences.edit()) {
            putString("access_token", tokenResponse.accessToken)
            putString("refresh_token", tokenResponse.refreshToken)
            putLong("expires_at", tokenResponse.accessTokenExpirationTime ?: 0)
            apply()
        }
    }
}
```

***

## 18. Troubleshooting

### Common Issues and Solutions

#### Issue 1: Invalid Redirect URI

**Error**: `Invalid parameter: redirect_uri`

**Cause**: Redirect URI in request doesn't match configured URIs in client.

**Solution**:

```text
1. Check client configuration:
   Clients → Select client → Settings → Valid redirect URIs

2. Ensure exact match (including protocol, port, path):
   ✓ https://app.example.com/callback
   ✗ http://app.example.com/callback (different protocol)
   ✗ https://app.example.com:8080/callback (different port)
   ✗ https://app.example.com/ (different path)

3. Use wildcards carefully:
   https://app.example.com/*  (matches any path)
   http://localhost:*/*  (matches any port and path on localhost)

4. Check application code:
   const redirectUri = 'https://app.example.com/callback';  // Must match exactly
```

#### Issue 2: CORS Errors

**Error**: `Access to fetch at 'https://auth.example.com/...' from origin 'https://app.example.com' has been blocked by CORS policy`

**Cause**: Web Origins not configured in client.

**Solution**:

```text
1. Add Web Origins in client:
   Clients → Select client → Settings → Web origins
   Add: https://app.example.com

2. Use + to allow all Valid redirect URIs:
   Web origins: +

3. In development, allow localhost:
   http://localhost:3000
   http://127.0.0.1:3000

4. Verify browser sees correct headers:
   Response headers should include:
   Access-Control-Allow-Origin: https://app.example.com
   Access-Control-Allow-Methods: GET, POST, OPTIONS
   Access-Control-Allow-Headers: Content-Type, Authorization
```

#### Issue 3: Token Expired

**Error**: `Token is not active`

**Cause**: Access token has exceeded its lifespan.

**Solution**:

```javascript
// Implement automatic token refresh
async function makeAuthenticatedRequest(url, options = {}) {
    // Check if token is expiring soon (30 seconds)
    const expiresAt = localStorage.getItem('token_expires_at');
    const now = Date.now() / 1000;
    
    if (expiresAt && now >= expiresAt - 30) {
        // Refresh token
        try {
            await refreshAccessToken();
        } catch (error) {
            // Refresh failed, redirect to login
            window.location.href = '/login';
            return;
        }
    }
    
    const accessToken = localStorage.getItem('access_token');
    
    const response = await fetch(url, {
        ...options,
        headers: {
            ...options.headers,
            'Authorization': `Bearer ${accessToken}`
        }
    });
    
    if (response.status === 401) {
        // Token invalid, try refresh
        try {
            await refreshAccessToken();
            // Retry request
            return makeAuthenticatedRequest(url, options);
        } catch (error) {
            window.location.href = '/login';
        }
    }
    
    return response;
}
```

#### Issue 4: User Not Found in LDAP

**Error**: `User not found` despite existing in LDAP

**Cause**: LDAP filter or search configuration incorrect.

**Solution**:

```text
1. Test LDAP connection:
   User federation → ldap-provider → Test connection

2. Test authentication:
   User federation → ldap-provider → Test authentication
   Enter LDAP username and password

3. Verify Users DN:
   Correct: ou=employees,dc=company,dc=com
   Not: ou=users,dc=company,dc=com (if users are in employees)

4. Check User LDAP Filter:
   Examples:
   (objectClass=inetOrgPerson)
   (&(objectClass=person)(!(employeeStatus=terminated)))
   (memberOf=cn=keycloak-users,ou=groups,dc=company,dc=com)

5. Test LDAP query manually:
   ldapsearch -H ldap://ldap.company.com \
     -D "cn=admin,dc=company,dc=com" \
     -W \
     -b "ou=employees,dc=company,dc=com" \
     "(uid=testuser)"

6. Check attribute mappings:
   Username LDAP attribute: uid (or sAMAccountName for AD)
   UUID LDAP attribute: entryUUID (or objectGUID for AD)

7. Trigger manual sync:
   User federation → ldap-provider → Synchronize all users
```

#### Issue 5: Session Not Shared Across Applications

**Error**: Users have to login separately for each application

**Cause**: SSO not configured correctly or different realms.

**Solution**:

```text
1. Verify all clients are in same realm:
   ✓ All clients in "customer-portal" realm
   ✗ Client A in "customer-portal", Client B in "other-realm"

2. Check redirect URIs:
   All applications must redirect to same Keycloak instance

3. Verify cookie domain:
   Keycloak sets SSO cookie for its domain
   All apps should be on same domain or subdomain:
   ✓ app1.example.com and app2.example.com (SSO works)
   ✗ app1.com and app2.com (SSO doesn't work - different domains)

4. Check browser cookie settings:
   Third-party cookies must be enabled
   SameSite cookie policy

5. Test SSO:
   Login to App 1 → Navigate to App 2 → Should be automatically logged in
```

#### Issue 6: High Database Connections

**Error**: `Too many database connections`

**Cause**: Connection pool exhausted or connection leaks.

**Solution**:

```sql
-- Check active connections
SELECT count(*) FROM pg_stat_activity WHERE datname = 'keycloak';

-- Check connection pool settings
SHOW max_connections;

-- Find idle connections
SELECT pid, usename, application_name, state, state_change 
FROM pg_stat_activity 
WHERE datname = 'keycloak' AND state = 'idle' 
ORDER BY state_change;

-- Kill idle connections (if needed)
SELECT pg_terminate_backend(pid) 
FROM pg_stat_activity 
WHERE datname = 'keycloak' AND state = 'idle' 
AND state_change < now() - interval '5 minutes';
```

**Adjust connection pool**:

```properties
# conf/keycloak.conf
db-pool-initial-size=20
db-pool-min-size=10
db-pool-max-size=100
```

#### Issue 7: Slow Performance

**Symptoms**: Slow login, token generation, page loads

**Diagnosis and Solutions**:

```bash
# 1. Check JVM memory
docker stats keycloak
# or
ps aux | grep keycloak

# Increase heap if needed:
export JAVA_OPTS="-Xms4g -Xmx8g"

# 2. Enable access logging to identify slow endpoints
# conf/keycloak.conf
log-level-org.keycloak.services=DEBUG

# 3. Check database performance
# PostgreSQL:
SELECT * FROM pg_stat_statements 
ORDER BY total_time DESC 
LIMIT 10;

# 4. Monitor with metrics
curl http://localhost:8080/metrics | grep http_server_requests

# 5. Check cache hit ratio
# Low cache hit ratio indicates cache is too small
cache-realm-max-entries=10000
cache-user-max-entries=10000

# 6. Profile with Java tools
jstack <keycloak-pid> > thread-dump.txt
jmap -dump:format=b,file=heap-dump.hprof <keycloak-pid>

# 7. Enable database connection pooling
# Use PgBouncer or similar
```

#### Issue 8: Email Not Sending

**Error**: Email verification/password reset emails not received

**Solution**:

```text
1. Test SMTP connection:
   Realm Settings → Email → Test connection

2. Check SMTP logs:
   /var/log/keycloak/keycloak.log
   Search for: "email" or "smtp"

3. Verify SMTP settings:
   Host: smtp.gmail.com
   Port: 587 (StartTLS) or 465 (SSL)
   Username: correct email
   Password: app-specific password (for Gmail)

4. Check firewall:
   telnet smtp.gmail.com 587

5. Test with CLI:
   echo "Subject: Test" | sendmail -v user@example.com

6. Common fixes:
   - Gmail: Use app-specific password
   - Office 365: Enable SMTP AUTH
   - AWS SES: Verify sender email
   - Check spam folder
   - Verify email templates are correct
```

#### Issue 9: Cannot Login to Admin Console

**Error**: Admin credentials not working

**Solution**:

```bash
# Reset admin password via CLI
/opt/keycloak/bin/kcadm.sh config credentials \
  --server http://localhost:8080 \
  --realm master \
  --user admin

# Or via environment variable
export KEYCLOAK_ADMIN=admin
export KEYCLOAK_ADMIN_PASSWORD=newpassword
systemctl restart keycloak

# Or directly in database (PostgreSQL)
# WARNING: Use only in emergency
UPDATE user_entity 
SET email_constraint = 'admin', 
    email = 'admin@example.com' 
WHERE username = 'admin' 
AND realm_id = 'master';

# Then reset password via email
```

#### Issue 10: Theme Not Loading

**Error**: Custom theme not appearing

**Solution**:

```bash
# 1. Verify theme directory structure
ls -la /opt/keycloak/themes/custom-theme/
# Should show: login/, account/, email/

# 2. Check theme.properties exists
cat /opt/keycloak/themes/custom-theme/login/theme.properties

# 3. Clear theme cache
rm -rf /opt/keycloak/data/tmp/*

# 4. Rebuild Keycloak
/opt/keycloak/bin/kc.sh build

# 5. Restart Keycloak
systemctl restart keycloak

# 6. Check logs for theme errors
tail -f /var/log/keycloak/keycloak.log | grep -i theme

# 7. Verify theme is selected in realm
Realm Settings → Themes → Login theme: custom-theme
```

### Debug Mode

**Enable detailed logging**:

```properties
# conf/keycloak.conf
log-level=DEBUG
log-level-org.keycloak=DEBUG
log-level-org.hibernate.SQL=DEBUG
```

**View logs**:

```bash
# Systemd service
journalctl -u keycloak -f

# Docker
docker logs -f keycloak

# File
tail -f /var/log/keycloak/keycloak.log
```

### Getting Help

**Resources**:
- **Official Documentation**: https://www.keycloak.org/documentation
- **Community Forum**: https://keycloak.discourse.group/
- **GitHub Issues**: https://github.com/keycloak/keycloak/issues
- **Stack Overflow**: Tag: [keycloak]
- **Mailing List**: keycloak-user@lists.jboss.org

**When Asking for Help**, provide:
1. Keycloak version
2. Deployment method (Docker, bare metal, Kubernetes)
3. Error message (full stack trace)
4. Configuration (sanitized, no secrets)
5. Steps to reproduce
6. Logs (relevant sections)
7. What you've already tried

***

## Conclusion

This comprehensive tutorial covered Keycloak from installation to advanced enterprise deployments. You now have the knowledge to:

✅ Install and configure Keycloak for production  
✅ Implement authentication flows (OIDC, SAML, social login)  
✅ Configure user federation with LDAP/Active Directory  
✅ Implement fine-grained authorization with OpenFGA  
✅ Secure your deployment with best practices  
✅ Monitor and optimize performance  
✅ Troubleshoot common issues  
✅ Implement real-world use cases  

Keycloak is a powerful, flexible identity management platform that can scale from small startups to large enterprises. Combined with OpenFGA for relationship-based access control, you can build sophisticated authorization systems that handle complex real-world scenarios.

**Next Steps**:
1. Set up a test environment following this guide
2. Experiment with different authentication flows
3. Integrate your first application
4. Gradually add advanced features (MFA, federation, authorization)
5. Load test and optimize for your use case
6. Join the Keycloak community

***
