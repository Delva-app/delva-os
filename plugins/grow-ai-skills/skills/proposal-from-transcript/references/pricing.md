# Pricing model

Two constraints have to hold at the same time:

1. Every total lands between **$15,000 and $25,000**.
2. Removing any single feature drops the price by **$1,000–$3,000**.

Those fight each other if you just add feature prices to a flat base fee: three
easy features would come out under $15k and five very-hard ones would blow past
$25k. The model below satisfies both by fixing the removal values and letting
the base line item float.

## Step 1 — score each feature

Three to five features, each one tier:

| Tier | Value | What it looks like |
|---|---|---|
| **Easy** | $1,000 | One CRUD screen. A form that saves. A static/marketing page. A single simple integration (webhook in, email out). No new data relationships. |
| **Medium** | $1,750 | A multi-step flow. Roles and permissions. A third-party API with real auth (OAuth, API keys, pagination). Filtering/search over their data. Exports/reports. |
| **Hard** | $2,500 | Real-time sync between systems. Billing and payments. A genuinely complex data model. An AI feature that has to be reliable (prompting + evals + fallbacks). Bulk import with validation. |
| **Very hard** | $3,000 | Multi-tenant with data isolation. A custom engine — scheduling, matching, routing, pricing. Migration off a live system without downtime. Anything where being wrong is expensive for them. |

Tier on **implementation risk**, not on how much the client cares about it. A
feature they talked about for twenty minutes can still be Easy, and saying so is
how the base fee stays honest.

Every feature carries a one-line *why this tier* justification. It goes in the
HTML on hover so the number can be defended live on the call.

## Step 2 — the arithmetic

```
feature_sum = sum of tier values                 # $3,000 (3 easy) .. $15,000 (5 very hard)
anchor      = 15,000 + (feature_sum - 3,000) * 0.8333
anchor      = round to nearest $500              # the headline total
base        = anchor - feature_sum               # the "Build & delivery" line item
```

That linear map takes the full possible range of `feature_sum` and lays it
exactly onto $15,000–$25,000. There is no mix of 3–5 features that escapes the
range, and no clamping is needed.

The base slides from about **$12,000** (light scope) down to about **$10,000**
(heavy scope), which is the right direction: heavier builds put more of the
money into the features themselves.

**Base line item = "Build & delivery."** It is real work, not padding, and
the proposal says so: architecture and data model, authentication and accounts,
hosting and deploy pipeline, QA, handoff and documentation, and two weeks of
post-launch support.

## Step 3 — the range check

If the honest read of the transcript is **more than five features**, or the five
you kept are all Hard/Very hard *and* there are more of the same waiting in the
backlog, the project is bigger than $25,000. Do not compress it to fit.

Say so, and propose a phase 1 instead: pick the 3–5 features that make a
shippable thing, price those with the model above, and put the rest in the
"Phase 2" list on the page. Flag it in chat before writing the file so the user
can decide whether to phase it or quote the bigger number outside this skill.

## Worked examples

| Features | feature_sum | anchor | base | Total |
|---|---|---|---|---|
| 3 x Easy | $3,000 | $15,000 | $12,000 | **$15,000** |
| Medium, Medium, Hard, Easy | $7,000 | $18,500 | $11,500 | **$18,500** |
| Very hard, Hard, Hard, Medium, Medium | $11,500 | $22,000 | $10,500 | **$22,000** |
| Hard, Hard, Medium | $6,750 | $18,000 | $11,250 | **$18,000** |
| 5 x Very hard | $15,000 | $25,000 | $10,000 | **$25,000** |

## Removal on the call

One feature comes out at a time. The total drops by exactly that feature's tier
value — $1,000, $1,750, $2,500, or $3,000 — and the base does **not** change.
The base is the cost of building anything at all, so it survives a cut, and
saying that out loud is a fair answer to "why didn't it drop more?"

Clicking the same feature again restores it. The page never lets two features
out at once: a proposal with two things crossed off stops reading as a
concession and starts reading as a different, smaller project.
