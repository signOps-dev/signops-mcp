# SignOps MCP server

<img src="logo.png" alt="SignOps" width="96" align="right">

Let a coding agent (Claude Code, Cursor, VS Code, Cline and any other MCP client) send push notifications to Android, iOS and the web, manage devices, topics and templates, and ask a user's trusted phone to approve a login, through [SignOps](https://signops.dev).

It is a hosted server. Nothing to install or run:

```
https://api.signops.dev/mcp
```

- Transport: streamable HTTP, POST only, JSON answers.
- Auth: your SignOps secret key in the `Authorization` header (`Bearer sk_test_...`), or the bare key in `X-API-Key` for clients that keep `Authorization` for themselves. There is no OAuth sign-in; the key is the login, and it decides the app and the mode.
- Without a key the server answers `initialize` and `tools/list`, so a client can show the tools. Calling a tool needs the key.
- Every tool is one call to the SignOps REST API made with your key, so it is logged, limited and billed exactly like the API.
- Listed in the official MCP Registry as `dev.signops/signops` ([server.json](server.json)).

This repository holds the setup instructions and client configs. The server itself runs on SignOps' API.

## 1. Get a key

1. Make a free account at [app.signops.dev](https://app.signops.dev) (no card).
2. In the dashboard, open **Develop**, then **API keys**, and copy a **test** key (`sk_test_...`).

Use a test key while building: pushes made with it reach only devices marked as test devices. Keep live keys (`sk_live_...`) out of agent configs you share or commit.

## 2. Connect your client

**Claude Code**

```sh
claude mcp add --transport http signops https://api.signops.dev/mcp \
  --header "Authorization: Bearer sk_test_..."
```

**Cursor**, in `~/.cursor/mcp.json` or `.cursor/mcp.json`

```json
{
  "mcpServers": {
    "signops": {
      "url": "https://api.signops.dev/mcp",
      "headers": { "Authorization": "Bearer sk_test_..." }
    }
  }
}
```

**VS Code**, in `.vscode/mcp.json`

```json
{
  "servers": {
    "signops": {
      "type": "http",
      "url": "https://api.signops.dev/mcp",
      "headers": { "Authorization": "Bearer sk_test_..." }
    }
  }
}
```

**Cline**, in `cline_mcp_settings.json` (Configure tab in the extension)

```json
{
  "mcpServers": {
    "signops": {
      "type": "streamableHttp",
      "url": "https://api.signops.dev/mcp",
      "headers": { "Authorization": "Bearer sk_test_..." },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

**Other clients:** any client that speaks streamable HTTP. Point it at `https://api.signops.dev/mcp` with the `Authorization` header, or `X-API-Key: sk_test_...`.

The same files are in [configs/](configs/).

## 3. Check it works

Ask the agent: "List my SignOps topics." It calls `list_topics`. An answer with an empty list means the key works and the app has no topics yet.

## Tools (23)

**Push**

| Tool | What it does |
|---|---|
| `send_push` | Send to one user, a list of up to 1,000, a topic, or everyone, now or scheduled up to 30 days ahead, optionally from a template. With a test key only test devices receive it. |
| `get_push` | One push by id: status, devices, sent and failed, per platform. |
| `list_pushes` | Pushes, newest first. |
| `cancel_push` | Cancel a scheduled push that has not started sending. |

**Devices and topics**

| Tool | What it does |
|---|---|
| `register_device` | Store a device's push token under your user id (Android, iOS or web). |
| `list_devices` | A user's devices, with platform and whether each is trusted for approvals. |
| `remove_device` | Remove one device; it gets no more pushes. |
| `list_topics` | Topics with how many users each has. |
| `subscribe_topic` | Add a user to a topic. |
| `unsubscribe_topic` | Take a user off a topic. |

**Templates**

| Tool | What it does |
|---|---|
| `list_templates` | Saved push templates with their fields. |
| `get_template` | One template by name. |
| `save_template` | Create or replace a template with `{{field}}` placeholders. |
| `delete_template` | Delete a template by name. |

**Login approval, passkeys and authenticator**

| Tool | What it does |
|---|---|
| `create_approval` | Send an approval request with number matching to the user's trusted phone. |
| `get_approval` | Status: pending, approved, denied, expired or cancelled. |
| `cancel_approval` | Cancel a pending approval request. |
| `list_passkeys` | A user's passkeys. |
| `get_authenticator` | Whether a user has an authenticator app confirmed, backup codes left, and whether they are locked. |
| `remove_authenticator` | Remove a user's authenticator app and backup codes, for a lost phone. |
| `unlock_user` | Clear the lock that five wrong codes put on a user. |

**User data**

| Tool | What it does |
|---|---|
| `export_user` | Everything SignOps holds on one user id, as JSON. |
| `get_consent` | A user's latest consent answer per purpose, and the history. |

Each tool carries MCP annotations (`readOnlyHint`, `destructiveHint`), so clients can ask before running the ones that change or delete data.

## Security

- The server is stateless: it keeps no session between requests, and each tool call is checked against your key like a direct API call.
- A test key cannot reach real users' devices. Start with one.
- Treat a key in an MCP config like any other secret: do not commit it. If one leaks, roll it under Develop, API keys: you get a new key and the old one stops working after 24 hours.
- To report a security problem, write to security@signops.dev ([security.txt](https://signops.dev/.well-known/security.txt)).

## Links

- Docs for the MCP server: [signops.dev/docs/#mcp](https://signops.dev/docs/#mcp)
- Full API docs: [signops.dev/docs](https://signops.dev/docs/)
- Pricing (free plan with 1,000 push devices): [signops.dev/pricing](https://signops.dev/pricing/)
- Libraries: [Node](https://github.com/signops-dev/signops-node), [web](https://github.com/signops-dev/signops-web), [Python](https://github.com/signops-dev/signops-python), [PHP](https://github.com/signops-dev/signops-php), [examples](https://github.com/signops-dev/signops-examples)

The instructions and configs in this repository are MIT licensed. The SignOps service has its own [terms](https://signops.dev/terms/).
