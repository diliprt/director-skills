---
name: director-04-delegation-brief
description: Slice an approved design pack (blueprint + baseline) into self-contained work packets that any coding agent — Claude, Codex, or a local model — can execute without access to this conversation, and generate fix briefs from gate reports. Use when the user says "create the briefs", "slice the work", "hand this to codex", "work packets", after a design baseline is approved, when an approved director-07-feature-impact map needs implementation, or when a fidelity or release gate report lists deviations that need fixing.
---

# Delegation briefs

This skill is part of a director-led pipeline: the user decides and approves; agents execute. A brief is the handoff artifact — a work packet complete enough that a cold agent in a different tool can pick it up, build the right thing, and prove it built the right thing. The director reviews briefs (one page each), not diffs.

## Preconditions

1. Read `product/STATE.md`, `product/briefs/index.md` if present, `product/blueprint/blueprint.md`, and `product/baseline/manifest.json`.
2. For new build work: blueprint and the relevant baseline screens must be `approved`. For feature work: an approved impact map in `product/impact/`. For fix work: a gate report under `product/gates/`. If the prerequisite is missing or unapproved, say so and stop — briefs written against an unsettled design are how scope drifts back in.
3. Read `product/decisions/index.md`; every brief inherits the accepted constraints.

## Amend mode

If briefs exist, reconcile before adding: read `index.md`, check statuses against reality where cheap to verify (does the code the brief promised exist?), and never renumber. Brief IDs `B-001`, `B-002`, … are assigned once, like screen IDs.

## Slicing

A brief should be one agent-session of work — one coherent feature slice, roughly one to three screens, or one backend service. Slice along seams where interfaces are stable (auth, one screen-flow, one API), not along layers ("all the UI" / "all the backend") — layer-sliced briefs deadlock on each other.

Produce `product/briefs/index.md`:

```markdown
| ID | Title | Status | Depends on | Assigned to |
|---|---|---|---|---|
| B-001 | Auth flow (S-01, S-02) | ready | — | |
| B-004 | Paywall (S-07) | draft | B-001 | |
```

Statuses: `draft`, `ready`, `in-progress`, `done`, `blocked`. Order the table topologically and state which briefs can run in parallel — the director uses this to fan work out across agents and worktrees.

## Brief format

One file per brief, `product/briefs/B-NNN-slug.md`:

```markdown
# B-NNN — Title

- Status: ready
- Depends on: B-NNN, … or —
- Screens: S-03, S-04 (or "backend — no screens")

## Goal
One paragraph: what exists when this is done, in outcome language.

## References
- Decisions: product/decisions/0002-….md, 0003-….md (constraints that bind this work)
- Blueprint: product/blueprint/blueprint.md vN — screens S-03, S-04; journey J-02
- Ground truth: product/baseline/screens/S-03-….html, S-04-….html

## Scope
In: …
Out: … (name the tempting adjacent work this brief must NOT do)

## Acceptance criteria
- [ ] Each criterion objectively checkable — a reviewer needs no conversation context to verify it.
- [ ] Screen S-03 renders all states listed in the blueprint (default, empty, offline).
- [ ] …

## Verification
Exact commands to run (tests, lint, build) and what passing looks like.

## Handoff notes
Branch/worktree convention, where to put new files, anything about the repo the agent can't infer.
```

The self-containment test: could Codex, opened cold in this repo with only this file, do the work? Every reference must be a repo path, not "as discussed" or "the mockup from earlier". If you find yourself writing prose context into a brief that isn't in any artifact, that's a signal the artifact (decision record, blueprint note) is missing — fix the artifact, then reference it.

## Fix-brief mode

Given a gate report (`product/gates/fidelity/...` or `.../release/...`), create one brief per confirmed deviation or blocker, referencing the report finding, the screen ID, and the baseline file that defines correct. Title them `B-NNN — Fix: <deviation>`. Severity `cosmetic` findings may be batched into a single brief.

## Loop protocol — the runnable handoff

After writing the briefs, also write `product/briefs/LOOP.md`: the self-terminating loop any build agent (Composer, Codex, Claude, a local model) runs until the work is *provably* done. This is what lets the director start an agent and walk away. It must contain:

1. **One iteration** = read `index.md` → pick the highest-priority brief whose dependencies are `done` → execute it → run that brief's Verification commands → only if they pass, tick the acceptance boxes and set status `done` → commit → next iteration.
2. **Exit criteria** — explicit, machine-checkable, all required: every non-blocked brief `done`, every acceptance checkbox in done briefs checked, the project's analyze/test commands green. State them as commands with expected output, not prose.
3. **Anti-early-victory rules**: never mark `done` without running the verification commands in that same iteration; visual criteria are checked against the named baseline mockup files; a blocker gets recorded in `index.md` and stops the loop with a status report — it does not get worked around.
4. **A paste-ready loop prompt** for external agents, and the equivalent `/loop` invocation for Claude Code.

The fidelity and release gates stay OUTSIDE the loop on purpose — the loop proves the briefs' own criteria; the gates are the director's independent check afterward. A loop that grades its own homework against the mockups is a self-check, not a gate.

## On completion

Update `product/STATE.md` (Briefs row: e.g. "9 total — 2 done, 6 ready, 1 blocked", date). Summarize for the director: how many briefs, the dependency shape, which can start immediately, which need credentials or decisions only the director can supply, and where LOOP.md is.
