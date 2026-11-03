# PingPong AI Marketplace

Agent Skills for MCP or the PingPong CLI.

## Plugins

| Plugin | Connectors | Type | Description |
| --- | --- | --- | --- |
| [pingpong-cli](./plugins/pingpong-cli) | MCP + CLI | Query / Reference | Three read-only Skills for account balances, balance movements, and transaction history. The plugin includes an MCP connection; the CLI is installed separately. |

### pingpong-cli — Billing Skills

All three tools belong to the business scope `reconciliation.read`.

| Skill | MCP tool | Scope | Description |
| --- | --- | --- | --- |
| [list-account-balances](./plugins/pingpong-cli/skills/list-account-balances/SKILL.md) | `list_account_balances` | `reconciliation.read` | Query current account balances, with optional currency and funding-account filters. |
| [list-balance-movements](./plugins/pingpong-cli/skills/list-balance-movements/SKILL.md) | `list_balance_movements` | `reconciliation.read` | Trace balance movements for an order or a defined time range. |
| [list-transaction-history](./plugins/pingpong-cli/skills/list-transaction-history/SKILL.md) | `list_transaction_history` | `reconciliation.read` | Query transactions by business source, order, or time range. |

Each Skill includes input constraints, execution steps, result interpretation, and troubleshooting. Business parameters use snake_case, such as `page_no`, `page_size`, and `order_id`.

Both history tools require an order ID or a complete time range of at most 365 days. Transaction history also requires `source`. The Skills preserve requested pagination and distinguish partial results from complete account data.

## Connectors

- **MCP:** The plugin packages a remote MCP connection. Enable it in your agent and complete that client's OAuth flow.
- **CLI:** Terminal-capable agents can use an installed and authenticated [PingPong CLI](https://github.com/pingpongx-labs/pingpong-cli) when you choose CLI execution. CLI installation and credentials are separate from the plugin.

Skills use the configured MCP connection by default and include CLI equivalents. They do not silently switch connectors or environments. Installing a Skill alone does not authenticate an account or grant tool permissions.

## Installation

This directory is prepared for local installation. A public marketplace Git URL and production MCP endpoint have not yet been confirmed for release.

Before installing, replace the placeholder URL in [the MCP configuration](./plugins/pingpong-cli/.mcp.json) with your confirmed MCP endpoint. The bundled `https://configure-before-use.invalid/mcp` address cannot connect. See [Setup](SETUP.md) for release preparation.

### Claude Code

From this marketplace directory:

```sh
claude plugin marketplace add "$PWD"
claude plugin install pingpong-cli@pingpong-marketplace
```

Enable the connection and complete OAuth. Skills use the plugin namespace; for example:

```text
/pingpong-cli:list-account-balances
```

### Codex

From this marketplace directory:

```sh
codex plugin marketplace add "$PWD"
codex plugin add pingpong-cli@pingpong-marketplace
```

Complete OAuth if prompted, start a new session, and check `/mcp` for connection status and available tools.

After the marketplace is published, you can replace the local directory with its actual Git URL:

```sh
codex plugin marketplace add "${PINGPONG_MARKETPLACE_GIT_URL:?Set the published marketplace Git URL}"
codex plugin add pingpong-cli@pingpong-marketplace
```

The CLI repository is a separate project; use it as a marketplace source only if it actually contains the marketplace manifests.

### Cursor

Cursor marketplace and plugin manifests are included. Once the repository is published, import its actual URL through **Settings → Plugins → Team Marketplaces**, where available, and install `pingpong-cli`. No public Cursor marketplace listing is currently claimed.

### Grok

A Grok marketplace manifest points to the same plugin directory. After publication, use the actual repository URL:

```sh
grok plugin marketplace add "${PINGPONG_MARKETPLACE_GIT_URL:?Set the published marketplace Git URL}"
grok plugin install pingpong-cli --trust
```

Check your installed client's `grok plugin --help` and review the plugin source before granting trust. This installation path has not been tested end to end here.

### Other agents and Skills-only installation

For agents that support local Skill imports, use the three folders under `plugins/pingpong-cli/skills/` and configure an authenticated connector separately. No one-command Skills-only installer has been validated for this repository.

See [Client installation and verification](CLIENTS.md) for platform-specific setup and acceptance checks.

### OAuth compatibility

The current embedded authorization server accepts HTTP localhost/127.0.0.1 callbacks. Clients requiring HTTPS callbacks need server-side compatibility changes. This affects hosted connectors and may also affect Cursor's default registration flow. Browser CORS settings do not change the callback policy.

Local Codex marketplace installation has been checked. Claude Code, Cursor, and Grok still require end-to-end installation, OAuth, and query verification.

## Usage

After connecting and authorizing the required tools, ask your agent:

```text
Use list_account_balances to query page_no=1 and page_size=1.
```

```text
Use list_balance_movements to trace order YOUR_ORDER_ID,
with page_no=1 and page_size=20.
```

```text
Use list_transaction_history for source GENERAL_ACCOUNT and order YOUR_ORDER_ID,
with page_no=1 and page_size=20.
```

Replace sample identifiers and sources with the actual values for your request. For CLI execution, after installing and authenticating the CLI in your chosen environment:

```sh
pingpong --env demo balances list --page_no 1 --page_size 1
```

Replace `demo` with your target environment.

## Updating and removing

For a published Git marketplace, refresh the source and follow the client's plugin-update prompts:

```sh
codex plugin marketplace upgrade pingpong-marketplace
```

Start a new session after updating. To remove the plugin and marketplace:

```sh
codex plugin remove pingpong-cli@pingpong-marketplace
codex plugin marketplace remove pingpong-marketplace
```

Removing a local plugin does not revoke server-side authorization.

## Data handling and support

These Skills perform read-only queries. They do not create invoices, subscriptions, payments, payouts, or FX conversions. Keep account credentials and financial records out of public reports.

For troubleshooting, provide the affected client, tool name, sanitized error, and diagnostic ID where available. A missing tool, a denied tool grant, and downstream token acquisition failure are different problems; reinstalling the plugin does not resolve all three.

## License

Apache-2.0. See [LICENSE](LICENSE).
