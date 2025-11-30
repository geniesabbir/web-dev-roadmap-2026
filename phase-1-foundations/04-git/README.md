# Git & GitHub

**Duration:** 1 week

## Learning Objectives

By the end of this section, you will:
- Understand version control concepts
- Use Git confidently for daily development
- Collaborate using GitHub workflows
- Write meaningful commit messages
- Handle merge conflicts
- Use branching strategies

---

## Day 1: Git Fundamentals

### Topics
- What is version control?
- Git vs GitHub
- Installing and configuring Git
- Creating repositories
- The three states (working directory, staging, repository)

### Setup
```bash
# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main

# Create your first repo
mkdir my-project
cd my-project
git init
```

### Essential Commands
```bash
# Check status
git status

# Stage files
git add filename.txt    # Single file
git add .               # All files

# Commit changes
git commit -m "Your message"

# View history
git log
git log --oneline

# View changes
git diff                # Unstaged changes
git diff --staged       # Staged changes
```

### Exercise
1. Create a new repository
2. Create 3 files with content
3. Stage and commit each file separately
4. View the commit history

---

## Day 2: Branching & Merging

### Topics
- Why branches?
- Creating and switching branches
- Merging branches
- Deleting branches
- Understanding HEAD

### Commands
```bash
# Create and switch to branch
git branch feature-name
git checkout feature-name
# Or in one command:
git checkout -b feature-name

# List branches
git branch          # Local
git branch -a       # All (including remote)

# Switch branches
git checkout main
git switch main     # Modern alternative

# Merge a branch
git checkout main
git merge feature-name

# Delete a branch
git branch -d feature-name    # Safe delete
git branch -D feature-name    # Force delete
```

### Exercise: Feature Branch Workflow
1. Create a `feature/add-header` branch
2. Make changes and commit
3. Create another `feature/add-footer` branch from main
4. Make changes and commit
5. Merge both into main

---

## Day 3: Remote Repositories

### Topics
- Remote repositories
- Cloning repositories
- Pushing and pulling
- Fetch vs Pull
- Tracking branches

### Commands
```bash
# Clone a repository
git clone https://github.com/user/repo.git

# Add remote
git remote add origin https://github.com/user/repo.git

# View remotes
git remote -v

# Push to remote
git push origin main
git push -u origin main    # Set upstream

# Pull from remote
git pull origin main

# Fetch without merging
git fetch origin
```

### Exercise: GitHub Setup
1. Create a GitHub account (if needed)
2. Create a new repository on GitHub
3. Push your local project to GitHub
4. Make changes on GitHub (edit a file)
5. Pull the changes locally

---

## Day 4: Collaboration & Conflicts

### Topics
- Merge conflicts
- Resolving conflicts
- Pull requests
- Code reviews
- Forking repositories

### Handling Merge Conflicts
```bash
# When you see this after merge/pull:
# CONFLICT (content): Merge conflict in file.txt

# 1. Open the file and look for conflict markers:
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> branch-name

# 2. Manually edit to resolve

# 3. Stage the resolved file
git add file.txt

# 4. Complete the merge
git commit
```

### Exercise: Simulate a Conflict
1. Create two branches from main
2. Edit the same line in both branches
3. Merge one branch into main
4. Merge the second branch (conflict!)
5. Resolve the conflict

### Pull Request Workflow
1. Fork or clone the repository
2. Create a feature branch
3. Make changes and commit
4. Push to your fork/branch
5. Open a Pull Request
6. Address review feedback
7. Merge when approved

---

## Day 5: Best Practices

### Commit Message Convention
Follow the Conventional Commits format:

```
type(scope): description

[optional body]

[optional footer]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting (not CSS)
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance tasks

Examples:
```bash
git commit -m "feat(auth): add login functionality"
git commit -m "fix(api): handle null response from server"
git commit -m "docs: update README with setup instructions"
git commit -m "refactor(utils): simplify date formatting function"
```

### .gitignore
Create a `.gitignore` file:
```
# Dependencies
node_modules/

# Build output
dist/
build/

# Environment variables
.env
.env.local

# IDE
.vscode/
.idea/

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*

# Test coverage
coverage/
```

### Branching Strategies

**GitHub Flow (Simple):**
```
main
  └── feature/my-feature
  └── feature/another-feature
  └── fix/bug-fix
```

**Git Flow (Complex projects):**
```
main
  └── develop
        └── feature/my-feature
        └── release/v1.0
        └── hotfix/critical-bug
```

### Useful Aliases
Add to your `.gitconfig`:
```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"
```

---

## Advanced Commands Reference

```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Stash changes
git stash
git stash pop
git stash list

# Cherry-pick a commit
git cherry-pick <commit-hash>

# Rebase (use with caution)
git rebase main

# Interactive rebase
git rebase -i HEAD~3

# Amend last commit
git commit --amend

# View who changed each line
git blame filename.txt

# Find which commit introduced a bug
git bisect start
git bisect bad
git bisect good <commit>
```

---

## GitHub Features to Know

- **Issues**: Bug tracking and feature requests
- **Pull Requests**: Code review and merging
- **Actions**: CI/CD automation
- **Projects**: Kanban boards for project management
- **Discussions**: Community conversations
- **Pages**: Free static site hosting
- **Gists**: Share code snippets

---

## Exercise: Complete Git Workflow

Practice the full workflow:

1. Clone a repository
2. Create a feature branch
3. Make multiple commits with good messages
4. Push to remote
5. Create a pull request
6. (Simulate) Receive feedback
7. Make additional commits
8. Merge the PR
9. Pull changes to local main
10. Delete the feature branch

---

## Resources

### Documentation
- [Pro Git Book](https://git-scm.com/book/en/v2) (Free)
- [GitHub Docs](https://docs.github.com/)

### Interactive Learning
- [Learn Git Branching](https://learngitbranching.js.org/)
- [GitHub Skills](https://skills.github.com/)

### Cheat Sheets
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)

---

## Checklist Before Moving On

- [ ] Can initialize and clone repositories
- [ ] Comfortable with add, commit, push, pull
- [ ] Can create and merge branches
- [ ] Can resolve merge conflicts
- [ ] Understand pull request workflow
- [ ] Write meaningful commit messages
- [ ] Have a properly configured .gitignore
- [ ] GitHub profile is set up
- [ ] Pushed at least one project to GitHub

---

**Phase 1 Complete!**

Before moving to Phase 2, ensure you've:
- [ ] Built your portfolio website
- [ ] Deployed it to GitHub Pages
- [ ] Completed all section exercises
- [ ] Feel confident with HTML, CSS, JavaScript, TypeScript, and Git

**Next:** [Phase 2: Frontend Mastery](../../phase-2-frontend/README.md)
