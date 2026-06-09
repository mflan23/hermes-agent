---
name: hermes-system-config
title: Hermes System Configuration & Provider Routing
description: Defines active providers, disabled providers, tool routing rules, and platform-specific fixes for Mary's Windows environment.
version: 1.0.0
metadata:
  hermes:
    autoload: true
    scope: global
    priority: critical
    tags: [Config, Providers, Routing, Windows, PDF, Tools]
---

# Hermes System Configuration v1.0

> Loads at session start, globally. Defines what is active, what is disabled, and how to handle known failure modes.

---

---

## PDF & Local File Handling

**Critical:** Local file paths (C:\Users\bayfl\...) must NEVER be passed to `web_extract` or any web tool. This causes blocked requests and silent retry loops.

**Correct routing for local files:**

| File Type | Correct Tool | Never Use |
|---|---|---|
| Local PDF (C:\...\file.pdf) | `ocr-and-documents` skill | `web_extract`, `web_tools` |
| Local text/markdown file | `read_file` tool | `web_extract` |
| Remote URL (https://...) | `web_extract` | `read_file` |
| Facebook/social media URL | Skip — no content to process | `web_extract` (always fails) |

**Known blocked URL patterns (do not attempt):**
- `https://www.facebook.com/*` — no content to process
- `C:\*` passed as URL — blocked as private network address
- Any URL scheme other than `http://` or `https://`

**Known court documents in local workspace:**
- `C:\Users\bayfl\Downloads\06052026.pdf` → use `ocr-and-documents`
- `C:\Users\bayfl\Downloads\bo2026 court.pdf` → use `ocr-and-documents`
- `C:\Users\bayfl\Downloads\2 BOP_ Research & Reports.pdf` → use `ocr-and-documents`

---

## Windows Terminal Error Handling

Hermes is running on **Windows**. The terminal tool encounters COM registration errors on this machine.

**`Class not registered` error:**
- This is a fatal Windows COM error. **Do not retry.**
- On first occurrence: report to user, propose alternative tool (Python script via `execute_code`, or file operation via `write_file` / `read_file`).
- Never retry the same terminal command that returned `Class not registered`.

**`execute_code` timeout (300s):**
- If a script times out, do not auto-retry the full script.
- Break the script into smaller steps and execute sequentially.
- Maximum script complexity per `execute_code` call: 5 logical operations.

**`ripgrep` / `rg` not installed:**
- `search_files` tool requires `ripgrep`. It is not installed on this machine.
- Use `read_file` + manual pattern matching, or Python `glob`/`os.walk` via `execute_code` instead.
- Install recommendation: https://github.com/BurntSushi/ripgrep#installation

---

## Godmode Skill — SUSPENDED

```
red-teaming\godmode — SUSPENDED
```

- This skill has triggered prompt injection security warnings twice in recent sessions.
- Do not load, invoke, chain, or reference this skill under any circumstances.
- If it appears in a skill chain or tool call, stop execution and alert user immediately:
  > ⚠️ **Security alert:** `godmode` skill was invoked. Execution stopped. Please review your active skill list.
- User must explicitly re-enable with full awareness of prompt injection risk before it can be used.

---

## NousResearch Hermes Auth

```
Nous inference auth: using NAS invoke JWT
```

- NAS JWT auth is configured and working.
- If JWT expires mid-session, report immediately and do not silently fall back to a different provider.

---

`hermes-system-config v1.0 — Hermes Agent`
`Author: Mary Bay Flanagan`
`Last Updated: June 2026`
`Status: Active — CRITICAL PRIORITY`
