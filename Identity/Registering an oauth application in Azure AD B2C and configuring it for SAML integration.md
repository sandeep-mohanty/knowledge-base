# ⚡ Automating Azure AD B2C App Registration as SAML Relying Party with Microsoft Graph  

You’re absolutely right ✅ — in the **SAML Relying Party integration scenario in B2C**, **claims/token mappings are configured in the custom policy XML**, *not* in the app manifest.  

So for our automation, we will focus solely on:  
- Setting **`requestedAccessTokenVersion = 2`**  
- Setting **`identifierUris`** to the **Service Provider (SP) Entity ID**  
- Setting **`web.redirectUris`** to the **ACS URL** (where SAML Assertion is sent)  
- Setting **`web.logoutUrl`** to the SP logout endpoint  

We’ll do this automation with both **Python (MSAL + Requests)** and **Postman/Newman**.  

---

# 📚 Prerequisites  

- **Azure AD B2C tenant** and admin/service principal credentials  
- Automation account (service principal) with → `Application.ReadWrite.All` + `Directory.ReadWrite.All`  
- Microsoft Graph endpoint: `https://graph.microsoft.com/v1.0`  
- Inputs from SP (Service Provider):  
  - **ACS URL** (Assertion Consumer Service endpoint)  
  - **Entity ID** (unique identifier/issuer)  
  - **Logout URL**  

# Part 1: 🐍 Python Script  

## Dependencies  

```bash
pip install msal requests
```

---

## Script  

```python
# b2c_saml_app_reg.py
import requests, msal, json

# 🔧 Fill with your automation App creds
TENANT_ID = "xxxx-your-b2c-tenant-id"
CLIENT_ID = "xxxx-automation-app-client-id"
CLIENT_SECRET = "xxxx-automation-app-secret"
AUTHORITY = f"https://login.microsoftonline.com/{TENANT_ID}"
SCOPE = ["https://graph.microsoft.com/.default"]
GRAPH_URL = "https://graph.microsoft.com/v1.0"

# Service Provider (SP) values
ACS_URL = "https://sp.example.com/saml/acs"
ENTITY_ID = "https://sp.example.com/saml/metadata"
LOGOUT_URL = "https://sp.example.com/saml/logout"

def get_token():
    app = msal.ConfidentialClientApplication(
        client_id=CLIENT_ID,
        client_credential=CLIENT_SECRET,
        authority=AUTHORITY
    )
    token = app.acquire_token_for_client(SCOPE)
    return token["access_token"]

token = get_token()
headers = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}

# Step 1: Create application
app_payload = {
    "displayName": "MySamlRelyingParty",
    "web": {
        "redirectUris": [ACS_URL],
        "logoutUrl": LOGOUT_URL
    },
    "identifierUris": [ENTITY_ID],
    "signInAudience": "AzureADandPersonalMicrosoftAccount",
    "requestedAccessTokenVersion": 2
}

resp = requests.post(f"{GRAPH_URL}/applications", headers=headers, json=app_payload)
if resp.status_code != 201:
    print("❌ Error creating app:", resp.text)
    exit(1)
app_data = resp.json()
print("✅ App registered")
print(json.dumps(app_data, indent=2))

object_id = app_data["id"]

# Step 2: (Optional) ensure values set in PATCH
update = {
    "requestedAccessTokenVersion": 2,
    "identifierUris": [ENTITY_ID]
}
resp2 = requests.patch(f"{GRAPH_URL}/applications/{object_id}", headers=headers, json=update)
if resp2.status_code in (200,204):
    print("✅ App manifest updated with requestedAccessTokenVersion=2 and identifierUris")
else:
    print("❌ Update failed:", resp2.text)
```

Run:  
```bash
python b2c_saml_app_reg.py
```

---

# Part 2: 📮 Postman / Newman Automation  

### Step 1: Acquire Token  

**POST** `https://login.microsoftonline.com/{{tenant_id}}/oauth2/v2.0/token`  
Body (x-www-form-urlencoded):  
```
client_id={{client_id}}
client_secret={{client_secret}}
scope=https://graph.microsoft.com/.default
grant_type=client_credentials
```

In **Tests Tab**:  
```js
const resp = pm.response.json();
pm.environment.set("access_token", resp.access_token);
```

---

### Step 2: Register Application  

**POST** `https://graph.microsoft.com/v1.0/applications`  
Headers:  
```
Authorization: Bearer {{access_token}}
Content-Type: application/json
```

Body:  
```json
{
  "displayName": "MySamlRelyingParty",
  "web": {
    "redirectUris": ["https://sp.example.com/saml/acs"],
    "logoutUrl": "https://sp.example.com/saml/logout"
  },
  "identifierUris": [
    "https://sp.example.com/saml/metadata"
  ],
  "signInAudience": "AzureADandPersonalMicrosoftAccount",
  "requestedAccessTokenVersion": 2
}
```

Capture `id` as variable:  
```js
pm.environment.set("object_id", pm.response.json().id);
```

---

### Step 3: Update Manifest (Optional Reinforce)  

**PATCH** `https://graph.microsoft.com/v1.0/applications/{{object_id}}`  
Body:  
```json
{
  "requestedAccessTokenVersion": 2,
  "identifierUris": [
    "https://sp.example.com/saml/metadata"
  ]
}
```

---

### Step 4: Run via Newman  

Export Postman collection/environment as JSON, then run in your CI/CD:  

```bash
newman run b2c_saml_registration.json -e env.json
```

---

# 🎯 Summary  

In this automation:  

- Application Registered → B2C **App object created**.  
- Critical attributes configured:  
  - `requestedAccessTokenVersion: 2` ✅  
  - `identifierUris: [SP EntityID]` ✅  
  - `web.redirectUris: [ACS URL]` ✅  
  - `web.logoutUrl: SP logout endpoint` ✅  
- Covered **two automation styles**:  
  - **🐍 Python** for flexible scripting and pipelines.  
  - **📮 Postman/Newman** for collaborative API workflows & CI.  

This ensures your **Azure AD B2C RP app configuration for SAML SPs** is **repeatable and codified**, eliminating portal clicks!  