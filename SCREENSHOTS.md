# Screenshot shot list

The manual pages already reference the images below. Capture each one and save it
under `docs/assets/screenshots/` with the **exact file name** in the table — the
pages will then show them with no further edits. Placeholder images are shipped
so the site builds before you add the real ones; just overwrite them.

## Capture conventions

- **Theme:** Dark is the primary look — capture in Dark unless the row says
  otherwise. Also capture the Themes page in Light.
- **Window size / scaling:** a fixed window size (≈ **1280×800**) at **100 %**
  display scaling, so images are uniform.
- **Format & name:** **PNG**, lowercase, exactly the file name in the table.
- **Data:** use neutral demo data (a couple of channels / points). Nothing you
  would not want public.
- **Annotations:** if you want numbered callouts or arrows, note it in the "Notes"
  column and either add them yourself (the OS snipping tool works) or leave it to
  the docs pass.

## Shot list

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

## Adding more

When a page needs a new image, add a row here, reference
`../assets/screenshots/<name>.png` from the page, and drop the PNG in. Keep names
`<area>-<window>-<variant>.png` (for example `iec104-master-interrogation.png`).
