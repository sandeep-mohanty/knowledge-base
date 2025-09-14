# Steps to Update "Cost Center" for a User in Microsoft Entra ID via Microsoft Graph API

This document outlines the general steps to update a user's `employeeOrgData.costCenter` property using Microsoft Graph API and OAuth 2.0 client credentials flow.

## 1. Create an OAuth App in Microsoft Entra ID

- Go to the Azure Portal.
- Navigate to "Microsoft Entra ID" > "App registrations".
- Click "New registration".
  - Name: Your app name (e.g., GraphAPIClient)
  - Supported account types: Choose based on your scenario (e.g., Single tenant)
  - Redirect URI: Not required for client_credentials flow
- Click "Register".

## 2. Set Required Application Permissions (Scopes)

- After the app is created, go to "API permissions".
- Click "Add a permission" > "Microsoft Graph".
- Choose "Application permissions".
- Search for and add the following permission:
  - `User.ReadWrite.All`
- Click "Add permissions".
- Click "Grant admin consent" for the tenant.

## 3. Generate a Client Secret

- Go to "Certificates & secrets" in the app registration.
- Click "New client secret".
  - Add description and expiration period.
- Click "Add".
- Copy and save the generated client secret securely.

## 4. Make a Token Request Using Client Credentials Grant Flow

Make a POST request to the token endpoint:

**URL:**

```
https://login.microsoftonline.com/{TENANT_ID}/oauth2/v2.0/token
```

**Headers:**

```
Content-Type: application/x-www-form-urlencoded
```

**Body (x-www-form-urlencoded):**

```
client_id={CLIENT_ID}
client_secret={CLIENT_SECRET}
grant_type=client_credentials
scope=https://graph.microsoft.com/.default
```

**Response:**

A successful response will return a JSON object containing the `access_token`.

Example:

```json
{
  "token_type": "Bearer",
  "expires_in": 3599,
  "access_token": "eyJ0eXAiOiJKV1QiLCJh..."
}
```

## 5. Use the Token to Update the User's Cost Center

Make a PATCH request to the Microsoft Graph API to update the `employeeOrgData.costCenter` property.

**URL:**

```
https://graph.microsoft.com/v1.0/users/{USER_EMAIL/OBJECT_ID}
```

**Headers:**

```
Authorization: Bearer {ACCESS_TOKEN}
Content-Type: application/json
```

**Body:**

```json
{
  "employeeOrgData": {
    "costCenter": "FIN-001"
  }
}
```

## 6. Verify the Cost Center Was Updated

Make a GET request to retrieve the user and check the `employeeOrgData.costCenter` value.

**URL:**

```
https://graph.microsoft.com/v1.0/users/{USER_ID/OBJECT_ID}?$select=employeeOrgData
```

**Headers:**

```
Authorization: Bearer {ACCESS_TOKEN}
```

**Response:**

```json
{
  "employeeOrgData": {
    "costCenter": "FIN-001"
  }
}
```

If the `costCenter` value matches the one you set, the update was successful.