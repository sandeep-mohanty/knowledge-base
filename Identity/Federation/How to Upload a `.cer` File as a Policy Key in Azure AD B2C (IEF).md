# How to Upload a `.cer` File as a Policy Key in Azure AD B2C (IEF)

## Goal

Add a `.cer` certificate file to Azure Active Directory B2C Identity Experience Framework (IEF) as a policy key, using a specific `StorageReferenceId`.

---

## Step-by-Step Instructions

### Step 1: Prepare your Certificate

- Ensure your certificate is in `.cer` format (X.509 public key only).
- If your certificate is in `.pem` or `.pfx`, convert it using OpenSSL:

```bash
openssl x509 -in mycert.pem -outform der -out mycert.cer
```

---

### Step 2: Upload Certificate via Azure Portal (Use "Upload" Option)

1. Go to the Azure Portal, and navigate to your Azure AD B2C tenant.
2. Go to **Identity Experience Framework** > **Policy Keys**.
3. Click **+ Add**.
4. Fill out the form:
   - **Name**: e.g., `MySigningCertKey` (this becomes your `StorageReferenceId`)
   - **Key type**: `X.509 Certificate`
   - **Key usage**: `Signature` or `Encryption`
   - **Options**: Select `Upload`
5. Once you select `Upload`, the file upload option will appear.
6. Upload your `.cer` file.
7. Click **Create**.

**Note:** If you do not select `Upload`, the portal will not allow you to submit your own `.cer` file. Options like `Generate` or `Manual` will not work for this purpose.

---

### Step 3: (Alternative) Upload Certificate via Azure CLI

```bash
az login
az account set --subscription "<your-subscription-id>"

az ad b2c policy key set \
  --name MySigningCertKey \
  --key-type X509Certificate \
  --usage Signature \
  --secret "@path/to/your/mycert.cer" \
  --b2c-tenant-name yourtenant.onmicrosoft.com
```

Replace the placeholders accordingly:

- `MySigningCertKey`: Name of the key (used as `StorageReferenceId`)
- `Signature`: Change to `Encryption` if needed
- `yourtenant.onmicrosoft.com`: Your B2C tenant domain
- `@path/to/your/mycert.cer`: Path to your certificate file

---

### Step 4: Reference the Key in Your Custom Policy XML

Add the following to your `TrustFrameworkExtensions.xml` file inside the `<PolicyKeys>` section:

```xml
<PolicyKeys>
  <Key Id="MySigningCertKey" StorageReferenceId="MySigningCertKey" />
</PolicyKeys>
```

Make sure:

- The `Id` is a unique identifier within your policy.
- The `StorageReferenceId` matches the name of the uploaded key.

---

## Troubleshooting Tips

- Ensure the certificate is in `.cer` format (X.509 public key only).
- Always select the `Upload` option when uploading in the portal.
- Use the correct key usage: `Signature` or `Encryption`.
- For private keys (`.pfx`), use Azure Key Vault or upload as a `Secret` key type.

---

## Summary

| Task                          | Method                        |
|------------------------------|-------------------------------|
| Upload `.cer` as policy key  | Azure Portal (`Upload`) / CLI |
| Reference in custom policy   | `<PolicyKeys>` XML section    |
| Key identifier               | `StorageReferenceId`          |

---

Need help with:

- Uploading private keys (`.pfx`)
- Using Azure Key Vault with IEF
- Understanding signing vs encryption key usage

Let me know!