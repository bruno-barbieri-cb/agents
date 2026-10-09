# Coinbase paper trading — start here

**A guide and local runtime, not a hosted paper exchange.** Use real Coinbase market data with virtual cash. Nothing in this package places a real order. Read [SKILL.md](SKILL.md) for the workflow; [host setup](references/host-setup.md) covers installation and requirements.

## Is this environment eligible?

All of these are required before setup or execution:

1. **Terminal execution:** the agent can run a shell command and read its result. Python 3.11+ with the standard-library `sqlite3` module must be available. Merely displaying code is not execution.
2. **Durable local storage:** an authorized, stable writable filesystem for the database, account locator, and saved requests, outside the installed code. It must support SQLite locking and hard links, and survive terminal exits and new conversations on the same host. A chat transcript, model memory, downloadable attachment, or ephemeral code-interpreter sandbox is not this storage. If persistence is unknown, stop setup; do not just warn and proceed.
3. **Market-data handoff:** access to the read-only Coinbase MCP product and order-book tools, and the ability to save their complete JSON results to files without inventing or changing values.
4. **Safe execution:** paper/live intent is resolved before acting. Use host-enforced read-only tool access when available. Otherwise the supervised pilot requires mandatory approval for real financial actions. Never broaden credentials or disable approvals to install this package.
5. **A resume path:** the agent can retrieve this guide and the saved absolute runtime/profile paths in a new conversation, through durable host configuration, a native skill, or an explicit user-provided locator. Do not rely on one bot remembering the setup.

Check capabilities, not brand names. A desktop terminal or a hosted agent VM may qualify, but only with appropriate storage and controls. A web chat with only MCP access does not qualify. A hosted terminal alone is insufficient. Local filesystem checks cannot prove host retention: record the host's documented storage lifecycle or the operator's confirmed durable local-disk location, then test a fresh conversation. VM deletion/replacement is a separate durability claim, never implied by a successful restart or file write.

**If a requirement is absent or unknown, do not initialize or trade.** Explain what is missing and offer to help set up a capable environment. Do not improvise a ledger in chat, claim to save it in memory, switch to live orders, or suggest real money as a substitute for paper trading. You may still explain the simulation or help design a strategy without claiming it ran.

Example response:

> This chat can explain Coinbase paper trading, but it cannot run and retain a paper account here. The setup needs a terminal with Python 3.11+, persistent files, and read-only Coinbase market data. Use a capable local terminal or persistent agent VM; I won't simulate an account by remembering balances in this chat.

## What to give an eligible agent

> Read START-HERE.md and SKILL.md in this bundle. Check every environment requirement first and report anything missing or unverified. Set up local PAPER trading only if the requirements pass. Find and resume my existing paper account. If none is found, ask me before creating a new account; missing expected state requires recovery, not new cash. Do not place a test trade unless I explicitly ask. Never call live order, preview, transfer, conversion, or payment tools. Record the installed runtime path and account-profile path so a fresh conversation can resume.

Extract the complete verified bundle, not just the Markdown file. The runtime is `scripts/paper.py`; the only dependencies are Python and SQLite. Verify the archive digest against the trusted handoff/release, then its internal `SHA256SUMS`. An internal checksum list detects corruption, not authenticity by itself. No download, package installation, OAuth change, or native registration happens automatically.

For an existing account, setup only binds its verified path and ID; it does not seed or reset balances. For a genuinely new account requested by the user, defaults are 10000 virtual USD and a 10 bps assumed fee. [Local commands](references/cli.md) explain init, bind, status, trade, backup, and recovery.

## Discovery and scope

This package is host-neutral. A native skill catalog is an optional convenience, not a runtime dependency. Merely extracting the ZIP does not register a skill. Without a native installation or an MCP advertisement, a new agent must be given this guide explicitly.

A future read-only MCP guide tool can advertise the capability and return this same guide plus a versioned bundle reference. That is discovery only: it does not make an unsupported chat client eligible, create a hosted account, toggle live order behavior, or supply shared storage. No MCP server change is included in this package.

Scope: manual crypto spot market simulation using saved book depth. No equities, leverage, shorts, resting orders, background scheduler, cloud synchronization, or live mode. Results are approximate and do not predict real returns.

## Verify before calling the pilot ready

Use [the QA checklist](references/qa.md) with a separate disposable QA account. Runtime tests and static instruction checks do not prove an agent will route correctly. Native discovery, cross-conversation continuity, and approval enforcement require tests in the actual host. Keep untested properties explicitly unverified.
