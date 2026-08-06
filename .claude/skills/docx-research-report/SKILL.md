---
name: docx-research-report
description: Given a topic, researches it and produces a formatted Word (.docx) report with a 40pt title, 20pt body text, Malgun Gothic font, and a configurable main accent color, using the bundled python-docx script. Use when the user asks to research a topic and turn it into a Word document/보고서, says "이 주제로 자료조사 후 워드 문서로 만들어줘", "docx 보고서로 정리해줘", or asks to generate a report file from a subject.
---

# Docx Research Report

## Overview

Take a topic from the user, research it, and render the findings as a styled `.docx` file via `scripts/generate_docx.py` (python-docx under the hood). Requires `python-docx` installed (`pip install python-docx`).

## Workflow

### Step 1: Confirm scope

Ask only for what's missing:
- Topic
- Output file name/location (default: `reports/<topic-slug>.docx` in the project root)
- Main accent color, hex (default `2E74B5`)

### Step 2: Research

Use WebSearch/WebFetch to gather current, accurate information on the topic. Group findings into logical sections (e.g., overview, key facts, comparison data). Keep track of the URLs used — they go in the report's source list. Never fabricate facts; if a claim can't be sourced, omit it.

### Step 3: Write the content JSON

Build a JSON file matching the schema in `references/content_schema.md` (see `templates/example_content.json` for a filled sample). Save it to a scratch path, e.g. `<topic-slug>_content.json`.

### Step 4: Generate the document

```
python "<skill-dir>/scripts/generate_docx.py" --input <topic-slug>_content.json --output reports/<topic-slug>.docx --color <hex>
```

`--color` is optional (defaults to `2E74B5`). Create the `reports/` folder if it doesn't exist yet.

### Step 5: Clean up and deliver

Delete the intermediate content JSON file. Report the saved `.docx` path to the user and summarize the report's sections inline in chat.

## Notes

- Title: 40pt bold, colored with the main accent color, centered.
- Section headings: 28pt bold, same accent color.
- Body text, bullets, table rows: 20pt, Malgun Gothic.
- Font is applied at the Word style level (Title / Heading 1 / Normal), not per-run, so the document's outline/navigation and table-of-contents features still work.
- If the target `.docx` is open in Word, saving fails with a permission error — ask the user to close it first.
