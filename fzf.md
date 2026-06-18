# `fzf` cheatsheet

A general-purpose command-line fuzzy finder. Reads a list on stdin, lets you
filter it interactively, and prints the selection on stdout.

> **Setup with oh-my-zsh:** enable the bundled plugin so the `Ctrl-T` / `Ctrl-R`
> / `Alt-C` key bindings and completion are loaded automatically (it knows the
> Debian paths, so this works even with older fzf like 0.38):
>
> ```sh
> # in ~/.zshrc
> plugins=(... fzf)
> ```
>
> Then re-login or `source ~/.zshrc`, and confirm with `bindkey '^T'`
> (should show `fzf-file-widget`). See [Shell integration](#shell-integration)
> for non-oh-my-zsh setups.

## Basics

> **Important:** running `fzf` on its own only *prints* your selection to the
> terminal — it is **not** saved to any variable, so once it scrolls past it's
> gone. To actually use the chosen file you must capture it **as you run fzf**:
>
> ```sh
> f=$(fzf)          # capture into a variable, reuse as $f
> vim "$f"
>
> vim "$(fzf)"      # or feed it straight into a command
> ```
>
> Smoothest of all: don't run bare `fzf` — type the command first and press
> `Ctrl-T`, which inserts the selection onto your command line instead of just
> printing it (see [Shell integration](#shell-integration)).

| Command | What it does |
| --- | --- |
| `fzf` | Fuzzy-find over files; **prints** the pick (capture with `$(...)`) |
| `vim $(fzf)` | Pick a file and open it |
| `find . -type f \| fzf` | Pipe any list into fzf |
| `fzf -m` | Multi-select with `Tab`; prints all chosen lines |
| `fzf -q 'foo'` | Start with an initial query |
| `fzf -f 'foo'` | Non-interactive: filter + print matches (no UI) |

## Search syntax (extended mode, on by default)

| Pattern | Matches |
| --- | --- |
| `foo` | Fuzzy match `foo` |
| `'foo` | Exact substring `foo` (quoted) |
| `^foo` | Prefix: lines starting with `foo` |
| `foo$` | Suffix: lines ending with `foo` |
| `!foo` | Inverse: lines **not** containing `foo` |
| `foo bar` | AND: contains both terms |
| `foo \| bar` | OR: contains either term |

## Keys inside the finder

| Key | Action |
| --- | --- |
| `↑` / `↓`, `Ctrl-P` / `Ctrl-N` | Move selection up / down |
| `Enter` | Accept current selection |
| `Tab` / `Shift-Tab` | Mark / unmark (with `-m`) and move |
| `Esc` / `Ctrl-C` / `Ctrl-G` | Abort |
| `Shift-↑` / `Shift-↓` | Scroll the preview window |

## Useful options

| Option | What it does |
| --- | --- |
| `-m`, `--multi` | Allow selecting multiple entries |
| `-e`, `--exact` | Exact match by default (instead of fuzzy) |
| `-i` / `+i` | Case-insensitive / case-sensitive |
| `--height=40%` | Use a window instead of full screen |
| `--layout=reverse` | List on top, prompt at the top |
| `--border` | Draw a border around the finder |
| `--prompt='> '` | Change the input prompt |
| `--header='...'` | Show a fixed header line |
| `--header-lines=N` | Treat first N input lines as a sticky header |
| `--ansi` | Honour ANSI colour codes in the input |
| `--no-sort` | Keep input order (don't sort by score) |
| `--tac` | Reverse the order of the input |
| `--cycle` | Wrap around at the top/bottom of the list |
| `-d`, `--delimiter` | Field delimiter (used with `--nth` / `--with-nth`) |
| `--nth=N` | Limit search to the Nth field |
| `--with-nth=N` | Display only the Nth field(s) |
| `-1`, `--select-1` | Auto-select if there is exactly one match |
| `-0`, `--exit-0` | Exit immediately if there are no matches |
| `--print0` / `--read0` | Use NUL as output / input separator |

## Preview window

```sh
# Show file contents (with bat) as you move through results
fzf --preview 'bat --color=always {}'

# Position and size the preview
fzf --preview 'cat {}' --preview-window=right:60%
```

Placeholders available in `--preview` and `--bind`:

| Token | Expands to |
| --- | --- |
| `{}` | The current line |
| `{+}` | All selected lines (multi-select) |
| `{q}` | The current query string |
| `{1}`, `{2}`, … | Field N (requires `--delimiter`) |

## Custom key bindings

```sh
# Toggle preview with Ctrl-/, reload the list with Ctrl-R
fzf --bind 'ctrl-/:toggle-preview' \
    --bind 'ctrl-r:reload(find . -type f)'
```

## Shell integration

Enable the keybindings and completion by adding the right line to your
`~/.bashrc` / `~/.zshrc` (then re-login or `source` it):

```sh
# fzf 0.48+ (single helper)
eval "$(fzf --zsh)"      # or --bash / --fish

# older fzf (e.g. Debian's 0.38) — source the example scripts instead
source /usr/share/doc/fzf/examples/key-bindings.zsh
source /usr/share/doc/fzf/examples/completion.zsh
# find the exact paths with: dpkg -L fzf | grep -E 'key-bindings|completion'
# or, with oh-my-zsh, just add the bundled plugin: plugins=(... fzf)
```

| Keybinding | Action |
| --- | --- |
| `Ctrl-T` | Paste selected files/dirs onto the command line |
| `Ctrl-R` | Fuzzy-search command history |
| `Alt-C` | `cd` into a selected subdirectory |
| `**` + `Tab` | Trigger fuzzy completion (e.g. `vim **<Tab>`) |

## Typing less: aliases & functions

Running `fzf` bare just prints your pick — it's meant to feed another command.
For ad-hoc use, lean on `Ctrl-T` and `**<Tab>` above. For repeated workflows,
wrap `$(fzf)` so you don't retype it.

Aliases *can* contain `$(fzf)` (alias expansion happens before the command runs,
so the substitution still executes at runtime):

```sh
alias fe='${EDITOR:-vim} "$(fzf)"'   # fuzzy-edit a file
```

Functions are more flexible (cancel-safe with `&&`, can take extra args):

```sh
# fuzzy-edit a file
fe() { local f; f=$(fzf) && ${EDITOR:-vim} "$f"; }

# fuzzy cd into a directory
fcd() { local d; d=$(find . -type d 2>/dev/null | fzf) && cd "$d"; }

# fuzzy-pick a file, then run any command on it: fo cat
fo() { local f; f=$(fzf) && "$@" "$f"; }
```

The `&&` matters: if you press `Esc` to cancel fzf, `$f` is empty and the
command is skipped instead of running on nothing.

Reuse the last selection with `$_` (the last arg of the previous command) to
avoid a second fzf:

```sh
cat "$(fzf)"
vim "$_"        # reopen the same file
```

## Recipes

```sh
# Pick a process and kill it (multi-select with Tab)
kill -9 $(ps -ef | sed 1d | fzf -m | awk '{print $2}')

# Fuzzy-switch git branch
git switch "$(git branch --format='%(refname:short)' | fzf)"

# Search file contents with ripgrep, preview the line, open in vim
vim "$(rg --line-number . | fzf -d: --nth=3 | cut -d: -f1)"

# Pick from command history and run it
print -z "$(fc -rl 1 | fzf | sed 's/^[ 0-9*]*//')"   # zsh: edit before running
```

## Troubleshooting key bindings

`Ctrl-T` / `Ctrl-R` / `Alt-C` are bound in the **shell where the integration is
loaded** — for SSH + screen/tmux that's the *remote* shell inside the session.

```sh
# 1. Is the binding loaded?  Want "fzf-file-widget", not "self-insert".
bindkey '^T'                 # zsh
bind -q fzf-file-widget      # bash
# If not loaded, add the source line (see Shell integration) and re-login.

# 2. Loaded but Ctrl-T does nothing? Something is eating the key.
stty -a | grep -o 'status = [^;]*'   # BSD/macOS maps ^T to SIGINFO
stty status undef                    # free it up

# 3. GNU Screen: default command key is Ctrl-A, so ^T passes through.
grep -i '^escape' ~/.screenrc        # only a problem if set to ^Tt
```

Quick discriminator: if `Ctrl-R` works but `Ctrl-T` doesn't, the integration is
loaded and something is intercepting `Ctrl-T` (stty/screen). If neither works,
the integration isn't loaded.

## Environment variables

| Variable | Effect |
| --- | --- |
| `FZF_DEFAULT_COMMAND` | Command used to generate the default list |
| `FZF_DEFAULT_OPTS` | Options applied to every fzf invocation |
| `FZF_CTRL_T_COMMAND` / `FZF_CTRL_T_OPTS` | Source / options for `Ctrl-T` |
| `FZF_CTRL_R_OPTS` | Options for `Ctrl-R` history search |
| `FZF_ALT_C_COMMAND` / `FZF_ALT_C_OPTS` | Source / options for `Alt-C` |

```sh
# Example: use ripgrep for the default source and a reverse layout everywhere
export FZF_DEFAULT_COMMAND='rg --files --hidden'
export FZF_DEFAULT_OPTS='--height=40% --layout=reverse --border'
```

---

> Options here are fzf's stable, long-standing flags. Confirm anything against
> your installed version with `fzf --help` or `man fzf`. The `fzf --bash/--zsh/--fish`
> integration helper requires fzf **0.48.0+**; on older versions source the
> `~/.fzf.{bash,zsh}` scripts instead.
