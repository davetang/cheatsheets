# `atuin` cheatsheet

Shell history kept in a SQLite database instead of a text file, searched through
a TUI bound to `Ctrl-R` and the up arrow, and optionally synced between machines
with end-to-end encryption.

## How it differs from the other history tools

Everything else in this repo that touches history — `Ctrl-R`, `fzf`, `hstr`,
`mcfly` — is a **better reader of `~/.bash_history`**. Atuin replaces the file.

A line in `~/.zsh_history` is a string, appended by the shell (often only when
the shell exits, capped by `HISTSIZE`, one file racing between every open
terminal). Atuin instead records an **event**, written by shell hooks at the
moment the command runs, through a pair of calls:

```
preexec  -> atuin history start   # command text, cwd, host, session, start time
precmd   -> atuin history end     # exit code, duration
```

So each entry carries context the flat file never had, and that context is what
you search:

| | Shell history / `fzf` / `hstr` / `mcfly` | Atuin |
| --- | --- | --- |
| Storage | Plain text, one line per command | SQLite database of records |
| Written | By the shell, usually at exit | By hooks, as each command starts and ends |
| Stored per entry | The command (plus a timestamp, if enabled) | Command, cwd, exit code, duration, host, user, session, start time |
| Scope | One file per shell, per machine | One history across zsh/bash/fish/nu, across machines |
| Truncation | `HISTSIZE` / `HISTFILESIZE` | None; you prune deliberately |
| Concurrent shells | Interleave and clobber each other | Independent records, no races |
| Querying | Substring or fuzzy match on the line | `WHERE` on any field — directory, exit code, host, session, time range |
| Sync | rsync your dotfiles and hope | Optional end-to-end encrypted sync, self-hostable |
| Scriptable | `grep` over a file | `atuin search --format` as a data source |

The practical consequences:

- **Search is scoped, not just fuzzy.** "What did I run *in this directory*",
  "*in this git repo*", "*in this shell session*", "*that failed*", "*last
  Tuesday*" are all first-class filters, not something you can express against a
  text file. That is the actual reason to switch — the matching algorithm is the
  small part.
- **Nothing is lost.** No `HISTSIZE` cap, no history dropped because a terminal
  was killed, no two shells overwriting each other's file.
- **History outlives the shell and the machine.** The same database serves bash
  at work and fish at home, and sync means a command you ran on the server last
  month is one `Ctrl-R` away on your laptop.
- **History becomes data.** `atuin search --format` emits fields you can pipe
  into `awk`, `sort`, `jq`, `parallel`.

The closest relative is `zsh-histdb` (also SQLite, also cwd-aware), but it is
zsh-only and has no sync. `mcfly` is the other genuinely different one: it also
uses a database and ranks results with a small neural network, but it seeds
itself from your shell history file and does not sync.

Two things it is *not*: it isn't a fuzzy finder (`fzf` still does everything
else — files, branches, processes), and it doesn't turn off your shell's own
`HISTFILE`. Both keep recording unless you disable the shell's history yourself.

## Setup

```sh
# install (any one of these)
curl --proto '=https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh
cargo install atuin
brew install atuin            # also: apt, pacman, dnf, nix, ...

# wire it into your shell, then restart it
echo 'eval "$(atuin init zsh)"'  >> ~/.zshrc
echo 'atuin init fish | source'  >> ~/.config/fish/config.fish

# bash needs ble.sh or bash-preexec first
curl https://raw.githubusercontent.com/rcaloras/bash-preexec/master/bash-preexec.sh -o ~/.bash-preexec.sh
echo '[[ -f ~/.bash-preexec.sh ]] && source ~/.bash-preexec.sh' >> ~/.bashrc
echo 'eval "$(atuin init bash)"' >> ~/.bashrc

# bring the old history across
atuin import auto              # picks the importer from $SHELL
```

`atuin init` flags, for when you don't want the default bindings:

| Flag | Effect |
| --- | --- |
| `--disable-up-arrow` | Leave the up arrow as your shell's own history |
| `--disable-ctrl-r` | Leave `Ctrl-R` alone (bind `atuin-search` yourself) |
| `--disable-ai` | Don't bind `?` to the AI assistant |
| `ATUIN_NOBIND=true` | Env var: register the widgets, bind nothing |

```sh
# zsh, binding it by hand
export ATUIN_NOBIND="true"
eval "$(atuin init zsh)"
bindkey '^r' atuin-search
```

Importers: `auto`, `bash`, `zsh`, `zsh-hist-db`, `fish`, `nu`, `nu-hist-db`,
`powershell`, `replxx`, `resh`, `xonsh`, `xonsh-sqlite`. Point them elsewhere
with `HISTFILE=/path/to/file atuin import zsh`.

## The search UI

| Key | Action |
| --- | --- |
| `Ctrl-R` / `↑` | Open the search |
| `Enter` | Run the selected command (insert it if `enter_accept = false`) |
| `Tab` | Insert into the prompt for editing, don't run |
| `Ctrl-R` *(inside)* | Cycle **filter** mode — global → host → session → directory |
| `Ctrl-S` | Cycle **search** mode — fuzzy → prefix → fulltext → skim |
| `Ctrl-N` / `Ctrl-J` / `↓` | Next result |
| `Ctrl-P` / `Ctrl-K` / `↑` | Previous result |
| `Alt-1` … `Alt-9` | Pick the *n*th result straight away |
| `Ctrl-O` | Inspector — full metadata for the selected entry |
| `Ctrl-Y` | Copy the selected command to the clipboard |
| `PgUp` / `PgDn` | Scroll a page at a time |
| `Ctrl-C` / `Ctrl-D` / `Esc` | Leave, restoring what you had typed |

On macOS, where `Alt` is awkward, set `ctrl_n_shortcuts = true` to move the
numeric picks onto `Ctrl-0`…`Ctrl-9`.

## Filter modes — the part that matters

| Mode | Searches |
| --- | --- |
| `global` | Everything, every machine |
| `host` | This machine only |
| `session` | This shell session only |
| `directory` | Commands run in the current directory |
| `workspace` | Commands run anywhere in the current git repo (`workspaces = true`) |

Set the starting mode with `filter_mode`, and give the up arrow its own with
`filter_mode_shell_up_key_binding = "directory"` — up arrow becomes "what do I
run *here*", `Ctrl-R` stays global.

## Search modes

| Mode | Matches |
| --- | --- |
| `prefix` | `query*` |
| `fulltext` | `*query*` |
| `fuzzy` | fzf-style fuzzy matching (default) |
| `skim` | skim's syntax |
| `daemon-fuzzy` | In-memory index in the daemon; fast, needs the daemon running |

Fuzzy mode takes the fzf operators:

| Token | Meaning |
| --- | --- |
| `sbtrkt` | Fuzzy match |
| `'wild` | Exact match |
| `^music` | Starts with |
| `.mp3$` | Ends with |
| `!fire` | Does not contain |
| `!^music` / `!.mp3$` | Does not start / end with |
| `a \| b` | Either (not supported by `daemon-fuzzy`) |

## Searching from the command line

The same query engine, without the TUI — this is where history stops being a
scrollback and starts being a dataset.

| Flag | Effect |
| --- | --- |
| `-i`, `--interactive` | Open the TUI instead of printing |
| `-c`, `--cwd <dir>` | Only commands run in this directory (`.` for here) |
| `--exclude-cwd <dir>` | Everything except that directory |
| `-e`, `--exit <code>` | Only this exit code (`--exit 0` = it worked) |
| `--exclude-exit <code>` | Everything but that exit code |
| `--before` / `--after` | Time bounds: `"yesterday 3pm"`, `"2026-07-01"`, `"last friday"` |
| `--limit N` / `--offset N` | Page through results |
| `-r`, `--reverse` | Oldest first |
| `--human` | Readable timestamps and durations |
| `-f`, `--format` | Choose the fields (see below) |
| `--delete` | Delete everything matching — no undo |
| `--delete-it-all` | Delete the entire history |

Format variables: `{command}`, `{directory}`, `{duration}`, `{time}`, `{exit}`,
`{host}`, `{user}`, `{relativetime}`. Wildcards `*` and `%` work in the query,
and a bare query is a prefix search.

```sh
atuin search --cwd . docker                          # docker commands run here
atuin search --exit 0 --after "yesterday 3pm" make   # makes that worked, since then
atuin search --exclude-exit 0 --limit 20             # the last 20 failures
atuin search -f "{time} {duration} {command}" --human git push
atuin search '*.fastq*'                              # wildcards
```

## `atuin history`

| Command | What it does |
| --- | --- |
| `atuin history list` | Everything, oldest first |
| `atuin history list --cwd` | Just this directory |
| `atuin history list --session` | Just this shell session |
| `atuin history list --cmd-only` | Commands with no metadata — pipe-friendly |
| `atuin history list --print0` | NUL-terminated, safe for multiline commands |
| `atuin history last` | The command you just ran |
| `atuin history prune -n` | Dry-run: what `history_filter`/`cwd_filter` would delete |
| `atuin history prune` | Actually delete those entries |
| `atuin history dedup -n` | Dry-run: duplicate command+cwd+host entries |
| `atuin history dedup --dupkeep 1` | Deduplicate, keeping the most recent copy |
| `atuin history tail` | Stream events from the daemon as they happen |

`atuin history start` / `atuin history end` are what the shell hooks call; you
shouldn't need them by hand.

## Stats

```sh
atuin stats                  # all time
atuin stats last week
atuin stats yesterday
atuin stats 2026-07-01
atuin wrapped 2026           # a year in review
```

Tune what counts as a distinct command in `config.toml`:

```toml
[stats]
common_subcommands = ["cargo", "git", "docker", "npm", "kubectl"]  # count "git push", not "git"
common_prefix = ["sudo"]                                          # ignore the prefix
ignored_commands = ["cd", "ls", "vi"]
```

## Sync

Optional, end-to-end encrypted, and self-hostable — the server never sees your
commands.

| Command | What it does |
| --- | --- |
| `atuin register -u <user> -e <email>` | Create an account (generates your key) |
| `atuin key` | Print the encryption key — **save this in a password manager** |
| `atuin login -u <user>` | Log in on another machine (asks for password + key) |
| `atuin sync` | Sync now |
| `atuin sync -f` | Full sync — use when something looks missing |
| `atuin status` | Account, sync time, record counts |
| `atuin logout` | Drop the local session |
| `atuin account change-password` | Change the sync password |
| `atuin account delete` | Delete the account and all synced data |

```toml
auto_sync = true
sync_frequency = "1h"                    # "0" syncs after every command
sync_address = "https://api.atuin.sh"    # or your own server
```

Lose the key and the history is unrecoverable — that is the point of the
encryption, and nobody can reset it for you.

## Store maintenance

The record store underneath sync, for when sync misbehaves:

| Command | What it does |
| --- | --- |
| `atuin store status` | Records held locally, per tag and per host |
| `atuin store verify` | Check every local record decrypts with the current key |
| `atuin store purge` | Drop local records that fail verification |
| `atuin store rekey [key]` | Re-encrypt the local store with a new key |
| `atuin store rebuild history` | Rebuild the SQLite history from the records |
| `atuin store push` / `pull` | One-way sync, `--tag` to narrow, `--force` to replace |
| `atuin doctor` | Check the install for the usual problems |
| `atuin info` | Paths and environment variables in use |

## Dotfiles, key-values, scripts

Aliases and environment variables that follow you between machines:

```toml
[dotfiles]
enabled = true
```

```sh
atuin dotfiles alias set k 'kubectl'
atuin dotfiles alias list
atuin dotfiles alias delete k
atuin dotfiles var set EDITOR 'nvim'      # -n for shell-local, not exported
atuin dotfiles var list
```

Restart the shell after changing either.

```sh
# small synced key-value store
atuin kv set -k api-host 'example.com'
atuin kv get api-host
atuin kv list --all                        # -n NAMESPACE to scope
atuin kv delete api-host

# saved, templated scripts
atuin scripts new deploy --tags ops        # --last N builds one from recent history
atuin scripts run deploy -v env=staging
atuin scripts list
```

## Config worth setting

`~/.config/atuin/config.toml` — `atuin default-config` prints the annotated
default.

| Option | Does |
| --- | --- |
| `search_mode = "fuzzy"` | Matching algorithm |
| `filter_mode = "global"` | Scope the TUI opens in |
| `filter_mode_shell_up_key_binding = "directory"` | Different scope for the up arrow |
| `workspaces = true` | Enable the git-repo filter mode |
| `style = "compact"` | `auto`, `full` or `compact` |
| `inline_height = 20` | Lines used by the TUI (`0` = fullscreen) |
| `invert = true` | Search bar at the top |
| `show_preview = true` | Show the full command under the list |
| `enter_accept = false` | Enter inserts instead of running — safer |
| `keymap_mode = "vim-insert"` | vim keys in the TUI |
| `exit_mode = "return-original"` | What `Esc` leaves on the prompt |
| `store_failed = true` | Keep commands that exited non-zero |
| `secrets_filter = true` | Refuse to store things that look like credentials |
| `history_filter = ["^secret-cmd"]` | Regexes never recorded |
| `cwd_filter = ["^/very/secret/dir"]` | Directories never recorded |
| `update_check = false` | Stop the version check on startup |
| `db_path` / `key_path` | `~/.local/share/atuin/history.db`, `.../key` |

```toml
[tmux]
enabled = true       # search in a tmux popup (tmux 3.2+)
width = "80%"
height = "60%"

[daemon]
enabled = true       # record via a background daemon: faster prompts, no lost entries
autostart = true
```

## Recipes

```sh
# What do I actually run in this project?
atuin history list --cwd --cmd-only | sort | uniq -c | sort -rn | head

# Everything that failed today
atuin search --exclude-exit 0 --after "today" -f "{time} {exit} {command}" --human

# The slowest commands you have ever run
atuin search -f "{duration} {command}" --human --limit 5000 | sort -rn | head

# Rebuild a session as a script
atuin history list --session --cmd-only > session.sh

# Feed history into fzf for a one-off pick
atuin search --limit 5000 --cmd-only | fzf

# Plain-text backup, independent of the database
atuin history list -f "{time} {command}" --human > history-$(date +%F).txt

# Purge a command you regret, everywhere
atuin search --delete 'aws configure set aws_secret_access_key*'

# Which machine ran that?
atuin search -f "{host} {time} {command}" --human 'terraform apply'

# Straight SQL, when the CLI won't express it
sqlite3 ~/.local/share/atuin/history.db \
  "select cwd, count(*) c from history group by cwd order by c desc limit 10;"
```

## Gotchas

- **Your shell keeps its own history too.** Atuin doesn't disable `HISTFILE`, so
  commands land in both. Fine, and a useful fallback — but don't expect
  `~/.zsh_history` to stay in sync, and remember it is a second copy to clean if
  you delete something sensitive.
- **`--delete` is immediate and permanent**, and it deletes every match, not the
  one you had in mind. Run the same query without `--delete` first.
- **`secrets_filter` is a heuristic**, not a guarantee. Anything genuinely
  secret belongs in `history_filter`/`cwd_filter`, or shouldn't be typed on a
  command line at all.
- **Bash needs `bash-preexec` or `ble.sh`.** Without one, nothing gets recorded.
  Source it *before* `atuin init bash`.
- **Sync is only as safe as your key backup.** `atuin key`, into a password
  manager, on the day you register.
- **The database grows** — it is designed to, since nothing is truncated. Millions
  of rows are fine; use `atuin history dedup` and `prune` rather than truncating
  by hand.
- **`Ctrl-R` means two things**: opening the search, then cycling filter modes
  once you are inside it.
- **Imported history is thin.** Old entries have no cwd, exit code or duration,
  so directory and exit-code filters won't find them.
- **A hook that fails can slow every prompt.** If your shell feels sluggish, try
  `atuin doctor` and the `[daemon]` setting.

---

> Pinned to **atuin 18.18.1** (July 2026). `atuin --help` and any
> `atuin <command> --help` are accurate for your build; the docs live at
> <https://docs.atuin.sh>. Recent releases also ship `atuin ai` (LLM command
> generation, bound to `?`) and `atuin mcp` (history search as an MCP server) —
> both opt-in.
