---
name: director-07-feature-impact
description: Before building a new feature on an existing app, produce a director-reviewable impact map — which screens and journeys change, what's new, what could regress, whether any standing decision is violated, and a rough size estimate. Use when the user says "add feature X", "what would it take to add", "impact of adding", "can we integrate", or proposes any change to an app that already has a product/blueprint/ directory. New features enter the pipeline through this skill — never straight to briefs or code.
---

# Feature impact

This skill is part of a director-led pipeline: the user decides and approves; agents execute. Its rule is simple: a feature never goes straight to a build brief. First the design absorbs it — an impact map shows the director exactly what the feature disturbs, they approve or reshape it, the blueprint and baseline are amended, and only then do briefs go out. The design stays the source of truth; the code chases it.

## Preconditions

1. Read `product/STATE.md` and everything under `product/blueprint/`. If no blueprint exists, this skill doesn't apply — recommend director-02-app-blueprint (for a new app) instead.
2. Read `product/decisions/index.md`, `product/baseline/manifest.json`, and — if the app has been built — enough of the codebase to ground the regression analysis in reality rather than in the blueprint alone.
3. Read `product/impact/` for prior impact maps; features often touch the same seams, and prior maps record what was fragile last time.

## The analysis

Work through these questions in order, against the blueprint first and the code second:

1. **Surface**: which existing screens (by S-ID) change, which are added (assign new S-IDs as `proposed`), which are provably untouched. "Untouched" claims matter as much as "changed" — they are what lets the director bound the risk.
2. **Navigation**: which edges in the graph are added or altered; does any journey (J-ID) gain, lose, or reorder steps.
3. **Decisions**: does the feature violate any accepted decision record's constraints, or require a decision that doesn't exist yet (new vendor, new data category)? A feature that needs a new decision must say so — that decision goes to the director, not into the code.
4. **Regressions**: which existing journeys share code paths, state, or backend endpoints with the change. List them as a watchlist for the fidelity gate.
5. **Size**: S / M / L with one sentence of rationale (screens touched × new backend surface is a good first-order estimate). Directors allocate by size; agents don't need this number, the director does.

## The artifact

`product/impact/F-NNN-slug.md` (stable IDs, never renumbered):

```markdown
# F-NNN — Feature title

- Status: proposed | approved | rejected
- Date: YYYY-MM-DD

## Summary
Two or three sentences: what the feature does for the user.

## Screens
| ID | Change | Notes |
|---|---|---|
| S-05 | modified | adds share button to completed-hike card |
| S-12 | new (proposed) | share preview & privacy toggle |
Untouched (verified): S-01–S-04, S-07 paywall, S-08 checkout journey.

## Journeys
J-03 gains one step (S-05 → S-12). J-01, J-02 unchanged.

## Decisions
No conflicts / Requires new decision: <e.g. image CDN choice> / Violates NNNN because …

## Regression watchlist
- Hike-history list rendering (shares S-05 components)
- …

## Size: M — one new screen, two modified, one new backend endpoint.

## Recommendation
Proceed / proceed-with-changes / defer, with one sentence of reasoning.
```

## Director review, then amendment

Present the map succinctly — lead with the summary line a director actually needs ("one new screen, two modified, checkout untouched, size M"). Then stop and wait for a decision on the map.

On approval:

1. Mark the map `approved`.
2. Amend the blueprint per director-02-app-blueprint's conventions (new version, changelog entry referencing F-NNN, proposed screens become active).
3. Flag the affected baseline screens for regeneration (director-03-design-baseline reads the version bump from the manifest comparison).
4. Offer to generate the briefs (director-04-delegation-brief) — but do not generate them unbidden; the director may want to batch features.
5. Update `product/STATE.md` notes column to record the in-flight feature.

On rejection, mark the map `rejected` and leave everything else untouched — rejected maps are kept, not deleted; they document roads not taken.
