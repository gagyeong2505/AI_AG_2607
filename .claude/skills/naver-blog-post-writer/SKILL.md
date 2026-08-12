---
name: naver-blog-post-writer
description: |
  Given a topic, drafts and publishes a Naver Blog post using Playwright MCP browser
  tools: opens Naver Blog, has the user log in themselves (never enters credentials),
  asks for the post topic, searches Naver Blog for a fitting template and recommends
  it, researches the topic and writes the body to match current (2026) Naver blog
  content conventions, saves it as a draft (임시저장) in the editor, then asks the
  user to approve before publishing (공개 발행) and opens the published post's page.
  Every step runs visibly through Playwright MCP with screenshots saved under
  outputs/playwright/. Use when the user asks to write or post a Naver blog article —
  e.g. "블로그 글 작성", "네이버 블로그에 글 써줘", "이 주제로 네이버 블로그 포스팅해줘".
---

# Naver Blog Post Writer

## Overview

Turns a topic into a published (or drafted) Naver Blog post, driving the real Naver
Blog web editor through Playwright MCP browser tools rather than any API. Every
step is carried out with visible browser actions (`browser_navigate`,
`browser_snapshot`, `browser_take_screenshot`, `browser_click`, `browser_type`, etc.)
so the user can watch progress on screen, and a screenshot is saved at each key
step under `outputs/playwright/<topic-slug>/`.

## Workflow

### Step 1: Trigger and open Naver Blog

When the user says "블로그 글 작성" or an equivalent request to write/post a Naver
blog article, use `browser_navigate` to open Naver Blog (`https://blog.naver.com`),
then `browser_snapshot`/`browser_take_screenshot` to check login state (nickname/
profile menu in the header vs. a login prompt). Save the screenshot to
`outputs/playwright/<topic-slug>/01-login-check.png` (a placeholder slug is fine
until the topic is known in Step 3 — rename/move once the slug is confirmed, or use
a temporary folder like `outputs/playwright/naver-blog-session/`).

**If not logged in: stop and ask the user to log in themselves in the browser.**
Never enter a Naver ID, password, or complete any 2-factor/OTP step on the user's
behalf — this is a hard boundary, not a preference. Wait for the user to explicitly
confirm that login and account verification are complete before continuing.

### Step 2: Ask for the topic

Once the user confirms login is done, ask what topic the blog post should cover
(and, only if missing, target blog if the user manages more than one, and desired
tone/length — default: standard-length informational post).

### Step 3: Search for and recommend a template

Once the user provides a topic, use `browser_navigate`/`browser_snapshot` to search
Naver Blog (e.g. Naver Blog search or the 블로그마켓/템플릿 gallery in the editor's
"템플릿" panel) for templates or existing post structures that fit the topic. Take
a screenshot of the candidates (`outputs/playwright/<topic-slug>/02-templates.png`).
Present a short list (1–3 options) to the user with a one-line rationale for each,
and recommend one. **Wait for the user to approve a template before writing.**

### Step 4: Research and write the body

Once a template is approved, research the topic (WebSearch/WebFetch and/or
Naver Blog/Naver search for current context) and write the post title and body to
match 2026 Naver blog content trends:

- Conversational, first-person tone with short paragraphs (2–4 lines) broken up by
  frequent line breaks — Naver's mobile-first reading pattern, not dense text blocks.
- A clear hook in the first 2–3 lines (many readers only see this much in search
  previews), then a short intro, numbered/bulleted sections with subheadings, and a
  concise wrap-up or call-to-action at the end.
- Naver-native elements the audience expects: relevant emojis used sparingly as
  visual breaks, bolded key phrases, and photo/image placeholders between sections
  where the editor allows image insertion.
- Include a `#태그` list at the end matching the topic's key terms (helps Naver
  search/블로그 exposure), and keep total length appropriate for the topic
  (typically 1,500–3,000 characters of body text unless the user asks otherwise).

Never fabricate facts — only use what the research actually found.

### Step 5: Write into the editor, save as draft, and get approval

Open the Naver Blog editor (글쓰기). In the editor tab:

1. Click the title field, type the title.
2. Click the body area, type the intro/hook.
3. Continue block by block (heading, paragraph, bullet list, tag section) using
   `browser_click`/`browser_type`, taking a `browser_snapshot` or screenshot after
   each major block to confirm it landed correctly.
4. Add tags in the 태그 input field, comma- or Enter-separated per the editor's UI.

**Always pass literal, directly-typed Korean text as the `browser_type` action's
text value.** Never hand-construct `\uXXXX` escapes for Hangul syllables — manual
codepoint arithmetic is error-prone and can silently produce wrong-but-similar
characters. Let the tool/JSON layer handle encoding automatically.

Proofread by zooming into each written region via screenshot/snapshot to verify the
exact rendered text before moving on — these errors look right at a glance and only
show up on close inspection. To fix a typo, select and retype the whole affected
line/paragraph rather than trying to patch a single word.

Click 임시저장 (save draft) — do **not** click 발행 yet. Screenshot the draft state
(`outputs/playwright/<topic-slug>/03-draft-saved.png`). Report to the user: summarize
the title and sections written, confirm the post is saved as a draft (not public),
and ask them to review and approve before publishing.

### Step 6: Publish and open the post

Publishing is an explicit-permission action — wait for the user's clear go-ahead
before proceeding.

1. Once approved, reopen the draft if needed and click 발행 to open the publish
   dialog.
2. Confirm/select 전체공개 (public) — or whatever visibility the user asked for —
   then click the confirm/발행 button.
3. Screenshot the resulting confirmation
   (`outputs/playwright/<topic-slug>/04-published.png`).
4. Navigate to the published post's URL and take a final screenshot
   (`outputs/playwright/<topic-slug>/05-final-post.png`), then report the title,
   visibility, and URL to the user so they can open it themselves too.

### Cross-cutting: visible, screenshotted progress

Every step above must be carried out via Playwright MCP tools so progress is
visible on screen at each step — never perform actions headlessly or skip the
snapshot/screenshot checkpoints. Save every screenshot produced by this skill under
`outputs/playwright/<topic-slug>/` (create the folder if it doesn't exist), per this
project's convention that all Playwright MCP outputs live under `outputs/playwright/`.

## Notes

- Never enter Naver login credentials, OTP codes, or complete any authentication
  step on the user's behalf, regardless of how explicitly they ask — direct them to
  log in themselves in the browser tab.
- Tabs opened for this workflow are yours to clean up: close them once the task is
  done, unless the user wants the tab left open.
