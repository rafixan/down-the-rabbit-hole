# Down the Rabbit Hole

Interactive, atmospheric web reader for classic literature. Live at **downtherabbithole.fyi**
(GitHub Pages). The Alice / Cheshire Cat motif is the site-wide brand ("down the rabbit hole").

## Structure

Each page is a **standalone, single-file HTML document** — embedded `<style>` and `<script>`,
no build step, no shared CSS/JS files, no framework. Editing a book = editing that one `.html`.

| File | Title | Theme |
|------|-------|-------|
| `index.html` | Down the Rabbit Hole | Landing / library, Alice-Cheshire purple |
| `1984.html` | Big Brother is Watching — 1984 | Telescreen red |
| `gatsby.html` | The Green Light — Gatsby | Celestial Eyes, art deco green |
| `prince.html` | Asteroid B-612 (The Little Prince) | Mood bottles (most developed) |
| `meditations.html` | The Emperor's Journal — Meditations | Marcus Aurelius medallion |
| `rilke.html` | No Feeling is Final (Rilke) | Candlelight, Worpswede tree |
| `dune.html` | Fear is the Mind-Killer — Dune | Spice orange, sandworm medallion |

Plus `404.html`, which links to every book.

Every page carries the same library nav **and** footer nav, each listing every book — so
adding one is two edits per existing page, not one. The book count is also baked into
`sitemap.xml`, the `404.html` copy, and this table.

**Adding or removing a book: follow the checklist in
`.claude/skills/book-pages/SKILL.md`.** It enumerates every place that needs updating.
This table itself went stale for weeks after Dune shipped — it said "five books" and
omitted `dune.html`.

## Conventions

- **Custom cursor** — `body { cursor: none }` + a `#cursor` element; preserve when editing.
- **CSS custom properties** — colors live in `:root` (e.g. `--purple-deep`, `--green-eye`).
  Reuse the existing palette rather than hardcoding hex.
- **Fonts** — Google Fonts (IM Fell English, Cormorant Garamond) via `<link>` in `<head>`.
- **Mood bottles** — interactive per-book vignette/animation feature, most fleshed out in
  `prince.html`. Recent work iterates per-book animations (Gatsby shimmer, Alice fade, etc.).
- **Save cards / canvas** — pages render a shareable image card via `<canvas>` (IG story
  format 1080×1920); cards include logo + title and auto-shrink text.
- **Images** — serve `.webp` from `assets/`; PNG sources kept in `assets/PNG/`. Compress
  before committing (a 4 MB Marcus image was cut to 79 KB — keep assets small).
- **Favicons/icons** — `favicon.png`, `favicon-32.png`, `apple-touch-icon.png` at root.

## Run locally

```
python3 -m http.server   # then open http://localhost:8000
```
(matches `.claude/launch.json`)

## Deploy

Push to `main` → `.github/workflows/pages.yml` builds and deploys to GitHub Pages. No staging.
`CNAME` pins the custom domain — don't delete it.
