# Integrating a Google Workspace Custom App with Your SCIM-Compatible Web Application

This guide walks you through creating a custom app in Google Workspace that can both:

1. **Provision users** into your SCIM-compatible application.  
2. **Enable Single Sign-On (SSO)** where Google acts as the Identity Provider (IdP) and your application is the Service Provider (SP).  

We will cover SCIM integration with OAuth 2.0 client credentials flow, and then SAML for SSO.

---

## Prerequisites

- **Access to the Google Admin Console:**  
  Sign in at [https://admin.google.com](https://admin.google.com).  

- **Google Workspace Account Requirements:**  
  You need to log in with a Google Workspace **admin account**. Specifically:  
  - The easiest choice is a **Super Admin** account, which has all privileges.  
  - Alternatively, a **custom admin role** with these **minimum permissions**:  
    - "Manage SAML apps" (under Apps) — to add and configure custom SAML apps.  
    - "Manage user provisioning" (under User & App settings) — to enable SCIM provisioning.  
    - "Read user information" — to view and provision users.  
    - "Assign applications to users" — to assign the app to users or groups.  
    - "Security settings" — to configure SSO settings.  

- **SCIM 2.0 user endpoints** implemented on your application (`/Users`, `/Groups` if needed).  
- **OAuth 2.0 token endpoint** available for Google to request access tokens using the **client credentials flow**.  
- **Client ID and client secret** that your system recognizes for issuing OAuth tokens.  
- **SAML SP metadata** from your application (or the ability to configure it manually).  

---

## Step 1: Prepare Your Application for SCIM Provisioning with OAuth 2.0

1. Ensure your SCIM API endpoints are protected and require a **Bearer Token**.  
   
2. Implement an **OAuth 2.0 token endpoint** on your side that supports this request format:

   ~~~http
   POST /oauth/token
   Content-Type: application/x-www-form-urlencoded

   grant_type=client_credentials&
   client_id=YOUR_CLIENT_ID&
   client_secret=YOUR_CLIENT_SECRET&
   scope=SCIM
   ~~~

3. Your token endpoint should return JSON such as:

   ~~~json
   {
     "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIg...",
     "token_type": "Bearer",
     "expires_in": 3600
   }
   ~~~

4. Google SCIM provisioning will automatically call this endpoint using the provided client_id and client_secret whenever it needs to provision users.

---

## Step 2: Create a Custom SAML / SCIM App in Google Workspace

1. Go to **Google Admin Console** → `Apps` → `Web and mobile apps`.  
2. Click **Add App** → `Add custom SAML app`.  
3. Give your app a meaningful name (e.g., **My SCIM + SSO App**).  
4. Google will provide an **IdP Metadata URL**.  
   - Copy this URL.  
   - In your application (as the Service Provider), configure Google as the IdP using this metadata URL.  
   - Your SP will automatically fetch and stay updated with the IdP’s details (SSO URL, Entity ID, certificate).  

This approach avoids manual copy/paste and keeps your integration secure and current.

---

## Step 3: Configure SSO in Your Application (SP Role)

1. In your app, paste the IdP Metadata URL you obtained.  
2. Define your **Assertion Consumer Service (ACS) URL** — this is where Google will POST the SAML response.  
3. Define your **Audience URI / SP Entity ID** that matches your application’s identifier.  
4. If needed, configure attribute mapping (for example:  
   - `givenName` → first name  
   - `sn` → last name  
   - `email` → user email address).  

---

## Step 4: Enable SCIM Provisioning for the App in Google Workspace

1. In the admin console, open the same custom app.  
2. Navigate to the **Provisioning** section.  
3. **Turn on provisioning** with the toggle/button to explicitly enable automatic SCIM operations.  
4. Provide your app’s SCIM details:  
   - **SCIM base URL** → something like `https://yourapp.com/scim/v2`.  
   - **Authentication** → choose **OAuth 2.0 client credentials**.  
   - **Token URL** → `https://yourapp.com/oauth/token`.  
   - **Client ID / Client Secret** provided by you.  
5. Test the connection. Google will attempt to get a token and hit your `/Users` endpoint.  

⚠️ **Important Notes about Provisioning in Google Workspace**:  
- Google Workspace uses **event-driven provisioning**. Provisioning is triggered when:  
  - A user or group is assigned to the SAML/SCIM app.  
  - A user’s attributes change.  
  - A user is suspended or deleted.  
- There is **no “Run provisioning now” or “Start sync” button** like in Microsoft Entra ID.  
- If you want to trigger an initial sync, assign users or groups to the app, which immediately pushes provisioning calls to your SCIM API.  

---

## Step 5: Assign Users to the App

1. In the admin console, select the app and go to **User access**.  
2. Assign some users or groups to validate provisioning.  
3. When provisioned, Google will call your SCIM `/Users` endpoint with the obtained bearer token.  
4. You should see user creation in your application.

---

## Step 6: Validate End-to-End Flow

- **Test SSO:** From the Google Workspace app launcher, click your custom app. You should be redirected and logged into your application via SAML SSO.  
- **Test Provisioning:** Add a new user to the app in Google Workspace. Confirm via your logs or API that the SCIM `/Users` POST request created the account.  
- **Update or Deactivate:** Try suspending the user in Google Workspace. Google should call your SCIM PATCH/DELETE endpoints to reflect the change.

---

## Bonus: Troubleshooting Tips

- **Token issues:** Double-check your token endpoint supports `application/x-www-form-urlencoded`. Google is picky.  
- **SCIM schema mismatches:** Ensure your JSON matches SCIM spec (e.g., `userName` required).  
- **Time sync:** SAML requires accurate system clocks. Use NTP. Unless you want to live the chaotic life of tracing 5-minute time drift bugs.  

---

## Final Thoughts

You’ve now connected Google Workspace to your SCIM + SAML enabled application:  
- SCIM keeps your user directory in sync using dynamic OAuth 2.0 tokens.  
- SAML lets your users sign in with their Google Workspace credentials.  
- Provisioning is **automatic and event-driven**, not batch‑triggered — no “Run sync now” button exists.  

Your app is now a first-class citizen in the Google ecosystem. Congratulations—you’ve basically built the kind of integration that SaaS vendors brag about at conferences! 🌟