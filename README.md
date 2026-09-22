# Glyph

A font file parser in the browser. It reads TrueType and WOFF files byte by
byte, draws every glyph from the outlines it decoded, applies the font's own
GPOS kerning, and checks all of it against the browser's text renderer.

Live: https://bxzex.github.io/glyph/

## What it reads

- **Containers**: raw TrueType/OpenType, and **WOFF 1**, whose tables are
  individually zlib-compressed. Those are inflated with `DecompressionStream`,
  and each table must come out at its declared size. WOFF 2 needs Brotli plus
  a glyf transform, so it is rejected with a clear message rather than
  half-read.
- **`head`, `hhea`, `maxp`, `hmtx`, `OS/2`, `name`**: units per em, vertical
  metrics, advances and side bearings (including the short tail of `hmtx`
  after `numberOfHMetrics`), weight, x-height, cap height and the name
  records, preferring Windows Unicode.
- **`cmap`** formats 4 and 12, including format 4 segments that go through
  `idRangeOffset` into the glyph ID array
- **`loca` + `glyf`**, short and long offsets. Simple glyphs decode the flag
  run-length encoding and the delta-encoded coordinates with their short and
  same-as-previous forms. **Composite glyphs** are read recursively with word
  or byte arguments and a uniform scale, an x/y scale or a full 2×2 transform,
  so an `é` is built from an `e` and an acute.
- **Quadratic outlines**: TrueType contours are B-splines, where two
  consecutive off-curve points imply an on-curve point at their midpoint. The
  path builder handles that, along with contours that start off-curve and
  contours with no on-curve points at all.
- **`GPOS` kerning**: the `kern` feature's lookups, including type 9
  extension lookups, PairPos format 1 (explicit pairs) and format 2 (class
  pairs), with Coverage and ClassDef in both formats and value records sized
  from their format bits

The inspector shows on-curve and off-curve points, the control polygon, point
numbers, metrics lines and the advance box. The type tester sets text entirely
from the parsed outlines and marks each kerning adjustment under the pair it
moved. Below it, the browser renders the same bytes loaded as a `FontFace`.

## Checked against the browser

Every font is also loaded into the browser's own engine, and three things are
compared automatically:

| | Fraunces | Playfair 700 | Inter | Space Grotesk | DM Serif |
|---|---|---|---|---|---|
| Advances equal to `measureText` | 209 + 6 shaped | 218/218 | 219/219 | 219/219 | 213/213 |
| Kerned pairs equal to the browser | 2,401/2,401 | 2,809/2,809 | 2,809/2,809 | 2,808/2,809 | 2,809/2,809 |
| Outline pixels agreeing exactly | 100.00% | 98.35% | 100.00% | 100.00% | 98.40% |
| Within one pixel of hinting snap | 100.00% | 99.996% | 100.00% | 100.00% | 99.996% |

Kerning is measured as the width of a pair with `fontKerning` on minus the
width with it off, which isolates GPOS from everything else.

Outlines are compared by rasterising each glyph both ways at about 340px. A
pixel counts against the parser only when one side has ink and the other has
paper.

Getting to a fair comparison took three corrections, and each one came from
looking at a picture rather than trusting a number:

1. **The first run said 85%.** Rendering the worst glyph showed the browser's
   letters one pixel fatter all round. Chrome rasterises text with a different
   gamma than paths, so a hard 50% threshold on anti-aliased edges is mostly
   noise at small sizes. The check now runs large and only counts ink against
   paper.
2. **Fraunces' `&` agreed 30%.** It was not an outline bug. The browser was
   drawing a different glyph, because Fraunces substitutes alternate forms of
   `& h m n s ñ` through GSUB by default and this parser does not shape. Those
   characters are detected by their advance mismatch, listed on the page and
   left out of the pixel test. Space Grotesk's one kerning miss is the same
   thing: `tt` becomes a ligature before kerning ever runs.
3. **Playfair and DM Serif stayed near 98%.** A diff image of `=` showed one
   row of pixels on one edge of a thin bar. That is FreeType's light hinting
   snapping horizontal edges to the pixel grid. Allowing exactly that one-pixel
   vertical snap brings both to 99.996%. The page reports the exact figure and
   the snapped figure side by side.

## Notes

One HTML file. No libraries, no build step. Fonts come from
[Fontsource](https://fontsource.org) on jsDelivr, or drop your own file on the
page. CFF (PostScript) outlines are recognised and explained but not drawn.

Built by [bxzex](https://bxzex.com).
