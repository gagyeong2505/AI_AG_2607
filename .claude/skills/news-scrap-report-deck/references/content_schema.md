# Content JSON Schema

Shared input format for both `scripts/generate_docx.py --input <this>.json` and `scripts/generate_pptx.py --input <this>.json`.

```json
{
  "title": "Report/deck title (required)",
  "subtitle": "Optional subtitle shown under the title",
  "intro": [
    "Intro paragraph 1 (overview of the topic and why these articles were picked)",
    "Intro paragraph 2 (optional)"
  ],
  "articles": [
    {
      "headline": "Article headline (required)",
      "source": "언론사명 (required)",
      "published": "2026-08-11 (optional)",
      "url": "https://... (required)",
      "screenshot": "outputs/playwright/<topic-slug>/01-headline-slug.png (required, path to the scraped screenshot)",
      "summary": "1-2 sentence summary of the article (required)",
      "key_points": ["Key point 1", "Key point 2"]
    }
  ],
  "conclusion": [
    "Closing paragraph or bullet 1",
    "Closing paragraph or bullet 2 (optional)"
  ],
  "sources": [
    "언론사명 — https://article-url",
    "Another source — URL"
  ]
}
```

Notes:
- `articles` is an ordered list; each renders as one report section and one deck slide, in the order given.
- `screenshot` must point to a real file on disk (the screenshot saved during Playwright scraping) — the generators skip embedding (with a warning) if the path doesn't resolve, but still render the rest of that article's content.
- `key_points` is optional per article; omit rather than passing an empty list if there's nothing beyond the summary.
- `conclusion` is optional — omit entirely if there isn't a synthesized takeaway yet.
- `sources` renders as a bulleted "참고 자료" section at the end of the report (and is not repeated in the deck) — populate it from the `url`/`source` of every scraped article.
- All text values are plain strings — no inline markdown/formatting is interpreted.
