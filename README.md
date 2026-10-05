# The Kemosabes - Band Website

Source for **https://thekemosabes.github.io**, the official site of The Kemosabes, a rhythm,
blues and soul band from Mumbai.

## Stack

One file, `index.html`, with all CSS and JS inline. No build step, no frameworks, no
dependencies. Images live under `assets/`. Keep it that way.

## Run it locally

```bash
python3 -m http.server 8765
```

Run it from the repo folder, then open http://localhost:8765/.

## Deploying

GitHub Pages serves `main`. Every push to `main` goes live within about a minute. Pages
caches for 10 minutes, so hard-refresh (Cmd/Ctrl+Shift+R) to see a change.

Before committing, run `git status` and read the list. Only `index.html`, `assets/`,
`README.md` and `.gitignore` belong in this repo. Never push drafts, press-kit source
files, lyrics or screenshots.

## Content rules

- No lyrics anywhere in the page source, including comments and hidden elements.
- Use exact venue spellings: Cobbler & Crew, Shisha Cafe, The Stables, Soul Fry,
  The Bluebop Cafe, High Spirits Cafe, High Note, antiSOCIAL. Check new venues against Google Maps.
- Band location is "Mumbai".
- Never guess a photographer's credit or handle. If it isn't confirmed, credit the plain
  name or leave it off.
- Keep each member's Kemosabe title when editing the Meet the Band section.

## Page structure

Hero → album banner and next-show ticket → From the Bandstand (videos) → Meet the Band → Our Shows →
On Instagram → Live Moments (photos) → Songbook (Originals / Covers) → Our Story →
Recommended Listening → Book Us.

Every section's `id` matches a link in the site menu (`#sidenav`). A new section needs a
menu link with an icon from the SVG sprite at the top of `<body>`.

## Navigation

- **Desktop and tablet (700px+):** an icon rail on the left that expands on hover, focus or
  tap. From 1760px wide it floats in beside the content. Below that it stays at the screen
  edge, so it never covers the hero photo.
- **Phone (under 700px):** a Menu button at the top left opens the same links as a panel.

## Carousels

Every carousel uses the same markup and one shared script:

```html
<div class="carousel" data-carousel>
  <button class="car-arrow car-prev">‹</button>
  <div class="SOMETHING car-track"> ...cards... </div>
  <button class="car-arrow car-next">›</button>
</div>
<div class="car-progress"><span class="car-progress-fill"></span></div>
```

- A carousel inside a closed `<details>` measures zero width. Keep it inside a `.year-group`,
  whose toggle listener fixes the measurement.
- The lightbox and click tracking match class names (`a.poster`, `a.poster-lg`, `a.lb`).
  If you rename a class, search the file for the old name.

## Adding photos (Live Moments)

- Each photo ships as a pair of `.webp` files: full size (long side 1400px) and a thumbnail
  (long side 640px, `-t` suffix).
- Name files `{venue}-{subject}.webp`: lowercase and hyphenated.
- The `<img>` shows the thumbnail. Its `width`/`height` must match the thumbnail's real
  pixels, and `style="--ar:W / H"` must use the same numbers, so the frame fits the photo.

## Adding a show

1. Poster: `assets/posters/YYYY-MM-DD.webp`, long side about 600-800px.
2. Ticket stub near the top of the page: shows only the next upcoming show.
3. Our Shows carousel: add a card first in the right year. Past shows stay as the archive.

## Hero

The hero photo shows all six members at every screen size. Logo and text sit on the brick
wall above the players' heads. After changing the hero, check phone, tablet, laptop and
large-monitor widths. No head should sit under the logo, the text, the icons or the
menu rail.

## Style

```
--ink:#0a0f24     background
--brass:#d9a441   accent, links, active states
--cream:#f3e6c9   body text
```

Fonts: Abril Fatface (headings), Ubuntu (body), Londrina Solid (labels, dates, credits).
Check the CSS for anything more specific.
