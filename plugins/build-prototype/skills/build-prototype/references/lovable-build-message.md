# Building through the Lovable MCP

## Attach the reference images to every build message

Not just the first one. Every message that changes how anything looks.

Describing a reference in words does not work. "Like Clay" or "like Linear"
means nothing to the builder — it has no idea what those look like, and it will
fall back to its own defaults while appearing to follow the instruction.

Observed on a real run: the references were attached to `create_project`, that
message stalled, and the build was actually triggered by a follow-up
`send_message` with no `files` array. The app UI came out fine by luck; the
landing page came out thin and generic, because a landing page is almost
entirely visual and had nothing to work from.

So: upload once, keep the `files` array, and pass it on every build and every
visual revision.

## Build the app first, then the landing page

Two passes, in this order. Do not ask for both in one message.

The landing page always loses when it shares a message with the app. The builder
spends its attention on data models, routes and auth, and the landing page
arrives as three thin sections — which is a problem, because the landing page is
the first thing the client sees and the thing they judge the build on.

App first also gives the landing page something real to show. Pass 2 can point
at screens that already exist and pull genuine product imagery into the hero,
instead of inventing a mockup.

## Call order

1. `list_workspaces` — if more than one, ask which. Never guess; builds consume
   that workspace's credits.
2. `get_file_upload_url` for each design reference and each client screenshot
   worth attaching. Cap at about six images; beyond that the builder starts
   averaging them into mush.
3. `create_project` with the full build message as `initial_message`, and
   **`wait: true`**. Send the whole thing at once — a large first message
   produces a far more coherent app than the same content dripped in over
   follow-ups, because the builder plans the data model against everything it
   knows.

   Do not use `wait: false` here. Observed on a real run: with `wait: false` the
   project scaffolds, the message comes back `status: "accepted"`, the project
   reports `status: "completed"` and `agentFinished: true` within about 20
   seconds. `list_files` shows only the bare shadcn starter.

   **A first build takes 10+ minutes and the read APIs show almost nothing while
   it runs.** During an active build, `list_messages` returned only the original
   user message with no assistant reply, a follow-up `send_message` never
   appeared in the thread at all, and `get_workspace` returned no credit balance
   field. Meanwhile the Lovable web editor showed the agent working continuously
   the whole time.

   So: **never diagnose a build from the API alone.** Every read signal here is
   consistent with "nothing is happening" and with "building normally," and
   there is no way to tell them apart from outside. On a real run this led to a
   confident and completely wrong conclusion that the account was out of
   credits.

   **The one API call that does tell the truth:** `get_message` with BOTH
   `message_id` and `thread_id` (both returned by `send_message`). That reports
   `status: "running"` while the agent works and `"completed"` when it is done.
   Poll that. Without `thread_id` it is useless, and `list_messages`,
   `get_project` and `list_files` are all unreliable mid-build.

   Failing that, the editor URL in a browser always shows the truth. Open
   `https://lovable.dev/projects/<id>` and look. If a build seems stalled, wait
   and look at the editor before sending anything — a duplicate `send_message`
   into a live build wastes credits and can confuse the agent mid-task.

   **Running out of credits does not look like silence.** It returns an explicit
   error naming the problem and linking to the billing page. So silence is never
   evidence of a credit problem — if there is no error, the build is running.
   Budget roughly: one full app build plus a landing page pass exhausted a free
   workspace. Measured cost of a landing page pass on a real run: 4.9 credits.
4. `render_project_widget` with the returned `projectId` so build progress is
   visible.
5. `enable_database` — needed for real auth and persisted seed data.
6. `send_message` with the seeding instruction as a second pass. Seeding is
   separate deliberately: asking for structure and content in one message
   reliably produces thin data.
7. `deploy_project`.
8. Write `preview_url` and the demo logins into `LINK.md`.

Use `get_diff` or `list_files` to check what was built before deploying if the
build looks wrong. Use `plan_mode=true` on `send_message` when a revision is
ambiguous enough that it is worth agreeing on an approach first.

## Message template

Fill every section. Length is fine — detail is what makes the difference here.

```
Build a working prototype of <product name> for <client business>.
This is a demo for a sales call. It must look finished and be fully
clickable. No real integrations.

## The business
<two or three sentences, in their language>

## The problem
<what they said is broken today, quoted where possible>

## Roles
<role>: sees <modules>. <what they do>
<role>: sees <modules>. <what they do>

## Landing page
Build a placeholder landing page only — it gets rebuilt properly in pass 2.

## Sign-in
Supabase auth. On the login page add a "Demo accounts" section with one
button per role that fills the credentials and signs in with one click:
<role> — <email> / <password>

## Modules
<name>: <list/detail/create/edit>. Fields: <fields>. Depth: full | shell

## The connected flow
<the one flow that must work end to end, spelled out step by step>

## Assumed — label these in the UI
<module>: we assumed <assumption>

## Not connected — show the screen, label it clearly
<integration>: show what it would look like, mark as not connected

## Design
<paste the direction from design.md, with the client's palette>
See attached references: Clay for landing page visuals, Linear for app UI.

## Also
Dark and light toggle. Fully mobile responsive. Global search across
<entities>. Role-appropriate notifications.
```

## Landing page message — pass 2

Sent after the app exists, with the reference images attached again. Scope it
explicitly to `/`, or the builder will wander into the app and undo work.

```
Rebuild ONLY the public landing page at `/`. Do not change the app behind
auth — it is finished and correct.

Look carefully at the two attached screenshots. clay-landing.jpeg is the
visual reference, linear-landing.jpeg is the layout reference. Match what
you see in them, do not approximate from my description.

## Headline
<four to seven words, an outcome — see design.md>
Subheading: <one sentence>

Look at Clay's headline in the attached screenshot: "Build systems to grow
revenue." Copy that shape.

## Sections — build all of these, top to bottom
<the ten-section structure from design.md, with this client's content>

Every section that can show the product must show the product. Use real
screenshots of the actual app screens, not invented mockups.

## Visual direction
<from design.md, with the client's palette>

## Still forbidden
<the hard nos from design.md>

Mobile responsive throughout.
```

Naming the current headline and saying it is wrong works better than only
supplying the replacement — the builder otherwise tends to keep its own copy
and reword around it.

## Seeding message

Only sent when the row counts in SKILL.md step 4 come up short:

```
Seed the database with realistic demo data so no screen is ever empty.

<primary entity>: 8-12 records. Use these names and details from the
client's own material: <list from transcript>
<transactional entity>: 20-30 records spread across the last two weeks,
in a realistic mix of statuses
Users: one per role, matching the demo login buttons
Activity feed: two weeks of backdated entries
Dashboard counts must be computed from this data, not hardcoded.

Use the client's real vocabulary throughout: <their terms>
No lorem ipsum, no "Client 1", no placeholder emails.
```

## Failure modes

**Build looks generic.** The design section was too short or the references
weren't attached. Send a follow-up naming what is wrong specifically — "the
landing page hero is a purple gradient, replace it with a solid brand-colour
block" beats "make it look better".

**Data is thin.** The seed message ran before `enable_database` finished, or was
folded into the first message. Re-send it alone.

**Auth doesn't work.** `enable_database` was skipped. Auth silently degrades to
a fake form that looks fine until someone clicks it — always click through the
login before handing over a link.

**Too many modules, all shallow.** The brief marked everything full depth. Go
back to the brief and mark one module full, the rest shell.

**Landing page thin or generic.** Almost always one of two causes: the reference
images weren't attached to the message that actually built it, or the landing
page shared a message with the app. Both are covered at the top of this file.

## What reliably works

Recorded so it doesn't get refactored away by someone tidying up:

- **One large first message.** The app came out coherent from a single detailed
  build message covering roles, modules, depth and the connected flow together.
  Don't break the app build into small messages.
- **A vocabulary table in the brief.** Feeding the client's exact terms produced
  SOP guides titled "Injector Pump Won't Prime", "Phase Converter Won't Come
  Online" and "New Pivot Activation — Start Wet", plus a demo technician named
  after someone mentioned in passing on the call. This is the single cheapest
  thing that makes a prototype feel built for one client.
- **Marking depth per module.** `full` versus `shell` in the brief carried
  through accurately — the full module got 34 steps, the shells stayed thin.
- **Letting the builder improve copy.** It rewrote a specified headline into a
  better one on its own. Keep the improvement; only override when it is wrong,
  and say explicitly what is wrong with the current version.
