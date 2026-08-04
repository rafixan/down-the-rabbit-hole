---
name: book-pages
description: How to change the Down the Rabbit Hole book pages safely. Use whenever editing any of the seven .html files — a feature, a fix, a style tweak, or a bulk edit — and before deploying. Encodes the seven-file fan-out, the verification gates that catch the failures this repo actually has, and the deploy ritual.
---

# Editing the book pages

Seven standalone single-file book pages: `alice.html`, `1984.html`,
`gatsby.html`, `prince.html`, `meditations.html`, `rilke.html`, `dune.html`.
`index.html` is **the library** — the scroll-journey landing page through all
seven worlds (it holds its own copies of the splash, weather, and burst
systems). `library.html` is a legacy redirect stub to `/` — leave it alone.
No build step, no shared CSS or JS. Every page carries its own copy of the
same components.

**The defining hazard: one logical change is a seven-file change.** Almost
every bug that has shipped here came from treating it as a one-file change,
or from a bulk edit that "succeeded" in a way nobody verified.

## Hard rules

1. **Touch one book, touch all seven** — unless the change is genuinely
   book-specific (its palette, its wording, its medallion). Shared components
   live in all seven copies: the quote overlay, Save/Share, the mood bottles,
   the guided tour, the top nav, the close control, the parallax script.
2. **After fixing a bug in one file, grep the other six for the same fault
   before declaring it done.** This is not optional. A `ReferenceError` fixed
   in `index.html` had a silent twin in `rilke.html` that had not surfaced yet
   only because a different code path reached it first.
3. **Stage files explicitly; never `git add -A`.** `.claude/skills/` and
   `.claude/launch.json` are tracked on purpose; `.claude/settings.local.json`
   is personal and gitignored. A blanket add sweeps in local settings and
   stray asset deletions.
4. **webp on-site, PNG as reference.** Serve `assets/*.webp`; keep sources in
   `assets/PNG/`. Compress before committing.
5. **Preserve the custom cursor** (`body { cursor: none }` + `#cursor`) and
   the `:root` palette. Reuse existing custom properties instead of hardcoding
   hex values — each book has its own.

## Verification gates

Run these in order. Each one catches a failure that has actually shipped here.

### Gate 1 — syntax, after any bulk edit

A match count proves a substitution ran. It does **not** prove the result
parses. A greedy regex once ate one closing brace in six of seven files and
every "files patched" counter still read green.

```bash
python3 - <<'PY'
import re, subprocess, os, glob, tempfile
p = os.path.join(tempfile.gettempdir(), "chk.js")
for f in sorted(glob.glob("*.html")):
    html = open(f, encoding="utf-8").read()
    bad = 0
    for b in re.findall(r'<script(?![^>]*\bsrc=)[^>]*>(.*?)</script>', html, re.S):
        with open(p, "w", encoding="utf-8") as fh: fh.write(b)
        if subprocess.run(["node", "--check", p], capture_output=True).returncode: bad += 1
    print(f"{f:<18} syntax_errors={bad}")
PY
```

Every file must report `0`. For CSS edits, also check the block you added has
balanced braces.

### Gate 2 — runtime, per book

Syntax-clean code still throws. Run the real handler and catch what it throws;
never stub the API under suspicion — stubbing `navigator.share` once hid the
exact broken path and produced false passes for several rounds.

In the preview, for each book: open a quote, then exercise Save and Share while
listening for errors.

```js
window.addEventListener('error', e => console.log('THREW:', e.message, e.lineno));
document.querySelector('.action-btn').click();
// then: saveBtn.click(), shareBtn.click() — expect a toast from each, no throw
```

### Gate 3 — read the diff

Before committing, `git diff` and actually read it. The brace-eating regex was
caught here, not by any counter.

### Gate 4 — live, after deploy

```bash
for f in "" alice.html dune.html gatsby.html prince.html meditations.html rilke.html 1984.html; do
  curl -s -H 'Cache-Control: no-cache' "https://downtherabbithole.fyi/$f" | grep -c "<marker you just added>"
done
```

Pages sends `cache-control: max-age=600`, so a stale tab can serve the old
page for ten minutes. When a user reports "still broken" right after a deploy,
suspect cache before suspecting code — have them hard-refresh, and on iOS
clear website data or use a private tab.

## Deploy

```bash
git add <the .html files>            # never -A, never .claude/
git commit -F - <<'EOF'
...
EOF
git pull --rebase --autostash origin main   # a phone session may have pushed
git push origin main
gh run watch "$(gh run list --branch main --limit 1 --json databaseId --jq '.[0].databaseId')" --exit-status
```

Then Gate 4. Push to `main` deploys via `.github/workflows/pages.yml`; there is
no staging. `CNAME` pins the domain — never delete it.

## Per-book differences that break naive bulk edits

Do not assume the seven files are identical. Known divergences:

| Thing | Divergence |
| --- | --- |
| Close button id | `closeBtn` everywhere except `prince.html`, which uses `closeQuote` |
| Copy fallback fn | `copyToClipboard`, except `alice.html` and `rilke.html` use `copyFallback` |
| Hero container | `.hero-container`, except `alice.html` `.cat-container` and `rilke.html` `.rilke-hero` |
| Float keyframe | `floatHero`, except `alice.html` uses `floatCat`; amplitudes differ |
| Close glyph size | 28–48px, all different — anything derived from it must be per-book |
| Title markup | `prince.html` has no `.headline` element |
| JS style | `dune.html` is ES5-ish (`var`, `function`); the rest use `const`/arrow |
| Palette | Every book has its own `:root` names (`--spice`, `--gold`, `--candle`, …) |

When a bulk edit depends on any of these, drive it from an explicit per-file
config and assert each substitution landed exactly once — do not let a silent
zero-match pass.

## Adding (or removing) a book

The book count is baked into a lot of places. This list already went stale once:
`CLAUDE.md` said "five books" and omitted `dune.html` for weeks after Dune
shipped. Work through all of it.

**In every existing book page:**

1. `.lib-row` — the library dropdown nav (the footer is just a single
   "Back to the surface" link now; nothing to add there)

**In `index.html` (the library):** a full world section — the `<section>`
markup (wash, panel, art, copy, quote, Enter pill, keep-falling chain), a
`WEATHER` entry matching the book's own ambient particles, a `BURSTS` entry
for its click particles, and a rail dot with the book's accent. The
second-to-last world's "Keep falling" points at the new one; the new last
world carries `.surface-return`.

Verify the dropdown counts match afterwards:

```bash
for f in alice.html 1984.html gatsby.html prince.html meditations.html rilke.html dune.html; do
  printf "%-18s lib-row:%s\n" "$f" "$(grep -c 'class="lib-row' $f)"; done
```

**The new page needs all of this** — it is easy to ship a book missing half of
it, because nothing visibly breaks:

- Social meta block (description, canonical, `og:*`, `twitter:*`) and a
  1200×630 card in `assets/og/`, composited onto the book's own background
  colour (flattening transparent art to JPEG turns the corners white)
- Cloudflare analytics beacon before `</body>`
- Parallax script, with the book's hero selector, and its float keyframe
  rewritten to `translate(var(--px, 0px), var(--py, 0px))`
- Close hint: its own `CLOSE_HINT_KEY` slug, its own label wording, the flash
  CSS in the book's accent
- Tap-anywhere dismiss, dialog focus management, `role`/`aria-modal`/`aria-label`
- Press feedback block and the 44px close hit area
- `:focus-visible` ring in the book's accent
- Hover-neutralise block for coarse pointers
- `--text-muted` must clear 4.5:1 against the book's own background — solve it,
  don't eyeball it
- Share/Save card renderer with the book's title treatment, its
  `document.fonts.load()` guard, and the window-load retry if that font is
  deferred off the render path

**Site-wide files:**

- `sitemap.xml` — add the URL
- `404.html` — add the link, **and the copy says "seven"**
- `CLAUDE.md` — the file table and any count in the prose
- This skill — it says "seven" in several places

## When something "used to work"

Diff against the last-working commit **before** forming a hypothesis.

```bash
git log --oneline -S'<symbol>' -- <file>    # find when it changed
git show <commit>^:<file> | ...             # read the version that worked
```

Restore what worked, then fix the actual defect. Do not redesign the feature
as a fix — that has cost real time here.
