---
name: meta-campaign-builder
description: >-
  Build a complete Meta (Facebook/Instagram) ad campaign from raw creative
  files through the Meta Ads MCP: run account preflight checks, ask every
  setting question up front as clickable multiple choice, host and upload the
  media, then create campaign, ad set, creatives and one ad per creative, all
  PAUSED. Use this skill whenever the user says "create me a campaign", "build
  a Meta campaign", "launch these ads", "set up a Facebook campaign", "make an
  ad campaign from these videos", or drops a folder of ad creatives and wants
  them turned into ads. ALSO use when the user has creatives ready and is
  asking what settings they need to choose. Ad copy is NOT drafted here: the
  user supplies primary text and headlines, or a dedicated copy skill does.
  Do NOT use for reporting on existing campaigns, editing a live campaign's
  budget or targeting, pausing or activating ads, or generating the creative
  files themselves.
---

# Meta Campaign Builder

Building a Meta campaign through the API fails in a specific, expensive way:
you discover blockers one at a time, at the end, after doing all the work.
Account has no payment method. Page never accepted Lead Gen ToS. Upload tool
isn't enabled on that account. Each one surfaces only when you attempt the
call that trips it, and several only appear at the very last step. Meanwhile
ad creative copy is immutable, so a wrong answer means rebuilding every
creative and every ad.

This skill front-loads all of it: check the account before building, ask
every question in one batch, then execute without stopping. The user answers
once and gets a finished campaign.

## Core rules

1. **Everything is created PAUSED. Never activate.** Every create tool returns
   an "Activate" affordance. Ignore it. Activation is the user's decision,
   made in Ads Manager after they have reviewed the build.
2. **Run preflight before asking questions.** Blockers you find in preflight
   change which questions are worth asking. Discovering them afterward wastes
   the user's answers.
3. **Confirm copy before building more than one creative.** Creative copy,
   media, link and CTA are immutable after creation. Changing any of them
   means new creatives plus new ads.

## Prerequisites

The Meta Ads MCP connected and authorized is the main requirement, but it is
not the only one. Check these before promising a build, because three of them
are outside the MCP entirely and one is outside the user's control.

**Handled by the skill, not the user:**

- **Media hosting.** The upload tools take a public URL, never a local path.
  Set this up automatically via `gh` — see `references/media-hosting.md`.
  Read that file whenever the creatives are local, which is nearly always.
  Do not ask the user to arrange hosting; just do it and tell them it is
  temporary and public.

  That reference also covers installing `gh` when missing and getting the
  user through `gh auth login` when it is unauthenticated, so "GitHub isn't
  connected" is a setup step, not a blocker.

**Installed by the skill if missing:**

- `ffmpeg` / `ffprobe` — reads creative dimensions and extracts poster
  frames. If absent, install it with the platform package manager rather
  than asking. It is a standard media tool, and the alternative is a
  visibly worse thumbnail.

  ```bash
  command -v ffmpeg >/dev/null || {
    case "$(uname -s)" in
      Darwin) brew install ffmpeg ;;
      Linux)  sudo apt update && sudo apt install -y ffmpeg ;;
    esac
  }
  ```

  Say what you are installing before you run it. If the install fails, needs
  a password you cannot supply, or no package manager exists, **do not
  block** — fall back to the Meta-generated `picture` thumbnail described in
  Step 4 and note the quality tradeoff. The build still completes.

**Needed on the Meta side, and only ever once per account:**

- A **payment method on the specific ad account**, or ads cannot be created.
- **Lead Gen ToS accepted** on the Page, for instant-form campaigns.
- **An existing lead form**, for instant forms. No MCP tool creates one.
- **A pixel**, for website-conversion campaigns.

These are one-time account setup, so hitting one mid-build is normal and not
a failure of the workflow. Surface it with the exact fix and the
account-pinned URL, then continue once it clears. Only the payment method is
worth flagging in preflight, because it blocks the very last step after all
the work is done.

**Outside anyone's control:** per-account tool rollout. Some tools are
enabled on some ad accounts and not others, and no permission grant changes
it. Detect it in preflight and route around it by choosing another account.

**On OAuth scope:** the accounts and Pages you can see are exactly what the
user authorized. If a Page they expect is missing from `ads_get_user_pages`,
the likely cause is a narrow grant at connect time, not a missing Page — have
them reconnect and allow access to all Pages and ad accounts. Do not conclude
a Page does not exist because it is absent from the list.

## Step 1: Preflight

Never skip this. Every check maps to a blocker that otherwise surfaces at the
worst moment.

Call `ads_get_ad_accounts` and read, for each account:

- `has_payment_method` — **if false, ad creation fails entirely.** Not just
  spending: the API publishes real objects and validates funding at create
  time. Campaigns and ad sets still create fine, which makes this look like a
  late-stage bug. Ads Manager appears to work without billing because the UI
  builds *drafts*, and the Marketing API has no draft status (only ACTIVE,
  PAUSED, DELETED, ARCHIVED). There is no workaround. The user must add a
  card to that specific ad account.
- `is_ads_mcp_enabled` and `is_queryable` — skip accounts where these are false.
- `account_status` — must be ACTIVE.
- `currency` and `min_daily_budget_cents` — budgets are integers in the
  currency's minor unit, so $30/day is `3000`.

Then probe the **per-account tool rollout**, which is real and silent. Some
tools are enabled on some ad accounts and not others in the same user's list.
The give-away is an unstructured error with no `error_code`:

```
"This tool is new and is being gradually rolled out across ad accounts."
```

That is the MCP declining, not Meta. Contrast with a genuine API rejection,
which arrives as structured JSON with `error_category` and `error_code`.
Distinguishing these matters: a rollout gate can be routed around by choosing
a different account, a Meta rejection cannot.

Probe by calling `ads_get_ig_accounts` on the candidate account. If it returns
the rollout message, assume `ads_creative_upload_video` and
`ads_creative_upload_image` may also be gated, and confirm with a single real
upload before committing to that account.

**Always pull the Page list live with `ads_get_user_pages`, on every run,
before offering the Page question.** Never populate it from memory, from an
earlier conversation, or from anything written in a project file. This skill
is most often run for a client whose Page you have never seen, and Page
access changes as clients are added and removed. A remembered Page ID is
either stale or belongs to someone else's campaign, and passing the wrong one
binds every creative to the wrong brand — which is unfixable by editing,
since creatives are immutable.

The same applies to ad accounts: pull them fresh with `ads_get_ad_accounts`
every run, since billing state, account status and tool availability all
change between sessions.

Note that
`ads_get_ad_account_pages` frequently returns `[]` even when the user owns a
usable Page, and **a Page ID that isn't listed under the account still works
when passed directly**. Do not treat an empty list as "no Page available."
`leadgen_tos_accepted` is only visible via `ads_get_ad_account_pages`, so when
that returns empty you cannot check ToS ahead of time and must let the ad set
create attempt reveal it.

Report preflight findings before asking questions, and if a hard blocker
exists (no payment method especially), say so up front rather than building
toward a wall.

## Step 2: Ask everything at once

### First, harvest what they already said

Before composing any question, re-read the user's own messages and pull out
every setting they already specified. People front-load detail in the opening
request — "create me a campaign, objective is leads, $50 a day, US only, here
are the videos" — and being asked those same things back is the fastest way
to make a skill feel broken.

Then read the files rather than asking about them. `ffprobe` gives dimensions,
duration and codec, which tells you the aspect ratio and placement fit. Count
the files. Note their names.

Print a short settings list showing what is already known and what is still
missing, so the user can correct a misreading before answering anything:

```
From what you've given me:
  Creatives     10 videos, 1080x1920 vertical, ~94s
  Objective     Leads
  Budget        $50/day
  Countries     US
  Ad account    Acme Main (only one with billing)

Still need: Page, placements, enhancements, CTA, destination URL, copy
```

If they contradict themselves across messages, surface it here rather than
silently picking one.

### Then ask only what's left

Use `AskUserQuestion` so answers are clickable. **Maximum 4 questions per
call**, 2-4 options each. Put your recommendation first and mark it
`(Recommended)`.

**Never ask a question the user has already answered**, whether they answered
it in their opening message, in an earlier turn, or implicitly through the
files. Drop those questions entirely rather than asking for confirmation. A
setting they stated is settled; re-asking reads as not having listened.

Drop questions preflight already resolved too — if only one ad account is
usable, that is not a question.

This means the number of calls varies. A user who specified nothing gets two
full calls. A user who front-loaded most of it may get one call with three
questions, or none at all. Build immediately once nothing is missing.

The tables below are the **candidate** questions, not a script. Ask only the
ones still open after harvesting, and collapse the rest.

**Call 1 — where it goes and what it does.** Ad account and Page come first,
because everything downstream is scoped to them: the media uploads into a
specific account, the creatives bind to a specific Page, and getting either
wrong means rebuilding rather than editing.

| Question | Options |
|---|---|
| Ad account | Every usable account from preflight, labelled with its blockers |
| Page | Every Page from `ads_get_user_pages` |
| Objective | Leads / Traffic / Sales / Engagement / Awareness |
| Daily budget | Offer 2-3 sensible amounts plus "Other" for typing |

Label each account option with what preflight found — "no billing, cannot
create ads", "upload gated" — so the choice is informed rather than a guess
at which name is right. Users routinely have several similarly named accounts
across several businesses, and picking the wrong one is the single most
common way this workflow goes sideways.

If only one account is usable, say so and skip the question rather than
offering a menu of one.

**Call 2 — the ad-level settings:**

| Question | Options |
|---|---|
| Placements | Facebook + Instagram / Facebook only / Instagram only |
| Countries | Their likely market, plus Other |
| Advantage+ enhancements | All off / All on / Meta's default |
| Call to action | Learn More / Sign Up / Get Quote / Book Now |

Ask for the destination URL in prose alongside these, since it is typed
rather than picked.

**Ad copy is out of scope for this skill.** Primary text, headlines and
descriptions need brand voice, offer detail and examples that this workflow
does not carry. Ask the user to supply them, or hand off to a dedicated
copy skill if one is installed. Do not invent claims, numbers or positioning
to fill the gap — copy is immutable once built, and a fabricated claim is
worse than a blank one. If the user has no copy ready, build nothing and say
what you need.

If objective is Leads, ask which kind, because the two paths have completely
different requirements:

- **Instant forms** (`LEAD_GENERATION`) — needs the Page to have accepted Lead
  Gen ToS **and** an existing lead form. There is no lead-form tool in the
  MCP, so the user must create the form in Ads Manager and supply its ID.
- **Website leads** (`OFFSITE_CONVERSIONS`) — needs a pixel ID in
  `promoted_object`. The API rejects the ad set without one.

If neither is available, say so and offer `LINK_CLICKS` traffic as a working
fallback under the same `OUTCOME_LEADS` campaign. It builds today and the
optimization goal can change later.

## Step 3: Host the media

**The upload tools accept a public URL only. There is no local file
parameter.** The server fetches the bytes itself, unauthenticated. Google
Drive, Dropbox and similar share links fail because they return an HTML
interstitial instead of raw bytes.

**Read `references/media-hosting.md` and follow it.** It carries the full
procedure: repo creation, the seed commit a release requires, videos as
release assets, images committed for `raw.githubusercontent.com`, the
verification checks, and cleanup timing. Set hosting up automatically rather
than asking the user to arrange it, and skip this step entirely if their
media is already at public URLs.

The condensed version, for reference:

```bash
gh repo create <owner>/<name> --public -d "temp host for ad creatives"
gh api repos/<owner>/<name>/contents/README.md -X PUT \
  -f message="init" -f content="$(printf 'init\n' | base64)"
gh release create v1 --repo <owner>/<name> -t "creatives" -n "temp"
gh release upload v1 *.mp4 --repo <owner>/<name>
```

A release must have at least one commit to exist, hence the README step.
Verify the owner with `gh api user --jq .login` — the CLI's stored account
label is often not the API login, and a mismatch 404s.

**Videos and images need different hosting, and this is the trap.** Release
assets redirect to a signed URL. Meta's *video* fetcher follows the redirect;
Meta's *image* fetcher does not, and reports:

```
Image Wasn't Downloaded: ... blocked by a robots.txt
```

So serve videos from release assets, and **commit images into the repo** and
serve them from `raw.githubusercontent.com`, which returns `image/jpeg`
directly with no redirect.

Verify before uploading. One `curl -sIL` saves a failed batch:

```bash
curl -sIL "<url>" | grep -iE "^HTTP|^content-type|^content-length"
```

Note that repo creation may be blocked by permission policy. If so, hand the
user the exact command to run themselves rather than abandoning the approach.

## Step 4: Upload to the ad account

Upload each video with `ads_creative_upload_video`. It returns immediately
with `video_status: "uploading"` — encoding is async. Poll `ads_get_ad_videos`
with the `video_ids` until each reads `ready`. Creating a creative against a
still-processing video is unreliable.

**Video creatives require an explicit thumbnail.** `ads_create_creative`
rejects a video with no `image_hash`/`image_url`:

```
At least one of image_hash or image_url must be provided.
```

There is no auto-generation on this tool. Two ways to supply one:

**Preferred, when `ffmpeg` is available** — extract a real poster frame at
full resolution:

```bash
for f in *.mp4; do
  ffmpeg -v error -ss 1 -i "$f" -frames:v 1 -q:v 2 "thumbs/${f%.mp4}.jpg" -y
done
```

Seek to ~1s rather than 0, since the first frame is often black. Then host
the JPGs per `references/media-hosting.md` and pass the raw URLs.

**Fallback, no `ffmpeg` needed** — once a video reads `ready`, request the
`picture` field from `ads_get_ad_videos` and pass that URL straight into
`ads_create_creative` as `image_url`. Meta accepts its own CDN URL, and this
skips thumbnail extraction and image hosting entirely.

The tradeoff: `picture` returns a small cropped render (around 160px square),
not a full-resolution frame from a vertical video. It creates a valid
creative, but the poster image is low resolution and centre-cropped. Use it
when `ffmpeg` is unavailable or the user wants speed; prefer real frames when
thumbnail quality matters.

## Step 5: Build

Create in order, verifying each before proceeding. All create PAUSED by
default.

**Campaign** — `ads_create_campaign`. Prefer CBO: set `campaign_daily_budget`
on the campaign rather than per ad set, unless the user asked for ad-set
budgets. The response includes `valid_optimization_goals` and
`recommended_optimization_goal` for the chosen objective. **Use those**
rather than guessing, since compatibility is enforced server-side.

**Ad set** — `ads_create_ad_set`. Put placements in `targeting`:

```json
{"geo_locations":{"countries":["US"]},
 "publisher_platforms":["facebook","instagram"]}
```

**Never invent interest IDs.** They are real 13-16 digit values from the
Targeting Search API, which this MCP does not expose. Fabricated IDs are
rejected. Use broad geo targeting and say plainly that the targeting does not
encode the user's stated audience — the copy is doing that work. Users
routinely assume "target IT firms" produced interest targeting. It did not.

**Creatives** — `ads_create_creative`, one per media file.

To turn Advantage+ enhancements **off**, use `degrees_of_freedom_spec`. It is
a **JSON-encoded string, not a nested object**, and it is **mutually exclusive**
with `advantage_plus_creative` and `advantage_plus_creative_features` —
passing both is rejected. The shortcuts cannot express opt-out, only opt-in,
so this field is the only way.

These 11 feature names are valid and verified:

```
image_touchups, text_optimizations, image_animation, video_auto_crop,
enhance_cta, adapt_to_placement, media_type_automation,
description_automation, site_extensions, profile_card, text_generation
```

`translate_text`, `music` and `cta_optimization` are rejected as invalid.
`standard_enhancements` is rejected as deprecated — individual features only.
Meta names invalid features precisely in the error, so if the valid set has
shifted, **build one probe creative first**, read the rejection, adjust, then
build the rest. One creative of risk instead of N.

For Instagram delivery, pass `instagram_user_id`. Without it the creative
will not deliver on Instagram surfaces even when the ad set targets Instagram
— created ads report `page_platform: FACEBOOK`. If `ads_get_ig_accounts` is
gated, tell the user Instagram delivery is not wired rather than assuming
Meta will fall back to the Page's linked account.

**Ads** — `ads_create_ad`, one per creative, referencing
`{"creative_id":"<id>"}`.

## Step 6: Verify and report

Query `ads_get_ad_entities` at ad level for `status` and `effective_status`.
Every ad should read `status: PAUSED`. `effective_status` showing
`PENDING_REVIEW` or `IN_PROCESS` is normal — Meta reviews paused ads too, and
this is not delivery.

Report as a table: campaign, ad set, ad count, creative count, budget,
placements, status. Then state the honest gaps. At minimum:

- **Opt-outs cannot be read back.** `degrees_of_freedom_spec` is not in
  `ads_get_creatives`'s readable fields. Acceptance without error is evidence,
  not confirmation. Tell the user to eyeball one ad in Ads Manager.
- Whether Instagram is actually wired.
- That targeting is broad unless real interest IDs were supplied.
- Temporary hosting should already be **deleted or made private by now**.
  Meta copies both video and thumbnail bytes into its own storage at
  creation time, so nothing downstream reads the source URLs. Do this as
  soon as the last creative exists and report it as done, rather than
  leaving a public repo of a client's unreleased creatives online.

## Known blockers, and what they actually mean

| Error | Real meaning | Fix |
|---|---|---|
| `No Payment Method` (subcode 1359188) | Account has no card. Blocks *creation*, not just spend | User adds card to that exact ad account |
| `Terms of Service Not Accepted` (1815089) | Page hasn't accepted Lead Gen ToS | facebook.com/legal/leadgen/tos on the right Page |
| `This tool is new and is being gradually rolled out` | MCP-side gate, no error code | Try another ad account |
| `Invalid Advantage+ creative feature "X"` | Name not valid for this account | Drop the named features, retry |
| `Creative should not include standard enhancements` | `standard_enhancements` deprecated | Use individual features |
| `Image Wasn't Downloaded ... robots.txt` | Image fetcher won't follow redirects | Serve from a direct URL |
| `At least one of image_hash or image_url` | Video creative needs a thumbnail | Extract a poster frame |
| `Unknown argument(s): client_conversation_id` | Not every tool accepts it | Drop it for that call |

A recurring cause of confusion worth pre-empting: users have several ad
accounts across several businesses with similar names. Billing changes,
ToS acceptances and Page permissions land on the wrong one constantly. When a
user says they fixed something and the API disagrees, **re-read
`ads_get_ad_accounts` and diff it against the previous read** — the changed
row shows where their fix actually landed. Give them account-pinned URLs
(`?act=<id>`) rather than generic ones.

## When NOT to apply

Do not use for reporting, optimization, or editing live campaigns. Do not use
when the user wants a single ad added to an existing ad set — that is one
`ads_create_ad` call, not this workflow. Do not use to generate creative
files.

Under-triggering costs a user the preflight checks and they hit blockers one
at a time, which is the exact failure this exists to prevent. Over-triggering
runs a heavy question flow over a one-line request. Under-triggering is worse
here, so lean toward firing when creatives are present and a campaign is
implied.

## Example

**User:** drops a folder of 8 MP4s and says "create me a campaign, going for
leads, 50 a day, US"

**Preflight finds:** two ad accounts, one with `has_payment_method: false`,
one gated on `ads_get_ig_accounts`. One Page, not listed under either account.

**Harvest and report first:**

> From what you've given me:
>   Creatives     8 videos, 1080x1920 vertical, ~30s
>   Objective     Leads
>   Budget        $50/day
>   Countries     US
>   Ad account    Acme Main (Acme Secondary has no billing, can't create ads)
>
> The IG account lookup is gated on Main, so Instagram delivery may not wire
> up. Your Page isn't listed under either account, which is normal and
> usually still works.
>
> Still need: Page, leads type, placements, enhancements, CTA, URL, copy

Objective, budget, countries and ad account are settled, so **none of them
appear in the questions**. Page is the only survivor from Call 1, so it folds
into a single call alongside leads type, placements and enhancements, with
CTA and URL following.

**Ask for copy separately**, since this skill does not draft it.

**Then build without stopping**, and close with the table plus the honest
gaps: opt-outs unverifiable, Instagram unconfirmed, targeting broad.
