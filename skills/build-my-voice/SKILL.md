---
name: build-my-voice
description: >-
  Build a durable voice file for someone from their own words: meeting
  transcripts and AI chat history (Claude, ChatGPT, or other exports).
  Extracts sentence patterns, words they actually use, words they'd
  never use, how blunt or hedged they are, how they open and close a
  message, and a bank of verbatim quotes — then files it somewhere
  durable and wires it into their CLAUDE.md so future drafts pull
  through it. Use this skill whenever
  the user invokes /build-my-voice, asks to "build my voice file",
  "capture how I write", "make AI sound like me", or wants to refresh
  an existing voice file with new transcripts. Also run as a step from
  build-my-os when setting up a new OS from scratch. Do NOT use this to
  write a single piece of content in someone's voice (that's the job of
  whatever skill is drafting the content, reading the voice file this
  skill produces) — this skill only builds and maintains the file.
---

# Build my voice

A CLAUDE.md that knows the business but not how someone writes or talks
is half done. This skill builds the missing half: a voice file
extracted from things they've actually said, not a guess at their tone.

## What makes a voice file work

**Quotes, not adjectives.** This is the whole skill. A voice file that
says "direct and conversational, avoids jargon" is worthless — every
person on earth thinks that describes them, and no agent can write from
it. A voice file that says:

> Corrections are terse and land the fix immediately, no cushioning:
> "no not that." "drop the word really." "you don't need the how just
> the what."

is usable, because the next agent can pattern-match against real
sentences. Every claim you make about how someone writes must be
followed by their actual words proving it.

Rule of thumb: **if a section has no quotes in it, it isn't finished.**
Aim for 30+ verbatim quotes in the finished file, pulled from different
contexts. A description tells an agent what to aim for. A quote shows it.

**Second rule: capture what they refuse, not just what they do.** The
banned words, the phrasings they've killed in a draft, the tone they
keep rejecting. Most of the value in a voice file is negative space —
"never uses em dashes," "kills 'excited to announce' on sight" — because
that's what stops an agent producing generic copy.

## Core rules

- **Depends on real uploads — never fabricate a voice from a guess.** If
  they have nothing to upload yet, say so plainly and stop rather than
  inventing a plausible-sounding voice. A made-up voice file is worse
  than none, because everything downstream trusts it.
- **Chat exports are the highest-signal material available.** Months of
  someone thinking out loud, asking questions, and reacting to drafts in
  their own words beats a handful of polished sent emails. Polished
  material has usually had their voice edited out of it.
- **Quote verbatim, including the typos.** Do not clean up their
  spelling, capitalization, or grammar in the quote bank. "systemate
  isnt bad but its not easy to read" carries more signal than a
  corrected version, because the lowercase and missing apostrophe are
  part of how they write.
- **File it somewhere durable and link it.** The output only pays off if
  future drafting sessions actually read it.

## Workflow

1. **Ask for uploads.**
   - Their AI chat history — Claude data export, ChatGPT export, or any
     other assistant they use regularly. This is the best source; ask
     for it first and by name.
   - Any meeting transcripts they have (calls, sales conversations,
     internal meetings — whatever exists).
   - Anything they've written and published as themselves: sent emails,
     DMs, social posts.

2. **Read everything, hunting for the eight things below.** Collect
   quotes as you go — you cannot write this file from memory of the
   material, only from lines you've actually pulled out.

   1. **Registers.** Most people have more than one. Typed vs. dictated
      is the most common split, and it matters enormously: dictated
      speech is full of run-ons and filler that they would never want in
      written content. Name each register, say which one content should
      be written in, and say so explicitly.
   2. **Sentence structure and rhythm.** Length, fragments vs. full
      sentences, how clauses get joined, whether they use subordinate
      clauses or just chain with commas and "and."
   3. **Vocabulary.** Their recurring words. The ones that show up over
      and over are the fingerprint. Also register: do they swear, use
      jargon, use slang?
   4. **Banned words and phrases.** Anything they've explicitly told an
      AI not to do, plus anything they consistently edit out of drafts.
      Quote the instruction where you can find it.
   5. **Punctuation and formatting quirks.** Em dashes, capitalization,
      whether they end sentences with periods in short messages, bullets
      vs. prose, emoji.
   6. **Tone and attitude.** Blunt or hedged, warm or clipped, do they
      push back, do they apologize, how do they handle disagreement.
   7. **Structural habits.** How they open. How they close. Whether they
      front-load the ask or build to it. One topic per message or many.
   8. **How they correct things.** This one is gold and most people miss
      it. The pattern someone uses when fixing a draft reveals what they
      actually care about better than anything they'd say if you asked
      them directly.

3. **Write the file** to the template in Output format below. Fill every
   section with quotes. If the uploaded material genuinely doesn't cover
   a section, write "not enough signal in the material" rather than
   guessing — and tell them what to upload to fill the gap.

4. **File it durably.** If a CLAUDE.md / wiki setup already exists (from
   build-my-os or otherwise), match its shape — a project skill at
   `.claude/skills/<name>-voice/SKILL.md` is the strongest option,
   because it loads on demand in every session. Otherwise
   `wiki/entities/Voice.md`. If nothing exists yet, ask where it should
   live, or default to a top-level `voice.md`.

5. **Wire it in.** Point to the voice file from CLAUDE.md (or wherever
   the person's standing rules live) so it's read before any drafting
   task, not just when someone remembers to ask for it. The line in
   CLAUDE.md should say to load it *before* drafting, not after.

6. **Show them one test draft.** Write three or four sentences of
   something real in their voice — a DM opener, a post's first lines —
   and ask if it sounds like them. This is the only actual test of
   whether the file works, and it catches an over-fitted voice file
   immediately. If they say it's off, ask what specifically, and fold
   the answer straight into the file as a correction.

7. **Refreshing an existing voice file:** if one already exists, read it
   first, then integrate new uploads as additions or corrections — don't
   discard prior signal, and flag anything the new material contradicts.

## Output format

```markdown
---
name: [firstname]-voice
description: [Name]'s actual writing voice, built from their [source].
  Use this skill any time you are writing anything in [Name]'s own words
  — posts, DMs, emails, website copy, bios, or any other content they'll
  publish or send as themselves. Read it before drafting, not after.
---

# [Name]'s Voice

Built [date] from [exactly what the material was, with rough volume —
e.g. "their full Claude.ai export, 2,655 messages / ~150k words" or
"six sales call transcripts, ~4 hours"]. This is a reference for
writing in [Name]'s actual voice — not a generic "sound human"
checklist.

## Registers — know which one you're writing in
[Name each register, describe when it shows up, and state plainly which
one content should be written in. Quote an example of each so they're
distinguishable. Skip this section only if the material genuinely shows
one consistent mode.]

## Sentence structure & rhythm
[Bullets. Every bullet ends in a real quote.]

## Vocabulary & diction
[Recurring words as a list. Formality level. Jargon or no jargon.
Swearing or none — say "never swears in the corpus" if that's true.]

## Banned — never write these
[Explicit instructions they've given, quoted. Plus patterns you observed
them editing out repeatedly. This section is as important as the rest of
the file combined.]

## Punctuation & formatting quirks
[Em dashes, capitalization habits, list usage, emoji, message length.]

## Tone & attitude
[How they come across, each claim backed by a quote.]

## Structural habits
[How they open, how they close, how they order information.]

## How they correct a draft
[The pattern they use when fixing something. Quote three or four real
corrections.]

## Verbatim example quotes
[The quote bank — the most valuable part of the file. Group by context:
editing/feedback, direct asks, their own published writing, casual. Aim
for 30+. Keep typos and lowercase exactly as written.]

## Living correction log
This file is a snapshot from [date]. It should keep growing: any time
[Name] corrects a draft, append the correction here under the most
relevant section. Quote the line, say what was wrong, give the fix.

- (append new corrections here as they happen)

## Applying this
[Two or three lines on how to use this when actually drafting. Point at
the single clearest reference example in the quote bank — the one piece
of their own real writing that best shows the target.]
```

## Calibration

### Bad (what this skill must never produce)

```markdown
## Tone
Sarah writes in a friendly, professional, and approachable tone. She is
direct but warm, and she avoids overly formal language. Her writing is
clear and concise, and she likes to keep things conversational.
```

Nothing there is false, and none of it is usable. It describes maybe
half the working population. An agent handed this will produce the same
generic copy it would have produced with no voice file at all.

### Good (same person, done right)

```markdown
## Tone & attitude
- Warm opener, then straight to business. Almost every client email
  starts with one line of human contact before the ask: "hope the launch
  week wasn't too brutal!" then immediately "quick thing on the July
  calendar."
- Softens requests with "quick" and "just" constantly — "quick question,"
  "just wanted to flag," "just so it's on your radar." Six instances
  across four emails.
- Never apologizes for following up. Where a lot of people write "sorry
  to chase," she writes "bumping this up your inbox."
- Pushes back on clients directly, but always with a reason attached:
  "I wouldn't send that one on Friday. Your Friday opens are half your
  Tuesdays."
```

Same person. The second version can actually be written from.

### The over-fit failure

The opposite mistake is treating one upload as the whole voice. If all
you have is six sales call transcripts, you've captured how they talk
when selling, which is not how they write a newsletter. Say what the
file is based on at the top, and name the gap: "built from sales calls
only — their written voice for long-form is not yet captured, upload a
few sent emails or posts to fill that in."

Same with dictated material. Someone's voice-to-text messages are full
of run-ons and "like" and "I mean." Writing content in that register
produces something they will hate, because it's how they talk, not how
they want to read. Flag the register split explicitly.
