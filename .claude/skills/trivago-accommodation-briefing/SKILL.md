---
name: trivago-accommodation-briefing
description: Compares accommodation prices via the trivago MCP tools (trivago-accommodation-search, trivago-accommodation-radius-search) and produces a lowest-price ranking with star rating, guest review score, and review highlights. Use when the user asks to find/compare hotels or stays, wants the cheapest option for a destination or date range, or asks for a "숙소 비교", "최저가 숙소", "호텔 브리핑", or similar accommodation briefing.
---

# Trivago Accommodation Briefing

## Overview

Given a destination (or a specific point of interest) and travel dates, search accommodations with the trivago MCP tools, rank results from lowest to highest price, and produce a briefing that pairs each price with its star rating and guest review score/highlights. Save the finished briefing as a Markdown file under the project's `travel/` folder.

## Workflow

### Step 1: Collect trip requirements

Before calling any tool, confirm with the user (ask only for what's missing):
- Destination — a place name/city (for `trivago-accommodation-search`) OR a specific landmark/address with known coordinates (for `trivago-accommodation-radius-search`)
- Arrival and departure dates (`YYYY-MM-DD`, arrival must be today or later, departure after arrival)
- Number of adults (required), children + ages (optional)
- Number of rooms (optional, must be ≤ adults)
- Any hard requirements: minimum star rating, minimum guest review score, amenities (breakfast, free cancellation, pool, etc.), budget ceiling, currency/country/language if not the default (US/USD/EN_US)

### Step 2: Choose and call the search tool

- Use `mcp__trivago__trivago-accommodation-search` when the user gives a destination name or point of interest (`query`).
- Use `mcp__trivago__trivago-accommodation-radius-search` when the user gives specific coordinates or a precise landmark/address you've geocoded to `latitude`/`longitude`.

Map the collected requirements directly onto the tool's parameters (`arrival`, `departure`, `adults`, `children`, `children_ages`, `rooms`, `filters`, `hotel_rating`, `review_rating`, `currency`, `country`, `language`). Only set `hotel_rating` / `review_rating` / `filters` flags the user actually asked for — leaving them all `false` returns the unfiltered result set, which is usually what you want for a full price comparison.

### Step 3: Rank by lowest price

From the results, sort ascending by total/nightly price (whichever the tool returns) and keep every result unless the user asked to cap the list — for a long result set, default to the top 10 cheapest plus any result that stands out (e.g., notably higher review score for only slightly more money).

### Step 4: Compile the briefing

For each accommodation in the ranked list, capture:
- Name and price (with currency)
- Star rating (hotel class)
- Guest review score and the number of reviews backing it, if provided
- 1–2 review highlights or standout amenities, if the tool result includes them
- A short note if it's the cheapest, best-rated, or best value (highest review score per price) pick

Write the briefing using `templates/briefing_template.md` as the structure.

### Step 5: Save the briefing to `travel/`

Save the completed briefing as a Markdown file in the project's `travel/` folder (create the folder if it does not already exist). Name the file `travel/<destination-slug>_<arrival-date>_briefing.md`, e.g. `travel/tokyo_2026-09-10_briefing.md`. Slugify the destination (lowercase, spaces → hyphens, strip punctuation). After saving, tell the user the file path and summarize the top 1–3 picks inline in the chat.

## Notes

- If the user only wants a quick answer without a saved file, still answer inline, but still save the file to `travel/` so it stays available for later comparison — do not skip Step 5 unless the user explicitly says not to save it.
- If trivago returns no results for the given filters, relax filters one at a time (starting with amenity filters, then rating thresholds) and tell the user what was relaxed.
- Never fabricate prices, ratings, or reviews — only report what the tool call returns.
