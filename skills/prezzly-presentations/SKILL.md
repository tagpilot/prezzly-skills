---
name: prezzly-presentations
description: Build HTML presentations compatible with Prezzly — presenter mode, speaker notes (data-notes), live slide control, and URL slide navigation via prezzly-present.js runtime.
---

# Prezzly presentations

When creating HTML presentations for Prezzly, follow this contract exactly so presenter mode, speaker notes, live slide control, and smooth slide transitions work correctly.

## Runtime (required, must be last script before `</body>`)

```html
  <script src="https://prezzly.io/runtime/v1/prezzly-present.js"></script>
</body>
```

The runtime MUST load after all other scripts (especially the slide engine). It handles:
- `?slide=N` URL parameter (1-based): jumps to slide N on load without a visible flash
- `postMessage` protocol for live presenter control between windows
- Speaker notes extraction and communication

> Prezzly also injects this script and a no-flash guard automatically when serving the presentation — you do not need to add anything extra for that. Include the `<script>` tag explicitly so the deck also works in local preview.

## Slide structure

- Each slide is an element with class `slide`.
- Exactly **one** slide has class `active` on load — always the **first** slide.
- Keyboard navigation (ArrowLeft / ArrowRight / Space) MUST change which slide has `active` — the runtime uses synthetic key events to navigate to `?slide=N`.
- Do not use `display:none` directly on individual `.slide` elements via inline `style` — use class-based CSS so the runtime's class observation works correctly.

```html
<div id="deck">
  <section class="slide active" data-notes="Welcome slide notes here.">
    <h1>Title</h1>
  </section>
  <section class="slide" data-notes="Second slide talking points.">
    <h2>Topic</h2>
  </section>
</div>
```

## Speaker notes

Add presenter notes on **every** slide using the `data-notes` attribute:

```html
<section class="slide" data-notes="Remind audience about the Q1 targets. Mention the 15% growth figure.">
```

- Notes are extracted from `data-notes` at upload time and shown in Prezzly's presenter view.
- Users can override notes in the Prezzly dashboard without changing the HTML.
- Always fill `data-notes` — an empty string is acceptable but a real note is better.

## URL slide navigation

The runtime reads `?slide=N` (1-based) on load and updates the URL on every slide change. This enables:
- Presenter preview thumbnails (prev/current/next shown as mini-iframes with `?slide=N`)
- Deep links to specific slides
- Sync between audience window and presenter window

Do not rely on `same-origin` access — presentations run in a sandboxed iframe (`allow-scripts` only, no `allow-same-origin`).

## What you get

- **Presenter view**: separate window with current / previous / next slide previews and speaker notes.
- **Live control**: navigate slides from the presenter window; audience window follows in real time.
- **Notes in Prezzly**: upload extracts `data-notes`; users can edit notes per slide in the dashboard.

## Full minimal template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>My presentation</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    .slide { display: none; min-height: 100vh; padding: 3rem; }
    .slide.active { display: block; }
  </style>
</head>
<body>
  <section class="slide active" data-notes="Opening remarks for the presenter.">
    <h1>Hello</h1>
  </section>
  <section class="slide" data-notes="Main point one — elaborate here.">
    <h2>Topic</h2>
  </section>
  <script>
    const slides = [...document.querySelectorAll('.slide')];
    let current = 0;
    function goTo(i) {
      slides[current].classList.remove('active');
      current = Math.max(0, Math.min(i, slides.length - 1));
      slides[current].classList.add('active');
    }
    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowRight' || e.key === ' ') { e.preventDefault(); goTo(current + 1); }
      if (e.key === 'ArrowLeft') { e.preventDefault(); goTo(current - 1); }
    });
  </script>
  <!-- Runtime must be the LAST script before </body> -->
  <script src="https://prezzly.io/runtime/v1/prezzly-present.js"></script>
</body>
</html>
```
