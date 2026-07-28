---
name: tax-rule-sourcing
description: Reference for the exact government sources behind every federal tax figure this app calculates — SE tax rates, the QBI deduction phase-out, safe harbor, the standard mileage rate, the meal deduction, Additional Medicare Tax, the standard deduction, and tax brackets. Use this whenever adding a new tax year to TAX_YEAR_PARAMS or MILEAGE_RATES, touching calcTotals() or any other tax math, or verifying/updating any dollar figure or percentage in index.html or demo.html — even a change that looks like "just one number." Never type a new tax figure from memory, a guess, or a single blog post; look it up against the real source listed here first.
---

# Tax Rule Sourcing

## Why this exists

Every number this app produces — a suggested payment, a "you owe" figure — is something a real person may act on. Tax figures aren't arbitrary; each one is a specific value published by a specific government agency, on a specific schedule. A wrong figure doesn't crash anything, it just quietly produces a wrong estimate. Treat every tax constant in this codebase as a claim that needs a citation, not a number to pattern-match from last year.

## The two categories

Figures in this app fall into two very different buckets. Knowing which one you're touching tells you how urgently it needs re-checking.

### Re-verify every year — these are inflation-indexed and change annually

| Figure | Where it lives | Source to check |
|---|---|---|
| Standard deduction | `TAX_YEAR_PARAMS[year].stdDeduction` | IRS.gov newsroom: search "IRS releases tax inflation adjustments for tax year `YYYY`" — this points to that year's Revenue Procedure (e.g. Rev. Proc. 2025-32 for tax year 2026). |
| Tax bracket thresholds | `TAX_YEAR_PARAMS[year].brackets` | Same Revenue Procedure as the standard deduction, same search. |
| QBI phase-out threshold + range | `TAX_YEAR_PARAMS[year].qbiPhaseOut` | Same Revenue Procedure, § 4.26 specifically ("Qualified Business Income"). 2026 values confirmed directly against the Rev. Proc. 2025-32 PDF (irs.gov/pub/irs-drop/rp-25-32.pdf): phase-in starts at **$201,750** for "All Other Returns" (single/HOH — this app's Single and HOH share this one value) / **$403,500** for Married Filing Jointly, phases out fully over the next $75,000 / $150,000. Note the IRS table also lists a *separate* $201,775 figure, but that one is for Married Filing Separately, a status this app doesn't support — don't reuse it for single/HOH by mistake (this exact mix-up was previously in this file and in the app's code). Also don't assume the QBI threshold equals the 24%→32% bracket boundary for the same filing status; for MFJ in 2026 they differ ($403,500 vs. the bracket's $403,550) even though they happened to align for single ($201,775) and (via a different mechanism) for HOH's own bracket ($201,750) — treat the two tables as independent every year. |
| Social Security wage base | `TAX_YEAR_PARAMS[year].ssWageBase` | Social Security Administration, not the IRS — search "Social Security wage base `YYYY` SSA announcement." Announced every October for the following year. 2026 value confirmed: $184,500. |
| Standard mileage rate | `MILEAGE_RATES` array | IRS.gov newsroom: search "IRS standard mileage rate `YYYY`." Usually announced once in December for the whole following year — but check for mid-year notices too. This isn't hypothetical: it happened for 2026 (72.5¢ Jan 1–Jun 30, bumped to 76¢ Jul 1–Dec 31 due to fuel costs), and happened once before in 2022. |

### Fixed by statute — rarely change, but still worth a 30-second sanity check

| Figure | Where it lives | Notes |
|---|---|---|
| SE tax rates: 92.35% of net business income is subject to SE tax; of that, 12.4% Social Security (capped at the wage base) + 2.9% Medicare (uncapped) | Hardcoded in `calcTotals()` as `0.9235`, `0.124`, `0.029` | Set by IRC §1401/§1402, unchanged since 1990. Nothing here is inflation-indexed except the wage base cap itself. |
| Additional Medicare Tax: 0.9% on wages + SE earnings above $200,000 (single/HOH/QSS) / $250,000 (MFJ) | `TAX_YEAR_PARAMS[year].addlMedicareThreshold` (present per-year for convenience, but the values shouldn't actually change) | Introduced by the ACA in 2013 (IRC §1401(b)(2) / §3101(b)(2)) and has stayed flat for over a decade. Verify against IRS Topic 560 or the Form 8959 instructions. Don't confuse this with the separate 3.8% Net Investment Income Tax (Form 8960) — different provision, not something this app models. |
| Meal deduction: 50% | Hardcoded as `* 0.5` wherever meals are calculated | IRC §274(n) — but this has had a real temporary override before (100% for restaurant meals in 2021–2022, under the Consolidated Appropriations Act, 2021), so "fixed" still means "check," not "assume." |
| Safe harbor structure: lesser of 90% of current-year tax or 100% of prior-year tax (110% if prior-year AGI exceeded $150,000) | Safe-harbor multiplier and the $150,000 threshold are hardcoded in `calcTotals()` | Set by IRC §6654(d)(1). The $150,000 figure is a flat statutory number, not inflation-indexed. Verify against the IRS Form 2210 instructions. |

## How to verify a figure — don't skip this

1. Search for the specific IRS Revenue Procedure or newsroom release for the target tax year (e.g. "IRS Revenue Procedure 2025-32" covers tax year 2026; look for the equivalent release once the following year's numbers are published).
2. **Prefer irs.gov and ssa.gov directly over tax-prep blogs and SEO sites, and prefer the actual Revenue Procedure PDF over an IRS newsroom summary page** (the newsroom page is a plain-English recap and omits full bracket/QBI tables). Blog sites are especially unreliable on the 2026 QBI threshold specifically — most repeat $201,775 for single/HOH, but the real Rev. Proc. 2025-32 table (§ 4.26) says $201,750 for "All Other Returns"; $201,775 is actually the Married Filing Separately figure. Confirmed 2026-07-27 by reading the raw Rev. Proc. 2025-32 PDF text directly (`pypdf` extraction from irs.gov/pub/irs-drop/rp-25-32.pdf, since WebFetch alone couldn't parse this particular PDF's embedded text).
3. If a figure genuinely can't be found on an official source yet (e.g. next year's numbers haven't been published), say so rather than filling in a plausible-looking guess. The app already has an honest way to handle this: `paramsForYear()` falls back to `DEFAULT_TAX_YEAR` and the UI shows a "using estimates" note — use that pattern rather than inventing a number.

## Where these land in the code

- **`TAX_YEAR_PARAMS`** (in both `index.html` and `demo.html`) — one entry per tax year: `stdDeduction`, `brackets`, `ssWageBase`, `addlMedicareThreshold`, `qbiPhaseOut`.
- **`MILEAGE_RATES`** — a flat date-ranged array, not year-keyed. Add a new `{from, to, rate}` entry for a rate change; don't edit past entries (mirrors the project's git-commit-based versioning philosophy — history stays intact).
- **`calcTotals(year)`** — where the fixed-by-statute percentages (`0.9235`, `0.124`, `0.029`, `0.009`, `0.20` for QBI, `0.5` for meals, and the safe-harbor `0.90`/`1.00`/`1.10` multipliers plus the `$150,000` AGI threshold) live directly in the formula, since they don't vary by year the way `TAX_YEAR_PARAMS` does.
