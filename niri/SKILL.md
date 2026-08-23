---
name: niri
description: Control the running niri Wayland compositor — list windows/workspaces/outputs, focus/move/resize windows and columns, manage workspaces, screenshots, spawn apps, watch event stream. Use when user asks to arrange windows, switch workspace, find a window, take a screenshot, or any desktop window management on this machine.
---

# niri compositor control

All commands go through `niri msg`. Add `-j` for JSON output (best for parsing).
Session must be live (`pgrep niri`); commands run as the logged-in user.

## Query state

```bash
niri msg -j windows            # all windows: id, title, app_id, workspace, layout
niri msg -j workspaces         # workspaces per output
niri msg -j outputs            # monitors
niri msg focused-window        # current focus (also -j)
niri msg pick-window           # click a window to identify it
```

Find a window: `niri msg -j windows | jq '.[] | select(.app_id=="firefox") | .id'`

## Actions

```bash
niri msg action focus-window --id <ID>
niri msg action close-window --id <ID>       # omit --id = focused
niri msg action fullscreen-window --id <ID>
niri msg action move-window-to-workspace <name|index> --id <ID>
niri msg action do-screen-transition          # overview
```

Columns (niri scrolls horizontally; a column holds stacked windows):

```bash
focus-column-left/right/first/last    move-column-left/right/to-index
focus-column-or-monitor-left/right    move-column-to-workspace <ws>
consume-window-into-column / expel-window-from-column   # merge/unmerge into column
toggle-column-tabbed-display / set-column-display tabbed
center-column / center-window
expand-column-to-available-width
resize: set-window-height <val> / set-column-width <val>  (e.g. 60 or "+10%")
```

Workspaces:

```bash
focus-workspace <name|index>     move-window-to-workspace <ws>
move-workspace-to-index <idx> --output <out>
set-workspace-name <name> --workspace <idx>
```

Monitors:

```bash
focus-monitor-left/right/up/down
move-window-to-new-workspace --monitor-next   # spread across outputs
power-off-monitors / power-on-monitors
```

## Screenshots & misc

```bash
niri msg action screenshot-screen    # saves to ~/Pictures/Screenshots
niri msg action screenshot-window
niri msg action screenshot           # interactive UI
niri msg action spawn -- <cmd...>    # launch app in session
niri msg action spawn-sh -- "<shell cmd>"
niri msg event-stream                # live JSON events (pipe through jq)
```

## Notes

- `screenshot-screen` prints nothing; file lands in `~/Pictures/Screenshots` — verify with `ls -t`.
- Window `id` is stable until closed; re-query after spawning apps.
- Full action list: `niri msg action --help`.
