# `btop` cheatsheet

A resource monitor that shows CPU, memory, disks, network and processes in
resizable boxes, with mouse support and a full-screen options menu. The
successor to `bashtop` and `bpytop`, rewritten in C++.

> **The cheatsheet inside btop:** press **`?`** (or `F1`, or `h`) for the help
> window listing every binding. **`Esc`** or `m` opens the main menu
> (Options / Help / Quit), and **`o`** (or `F2`) jumps straight to the options.
> Every setting in `btop.conf` can be changed live from there.

> **btop overwrites `btop.conf` when it exits.** `save_config_on_exit` defaults
> to `true`, so editing the file while btop is running loses your edits. Either
> quit first, or change settings from the options menu (`o`) and let btop write
> them, or set `save_config_on_exit = False` if you want the file to be yours
> alone. `Ctrl-r` re-reads the config from disk without restarting.

## The one thing that trips everyone up

`+` and `-` mean two different things depending on whether a process is
selected:

| State | `+` / `-` does |
| --- | --- |
| Nothing selected | Add / subtract 100 ms from the update timer |
| A process selected, tree view on | Expand / collapse that process's branch |

So if the refresh rate suddenly stops responding to `+`, you have a process
selected — press `Esc`... except `Esc` opens the main menu. Click elsewhere, or
scroll the selection off the list, to deselect.

## Global

| Key | Action |
| --- | --- |
| `Esc` / `m` | Main menu (Options, Help, Quit) |
| `o` / `F2` | Options menu |
| `?` / `F1` / `h` | Help — the full binding list |
| `q` / `Ctrl-c` | Quit |
| `Ctrl-z` | Suspend to the background (`fg` to return) |
| `Ctrl-r` | Reload `btop.conf` from disk |
| `+` / `-` | Update timer ±100 ms (when no process is selected) |
| `p` / `P` | Cycle view presets forwards / backwards |

Mouse 1 clicks buttons and selects processes; the scroll wheel scrolls whatever
list is under the cursor. Set `disable_mouse = True` if your terminal's own
selection matters more.

## Boxes

| Key | Toggles |
| --- | --- |
| `1` | CPU box |
| `2` | MEM box |
| `3` | NET box |
| `4` | PROC box |
| `5` … `9` | GPU 1 … GPU 5 (builds with GPU support) |
| `0` | GPU 6 |

Toggling a box off gives the rest more room; toggling to a layout that can't
fit the terminal pops up a size error instead. Hiding a box also clears the
current preset, since you've diverged from it.

## Presets

Presets are named layouts you cycle with `p` and `P` — a wide "everything" view
for a big terminal, a CPU-and-processes view for a small pane.

```
# btop.conf — three presets, whitespace-separated
presets = "cpu:1:default,proc:0:default cpu:0:default,mem:0:default,net:0:default cpu:0:block,net:0:tty"
```

The format is `box:P:G` per box, commas between boxes in one preset, spaces
between presets. `P` is `0` or `1` for the box's default or alternate position;
`G` is the graph symbol for that box (`default`, `braille`, `block`, `tty`).
Preset 0 is always "all boxes, default settings" and is not written in the
config. Maximum 9 custom presets.

Start on a specific one with `btop -p 2`.

## Processes: moving around

| Key | Action |
| --- | --- |
| `Up` / `Down` | Select in the process list |
| `Pg Up` / `Pg Down` | Jump one page |
| `Home` / `End` | First / last page |
| `Enter` | Detailed view for the selected process (`Enter` again closes it) |
| `u` | Pause the process list — freeze it to read a fast-moving row |

## Processes: what's shown

| Key | Action |
| --- | --- |
| `Left` / `Right` | Previous / next sorting column |
| `r` | Reverse the sort order |
| `e` | Toggle tree view |
| `E` | Collapse / expand **all** branches in tree view |
| `Space` | Expand/collapse the selected branch |
| `C` | Expand/collapse only the selected process's *children* |
| `c` | Per-core CPU% — usage of the core it's on, not of the whole CPU |
| `%` | Memory as bytes vs. percent |

Sorting columns are `pid`, `program`, `arguments`, `threads`, `user`, `memory`,
`cpu lazy`, `cpu direct`. The default is **`cpu lazy`**, which lets the top
process settle over a few samples so the list doesn't thrash; `cpu direct`
re-sorts every update and is jumpier but truthful. Clicking a column header
sorts by it too.

## Processes: filter and follow

| Key | Action |
| --- | --- |
| `f` or `/` | Enter a filter — the list narrows as you type |
| `Enter` | Commit the filter and leave the input (filter stays applied) |
| `Esc` | Cancel — restores the filter as it was before you started typing |
| `Delete` | Clear an applied filter |
| `F` | Follow the selected process — keep it in view as the sort shuffles |

Start the filter with `!` for a regular expression: `!^(ssh|sshd)$`. A plain
filter matches the command and arguments as a substring.

`F` again on the same process stops following. `btop -f nginx` starts with a
filter already applied, which is the quick way to launch a targeted view from
the shell.

## Processes: signals and priority

| Key | Action |
| --- | --- |
| `t` | Terminate — SIGTERM (15) |
| `k` | Kill — SIGKILL (9) |
| `s` | Signal menu — pick or type any signal number |
| `N` | Renice — set a new nice value |

All four work on the selected process, or on the process in the detailed view.
Each opens a confirmation box: `Enter` or `Space` sends, `Esc` or `q` backs
out. In the signal menu, arrows (or `hjkl`) move over the signal grid,
`Backspace` deletes a typed digit.

Sending a signal to a process you don't own fails with "Insufficient
permissions" — run btop under `sudo` if that's what you're there to do.

> With `vim_keys = True`, `k` is taken by "move up", so **kill becomes `K`**
> and **help becomes `H`**.

## Memory and disks

| Key | Action |
| --- | --- |
| `d` | Toggle the disks panel inside the MEM box |
| `i` | Disk I/O mode — replaces the usage meters with read/write graphs |

Disks come from `/etc/fstab` by default (`use_fstab = True`), which also
disables `only_physical`. If a mounted disk isn't listed, that's usually why:
set `use_fstab = False` to read the live mount table instead, or list
mountpoints explicitly in `disks_filter`.

## Network

| Key | Action |
| --- | --- |
| `b` / `n` | Previous / next network device |
| `a` | Toggle auto-scaling of the graphs |
| `y` | Toggle synced scaling — up and down share one scale |
| `z` | Reset the totals counter for the current device |

Auto-scaling rescales down to 10 KiB at the lowest, so an idle interface still
shows shape. Turn it off and the fixed `net_download` / `net_upload` ceilings
apply, which is what you want when comparing two interfaces by eye.

## Menu navigation

Once a menu is open, btop's keys belong to the menu:

| Key | Action |
| --- | --- |
| `Up` / `Down` | Move between options |
| `Tab` / `Shift-Tab` | Next / previous category page |
| `Left` / `Right` | Change the highlighted value |
| `Enter` | Edit a text field (type, then `Enter` to accept) |
| `Esc` / `q` / `o` / `Backspace` | Close the options menu |

Options take effect immediately, so you can watch the UI change as you scroll
through them. `background_update = False` stops the main UI redrawing behind
the menu if the flicker bothers you.

## Command line

| Command | What it does |
| --- | --- |
| `btop` | Start with the saved config |
| `btop -p 2` | Start with preset 2 (0–9) |
| `btop -f nginx` | Start with a process filter applied |
| `btop -u 500` | Start with a 500 ms update rate |
| `btop -c ~/alt.conf` | Use a different config file |
| `btop --default-config` | Print the default config to stdout |
| `btop --themes-dir ~/themes` | Use a custom themes directory |
| `btop -t` / `btop --no-tty` | Force / forbid TTY mode (ANSI graphs, 16 colours) |
| `btop -l` | 256 colours only — no truecolor |
| `btop --force-utf` | Override the automatic UTF locale detection |
| `btop -d` | Debug mode: extra logging and timing metrics |
| `btop -V` / `btop --version` | Version (twice for build details) |

`btop --default-config > ~/.config/btop/btop.conf` is the clean way to start a
config from scratch — the output is fully commented.

## Config

The config lives at `$XDG_CONFIG_HOME/btop/btop.conf`, falling back to
`~/.config/btop/btop.conf`. Logs go to `~/.local/state/btop.log`.

| Option | Default | What it does |
| --- | --- | --- |
| `color_theme` | `Default` | A `.theme` file name, or `Default` / `TTY` |
| `theme_background` | `True` | `False` lets the terminal background show through |
| `truecolor` | `True` | 24-bit colour; `False` converts to a 256-colour cube |
| `vim_keys` | `False` | `h j k l g G` for navigation |
| `disable_mouse` | `False` | Turn off all mouse events |
| `update_ms` | `2000` | Refresh interval; 2000 ms or above gives better graphs |
| `graph_symbol` | `braille` | `braille` (finest), `block`, or `tty` |
| `shown_boxes` | `cpu mem net proc` | Plus `gpu0`–`gpu5` |
| `proc_sorting` | `cpu lazy` | See the sorting note above |
| `proc_tree` | `False` | Start in tree view |
| `proc_per_core` | `False` | Process CPU% relative to one core |
| `proc_mem_bytes` | `True` | Memory in bytes rather than percent |
| `proc_cpu_graphs` | `True` | A sparkline per process row |
| `proc_filter_kernel` | `False` | Hide kernel threads, htop-style (Linux) |
| `cpu_bottom` | `False` | Move the CPU box to the bottom |
| `proc_left` | `False` | Move the process box to the left |
| `mem_below_net` | `False` | Swap the MEM and NET box positions |
| `check_temp` | `True` | Read CPU temperatures |
| `temp_scale` | `celsius` | Also `fahrenheit`, `kelvin`, `rankine` |
| `show_uptime` | `True` | Uptime in the CPU box |
| `show_swap` | `True` | Swap in the memory box |
| `show_disks` | `True` | Split the MEM box to show disks |
| `use_fstab` | `True` | Read the disk list from `/etc/fstab` |
| `only_physical` | `True` | Hide network/RAM disks (ignored when `use_fstab`) |
| `io_mode` | `False` | Start with disk I/O graphs |
| `net_auto` | `True` | Auto-rescale the network graphs |
| `base_10_sizes` | `False` | KB = 1000 instead of KiB = 1024 |
| `clock_format` | `%X` | `strftime`, empty to disable |
| `background_update` | `True` | Keep redrawing the UI behind menus |
| `save_config_on_exit` | `True` | Write settings on quit — see the note up top |
| `log_level` | `WARNING` | `ERROR`, `WARNING`, `INFO`, `DEBUG` |

`clock_format` also understands `/host`, `/user` and `/uptime` on top of the
usual `strftime` codes:

```ini
# btop.conf
clock_format = "/user@/host  %H:%M  up /uptime"
update_ms = 1000
vim_keys = True
proc_tree = True
proc_filter_kernel = True
save_config_on_exit = False
disks_filter = "/ /home /mnt/data"
```

`disks_filter` takes full mountpoints separated by whitespace; only matching
disks are shown.

### Themes

Themes are `.theme` files searched for in `~/.config/btop/themes/` and then the
system directories (`/usr/share/btop/themes`, `/usr/local/share/btop/themes`).
Drop a file in the user directory and it appears in the `color_theme` picker in
the options menu — that's easier than typing the name into the config, since
the picker only lists what it actually found.

bpytop and bashtop theme files work unchanged.

---

> This sheet is pinned to **btop 1.4.7**. Bindings are context-sensitive — the
> process keys only apply while the PROC box is shown, and `k`/`h` move when
> `vim_keys` is on — so `?` inside btop is always the authoritative list. GPU
> monitoring only exists in builds compiled with `GPU_SUPPORT=true`, and Intel
> GPU readings additionally need `make setcap` or root. Full docs:
> <https://github.com/aristocratos/btop>.
