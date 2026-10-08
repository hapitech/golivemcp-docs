<p align="center"><img src="logo.png" width="96" alt="GoLive MCP logo"></p>

# GoLive MCP

**Your agents build. We help you go live.**

GoLive MCP is a remote MCP server for the last mile of shipping software. A coding agent can
write the app; GoLive gives it the rest as tools: check and buy a domain inside a budget, manage
DNS, set up email on the domain, deploy from a GitHub repo, set environment variables and rotate
secrets without ever reading them back, create buckets and volumes, route a hostname to the app and
check its certificate, and run security, privacy and regulation, and patent audits before launch.

- **Endpoint (Streamable HTTP):** `https://golivemcp.com/mcp`
- **Website:** https://golivemcp.com
- **Tool registry (JSON):** https://golivemcp.com/tools.json
- **Official MCP Registry:** `com.golivemcp/golive`
- **Auth:** OAuth 2.1 (dynamic client registration, PKCE) or an API key in `x-api-key`
  / `Authorization: Bearer`. Invite-only beta.

This repository holds documentation only. The server is hosted; there is nothing to install.

## Connect

Claude Code:

```bash
claude mcp add --transport http golive https://golivemcp.com/mcp
```

Claude.ai or Claude Desktop: Settings, Connectors, Add custom connector, URL
`https://golivemcp.com/mcp`, then approve with your GoLive key.

Any MCP client:

```json
{
  "mcpServers": {
    "golive": {
      "type": "http",
      "url": "https://golivemcp.com/mcp",
      "headers": { "x-api-key": "glk_..." }
    }
  }
}
```

Clients that only speak stdio can bridge with `npx mcp-remote https://golivemcp.com/mcp`.
See [llms-install.md](llms-install.md).

## Safety model

- **Dry run first.** Every tool that changes something accepts `dry_run: true` and returns the plan.
- **Confirm the target.** Destructive and spending tools need `confirm` set to the exact target.
- **Budgets default to zero.** Buying or provisioning needs an operator-opened budget with an
  amount, a per-action cap and an expiry. Agents cannot open or widen one.
- **Secrets go in, never out.** Tools answer with names, lengths and the last 4 characters.
- **No keys handed over.** Registrar, cloud and DNS credentials stay inside the platform.
- **Every attempt is audited,** refusals included (`audit_log`).

## Example requests

- "Is launchpad.dev available, and what would it cost? Plan the purchase but don't buy it."
- "Deploy acme/launchpad from main and tell me when it is live."
- "Set STRIPE_SECRET_KEY on launchpad and redeploy."
- "Route launchpad.dev to the launchpad app and check its certificate."
- "Run a security audit and a privacy and regulation audit on https://github.com/acme/launchpad."

## Tools

Some tools are marked `soon` on https://golivemcp.com while their platform routes ship; they
answer `capability_pending` until then.

| Group | Tool | What it does |
|---|---|---|
| Domains | `check_domain` | Is a domain available, and what does it cost to register and renew? Free, no side effects. |
| Domains | `buy_domain` | Registers a domain and creates its Cloudflare zone. Spends money: needs an open golive domains budget AND an open fleet budget, a reason, and confirm equal to the domain. Returns the domain and zone id, never a credential. |
| Domains | `list_domains` | Domains (Cloudflare zones) golive can manage, with their status. |
| Domains | `domain_status` | Whether a domain's Cloudflare zone is active, and the nameservers it should point at. |
| DNS | `list_records` | DNS records of a zone on our Cloudflare account, optionally filtered by name or type. |
| DNS | `upsert_record` | Creates a record, or updates the one with the same name and type. Zones on our Cloudflare account only. |
| DNS | `delete_record` | Deletes one DNS record by id. Destructive: confirm with the record id. |
| Email | `setup_domain_email` | Adds the domain to AI Mail MCP and applies its MX, SPF, DKIM and DMARC records (zones on our Cloudflare account), so the domain can send and receive. |
| Email | `create_mailbox` | Creates a mailbox (e.g. hello@example.com) on a domain already set up for email. |
| Deploy | `create_app` | Creates a new service that deploys from a GitHub repository branch, and registers it. Provisions a billable resource: needs an open resources budget. |
| Deploy | `deploy` | Deploys the latest commit of the app's branch (or a given commit). |
| Deploy | `deploy_status` | The current and recent deployments of an app: status, commit, and when. |
| Deploy | `logs` | Recent runtime or build log lines of an app, with secrets scrubbed by imagia. |
| Deploy | `rollback` | Redeploys the previous successful deployment (or a given one). Destructive: replaces what is live; confirm with the app id. |
| Deploy | `restart` | Restarts the running deployment without rebuilding. |
| Config and secrets | `set_env` | Sets one variable on an app and redeploys it. The value goes in and is never echoed: the answer carries its length and last 4 characters only. Confirm with the variable name. |
| Config and secrets | `list_env` | Variable NAMES on an app, each with its length and last 4 characters. Never values. |
| Config and secrets | `rotate_secret` | Generates a new random value for a secret, writes it to every service that declares it, redeploys them and verifies. The value is generated server side and never returned. Confirm with the variable name. |
| Storage | `create_bucket` | Creates an S3-compatible object storage bucket for an app. Provisions a billable resource: needs an open resources budget. |
| Storage | `bucket_credentials` | Issues credentials scoped to one bucket and writes them straight into the app's environment. Returns the variable names only; the credentials never pass through the agent. |
| Storage | `create_volume` | Creates a persistent disk and mounts it on the app. Provisions a billable resource: needs an open resources budget. |
| Routing | `attach_domain` | Routes a hostname on one of our zones to an app: a proxied DNS record plus the edge route to the app's generated hostname. No registrar or DNS change outside our account. |
| Routing | `ssl_status` | Whether a hostname serves valid HTTPS right now, plus its zone status. |
| Audits | `security_audit` | Starts a security audit (dependencies, secrets in code, auth, injection, headers) of an imagia project or a git repository. Returns a report id for get_report. |
| Audits | `compliance_audit` | Starts a compliance audit: privacy policy and terms, data handling, cookies and consent, and applicable regulation (GDPR, CCPA and similar). Returns a report id for get_report. |
| Audits | `patent_audit` | Starts a patent risk review of what the app does against published patents, and flags features worth a lawyer's look. Returns a report id for get_report. Not legal advice. |
| Audits | `get_report` | Progress and findings of an audit started with security_audit, compliance_audit or patent_audit; or, given project_id without report_id, the recent reports for that project. |
| Brand | `generate_logo_options` | Designs 1 to 6 distinct logo marks (default 4) for a brand. Each option has an id, a one-line concept note, a clean square SVG (validated: no scripts, fonts or external references) and a 256 px PNG preview as base64. Costs a model call; dry_run shows the price. Pass an id to export_logo_assets for the favicon and app-icon set. |
| Brand | `export_logo_assets` | Turns one logo option into a full icon set: favicon.svg, favicon.ico (16, 32, 48), PNGs from 16 to 512, apple-touch-icon (180), icon-192 and icon-512, a web manifest, and the <head> tags. Give option_id from generate_logo_options (kept 24 h) or the svg itself. Files come back base64. Free, no side effects. |
| GitHub | `connect_github` | Returns a one-time link (30 minutes) that installs the GoLive GitHub App, or adds repos to it, and connects the installation to this account. Open it in a browser signed in to GitHub; GitHub asks you to authorize GoLive so it can confirm you can see the installation. Also lists the installations already connected. GoLive gets read access to code and opens PRs; it never pushes. |
| GitHub | `list_repos` | Repositories this account's GitHub installations cover, refreshed from GitHub, with the GoLive app and deploy branch each one is linked to. Free. |
| GitHub | `link_repo` | Binds a repository and its deploy branch to a GoLive app, so deploys and preflight checks use that repo. The repo must be covered by this account's GitHub installation and the branch must exist. |
| GitHub | `read_repo_file` | One text file from a connected repository at a branch, tag or commit (default: the linked deploy branch, else the default branch), with the commit SHA it was read at. Up to 256 KB; binary files are refused. Free. |
| Governance | `budget_status` | What golive may still spend or provision: its own open budgets and the fleet domain budget behind them. Budgets are opened by an operator, never by a tool. |
| Governance | `get_usage` | What this key has used and been charged: calls, charges and pending holds by tool or by day, plus the account's spend limits and what remains. Refusals and dry runs are listed at $0. Free. |
| Governance | `audit_log` | Every mutating attempt through golive, newest first: who, what, dry run or real, outcome (including refusals) and cost. Secret values are never stored. |

## Links

- Privacy: https://golivemcp.com/privacy
- Server card: https://golivemcp.com/.well-known/mcp/server-card.json
- llms.txt: https://golivemcp.com/llms.txt

GoLive MCP is a Hapi product.
