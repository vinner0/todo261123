# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A deliberately minimal to-do tracker: plain HTML + JavaScript, **no frameworks, no build step, no tests, no server**. Open `index.html` directly in a browser. Keep it that way; the owner rejected extras such as a counter and "clear completed", so don't add features that weren't asked for.

## Files

- `index.html` is the whole app: inline CSS and JS.
- `mockup.html` is a static page showing the app in iPhone (393×852) and Galaxy S25 (360×780) frames. Each frame is an `<iframe srcdoc>` holding an HTML-escaped **copy** of `index.html` plus a seed script (three sample tasks) before `</body>`. It does not read `index.html` at runtime, so after any change to `index.html` the copies in `mockup.html` must be regenerated (it was originally built by a PowerShell script that escaped and embedded `index.html`).

## Architecture of index.html

- State is one `tasks` array of `{id, text, done, cat, due}`. `render()` rebuilds the whole `<ul>` from it; every mutation calls `save()` then `render()`.
- Editing is a per-row mode: `editingId` selects the row that `render()` draws as inputs with Save/Cancel (Enter saves, Esc cancels). It uses explicit buttons, not blur-to-save, because blur breaks when focus moves between the row's fields.
- Categories come from the `CATS` constant. `fillCats()` adds an extra option if a stored task has a category no longer in `CATS`.
- Theming uses CSS variables on `:root` with a `:root[data-theme=dark]` override. The theme starts from `prefers-color-scheme`, then the saved choice wins. The toggle is an emoji button.
- Overdue means `due < today()` and not done. `today()` builds a local-time `YYYY-MM-DD` string, so don't swap it for `toISOString()` (UTC would mark dates wrong near midnight).
- User text is inserted with `textContent`; keep it that way (no `innerHTML` for task text).
- `localStorage` keys are `tasks` and `theme`. All access is in try/catch, so the app still works when storage is blocked.

## Gotchas

- The desktop app's browser pane loads local files as `data:` URLs, where `localStorage` throws. Persistence can't be checked there; open the file in a normal browser to test it.
- The CSS class for overdue is `.meta.overdue` (the class sits on the `.meta` element itself). A descendant selector like `.meta .overdue` won't match.
- The layout was checked at 393 px and 360 px wide: no horizontal overflow, and the add form and edit row wrap. Re-check after layout changes.
