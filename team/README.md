# Leadership headshots

The Company page (`/company`) loads each leader's photo from this folder by
filename. Drop a square-ish image at each path below. Until a file exists, the
page shows an initials tile as a graceful fallback — nothing breaks.

Expected files (PNG; edit the `slug` in `src/routes/company.tsx` if you prefer
another extension):

- `mirella-di-girolamo.png`  — Mirella Di Girolamo · Chief Executive Officer
- `michele-sciabarra.png`    — Michele Sciabarra · Founder & Director
- `michele-manzani.png`      — Michele Manzani · Chief Technology Officer
- `massimo-tozzi.png`        — Massimo Tozzi · Chief Revenue Officer
- `daniele-evangelisti.png`  — Daniele Evangelisti · Chief Marketing Officer

Tips:
- Square crops look best (the grid uses `aspect-square object-cover`).
- Photos are shown grayscale and turn to color on hover — that's intentional
  styling, not a broken image.
