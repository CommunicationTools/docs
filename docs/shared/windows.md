# Windows and monitors

SCADAScop's windows do not have to stay inside the main window. Drag the
Communication Monitor, a chart, an HMI display or a point map onto a second
monitor, dock several windows together there, or open a **Desktop** — an empty
window of its own to dock into.

## Windows outside the main window

**View → Windows outside the main window** is on by default.

- Drag a window by its title bar (or a docked window by its tab) past the edge
  of the main window: it becomes a window of its own, with its own taskbar
  entry, which you can put on another monitor.
- Drag other windows onto it to dock them there — as tabs, or side by side using
  the docking targets that appear while you drag.
- Drag it back over the main window to put it back inside.
- **View → Bring all windows back** moves every window that is outside the main
  window back into it (the menu shows how many there are).
- Turning the setting off brings every window back first.
- **View → Always on top** keeps the detached windows on top too.

Windows outside the main window reopen where you left them at the next start.
If that monitor is no longer connected, they come back on one that is.

Dialogs open over the window you are working in: a dialog started from a window
on the second monitor appears on that monitor.

When **View → Scaling → Auto** is selected, the text in a window on a monitor
with a different Windows scale (for example 100 % and 150 %) follows that
monitor. Spacing and padding keep the main window's scale.

!!! note "Linux"
    Works on X11. Under a Wayland session the window system does not let an
    application place its own windows; the menu item is then shown disabled and
    every window stays inside the main window.

## Desktops

A Desktop is an empty window for docking windows into — a second screen for
SCADAScop. It has a normal title bar, so it can be moved, maximized, snapped
(<kbd>Win</kbd>+<kbd>←</kbd> / <kbd>→</kbd>) and minimized like any other window,
and it never merges back into the main window.

- **View → Desktops → New desktop** opens *Desktop 2*, *Desktop 3*, … A new
  desktop opens on a monitor other than the main window's; with one monitor it
  opens over the main window.
- Drag a window's title bar or tab onto the desktop to dock it there; dock more
  windows beside it or as tabs.
- **View → Desktops → Desktop N** brings a desktop to the front.
- Closing a desktop (its close button, or **View → Desktops → Close all
  desktops**) moves the windows docked on it back into the main window.

Desktops, their size and place, and the windows docked on them are restored at
the next start. They are part of your window layout (stored in your user
folder), not of the workspace file.

## Where it is stored

| Setting | File |
|---|---|
| Windows outside the main window (`DetachableWindows=`) | `scadascop.ini` |
| Open desktops (`Desktops=`, e.g. `2,3`) | `scadascop.ini` |
| Window places, monitors, what is docked where | `scadascop_layout.ini` |

Both files are in the per-user settings folder (`%APPDATA%\CommunicationTools`,
Linux `~/.config/CommunicationTools`).

!!! tip
    The standalone tools keep every window inside their main window.
