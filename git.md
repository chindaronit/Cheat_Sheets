# Git Cheat Sheet

| **Category**     | **Command**                                          | **Description**                          |
| ---------------- | ---------------------------------------------------- | ---------------------------------------- |
| **Setup**        | `git config --global user.name "Name"`               | Set **username**                         |
|                  | `git config --global user.email "email@example.com"` | Set **email**                            |
|                  | `git config --list`                                  | Show **configuration**                   |
| **Repository**   | `git init`                                           | Initialize **new repo**                  |
|                  | `git clone <url>`                                    | Clone **remote repo**                    |
|                  | `git status`                                         | Show **file status**                     |
|                  | `git remote -v`                                      | List **remote repos**                    |
|                  | `git remote add origin <url>`                        | Add **remote repo**                      |
| **Workflow**     | `git add <file>`                                     | **Stage file**                           |
|                  | `git add .`                                          | Stage **all changes**                    |
|                  | `git commit -m "msg"`                                | Commit **staged changes**                |
|                  | `git commit -am "msg"`                               | Stage & commit **tracked files**         |
|                  | `git push`                                           | Push to **remote**                       |
|                  | `git pull`                                           | Pull from **remote**                     |
| **Branching**    | `git branch`                                         | List **branches**                        |
|                  | `git branch <name>`                                  | Create **new branch**                    |
|                  | `git checkout <name>`                                | Switch to **branch**                     |
|                  | `git checkout -b <name>`                             | Create & switch **branch**               |
|                  | `git merge <branch>`                                 | Merge **branch into current**            |
|                  | `git branch -d <name>`                               | Delete **branch**                        |
| **History**      | `git log`                                            | Show **commit history**                  |
|                  | `git log --oneline`                                  | Show **compact history**                 |
|                  | `git log --graph --oneline --all`                    | Show **graph view**                      |
|                  | `git show <commit>`                                  | Show **commit details**                  |
| **Undo Changes** | `git restore <file>`                                 | Discard **working directory changes**    |
|                  | `git restore --staged <file>`                        | Unstage **file**                         |
|                  | `git reset --soft <commit>`                          | Undo commit, **keep staged changes**     |
|                  | `git reset --hard <commit>`                          | Undo commit, **discard changes**         |
|                  | `git revert <commit>`                                | Revert commit by creating **new commit** |
| **Stashing**     | `git stash`                                          | Save **changes temporarily**             |
|                  | `git stash list`                                     | List **stashes**                         |
|                  | `git stash apply`                                    | Apply **latest stash**                   |
|                  | `git stash pop`                                      | Apply & remove **latest stash**          |
|                  | `git stash drop`                                     | Delete **stash**                         |
| **Remote**       | `git fetch origin`                                   | Fetch **changes from remote**            |
|                  | `git pull origin <branch>`                           | Pull **branch**                          |
|                  | `git push origin <branch>`                           | Push **branch**                          |
|                  | `git push -u origin <branch>`                        | Push & set **upstream**                  |
| **Tags**         | `git tag`                                            | List **tags**                            |
|                  | `git tag <name>`                                     | Create **tag**                           |
|                  | `git tag -a <name> -m "msg"`                         | Create **annotated tag**                 |
|                  | `git push origin <tag>`                              | Push **tag**                             |
|                  | `git push origin --tags`                             | Push **all tags**                        |
| **Shortcuts**    | `git diff`                                           | Show **changes**                         |
|                  | `git diff --staged`                                  | Show **staged changes**                  |
|                  | `git log -p`                                         | Show commits **with diffs**              |
|                  | `git clean -fd`                                      | Remove **untracked files & directories** |
|                  | `git config --global alias.co checkout`              | Create **alias** (`co`) for checkout     |

## 🔹Advanced

### 🔹 Branching & Merging

```bash
git merge --no-ff <branch>    # Merge without fast-forward (preserve branch history)
git merge --squash <branch>   # Merge branch as a single commit
git rebase <branch>           # Rebase current branch onto another
git rebase -i <commit>        # Interactive rebase for editing commits
git cherry-pick <commit>      # Apply specific commit to current branch
git branch --contains <commit> # Show branches containing a commit
git reflog                    # Show all actions and HEAD changes
```

### Git Merge vs Git Rebase

`git merge` combines the histories of two branches into one.

```
A---B---C---------M
         \       /
          D-----E
```

**Pros**

- Preserves the true history of how development happened.
- Safe for collaborative work—history remains intact.
- Easy to see when branches were merged.

**Cons**

- History can get messy with many merge commits.
- Harder to read linear progression of commits.

`git rebase` moves or “replays” commits from one branch onto another, creating a linear history.

Git rewrites the feature branch commits D and E, putting them on top of main:

```
A---B---C---D'---E'
```

- The original commits D and E are replaced by D' and E' with new hashes.

**Pros**

- Produces clean, linear history.
- Easier to read and navigate with git log.
- Ideal for small feature branches before merging into main.

**Cons**

- Rewrites history—dangerous if the branch is already pushed/shared.
- Can cause conflicts that need to be resolved manually for each commit.

### 🔹 Undo / History Editing

```bash
git reset --mixed <commit>    # Reset HEAD, keep working directory
git reset --keep <commit>     # Reset while keeping uncommitted changes that do not conflict
git revert -n <commit>        # Revert without committing immediately
git commit --amend             # Modify the last commit (message or content)
git filter-branch --tree-filter 'rm -rf <file>' HEAD # Remove a file from history
git rebase --onto <newbase> <upstream> <branch>     # Move branch onto another base
```

`git revert` – “Undo by creating a new commit”

_Suppose your commit history looks like this:_

```
A --- B --- C   (HEAD)
```

_You want to undo commit B_:
`git revert B`

- Git creates a new commit (say D) that reverses the changes made in B

```
A --- B --- C --- D
```

- **Note**: Nothing is removed from history; the undo is recorded as a new commit.

`git reset` – “Move HEAD to an earlier commit”

- Moves the current branch pointer (HEAD) to a previous commit.
- Can optionally **keep or discard** changes in your working directory and
  staging area.
- Rewrites history, so **dangerous on shared branches**.

**Types of reset** \
`--soft` Moves HEAD to commit and
Keeps staged changes (index) \
`--mixed (default)` Moves HEAD and Unstages changes but keeps them in working directory \
`--hard` Moves HEAD and Discard all changes in working directory and staging

**Example**

_Commit history:_

```
A --- B --- C   (HEAD)
```

- Soft reset to `B`:

```bash
git reset --soft B # HEAD moves to B. Changes from C remain staged.
```

- Hard reset to `B`:

```bash
git reset --hard B # HEAD moves to B. Changes from C are gone completely.
```

### 🔹Stashing & Work-in-Progress

```bash
git stash save "message"       # Save changes with message
git stash branch <branch>      # Create branch from stash
git stash apply stash@{n}      # Apply specific stash
git stash drop stash@{n}       # Drop specific stash
git stash clear                # Drop all stashes
```

### Git Stash

Git Stash is a way to temporarily save changes in your working directory that you are not ready to commit yet. It’s like putting your changes in a “drawer” so you can switch branches or work on something else without losing your work.

**Why use Git Stash?**

- You’re working on a feature but need to switch branches to fix a bug.
- You have unfinished changes but don’t want to commit a messy or incomplete state.
- You want to test something quickly without committing the current changes.

Git will allow the switch only if the changes do not conflict with files in other branch. That's why git stash becomes important on stashing your changes it cleans your working directory allowing you to switch branches ( even in conflicts ) and do changes come back and apply the temporarily saved code.

### 🔹Remote & Collaboration

```bash
git remote show origin         # Show remote details
git fetch --all                # Fetch all remotes
git pull --rebase              # Rebase instead of merge when pulling
git push --force-with-lease    # Force push safely
git push --tags                # Push all tags
git remote prune origin        # Remove deleted remote branches
```

### 🔹Diff & Log Analysis

```bash
git diff <commit1> <commit2>       # Compare two commits
git diff --stat                     # Show summary of changes
git diff --name-only                # Show only changed file names
git log --stat                       # Commit log with file stats
git log --since="2 weeks ago"       # Filter log by date
git log --author="Author Name"      # Filter by author
git blame <file>                    # Show last modification per line
```
