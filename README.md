# CM Project — Financial Tracker

Single-file HTML financial tracker for a PGK-currency gaming operation with MYR reserve.

## Overview

- **Storage:** `localStorage` (no server, no backend)
- **Currency:** PGK (primary) + MYR (RM reserve)
- **Users:** 1 operator (end-of-day data entry) + PM (review)
- **Platforms:** LUCKY888, LUCKY777
- **Banks:** 2x BSP + 1x Kina (PGK side), 3+ MYR accounts

## Architecture Decision

Exchange transactions link to **Platform**, not bank. One entry drives:
1. MYR bank ledger (`buildLedger()`)
2. PGK platform reconciliation (`buildPGKLedger()`)

## Status

### Built
- PDF export engine (no CDN)
- Funder Movements / Funder View fixes
- PGK entry fields: `txnCount`, `newDep`, `rebate`, `bankCharges`
- Exchange platform field
- Updated reconciliation formula
- PGK platform reconciliation view

### Pending
- Per-PGK-bank deposit field in Daily Entry
- Internal PGK bank-to-bank transfers (structured, not via Adjustments)
- PIN-based permission levels (Operator vs PM)
- Setup scalability audit

### Excluded
- Kina Bank monthly limit tracker

## Files

- `financial_tracker.html` — main app (3417 lines)

## How to run

Open `financial_tracker.html` in any modern browser. Data persists in browser `localStorage`.
