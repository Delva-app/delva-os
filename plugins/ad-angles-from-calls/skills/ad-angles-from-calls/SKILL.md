---
name: ad-angles-from-calls
description: >-
  Mine a body of recorded calls — sales, discovery, onboarding, client check-ins
  — for the pain points buyers state in their own words, rank those pains by how
  often they come up and how much they hurt, and turn the top ones into static
  and video ad concepts with copy. Every pain is evidenced by verbatim quotes
  with speaker and call attribution; nothing is invented. Output is a single
  self-contained HTML report. Use this skill whenever the user wants ad angles,
  ad ideas, hooks, creative concepts, messaging, or copy grounded in what
  customers actually say, asks "what do our clients complain about", "what
  should our ads say", "pull the pain points out of our calls", "mine our
  transcripts", "what angles should we test", or wants to refresh creative that
  has fatigued. ALSO use when someone is about to write ads, a landing page, or
  a VSL for a business that has call recordings sitting unused. Do NOT use to
  price or scope a project from a call (that's proposal-from-transcript), to
  build a prototype from requirements (build-prototype), to cut or caption
  footage (ad-variant-matrix), or to launch ads into an ad account
  (meta-campaign-builder) — this skill stops at concepts and copy.
---

# Ad Angles from Calls

Most ad copy fails because it is written from the seller's side of the desk. The
agency guesses at the pain, phrases it in category language, and ships a hook
nobody recognises as their own problem. Meanwhile the exact sentence that would
have worked was said out loud by a prospect on a call three months ago and is
sitting in a transcript nobody reopened.

This skill's whole job is to close that gap: the winning hook already exists as
a spoken sentence, and the work is retrieval and ranking, not invention. Which
means the failure mode to fear is not a boring concept — it's a *plausible* one
that no one actually said. A pain point with no quote behind it is a guess
wearing evidence's clothes, and it will get tested, lose, and be blamed on the
channel.

## Core rule

**Every pain point must carry at least one verbatim quote, and quotes are
transcribed exactly — filler words, false starts, bad grammar and all.** Do not
clean up "like, we're just kind of drowning in it honestly" into "we are
overwhelmed." The unpolished version is the deliverable; it is the phrasing that
will out-convert whatever you would have written, precisely because it is how
the buyer's own head sounds. If you cannot find a quote for a theme you believe
in, list it in *Unevidenced hypotheses* at the bottom of the report rather than
promoting it into the ranking.

## Config — edit this block, not the workflow

```yaml
subject:      Grow AI              # whose ads these are (a client name when run for a client)
output_dir:   reports              # where the HTML report is written, relative to project root
lookback:     all                  # how far back to pull calls; default is every call ever recorded
min_calls:    8                    # below this, say the sample is thin rather than ranking confidently
top_pains:    6                    # how many pains get ad concepts built on them
brand_color:  "#111111"            # accent colour of the report
```

These are the only installation-specific values. Anyone — Grow AI, or Grow AI
running this for one of their own clients — drops this folder into their setup
and changes these six lines instead of editing instructions, which is what keeps
the skill portable. When the user names a client in the request, that overrides
`subject` for the run; write the report to `output_dir/<subject>-ad-angles-<date>.html`.

## Workflow

### 1. Find the calls before asking for them

Never open by asking the user to upload transcripts. Most businesses already
have hundreds of calls recorded and don't think of them as an asset. Probe, in
this order, and stop at the first source with real volume:

1. **A dedicated meeting-recorder MCP** — Fathom, Gong, Grain, Fireflies, Otter,
   Read. With Fathom: `list_meetings` across the lookback window, then
   `get_meeting_transcript` per call. Best case — already speaker-labelled.
2. **Google Meet's Gemini note taker, via the Google Drive MCP.** See
   *Finding Gemini transcripts* below — this is the most common source and the
   easiest one to under-collect from.
3. **A local folder** of transcripts (`.md`, `.txt`, `.vtt`, `.srt`, `.json`) —
   the project, `input/`, `transcripts/`, `calls/`, and anywhere the user's
   CLAUDE.md points for call material.
4. **Notion / other doc stores** — titles that look like calls: "discovery",
   "call", "intro", a person's name plus a date, Zoom/Meet auto-titles.
5. **Only then ask** — concretely. "What do you record calls with?" gets a
   useful answer; "please provide transcripts" gets silence.

### Finding Gemini transcripts

Gemini's note taker has no API of its own. Google Meet writes its artifacts into
the **meeting organiser's** Drive as ordinary Google Docs, so the Drive MCP
reads them like any other document. Three artifacts may exist per call,
depending on what the host switched on:

| Doc | Typical title | Use it for |
|---|---|---|
| Transcript | `<Meeting> - <date> - Transcript` | **Quotes.** Speaker-labelled, verbatim. |
| Gemini notes | `<Meeting> - <date> - Notes by Gemini` | Finding that a call happened. Never for quotes. |
| Recording | `<Meeting> - <date>` (`.mp4`) | Last resort — needs transcribing first. |

**Never quote from the Gemini notes doc.** It paraphrases in Gemini's voice, and
a paraphrase presented as the buyer's own words is the exact failure this skill
exists to prevent. If a call has notes but no transcript, treat that call as
evidence a pain exists, and go find the quote elsewhere.

Search in this order, because each step catches calls the previous one misses:

1. **Locate the folder, then enumerate it.** Find `Meet Recordings` (older
   Workspace tenants use `Meet Recordings`; some use `Meet transcripts`), take
   its id, and list children with `parentId = '<id>'`. Enumerating the folder
   beats title search, because a doc someone renamed still lives in the folder.
2. **Title sweep**, to catch anything moved out of that folder:
   `title contains 'Transcript' or title contains 'Notes by Gemini'`.
3. **Full-text sweep**, to catch renamed *and* moved docs: Meet transcripts open
   with a boilerplate attendee/date header, so `fullText contains 'Transcript'`
   combined with a known client name surfaces strays.
4. **Shared drives and other hosts** — see below.

**The organiser trap, which is the whole ballgame for "every client call."** The
transcript lands in the Drive of whoever *hosted* the call, not the company's.
In an agency where different people run different accounts, one connected Google
account sees only that person's calls, and the search will look like it
succeeded — it returns real transcripts, just a fraction of them. So before
analysing, always establish whose calls you're holding:

- Check whether the artifacts live on a **shared drive** the whole team can see
  (`sharedWithMe = true`, plus shared-drive folders). If they do, you likely have
  everything.
- If they're in one person's My Drive, ask directly: **who else at the company
  hosts client calls?** Then get those accounts' recordings shared, or say
  plainly in the report that coverage is limited to the hosts you could read.

**Reconcile against the calendar.** If a Google Calendar MCP is connected, list
past events across the lookback window that had a Meet link and external
attendees, and diff that list against the transcripts you found. This is the
only way to know what's *missing* rather than what's present, and it routinely
surfaces two things worth reporting: calls where transcription was never turned
on, and whole categories of call hosted by someone whose Drive you can't see.
State the coverage number in the report's method note — "84 of 112 external
calls had transcripts" is a materially different document from an unqualified
ranking.

### When the transcripts exist but you can't reach them

This is the common case, and it is not the same as having no transcripts. Meet
and Gemini write to the **meeting organiser's** Drive, so if the subject's calls
were hosted on their Workspace and this Claude is connected to a different
Google account, every search above returns nothing while hundreds of transcripts
sit one permission away. Don't report "no calls found" — report the access gap,
and give them the two fixes:

- **Share the folder.** Ask the subject to share their `Meet Recordings` folder
  (and any transcript docs outside it) with the Google account this Claude is
  connected to. Name that account explicitly in the ask.
- **Connect their account.** Add the Google Drive connector under a Workspace
  account that already has the recordings — cleanest when the skill is being run
  by the subject on their own calls.

If nothing is connected at all, say which connector to add before you go
further, rather than falling back to whatever stray transcript happens to be on
disk. One local file is a sample of one, and a ranking built on it is worse than
no ranking, because it looks like data.

### Then take all of them

Default to **every client call available**, not a recent slice or a sample —
that's why `lookback` defaults to `all`. Frequency is half the score, so a
truncated corpus doesn't just give you less, it silently reweights the ranking
toward whatever the last quarter happened to be about. If the volume is genuinely
too large to read in one pass, process in batches and accumulate the tally
across batches; never substitute a sample and present it as the whole.

Report what you found before analysing: how many calls, date range, call types,
and anything you know exists but couldn't read. If the count is under
`min_calls`, say so plainly and continue anyway — a thin sample gives a
directional read rather than a ranking, and the user needs to know which one
they're holding.

### 2. Read for pain, not for summary

Work through every call. You are hunting for four things, and only the first is
obvious:

- **Stated pain** — "we're spending way too much time on X."
- **Cost of the pain** — money, hours, headcount, missed deals, personal stress.
  This is what converts a mild annoyance into a buying trigger, and it's usually
  said as an aside, not as a complaint.
- **Failed prior attempts** — "we tried Y and it didn't stick." Gold for ad
  angles, because it names the competitor or approach the reader has already
  written off and positions against it without you having to.
- **The words themselves** — jargon, metaphors, the way they describe their own
  situation. Collect these even when they're not attached to a pain.

Skip the seller's side entirely. On a sales call the rep will articulate pain
better and more fluently than the prospect, and it's contaminated — it's the
existing marketing being played back. If a transcript has speaker labels, filter
to the buyer. If it doesn't, judge by role from context and note the
uncertainty.

### 3. Score and rank

For each distinct pain, record:

- **Frequency** — number of *distinct calls* it appears in (not mentions; one
  person ranting five times is one call).
- **Severity 1–5** — judged from the language, not the topic. Anchor it:
  - 5 — existential or emotional: "it's killing us", "I can't keep doing this",
    quantified loss, or it's the reason they took the call.
  - 4 — costly and recurring, with a number attached.
  - 3 — real friction, described flatly, no cost named.
  - 2 — mentioned when prompted by the rep.
  - 1 — agreed with a leading question. Barely evidence.
- **Score** = frequency × severity. Rank descending.

Severity is scored from *how they said it*, because a mild-sounding topic
delivered with heat outsells a dramatic topic delivered flatly — the heat is the
buying signal, the topic is just the subject line. Where two pains are within a
few points, break the tie toward the one with the more quotable line, since a
ranking's only purpose here is deciding what gets made first.

Prompted mentions (severity 1–2) get counted but flagged, because a pain that
only ever appears after the rep raised it is the seller's hypothesis echoed
back, and ads built on it test the agency's assumptions rather than the market's.

### 4. Build ad concepts for the top `top_pains`

Per pain, produce **two static concepts and two video concepts**, and derive
each one from the quote rather than from the theme. Concretely: the headline
should be recognisably close to something a real person said. A pain summarised
as "inefficient reporting workflows" yields nothing; the same pain quoted as
"I'm rebuilding the same deck every Monday at 11pm" is the ad.

**Static concept** — angle name, headline (≤10 words, buyer's phrasing),
subhead, visual direction in one line, and the quote it came from.

**Video concept** — angle name, hook line for the first 3 seconds (spoken, not
written-on-screen), a 4–6 beat structure, CTA, and the quote it came from.

Vary the *mechanism* across concepts, not just the wording — problem-agitate,
mirrored objection, before/after, a customer's own words on screen as the whole
ad, a demo against the specific failed prior attempt. Two rewrites of the same
idea read as coverage but test as one ad, so the matrix should give the media
buyer genuinely different things to learn from.

Anything written as the subject's own voice (LinkedIn-style scripts, first-person
copy for Aidan or a named founder) goes through the relevant voice skill —
`aidans-voice` for Aidan. Load it before drafting, not after.

### 5. Write the report and hand it over

Fill `assets/report-template.html` and write it to the resolved path. It is a
single self-contained file — no external CSS, JS, fonts, or images — so it can
be emailed, dropped in Slack, or opened on a phone by a client with no setup.
Then in chat: the top three pains in one line each with their score, the file
path, and anything surprising (a pain that only shows up in lost deals, a phrase
repeated across unrelated companies).

## Output format

Use `assets/report-template.html` exactly. Sections, in order:

1. **Header** — subject, date, calls analysed, date range, source.
2. **Top three at a glance** — three cards, each: pain in the buyer's words,
   score, one quote.
3. **Full pain ranking** — table: rank, pain, frequency, severity, score, and an
   expandable quote list with speaker + call attribution per quote.
4. **Ad concepts** — one block per top pain, containing its two statics and two
   videos in the shapes above.
5. **Language bank** — the buyer's raw phrases, verbatim, ungrouped. Copywriters
   raid this section more than any other, which is why it stays unedited.
6. **Anti-pains** — things the calls show are *not* a problem, with the quote
   proving it. See below.
7. **Unevidenced hypotheses** — themes with no quote behind them, clearly marked
   as not derived from the calls.
8. **Method note** — how severity was scored, which sources were read, calendar
   coverage (`X of Y external calls had transcripts`), whose Drives were and
   weren't reachable, and a sample-size caveat if under `min_calls`.

### Anti-pains

Record every place the buyer says something is already handled, and quote it.
These are as commercially useful as the pains and much rarer, because a buyer
will volunteer a complaint unprompted but only names a working system when
asked to consider replacing it.

An anti-pain kills a whole family of ads before the spend. When someone says
their scheduling and invoicing "works really well", a headline about scheduling
doesn't merely underperform — it tells that reader the ad isn't for them, in the
first three seconds, and the rest of the creative never gets seen. Agencies
reliably discover this after the test rather than before, because nothing in a
normal pain-mining pass has a place to put "he told us not to bother."

Watch for the two forms: the flat dismissal ("that side works really well"),
and the deferral ("months or years down the road"), which is the same signal
said politely. Both belong in the section.

## Calibration

**Under-use** looks like ranking six pains that are really one pain described
six ways, or producing concepts that are just the pain restated as a headline.
Merge duplicates aggressively; the ranking is worthless if the top three
overlap.

**Over-use** looks like inflating the list with severity-1 prompted mentions to
hit `top_pains`, or writing ad concepts for pains that appeared in one call.
When there genuinely aren't `top_pains` well-evidenced pains, ship fewer and say
why. Over-use is worse here: a short honest report gets acted on, a padded one
sends a media budget after noise.

Don't editorialise about the subject's sales process. The user asked for ad
angles; observations about how the reps handle objections belong in a sentence
in chat at most, not in the report.

## Example

**Input:** user says "we've got a bunch of calls in Fathom, can you find what
our clients actually complain about and give me some ad ideas" — 23 calls found,
Jan–Aug, mostly discovery.

**Excerpt of the ranking:**

| # | Pain | Freq | Sev | Score |
|---|------|------|-----|-------|
| 1 | Can't tell which ads are actually working | 17 | 5 | 85 |
| 2 | Agency sends reports nobody can act on | 12 | 4 | 48 |
| 3 | Burned by a previous agency's 6-month contract | 9 | 5 | 45 |

Quotes behind #1 (verbatim, as spoken):

> "Honestly I've got like four dashboards open and they all say something
> different, so I just… I kind of go with my gut at this point." — Marcus T.,
> discovery, 2026-03-11

> "I don't know what's working. I know what's spending." — Priya N., discovery,
> 2026-06-02

**One static built from it:**

- Angle: *the four-dashboard problem*
- Headline: "I know what's spending. I don't know what's working."
- Subhead: One number per campaign. Updated daily. That's the whole report.
- Visual: four cluttered dashboard screenshots, greyed, collapsing into one
  large number.
- From: Priya N., discovery, 2026-06-02

Note what the headline did: it is the quote, near enough verbatim, with nothing
added. That is the normal case, not a lucky one — when a line survives being
said out loud under real frustration, it does not need improving.
