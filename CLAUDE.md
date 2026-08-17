# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Lecture site for NCU IS2026 / 情報システム論 (Information Systems). Plain static HTML/JS — no build system, no package manager, no tests, no framework. All site content is in Japanese.

Deployment is GitHub Pages serving the root of the `main` branch. Pushing to `main` publishes immediately to https://kklab.mobi/kklab-is2026/ — treat pushes to `main` as production deploys.

## Local preview

No build step. Serve locally with:

```bash
python3 -m http.server 8000
```

Use `http://localhost:8000` specifically: the attendance/verify pages force-redirect `http:` → `https:` for any hostname other than `localhost`, and the confirmation-code logic requires `crypto.subtle`, which is only available in secure contexts (HTTPS or localhost).

## Architecture: tamper-resistant attendance flow

The core mechanism spans three files and both sides must stay in sync:

- `attendance/attendance01.html` (…〜`attendance30.html`) — student-facing quiz. On an all-correct submission it fetches server time (never the device clock), computes a confirmation code, and shows a result screen the student screenshots and submits via Teams.
- `verify.html` — instructor-facing page. The instructor re-enters the screenshot's student ID / name / time / code; it recomputes the code and reports match or mismatch (a mismatch means the screenshot was likely altered).
- `code.js` — shared logic loaded by both pages: `computeCode(pageId, studentId, studentName, timeText)` returns `XXXX-XXXX` derived from a SHA-256 hash of the pipe-joined, NFKC-normalized fields plus an obfuscated key (`codeKey()`); `fetchServerTime()` gets JST time from timeapi.io with worldtimeapi.org as fallback, returning `null` on failure (deliberately no device-clock fallback).

Consequences for changes:

- Any change to `computeCode`, `fieldNormalize`, `codeKey`, or the time-string format (`formatJst`) changes the codes and invalidates verification of already-submitted screenshots. Don't touch these casually.
- The lesson pages `tutorials/htmlcss30/day01–30.html` embed the same quiz + confirmation-code mechanism with `PAGE_ID` `"htmlcss30-day01"`…`"htmlcss30-day30"`, loading `code.js` via `../../code.js`; `verify.html` lists them in a second `<optgroup>`.
- The `pageId` passed to `computeCode` must match between a quiz page (`PAGE_ID` constant, e.g. `"attendance01"`) and the corresponding `<option>` value in `verify.html`. A new attendance page needs its own `PAGE_ID` and corresponding verify support, plus a link from `index.html`. `PAGE_ID` is independent of the file path — moving/renaming a page file does not change its codes, but changing `PAGE_ID` does.
- New attendance pages should keep the HTTPS-redirect snippet in `<head>` and load `code.js` (from `attendance/`, via `../code.js`).

## Layout

- Attendance pages live in `attendance/` (`attendance/index.html` is the 問題集 index linked from the top page), tutorial/self-study pages (e.g. `htmlcss30.html`) in `tutorials/`. `index.html`, `verify.html`, and the shared `code.js` stay at the repo root. Pages in subdirectories reference root files with `../` relative paths.

## Conventions

- When making a user-visible change, add a dated entry (Japanese, newest first) to the 更新履歴 section of `README.md` — that file is the change log.
- Pages are self-contained: styles are inline `<style>` blocks per page, sharing the same visual language (same font stack, `#f5f6f8` background, blue `#2563eb` accents, rounded white cards).
