# Host-neutral installation and account continuity

Read [the requirements](../START-HERE.md#is-this-environment-eligible) and [the paper workflow](../SKILL.md) first. This is installation guidance, not authorization for trades. Stop if a required capability is absent or its availability is unknown. Nothing here changes the Coinbase MCP backend.

## 1. Check the actual host

- Verify an agent-accessible terminal and Python/SQLite, for example `python3 --version` and `python3 -c 'import sqlite3; print(sqlite3.sqlite_version)'`. Python must be 3.11+. Do not install dependencies without permission.
- Identify the durable local filesystem or host-documented persistent volume, its retention/deletion rules, and who can access it. On a desktop, an operator-confirmed persistent home directory can qualify. On a hosted VM, do not infer durability from `/workspace`, `/tmp`, a successful write, or the existence of a terminal. Unknown persistence blocks account setup. Do not create a chat-only or session-only substitute account.
- Discover the real read-only product/book MCP tool schemas and the supported file handoff. Do not fetch private balances or execute a preview to test authentication.
- Verify host-enforced read-only restrictions or mandatory real-financial-action approvals. Prompt instructions alone do not isolate a broadly authorized agent. If neither control is available, block execution of this pilot and report the missing backstop.
- Establish how a new conversation will retrieve the installed skill and its profile path. Distinguish skill-catalog sharing, filesystem sharing, and VM retention; they are different properties.

## 2. Install code, not state

Verify the trusted bundle digest and then internal `SHA256SUMS`. Keep the complete package at a stable code path. Keep account databases, locators, saved requests, and backups outside code and repositories. Do not overwrite an existing installation blindly. For an upgrade, keep the old bundle until the new one is checked; point the host at the new runtime while retaining the account locator.

The canonical entry is `SKILL.md`, with `references/cli.md`, `references/mcp.md`, and `scripts/paper.py` beside it. Use absolute installed paths in tool calls; a shell variable set in one call need not survive the next. A skill version change does not imply a database migration. Check `doctor`; unsupported schemas/models stop rather than auto-migrate.

## 3. Bind an account once

Recommended scope is a named default paper account per user/profile on an authorized shared machine, not automatically one account per bot. Different strategies may use explicitly separate profiles. QA uses a separate state/profile directory. A second VM with the same pathname does not share this account.

Read any existing installation record before choosing paths. If it specifies a profile, run that profile's `doctor`; failure means recovery. If no profile exists but the user already has an account, inspect the known database with `--state ... doctor`, verify its account ID, then use `bind`. Do not initialize a second default account just because a new bot cannot locate the first.

For an explicitly new account only, use a stable database path and `init`, then bind the returned ID to a stable profile path. If interrupted between those steps, resume by inspecting and binding the already-created database. See [commands and recovery](cli.md). `bind` never changes balances. Repeating the same binding succeeds; a different existing binding is not overwritten.

Record outside chat, using supported host configuration or a private file reachable from the installed entry:

- Absolute skill/runtime and reference paths; package version and trusted digest.
- Absolute profile path; expected account ID and database path from successful setup.
- Account alias/scope and authorized storage-sharing scope.
- Storage lifecycle evidence, including what is **not** retained across VM replacement.
- Discovery method and effective read-only/approval controls; distinguish verified controls from proposed ones.

A missing record is not evidence that no account exists. Records contain local paths/IDs, not Coinbase credentials. Do not publish them in source or the package.

## 4. Optional native discovery adapter

Use the host's documented skill/plugin mechanism if available. Inspect its current API and scope rather than guessing commands. Register only `coinbase-paper-trading` for this pilot; preserve existing skills, unrelated settings, and approval controls. Make the canonical description visible for “paper,” “practice,” “demo,” and “without real money.”

If the host registers Markdown without bundling helpers, retain the full package at the stable code location. Include the absolute path to the canonical `SKILL.md` plus the saved profile path in the registration body; instruct the agent to read the canonical file before acting. Ensure all referenced files are actually accessible to fresh bots. Do not point relative links into a Markdown-only catalog entry or render an installed badge as proof. Where native installation carries the full skill directory, keep relative references intact.

Optionally merge [the connector routing hint](connector-instructions.md) into the host's supported Coinbase connector instructions after saving the prior instructions. Replace placeholders with verified paths, preserve other instructions, and check for unresolved placeholders. Do not create a machine-global paper/live flag. If this hook is unavailable, explicitly give the guide and locator to the agent; the simulator still works, but automatic cold discovery is not established.

For hosts exposing tools named `UpdateSkill` or `SetMcpInstructions`, treat those names as hints, not a promised API or permission scope. Inspect supported schemas, read existing configuration first, and verify returned catalog/configuration records. Do not assume user-wide visibility, helper-file persistence, or new-bot refresh from a successful write alone.

## 5. Confirm and test

Report **PAPER configured**, not “live sandbox,” with the account ID, profile/state paths, defaults or selected assumptions, and verified storage scope. A capability question alone must never initialize an account or place a smoke trade.

Follow [host acceptance](qa.md) before claiming automatic discovery or cross-bot resume. Keep unsupported or untested hosts unverified; a desktop test is not evidence of a hosted VM's persistence. For rollback, restore only the configuration entries changed during installation and point to the retained old code if needed. Never delete or reset the paper database as part of uninstalling, rollback, or a failed discovery test.
