# buildium-reports

Self-contained Daisy Buildium financial report generator.

**One file, one command.** `buildium_report.py` pulls financial data from the
Buildium API for a single association and produces Excel + PDF reports. It
supports 14 report types (general ledger, income statement, balance sheet,
trial balance, AP/AR aging, budget vs actual, check register, rent roll,
owner statement, vendor payment summary, bank register, deposit detail,
board roster) plus a batch "all" mode that runs 13 of them in one shot.

The Daisy logo used in the PDF headers is embedded in the script as a
base64 blob, so **no separate assets directory is required** at runtime —
the repo ships one as a convenience for local dev, but the script decodes
the embedded blob automatically to a temp path when `assets/daisy-logo.png`
isn't present next to it.

## Why this repo exists

This repo is public so Claude Code routines can `curl` the script without
needing a `GITHUB_TOKEN` or the Claude GitHub App installed. The canonical
copy lives under `christopherschneider-gif/daisy-agent-platform` at
`scripts/buildium_reports/buildium_report.py`; this repo mirrors it for
anonymous access.

## Raw URLs

- Script: <https://raw.githubusercontent.com/christopherschneider-gif/buildium-reports/main/buildium_report.py>
- Logo:   <https://raw.githubusercontent.com/christopherschneider-gif/buildium-reports/main/assets/daisy-logo.png>

## Running locally

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv/Scripts/activate
pip install requests python-dotenv openpyxl reportlab Pillow

# Create .env next to the script:
#   BUILDIUM_CLIENT_ID=...
#   BUILDIUM_CLIENT_SECRET=...

python3 buildium_report.py \
  --building "571 Hudson Street Apartment Corp" \
  --association-id 170555 \
  --report-type all \
  --start-date 2025-01-01 \
  --end-date 2025-12-31 \
  --basis Cash
```

Output lands in `./out/` as `<building> - <report_type> - <start> to <end>.xlsx`
and `.pdf`.

## Report types

| slug                    | description                                      |
|-------------------------|--------------------------------------------------|
| `general_ledger`        | All GL transactions grouped by account           |
| `income_statement`      | Revenue, expenses, net income                    |
| `balance_sheet`         | Assets, liabilities, equity (A = L + E)          |
| `trial_balance`         | Debit/credit columns per account                 |
| `ap_aging`              | Unpaid bills bucketed by age                     |
| `ar_aging`              | Outstanding owner balances bucketed by age       |
| `budget_vs_actual`      | Budget → actual → variance per P&L account       |
| `check_register`        | All checks written from the operating bank       |
| `rent_roll`             | Units, owners, current balance                   |
| `owner_statement`       | Per-owner transaction detail (needs a unit)      |
| `vendor_payment_summary`| Paid bills grouped by vendor                     |
| `bank_register`         | All bank transactions                            |
| `deposit_detail`        | Deposit slips with constituent payments          |
| `board_roster`          | Board members + contact info                     |
| `all`                   | Runs 13 reports in one invocation                |

Every report runs a cross-validation pass against Buildium's own
`/v1/glaccounts/balances` endpoint and emits a Cross-Validation tab in
the workbook so any drift between the transaction-sum and balance-endpoint
period activity is flagged explicitly.

## License

Internal — Daisy Living.
