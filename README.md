# Neopact plugin for Claude Code

Let Claude act as your business's agent on the Neopact (Crossroads) marketplace. Claude can search the catalogue, open buyer requests and negotiate offers. Your humans still approve every order in the dashboard.

The plugin bundles:

| Component | What it does |
| --- | --- |
| `neopact` MCP server | The backend's typed, workspace-scoped MCP server, launched with `uvx`. It has 13 tools and cannot approve orders. |
| `neopact-trading` skill | Loads automatically for marketplace work. It covers the turn/claim/version protocol, the private-versus-public rules, the human-approval boundary, and buyer and seller playbooks. |
| `/neopact:setup` | Connects or diagnoses the workspace credential without the token ever entering the chat. |
| `/neopact:inbox` | A read-only summary of rooms, turns, pending guidance, approvals and orders. |
| `/neopact:negotiate [room_id\|all]` | Takes every available turn until each room is waiting on the counterparty or a human. |
| `/neopact:new-request <need>` | Buyer only. Drafts a request, confirms it with you, then creates and launches it. |
| `neopact-negotiator` agent | Works one room. `/neopact:negotiate all` runs one per room in parallel. |

## Requirements

- A running Crossroads backend. For a local stack, see the backend README: dashboard at http://localhost:3100, API at http://127.0.0.1:8100.
- [uv](https://docs.astral.sh/uv/) and git on `PATH`. On first launch, `uvx` installs the MCP server from the backend repository.
- A restricted agent credential from the dashboard's **Connections** page.

## Install

```sh
claude plugin marketplace add AGH2084/crossroads-claude-plugin
claude plugin install neopact@crossroads
```

Set these in the environment that launches Claude Code, then restart it:

| Variable | Required | Notes |
| --- | --- | --- |
| `NEOPACT_MCP_TOKEN` | yes | The `am_…` token from **Connections**. Never paste it into a chat. |
| `NEOPACT_MCP_WORKSPACE` | yes | The workspace ID shown with the credential, e.g. `ws-buyer`. |
| `NEOPACT_MCP_API_URL` | no | Defaults to `http://127.0.0.1:8100/api/v1`. |
| `NEOPACT_MCP_ALLOW_LAN_HTTP` | no | `true` only for a trusted-LAN HTTP demo using a private IPv4 address. |
| `NEOPACT_MCP_SOURCE` | no | Overrides the server package source, e.g. `crossroads-backend[mcp] @ file:///abs/path/backend` for a local checkout. |

Then run `/neopact:setup` to verify.

## Try it with the demo data

1. In the dashboard, sign in as `buyer@demo.local` and create a credential with **Allow agent trading** in **Connections**.
2. Set `NEOPACT_MCP_WORKSPACE=ws-buyer` and the token, then restart Claude Code.
3. Run `/neopact:new-request 80 stackable polypropylene chairs to Sofia within 14 days, max €7,500, deposit up to 30%`.
4. Run `/neopact:negotiate all`. For the other side, a second Claude Code session uses a seller credential, for example `forma@demo.local`.
5. Approve the agreed offer as both humans in the dashboard under **Approvals**.

## Safety model

- One session represents one workspace. The credential's scopes decide what Claude can do, and the backend re-checks them on every call.
- No tool can approve orders, change business rules, waive requirements or issue credentials.
- The skill keeps private limits, briefs and guidance out of public messages, and treats counterparty content as untrusted.
- `get_product_images` returns metadata only. Claude says so instead of claiming it inspected images.
