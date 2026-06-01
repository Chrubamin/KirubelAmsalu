# CLAUDE.md — Kirubel Amsalu Personal Website

## Project Overview

Static single-page academic portfolio for Kirubel Amsalu, Ph.D — Plasma Scientist and Postdoctoral Researcher at KRISS (Korean Research Institute of Standards and Science). Deployed via GitHub Pages from the `main` branch of `https://github.com/Chrubamin/KirubelAmsalu`.

## Stack

- HTML5 / CSS3 / vanilla JS + jQuery 3.x
- Bootstrap 4 grid
- Isotope.js (portfolio filtering)
- AOS (Animate on Scroll) — loaded from CDN, must call `AOS.refresh()` after section transitions
- Typed.js (hero typing animation)
- Canvas API + `requestAnimationFrame` (plasma species animation)

## Architecture — Non-Scrolling SPA

Sections are **not shown by scrolling**. They are toggled via CSS class:

```css
section {
  position: absolute;
  opacity: 0;
  top: 140px;
  bottom: 100%;   /* effectively off-screen */
}

section.section-show {
  top: 0;
  opacity: 1;
  padding-top: 100px;   /* offset for fixed header */
  min-height: 100vh;    /* cover full viewport, hiding fixed bg behind */
}
```

Navigation clicks add/remove `section-show` via jQuery (see `assets/js/main.js`). The header collapses from `height: 100vh` to `height: 90px; position: fixed` when any section is open.

**Implications:**
- AOS is scroll-based — it won't trigger. Call `AOS.refresh()` every time a section becomes visible.
- Isotope layout must be lazy-initialized on first section open (items have 0 effective layout when page loads with the section hidden). Never call `isotope()` on `window.load` alone.
- `ResizeObserver` on `#header` is needed to detect the 100vh→90px collapse and resize the canvas drawing buffer accordingly.

## Key Files

| File | Purpose |
|------|---------|
| `index.html` | All HTML content; inline `<script>` blocks for canvas, stats counter, contact form |
| `assets/css/style.css` | All styles including section layout logic, animations, dark skill cards |
| `assets/js/main.js` | Nav, isotope (lazy-init), AOS refresh |
| `assets/img/skills/` | Instrument / software skill images |
| `assets/img/project/` | Publication graphical abstracts |
| `assets/img/background/` | Section background images (`pla.jpg`, `About.jpg`, `contact.jpg`) |

## Sections

| Nav label | HTML id | Notes |
|-----------|---------|-------|
| Home | `#header` | Canvas plasma animation, Typed.js hero |
| About | `#about` | Profile photo + bio + interests |
| Education | `#education` | Timeline with pulsing dots |
| Experience | `#experience` | Timeline with pulsing dots |
| Projects | `#portfolio` | Isotope grid with filter buttons; stats bar (citations counter) |
| Skills | `#skills` | Dark `.skill-card` grid (Hardware + Software boxes) |
| Resume | external link | Opens Google Drive PDF |
| Contact | `#contacts` | Info boxes + mailto contact form |

## Skills Cards

Skill cards (`div.skill-card`) are **static** — no floating/bounce animation. The hover effect (`transform: translateY(-6px) scale(1.04)`) is the only animation. Do not re-add float animations.

## Portfolio Filtering

Filter buttons use Isotope with these data-filter values:

```html
<li data-filter="*">All</li>
<li data-filter=".filter-personal">First Author</li>
<li data-filter=".filter-collaborative">Collaborative</li>
```

Items must have the matching class directly on the `.portfolio-item` element (not on a child). Isotope is lazy-initialized (see `main.js`) — it initializes the first time `#portfolio` gains `section-show`, ensuring items have real dimensions when measured.

## Hero Canvas (Plasma Species Animation)

`#hero-canvas` is `position: absolute; width: 100%; height: 100%` inside `#header`. It renders ~33 labeled plasma species (electrons, ions, OH radicals, etc.) with pulsing opacity and faint connection lines. The drawing buffer is rebuilt via `resize()` when the header size changes. A `ResizeObserver` on `canvas.parentElement` handles the 100vh → 90px header collapse.

## Citations Counter

Stats in `#portfolio` count up on first visit. Triggered by a `MutationObserver` on `#portfolio` watching for `section-show`. The counter runs once (`var ran = false` guard). The `stat-cit` element has a child `<span id="stat-cit-plus">` for the "+" suffix, revealed after the count reaches 220.

## Contact Form

Uses `mailto:` — no backend. On submit, the form composes a mailto URI with name, email, subject, and message pre-filled, then sets `window.location.href`. This opens the user's default email client.

## Deployment

Pushing to `main` branch auto-deploys to GitHub Pages. The remote URL is `https://github.com/Chrubamin/KirubelAmsalu.git` (repo was renamed from `KA`).

## Common Pitfalls

- **Half-page blank space**: caused by `section.section-show { top: 95px }` — the section doesn't cover the top 95px, letting the fixed `body::before` background show through. Fix: `top: 0; padding-top: 100px`.
- **AOS not triggering**: sections aren't scrolled into view, so AOS never fires. Always call `AOS.refresh()` after toggling `section-show`.
- **Isotope filters broken**: if Isotope is initialized on `window.load` while the section is hidden, items may have wrong cached dimensions. Use lazy-init (see main.js).
- **Canvas invisible in nav bar**: after header collapses, canvas CSS height is 90px but drawing buffer stays at old height unless `ResizeObserver` triggers `resize()`. Add `ResizeObserver` on the header element.
- **`#about` position conflict**: `#about { position: relative }` overrides `section { position: absolute }`. Use `z-index` on child elements for overlays instead.
