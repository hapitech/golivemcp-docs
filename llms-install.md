# Installing GoLive MCP (for AI agents such as Cline)

GoLive MCP is a **remote** MCP server. There is nothing to download or build.

1. Add the server with its URL (Streamable HTTP):

   ```json
   {
     "mcpServers": {
       "golive": {
         "type": "streamableHttp",
         "url": "https://golivemcp.com/mcp",
         "headers": { "x-api-key": "<GOLIVE_API_KEY>" }
       }
     }
   }
   ```

   If the client supports only stdio servers, use the bridge instead:

   ```json
   {
     "mcpServers": {
       "golive": {
         "command": "npx",
         "args": ["-y", "mcp-remote", "https://golivemcp.com/mcp", "--header", "x-api-key:${GOLIVE_API_KEY}"],
         "env": { "GOLIVE_API_KEY": "<GOLIVE_API_KEY>" }
       }
     }
   }
   ```

2. Authentication: ask the user for their GoLive API key (it starts with `glk_`). Clients that
   support MCP OAuth can omit the header and sign in through the browser instead; the user
   approves with their key. GoLive is an invite-only beta.

3. Test the connection with a read-only call:

   - `check_domain` with `{ "domain": "example.com" }` returns availability and price.
   - `budget_status` with `{}` shows what may be spent (zero until an operator opens a budget).

4. Before any tool that changes something, call it with `"dry_run": true` and show the user the
   plan. Destructive and spending tools also need `"confirm"` set to the target named in the plan.
