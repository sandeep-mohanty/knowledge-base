# From Local to Remote: Hosting Code on GitHub

### Prerequisites

1. **Git Installed**: Ensure Git is installed on your machine. Download it from [git-scm.com](https://git-scm.com/).
2. **GitHub Account**: Create an account at [github.com](https://github.com/).
3. **Directory with Code and Documentation**: Have a local directory ready with your files.

---

### Step 1: Initialize a Git Repository in Your Local Directory

1. Open your terminal or command prompt.
2. Navigate to the directory:
   ```bash
   cd /path/to/your/directory
   ```
3. Initialize the directory as a Git repository:
    ```bash
    git init
    ```  
---
### Step 2: Add Files to the Staging Area

1. Check the status of your repository:  
    ```bash
    git status
    ```  
2. Add all files to the staging area:
    ```bash
    git add .
    ```  
    Alternatively, add specific files:

    ```bash
    git add <file_name>
    ```
---

### Step 3: Commit Your Changes

1. Commit the staged files:
   ```bash
   git commit -m "Initial commit"
   ```
---

### Step 4: Create a New Repository on GitHub

1. Log in to GitHub.
2. Click the + icon and select New repository .
3. Fill in the details:
   - Repository name: e.g., `my-project`
   - Description: Optional
   - Visibility: Public/Private  
4. Do not initialize with a README, `.gitignore`, or license.
5. Click Create repository .  
---  

### Step 5: Connect Your Local Repository to GitHub  
1. Copy the repository SSH URL:
   ```bash
   git@github.com:<username>/<repository-name>.git
   ```
2. Add the Remote Repository :  
   ```bash
   git remote add origin git@github.com:<username>/<repository-name>.git
   ```  
   Replace `<username>` and `<repository-name>` with your actual GitHub username and repository name.
3. Verify the Remote :  
   Confirm that the remote has been added correctly:  
   ```bash
   git remote -v
   ```  
   The output should look like this:
   ```bash
   origin  git@github.com:<username>/<repository-name>.git (fetch)
   origin  git@github.com:<username>/<repository-name>.git (push)
   ```  
4. Ensure SSH Key is Configured :  
   To use the Git protocol, you must have an SSH key configured on your machine and added to your GitHub account.  
---

### Step 6: Push Your Code to GitHub  
1. Push your code:
   ```bash
   git push -u origin main
   ```  
   If using `master` branch:
   ```bash
   git push -u origin master
   ```
---  

### Step 7: Add to the SSH Agent  

To ensure all your SSH keys are available for authentication:  

1. Start the SSH agent (if not already running):
```bash
eval "$(ssh-agent -s)"
```  
2. Add private key to the SSH agent:  
```bash  
ssh-add ~/.ssh/id_rsa_ado
ssh-add ~/.ssh/id_rsa_github
```  
3. Verify that the keys have been added:  
```bash  
ssh-add -l
``` 
You should see a list of the keys currently loaded in the SSH agent.

### Step 8: Verify the Push  
1. Go to your GitHub repository in the browser.
2. Refresh the page to confirm your files are uploaded.  
---

