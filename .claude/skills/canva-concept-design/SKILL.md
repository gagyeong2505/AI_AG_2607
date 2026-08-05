---
name: canva-concept-design
description: Given a topic, derives at least 3 distinct Canva design concepts (plus a matching brand template recommendation if one exists) and waits for user approval before generating anything, then confirms the desired deliverable type, creates a draft via the Canva MCP tools and sends a review link, and on final approval exports it as PNG and files it into a "canva" folder (with a subfolder named after the topic) in the user's Canva account. Use when the user asks to create a Canva design/poster/카드뉴스/인스타 포스트/프레젠테이션 from a topic, e.g. "캔바로 디자인 만들어줘", "이 주제로 카드뉴스 뽑아줘", "Canva로 포스터 만들어줘".
---

# Canva Concept Design

## Overview

Turns a topic into a finished, filed Canva asset using the Canva MCP tools (`mcp__canva__*`), with two mandatory human approval gates: concept approval and save approval. Nothing is generated or saved to the user's account without explicit sign-off at each gate.

## Workflow

### Step 1: Get the topic

Ask for a topic if the user hasn't already given one.

### Step 2: Derive ≥3 design concepts, and check for a matching template — STOP for approval

Come up with at least 3 distinct design concepts for the topic (a 4th is fine). Each concept needs: a name, color palette/mood, layout style, and typography feel.

Also call `mcp__canva__search-brand-templates` with `query` set to the topic. If it returns one or more templates that genuinely fit the topic, recommend the best 1-2 alongside the concepts (name + `create_url`) as a ready-made alternative. If nothing relevant comes back, skip this silently — don't mention the search.

Present all concepts (and any matching templates) together and wait for the user to pick one option.

**Do not call any other Canva MCP tool before an option is chosen.**

> If the user picks a template instead of a concept: skip Steps 3-5 and go to Step 6 using `mcp__canva__create-design-from-brand-template` with the chosen `brand_template_id`, in place of `generate-design` + `create-design-from-candidate`.

### Step 3: Confirm the deliverable type (skip if a template was chosen in Step 2)

Ask what output the user wants and map it to `generate-design`'s `design_type`:

| User asks for | design_type |
|---|---|
| 카드뉴스 / 인스타 포스트 | `instagram_post` |
| 포스터 | `poster` |
| 프레젠테이션 / 슬라이드 | `presentation` |
| 인포그래픽 | `infographic` |
| 기타 | see the full enum in `generate-design`'s schema |

For `presentation`, default to `length: "short"` (1-5 slides) unless the user asks for more — extra slides multiply generation latency.

### Step 4: Generate candidates (skip if a template was chosen in Step 2)

Call `mcp__canva__generate-design` with a `query` that folds in the topic + the approved concept's style description + `design_type`. This returns `job.id` and `generated_designs[]`, each with a `candidate_id` and a `url` (a real `canva.com/d/...` link).

### Step 5: Let the user pick a candidate (skip if a template was chosen in Step 2)

**Do not try to render or download `thumbnails[].url`** (`design.canva.ai/...`) — those redirect to a login-gated page and are not fetchable outside the user's authenticated Canva session. Instead, list each candidate's `url` field (`canva.com/d/...`) and ask the user to open them in their browser and pick one.

### Step 6: Create the draft and send the review link — STOP for approval

Call `mcp__canva__create-design-from-candidate` with the chosen `job_id` + `candidate_id` — or, if a template was chosen in Step 2, `mcp__canva__create-design-from-brand-template` with the chosen `brand_template_id`. Either way this returns a design ID and `urls.view_url` / `urls.edit_url`. Send the view URL to the user and ask them to approve saving it.

**Do not export or file anything before this approval.**

### Step 7: On save approval — export PNG and file it

1. `mcp__canva__get-export-formats` on the design ID to confirm `png` is supported before exporting.
2. `mcp__canva__export-design` with `format: {"type": "png"}` → returns a time-limited download URL.
3. Ensure a `canva` folder containing a subfolder named after the topic exists in the user's Canva account:
   - `mcp__canva__search-folders` (query `"canva"`, ownership `owned`) for a root-level `canva` folder; reuse its ID if found, otherwise `mcp__canva__create-folder` with `name: "canva"`, `parent_folder_id: "root"`.
   - Search for an existing topic-named subfolder inside it; reuse if found, otherwise `mcp__canva__create-folder` with `name: <the topic>`, `parent_folder_id: <the canva folder's id>`.
4. `mcp__canva__move-item-to-folder` to move the design into the topic subfolder.
5. Report back the PNG download link (mention it expires) and the folder link.

## Notes

- The two approval gates (Step 2, Step 6→7) are mandatory — never skip or merge them. If the user rejects a concept, template, or candidate, offer alternatives instead of re-generating speculatively.
- Only recommend a template when it's a genuine topical match — don't pad the options with loosely related templates just to have something to show.
- Reuse existing `canva` / topic folders across runs instead of creating duplicates with the same name.
- Folder and topic names are used verbatim (no slugifying) — Canva folder names accept Korean/Unicode.
- If `generate-design` errors with something like "Common queries will not be generated," ask the user for more specific detail and retry.
