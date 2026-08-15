# Brand — Grow AI Partners

Everything here is already wired into `assets/template.html`. This file exists so
the values can be changed in one place, and so a future logo swap is a
five-minute job instead of a hunt through CSS.

## The mark

| File | What it is |
|---|---|
| `assets/logo-source.png` | The trimmed original, 142 × 47 — keep, never edit |
| `assets/logo@4x.png` | 568 × 188, resampled and sharpened. **This is the one that ships** |
| `assets/logo.b64.txt` | `logo@4x.png` as a `data:image/png;base64,…` string |

**Why 4x.** The source was a screenshot, so at 1:1 it renders soft on any retina
display — and a fuzzy logo on a $21,000 quote is the kind of detail that makes a
whole document feel cheap. Shipping it at four times the display size means the
browser is always downsampling, which is sharp, instead of upscaling, which is
not. Resampling can't invent detail the screenshot never had; if a vector or
high-resolution original ever turns up, use it and drop the upscale step.

**Trimmed is the important word.** The original was a 278 × 76 screenshot, half
of it white margin. Sized by height in CSS, that margin became a band of dead
space above the headline and the header looked broken. Any replacement gets
cropped to its content before it goes in — padding belongs to the layout, which
can adjust it, not to the image file, which can't.

The template's `{{LOGO_DATA_URI}}` token takes the contents of the `.b64.txt`
file. Inlined, never linked — proposals get emailed, downloaded and screen-shared
offline, and a relative path is a broken image in all three.

**Placement:** top **right**, 46px tall, on its own `.brandbar` row above the
headline (40px and left-aligned on narrow screens). The row is deliberately
tight: 22px of sheet padding above it and 14px below, so the band reads as the
logo with a thin margin rather than a header. The sheet's generous 60px side
padding and 64px bottom stay as they are; only the top is pulled in. The headline block below it
runs the full width of the sheet.

A row of its own is only affordable because the file is cropped: the row is as
tall as the mark and not a pixel more. With the original untrimmed screenshot the
same layout left a visible band of dead space, which is what got it moved
alongside the headline for a while. Trimming the asset fixed the real problem, so
the row came back. Height is set on the trimmed file, so 52px is 52px of actual
mark.

Nothing joins the mark on the right — no validity line, no tagline, no phone
number. A header that accumulates small print is how a proposal starts looking
like a flyer.

### Swapping the logo

```bash
cd assets
cp /path/to/new-logo.png logo-source.png
# 1. crop away any baked-in white or transparent margin, then upscale
python3 - <<'EOF'
from PIL import Image, ImageFilter
im = Image.open('logo-source.png').convert('RGBA'); px = im.load(); w, h = im.size
ink = lambda p: p[3] > 10 and not (p[0] > 243 and p[1] > 243 and p[2] > 243)
xs = [x for x in range(w) for y in range(h) if ink(px[x, y])]
ys = [y for y in range(h) for x in range(w) if ink(px[x, y])]
im = im.crop((min(xs), min(ys), max(xs) + 1, max(ys) + 1))
im.save('logo-source.png')
# 2. ship it at 4x so the browser downsamples instead of upscaling
w, h = im.size
big = im.resize((w * 4, h * 4), Image.LANCZOS)
big = big.filter(ImageFilter.UnsharpMask(radius=2.4, percent=115, threshold=2))
big.save('logo@4x.png')
EOF
# 3. re-inline it (no trailing newline — it lands inside an src attribute)
python3 -c "import base64;print('data:image/png;base64,'+base64.b64encode(open('logo@4x.png','rb').read()).decode(),end='')" > logo.b64.txt
```

Then re-sample the palette below if the new mark's colours differ, and update
both this file and the `:root` block in the template.

## Palette

Sampled directly from the logo pixels, so the page and the mark can't drift apart.

| Token | Hex | Where it comes from | Used for |
|---|---|---|---|
| `--accent` | `#1b2ce8` | Electric blue in the arrow mark | Eyebrow, links, bullets, hover states, focus rings |
| `--accent-2` | `#4a90de` | Light blue, gradient end of the mark | Right end of the top rule only |
| `--navy` | `#26357b` | The "GROW AI" wordmark | The total figure and the rule above it |
| `--ink` | `#171d33` | Navy-black, derived from the wordmark | Body headings, feature names |
| `--ink-2` | `#4a5364` | — | Body copy |
| `--ink-3` | `#7b8592` | Close to the grey in "PARTNERS" | Labels, scope lines, footer |
| `--rule` | `#e2e5ef` | — | Table rules and borders, faintly blue-tinted |
| `--wash` | `#f5f6fb` | — | Page background behind the sheet |

Two rules that matter more than the hexes:

**One accent, used sparingly.** `--accent` is for the things you want a
client's eye to land on — never for large fills. The blue on this logo is
saturated; a proposal with blue panels reads as a template, not a quote.

**`--accent-2` only appears in the top rule.** It exists to echo the mark's
gradient in a 3px line. It is not a second accent colour and shouldn't spread to
buttons or text.
