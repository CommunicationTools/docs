# Screenshot shot list

The manual pages reference the images below. Capture each one and save it
under `docs/assets/screenshots/` with the **exact file name** in the table — the
pages will then show them with no further edits. A missing image breaks
`mkdocs build --strict`, so every name in a page must exist here.

## Capture conventions

- **Theme:** Dark is the primary look — capture in Dark unless the row says
  otherwise. Also capture the Themes page in Light.
- **Window size / scaling:** the SCADAScop 2.0 set was taken on a 1920×1080
  screen at **100 %** scaling with the main window maximized, then cropped to
  the window of interest; keep new shots at 100 % so text sizes match.
- **Format & name:** **PNG**, lowercase, `<area>-<window>-<variant>.png`
  (`scadascop-display-edit.png`, `scopctl-status.png`).
- **Data:** the SCADAScop 2.0 set uses `docs/sample-workspaces/feeder-demo.ssw`
  (one DNP3 outstation *OS10* with the feeder-bay script, the SCADA master, one
  display). Re-shoot from it so the pictures stay consistent. Nothing you would
  not want public.
- **Annotations:** none so far; if a page needs callouts, note it in the
  "Notes" column.

## Shot list — SCADAScop 2.0 features (captured 2026-11-02)

| File | Page | Window / state | Status |
|---|---|---|---|
| `scadascop-overview.png` | SCADAScop chapter, Script examples | Main window, feeder-demo: Channels tree (docked), display (run), master map (Indications, Tag column), Script window | ✔ |
| `scadascop-display-run.png` | HMI displays | The *Feeder 1 bay* display window in Run mode, breaker closed, live values | ✔ |
| `scadascop-display-edit.png` | HMI displays | Same display in Edit mode, gauge selected, properties panel | ✔ |
| `scadascop-display-add-menu.png` | HMI displays | Edit mode, **Add..** menu open | ✔ |
| `scadascop-display-tag-browser.png` | HMI displays | Tag browser opened from a Browse button, every tag of SCADA/OS10 | ✔ |
| `scadascop-display-command.png` | HMI displays | Command dialog after the SELECT: *Selected – select confirmed*, Execute / Cancel | ✔ |
| `scadascop-displays-window.png` | HMI displays | Tools › Displays with the one display listed | ✔ |
| `scadascop-script-window.png` | RTU scripting | Script window with Tags panel: editor, console, tag list with cmd / sp marks | ✔ |
| `scadascop-script-library.png` | RTU scripting | Library viewer (read-only stdlib.lua) | ✔ |
| `scadascop-rtu-context-menu.png` | RTU scripting | Right-click on the slave RTU: Points…, Settings…, Communication Tests…, Script… | ✔ |
| `scadascop-slave-points-tags.png` | RTU scripting | Slave Points window, Measurands tab, Tag column with names | ✔ |
| `scadascop-master-map-tag-column.png` | RTU scripting | Master map window, Indications tab, Tag column | ✔ |
| `scadascop-watch-window.png` | Watch window | Watch 1 with eight master points, rows flashing | ✔ |
| `scadascop-view-desktops-menu.png` | Windows and monitors | View menu with the Desktops submenu open | ✔ |
| `scadascop-desktop-window.png` | Windows and monitors | *Desktop 2* with the display docked in it | ✔ |
| `scadascop-web-interface-dialog.png` | Headless & web | Tools › Web Interface, server running on 127.0.0.1:8080 | ✔ |
| `scopctl-status.png` | scopctl | Terminal: `scopctl status --rtus` then `scopctl script status` against feeder-demo | ✔ |
| `scopctl-tui.png` | scopctl | Terminal: `scopctl` full-screen Overview (F1) | ✔ |

## Shot list — earlier pages

| File (`docs/assets/screenshots/…`) | Tool | Window / state to set up | Notes |
|---|---|---|---|
| `first-run-overview.png` | DNPScop Master + Slave | Both apps side by side, Master polling the Slave, a point map visible | Used on First run |
| `shared-channels.png` | any tool (DNPScop Master) | Channels window with 2–3 channels, one expanded to show its RTU, a right-click context menu open | Shows the tree + menu |
| `shared-points.png` | ModbusScop Slave | A register map window with a few named points; header right-click menu open | Shows the column menu |
| `shared-monitor.png` | any tool (DNPScop Master) | Communication Monitor with traffic, a row selected so the layered decode pane shows | — |
| `shared-pcap.png` | SCADAScop | Communication Monitor with the PCAP capture controls visible | — |
| `shared-workspace.png` | any tool | File menu open showing Save / Open / New Workspace | Small crop is fine |
| `shared-fanout.png` | any Master | New Channel dialog with Count > 1 and the Addressing block + preview visible | — |
| `shared-tls.png` | any tool | Channel Settings dialog, TLS section | — |
| `shared-themes.png` | any tool | View → Theme → Settings… editor open | Capture one in Light too if easy |
| `shared-updates.png` | any public tool | Help → Check for updates… result dialog | Can be the "up to date" or "new version" state |
| `dnp3-master-commflow.png` | DNPScop Master | Messages window → Communication Flow tab, a phase expanded with a few rows | — |
| `dnp3-master-inspect.png` | DNPScop Master | Messages window → Inspect tab, a Group read row expanded with Range selected | Key screenshot |
| `dnp3-slave-commtest.png` | DNPScop Slave | RTU → Communication Tests window, a couple of faults armed with non-zero Fired counts | Key screenshot |
| `charts-window.png` | any Master (or SCADAScop) | Tools → Charts with a chart or two plotting a measurand / indication over time | Used on Charts |
| `web-interface.png` | SCADAScop (web UI) | The headless web page in a browser — groups/channels/RTUs, the status feed, the simulation box | Used on Headless; a browser window, not the app |

## Still wanted (pages without a picture)

| File | Page | Window / state |
|---|---|---|
| `shared-simulation.png` | Simulation | A slave Points window with a point's simulation settings (Sine / Ramp) open, the simulation toolbar control running |
| `shared-p2p.png` | P2P monitoring | New Channel → P2P Monitoring dialog, or a running P2P channel's Communication Monitor |
| `scadascop-floatconv.png` | SCADAScop chapter | Tools → Floating-Point Converter with a value entered |
| `scadascop-channels-rail.png` | SCADAScop chapter | The Channels panel collapsed to the icon rail |
| `modbus-value-lists.png` | Modbus chapter | RTU window → Lists tab with an enumeration and a bit-name list |
| `iec104-master-interrogation.png` | IEC 104 chapter | Master map after a general interrogation |

## Adding more

When a page needs a new image, add a row here, reference
`../assets/screenshots/<name>.png` from the page, and drop the PNG in.
