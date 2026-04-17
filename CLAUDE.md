# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page Tic Tac Toe (`index.html`) with two game variants — Classic 3x3 and Ultimate (9 nested mini-boards). Vanilla HTML/CSS/JS, no build system, no dependencies, no tests. Visual language is Swiss/Helvetica: off-white canvas, near-black ink, single yellow accent (`#ffd500`).

## Running

Open `index.html` directly in a browser. There is no dev server, no `npm`, no build step. The execution sandbox does not have `xdg-open`, `python`, or `node` available, so Claude cannot launch a browser — give the user the `file://` URL and have them open it.

## Architecture

Everything lives in `index.html`. The `<script>` block is structured as four sections separated by `/* --------------- X --------------- */` comment banners:

- **Top-level state** — `variant` (`'classic' | 'ultimate'`), `mode` (`'pvp' | 'pvc'`, classic-only), `scores`, and a single `state` object whose shape depends on the active variant.
- **CLASSIC** — `renderClassic`, `playClassic`, `computerMoveClassic`, `bestMoveClassic`, `minimax`, `checkEndClassic`. Minimax is full-depth (9 cells) so it always plays optimally.
- **ULTIMATE** — `renderUltimate`, `playUltimate`. The Ultimate move rule: the cell *index* you play in determines which mini-board your opponent must play next. If that target board is already won/drawn, `state.activeBoard` becomes `null` and the opponent can pick any open board. There is no AI for Ultimate; the mode toggle is hidden when Ultimate is selected (`updateModeVisibility`).
- **SHARED** — `winLineOf` / `winnerOf` (used for both 3x3 boards and the Ultimate big board), the `render()` dispatcher, `updateStatus` (renders via `innerHTML` because the current player's letter is wrapped in a `<span class="mark">` for the yellow highlight), and `updateScoreDisplay`.

Render is full re-render on every state change — no diffing. `state.lastMove` drives the `pop` animation and (in Ultimate) the `last` highlight class.

Visual contract worth preserving when editing styles: X is rendered as a solid heavy Helvetica letter; O is rendered as the same letter glyph with `color: transparent` + `-webkit-text-stroke` for an outlined look. This is how the two players are distinguished without using a second accent color.

## Git workflow

This project is version-controlled at https://github.com/lifeinprogrezz/tic-tac-toe-swiss (public, `main`, `origin` already configured). The user wants each meaningful change committed locally and pushed so they always have a backup and can revert easily. Default to commit + push after each task; ask only when scope is unclear.

Sandbox quirks that affect git operations:

- `git` is at `/usr/bin/git` and **not on PATH** — invoke as `/usr/bin/git ...`. When running `gh` commands that shell out to git (e.g. `gh repo create --push`), prepend `PATH="/usr/bin:$PATH"`.
- Global git identity is **not set** and the harness forbids modifying git config. Pass identity per-commit via env vars:
  ```
  GIT_AUTHOR_NAME="lifeinprogrezz" GIT_AUTHOR_EMAIL="hello@lifeinprogrezz.com" \
  GIT_COMMITTER_NAME="lifeinprogrezz" GIT_COMMITTER_EMAIL="hello@lifeinprogrezz.com" \
  /usr/bin/git commit -m "..." -m "..."
  ```
- `cat`, `head`, `find`, `which`, `xdg-open` are unavailable — use multiple `-m` flags for multi-paragraph commit messages instead of heredocs, and use `Glob`/`Grep`/`Read` instead of shell file utilities.
- `gh` is authenticated as `lifeinprogrezz`.
