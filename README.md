# Glyph

Load a font file and it draws every glyph from the outlines it parsed out of the file.

https://bxzex.github.io/glyph/

It reads TTF and WOFF 1 directly. That covers cmap, hmtx, loca and glyf (composite glyphs included), plus GPOS kerning. You can inspect any glyph's points and curves, or type in the tester and compare the result with the browser rendering the same file.

To check it, I compared advances, kerning and rendered pixels against the browser on five Google fonts. Kerning matched on every pair except one, `tt` in Space Grotesk, which the browser turns into a ligature before kerning even runs. A few characters in Fraunces don't match because the browser swaps them for alternates, and Glyph doesn't do that. The page lists those.

WOFF2 and CFF fonts aren't supported yet.
