# Print Shop

Get documents printed: prepare PDFs for digital printing, park them in a printing folder (by date, by subject or per printer), and submit them to print shops the way each one takes jobs (Drive share with their Gmail, email, web upload or walk-in).

Formerly `digital-printing` (renamed in 2.0.0, 2026-11-15). Existing config under `digital-printing/` is migrated on first use.

The plugin covers the full pre-print pipeline: profiling a PDF, fixing the obscure problems that bite on office printers (non-embedded fonts, live transparency, oversized images, mixed color spaces), preparing the layout (resize, bind, strip footers, add cover pages, watermarks, branded footers), bundling files into a print job folder with a formal order document, and shipping it to a digital printer by email or Google Drive share.

> **Scope:** v1 is documents (PDFs) only. Skills for image and banner prep (JPEG/TIFF/canvas-size/print-bleed for posters, etc.) will be added later.

## Skills

### Page-level transforms

| Skill | Purpose |
|---|---|
| `resize-page` | Rescale page size — Letter ↔ A4, A5 ↔ A4, custom dimensions |
| `color-to-grayscale` | Convert color PDF to grayscale for monochrome printing |
| `remove-pages` | Drop specified pages, ranges, or auto-detected blanks |
| `strip-footers-and-page-numbers` | Whiteout or text-aware footer removal |
| `optimize-for-binding` | Add gutter; mirror margins for duplex |
| `add-cover-page` | Generate matching-size cover with title and prepend |

### Overlays

| Skill | Purpose |
|---|---|
| `add-watermark` | 45° diagonal text watermark (DRAFT, PRIVATE, etc.) at low opacity |
| `burn-in-footer` | Permanent footer label (CLIENT CONFIDENTIAL, EMBARGOED UNTIL, etc.) with PNG preview verification |

### Pre-flight & quality fixes

| Skill | Purpose |
|---|---|
| `verify-bleed-safe` | Read-only check: does any content extend into the unprintable margin? |
| `add-bleed` | Add 3mm bleed for commercial print shops that trim |
| `embed-fonts` | Force-embed all fonts (fixes the most common print-mismatch bug) |
| `flatten-transparency` | Eliminate live transparency (banding/color shifts on cheap printers) |
| `downsample-images` | Cap embedded images at 300 DPI; reduces file size and printer memory load |
| `normalize-color-profile` | Standardize on sRGB (office) or CMYK FOGRA39 (print shop) |

### Metadata

| Skill | Purpose |
|---|---|
| `strip-metadata` | Wipe author, title, producer, comments, XMP |
| `write-metadata` | Set title, author, subject, keywords |

### Job packaging & sharing

| Skill | Purpose |
|---|---|
| `create-job-folder` | Bundle one or more verified PDFs into a single job folder with a manifest |
| `create-print-order` | Generate a formal printer-facing order (PDF + JSON) — copies, color, paper, binding, finishing, contact info |
| `zip-job-folder` | Archive the folder for transmission |
| `file-for-print` | File a finished file into the printing folder: by date, by subject, or into a printer's queue |
| `add-printer` | Register a printer: email / Gmail, how it takes jobs, extra steps; creates its queue folder |
| `send-to-printer` | Submit a file to a registered printer by its saved procedure, and log it |
| `share-job-folder` | Send the job to a printer or recipient via email (zip attached) and/or Google Drive share link |

## Agent

| Agent | Purpose |
|---|---|
| `print-prep-orchestrator` | Profiles a PDF (fonts, images, transparency, color, bleed safety, metadata) and orchestrates the right combination of atomic skills to make it print-ready, with a confirm-before-execute plan |

## Typical workflows

**One-shot prep, then upload:**

```
Hand a PDF to the orchestrator → it profiles & produces a print-ready output → file-for-print
```

**Park it and send it to a print shop:**

```
file-for-print (printer queue) → send-to-printer   # add-printer first for a new shop
```

**Build a multi-file job for a print shop:**

```
For each file: run orchestrator → verify-bleed-safe
Then: create-job-folder → create-print-order → zip-job-folder → share-job-folder
```

**Quick fix for a misbehaving PDF:**

```
embed-fonts → flatten-transparency → downsample-images
```

## Configuration

All user data lives in a private overlay directory, not in the plugin: `config.json`
(printing folder, its By Date / By Subject / Printers layout, contact block) and
`printers.json` (each printer's addresses, submission method and procedure, queue folder,
history). Location, resolution order and schemas: [references/user-data.md](references/user-data.md).

The printing folder itself (local or Google Drive) is the user's; the plugin stores only a
pointer to it. Nothing is written under `~/.claude/plugins/`.

## Tooling assumptions

The skills assume these standard CLI tools are available. Skills will check and prompt to install if missing:

- `ghostscript` (`gs`) — most heavy lifting (resize, grayscale, embed, flatten, downsample, color profile)
- `qpdf` — page selection, overlays, linearize
- `poppler-utils` (`pdfinfo`, `pdffonts`, `pdfimages`, `pdftotext`) — probing
- `mupdf-tools` (`mutool`) — alternative geometry probe, transparency detection
- `exiftool` — metadata read/write
- `pdfjam` (from `texlive-extra-utils`) — gutter/binding offsets, bleed
- `python3` with `reportlab` — overlay PDF generation for watermarks/footers
- `typst` (optional) — clean cover pages and order documents
- `zip`

## Installation

```bash
claude plugins install print-shop@danielrosehill
```

Then restart Claude Code.

## License

MIT
