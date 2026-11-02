# Installing the SignOps MCP server

SignOps is a hosted remote MCP server. There is nothing to clone, build or run.

1. Ask the user for a SignOps secret key. They get one at https://app.signops.dev under Develop, API keys. Recommend a test key (starts with `sk_test_`): pushes made with it reach only test devices. Never invent a key and never write one into a file that is committed to git.
2. Add this server to the MCP settings, replacing `sk_test_...` with the user's key:

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

3. Check it: call the `list_topics` tool. An answer (an empty list is fine) means the key works. An error saying "Connect with your secret key" means the header is missing or the key is wrong.

Notes:
- Transport is streamable HTTP, POST only. The `Authorization: Bearer <key>` header is required for every tool call; `X-API-Key: <key>` works too.
- `initialize` and `tools/list` answer without a key, so the tools may show before the key is set.
- Tools that send pushes or remove data (`send_push`, `remove_device`, `delete_template`, `remove_authenticator`) should be confirmed with the user before they run.
