# Install

The tools are self-contained Windows executables — there is no installer and
nothing is written to the registry.

## Download

Each tool is published on its GitHub release page under
[github.com/CommunicationTools](https://github.com/CommunicationTools). Download
the archive (`.zip` / `.7z`) for the tool you want, or the SCADAScop archive to
get the whole suite in one shell.

## Requirements

- Windows 10 or later, 64-bit.
- A serial port / USB-serial adapter for RTU links, or a network connection for
  TCP links. No driver is installed by the tools themselves.

The tools also run on Linux desktops: see
**[Running on Linux](linux.md)** for what differs there.

## Folder layout

Unzip anywhere (for example `C:\Tools\ScadaScop\`). A release archive contains:

```
<Tool>.exe                  the application
Run<Tool>-GPU.bat           start with the GPU (hardware) renderer — the default
Run<Tool>-CPU.bat           start with the CPU (software) renderer
mesa\opengl32.dll           bundled software OpenGL, used by CPU rendering
resources\certs\            demo TLS certificates (for the TLS walk-through)
tools\New-ScopLabCerts.ps1  a PowerShell script to generate lab certificates
```

Settings are kept in `.ini` files in your user folder,
`%APPDATA%\CommunicationTools` on Windows and `~/.config/CommunicationTools`
on Linux (for example `dnpscop_master.ini`, and shared colour overrides in
`scop_colors.ini`). The program folder is never written, so the tools can run
from a read-only location. To keep the settings somewhere else, for example
beside a portable copy, set the `SCOP_USER_DIR` environment variable to that
folder before starting the tool.

!!! tip "Keep the `resources` and `tools` folders next to the executable"
    The TLS walk-through and the "generate lab certificates" button look for
    `resources\certs` and `tools\New-ScopLabCerts.ps1` beside the tool.

## GPU or CPU rendering

Every tool draws its interface with the **GPU** by default (hardware OpenGL 3) —
the fastest and lightest option. On a machine with no usable GPU or OpenGL 3
driver (an older virtual machine, some remote-desktop sessions), a tool can fall
back to **CPU** rendering using the bundled Mesa software OpenGL
(`mesa\opengl32.dll`, which must sit next to the executable).

Two batch files ship beside each tool, so you can pick without typing a command:

- **`Run<Tool>-GPU.bat`** — the normal, GPU-accelerated way (what double-clicking
  the `.exe` already does).
- **`Run<Tool>-CPU.bat`** — forces software rendering (passes `--cpu`); use it
  when the GPU path shows a blank window or refuses to start.

You can also switch from inside the app under **View → Rendering**:

- **GPU (hardware)** / **CPU (software)** — the choice takes effect on the next
  restart (the active renderer is shown in the menu and in Help → About).
- **Refresh rate** — cap how often the interface redraws. Lowering it noticeably
  reduces CPU use, which is handy on a busy bench machine or when running on the
  CPU renderer; the data and the Communication Monitor keep updating regardless.

!!! note "CPU mode needs Mesa"
    CPU rendering looks for `mesa\opengl32.dll` next to the executable. Keep the
    `mesa` folder with the tool if you intend to use software rendering.

## First launch

Double-click the executable. On first run the tool opens with an empty
workspace — continue to **[First run](first-run.md)**.
