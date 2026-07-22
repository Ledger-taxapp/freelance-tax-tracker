# Ledger — Freelancer Tax Tracker: Build Log

A running record of every version we've built, in order. New sessions should append new version entries at the bottom rather than editing old ones, so this stays an accurate history.

**Files tracked:**
- `freelancer-tax-tracker.html` — the working tool
- `freelancer-tax-tracker-demo.html` — same tool, pre-loaded with a sample landscaping business (Marcus Reyes) for demoing

**Note on numbering:** renumbered at your request to a flat v1.0–v1.15 sequence. Reconstructing from the session, 15 versions carried an actual content change; the 16th (v1.15) is a re-share of v1.14 with no code changes — the files hadn't rendered in chat the first time, so they were re-sent. Flagging that here rather than inventing a feature that didn't happen, in case your save count was based on that duplicate download.

---

## v1.0 — Core v1 build
- Added QBI (Qualified Business Income) deduction, 20% of net profit, capped by taxable income
- Added Meals 50%-deductible toggle on expense entries
- Added Mileage entry type using the IRS standard rate, date-aware (2026's mid-year rate change from 72.5¢ to 76¢/mi)

## v1.1 — Accuracy & trust-building round
- Fixed year-scoping bug: entries now filter correctly by tax year instead of accumulating across years
- Added W-2 wages field, factored into SE tax and combined federal tax
- Added Additional Medicare Tax (0.9% over threshold)
- Added Payment entry type + safe harbor target calculation (lesser of 90% this year / 100–110% last year)
- Added "How this is calculated" methodology section
- Added persistent disclaimer banner
- Added ability to edit existing entries
- Added CSV export
- Added delete confirmation (no more accidental deletes)
- Added input validation warnings (large amounts, out-of-year dates)

## v1.2 — Glossary launch
- Added "Tax terms, explained" section — 9 plain-language glossary entries
- Added inline "?" info buttons next to jargon throughout the app
- Clicking "?" jumps to and highlights the relevant glossary term

## v1.3 — Tooltip upgrade
- Fixed a load-order bug that left the year selector and methodology section blank on first load
- Replaced click-to-jump with a hover/tap tooltip (quick summary + "View more" link)
- Softened glossary intro copy to not assume the reader is new to self-employment

## v1.4 — Wording pass
- Changed "regular job" / "day job" wording to "W-2 job" (removes the implied hierarchy over self-employment)

## v1.5 — QA audit: high-severity fixes
- Ran a structured QA pass across ~30 scenarios (income levels, filing statuses, W-2 combos); logged 10 issues, 3 high severity
- Fixed: W-2 withholding wasn't netted against the payment target (was double-counting tax already withheld)
- Fixed: MFJ households had no correct way to enter W-2 income when the freelancer and W-2 earner were different people — split into "Your W-2 wages" / "Spouse's W-2 wages"
- Fixed: QBI deduction had no income-based phase-out — now models the real 2026 phase-out range ($201,775–$276,775 single/HOH, $403,550–$553,550 joint)

## v1.6 — QA audit: remaining fixes
- Fixed CSV export: added a "Deductible" column so meal line items reconcile with the summary total
- Extended mileage rate table with verified 2023/2024 rates; added a warning when a date falls outside known rates
- Added a tax-year-approximation caveat to CSV exports for out-of-range years
- Added context notes for viewing past/future years on the due-dates card; relabels "Suggested next payment" to "Amount still owed" for past years
- Added an overpayment indicator when logged payments exceed the target
- Split "cancel edit" into a full reset (explicit Cancel) vs. a light reset (after a normal Add), so batch entry isn't disrupted
- Fixed a stale field reference in CSV export left over from the W-2 withholding change

## v1.7 — Glossary accordion + layout fixes
- Converted each glossary term into its own collapsible entry; "View more" now opens and highlights just the relevant term instead of the whole list
- Restructured the "Your details" card to fix uneven field widths
- Fixed the Income/Expense/Mileage/Payment toggle buttons rendering unequal widths on phone (switched to CSS grid)
- Fixed tooltips detaching from their trigger button while scrolling (was `position: fixed`, now scrolls with the page)

## v1.8 — Dashboard launch
- Added bottom navigation bar (Ledger | Dashboard)
- Added Payment Progress bar (covered vs. target)
- Added Income vs. Expenses by Month chart
- Added "Where your tax bill comes from" tax breakdown chart
- Added Quick Stats row (effective tax rate, miles logged, days to next payment)

## v1.9 — Demo file
- Built and shared a demo file: Marcus Reyes, solo landscaping business, ~165 sample entries (seasonal income, mileage, meals, subcontractor payments, 2 estimated tax payments)

## v1.10 — Dashboard expansion + Terms as its own page
- Added "View all entries" / "Show fewer" toggle to the ledger (shows 3 most recent by default)
- Added "Your year so far" snapshot card to the Dashboard (earnings / expenses / payments)
- Added info "?" buttons to Dashboard tax figures, linking to the glossary
- Promoted the glossary from a Ledger section into its own "Terms" destination in the nav bar
- Added two new glossary terms: Federal income tax, Effective tax rate

## v1.11 — Interactive charts
- Fixed a CSS grid bug causing stat boxes to overflow their card on phone
- Restructured "Your details" again — filing status on its own row (fixes MFJ/HOH text getting clipped)
- Added hover/tap tooltips to the monthly bar chart, tax breakdown chart, and progress bar, each with a "focus" grow/highlight effect
- Added an encouraging, percentage-tiered message under the progress bar

## v1.12 — Mobile polish round
- Fixed inconsistent label heights causing misaligned input boxes on phone
- Fixed large dollar figures wrapping mid-number in 3-column stat rows
- Redesigned the progress bar: encouraging message moved into the hover tooltip, "left to go" shown directly on the bar, percentage consolidated into one summary sentence
- Fixed a bug requiring two taps (instead of one) to trigger tooltips on mobile

## v1.13 — Alignment fixes
- Fixed label text sitting at inconsistent vertical positions between adjacent fields (e.g., "Tax year" vs. "W-2 wages")
- Fixed the Terms page shifting horizontally relative to Ledger/Dashboard on desktop (scrollbar width difference)

## v1.14 — Entry form restructure, categories, and filtering
- Restructured the entry form: Note is now its own full-width row, Add/Cancel moved to a dedicated row at the bottom
- Added Expense Categories (11 categories, optional, shown as a ledger tag and CSV column)
- Fixed edit mode being invisible when scrolled far down the ledger — now auto-scrolls to the entry form
- Added an entry filter: toggle by type (Income/Expense/Mileage/Payment) and filter by date range
- Added a disclaimer asterisk + footnote to the progress bar's "left to go" figure
- Moved the progress bar percentage into the green fill itself when there's room; falls back to the summary sentence when the fill's too narrow

## v1.15 — Re-share
- No code changes. Files from v1.14 hadn't rendered in chat on the first attempt; re-sent as-is.

---

*Deferred / discussed but not yet built: Income categories, state income tax, Married Filing Separately / Qualifying Surviving Spouse statuses, document/receipt upload, tax-savings planner (retirement & health insurance what-ifs), multi-year comparison view, record-keeping checklist, expense-category breakdown chart.*
