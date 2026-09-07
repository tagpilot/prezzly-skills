---
name: prezzly-presentations
description: Build HTML presentations and dashboards for Prezzly and upload them via MCP. Covers slides vs dashboard (ask when unsure), new deck vs revision (ask when unsure), inventory of local assets, uploadUrl + curl for binaries, add_files, and add_revision.
---

# Prezzly presentations

Use this skill when the user wants you to **build** a Prezzly deck and/or **upload** it. Do not list the library first. Do not upload a placeholder.

## When to use

- Build a new HTML presentation (slides) or dashboard (scrollable page).
- Upload a finished folder or HTML to Prezzly via MCP.
- Update an existing deck when the user wants a new version of that same deck.

## Slides vs dashboard

`kind` is chosen at upload time on `create_presentation`. The server does not infer it from the HTML. Check `index.html` before choosing.

- `kind=presentation` only if `index.html` has `.slide` elements and exactly one also has class `active`. 16:9 slides. CSS should hide non-active slides (`.slide { display: none }` and `.slide.active { display: block }`). Design each slide to fill a 16:9 frame.
- `kind=dashboard`: a scrollable page without `.slide` elements. Mark major sections with `id` or `data-prezzly-section` (fallback: `h1`–`h3`) so section navigation works.

If you are not sure, **stop and ask**. Do not pick for them. Ask one question: slides (presentation) or a scrollable page (dashboard)? Wait for the answer.

Typical cases that need a question: you have not opened `index.html` yet; the file is mixed (both `.slide` and a long scroll page); the user said "prezka" / "slides" but the file has no `.slide`; the user said dashboard / "strona" but the file has `.slide`.

If an upload `warning` says the kind is wrong and the file is unambiguous, call `update_presentation` with the suggested `kind`. If the file is mixed or the user asked for the other type, ask first. Do not create a second deck.

Speaker notes: `data-notes` on each `.slide`, or later via `get_notes` / `set_note` / `set_notes`. UI notes override code notes.

Content runs in a sandboxed iframe: no `localStorage`, no cookies, no top-level navigation.

## Runtime

Prezzly injects the runtime when it serves the deck. You do **not** need your own JavaScript for arrow keys. Prezzly toggles the `active` class (keyboard, presenter view, `?slide=N`, thumbnails).

Optional and harmless:

```html
<script src="https://api.prezzly.ai/runtime/v1/prezzly-present.js"></script>
```

Do not require `prezzly.io` as the runtime host. Do not add a custom ArrowLeft/ArrowRight handler unless the deck already has one and you want to keep it.

## New presentation vs revision

`create_presentation` makes a **new** deck. `add_revision` is a **new version of the same** deck. History, share links, and comments stay on that deck.

A large rewrite is still often a revision. How big the change is does **not** decide this.

Decide only when the user was explicit:

- Same deck, new version: they named the deck, pasted a Prezzly URL or presentation id, or said update / new version / next revision / replace the current one → Edit (same deck, new revision).
- New deck: first upload in the thread, or they said new presentation / another deck / a separate copy → `create_presentation`.

If you are not sure, **stop and ask**. Do not pick for them. Ask one question: new version of the existing deck, or a new presentation? Wait for the answer.

Typical cases that need a question: the thread already has a deck, the title is similar, they said "przerób" / "zrób od nowa" / "nowa wersja" without saying which, or they ask to upload a rebuilt folder after an earlier upload.

Do not call `list_presentations` to guess. Do not create a second deck "just in case".

## Inventory before upload

Skip a full inventory when you are only changing copy in an already-uploaded `index.html`. Do not read the whole HTML into chat just to paste it into MCP.

For a first upload, or when you added/removed/renamed assets, from the folder that contains `index.html`:

1. List **every** file (`ls -R` / `find`). Do not filter by extension. Images, fonts, and media count.
2. Collect relative references from `index.html` and every `.css` (`src`, `href`, `srcset`, `url()`). Skip `https:`, `data:`, `#`, and `mailto:`.
3. Each referenced path must exist on disk. If a file is missing on disk, tell the user. Do not upload a half deck.

## Upload

Resolve new presentation vs revision first. If this is a new version of an existing deck, skip `create_presentation` and go to Edit.

Never put image or font bytes in MCP tool arguments. The model cannot generate megabytes of base64.

If `create_upload_link` or `add_files` is not in your tool list, the client has a stale catalog. Stop. Tell the user to reload the Prezzly MCP server (Cursor: Settings -> MCP -> toggle). Do not send HTML, a zip, or a placeholder through `add_revision`.

### Chat clients (Claude.ai, ChatGPT, no disk)

Send `files` with one self-contained `index.html`. Put CSS/JS in that HTML. Use https image URLs or small data URIs. Optional: `add_files` with `url` so the server fetches a remote asset.

```
create_presentation({ "title": "Q1 Review", "kind": "presentation", "files": [{ "path": "index.html", "content": "<!doctype html>..." }] })
```

Do not build a zip in the browser.

### Local clients (Cursor, Claude Code, files on disk)

1. Call `create_presentation` with title and kind only. Do not put `index.html` or binaries in tool arguments.

```
create_presentation({ "title": "Q1 Review", "kind": "dashboard" })
```

2. Read `uploadUrl`. Prefer one zip of the whole folder (unwraps a single top-level folder). Use a zip when any path has spaces or non-ASCII characters (`curl -T` does not encode the URL).

```
cd <dir> && zip -r /tmp/deck.zip . -x '.git/*' '.cursor/*' '.agents/*' 'node_modules/*' 'skills-lock.json' '*.zip' '*.code-workspace'
curl -sS -T /tmp/deck.zip -H 'Content-Type: application/zip' "<uploadUrl>"
```

Or upload `index.html` first, then each remaining ASCII path:

```
curl -sS -T index.html "<uploadUrl>index.html"
curl -sS -T "assets/ozon.webp" "<uploadUrl>assets/ozon.webp"
```

`curl -T` to `uploadUrl` (trailing slash) is fine: the server unpacks a zip even if curl appends `deck.zip` to the path.

3. Read `missingAssets` from each curl JSON body. Repeat until the list is empty.
4. If `uploadUrl` expired, call `create_upload_link` and continue.
5. Call `get_presentation_files` once. When `missingAssets` is empty, give the user `viewUrl`. Do not curl the content URL (it needs a session or share token).

Add `-w '\n%{http_code}\n'` to each `curl` so you see the status. `curl -sS` prints the JSON body on 4xx (it does not fail the command), so always read the body, not just the exit code.

### Reading upload responses

| Status | Body signal | Do |
|--------|-------------|-----|
| 200 | `missingAssets` non-empty | Upload the listed paths. Not done yet. |
| 200 | `missingAssets: []` | Call `get_presentation_files`, then give `viewUrl`. |
| 200 | `warning` set | Act on the warning (stub, shrink, missing, or wrong kind) before finishing. |
| 200 | `warning` mentions `update_presentation` and `kind=` | Call `update_presentation` with the suggested `kind`. Do not create a second deck. |
| 401 | `create_upload_link` hint | Token expired. Call `create_upload_link`, retry. |
| 402 | `code: plan_limit` | Storage or upload cap hit. Tell the user; do not retry. |
| 409 | `Upload index.html or a zip first` | Empty deck. Upload `index.html` or a zip before assets. |
| 409 | `files` + `missingAssets` listed | Path exists with different bytes. To replace it use `add_revision` or a zip to `uploadUrl`. |
| 400 | `index.html cannot be added this way` | Replace HTML with a zip to `uploadUrl` (local) or `add_revision` files (chat), not a single PUT of index.html. |

Re-`curl -T` of the exact same bytes is safe: it returns 200, not 409.

Do not run `zip -j`. Do not put binaries in `archive` or `files[].content`.

## Edit

Only after the user chose an existing deck (or you asked and they said revision).

`update_presentation` can rename, move, or change `kind`. Use `kind` when an upload warning said the content type was wrong, or the user asked to treat the deck as slides vs a dashboard. It does not upload files.

If `create_upload_link` is not in the tool list, **stop**. Tell the user to reload the Prezzly MCP server (Cursor: Settings -> MCP -> toggle). Do not fall back to stuffing HTML, a zip, or base64 into MCP arguments.

### Local clients (Cursor, files on disk)

Do not paste `index.html` or binaries into `add_revision` / `files` / `archive`. That copies the whole file through the model and is slow.

1. Call `create_upload_link` with the presentation id. Read `uploadUrl`.
2. Zip what changed. A text-only tweak: zip `index.html` from the deck folder. Previously uploaded assets still referenced by the HTML are reused (`reusedAssets`). New or replaced images go in the same zip.
3. `curl -sS -T /tmp/deck.zip -H 'Content-Type: application/zip' -w '\n%{http_code}\n' "<uploadUrl>"`
4. Call `get_presentation_files`. When `missingAssets` is empty, give `viewUrl`.

A zip to `uploadUrl` is a new revision. A single PUT of `index.html` to an existing deck is rejected (400) - do not try that.

### Chat clients (no disk)

`add_revision` with `files` containing the real `index.html`. Assets still referenced are reused. Do not send a placeholder. Optional: `add_files` with `url` for a remote asset.

### After upload

- Compare `revision.totalBytes` with the previous revision from `get_presentation` (`revisions[]`). If the new one is much smaller and that was not intended, call `restore_revision`.
- Add missing assets to the current revision: `curl -T` to `uploadUrl`, or `add_files` with `url` / small `content`. Existing paths are rejected; zip a new revision to replace.
- `restore_revision` rolls back.
- Do not call `create_presentation` for an update. A full rebuild of the same deck is still a revision.

## Share links

`create_share_link` creates a public view-only link. On Pro, Team, and
Enterprise it never expires by default; pass `ttlDays` to request a dated link.
On Free it expires after 7 days by default and is capped at 30 days. Pass
`neverExpires: true` only for paid plans. Free returns a structured
`plan_limit` error. A never-expiring link has `expiresAt: null` in
`create_share_link` and `get_presentation`.

## Slide markup

```html
<section class="slide active" data-notes="Opening remarks.">
  <h1>Hello</h1>
</section>
<section class="slide" data-notes="Main point.">
  <h2>Topic</h2>
</section>
```

## Do not

- Upload a stub, placeholder, or empty shell and promise to revise later.
- Send a placeholder `index.html` through `add_revision` to get `uploadUrl`. Use `create_upload_link`.
- Upload a zip to `uploadUrl<name>.zip` as a regular asset. Zip goes to `uploadUrl` (or is unpacked if the path ends in `.zip`).
- Decide new presentation vs revision when unsure. Ask.
- Guess `kind` when the file is mixed, unchecked, or conflicts with what the user said. Ask.
- Call `list_presentations` to guess which deck to update, or before a first upload.
- Filter the folder by `html`/`css`/`js`/`md` and ignore images.
- Put image, font, zip, or a local `index.html` in tool arguments (`archive` or `files[].content`). Local clients use `uploadUrl` + curl.
- Fall back to pasting HTML into `add_revision` when `create_upload_link` is missing. Reload MCP instead.
- Flatten a zip (`zip -j`). Keep relative paths. A zip that contains only `index.html` is fine for a text-only revision.
- Tell the user the upload is done while `missingAssets` is non-empty or `warning` is set.
- Ignore a kind-mismatch `warning`. Call `update_presentation` with the suggested `kind`.
- Give `viewUrl` without calling `get_presentation_files` after the last upload.
