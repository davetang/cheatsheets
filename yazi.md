# `yazi` cheatsheet

A blazing-fast terminal file manager with vim-like keys, async I/O and image
previews. Three panes: parent directory, current directory, preview.

> **Set up the shell wrapper first.** Quitting yazi with `q` leaves you in the
> directory you *started* in, not the one you navigated to. The official `y`
> wrapper fixes that by having yazi write its cwd to a temp file on exit:
>
> ```sh
> # in ~/.bashrc or ~/.zshrc
> function y() {
> 	local tmp="$(mktemp -t "yazi-cwd.XXXXXX")" cwd
> 	yazi "$@" --cwd-file="$tmp"
> 	IFS= read -r -d '' cwd < "$tmp"
> 	[ -n "$cwd" ] && [ "$cwd" != "$PWD" ] && builtin cd -- "$cwd"
> 	rm -f -- "$tmp"
> }
> ```
>
> Then run `y` instead of `yazi`. Use `Q` to quit *without* changing directory.

> **The cheatsheet inside yazi:** press **`~`** (or `F1`) for the help page — it
> lists every binding for the current mode and can be filtered with `/`. `Esc`
> closes it.

## Navigation

| Key | Action |
| --- | --- |
| `j` / `k` | Down / up |
| `l` / `h` | Enter directory (or open file) / go to parent |
| `gg` / `G` | Top / bottom |
| `Ctrl-u` / `Ctrl-d` | Half page up / down |
| `Ctrl-b` / `Ctrl-f` | Full page up / down |
| `J` / `K` | Scroll the **preview** pane down / up |
| `gh` | Go home (`~`) |
| `gc` | Go to `~/.config` |
| `gd` | Go to `~/Downloads` |
| `g<Space>` | Type a path to jump to |
| `z` | Jump with zoxide (frecent directories) |
| `Z` | Jump with fzf (fuzzy find) |
| `.` | Toggle hidden files |

## Selection

| Key | Action |
| --- | --- |
| `Space` | Toggle selection and move down |
| `v` | Visual mode — select while moving |
| `V` | Visual mode — unselect while moving |
| `Ctrl-a` | Select all |
| `Ctrl-r` | Invert the selection |
| `Esc` | Clear selection / cancel the current mode |

## File operations

| Key | Action |
| --- | --- |
| `o` / `Enter` | Open with the default opener |
| `O` / `Shift-Enter` | Open with — pick the opener interactively |
| `y` | Yank (copy) |
| `x` | Yank (cut) |
| `p` | Paste |
| `P` | Paste, overwriting existing files |
| `Y` / `X` | Cancel the pending yank |
| `-` | Create a symlink to the yanked file (absolute) |
| `_` | Create a symlink to the yanked file (relative) |
| `d` | Move to trash |
| `D` | Delete permanently |
| `a` | Create a file (end the name with `/` for a directory) |
| `r` | Rename |
| `;` | Run a shell command |
| `:` | Run a shell command and block until it finishes |
| `w` | Task manager (progress of copies, deletes, previews) |

> Deleting and overwriting pop up a confirmation: `Enter` confirms, `Esc` backs
> out. Bulk rename works by selecting files and pressing `r`, which opens your
> `$EDITOR` with one filename per line.

## Copy paths (`c` submenu)

| Key | Copies to the clipboard |
| --- | --- |
| `cc` | The full path |
| `cd` | The parent directory path |
| `cf` | The filename |
| `cn` | The filename without its extension |

## Find, filter, search

Four similar-looking things — this is the section worth remembering:

| Key | What it does | Scope |
| --- | --- | --- |
| `/` | **Find** — jump to the next matching name as you type | Current directory |
| `?` | Find backwards | Current directory |
| `n` / `N` | Repeat the find forwards / backwards | Current directory |
| `f` | **Filter** — hide everything that doesn't match | Current directory |
| `s` | **Search** by filename with `fd` | Recursive |
| `S` | **Search** by *file contents* with `ripgrep` | Recursive |
| `Ctrl-s` | Cancel a running search and keep what was found | — |

Search results appear as a virtual directory; `Esc` leaves it and returns to the
real one.

## Sorting (`,` submenu)

| Key | Sort by |
| --- | --- |
| `,a` | Alphabetical |
| `,n` | Natural (so `file10` follows `file9`) |
| `,e` | Extension |
| `,s` | Size |
| `,m` | Modified time |
| `,b` | Created (birth) time |
| `,r` | Random |

Use the capital letter (e.g. `,M`) to reverse the order.

## Tabs

| Key | Action |
| --- | --- |
| `t` | New tab at the current directory |
| `1` … `9` | Switch to tab N |
| `[` / `]` | Previous / next tab |
| `{` / `}` | Move the current tab left / right |
| `Ctrl-c` | Close the tab (quits if it's the last one) |

## Quitting

| Key | Action |
| --- | --- |
| `q` | Quit (writes the cwd, so the `y` wrapper can `cd`) |
| `Q` | Quit without writing the cwd |
| `Ctrl-z` | Suspend to the background (`fg` to return) |

## Input boxes (rename, create, filter, shell)

Input starts in insert mode and is vim-modal:

| Key | Action |
| --- | --- |
| `Esc` | Leave insert mode (`Esc` again cancels) |
| `i` / `a` | Back to insert, before / after the cursor |
| `Enter` | Confirm |
| `Ctrl-a` / `Ctrl-e` | Start / end of line |
| `Ctrl-w` | Delete the previous word |
| `Ctrl-u` | Delete to the start of the line |

## Command line

| Command | What it does |
| --- | --- |
| `yazi` | Open the current directory |
| `yazi <path>` | Open a specific directory or file |
| `yazi --cwd-file=<file>` | Write the exit directory to a file (see the `y` wrapper) |
| `yazi --chooser-file=<file>` | File-picker mode: write the chosen paths and exit |
| `yazi --clear-cache` | Clear the preview cache |
| `yazi --debug` | Print diagnostics (include this in bug reports) |
| `ya pkg add <owner>/<repo>` | Install a plugin or flavor |
| `ya pkg list` | List installed packages |
| `ya pkg upgrade` | Update everything installed |
| `ya pkg delete <name>` | Remove a package |
| `ya emit <command>` | Send a command to a running yazi instance |

> `--chooser-file` is what makes yazi usable as a file picker for other tools,
> e.g. from a Neovim or shell function that reads the file back afterwards.

## Config

Files live in `~/.config/yazi/` (override with `YAZI_CONFIG_HOME`):

| File | Purpose |
| --- | --- |
| `yazi.toml` | Behaviour: layout, sorting, openers, preview |
| `keymap.toml` | Keybindings |
| `theme.toml` | Colours (or set a flavor) |
| `init.lua` | Lua startup script, plugin setup |
| `plugins/` , `flavors/` | Installed packages |

```toml
# yazi.toml
[mgr]
ratio          = [1, 4, 3]   # parent : current : preview
sort_by        = "natural"
sort_dir_first = true
show_hidden    = false
linemode       = "size"      # none | size | mtime | permissions | owner

[preview]
max_width  = 600
max_height = 900

[opener]
edit = [ { run = '$EDITOR "$@"', block = true } ]
```

Add your own keys without discarding the defaults by *prepending*:

```toml
# keymap.toml
[[mgr.prepend_keymap]]
on   = "!"
run  = 'shell "$SHELL" --block'
desc = "Open a shell here"

[[mgr.prepend_keymap]]
on   = [ "g", "r" ]
run  = 'shell -- ya emit cd "$(git rev-parse --show-toplevel)"'
desc = "Go to the repo root"
```

Use `append_keymap` to add lower-priority bindings, and `keymap` to replace the
whole set.

### Flavors

```sh
ya pkg add yazi-rs/flavors:catppuccin-mocha
```

```toml
# theme.toml
[flavor]
dark  = "catppuccin-mocha"
light = "catppuccin-latte"
```

---

> This sheet is pinned to **yazi 26.5.6**. Bindings are mode-sensitive, so `~`
> inside yazi is always the authoritative list, and `yazi.toml` renamed its
> `[manager]` section to `[mgr]` in an earlier release — older blog posts and
> dotfiles will still show the old name. Full docs: <https://yazi-rs.github.io>.
