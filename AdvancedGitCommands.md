# Advanced Git Commands Reference

## Advanced Branching Operations

1. Create and switch to feature branch:
   ```bash
   git checkout -b feature/new-feature
   ```

2. Merge feature branch into main:
   ```bash
   git checkout main
   git merge feature/new-feature
   ```

3. Handle merge conflicts:
   ```bash
   git status                    # Check which files have conflicts
   git diff                      # View the conflicts
   # After manually resolving conflicts:
   git add .                     
   git commit -m "Resolved merge conflicts"
   ```

4. Delete feature branch after merging:
   ```bash
   git branch -d feature/new-feature    # Local branch deletion
   git push origin --delete feature/new-feature    # Remote branch deletion
   ```

5. Update feature branch with main changes:
   ```bash
   git checkout feature/new-feature
   git rebase main
   ```


## Git Stash Operations

1. Save current changes to stash:
   ```bash
   git stash save "work in progress on feature X"
   ```

2. List all stashes:
   ```bash
   git stash list
   ```

3. Apply most recent stash:
   ```bash
   git stash apply    # Keeps stash in list
   git stash pop      # Removes stash after applying
   ```

4. Apply specific stash:
   ```bash
   git stash apply stash@{n}    # n is stash index number
   ```

5. Create branch from stash:
   ```bash
   git stash branch new-branch stash@{n}
   ```

6. Remove stashes:
   ```bash
   git stash drop stash@{n}    # Remove specific stash
   git stash clear             # Remove all stashes
   ```




## Git Log and History Operations

1. View commit history:
   ```bash
   git log                          # Full commit history
   git log --oneline               # Compact view
   git log --graph --oneline       # Visual commit tree
   ```

2. Search commit history:
   ```bash
   git log --grep="keyword"        # Search commit messages
   git log -S"code string"         # Search for code changes
   git log --author="name"         # Search by author
   ```

3. Compare changes:
   ```bash
   git diff commit1..commit2       # Changes between commits
   git diff branch1..branch2      # Changes between branches
   git show commit_hash           # View specific commit changes
   ```

4. View file history:
   ```bash
   git log --follow filename      # Track file history through renames
   git blame filename            # See who changed each line
   ```





## Remote Repository Management

1. Working with multiple remotes:
   ```bash
   git remote -v                                    # List all remotes
   git remote add upstream repository-url          # Add new remote
   git remote rename origin new-name              # Rename remote
   git remote remove remote-name                  # Remove remote
   ```

2. Advanced remote operations:
   ```bash
   git fetch --all                                # Update all remotes
   git remote prune origin                       # Clean up deleted remote branches
   git push --all origin                         # Push all branches
   ```

3. Remote branch management:
   ```bash
   git branch -r                                  # List remote branches
   git checkout -t origin/branch-name            # Track remote branch
   git push origin --delete branch-name          # Delete remote branch
   ```

4. Syncing with upstream:
   ```bash
   git fetch upstream                            # Get upstream changes
   git merge upstream/main                       # Merge upstream changes
   git rebase upstream/main                      # Rebase on upstream
   ```






## Git Configuration Management

1. Global user settings:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   git config --list                              # View all settings
   ```

2. SSH key setup:
   ```bash
   ssh-keygen -t ed25519 -C "your.email@example.com"
   ssh-add ~/.ssh/id_ed25519
   ```

3. Create useful aliases:
   ```bash
   git config --global alias.st status
   git config --global alias.co checkout
   git config --global alias.br branch
   git config --global alias.ci commit
   ```

4. Set default behaviors:
   ```bash
   git config --global core.editor "code --wait"  # Set VS Code as editor
   git config --global pull.rebase true          # Auto rebase on pull
   git config --global core.autocrlf true        # Line ending handling
   ```









