---
name: xlsx-research-report
description: Given a topic, researches it and produces a formatted Excel (.xlsx) report; given a specific existing file (script, data file, or spreadsheet), analyzes it first and then generates or updates the .xlsx accordingly. Every sheet gets an auto-numbered ID column, bold title cells, italic quote cells, 10pt body text, and auto-generated charts for growth/comparison numeric data, via the bundled openpyxl script. Use when the user asks to research a topic and turn it into an Excel file/보고서, says "이 주제로 자료조사 후 엑셀로 만들어줘", "xlsx 보고서로 정리해줘", mentions a specific file and asks to turn it into a spreadsheet, or asks to analyze a file and generate an Excel output from it.
---

# Xlsx Research Report

## Overview

Produce a styled `.xlsx` workbook via `scripts/generate_xlsx.py` (openpyxl under the hood), from either a research topic or an existing file the user points to. Requires `openpyxl` installed (`pip install openpyxl`).

## Workflow Decision Tree

- User gives a **topic** (no specific file referenced) → **Mode A: Topic Research**
- User references a **specific existing file** (a script like `create_xlsx.py`, a data file such as `.csv`/`.json`/`.xlsx`, or a document) and asks to generate/update a spreadsheet from it → **Mode B: File Analysis**

Both modes converge on the same output step (Step 3/4 below): building a `content.json` and running `scripts/generate_xlsx.py`.

## Mode A: Topic Research

### Step A1: Confirm scope

Ask only for what's missing:
- Topic
- Output file name/location (default: `reports/<topic-slug>.xlsx` in the project root)
- Sheet breakdown, if the topic naturally splits into multiple tables (e.g., "시장 동향" + "부서별 실적")

### Step A2: Research

Use WebSearch/WebFetch to gather current, accurate information on the topic. Identify which parts of the findings are:
- **Tabular facts** → become `headers`/`rows`
- **A section title or headline** → put in a column whose header contains 제목/title (auto-bolded)
- **A direct quote or cited statement** → put in a column whose header contains 인용구/quote, or leave the value wrapped in quotes (auto-italicized)
- **Growth rates, YoY/MoM changes, comparisons, revenue, scores** → keep as numeric columns with a matching header keyword (성장률, 매출, growth, rate, comparison, …) so a chart gets generated automatically

Never fabricate facts; if a claim can't be sourced, omit it. Keep track of the URLs used and mention them to the user when reporting the result.

### Step A3: Write the content JSON

Build a JSON file matching `references/content_schema.md` (see `templates/example_content.json` for a filled sample with a title column, a quote column, and a chart-triggering numeric column). Save it to a scratch path, e.g. `<topic-slug>_content.json`.

## Mode B: File Analysis

### Step B1: Read and understand the referenced file

- If it's a **Python script** (e.g. `create_xlsx.py`): read it to understand what data/columns it currently produces, and what the user wants changed or extended.
- If it's a **data file** (`.csv`, `.json`, an existing `.xlsx`): read/parse it to extract headers and rows.
- If it's an **existing `.xlsx`**: use `openpyxl.load_workbook` (read-only load is fine for inspection) to pull out sheet names, headers, and row data.

### Step B2: Decide the output shape

Map what was found onto the `content.json` schema (`references/content_schema.md`): one sheet per logical table, headers in the same order as the source, rows carried over as-is. Note any columns that look like titles or quotes so they can be listed in `title_columns`/`quote_columns`, and any numeric columns worth charting.

### Step B3: Write the content JSON

Same as Step A3 — save a `content.json` reflecting the analyzed file's structure (plus whatever the user asked to add/change).

## Step 3 (both modes): Generate the workbook

```
python "<skill-dir>/scripts/generate_xlsx.py" --input <slug>_content.json --output reports/<slug>.xlsx
```

Flags:
- `--no-id` — skip the auto-numbered ID column
- `--no-charts` — skip chart auto-generation

Create the `reports/` folder if it doesn't exist yet.

## Step 4: Clean up and deliver

Delete the intermediate content JSON file. Report the saved `.xlsx` path to the user, list the sheets created, and note which (if any) columns triggered a chart. In Mode A, also share the source URLs used for research.

## Notes

- Header row: bold white text on a colored fill (`2E74B5` by default, configurable per sheet via `header_color`), centered, thin border.
- Body cells: 10pt, centered; existing font name/color/underline is preserved when re-styling (uses `copy.copy`), only size/bold/italic are adjusted.
- ID column: inserted as column A, numbered only for rows that have at least one non-empty value elsewhere in the row.
- Title columns: bold. Quote columns: italic. Detected by header keyword (제목/title/headline/heading, 인용구/quote/quotation/citation) or explicit `title_columns`/`quote_columns` in the JSON; quoted string values are auto-italicized even without a matching header.
- Charts: only created when 2+ numeric values exist in a column whose header matches a growth/comparison keyword. `LineChart` for time-series headers (연도/월/날짜/year/month/date), `BarChart` otherwise. Positioned to the right of the table so it never overlaps the data. All matching numeric columns are combined into one chart per sheet — never duplicated per column.
- Every formatting/charting step is wrapped in its own try/except inside the script, so one failure (e.g. a malformed chart range) doesn't stop the rest of the workbook from being generated and saved.
- If the target `.xlsx` is open in Excel, saving fails with a permission error — ask the user to close it first.
- Excel sheet names are capped at 31 characters; longer `name` values are truncated automatically.
