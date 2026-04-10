# Artifact Archive

Centralized archive of documents, presentations, images, and reports created across personal projects.

See [INDEX.md](INDEX.md) for the full catalog.

## Structure

```
projects/
  <project-name>/
    INDEX.md          # Per-project artifact index
    docs/             # PDF, DOCX
    presentations/    # PPTX
    spreadsheets/     # XLSX
    images/           # PNG, JPG, SVG, GIF
    reports/          # HTML, MD (articles/reports)
    other/            # Everything else
```

## Usage

Artifacts are archived automatically by Claude Code via the `archive-artifact.sh` script.

```bash
bash ~/.claude/scripts/archive-artifact.sh "<source_path>" "<project_name>" "<description>"
```
