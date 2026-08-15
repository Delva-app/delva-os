# Design direction

Fold this into every build message. Without an explicit direction, AI builders
default to purple gradients, glassmorphism and stock illustration, which clients
read as generated. A prototype that looks like a real product gets its ideas
taken seriously; a prototype that looks like a template gets everything in it
discounted.

Reference images live in `../assets/design-refs/`. Attach both to every build.

---

## Landing page — Clay's visual approach, Linear's structure

`clay-landing.jpeg`

What to take from Clay: **confidence in colour.** Large saturated brand-colour
blocks, generous rounded corners, a floating pill nav, oversized type, sections
that alternate colour so the page reads as a sequence rather than a scroll.
It commits to a palette and repeats it everywhere.

What to leave: the volume of sections, the bespoke 3D illustration, the density
of nav. A prototype needs one screen's worth of that energy, not twelve.

What to take from Linear: **structure and restraint in layout.** Clear headline,
one subheading, one obvious call to action, then a large product screenshot
immediately. Nothing between the headline and seeing the product.

Note: use Linear's *format*, not its dark palette. Landing pages here are light
or brand-coloured — whatever the client's brand suggests. Dark is a choice, not
the default.

## The headline

Short, declarative, and about the outcome. Clay's is **"Build systems to grow
revenue"** — five words, an outcome, no hedging, no explanation.

Copy that shape. Four to seven words. State what the customer gets.

Never write the problem as the headline. "Stop answering the same question from
the field" was tried on a real build and it is weak: it leads with the pain, it
is long, and it makes the product sound like a complaint. The subheading is
where nuance goes; the headline is a flag planted in the ground.

Never name the owner in the headline. A real build produced "<Owner>'s knowledge,
in the field" — it reads as cute, and it breaks the moment anyone but that one
person looks at the page.

Good shapes: "Solve it in the field." · "Every answer, on site." · "Answers
before the phone call."

## The structure to build

This is a home page, not a splash screen. Eight sections minimum — the failure
mode is three thin sections and a lot of whitespace.

1. Nav — logo left, sign in right
2. **Hero** — headline, one subheading, one primary button
3. **Product shot** — the real app, large, immediately below the hero
4. **The problem** — three or four cards naming what goes wrong today, in the
   client's own words, ideally lifted from what they actually said
5. **How it works** — three numbered steps, each with a small product image
6. **Feature sections** — three or four, alternating brand-colour backgrounds,
   each pairing one benefit with one real screenshot
7. **Who it's for** — one block per user role, named as the client names them
8. **Proof or numbers** — plausible outcome figures, clearly framed as
   illustrative
9. **Closing call to action** — full-bleed brand colour, one button into sign in
10. Footer

Every section that can carry a real product image should carry one. A landing
page for software that never shows the software is the most common way these
come out looking thin.

## App UI — Linear

`linear-landing.jpeg`, product panel

Dense, sidebar-led, restrained. Small tight type, real spacing, one accent
colour used sparingly for status and action. Tables with actual rows. Status
communicated by small coloured dots and text, not big coloured pills.

Dark or light follows the client's brand. Linear's density and discipline is the
reference, not its specific colours.

## Palette

Pull the client's colours from any screenshot, logo or website in `input/`. If
there is nothing to pull from, choose one confident accent appropriate to the
industry and use it consistently.

One accent colour. Neutral greys for everything else. Semantic colours only for
status. Never more than two accents.

## Hard nos

- Purple-to-blue gradients
- Glassmorphism and frosted panels
- Stock illustration and generic 3D blobs
- Emoji as interface icons
- Rainbow-coloured category tags
- Centred body text
- More than two typefaces
