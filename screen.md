# GNU `screen` cheatsheet

A terminal multiplexer: keep shells running after you disconnect, split the
screen into regions, and reattach from anywhere (great over SSH).

> **The cheatsheet inside screen:** press **`Ctrl-a ?`** to open the key-bindings
> help page — it lists every command and its key, so it's a complete reference on
> its own. Press `Space` or `Enter` to leave it.

## The prefix key

Almost every command starts with the **prefix** `Ctrl-a` (written `C-a` below),
followed by another key. Change it in `.screenrc` with `escape ^Aa`.

## Sessions (from the shell)

| Command | What it does |
| --- | --- |
| `screen` | Start a new session |
| `screen -S <name>` | Start a named session |
| `screen -ls` | List running sessions |
| `screen -r` | Reattach to a detached session |
| `screen -r <name>` | Reattach to a specific session |
| `screen -d -r` | Detach it elsewhere, then reattach here |
| `screen -x` | Attach to an already-attached session (shared view) |
| `screen -S <name> -X quit` | Kill a session by name |

## Windows (inside screen, after `C-a`)

| Key | Action |
| --- | --- |
| `C-a c` | Create a new window |
| `C-a n` / `C-a p` | Next / previous window |
| `C-a <0-9>` | Jump to window by number |
| `C-a C-a` | Toggle between current and last window |
| `C-a "` | Show a selectable list of windows |
| `C-a w` | Show the window bar (list) briefly |
| `C-a A` | Rename the current window |
| `C-a k` | Kill the current window |

## Split regions / panes

| Key | Action |
| --- | --- |
| `C-a S` | Split horizontally (top/bottom) |
| `C-a \|` | Split vertically (left/right) |
| `C-a Tab` | Move focus to the next region |
| `C-a X` | Close the current region |
| `C-a Q` | Close all regions except the current one |

> A new region is empty until you create or select a window in it
> (`C-a c` or `C-a "`).

## Detach & reattach

| Key / Command | Action |
| --- | --- |
| `C-a d` | Detach (session keeps running in the background) |
| `screen -r` | Reattach later (see Sessions above) |

## Copy / scrollback mode

| Key | Action |
| --- | --- |
| `C-a [` | Enter copy/scrollback mode (scroll with arrows / `C-u` / `C-d`) |
| `Space` | Set start of selection, then move and `Space` again to copy |
| `Enter` | Finish copying |
| `C-a ]` | Paste the copied text |

## Other useful bits

| Key / Command | Action |
| --- | --- |
| `C-a :` | Enter command mode (type any screen command) |
| `C-a H` | Toggle logging the window to a file |
| `C-a x` | Lock the screen (password protect) |
| `C-a M` | Monitor window for activity |
| `C-a _` | Monitor window for silence |
| `C-a *` | Show all displays attached to the session |

## `.screenrc` essentials

```sh
escape ^Aa                 # prefix key (default C-a)
defscrollback 10000        # scrollback buffer size
startup_message off        # skip the copyright splash
autodetach on              # detach (not kill) if the connection drops
# Pre-create named windows on startup:
screen -t shell 0
screen -t monitor 1 htop
```

---

> When in doubt, `Ctrl-a ?` shows every binding, and `man screen` has the full
> documentation.
