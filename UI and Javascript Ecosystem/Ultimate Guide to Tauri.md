# 🦀🚀 Ultimate Guide to Tauri (with 3 Real Apps!)

Welcome back! Since you’ve tamed the mighty Electron beast, let’s dive into **Tauri**, its lightweight, security-focused cousin.  

We’ll learn **what Tauri is**, how it differs from Electron, and then build **the same 3 applications** (Postgres Viewer, OIDC Tester, SAML Tester) but in **Tauri** style. By the end, you’ll wield both frameworks like a desktop-app Jedi. ✨

---

## 🧩 What is Tauri?

**Tauri** lets you build cross-platform desktop applications using **Web technologies (HTML, CSS, JS)** for the **frontend**, but instead of shipping Chromium, it uses the system’s default WebView (WebKit on macOS/Linux, Edge’s WebView2 on Windows).  

The **backend** is written in **Rust**, giving you:
- **Tiny app bundles** (less than 10MB, compared to Electron’s 100+MB).
- **Security by design** (IPC is explicit).
- **Performance** (native Rust backend).
- **Frontend freedom**: You can use React, Vue, Svelte, or even vanilla HTML.

---

## ⚖️ Tauri vs Electron (Big Picture)

| Feature              | Electron                      | Tauri                                |
|----------------------|-------------------------------|--------------------------------------|
| UI Engine            | Bundled Chromium              | Native WebView (system WebView2/WKWebView, etc.) |
| Backend              | Node.js                       | Rust                                 |
| App Size             | Large (50–200 MB)             | Tiny (<10 MB)                        |
| Security             | Powerful but looser IPC       | Default secure IPC with explicit commands |
| Performance          | Heavy RAM                     | Lightweight & fast                   |

---

## ⚙️ Setting Up a Tauri Project

Install prerequisites:
- **Rust toolchain**
  ```bash
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```
- **Node.js & npm**  
- **Tauri CLI**
  ```bash
  cargo install create-tauri-app
  ```

Create a new project:
```bash
npx create-tauri-app tauri-demo
cd tauri-demo
npm run tauri dev
```

You now have a Tauri app skeleton.

---

# 🛠 Building the Same 3 Apps in Tauri

Tauri’s separation of concerns:  
- **Frontend (JS/HTML)** → Runs in WebView  
- **Backend (Rust)** → Handles system/database/auth

Communication between JS ↔ Rust is via **tauri::command**.

---

## 1) PostgreSQL Viewer

We’ll use the `tokio-postgres` crate in Rust to connect to Postgres.  
The frontend will be a simple HTML + JS page.

---

### Rust Backend (`src-tauri/src/main.rs`)

```rust
#![cfg_attr(
    all(not(debug_assertions), target_os = "windows"),
    windows_subsystem = "windows"
)]

use tauri::command;
use tokio_postgres::{NoTls};

#[command]
async fn run_query(query: String) -> Result<String, String> {
    let (client, connection) =
        tokio_postgres::connect("host=localhost user=postgres password=mypassword dbname=testdb", NoTls)
        .await.map_err(|e| e.to_string())?;

    // Spawn connection in background
    tokio::spawn(async move {
        if let Err(e) = connection.await {
            eprintln!("DB connection error: {:?}", e);
        }
    });

    let rows = client.query(query.as_str(), &[])
        .await.map_err(|e| e.to_string())?;

    let mut result = Vec::new();
    for row in rows {
        let mut values = Vec::new();
        for (idx, col) in row.columns().iter().enumerate() {
            let v: String = match row.try_get(idx) {
                Ok(val) => val,
                Err(_) => "NULL".into()
            };
            values.push(format!("{}={}", col.name(), v));
        }
        result.push(values.join(", "));
    }

    Ok(result.join("\n"))
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![run_query])
        .run(tauri::generate_context!())
        .expect("error while running Tauri application");
}
```

---

### Frontend (`src/index.html`)
```html
<!DOCTYPE html>
<html>
  <body>
    <form id="queryForm">
      <textarea id="sqlQuery" rows="5" cols="50"></textarea><br>
      <button type="submit">Run</button>
    </form>
    <pre id="results"></pre>

    <script type="module">
      import { invoke } from '@tauri-apps/api';

      const form = document.getElementById('queryForm');
      const input = document.getElementById('sqlQuery');
      const results = document.getElementById('results');

      form.addEventListener('submit', async (e) => {
        e.preventDefault();
        const res = await invoke("run_query", { query: input.value });
        results.innerText = res;
      });
    </script>
  </body>
</html>
```

✨ Congratulations! You’ve got a **Postgres Viewer in Tauri**.

---

## 2) OIDC Tester (Azure AD B2C)

Since Tauri doesn’t let you reuse Node.js libraries directly (like openid-client), we’ll use a **Rust OIDC library (`openidconnect`)**.  
The flow:
1. Rust backend initiates auth URL  
2. Opens Tauri WebView with login page  
3. Handles callback locally  
4. Returns token to frontend

---

### Dependencies (`Cargo.toml`)
```toml
[dependencies]
tauri = { version = "1", features = ["api-all"] }
openidconnect = "2"
url = "2"
tokio = { version = "1", features = ["full"] }
```

### Rust Backend (`src-tauri/src/main.rs`)
```rust
use tauri::command;
use openidconnect::core::*;
use openidconnect::{AuthorizationCode, ClientId, ClientSecret, CsrfToken, IssuerUrl, RedirectUrl, Scope, TokenResponse};
use tokio;

#[command]
async fn oidc_login() -> Result<String, String> {
    let issuer = IssuerUrl::new("https://<tenant>.b2clogin.com/<tenant>.onmicrosoft.com/v2.0/".to_string()).unwrap();

    // Typically you'd discover configuration from .well-known,
    // but for brevity, we're just simulating a token
    Ok("{ \"id_token\": \"XYZ123\", \"access_token\": \"ABC456\" }".into())
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![oidc_login])
        .run(tauri::generate_context!())
        .expect("error while running Tauri");
}
```

### Frontend (`src/index.html`)
```html
<!DOCTYPE html>
<html>
  <body>
    <button id="loginBtn">Login with OIDC</button>
    <pre id="result"></pre>

    <script type="module">
      import { invoke } from "@tauri-apps/api";

      document.getElementById("loginBtn").addEventListener("click", async () => {
        const res = await invoke("oidc_login");
        document.getElementById("result").innerText = res;
      });
    </script>
  </body>
</html>
```

This example mocks the token return for simplicity, but in practice you’d spin up a local server in Rust to capture the redirect and then parse tokens via `openidconnect`.

---

## 3) SAML Tester (Azure AD B2C)

Unlike Node.js where we used `passport-saml`, in Tauri we’ll lean on **Rust SAML libraries** (such as `saml2`). If not available, we can process raw XML assertions.  

---

### Rust Backend (`src-tauri/src/main.rs`)
```rust
use tauri::command;

#[command]
async fn saml_parse(assertion: String) -> Result<String, String> {
    // For demo, just echo the SAML XML back
    Ok(format!("Received SAML Assertion:\n{}", assertion))
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![saml_parse])
        .run(tauri::generate_context!())
        .expect("error while running Tauri");
}
```

### Frontend (`src/index.html`)
```html
<!DOCTYPE html>
<html>
  <body>
    <p>Paste your SAML assertion XML:</p>
    <textarea id="samlXml" rows="10" cols="70"></textarea><br>
    <button id="parseBtn">Parse Assertion</button>
    <pre id="result"></pre>

    <script type="module">
      import { invoke } from "@tauri-apps/api";

      document.getElementById("parseBtn").addEventListener("click", async () => {
        const xml = document.getElementById("samlXml").value;
        const res = await invoke("saml_parse", { assertion: xml });
        document.getElementById("result").innerText = res;
      });
    </script>
  </body>
</html>
```

In a real integration, you’d configure Azure AD B2C to POST the SAML response to your local Tauri app’s embedded server, then parse it in Rust.

---

# 🎉 Wrapping Up

We just ported your **Electron projects into Tauri**:

- 🗄 A **Postgres Viewer** using Rust `tokio-postgres`  
- 🔑 An **OIDC Tester** integrating with Azure AD B2C via Rust `openidconnect`  
- 🛡 A **SAML Tester** that processes assertions in Rust  

### Key Lessons:
- **Electron** = Node.js + Chromium  
- **Tauri** = Rust + System WebView  
- Electron is heavier but easier with npm packages.  
- Tauri is lighter, more secure, and Rust gives you crazy performance—like writing desktop apps with a jetpack strapped on.  
