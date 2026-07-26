# `lazygit` cheatsheet

A terminal UI for Git: stage individual lines, rewrite history, and juggle
branches with single keypresses instead of long `git` invocations.

> **The cheatsheet inside lazygit:** press **`?`** (or `x`) to open the menu for
> whatever panel you're in — it lists every binding *for the current context*,
> so it's always the authoritative reference. `Esc` closes it.

## Starting it

| Command | What it does |
| --- | --- |
| `lazygit` | Open the repo in the current directory |
| `lazygit -p <path>` | Open a repo elsewhere |
| `lazygit -f <file>` | Filter mode: only the history of one file |
| `lazygit status` / `branch` / `log` / `stash` | Start focused on that panel |
| `lazygit -g <dir> -w <tree>` | Explicit `--git-dir` / `--work-tree` |
| `lazygit -cd` | Print the config directory |
| `lazygit -c` | Print the default config (handy as a template) |
| `lazygit --version` | Show version |

## The panels

Five side panels on the left, one main view on the right. Jump straight to a
panel with its number:

| Key | Panel | Tabs (cycle with `[` / `]`) |
| --- | --- | --- |
| `1` | Status | — |
| `2` | Files | Files, Submodules |
| `3` | Branches | Local Branches, Remotes, Tags, Worktrees |
| `4` | Commits | Commits, Reflog |
| `5` | Stash | — |

## Navigation (everywhere)

| Key | Action |
| --- | --- |
| `j` / `k`, `↓` / `↑` | Move down / up in a list |
| `Tab` | Next panel |
| `[` / `]` | Previous / next tab |
| `<` / `>` | Jump to top / bottom of the list |
| `,` / `.` | Previous / next page |
| `PgUp` / `PgDn` | Scroll the main (right-hand) view |
| `H` / `L` | Scroll the view left / right |
| `/` | Search the current view |
| `v` | Toggle range select (then move to extend) |
| `Enter` | Drill into the selected item |
| `Esc` | Go back / cancel |
| `q` | Quit |

## Global commands

| Key | Action |
| --- | --- |
| `?` / `x` | Open the context menu (all bindings for this panel) |
| `R` | Refresh |
| `p` / `P` | Pull / push |
| `z` / `Ctrl-z` | Undo / redo (reflog-backed — see below) |
| `m` | Merge/rebase options (continue, abort) |
| `:` | Run an arbitrary shell command |
| `@` | Command log menu (shows the `git` commands lazygit ran) |
| `+` / `_` | Next / previous screen mode (normal, half, full) |
| `{` / `}` | Less / more context lines in diffs |
| `Ctrl-w` | Toggle ignoring whitespace in diffs |
| `Ctrl-s` | Filter by path |
| `W` / `Ctrl-e` | Diff menu (diff any two refs) |
| `Ctrl-p` | Custom patch options |
| `Ctrl-r` | Switch to a recent repo |
| `Ctrl-o` | Copy the selected item (hash, branch, filename) to the clipboard |

## Files panel (`2`)

| Key | Action |
| --- | --- |
| `Space` | Stage / unstage the file |
| `a` | Stage / unstage everything |
| `Enter` | Drill in to stage individual hunks or lines |
| `c` | Commit staged changes |
| `w` | Commit skipping pre-commit hooks |
| `C` | Commit using your `$EDITOR` |
| `A` | Amend the last commit with what's staged |
| `d` | Discard changes (menu: all / unstaged / staged) |
| `D` | Reset options (nuke the working tree) |
| `i` | Add to `.gitignore` / exclude |
| `e` / `o` | Edit / open the file |
| `s` / `S` | Stash all changes / stash options |
| `f` | Fetch |
| `g` | Upstream reset options |
| `M` | Open the external merge tool |
| `` ` `` | Toggle flat list vs file tree |
| `-` / `=` | Collapse / expand all directories |

> In the commit message box, `Enter` commits and `Alt-Enter` inserts a newline
> (so you can write a body without leaving the TUI).

## Staging hunks and lines (`Enter` on a file)

This is lazygit's headline feature: `git add -p` without the y/n/s/e prompts.

| Key | Action |
| --- | --- |
| `Space` | Stage / unstage the selected line (or selection) |
| `a` | Toggle selecting the whole hunk |
| `v` | Range select — move up/down to grow the selection |
| `←` / `→` | Previous / next hunk |
| `d` | Discard the selected line(s) |
| `E` | Edit the hunk by hand |
| `Tab` | Switch between the unstaged and staged views |
| `c` | Commit what's staged |
| `Esc` | Back to the files panel |

## Branches panel (`3`)

| Key | Action |
| --- | --- |
| `Space` | Check out the selected branch |
| `n` | New branch from the selected branch |
| `c` | Check out by name (accepts a hash or tag) |
| `F` | Force checkout (discards local changes) |
| `d` | Delete branch (local / remote) |
| `r` | Rebase the checked-out branch onto this one |
| `M` | Merge this branch into the checked-out one |
| `f` | Fast-forward this branch from its upstream |
| `u` | Set / unset the upstream |
| `R` | Rename the branch |
| `T` | Create a tag pointing here |
| `g` | Reset the checked-out branch to this one |
| `o` | Create a pull request (`O` for options) |
| `Enter` | List this branch's commits |

**Remotes tab:** `n` add, `e` edit, `d` remove, `f` fetch.
**Tags tab:** `n` create, `d` delete, `P` push, `Space` check out.
**Worktrees tab:** `n` new, `Space` switch to, `d` remove, `o` open.

## Commits panel (`4`) — rewriting history

Most of these start (or edit) an interactive rebase for you.

| Key | Action |
| --- | --- |
| `s` | Squash the commit down into the one below |
| `f` | Fixup: like squash, but discard this message |
| `r` | Reword the commit message |
| `R` | Reword using your `$EDITOR` |
| `d` | Drop the commit |
| `e` | Edit the commit (stop the rebase here) |
| `i` | Start an interactive rebase from here |
| `Ctrl-j` / `Ctrl-k` | Move the commit down / up |
| `A` | Amend the commit with staged changes |
| `a` | Reset the commit's author (`A` in the amend menu sets it) |
| `F` | Create a fixup commit targeting this one |
| `S` | Apply/squash the fixup commits above |
| `t` | Revert the commit (creates a new commit) |
| `T` | Tag the commit |
| `c` / `C` | Copy commit / copy a range (cherry-pick clipboard) |
| `v` | Paste the copied commits (cherry-pick) |
| `n` | New branch starting at this commit |
| `g` | Reset the current branch to this commit (soft/mixed/hard) |
| `b` | Bisect options |
| `o` | Open the commit in your browser |
| `Enter` | Browse the commit's files |

> Rewriting only works on commits that aren't yet pushed — the same rule as
> plain `git rebase`. See [`git.md`](git.md) for the command-line equivalents.

## Rebase, merge & conflicts

| Key | Action |
| --- | --- |
| `m` | Continue or abort an in-progress rebase/merge |
| `Enter` (on a conflicted file) | Open the conflict resolution view |
| `Space` | Pick the highlighted hunk |
| `b` | Pick **b**oth hunks |
| `←` / `→` | Previous / next conflict |
| `z` | Undo the last conflict decision |

Resolving all conflicts and staging the files lets you press `m` → *continue*.

## Stash panel (`5`)

| Key | Action |
| --- | --- |
| `Space` | Apply the stash |
| `g` | Pop the stash (apply and drop) |
| `d` | Drop the stash |
| `r` | Rename the stash |
| `n` | New branch from the stash |
| `Enter` | Browse the stashed files |

From the files panel: `s` stashes everything, `S` opens options (keep staged
changes, stash unstaged only, include untracked…).

## Custom patches (`Ctrl-p`)

Build a patch out of arbitrary lines from *old* commits, then move it around —
the killer feature for splitting a commit after the fact.

1. Go to a commit in the Commits panel and press `Enter` to list its files.
2. Press `Space` on a file to add all of it, or `Enter` then `Space` to pick
   individual lines.
3. Press `Ctrl-p` for the patch menu: remove the patch from its commit, move it
   into a new commit, move it into the index, apply it to the working tree, or
   reset the patch.

## Undo (`z`)

Lazygit reads the reflog, so `z` walks back through your recent actions
(commits, rebases, checkouts, resets) and `Ctrl-z` redoes them. It cannot undo
work that was never committed — discarded changes are gone for good.

## Config

Config lives at `~/.config/lazygit/config.yml` on Linux
(`~/Library/Application Support/lazygit/config.yml` on macOS); find it with
`lazygit -cd`, and dump the full default set with `lazygit -c`.

```yaml
gui:
  showFileTree: true          # tree view in the files panel
  nerdFontsVersion: "3"       # icons, if your font has them
  sidePanelWidth: 0.3
  scrollHeight: 2
git:
  autoFetch: true
  paging:
    colorArg: always
    pager: delta --dark --paging=never   # or diff-so-fancy
  commit:
    signOff: false
os:
  editPreset: nvim            # how lazygit opens files
keybinding:
  universal:
    quit: q
```

### Custom commands

Bind your own commands to keys, with prompts:

```yaml
customCommands:
  - key: 'E'
    context: 'files'
    description: 'Empty commit'
    command: 'git commit --allow-empty -m "{{index .PromptResponses 0}}"'
    prompts:
      - type: 'input'
        title: 'Commit message'
```

Useful context values: `files`, `localBranches`, `commits`, `stash`, `global`.

---

> Bindings are context-sensitive, so `?` inside lazygit always beats a printed
> list — this sheet covers version 0.63.1. Full docs and the complete keybinding
> reference: <https://github.com/jesseduffield/lazygit>.
