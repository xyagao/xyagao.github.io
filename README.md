# Xiaoya Gao — personal website

Static site (plain HTML + one CSS file). No build step, no dependencies.

Design follows sites.google.com/view/shuhuasi: serif typography and centred
section headings. Papers live on `research/` only — the home page is the intro. The hero departs from
it — photo sits beside the intro (as on guangbinhong.com) rather than centred
above it, and the layout is full-bleed rather than a centred column.

## Structure

```
website/
├── index.html          Home — photo, bio, references, contact
├── research/index.html JMP, working papers, work in progress
├── teaching/index.html Courses taught (instructor + TA)
├── css/style.css       All styling (design tokens at the top)
├── images/profile.jpg  Headshot (square, shown at 300x300)
├── favicon.ico         XG monogram, multi-size (16/32/48)
├── images/favicon-*.png    16 / 32 / 48 — no ring, larger letters
├── images/apple-touch-icon.png  180 — with ring
├── images/icon-512.png      512 — with ring
└── files/cv.pdf        Copy of CV_XG_2026August.pdf
```

## Preview locally

```bash
python3 -m http.server 4173 --directory website
```

Then open http://localhost:4173

## Common edits

**Width** — three tokens in `css/style.css`:
- `--frame` (1140px) — the widest the page ever gets. **Do not remove this cap.**
  Without it, text on a 1920px monitor runs to ~175 characters per line and
  paragraphs stop wrapping altogether; the layout looks broken on any large
  display even though it seems fine on a laptop.
- `--gutter` — `clamp(24px, 4vw, 72px)`, the side padding.
- `--measure` (44rem) — the reading column for prose blocks inside the frame.

Between 760px and 1920px this holds the intro at 56–81 characters per line,
which is the readable band.

**Typography** — `--head` (Lora) and `--body` (Gentium Book Plus) at the top of
`css/style.css`. Body is 18px; the reference site uses 14.7px, which is too small
to read comfortably.

**Light / dark** — a `☾ / ☀︎` button in the header. It sets `data-theme` on
`<html>` and remembers the choice in `localStorage`.

**The site always opens in light mode**, deliberately — it ignores the reader's
OS dark-mode setting. Dark applies only when someone presses the toggle. Do not
re-add a `prefers-color-scheme` block; that would undo it.

Three parts, all of which must agree:
1. the light tokens on `:root` and the dark ones on `:root[data-theme="dark"]`
   (both in `css/style.css`);
2. a small script at the top of `<head>` on every page that applies the stored
   theme *before first paint* — without it the page flashes white on load;
3. the toggle script before `</body>`.

To change the symbol, edit `MOON` and `SUN` in that toggle script on all three
pages. `\u2600\uFE0E` is a deliberate sun-plus-variation-selector; without the
`\uFE0E` the glyph renders as a colour emoji on macOS and iOS.

Dark values are `--ink #E6E9E4` (14.5:1), `--muted #9BA79D` (7.1:1) and
`--accent #A9C3B0` (9.4:1) on `--bg #151916`.

**Link colour** — `--accent` (`#002A5C`). The reference site uses black links;
navy keeps links identifiable.

**Photo** — `.hero-photo` is 320px wide with `height: auto`, so the full
photo shows at its own aspect ratio; nothing is cropped. The source is the
2852x3993 portrait in `Job Market/photo.jpg`, scaled to 1000x1400 (134 KB) for
retina. To swap it, resize the new photo to 1000px wide and overwrite
`images/profile.jpg`, then bump `?v=` on the `<img>` in `index.html`.

**Update the CV** — do *not* overwrite `files/cv.pdf` by hand. It is generated:

```bash
python3 ../CV/make_web_cv.py
```

`CV/CV_XG.tex` stays the single source of truth. The script derives the public
copy from it, removing the personal phone number and pointing the Website line
at gaoxiaoya.com. The full CV (with phone) is what goes to search committees —
`CV/CV_XG_2026August.pdf`. Re-run the script after any CV edit.

It aborts rather than publishing if the phone number survives the transform, or
if the contact block in the `.tex` changes shape.

**Add a paper** — copy a `<div class="pub-entry">` block. Classes available:
`pub-title`, `pub-authors`, `pub-venue` (italic — used for "Draft available upon
request"), `pub-abstract` (a `<details>` toggle).

Papers appear on `research/` only.

**Reference links** (verified Aug 2026) — in `index.html`:
- Yoram Halevy — https://yoram-halevy.faculty.economics.utoronto.ca/
- Colin Stewart — https://sites.google.com/view/colinstewart
- Marcin Pęski — https://marcinpeski.github.io/

**Add a PDF later** — drop it in `files/`, then add inside the `pub-entry`:

```html
<div class="pub-links"><a class="pub-link" href="files/jmp.pdf">PDF</a></div>
```

(The `.pub-links` / `.pub-link` styles were removed with the serif rewrite —
re-add them to `css/style.css` if you use this.)

## Deploying to GitHub Pages

1. Create a repo named `<your-username>.github.io`.
2. Push the *contents* of `website/` to the repo root:

```bash
cd website && git init && git add -A && git commit -m "Initial site" && git branch -M main
```

3. Add the remote, push, then enable Pages in Settings → Pages (source: `main`, `/root`).

All internal links are relative, so the site also works from a subdirectory.


## Favicon

An "XG" monogram in Georgia Bold: warm beige serif letters on a deep sage disc.
Colours are `#40584A` (sage) and `#E8DCC8` (beige) — 5.71:1, and the sage is the
same family as the site's `--accent` (`#4A6250`), a shade deeper for richness.

It ships in **two cuts**, because a monogram that works at 512px turns to mush
at 16px:
- **Large** (180, 512) — with the pink ring, letters at 0.40 of the canvas.
- **Small** (16, 32, 48) — no ring, letters at 0.48. The ring costs more
  legibility than it buys once the icon is tab-sized.

Letter size is capped by geometry: the glyph bounding box's *corners* must sit
inside the disc, so the limit is `hypot(w/2, h/2) <= radius * 0.94`. Above about
0.50 the X and G poke through the circle's edge.

To regenerate after changing colours or letters, re-run the generator script
(see git history) or redraw at 2048px and downsample with LANCZOS — drawing
directly at 32px gives jagged edges, since Pillow's `ellipse` is not
antialiased.