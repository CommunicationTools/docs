# Communication Tools — documentation

The user manual for the Scop SCADA protocol suite (ModbusScop, DNPScop,
IEC104Scop, IEC101Scop and SCADAScop), built with
[MkDocs](https://www.mkdocs.org/) and the
[Material theme](https://squidfunk.github.io/mkdocs-material/) and published to
GitHub Pages.

## Build it locally

```bash
python -m venv .venv
. .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve                    # live preview at http://127.0.0.1:8000
```

To build the site exactly as CI does (fails on a broken link or missing asset):

```bash
mkdocs build --strict
```

The static site lands in `site/`.

## Publish

Pushing to `main` runs `.github/workflows/docs.yml`, which builds the site and
deploys it to GitHub Pages. Enable Pages once under **Settings → Pages →
Source: GitHub Actions**.

## Optional: PDF export

An offline, single-file PDF of the whole manual can be produced with
[mkdocs-with-pdf](https://github.com/orzih/mkdocs-with-pdf). It is off by
default because it pulls in WeasyPrint, which needs system libraries
(`libpango`, `libcairo`, HarfBuzz and fonts) that make the build heavier and
more brittle. To turn it on:

1. `pip install mkdocs-with-pdf` (and the WeasyPrint system libs for your OS).
2. Uncomment the `with-pdf` block in `mkdocs.yml`.
3. Build with the plugin enabled:

   ```bash
   ENABLE_PDF_EXPORT=1 mkdocs build
   # Windows PowerShell:  $env:ENABLE_PDF_EXPORT=1; mkdocs build
   ```

   The PDF lands at `site/pdf/communication-tools-manual.pdf`. (Skip `--strict`
   for the PDF build — WeasyPrint emits rendering notices that `--strict` would
   treat as fatal.)

## Structure

```
docs/
  index.md                 landing page
  getting-started/         install, first run, concepts
  shared/                  UI shared by every tool (channels, monitor, ...)
  protocols/               one chapter per protocol (master + slave)
  how-to/                  task-based walk-throughs
  reference/               shortcuts, workspace keys, message catalogs, spec
  assets/screenshots/      PNG screenshots (see SCREENSHOTS.md)
  assets/diagrams/         SVG concept diagrams
```

