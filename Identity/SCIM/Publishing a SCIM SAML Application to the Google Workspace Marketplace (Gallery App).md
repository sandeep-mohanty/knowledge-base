# Publishing a SCIM + SAML Application to the Google Workspace Marketplace (Gallery App)

So far, we’ve covered how to integrate your SCIM + SAML-compatible app with Google Workspace as a **custom app** inside one organization.  
This tutorial explains how to go one step further: publishing your app to the **Google Workspace Marketplace**, so any Workspace administrator across organizations can discover and deploy it.

---

## What is a Google Workspace Marketplace App?

- The **Google Workspace Marketplace** is the "app gallery" where third-party integrations can be browsed and installed by Workspace domain administrators.  
- By publishing your app here, it becomes available as a **gallery app**, rather than requiring each customer to manually configure a "Custom App".  
- This greatly reduces onboarding friction for your customers.

---

## Step 1: Prerequisites Before Publishing

1. **Google Cloud Project**  
   - You will need a Google Cloud project linked to your Google developer account.  
   - Go to [Google Cloud Console](https://console.cloud.google.com/).  
   - Enable the **Google Workspace Marketplace SDK** for your project.

2. **Domain-wide SAML + SCIM Setup Ready**  
   - Your app already needs working **SAML SSO** setup (Google IdP + your SP).  
   - Your SCIM 2.0 API must be tested and functional with OAuth bearer token authorization.  

3. **Branding and Assets**  
   - App name, logo, description, and marketing text for Marketplace listing.  
   - Support documentation URLs (admin setup steps, end-user guide).

4. **Verification**  
   - A verified domain with ownership proof in Google Search Console.  
   - OAuth client verification (if your app uses Google APIs alongside SCIM/SSO).

---

## Step 2: Building the Marketplace App Configuration

1. Navigate to your Google Cloud project.  
2. Go to **APIs & Services** → **Google Workspace Marketplace SDK** → Enable.  
3. Configure **App Information**:  
   - Name, description, company details, privacy policy, support URL.  
   - Upload logos and icons.  
4. Configure **App Integrations**:  
   - **SSO Setup**: Provide SAML metadata exchange instructions. (Your app will consume the IdP Metadata URL from Google.)  
   - **User Provisioning**: Define the SCIM Base URL and OAuth 2.0 client credentials flow setup.  
   - **OAuth scopes** (if your app calls Google APIs — not always needed for pure SCIM/SSO).  

5. Configure **Visibility**:  
   - Private (for testing within your domain).  
   - Public (to list in the Marketplace for all Workspace domains).  

---

## Step 3: Approval Process from Google

- For **public listing**, apps undergo a **Google Security and Compliance Review**, which includes:  
  - **OAuth Verification**: If your app asks for Google APIs/data access, it will need OAuth consent screen verification.  
  - **Penetration Testing / Security Assessment**: Google requires third‑party security assessment for apps accessing sensitive scopes.  
  - **Brand Review**: Logos, app descriptions, and privacy policy must meet standards.  
  - **SAML/SCIM Setup Validation**: Google will review your documentation to ensure admins can configure SSO + provisioning correctly.

---

## Step 4: Configuring a Marketplace (Gallery) App Once Approved

For administrators who install your app from the Marketplace:  
1. Admin goes to [Google Admin Console](https://admin.google.com) → **Apps** → **Google Workspace Marketplace apps**.  
2. Finds your app in the Marketplace, clicks **Install**.  
3. During installation, admins will:  
   - Accept scopes and permissions requested by your app.  
   - Configure **SSO via SAML** by using the IdP metadata exchange.  
   - Enable **SCIM provisioning** by entering client credentials provided by your app platform.  
4. When enabled, the app behaves like the earlier **Custom App**, but the Marketplace listing provides all base config (URLs, metadata) — admins only enter tenant‑specific details.

---

## Step 5: SCIM Compliance Checks — Does Google Provide a Validator?

- **Microsoft Entra ID (Azure AD)** includes an official **SCIM validator tool** to check endpoints for RFC 7644 compliance.  
- **Google Workspace currently does *not* provide an official SCIM validator tool**.  
- Validation typically happens via:  
  - Google’s internal review of your app during Marketplace approval.  
  - Your own testing by enabling SCIM provisioning in a test Workspace tenant.  
  - Ensuring your implementation aligns with:  
    - [RFC 7643: SCIM Schema](https://datatracker.ietf.org/doc/html/rfc7643)  
    - [RFC 7644: SCIM Protocol](https://datatracker.ietf.org/doc/html/rfc7644)  

✅ **Tip:** For self-validation, you can use:  
- **Postman collections** for SCIM 2.0 test cases.  
- **Open-source SCIM tools** like [SCIM-SDK](https://github.com/PowerShell/SCIM) or libraries in Java/Python/Node to simulate compliance checks.  

---

## Step 6: Differences Between Custom App vs Gallery App

| Feature               | Custom App (per tenant)          | Marketplace/Gallery App (public)             |
|-----------------------|----------------------------------|---------------------------------------------|
| Setup effort          | Each admin configures manually   | Preconfigured metadata; admin enters fewer details |
| Discoverability       | Hidden; requires vendor docs     | Searchable in Google Workspace Marketplace   |
| Approval              | Immediate (self-configured)      | Requires Google app review and approval      |
| SCIM validation tool  | Manual testing only              | Manual testing only (no official validator)  |

---

## Final Thoughts

By publishing your app as a **Google Workspace Marketplace app**:  
- You reduce friction for admins (no manual typing of SCIM URLs, SSO metadata, etc.).  
- Your app becomes discoverable by thousands of Workspace organizations.  
- Approval requires strong branding, security review, and compliance with both OAuth and SCIM standards.  
- Remember that **Google does not offer a SCIM validator tool** like Microsoft Entra ID does — so you must test compliance thoroughly yourself and be ready for Google’s review process.  

With your Marketplace listing live, your SCIM + SAML app can reach a far wider audience and provide a seamless setup flow for admins across all Google Workspaces 🚀.