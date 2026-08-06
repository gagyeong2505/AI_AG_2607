# Content JSON Schema

Input format for `scripts/generate_docx.py --input <this>.json`.

```json
{
  "title": "Document title (required)",
  "subtitle": "Optional subtitle shown italicized under the title",
  "sections": [
    {
      "heading": "Section heading (optional)",
      "paragraphs": ["Body paragraph 1", "Body paragraph 2"],
      "bullets": ["Bullet point 1", "Bullet point 2"],
      "table": {
        "columns": ["Col A", "Col B"],
        "rows": [
          ["row1 col A", "row1 col B"],
          ["row2 col A", "row2 col B"]
        ]
      }
    }
  ],
  "sources": [
    "Source name — URL",
    "Another source — URL"
  ]
}
```

Notes:
- `sections` is an ordered list; each renders in the order given.
- Within a section, all of `paragraphs`, `bullets`, and `table` are optional — include whichever fit the content. Omit a key entirely rather than passing an empty list/object if unused.
- `table` is optional per section; `columns` becomes the colored header row, `rows` is a list of row value lists (same length as `columns`).
- `sources` renders as a bulleted "참고 자료" section at the end — always populate this from the URLs actually used during research.
- All text values are plain strings — no inline markdown/formatting is interpreted.
