---
name: build-my-os
description: >-
  Set up someone's AI from scratch in one sitting: hand them the LLM Wiki
  pattern to instantiate their directory and CLAUDE.md schema, open the
  export pages for whichever AI tools they use so their ChatGPT and
  Claude chat history is processing in the background, interview
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
   the business but not how they write or talk is half done. Step 4
   hands off to `build-my-voice`, which builds a voice file from
   what they've already said — meeting transcripts and their own AI
   chat history.
4. **It leaves them able to check their own work.** A setup that's never
   audited goes stale silently. Step 5 installs `os-audit` so
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
  whole list. A wall of nine questions makes people quit.
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
  where every project skill goes (`os-audit` in step 5, their voice
  file's companion skill if `build-my-voice` produces one, anything they
  add later), and a skill only loads if it sits at
  `.claude/skills/<skill-name>/SKILL.md` — one folder per skill, each
  with its own SKILL.md. Creating it up front means step 5 has somewhere
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

### 2. Kick off their AI chat exports

Do this **before the interview**, not after. Both exports are processed
on the provider's side and arrive by email — Claude's usually within
minutes, ChatGPT's anywhere from minutes to a few days — so starting
them now means the file is often in hand by the time step 4 needs it.
Their chat history is the single best voice source there is: thousands
of messages they typed themselves, unedited.

**Ask which of the two they use, and say what it's for.** This is a
checklist question, not a preference question. It is not "which do you
use most" or "which do you prefer" — it's "do you use both of these,
one, or neither." Asked the preference way, people pick whichever one
they happen to be sitting in and you lose the account with four years
of their writing in it. Say up front that it's for pulling their
context, so they understand there's no wrong answer and no reason to
leave one out:

> This is going to pull your past chat history so it gets your context.
> Which of these do you use?
> **A.** ChatGPT  **B.** Claude  **C.** Both  **D.** Neither

Those two lines and those four options, as written. Don't add
qualifiers, don't expand the pitch. Use `AskUserQuestion` if it's
available (multiSelect on, one option each for ChatGPT and Claude) so
they can tick both rather than typing a letter.

**Then open the export page for each one they named.** Don't describe
where the button is — put them on the exact screen:

| Tool | Open this | Then |
|---|---|---|
| ChatGPT | `https://chatgpt.com/#settings/DataControls` | Under *Export data* → **Export** → **Confirm export** |
| Claude | `https://claude.ai/settings/data-privacy-controls` | **Export data** |

**If the page offers any choice about what to include, take the option
that gets everything.** Where there's a scope selector, pick the custom
or "select data" path and tick every box rather than accepting a
narrower default — a partial export defeats the point, and the whole
history is what makes the voice file and the context worth having. Tell
them this out loud when you send them to the page, in one line: pick
everything, tick every box. (ChatGPT's export is currently
all-or-nothing with no selector, so there's nothing to choose there;
say nothing if no selector appears.)

Open it for them with `open "<url>"` on macOS (`xdg-open` on Linux,
`start` on Windows). If you can't open a browser in this session, print
the URL as a clickable link and say which button to click on the page.
Open both tabs if they said both.

Tell them what happens next, in one line each:

- The link arrives **by email**, not in the browser — check inbox
  (and spam) for it.
- The download link **expires 24 hours** after it arrives, so download
  the zip when they see it.
- ChatGPT's export can take up to 7 days in the worst case; Claude's is
  usually quick.
- When the zip lands, **drop it into the chat** (or say where it is on
  disk) — that's all that's needed.

Two things that will come up:

- **ChatGPT Business/Enterprise workspaces can't self-export.** If they
  hit that, note it and move on — don't stall the build.
- **Mobile can't export Claude.** It's web or desktop app only.

Then **go straight into the interview while it processes.** Do not sit
and wait for the file. If the zip arrives mid-session, take it and carry
on. If it hasn't arrived by the end of the session, say so plainly and
tell them to drop it in whenever it lands and re-run
`/build-my-voice` — a voice file built later is fine; a fabricated one
is not.

If they answered **D (neither)**, skip this entirely and lean on meeting
transcripts in step 4 instead.

### 3. Interview (one question at a time, adaptive)

Ask these nine seed questions in order, one at a time. Each question has a
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

**About the work**

8. **Tools and where things live.** Ask for a full inventory, not a
   highlight reel: *"List every single tool you use in the business —
   try to get all of them, not just the main ones. Things like Gmail,
   Slack, Stripe, Google Drive, Notion, HubSpot, QuickBooks, Calendly,
   Canva, GitHub. And where does your key info actually live — docs,
   CRM, drives — that the AI should be able to pull from?"*

   Give the examples out loud. People blank on this question and name
   two tools; a concrete list unsticks them and they'll come back with
   twelve. Prompt once more after their first answer — "anything for
   invoicing, scheduling, storage, or support?" — since those four are
   the usual blind spots.

   This question is what makes step 6 fast. Every tool named here gets
   connected in step 6 without asking them again, so an exhaustive
   answer now means they do nothing later but click authorize.

   **As soon as they finish answering, launch a background subagent to
   research the list, then carry straight on to Q9.** Do not make them
   wait for it and do not narrate it beyond a half-sentence. Spawn a
   `general-purpose` agent with roughly:

   > For each of these tools — [list] — determine the best way to
   > connect it to Claude Code today. Use WebSearch; do not answer from
   > memory, since MCP and CLI availability changes constantly. For each
   > tool return: (a) official CLI, with the exact install and auth
   > commands, if a real one exists; (b) official MCP server, with the
   > exact `claude mcp add` command and transport, if one exists;
   > (c) whether it's an Anthropic-hosted first-party connector
   > configured on claude.ai; (d) plain API key only, with the env var
   > name and the URL to get the key. Recommend one path per tool using
   > the CLI-first rule. Say "none found" rather than guessing a command
   > — a wrong command wastes the user's time in the next step.

   Why a subagent: the research is a dozen web searches whose output is
   mostly noise, and it runs during Q9 for free. Why it must search:
   this is the fastest-moving corner of the ecosystem, and a plausible
   invented `npx` package name is worse than no answer.

   When it comes back, save the result to `wiki/entities/` as a working
   note or hold it for step 6 — either way, step 6 reads it instead of
   re-deriving the same thing.

9. **Brain dump — everything the first eight didn't ask.** Always last,
    always asked, no matter how full the wiki already looks. Eight
    questions get the shape of the business; they don't get the stuff
    in their head that only they know to mention. Open the floor and
    give them a list to react to, because a blank "anything else?"
    gets "no, I think that's it" almost every time:

    > Last one, and it's the open one. Dump anything else in your head
    > that I haven't asked about. No structure needed, no full
    > sentences, ramble as long as you want. Things worth including:
    >
    > - **Goals** — what you want to happen in 6 or 12 months
    > - **Challenges** — what's stopping you from getting there right now
    > - **Ideas** — things you're thinking about trying, or want to do
    > - **Strengths** — what you're genuinely good at
    > - **Weaknesses** — what you avoid, or keep getting wrong
    > - Anything else: people, history, opinions, context you'd have to
    >   explain to a new hire in week one

    Treat the list as prompts, not a form — they don't have to hit
    every bullet. If they give you three lines, ask one follow-up on
    whichever bullet they skipped that matters most for their business,
    then stop. Don't interrogate them at the end of a long interview.

    Then **split the dump into real pages** rather than dropping it
    somewhere whole as a lump of text. Goals become
    `wiki/concepts/Goals.md`, current challenges go to `_hot.md` active
    threads (and a project page if it's live work), ideas become
    `wiki/concepts/Ideas.md` as a running list, strengths and
    weaknesses go on their own `wiki/entities/<Their Name>.md`
    alongside Q6. Anything that doesn't fit an existing page gets its
    own. Link them like everything else.

Adapt. If an earlier answer already covers a later question, don't ask it
again, confirm it and move on. If they run a kind of business that raises
an obvious gap these nine don't cover, ask about it. Q9 is the
catch-all, but a question you should have asked outright belongs in the
interview, not left to the dump.

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
| Q8 tools | `wiki/entities/` one page per tool named; step 6 connects each |
| Q9 brain dump | split across `wiki/concepts/Goals.md`, `wiki/concepts/Ideas.md`, `wiki/entities/<Their Name>.md`, `_hot.md`; new pages as needed |

Nothing from the interview belongs in CLAUDE.md. Every answer is a page. Link the pages to each
other as you write them, and add each one to `index.md`.

### 4. Hand off to build-my-voice

Once the schema, directory, and pages are in place, invoke the
`build-my-voice` skill to build out how they actually write and talk.
Pass it the directory shape step 1 built (so it knows where to file the
voice file and how to wire it into the CLAUDE.md this session produced).

Check first whether the export from step 2 has landed. If it has, hand
the zip straight to `build-my-voice`. If it hasn't, ask once whether the
email has shown up, then carry on without it and leave the voice file as
the open item — they re-run `/build-my-voice` with the zip when it
arrives.

That skill depends on real uploads (chat exports, transcripts) and will
say plainly if the person has nothing to upload yet rather than
fabricating a voice — don't skip past that outcome silently if it
happens; note it as the one open item.

### 5. Install os-audit

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

### 6. Wire up connections

Now that the directory exists and there's somewhere for the answers to
live, look at what's actually connected in this session — Drive,
calendar, CRM, email, whatever MCP tools are visible — and reconcile it
against what they told you in Q8.

1. For each connected tool that matters to their work, write or update
   its page in `wiki/entities/`: what it is, what lives in it, what the
   agent is allowed to do with it.
2. For anything they named in Q8 that **isn't** connected, connect it —
   see *Connecting a tool* below. Do the work for them; never hand them
   a docs page and tell them to figure it out.
3. If something can't be connected (no integration exists, they don't
   have an account), leave a page noting it as not yet wired up. A
   named-but-missing tool is useful information, not a failure. Do not
   stop the build over it.

This is deliberately late in the flow. Connections are worth most once
there's a directory to route them into, and gating the whole build on
them just means people bounce off before they have anything.

#### Connecting a tool

**Your job is to do everything except the click that only they can make.**
They should never have to hunt through a settings menu.

Start from the Q8 research subagent's report — it already has the per-tool
path and the exact commands. If it flagged something as "none found," check
once yourself before telling them a tool can't be wired up. Three paths —
pick by what the tool actually is, don't default to the connectors page:

**A. Mature CLI exists** (`gh`, `stripe`, `vercel`, `supabase`, `aws`,
`wrangler`, `railway`…) — **prefer this.** Install it for them, run the
auth command, and let it open the browser itself:

```
brew install gh && gh auth login --web
```

Why first: a CLI costs zero context until it's called, exposes the full
API rather than a curated subset, and is scriptable. An MCP server loads
every one of its tool schemas into the window whether or not it gets
used. Then write the tool's `wiki/entities/` page with **the exact
commands their agent should use** — a CLI has to be taught; unlike MCP
it doesn't announce itself.

**B. Local or remote MCP server** (Stripe, Apify, Notion, Linear,
Sentry, Playwright, anything with an npx package or an MCP URL) — the
tool has no good CLI, or auth is per-user OAuth. Run it for them:

```
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

That prints an OAuth URL. **Open it for them** and say exactly what to
click — "authorize, then sign in with Google." That's their entire job.
Use `--scope user` if the tool should follow them across every project,
`--scope project` (writes `.mcp.json`, commit it) if it belongs to this
OS only. Confirm with `claude mcp list` before moving on.

**C. Anthropic-hosted connector** (Google Drive / Calendar / Gmail,
Canva, Calendly, Fathom, Slack, and the rest of the first-party list) —
these are **account-level, authorized on claude.ai, and synced down into
Claude Code**. You can spot them in a session by the `mcp__claude_ai_`
tool prefix. There is no CLI or `claude mcp add` path; the connectors
page is genuinely the only way in, so don't waste a turn trying.

Still don't make them navigate. Open the connectors page directly:

```
open "https://claude.ai/settings/connectors"
```

Tell them the exact connector name to enable and that it's authorize +
sign in with Google, nothing more. Then have them start a fresh Claude
Code session — connectors sync at session start, so the tools won't
appear in the current one.

**Choosing between them:** mature CLI → A. Hosted SaaS with OAuth and no
real CLI → B. Google Workspace and the other first-party names → C, no
choice. If a tool offers both a CLI and an MCP server, take the CLI
unless they specifically want the model discovering the capability on
its own without being taught.

Record which path each tool took on its `wiki/entities/` page — it's the
first thing anyone needs when a connection breaks six months later.

### 7. Seed persistent memory

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
   in drafts," "always confirm before sending client emails," a
   communication-style rule they mentioned — write it as a `feedback` memory
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

### 8. Turn on Remote Control

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

### 9. Hand off

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
[Any communication preference they volunteered, as instructions: "Be
blunt. Lead with the answer. Skip preamble." Standing behavior, so it
lives here. Leave the section out entirely if they never said anything
— do not invent a style for them.]

## Voice
[Pointer to the voice file built in step 4, once it exists.]

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
[Anything they said must never happen, as a plain list. Usually surfaces
in the brain dump or in passing during the interview. Standing behavior,
so it lives here. Leave it out if nothing came up.]
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
is good, take it and move on. Nine sharp questions beat twenty tired ones.

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

The ChatGPT export they kicked off back in step 2 lands mid-interview.
Step 4 builds the voice file from it plus two client call transcripts:
short, blunt, no filler, matching how they actually write client emails.
