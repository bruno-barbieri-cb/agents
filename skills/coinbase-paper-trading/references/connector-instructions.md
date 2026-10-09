# Optional connector routing hint

This is a host-side instruction template, not an MCP server feature or a permission boundary. Apply only through a supported configuration mechanism, preserving the existing instructions and approvals. Replace both placeholders with verified absolute paths after installation. If setup is blocked, do not advertise a ready account; report the missing requirements instead.

---

Coinbase MCP order tools affect real funds. Local paper trading is a separate capability, not an MCP environment switch. A live-only connector environment does not mean local paper trading is unavailable.

For paper, practice, demo, simulated trading, or trading without real money, discover the installed `coinbase-paper-trading` skill and read its canonical entry at `{{SKILL_PATH}}`. Use the saved account locator at `{{PROFILE_PATH}}`. The guide requires terminal execution, Python 3.11+ with SQLite, durable files that survive new conversations, complete read-only MCP market-data handoff, and read-only restrictions or mandatory financial-action approvals. If a requirement is absent or unverified, stop setup and execution; explain the missing capability. Never keep a pretend ledger in chat memory.

“Can I paper trade?” is an information/readiness question, not permission to create an account or place a test trade. Explicit paper requests and unambiguous follow-ups in the same paper conversation use only read-only market data plus the local simulator. An ambiguous new-conversation buy/sell requires asking paper versus live. Do not infer intent merely from an installed account, previous bots, or the MCP environment.

For paper requests, never call live order, preview, conversion, transfer, x402, or payment tools. An explicit request to switch to real trading leaves the paper workflow and requires the normal separate live authorization and host approvals. Never repurpose paper IDs for live orders. Do not change a shared paper/live flag.

Resume the saved paper account, verifying its expected ID. Missing bindings, missing state, or identity mismatches mean stop and recover, not initialize a new balance. Do not guess a new per-bot state path. Check actual installation before claiming availability; a catalog entry alone does not prove its code, storage, or account is accessible.
