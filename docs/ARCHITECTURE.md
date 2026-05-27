# elitez-intel-kit — Architecture

## Purpose

Canonical shared front-end JavaScript and vendored UMD bundles for Elitez
competitor-intelligence sites. Extracted verbatim from
`competitor-intel-template` on 2026-05-19 to de-duplicate the ~13 forks that
had drifted to varying degrees. Consumed as a public git submodule pinned at
a tag; CSS is intentionally excluded so each brand keeps its own stylesheet.

The kit is the floor (data loaders, DOM helpers, viz, report pipeline,
auth-gate); the host repo is the ceiling (CSS tokens, data JSON, page HTML,
optional per-brand `radar.js` override).

## Versioning

- Current tag: `v1.0.0` (commit `178bbab`, 2026-05-19)
- HEAD: `b22ec3d` — security pins for Supabase / DOMPurify / marked + SRI
  on CDN tags (post-tag; not yet promoted to `v1.1.0`)
- Test suite green at tag time: **25/25 pass** (re-verified 2026-05-27)
- Semver. Consumers pin a commit via the submodule pointer; bump
  deliberately with `git submodule update --remote` followed by a regression
  pass across the consumer.
- `package.json` declares `"private": true` — npm publishing is deferred.

## Tech stack

- **Runtime:** browser. ES2020 modules served directly via `<script type="module">`.
- **No build step.** No bundler, no transpiler, no TypeScript. Files are
  served as authored.
- **Node:** used only for the test runner (`node --test`) on `.mjs` test files.

### Vendored libraries (UMD, pinned)

| File | Version | Purpose |
|------|---------|---------|
| `vendor/chart.umd.js` | Chart.js 4.4.1 | Radar charts (and any future Chart.js usage) |
| `vendor/html2canvas.min.js` | html2canvas 1.4.1 | Rasterises `.pdf-page` divs for PDF export |
| `vendor/jspdf.umd.min.js` | jsPDF 2.5.1 | Assembles PDF from rasterised pages |

Supabase JS (`@supabase/supabase-js@2.106.1`) is **not vendored** — it is
loaded from jsDelivr with SRI via the `<script>` tag documented in
`js/auth-gate.js`. Same for DOMPurify and marked (post-tag pin).

## Public surface (modules consumers import)

All paths are relative to the kit root.

| Module | Exports / Responsibility |
|--------|--------------------------|
| `js/app.js` | `loadAppData(dataPath = '../data')`, `mountSampleBanner`, `mountUnstyledBanner`. Loads the 4 canonical JSON files (`competitors`, `market-intelligence`, `pricing-strategy`, `whitespace-framework`) and parks them on `window.AppData`. |
| `js/dom.js` | `h(tag, props, ...children)`, `clear(node)`, `mount(parent, ...children)`. XSS-safe DOM construction — replaces all `innerHTML` use in consumers. |
| `js/format.js` | `fmtSGD`, `fmtSGDFull`, `fmtRange`, `fmtScore`, `fmtPct`. Currency-aware display formatters; reads `currency_label` off `window.AppData.market` or `brand_tokens`. |
| `js/auth-gate.js` | Supabase email-OTP overlay gate. IIFE; exposes `window._sbReady` (Promise) and `window._sb`. Hard-codes the shared Elitez Supabase project `suehogmzjspagcsrqvsw`; sign-ups are whitelisted to `elitez.asia` / `dhc.com.sg` by a `Before User Created` Postgres hook. |
| `js/viz/heatmap.js` | `cellCount`, `cellBand`, `buildCellDetail`, `renderHeatmap`, `renderCellDetail`. HTML+CSS-grid whitespace heatmap. |
| `js/viz/radar.js` | `buildRadarData({ dimensions, scores })`. Chart.js radar dataset builder; `"us"` gets thick + filled styling. **Template default; per-brand overrides are expected** (see warnings). |
| `js/viz/search.js` | `matchesCompetitor`, `debounce`, `wireSearch`. Pure filter predicate + debounced input wiring. |
| `js/report/page-templates.js` | `buildSections(data)` — section registry mapping each report chapter to a renderer + page-count function. |
| `js/report/pdf-generator.js` | `generatePDF(...)`, `PreflightError`. Rasterises each `.pdf-page` into an A4 PDF (scale 1.0, JPEG 0.62, compress). Runs `preflight-dom` Phase 4 before the rasterise loop. |
| `js/report/preflight-dom.js` | `validateDOM`, `formatViolations`. Phase 4 structural pre-flight gate — catches missing sections, TOC drift, blank-page leakage, missing footers, white-cover failure mode. All checks run; nothing short-circuits. |
| `js/report/toc.js` | `computePageIndex(sections, data)`. Pure page-numbering. |

## Consumers (9 repos)

Confirmed by 2026-05-19 portfolio scan + portfolio health report
(`/Users/derrickteo/codings/docs/ARCHITECTURE-HEALTH-REPORT.md`). Consumed
as a git submodule at `template/assets/kit` (or `assets/kit` for
flat-layout repos).

### Tier-A pilots (Phase 0 — 2026-05-19)

- `competitor-intel-template` — the original; kit was extracted from this repo
- `market-tracker-research`
- `flashcart-research`

### Tier-B (Phase 1+2 — through 2026-05-20)

- `competitor-intel-self` (now merged into `competitor-intel-template` as the worked example)
- `casket`
- `elitez-security`
- `elitezaviation`
- `elix-eor`
- `mri`
- `XinceAI`

### Phase 3 — deliberately NOT migrated

- `elitez-events`
- `Lumana`
- `Elitez-marketing-services`

These three diverged too heavily (own data shapes, own viz, own report flow)
to absorb the kit without a rewrite. Honour the divergence; do not retrofit.

## Test suite

- Location: co-located `_tests/` directories
  - `js/_tests/dom.test.mjs`
  - `js/viz/_tests/heatmap.test.mjs`
  - `js/viz/_tests/radar.test.mjs`
  - `js/viz/_tests/search.test.mjs`
  - `js/report/_tests/toc.test.mjs`
- Runner: Node's built-in test runner (`node --test`)
- Command: `npm test`
- Current pass rate: **25/25 green** (verified 2026-05-27)
- Coverage is pure-functions-only by design. Renderers (`renderHeatmap`,
  `renderCellDetail`, PDF rasterise) are not exercised because they need a
  DOM; the pure builders they delegate to (`buildCellDetail`, `buildRadarData`,
  `matchesCompetitor`, `computePageIndex`) are fully covered.
- `market-funnel.test.mjs` was deliberately dropped at v1.0.0 (orphan — its
  module is slated for a later release). See CHANGELOG.

## Local dev

```bash
git clone https://github.com/derrick-pixel/elitez-intel-kit.git
cd elitez-intel-kit
npm test                  # 25/25 green expected
```

### Cutting a new version

1. Land changes, ensure `npm test` is green.
2. Bump `package.json` version.
3. Update `CHANGELOG.md`.
4. Tag: `git tag vX.Y.Z && git push --tags`.
5. In each consumer: `cd template/assets/kit && git fetch && git checkout vX.Y.Z && cd - && git add . && git commit`.
6. Smoke-test every consumer page that touches the changed module — there
   is no consumer-side CI for this.

## Deploy / distribution model

- **Distribution:** public git submodule.
  `git submodule add https://github.com/derrick-pixel/elitez-intel-kit.git template/assets/kit`
- **Pinning:** consumers pin a commit (typically a tag). The submodule
  pointer is committed to the consumer repo.
- **CSS excluded** — per-brand and unshareable. Each consumer keeps its own
  brand stylesheet untouched by kit upgrades.
- **Per-brand `radar.js` override** — each consumer may keep its own
  `js/viz/radar.js` outside the submodule with its own palette / styling.
  The kit's `radar.js` is the template default, not a contract.

## Edge cases & warnings for future developers

This section is load-bearing. Read before touching the kit or any consumer.

### 1. GitHub Pages DOES initialise public HTTPS git submodules

Verified by canary on 2026-05-19. The kit being public over HTTPS is
non-negotiable for this reason. If the repo is ever flipped to private or
the URL is changed to SSH, every consumer's GitHub Pages build will start
shipping with an empty `template/assets/kit/` folder and 404 every kit
import.

### 2. `dom.js` was REMOVED from kit-consuming repos

When a repo adopts the kit, its local `dom.js` is deleted. Any non-kit
consumer JavaScript that still imports `./dom.js` must repoint to
`../../kit/js/dom.js` (or the equivalent relative path for that repo's
layout). Grep for `from ['"].*dom\.js['"]` on every adoption.

### 3. Flat-layout consumers must keep the `../` prefix on `auth-gate.js`

Some consumers have a flat repo layout (no `template/` directory).
When repointing `auth-gate.js` from `assets/js/auth-gate.js` to the kit
location, the file moves UP a directory level. Do not drop the `../`
prefix — it is required because the kit sits one level deeper than the
original co-located path. Easy bug to ship; check the rendered page in
DevTools after migration.

### 4. `loadAppData()` defaults to `../data` — spawn-offs MUST pass `./data`

`app.js`:
```js
export async function loadAppData(dataPath = '../data') { ... }
```

The default assumes the original `template/pages/...html` layout where data
lives one directory up. Spawn-off / flat-layout repos that put their data at
`./data` MUST pass the path explicitly:

```js
loadAppData('./data')
```

This broke **8 of 18 consumers** in the May-20 sweep. Always grep for
`loadAppData(` on adoption and audit every call site.

### 5. `radar.js` is a legitimate per-brand override

The kit ships `js/viz/radar.js` with a template palette (blue + red + green
+ etc., with `"us"` in warm amber). Several consumers keep their own
`radar.js` outside the submodule with brand-specific colours. This is by
design — do not "fix" it by deleting the consumer's local copy and
re-pointing imports to the kit version. Verify the brand palette before
touching radar.

### 6. `app.js` carries the `dataPath` parameter — there is no fork

Before assuming a consumer has forked `app.js`, check whether the
divergence is just the `dataPath` argument flowing through. Most "forks"
turn out to be different call sites, not different code.

### 7. `auth-gate.js` Supabase project is hard-coded

`SUPABASE_URL` and `SUPABASE_ANON_KEY` are baked into the IIFE
(project `suehogmzjspagcsrqvsw` — the shared Elitez auth + ESOP project).
Rotating those credentials is a kit-level change that fans out to every
consumer on the next submodule bump. Plan accordingly.

### 8. Static-page auth-gate is UI access control, not data confidentiality

The gate hides the rendered view but does NOT hide page source. Any data
baked into the HTML (e.g. competitor names in `<table>` cells) remains
visible via View Source. Moving data into Supabase with RLS is a separate
planned effort. Tell stakeholders this when they ask "is the page secure?"

### 9. Pre-flight DOM gate is all-or-nothing

`preflight-dom.js` runs ALL checks every time and does not short-circuit.
This is deliberate so a single re-run surfaces every violation. Don't
"optimise" it by bailing on the first failure — you'll multiply round-trips.

## Known tech debt

- **Three unmigrated forks (Phase 3)** — `elitez-events`, `Lumana`,
  `Elitez-marketing-services`. Deliberate; documented above.
- **npm publishing deferred.** `package.json` is `"private": true`.
  Distribution model is git submodule only. If a consumer outside the
  Elitez portfolio ever needs the kit, the publish question reopens.
- **Post-tag commit (`b22ec3d`) not promoted to v1.1.0.** Security pins
  + SRI sit on the tip of `main` but no consumer is pulling them yet.
  Promote when the next consumer-side regression slot is available.
- **`market-funnel.test.mjs` and its module are pending.** Dropped at
  v1.0.0; flagged as "later release" in CHANGELOG. No tracking issue yet.
- **No consumer-side CI.** Upgrading the submodule pin in a consumer
  relies on manual smoke-testing of every page that touches the changed
  module. A "verify intel pages by loading them" memory exists precisely
  because silent breakage has happened before.
- **Renderers untested.** Pure builders have 25 tests; `renderHeatmap`,
  `renderCellDetail`, `renderPages`, and the PDF rasterise loop have zero.
  A jsdom-based render harness is the obvious next step.
- **No public docs site / no API reference** beyond this file + README.
  Consumers learn the kit by reading the source.

## Related memories / docs

- Portfolio health report:
  `/Users/derrickteo/codings/docs/ARCHITECTURE-HEALTH-REPORT.md`
- Extraction plan:
  `/Users/derrickteo/codings/docs/superpowers/plans/2026-05-19-intel-kit-extraction.md`
- Memory: `project_intel_kit.md` — submodule strategy, Phase 0/1 done,
  Phase 2/3 status
- Memory: `feedback_load_app_data_default.md` — the `'./data'` gotcha
- Memory: `feedback_intel_page_verification.md` — visually confirm data
  populated after every kit upgrade; framework alone is not "done"
- Memory: `project_maintenance_2026_05_20.md` — May-20 sweep that
  surfaced the `loadAppData` default-path breakage
- Consumer-side reference example: `competitor-intel-template` (which
  absorbed `competitor-intel-self` as the worked example on 2026-05-19)
