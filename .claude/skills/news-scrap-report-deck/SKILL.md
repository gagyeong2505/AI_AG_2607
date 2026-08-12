---
name: news-scrap-report-deck
description: Given a topic or a list of article URLs, scraps news articles with Playwright MCP (full-page screenshots saved to outputs/playwright/), analyzes their content, and produces both a Word (.docx) report via python-docx and a PowerPoint (.pptx) presentation via python-pptx using the bundled scripts. Use when the user asks to scrap/screenshot articles and turn them into a report and slide deck, e.g. "playwright로 기사 스크랩해서 보고서랑 발표자료 만들어줘", "이 기사들 스크랩하고 분석해서 ppt로 만들어줘", "뉴스 스크랩 후 워드 보고서와 파워포인트 둘 다 만들어줘".
---

# News Scrap Report Deck

## Overview

Given a topic or a set of article URLs, use Playwright MCP to open each article, save a full-page screenshot, and extract its text. Analyze the scraped articles, then render the findings as both a styled `.docx` report (`scripts/generate_docx.py`) and a `.pptx` slide deck (`scripts/generate_pptx.py`) — one shared JSON content file drives both. Requires `python-docx` and `python-pptx` (`pip install python-docx python-pptx`).

## Workflow

### Step 1: Confirm scope

Ask only for what's missing:
- Topic/keyword to search for, OR a specific list of article URLs to scrap directly
- Number of articles to include (default: 3–5)
- Report/deck title
- Output file name base (default: topic slug)
- Main accent color, hex (default `2E74B5`)

### Step 2: Find and scrap articles with Playwright MCP

If given a topic instead of URLs, articles must be sourced from Naver News only (`https://news.naver.com`). Use `browser_navigate` to Naver News search/section pages, then `browser_find`/`browser_snapshot` to pick the target articles. Do not use other news sources (e.g. Google News) for topic-based search.

For each article:
1. `browser_navigate` to the article URL.
2. `browser_take_screenshot` with `fullPage: true`, saving to `outputs/playwright/<topic-slug>/<NN>-<article-slug>.png` (create the folder if it doesn't exist). Every Playwright MCP output must live under `outputs/playwright/` per project convention — never save scrap screenshots elsewhere.
3. `browser_snapshot` (or `browser_find`) to read the headline, source/언론사, publish date if shown, and body text needed for analysis.

### Step 3: Analyze

For each article, write a 1–2 sentence summary and 2–4 key points. Across all articles, note shared themes or takeaways for the report/deck intro and conclusion. Never fabricate facts — only use what was actually scraped.

### Step 4: Write the content JSON

Build a JSON file matching the schema in `references/content_schema.md` (see `templates/example_content.json` for a filled sample). Use the actual screenshot paths saved in Step 2. Save the JSON to a scratch path, e.g. `<topic-slug>_content.json`.

### Step 5: Generate the report and the deck

```
python "<skill-dir>/scripts/generate_docx.py" --input <topic-slug>_content.json --output outputs/reports/<topic-slug>_report.docx --color <hex>
python "<skill-dir>/scripts/generate_pptx.py" --input <topic-slug>_content.json --output outputs/reports/<topic-slug>_deck.pptx --color <hex>
```

`--color` is optional on both (defaults to `2E74B5`). Create the `outputs/reports/` folder if it doesn't exist yet.

### Step 6: Clean up and deliver

Delete the intermediate content JSON file. Report the saved `.docx` and `.pptx` paths plus the screenshot folder to the user, and summarize the article list inline in chat.

## Notes

- Report styling: 40pt title / 28pt heading / 20pt body, Malgun Gothic, configurable accent color, each article's screenshot embedded at page width.
- Deck styling: title slide → overview slide → one slide per article (screenshot on the left, headline/summary/key points on the right) → conclusion slide, Malgun Gothic throughout.
- A `screenshot` path that doesn't exist on disk is skipped with a printed warning rather than failing the whole generation — double-check paths in the content JSON if an image is missing from the output.
- If the target `.docx`/`.pptx` is already open in Word/PowerPoint, saving fails with a permission error — ask the user to close it first.
