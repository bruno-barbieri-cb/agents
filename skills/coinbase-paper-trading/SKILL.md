---
name: coinbase-paper-trading
description: Simulate Coinbase crypto spot market orders with virtual cash in a local paper account. Use for paper trading, practice or demo trades, trading without real money, trying a strategy idea, or checking simulated positions and P&L. Requires a terminal, Python 3.11+, durable local storage, and read-only Coinbase MCP market data; unsupported in chat-only or ephemeral execution environments. Never places real orders, reads balances, or spends funds.
metadata:
  author: Coinbase
  version: 1.1.0
---

# Coinbase paper trading

Every result is **PAPER**: virtual cash, simulated fills, no real order. No API key, Coinbase CLI, Node.js, credentials, or portfolio access is needed or used.

## Required capabilities — check before setup or execution

- Agent-accessible terminal execution and Python 3.11+ with standard-library SQLite.
- Authorized durable local files for state, profiles, and saved requests, outside installed code. Storage must support SQLite locking/hard links and survive terminal exits and new conversations. Verify host retention or an operator-confirmed persistent disk; a successful write is not proof.
- Read-only Coinbase MCP product/book access and a complete JSON-to-file handoff.
- Host-enforced read-only tools or mandatory real-financial-action approvals for the supervised pilot.
- A way to retrieve this guide and the saved profile path in a fresh conversation; do not rely on chat memory.

If any requirement is missing or unknown, **stop setup and trading** and explain what is needed. Chat-only clients and ephemeral code sandboxes without durable storage are unsupported. Never invent a ledger in the conversation, promise memory-based persistence, or fall back to real money. Check capabilities, not client brand. See [requirements and onboarding](START-HERE.md); filesystem tests cannot certify VM retention.

Use both references together:
- [MCP market-data handoff](references/mcp.md): fetch product and order-book data and save the trade request.
- [Local simulator](references/cli.md): run the bundled `scripts/paper.py`. Here `cli.md` documents that script, not the Coinbase CLI.

Use the stable absolute runtime and account-profile paths recorded by the host installation. Run `python3 <skill-dir>/scripts/paper.py --profile <profile-file> COMMAND`. The profile pins the database path and expected account ID; it is not a paper/live switch. Shell variables and conversation memory are not durable configuration. See [host setup](references/host-setup.md) to install this capability for fresh bots.

## Resolve intent before acting

- “Can I paper trade?” is a capability question: explain the local simulator and check readiness without creating an account or placing a trade. A live-only MCP environment does not mean local paper trading is unavailable.
- Explicit paper, practice, demo, or simulated trades use this workflow only. An unambiguous follow-up in the same paper conversation stays paper.
- A buy/sell request in a fresh conversation, conflicting instructions, or uncertain paper/live intent requires clarification before acting. Do not infer live intent from the connector environment or paper intent merely from an installed account.
- An explicit switch to real money leaves this workflow and requires separate live authorization and host approval. No global mode flag; one bot must not change another bot's execution context.

## Set up

1. Read the host's saved installation context: absolute runtime path, profile path, account scope, and storage limitations. If these are unavailable, inspect the host catalog/configuration or follow [host setup](references/host-setup.md); do not guess a new per-bot database path.
2. For a configured profile, run `--profile <profile-file> doctor`, then `status`. A missing binding, missing database, invalid state, or account-ID mismatch is a recovery problem. Stop; never fall back to `--state ... init` or silently seed new cash.
3. On first setup without a binding, establish whether the user already has a paper account. Inspect and bind that existing database using its verified account ID. If the user expects an account but it cannot be found, ask for recovery information rather than initializing.
4. Only for an explicitly new account, select a stable state path outside code and repositories, run `--state <state-file> init`, then `bind` it as described in [local commands](references/cli.md). Defaults are 10000 USD and a 10 bps assumed fee, not the user's Coinbase fee tier. If setup stops between init and bind, bind the existing file on retry.
5. Report PAPER, account ID, cash/currency, fee assumption, profile and state paths, and verified storage scope. Unknown persistence blocks setup; VM replacement durability is a separate claim. Matching paths on different VMs do not mean shared state.

`init` refuses existing files and cannot use `--profile`. `bind` never changes balances or overwrites a different binding. There is no reset command: an explicitly requested separate account needs separate state and profile paths; keep the originals.

## Paper trade

1. Use the user's specified product, side, and size; clarify missing terms. Use exactly one of `base_size` or `quote_size`. A BUY can use either; a SELL uses `base_size` only. `quote_size` is the total spend including fees.
2. Before any execution, save a request file with a newly generated UUID `request_id` and the `account_id`. Keep requests under the verified durable state root, outside installed code, for example `<state-dir>/requests/<request_id>.json`. Use `umask 077` when creating files and directories. Never reuse an example UUID or overwrite a different request.
3. Fetch `coinbase_products_get` once per product per session and reuse that exact metadata. Fetch `coinbase_products_book` for each new order, copy the responses into the request without changing values, and run `trade` within 60 seconds of the book time.
4. Report the receipt as a PAPER fill: size, notional, fee, cash, and snapshot time.

**Retries.** If a result is uncertain, run `trade` again with the same file. A completed request returns its original receipt, with no new market-data reads. If the book was stale and nothing filled, refresh the book in the same file once, keeping the same `request_id` and terms. Never create a new ID to retry. A new order the user asks for gets a new file and ID.

**Rejections.** Explain the error code. Do not shrink the size, switch product or currency, invent data, or move to live trading.

## Review

- `status --books FILE` values open positions at fresh best bids. Without fresh marks, positions are unpriced and equity and unrealized P&L are `null`.
- `history --limit N` lists recent paper orders.
- `backup --output FILE` writes a consistent copy to a new file. Restore deliberately and verify the same account ID before binding a new profile; see [recovery](references/cli.md#idempotency-and-recovery).

## Model and limits

- Fills use only the saved order-book snapshot: asks for buys and bids for sells, best price first, weighted by depth. The full size fills or the trade is rejected. A quote budget buys the largest base increment it can afford after the fee, which is rounded up to the quote increment. Snapshots older than 60 seconds, or more than 5 seconds in the future, are rejected.
- Positions use weighted-average cost. Buy fees add to cost basis and sell fees reduce proceeds. Marks use the best bid and exclude hypothetical exit fees.
- Scope: crypto spot market orders, one quote currency per account, run manually. There are no equities, derivatives, leverage, shorts, limit or resting orders, backtests, scheduling, cloud sync, or live mode. Internet equity quotes may inform separate research, but cannot be used for fills or marks here.
- Results do not predict real returns. The model has no queue position, latency, market impact after the snapshot, or adverse selection. The state file is local and user-editable.

## Safety boundary

The paper workflow uses only read-only product and order-book tools plus this local script. Never call order, preview, conversion, transfer, x402, or payment tools for a paper request. Live trading is a separate authorized workflow, not a fallback. These instructions are not an enforcement boundary for an agent with live permissions: use read-only tool restrictions where supported and keep mandatory financial-action approvals on. Optional candle research uses the read-only tool described in the MCP reference. Never use candle closes or reference prices as fills.
