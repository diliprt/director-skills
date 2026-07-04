---
name: director-01-decision-record
description: Create and maintain architecture decision records (ADRs) that act as standing orders for every agent working in a repo. Use whenever the user makes or discusses a technology, vendor, architecture, or policy choice ("we'll use Flutter", "let's go with AWS", "Stripe for payments", "record this decision", "why did we choose X"), when kicking off a new app or project, or before drafting blueprints or delegation briefs in a repo that has no product/decisions/ directory yet. Also use when a new choice contradicts an earlier one — the record must be superseded, not silently changed.
---

# Decision records

This skill is part of a director-led pipeline: the user decides and approves; agents execute. All pipeline state lives in the app repo under `product/` — this skill folder holds instructions only. Decision records are the top of that pipeline: choices made once, written down, and loaded by every future agent session (Claude, Codex, or local models) as standing orders. Without them, every session re-makes the director's decisions from scratch, differently each time.

## Before doing anything

1. Read `product/STATE.md` and `product/decisions/index.md` if they exist. If they exist, you are in **amend mode**: the goal is to add, update, or supersede records — never to regenerate the set.
2. If `product/decisions/` does not exist, you are in **kickoff mode**: create the directory and run the kickoff interview below.
3. Never delete or rewrite an accepted record's Decision section. Choices change by **superseding**: write a new record, set the old one's status to `superseded by NNNN`, and link both ways. The history of what was decided and reversed is itself valuable to a director.

## Kickoff interview (new project)

Interview the user like a consultant, not a form. Cover these areas, but only ask about what is genuinely undecided — if the user already stated a choice in conversation, capture it and confirm rather than re-asking:

- Platform strategy — native (Kotlin/Swift), Flutter, or React Native, and why
- Cloud and backend shape — AWS or Azure; serverless vs. containers; managed services to prefer
- Authentication — provider and account model
- Payments — in-app purchase obligations (Apple/Google rules) vs. Stripe or other PSP; who owns subscription state
- Data — primary datastore, offline strategy, sync
- Design system — seed brand/tokens, or generate fresh
- Delivery — CI/CD expectations, store tracks, release cadence
- Non-negotiables — compliance, data residency, budget ceilings

The user is a technical director, not a hands-on developer: frame options as outcomes, costs, and risks in plain English, recommend a default for each, and keep each exchange short. Two or three decisions per question round is plenty; don't fire twenty questions at once.

## Record format

One file per decision: `product/decisions/NNNN-short-slug.md` (zero-padded, e.g. `0003-payments-storekit-play-billing.md`). Keep each record to roughly one page. Use exactly this template:

```markdown
# NNNN — Title stating the decision

- Status: accepted | superseded by NNNN | deprecated
- Date: YYYY-MM-DD
- Supersedes: NNNN (omit if none)

## Context
Why this decision was needed. 2–5 sentences, plain English.

## Decision
The choice, stated as a directive. One or two sentences.

## Consequences
What this buys and what it costs. Include the rejected alternatives in one line each.

## Constraints for agents
- Bullet list of enforceable rules an agent can follow mechanically.
- Example: "All backend endpoints deploy as AWS Lambda behind API Gateway; do not introduce EC2 or containers without a new decision record."
```

The **Constraints for agents** section is the load-bearing part — it is what turns a decision into an enforceable standing order. Write constraints so that a code reviewer could check compliance without asking anyone.

## Index

Maintain `product/decisions/index.md`: a table of ID, title, status, date — one line per record, accepted records first. Agents read the index to discover which records to load.

## Enforcement wiring

After writing or changing records, make sure the repo actually routes agents to them:

- Ensure `CLAUDE.md` and `AGENTS.md` at the repo root each contain a short section (create the files if missing, append a section if present — never clobber other content):

  ```markdown
  ## Standing decisions
  Before designing or writing code, read product/decisions/index.md and comply
  with the Constraints for agents in every accepted record. If a task requires
  violating a constraint, stop and ask — do not work around it.
  ```

This is what makes the records tool-neutral: Codex and local models read `AGENTS.md`, Claude reads `CLAUDE.md`, and both point at the same files.

## On completion

- Update `product/STATE.md` (create from the template below if missing): set the Decisions row to the count of accepted records and today's date.
- Summarize for the user in plain English: what was recorded, what superseded what, and any decision areas still open.

```markdown
# Pipeline state

| Stage | Status | Version | Updated | Notes |
|---|---|---|---|---|
| Decisions | — | — | — | |
| Blueprint | — | — | — | |
| Design baseline | — | — | — | |
| Briefs | — | — | — | |
| Fidelity gate | — | — | — | |
| Release gate | — | — | — | |
```
