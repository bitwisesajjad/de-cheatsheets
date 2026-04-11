# Git Cheat Sheet

A practical reference for Git commands used in day-to-day development and data engineering workflows.

---

## First-Time Setup

### Configure your identity
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Set default branch name to main
```bash
git config --global init.defaultBranch main
```

### Set your default editor
```bash
git config --global core.editor "code --wait"   # VS Code
git config --global core.editor "nano"           # nano
```

### Check your current config
```bash
git config --list
```

---

## Starting a Repository

### Initialize a new local repo
```bash
git init
```

### Clone an existing remote repo
```bash
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my_folder   # clone into a specific folder
```

---

## Daily Workflow

### Check the status of your working directory
```bash
git status
```

### See what changed in your files
```bash
git diff                        # unstaged changes
git diff --staged               # staged changes ready to commit
```

### Stage files
```bash
git add file.py                 # stage a specific file
git add .                       # stage everything in current directory
git add src/                    # stage an entire folder
git add *.py                    # stage all Python files
```

### Commit staged changes
```bash
git commit -m "your message"
git commit -am "message"        # stage all tracked files and commit in one step
```

### View commit history
```bash
git log
git log --oneline               # compact one-line view
git log --oneline --graph       # visual branch graph
git log --oneline -10           # last 10 commits
```

---

## Pushing for the First Time

### Connect a local repo to a remote
```bash
git remote add origin https://github.com/user/repo.git
```

### Push and set the upstream tracking branch
```bash
git push -u origin main
```

### After the first push — just use
```bash
git push
```

---

## Working with Remotes

### List remotes
```bash
git remote -v
```

### Pull latest changes from remote
```bash
git pull
```

### Fetch without merging
```bash
git fetch origin
```

### Push to remote
```bash
git push
git push origin branch_name    # push a specific branch
```

### Remove a remote
```bash
git remote remove origin
```

---

## Branching

### List all branches
```bash
git branch                      # local branches
git branch -a                   # local and remote branches
```

### Create a new branch
```bash
git branch feature/my-feature
```

### Switch to a branch
```bash
git checkout feature/my-feature
git switch feature/my-feature   # modern syntax
```

### Create and switch in one step
```bash
git checkout -b feature/my-feature
git switch -c feature/my-feature    # modern syntax
```

### Rename a branch
```bash
git branch -m old-name new-name
```

### Delete a branch
```bash
git branch -d feature/my-feature        # safe delete (merged only)
git branch -D feature/my-feature        # force delete
git push origin --delete feature/my-feature   # delete from remote
```

---

## Merging & Rebasing

### Merge a branch into current branch
```bash
git merge feature/my-feature
```

### Merge without fast-forward (keeps branch history visible)
```bash
git merge --no-ff feature/my-feature
```

### Rebase current branch onto main
```bash
git rebase main
```

### Abort a rebase in progress
```bash
git rebase --abort
```

---

## Undoing Things

### Unstage a file (keep changes)
```bash
git restore --staged file.py
```

### Discard changes in a file (revert to last commit)
```bash
git restore file.py
```

### Undo the last commit but keep changes staged
```bash
git reset --soft HEAD~1
```

### Undo the last commit and unstage changes
```bash
git reset HEAD~1
```

### Undo the last commit and discard all changes
```bash
git reset --hard HEAD~1
```

### Revert a commit by creating a new undo commit
```bash
git revert abc1234
```

> Use `revert` on shared/public branches. Use `reset` only on local or private branches.

### Fix the last commit message
```bash
git commit --amend -m "corrected message"
```

### Add a forgotten file to the last commit
```bash
git add forgotten_file.py
git commit --amend --no-edit
```

---

## Stashing

### Save current changes temporarily without committing
```bash
git stash
git stash push -m "work in progress on feature x"
```

### List stashes
```bash
git stash list
```

### Apply the latest stash and keep it in the list
```bash
git stash apply
```

### Apply the latest stash and remove it from the list
```bash
git stash pop
```

### Drop a specific stash
```bash
git stash drop stash@{0}
```

---

## Tagging

### Create a tag at the current commit
```bash
git tag v1.0.0
git tag -a v1.0.0 -m "release version 1.0.0"   # annotated tag
```

### Push tags to remote
```bash
git push origin v1.0.0
git push origin --tags              # push all tags
```

### List all tags
```bash
git tag
```

---

## Viewing & Comparing

### Show details of a specific commit
```bash
git show abc1234
```

### Compare two branches
```bash
git diff main feature/my-feature
```

### See which branch a commit is on
```bash
git branch --contains abc1234
```

### Find who changed a line — blame
```bash
git blame file.py
```

---

## .gitignore

### Common entries for a data engineering project
```gitignore
# Python
__pycache__/
*.pyc
*.pyo
.venv/
venv/
*.egg-info/

# Environment & secrets
.env
.env.*
!.env.example

# Data files — don't commit raw data
data/raw/
data/processed/
*.csv
*.parquet
*.json
*.gz

# Notebooks checkpoints
.ipynb_checkpoints/

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Logs
logs/
*.log

# Models
models/
*.pkl
*.joblib
```

### Tell Git to stop tracking a file already committed
```bash
git rm --cached file.csv
git rm --cached -r data/raw/
```

---

## ETL Patterns

### Typical first push of a new project
```bash
git init
git add .
git commit -m "initial commit"
git remote add origin https://github.com/user/repo.git
git push -u origin main
```

### Daily feature branch workflow
```bash
git switch -c feature/add-ingestion-step
# make changes
git add .
git commit -m "add ingestion step for orders data"
git push origin feature/add-ingestion-step
# open a pull request on GitHub
```

### Sync your branch with the latest main before merging
```bash
git switch main
git pull
git switch feature/my-feature
git rebase main
```

### Accidentally committed a secret or large file — remove it from history
```bash
git rm --cached .env
echo ".env" >> .gitignore
git add .gitignore
git commit -m "remove .env and add to gitignore"
git push --force
```

---

*Last updated: 2026*
