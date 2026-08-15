---
name: proposal-from-transcript
description: >-
  Turn a sales call transcript into a priced software proposal — a single
  self-contained HTML page with 3–5 of the features the client actually asked
  for, each rated easy/medium/hard/very hard, a fixed total between $15,000 and
  $25,000, and a live control that drops the price by that feature's value when
  one is removed mid-call. Use this skill whenever the user drops a discovery or
  sales call transcript and wants a proposal, a quote, a price, an estimate, or a
  scope of work, says "price this up", "what should I charge them", "write the
  proposal", "turn this call into a quote", or is heading into a closing call and
  needs a number to present. ALSO use when an existing proposal needs re-scoping
  after client pushback. Do NOT use to build a working demo or clickable
  prototype (that's build-prototype), to write a retainer or content-services
  proposal with no software build in it, or for projects the user has already
  said fall outside the $15k–$25k range.
---

# Proposal from Transcript

A proposal presented on a live call is a negotiation instrument, not a document.
It has one job: give the client a number they can say yes to, and a legitimate
way to make that number smaller without either side inventing a discount.
Everything here follows from that.

That reframe kills two failure modes. Listing every request from the call
produces a bloated scope that reads as expensive and unfocused. And a flat
number with no structure leaves only one move when the client flinches — cutting
price for nothing, which teaches them the first number was fake. Structured
difficulty ratings mean a smaller price always costs them something real.

## Core rule

**The price is derived, never chosen.** Read
`references/pricing.md` and follow its arithmetic exactly. Every total lands in
$15,000–$25,000 and every single-feature removal drops it by $1,000–$3,000
because the model is built that way, not because you nudged a number to fit. If
you find yourself picking a total that feels right and back-filling the line
items, stop — you've lost the thing that makes the page defensible on a call.

## Config — edit this block, not the workflow

```yaml
output_dir:  proposals                    # where finished proposals are written
prepared_by: Grow AI Partners             # the "From" line on the page
contact:     you@example.com              # the footer line
logo:        assets/logo@4x.png           # brand mark, top right of every page
```

These are the only installation-specific values in the skill. Anyone can drop
this folder into their own setup and change these four lines instead of hunting
through the instructions — which is the point, since a hardcoded path is the one
thing that breaks a skill the moment it leaves the machine it was written on.

Resolving `output_dir`, in order:
1. If the user names a folder in the request, that wins.
2. Otherwise use the value above, relative to the project root, and create it if
   it doesn't exist.
3. If it can't be created — no such project root, read-only path — write next to
   the transcript instead and say so in chat. Never fail the run over a path.

Filename stays `<client-slug>-<YYYY-MM-DD>.html`.

## The deliverable is a link, not a file

**Every run ends with a published artifact URL.** A local HTML file can only be
presented by the person who has it — it can't be forwarded to the partner who
wasn't on the call, and that's usually who actually signs. So the run isn't
finished when the file is written; it's finished when there's a link.

After writing the file, publish it and hand over the URL:

1. Strip the wrapper — the Artifact publisher supplies `<!doctype>`, `<head>` and
   `<body>` itself. Write a publish copy containing a `<title>`, then the
   `<style>` block, then everything inside `<body>`. Nothing else changes.
2. Publish with the Artifact tool. Title is the client's name plus "Proposal"
   ("Brown Irrigation Proposal"). Everything is already inline, so it passes the
   no-external-requests rule as-is.
3. Report the URL in chat, and say it's private until shared from the page's
   share menu. Don't imply the client can already open it.

Keep both. The local file under `output_dir` stays the source of truth and the
thing that survives; the artifact is the copy that travels. Re-publishing the
same path updates the same URL, so revisions after client feedback don't strand
a link that's already been sent.

## Branding — read `references/brand.md`

Every proposal carries the Grow AI Partners mark in the top-right corner, on its
own row above the headline, and uses the palette sampled from that mark. Both are already in
`assets/template.html` — the logo as a base64 data URI, the colours as CSS
variables — so a normal run needs no branding work at all. Read
`references/brand.md` only when swapping the logo or changing a colour, and
change it there rather than in a finished proposal.

The logo is inlined, not linked, because these pages get emailed, opened from a
Downloads folder, and screen-shared offline. A relative `src` is a broken image
in every one of those cases, which is the worst possible first impression on a
document whose whole job is looking like it cost $20,000.

## Keeping this skill current

**Any feedback on a proposal is a change to this skill, not to that one file.**
If the user says it's too wordy, too narrow, priced wrong, or wants a
section gone, edit `SKILL.md` and `assets/template.html` first, then regenerate
the proposal from the updated template. Fixing only the output means the next run
reproduces the thing that was just rejected, and the same note has to be given
twice.

Say in chat which file you changed, so it's clear the fix is permanent.

Standing notes from feedback so far:
- **Short beats complete.** This page has been cut down twice in review. Default
  to fewer words and fewer sections; if something feels missing, it'll be asked
  for.
- **Wide table, not a narrow column.** Scope reads as a table with the price
  column on the right, never as stacked paragraphs.
- **Three columns, ruled.** Feature / Difficulty / Price are separated by
  vertical hairlines that run down through the base and total rows. Without them
  the table reads as one run of text and the price column stops being scannable.
- **Difficulty pills are all one width, centred.** Ragged, variously-sized pills
  were the first thing noticed on review. `.tier` has a fixed `width` for exactly
  this reason — don't switch it back to shrink-to-fit padding.
- **The logo gets its own row, right-aligned.** That only works because the file
  is cropped to the mark. The complaint about "a ton of white space" was never
  about the row; it was the untrimmed screenshot padding the row out. Fix the
  asset, not the layout.
- **Trim the logo file, don't pad the layout, and ship it at 4x.** Baked-in
  whitespace pushes the page down; a 1:1 screenshot renders soft on retina. See
  `references/brand.md`.
- **The logo band is thin.** Sheet padding above it and the margin below it are
  cut right down, so the top of the page is the logo plus a hairline of air. Page
  top is the one place where the usual generous whitespace reads as an empty
  header instead of as breathing room.
- **No small print in the header.** The validity line lived up there once and was
  cut. Nothing sits beside the mark.
- **No em dashes anywhere on the page.** Not in the pitch line, the scope lines,
  the hover justifications, the Phase 2 list or the "was" line the JS writes.
  Use a full stop, a colon, or a comma. Em dashes are the clearest tell that a
  page was generated rather than written, and this one goes in front of a client
  who is deciding whether $21,000 buys careful work. Check the rendered text
  before publishing, not just the strings you typed.
- **The headline block runs the full width of the sheet.** Nothing shares its
  line, so nothing narrows it. Squeezing the text into a column to make room
  beside the logo was tried and rejected.

## Workflow

**1. Read the transcript in full before writing anything.** Pull out every
request, complaint, current workaround, and named tool. Cheap now, impossible
later — the client's own phrasing is what makes the page land, and you can't
recover it from a summary.

**2. Collapse to 3–5 features.** Almost every call produces eight to fifteen
asks. Merge the ones that are the same feature described twice, then keep the
3–5 that make a shippable thing, biased toward what they said hurts most. If
they mentioned it once in passing, it isn't a headline feature.

Name each feature in the client's words, not product-speak. "Job board sync"
beats "Unified Data Aggregation Layer" — they need to recognize their own idea.

**3. Everything else goes on the Phase 2 list.** Never silently drop a request.
An unlisted ask reads as "they weren't listening"; a listed-but-deferred ask reads
as discipline, and it anchors the next project.

**4. Rate each feature** against the tier rubric in `references/pricing.md`, and
write a one-line justification per rating. The justification is not decoration —
it renders on hover and is what the person presenting says out loud when asked
"why is that one hard?" Rate on implementation risk, not on how excited the
client was.

**5. Run the arithmetic** from `references/pricing.md`: feature values → anchor →
base fee. If the honest scope is bigger than the model's range, say so in chat
before writing the file rather than compressing five hard features into $25k.

**6. Write one line of scope per feature.** One sentence, under about 15 words,
saying what it does in their language. It sits under the feature name in the
table and that's all the room there is.

Brevity here is the point, not a limitation. The page is read in ten seconds on
a screen-share while the seller talks over it — paragraphs of scope compete with
them and get skipped. Anything that needs more explanation than one line gets
explained out loud, and anything genuinely load-bearing that's *excluded* goes in
the "Not included" list rather than buried in prose.

**7. Build the page** from `assets/template.html`. Replace every `{{TOKEN}}`,
repeat the `<tr class="feat">` block once per feature, strip the `REPEAT`
comments (they span two lines — match across newlines or they survive), and write
the file to `output_dir` from the config block above. `{{PREPARED_BY}}` and
`{{CONTACT}}` come from that block too.

`{{LOGO_DATA_URI}}` is replaced with the contents of `assets/logo.b64.txt`
verbatim — one long `data:image/png;base64,…` string, no quotes added, no line
breaks introduced. If that file is missing, regenerate it from `assets/logo@4x.png`:
`python3 -c "import base64;print('data:image/png;base64,'+base64.b64encode(open('assets/logo@4x.png','rb').read()).decode(),end='')" > assets/logo.b64.txt`

Everything lives in one table — feature, difficulty, price, remove — with the
build fee and the total as the last two rows, so the whole commercial picture is
one scan down a single column of numbers. The JS reads the base fee from
`[data-base]` and each feature's value from `data-value` and sums them, so the
total can't drift from the rows as long as `{{TOTAL}}` matches the arithmetic.

**8. Open it and verify** before handing it over: the logo renders (not a broken
image icon), sits top right without a band of dead space under it, and looks
crisp rather than fuzzy, no em dash survives anywhere in the rendered text, the difficulty pills
line up in one centred column of equal-width
chips, total matches the arithmetic,
each removal drops by the right amount and restores cleanly, no two features can
be off at once, hover tooltips read well, and nothing overflows at laptop width.
A broken interaction discovered live on the call is worse than no interaction.

One thing the first build run turned up, already handled in the template — don't
undo it. The total is written by a plain timer backstop as well as the animation
frame loop, because a stale headline number is the one failure that can't be
talked around on a call.

Note also that the delivery time under the table is static text and does not
re-flow on removal. Cutting a feature that owns real calendar time is a verbal
"and that pulls a week out", not something the page claims on its own.

**9. Publish it** as described in "The deliverable is a link" above, and give the
URL.

**10. Report in chat**: the features and tiers, the total, and the removal values,
so the read of the call can be sanity-checked without opening the file.

## Design constraints for the page

Short and wide, scannable in one screen. A document, not a landing page: no hero
image, no marketing copy, one accent colour, generous whitespace. The logo
in the header and the 3px brand rule at the top of the sheet are the only
branding — they mark the page as Grow AI's without turning it into a pitch deck.
The whole thing should fit on about one screen — if it's scrolling for pages, the
copy is too long, not the design too small.

Practical constraints, because it gets screen-shared live: readable at 100% zoom
on a laptop, no layout shift when a feature is removed, works offline as a local
file, prints cleanly. The template encodes all of this; don't add flourishes.

## Removal behavior

One feature out at a time, restorable. The base fee never changes when a feature
is cut — it's the cost of building anything at all, and that's the honest answer
to "why didn't it drop more?" Two features crossed off stops reading as a
concession and starts reading as a different, smaller project, which is why the
page enforces one.

## When NOT to apply

- **A prototype request is not a proposal request.** "Show them something" →
  `build-prototype`. "Tell them what it costs" → this skill. They pair well in
  sequence; don't run this one when the ask was a demo.
- **No software build in scope** (pure retainer, content, ads management) — the
  difficulty model has nothing to bite on.
- **Scope clearly outside $15k–$25k.** If the user has already said it's a $60k
  build or a $4k script, the model doesn't apply. Say so instead of forcing it.
- Under-triggering is the worse error here: a request for "a quote", "a number",
  or "a scope of work off this call" is this skill even when the word "proposal"
  never appears.

## Worked example

Input — messy transcript excerpt:

> **Client:** so right now Maria's basically copy-pasting from the job boards
> into the sheet every morning, takes her like two hours, and half the time the
> role's already filled by the time we see it. And then we've got the candidate
> side, they email us CVs and it just sits in an inbox…
> **Client:** the dream would be, I open one screen and I see every live role and
> who we've got that fits. Oh and eventually invoicing, but that's not urgent.

Features kept (3), rated:

| Feature | Tier | Why |
|---|---|---|
| Live job board sync | Hard | Three external sources, each with its own auth and pagination, deduped and refreshed on a schedule |
| Candidate inbox → structured profiles | Medium | Email ingestion plus CV parsing into fields your team can filter |
| One-screen role and match view | Medium | Single dashboard over both datasets with filtering |

Arithmetic: feature_sum = 2,500 + 1,750 + 1,750 = **$6,000** → anchor =
15,000 + (6,000 − 3,000) × 0.8333 = 17,500 → base = 17,500 − 6,000 =
**$11,500**. Total **$17,500**.

Phase 2 list: invoicing, placement commission tracking, client-facing portal.

On the call: removing the match view drops it to **$15,750**; removing job board
sync drops it to **$15,000** — and cutting the thing that saves Maria two hours a
day is a conversation the client usually talks themselves out of, which is
exactly what the control is for.
