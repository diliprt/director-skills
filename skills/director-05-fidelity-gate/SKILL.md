---
name: director-05-fidelity-gate
description: Verify a built app against its approved blueprint and design baseline — walk every journey, exercise every screen and state, capture screenshots, compare against the ground-truth mockups, and report deviations in plain English with severity. Use when the user says "run the fidelity gate", "does the build match the design", "check the build against the mockups", after build briefs complete, or before a release gate. Requires product/blueprint/ and product/baseline/ to exist.
---

# Fidelity gate

This skill is part of a director-led pipeline: the user decides and approves; agents execute. The fidelity gate is what makes design-ahead enforceable rather than aspirational — it walks the built app through every approved journey and compares what renders against the baseline mockups. Without this gate, "matches the design" is an opinion; with it, it's a report.

## Preconditions

1. Read `product/STATE.md`, `product/blueprint/` (screens + journeys), and `product/baseline/manifest.json`. Only `approved` baseline screens are checkable ground truth; list unapproved ones as `not verifiable — no approved baseline` rather than skipping silently.
2. Locate the built app and how to run it (project README, launch config, or the briefs' verification sections).

## Driving the app

Use the best available harness, in this order of preference:

- **Web / web-based mobile shell**: Playwright or the preview tooling — navigate, click, screenshot.
- **iOS**: `xcrun simctl` (boot simulator, install, launch, `simctl io screenshot`) plus UI automation if the project has it.
- **Android**: emulator + `adb` (launch, `adb exec-out screencap`) plus Espresso/Maestro if present.
- **Nothing drivable available**: do not fake it. Produce the report with every unverified item marked `needs manual walkthrough`, and give the user a numbered manual checklist (journey by journey) so a human walkthrough still produces a filled-in report.

The gate's credibility rests on one rule: **never mark a screen or journey `pass` that was not actually exercised.** A partial honest report beats a complete optimistic one.

## What to check

For each journey in `journeys.md`, walk it step by step:

- Every navigation edge works and lands on the right screen (screen identity, not pixel identity).
- Each screen reached: capture a screenshot, then compare against its baseline mockup **structurally and semantically** — are the elements present, is the hierarchy right, do the tokens (colors, type scale, spacing rhythm) match? This is not a pixel diff; fonts and rendering engines differ. It is "would the director recognize this as the screen they approved?"
- Exercise the non-default states listed in the blueprint (empty, loading, error, offline) wherever they can be induced — airplane-mode the emulator, use empty accounts, force errors where test hooks exist. States are where builds most often silently diverge from designs.
- Note differences that look intentional (an improvement an agent made unprompted) separately from defects — the director may want to keep them, but they must be surfaced, because unreviewed "improvements" are how the contract erodes.

## The report

`product/gates/fidelity/YYYY-MM-DD/report.md`, with screenshots saved alongside in `shots/` and referenced by relative path:

```markdown
# Fidelity gate — YYYY-MM-DD

## Executive summary
≤10 lines, plain English: verdict per journey, count of blockers/deviations/cosmetics,
and the single most important finding. Written for a director, not a developer.

## Verdicts
| Journey | Result |
|---|---|
| J-01 First run | pass |
| J-02 Core loop | 2 deviations |

## Findings
### FID-01 — S-04 active-hike screen missing offline state  [blocker]
Blueprint v2 lists offline for S-04; build shows infinite spinner with radios off.
Baseline: ../../baseline/screens/S-04-active-hike--offline.html · Shot: shots/S-04-offline.png

### FID-02 — onboarding added a permissions screen not in blueprint  [deviation]
…

## Intentional-looking differences (for director review)
…

## Not verifiable
| Item | Reason |
|---|---|
```

Severities: `blocker` (journey broken or approved content missing), `deviation` (present but doesn't match the contract), `cosmetic` (recognizably right, minor drift).

## On completion

1. Update `product/STATE.md` (Fidelity gate row: pass / N findings, date).
2. Give the user the executive summary directly in conversation — never make them open the file to learn the verdict.
3. Offer to generate fix briefs (director-04-delegation-brief fix mode) for blockers and deviations. Cosmetics: ask whether to batch-fix or accept; accepted cosmetics get noted in the report so the next run doesn't re-flag them.
