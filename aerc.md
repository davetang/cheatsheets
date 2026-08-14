# `aerc` cheatsheet

A terminal email client with vim-like keys, an embedded terminal, and native
support for reviewing patches sent by email. Three panes: folder sidebar,
message list, and a message view.

> **Your editor is already vim.** aerc's compose editor defaults to `$VISUAL`,
> then `$EDITOR`, then `vi`. With `EDITOR=vim` exported you need to configure
> nothing — aerc runs vim in an embedded terminal when you compose. Only set
> `[compose] editor` in `aerc.conf` if you want something *different* from
> `$EDITOR` (see [Config](#config)). Watch out for a stale `$VISUAL`, though:
> it wins over `$EDITOR`.

> **The cheatsheet inside aerc:** press **`?`** (`:help keys`) for every binding
> in the current context. `:help <topic>` opens the man pages — `:help tutorial`
> is the one to read first, and works from the shell as `man aerc-tutorial`.

## The one thing that trips everyone up

aerc has *contexts*, and each one has its own bindings. In the message list `:`
opens aerc's command line. But once focus is inside the **embedded terminal**
(the vim session while composing, the pager while reading), `:` belongs to that
program, so aerc's command line moves to **`<C-x>`**.

| Where you are | aerc's command line | Escape hatch |
| --- | --- | --- |
| Message list | `:` | — |
| Message viewer | `:` | `/` toggles key passthrough to the pager |
| Compose headers | `:` | `<Tab>` to move on |
| Compose editor (vim) | `<C-x>` | `:wq` in vim ends the edit |
| Terminal tab | `<C-x>` | `:close` closes the tab |

## Replying: the whole lifecycle

This is the part that isn't obvious. Replying is not one keypress — it's a
three-stage pipeline, and you move *forward* through it by quitting the editor.

1. **Pick the reply type** on a selected message (works in the list *and* in the
   viewer):

   | Key | What it does |
   | --- | --- |
   | `rr` | Reply **all**, empty body |
   | `rq` | Reply **all**, body pre-filled with the **q**uoted original |
   | `Rr` | Reply to the **sender only**, empty body |
   | `Rq` | Reply to the sender only, quoted |
   | `f` | Forward (message viewer) |
   | `C` / `m` | Compose a new message |

   Mnemonic: lowercase `r` = reply to everyone, uppercase `R` = reply to one
   person. The second letter picks plain (`r`) or quoted (`q`).

2. **The compose view.** You get To/From/Subject fields on top and vim below.
   Fill the headers, then move into the body and write.

   | Key | Action |
   | --- | --- |
   | `<Tab>` / `<backtab>` | Next / previous header field |
   | `<C-j>` / `<C-k>` | Next / previous field — **also works from inside vim** |
   | `<C-o>` | Tab-complete an address (needs `address-book-cmd`) |
   | `<A-p>` / `<A-n>` | Switch to the previous / next account |
   | `<C-x>` | aerc's command line, while vim has focus |

   Then **quit vim normally with `:wq`**. That doesn't send anything; it hands
   the message to the review screen.

3. **The review screen.** Nothing has been sent yet. Every action is a single
   key and they are listed on screen:

   | Key | Action |
   | --- | --- |
   | `y` | **Send** |
   | `e` | Back to vim to edit the body and headers |
   | `a` | Attach a file (prompts for a path) |
   | `d` | Detach a file |
   | `v` | Preview the message as it will be sent |
   | `p` | Postpone — save to the drafts/postpone folder |
   | `s` / `x` | Toggle PGP signing / encryption |
   | `q` | Quit — asks whether to discard or postpone |
   | `n` | Abort and discard immediately, no confirmation |

> Two vim-specific notes. `<C-j>` and `<C-k>` are intercepted by aerc while
> you're in the editor, so they won't do their usual vim thing. And quitting vim
> with an error — `:cq` — makes aerc discard the message outright, which is a
> fast "never mind" that skips the review screen.

Pick up a postponed draft by opening the Drafts folder and pressing `<Enter>`
(bound to `:recall` there instead of `:view`).

## Message list

| Key | Action |
| --- | --- |
| `j` / `k` | Next / previous message |
| `<C-d>` / `<C-u>` | Half page down / up |
| `<C-f>` / `<C-b>` | Full page down / up |
| `g` / `G` | First / last message (note: `g`, not `gg`) |
| `<Enter>` | Open the message |
| `s` / `S` | Horizontal / vertical split preview alongside the list |
| `zz` / `zt` / `zb` | Scroll the list so the selection is centre / top / bottom |
| `.` | Repeat the last command |
| `q` | Quit (confirms first) |

## Folders

| Key | Action |
| --- | --- |
| `J` / `K` | Next / previous folder in the sidebar |
| `L` / `H` | Expand / collapse the folder tree node |
| `tf` | Toggle the folder node |
| `c` | Change folder — prompts with completion (`:cf`) |
| `:mkdir <name>` | Create a folder and switch to it |
| `:rmdir` | Delete the current folder (`-f` if it isn't empty) |
| `:check-mail` | Poll for new mail now |

## Acting on messages

| Key | Action |
| --- | --- |
| `d` | Delete, with a confirmation prompt |
| `D` | Delete immediately |
| `a` | Archive (flat — everything into one archive folder) |
| `A` | Archive the whole thread |
| `b` | Bounce/resend to another address, unmodified |
| `\|` | Pipe the message into a shell command |
| `:mv <folder>` | Move to a folder |
| `:cp <folder>` | Copy to a folder |
| `:read` / `:unread` | Mark read / unread (`-t` to toggle) |
| `:flag` / `:unflag` | Set / clear a flag (`-x seen\|answered\|flagged\|draft`) |

`:archive` takes a scheme: `flat`, `year`, or `month` — the latter two file
mail into dated subfolders. The default binding uses `flat`.

## Selecting several messages

| Key | Action |
| --- | --- |
| `<Space>` | Toggle the mark and move down |
| `v` | Toggle the mark in place |
| `V` | Visual mark mode — mark while moving |
| `A` | Mark the whole thread (then archives it) |
| `:mark -a` | Mark every message in the folder |
| `:unmark -a` | Clear all marks |
| `:remark` | Re-select the last set of marks, to chain another command |

Any command that acts on "the message" acts on **all marked messages** instead,
when marks exist. So `<Space><Space><Space>` then `D` deletes three.

`:mark` also takes filters: `:mark -u` marks unread, `:mark -s <pattern>`
matches against `From:`, `:mark -r <pattern>` against `To:`/`Cc:`/`Bcc:`.

## Reading a message

| Key | Action |
| --- | --- |
| `q` | Close the viewer |
| `J` / `K` | Next / previous message without leaving the viewer |
| `<C-j>` / `<C-k>` | Next / previous **part** of a multipart message |
| `H` | Toggle full headers |
| `o` / `O` | Open the part with the system opener |
| `S` | Save the part to a path (`:save -a <dir>` saves all attachments) |
| `<C-l>` | Open a link from the message |
| `<C-y>` | Copy a link to the clipboard |
| `/` | Hand keys to the pager, so `/` searches *inside* the message |

The body is rendered by `less` through a filter, so scrolling is `less`'s own
vim-like keys. `/` above turns on **key passthrough** — aerc stops intercepting
and forwards everything to the pager; `<Esc>` gives control back to aerc.

## Search and filter

| Key | Action |
| --- | --- |
| `/` | **Search** — highlights matches, list stays whole |
| `\` | **Filter** — hides everything that doesn't match |
| `n` / `N` | Next / previous search result |
| `<Esc>` | Clear the search or filter |

Both take the same flags:

| Flag | Matches |
| --- | --- |
| *(bare terms)* | The subject line, case-insensitively |
| `-b` / `-a` | The body / the entire message text |
| `-f <from>` | The `From:` header |
| `-t <to>` / `-c <cc>` | The `To:` / `Cc:` header |
| `-H <header>:<value>` | Any header |
| `-r` / `-u` | Read / unread messages |
| `-x <flag>` / `-X <flag>` | With / without a flag (`seen`, `answered`, `forwarded`, `flagged`, `draft`) |
| `-v` | Invert — exclude what matches |
| `-d <since[..until]>` | A date range |

Dates are generous about format: `-d 2026-01-01..2026-02-01`, `-d today`,
`-d yesterday`, `-d "last week"`, `-d Mon..Fri`, `-d Feb..Mar`, or relative
offsets like `-d 1w` and `-d 8d`.

```
:filter -u -d "last week"        # unread, from the past week
:filter -f torvalds -a "rebase"  # from Linus, "rebase" anywhere in the message
:search -x flagged
```

## Threads

| Key | Action |
| --- | --- |
| `T` | Toggle threading on / off |
| `<tab>` / `za` | Fold / unfold the thread under the cursor |
| `zc` / `zo` | Fold / unfold |
| `zM` / `zR` | Fold / unfold **all** threads |

## Tabs, terminal, splits

| Key | Action |
| --- | --- |
| `<C-n>` / `<C-p>` | Next / previous tab |
| `]t` / `[t` | Same, if you prefer the vim-unimpaired style |
| `<C-t>` | New terminal tab running `$SHELL` |
| `$` or `!` | Prompt for a command, run it in a new terminal tab |
| `\|` | Pipe the selected message into a command, show the output |
| `<C-z>` | Suspend aerc (`fg` to return) |
| `<C-c>` / `<C-q>` | Quit, with a confirmation |

`:pin-tab` keeps a tab from being reused; `:cd <dir>` changes the directory new
terminals start in, and `:pwd` shows it.

## Commands worth knowing

Everything is a command; the keys are just bindings. Press `:` and:

| Command | What it does |
| --- | --- |
| `:new-account` | The setup wizard — run this if you skipped it |
| `:help keys` | The full binding list for the current context |
| `:sort -r date` | Sort the list, `-r` reverses the next criterion |
| `:save -a ~/dl/` | Save every attachment to a directory |
| `:export-mbox <file>` | Dump the folder to an mbox file |
| `:import-mbox <path>` | Load an mbox file into the folder |
| `:eml <path>` | Open a `.eml` file in a tab |
| `:reload` | Re-read the config files without restarting |
| `:choose -o y 'Sure?' delete` | Build your own confirmation prompt |
| `:prompt 'Quit?' quit` | Ask before running a command |

## Reviewing patches

The reason aerc exists. On a message containing a patch series:

| Key | Action |
| --- | --- |
| `pl` | List the patches applied to the current project |
| `pa` | Apply the selected patch to a tag |
| `pd` | Drop an applied patch |
| `pb` | Rebase the project |
| `pt` | Open a terminal in the project directory |
| `ps` | Switch the active project |

`:pipe` is the low-tech alternative and works anywhere: `| git am -3`.

## Starting aerc

| Command | What it does |
| --- | --- |
| `aerc` | Open all configured accounts |
| `aerc -a work` | Load only the named account(s), comma-separated |
| `aerc mbox:/path/file` | Open an mbox file as a temporary account |
| `aerc "mailto:me@example.org?subject=Hi"` | Open the composer, pre-filled |
| `aerc :compose` | Run any aerc command at startup |
| `aerc -C ./aerc.conf` | Use a different `aerc.conf` |

If aerc is already running, `mailto:`, `mbox:` and `:command` are handed to that
instance over IPC rather than starting a second one. `-I` disables that.

## Config

Files live in `~/.config/aerc/`:

| File | Purpose |
| --- | --- |
| `aerc.conf` | Behaviour: UI, viewer, compose, filters, openers |
| `accounts.conf` | Accounts and credentials — **must be mode `0600`** |
| `binds.conf` | Keybindings |
| `stylesets/` | Colour schemes |

aerc refuses to start if `accounts.conf` is group- or world-readable; either
`chmod 600` it or set `[general] unsafe-accounts-conf = true`, which is a bad
trade. If `accounts.conf` doesn't exist, the `:new-account` wizard runs
automatically on first start.

### Compose options for a vim user

```ini
# aerc.conf
[compose]
# Only needed if you want something other than $EDITOR:
# editor = nvim

# Edit To/Cc/Subject as text at the top of the buffer in vim, instead of
# using aerc's separate header widgets. Disables :cc, :bcc and :header.
edit-headers = true

# Start with the cursor in the body rather than the To field.
focus-body = true

# Which header fields to show, when edit-headers is false.
header-layout = To|From,Subject

# Only for editors that can't emit CRLF line endings. vim can, so leave false.
lf-editor = false
```

`edit-headers = true` is the setting that makes aerc feel natural if the
separate-fields-then-editor split is what confused you: you get one vim buffer
with the headers as text, write the whole email, `:wq`, review, `y`.

### Rendering HTML mail

```ini
# aerc.conf
[filters]
text/plain = colorize
text/calendar = calendar
text/html = ! html          # requires w3m
```

The `!` prefix marks a filter that needs the terminal. The built-in `html`
filter shells out to `w3m`, so install that first.

### Custom bindings

Bindings go in the section for their context; a bare key sequence maps to a
command plus `<Enter>`, and `<space>` leaves the command line open for an
argument.

```ini
# binds.conf
[messages]
gi = :cf INBOX<Enter>
ga = :cf Archive<Enter>
gs = :cf Sent<Enter>
M  = :move<space>
u  = :read -t<Enter>
<C-r> = :check-mail<Enter>

[view]
w = :toggle-headers<Enter>
```

Writing your own `binds.conf` **replaces** the defaults entirely, so start from
the shipped one:

```sh
curl -O https://git.sr.ht/~rjarry/aerc/blob/master/config/binds.conf
```

---

> This sheet is pinned to **aerc 0.22.0**. Bindings are context-sensitive, so
> `?` inside aerc is always the authoritative list for wherever you happen to
> be. Full docs: <https://aerc-mail.org>, wiki at <https://man.sr.ht/~rjarry/aerc/>.
