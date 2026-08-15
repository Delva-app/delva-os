# The archetype

Read this before writing a brief.

Across real sales calls in unrelated industries — irrigation field service and a
hormone/weight-loss clinic — the prototype that got built was structurally
identical. Different nouns, same skeleton. Both had roles with different menus,
a sidebar, a dashboard of counts, a core record with a detail page and history,
a second linked entity, search, notifications, a dark/light toggle, and an AI
assistant over a knowledge base.

That is not coincidence. It is what a business application is. Treat it as a
template to fill rather than a structure to invent, and the build gets faster
and better at the same time.

---

## 1. Landing page

What the platform is, who it's for, three or four feature blocks, a login
button. Placeholder branding the client redirects later.

Clients read this first and judge the entire build on it. See `design.md`.

## 2. Sign-in

Real Supabase auth through Lovable. Real session, real redirect. Fake auth gets
noticed instantly and undermines everything else.

## 3. Demo account switcher

On the login page, one button per role: "Sign in as Admin", "Sign in as
Technician", "Sign in as Patient". Pre-filled credentials, one click.

This is the highest-leverage detail in the build and it costs almost nothing.
The usual pattern is emailing credentials after the call and hoping the client
logs in later. Most don't. Buttons on the login page remove that gap, and they
let the client see the role structure without being walked through it.

There is a second benefit nobody predicts: **the builder can use them too.** On
a real run, asked for real screenshots on the landing page, the agent launched
Playwright, clicked the demo login buttons to authenticate itself, walked the
Ask flow, and captured live screens of the working app to use as marketing
imagery. Without one-click demo accounts it would have invented mockups. Build
the switcher before the landing page pass for this reason alone.

## 4. Dashboard

Counts, a recent activity feed, org-level stats. Everything clickable through to
the thing it counts.

Dead numbers read as a mockup. A clickable number reads as a system. Wire every
tile through, even to a plain list.

## 5. Three to five core modules

The client's real entities: list view, detail view, create form, edit. Their
words, not generic ones — "Pivots" not "Assets", "Lab Panels" not "Records".

Depth is not equal across modules. The one they complained about gets full
depth. The rest can be list plus detail and nothing more.

## 6. One connected flow

Creating record A pulls from record B and updates what role C sees.

In the irrigation build this was: create a job, pick a client, pick their
equipment, assign a technician, and the technician's own view updates. That
sequence is the moment on a demo call where the client stops seeing screens and
starts seeing a system. Nothing else in the build produces that moment.

Pick exactly one. Make it work properly. Fake the rest.

## 7. Assumption modules

One or two things the material hints at but never confirms, built out and
labelled in the UI as assumed. See the reasoning in SKILL.md — cutting these is
how the client narrows scope, and a client who cuts something has bought in.

---

## Always included

- Dark and light toggle. Clients notice it and comment on it unprompted.
- Mobile responsive. It gets demoed on a phone on the call.
- Global search over the main entities.
- Notifications, role-appropriate.

These are near-free with shadcn and each one is disproportionately noticed.

## Usually worth including

An AI assistant panel over the client's own documents, with an escalation path
to a human when it cannot answer. This appeared independently in both reference
calls and is the single most requested capability in this category of work. If
the material mentions questions, support, protocols, SOPs, or staff answering
repetitive queries, build it.
