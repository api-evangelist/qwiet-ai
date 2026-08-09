# Harness Code Security MCP Setup

Skills in this package call **Harness Code Security MCP** tools (`sl_whoami`, `sl_list_applications`, …). For standalone skills, run **`/setup-harness-code-security-mcp`** first. The same MCP server also ships inside the Harness SAST and SCA Cursor/Claude plugins.

## Credentials

- Run `sl auth` so `~/.shiftleft/config.json` has `orgId` and `accessToken`
- First-time CLI install: call **`sl_ensure_cli`** before **`sl_analyze`** (can take 1–3 minutes to download). Watch MCP server stderr for `[sl]` progress lines.
- If the host reports a tool timeout but `~/.shiftleft/sl` appears afterward, retry the scan — the install may have finished after the client gave up.

### Active organization (required)

**Org scope:** API tools use **`orgId` from `config.json` only** (via `SL_HOME`, default `~/.shiftleft`). **`sl_analyze` does not need `sl_list_applications`** — pass an app id (or omit for workspace folder name); `sl` creates the app on analyze.

- Do **not** pick an org from `sl_whoami` → `me.membership` for API calls.
- `me.membership` is informational (orgs your user can access). To work in another org, re-run `sl auth` with that org's id + token, then restart or reconnect MCP.
- `sl_whoami` returns **`activeOrganizationId`** — that is the org in use; it must match `orgId` in config.

## Standalone Skills (Recommended)

Install the skills:

```bash
npx skills add ShiftLeftSecurity/skills -g --skill '*'
```

Then run:

```text
/setup-harness-code-security-mcp
```

The setup skill checks Node.js, npm, `harness-code-security-mcp`, Qwiet auth, and MCP config. It can install the npm launcher with:

```bash
node scripts/doctor.mjs --install
```

### Claude Code (npm package)

In the terminal, or via Bash from inside a Claude Code session:

```bash
claude mcp add harness-code-security-mcp -- npx -y harness-code-security-mcp
```

If an agent struggles with the `--` separator, use JSON:

```bash
claude mcp add-json harness-code-security-mcp '{"type":"stdio","command":"npx","args":["-y","harness-code-security-mcp"]}'
```

After adding, run `/mcp` to verify connection. Tools are prefixed: `mcp__harness-code-security-mcp__sl_whoami`.

### Cursor and other agents

```json
{
  "mcpServers": {
    "harness-code-security-mcp": {
      "command": "harness-code-security-mcp"
    }
  }
}
```

Requires `npm install -g harness-code-security-mcp` (or use `npx` in `command`/`args`).

Defaults: API host `app.shiftleft.io`, config dir `~/.shiftleft`. Override with `QWIET_API_HOST`, `SL_HOME`, or `WORKSPACE_FOLDER` only when needed.

The launcher checks for MCP updates at startup at most once per day. Add `--no-auto-update` to disable update checks, or `--version-pin x.y.z` for managed environments.

## Cursor CLI Plugin

```bash
agent --plugin-dir /path/to/curness/release/curness-cursor-plugin
```

The plugin includes **`mcp.json`** that starts bundled **`slmcp/slmcp.cjs`** (no `node_modules`). Cloud skills ship in **`skills/`**.

## Claude Code plugin

```bash
claude --plugin-dir /path/to/curness/release/curness-claude-plugin
```

Same bundled **`slmcp/slmcp.cjs`**.

## Cursor / local development (MCP only)

From the **curness** repo after `cd packages/slmcp && npm install && npm run build`, use the launcher:

```json
{
  "mcpServers": {
    "harness-code-security-mcp": {
      "command": "node",
      "args": ["/absolute/path/to/curness/packages/slmcp/dist/slmcp-launcher.cjs"]
    }
  }
}
```

## Tool names in chat

Hosts prefix MCP tools differently. In skills we use **logical names** (`sl_whoami`). In your session, search `/mcp` or tool list for names containing `sl_whoami`.

## Triage-friendly responses (default)

| Tool                       | Default shape                                                                                                             |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `sl_list_findings`         | `counts`, `top_actionable` (critical/high ids), `next_steps` — not full finding blobs                                     |
| `sl_get_findings`          | `{ findings: [...], errors: [...] }` for up to 10 ids in one call                                                         |
| `sl_get_recommended_fixes` | Batch envelope; each fix includes `edits[]` (`file`, `old_string`, `new_string`) when `WORKSPACE_FOLDER` matches the repo |

**Finding ids:** Use values from `top_actionable[].id`. Do not pass ordinal labels (`"1"`, `"2"`) unless that id appeared in list output.

**Avoid `full: true`** on list/get tools unless debugging — payloads can exceed agent context (100KB+).
