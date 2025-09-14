# Guide to Create SSH Keys and Configure Passwordless Login with a Friendly Alias

This guide will walk you through the process of generating SSH keys on your local machine, setting up passwordless login to a remote SSH server, and creating a configuration file with a friendly alias for easy access.

---

## Step 1: Generate SSH Key Pair

1. **Open a Terminal**  
   Open your terminal or command prompt on your local machine.

2. **Generate the SSH Key Pair**  
   Run the following command to generate an SSH key pair (public and private keys):

   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   - `-t rsa`: Specifies the type of key to create (RSA).
   - `-b 4096`: Specifies the key size (4096 bits for stronger security).
   - `-C "your_email@example.com"`: Adds a comment (usually your email) to identify the key.

3. **Save the Key Files**  
   When prompted, specify the file location to save the keys. Press `Enter` to accept the default location (`~/.ssh/id_rsa`).

4. **Set a Passphrase (Optional)**
   You can set a passphrase for additional security. If you want passwordless login, leave it blank by pressing `Enter`.
   ```bash
   Enter passphrase (empty for no passphrase):
   Enter same passphrase again:
   ```
5. **Verify the Key Files**  
   After generation, two files will be created:  
   - `~/.ssh/id_rsa`: The private key (keep this secure).
   - `~/.ssh/id_rsa.pub`: The public key (used for authentication).
---
## Step 2: Copy the Public Key to the Remote Server

1. **Copy the Public Key**  
   Use the `ssh-copy-id` command to copy your public key to the remote server:
   ```bash
   ssh-copy-id ~/.ssh/id_rsa.pub username@remote_server_ip
   ```
   Replace `username` with your remote server username and `remote_server_ip` with the server's IP address or hostname.

2. **Authenticate Manually (First Time)**
   If this is your first time connecting to the server, you will be prompted to accept the server's fingerprint and enter your password. Enter your password when prompted.
   ```
   username@remote_server_ip's password:
   ```
3. **Verify the key is copied**  
   After successful copying, test the connection:
   ```bash
   ssh username@remote_server_ip
   ```
   If everything is set up correctly, you should log in without being prompted for a password.
---
## Step 3: Create an SSH Config file with a friendly alias

1. **Navigate to the `.ssh` Directory**  
   Ensure you are in the `.ssh` directory:
   ```bash
   cd ~/.ssh
   ```
2. **Create or Edit the config file**  
   Open the `config` file in your preferred text editor (e.g., `nano`, `vim`, `gedit`):
   ```bash
   gedit config
   ```
3. **Add Configuration for the Remote Server**  
   Add the following lines to the `config` file:
   ```plaintext
    Host friendly_alias
        HostName remote_server_ip
        User username
        IdentityFile ~/.ssh/id_rsa
   ```
   - Replace `friendly_alias` with a name you want to use to connect to the server.
   - Replace `remote_server_ip` with the server's IP address or hostname.
   - Replace `username` with your remote server username.
4. **Save and Exit**  
   `Save` the file and `exit` the editor. For gedit, press `Ctrl+S` to save and `Ctrl+X` to exit.
---
## Step 4: Test the Friendly Alias
1. **Connect using the `Alias`**  
   Use the alias to connect to the remote server:
   ```bash
   ssh friendly_alias
   ```
2. **Verify Passwordless Login**  
   You should now be logged into the remote server without entering a password.
---
## Additional Notes
- **Permissions** : Ensure the `.ssh` directory and its files have the correct permissions:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/config
```
- **Multiple Servers** : You can add multiple configurations to the `config` file, each with its own alias.
- **Backup Your Keys** : Keep a secure backup of your private key (`id_rsa`) in case you need to restore it on another machine.
---