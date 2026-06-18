# `bat` cheatsheet

A `cat` clone with syntax highlighting, Git integration, and paging.

## Basics

| Command | What it does |
| --- | --- |
| `bat file.txt` | Print a file with syntax highlighting and line numbers |
| `bat file1 file2` | Concatenate multiple files |
| `bat -` | Read from stdin (e.g. `... \| bat`) |
| `bat -p file` | Plain output: no line numbers, no grid, no header |
| `bat -n file` | Show line numbers only (no grid) |
| `bat -A file` | Show non-printable characters (tabs, spaces, newlines) |

## Display options

| Command | What it does |
| --- | --- |
| `bat -l yaml file` | Force a language for highlighting |
| `bat --theme=ansi file` | Pick a theme (`bat --list-themes` to see all) |
| `bat --style=numbers,grid file` | Choose which elements to show |
| `bat --style=plain file` | Same as `-p` |
| `bat -r 20:40 file` | Print only lines 20–40 |
| `bat -H 25 file` | Highlight line 25 |
| `bat --wrap=never file` | Disable line wrapping |

## Git & diffs

| Command | What it does |
| --- | --- |
| `bat file` | Shows Git modification markers in the gutter (+/~) |
| `git diff \| bat` | Highlight a diff (or `bat -l diff`) |

## Paging (uses `less` by default)

| Key | Action |
| --- | --- |
| `j` / `k` | Down / up one line |
| `Space` / `b` | Forward / back one page |
| `d` / `u` | Forward / back half a page |
| `15j` | Scroll down 15 lines (any count prefix works) |
| `15d` | Scroll 15 lines **and** make 15 the new `d`/`u` default |
| `/pattern` | Search forward; `n` / `N` for next / previous |
| `g` / `G` | Jump to start / end |
| `q` | Quit |

Force paging on/off: `bat --paging=always` or `bat --paging=never`.

## Handy integrations

```sh
# Use bat as the pager for --help output
alias bathelp='bat --plain --language=help'
command --help | bathelp

# Colourful man pages
export MANPAGER="sh -c 'col -bx | bat -l man -p'"

# Preview window for fzf
fzf --preview 'bat --color=always --style=numbers {}'
```

## Config

```sh
bat --config-file        # path to the config file
bat --generate-config-file
```

Set persistent defaults in that file, e.g.:

```
--theme="ansi"
--style="numbers,changes"
--paging=auto
```

> Note: on Debian/Ubuntu the binary may be installed as `batcat` (alias it to `bat`).
