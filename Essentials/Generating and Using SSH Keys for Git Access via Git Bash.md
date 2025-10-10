# Step-by-Step Guide: Generating and Using SSH Keys for Git Access via Git Bash

This guide walks you through generating an SSH key pair on your laptop using **Git Bash**, adding it to your SSH agent, and configuring your Git repository’s hosting service (like GitHub, GitLab, or Bitbucket) so that you can safely clone, pull, and push using SSH.

---

## 1. Open Git Bash

Launch **Git Bash** from your Start Menu or right-click in any folder and select “Git Bash Here”.

---

## 2. Check for Existing SSH Keys

Before creating a new one, check if an SSH key already exists.

    ls -al ~/.ssh

If you see files like `id_rsa` and `id_rsa.pub`, you already have a key pair. You may use those, or back them up and generate a new one.

---

## 3. Generate a New SSH Key Pair

Run the following command, replacing your real email address:

    ssh-keygen -t ed25519 -C "your_email@example.com"

If your system doesn’t support *ed25519*, use RSA instead:

    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

When prompted:

- **File to save the key:** Press `Enter` to accept the default (`~/.ssh/id_ed25519` or `~/.ssh/id_rsa`).
- **Passphrase (optional):** You may set one for added security or leave it blank.

---

## 4. Start the SSH Agent and Add Your Key

Ensure the SSH agent is running and add your key to it.

    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519

(Use your private key filename if it differs.)

---

## 5. Copy Your Public Key to Clipboard

Display your public key and copy it:

    cat ~/.ssh/id_ed25519.pub

Then manually select and copy the entire key (it begins with `ssh-ed25519` or `ssh-rsa` and ends with your email).

For convenience on Windows, you can also copy directly to the clipboard:

    clip < ~/.ssh/id_ed25519.pub

---

## 6. Add the SSH Key to Your Git Repository Hosting Service

### **For GitHub:**
- Go to **GitHub → Settings → SSH and GPG keys → New SSH key**.
- Paste your copied public key into the field.
- Give it a title (e.g., “My Laptop”) and save.

### **For GitLab:**
- Go to **User Settings → SSH Keys**, paste the key, set a title, and click **Add key**.

### **For Bitbucket:**
- Go to **Personal Settings → SSH Keys → Add Key**, then paste and save.

---

## 7. Test Your SSH Connection

Verify that your SSH key works by connecting to your Git host:

**GitHub:**
    
    ssh -T git@github.com

**GitLab:**
    
    ssh -T git@gitlab.com

You should get a success message like:  
“Hi *username*! You’ve successfully authenticated…”

---

## 8. Clone Your Repository Using SSH

When you clone a repository, make sure to use the SSH URL:

    git clone git@github.com:yourusername/your-repo.git

This ensures all subsequent `git pull` and `git push` commands use your secure SSH authentication.

---

## 9. Confirm Everything Works

You can now navigate into your cloned repo, make changes, and push:

    cd your-repo
    git add .
    git commit -m "Test commit"
    git push

If you see no authentication errors—congratulations, your SSH setup is working beautifully!

---

**Summary:**
- Generated SSH key pair with `ssh-keygen`
- Added key to SSH agent
- Registered public key with your Git hosting service
- Verified and successfully connected over SSH

That’s it — your laptop now talks to your Git repositories securely without needing passwords every time. Time to push some brilliant code!