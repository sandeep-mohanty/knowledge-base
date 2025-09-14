# Passwordless.dev + Angular + Node.js Tutorial  
*A simple demo app for registering, logging in with, and unregistering passkeys.*

This tutorial shows you how to build a minimal Angular + Node.js demo that lets a user:

- **Register a passkey** (WebAuthn).
- **Authenticate with a passkey** (login).
- **Unregister/delete passkeys** when needed.

> Uses [passwordless.dev](https://passwordless.dev), which wraps WebAuthn + FIDO2 into a friendly SDK.

---

## 1. Get Your Passwordless.dev API Keys

Every app needs:

- **Public API key** → used by the **frontend** (safe to expose).
- **Secret API key** → used only by the **backend** (keep safe!).

You can generate both on [https://passwordless.dev/](https://passwordless.dev/) after signing in and making an application entry.

⚠️ Never expose your secret key in Angular or any client app.

---

## 2. App Architecture

We’ll create:

- Angular frontend (collects user email and triggers flows).
- Express.js backend (makes secure API calls to passwordless.dev).

### Flows:
1. **Registration:**  
   Backend fetches `register/token` → frontend calls `client.register(token)` → passkey saved.
2. **Login:**  
   Backend fetches `signin/token` → frontend calls `client.signin({ token })` → user authenticated.
3. **Unregister/Delete:**  
   Backend uses credentials API → frontend calls an endpoint to delete credentials.

---

## 3. Angular Frontend

Initialize Angular:

    ng new passkey-demo
    cd passkey-demo
    ng add @angular/material

Install the official Passwordless SDK:

    npm install @passwordlessdev/passwordless-client

### `app.component.html`

    <div style="margin: 2rem;">
      <h1>Passwordless.dev Demo</h1>

      <mat-form-field appearance="outline">
        <mat-label>Email</mat-label>
        <input matInput [(ngModel)]="email" />
      </mat-form-field>

      <div style="margin-top: 1rem;">
        <button mat-raised-button color="primary" (click)="registerPasskey()">
          Register Passkey
        </button>

        <button mat-raised-button color="accent" (click)="signInPasskey()">
          Sign in with Passkey
        </button>

        <button mat-raised-button color="warn" (click)="unregisterPasskey()">
          Unregister Existing Passkey
        </button>
      </div>
    </div>

### `app.component.ts`

    import { Component } from '@angular/core';
    import { HttpClient } from '@angular/common/http';
    import * as Passwordless from '@passwordlessdev/passwordless-client';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html'
    })
    export class AppComponent {
      email: string = '';
      client: Passwordless.Client;

      constructor(private http: HttpClient) {
        // Initialize SDK with public key
        this.client = new Passwordless.Client({ apiKey: "YOUR_PUBLIC_API_KEY" });
      }

      async registerPasskey() {
        try {
          const { token } = await this.http.post<{ token: string }>(
            'http://localhost:3000/api/register-token',
            { email: this.email }
          ).toPromise();

          await this.client.register(token);
          alert('Passkey registered successfully!');
        } catch (e) {
          console.error(e);
          alert('Registration failed');
        }
      }

      async signInPasskey() {
        try {
          const { token } = await this.http.post<{ token: string }>(
            'http://localhost:3000/api/signin-token',
            { email: this.email }
          ).toPromise();

          const result = await this.client.signin({ token });
          console.log('Signin result:', result);
          alert(`Authentication successful for ${this.email}`);
        } catch (e) {
          console.error(e);
          alert('Signin failed');
        }
      }

      async unregisterPasskey() {
        try {
          await this.http.post(
            'http://localhost:3000/api/unregister-passkeys',
            { email: this.email }
          ).toPromise();

          alert('Passkeys removed for this email!');
        } catch (e) {
          console.error(e);
          alert('Unregister failed');
        }
      }
    }

### `app.module.ts`

    import { NgModule } from '@angular/core';
    import { BrowserModule } from '@angular/platform-browser';
    import { HttpClientModule } from '@angular/common/http';
    import { FormsModule } from '@angular/forms';
    import { AppComponent } from './app.component';

    @NgModule({
      declarations: [AppComponent],
      imports: [
        BrowserModule,
        HttpClientModule,
        FormsModule
      ],
      bootstrap: [AppComponent]
    })
    export class AppModule {}

---

## 4. Node.js Backend (Express)

Setup:

    mkdir backend
    cd backend
    npm init -y
    npm install express axios cors

Create `server.js`:

    const express = require('express');
    const axios = require('axios');
    const cors = require('cors');

    const app = express();
    app.use(cors());
    app.use(express.json());

    // Replace with your real keys
    const SECRET_API_KEY = "YOUR_SECRET_API_KEY";
    const PASSWORDLESS_BASE = "https://v4.passwordless.dev";

    // Registration token
    app.post('/api/register-token', async (req, res) => {
      const { email } = req.body;
      try {
        const response = await axios.post(
          `${PASSWORDLESS_BASE}/register/token`,
          { userId: email, username: email },
          { headers: { "ApiSecret": SECRET_API_KEY, "Content-Type": "application/json" } }
        );
        res.json({ token: response.data.token });
      } catch (err) {
        console.error(err.response?.data || err);
        res.status(500).json({ error: "Failed to get registration token" });
      }
    });

    // Sign-in token
    app.post('/api/signin-token', async (req, res) => {
      const { email } = req.body;
      try {
        const response = await axios.post(
          `${PASSWORDLESS_BASE}/signin/token`,
          { userId: email },
          { headers: { "ApiSecret": SECRET_API_KEY, "Content-Type": "application/json" } }
        );
        res.json({ token: response.data.token });
      } catch (err) {
        console.error(err.response?.data || err);
        res.status(500).json({ error: "Failed to get signin token" });
      }
    });

    // Unregister credentials
    app.post('/api/unregister-passkeys', async (req, res) => {
      const { email } = req.body;
      try {
        const list = await axios.get(
          `${PASSWORDLESS_BASE}/credentials?userId=${encodeURIComponent(email)}`,
          { headers: { "ApiSecret": SECRET_API_KEY } }
        );

        const credentials = list.data;

        for (const cred of credentials) {
          await axios.delete(
            `${PASSWORDLESS_BASE}/credentials/${cred.credentialId}`,
            { headers: { "ApiSecret": SECRET_API_KEY } }
          );
        }

        res.json({ success: true });
      } catch (err) {
        console.error(err.response?.data || err);
        res.status(500).json({ error: "Failed to unregister passkeys" });
      }
    });

    app.listen(3000, () => console.log('Backend running http://localhost:3000'));

---

## 5. Putting It Together

- Start backend:

      node server.js

- Start Angular frontend:

      ng serve

- Visit [http://localhost:4200](http://localhost:4200):

  - Enter email.
  - **Register Passkey** → Creates passkey prompt.
  - **Sign in with Passkey** → Authenticates via chosen method.
  - **Unregister Existing Passkey** → Deletes all credentials for that email.

---

## 6. Notes

- This demo uses `email` as `userId`. In production, map actual user IDs in your own DB.
- **Sign-in** result: The passwordless.dev service has already verified the WebAuthn challenge, so if the client’s `signin` promise resolves, you can trust authentication completed. Tie this into your session (JWT, cookie, etc.).
- **Unregister**: Current demo removes *all* passkeys for that email. Extend if you want selective deletion.

---

## 7. Summary

- **Registration**: `/register/token` → `client.register(token)`
- **Login**: `/signin/token` → `client.signin({ token })`
- **Unregister**: `/credentials/:id` → Delete credentials

That’s the full lifecycle of a passkey in Passwordless.dev using Angular + Node.js.