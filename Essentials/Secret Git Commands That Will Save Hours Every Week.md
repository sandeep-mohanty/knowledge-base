# 10 Secret Git Commands That Will Save You 5+ Hours Every Week

Git is the backbone of modern software development, but it can also feel like a labyrinth of commands. While `git clone`, `git commit`, and `git push` are your bread and butter, there’s a whole world of Git commands that most developers never touch. These hidden gems can save you hours, solve tricky problems, and make you look like a Git wizard.

Let’s dive into **10 underrated Git commands** that will transform how you work with version control.

---

## 1. `git restore`: Your Undo Button for Git

Ever messed up a file or accidentally staged something? `git restore` is here to save the day.

```bash
# Discard changes in a file
git restore <file-name>

# Unstage a file (but keep the changes)
git restore --staged <file-name>
```
**Why it’s awesome :** Cleaner and more intuitive than `git checkout` or `git reset`. Think of it as Git’s version of Ctrl+Z.

## 2. `git switch`: A Smarter Way to Change Branches

Tired of using `git checkout` for everything? `git switch` is purpose-built for branch operations.

```bash
# Switch to an existing branch
git switch <branch-name>

# Create and switch to a new branch
git switch -c <new-branch-name>
```
**Why it’s awesome :** It’s like having a dedicated tool for branch management instead of a multi-purpose one.

## 3. `git sparse-checkout`: Work Smarter, Not Harder
Working in a massive monorepo? Use `git sparse-checkout` to check out only the files or directories you need.

```bash
# Enable sparse checkout
git sparse-checkout init --cone

# Add specific directories
git sparse-checkout set <dir1> <dir2>
```
**Why it’s awesome :** Perfect for large projects where you don’t need the entire repository. Think of it as cherry-picking files instead of branches.

## 4. `git range-diff`: Compare Commit Ranges Like a Pro
Need to compare two versions of a branch or patch series? `git range-diff` makes it easy.

```bash
git range-diff <commit-range-1> <commit-range-2>
```
**Why it’s awesome :** A game-changer for code reviews and rebasing workflows. No more squinting at diffs trying to figure out what changed.

## 5. `git notes`: Attach Metadata to Commits Without the Mess
Sometimes, a commit message isn’t enough. Use `git notes` to add context without cluttering the commit history.

```bash
# Add a note to a commit
git notes add -m "Your note here" <commit-hash>

# View notes
git log --show-notes
```
**Why it’s awesome :** Like leaving sticky notes on your commits—helpful, unobtrusive, and easy to manage.

## 6. `git worktree`: Work on Multiple Branches at Once
Switching branches back and forth is a pain. With `git worktree`, you can create separate directories for each branch.

```bash
# Create a new worktree for a branch
git worktree add ../new-directory <branch-name>

# List all worktrees
git worktree list
```
**Why it’s awesome :**  Like having multiple workspaces for your code. No more juggling branches—just parallel workflows.

## 7. `git bisect`: Find Bugs Like a Detective
Trying to pinpoint when a bug was introduced? `git bisect` performs a binary search through your commit history.

```bash
# Start bisect
git bisect start

# Mark a commit as bad
git bisect bad

# Mark a commit as good
git bisect good <commit-hash>

# Reset when done
git bisect reset
```
**Why it’s awesome :**  Like having a detective on your team, helping you track down bugs with surgical precision.

## 8. `git replace`: Rewrite History Without Breaking Things
Need to fix a historical commit without rebasing? `git replace` lets you create a replacement commit.
```bash
# Create a replacement commit
git replace <old-commit-hash> <new-commit-hash>
```
**Why it’s awesome :**  A non-destructive way to fix mistakes in your history. Think of it as a stealth edit for your commits.

## 9. `git fsck`: Find and Fix Repository Corruption
Worried about repository integrity? `git fsck` checks your repository for errors.
```bash
git fsck --full
```
**Why it’s awesome :** Your first line of defense against repository corruption. Think of it as Git’s version of a health check.

## 10. `git alias`: Create Your Own Git Commands
Tired of typing long commands? Use `git alias` to create shortcuts.
```bash
# Add an alias
git config --global alias.co checkout

# Use the alias
git co <branch-name>
```
**Why it’s awesome :**  Customize Git to fit your workflow perfectly. It’s like creating your own cheat codes for version control.

## Deeper Dive: Mastering `git bisect`
Let’s take a closer look at `git bisect`, one of the most powerful yet underused Git commands.

**Scenario**: 
- Your app is crashing.
- The bug wasn’t there last month, but it’s here now.
- You have hundreds of commits to sift through.

Instead of manually checking each commit, `git bisect` automates the process:
```bash
# Start the bisect session
git bisect start

# Mark the current commit as "bad"
git bisect bad

# Mark a known "good" commit (e.g., from last month)
git bisect good <commit-hash>

# Git will automatically check out a commit in the middle. Test your app and mark it as "good" or "bad":
git bisect good  # or git bisect bad

# Repeat until Git identifies the exact commit that introduced the bug.
```
**Pro Tip** : Automate this process by writing a script:
```bash
git bisect run ./test-script.sh
```

## Final Thoughts
Git is more than just commit, push, and pull. These hidden gems can save you time, solve complex problems, and make you a more efficient developer. Whether you’re debugging with git bisect, managing multiple branches with git worktree, or cleaning up history with git replace, these commands are your secret weapons.

So, the next time you’re stuck in a Git rabbit hole, remember: there’s probably a command for that. Happy coding, and may your merges always be conflict-free! 🚀
