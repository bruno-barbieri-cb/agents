# Changelog

## 0.0.1 — Unreleased

- Treat this repository as canonical source instead of an installable plugin root.
- Generate portable, Claude, and Codex distributions from one skill library.
- Isolate OpenAI assets under the portable `com.openai` extension directory.
- Publish versioned archives, checksums, unpacked packages, and release metadata through GitHub Pages.
- Add `coinbase-paper-trading`: a local, stdlib-only Python simulator for crypto spot market orders that uses read-only MCP product and order-book data. Distributions include the bundled script without Python caches.
- Add a host-neutral paper-only bundle with mandatory terminal, Python/SQLite, durable-storage, data-handoff, and approval requirements. Native skill/connector registration remains optional; unsupported chat-only environments stop instead of keeping a ledger in memory.
- Add saved paper-account locators (`bind` / `--profile`) with identity checks and no automatic reseeding, plus explicit paper/live intent routing and host acceptance tests. Paper skill version 1.1.0 retains the existing fill model and database schema.
