# Day 5: Collaboration Workflow

## Git Workflow Models

Different teams use different workflows based on their needs. Here are the most common ones.

### Feature Branch Workflow

The simplest workflow for teams.

```
main ────●────●────●────●────●────●────●
              \         /         /
               ●───────●         /
               feature-A        /
                    \          /
                     ●────●───●
                     feature-B

Rules:
1. main is always deployable
2. Create branch for each feature
3. Merge via Pull Request
4. Delete branch after merge
```

```bash
# Daily workflow
git switch main
git pull

# Start new feature
git switch -c feature/user-profile

# Work and commit
git add .
git commit -m "Add user profile component"

# Push and create PR
git push -u origin feature/user-profile
# Create PR on GitHub

# After PR is merged
git switch main
git pull
git branch -d feature/user-profile
```

### GitFlow Workflow

More structured workflow for larger projects with scheduled releases.

```
main     ────●─────────────────●─────────────────●──── (releases only)
              \               /                 /
develop  ────●────●────●────●────●────●────●──●────── (integration)
                   \     /        \     /
                    ●───●          ●───●
                  feature-A      feature-B

Additional branches:
- release/* : Prepare releases
- hotfix/*  : Emergency production fixes
```

```bash
# Feature development
git switch develop
git pull
git switch -c feature/new-feature
# ... work ...
git push -u origin feature/new-feature
# Create PR to develop

# Preparing release
git switch develop
git pull
git switch -c release/v1.2.0
# ... final testing, version bumps ...
git push -u origin release/v1.2.0
# When ready, merge to main AND develop

# Hotfix (from main)
git switch main
git pull
git switch -c hotfix/critical-bug
# ... fix ...
git push -u origin hotfix/critical-bug
# Merge to main AND develop
```

### Trunk-Based Development

Simple workflow where everyone works on main (or short-lived branches).

```
main ────●────●────●────●────●────●────●────●────●
         │    │    │    │    │    │    │    │    │
         1hr  1hr  1hr  1hr  1hr  1hr  1hr  1hr  1hr
              │
              └── Small commits, multiple times per day

Or with short-lived branches:
main ────●────●─────●────●─────●────●
              \    /     \    /
               ●──●       ●──●
           (1-2 days)  (1-2 days)
```

```bash
# Option 1: Direct commits (small teams)
git switch main
git pull
# ... make small change ...
git commit -am "Small focused change"
git push

# Option 2: Short-lived branches
git switch main
git pull
git switch -c quick-fix
# ... work for a few hours to a day ...
git push -u origin quick-fix
# Create PR, merge quickly, delete branch
```

---

## Code Review Process

### Creating a Good Pull Request

```markdown
## Description
Brief description of what this PR does.

## Changes
- Added user authentication
- Created login/register forms
- Integrated JWT tokens

## Screenshots (if UI changes)
[Before/After screenshots]

## Testing
- [ ] Unit tests pass
- [ ] Manual testing completed
- [ ] Works on mobile

## Related Issues
Closes #123
Related to #456

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-reviewed the code
- [ ] Added/updated tests
- [ ] Updated documentation
```

### Reviewing Code

```bash
# Fetch PR locally
gh pr checkout 123
# or
git fetch origin pull/123/head:pr-123
git switch pr-123

# Review the changes
git diff main...pr-123

# Test locally
npm install
npm test
npm start

# Leave review on GitHub
# - Approve
# - Request changes
# - Comment
```

### Review Best Practices

**For Reviewers:**
- Be constructive and kind
- Ask questions instead of making demands
- Focus on logic, not style (use linters for style)
- Test the code locally
- Approve quickly for small changes

**For Authors:**
- Keep PRs small (<400 lines ideally)
- Respond to all comments
- Don't take feedback personally
- Explain complex changes in PR description

---

## Handling Common Scenarios

### Scenario 1: Updating Your Branch with Latest Main

```bash
# Method 1: Merge main into your branch
git switch feature/my-feature
git fetch origin
git merge origin/main
# Resolve conflicts if any
git push

# Method 2: Rebase (cleaner history)
git switch feature/my-feature
git fetch origin
git rebase origin/main
# Resolve conflicts if any
git push --force-with-lease
```

### Scenario 2: Someone Else Pushed to Your PR Branch

```bash
# Always pull before pushing
git switch feature/shared-branch
git pull
# ... make your changes ...
git push

# If you get rejection error:
git pull --rebase
git push
```

### Scenario 3: Squashing Commits Before Merge

```bash
# Interactive rebase to squash commits
git switch feature/my-feature
git rebase -i HEAD~5  # Last 5 commits

# In editor, change 'pick' to 'squash' (or 's') for commits to combine:
# pick abc1234 First commit
# squash def5678 Second commit
# squash ghi9012 Third commit

# Save and close, then edit combined commit message
git push --force-with-lease
```

### Scenario 4: Accidentally Committed to Wrong Branch

```bash
# If you committed to main instead of feature branch

# 1. Create the branch you wanted
git branch feature/my-feature

# 2. Reset main back
git reset --hard HEAD~1  # Remove last commit from main

# 3. Switch to feature branch (your commit is there)
git switch feature/my-feature
```

### Scenario 5: Need to Undo a Pushed Commit

```bash
# Option 1: Revert (safe - creates new commit)
git revert abc1234
git push

# Option 2: Reset and force push (ONLY if not shared)
git reset --hard HEAD~1
git push --force-with-lease
```

---

## Team Conventions

### Branch Naming

```bash
# Feature branches
feature/user-authentication
feature/JIRA-123-add-search
feat/dark-mode

# Bug fixes
bugfix/login-error
fix/cart-calculation
bug/JIRA-456-missing-validation

# Hotfixes (production)
hotfix/security-patch
hotfix/payment-crash

# Other
docs/update-readme
refactor/auth-module
test/add-unit-tests
chore/update-dependencies
```

### Commit Message Convention

```bash
# Conventional Commits format
<type>(<scope>): <subject>

<body>

<footer>

# Examples
feat(auth): add JWT token refresh

Implemented automatic token refresh when
access token expires. Refresh happens
transparently without user action.

Closes #123

# Types
feat     : New feature
fix      : Bug fix
docs     : Documentation only
style    : Formatting, white-space
refactor : Code restructuring
perf     : Performance improvement
test     : Adding tests
chore    : Maintenance
ci       : CI/CD changes
build    : Build system changes
```

### Commit Message Template

```bash
# Create template file
cat > ~/.gitmessage << 'EOF'
# <type>(<scope>): <subject>
# |<----  Using a maximum of 50 characters  ---->|

# Explain why this change is being made
# |<----   Try to limit each line to 72 characters   ---->|

# Provide links or keys to any relevant tickets, articles or other resources
# Example: Fixes #23

# --- COMMIT END ---
# Type can be:
#   feat     (new feature)
#   fix      (bug fix)
#   docs     (documentation)
#   style    (formatting)
#   refactor (restructuring)
#   perf     (performance)
#   test     (tests)
#   chore    (maintenance)
EOF

# Configure Git to use it
git config --global commit.template ~/.gitmessage
```

---

## CI/CD Integration

### GitHub Actions for PR Validation

```yaml
# .github/workflows/pr-check.yml
name: PR Validation

on:
  pull_request:
    branches: [main, develop]

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

      - name: Test
        run: npm test -- --coverage

      - name: Build
        run: npm run build
```

### Auto-labeling PRs

```yaml
# .github/workflows/labeler.yml
name: Labeler

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v4
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"

# .github/labeler.yml
documentation:
  - docs/**/*
  - '**/*.md'

frontend:
  - src/components/**/*
  - src/pages/**/*

backend:
  - src/api/**/*
  - src/server/**/*

tests:
  - tests/**/*
  - '**/*.test.ts'
```

---

## Conflict Prevention Strategies

### Keep Branches Short-Lived

```bash
# Bad: Branch alive for weeks
feature/massive-refactor  # 50 commits, 200 files changed

# Good: Small, focused branches
feature/refactor-auth-step1  # 5 commits
feature/refactor-auth-step2  # 5 commits
feature/refactor-auth-step3  # 5 commits
```

### Regular Syncing

```bash
# Sync with main at least daily
git fetch origin
git merge origin/main
# or
git rebase origin/main
```

### Communication

```markdown
# Announce when working on shared files

Slack/Teams message:
"Hey team, I'm refactoring the user service today.
Might cause conflicts if you're also working there.
Let's sync if needed!"
```

### File/Module Ownership

```yaml
# .github/CODEOWNERS
# These owners will be auto-requested for review

# Global owners
*       @team-lead

# Frontend
/src/components/    @frontend-team
/src/pages/         @frontend-team

# Backend
/src/api/           @backend-team
/src/services/      @backend-team

# DevOps
/.github/           @devops-team
/docker/            @devops-team

# Specific file
/src/config.ts      @team-lead @security-team
```

---

## Advanced Collaboration

### Git Bisect (Find Bug Introduction)

```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark a known good commit
git bisect good abc1234

# Git will checkout middle commit
# Test it, then mark as good or bad
git bisect good
# or
git bisect bad

# Repeat until Git finds the first bad commit

# End bisect
git bisect reset
```

### Git Blame (Who Changed What)

```bash
# See who last modified each line
git blame filename.js

# Ignore whitespace changes
git blame -w filename.js

# Show original author (ignore moves/copies)
git blame -C filename.js

# See changes in specific line range
git blame -L 10,20 filename.js
```

### Git Log for Investigation

```bash
# Search commit messages
git log --grep="bug fix"

# Search code changes
git log -S "functionName"

# Changes by author
git log --author="John"

# Changes to specific file
git log -- path/to/file.js

# Changes in date range
git log --since="2024-01-01" --until="2024-06-30"

# Combined
git log --author="John" --since="2024-01-01" -- src/
```

---

## Practice Exercises

### Exercise 1: Complete PR Workflow

```bash
# 1. Clone a repo (or use existing)
git clone https://github.com/YOUR/repo.git
cd repo

# 2. Create feature branch
git switch -c feature/add-greeting

# 3. Make changes
echo 'export const greet = (name) => `Hello, ${name}!`;' > greeting.js
git add .
git commit -m "feat: add greeting function"

# 4. Push and create PR
git push -u origin feature/add-greeting
# On GitHub: Create PR with description

# 5. After approval, merge via GitHub UI

# 6. Clean up locally
git switch main
git pull
git branch -d feature/add-greeting
```

### Exercise 2: Resolve PR Conflicts

```bash
# 1. Create a conflict scenario
git switch main
echo "Line 1" > shared.txt
git add . && git commit -m "Add shared file"
git push

# 2. Create branch A and modify
git switch -c branch-a
echo "Branch A content" > shared.txt
git commit -am "Branch A changes"
git push -u origin branch-a

# 3. Create branch B from main and modify same file
git switch main
git switch -c branch-b
echo "Branch B content" > shared.txt
git commit -am "Branch B changes"
git push -u origin branch-b

# 4. Merge branch-a to main first (via PR)
# 5. Try to merge branch-b - conflict!
# 6. Resolve by updating branch-b:
git switch branch-b
git fetch origin
git merge origin/main
# Resolve conflict in shared.txt
git add shared.txt
git commit -m "Resolve merge conflict"
git push
```

### Exercise 3: Squash Commits

```bash
# 1. Create branch with multiple commits
git switch -c feature/squash-practice
echo "step 1" > feature.txt && git add . && git commit -m "Step 1"
echo "step 2" >> feature.txt && git commit -am "Step 2"
echo "step 3" >> feature.txt && git commit -am "Step 3"
echo "step 4" >> feature.txt && git commit -am "Step 4"

# 2. Squash last 4 commits
git rebase -i HEAD~4

# 3. In editor:
# pick abc1234 Step 1
# squash def5678 Step 2
# squash ghi9012 Step 3
# squash jkl3456 Step 4

# 4. Save, then write new combined message

# 5. View clean history
git log --oneline
```

### Exercise 4: Use Git Bisect

```bash
# 1. Create history with a bug introduced
git init bisect-practice && cd bisect-practice
for i in {1..10}; do
  echo "Commit $i" >> file.txt
  git add . && git commit -m "Commit $i"
done

# 2. Pretend commit 5 introduced a bug
# 3. Use bisect to find it
git bisect start
git bisect bad  # Current is bad
git bisect good HEAD~8  # Commit 2 was good

# 4. Git checks out middle, test and mark
# Repeat until found
git bisect reset
```

---

## Key Commands Summary

| Command | Description |
|---------|-------------|
| `git fetch origin` | Download remote changes |
| `git pull --rebase` | Pull with rebase |
| `git push -u origin branch` | Push with tracking |
| `git push --force-with-lease` | Safe force push |
| `git rebase -i HEAD~n` | Interactive rebase |
| `git bisect start` | Start binary search |
| `git blame file` | Show line-by-line changes |
| `git log --grep="text"` | Search commits |
| `gh pr create` | Create PR (GitHub CLI) |
| `gh pr checkout 123` | Checkout PR locally |

---

## Key Takeaways

1. **Choose the right workflow** - Feature branch for most teams
2. **Keep PRs small** - Easier to review and less conflicts
3. **Sync frequently** - Merge/rebase main often
4. **Review thoroughly** - But don't block progress
5. **Communicate** - Announce when working on shared code
6. **Automate** - Use CI/CD for testing and validation
7. **Document conventions** - Branch names, commit messages

---

## Self-Check Questions

1. What's the difference between Feature Branch and GitFlow workflows?
2. How do you update your feature branch with latest main?
3. What makes a good pull request?
4. How do you find when a bug was introduced?
5. What is CODEOWNERS file for?
6. When should you use `--force-with-lease` vs `--force`?

---

## Phase 1 Complete!

Congratulations! You've completed Phase 1: Foundations. You now know:

- **HTML/CSS** - Structure and styling
- **JavaScript** - Programming fundamentals
- **TypeScript** - Type-safe JavaScript
- **Git & GitHub** - Version control and collaboration

**Next:** Continue to [Phase 2: Frontend](../../../phase-2-frontend/README.md) to learn React, Next.js, and modern frontend development!
