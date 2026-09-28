# The Kemosabes — Website

Internal maintainer notes. Not linked from the site itself; lives in the repo for whoever
edits this next (Claude, Karan, or anyone else on the band).

**Live site:** https://thekemosabes.github.io
**Repo:** https://github.com/thekemosabes/thekemosabes.github.io (GitHub account: `thekemosabes`)
**Local copy:** `<repo folder>`

## Stack

One file, `index.html` — all CSS and JS inline, no build step, no frameworks, no
dependencies. Images live under `assets/`. That's the entire site. Keep it that way;
don't introduce a bundler/framework/package.json unless explicitly asked.

## Local dev

```bash
python3 -m http.server 8765 --directory <repo folder>
```

Then open http://localhost:8765/.

## ⚠️ Before every `git add -A`: check what you're actually staging

This bit the project once already — worth reading before you push.

When this repo's local copy moved from a temporary scratch folder to its permanent home,
the app auto-copied *everything* from the old location into the new one at the same paths —
not just the site, but a `build/` archive of old drafts, and a `src/` folder containing the
band's actual press-kit PDF, unreleased lyrics `.docx` files, and a raw saved copy of a
YouTube page (which happened to contain YouTube's own embedded API keys, and GitHub's
secret scanner flagged it as a leaked credential). A blind `git add -A` swept all of it into
a commit and pushed it to this **public** repo before anyone noticed. Fixing it for real
required both an untracking commit *and* a full `git filter-repo` history rewrite +
force-push, since removing a file in a new commit does not remove it from earlier commits —
anyone can still dig it out of git history/GitHub's commit view until the history itself is
rewritten.

**Rule going forward:**
- Before any `git add -A`, run `git status` first and actually read the list — especially
  right after a directory move, a restore-from-backup, or anything else that could have
  dropped in files you didn't create in this session.
- Only this repo's real content should ever be tracked: `index.html`, `assets/`,
  `README.md`, `.gitignore`. If you see `build/`, `src/`, a `.zip`, loose screenshots, or
  anything that isn't the site itself show up as untracked, that's a signal something
  landed here that shouldn't have — `.gitignore` already excludes the known offenders
  (`build/`, `src/`, `kemosabes-site.zip`, `qa-photo-credits.png`, `_spec/`), but a new one
  could show up under a different name.
- If something sensitive does get pushed anyway: removing it in a follow-up commit is not
  enough on its own. It needs `git filter-repo --path <bad-path> --invert-paths` (or BFG)
  run against full history, then `git push --force`. That's a history rewrite — always
  confirm with Karan before force-pushing, but don't treat a plain removal commit as "handled"
  when the leak is still sitting in history.

## Deploying (push = live)

```bash
cd <repo folder>
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

## Copy rule: don't repeat the same descriptor twice in a row

The hero tagline ("The Ultimate Juke Joint Experience") deliberately does **not** say
"...Blues & Soul Experience" — the subline directly below it already says "A Rhythm, Blues,
& Soul Band," so repeating "Blues & Soul" in both lines back-to-back reads redundant. When
editing copy that sits in a stack (tagline → subline → kicker, etc.), check the adjacent
lines for repeated descriptors before finalizing wording, not just each line in isolation.

## Responsive container widths (`.wrap` / `.wrap-wide`)

Base: `.wrap{max-width:640px}` (text-heavy sections — intros, songbook, footer) and
`.wrap-wide{max-width:1040px}` (carousels, galleries, the hero). Both scale up at two large
breakpoints so the page doesn't look like a narrow stranded column with huge dead margins on
big monitors — this was a real complaint ("spread out better on desktop"), fixed by:
```css
@media (min-width:1400px){ .wrap{max-width:760px;} .wrap-wide{max-width:1320px;} }
@media (min-width:1800px){ .wrap{max-width:840px;} .wrap-wide{max-width:1520px;} }
```
`.wrap` is intentionally kept *narrower* than `.wrap-wide` even at these larger sizes —
paragraph text shouldn't stretch past a comfortable reading width, but carousels/galleries
should use the extra room. If you add a third breakpoint or change these numbers, update
both classes together so the ratio between them stays sensible.

## Mobile side-nav: transparent, not a solid block

The mobile/tablet nav drawer (`.sidenav` under the `max-width:1199px` media query) is
deliberately translucent — `background:rgba(10,15,36,.5)` + `backdrop-filter:blur(18px)`,
with a text-shadow on the links for legibility — so you can still see the blurred hero
photo through it instead of it feeling like an opaque panel dropped on top of the page. The
collapsed state is just the small "☰ Menu" pill (`.nav-toggle`) — don't replace that with
an always-visible full nav on small screens; the pattern is "small indicator → tap → glass
drawer expands," not "drawer always partially open."

## `og-image.jpg` and link-preview caching (a real gotcha this session hit)

`assets/og-image.jpg` (1200×630) should be the band logo centered on the site's dark
background with the brass glow — **not** a candid/solo photo of one member. If you regenerate
it, keep the same filename so no HTML changes are needed.

**The confusing part:** WhatsApp (and most chat apps) snapshot the link preview *once, at
the moment a message is sent*, and bake it into that message forever — reopening an old chat
bubble will keep showing the old image even after the live file is fixed and verified correct
(checksum-diff the live file against your local copy to prove this to yourself before chasing
a phantom bug). To see a fix, send the link in a **new** message, or append a throwaway query
string (`?x=1`) to force a guaranteed-fresh fetch. Meta's Sharing Debugger can force a re-scrape
but requires logging into a Facebook account — not something to do on the band's behalf without
asking first.

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
