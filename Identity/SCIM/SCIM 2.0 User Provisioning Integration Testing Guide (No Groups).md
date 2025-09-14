# SCIM 2.0 User Provisioning Integration Testing Guide (No Groups)

This guide outlines how to test your application's SCIM 2.0 user provisioning endpoints with identity providers like Okta, OneLogin, and Ping Identity. **Group provisioning is not covered.**

---

## ✅ Pre-requisites (User Provisioning Only)

Ensure your SCIM server supports the following endpoints:

| Endpoint | Purpose |
|---------|---------|
| `/scim/v2/Users` | Create, update, retrieve, delete users |
| `/scim/v2/Schemas` | Define your user schema (optional if using standard) |
| `/scim/v2/ServiceProviderConfig` | Tells IdP what features are supported |
| `/scim/v2/ResourceTypes` | Defines the available SCIM resource types (optional) |

Also ensure:
- Your SCIM server uses **Basic Auth** (required for Okta).
- Supports:
  - `PATCH` (for updates)
  - `DELETE` or `PATCH active=false` (for deactivation)
  - `GET` with filters and pagination
  - `POST` to create users
- Your `/Schemas` endpoint defines the standard user schema:  
  `urn:ietf:params:scim:schemas:core:2.0:User`

---

# 🧪 Identity Provider Integration Testing (User Provisioning Only)

---

## 🔹 1. Okta SCIM Integration (User Provisioning Only)

### ✅ Step 1: Create SCIM App in Okta

1. Go to **Applications > Applications** → Click **Create App Integration**.
2. Choose:
   - Platform: Web
   - Sign-On Method: SWA (or leave blank if you don’t need SSO)
3. Click **Create**.

---

### ✅ Step 2: Configure SCIM Provisioning

1. Go to **Provisioning > Integration**.
2. Enable **SCIM provisioning**.
3. Fill in:
   - **SCIM Base URL**: `https://yourdomain.com/scim/v2/`
   - **Unique identifier field for users**: `userName`
   - **Authentication Mode**: Basic Auth
     - Set a test username/password
4. Click **Test Connector Configuration**.
   - Okta will call `/ServiceProviderConfig`, `/Schemas`, `/ResourceTypes`.

---

### ✅ Step 3: Set Attribute Mappings

Go to **Provisioning > To App**, then map:

| Okta Attribute | SCIM Field |
|----------------|------------|
| `user.firstName` | `name.givenName` |
| `user.lastName` | `name.familyName` |
| `user.email` | `emails.value` |
| `user.login` | `userName` |

Save mappings.

---

### ✅ Step 4: Assign Test User

1. Go to the **Assignments** tab.
2. Assign a test user to the app.
3. This should trigger:
   - `POST /Users` — user creation
   - `PATCH /Users/{id}` — updates
   - `DELETE /Users/{id}` or `PATCH active=false` — deactivation
4. Monitor your SCIM server logs for request validation.

---

## 🔹 2. OneLogin SCIM Integration (User Provisioning Only)

### ✅ Step 1: Create SCIM App

1. Log into OneLogin Admin.
2. Go to **Applications > Add App**.
3. Search and add **SCIM Provisioner with SAML (or OIDC)**.

---

### ✅ Step 2: Configure SCIM Base URL

1. Go to the **Configuration** tab.
2. Enter:
   - **SCIM Base URL**: `https://yourdomain.com/scim/v2/`
   - **SCIM Bearer Token** or Basic Auth credentials

---

### ✅ Step 3: Enable Provisioning

1. Go to the **Provisioning** tab.
2. Enable:
   - Create Users
   - Update User Attributes
   - Deactivate Users
3. Save.

---

### ✅ Step 4: Attribute Mappings

Go to the **Parameters** tab to map:

| OneLogin Field | SCIM Field |
|----------------|------------|
| `First Name` | `name.givenName` |
| `Last Name` | `name.familyName` |
| `Email` | `emails.value` |
| `Username` | `userName` |

---

### ✅ Step 5: Assign User and Test

1. Assign a test user to the app.
2. Ensure your SCIM server receives:
   - `POST /Users`
   - `PATCH /Users/{id}`
   - `DELETE` or `PATCH active=false`

---

## 🔹 3. Ping Identity (PingOne)

### ✅ Step 1: Create App in PingOne

1. Go to **Connections > Applications > Add Application**.
2. Choose **Web App**.

---

### ✅ Step 2: Enable SCIM Provisioning

1. Go to **Provisioning** > Enable automatic provisioning.
2. Enter:
   - SCIM URL: `https://yourdomain.com/scim/v2/`
   - Authentication: Basic Auth or Bearer token

---

### ✅ Step 3: Attribute Mapping

Map user attributes such as:

| PingOne Field | SCIM Field |
|---------------|------------|
| `Username` | `userName` |
| `First Name` | `name.givenName` |
| `Last Name` | `name.familyName` |
| `Email` | `emails.value` |

---

### ✅ Step 4: Assign Users and Monitor

1. Assign users to the application.
2. Monitor your SCIM server logs for:
   - `POST /Users`
   - `PATCH /Users/{id}`
   - `DELETE` or `PATCH active=false`

---

# 📋 SCIM User Provisioning Test Checklist

| Test | Expected Result | ✅ |
|------|------------------|----|
| `GET /ServiceProviderConfig` | Returns supported features | ✅ |
| `GET /Schemas` | Returns supported user schema | ✅ |
| `POST /Users` | Creates user | ✅ |
| `PATCH /Users/{id}` | Updates user attributes | ✅ |
| `DELETE /Users/{id}` or `PATCH active=false` | Deactivates user | ✅ |
| Attribute Mapping | Correct SCIM fields used | ✅ |
| Basic Auth | Authenticated successfully | ✅ |

---

# 🧰 Optional: Use Postman to Simulate SCIM Calls

Manually test your SCIM API without an IdP using Postman or curl.

### POST /Users

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "userName": "jdoe",
  "name": {
    "givenName": "John",
    "familyName": "Doe"
  },
  "emails": [
    {
      "value": "jdoe@example.com",
      "primary": true
    }
  ]
}
```

### PATCH /Users/{id}

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [
    {
      "op": "replace",
      "path": "name.givenName",
      "value": "Johnny"
    }
  ]
}
```

---

# 📚 Final Tips

- You don’t need to implement `/Groups`, `/GroupMembers`, or group-related logic.
- Document clearly for your customers: “Group provisioning not supported”.
- Ensure correct SCIM status codes (`201`, `204`, `400`, `401`, etc.).
- Return responses with `application/scim+json` content-type.
- Consider rate limiting and retry logic during bulk provisioning.

---