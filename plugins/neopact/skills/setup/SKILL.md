---
name: setup
description: Connect Claude to a Neopact workspace or diagnose a broken connection — checks uv, the credential environment and the MCP server, without ever handling the token itself.
disable-model-invocation: true
allowed-tools: Bash(uvx --version), Bash(uv --version), mcp__plugin_neopact_neopact__get_instructions
---

# Neopact setup and diagnosis

Get the user to a working `neopact` MCP connection. **Never ask the user to paste their token into this conversation, and never print it.** The user sets credentials outside the chat.

## 1. Try the connection

Call `mcp__plugin_neopact_neopact__get_instructions`.
- If it succeeds, report the workspace, role and scopes. Suggest `/neopact:inbox` and stop.
- If the tool doesn't exist or errors, continue with the checks below.

## 2. Check prerequisites

Run `uvx --version`. If it's missing, tell the user to install uv (https://docs.astral.sh/uv/getting-started/installation/):
- Windows (PowerShell): `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"`
- macOS/Linux: `curl -LsSf https://astral.sh/uv/install.sh | sh`

Git must also be installed, because the server is fetched from GitHub on first launch.

Check the variables' presence only, never their values:
- Windows (PowerShell): `[bool]$env:NEOPACT_MCP_TOKEN; [bool]$env:NEOPACT_MCP_WORKSPACE; $env:NEOPACT_MCP_API_URL`
- Bash: `[ -n "$NEOPACT_MCP_TOKEN" ] && echo token-set; [ -n "$NEOPACT_MCP_WORKSPACE" ] && echo workspace-set; echo "${NEOPACT_MCP_API_URL:-default}"`

## 3. Explain what is needed

| Variable | Value |
| --- | --- |
| `NEOPACT_MCP_TOKEN` | Restricted `am_…` agent token from the dashboard's **Connections** page. Required. |
| `NEOPACT_MCP_WORKSPACE` | The workspace ID shown with that credential, e.g. `ws-buyer`. Required. |
| `NEOPACT_MCP_API_URL` | Backend API root. Defaults to `http://127.0.0.1:8100/api/v1`. |
| `NEOPACT_MCP_ALLOW_LAN_HTTP` | `true` only for a trusted-LAN demo against a private IPv4 address over HTTP. |
| `NEOPACT_MCP_SOURCE` | Optional. Overrides where the server is installed from, e.g. `crossroads-backend[mcp] @ file:///C:/path/to/backend`. |

Steps for the user:
1. Open the dashboard (locally http://localhost:3100) and go to **Connections**.
2. Create a credential. Read-only is the default. Tick **Allow agent trading** only if Claude should negotiate or create requests and products.
3. Set the variables in the environment that launches Claude Code, then **fully restart Claude Code**. Options:
   - **Windows:** run `setx NEOPACT_MCP_WORKSPACE ws-buyer`, and `setx NEOPACT_MCP_TOKEN <token>` in their own terminal (not here). Then open a new terminal.
   - **macOS/Linux:** add `export` lines to a private shell profile, or load them from a secret store.
   - **Claude Code settings:** add an `"env"` block to `~/.claude/settings.json`. This is simple, but it stores the token in plain text on disk.

Credentials expire after seven days and can be revoked from **Connections**. Use separate credentials for a buyer and a seller agent. One Claude Code session represents one workspace.

## 4. Verify

After the restart, the user runs `/mcp` to confirm `plugin:neopact:neopact` is connected, and then `/neopact:setup` again.

Common failures:
- Server exits at startup: a variable is missing or malformed. The token must start with `am_`, and the URL must end in `/api/v1`.
- HTTP refused: a non-loopback URL needs HTTPS, or the trusted-LAN opt-in above.
- 401: the credential is expired or revoked. Issue a new one.
