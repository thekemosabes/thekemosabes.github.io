# The Kemosabes — Website

Internal maintainer notes. Not linked from the site itself; lives in the repo for whoever
edits this next (Claude, Karan, or anyone else on the band).

**Live site:** https://thekemosabes.github.io
**Repo:** https://github.com/thekemosabes/thekemosabes.github.io (GitHub account: `thekemosabes`)
**Local copy:** `/Users/karangera/Documents/TheKemosabesWebsite`

## Stack

One file, `index.html` — all CSS and JS inline, no build step, no frameworks, no
dependencies. Images live under `assets/`. That's the entire site. Keep it that way;
don't introduce a bundler/framework/package.json unless explicitly asked.

## Local dev

```bash
python3 -m http.server 8765 --directory /Users/karangera/Documents/TheKemosabesWebsite
```

Then open http://localhost:8765/.

## Deploying (push = live)

```bash
cd /Users/karangera/Documents/TheKemosabesWebsite
git add -A
git commit -m "describe what changed"
git push
```

GitHub Pages rebuilds automatically within ~1 minute of every push to `main`. No dashboard
steps needed. `gh` is already authenticated on this machine (`gh auth status` to confirm) —
git push works with no credential prompt.

## Hard content rules (stated explicitly, more than once)

- **No lyrics anywhere in the page source.** Not in a hidden div, not in a comment. Song
  cards get a title + one-line description + links to live versions only.
- **Exact venue name spellings** (used consistently everywhere): Cobbler & Crew · Shisha
  Cafe · The Stables · Soul Fry · The Bluebop Cafe · High Spirits Cafe · antiSOCIAL.
- **Location is "Mumbai"**, not "Mumbai & Pune" (footer, meta description, etc.) even though
  some venues are in Pune.
- **Never fabricate a photo credit or an Instagram handle.** If a photo's photographer isn't
  confirmed (no watermark, no known handle), either credit them by the plain-text name only
  (no link) or leave the credit off the figcaption entirely. Do not guess at a handle URL.
  Confirmed photographers so far:
  - **David Lall** — `https://www.instagram.com/shotinfocus101`
  - **Pallavi Gawas** — `https://www.instagram.com/tothegloriousunknown`
  - **Walrus Photography** and **Celeste X Frames** — real names seen on watermarks, but no
    confirmed handle yet, so they're credited as plain text with no link. Ask the band before
    inventing one.
- Karan Gera goes by "**The Brown Kemosabe**" on the site; each member has a similar
  Kemosabe title (see `#band` section) — keep those when editing member content.

## Structure (section IDs, in page order)

`#album` → `#links` (main link stack) → `#watch` ("From the Bandstand", video carousel) →
`#stages` ("Our Shows So Far", per-year poster carousels) → `#gram` ("From the 'Gram",
Instagram reel carousel) → `#moments` ("Caught in the Act", live-photo carousel) →
`#songbook` (Originals/Covers tabs) → `#band` (Meet the Band) → `#story` (Our Story +
Gospel) → `#listening` (Spotify playlists) → footer/Book Us.

## The carousel pattern (used 4 times: Bandstand, Shows, Gram, Moments)

```html
<div class="carousel" data-carousel>
  <button class="car-arrow car-prev">‹</button>
  <div class="SOMETHING car-track"> ...cards... </div>
  <button class="car-arrow car-next">›</button>
</div>
<div class="car-progress"><span class="car-progress-fill"></span></div>
```

One generic JS block (`document.querySelectorAll("[data-carousel]")...`) wires up *every*
carousel on the page — arrows, scroll-snap, and the progress bar — automatically. To add a
new carousel, just use this markup; you don't need to touch the JS.

**Gotcha #1 — closed accordions.** A carousel inside a closed `<details>` (like the 2025/2024
year groups in Shows) has zero width while hidden, so its scroll math is wrong until it's
opened. There's a `toggle` listener on every `.year-group` that re-fires `window` resize to
fix this — if you add another collapsible carousel, make sure it's still inside a
`.year-group` (or extend that listener) or it'll silently mis-measure.

**Gotcha #2 — class name drift.** The lightbox click-handler and the analytics `labelFor()`
classifier both hard-match specific class names (currently `a.poster, a.poster-lg, a.lb`).
If you rename or restyle a clickable card's class, **grep the whole file for the old class
name** before you're done — this exact bug (renamed `.poster` → `.poster-lg` during the Shows
carousel conversion, forgot to update the JS selector, posters silently stopped opening the
lightbox) shipped once already and took a QA pass to catch.

## Asset naming convention (`assets/gallery/`)

Every photo ships as a pair: a full version and a thumbnail, both `.webp`.

- Full: long side **1400px**, quality ~82.
- Thumb: long side **640px**, quality ~80, filename suffix **`-t`** (e.g.
  `antisocial-jump-airborne.webp` + `antisocial-jump-airborne-t.webp`).
- Filename pattern: `{venue-or-context}-{subject}.webp` — lowercase, hyphenated, no spaces.
- The gallery `<figure>` links to the **full** image (`target="_blank"`, opens in the in-page
  lightbox via `a.lb`) and displays the **thumb** as the visible `<img>`.
- `width`/`height` attributes on the `<img>` must match the **thumbnail's** actual pixel
  dimensions, not the full image's — the browser uses these to reserve layout space before
  the image loads.

Quick resize recipe (Pillow):
```python
from PIL import Image, ImageOps
im = ImageOps.exif_transpose(Image.open(src)).convert("RGB")
scale = 1400 / max(im.size)
im.resize([round(d*scale) for d in im.size], Image.LANCZOS).save(full_out, "WEBP", quality=82, method=6)
```

## Photo sourcing

Band photos live across many albums in the band's Google Photos (`thekemosabes@gmail.com`),
not just the "A Scroll through" compilation album — that compilation stores *downscaled*
copies, so for full resolution go to the per-show album (Albums tab → e.g. "At The Crossroads
- Live at Stables"). To pull a full-res image out of an *authenticated* (non-share-link)
Google Photos page: open the photo, read the `<img>` `src` from the DOM (domain
`photos.fife.usercontent.google.com`), then `fetch(url, {credentials:'include'})` from page
JS and base64-encode the result — a plain `curl` on that URL will fail (needs the session
cookie). A `photos.google.com/share/.../photo/...` link's `lh3.googleusercontent.com` image
*can* be `curl`'d directly (no auth needed), just bump the `=w###-h###` suffix for higher res.

## Analytics & subscribe (both wired, one needs a step from the band)

- **Click analytics**: GoatCounter, free, privacy-friendly, no cookie banner needed. The
  `<script>` tag near `</head>` has a placeholder site code (`THEKEMOSABES.goatcounter.com`)
  that silently no-ops until replaced. To activate: sign up free at goatcounter.com, then
  swap `THEKEMOSABES` for the real site code in that one line. A single delegated click
  handler (`track()` / `labelFor()` near the top of the main `<script>`) already tags every
  video play, reel play, tab switch, carousel arrow, lightbox open, and outbound link — no
  further JS changes needed when adding content, the classifier picks new instances up
  automatically as long as it reuses the existing classes (`.yt`, `.ig-play`, `.tab-btn`,
  etc).
- **Subscribe / mailing list**: a real Google Form ("The Kemosabes — Get Updates"), linked as
  the "Get Updates on Shows & Releases" pill. Responses land in a Google Sheet linked to that
  form (in the band's Drive). No backend, no third-party mailing service.

## Domain

Currently just `thekemosabes.github.io` (free). Band is considering a custom domain —
**on hold pending band approval**, do not purchase anything. If/when they say go: cheapest
honest (flat, no bait-and-switch renewal) options checked so far were `thekemosabes.in`
($7.83/yr flat) and `thekemosabes.com` ($11.08/yr flat); the sub-$3 gTLDs (`.live`, `.online`,
`.rocks`, etc.) all balloon to $18–29/yr on renewal.

## Style, at a glance (read the CSS for anything more specific — don't assume, it has
changed several times already)

```
--ink:#0a0f24        page background
--brass:#d9a441      accent / links / active states
--cream:#f3e6c9       body text on dark
```
Fonts currently: Abril Fatface (headings/titles), Ubuntu (body), Londrina Solid (labels,
kickers, dates, credits — uppercase, letter-spaced). This has changed more than once over the
life of the project — if in doubt, check the actual `<link>`/`font-family` in the file rather
than trusting a stale memory of an earlier revision.
