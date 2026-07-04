---
name: director-02-app-blueprint
description: Create and maintain the screen map, navigation graph, screen states, and user journeys for an app — the director-level design contract that precedes any build. Use when the user says "blueprint", "screen map", "what screens do we need", "plan the app UX", "navigation flow", when kicking off a new app after decisions are recorded, or when a feature or redesign changes which screens exist or how they connect. Also use in amend mode when an approved director-07-feature-impact map requires blueprint changes.
---

# App blueprint

This skill is part of a director-led pipeline: the user decides and approves; agents execute. The blueprint is the design contract — the complete inventory of screens, their states, how navigation connects them, and the journeys users take through them. Everything downstream (mockups, delegation briefs, fidelity gates) references the blueprint by stable screen IDs. Nothing gets built that isn't on the map.

## Before doing anything

1. Read `product/STATE.md` and everything under `product/blueprint/` if present. If a blueprint exists, you are in **amend mode**: change only what the request requires, preserve everything else, and bump the version with a changelog entry. Never regenerate an approved blueprint from scratch — that destroys the contract downstream artifacts point at.
2. Read `product/decisions/index.md` and comply with all accepted records. If no decisions exist, recommend running the director-01-decision-record skill first — a blueprint drawn before platform and payment decisions usually has to be redrawn.
3. Read any PRD or product notes in the repo (`product/`, `docs/`, README) so the blueprint reflects what the product is actually for.

## Stable IDs — the one unbreakable rule

Screens get IDs `S-01`, `S-02`, … assigned once and never reused or renumbered. Briefs, mockups, and gate reports reference these IDs; renumbering silently corrupts every downstream artifact. A removed screen keeps its ID with status `retired`. New screens take the next free number.

## Artifacts

All under `product/blueprint/`:

**`blueprint.md`** — header + screen inventory:

```markdown
# Blueprint — <app name>

- Version: N
- Status: draft | approved
- Approved: YYYY-MM-DD (by the user, only after explicit approval)

## Changelog
- vN (YYYY-MM-DD): what changed and why (reference F-NNN impact maps where relevant)

## Screens

| ID | Name | Purpose | States | Entry points | Status |
|---|---|---|---|---|---|
| S-01 | Onboarding | ... | default, loading, error | app launch (first run) | active |
```

Every screen lists its states explicitly — `default` plus whichever of `empty`, `loading`, `error`, `offline` apply. Missing states are the single most common thing agents forget to build and directors forget to ask for; listing them here is what makes the fidelity gate able to check them later.

**`navigation.md`** — the navigation graph as a Mermaid flowchart, using screen IDs as node labels, with the structure annotated (tab bar, stack, modal, interrupt). Keep it renderable: one graph for the main structure, separate small graphs for complex sub-flows rather than one giant tangle.

**`journeys.md`** — ordered walks through the graph:

```markdown
## J-01 — First run to first value
S-01 → S-03 → S-04
Success: user reaches <core value moment> without creating an account.
```

Journeys must cover at minimum: first-run, the core value loop, the payment/upgrade path, and account recovery. Each journey states a success criterion — the fidelity gate walks these journeys verbatim.

## Working method

Draft the complete screen inventory before drawing navigation — screens missing from the inventory are how scope leaks in later. When drafting, actively hunt for screens product people forget: settings, legal/privacy, permission priming, error and offline states, the paywall's decline path, account deletion (store review requires it).

## Director review — this is the point of the skill

Present the draft as something a director reviews in minutes, not a spec they must decode:

- Render or describe the navigation graph visually.
- Summarize in plain English: how many screens, the shape of the app, where the paywall sits.
- Call out the scope drivers explicitly: which screens or flows carry most of the build cost, and what could be cut or deferred.
- Invite pushback on scope ("merge these two onboarding screens?") — a ten-minute cut here saves days downstream.

Then stop and wait for explicit approval. Only after the user approves: set `Status: approved`, record the date, and update `product/STATE.md` (Blueprint row: approved, version, date). In amend mode, approval applies to the delta — present the diff, not the whole blueprint.

An unapproved blueprint blocks the pipeline downstream (director-03-design-baseline checks for approval). Do not mark approval yourself, and do not treat silence as approval.
