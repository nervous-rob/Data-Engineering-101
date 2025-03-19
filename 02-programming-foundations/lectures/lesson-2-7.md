# Lesson 2.7: Version Control

## Navigation
- [← Back to Lesson Plan](../2.7-version-control.md)
- [← Back to Module Overview](../README.md)

## Overview

Version control systems (VCS) are essential tools in modern software development and data engineering. They track changes to files over time, enable collaboration among team members, and provide mechanisms to manage different versions of code and configurations. In data engineering, version control is particularly important for maintaining data pipelines, ETL scripts, infrastructure configurations, and documentation. This lesson explores version control concepts and practices with a focus on Git, the most widely used version control system today.

## Learning Objectives
- Understand fundamental version control concepts
- Master Git basic and advanced operations
- Learn collaborative workflows using Git
- Explore best practices for version control in data engineering projects

## Version Control Fundamentals

### What is Version Control?

Version control is a system that records changes to files over time so that you can recall specific versions later. It allows you to:

1. Track changes to files and directories
2. Revert files to previous states
3. Compare changes over time
4. Identify who made specific changes
5. Collaborate with others without overwriting each other's work
6. Maintain multiple versions of a project simultaneously

### Types of Version Control Systems

Version control systems have evolved over time:

1. **Local Version Control Systems**
   - Store file changes on the local machine
   - Examples: RCS (Revision Control System)
   - Limitation: No collaboration capabilities

2. **Centralized Version Control Systems (CVCS)**
   - Store changes on a central server
   - Examples: SVN (Subversion), Perforce
   - Advantages: Better collaboration, administrative control
   - Disadvantages: Single point of failure, dependency on network access

3. **Distributed Version Control Systems (DVCS)**
   - Each user has a complete copy of the repository
   - Examples: Git, Mercurial
   - Advantages: Work offline, multiple backups, flexible workflows
   - Currently the dominant approach for most software development

### Why Git?

Git has become the de facto standard for version control due to its:

1. **Speed and Performance**: Operations are optimized to be fast, even for large projects
2. **Distributed Nature**: Each clone is a full backup of the repository
3. **Branching Model**: Lightweight, powerful branching and merging
4. **Data Integrity**: Uses SHA-1 hashes to ensure data integrity
5. **Ecosystem**: Wide adoption with platforms like GitHub, GitLab, and Bitbucket
6. **Open Source**: Free to use with active community support

## Git Fundamentals

### Core Concepts

To understand Git effectively, it's important to grasp its underlying data model:

1. **Repository (Repo)**: A collection of files and their complete history
2. **Commit**: A snapshot of the repository at a specific point in time
3. **Branch**: A lightweight, movable pointer to a commit
4. **Remote**: A repository stored on another location (like GitHub)
5. **Working Directory**: The files you currently see and edit
6. **Staging Area (Index)**: A middle ground where changes are prepared before committing

### Git's Three States

Files in a Git repository can exist in three states:

1. **Modified**: Changes made to a file but not yet staged or committed
2. **Staged**: Modified files marked to be included in the next commit
3. **Committed**: Data safely stored in the local repository

### Setting Up Git

Before using Git, you need to set it up:

```bash
# Install Git (varies by operating system)
# On Ubuntu/Debian
sudo apt-get install git

# On macOS with Homebrew
brew install git

# On Windows, download and install from git-scm.com

# Configure user information
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"  # For VS Code

# Check configuration
git config --list
```

### Creating and Cloning Repositories

You can start a Git repository in two ways:

```bash
# Initialize a new repository in the current directory
git init

# Clone an existing repository
git clone https://github.com/username/repository.git

# Clone to a specific directory
git clone https://github.com/username/repository.git my-project
```

### Basic Workflow

The typical Git workflow involves:

```bash
# Check status of your repository
git status

# Add changes to the staging area
git add filename.py  # Add a specific file
git add .            # Add all changes in the current directory
git add -p           # Interactively choose which changes to stage

# Commit staged changes
git commit -m "Add functionality to process CSV files"

# View commit history
git log
git log --oneline    # Compact view
git log --graph      # Show branch structure
```

### Working with Branches

Branches allow parallel development and isolation of features:

```bash
# List branches
git branch           # Local branches
git branch -r        # Remote branches
git branch -a        # All branches

# Create a new branch
git branch feature-name

# Switch to a branch
git checkout feature-name

# Create and switch in one command
git checkout -b new-feature

# Rename a branch
git branch -m old-name new-name

# Delete a branch
git branch -d feature-name    # Safe delete (prevents deleting unmerged branches)
git branch -D feature-name    # Force delete
```

### Merging and Rebasing

Combining work from different branches:

```bash
# Merge a branch into the current branch
git checkout main
git merge feature-branch

# Merge with a commit message
git merge feature-branch -m "Merge feature into main"

# Rebase current branch onto another
git checkout feature-branch
git rebase main
```

#### Merge vs. Rebase

- **Merge**: Creates a new "merge commit" that combines changes from both branches
  - Preserves complete history and chronological order
  - Can create complex history with many merge commits

- **Rebase**: Moves the entire branch to begin on the tip of another branch
  - Creates linear history
  - Rewrites commit history, which can cause issues in shared branches

### Handling Conflicts

Conflicts occur when Git can't automatically merge changes:

```bash
# When conflict occurs during merge or rebase
# 1. Files with conflicts will be marked in the working directory
# 2. Edit files to resolve conflicts manually
# 3. Stage the resolved files
git add resolved-file.py

# 4. Complete the merge
git merge --continue
# Or for rebase
git rebase --continue

# Abort the operation if needed
git merge --abort
git rebase --abort
```

Conflict markers in files look like:

```
<<<<<<< HEAD
Code from the current branch
=======
Code from the branch being merged
>>>>>>> feature-branch
```

### Viewing Changes

Git provides several ways to inspect changes:

```bash
# Show changes between working directory and staging area
git diff

# Show changes between staging area and last commit
git diff --staged

# Show changes between two commits
git diff commit1 commit2

# Show changes in a specific file
git diff -- filename.py

# Show changes introduced by a specific commit
git show commit-hash
```

## Advanced Git Operations

### Stashing Changes

Stashing allows you to temporarily store modifications without committing:

```bash
# Stash changes
git stash

# Stash with a message
git stash save "Work in progress on feature X"

# List stashes
git stash list

# Apply the most recent stash without removing it
git stash apply

# Apply a specific stash
git stash apply stash@{2}

# Apply and remove the most recent stash
git stash pop

# Remove a stash
git stash drop stash@{1}

# Clear all stashes
git stash clear
```

### Tagging

Tags mark specific points in history, typically for releases:

```bash
# List tags
git tag

# Create a lightweight tag
git tag v1.0.0

# Create an annotated tag with a message
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag a specific commit
git tag -a v0.9.0 commit-hash

# Push tags to remote
git push origin v1.0.0    # Push specific tag
git push --tags           # Push all tags

# Delete a tag
git tag -d v1.0.0
```

### Interactive Rebase

Interactive rebase allows you to modify commits:

```bash
# Start interactive rebase to modify the last 3 commits
git rebase -i HEAD~3

# Options in interactive rebase include:
# - pick: use the commit as is
# - reword: change the commit message
# - edit: amend the commit
# - squash: combine with previous commit
# - fixup: combine with previous commit, discard message
# - drop: remove the commit
```

### Reflog

Git's reference logs track when references (like branches) change:

```bash
# View reflog
git reflog

# Recover deleted branch using reflog
git checkout -b recovered-branch commit-hash
```

### Cherry-Pick

Cherry-picking applies changes from specific commits to the current branch:

```bash
# Apply a single commit
git cherry-pick commit-hash

# Apply multiple commits
git cherry-pick commit1 commit2 commit3
```

### Reset and Revert

Options for undoing changes:

```bash
# Reset staging area to match most recent commit
git reset

# Reset staging area and working directory
git reset --hard

# Move branch pointer to a specific commit
git reset commit-hash        # Keep changes in working directory
git reset --soft commit-hash # Keep changes staged
git reset --hard commit-hash # Discard all changes

# Create a new commit that undoes changes from another commit
git revert commit-hash
```

### Git Hooks

Hooks are scripts that run automatically on specific Git events:

```bash
# Hooks are stored in .git/hooks directory
# Common hooks include:
# - pre-commit: Run before a commit is created
# - post-commit: Run after a commit is created
# - pre-push: Run before pushing to a remote
# - post-merge: Run after a merge is completed
```

Example pre-commit hook that checks for Python syntax errors:

```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "Checking Python syntax..."
for file in $(git diff --cached --name-only | grep '\.py$'); do
    python -m py_compile "$file"
    if [ $? -ne 0 ]; then
        echo "Python syntax error in $file"
        exit 1
    fi
done
```

## Collaborative Development

### Remote Repositories

Working with remotes allows collaboration:

```bash
# List remote repositories
git remote -v

# Add a remote
git remote add origin https://github.com/username/repo.git

# Remove a remote
git remote remove origin

# Rename a remote
git remote rename origin upstream

# Fetch changes from a remote
git fetch origin

# Pull changes (fetch + merge)
git pull origin main

# Push changes to a remote
git push origin feature-branch

# Set upstream branch
git push -u origin feature-branch
```

### Pull Requests

Pull requests (PRs) are not a Git feature but are provided by platforms like GitHub, GitLab, and Bitbucket. They allow:

1. Proposing changes to a repository
2. Discussing and reviewing code
3. Adding follow-up commits based on feedback
4. Merging changes once approved

Typical workflow:

1. Fork a repository (on the platform)
2. Clone your fork: `git clone https://github.com/your-username/repo.git`
3. Create a branch: `git checkout -b feature-branch`
4. Make changes and commit: `git commit -m "Add feature"`
5. Push to your fork: `git push origin feature-branch`
6. Create a pull request (on the platform)
7. Address review feedback and update PR
8. Maintainer merges the PR

### Collaborative Workflows

Several collaborative workflows are common with Git:

#### 1. Centralized Workflow

- Single main branch (often called "main" or "master")
- All developers commit directly to main
- Simple but can lead to conflicts with larger teams

#### 2. Feature Branch Workflow

- Main branch contains stable code
- New features developed in dedicated branches
- Features merged back to main when complete
- Allows parallel development without affecting stability

#### 3. Gitflow Workflow

- Two main branches: main (production) and develop (integration)
- Feature branches branch off develop
- Release branches prepare for production
- Hotfix branches fix production issues
- More structured but complex for simple projects

#### 4. Forking Workflow

- Each developer forks the main repository
- Changes made in personal fork
- Pull requests used to propose changes to original repo
- Common in open-source projects

### Code Review Best Practices

Code reviews are essential for collaborative development:

1. **Keep reviews small**: Aim for <400 lines of code per review
2. **Focus on readability and maintainability**: Is the code clear?
3. **Look for edge cases**: Consider unexpected inputs and error handling
4. **Verify correctness**: Does the code work as intended?
5. **Be constructive**: Provide specific, actionable feedback
6. **Automate where possible**: Use linters and automated tests

## Git for Data Engineering

### Version Control for Data Engineering Assets

Data engineering projects involve various assets that benefit from version control:

1. **Code Assets**:
   - ETL scripts
   - Data transformation code
   - Pipeline definitions
   - SQL queries and migrations

2. **Configuration Assets**:
   - Infrastructure-as-Code (IaC) templates
   - Configuration files
   - Environment variables (template files)
   - Authentication settings (templates, not credentials)

3. **Documentation**:
   - Data dictionaries
   - Data lineage diagrams
   - Architecture diagrams
   - README files and wikis

### Handling Large Files

Data engineering often involves large files that don't work well with Git:

```bash
# Git Large File Storage (LFS) extension
# Install Git LFS
git lfs install

# Track specific file patterns
git lfs track "*.parquet"
git lfs track "*.csv"

# Make sure .gitattributes is tracked
git add .gitattributes

# Use normally
git add large-file.parquet
git commit -m "Add large parquet file"
```

### Managing Secrets

Never store sensitive information in Git:

1. Use environment variables for sensitive values
2. Implement `.gitignore` to exclude secret files
3. Consider tools like git-crypt or git-secret for encrypted files
4. Use secret management services like HashiCorp Vault or AWS Secrets Manager

Example `.gitignore` for data engineering:

```
# Credentials
*.pem
*.key
*.env
credentials.yaml
secret.json

# Large data files
*.csv
*.parquet
*.avro
*.orc
data/

# Temporary files
*.tmp
.ipynb_checkpoints/
__pycache__/
```

### Versioning Data Models and Schemas

Track changes to data schemas:

1. Store database migration scripts in Git
2. Version data model definitions
3. Document schema changes in commit messages
4. Consider using database migration tools that integrate with Git

### CI/CD Integration

Git works well with CI/CD pipelines for data engineering:

1. Trigger data pipeline tests on PR creation
2. Automatically deploy pipelines after merge to main
3. Run data validation checks on schema changes
4. Generate documentation from code

## Best Practices

### Commit Messages

Good commit messages improve collaboration and historical understanding:

```
# Format: <type>: <subject>
# 
# <body>
# 
# <footer>

feat: add CSV validation to data ingestion pipeline

- Implements header validation against schema
- Adds data type checking for each column
- Includes configurable error thresholds

Closes #123
```

Common commit types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting changes
- `refactor`: Code restructuring
- `test`: Adding/updating tests
- `chore`: Maintenance tasks

### Branching Strategy

Effective branching helps manage development:

1. **Keep main branch stable**: Only merge tested, reviewed code
2. **Use descriptive branch names**: `feature/add-validation`, `fix/csv-parsing-error`
3. **Delete branches after merging**: Keeps repository clean
4. **Protect important branches**: Use branch protection rules
5. **Consider using release branches**: Separate development from release preparation

### Repository Organization

Structure your repositories effectively:

1. **Monorepo vs. Multiple Repositories**:
   - Monorepo: All related code in one repository
   - Multiple repos: Separate repositories for different components

2. **Data Engineering Repository Structure**:
   ```
   data-pipeline/
   ├── .github/            # GitHub specific files (workflows, templates)
   ├── docs/               # Documentation
   ├── src/                # Source code
   │   ├── extractors/     # Data extraction components
   │   ├── transformers/   # Data transformation components
   │   └── loaders/        # Data loading components
   ├── tests/              # Test files
   ├── config/             # Configuration files
   ├── scripts/            # Utility scripts
   ├── .gitignore          # Git ignore file
   ├── README.md           # Repository documentation
   └── requirements.txt    # Dependencies
   ```

### Documentation

Documentation should be treated as code:

1. **README files**: Provide overview, setup instructions, and usage examples
2. **Code comments**: Explain "why" not "what"
3. **Architecture diagrams**: Visualize system components
4. **Contribution guidelines**: Help others collaborate effectively

### .gitignore for Data Engineering

Properly configured `.gitignore` files prevent unwanted files from being tracked:

```
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# Distribution / packaging
dist/
build/
*.egg-info/

# Virtual environments
venv/
env/
.env/

# Data files
*.csv
*.tsv
*.parquet
*.avro
*.orc
*.json
data/

# Notebooks
.ipynb_checkpoints/

# Logs
logs/
*.log

# Local configuration
.env
.env.local
config.local.yaml
credentials.json

# IDE specific files
.idea/
.vscode/
*.swp
*.swo

# OS specific files
.DS_Store
Thumbs.db
```

## Practical Examples

### Example 1: Setting Up a Data Pipeline Project

```bash
# Create a new repository
mkdir data-pipeline
cd data-pipeline
git init

# Create basic structure
mkdir -p src/{extractors,transformers,loaders} tests config docs

# Create initial files
touch README.md
touch .gitignore
touch requirements.txt
touch src/__init__.py

# Add files to staging area
git add .

# Initial commit
git commit -m "Initial project structure"

# Create development branch
git checkout -b develop

# Add a feature branch
git checkout -b feature/csv-extractor

# After making changes to implement CSV extractor
git add src/extractors/csv_extractor.py tests/test_csv_extractor.py
git commit -m "feat: implement CSV extractor with header validation"

# Push to remote (assuming remote is set up)
git push -u origin feature/csv-extractor
```

### Example 2: Collaborative Bug Fix

```bash
# Fetch latest changes from remote
git fetch origin

# Create bug fix branch from main
git checkout -b fix/date-parsing-error origin/main

# Fix the issue
# (Make changes to the code)

# Add changes
git add src/transformers/date_transformer.py

# Commit with descriptive message
git commit -m "fix: correct date parsing for non-standard formats

The date parser was failing when encountering dates in MM/DD/YYYY format.
This fix adds support for multiple date formats and prioritizes the
format specified in the configuration.

Fixes #456"

# Push to remote
git push -u origin fix/date-parsing-error

# Create pull request (on GitHub/GitLab/etc.)
# After review and approval, merge to main
```

### Example 3: Managing Schema Migrations

```bash
# Create branch for schema change
git checkout -b feature/add-user-preferences

# Add migration script
mkdir -p migrations
touch migrations/V2023_07_15__add_user_preferences.sql

# Add the SQL for the migration
# (Edit the migration file)

# Update related code to use new schema
# (Edit other files)

# Commit changes
git add migrations/V2023_07_15__add_user_preferences.sql
git add src/models/user.py
git commit -m "feat: add user preferences table and model

This change introduces a new user_preferences table to store
user-specific settings and preferences. The migration includes:
- New table creation
- Foreign key relationship to users table
- Default preferences population

The user model has been updated to include preference handling."

# Push and create PR for review
git push -u origin feature/add-user-preferences
```

### Example 4: Handling Configuration Changes

```bash
# Template for configuration file
touch config/config.template.yaml

# Add template with placeholder values
# (Edit config.template.yaml)

# Add local configuration to gitignore
echo "config/config.local.yaml" >> .gitignore

# Create local configuration file (not tracked)
cp config/config.template.yaml config/config.local.yaml

# Commit template
git add config/config.template.yaml .gitignore
git commit -m "chore: add configuration template with documentation

Adds a configuration template file with documented options
and placeholder values. Local configuration should be created
as config/config.local.yaml which is excluded from version control."
```

## Git Tools and Extensions

Several tools enhance Git for data engineering:

1. **Git LFS**: Handles large files efficiently
2. **GitKraken/Sourcetree**: Visual Git clients for easier visualization
3. **pre-commit**: Framework for managing Git hooks
4. **git-crypt**: Transparent encryption for sensitive files
5. **git-secrets**: Prevents committing secrets and credentials
6. **dvc (Data Version Control)**: Git extension for data version control
7. **GitLab/GitHub/Bitbucket**: Platforms with CI/CD, issue tracking, and more

## Conclusion

Version control with Git is an essential skill for data engineers. By effectively managing code, configurations, and documentation, you can:

1. Collaborate efficiently with team members
2. Track changes and understand project history
3. Maintain stable production systems while developing new features
4. Recover from errors and issues
5. Automate testing and deployment

Remember that version control is not just about the technical aspects of Git—it's about establishing effective workflows and communication patterns that enable teams to work together efficiently. As a data engineer, investing time in mastering Git and developing good version control habits will pay dividends throughout your career.

## Resources

- [Pro Git Book](https://git-scm.com/book/en/v2) - Comprehensive and free Git reference
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf) - Quick reference for common commands
- [Oh Shit, Git!?!](https://ohshitgit.com/) - Solutions for common Git mistakes
- [Learn Git Branching](https://learngitbranching.js.org/) - Interactive Git visualization tool
- [GitHub Learning Lab](https://lab.github.com/) - Interactive courses for Git and GitHub

## Next Steps

- Practice basic Git operations on a personal project
- Contribute to an open-source project
- Set up branch protection rules and code review processes
- Implement CI/CD pipelines with Git integration
- Explore Git integrations with data engineering tools