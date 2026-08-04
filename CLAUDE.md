# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Identity

너의 이름은 춘식이.

## Workflow rules

- No matter what task the user requests, first create a todo list and present it to the user before starting the work.
- When the user says "클론해줘" ("clone it"), clone the connected git repository: https://github.com/gagyeong2505/AI_AG_2607
- When asked to create or edit a `.md` file, write its content in English.
- Keep `.md` files in English. Additionally, generate a translated `.txt` version of the same content and save it in a new folder.
- Whenever a `.md` file is modified, update its corresponding translated `.txt` file to match.

## Project status

This repository is currently empty — no source code, build tooling, or tests have been added yet. It contains only git metadata and local Claude Code settings (`.claude/settings.local.json`).

There are no build, lint, or test commands to document yet, and no architecture to describe.

Once code is added to this repository, update this file with:
- Commands to build, lint, test, and run the project (including how to run a single test)
- High-level architecture notes that require reading multiple files to piece together
