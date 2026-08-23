---
name: dms-shell
description: Control DankMaterialShell (DMS) desktop shell and desktop automation — notifications/toasts, media (mpris), volume/mic, brightness, night mode, wallpaper, theme, lock screen, launcher/spotlight, clipboard, screenshots, tray, power menu. Use when user asks to change wallpaper/theme, control media/volume/brightness, toggle DND, lock screen, show a notification, or automate the DMS shell on this machine.
---

# DMS shell control + desktop automation

DMS CLI: `dms <cmd>` for standalone utilities, `dms ipc call <section> <cmd> [args...]` to drive the running shell.
Shell must be live (`pgrep -f "dms run"`). Full command surface: `dms ipc --help`.

Syntax note: some installs use `dms ipc call section cmd args`, others `dms ipc section cmd args`. If `call` errors, drop it.

## Notifications & toasts

```bash
dms notify "Title" "Body"                    # desktop notification
dms notify "Saved" "report.pdf" --file ~/x.pdf --icon download
dms ipc call toast info "message"            # also: warn / error / infoWith
dms ipc call toast dismiss                   # hide current toast
dms ipc call notifications toggleDoNotDisturb
dms ipc call notifications getDoNotDisturb   # query state
```

## Media / audio

```bash
dms ipc call mpris list                      # players + state
dms ipc call mpris playPause | play | pause | next | previous
dms ipc call mpris setvolume 50
dms ipc call mic mute | status | setvolume 30
pamixer --get-volume                         # raw output volume (or dms brightness analog below)
```

## Brightness / night mode / power

```bash
dms brightness set 70 --mon all              # also get, increment, decrement
dms ipc call night toggle | status | setTargetTemp 4000
dms ipc call powerprofile set performance | list
dms ipc call dpms off                        # screen off (also in niri action)
```

## Wallpaper / theme

```bash
dms ipc call wallpaper set ~/Pictures/w.jpg
dms ipc call wallpaper next                  # cycle; get = current path
dms ipc call theme dark | light | toggle | getMode
```

## Launcher / spotlight / widgets

```bash
dms ipc call launcher openQuery "firefox"    # search & launch via shell UI
dms ipc call spotlight openQuery "how to x"
dms ipc call notepad open / toggle           # sticky notes widget
dms ipc call dock toggleAutoHide | status
dms ipc call settings get <key> / set <key> <value>
```

## Lock / session

```bash
dms ipc call lock lock | unlock | isLocked | status
dms ipc call powermenu open                  # logout/reboot/shutdown UI
```

## Desktop automation toolkit (system-wide)

```bash
wl-copy "text" / wl-paste                    # clipboard (Wayland)
grim -g "$(slurp)" shot.png                  # region screenshot → then read it (vision)
wtype "text"                                 # inject keystrokes into focused window
niri msg action screenshot-screen            # full screen (see niri skill)
dms open ~/file.pdf                          # open with app picker
```

Screenshot → analyze loop: `grim -g "$(slurp)" /tmp/s.png` then `read /tmp/s.png` (native image input).

## Notes

- Query-style IPC (`status`, `list`, `get`) prints JSON — safe to parse.
- Don't spam `lock` while user is active; confirm before `powermenu` actions like reboot.
- `dms doctor` diagnoses missing deps; `dms restart` reloads the shell after config edits.
