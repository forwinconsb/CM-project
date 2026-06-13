# CLAUDE.md — guide for AI-assisted development of this project

This file is read first by Claude Code (and any AI assistant) before working on this repo.
It captures how the app is built and the conventions to follow so future updates are safe.

## What this is
A single-file financial tracker (`financial_tracker.html`) for a PGK-currency gaming
operation with MYR (RM) as reserve currency. No build step, no backend — it runs by
opening the HTML file in a browser. All data persists in browser `localStorage`.

- **Operator** enters end-of-day data from platform screenshots; **PM** reviews.
- Owner is a **beginner** to git/CLI, on **Windows**, reads **Chinese** (app is bilingual EN/中文).

## Architecture (must preserve)
- **One global state object `S`** holds everything; `persist()` writes it to localStorage,
  `loadData()` reads it back (merging over defaults so new fields are safe).
- **Exchange transactions link to a Platform, NOT a bank.** One exchange entry feeds both
  the MYR bank ledger (`buildLedger`) and the PGK platform reconciliation (`buildPGKLedger`).
  Do not change this — it is a deliberate decision.
- **Reconciliation formula:**
  `Expected Closing = Opening + Deposit − Withdraw − Bank Charges − ExchangeOut + ExchangeIn ± Adjustments`
- **P&L uses fixed monthly FX rates** (`S.fxRates`, one rate per month, independent) to translate
  PGK→MYR; an FX gain/loss line bridges fixed vs actual exchange rates. Cash figures are a memo only.
- **Accounts/roles** (`S.accounts`): Operator/Manager/Admin/custom tab access via `applyAccess()`.
  This is role separation, NOT real security (client-side). Real security = the cloud migration.

## Conventions — follow these
1. **Mobile check is mandatory.** After ANY UI change, verify at 375px width: no horizontal
   overflow, grids readable. There is an `@media print` and a mobile media query already.
2. **New user-facing strings: prefer ASCII** where practical. The file has been hit by an
   encoding corruption before (BOM + mojibake on `✓`/emoji). Existing emoji are fine; just be
   careful introducing new multi-byte characters and never let a tool re-save with a BOM.
3. **PDF = the browser Print button** (`doPrint()` + `@media print`), which prints the current
   view exactly. Do NOT re-introduce per-tab custom PDF generators (they drift from the screen).
   CSV exports use the shared `dlFile()` helper — keep it.
4. **Adding a feature that needs a new `S` field:** just add it with a default. The merge-on-load
   handles old data automatically. No migration needed.
5. **Renaming/restructuring an existing `S` field:** bump `SCHEMA_VERSION` and add an
   `if(v<N){ ...transform p...; v=N; }` block in `migrateData()`. Old data upgrades on load.
6. **Verify before committing:** run the preview server (`.claude/launch.json` "tracker", serves
   the `CM-project` folder), check zero console errors, then commit + push.
7. **Don't run background-task chips that edit `financial_tracker.html` while actively editing it**
   — concurrent edits corrupted the file once (removed shared `dlFile`, added a BOM).

## Roadmap
See `ROADMAP.md`. Phase 2 = move online. Recommended path: **Method 2 (Supabase JSON sync)** —
store the `S` blob in the cloud with near-zero rewrite (fits the one-operator-at-a-time pattern),
then graduate to full tables + row-level security + realtime if the team grows.

## Repo
- GitHub: https://github.com/forwinconsb/CM-project
- Commit + push when the owner asks; keep commit messages descriptive.
