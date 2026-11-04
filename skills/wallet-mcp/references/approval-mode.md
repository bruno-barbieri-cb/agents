---
title: "Approval Mode"
description: "Skill reference for how Wallet MCP returns approval URLs and request IDs for every write call."
---

> Today the Wallet MCP exposes a single execution mode for write tools: **approval mode** (the user manually approves each transaction via a returned URL). Other modes may be added later. Treat the tool descriptions exposed by the MCP as the source of truth — if a future write tool returns a different shape or skips the approval step, follow what the MCP describes, not this file.

# Approval Mode

In approval mode, every write call (send, swap, sign, batched calls, and any plugin-prepared transaction routed through Wallet MCP) returns an **approval URL** plus a **request ID**. The agent opens the URL for the user (or shows it on chat-only surfaces), the user approves the action in Coinbase Wallet Account, and the agent polls the request ID until it confirms the outcome.

## Flow

1. **Call the write tool.** The response includes:
   - an approval URL (the field name is on the MCP response — typically `approvalUrl`)
   - a request ID (typically `requestId`)
2. **Show the user the link.** Present it as **"Approve Transaction"** (or similar neutral language). Refer to the approval destination as Coinbase Wallet Account, not as the raw URL hostname or an implementation-specific provider. Just give the user the link to click.
   - Beginner-friendly phrasing: _"Open this to approve the transaction: [Approve Transaction](https://<approval-url>)"_
   - Terse phrasing: _"[Approve Transaction](https://<approval-url>)"_
3. **In CLI harnesses, open the link automatically, right away.** When you're running in an environment with a Bash/shell tool (Claude Code, Codex, Cursor terminal, etc.), open the URL in the user's default browser as soon as the write tool returns, without asking first. Opening the page doesn't move funds: the user still reviews and approves there. Always print the link too as a fallback, then run the platform-appropriate open command:
   - macOS: `open "<url>"`
   - Linux: `xdg-open "<url>"` (fall back to `wslview` under WSL)
   - Windows: `start "" "<url>"` (or via PowerShell: `Start-Process "<url>"`)

   Skip this step on chat-only surfaces (ChatGPT, Claude.ai) — they don't have a shell, so just show the link.
4. **Poll for the outcome without waiting to be told.** Right after opening the link, call the status tool (typically `get_request_status`) with the request ID about every 5 seconds. Don't make the user come back and say "done".
   - Keep polling while it's pending (the user hasn't acted yet). Approval links expire after about 10 minutes, so stop then and tell the user the request expired.
   - Stop when it reaches a final status (completed, failed, rejected, expired) and report that status.
   - If the user says they approved before polling sees it, poll right away instead of waiting for the next tick.
   - On surfaces that can't run a background wait between tool calls, poll once after the user says they approved, then a few more times if it's still pending.
5. **Confirm the transaction landed before reporting success.** Report success only when the status tool confirms completion. Include the transaction hash.
   - **Solana `send` / `swap`:** the status tool never reports completion. Once it returns `signed` (approved and being sent), poll `get_transaction_history` on `chain: solana` about every 5 seconds, for up to about 2 minutes, until the new transaction shows up. Then report its status and hash. While `solanaState` is `pending`, it's still in flight. If it's `not_pending` and history still has nothing new after the 2 minutes, say the outcome is unknown and ask the user to check their activity. Don't start the same send or swap again while it may still be in flight.

## Common mistakes

- Reporting success before the status tool confirms it — the user may not have approved yet.
- Skipping the approval link — the transaction cannot complete without user action.
- Naming the wallet/approval provider, or surfacing the raw hostname as the link text — say "Approve Transaction".
- Waiting for the user to say "done" before checking status. Poll on your own, about every 5 seconds, so the outcome shows up without the user having to report back.
- Polling faster than every few seconds, or forever. Stop at a final status or when the link expires.
- Reporting a Solana send or swap as done on `signed` without seeing it in `get_transaction_history`.
- Forgetting to auto-open the link in CLI harnesses where a shell is available. Printing it alone makes the user copy-paste.
- Trying to auto-open in chat-only harnesses where no shell exists — that just produces an error.
