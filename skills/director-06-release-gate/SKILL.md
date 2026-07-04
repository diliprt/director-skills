---
name: director-06-release-gate
description: Run the pre-release go/no-go — tests, security scans, dependency audit, store-compliance checks, backend readiness, and the latest fidelity results — aggregated into a short plain-English decision report with a recommendation. Use when the user says "release gate", "are we ready to ship", "go/no-go", "pre-release check", before any store submission, TestFlight/Play promotion, or production backend deploy. The gate recommends; the user decides.
---

# Release gate

This skill is part of a director-led pipeline: the user decides and approves; agents execute. The release gate is the director's second signature — it aggregates every mechanical check into one two-page report with a recommendation, written so a decision can be made without reading raw tool output. Its job is honesty, not reassurance.

## Preconditions

1. Read `product/STATE.md`, the latest report under `product/gates/fidelity/`, and the previous report under `product/gates/release/` (for the accepted-risk ledger).
2. If there is no fidelity report, or it predates significant code changes (check git log), flag that prominently — a release gate over a stale fidelity result is checking the wrong build. Offer to run director-05-fidelity-gate first, but proceed if the user says so, marking fidelity as `stale` in the checks table.
3. Read `product/decisions/` — several checks (payments, data residency) are defined by the decision records.

## The checks

Run everything the project's tooling supports; every check ends in exactly one of `pass`, `fail`, or `not run (reason)`. Never silently omit a check — an absent row reads as "fine", which is the one thing it isn't.

**Correctness**
- Full test suite, via the project's own commands. A failing suite is a blocker by default — do not rationalize red tests as flaky; if the user has previously accepted a specific flaky test, it lives in the risk ledger, not in your judgment.
- Build succeeds in release configuration (both platforms for mobile).

**Security**
- Dependency audit (`npm audit` / `pip-audit` / `flutter pub outdated --mode=null-safety` / platform equivalent; use what the repo has).
- Secrets scan: grep the repo and build artifacts for API keys, tokens, hardcoded credentials.
- OWASP Mobile Top 10 quick pass: insecure storage (tokens in plaintext prefs?), insecure communication (cert pinning / plain HTTP?), insufficient auth on backend endpoints, leaky logging of PII. Depth-limited — this is a gate, not an audit; recommend a full audit when findings suggest one.

**Store compliance** (mobile)
- Permission strings present and honest for every permission requested (App Store rejects vague ones).
- Privacy manifest / Play data-safety answers consistent with what the code actually collects.
- In-app purchase rules: digital goods bought in-app use StoreKit/Play Billing per the payments decision record; restore-purchases works; account deletion is reachable (both stores require it).
- Version/build numbers bumped; release notes exist.

**Backend** (if the app has one)
- Deploy target matches the decision records (right cloud, right services).
- Migrations applied cleanly to a staging copy; rollback step documented.
- Monitoring/alerting exists for the endpoints this release adds.

## The report

`product/gates/release/YYYY-MM-DD/report.md`:

```markdown
# Release gate — YYYY-MM-DD — <version>

## Recommendation: go | no-go | go-with-risks
One paragraph: the reasoning a director needs, nothing else.

## Blockers
1. …each with: what, evidence, what fixing takes.

## Risks (not blocking, need acknowledgment)
| # | Risk | First raised | Status |
|---|---|---|---|
| R-3 | … | 2026-06-20 | carried (accepted 2026-06-21) |

## Checks
| Check | Result | Evidence |
|---|---|---|

## Decision
Pending — awaiting user. (Then: go / no-go, date, and any newly accepted risk numbers.)
```

The **risk ledger** carries forward: copy previously accepted risks from the last report with their original dates and `carried` status, and add new ones. This is what prevents the same risk being silently re-accepted forever, and what stops a new session re-litigating what the director already ruled on.

## Conduct

- Report failures plainly, with the failing output quoted in the evidence appendix. Never soften "fail" into "mostly passing".
- The recommendation is yours; the decision is not. Present the report summary in conversation, then stop and wait. Record the user's decision in the Decision section verbatim.
- On decision, update `product/STATE.md` (Release gate row: decision, version, date). If the decision is go, offer the next step (store submission / deploy runbook) but do not execute irreversible publishing steps without the user asking.
