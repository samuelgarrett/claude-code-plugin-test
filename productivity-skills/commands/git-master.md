# Git Master

**Description**: Master Git workflows, branching strategies, and advanced techniques

## Git Workflow

### Feature Branch Workflow
```bash
# 1. Create feature branch
git checkout -b feature/user-authentication

# 2. Make changes and commit regularly
git add .
git commit -m "Add login form component"

# 3. Push to remote
git push -u origin feature/user-authentication

# 4. Keep branch updated with main
git fetch origin
git rebase origin/main

# 5. Create pull request (via GitHub/GitLab UI)

# 6. After merge, clean up
git checkout main
git pull origin main
git branch -d feature/user-authentication
```

## Essential Git Commands

### Committing
```bash
# Stage specific files
git add file1.js file2.js

# Stage all changes
git add .

# Commit with message
git commit -m "Add user authentication"

# Amend last commit
git commit --amend -m "Updated message"

# Add to last commit without changing message
git add forgotten-file.js
git commit --amend --no-edit
```

### Branching
```bash
# Create and switch to branch
git checkout -b feature/new-feature

# Switch branches
git checkout main

# List branches
git branch              # Local
git branch -r           # Remote
git branch -a           # All

# Delete branch
git branch -d feature/old-feature
git branch -D feature/old-feature  # Force delete

# Rename branch
git branch -m old-name new-name
```

### Syncing
```bash
# Fetch changes
git fetch origin

# Pull changes (fetch + merge)
git pull origin main

# Pull with rebase (cleaner history)
git pull --rebase origin main

# Push changes
git push origin feature/my-feature

# Force push (use carefully!)
git push --force-with-lease origin feature/my-feature
```

### Undoing Changes
```bash
# Discard unstaged changes
git checkout -- file.js

# Unstage files
git reset HEAD file.js

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Revert a commit (creates new commit)
git revert abc123
```

## Advanced Techniques

### Interactive Rebase
```bash
# Clean up last 3 commits
git rebase -i HEAD~3

# In editor:
# pick abc123 First commit
# squash def456 Second commit
# fixup ghi789 Third commit

# Squash = combine commits, keep both messages
# Fixup = combine commits, discard this message
```

### Cherry-Pick
```bash
# Apply specific commit from another branch
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456 ghi789
```

### Stash
```bash
# Save work in progress
git stash

# Stash with message
git stash save "WIP: working on feature X"

# List stashes
git stash list

# Apply stash
git stash pop           # Apply and remove
git stash apply         # Apply and keep

# Apply specific stash
git stash apply stash@{2}

# Delete stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

### Finding Issues
```bash
# Search commits
git log --grep="bug fix"

# Show changes in file
git log -p file.js

# Find who changed a line
git blame file.js

# Find when bug was introduced (binary search)
git bisect start
git bisect bad          # Current commit is bad
git bisect good abc123  # Known good commit
# Git will checkout commits to test
git bisect good/bad     # Mark each
git bisect reset        # When done
```

## Commit Message Best Practices

```
# Good commit message format
type(scope): Short description (max 50 chars)

Longer explanation if needed (wrap at 72 chars).
Explain WHAT and WHY, not HOW.

- Bullet points are okay
- Use present tense: "Add feature" not "Added feature"

Closes #123

# Types:
feat:     New feature
fix:      Bug fix
docs:     Documentation
style:    Formatting (no code change)
refactor: Code restructuring
test:     Adding tests
chore:    Maintenance tasks
```

Examples:
```
feat(auth): Add JWT authentication

Implement JWT-based authentication system with refresh tokens.
Users can now login and maintain session across page reloads.

Closes #456

---

fix(cart): Prevent duplicate items being added

When clicking "Add to cart" rapidly, duplicate items were added.
Added debouncing to prevent this issue.

Fixes #789
```

## Git Configuration

```bash
# User identity
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Default editor
git config --global core.editor "code --wait"

# Useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm "commit -m"
git config --global alias.lg "log --oneline --graph --all --decorate"

# Auto-setup remote tracking
git config --global push.autoSetupRemote true
```

## .gitignore Best Practices

```
# .gitignore - Common patterns

# Dependencies
node_modules/
venv/
__pycache__/

# Build outputs
dist/
build/
*.pyc
*.class

# Environment files
.env
.env.local
.env.*.local

# IDE files
.vscode/
.idea/
*.swp
*.swo
.DS_Store

# Logs
logs/
*.log

# Test coverage
coverage/
.nyc_output/
```

## Resolving Merge Conflicts

```bash
# 1. Attempt merge
git merge feature/other-branch

# CONFLICT message appears

# 2. Check conflicted files
git status

# 3. Open files and resolve conflicts
# Look for markers:
# <<<<<<< HEAD
# Your changes
# =======
# Their changes
# >>>>>>> feature/other-branch

# 4. Edit file to resolve
# Remove markers, keep desired code

# 5. Mark as resolved
git add resolved-file.js

# 6. Complete merge
git commit
```

## Branch Strategies

### Gitflow
```
main         - Production code
develop      - Integration branch
feature/*    - New features
hotfix/*     - Urgent fixes
release/*    - Release preparation
```

### Trunk-Based
```
main         - Always deployable
feature/*    - Short-lived (< 1 day)
```

## When to Use This Skill

- Managing Git repositories
- Resolving merge conflicts
- Cleaning up commit history
- Implementing branching strategies
- Debugging with Git
- Setting up Git workflows

---

**Remember**: Commit often, write clear messages, and keep branches short-lived!
