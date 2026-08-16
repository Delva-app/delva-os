---
name: build-prototype
description: >-
  Turn raw client material — sales call transcripts, requirement documents,
  screenshots, scrappy notes — into a live clickable prototype with a landing
  page, working sign-in and seeded demo data, built and deployed through the
  Lovable MCP, returning a URL the client can open. Use this skill whenever the
  user drops a client call transcript or requirements into a project folder and
  wants something to show, says "build the prototype", "turn this call into an
  app", "make an MVP for this client", "spin up a demo", or is preparing for a
  follow-up call where a client expects to see their idea working. ALSO use when
  the user has already built a prototype and wants to revise it from client
  feedback. Do NOT use for production software, for adding features to a real
  app, for writing proposals or price estimates, or when the user wants the
  requirements written up but explicitly not built.
---

# Build Prototype

A prototype's only job is to make the client say "yes, but change this." It is a
conversation device, not software. Everything in this skill follows from that:
the client decides in about ten seconds of first load, they decide on what they
can see and click, and the most valuable thing a prototype produces is the
client cutting something from it.

That reframe kills the usual failure mode. The instinct is to be faithful to the
requirements and cautious about anything unstated. That produces a thin, hedged
app that shows nothing and provokes nothing. Build the confident version, get it
wrong in visible places, and let the client correct you.

## Before the first run

This skill builds through the Lovable MCP server. If `mcp__lovable__*` tools
aren't available, stop and have the user connect it — there is no fallback and
no point composing a brief that can't be built:

```
claude mcp add --transport http -s user lovable https://mcp.lovable.dev
```

Then `/mcp` in Claude Code, select `lovable`, and authenticate in the browser.
OAuth is the only supported method. Builds run in the signed-in user's own
Lovable workspace and spend that workspace's credits, so if `list_workspaces`
returns more than one, ask which — never guess.

Check the credit balance before starting. A first build plus a seeding pass is
substantial, and running out midway leaves a half-built app.

## Core rule

Never ship an empty prototype. Empty tables, "Client 1", `test@test.com`, or a
dashboard of zeros will lose the deal no matter how good the architecture is.
Every screen must be full of data drawn from the client's own words.

## The archetype

Client prototypes vary far less than they appear to. Read
`references/archetype.md` before writing any brief — it is the seven-part shape
that every one of these builds takes, and the reasoning behind each part. Fill
in the client's nouns rather than inventing a structure per client.

## Workflow

Work in a folder per client opportunity:

```
<client-slug>/
  input/          raw material — transcripts, docs, screenshots
  brief.md        generated, human-edited
  revisions.md    feedback for follow-up rounds
  LINK.md         preview URL + demo logins
```

### 1. Read everything in `input/`

Read all of it before writing anything. Transcripts matter more than documents:
a document says what someone thought to write down, a transcript catches the
throwaway line where they name the real problem. In one irrigation call the
buying signal was a passing complaint about answering technician phone calls
during the demo itself, not anything in the stated requirements.

Pull out, specifically:

- **Their exact vocabulary.** "Pivot", "wet command", "in-body scan", "SOP".
  Using the client's own words in the UI is the difference between a prototype
  that feels built for them and one that feels generated. Never normalize their
  terms into generic ones.
- **What they complained about.** Complaints identify which module has to be
  good. Everything else can be a shell.
- **What they already have.** Anything they say works fine today is a candidate
  to cut, not to build.

### 2. Write `brief.md`

Use the exact structure in `assets/brief-template.md`. Then stop and hand it to
the human to edit. Do not build straight through.

This checkpoint exists because the person running the skill knows things about
the client that were never said on the call, and it is far cheaper to fix a
wrong assumption in markdown than in a built app.

### 3. Build through Lovable — app first, landing page second

Two passes, never one. Read `references/lovable-build-message.md` for the MCP
call order and message templates, and `references/design.md` for the direction
to fold into both messages.

**Always attach the images in `assets/design-refs/` to every build message.**
Describing a reference in words does nothing — the builder has never seen Clay
or Linear and will quietly use its own defaults. This has already produced one
thin, generic landing page on a real run.

The landing page gets its own pass because it loses every time it shares a
message with the app, and it is the first thing the client sees.

### 4. Check the seed, then top it up

The initial build usually seeds itself without being asked, and it seeds the
full-depth module well — one real run produced 5 SOP guides, 34 steps and 30
branching answers unprompted. What it under-seeds is the transactional tables
nobody is looking at while it builds.

So this step is verify-then-fix, not seed-from-scratch. Count the rows first:

```sql
SELECT relname AS table_name, n_live_tup AS rows
FROM pg_stat_user_tables WHERE schemaname='public'
ORDER BY n_live_tup DESC;
```

Run it with `query_database`. Targets:

- 8–12 records in the primary entity
- 20–30 in the transactional one
- one user per role, matching the demo login buttons
- around two weeks of backdated activity in any feed

On the real run, the deep tables were fine and the transactional ones came in at
7 jobs and 6 questions against a 20–30 target. Only top up what is short —
re-seeding tables that are already good wastes credits and risks the builder
rewriting content that was correct.

Also verify the demo accounts exist and their roles are right, because the login
buttons are useless if the rows behind them are missing:

```sql
SELECT p.full_name, p.email, r.role FROM profiles p
LEFT JOIN user_roles r ON r.user_id = p.id;
```

### 5. Write `LINK.md` and hand it over

Preview URL, the demo logins, and a one-line note on what was assumed. The
assumptions line is what starts the next call.

Add one line naming **the module worth graduating** — the full-depth one that
carries the client's actual pain. If they say yes, that module is what gets
built for real first, and recording it now saves rereading the transcript later.

### 6. Leave it graduatable

If the client says yes, the prototype gets exported out of Lovable and rebuilt
on Delva-owned infrastructure by the `build-production` skill. Two things make
that cheap or expensive, and both are decided here:

- **The schema must live in `supabase/migrations/`.** Confirm the files exist
  with `list_files` before handing over. Without them the schema exists only
  inside Lovable's Supabase instance, and the export produces a repo that cannot
  boot.
- **Seed data is not in git.** Lovable seeds by writing rows directly, so
  migrations carry the tables and none of the content. Nothing to fix here —
  just do not assume the seed travels with the code.

### Revisions

For a second round, read `revisions.md` and send it through `send_message` on
the existing project. Never rebuild from scratch — it loses everything the
client already approved.

## Deliberate assumptions

Build one or two modules the material only hints at, and label them in the UI as
assumed.

This feels wrong and is the highest-value part of the process. On a real call,
a builder showed a time-tracking module nobody had asked for; the client
immediately said they already had that, cut it, and focus on the SOP system
instead. That single exchange narrowed the scope and closed the deal. A question
in a document would not have produced it — the client had to see the wrong thing
to know what the right thing was.

## Calibration

**Under-building** looks like: only what was explicitly stated, empty states
everywhere, one role, no seeded data, integrations left as blank pages. The
client cannot tell what they are looking at and gives no useful feedback.

**Over-building** looks like: real integrations, production auth, edge-case
handling, ten modules of equal depth. Slow, expensive, and the client still only
looks at the first two screens.

Under-building is much worse. A prototype that is too rich gets cut down in one
call; a prototype that is too thin ends the conversation.

Where the effort belongs: the landing page, the first dashboard, and the one
connected flow. Everything else can be a well-dressed shell, and nobody will
click far enough to notice.

## Write findings back into this skill

Every real run teaches something the skill didn't know: an MCP call that lies
about its status, a phrasing that reliably produces better output, a failure
that looks like success. When that happens, update this skill before finishing
the task — not "later", because later never comes and the next run repeats the
mistake.

Where each kind of finding goes:

- MCP behaviour, call order, anything that failed → `references/lovable-build-message.md`
- Visual output drifting from the intent → `references/design.md`
- A structural pattern showing up across clients → `references/archetype.md`
- Something about the brief or the process → this file

Write the observed symptom, not just the fix. "Returns `status: completed` with
`agentFinished: true` in 20 seconds and nothing is built" is recognisable next
time; "use wait: true" alone is not, and the reader has no way to tell whether
the rule still applies.

If this skill is installed in more than one place, sync every copy — otherwise
the fix only lives in one and the next person hits the same wall.

## What this never does

No real integrations. If the client asked for Apple Health, an EMR, Zoho or
Stripe, build the screen showing what it would look like and label it as not
connected. Clients accept this readily when told plainly and resent discovering
it themselves.

No production auth hardening, no real client data, no compliance claims. If the
material raises HIPAA, PII, or anything similar, note it in `LINK.md` as a real
build consideration and keep it out of the prototype.

None of this is a limitation to apologise for. Lovable's Supabase instance is
disposable by design — it exists to make the demo real enough to argue with, and
it is thrown away at the graduation step. Building production concerns into a
prototype spends credits on work that gets deleted.

---

## Customizing this skill

This file ships from the `delva-os` marketplace and is **replaced wholesale** every time you run `/plugin marketplace update delva-os`. Edits made here are lost on the next update.

To customize it and keep the change:

- **Project-specific inputs** (your brand, your pricing, your templates) belong in your own repo, not in this plugin. Point the skill at them from your `CLAUDE.md` and it will read them.
- **Changing the skill itself:** copy `skills/build-prototype/` into `.claude/skills/` in your project and disable the plugin (`/plugin`). Your copy is then yours, and updates stop touching it.
