---
name: setup-harness-code-security-mcp
description: Install and verify the harness-code-security-mcp npm MCP server. Use when sl_* MCP tools are missing, setup is requested, or Qwiet auth needs diagnosis.
argument-hint: "[--install] [--package harness-code-security-mcp] [--version latest]"
---

# Set up Harness Code Security MCP

Install and connect the **`harness-code-security-mcp`** npm package (`sl_whoami`, `sl_analyze`, triage, AutoFix).

## Doctor

```bash
node scripts/doctor.mjs
```

Install the npm launcher if missing:

```bash
node scripts/doctor.mjs --install
```

## Prerequisites

1. Node.js 20+
2. `sl auth` → `~/.shiftleft/config.json` with `orgId` and `accessToken`
3. MCP server **`harness-code-security-mcp`** configured in the agent

## Claude Code

Terminal or Bash inside a session:

```bash
claude mcp add harness-code-security-mcp -- npx -y harness-code-security-mcp
```

If `--` parsing fails, use JSON:

```bash
claude mcp add-json harness-code-security-mcp '{"type":"stdio","command":"npx","args":["-y","harness-code-security-mcp"]}'
```

Run `/mcp` to verify. Tools are prefixed: `mcp__harness-code-security-mcp__sl_whoami`.

**Windows:** `claude mcp add harness-code-security-mcp -- cmd /c npx -y harness-code-security-mcp`

## Cursor and other agents

```json
{
  "mcpServers": {
    "harness-code-security-mcp": {
      "command": "harness-code-security-mcp"
    }
  }
}
```

Use `npx -y harness-code-security-mcp` instead of a global install if preferred. No `env` block required (defaults: `app.shiftleft.io`, `~/.shiftleft`).

## Verify

Reconnect MCP or start a new session, then call `sl_whoami`. Success → `activeOrganizationId` matches config `orgId`.
