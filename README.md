# expense-manager-design-system

The single source of truth for how the **Expense Manager** looks: the Aura mechanism inherited from
[`@jcgueriaud1/vaadin-aura-design-system`](https://github.com/jcgueriaud1/vaadin-aura-design-system) v1.0.0,
with app policy (brand, components) and app views layered on top — consumed by Claude Design for
generation and by DramaFinder for verification.

> **This repo is never published.** `private: true`, no publish workflow, no npm package. It is a leaf
> consumer of the base package, imported directly into Claude Design.

## Division of labor

| Concern | Lives in |
|---|---|
| Tokens mechanism, canonical component examples, global rules, snapshot generator | base package (published) |
| Brand overrides, app components, app views, branded computed snapshots | **this repo** |
| Screen design work | Claude Design project "Expense Manager" (imports this repo) |
| Implementation | expense-manager app repo (Claude Code handoff) |
| Verification | DramaFinder DesignSpecVerifier ← branded computed snapshots from this repo |

Views are composition examples / page templates to the importer, app screens to humans. The designer
works in the Claude Design project; durable outcomes (reusable components, finalized views) land here
via `/design-sync` or PR — **the repo is the durable record, the project is the workspace.**

## Repo layout

```
expense-manager-ds/
  .npmrc                    @jcgueriaud1 scope → GitHub Packages (token via env, never committed)
  package.json              private: true; deps: base ^1.0.0; devDeps: style-dictionary,
                            @vaadin/aura + playwright pinned EXACTLY to the base's pins
  tokens/overrides.json     the only hand-edited token file; semantic layer only
  build/resolve.mjs         merge base+overrides → tokens.resolved.json (preserves $extensions)
                            + theme/tokens.css (ONLY diverged inputs) + contract check (exit 1)
  build/assemble-import.mjs → design-system/: resolved tokens, css, computed/, base components
                            + DESIGN.md, app components/, optionally views/
  components/               app-specific reusable components (ExpenseCard, StatusBadge, …)
  views/                    app screens: specs in, synced/designed views out
  design-system/            GENERATED, committed — the self-contained Claude Design import folder
  DESIGN.md                 app policy: "extends base vX; deltas below" + expense-domain rules
  .github/workflows/
    validate.yml            resolve + contract + snapshot hash check (browserless, every push)
    snapshot.yml            branded snapshot regeneration (browser; on token/dep changes only)
```

## Roadmap

Tracked as issues, grouped by [milestone](../../milestones).

### [Phase A — Scaffold](../../milestone/1)
- [ ] #1 — Repo scaffold + base package consumption
- [ ] #2 — Empty token overlay (`tokens/overrides.json` = `{}`)

**Done when:** `npm ci` is green from a clean clone with only a `read:packages` token.

### [Phase B — Pipeline](../../milestone/2)
Mechanism before policy — built and validated with overrides still empty.
- [ ] #3 — `build/resolve.mjs` with the semantic-layer contract
- [ ] #4 — Ten-minute generator test against the empty overlay
- [ ] #5 — CI split: `validate.yml` (browserless) + `snapshot.yml` (Chromium)

**Done when:** positive and negative CI runs behave as specified; `design-system/` is self-contained on `main`.

### [Phase C — Branding](../../milestone/3)
- [ ] #6 — Expense Manager brand values
- [ ] #7 — Branded computed snapshots + UX review

**Done when:** branded light + dark snapshots committed, hash-checked, reviewed.

### [Phase D — Claude Design](../../milestone/4)
- [ ] #8 — Claude Design import (go/no-go gate)
- [ ] #9 — `/design-sync` loop and where views land
- [ ] #10 — Views-in-import experiment
- [ ] #11 — Designer workflow trial (approval screen)

**Done when:** findings documented (publishable AX material either way); go/no-go decided on #8.

### [Phase E — Verification + handoff](../../milestone/5)
- [ ] #12 — DesignSpecVerifier consumes branded computed snapshots
- [ ] #13 — Full spec-unit: design → implement → verify
- [ ] #14 — Open question #9: second app — extract template or promote to base?

**Done when:** one screen passes end-to-end with verification verdicts logged, both colour schemes.

## Critical path & sequencing

**#1 → #3 → #4 → #5 → #8.**

- Branding (#6, #7) can trail #4 and run in parallel with #5.
- #12 needs only #4's snapshots — unbranded is fine to start, so DramaFinder wiring does **not** wait
  on Claude Design. If #8 disappoints, the verification half still lands.
- #10 and #11 need Phase C done.

## Rules that are easy to get wrong

- **Semantic layer only.** An override whose base token lacks
  `$extensions["com.vaadin.aura"].layer === "semantic"` fails the build (exit 1). Primitives are off limits.
- **`theme/tokens.css` emits only diverged inputs.** Base tokens ≡ Aura defaults. Emitting all 22 inputs
  walks into the `:where(:root)` density trap. Empty overrides → empty CSS.
- **`tokens.resolved.json` must preserve `$extensions`** — the base's snapshot generator reads `cssVar` from it.
- **Text and border colours are not directly settable.** Tune them via `aura.background.*` + `aura.contrast-level`.
- **Verify against `computed/`, never `tokens.resolved.json`.** Inputs are not comparable values.
  sRGB, tolerance 2 from the header, alpha-composited over the backing surface; `px` strings → rect assertions.

## Open questions

| # | Question | Answered by |
|---|---|---|
| 1 | Claude Design preview: real web components or approximation? | #8 |
| 2 | Does the importer read the `design-system/` folder as intended? | #8 |
| 5 | What `/design-sync` pushes back, and where views land | #9 |
| 7 | Does the snapshot generator accept the overlay resolved file as-is? | #4 |
| 8 | Views in the import: better composition or diluted component signal? | #10 |
| 9 | Second app: extract overlay mechanics to a template, or promote shared components to the base? | #14 (Phase E retro) |

## Explicitly out of scope

- Publishing this repo as a package — **never**.
- A separate overlay repo (the v2 plan) — merged into this single repo; extraction is mechanical
  if/when a second app consumes the base (question #9).
- Multi-density snapshot matrices — default density only, per the base package's decision.
