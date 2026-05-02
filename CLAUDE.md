# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**M5 Literacy Suite** is a collection of standalone, self-contained HTML files — each an interactive phonics and literacy tool designed for primary-school classrooms. There is no build system, no package manager, and no backend. Every file runs directly in a browser.

The canonical entry point is `index.html`, which links to the active tools. Files with a numeric suffix (e.g. `m5_blending_board1.html`, `soundmatch2.html`) are archived/prior versions kept for reference — the unnumbered files are the current ones.

## Active Tools

| File | Tool | Purpose |
|---|---|---|
| `speed_sounds.html` | Speed Sound Deck | Phonics flashcard system (Sets 1–3) with mastery tracking, practice mode, and teacher dashboard |
| `m5_blending_board.html` | M5 Blending Board | Drag-and-drop letter-tile blending activity with session recording |
| `M5 Sound Matching Board.html` | Sound Matching Board | Teacher-controlled grapheme matching tiles with session data |
| `soundmatch.html` | Sound Match (Initial Sound) | Student drag picture-to-letter game with persistent profiles and dashboard |

## Architecture

### Single-file design
Each tool is fully self-contained: CSS, HTML, and JS all live in one `.html` file with no imports from other project files. Adding a shared utility means duplicating it into each file.

### Teacher/Student split layout
`m5_blending_board.html` and `M5 Sound Matching Board.html` use a two-panel layout:
- **Left panel** (20% width): teacher controls (student/adult name, word input, support scales, action buttons)
- **Right panel** (80% width): student-facing display (letter tiles, drop zone)
- A toggle button collapses/expands the teacher panel

`soundmatch.html` uses a similar collapsible `<aside class="teacher">` / `<main class="student">` pattern with a CSS-variable-driven resizable dragbar.

### Data persistence
All tools persist data exclusively in **`localStorage`** — no server, no database. Key names:
- `speedSoundsMastery` — mastery status per sound (speed_sounds.html)
- `speedSoundsContrast` — high-contrast toggle state
- `speedSoundsStudentName` — last-used student name
- `m5BlendingBoardProgressData` — blending board session records
- `soundmatch-profiles` — per-student profiles including session history and per-sound mastery (soundmatch.html)
- Sound Matching Board uses the same `localStorage` pattern with key `m5SoundMatchingBoardProgressData`

Admin data download is protected by a hardcoded password (`"1234"`) via `prompt()`.

### Speech synthesis
- `speed_sounds.html`: Uses the Web Speech API (`speechSynthesis`) with a `phoneticMap` object mapping graphemes to phonetic strings optimised for TTS (e.g. `'sh': 'shh. shh. shh.'`). No external TTS.
- `m5_blending_board.html`: Web Speech API with voice priority ranking (prefers Google/Microsoft neural voices).
- `soundmatch.html`: Calls the **OpenAI TTS API** (`tts-1`, voice `alloy`) with a fallback to `speechSynthesis`. The API key is hardcoded in the source at line 826.

### Phonics data (speed_sounds.html)
Sounds are grouped into three sets following the Read Write Inc. progression:
- **Set 1**: 31 sounds — single letters + digraphs (`sh`, `th`, `ch`, `qu`, `ng`, `nk`)
- **Set 2**: 12 vowel teams (`ay`, `ee`, `igh`, `ow`, `oo`, `ar`, `or`, `air`, `ir`, `ou`, `oy`)
- **Set 3**: 19 alternative spellings and split digraphs (`a-e`, `i-e`, `o-e`, `u-e`, `tion`, etc.)

`parseWordIntoGraphemes()` in `m5_blending_board.html` splits words into graphemes using a priority-ordered list (longest match first) — the order of the `graphemeList` array is significant.

### Drag-and-drop (Blending Board and Sound Matching Board)
Both tiles and drop zones support **both mouse drag-and-drop** (HTML5 `draggable`) and **touch** (custom `touchstart`/`touchmove`/`touchend` handlers that clone tiles and track position manually). Touch handling on the drop zone uses `ontouchstart` with `e.preventDefault()` to suppress scroll. The `viewType` string (`'BB'` or `'SMB'`) is threaded through drag events to prevent cross-component drops.

### Versioning convention
New iterations are developed by uploading a new file as the canonical name; the previous version is renamed with an incrementing suffix (`1`, `2`, etc.) before upload. The numbered files are not linked from `index.html`.

## UI Conventions

- **Font stack**: `'Sassoon Primary', 'Comic Sans MS', Arial, sans-serif` — primary-school-friendly, single-storey `a`. `soundmatch.html` also includes `Andika` and `"Chalkboard SE"` ahead of Comic Sans.
- **Colour palette** (consistent across all tools):
  - Blue `#007aff` — primary action / brand
  - Green `#34c759` — correct / success
  - Red `#ff3b30` — incorrect / destructive
  - Purple `#af52de` — reset / secondary
  - Orange `#ff9500` — navigation / report
- **Watermark**: Every tool has a fixed bottom-right watermark `<div class="watermark">` with `pointer-events: none`.
- **No user-scalable zoom**: All tools set `maximum-scale=1.0, user-scalable=no` in the viewport meta to prevent accidental pinch-zoom during classroom use.
- **Print styles**: `m5_blending_board.html` supports two print modes toggled via body classes: `teacher-print-view` (full data table) and `student-print-view` (child-friendly word cards with star/pencil icons).

## External Dependencies

`speed_sounds.html` is the only tool with CDN dependencies:
- Google Fonts: Poppins (`fonts.googleapis.com`)
- jsPDF 2.5.1 (`cdnjs.cloudflare.com`) — for PDF report export
- html2canvas 1.4.1 (`cdnjs.cloudflare.com`) — used with jsPDF

All other tools are fully offline-capable after initial load (except `soundmatch.html` TTS which needs the OpenAI API).
