# Running on Linux

The tools also build and run on Linux desktops (x86-64). They behave as on
Windows; this page covers what is different: checking that a build fits your
system, file dialogs, the firewall, serial ports and working over SSH.

## Check your system first

A Linux build runs on a system whose C library (glibc) is the same version or
newer than the one it was built on. To see what your machine has:

```bash
ldd --version | head -1
cat /etc/os-release | head -2
```

The first line of the first command ends with the glibc version (for example
`2.34`); the second command names the distribution and its release.

If the build is too new for the machine, the tool does not start and prints
lines such as:

```
./SCADAScop: /lib64/libc.so.6: version `GLIBC_2.38' not found (required by ./SCADAScop)
```

That is not a damaged download. Use a build made for an older system, or run
the tool on a newer distribution.

## Start a tool

Unpack the archive anywhere, then from a terminal in that folder:

```bash
chmod +x SCADAScop        # once, if the archive did not keep the permission
./SCADAScop
```

The GUI tools need a desktop session (a display). Start them from a terminal
inside the desktop, not from a plain SSH login; over SSH use
[headless mode](#over-ssh-headless-and-the-web-interface) instead.

Settings are kept in `~/.config/CommunicationTools`. To keep them somewhere
else, set `SCOP_USER_DIR` to that folder before starting the tool.

!!! note "No usable GPU"
    The `--cpu` switch and the `Run<Tool>-CPU.bat` files are for Windows. On
    Linux, software rendering is selected with a Mesa variable:
    `LIBGL_ALWAYS_SOFTWARE=1 ./SCADAScop`.

## File dialogs

Open, Save and Browse use the desktop's own file chooser. The tool looks for a
way to show it in this order:

1. the **desktop portal** (xdg-desktop-portal), which is part of a normal GNOME
   or KDE session, so nothing has to be installed;
2. the **zenity** program (**kdialog** first on KDE), if the portal does not
   answer.

If neither is available the Browse buttons do nothing and the terminal shows a
line saying what to install (for example `sudo dnf install zenity`).

To force one of them, for instance to find out why a dialog does not appear,
set `SCOP_FILE_DIALOG` when starting the tool:

```bash
SCOP_FILE_DIALOG=portal ./SCADAScop
```

| Value | Effect |
|---|---|
| `portal` | only the desktop portal; no fallback to a helper program |
| `zenity` | only zenity |
| `kdialog` | only kdialog |
| `none` | no file dialog at all |

Without the variable the order above applies.

A Save name typed without an extension gets the right one added (`plant`
becomes `plant.ssw`), as on Windows.

## Network ports and the firewall

Most distributions block incoming connections by default. A slave (server)
channel, or the web interface, is reachable from another computer only after
its port is opened. With firewalld (Fedora, RHEL, CentOS, AlmaLinux, Rocky):

```bash
sudo firewall-cmd --add-port=20000/tcp               # until the next reboot
sudo firewall-cmd --add-port=20000/tcp --permanent   # keep it
```

The usual ports are 502 (Modbus TCP), 20000 (DNP3) and 2404 (IEC 60870-5-104);
use the port set in the channel. Master (client) channels connect outwards and
need no rule.

!!! note "Ports below 1024"
    Linux lets only privileged programs listen on ports below 1024, which
    includes Modbus 502. Either use a higher port on the slave channel, or
    allow the tool once with
    `sudo setcap 'cap_net_bind_service=+ep' ./SCADAScop`.

To test against Windows, run a slave on one machine and the matching master on
the other, in either direction, with the slave's port opened in the firewall of
the machine it runs on.

## Serial ports

Serial ports are named by device path: `/dev/ttyS0` for a built-in port,
`/dev/ttyUSB0` or `/dev/ttyACM0` for a USB adapter. Type the path in the
channel's serial port field in place of `COM1`.

Your user must be allowed to open the port. On most distributions that means
membership of the `dialout` group:

```bash
sudo usermod -aG dialout $USER
```

Log out and in again for the change to apply.

In a virtual machine, pass the USB adapter through to the VM in the
hypervisor's settings so it shows up as `/dev/ttyUSB0` inside it.

## Over SSH: headless and the web interface

**SCADAScop** runs without a window, so it can be started over SSH and watched
from a browser on another computer:

```bash
./SCADAScop --headless plant.ssw --start-all --web 8080 --web-bind 0.0.0.0
```

Then open the web port in the firewall and browse to
`http://<linux-machine>:8080/` from the other computer:

```bash
sudo firewall-cmd --add-port=8080/tcp
```

When the web interface is bound to anything other than `127.0.0.1` it requires
an access token; see [Headless and the web interface](../shared/headless.md)
for the token, the read-only mode and the other switches.

## TLS lab certificates

The **Generate lab certificates** button in the TLS section works on Linux. The
`tools\New-ScopLabCerts.ps1` script is PowerShell and applies to Windows only.
