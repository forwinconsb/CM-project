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
- **Reconciliation formula (PGK platforms):**
  `Expected Closing = Opening + Deposit − Withdraw − Bank Charges − ExchangeOut + ExchangeIn ± Adjustments`
- **MYR bank reconciliation is MONTHLY, statement-anchored (no daily bank entry).** Operators do
  NOT enter a daily MYR closing. The MYR balance is computed from transactions (`buildLedger`:
  exchanges, due settlements, funder moves, bank-paid expenses). Each month: opening = the
  PREVIOUS month's entered statement balance (`getStmtBalance(bank, prevMonth(m))`); book closing =
  opening + in − out; you reconcile it against THIS month's statement balance typed in the
  Reconciliation tab (`S.stmtBalances`, key `bank||YYYY-MM`). So statement balances chain
  month-to-month and are the only manual MYR figure.
- **P&L uses fixed monthly FX rates** (`S.fxRates`, one rate per month, independent) to translate
  PGK→MYR; an FX gain/loss line bridges fixed vs actual exchange rates. Cash figures are a memo only.
- **Accounts/roles** (`S.accounts`): Operator/Manager/Admin/custom tab access via `applyAccess()`.
  This is role separation, NOT real security (client-side). Real security = the cloud migration.
  - **View-only accounts** (`account.viewOnly`, e.g. the Funder role): `applyAccess()` adds a
    `body.readonly` class; CSS hides all add/edit/delete actions and entry forms (`.op-only`).
- **Period lock / month-end close** (`S.lockedMonths`, array of `'YYYY-MM'`): once a month is
  locked (Admin only, button in Reconciliation), every transaction *dated in that month* is frozen.
  **Invariant — any function that creates, edits, or deletes a dated record MUST call
  `lockBlocked(date)` and bail if it returns true** (see existing guards in `saveDay`, `saveFndTxn`,
  `saveExpTxn`, `savePlat2`, `saveExchDay`, `saveDueReceipt`, all the `delete*`/`*Edit` fns).
  Frozen-fund write-off/recovery and due settlements key off their OWN (current) date, so collecting
  old debts after a past month is locked still works — do not key the check off the record's age.

## Conventions — follow these
1. **Mobile check is mandatory.** After ANY UI change, verify at 375px width: no horizontal
   overflow, grids readable. There is an `@media print` and a mobile media query already.
1a. **Bilingual (EN / 中文) — translate new text.** The app switches language via `data-i18n`
   attributes + the `LANG` object (`LANG.en` / `LANG.zh`) read through `tr(key)`. When adding ANY
   user-facing wording, add a key with BOTH an English and a **Chinese** value (put the 中文 string
   in `LANG.zh`) and reference it — do not hardcode an English-only string in new features, or it
   stays English in 中文 mode. **Priority:** operator-facing tabs (Daily Entry, PGK Platforms,
   Exchanges, Expenses) must be translated (the operator reads Chinese); PM/report tabs are lower
   priority. Note: features added rapidly may have English-only text — a translation pass is
   worthwhile when convenient.
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

## Deploying updates safely
Code and data are separate — a code update never changes saved data; only `migrateData()`
touches data shape, and it upgrades old data automatically on load. Users receive an update
only when they reopen/refresh — it is never pushed onto an in-progress session.

Before pushing any update (especially a major/data-restructure one):
1. **Backup data first** — Setup → Export full backup (.json). (Online later: Supabase keeps backups.)
2. **Test the new version against a COPY of real data** — confirm zero console errors and that
   existing entries still read correctly.
3. **Deploy off-hours** — after the operator has finished end-of-day entry.
4. **Keep rollback ready** — git history is the rollback for code; the .json export restores data.
5. **Tell users to refresh / reopen** to pick up the new version.

Minor updates (new field/feature) are essentially invisible — merge-on-load handles old data.

**Offline gotcha (current setup):** `localStorage` is tied to the file's location. When replacing
the HTML, put it in the SAME folder/path, or the browser won't find the old data (it's not lost —
it's under the old location; Import the .json backup to recover). Going online (Method 2) removes
this risk entirely — same URL always.
