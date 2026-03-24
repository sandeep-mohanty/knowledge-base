# I Collected Every Useful Git Commands You Need to Know

Git is one of the most used tools in Software Development. This article contains all the git commands you will need for effective collaboration and project management.

---

## Getting Started with Git

### Configuring Username and Password
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```
The command above configures the username and email to be used for all Git repos on your local machine.

You can check everything in the Global Configuration of Git using the following command:
```bash
git config --global --list
```

### Creating and Cloning Repositories
```bash
git init                          # Initialize a new Git repository
git clone <repository-url>        # Clone an existing repository
git clone --depth 1 <repo-url>    # Shallow clone with only the latest commit
```

---

## Daily Use Git Commands

### Checking Status and History
```bash
git status                        # View the state of your working directory
git log                           # View commit history
git log --oneline                 # Condensed commit history
git log --graph --decorate        # Visual representation of branch history
git blame <file>                  # See who changed which lines in a file
```

### Staging Changes and Committing


```bash
git add <file>                    # Stage specific file
git add .                         # Stage all changes
git add -p                        # Interactively stage changes
git commit -m "commit message"    # Commit staged changes
git commit -am "commit message"   # Stage and commit all changes in tracked files
```

---

## Commands for Remote Repositories
```bash
git remote add origin <repo-url>  # Add remote repository
git remote -v                     # List remote repositories
git fetch                         # Download changes from remote
git pull                          # Fetch and merge remote changes
git push                          # Upload local commits to remote
git push -u origin <branch>       # Set up tracking and push branch
```

---

## Managing Branches

### Creating Branches
```bash
git branch                        # List local branches
git branch -a                     # List all branches (local and remote)
git branch <branch-name>          # Create new branch
git checkout <branch-name>        # Switch to a branch
git checkout -b <branch-name>     # Create and switch to new branch
git switch <branch-name>          # Switch to a branch (new Git syntax)
git switch -c <branch-name>       # Create and switch to new branch (new Git syntax)
```

### Merging and Rebasing


```bash
git merge <branch>                # This is used to merged branch to current branch
git merge --no-ff <branch>        # Create merge commit even if fast-forward is possible
git rebase <branch>               # Rebase current branch onto another branch
git rebase -i HEAD~n              # Interactive rebase of last n commits
```

### Branch Cleanup
```bash
git branch -d <branch-name>       # Delete local branch (safe)
git branch -D <branch-name>       # Force delete local branch
git push origin --delete <branch> # Delete remote branch
git remote prune origin           # Remove references to deleted 
                                  # remote branches
```

---

## Other Git Commands

### Undo Changes
This is a significant use case of working with Git. It helps us undo the mistakes we make.

```bash
git restore <file>              # This is used to undo changes in a file before staging
git restore --staged <file>     # This removes the file from staging area
git reset <commit>              # This command will move the HEAD to a commit without removing the files.
git reset --hard <commit>       # Command used to reset everything to a commit and delete changes
git revert <commit>             # To create new commit that cancels previous commit
git clean -fd                   # To remove untracked files and folders
```

### Stashing Work
Using stash, we can temporarily save messy changes without committing.

```bash
git stash         # Command to save current uncommitted changes
git stash list    # To list saved stashes
git stash apply   # This is used to reapply the last stash
git stash pop     # Apply and remove last stash from stash list
git stash drop    # To delete a stash
git stash clear   # Remove all stashes
```

---

## Git Diff Commands
These commands are used to show differences in files.

```bash
git diff                      # Differences in working directory compared to last commit
git diff --staged             # Differences between staged and committed changes
git diff <branch1> <branch2>  # Differences between two branches
git diff <commit1> <commit2>  # Differences between two commits
```

---

## Git Pull vs Git Fetch



| Feature | `git fetch` | `git pull` |
| :--- | :--- | :--- |
| **Action** | Downloads remote changes but does not integrate them. | Downloads remote changes and immediately merges them. |
| **Safety** | High (gives you a chance to review). | Low (can cause immediate merge conflicts). |
| **Command** | `git fetch origin` | `git pull origin <branch>` |

These are some **useful git commands** you should be aware of as a developer. Even if you are a solo developer, it is nice to practice using a version control system like Git to save and track your work.