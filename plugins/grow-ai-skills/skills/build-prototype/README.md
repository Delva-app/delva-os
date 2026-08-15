# build-prototype

A Claude Code skill. Turns client call transcripts and requirement documents
into a live clickable prototype — landing page, working sign-in, seeded demo
data — built and deployed through Lovable, returning a URL you can send.

Built for turning sales calls into something a prospect can open and click
before they've paid for anything.

## Install

Copy the `build-prototype` folder into either:

- `~/.claude/skills/build-prototype` — available in every project
- `<your-project>/.claude/skills/build-prototype` — that project only

Keep the folder name as `build-prototype`; it has to match the `name` in
`SKILL.md`.

## Connect Lovable

The skill builds through Lovable's MCP server. Once per machine:

```bash
claude mcp add --transport http -s user lovable https://mcp.lovable.dev
```

Then run `/mcp` inside Claude Code, choose `lovable`, and sign in when the
browser opens. OAuth is the only supported method.

Builds run in your own Lovable workspace and spend your credits.

## Use

Make a folder for the client, drop everything they gave you into `input/`:

```
acme-corp/
  input/
    discovery-call.txt
    requirements.pdf
    their-current-dashboard.png
```

Then:

```
/build-prototype acme-corp
```

Claude reads it all and writes `brief.md` — roles, modules, the one flow that
has to work, its assumptions, and the client's own vocabulary. **Read and edit
it.** You know things about the client that were never said on the call.

Run the command again and it builds, seeds, deploys, and writes the live URL
and demo logins to `LINK.md`.

For a second round, put the client's feedback in `revisions.md` and run it
again. It edits the existing project rather than rebuilding.

## What's in here

| File | What it's for |
|---|---|
| `SKILL.md` | The workflow. Loaded when the skill triggers. |
| `references/archetype.md` | The seven-part shape every one of these prototypes takes |
| `references/design.md` | Design direction — what to copy, what to avoid |
| `references/lovable-build-message.md` | MCP call order, message templates, failure modes |
| `assets/brief-template.md` | The structure of a brief |
| `assets/design-refs/` | Clay and Linear screenshots, attached to every build |

## Swapping the design references

`assets/design-refs/` holds two screenshots that get attached to every build:
Clay for landing page visuals, Linear for app UI. They're what stop the output
drifting into generic AI-template styling.

To change the house style, replace those images and update
`references/design.md` to describe what to take from the new ones. Everything
else stays as it is.

## Known gotchas

All learned the hard way on real runs, all documented in full in
`references/lovable-build-message.md`:

- **A first build takes 10+ minutes and the MCP read APIs show nothing useful
  while it runs.** Message threads look empty, project status reports
  `completed`, and the commit SHA doesn't move. None of that means the build
  failed. Open the Lovable editor in a browser to see the truth.
- **Running out of credits is not silent.** It returns an explicit error with a
  billing link. So silence never means credits — if there's no error, it's
  building.
- **Never send a second message into a build that might still be running.** It
  spends credits and can derail the agent.
- **The reference images must be attached to every message that changes how
  anything looks.** Describing them in words does nothing — the builder has
  never seen Clay or Linear and will quietly use its own defaults.
- **The landing page needs its own pass, after the app.** Share a message with
  the app and it comes out as three thin sections, which is the first thing the
  client sees.

## Cost

Roughly one app build plus one landing page pass exhausts a free Lovable
workspace. Budget a paid plan per prototype, plus headroom for revision rounds.

## Limits

Prototypes only. No real integrations — anything the client asked to connect to
gets a screen showing what it would look like, labelled as not connected. No
production auth hardening, no real data, no compliance claims.
