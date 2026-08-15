---
name: build-my-os
description: >-
  Set up someone's AI from scratch in one sitting: hand them the LLM Wiki
  pattern to instantiate their directory and CLAUDE.md schema, interview
  them one question at a time about their business and how they work and
  file those answers as wiki pages, hand off to build-my-voice for a
  voice file from their meeting transcripts and AI chat history, wire up
  their connected tools, and install os-audit so the setup can be checked
  for drift later. Use this skill whenever the user
  invokes /build-my-os, or asks to "set up my Claude", "build me a
  CLAUDE.md", "make my AI know my business", "onboard my AI", "build my
  operating system", "create my operating system for Claude", or says
  their AI keeps forgetting their context and they want to fix it for
  good. ALSO use it when someone new to Claude Code asks how to get
  started or how to make the AI actually useful for their work. Do NOT
  use for editing one section of an existing CLAUDE.md (just edit it),
  for auditing a setup that already exists (that's os-audit — this
  skill always builds fresh), for writing project code, or for general
  prompt-writing advice unrelated to a persistent setup file.
---

# Build my OS

Most people use AI like a slot machine: open a blank chat, paste a
question, hope. The AI forgets them the moment the tab closes. A
CLAUDE.md fixes that. It is a file the agent reads first, every session,
so it starts from everything it already knows about the person and their
business instead of a blank box. This skill builds that setup for
someone from scratch, every time — a fresh install, not a patch job.

Three things make this skill worth more than a generic "answer
questions, get a file" prompt, and you must deliver all three:

1. **It builds the real directory, not just prose.** The LLM Wiki
   pattern (below) sets up `raw/`, `wiki/`, `index.md`, `log.md`, and the
   CLAUDE.md schema together, as one system — not a CLAUDE.md floating
   alone with nowhere to route to.
2. **It teaches as it builds.** When you make a call, say why in one
   line, so they finish the session understanding their own setup, not
   just holding a file they can't maintain.
3. **It gives them a voice, not just a schema.** A CLAUDE.md that knows
   the business but not how they write or talk is half done. Step 3
   hands off to `build-my-voice`, which builds a voice file from
   what they've already said — meeting transcripts and their own AI
   chat history.
4. **It leaves them able to check their own work.** A setup that's never
   audited goes stale silently. Step 4 installs `os-audit` so
   they (or their agent) can check the setup for drift later.

## The one rule that decides whether this worked

**CLAUDE.md is a router, not a container.** It holds the schema — what
the folders are, what the conventions are, which workflow to run, where
things live. It does **not** hold their offer, their pricing, their ICP,
their bottleneck, or any other business fact. Those are pages in the
wiki.

This is the single most common way this skill fails. A CLAUDE.md with
`## What the business does` followed by offer, pricing, and ideal client
means the build went wrong, even if every fact in it is correct. That
file is a context dump wearing a schema's clothes. It bloats every
session with facts most queries never need, it goes stale silently
because nothing routes to it, and it leaves the wiki folders empty — a
directory with nothing in it.

The test: **could you swap in a different person's business and change
almost nothing in CLAUDE.md?** If yes, it's a router and you did it
right. Their name, their folder names, and their hard rules will differ.
Their pricing should not appear at all.

So when the interview produces "$3k/month retainer," that sentence goes
in `wiki/concepts/Offer.md` and, if it's a live number they'll be asked
about often, in `_hot.md`. What goes in CLAUDE.md is the rule that
pricing questions get answered from the wiki.

## Core rules

- **This is always a fresh build.** Do not scan for or read an existing
  `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, or prior wiki content before
  starting, and do not produce an audit of what's already there. If a
  CLAUDE.md already exists in the project, tell them plainly that this
  skill replaces it with a new one built from scratch and confirm before
  overwriting — but don't inventory or critique the old one. This skill
  is not for incremental fixes; point them at editing the file directly
  for that.
- **Never ask them to paste anything they don't already have to hand.**
  They are running you inside their own project. The one exception is
  the uploads in the voice-file step (chat exports, transcripts) — those
  genuinely have to come from them.
- **One question at a time.** Ask a seed question, read the answer, ask a
  sharp follow-up if the answer is thin, then move on. Never dump the
  whole list. A wall of ten questions makes people quit.
- **Nothing here is a gate.** Connected tools, voice uploads, an
  available os-audit: all of them are nice to have, none of them stop
  the build. If something is missing, note it as an open item and keep
  going. They end the session with a working directory either way.
- **Write like a person.** Everything you output (the CLAUDE.md prose,
  the chat) must read like a human wrote it. No em dashes, no hype words
  (leverage, unlock, supercharge, seamless, robust), no "it's not X, it's
  Y" reframes. Short lines. Say the thing.

## Workflow

### 1. Hand them the LLM Wiki pattern

Paste the LLM Wiki pattern (reproduced in
full in `references/llm-wiki-pattern.md`, alongside this file) into the
conversation as if they'd shared it themselves, and work through it with
them exactly the way the pattern describes: instantiate the three
layers together, in collaboration with them, adapted to their domain.

- **Raw sources** — wherever their curated source material will live
  (`raw/`), immutable.
- **The wiki** — the LLM-maintained knowledge base (`wiki/` or
  `knowledge/`), split into subfolders that match their business.
  Not every subfolder from Bran Brain's own layout (`sources/`,
  `entities/`, `concepts/`, `projects/`, `answers/`) applies to every
  business — pick the ones their work actually produces. A solo
  consultant may need only `entities/` and `projects/`.
- **The schema** — `CLAUDE.md` itself, plus `index.md` (the catalog) and
  `log.md` (the append-only timeline). Add `_hot.md` (a short hot cache
  of active threads and key numbers, read first) if they have more than
  a handful of ongoing threads.
- **The skills folder** — `.claude/skills/` inside the project root.
  Create it in this step, even though nothing lives in it yet. This is
  where every project skill goes (`os-audit` in step 4, their voice
  file's companion skill if `build-my-voice` produces one, anything they
  add later), and a skill only loads if it sits at
  `.claude/skills/<skill-name>/SKILL.md` — one folder per skill, each
  with its own SKILL.md. Creating it up front means step 4 has somewhere
  to put things and they have an obvious place to drop skills people
  send them. Mention it when you walk them through the layout: this is
  the folder that makes a skill installable by dragging it in.

Note that `CLAUDE.md` sits at the project root, not inside `.claude/` —
that's where it gets auto-loaded from.

This step produces the CLAUDE.md, and it produces it as a **router**
(see the rule above). Write the schema now: the folder map, the page
conventions, the ingest and query workflows, the memory precedence. It
should be a complete, working file before the interview starts, with no
business facts in it yet and no blanks waiting to be filled.

Create the folders and navigation files with real starting content, not
stubs. `index.md` and `log.md` should reflect this setup session as
their first entries.

### 2. Interview (one question at a time, adaptive)

Ask these ten seed questions in order, one at a time. Each question has a
short breakdown of what a full answer covers. **Show the breakdown** so
they know exactly what to include, then wait for their answer. Vague or
partial answers are the norm, so if they skip part of the breakdown, ask
one tight follow-up for the missing piece before moving on. Specific
answers are the whole point: a generic answer produces a generic file.

**About the business**

1. **Your offer.** What exactly do you sell? Break it down:
   - The core outcome you deliver (the result, not the label)
   - What's included: deliverables, scope, timeline
   - What you charge and the pricing model (one-time, retainer, %, tiers)
   - Any guarantee or risk reversal you offer

2. **Your ideal client (ICP).** Who is the perfect buyer, exactly?
   - Who they are: role, industry, company size or stage, or life situation
   - The trigger that makes them need you right now
   - Who's a bad fit, the client you turn away

3. **The core problem.** What painful problem are they actually paying you
   to solve, and what does it cost them if they don't fix it?

4. **Your edge (unique mechanism).** Why you over the alternatives? The
   specific method, angle, or proof that makes you different, not just a
   claim of "better."

5. **Current bottleneck.** What's the one constraint holding the business
   back right now (leads, delivery, time, cash, hiring)? This tells the AI
   what to prioritize.

**About you**

6. **Your role and time.** What's your role, and where does your time
   actually go? What are you doing when you're most valuable, versus what
   eats your day?

7. **First handoff.** If AI could take one thing off your plate first,
   what is it, and what makes it slow or annoying today?

8. **How it should talk to you.** Blunt or warm? Short or detailed? Should
   it push back or just execute? Any format you like (bullets, answer
   first)?

**About the work and rules**

9. **Tools and where things live.** What tools do you work in day to day,
   and where does your key info live (docs, CRM, drives) that the AI
   should know about or pull from? Step 5 checks this against what's
   actually connected.

10. **Hard rules.** What must it never do? Brand or tone lines, compliance,
    claims you can't make, price or approval rules, off-limits topics.

Adapt. If an earlier answer already covers a later question, don't ask it
again, confirm it and move on. If they run a kind of business that raises
an obvious gap these ten don't cover, ask about it.

**File each answer as you go, into the directory step 1 built.** Not into
CLAUDE.md. Write the page as soon as the answer is good enough, so they
watch the wiki fill up question by question — that's the moment the
directory stops being empty folders and starts being their second brain.
Default mapping, adapted to the folders you actually created:

| Answer | Where it goes |
|---|---|
| Q1 offer, pricing, guarantee | `wiki/concepts/Offer.md`; price to `_hot.md` |
| Q2 ICP + bad fit | `wiki/concepts/Ideal Client.md` |
| Q3 core problem | `wiki/concepts/Problem We Solve.md` |
| Q4 edge / mechanism | `wiki/concepts/Our Edge.md` |
| Q5 bottleneck | `_hot.md` active threads + `wiki/projects/` if it's live work |
| Q6 role and time | `wiki/entities/<Their Name>.md` |
| Q7 first handoff | `wiki/projects/` as the first project page |
| Q8 how it should talk | CLAUDE.md (standing behavior) + memory in step 6 |
| Q9 tools | `wiki/entities/` one page per real tool; step 5 confirms |
| Q10 hard rules | CLAUDE.md (standing behavior) |

Only Q8 and Q10 belong in CLAUDE.md, because they are standing rules
rather than facts. Everything else is a page. Link the pages to each
other as you write them, and add each one to `index.md`.

### 3. Hand off to build-my-voice

Once the schema, directory, and pages are in place, invoke the
`build-my-voice` skill to build out how they actually write and talk.
Pass it the directory shape step 1 built (so it knows where to file the
voice file and how to wire it into the CLAUDE.md this session produced).

That skill depends on real uploads (chat exports, transcripts) and will
say plainly if the person has nothing to upload yet rather than
fabricating a voice — don't skip past that outcome silently if it
happens; note it as the one open item.

### 4. Install os-audit

Make sure `os-audit` is available in this project so the
setup can be checked for drift later — routing pointing at things that
don't exist, stale numbers, orphan pages. It belongs at
`.claude/skills/os-audit/SKILL.md`, in the `.claude/skills/` folder step
1 created. If you have the skill available in this session, copy it
there directly. If it's already present as a plugin skill, confirm that
and leave it alone rather than installing a second copy. If it isn't
available at all, say so plainly and tell them where to drop it — don't
silently skip it, and don't write a hollow stub SKILL.md in its place.
Mention in chat that running `/os-audit` periodically (the CLAUDE.md's
own Lint workflow, if step 1 produced one, is a good trigger point) is
how they keep this from going stale.

### 5. Wire up connections

Now that the directory exists and there's somewhere for the answers to
live, look at what's actually connected in this session — Drive,
calendar, CRM, email, whatever MCP tools are visible — and reconcile it
against what they told you in Q9.

1. For each connected tool that matters to their work, write or update
   its page in `wiki/entities/`: what it is, what lives in it, what the
   agent is allowed to do with it.
2. For anything they named in Q9 that **isn't** connected, say so plainly
   and leave a page noting it as not yet wired up. A named-but-missing
   tool is useful information, not a failure.
3. If nothing at all is connected, that's fine. Note it as an open item,
   tell them what connecting one would unlock for their setup
   specifically, and move on. Do not stop the build over it.

This is deliberately late in the flow. Connections are worth most once
there's a directory to route them into, and gating the whole build on
them just means people bounce off before they have anything.

### 6. Seed persistent memory

Everything they just told you in the interview has to land somewhere,
and not all of it belongs in the wiki/CLAUDE.md you just built. Claude
Code has a separate auto-loading memory store, per project, at
`~/.claude/projects/<escaped-project-path>/memory/` (one markdown file
per fact, indexed by a `MEMORY.md` in that same folder) — it loads into
every session the same way CLAUDE.md does. Right now, for a fresh build,
that folder is either empty or doesn't exist. Don't leave it that way:

1. Create the memory directory if it isn't there yet (write directly —
   don't shell out to check first).
2. Seed it with a `user` memory: a short file capturing who they are and
   what the business does in one or two lines, `description` tuned for
   recall (e.g. "founder of a done-for-you email agency for Shopify
   brands, prefers blunt short answers"). This is the fact most worth
   having on hand before any wiki lookup happens. Keep it to identity —
   it points at the wiki for anything live.
3. If anything from the interview is a **standing correction or working
   preference** rather than a business fact — e.g. "never use em dashes
   in drafts," "always confirm before sending client emails," the
   communication-style rule from Q8 — write it as a `feedback` memory
   with a **Why** line.
   Facts about the business (offer, ICP, pricing) stay in the wiki and
   never get copied here. Memory notes point at wiki pages rather than
   restating their numbers, so there's only ever one place to update.
4. Write `MEMORY.md` as the index: one line per file, `- [Title](file.md)
   — one-line hook`.
5. Explain the three-store split to them in one short paragraph before
   moving on, so they understand where things live going forward: this
   memory folder for how-you-work facts, the wiki for business facts,
   CLAUDE.md for standing rules — and if two disagree, the wiki wins.
   This mirrors the `## Memory systems` section already written into
   their CLAUDE.md in step 1 — point them at it rather than repeating
   yourself at length.

If this Claude Code install has no cross-session memory feature
available, say so plainly and skip this step — don't fabricate a memory
folder that won't actually get read.

### 7. Turn on Remote Control

An OS is only useful if they can actually reach it. Remote Control lets
them drive this session from their phone or another machine, which is
the difference between "my AI knows my business" and "my AI knows my
business and I can ask it something while I'm walking to a meeting."
Don't leave it as a per-session toggle they'll forget — turn it on
permanently:

1. Read `~/.claude/settings.json` (create it as `{}` if it doesn't
   exist).
2. Set `"remoteControlAtStartup": true` at the top level, preserving
   every other key in the file. Edit the existing file — never overwrite
   it with a fresh object.
3. Tell them what this did and didn't do — be precise, because the
   setting alone is not the whole job:
   - It takes effect at **session start**, so it turns on from their
     next session, not this one.
   - It does **not** authenticate the remote-control daemon. That's a
     one-time user action you cannot do for them: they run
     `/remote-control` themselves once and complete the auth/pairing
     prompt. After that, the flag brings it up automatically every
     session.
   - If `~/.claude/daemon-auth-status.json` exists and reads
     `"status":"auth_required"`, that's exactly the state they're in —
     say so and point them at the one-time step.

This is a user-level setting, so it applies to all their projects, not
just this one — say that when you tell them. If they push back or want
it project-only, drop it and move on; don't argue it.

### 8. Hand off

Tell them the directory is built, what each piece is for, and the single
highest-value next step (usually: ingest their first real source, or
finish the voice file if they hadn't uploaded anything yet). List any
open items from steps 3 and 5 in one line each. Keep it short.

## Output format

### CLAUDE.md

Schema only. No offer, no pricing, no ICP — those are pages.

```markdown
# [Business or person's name] — Operating Context

You are the agent for [name/business]'s second brain. [Name] curates
sources and asks questions; you do the writing, filing, and
cross-referencing. Every interaction in this directory follows this
schema.

**Scope:** [What belongs in this directory and what doesn't, in two or
three lines. Written so a future agent can decide where something goes.]

## Architecture

[The three layers, as literal folders that exist:]
1. `raw/` — immutable source documents. Never edit or delete.
2. `wiki/` — the LLM-maintained knowledge base. You own this layer.
3. `CLAUDE.md` — this file, the schema.

[Plus the navigation files that exist: `index.md` the catalog,
`log.md` the append-only timeline, `_hot.md` the hot cache if built.]

## Folder conventions

[The literal folder tree created in step 1, one line each on what goes
where. File naming convention.]

## Page conventions

[Frontmatter block. Link liberally. Cite sources. Flag contradictions
rather than overwriting. Pages stay current, not chronological.]

## How to work with me
[Q8, as instructions: "Be blunt. Lead with the answer. Skip preamble."
This is standing behavior, so it lives here.]

## Voice
[Pointer to the voice file built in step 3, once it exists.]

## Workflows

### Ingest (when [name] drops a source or pastes content)
1. Save it in `raw/` (or confirm it's already there).
2. Read it fully, summarize key takeaways, note anything that
   contradicts existing pages.
3. Write or update the relevant page(s) in the wiki folders.
4. Update `index.md` and `_hot.md` if a key number or active thread
   changed.
5. Log it in `log.md`.

### Query (when [name] asks a question)
1. Check `_hot.md` first if present, then `index.md`, then read the
   candidate pages directly.
2. Answer with citations to the pages used.
3. If the answer has lasting value, file it as a page and log it.

[Adapt or trim these two if the business doesn't produce enough source
material to warrant a formal ingest workflow — a lean setup might only
need Query.]

## Memory systems
Three stores, don't duplicate facts across them:
- **CLAUDE.md** (this file) — standing rules and conventions. Rarely changes.
- **The wiki** (`wiki/` + `index.md` + `_hot.md`) —
  source of truth for business facts.
- **Auto-memory** (`~/.claude/projects/.../memory/`) — durable facts
  about how [name] works: preferences, corrections, working style. Points
  at the wiki rather than restating its live numbers.

If two disagree, the wiki wins for business facts.

## Hard rules (never break these)
[Q10, as a plain list. Standing behavior, so it lives here.]
```

### A wiki page

```markdown
---
type: concept
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: []
---

# Offer

[The actual content — outcome, what's included, pricing, guarantee.
Linked to [[Ideal Client]] and [[Our Edge]].]
```

## Calibration

The failure to avoid most is **business facts in CLAUDE.md**. Re-read
the router rule at the top of this file before you write a single
heading. If you find yourself typing a price, a client type, or a
deliverable into CLAUDE.md, stop: that's a page.

The second failure is a **generic setup** that could belong to anyone:
"I run a business and want help being productive." That does nothing.
Push every answer until it's specific enough that the pages could only
be this person's. A vague answer is a signal to ask the follow-up, not
to write it down as given.

The opposite failure is **interrogation**: dumping all ten at once, or
grinding through follow-ups on a point that's already clear. If an answer
is good, take it and move on. Ten sharp questions beat twenty tired ones.

A fourth: **skipping step 1**. The directory and CLAUDE.md come from
working the LLM Wiki pattern with them, not from a template you fill in
unilaterally after the interview. The interview answers are content that
gets filed into the structure step 1 built — never the other way around.

## Example

Step 1 sets up `raw/`, `wiki/entities/`, `wiki/concepts/`,
`wiki/projects/`, `index.md`, `log.md`, `.claude/skills/`, and a
complete router `CLAUDE.md` for an e-commerce email agency. Then Q1
(offer), shown with its breakdown: core outcome, what's included,
pricing, guarantee.

User (Q1): "I do email marketing for e-commerce brands."

That's an industry, and it skips most of the breakdown. Follow up for the
missing pieces: "What exactly do you deliver, what do you charge, and do
you have a guarantee?"

User: "I write and manage their whole email calendar. $3k a month
retainer. If they don't make back my fee in 60 days I work free until they do."

Now Q1 is real. It gets filed as `wiki/concepts/Offer.md`:

```markdown
---
type: concept
created: 2026-08-13
updated: 2026-08-13
tags: [offer, pricing]
---

# Offer

Done-for-you email: the full calendar, written and managed.

- **Pricing:** $3k/month retainer.
- **Guarantee:** Make back the fee in 60 days, or I work free until they do.

Sold to [[Ideal Client]]. What makes it different: [[Our Edge]].
```

`_hot.md` gets one line: `Retainer: $3k/mo.` `index.md` gets
`- [[Offer]] — what we sell, pricing, guarantee.` CLAUDE.md gets
nothing, because nothing about this changed the schema.

Then Q2 (ICP) gets "founders of 7-figure Shopify skincare brands,
usually right after a launch flopped," which becomes
`wiki/concepts/Ideal Client.md` and links back to `[[Offer]]`.

After the interview, step 3 asks for their Claude/ChatGPT export and any
client call transcripts, and builds the voice file from them — short,
blunt, no filler, matching how they actually write client emails.
