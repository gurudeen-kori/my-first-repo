# MODULE 2: GIT & GITHUB

## 📚 Overview
Git is essential for DevOps. This module covers version control, collaboration, branching strategies, and GitHub workflows needed for modern development and deployment pipelines.

---

## 1. VERSION CONTROL BASICS

### Why Version Control?
- **History tracking** - See all changes over time
- **Collaboration** - Multiple developers on same codebase
- **Branching** - Parallel development
- **Rollback** - Revert to previous versions
- **Accountability** - Track who changed what

### VCS Types
```
Centralized (CVS, SVN)
  ↓ Server holds everything
  ↓ Single point of failure
  ↓ Must be connected to server

Distributed (Git, Mercurial)
  ↓ Each developer has full repository
  ↓ Works offline
  ↓ Better for collaboration
```

---

## 2. GIT ARCHITECTURE

### Git Objects
```
Blob        - File content (immutable)
Tree        - Directory structure (list of blobs/trees)
Commit      - Snapshot of project (tree + metadata)
Tag         - Named reference to commit
Reference   - Pointer to commit (branch, HEAD)

commit abc123
├── tree xyz789
│   ├── blob file1.txt
│   ├── blob file2.txt
│   └── tree subdir
│       └── blob file3.txt
├── parent: abc122
├── author: john <john@example.com>
├── committer: john <john@example.com>
└── message: "Initial commit"
```

### Git Refs
```
HEAD              - Current commit
refs/heads/*      - Local branches
refs/remotes/*    - Remote branches
refs/tags/*       - Tags
refs/stash        - Stash entries
```

### Git Internals
```
.git/
├── objects/       - All git objects (blobs, trees, commits)
├── refs/          - References (branches, tags)
├── HEAD           - Points to current branch
├── config         - Repository config
├── index          - Staging area
├── hooks/         - Git hooks
└── logs/          - Reflog
```

---

## 3. REPOSITORY SETUP

### Initial Setup
```bash
# Global configuration
git config --global user.name "John Doe"
git config --global user.email "john@example.com"
git config --global core.editor "nano"
git config --list                    # Show all config
git config --global --edit           # Edit global config

# Repository configuration
git init                             # Create new repository
git config user.name "John"          # Local user name
git config user.email "john@local.com"  # Local email
```

### Clone Repository
```bash
# Clone via HTTPS
git clone https://github.com/user/repo.git

# Clone via SSH
git clone git@github.com:user/repo.git

# Clone specific branch
git clone -b main --single-branch repo.git

# Clone with limited history
git clone --depth 1 repo.git

# Clone to custom directory
git clone repo.git custom-dir
```

### Remote Management
```bash
# Add remote
git remote add origin https://github.com/user/repo.git
git remote add upstream https://github.com/upstream/repo.git

# View remotes
git remote -v                       # With URLs

# Remove remote
git remote remove origin

# Change remote URL
git remote set-url origin new-url

# Rename remote
git remote rename origin upstream
```

---

## 4. BASIC WORKFLOW

### Staging and Committing
```bash
# Check status
git status                          # Show changes
git diff                           # Show unstaged changes
git diff --staged                  # Show staged changes
git diff commit1 commit2           # Compare commits

# Stage changes
git add file.txt                   # Stage specific file
git add .                          # Stage all changes
git add -p                         # Interactive staging (hunks)

# Unstage
git reset HEAD file.txt            # Unstage file
git reset HEAD                     # Unstage all

# Commit
git commit -m "Add feature X"      # Simple commit
git commit -m "Title" -m "Details" # Multi-line commit
git commit -am "Update"            # Stage + commit tracked files
git commit --amend                 # Modify last commit
git commit --amend --no-edit       # Add to last commit without changing message

# View history
git log                            # Full commit history
git log --oneline                  # Condensed view
git log --graph --oneline --all    # Branch visualization
git log -5                         # Last 5 commits
git log --author="John"            # Filter by author
git log --since="2 weeks ago"      # Time filter
git log -p                         # Show changes per commit
git log --stat                     # Show statistics
git show commit_hash               # Show specific commit
git blame file.txt                 # Show who changed each line
```

### Undoing Changes
```bash
# Discard working directory changes
git checkout -- file.txt           # Restore file
git restore file.txt               # Modern syntax

# Unstage file
git reset HEAD file.txt            # Keep changes

# Revert commit (creates new commit)
git revert commit_hash             # Undo specific commit

# Hard reset (DANGEROUS - lose changes)
git reset --soft HEAD~1            # Undo commit, keep changes staged
git reset --mixed HEAD~1           # Undo commit, keep changes unstaged (default)
git reset --hard HEAD~1            # Undo commit, lose all changes

# Clean untracked files
git clean -fd                      # Remove untracked files and directories
git clean -fdx                     # Also remove ignored files
```

---

## 5. BRANCHING

### Branch Basics
```bash
# List branches
git branch                         # Local branches
git branch -a                      # All branches (local + remote)
git branch -r                      # Remote branches only

# Create branch
git branch feature/new-login       # Create locally
git switch -c feature/new-login    # Create and switch
git checkout -b feature/new-login  # Old syntax

# Switch branch
git switch main                    # Modern syntax
git checkout main                  # Older syntax

# Delete branch
git branch -d feature/old          # Delete local
git branch -D feature/old          # Force delete
git push origin --delete feature/old  # Delete remote

# Branch information
git branch -v                      # Show last commit per branch
git branch --merged                # Branches merged into current
git branch --no-merged             # Branches not merged
```

### Branch Strategies
```
Main Branch (Production)
  ↓
Develop Branch (Integration)
  ↓
Feature Branches (Feature Development)
  ├── feature/login
  ├── feature/dashboard
  └── feature/api

Release Branches (Release Preparation)
  └── release/1.0.0

Hotfix Branches (Production Fixes)
  └── hotfix/critical-bug

Popular strategies:
1. Git Flow - Complex, good for releases
2. GitHub Flow - Simple, CI/CD friendly
3. Trunk-based - Continuous integration
```

---

## 6. MERGE VS REBASE

### Merge
```bash
# Merge branch into current branch
git merge feature/login            # 3-way merge
git merge feature/login --squash   # Squash commits before merging
git merge feature/login --no-ff    # Create merge commit even if fast-forward

# Merge conflicts
git status                         # Show conflicts
# Edit conflicted files, resolve manually
git add resolved_file.txt
git commit -m "Resolve merge conflict"

# Abort merge
git merge --abort
```

### Rebase
```bash
# Rebase current branch onto another
git rebase main                    # Replay commits on main
git rebase -i HEAD~3              # Interactive rebase last 3 commits

# Handle conflicts during rebase
# Edit conflicted files
git add resolved_file.txt
git rebase --continue
# or
git rebase --abort                # Abort rebase
```

### Merge vs Rebase
```
Merge:
  ✓ Preserves history
  ✓ Safe for shared branches
  ✗ Creates merge commits
  ✗ Complex history

Rebase:
  ✓ Linear history
  ✓ Clean commits
  ✗ Rewrites history (don't use on shared branches)
  ✗ Can lose commits if something goes wrong
```

### Interactive Rebase
```bash
git rebase -i HEAD~5              # Rebase last 5 commits

# Commands in interactive mode:
pick    - Use commit
reword  - Use commit, edit message
squash  - Use commit, meld into previous
fixup   - Like squash, discard log message
drop    - Remove commit
exec    - Run shell command
```

---

## 7. CHERRY-PICK

### Cherry-pick Specific Commits
```bash
# Apply specific commit to current branch
git cherry-pick abc123            # Apply single commit
git cherry-pick abc123..def456    # Apply range (exclusive)
git cherry-pick abc123^..def456   # Apply range (inclusive)

# Multiple commits
git cherry-pick commit1 commit2 commit3

# Continue after conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort
```

### Use Cases
```
1. Backport bug fix to release branch
2. Apply specific feature across branches
3. Recover deleted commits
4. Merge specific commits without full merge
```

---

## 8. GITIGNORE

### .gitignore Patterns
```bash
# Ignore file
*.log
debug.txt

# Ignore directory
build/
node_modules/
__pycache__/

# Ignore all but specific
*.txt
!important.txt

# Ignore in specific directory
docs/*.tmp

# Ignore everywhere
**/temp/

# Negation
!.gitkeep
```

### .gitignore Template
```
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/

# Node
node_modules/
npm-debug.log
.env

# Java
target/
*.class
*.jar

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Temporary
temp/
tmp/
```

### Global .gitignore
```bash
# Create global ignore
nano ~/.gitignore_global

# Add IDE patterns
.vscode/
.idea/
*.swp

# Configure git
git config --global core.excludesfile ~/.gitignore_global
```

---

## 9. TAGS

### Create and Manage Tags
```bash
# Create lightweight tag
git tag v1.0.0                    # Tag current commit
git tag v1.0.0 abc123            # Tag specific commit

# Create annotated tag
git tag -a v1.0.0 -m "Version 1.0.0"

# List tags
git tag                           # All tags
git tag -l "v1.*"                # Filter tags

# Show tag
git show v1.0.0                  # Show tag info and commit

# Delete tag
git tag -d v1.0.0                # Delete local
git push origin --delete v1.0.0  # Delete remote

# Push tags
git push origin v1.0.0           # Push specific tag
git push origin --tags           # Push all tags
```

### Semantic Versioning
```
MAJOR.MINOR.PATCH
1.2.3
│ │ │
│ │ └─ Patch: Bug fixes
│ └─── Minor: New features (backward compatible)
└───── Major: Breaking changes

Examples:
v1.0.0  - Initial release
v1.1.0  - New features added
v1.1.1  - Bug fix
v2.0.0  - Breaking changes
```

---

## 10. GITHUB

### GitHub Features
```
Repositories     - Code storage
Branches         - Version management
Pull Requests    - Code review and merge
Issues           - Bug tracking and features
Projects         - Task management
Actions          - CI/CD automation
Releases         - Version packaging
Wiki             - Documentation
```

### Repository Settings
```
General
  - Repository name
  - Description
  - Visibility (Public/Private)
  - Default branch

Collaborators
  - Manage access
  - Assign roles (Owner, Maintainer, Developer)

Branch Protection
  - Require PR reviews
  - Require status checks
  - Require up-to-date branches
  - Dismiss stale reviews
```

### SSH Keys on GitHub
```bash
# Generate key
ssh-keygen -t ed25519 -C "github@example.com"

# Add to SSH agent
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub

# Add to GitHub Settings > SSH and GPG keys

# Test connection
ssh -T git@github.com
```

---

## 11. PULL REQUESTS

### Creating PRs
```bash
# Push branch to GitHub
git push -u origin feature/new-login

# On GitHub: Create Pull Request
1. Go to repository
2. Click "New Pull Request"
3. Select base branch (main) and compare branch (feature/new-login)
4. Add title and description
5. Click "Create Pull Request"

# Or use GitHub CLI
gh pr create --title "Add login feature" --body "Implements user login"
```

### PR Description Template
```markdown
## Description
Brief explanation of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## How to Test
Steps to test the changes

## Checklist
- [ ] Code follows style guidelines
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No new warnings generated
```

### PR Workflow
```
1. Create feature branch
   git switch -c feature/new-feature

2. Make changes and commit
   git add .
   git commit -m "Add feature"

3. Push to GitHub
   git push -u origin feature/new-feature

4. Create Pull Request on GitHub

5. Wait for reviews and CI/CD checks

6. Address feedback
   git add .
   git commit -m "Address review comments"
   git push

7. Merge PR
   # Click "Merge pull request" on GitHub

8. Delete branch
   git branch -d feature/new-feature
   git push origin --delete feature/new-feature
```

### PR Management
```bash
# List PRs (GitHub CLI)
gh pr list                         # All open PRs
gh pr list --state closed          # Closed PRs
gh pr list --assignee @me          # Assigned to me

# View PR details
gh pr view 123                      # PR #123 details

# Check out PR locally
gh pr checkout 123                  # Checkout PR branch

# Add comment
gh pr comment 123 -b "LGTM!"

# Approve PR
gh pr review 123 --approve

# Merge PR
gh pr merge 123                     # Merge PR
gh pr merge 123 --squash            # Squash and merge
gh pr merge 123 --rebase            # Rebase and merge
```

---

## 12. GITHUB ACTIONS

### CI/CD with GitHub Actions
```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        pip install pytest
        pip install -r requirements.txt
    
    - name: Run tests
      run: pytest
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

### Common Workflows
```yaml
# Build and Deploy
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: docker build -t myapp:latest .
    
    - name: Push to Docker Hub
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | \
        docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push myapp:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to server
      run: |
        ssh -i ${{ secrets.DEPLOY_KEY }} user@server
        docker pull myapp:latest
        docker-compose up -d
```

### Secrets and Variables
```yaml
# Use secrets
- name: Login to Docker
  run: echo ${{ secrets.DOCKER_PASSWORD }} | docker login

# Use variables
- name: Deploy
  env:
    DEPLOY_ENV: ${{ vars.DEPLOY_ENV }}
  run: ./deploy.sh
```

---

## 13. ADVANCED GIT

### Stashing
```bash
# Stash changes
git stash                         # Stash all changes
git stash save "WIP: feature X"  # Stash with message
git stash -u                      # Include untracked files

# List stashes
git stash list                    # All stashes
git stash show stash@{0}          # Show stash contents

# Apply stash
git stash apply stash@{0}         # Apply and keep stash
git stash pop stash@{0}           # Apply and remove stash
git stash pop                     # Apply most recent

# Delete stash
git stash drop stash@{0}          # Delete specific
git stash clear                   # Delete all
```

### Reflog
```bash
# Git reference log
git reflog                        # All HEAD changes
git reflog show branchname        # Branch-specific reflog

# Recover lost commits
git reflog                        # Find commit hash
git reset --hard abc123          # Restore to that commit
```

### Bisect
```bash
# Find commit that introduced bug
git bisect start                  # Start bisect
git bisect bad HEAD               # Mark current as bad
git bisect good v1.0              # Mark v1.0 as good

# Git will checkout middle commit
# Test if bug exists
git bisect bad                    # If bug exists
git bisect good                   # If no bug

# Repeat until found
git bisect reset                  # End bisect
```

### Hooks
```bash
# Create pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
npm run lint || exit 1
npm run test || exit 1
EOF

chmod +x .git/hooks/pre-commit

# Common hooks
pre-commit      - Before commit
commit-msg      - Validate message
pre-push        - Before push
post-merge      - After merge
pre-rebase      - Before rebase
```

---

## HANDS-ON LABS

### Lab 1: Repository Setup and Basic Workflow
```bash
# Create and initialize repo
mkdir myproject && cd myproject
git init
git config user.name "John Doe"
git config user.email "john@example.com"

# Create and commit files
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit: add README"

# Check history
git log --oneline
git show HEAD
```

### Lab 2: Branching and Merging
```bash
# Create feature branch
git switch -c feature/login

# Make changes
echo "Login page" > login.html
git add login.html
git commit -m "Add login page"

# Switch to main
git switch main

# Merge feature
git merge feature/login

# View history
git log --graph --oneline --all
```

### Lab 3: Merge Conflict Resolution
```bash
# Create conflicting changes
git switch -c conflict-test
echo "Version A" > file.txt
git add file.txt
git commit -m "Version A"

git switch main
echo "Version B" > file.txt
git add file.txt
git commit -m "Version B"

# Try merge (will conflict)
git merge conflict-test

# Resolve manually
nano file.txt
# Edit to choose version

git add file.txt
git commit -m "Resolve conflict"
```

### Lab 4: GitHub Actions Workflow
```yaml
# Create .github/workflows/simple.yml
name: Simple Workflow

on:
  push:
    branches: [main]

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
    - run: echo "Hello from GitHub Actions!"
    
    - uses: actions/checkout@v3
    
    - run: ls -la
```

### Lab 5: Pull Request Workflow
```bash
# Fork repository on GitHub

# Clone fork
git clone https://github.com/youruser/original-repo.git

# Add upstream
git remote add upstream https://github.com/original/repo.git

# Create feature branch
git switch -c feature/improvement

# Make changes
echo "Improvement" > feature.txt
git add feature.txt
git commit -m "Add improvement"

# Push to fork
git push -u origin feature/improvement

# Create PR on GitHub
# On GitHub: Compare and create pull request
```

---

## INTERVIEW QUESTIONS

### Beginner Level
1. **What is Git and why use it?**
   - Distributed version control system
   - Tracks changes, enables collaboration
   - Allows branching and merging

2. **Difference between Git and GitHub?**
   - Git: Version control software
   - GitHub: Hosting service for Git repos

3. **What is a commit?**
   - Snapshot of project state
   - Contains: tree (files), parent, author, message

4. **Explain git workflow: add → commit → push**
   - Add: Stage changes
   - Commit: Create snapshot with message
   - Push: Upload to remote repository

5. **What is .gitignore?**
   - File that lists what Git should ignore
   - Prevents tracking build files, credentials, etc.

### Intermediate Level
6. **Difference between merge and rebase?**
   - Merge: Combines branches, preserves history
   - Rebase: Replays commits, linear history

7. **What is a pull request?**
   - Request to merge branch into another
   - Enables code review before merging

8. **How to undo a commit that's already pushed?**
   - Use `git revert` (creates new commit undoing changes)
   - Don't use `git reset --hard` (rewrites history)

9. **What is a merge conflict?**
   - When same file changed in different ways
   - Require manual resolution

10. **Explain git branching strategy?**
    - Feature branches for development
    - Main branch for production
    - Develop branch for integration

### Advanced Level
11. **How does cherry-pick work and when to use?**
    - Applies specific commits to current branch
    - Use for: backporting fixes, applying specific changes

12. **Explain git internals (objects, refs, HEAD)?**
    - Objects: Blob, Tree, Commit, Tag
    - Refs: Pointers to commits
    - HEAD: Points to current commit

13. **How to recover lost commits?**
    - Use `git reflog` to see all HEAD changes
    - Use `git reset --hard commit_hash` to recover

14. **What's the difference between squash, rebase, and merge?**
    - Merge: Keeps all commits, creates merge commit
    - Squash: Combines commits into one
    - Rebase: Replays commits on top of base

15. **How to handle secrets in Git?**
    - Use `.gitignore` for local secrets
    - Use GitHub Secrets for CI/CD
    - Use tools like git-crypt or BlackBox

### Scenario-based
16. **You pushed sensitive data. How to remove?**
    - Use `git filter-branch` or `git filter-repo`
    - Remove from history and rewrite
    - Notify users to rotate credentials
    - Force push to update remote (dangerous)

17. **How to collaborate on GitHub?**
    - Fork the repo
    - Clone and create feature branch
    - Make changes and push
    - Create pull request
    - Address review feedback
    - Merge when approved

18. **PR has conflicts. How to resolve?**
    - Fetch latest main: `git fetch origin main`
    - Rebase on main: `git rebase origin/main`
    - Resolve conflicts in editor
    - Complete rebase: `git rebase --continue`
    - Force push: `git push --force-with-lease`

19. **How to organize commits in PR?**
    - One logical change per commit
    - Write clear commit messages
    - Use interactive rebase to squash/reorder
    - Keep commits related to single issue

20. **Best practices for commit messages?**
    - First line: brief summary (50 chars)
    - Blank line
    - Detailed explanation (wrap at 72 chars)
    - Reference issue: "Fixes #123"
    - Explain why, not what

---

## BEST PRACTICES

1. **Commit messages**
   - Clear, descriptive messages
   - Use imperative mood: "Add feature" not "Added feature"
   - Reference related issues

2. **Branching**
   - Use meaningful branch names: feature/*, bugfix/*, hotfix/*
   - Keep branches short-lived
   - Delete merged branches

3. **Code review**
   - Review before merging
   - Test PR locally
   - Provide constructive feedback

4. **History**
   - Keep history clean
   - Use rebase for feature branches
   - Use merge for main branch

5. **Collaboration**
   - Keep main branch stable
   - Use develop branch for integration
   - Require PR reviews
   - Require status checks

---

**Total Interview Questions: 40+**
**Total Labs: 5 hands-on exercises**
**Total Commands: 100+**
