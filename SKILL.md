---
name: datasheet-to-orcad
description: Convert IC datasheet PDFs into OrCAD Capture symbol libraries (.olb) and schematics (.dsn) using the Module_01 pipeline. Use when the user wants to generate OrCAD symbols from datasheets, run the symbol generation pipeline, troubleshoot extraction issues, or work with the Module_01 project at D:\WorkSpace\Claude\Module_01.
---

# Datasheet to OrCAD Symbol Generation

Convert IC datasheet PDFs → `.olb` (symbol library) + `.dsn` (schematic) via a 5-stage pipeline with dual-channel cross-validation.

## Project Location

`D:\WorkSpace\Claude\Module_01`

## Quick Start

```bash
# CLI mode (interactive, asks at each decision point)
C:/Users/scott/.conda/envs/module01/python.exe run.py <datasheet.pdf> [output_dir]

# Web mode (browser UI, auto-resolves ambiguities)
C:/Users/scott/.conda/envs/module01/python.exe -m web.server
# → http://127.0.0.1:5000/
```

Default output: `D:\Data\01_Lib2023\02_sch\` (CLI only, requires confirmation).

## The 5-Stage Pipeline

| Stage | Module | What it does |
|-------|--------|--------------|
| 1. Page Location | `pdf_locator.py` | Find pin table + pin diagram pages (bookmark → TOC → keyword → ratio fallback) |
| 2. Extraction | `mineru.py` + `vision.py` | Text channel (MinerU API) extracts pin tables; Vision channel (DeepSeek) extracts pin diagram structure |
| 3. Merge | `merge.py` | Join by pin number; conflicts recorded, never guessed |
| 4. Review | `review.py` | Independent fresh-session verification to catch self-confirmation bias |
| 5. Generation | `symbol.py` + `capture.py` | Geometry calculation → TCL script → `tclsh.exe` → `.olb` + `.dsn` |

## Interactive Prompts (CLI Mode)

| Prompt | When | How to respond |
|--------|------|----------------|
| Target device | Multi-device datasheet (e.g., ADS1113/1114/1115) | Enter device model or select from list |
| Package code | Multiple packages available (e.g., RUG/DGS/DYN) | Enter package code |
| Conflict resolution | Text vs vision channel disagree | Choose which source to trust |
| Review differences | Independent review disagrees with extraction | Choose extraction or review value |
| Output directory | Before writing files | Confirm with `y` |

## Key Design Principles

- **No guessing**: Ambiguities stop and ask the user
- **Evidence-first**: Every pin has page number + source channel attached
- **Dual-channel**: Text for precise numbers/names; Vision for spatial structure (which side, which package)
- **Independent review**: Fresh model call without prior context to catch bias

## Common Issues

| Symptom | Cause / Fix |
|---------|-------------|
| Console garbled Chinese | Set `PYTHONIOENCODING=utf-8` before running |
| MinerU TLS error | Already has `curl` fallback; retry or check `.cache/mineru/` |
| `.olb`/`.dsn` locked | Close Capture first; `.lck` file means it's open |
| Pin order wrong in symbol | Left side: top→bottom = pin 1,2,3...; Right side: top→bottom = max,min-1,... (DBO Y-axis is inverted) |
| Vision returns empty | Auto-retries in `vision.py`; usually transient network issue |

## Pin Type Mapping (DBO values)

| Value | Type |
|-------|------|
| 0 | Input |
| 1 | Bidirectional |
| 2 | Output |
| 4 | Passive (default for all pins) |
| 7 | Power (VCC/GND) |

## Regression Testing

After modifying parsing logic in `app/`:

```bash
C:/Users/scott/.conda/envs/module01/python.exe tests/regress.py
```

Runs offline (no vision API calls, uses cached MinerU results). Covers: table parsing, figure page location, device name recognition. **Always run after changing `extract.py`, `pdf_locator.py`, or `pkgname.py`.**

## Environment Dependencies

| Dependency | Location |
|------------|----------|
| Python | conda env `module01` (`C:\Users\scott\.conda\envs\module01`) |
| Cadence tclsh | `C:\Cadence\SPB_17.2\tools\bin\tclsh.exe` |
| MinerU API key | `MinerU_API_KEY.md` in project root |
| DeepSeek API key | Env var `DEEPSEEK_API_KEY` or Windows registry fallback |

## Intermediate Artifacts

- `out/<device>_spec.json` — Pin table + evidence + conflicts (human-readable)
- `.cache/mineru/` — SHA256-keyed MinerU parse cache (avoids re-upload)
- `.cache/render/` — Rendered PNG cache

## Web Mode Differences

- No interactive prompts; auto-resolves OCR confusables (O/0, I/1/l) by trusting review
- Other conflicts listed in "待人工核对" for manual review
- Output in `web_out/<job_id>/` (never writes to `02_sch\`)
- API: `POST /api/jobs` → `GET /api/jobs/{id}` → `GET /api/jobs/{id}/download/{olb|dsn|spec|tcl}`
