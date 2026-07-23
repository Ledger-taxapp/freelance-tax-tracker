# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working with the user

The user is new to a lot of the concepts involved in this project — git, hosting, browser APIs, general dev terminology. When using technical terms or jargon, explain what they mean in plain language as you go rather than assuming familiarity. Over-explain rather than assume.

## What this is

A freelance/self-employed federal tax estimator: log income, expenses, mileage, and estimated payments, and it projects SE tax, QBI deduction, Additional Medicare Tax, federal income tax, and a safe-harbor quarterly payment target. Each build is one **self-contained HTML file** — inline `<style>` and inline `<script>`, no build step, no bundler, no external JS dependencies (only Google Fonts are loaded remotely). It's grown past "single-page" — Ledger, Dashboard, and Terms are already separate views within the file (see Architecture below), and more views/pages may be added later.

This is a git repo, hosted on GitHub and deployed as a real website via GitHub Pages (see Hosting & deployment below) — it's also installable to a phone's home screen as a PWA (Progressive Web App: a normal website that can behave like an installed app, with its own icon and offline support).

There is no test suite, linter, or package.json. "Running" the app means either opening `index.html` directly in a browser (double-click, or `open index.html`), or visiting the live deployed URL — both work the same way except that service-worker/offline behavior and "Add to Home Screen" can only be fully exercised on the live URL (browsers restrict service workers to real HTTPS origins, not local files). Validate changes by loading the app and exercising the flow by hand — add an entry, switch tax years, switch filing status, check Dashboard/Terms views.

## File/versioning convention

- `index.html` is the canonical, live app. `demo.html` is the same app pre-loaded with sample data (a fictional landscaping business, ~165 entries) for demoing; it should stay in sync feature-for-feature with `index.html`.
- **Versioning is git-based now, not file-based.** The old convention (copying the whole file to a new `freelancer-tax-tracker-vX.Y.html` for every change) has been retired — every change is now a git commit with a message describing it, directly on `index.html`/`demo.html`. Don't create new versioned copies of these files.
- `archive/` holds the old `freelancer-tax-tracker-v1.0.html` through `v1.15.html`/`v1.15-demo.html` files from before the switch to git — kept for history, not maintained going forward.
- [tax-tracker-changelog_1.md](tax-tracker-changelog_1.md) is a **frozen historical record** of the pre-git era (through v1.15) — it documents the "deferred / discussed but not yet built" list that's still a good source of what's next, but new changes should no longer be appended here; they belong in git commit messages instead.

## Hosting & deployment

- Live app: `https://ledger-taxapp.github.io/freelance-tax-tracker/` — Demo: `https://ledger-taxapp.github.io/freelance-tax-tracker/demo.html`
- Hosted via **GitHub Pages**, serving directly from the `main` branch root of the `Ledger-taxapp/freelance-tax-tracker` GitHub repo (a GitHub **organization**, chosen specifically so the URL/repo isn't tied to the user's personal GitHub account/username).
- Deployment is push-based: committing and pushing to `main` is what publishes changes — GitHub Pages rebuilds automatically (takes a minute or two), there's no separate deploy step or CI config.
- The user runs `git push` themselves from their own Terminal (not something to run on their behalf without confirming first — see repo-level safety norms) since it requires their GitHub credentials.

## Architecture (per file)

Everything lives in one IIFE at the bottom of `index.html` (and identically in `demo.html`, aside from the seeded sample data). Key pieces:

- **`TAX_YEAR_PARAMS`** — per-year federal tax constants (standard deduction, brackets, SS wage base, Additional Medicare threshold, QBI phase-out range) keyed by year. Adding a new tax year means adding an entry here; `paramsForYear()` falls back to `DEFAULT_TAX_YEAR` for unlisted years and the UI surfaces a "using estimates" note when that happens.
- **`MILEAGE_RATES`** — a date-ranged table of the IRS standard mileage rate; `mileageRateFor(dateStr)` looks up the rate effective on a given date (rates can change mid-year).
- **`state`** — the single mutable app state object (filing status, entries array, W-2/withholding/prior-year fields, selected year). Persisted via the browser's own `localStorage` (synchronous, standard Web Storage API — this replaced an earlier sandbox-only `window.storage` API that wouldn't have worked once deployed to a real site), keyed by `STORAGE_KEY`. Data stays entirely on the visitor's own device — nothing is ever sent to a server. `save()` fires on every mutation; `load()` runs once at startup and calls `renderAll()`.
- **PWA files** — `manifest.json` / `demo-manifest.json` (name, icons, colors, standalone display — one per entry point so each can be installed to a home screen independently) and `sw.js` (a shared service worker for offline support). It's network-first for the HTML pages themselves (so an online visitor always gets the latest published version, not a stale cached one — the cache is only a fallback when offline), and stale-while-revalidate for static assets like icons. Both `index.html` and `demo.html` register the same `sw.js` and link their respective manifest in `<head>`.
- **`calcTotals(year)`** — the tax engine. Pure function of `state` + a year: computes net business income, SE tax (with the per-person SS wage-base cap — only the filer's own W-2 wages reduce it, never a spouse's), the QBI deduction (with income-based phase-out taper), Additional Medicare Tax (on combined household income), federal bracket tax, the safe-harbor target, and the suggested next payment. This is the one function to understand before touching any tax math — nearly every UI number comes from its return value.
- **`entries`** — each is `{id, type: income|expense|payment, date, amount, note, ...}` with optional flags: `isMileage` (expense sub-type; `amount` is pre-computed `miles × rate`), `isMeal` (50%-deductible expense), `category` (one of 11 fixed expense categories). Mileage is *not* a separate `type` — it's stored as `type: 'expense'` with `isMileage: true`.
- **Rendering is imperative, not reactive**: there's no framework/virtual DOM. Every state mutation is followed by an explicit `save()` + a render call (`renderAll()` for broad changes, or a narrower `renderStats()`/`renderLedger()`/etc. when only part of the UI is affected). `renderAll()` is the top-level cascade — read it to see the full render order.
- **Three views, one DOM**: Ledger / Dashboard / Terms are `<div id="view-*">` blocks toggled via `display` in `switchView()`, driven by the bottom nav. There's no routing/history — `currentView` is just in-memory state.
- **Dashboard charts are hand-built inline SVG** (bar chart, donut chart, progress bar), not a charting library. `wireChartShape()` attaches shared hover/tap tooltip behavior to any element with class `chart-shape` + `data-label`/`data-value` attributes — follow that pattern for new chart elements.
- **Glossary/tooltip system**: `GLOSSARY_SUMMARIES` holds short tooltip blurbs; the Terms view has the full `<details class="term">` entries. `.info-btn[data-term="..."]` buttons show a hover/tap tooltip (`wireInfoButton`) whose "View more" jumps to and highlights the matching term (`openGlossaryTerm`) in the Terms view.
- **Mobile-first touch handling**: `bindTap()` is a shared helper distinguishing a real tap from a scroll/drag on touch devices (used for ledger rows, delete confirm, chart shapes, info buttons) — reuse it rather than adding raw `click` listeners for anything tappable, to avoid the "double-tap needed" bugs fixed in past versions.

## Domain rules worth knowing before editing tax logic

- Federal only — no state tax is modeled anywhere. This is a deliberate v1 scope decision, not a gap to casually fill in.
- v1 only supports Single, Married Filing Jointly, and Head of Household. Married Filing Separately and Qualifying Surviving Spouse are deferred to v2 — don't add them piecemeal (e.g. just adding the option to `#status`) without also working through their bracket/std-deduction/QBI-phase-out/safe-harbor implications in `TAX_YEAR_PARAMS` and `calcTotals()`.
- SE tax = 92.35% of net business income; 12.4% SS (capped at the year's wage base, reduced only by the filer's *own* W-2 wages) + 2.9% Medicare (uncapped).
- QBI deduction: 20% of net business income (after the deductible half of SE tax), capped at 20% of taxable income before the deduction, with a straight-line phase-out taper over the published income range — assumes no W-2 payroll/business property of their own (typical solo freelancer case).
- Safe harbor target = lesser of 90% of projected current-year tax or 100% of prior-year tax (110% if prior-year AGI > $150k).
- "Already covered" = W-2 withholding (treated as paid evenly across the year) + logged estimated payments.
- Meals are 50% deductible; mileage deduction = miles × the rate in effect on the trip date.
