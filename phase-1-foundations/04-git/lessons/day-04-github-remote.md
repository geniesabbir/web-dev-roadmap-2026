# Day 4: GitHub & Remote Repositories

## Understanding Remote Repositories

A remote repository is a version of your project hosted on the internet or a network (like GitHub, GitLab, or Bitbucket).

### Local vs Remote

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Computer                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Working Directory → Staging → Local Repository      │   │
│  └──────────────────────────────────────────────────────┘   │
│                            │                                 │
│                     git push │ git pull/fetch               │
│                            ▼                                 │
└───────────────────────────────────────────────────────────--┘
                             │
                    Internet │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    GitHub (Remote)                           │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Remote Repository                        │   │
│  │              (origin/main)                            │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## Setting Up GitHub

### Create a GitHub Account

1. Go to [github.com](https://github.com)
2. Click "Sign up"
3. Choose a username (this will be in your URLs)
4. Verify your email

### Configure Git with GitHub

```bash
# Set your GitHub username
git config --global user.name "YourGitHubUsername"

# Set your GitHub email (use the one from GitHub)
git config --global user.email "your-email@example.com"
```

### Authentication Methods

#### Option 1: Personal Access Token (HTTPS)

```bash
# 1. Go to GitHub → Settings → Developer Settings → Personal Access Tokens
# 2. Generate new token (classic)
# 3. Select scopes: repo, workflow (at minimum)
# 4. Copy the token

# When you push for the first time, use:
# Username: your-github-username
# Password: your-personal-access-token

# Store credentials
git config --global credential.helper store
# or on macOS:
git config --global credential.helper osxkeychain
```

#### Option 2: SSH Keys (Recommended)

```bash
# 1. Generate SSH key
ssh-keygen -t ed25519 -C "your-email@example.com"
# Press Enter for default file location
# Enter passphrase (optional but recommended)

# 2. Start SSH agent
eval "$(ssh-agent -s)"

# 3. Add key to agent
ssh-add ~/.ssh/id_ed25519

# 4. Copy public key
cat ~/.ssh/id_ed25519.pub
# Or on macOS:
pbcopy < ~/.ssh/id_ed25519.pub

# 5. Add to GitHub:
# GitHub → Settings → SSH and GPG Keys → New SSH Key
# Paste your public key

# 6. Test connection
ssh -T git@github.com
# Output: Hi username! You've successfully authenticated...
```

---

## Creating & Connecting Repositories

### Create Repository on GitHub

1. Click "+" → "New repository"
2. Enter repository name
3. Choose public or private
4. Optionally add README, .gitignore, license
5. Click "Create repository"

### Connect Local Repo to GitHub

```bash
# If you have a local repo and created empty repo on GitHub

# Add remote (HTTPS)
git remote add origin https://github.com/username/repo-name.git

# Add remote (SSH)
git remote add origin git@github.com:username/repo-name.git

# Push to GitHub
git push -u origin main
# -u sets upstream, so future pushes just need: git push
```

### Clone Existing Repository

```bash
# Clone with HTTPS
git clone https://github.com/username/repo-name.git

# Clone with SSH
git clone git@github.com:username/repo-name.git

# Clone into specific folder
git clone https://github.com/username/repo-name.git my-folder

# Clone specific branch
git clone -b develop https://github.com/username/repo-name.git
```

---

## Working with Remotes

### Managing Remote Connections

```bash
# List remotes
git remote
# Output: origin

# List remotes with URLs
git remote -v
# Output:
# origin  git@github.com:username/repo.git (fetch)
# origin  git@github.com:username/repo.git (push)

# Add a remote
git remote add upstream https://github.com/other/repo.git

# Remove a remote
git remote remove upstream

# Rename a remote
git remote rename origin github

# Change remote URL
git remote set-url origin git@github.com:username/new-repo.git

# Show remote details
git remote show origin
```

### Fetching vs Pulling

```bash
# FETCH: Download changes but don't merge
git fetch origin
# Updates origin/main but not your local main

# See what was fetched
git log main..origin/main

# Now merge if you want
git merge origin/main

# PULL: Fetch + Merge in one command
git pull origin main
# or just (if tracking is set)
git pull

# Pull with rebase (cleaner history)
git pull --rebase origin main
```

### Pushing Changes

```bash
# Push current branch to remote
git push origin main

# Push with upstream tracking
git push -u origin main
# Now future pushes just need: git push

# Push all branches
git push --all origin

# Push tags
git push --tags

# Force push (DANGEROUS - rewrites remote history)
git push --force origin main
# Safer force push (fails if remote has new commits)
git push --force-with-lease origin main

# Delete remote branch
git push origin --delete feature-branch
```

---

## Branch Tracking

### Understanding Tracking

```bash
# When you clone, main automatically tracks origin/main

# Check tracking
git branch -vv
# Output:
# * main     abc1234 [origin/main] Last commit message
#   feature  def5678 No tracking

# Set upstream for existing branch
git branch --set-upstream-to=origin/feature feature

# Or push with -u to set tracking
git push -u origin feature
```

### Working with Remote Branches

```bash
# List remote branches
git branch -r

# List all branches (local + remote)
git branch -a

# Create local branch from remote
git switch -c feature origin/feature
# or
git checkout -b feature origin/feature

# Track a remote branch
git branch -u origin/feature

# See which branches are tracking what
git branch -vv
```

---

## GitHub Features

### README.md

```markdown
# Project Name

Short description of what the project does.

## Installation

```bash
npm install my-project
```

## Usage

```javascript
const project = require('my-project');
project.doSomething();
```

## Features

- Feature 1
- Feature 2
- Feature 3

## Contributing

Pull requests are welcome. For major changes, please open an issue first.

## License

[MIT](https://choosealicense.com/licenses/mit/)
```

### Issues

```
Creating Issues:
1. Go to repo → Issues → New Issue
2. Add descriptive title
3. Describe the problem or feature request
4. Add labels (bug, enhancement, question)
5. Assign to someone (optional)
6. Add to project (optional)

Linking Issues in Commits:
git commit -m "Fix login bug

Fixes #42"

# Keywords that close issues:
# close, closes, closed, fix, fixes, fixed, resolve, resolves, resolved
```

### Pull Requests

```bash
# 1. Create feature branch
git switch -c feature/new-feature

# 2. Make changes and commit
git add .
git commit -m "Add new feature"

# 3. Push branch to GitHub
git push -u origin feature/new-feature

# 4. On GitHub:
#    - Click "Compare & pull request"
#    - Add title and description
#    - Request reviewers
#    - Add labels
#    - Click "Create pull request"

# 5. After PR is approved and merged:
git switch main
git pull
git branch -d feature/new-feature
```

### GitHub CLI (gh)

```bash
# Install GitHub CLI
# macOS:
brew install gh

# Windows:
winget install GitHub.cli

# Authenticate
gh auth login

# Common commands
gh repo create my-repo --public
gh repo clone owner/repo
gh pr create --title "My PR" --body "Description"
gh pr list
gh pr checkout 123
gh pr merge 123
gh issue create
gh issue list
```

---

## Forking Workflow

### What is Forking?

Forking creates your own copy of someone else's repository.

```
Original Repo (upstream)
        │
        │  Fork
        ▼
Your Fork (origin)
        │
        │  Clone
        ▼
Local Copy
```

### Fork and Contribute

```bash
# 1. Fork on GitHub (click "Fork" button)

# 2. Clone your fork
git clone git@github.com:YOUR-USERNAME/repo.git
cd repo

# 3. Add upstream remote
git remote add upstream https://github.com/ORIGINAL-OWNER/repo.git

# 4. Verify remotes
git remote -v
# origin    git@github.com:YOUR-USERNAME/repo.git (fetch)
# origin    git@github.com:YOUR-USERNAME/repo.git (push)
# upstream  https://github.com/ORIGINAL-OWNER/repo.git (fetch)
# upstream  https://github.com/ORIGINAL-OWNER/repo.git (push)

# 5. Create feature branch
git switch -c my-feature

# 6. Make changes and commit
git add .
git commit -m "Add my feature"

# 7. Push to YOUR fork
git push -u origin my-feature

# 8. Create Pull Request on GitHub
# From: YOUR-USERNAME:my-feature
# To: ORIGINAL-OWNER:main
```

### Keeping Fork Updated

```bash
# 1. Fetch upstream changes
git fetch upstream

# 2. Switch to main
git switch main

# 3. Merge upstream into your main
git merge upstream/main

# 4. Push to your fork
git push origin main

# Or rebase your feature branch
git switch my-feature
git rebase upstream/main
git push -f origin my-feature  # Force push needed after rebase
```

---

## GitHub Actions (CI/CD Basics)

### Simple Workflow Example

```yaml
# .github/workflows/ci.yml

name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Build
      run: npm run build
```

### Common Workflow Triggers

```yaml
on:
  # On push to specific branches
  push:
    branches: [ main, develop ]

  # On pull request
  pull_request:
    branches: [ main ]

  # On schedule (cron)
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight

  # Manual trigger
  workflow_dispatch:

  # On tag
  push:
    tags:
      - 'v*'
```

---

## Best Practices

### Commit Messages

```
Format:
<type>: <short description>

<longer description if needed>

<footer with issue references>

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation
- style: Formatting (no code change)
- refactor: Code restructure
- test: Adding tests
- chore: Maintenance

Examples:
feat: add user authentication

fix: resolve login redirect issue

Fixes #42

docs: update API documentation

refactor: simplify user service
```

### Repository Organization

```
my-project/
├── .github/
│   ├── workflows/        # GitHub Actions
│   ├── ISSUE_TEMPLATE/   # Issue templates
│   └── PULL_REQUEST_TEMPLATE.md
├── src/                  # Source code
├── tests/                # Tests
├── docs/                 # Documentation
├── .gitignore
├── README.md
├── LICENSE
├── package.json
└── CONTRIBUTING.md
```

### Protecting Main Branch

On GitHub:
1. Settings → Branches → Add rule
2. Branch name pattern: `main`
3. Enable:
   - Require pull request before merging
   - Require approvals (1 or more)
   - Require status checks to pass
   - Require branches to be up to date

---

## Practice Exercises

### Exercise 1: Create and Push to GitHub

```bash
# 1. Create new directory
mkdir github-practice && cd github-practice
git init

# 2. Create files
echo "# GitHub Practice" > README.md
echo "console.log('Hello');" > index.js
git add . && git commit -m "Initial commit"

# 3. Create repo on GitHub (github.com/new)

# 4. Connect and push
git remote add origin git@github.com:YOUR-USERNAME/github-practice.git
git push -u origin main
```

### Exercise 2: Clone and Modify

```bash
# 1. Clone a public repo
git clone https://github.com/octocat/Hello-World.git
cd Hello-World

# 2. Create a branch
git switch -c my-changes

# 3. Make changes
echo "My addition" >> README

# 4. Commit
git commit -am "Add my changes"

# 5. View status
git log --oneline
```

### Exercise 3: Fork Workflow

```bash
# 1. Fork a repo on GitHub
# 2. Clone YOUR fork
git clone git@github.com:YOUR-USERNAME/forked-repo.git
cd forked-repo

# 3. Add upstream
git remote add upstream https://github.com/ORIGINAL/repo.git

# 4. Create feature branch
git switch -c my-feature

# 5. Make changes, commit, push
echo "change" >> file.txt
git commit -am "My contribution"
git push -u origin my-feature

# 6. Create PR on GitHub
```

### Exercise 4: Sync Fork

```bash
# 1. Fetch upstream
git fetch upstream

# 2. Merge upstream main
git switch main
git merge upstream/main

# 3. Push to your fork
git push origin main

# 4. Rebase feature branch
git switch my-feature
git rebase main
```

---

## Key Commands Summary

| Command | Description |
|---------|-------------|
| `git remote add <name> <url>` | Add remote |
| `git remote -v` | List remotes |
| `git clone <url>` | Clone repository |
| `git fetch <remote>` | Download remote changes |
| `git pull` | Fetch and merge |
| `git push` | Upload local commits |
| `git push -u origin <branch>` | Push with tracking |
| `git push --force-with-lease` | Safe force push |
| `git branch -r` | List remote branches |
| `git remote show origin` | Show remote details |

---

## Key Takeaways

1. **Remote = copy on server** - Usually GitHub, GitLab, or Bitbucket
2. **origin** - Default name for your remote
3. **upstream** - Convention for original repo (when forking)
4. **Fetch vs Pull** - Fetch downloads, pull downloads + merges
5. **SSH is easier** - Set it up once, never enter credentials again
6. **Fork for contribution** - Fork, clone, branch, push, PR
7. **Protect main** - Use branch protection rules

---

## Self-Check Questions

1. What's the difference between `git fetch` and `git pull`?
2. How do you add a remote repository?
3. What does `-u` do in `git push -u origin main`?
4. How do you authenticate with GitHub?
5. What's the difference between origin and upstream?
6. How do you sync a fork with the original repository?

---

**Next Lesson:** [Day 5 - Collaboration Workflow](./day-05-collaboration-workflow.md)
