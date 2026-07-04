---
name: director-03-design-baseline
description: Turn an approved app blueprint into concrete visual ground truth — design tokens plus self-contained per-screen HTML mockups that builds are later verified against. Use when the user says "mockups", "design baseline", "ground truth", "make the comps", after a blueprint is approved, or when blueprint changes (new version, approved director-07-feature-impact map) require specific screens to be regenerated. Do not use for production frontend code — this produces review artifacts, not app code.
---

# Design baseline

This skill is part of a director-led pipeline: the user decides and approves; agents execute. The baseline is the literal definition of "done" for the UI — per-screen mockups the director approves in a browser like agency comps, and that the fidelity gate later compares the built app against. Code chases these files, never the reverse.

## Preconditions

1. Read `product/STATE.md`, `product/blueprint/blueprint.md`, and `product/baseline/manifest.json` if present.
2. The blueprint must exist with `Status: approved`. If it doesn't, say so and stop — offer to run director-02-app-blueprint first. Mockups drawn before the screen map is settled get thrown away, and worse, they anchor everyone to an unapproved design.
3. Read `product/decisions/` — decisions about platform and design system constrain what you draw (e.g. Material vs. iOS-flavored components for a Flutter app).

## Amend mode

If a baseline already exists, regenerate **only** the screens whose blueprint entries changed since their recorded blueprint version (compare against `manifest.json`), plus any screens the user explicitly names. Approved screens whose blueprint entry is unchanged are frozen — do not touch them, even to "improve" them. Consistency of the contract beats aesthetic drift.

## Tokens first

`product/baseline/tokens.json` — colors, type scale, spacing, radii, and any brand assets, e.g.:

```json
{
  "color": { "primary": "#1D6F5C", "surface": "#FFFFFF", "text": "#1A1A1A", "danger": "#B3261E" },
  "type": { "family": "system-ui", "scale": { "title": 28, "body": 16, "caption": 13 } },
  "space": [4, 8, 12, 16, 24, 32],
  "radius": { "control": 8, "card": 16 }
}
```

If the user has an existing app, brand, or design system to seed from, extract tokens from it. Otherwise propose two or three visual directions (show a single sample screen in each) and let the director pick **before** mass-producing screens — one cheap decision instead of thirty expensive regenerations.

## Screen mockups

One file per screen under `product/baseline/screens/`, named by stable ID: `S-03-trail-detail.html`. Where a screen's states differ materially, separate files: `S-03-trail-detail--empty.html`, `--offline.html`. Rules that make these files useful as ground truth:

- **Self-contained**: inline CSS, no external dependencies, no JavaScript required to render. Any browser, any year, any agent can open them.
- **Device-framed**: render inside a fixed 390×844 frame (a plausible phone viewport) so proportions are honest.
- **Token-driven**: every color, size, and spacing value comes from tokens.json values. The mockup demonstrates the design system; hardcoded one-off values leak into the build.
- **Realistic content**: plausible names, numbers, and imagery placeholders — lorem ipsum hides layout problems the director should catch now.
- **Annotated sparingly**: an HTML comment block at the top of each file stating screen ID, blueprint version, states covered, and any interaction notes an implementing agent needs ("pull-to-refresh here", "this list virtualizes").

Also generate `product/baseline/index.html` — a simple gallery linking every screen file, grouped by journey, so the director can click through the whole app in one sitting.

## Manifest — what the fidelity gate reads

`product/baseline/manifest.json`, one entry per screen file:

```json
{
  "screens": [
    { "id": "S-03", "file": "screens/S-03-trail-detail.html", "states": ["default", "empty", "offline"],
      "blueprintVersion": 2, "status": "approved", "approved": "2026-07-03" }
  ]
}
```

## Director review

Tell the user to open `index.html` and click through. Collect feedback screen by screen, apply revisions, and only mark entries `approved` in the manifest when the user explicitly approves (whole set or named screens). Then update `product/STATE.md` (Design baseline row: approved N/M screens, version, date).

An unapproved baseline blocks director-04-delegation-brief and director-05-fidelity-gate downstream. Do not approve on the user's behalf.
