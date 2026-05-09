# Git Essentials Commands with Real-World Examples
**Master the Most Important Git Commands with Real-World Examples**

### 🔥 Basic GIT Description
Git is a powerful version control system that helps developers track changes, collaborate efficiently, and manage code with confidence. This cheat sheet covers the most essential Git commands you’ll use in daily development — from initializing a repository and committing changes to branching, merging, and resolving conflicts. Whether you’re a beginner or an experienced developer, this guide serves as a quick reference to streamline your workflow and avoid common mistakes.

### 🔥 Git Commands Cheat Sheet
Git is your **code time machine** — it tracks changes, helps teams collaborate, prevents conflicts, and allows you to revert mistakes easily.

Below is a simple and practical guide to the most important Git commands.



---

### 🟦 Basic Git Commands

**1. git init – Initialize a Git Repository**  
Creates a new Git repository in the current project folder.
```bash
git init
```

**2. git clone – Copy a Remote Repository**  
Downloads a complete copy of a repo from GitHub or any Git server.
```bash
git clone https://github.com/user/repo.git
```

**3. git status – View Working Directory Status**  
Shows changes you made, staged files, and untracked files.
```bash
git status
```

**4. git add – Stage Files for Commit**  
Moves files to the staging area.
```bash
git add file.txt
git add .              # add all changes
```

**5. git commit – Save Changes to Repo**  
Records staged changes.
```bash
git commit -m "Initial commit"
```

**6. git config – Configure Git Username & Email**  
Sets author identity for commits.

📌 **Example:**
```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

**7. git log – View Commit History**  
Shows commit messages, authors, and timestamps.
```bash
git log
```

**8. git show – Show Details of a Commit**  
Displays changes, metadata, and diffs for a commit.
```bash
git show <commit-hash>
```

**9. git diff – Compare Changes**  
Shows what changed before committing or between commits.
```bash
git diff             # unstaged changes
git diff --staged    # staged changes
```

**10. git reset – Unstage or Undo Commits**  
Undo staged files or move HEAD to a previous commit.
```bash
git reset HEAD file.txt
```

---

### 🟩 Branching & Merging



**11. git branch – List/Create Branches**
```bash
git branch                   # list branches
git branch feature-login     # create new branch
```

**12. git checkout – Switch Branches**  
Older method for switching branches.
```bash
git checkout feature-login
```

**13. git switch – Modern Branch Switch Command**
```bash
git switch feature-login
```

**14. git merge – Merge Branches**  
Combines one branch into another.
```bash
git merge feature-login
```

**15. git rebase – Reapply Commits**  
Cleans commit history when merging.
```bash
git rebase main
```



**16. git cherry-pick – Apply a Specific Commit**  
Used to apply a single commit from another branch.
```bash
git cherry-pick <commit-hash>
```

---

### 🟧 Remote Repository Commands

**17. git remote – Manage Remote URLs**  
Add or check remotes.
```bash
git remote add origin https://github.com/user/repo.git
```

**18. git push – Upload Changes to Remote**  
Sends commits to GitHub or other servers.
```bash
git push origin main
```

**19. git pull – Download & Merge Changes**  
Fetches remote updates and merges them.

📌 **Example:**
```bash
git pull origin main
```

**20. git fetch – Download Changes (No Merge)**  
Updates local metadata without affecting working files.
```bash
git fetch origin
```

**21. git remote -v – Show Remote URLs**  
Displays all connected remotes.
```bash
git remote -v
```

---

### 🟨 Stashing & Cleaning

**22. git stash – Save Uncommitted Work**  
Temporarily store changes without committing.

📌 **Example:**
```bash
git stash
```

**23. git stash pop – Restore Stashed Work**  
📌 **Example:**
```bash
git stash pop
```

**24. git stash list – View Stashes**
```bash
git stash list
```

**25. git clean – Remove Untracked Files**
```bash
git clean -f
```

---

### 🟪 Tagging

**26. git tag – Create a Tag**  
Usually used for release versions.
```bash
git tag -a v1.0 -m "Version 1.0"
```

**27. Delete a Tag**  
📌 **Example:**
```bash
git tag -d v1.0
```

**28. Push Tags to Remote**  
📌 **Example:**
```bash
git push origin --tags
```

---

### 🟥 Advanced Git Commands

**29. git bisect – Find Bug Introduced Commit**  
Binary search through commits to find where a bug started.
```bash
git bisect start
```

**30. git blame – Show Line-by-Line Authors**  
Shows who changed each line of a file.
```bash
git blame file.txt
```

**31. git reflog – View All Reference Logs**  
Shows all changes to HEAD (including deleted commits).
```bash
git reflog
```

**32. git submodule – Manage Submodules**  
Used when a project includes another Git repo inside it.
```bash
git submodule add https://github.com/user/repo.git
```

**33. git archive – Create a Zip Archive of Repo**  
📌 **Example:**
```bash
git archive --format=zip HEAD > archive.zip
```

**34. git gc – Garbage Collection**  
Cleans up unnecessary files and optimizes the repo.
```bash
git gc
```

---

### 🟦 GitHub-Specific (GH CLI) Commands

**35. gh auth login – Login to GitHub**
```bash
gh auth login
```

**36. gh repo clone – Clone Repo from GitHub**  
📌 **Example:**
```bash
gh repo clone user/repo
```

**37. gh issue list – List GitHub Issues**
```bash
gh issue list
```

**38. gh pr create – Create Pull Request**
```bash
gh pr create --title "New Feature" --body "Description of the feature"
```

**39. gh repo create – Create a New GitHub Repository**
```bash
gh repo create my-repo
```