# Local paper simulator

Apply the [paper workflow](../SKILL.md). This reference documents the bundled `scripts/paper.py`, not the Coinbase CLI. It needs only Python 3.11+, without credentials, network access, or extra packages.

After installation, every call names the script and saved account profile by absolute path:

```sh
python3 /abs/skill/scripts/paper.py --profile /abs/paper/profiles/default.json COMMAND
```

The state file is SQLite on a writable local filesystem with hard-link support. It must be outside the installed skill directory, and each file holds one account. Name the database file itself, not a symlink. The script never resets or deletes state. A missing, corrupt, or unsupported file returns an error and is never recreated.

The profile is a small JSON locator, not another ledger: `{"version":1,"state":"/abs/paper/account.sqlite3","account_id":"paper-<uuid>"}`. The engine verifies the expected ID when opening the database, including before a trade inside its transaction. Missing or mismatched accounts fail closed. No default path is guessed. `--state` is still supported for deliberate first setup, diagnostics, and recovery; use the installed profile for routine commands.

## Bind once; resume from any authorized bot

Inspect an existing account first. Only if the user explicitly wants a genuinely new account, run `init` at a new path. Then bind the verified ID returned by `doctor` or `init`:

```sh
python3 /abs/skill/scripts/paper.py --state /abs/paper/account.sqlite3 doctor
# For a genuinely new account only:
python3 /abs/skill/scripts/paper.py --state /abs/paper/account.sqlite3 init --cash 10000 --currency USD --fee-bps 10
# Bind the existing account; substitute its actual ID:
python3 /abs/skill/scripts/paper.py bind --state /abs/paper/account.sqlite3 --profile /abs/paper/profiles/default.json --account-id "paper-<uuid>"
python3 /abs/skill/scripts/paper.py --profile /abs/paper/profiles/default.json status
```

Do not run this block blindly: `doctor` returning MISSING does not prove the user has no prior account. `bind` requires a valid existing account and the matching ID. Repeating the same binding is safe; binding a different path/ID at an existing profile is rejected. `init --profile` is always rejected. Profiles and newly created directories are private; existing unrelated directory permissions are unchanged.

`doctor` returning `ok: true` means the diagnostic ran, not that paper trading is ready. With `--state`, inspect `state_status`: MISSING means no account at that path. With a configured `--profile`, missing state is an error. Even `state_status: OK` does not verify host persistence, market-data access, or permission controls; those requirements must be checked separately.

| Command | Purpose |
| --- | --- |
| `doctor` | Check runtime and state health. Never creates an account. |
| `init --cash 10000 --currency USD --fee-bps 10` | Create a new account and return `account_id`. Refuses an existing file. |
| `bind --state FILE --profile FILE --account-id ID` | Save an existing account's path and expected ID. Creates no virtual cash. |
| `trade --request FILE` | Fill or reject a saved request in one atomic step. `--request -` reads stdin. |
| `status [--books FILE]` | Show cash, positions, average cost, and realized, unrealized, and total P&L, plus equity. |
| `history [--limit N]` | Return recent orders as `{"orders": [...]}`. |
| `backup --output FILE` | Write a consistent copy to a new absolute path. Never overwrites. |

## Output

Success prints JSON with `"mode": "PAPER"`. All money and size values are decimal strings. Report them as given; do not convert them to floats.

- `trade` receipts and `history.orders` entries include `request_id`, `account_id`, `product_id`, `side`, `base_size`, `notional`, `fee`, `cash`, and `snapshot_time`.
- `average_price` is the depth-weighted fill price before cash rounding and fees. Buy notional rounds up to the quote increment, sell notional rounds down, and fees round up on the rounded notional. Partial-sale cost basis is allocated proportionally, rounded half-even; a full close removes the remaining basis.
- `status` includes `account_id`, `currency`, `initial_cash`, `cash`, `fee_bps`, `realized_pnl`, `unrealized_pnl`, `total_pnl`, `equity`, and `positions`. Each position includes `product_id`, `quantity`, `cost_basis`, `average_cost`, `mark`, `market_value`, `unrealized_pnl`, and its mark status and time. With no fresh mark, the valuation fields are `null`; they are never valued at cost. With no positions, equity equals cash.

Errors print JSON with an error code and message and exit nonzero. A rejected trade changes nothing. Report the code. Do not loosen the request to get it through.

`STATE_PATH_UNAVAILABLE` means the location cannot create state or a backup. Choose a writable local filesystem with hard-link support; do not change permissions on unrelated directories. `STATE_BUSY` means another process holds the database; retry the same request, not a new ID.

`PROFILE_NOT_FOUND`, `PROFILE_INVALID`, `STATE_NOT_FOUND`, and `ACCOUNT_MISMATCH` on a configured profile require recovery, not new initialization. `PROFILE_EXISTS` refuses a different binding. `PROFILE_PATH_UNAVAILABLE` means the filesystem cannot safely create the locator. Report the code and preserve existing files.

Inputs use plain decimal strings with up to 15 integer and 18 fractional digits, not exponent notation. Product minimums apply to the notional before fees, so a budget equal to the product's minimum can be too small once fees are included. Explain `BELOW_MINIMUM`; do not increase the user's budget automatically.

## Idempotency and recovery

The script checks the `request_id` before market freshness. Rerunning a completed request returns the original receipt, even with a stale or refreshed book. Reusing an ID with a different account, product, side, or size is rejected as a conflict. Concurrent runs of the same file fill once. Two different IDs are two orders, even when their terms match.

Keep the original database and profile. To inspect a backup, pass its path as `--state`; account and completed request IDs are preserved. If the user approves resuming that backup, verify its ID and bind it at a **new** profile path, then update the host's recorded locator explicitly. Do not automatically overwrite an old binding, delete state, or run two writable copies of the same account as if they were synchronized. Back up the locator and database separately; `backup` copies the database only.
