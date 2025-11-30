# Day 1: Git Basics

## What is Git?

Git is a **distributed version control system** that tracks changes in your code over time. It allows you to:

- Save snapshots (commits) of your project at any point
- Go back to any previous version
- Work on multiple features simultaneously (branches)
- Collaborate with others without conflicts
- See who changed what and when

### Why Version Control?

```
Without Git:
project/
├── index.html
├── index_v2.html
├── index_final.html
├── index_final_v2.html
├── index_REALLY_final.html
└── index_REALLY_final_fixed.html  😱

With Git:
project/
└── index.html  (with full history of all changes)
```

### Git vs GitHub

| Git | GitHub |
|-----|--------|
| Version control software | Web hosting service for Git repos |
| Runs locally on your machine | Cloud-based platform |
| Command-line tool | Web interface + features |
| Free and open source | Free + paid features |
| Created by Linus Torvalds | Owned by Microsoft |

---

## Installing Git

### Check if Git is Installed

```bash
git --version
# git version 2.40.0 (or similar)
```

### Installation by Platform

**macOS:**
```bash
# Using Homebrew (recommended)
brew install git

# Or using Xcode Command Line Tools
xcode-select --install
```

**Windows:**
```bash
# Download from https://git-scm.com/download/win
# Or using winget
winget install Git.Git

# Or using Chocolatey
choco install git
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install git
```

**Linux (Fedora):**
```bash
sudo dnf install git
```

---

## Configuring Git

### Essential Configuration

```bash
# Set your identity (required for commits)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Verify configuration
git config --list

# Check specific setting
git config user.name
git config user.email
```

### Recommended Settings

```bash
# Set default branch name to 'main'
git config --global init.defaultBranch main

# Set default editor (VS Code)
git config --global core.editor "code --wait"

# Set default editor (Vim)
git config --global core.editor vim

# Enable colored output
git config --global color.ui auto

# Set line ending handling (important for cross-platform)
# On Windows:
git config --global core.autocrlf true
# On macOS/Linux:
git config --global core.autocrlf input

# Store credentials (avoid entering password repeatedly)
git config --global credential.helper store
# Or for macOS Keychain:
git config --global credential.helper osxkeychain
```

### Configuration Levels

```bash
# System level (all users)
git config --system setting.name value
# Stored in: /etc/gitconfig

# Global level (current user - most common)
git config --global setting.name value
# Stored in: ~/.gitconfig

# Local level (current repository only)
git config --local setting.name value
# Stored in: .git/config

# Priority: local > global > system
```

### View All Configuration

```bash
# Show all settings and where they come from
git config --list --show-origin

# Show global config file
cat ~/.gitconfig
```

---

## Creating a Repository

### Initialize a New Repository

```bash
# Navigate to your project folder
cd my-project

# Initialize Git
git init

# Output: Initialized empty Git repository in /path/to/my-project/.git/
```

This creates a hidden `.git` folder containing:
```
.git/
├── HEAD          # Points to current branch
├── config        # Local configuration
├── description   # Description (for GitWeb)
├── hooks/        # Scripts that run on events
├── info/         # Additional info
├── objects/      # All content (commits, files, etc.)
└── refs/         # Branch and tag references
```

### Clone an Existing Repository

```bash
# Clone with HTTPS
git clone https://github.com/username/repository.git

# Clone with SSH (requires SSH key setup)
git clone git@github.com:username/repository.git

# Clone into specific folder
git clone https://github.com/username/repository.git my-folder

# Clone specific branch
git clone --branch develop https://github.com/username/repository.git

# Shallow clone (only latest history - faster for large repos)
git clone --depth 1 https://github.com/username/repository.git
```

---

## The Three States of Git

Understanding these three states is crucial for using Git effectively:

```
┌─────────────────────────────────────────────────────────────┐
│                      Your Project                            │
├─────────────────┬─────────────────┬─────────────────────────┤
│  Working        │   Staging       │       Repository        │
│  Directory      │   Area          │       (.git)            │
│                 │   (Index)       │                         │
│  ┌──────────┐   │   ┌──────────┐  │   ┌──────────┐          │
│  │ file.txt │   │   │ file.txt │  │   │ commit 1 │          │
│  │ (edited) │   │   │ (staged) │  │   │ commit 2 │          │
│  └──────────┘   │   └──────────┘  │   │ commit 3 │          │
│                 │                 │   └──────────┘          │
└─────────────────┴─────────────────┴─────────────────────────┘
        │                  │                    ▲
        │    git add       │     git commit     │
        └─────────────────►└────────────────────┘
```

1. **Working Directory** - Your actual files on disk that you edit
2. **Staging Area (Index)** - Files prepared for the next commit
3. **Repository (.git)** - Permanent history of all commits

### File Lifecycle

```
Untracked ──git add──► Staged ──git commit──► Committed
     ▲                    │                       │
     │                    │ git restore --staged  │
     │◄───────────────────┘                       │
     │                                            │
     │         git restore (or edit file)         │
Modified ◄────────────────────────────────────────┘
```

---

## Basic Git Workflow

### Check Repository Status

```bash
git status

# Short status format
git status -s
# or
git status --short

# Output legend for short format:
# ?? = Untracked
# A  = Added (staged)
# M  = Modified
#  M = Modified but not staged
# MM = Modified, staged, then modified again
# D  = Deleted
```

### Tracking Files (git add)

```bash
# Add specific file
git add filename.txt

# Add multiple files
git add file1.txt file2.txt file3.txt

# Add all files in current directory
git add .

# Add all files matching pattern
git add *.js
git add src/*.ts

# Add all modified and deleted files (not new untracked)
git add -u

# Add all files (including untracked)
git add -A
# or
git add --all

# Interactive staging
git add -p
# This lets you choose which changes to stage
```

### Viewing Changes (git diff)

```bash
# Changes in working directory (not staged)
git diff

# Changes that are staged
git diff --staged
# or
git diff --cached

# Changes between working directory and last commit
git diff HEAD

# Changes in specific file
git diff filename.txt

# Show only names of changed files
git diff --name-only

# Show summary of changes
git diff --stat
```

### Committing Changes

```bash
# Commit with message
git commit -m "Add login feature"

# Commit with multi-line message
git commit -m "Add login feature" -m "This includes form validation and session handling"

# Open editor for detailed commit message
git commit

# Add all modified files AND commit (skips staging for tracked files)
git commit -am "Update styles"

# Amend the last commit (change message or add files)
git commit --amend -m "New message"

# Amend without changing message
git commit --amend --no-edit
```

### Viewing History (git log)

```bash
# Full commit history
git log

# Compact one-line format
git log --oneline

# Show last N commits
git log -5

# Show commits with diff
git log -p

# Show commit statistics
git log --stat

# Show graph of branches
git log --graph --oneline --all

# Customized format
git log --pretty=format:"%h - %an, %ar : %s"
# %h = short hash
# %an = author name
# %ar = relative date
# %s = subject (commit message)

# Search commits by message
git log --grep="fix bug"

# Search commits by author
git log --author="John"

# Filter by date
git log --since="2024-01-01" --until="2024-12-31"

# Show commits affecting specific file
git log -- filename.txt

# Show all branches
git log --all --graph --oneline
```

---

## Undoing Changes

### Discard Changes in Working Directory

```bash
# Discard changes in specific file (DANGEROUS - loses changes!)
git restore filename.txt

# Discard all changes in working directory
git restore .

# Old syntax (still works)
git checkout -- filename.txt
```

### Unstage Files

```bash
# Unstage specific file (keep changes in working directory)
git restore --staged filename.txt

# Unstage all files
git restore --staged .

# Old syntax
git reset HEAD filename.txt
```

### Undo Last Commit

```bash
# Undo commit, keep changes staged
git reset --soft HEAD~1

# Undo commit, unstage changes (default)
git reset HEAD~1
# or
git reset --mixed HEAD~1

# Undo commit, discard all changes (DANGEROUS!)
git reset --hard HEAD~1

# Undo specific number of commits
git reset HEAD~3  # Undo last 3 commits
```

### Revert a Commit (Safe for Shared History)

```bash
# Create new commit that undoes changes from a specific commit
git revert <commit-hash>

# Revert without automatically creating commit
git revert --no-commit <commit-hash>

# Revert last commit
git revert HEAD
```

### Difference Between Reset and Revert

```
git reset (rewrites history):
commit1 ── commit2 ── commit3 (HEAD)
                          │
                     git reset HEAD~1
                          ▼
commit1 ── commit2 (HEAD)
# commit3 is gone from history

git revert (adds new commit):
commit1 ── commit2 ── commit3 (HEAD)
                          │
                     git revert HEAD
                          ▼
commit1 ── commit2 ── commit3 ── revert-commit3 (HEAD)
# commit3 still in history, new commit undoes its changes
```

---

## Ignoring Files (.gitignore)

Create a `.gitignore` file in your repository root:

```gitignore
# Dependencies
node_modules/
vendor/
bower_components/

# Build output
dist/
build/
out/
.next/

# Environment and secrets
.env
.env.local
.env.*.local
*.pem
secrets.json

# IDE and editor files
.vscode/
.idea/
*.swp
*.swo
*~

# Operating system files
.DS_Store
Thumbs.db
desktop.ini

# Logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Testing
coverage/
.nyc_output/

# Temporary files
*.tmp
*.temp
.cache/

# Compiled files
*.class
*.pyc
*.o
*.so

# Specific file
config/local-settings.json

# All files with extension in any directory
**/*.bak

# But include this specific file
!important.bak
```

### Common .gitignore Patterns

```gitignore
# Ignore all .log files
*.log

# Ignore all files in build directory
build/

# Ignore files in any 'temp' directory
**/temp/

# Ignore files starting with dot
.*

# But NOT .gitignore itself
!.gitignore

# Ignore all .txt files except important.txt
*.txt
!important.txt

# Ignore all files in directory except specific one
logs/*
!logs/.gitkeep
```

### Check What's Being Ignored

```bash
# List ignored files
git status --ignored

# Check if specific file is ignored
git check-ignore -v filename.txt

# List all ignored files in repo
git ls-files --ignored --exclude-standard
```

### Global .gitignore

```bash
# Create global gitignore
touch ~/.gitignore_global

# Configure Git to use it
git config --global core.excludesfile ~/.gitignore_global

# Add common patterns to global gitignore
echo ".DS_Store" >> ~/.gitignore_global
echo "Thumbs.db" >> ~/.gitignore_global
echo "*.swp" >> ~/.gitignore_global
```

---

## Practice Exercises

### Exercise 1: Initialize and First Commit

```bash
# Create a new project directory
mkdir my-first-git-project
cd my-first-git-project

# Initialize Git
git init

# Create some files
echo "# My Project" > README.md
echo "console.log('Hello');" > app.js

# Check status
git status

# Stage files
git add .

# Commit
git commit -m "Initial commit: Add README and app.js"

# View history
git log
```

### Exercise 2: Making Changes

```bash
# Edit a file
echo "console.log('Hello, Git!');" > app.js

# Check what changed
git diff

# Stage and commit
git add app.js
git commit -m "Update greeting message"

# View history
git log --oneline
```

### Exercise 3: Working with .gitignore

```bash
# Create files
echo "SECRET=abc123" > .env
echo "test" > notes.txt
mkdir node_modules
touch node_modules/package.js

# Create .gitignore
cat > .gitignore << EOF
.env
node_modules/
*.log
EOF

# Check status (node_modules and .env should not appear)
git status

# Add and commit .gitignore
git add .gitignore notes.txt
git commit -m "Add notes and gitignore"
```

### Exercise 4: Undoing Changes

```bash
# Make a change
echo "bad code" >> app.js

# View the change
git diff

# Discard the change
git restore app.js

# Verify it's gone
git diff

# Make and stage a change
echo "more code" >> app.js
git add app.js

# Unstage it
git restore --staged app.js

# Check status
git status
```

---

## Key Git Commands Summary

| Command | Description |
|---------|-------------|
| `git init` | Initialize new repository |
| `git clone <url>` | Clone existing repository |
| `git status` | Check repository status |
| `git add <file>` | Stage file for commit |
| `git add .` | Stage all files |
| `git commit -m "msg"` | Commit staged changes |
| `git diff` | View unstaged changes |
| `git diff --staged` | View staged changes |
| `git log` | View commit history |
| `git log --oneline` | Compact history view |
| `git restore <file>` | Discard file changes |
| `git restore --staged <file>` | Unstage file |
| `git reset HEAD~1` | Undo last commit |
| `git revert <hash>` | Create reverting commit |

---

## Key Takeaways

1. **Git tracks changes** - Every commit is a snapshot of your project
2. **Three states** - Working directory → Staging area → Repository
3. **Configuration** - Set your name and email before committing
4. **.gitignore** - Exclude files you don't want tracked
5. **Commit often** - Small, focused commits are better than large ones
6. **Write good messages** - Future you will thank present you
7. **Check status** - Always know what state your files are in

---

## Self-Check Questions

1. What's the difference between Git and GitHub?
2. What are the three states in Git?
3. How do you stage a file?
4. How do you view uncommitted changes?
5. What does `.gitignore` do?
6. What's the difference between `git reset` and `git revert`?

---

**Next Lesson:** [Day 2 - Branching](./day-02-branching.md)
