---
name: document-skills
description: >
  Use this skill whenever the user wants to create, read, edit, convert, or manipulate
  any office or document file — regardless of format. Covers all four major document types:
  Word (.docx), PDF (.pdf), PowerPoint (.pptx), and Excel (.xlsx / .csv / .tsv).
  Trigger on any mention of: "Word doc", "report", "memo", "letter", "template",
  "PDF", "merge PDFs", "fill form", "slide deck", "presentation", "pitch deck",
  "spreadsheet", "Excel", "financial model", "data table", or any file extension
  (.docx, .pdf, .pptx, .xlsx, .xlsm, .csv, .tsv). Also trigger when the user uploads
  one of these file types and asks to do anything with it — read, extract, edit, convert,
  or produce a new file. When in doubt about which document format is involved, use this skill
  to route correctly. Do NOT use for Google Docs/Sheets/Slides API integrations, HTML reports
  served in a browser, or database/pipeline tasks that don't produce a document file.
---

# Document Skills — Router

This skill bundles four specialized sub-skills. Your first job is to identify the
document format involved, then read and follow the corresponding sub-skill.

## Format Detection

| Trigger / File Type | Sub-skill to read |
|---------------------|-------------------|
| `.docx`, Word doc, report, memo, letter, template | [docx/GUIDE.md](docx/GUIDE.md) |
| `.pdf`, PDF form, merge/split/watermark PDF | [pdf/GUIDE.md](pdf/GUIDE.md) |
| `.pptx`, slide deck, presentation, pitch deck | [pptx/GUIDE.md](pptx/GUIDE.md) |
| `.xlsx`, `.xlsm`, `.csv`, `.tsv`, spreadsheet, financial model | [xlsx/GUIDE.md](xlsx/GUIDE.md) |

## Routing Instructions

1. **Identify the format** from the user's request or uploaded file extension.
2. **Read the matching sub-skill** listed in the table above.
3. **Follow that sub-skill's instructions** exactly — each has its own tools, libraries, and critical rules.
4. If the task involves **multiple formats** (e.g. "convert this DOCX to PDF"), read both relevant sub-skills before starting.

## Cross-Format Tasks

Common multi-format workflows:

| Task | Sub-skills to read |
|------|--------------------|
| Convert DOCX → PDF | docx + pdf |
| Export XLSX data to PDF report | xlsx + pdf |
| Insert charts from XLSX into PPTX | xlsx + pptx |
| Convert PPTX slides to PDF | pptx + pdf |

## Notes

- Each sub-skill is self-contained and authoritative for its format.
- Always prefer the sub-skill's guidance over general knowledge — it encodes environment-specific constraints (available libraries, rendering quirks, output paths).
- Output files go to `/mnt/user-data/outputs/` unless the user specifies otherwise.