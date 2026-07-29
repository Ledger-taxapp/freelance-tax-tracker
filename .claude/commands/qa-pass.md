Run a comprehensive pre-launch quality pass on the Ledger app (index.html + demo.html), covering calculation accuracy, copy, UI/UX bugs, and cross-file sync. This mirrors the full audit done on 2026-07-28 — follow the same rigor, don't skip steps to save time.

Use TaskCreate to track each phase below so progress is visible.

## 1. Verify every 2026 tax figure against primary sources

Load the `tax-rule-sourcing` skill first. For each figure in `TAX_YEAR_PARAMS` (standard deduction, all bracket thresholds for all filing statuses, QBI phase-out thresholds and ranges, Social Security wage base, Additional Medicare threshold) and `MILEAGE_RATES`:

- Go to the primary source, not a blog/aggregator summary. For IRS figures, that means the actual Revenue Procedure PDF (e.g. irs.gov/pub/irs-drop/rp-XX-XX.pdf), not just the IRS newsroom recap page (which omits full tables). If WebFetch can't extract PDF text cleanly, download it and use `pypdf` via Bash to extract and grep the real text.
- Cross-check bracket-boundary figures against QBI-threshold figures separately — they are NOT guaranteed to be identical for a given filing status/year even when they look like they should be (this bit the app before: single's QBI threshold happened to equal its bracket boundary, but MFJ's and HOH's did not).
- Confirm the SS wage base is sourced from ssa.gov, not irs.gov.
- Confirm the meal deduction is still 50% (check for any temporary legislative override) and the safe-harbor 90%/100%/110%/$150k structure hasn't changed (IRC §6654(d)(1)).
- If any figure changed since it was last verified, fix it in both files, plus every place it's echoed in tooltips/glossary copy, plus the `tax-rule-sourcing` skill doc itself so the correction doesn't get lost.

## 2. Scenario-test the actual calcTotals() logic

Pull the current `calcTotals()` and `TAX_YEAR_PARAMS` verbatim (copy-paste, not reimplemented from memory) into a script runnable via the Browser tool's `javascript_exec` (no local Node available in this environment). Build a matrix covering at minimum: a net loss, a simple mid-income case, the QBI phase-out taper at multiple points (below/at/deep-in/past the threshold) for single, MFJ, and HOH separately, the SS wage-base cap triggered two ways (net SE alone, and W-2 wages reducing the room), Additional Medicare Tax on and off, and all three safe-harbor branches (90% wins, 100% wins, 110% multiplier).

Hand-verify at least 3-4 of the most complex scenarios by working the arithmetic out independently (not just re-running the same formula) — this is what actually catches a logic bug, not just a bad input constant. Then cross-check one scenario against the real running app via the actual UI (fill fields, read rendered DOM values) to confirm the script's copy of the logic matches production exactly.

## 3. Copy, spelling, and tone audit

Spawn a background general-purpose agent to read every user-facing string in both files (labels, placeholders, tooltips, full glossary/Terms entries, ToS, welcome card, dashboard labels, warning/error messages, Contact form copy) and report: spelling/grammar errors, terminology consistency (same concept named the same way everywhere), condescending or salesy tone, and plain-language readability for someone with no tax background. Continue other work while it runs in the background.

## 4. Live browser bug sweep

Using the Browser tool, on a genuinely fresh load (navigate away to the other file and back if reusing a tab, to avoid stale bfcache state — verify freshness before trusting any result):

- Click every "?" info button across every view, confirm no console errors and real tooltip content.
- Open every collapsed glossary/Terms entry.
- Test CSV export, the filter panel, the sort dropdown, and the bottom-sheet edit flow.
- Hover/tap every chart shape on the Dashboard and confirm tooltips render.
- Add an entry, reload the page for real, confirm it persisted via localStorage.
- Spot-check any date-sensitive logic (e.g. the mid-year mileage rate boundary) — use the `change` event, not `input`, when scripting a native date input, or you'll get a false negative.
- Clean up every test entry you add along the way — don't leave synthetic data in real localStorage state.

## 5. Verify index.html and demo.html stay in sync

Diff the calculation-critical sections (`calcTotals`, `TAX_YEAR_PARAMS`, `MILEAGE_RATES`) directly — they should be byte-identical. Confirm the only differences between the full files are the intentional ones (demo banner, `DEMO_ENTRIES`, manifest link, page title).

## 6. Compile and act

Collect everything from steps 1-5 into one report. Fix clear-cut issues (typos, confirmed wrong figures, obvious bugs) directly in both files and verify the fix in-browser. Flag anything that's a genuine judgment call (wording preference, a figure that couldn't be confirmed from a primary source) for the user instead of guessing. Present a clear before/after summary — don't just say "all good," show what was actually found and changed.
