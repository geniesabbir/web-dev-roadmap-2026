# Day 2: Git Branching

## Understanding Branches

A branch is simply a pointer to a specific commit. Branches allow you to work on different features or experiments without affecting the main codebase.

### How Branches Work

```
                    feature-branch (HEAD)
                         │
                         ▼
                    ┌─────────┐
                    │ commit4 │
                    └────┬────┘
                         │
main ──────────►    ┌─────────┐
                    │ commit3 │
                    └────┬────┘
                         │
                    ┌─────────┐
                    │ commit2 │
                    └────┬────┘
                         │
                    ┌─────────┐
                    │ commit1 │
                    └─────────┘
```

- **main**: The default branch (was called "master" historically)
- **HEAD**: Points to your current location (current branch + commit)
- Each branch is an independent line of development

### Why Use Branches?

1. **Feature Development** - Work on new features without affecting main code
2. **Bug Fixes** - Create hotfix branches for urgent issues
3. **Experimentation** - Try new ideas safely
4. **Team Collaboration** - Multiple people work simultaneously
5. **Code Review** - Review changes before merging to main

---

## Creating Branches

### Create a New Branch

```bash
# Create new branch (stay on current branch)
git branch feature-login

# Create and switch to new branch
git checkout -b feature-login

# Modern syntax (Git 2.23+)
git switch -c feature-login

# Create branch from specific commit
git branch feature-login abc1234

# Create branch from another branch
git branch feature-login develop
```

### Listing Branches

```bash
# List local branches (* marks current branch)
git branch
# Output:
#   develop
# * main
#   feature-login

# List all branches (including remote)
git branch -a

# List remote branches only
git branch -r

# List branches with last commit info
git branch -v

# List branches with tracking info
git branch -vv

# List merged branches
git branch --merged

# List unmerged branches
git branch --no-merged
```

---

## Switching Branches

### Change Current Branch

```bash
# Switch to existing branch
git checkout feature-login

# Modern syntax (Git 2.23+)
git switch feature-login

# Switch to previous branch
git checkout -
git switch -

# Switch and discard local changes (DANGEROUS)
git checkout -f feature-login
```

### What Happens When You Switch?

```
Before switching (on main):
Working Directory has files from main

After: git switch feature-login
Working Directory now has files from feature-login
```

### Handling Uncommitted Changes

```bash
# If you have uncommitted changes, Git will:
# 1. Allow switch if changes don't conflict with target branch
# 2. Block switch if changes would be overwritten

# Options when blocked:
# 1. Commit the changes
git commit -am "WIP: save progress"

# 2. Stash the changes (temporary storage)
git stash
git switch feature-login
git stash pop  # Restore changes

# 3. Discard changes (DANGEROUS)
git restore .
git switch feature-login
```

---

## Renaming and Deleting Branches

### Rename Branches

```bash
# Rename current branch
git branch -m new-name

# Rename specific branch
git branch -m old-name new-name

# Rename current branch (force if branch exists)
git branch -M new-name
```

### Delete Branches

```bash
# Delete merged branch
git branch -d feature-login

# Force delete (even if not merged) - DANGEROUS
git branch -D feature-login

# Delete remote branch
git push origin --delete feature-login
# or
git push origin :feature-login

# Prune stale remote-tracking branches
git fetch --prune
# or
git remote prune origin
```

---

## HEAD and Branch Pointers

### Understanding HEAD

```bash
# HEAD points to current branch (or commit if detached)
cat .git/HEAD
# Output: ref: refs/heads/main

# See where HEAD points
git log --oneline -1 HEAD

# HEAD~1 means "parent of HEAD"
git log --oneline -1 HEAD~1

# HEAD~2 means "grandparent of HEAD"
git log --oneline -1 HEAD~2

# HEAD^1 also means parent (for merge commits, ^1 and ^2 are different parents)
```

### Detached HEAD State

```bash
# Checkout specific commit (detached HEAD)
git checkout abc1234

# You're now in "detached HEAD" state
# Any commits made here won't belong to any branch

# To save work in detached HEAD:
git switch -c new-branch-name

# To return to a branch:
git switch main
```

---

## Branch Workflow Examples

### Feature Branch Workflow

```bash
# 1. Start on main branch
git switch main

# 2. Pull latest changes
git pull

# 3. Create feature branch
git switch -c feature/user-profile

# 4. Work on feature (multiple commits)
# ... edit files ...
git add .
git commit -m "Add user profile component"

# ... more edits ...
git add .
git commit -m "Add profile picture upload"

# 5. Push feature branch to remote
git push -u origin feature/user-profile

# 6. Create Pull Request on GitHub

# 7. After PR is merged, clean up
git switch main
git pull
git branch -d feature/user-profile
```

### Hotfix Workflow

```bash
# 1. Start from main (production branch)
git switch main
git pull

# 2. Create hotfix branch
git switch -c hotfix/critical-bug

# 3. Fix the bug
# ... fix code ...
git add .
git commit -m "Fix critical security vulnerability"

# 4. Push and create PR
git push -u origin hotfix/critical-bug

# 5. After merge, update main and tag
git switch main
git pull
git tag -a v1.0.1 -m "Security patch"
git push --tags

# 6. Merge hotfix into develop (if using GitFlow)
git switch develop
git merge hotfix/critical-bug
git push

# 7. Clean up
git branch -d hotfix/critical-bug
git push origin --delete hotfix/critical-bug
```

---

## Viewing Branch Information

### Compare Branches

```bash
# Commits in feature-branch not in main
git log main..feature-branch

# Commits in main not in feature-branch
git log feature-branch..main

# All commits different between branches
git log main...feature-branch

# Compare files between branches
git diff main..feature-branch

# Compare specific file
git diff main..feature-branch -- path/to/file.js

# Show files changed between branches
git diff --name-only main..feature-branch

# Show summary
git diff --stat main..feature-branch
```

### Find Branch Origin

```bash
# Find where branch diverged from main
git merge-base main feature-branch

# Show commits since branching from main
git log $(git merge-base main feature-branch)..feature-branch
```

### Branch Visualization

```bash
# ASCII graph of all branches
git log --graph --oneline --all

# More detailed graph
git log --graph --oneline --all --decorate

# Pretty graph with colors
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --all
```

---

## Stashing Changes

Stash lets you save uncommitted changes temporarily.

### Basic Stash Operations

```bash
# Save current changes to stash
git stash

# Save with message
git stash save "WIP: login form"
# or (newer syntax)
git stash push -m "WIP: login form"

# Include untracked files
git stash -u
# or
git stash --include-untracked

# Include all files (even ignored)
git stash -a
# or
git stash --all
```

### Viewing Stashes

```bash
# List all stashes
git stash list
# Output:
# stash@{0}: WIP on main: abc1234 Last commit message
# stash@{1}: On feature: def5678 Another message

# Show stash contents
git stash show
# Show with diff
git stash show -p
# Show specific stash
git stash show stash@{1}
```

### Applying Stashes

```bash
# Apply most recent stash (keep in stash list)
git stash apply

# Apply and remove from stash list
git stash pop

# Apply specific stash
git stash apply stash@{1}
git stash pop stash@{2}

# Apply stash to different branch
git switch other-branch
git stash apply
```

### Managing Stashes

```bash
# Drop specific stash
git stash drop stash@{0}

# Clear all stashes (DANGEROUS)
git stash clear

# Create branch from stash
git stash branch new-branch-name stash@{0}
```

### Stash Workflow Example

```bash
# You're working on feature-A but need to fix urgent bug

# 1. Save current work
git stash push -m "WIP: feature-A user form"

# 2. Switch to main and create hotfix
git switch main
git switch -c hotfix/urgent-bug
# ... fix bug ...
git commit -am "Fix urgent bug"
git push

# 3. Return to feature work
git switch feature-A
git stash pop

# 4. Continue working
# ... your changes are back ...
```

---

## Common Branch Naming Conventions

### Prefixed Branch Names

```
feature/      - New features
  feature/user-authentication
  feature/shopping-cart
  feature/dark-mode

bugfix/       - Bug fixes
  bugfix/login-error
  bugfix/cart-calculation

hotfix/       - Urgent production fixes
  hotfix/security-patch
  hotfix/payment-crash

release/      - Release preparation
  release/v2.0.0
  release/2024-q1

docs/         - Documentation changes
  docs/api-reference
  docs/readme-update

refactor/     - Code refactoring
  refactor/auth-module
  refactor/database-layer

test/         - Testing related
  test/e2e-coverage
  test/unit-tests
```

### Including Issue Numbers

```
feature/123-user-login
bugfix/456-fix-validation
issue/789-update-styles
```

### Branch Naming Best Practices

1. **Use lowercase** - `feature/login` not `Feature/Login`
2. **Use hyphens** - `user-profile` not `user_profile` or `userProfile`
3. **Be descriptive** - `feature/add-payment-gateway` not `feature/new-feature`
4. **Keep it short** - Under 50 characters if possible
5. **Include ticket numbers** - `feature/JIRA-123-login`

---

## Practice Exercises

### Exercise 1: Basic Branching

```bash
# Create a new project
mkdir branch-practice && cd branch-practice
git init
echo "# Branch Practice" > README.md
git add . && git commit -m "Initial commit"

# Create and switch to new branch
git switch -c feature/greeting

# Make changes
echo "console.log('Hello!');" > app.js
git add . && git commit -m "Add greeting"

# View branches
git branch -v

# Switch back to main
git switch main

# View log graph
git log --oneline --all --graph
```

### Exercise 2: Multiple Branches

```bash
# Create two feature branches from main
git switch main
git switch -c feature/header
echo "<header>My Header</header>" > header.html
git add . && git commit -m "Add header"

git switch main
git switch -c feature/footer
echo "<footer>My Footer</footer>" > footer.html
git add . && git commit -m "Add footer"

# View all branches
git log --oneline --all --graph

# List branches with info
git branch -v
```

### Exercise 3: Stashing

```bash
# Make changes without committing
git switch main
echo "work in progress" >> README.md

# Try to switch (may fail)
git switch feature/header

# Stash changes
git stash push -m "WIP: readme update"

# Now switch works
git switch feature/header

# View stash
git stash list

# Return to main and restore
git switch main
git stash pop

# Verify changes are back
cat README.md
```

### Exercise 4: Clean Up Branches

```bash
# Merge a branch (we'll cover merge in detail tomorrow)
git switch main
git merge feature/header

# Delete merged branch
git branch -d feature/header

# Try to delete unmerged branch
git branch -d feature/footer  # Will fail

# Force delete
git branch -D feature/footer

# View remaining branches
git branch
```

---

## Key Commands Summary

| Command | Description |
|---------|-------------|
| `git branch` | List local branches |
| `git branch <name>` | Create new branch |
| `git branch -d <name>` | Delete merged branch |
| `git branch -D <name>` | Force delete branch |
| `git branch -m <new>` | Rename current branch |
| `git switch <name>` | Switch to branch |
| `git switch -c <name>` | Create and switch |
| `git checkout <name>` | Switch (older syntax) |
| `git checkout -b <name>` | Create and switch (older) |
| `git log --graph --all` | Visualize branches |
| `git stash` | Stash changes |
| `git stash pop` | Apply and remove stash |
| `git stash list` | List stashes |

---

## Key Takeaways

1. **Branches are lightweight** - Just pointers to commits
2. **Branch often** - For features, fixes, experiments
3. **Keep main clean** - Don't commit directly to main
4. **Name branches well** - Use conventions like `feature/`, `bugfix/`
5. **Use stash** - Save WIP when switching branches
6. **Clean up** - Delete merged branches
7. **HEAD** - Your current position in the repo

---

## Self-Check Questions

1. What is a branch in Git?
2. How do you create a new branch and switch to it in one command?
3. What's the difference between `git branch -d` and `git branch -D`?
4. What is a detached HEAD state?
5. How do you save uncommitted changes temporarily?
6. What naming convention would you use for a bug fix branch?

---

**Next Lesson:** [Day 3 - Merging & Conflicts](./day-03-merging-conflicts.md)
