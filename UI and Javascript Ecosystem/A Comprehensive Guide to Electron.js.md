# 🚀 A Comprehensive Guide to Electron.js (with 3 Real Apps!)

Welcome, curious developer! We’re about to embark on a journey through **Electron.js**, learning what it is, how it works, and then applying that knowledge to build **three practical desktop applications**:  

1. **PostgreSQL Viewer**  
2. **OIDC Tester**  
3. **SAML Tester**  

These apps will integrate with modern identity systems (Azure AD B2C custom policies), giving you hands-on experience in connecting Electron with authentication flows and databases.

---

## 🧩 What is Electron.js?

**Electron.js** is a framework that lets you build **desktop applications** using **JavaScript, HTML, and CSS**.  

Think of it as the love child of **Chromium** (the rendering engine behind Chrome) and **Node.js** (the runtime for JavaScript on the server). You write a web app, and Electron packages it into a cross-platform desktop app that can run on macOS, Windows, and Linux.  

Key features include:
- **Frontend in Chromium**: You use web technologies to make your UI.  
- **Backend in Node.js**: Full Node.js capability for filesystem, networking, and databases.  
- **Packaged apps**: Bundle everything into executable installers.  
- **System-level integrations**: Notifications, menus, tray icons, etc.  

---

## 🏛 Electron Architecture

An Electron app consists of **two primary processes**:

1. **Main Process** (the “brains”)
   - Runs in Node.js.
   - Controls application life-cycle and system-level interactions (menus, windows, etc).
   - Each Electron app has one main process.

2. **Renderer Processes** (the “faces”)
   - Runs in Chromium.
   - Handles UI in separate processes (one per BrowserWindow).
   - Can communicate with the main process using **IPC (Inter-Process Communication)**.

```
Main Process (Node.js)
     |
     |-- Creates windows
     |
Renderer Process (Chromium + UI)
     ⬆ IPC ⬇
```

---

## ⚙️ Setting Up Your Environment

Before we dive into code, let’s set up a new Electron project.

```bash
# Install dependencies
npm init -y
npm install electron --save-dev
```

Add a **start script** to your `package.json`:

```json
"scripts": {
  "start": "electron ."
}
```

Create the following starter files:

```
/project-root
 ├── main.js         <-- Main process entry
 ├── index.html      <-- Renderer process entry
 ├── renderer.js     <-- Renderer logic
 └── package.json
```

### Example Boilerplate

**main.js**
```javascript
const { app, BrowserWindow } = require('electron');

function createWindow() {
  const win = new BrowserWindow({
    width: 1000,
    height: 700,
    webPreferences: {
      nodeIntegration: true, // expose node in renderer
      contextIsolation: false
    }
  });
  win.loadFile('index.html');
}

app.whenReady().then(createWindow);
```

**index.html**
```html
<!DOCTYPE html>
<html>
<head>
  <title>My First Electron App</title>
</head>
<body>
  <h1>Hello, Electron!</h1>
  <script src="renderer.js"></script>
</body>
</html>
```

**renderer.js**
```javascript
console.log("Renderer process loaded!");
```

Run your app:

```bash
npm start
```

Boom 💥! You’ve just birthed your first little Electron window.

---

# 🛠 Now Let’s Build 3 Real Apps

## 1) PostgreSQL Viewer (Desktop SQL Client)

The idea: A GUI desktop app that connects to PostgreSQL, executes queries, and displays results.

---

### Installing Dependencies

You need the `pg` library to connect to PostgreSQL:

```bash
npm install pg
```

---

### Example Code

**renderer.js**
```javascript
const { ipcRenderer } = require('electron');

document.addEventListener('DOMContentLoaded', () => {
  const queryForm = document.getElementById('queryForm');
  const queryInput = document.getElementById('sqlQuery');
  const resultsDiv = document.getElementById('results');

  queryForm.addEventListener('submit', (e) => {
    e.preventDefault();
    ipcRenderer.send('run-query', queryInput.value);
  });

  ipcRenderer.on('query-results', (event, rows) => {
    resultsDiv.innerHTML = JSON.stringify(rows, null, 2);
  });
});
```

**index.html**
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Postgres Viewer</title>
  </head>
  <body>
    <form id="queryForm">
      <textarea id="sqlQuery" rows="5" cols="50"></textarea>
      <br>
      <button type="submit">Run</button>
    </form>
    <pre id="results"></pre>
    <script src="renderer.js"></script>
  </body>
</html>
```

**main.js**
```javascript
const { app, BrowserWindow, ipcMain } = require('electron');
const { Client } = require('pg');

function createWindow() {
  const win = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: { nodeIntegration: true, contextIsolation: false }
  });
  win.loadFile('index.html');
}

// Handle queries from renderer
ipcMain.on('run-query', async (event, query) => {
  const client = new Client({
    user: 'postgres',
    host: 'localhost',
    database: 'testdb',
    password: 'mypassword',
    port: 5432,
  });
  await client.connect();
  const result = await client.query(query);
  event.sender.send('query-results', result.rows);
  await client.end();
});

app.whenReady().then(createWindow);
```

Voilà! You have a Postgres Viewer GUI 🎉.

---

## 2) OIDC Tester (with Azure AD B2C)

We want to test an **OpenID Connect** authentication flow against Azure AD B2C.  

Electron can embed a small browser window to handle the authentication.

---

### Dependencies

```bash
npm install openid-client
```

---

### Example Flow

- User clicks "Login"
- Electron opens a BrowserWindow pointing to the Azure AD B2C login URL
- After login, we intercept the redirect URI and extract tokens.

**main.js**
```javascript
const { app, BrowserWindow, ipcMain, session } = require('electron');
const { Issuer, generators } = require('openid-client');

let mainWin;
async function createWindow() {
  mainWin = new BrowserWindow({
    width: 1000,
    height: 700,
    webPreferences: { nodeIntegration: true, contextIsolation: false }
  });
  mainWin.loadFile('index.html');
}

ipcMain.on('oidc-login', async (event) => {
  const b2cIssuer = await Issuer.discover('https://<your-tenant-name>.b2clogin.com/<your-tenant-name>.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=<your-policy-name>');
  
  const client = new b2cIssuer.Client({
    client_id: '<CLIENT_ID>',
    redirect_uris: ['http://localhost/callback'],
    response_types: ['code'],
  });

  const nonce = generators.nonce();
  const url = client.authorizationUrl({
    scope: 'openid profile email',
    nonce,
  });

  const authWin = new BrowserWindow({ width: 800, height: 600 });
  authWin.loadURL(url);

  const filter = { urls: ['http://localhost/callback*'] };
  session.defaultSession.webRequest.onCompleted(filter, async ({ url }) => {
    const params = client.callbackParams(url);
    const tokenSet = await client.callback('http://localhost/callback', params, { nonce });
    event.sender.send('oidc-token', tokenSet);
    authWin.close();
  });
});

app.whenReady().then(createWindow);
```

**renderer.js**
```javascript
const { ipcRenderer } = require('electron');

document.getElementById('loginBtn').addEventListener('click', () => {
  ipcRenderer.send('oidc-login');
});

ipcRenderer.on('oidc-token', (event, tokenSet) => {
  document.getElementById('result').innerText = JSON.stringify(tokenSet, null, 2);
});
```

**index.html**
```html
<!DOCTYPE html>
<html>
  <head><title>OIDC Tester</title></head>
  <body>
    <button id="loginBtn">Login with Azure AD B2C</button>
    <pre id="result"></pre>
    <script src="renderer.js"></script>
  </body>
</html>
```

Now you can authenticate directly with your Azure AD B2C tenant. ✨

---

## 3) SAML Tester (with Azure AD B2C)

Azure AD B2C can be configured to support **SAML flows**.  
Electron can simulate a **SAML SP (Service Provider)** tester.

Since full SAML response parsing requires dedicated libraries, we’ll use `passport-saml`.

---

### Dependencies

```bash
npm install passport-saml express
```

---

### Example Flow

We’ll host a small **local express server** inside Electron to receive the SAML assertion, then display it in the app.

**main.js**
```javascript
const { app, BrowserWindow, ipcMain } = require('electron');
const express = require('express');
const bodyParser = require('body-parser');
const saml = require('passport-saml').Strategy;

let mainWin;

function createWindow() {
  mainWin = new BrowserWindow({
    width: 1000,
    height: 700,
    webPreferences: { nodeIntegration: true, contextIsolation: false }
  });
  mainWin.loadFile('index.html');
}

// Local web server for SAML responses
const server = express();
server.use(bodyParser.urlencoded({ extended: false }));

server.post('/saml/acs', (req, res) => {
  console.log("SAML Response:", req.body);
  mainWin.webContents.send('saml-assertion', req.body);
  res.send('<h1>Login Successful, you can close this window</h1>');
});

server.listen(3000, () => {
  console.log('SAML SP listening on http://localhost:3000');
});

app.whenReady().then(createWindow);
```

**renderer.js**
```javascript
const { ipcRenderer } = require('electron');

ipcRenderer.on('saml-assertion', (event, assertion) => {
  document.getElementById('result').innerText = JSON.stringify(assertion, null, 2);
});
```

**index.html**
```html
<!DOCTYPE html>
<html>
  <head><title>SAML Tester</title></head>
  <body>
    <p>Configure Azure AD B2C to send a SAML assertion to http://localhost:3000/saml/acs</p>
    <pre id="result"></pre>
    <script src="renderer.js"></script>
  </body>
</html>
```

You now have a **SAML tester** that prints out the raw assertion from Azure AD B2C.

---

# 🎉 Wrapping Up

- You learned **how Electron works** (Main + Renderer processes).  
- You built **a PostgreSQL SQL Viewer** desktop app.  
- You integrated **OIDC Authentication** via Azure AD B2C.  
- You created a **SAML Tester** using an embedded Node.js server.  

Electron is a Swiss-army knife for building modern desktop apps, blending **frontend web dev comfort** with **backend Node.js power**.  
