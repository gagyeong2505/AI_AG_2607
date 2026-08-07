# content.json Schema

Input file consumed by `scripts/generate_xlsx.py`. Top level is a single object with one key, `sheets`, an array of sheet definitions. Each sheet becomes one worksheet tab in the output workbook, in array order.

## Top level

```json
{
  "sheets": [ <sheet>, ... ]
}
```

## `<sheet>` object

| Field | Type | Required | Default | Notes |
|---|---|---|---|---|
| `name` | string | yes | — | Sheet/tab name. Truncated to 31 characters (Excel limit). |
| `headers` | array of string | yes | — | Column headers, left to right. Becomes row 1. |
| `rows` | array of array | yes | — | Data rows. Each inner array's values line up positionally with `headers`. Use `null`/omit trailing values for blank cells. A row with every cell empty is skipped when the ID column is numbered. |
| `add_id_column` | boolean | no | `true` (or the `--no-id` CLI flag) | Whether to insert an `ID` column as column A for this sheet. |
| `id_header` | string | no | `"ID"` | Header text for the inserted ID column. |
| `header_color` | string (hex, no `#`) | no | `"2E74B5"` | Fill color for the header row. |
| `title_columns` | array of string | no | `[]` | Header names to force-treat as "title" (bold) columns, in addition to auto-detection by keyword (제목/title/headline/heading). |
| `quote_columns` | array of string | no | `[]` | Header names to force-treat as "quote" (italic) columns, in addition to auto-detection by keyword (인용구/quote/quotation/citation) and auto-detection of quoted string values. |

## Auto-detected behavior (no schema field needed)

- **Body font size**: every data cell is set to 10pt.
- **Title columns**: any header containing 제목/title/headline/heading (case-insensitive) is bolded automatically.
- **Quote columns**: any header containing 인용구/quote/quotation/citation, or any cell value wrapped in quote characters (`"`, `'`, `"…"`, `「…」`), is italicized automatically.
- **Charts**: if 2+ numeric values exist in a column whose header matches a comparison/growth keyword (성장률, 증감률, 매출, growth, rate, revenue, etc.), a chart is generated to the right of the table. `LineChart` is used when any header mentions a time unit (연도/월/날짜/year/month/date); otherwise `BarChart`. The chart's category axis uses column B (the first column after ID).

## Example

See `templates/example_content.json` for a filled sample with a title column, a quote column, and a chart-triggering numeric column.

## Minimal example

```json
{
  "sheets": [
    {
      "name": "직원 목록",
      "headers": ["이름", "부서", "직급", "입사일"],
      "rows": [
        ["홍길동", "개발팀", "대리", "2022-03-02"],
        ["김철수", "기획팀", "과장", "2019-07-15"]
      ]
    }
  ]
}
```
