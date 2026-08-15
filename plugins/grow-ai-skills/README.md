# Grow AI Skills

One plugin holding the skills Grow AI actually runs. Install once, get all three.

## Skills

| Skill | What it does |
|---|---|
| `proposal-from-transcript` | Turns a sales call transcript into a finished, branded HTML proposal with scope and pricing. |
| `build-prototype` | Turns a client call or requirements doc into a live clickable prototype (landing page, sign-in, seeded demo data) deployed via the Lovable MCP, and returns a URL. |
| `meta-campaign-builder` | Turns a folder of creatives into a complete Meta campaign — account preflight, all settings asked up front, media uploaded, campaign/ad set/creatives/ads created **paused**. |

## MCP servers

Ships `.mcp.json` with the Meta Ads MCP (`https://mcp.facebook.com/ads`), used by `meta-campaign-builder`. `build-prototype` additionally expects the Lovable MCP to be configured on the host.

## Install

```
/plugin marketplace add <this repo>
/plugin install grow-ai-skills@delva-os
```

## Adding to it

Drop a new skill folder into `skills/` — each is a self-contained directory with `SKILL.md` plus any `assets/` and `references/`. No manifest edit needed; the plugin picks up whatever is in `skills/`.
