# `git` cheatsheet

A task-oriented reference for everyday Git. Commands are grouped by what you're
trying to do; the "Undo & recover" section is the one worth bookmarking.

## Setup & config

| Command | What it does |
| --- | --- |
| `git config --global user.name "Name"` | Set your commit author name |
| `git config --global user.email "you@x.com"` | Set your commit email |
| `git config --global init.defaultBranch main` | Default branch name for `init` |
| `git config --global pull.rebase true` | Rebase instead of merge on `pull` |
| `git config --list --show-origin` | Show all settings and which file set them |
| `git config --global alias.co checkout` | Create an alias (`git co`) |

## Starting & cloning

| Command | What it does |
| --- | --- |
| `git init` | Create a repo in the current directory |
| `git clone <url>` | Clone a remote repo |
| `git clone --depth 1 <url>` | Shallow clone (latest commit only, faster) |
| `git clone -b <branch> <url>` | Clone a specific branch |

## Status, staging & committing

| Command | What it does |
| --- | --- |
| `git status` | Show working tree status |
| `git status -s` | Short status format |
| `git add <file>` | Stage a file |
| `git add -A` | Stage all changes (incl. deletions) |
| `git add -p` | Interactively stage hunks |
| `git restore --staged <file>` | Unstage a file (keep changes) |
| `git commit -m "msg"` | Commit staged changes |
| `git commit -a -m "msg"` | Stage tracked changes and commit |
| `git commit --amend` | Replace the last commit (edit message/contents) |
| `git commit --amend --no-edit` | Add staged changes to last commit, keep message |

## Branches

| Command | What it does |
| --- | --- |
| `git branch` | List local branches |
| `git branch -a` | List local + remote branches |
| `git switch <branch>` | Switch to a branch |
| `git switch -c <branch>` | Create and switch to a new branch |
| `git branch -d <branch>` | Delete a merged branch (`-D` to force) |
| `git branch -m <old> <new>` | Rename a branch |
| `git merge <branch>` | Merge a branch into the current one |

> `switch` / `restore` are the modern split of the old, overloaded `checkout`,
> which still works: `git checkout -b <branch>`, `git checkout <file>`.

## Remotes, push & pull

| Command | What it does |
| --- | --- |
| `git remote -v` | List remotes and their URLs |
| `git remote add origin <url>` | Add a remote named `origin` |
| `git fetch` | Download remote changes without merging |
| `git pull` | Fetch + merge (or rebase) into current branch |
| `git push` | Push current branch to its upstream |
| `git push -u origin <branch>` | Push and set upstream tracking |
| `git push --force-with-lease` | Force-push safely (aborts if remote moved) |

## Inspecting history

| Command | What it does |
| --- | --- |
| `git log --oneline --graph --all` | Compact, visual branch history |
| `git log -p <file>` | Show full diff of changes to a file |
| `git log --author="Name"` | Filter commits by author |
| `git log --since="2 weeks ago"` | Filter commits by date |
| `git show <commit>` | Show a commit's message and diff |
| `git blame <file>` | Who last changed each line (`-w` ignores whitespace) |
| `git diff` | Unstaged changes |
| `git diff --staged` | Staged changes (vs last commit) |
| `git diff <a> <b>` | Difference between two commits/branches |

## Stashing

| Command | What it does |
| --- | --- |
| `git stash` | Shelve uncommitted changes, clean the tree |
| `git stash -u` | Include untracked files |
| `git stash list` | List stashes |
| `git stash pop` | Reapply the latest stash and drop it |
| `git stash apply` | Reapply but keep the stash |
| `git stash drop` | Delete a stash |

## Undo & recover

| Situation | Command |
| --- | --- |
| Discard unstaged changes to a file | `git restore <file>` |
| Unstage a file (keep edits) | `git restore --staged <file>` |
| Fix the last commit message | `git commit --amend` |
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` |
| Undo last commit, keep changes unstaged | `git reset HEAD~1` |
| Discard last commit **and** its changes | `git reset --hard HEAD~1` |
| Undo a pushed commit safely (new commit) | `git revert <commit>` |
| Restore a deleted branch / lost commit | `git reflog` then `git switch -c <name> <hash>` |

> `git reflog` is your safety net: it records where `HEAD` has been, so even
> after a bad `reset --hard` you can usually find the lost commit's hash and
> recover it.

## Tags

| Command | What it does |
| --- | --- |
| `git tag` | List tags |
| `git tag -a v1.0 -m "msg"` | Create an annotated tag |
| `git push origin v1.0` | Push a tag (`--tags` pushes all) |

## Rebase & history surgery

| Command | What it does |
| --- | --- |
| `git rebase <branch>` | Replay current branch's commits onto another |
| `git rebase -i HEAD~3` | Interactively squash/reorder/edit last 3 commits |
| `git rebase --abort` | Bail out of an in-progress rebase |
| `git cherry-pick <commit>` | Apply a single commit onto the current branch |

> Don't rebase commits you've already pushed and shared — it rewrites history.

## Finding things

| Command | What it does |
| --- | --- |
| `git grep "pattern"` | Search tracked files for a pattern |
| `git bisect start` / `good` / `bad` | Binary-search history for the commit that broke something |
| `git log -S"text"` | Find commits that added/removed a string ("pickaxe") |

## Cleaning

| Command | What it does |
| --- | --- |
| `git clean -n` | Dry-run: list untracked files that would be removed |
| `git clean -fd` | Remove untracked files and directories |
| `git clean -fdx` | Also remove `.gitignore`d files (truly pristine) |

## GitHub CLI (`gh`)

`gh` is GitHub's official command-line tool for repos, pull requests, issues,
and more — it talks to GitHub's API so you can stay in the terminal.

### Auth & setup

| Command | What it does |
| --- | --- |
| `gh auth login` | Authenticate with GitHub (browser or token) |
| `gh auth status` | Show who you're logged in as |
| `gh auth setup-git` | Use `gh` as the credential helper for `git` |

### Repositories

| Command | What it does |
| --- | --- |
| `gh repo create <name>` | Create a new GitHub repo |
| `gh repo clone <owner>/<repo>` | Clone a repo |
| `gh repo fork <owner>/<repo>` | Fork a repo (and optionally clone it) |
| `gh repo view --web` | Open the current repo in a browser |

### Pull requests

| Command | What it does |
| --- | --- |
| `gh pr create` | Open a PR from the current branch (interactive) |
| `gh pr create --fill` | Use commit info for the title/body |
| `gh pr list` | List open PRs |
| `gh pr status` | Show PRs relevant to you |
| `gh pr view <n>` | View a PR (`--web` to open in browser) |
| `gh pr checkout <n>` | Check out a PR branch locally |
| `gh pr diff <n>` | Show a PR's diff |
| `gh pr review --approve` | Approve a PR (`--request-changes`, `--comment`) |
| `gh pr merge <n>` | Merge a PR (`--squash`, `--rebase`, `--delete-branch`) |

### Issues

| Command | What it does |
| --- | --- |
| `gh issue create` | Open a new issue (interactive) |
| `gh issue list` | List open issues |
| `gh issue view <n>` | View an issue (`--web` to open in browser) |
| `gh issue close <n>` | Close an issue |

### Other handy bits

| Command | What it does |
| --- | --- |
| `gh run list` | List recent GitHub Actions workflow runs |
| `gh run watch` | Live-watch a running workflow |
| `gh release create v1.0` | Create a release (attach files as extra args) |
| `gh gist create <file>` | Create a gist from a file |
| `gh api <endpoint>` | Call any GitHub REST/GraphQL API endpoint |

---

> Git commands are stable across versions; `git <command> --help` (or
> `git help <command>`) opens the full manual for any command. For `gh`, use
> `gh <command> --help`.
