# CM Project — Roadmap

## Current architecture (Phase 1 — now)

Single-file HTML app (`financial_tracker.html`), localStorage, no server.

- 1 operator enters data end-of-day; PM reviews via PDF exports
- Data lives in the browser on one computer
- **Backup discipline:** Setup tab → Export full backup (.json) after every session; OneDrive syncs it

### Done
- PDF export engine (no CDN)
- Internal PGK bank-to-bank transfers (zero-sum, audit trail)
- Per-PGK-bank deposit verification vs platform totals
- Setup: rename with history cascade, safe delete warnings, platform ↔ PGK bank link
- PGK entry fields: txn count, new depositors, rebate, bank charges
- Exchange ↔ platform link (entry, listing, edit, CSV)
- Reconciliation formula with auto-linked exchanges
- PGK Platform Reconciliation view (computed vs entered closing per platform/day)

### Parked
- PIN-based permission levels (cosmetic only in client-side HTML — real roles come in Phase 2)
- Kina Bank monthly limit tracker (excluded by decision)

---

## Phase 2 — Cloud migration (when triggered)

**Triggers (migrate when ANY of these happens):**
1. A second operator joins
2. PM needs live remote access from own device
3. More than ~5 platforms
4. Data approaching localStorage limits (~5MB)

**Target architecture: Supabase (PostgreSQL + Auth + Realtime)**

```
Users log in (operator / PM / admin roles)
        → Web app (same screens as today)
        → Supabase cloud DB
           ├── Auth: email/password login
           ├── Row-level security: DB enforces per-role access (real security)
           ├── Realtime: PM sees entries live as operator saves
           └── Automatic backups
```

**Why Supabase:** SQL fits ledger data; role enforcement at database level (not just hidden buttons); built-in realtime; free tier covers current scale; no server maintenance.

**Migration plan (high level):**
1. Create Supabase project; design tables mirroring the `S` object:
   `platforms`, `pgk_banks`, `myr_banks`, `exchangers`, `funders`, `exp_types`,
   `plat_entries`, `day_entries`, `exchanges`, `transfers`, `pgk_bank_deps`, `expenses`, `funder_txns`, `stmt_balances`
2. Add login page + role assignment (operator / PM)
3. Row-level security policies: operator → entry tables only; PM → everything
4. Swap localStorage read/write for Supabase calls (logic and formulas unchanged)
5. Enable realtime subscriptions on entry tables
6. One-time import of the latest .json backup
7. Host the app (Supabase hosting or GitHub Pages)

**Effort estimate:** several working sessions, not a single day. The current HTML is the blueprint — all formulas and workflows carry over.

---

## Operating rules (any phase)

- Exchange transactions link to **Platform**, not bank (deliberate architecture — do not change)
- Reconciliation formula:
  `Expected Closing = Opening + Deposit − Withdraw − Bank Charges − ExchangeOut + ExchangeIn ± Adjustments`
- Internal bank transfers are zero-sum and never affect reconciliation
- Renames must cascade through historical records (already implemented)
