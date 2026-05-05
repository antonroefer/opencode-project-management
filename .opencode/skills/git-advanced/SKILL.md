---
name: git-advanced
description: Advanced Git workflows beyond basic commits. Covers rebasing, cherry-picking, submodules, hooks, bisect, worktree management, and conflict resolution strategies. Use when performing complex git operations, rewriting history, managing submodules, resolving difficult merges, or when the user mentions rebase, cherry-pick, bisect, git hooks, submodules, or worktrees.
---

# Git Advanced Skill

Master advanced Git operations for complex version control scenarios.

## When to Use
- Rewriting commit history (rebase, amend)
- Cherry-picking specific commits
- Managing git submodules
- Setting up git hooks
- Using git bisect to find bugs
- Managing multiple working directories (worktrees)
- Resolving complex merge conflicts

## Interactive Rebase

### Basic Rebase
```bash
# Rebase current branch onto main
git rebase main

# Interactive rebase last 3 commits
git rebase -i HEAD~3
```

### Interactive Rebase Options
```
pick 1234567 Add feature
reword 2345678 Fix bug  # Change commit message
edit 3456789 Update docs  # Pause and amend
squash 4567890 Minor fix  # Combine with previous
fixup 5678901 Typo fix  # Squash, discard message
drop 6789012 Unneeded  # Remove commit
```

### Rebase Workflow
```bash
# 1. Start interactive rebase
git rebase -i HEAD~5

# 2. Edit commits as needed
# 3. If conflicts occur:
git status  # See conflicted files
# Fix conflicts in editor
git add <fixed-file>
git rebase --continue

# 4. Abort if needed
git rebase --abort
```

## Cherry-Picking

### Basic Cherry-Pick
```bash
# Cherry-pick a single commit
git cherry-pick <commit-hash>

# Cherry-pick multiple commits
git cherry-pick <hash1> <hash2> <hash3>

# Cherry-pick without auto-commit (to modify)
git cherry-pick -n <commit-hash>
```

### Cherry-Pick Workflow
```bash
# 1. Find the commit you want
git log --oneline feature-branch

# 2. Switch to target branch
git checkout main

# 3. Cherry-pick the commit
git cherry-pick a1b2c3d

# 4. Handle conflicts if any
git status
# Fix conflicts, then:
git add .
git cherry-pick --continue
```

## Git Submodules

### Adding Submodules
```bash
# Add a submodule
git submodule add https://github.com/user/repo.git libs/repo

# Clone with submodules
git clone --recurse-submodules https://github.com/user/project.git

# Update submodules after clone
git submodule update --init --recursive
```

### Working with Submodules
```bash
# Update submodule to latest remote commit
cd libs/repo
git fetch origin
git checkout v1.0.0
cd ../..
git add libs/repo
git commit -m "Update submodule to v1.0.0"

# Pull with submodule updates
git pull --recurse-submodules
```

## Git Hooks

### Common Hooks
```
.git/hooks/
├── pre-commit        # Run before commit (lint, format)
├── commit-msg        # Validate commit message
├── pre-push          # Run before push (tests)
├── post-merge        # After merge/pull
└── pre-commit.sample # Example files
```

### Pre-Commit Hook Example
```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "Running linter..."
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint failed. Commit aborted."
  exit 1
fi

echo "Running tests..."
npm test
if [ $? -ne 0 ]; then
  echo "Tests failed. Commit aborted."
  exit 1
fi
```

### Making Hooks Executable
```bash
chmod +x .git/hooks/pre-commit
```

## Git Bisect

### Finding a Bug with Bisect
```bash
# 1. Start bisect
git bisect start

# 2. Mark current (bad) commit
git bisect bad

# 3. Mark a known good commit
git bisect good v1.0.0

# 4. Git checks out a commit in the middle
#    Test it, then mark:
git bisect good  # or
git bisect bad

# 5. Repeat until Git finds the culprit
# 6. Reset when done
git bisect reset
```

### Automated Bisect
```bash
# Run a script to automatically find bad commit
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
git bisect run npm test
git bisect reset
```

## Git Worktrees

### Basic Worktree Operations
```bash
# Create a new worktree for a branch
git worktree add ../hotfix-branch hotfix

# Create worktree with new branch
git worktree add -b feature/new-feature ../feature-worktree

# List worktrees
git worktree list

# Remove a worktree
git worktree remove ../hotfix-branch
```

### Use Cases
- Work on multiple branches simultaneously
- Keep dirty working directory while switching contexts
- Run tests in one worktree while developing in another

## Conflict Resolution

### Merge Conflicts
```bash
# When merge conflicts occur
git status  # See conflicted files

# Edit files to resolve conflicts
# Look for conflict markers:
# <<<<<<< HEAD
# Current changes
# =======
# Incoming changes
# >>>>>>> branch-name

# After resolving:
git add <resolved-file>
git commit
```

### Using Merge Tools
```bash
# Configure merge tool
git config --global merge.tool vimdiff

# Use merge tool
git mergetool
```

### Rebase Conflicts
```bash
# During rebase, if conflict:
git status
# Fix conflicts
git add .
git rebase --continue

# Or skip the problematic commit
git rebase --skip

# Or abort
git rebase --abort
```

## Reflog (Recovery)

### View Reflog
```bash
# See all recent HEAD positions
git reflog

# Recover a lost commit
git reset --hard HEAD@{2}

# Recover a deleted branch
git branch recovered-branch HEAD@{3}
```

## Verification

After advanced git operations:
1. Run `git status` to ensure clean state
2. Run `git log --oneline -10` to verify commit history
3. For rebase: ensure no conflicts remain
4. For cherry-pick: verify changes are correct
5. For submodules: run `git submodule status` to check
