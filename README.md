# Director Skills

A pipeline of seven [Claude Code](https://claude.com/claude-code) skills for **directing** app development instead of hand-coding it. You make the decisions and approve the design; AI agents do the building. What comes back to you is reviewable in plain English and visual mockups — not diffs.

> **Design is the contract. Code chases the design.**

These are agent skills — plain-Markdown instruction files any capable coding agent can follow. They work in Claude Code out of the box, and because all project state is stored as ordinary files, agents in other tools (Codex, Cursor/Composer, local models) can execute the same work by reading the repo.

---

## Who this is for

**Technical directors, architects, founders, and product leads** who commission software built largely by AI agents and need to stay in control of *what* gets built without living in the code.

If you can read a screen mockup and a one-page decision memo, and you'd rather approve those than review pull requests, this is aimed at you. It suits someone who understands software but doesn't want to — or shouldn't — be the one writing every line. It is **not** aimed at hands-on developers looking for a coding copilot; there are better tools for that. This is about **governing** a build, not performing one.

## The problem it solves

Agents are good at building and bad at knowing what you meant. Left alone they produce plausible-looking screens that quietly drift from your intent — and you find out too late, buried in code you didn't want to read. This pipeline fixes the ordering: your decisions and the approved design become a written contract *first*, agents build against it, and an independent gate verifies the build matches before anything ships. Your judgment is captured once and enforced on every agent session, across every tool.

## The pipeline

```
①  decision-record ──► ②  app-blueprint ──► ③  design-baseline ──► ④  delegation-brief
                                                                           │
                                                                    (agents build,
                                                                     LOOP.md drives them)
                                                                           │
                                                                           ▼
       ⑥  release-gate  ◄──────────────────────  ⑤  fidelity-gate
              │
              ▼                                   ⑦  feature-impact
        ship (your go/no-go)                      (re-entry point for changes
                                                   to an app that already exists)
```

You personally touch only the **approval points** (bold below). Everything between them is agent work.

| # | Skill | What it produces | Your role |
|---|-------|------------------|-----------|
| 1 | **director-01-decision-record** | One-page architecture/vendor/policy decisions that bind every agent | **Answer the interview, own the choices** |
| 2 | **director-02-app-blueprint** | Screen map, navigation graph, states, and user journeys | **Review the map, approve** |
| 3 | **director-03-design-baseline** | Design tokens + per-screen HTML mockups (the visual ground truth) | **Pick a visual direction, approve the comps** |
| 4 | director-04-delegation-brief | Self-contained work packets + a self-terminating build `LOOP.md` | Skim briefs, supply credentials |
| 5 | director-05-fidelity-gate | Walks the built app against the mockups; deviation report | **Read the report, rule on deviations** |
| 6 | director-06-release-gate | Tests + security + store-compliance → go/no-go recommendation | **The ship decision** |
| 7 | director-07-feature-impact | Impact map for a new feature *before* it's built | **Approve or reshape the feature** |

## How it works — the mechanics that make it durable

- **All state lives in the repo under `product/`.** Decisions, blueprint, mockups, briefs, gate reports — every artifact is a plain `.md`, `.html`, or `.json` file. The skills hold *instructions only*; the *state* travels with the app.
- **`product/STATE.md` is the resume point.** Re-run any skill and it reads STATE.md first, then amends existing artifacts — it never regenerates from scratch. You never lose approved work.
- **Stable IDs, never renumbered.** Screens (`S-01`), briefs (`B-001`), features (`F-001`), decisions (`0001`) get an ID once. Downstream artifacts reference those IDs, so nothing silently breaks when things change.
- **Approvals are explicit and gated.** A skill marks something `approved` only when you say so, and downstream skills refuse to run against unapproved upstream work. The design can't be skipped.
- **Tool-neutral by construction.** `decision-record` writes both a `CLAUDE.md` and an `AGENTS.md` pointer into the app repo, so agents in any tool load the same standing orders and can execute the same briefs cold.
- **The build loop can't declare false victory.** `LOOP.md` (emitted by `delegation-brief`) has machine-checkable exit criteria and anti-early-victory rules. Agents prove the work is done by running commands; the fidelity gate is a separate, independent check *you* own.

## Installation (Claude Code)

Copy the skill folders into your personal skills directory:

```bash
git clone https://github.com/diliprt/director-skills.git
cp -R director-skills/skills/director-0* ~/.claude/skills/
```

They register automatically. Invoke by name (`/director-01-decision-record`) or just talk — the skills trigger on natural phrasing like *"record this decision"*, *"what screens do we need"*, *"are we ready to ship"*, or *"what would it take to add feature X"*.

Using another agent? Point it at a skill's `SKILL.md` and tell it to follow the instructions — they're written to be tool-agnostic.

## A typical run

1. **Decisions** — a short interview captures platform, cloud, payments, data, delivery choices as standing orders.
2. **Blueprint** — agents draft the screen map and journeys; you cut scope in ten minutes that would cost days later.
3. **Baseline** — you pick a visual direction, agents produce mockups for every screen; you approve them in a browser like agency comps.
4. **Briefs + loop** — the design is sliced into work packets and a build loop; you hand it to an agent and walk away.
5. **Fidelity gate** — the built app is walked screen-by-screen against your mockups; deviations come back in plain English.
6. **Release gate** — one go/no-go report covering tests, security, and store compliance.
7. **Feature-impact** — later changes enter here: an impact map you approve before anything is rebuilt.

Total director time for a small app: a few hours, all of it decisions and visual review — none of it code.

## Design principles

- **You review artifacts, not diffs.** Every human touchpoint is a document or a picture.
- **Nothing is built that isn't on the approved map.** Scope leaks are caught at design time, when they're cheap.
- **Decisions are made once and enforced everywhere.** Your judgment scales across every agent and every session.
- **Verification is independent of the builder.** An agent doesn't get to grade its own homework against the design.

## Repository layout

```
skills/
  director-01-decision-record/SKILL.md
  director-02-app-blueprint/SKILL.md
  director-03-design-baseline/SKILL.md
  director-04-delegation-brief/SKILL.md
  director-05-fidelity-gate/SKILL.md
  director-06-release-gate/SKILL.md
  director-07-feature-impact/SKILL.md
```

Each `SKILL.md` is self-contained: YAML frontmatter (name + when-to-trigger description) followed by the instructions the agent executes.

## License

MIT — see [LICENSE](LICENSE).
