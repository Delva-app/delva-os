# Delva OS

A marketplace of skills. The main one builds you an AI operating system:
a `CLAUDE.md` your agent reads first every session, a real knowledge
directory behind it, and a voice file so anything written as you
actually sounds like you. Others do specific jobs.

## Install

Add the marketplace once:

```
/plugin marketplace add Delva-app/delva-os
```

Then install whichever plugins you want:

```
/plugin install delva-os@delva-os                  # the OS builder
/plugin install meta-campaign-builder@delva-os     # Meta ad campaigns
```

Restart Claude Code, or run `/plugin` to confirm they're enabled. Skills
work in every project once installed.

To pick up later updates, run `/plugin marketplace update delva-os`.

If you'd rather not use the plugin system, copy the skill folders under
`plugins/*/skills/` into `.claude/skills/` in your project. Same result,
but per project and without updates.

## Plugins

### `delva-os`

The OS builder: `/build-my-os`, `/build-my-voice`, `/os-audit`. Details
below.

### `meta-campaign-builder`

Turns a folder of ad creatives into a complete, paused Meta campaign.
Runs account preflight first so blockers surface before you answer
anything, asks every setting up front as clickable multiple choice, then
hosts and uploads the media and builds the campaign, ad set, creatives
and one ad per creative. Nothing is ever activated.

It bundles the Meta Ads MCP, so installing the plugin wires up the
connection too. You'll still authorize it in the browser.

It does not write ad copy — you supply primary text and headlines.

Start it with "create me a campaign" and drop your creatives in.

## What's in `delva-os`

### `/build-my-os`

The main one. Run it in a fresh folder and it builds the whole setup in
one sitting:

- Sets up the directory: `raw/` for source material you drop in,
  `wiki/` for the knowledge base your agent maintains, `index.md`,
  `log.md`, and `.claude/skills/`.
- Writes `CLAUDE.md` as a **router** — the schema, conventions, and
  workflows. Not a dump of your business facts. Those become pages.
- Interviews you, one question at a time, about your offer, your ideal
  client, your bottleneck, how you want to be spoken to, and your hard
  rules. Each answer gets filed as a wiki page as you go.
- Hands off to `/build-my-voice`.
- Installs `/os-audit` into your project.
- Wires up whatever tools you have connected.
- Seeds Claude Code's persistent memory with how you work.
- Turns on Remote Control so you can reach your setup from your phone.

Run it in the folder you want to be your second brain. Not in a code
repo.

### `/build-my-voice`

Builds a voice file from your own words — your Claude or ChatGPT export,
meeting transcripts, sent emails. Pulls out your sentence patterns, the
words you actually use, the ones you'd never use, and a bank of 30+
verbatim quotes. Anything written as you reads through it afterwards.

It needs real material from you. It will say so plainly rather than
inventing a voice that isn't yours.

Runs automatically as a step inside `/build-my-os`, or on its own to
refresh an existing voice file with new material.

### `/os-audit`

Read-only health check on a setup that already exists. Finds routing
that points at files that aren't there, stale numbers, duplicate
folders, orphan pages. Run it every month or so, or whenever your agent
starts missing things it should know.

## The idea

Most people use AI like a slot machine: open a blank chat, paste a
question, hope. It forgets you the moment the tab closes.

The fix is a file it reads first, every time, plus a directory behind
that file with your actual context in it. Then you're not re-explaining
your business every session.

The setup this builds is yours. Plain markdown in a folder on your
machine. No account, no lock-in, and you can read every file in it.
