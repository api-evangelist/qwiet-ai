---
name: run-security-scan
description: Run code analysis via sl analyze or look up package CVEs (Intelligent SCA) via Qwiet. Invoke when the user runs /run-security-scan or asks to scan code or run sl analyze. Do NOT use for triage or autofix — those require /triage-vuln and /autofix-vuln.
argument-hint: "[app] [branch]"
---

# Run security scan

**Code analysis** and **Intelligent SCA** via the **slmcp** MCP server (Qwiet API + Qwiet CLI). If MCP tools are missing, run `/setup-harness-code-security-mcp` first or see [mcp-setup.md](references/mcp-setup.md).

## Scope (this skill only)

Run **`sl_ensure_cli`** + **`sl_analyze`** (or package CVE lookup). You may call **`sl_list_findings`** once to give a **short scan summary** (`counts`, `top_actionable` titles).

**Stop here.** Do **not** deep-triage findings, fetch dataflows, read source for exploitability, or request AutoFix in this skill. MCP tools stay available, but the workflows for those steps live in other skills — see **Next step** below.

## Prerequisites

- `sl auth` → `~/.shiftleft/config.json`
- **slmcp** MCP server (install with `/setup-harness-code-security-mcp`; also included in the Harness SAST and SCA Cursor/Claude plugins)

## Code analysis (`sl analyze`)

1. **`sl_ensure_cli`** — install or update the Qwiet CLI under `~/.shiftleft` (separate tool call so install errors return quickly with stderr traces).
2. **`sl_analyze`** — run `sl analyze --wait` in the workspace. Do **not** call `sl_list_applications` first — `sl` creates the app if needed.

Analysis runs on the local workstation; findings upload to Qwiet AI by Harness.

| Input    | Default when omitted            |
| -------- | ------------------------------- |
| `app`    | Workspace folder name           |
| `branch` | Current git branch, else `main` |

Example: `sl_ensure_cli` with `{}`, then `sl_analyze` with `{}` or `{ "app": "my-service", "branch": "feature/foo" }`.

### Scan id for triage (important)

`sl_analyze` returns **JSON** (not raw `sl` stdout). Use **`scan_id`** from that response for `sl_list_findings` and dataflow tools.

- **`scan_id`** — per-language API scan id (e.g. `"5"`) — **use this for findings**
- **`polyglot_scan_id`** — compound scan id sometimes printed in human `sl` output (e.g. `"6"`) — **do not** pass to `sl_list_findings`

If `scan_id` is missing, call **`sl_list_branch_scans`** and use **`latest.id`**.

When the user invokes this skill for a normal repo scan, run the workflow above — do not ask SAST vs CVE vs list-scans unless they asked for something else.

## Next step (required handoff)

After summarizing the scan, tell the user to run **`/triage-vuln`** — or invoke that skill yourself if the user said "triage" in a **new** message; do not continue triage on this turn. Some hosts prefix skills, for example `/harness-sast-and-sca:triage-vuln`.

Pass along **`app`** and **`scan_id`** from `sl_analyze`. Triage uses **`top_actionable[].id`** — not ids copied from a prose summary.

## Intelligent SCA / package CVEs

1. Build **PURLs** (e.g. `pkg:npm/lodash@4.17.21`)
2. **`sl_lookup_package_cves`** with `{ "purls": ["..."] }`
3. Summarize severities and fixed versions

## Notes

- **Harness SAST and SCA** extension / **Secure AI Coding** (on-save IDE analysis) is a different workflow — no `scan_id` from this MCP path
- **`sl analyze`** uses the project workspace as cwd when the MCP server starts
