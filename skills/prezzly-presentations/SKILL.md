---
name: prezzly-presentations
description: Build HTML presentations and dashboards for Prezzly and upload them with one MCP call (create_presentation). Covers slides vs dashboard, runtime injection, chat vs local folder upload, and add_revision.
---

# Prezzly presentations

Use this skill when the user wants you to **build** a Prezzly deck and/or **upload** it. Upload is one tool call. Do not list the library first. Do not upload a placeholder.

## When to use

- Build a new HTML presentation (slides) or dashboard (scrollable page).
- Upload a finished folder or HTML to Prezzly via MCP.
- Update an existing deck the user already named.

## Slides vs dashboard

`kind` is chosen at upload time on `create_presentation`. It is not inferred from the HTML.

- `kind=presentation` (default): 16:9 slides. Each slide is an element with class `slide`. Exactly one also has class `active`. CSS should hide non-active slides (`.slide { display: none }` and `.slide.active { display: block }`). Design each slide to fill a 16:9 frame.
- `kind=dashboard`: one vertically scrollable page, not slides. Mark major sections with `id` or `data-prezzly-section` (fallback: `h1`–`h3`) so section navigation works.

Speaker notes: `data-notes` on each `.slide`, or later via `get_notes` / `set_note` / `set_notes`. UI notes override code notes.

Content runs in a sandboxed iframe: no `localStorage`, no cookies, no top-level navigation.

## Runtime

Prezzly injects the runtime when it serves the deck. You do **not** need your own JavaScript for arrow keys. Prezzly toggles the `active` class (keyboard, presenter view, `?slide=N`, thumbnails).

Optional and harmless:

```html
<script src="https://api.prezzly.ai/runtime/v1/prezzly-present.js"></script>
```

Do not require `prezzly.io` as the runtime host. Do not add a custom ArrowLeft/ArrowRight handler unless the deck already has one and you want to keep it.

## Upload (one call)

Call `create_presentation` once. Then stop and give the user `viewUrl`. The deck is already live.

### Chat clients (Claude.ai, no disk)

Send `files` with one self-contained `index.html`. Put CSS/JS in that HTML. Use https image URLs or small data URIs. Do not build a zip in the browser.

```
create_presentation({ "title": "Q1 Review", "kind": "presentation", "files": [{ "path": "index.html", "content": "<!doctype html>..." }] })
```

### Local clients (Cursor, Claude Code, files on disk)

Zip `index.html` plus assets as they are. Send `archive` as that zip, base64-encoded. Do not recompress or convert images. Do not base64 each file by hand.

```
create_presentation({ "title": "Q1 Review", "kind": "dashboard", "archive": "<base64 zip>" })
```

A single top-level folder in the zip is unwrapped. A lone `.html` file is renamed to `index.html`.

## Edit

Only if the user pointed at an existing deck. Call `add_revision` with a new `archive` or a new full `index.html` in `files`. Revisions are history. `restore_revision` rolls back.

Do not call `create_presentation` again for an existing deck.

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
- Call `list_presentations` before a first upload.
- Base64 every image by hand when you have a local folder. Use `archive`.
- Build a zip inside Claude.ai. Use `files` there.
- Treat `prezzly.io` as a required runtime URL.
- Tell the user the upload is done if the tool returns a stub `warning`. Call `add_revision` with the real files instead.
