# Day 3: Merging & Conflict Resolution

## Understanding Merging

Merging combines changes from different branches into one branch.

### Types of Merges

```
1. FAST-FORWARD MERGE (linear history)
   When main hasn't changed since branching:

   Before:
   main ────●────●────●
                        \
                         ●────●  feature

   After git merge feature:
   main ────●────●────●────●────●
                    (feature moved to main)


2. THREE-WAY MERGE (diverged history)
   When both branches have new commits:

   Before:
   main ────●────●────●────●
                        \
                         ●────●  feature

   After git merge feature:
   main ────●────●────●────●
                        \     \
                         ●────●
                               \
                                ●  (merge commit)
```

---

## Fast-Forward Merge

### When It Happens

```bash
# Create a simple linear history
git switch main
echo "main content" > file.txt
git add . && git commit -m "Initial commit"

# Create feature branch
git switch -c feature/simple
echo "feature content" >> file.txt
git add . && git commit -m "Add feature"

# Switch back to main (no new commits on main)
git switch main

# Merge feature - this will fast-forward
git merge feature/simple
# Output: Fast-forward
```

### Preventing Fast-Forward

```bash
# Force a merge commit even when fast-forward is possible
git merge --no-ff feature/simple -m "Merge feature/simple"

# This creates a merge commit that shows the feature was developed separately
```

### When to Use --no-ff

- **Use --no-ff** for feature branches to preserve feature history
- **Allow fast-forward** for quick fixes or small changes
- Many teams use `--no-ff` by default for clarity

---

## Three-Way Merge

### When It Happens

```bash
# Start with main
git switch main
echo "initial" > file.txt
git add . && git commit -m "Initial"

# Create feature branch
git switch -c feature/complex
echo "feature code" >> file.txt
git add . && git commit -m "Feature work"

# Go back to main and make changes
git switch main
echo "main update" >> README.md
git add . && git commit -m "Update main"

# Now main has diverged - merge will create merge commit
git merge feature/complex -m "Merge feature/complex into main"
```

### Understanding the Three Sources

```
        OURS (current branch - main)
              │
              ▼
    ┌─────────────────┐
    │   merged file   │ ◄── Merge Result
    └─────────────────┘
              ▲
              │
    THEIRS (branch being merged - feature)


The "base" is the common ancestor of both branches
```

---

## Merge Conflicts

### What Causes Conflicts

Conflicts occur when:
1. Same lines in a file are changed in both branches
2. A file is modified in one branch and deleted in another
3. Both branches add a file with the same name

### Conflict Markers

```
When a conflict occurs, Git adds markers to the file:

<<<<<<< HEAD
This is the content from the current branch (main)
=======
This is the content from the branch being merged (feature)
>>>>>>> feature/branch-name
```

### Conflict Example

```bash
# Create conflict scenario
git switch main
echo "Line 1" > conflict.txt
echo "Line 2" >> conflict.txt
git add . && git commit -m "Add conflict.txt"

# Create feature branch and modify same line
git switch -c feature/change-a
sed -i '' 's/Line 2/Line 2 - Modified by A/' conflict.txt
git commit -am "Change A"

# Go back to main and modify same line differently
git switch main
sed -i '' 's/Line 2/Line 2 - Modified by main/' conflict.txt
git commit -am "Change main"

# Try to merge - conflict!
git merge feature/change-a
# Output: CONFLICT (content): Merge conflict in conflict.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

---

## Resolving Conflicts

### Step-by-Step Resolution

```bash
# 1. See which files have conflicts
git status
# Output: both modified: conflict.txt

# 2. Open the conflicting file
# You'll see conflict markers:
<<<<<<< HEAD
Line 2 - Modified by main
=======
Line 2 - Modified by A
>>>>>>> feature/change-a

# 3. Edit the file to resolve (choose one, combine, or write new):
Line 1
Line 2 - Modified by main and A  # Combined version

# 4. Remove ALL conflict markers (<<<<, ====, >>>>)

# 5. Stage the resolved file
git add conflict.txt

# 6. Complete the merge
git commit -m "Merge feature/change-a, resolve conflict"

# Or just use default message
git commit  # Opens editor with pre-filled merge message
```

### Using Merge Tools

```bash
# Configure merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Or use VS Code directly
git config --global merge.tool "code"
git config --global mergetool.code.cmd "code --wait --merge $REMOTE $LOCAL $BASE $MERGED"

# Launch merge tool when conflict occurs
git mergetool
```

### Abort the Merge

```bash
# If you want to cancel the merge entirely
git merge --abort

# This restores the repository to the pre-merge state
```

---

## Conflict Resolution Strategies

### Strategy 1: Keep Our Version

```bash
# During merge, keep current branch version for conflicting files
git checkout --ours filename.txt
git add filename.txt

# For all conflicts
git checkout --ours .
git add .
```

### Strategy 2: Keep Their Version

```bash
# Keep the incoming branch version
git checkout --theirs filename.txt
git add filename.txt

# For all conflicts
git checkout --theirs .
git add .
```

### Strategy 3: Manual Resolution

```bash
# Edit files manually, then stage
# This is the most common approach

# 1. Open file in editor
# 2. Find conflict markers
# 3. Decide what final content should be
# 4. Remove conflict markers
# 5. Save and stage
git add resolved-file.txt
```

### Strategy 4: Using merge strategies

```bash
# Accept all of their changes (whole merge)
git merge -X theirs feature-branch

# Accept all of our changes (whole merge)
git merge -X ours feature-branch

# Merge with specific strategy
git merge -s recursive -X patience feature-branch
```

---

## Advanced Merge Scenarios

### Merging Specific Commits (Cherry-Pick)

```bash
# Apply a specific commit to current branch
git cherry-pick abc1234

# Apply multiple commits
git cherry-pick abc1234 def5678

# Apply without committing
git cherry-pick --no-commit abc1234

# Continue after resolving conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort
```

### Squash Merge

```bash
# Combine all feature branch commits into one
git merge --squash feature-branch

# This stages all changes but doesn't commit
# You then create a single commit
git commit -m "Add complete feature (squashed)"

# Result: Clean history with one commit instead of many
```

### Merge Without Committing

```bash
# Merge but don't auto-commit
git merge --no-commit feature-branch

# Review changes
git diff --cached

# Then commit or abort
git commit -m "Merge after review"
# or
git merge --abort
```

---

## Viewing Merge Information

### See Merge History

```bash
# Show merge commits
git log --merges

# Show merge commits with details
git log --merges --oneline

# Show what was merged
git log --merges --first-parent

# See branches merged into main
git branch --merged main

# See branches NOT merged into main
git branch --no-merged main
```

### Check If Branches Can Merge Cleanly

```bash
# Try merge without actually merging
git merge --no-commit --no-ff feature-branch

# Check result
git diff --cached
git status

# Abort the test merge
git merge --abort
```

### See Conflicts Before Merging

```bash
# Check what would conflict
git merge --no-commit feature-branch
git diff --name-only --diff-filter=U

# Abort
git merge --abort
```

---

## Rebasing (Alternative to Merging)

### What is Rebasing?

Rebasing replays your commits on top of another branch, creating a linear history.

```
Before rebase:
main ────●────●────●
                    \
                     ●────●  feature

After git rebase main (from feature):
main ────●────●────●
                    \
                     ●'────●'  feature (rebased)

The commits on feature are replayed on top of main
```

### Basic Rebase

```bash
# On feature branch
git switch feature-branch

# Rebase onto main
git rebase main

# If conflicts occur:
# 1. Resolve conflicts
# 2. Stage resolved files
git add .
# 3. Continue rebase
git rebase --continue

# Or abort
git rebase --abort
```

### When to Rebase vs Merge

**Use Rebase:**
- Updating feature branch with latest main
- Cleaning up local commits before pushing
- When you want linear history

**Use Merge:**
- Integrating completed feature into main
- When branch is already pushed/shared
- When preserving branch history matters

### Golden Rule of Rebasing

**Never rebase commits that exist outside your repository (pushed to shared branch)**

```bash
# SAFE: Rebase your local feature branch
git switch feature-branch
git rebase main

# DANGEROUS: Rebase and force push shared branch
git rebase main
git push --force  # This rewrites shared history!
```

---

## Best Practices for Merging

### Before Merging

```bash
# 1. Ensure your branch is up to date
git switch main
git pull

# 2. Update your feature branch with latest main
git switch feature-branch
git merge main  # or git rebase main

# 3. Run tests
npm test

# 4. Review changes
git log main..feature-branch
git diff main..feature-branch
```

### During Merge

```bash
# 1. Use meaningful merge messages
git merge feature-branch -m "Merge: Add user authentication module"

# 2. Take time with conflicts - don't rush
# 3. Test after resolving conflicts
```

### After Merging

```bash
# 1. Clean up feature branch
git branch -d feature-branch

# 2. Push merged main
git push

# 3. Run full test suite
npm test
```

---

## Practice Exercises

### Exercise 1: Simple Merge

```bash
# Setup
mkdir merge-practice && cd merge-practice
git init
echo "# Project" > README.md
git add . && git commit -m "Initial"

# Create and merge feature
git switch -c feature/add-docs
echo "## Documentation" >> README.md
git commit -am "Add docs section"

git switch main
git merge feature/add-docs

# View result
git log --oneline --graph
```

### Exercise 2: Create and Resolve Conflict

```bash
# Create conflict
git switch main
echo "Original content" > shared.txt
git add . && git commit -m "Add shared file"

# Branch A changes
git switch -c branch-a
echo "Content from A" > shared.txt
git commit -am "Branch A change"

# Main changes
git switch main
echo "Content from main" > shared.txt
git commit -am "Main change"

# Merge and resolve
git merge branch-a
# Conflict! Edit shared.txt, remove markers
echo "Combined content from main and A" > shared.txt
git add shared.txt
git commit -m "Merge branch-a, resolve conflict"
```

### Exercise 3: Squash Merge

```bash
# Create feature with multiple commits
git switch -c feature/multi
echo "step 1" > feature.txt
git add . && git commit -m "Step 1"
echo "step 2" >> feature.txt
git commit -am "Step 2"
echo "step 3" >> feature.txt
git commit -am "Step 3"

# View commits
git log --oneline

# Squash merge to main
git switch main
git merge --squash feature/multi
git commit -m "Add complete feature (3 steps combined)"

# View clean history
git log --oneline
```

### Exercise 4: Using --ours and --theirs

```bash
# Setup conflict
git switch main
echo "keep this version" > config.txt
git add . && git commit -m "Add config"

git switch -c other
echo "discard this version" > config.txt
git commit -am "Other config"

git switch main

# Merge keeping ours for conflicts
git merge other
git checkout --ours config.txt
git add config.txt
git commit -m "Merge other, keep our config"

cat config.txt  # Should show "keep this version"
```

---

## Key Commands Summary

| Command | Description |
|---------|-------------|
| `git merge <branch>` | Merge branch into current |
| `git merge --no-ff <branch>` | Force merge commit |
| `git merge --squash <branch>` | Squash all commits |
| `git merge --abort` | Cancel ongoing merge |
| `git checkout --ours <file>` | Keep our version |
| `git checkout --theirs <file>` | Keep their version |
| `git cherry-pick <commit>` | Apply specific commit |
| `git rebase <branch>` | Rebase onto branch |
| `git mergetool` | Open merge tool |
| `git log --merges` | Show merge commits |

---

## Key Takeaways

1. **Fast-forward** - Simple merge when branches haven't diverged
2. **Three-way merge** - Creates merge commit when both branches have changes
3. **Conflicts** - Occur when same lines are changed differently
4. **Resolve carefully** - Don't rush conflict resolution
5. **--no-ff** - Creates explicit merge commits for feature branches
6. **Rebase** - Alternative that creates linear history
7. **Never force-push shared branches** - Rewriting shared history causes problems

---

## Self-Check Questions

1. What's the difference between fast-forward and three-way merge?
2. What do the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) mean?
3. How do you abort a merge with conflicts?
4. When should you use `git merge --squash`?
5. What's the golden rule of rebasing?
6. How do you keep your version of a file during conflict resolution?

---

**Next Lesson:** [Day 4 - GitHub & Remote Repositories](./day-04-github-remote.md)
