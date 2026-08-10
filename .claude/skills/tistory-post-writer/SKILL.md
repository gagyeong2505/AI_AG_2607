---
name: tistory-post-writer
description: |
  Given a topic, researches it and writes it up as a Tistory blog post using the
  claude-in-chrome browser extension: opens Tistory, has the user log in themselves
  (never enters credentials), researches the topic, drafts a title/body/tags in the
  post editor, saves it as a draft, then asks the user to approve and pick a
  visibility (public / protected / private) before publishing. Use when the user asks
  to write or post a blog article to their Tistory — e.g. "티스토리에 글 써줘",
  "이 주제로 티스토리 포스팅해줘", "블로그에 올려줘", "조사해서 블로그 글로 작성".
---

# Tistory Post Writer

## Overview

Turns a topic into a published (or drafted) Tistory post, driving the real Tistory
web editor through the claude-in-chrome browser extension rather than any API.

## Workflow

### Step 1: Confirm scope

Ask only for what's missing:
- Topic
- Target blog, if the user has more than one Tistory blog
- Desired tone/length (default: standard-length informational blog post)

### Step 2: Load browser tools

Invoke the `claude-in-chrome` skill, then load the core tool set via ToolSearch
(`select:mcp__claude-in-chrome__tabs_context_mcp,...`) per that skill's instructions
before calling any `mcp__claude-in-chrome__*` tool.

### Step 3: Research

Use WebSearch/WebFetch and/or claude-in-chrome (e.g. Google News search) to gather
current, accurate information on the topic. Keep track of sources — they inform the
post but Tistory posts don't need a formal citation list like a report does.

### Step 4: Open Tistory and confirm login

Navigate to the user's Tistory (or `https://www.tistory.com`) and check whether the
user is logged in (avatar/nickname in the header vs. a login prompt).

**If not logged in: stop and ask the user to log in themselves in the browser.**
Never enter a password, Kakao account credentials, or complete any login step —
this is a hard boundary, not a preference. Wait for the user to confirm login is
done before continuing.

### Step 5: Open the post editor and write

Click 글쓰기 (opens `.../manage/newpost` in a new tab). Then, in that tab:

1. Click the title field, type the title.
2. Click the body area, type the intro paragraph.
3. For each subsequent block: press `Return Return` (via the `computer` `key`
   action) for a blank line before a heading, then `Return` before a paragraph
   under that heading. Bullet lists can be typed as literal `"- "`-prefixed lines
   joined by `\n` inside a single `type` call.
4. Add tags in the `#태그입력` field at the bottom (comma-separated), then press
   `Return`.
5. Prefer `browser_batch` to combine clicks/types/screenshots into one round trip.

**Critical: always pass literal, directly-typed Korean text as the `type` action's
`text` value.** Never hand-construct `\uXXXX` escapes for Hangul syllables — manual
codepoint arithmetic is extremely error-prone and silently produces wrong-but-similar
characters (e.g. 아셴→아션, 갖췄을→갖춤을, 뮤직→럮미직, 도넛→도넫). If a tool layer
needs JSON escaping, let the JSON encoder do it automatically.

### Step 6: Proofread

Zoom into each written region to verify the exact rendered text — these errors look
right at a glance and only show up on close inspection.

To fix a typo, **triple-click the whole paragraph/line and retype it in full** —
do not double-click. Double-clicking Korean text with no internal spaces selects a
whole contiguous run (word + trailing space), so retyping only the fixed word can
silently swallow adjacent text or collapse spacing.

### Step 7: Save as a draft

Click 임시저장 (save draft). Do **not** click 완료/발행 yet.

Report to the user: summarize the sections written, confirm the post is saved as a
draft (not public), and ask them to review and approve before publishing.

### Step 8: Publish (only after explicit approval)

Publishing (public or protected visibility) is an explicit-permission action —
always get the user's go-ahead for the *specific* visibility level (공개 /
공개(보호) / 비공개) before clicking the final confirm button. Saving as a private
draft the user explicitly requested does not need further confirmation.

1. Click 완료 to open the 발행 dialog.
2. If the confirm button appears cut off at the right edge of the screenshot, the
   browser window is narrower than the dialog layout: call `resize_window` on the
   tab (e.g. 1280x900), then re-screenshot.
3. Select the requested visibility radio button, then click the confirm button
   (its label changes with the selection, e.g. "공개 발행" / "비공개 저장").
4. Confirm the final result to the user from the 글 관리 (post list) page — title,
   visibility, timestamp, URL.

## Notes

- Never enter Tistory or Kakao login credentials on the user's behalf, regardless of
  how explicitly they ask — direct them to log in themselves in the browser tab.
- Tabs opened for this workflow are yours to clean up: close them once the task is
  done, unless the user wants the tab left open.
