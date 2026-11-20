# Setup

This is the root of a standalone multi-client marketplace. No remote repository has been published by this task.

<a id="environments"></a>
## Environments

Choose the target environment before installation. Set the full MCP endpoint in `plugins/pingpong-cli/.mcp.json`; the domain root alone is not the MCP path.

| Environment | CLI profile | MCP endpoint |
| --- | --- | --- |
| Sandbox | `demo` | Use the sandbox endpoint provided by your administrator |
| Production | `production` | `https://mcp.pingpongx.com/mcp` |

The production service domain is `https://mcp.pingpongx.com/`; the MCP transport path is `/mcp`. No sandbox domain is assumed. Internal test is a separate environment and must not be substituted for sandbox.

The bundled configuration points to production at `https://mcp.pingpongx.com/mcp`. For sandbox use, replace only the `url` value with the sandbox endpoint provided by your administrator before installation. Do not add credentials or token headers to a public package.

The configured URL determines the environment; the plugin name does not switch it. Authorize each environment separately. A release for a specific environment should clearly identify that environment and bundle its corresponding URL.

For CLI usage, configure `PLATFORM_MCP_DEMO_URL` or `PLATFORM_MCP_PRODUCTION_URL` and select `--env demo` or `--env production`. The CLI's default profile is `demo`; profile defaults do not configure a plugin's MCP URL. See the [CLI guide](docs/cli.md) for installation, authentication, and troubleshooting.

## Install

After configuring the endpoint, run from this directory:

```sh
codex plugin marketplace add "$PWD"
codex plugin add pingpong-cli@pingpong-marketplace
```

Complete OAuth and start a new session. Check `/mcp`, discover the three tools, and then run an authorized query. Installing the package does not prove remote authentication or business success.

## Authorization

Tool discovery does not establish permission to call every tool. The token's scopes and the account's selected tool grants must satisfy server policy.

## Prepare publication

- Copy this entire directory, including hidden `.agents`, `.codex-plugin`, and `.mcp.json` files, to the target repository root.
- Confirm the endpoint, plugin version, applicable license, and repository URL before publishing.
- Verify installation and actual OAuth/tool calls with the target client.
- Keep internal test packages, account credentials, logs, and user configuration out of the public repository.

Skills are maintained in platform-mcp under `plugins/skills/reconciliation.read/<tool>/`. The packager maps source directories to tool-based Skill names. When a tool contract changes, update these Skill copies together with the service and client documentation.

For other clients, see [Client installation](CLIENTS.md). Copy all four root marketplace directories and all three plugin manifests; Grok uses the shared plugin layout.

## Repository layout

```text
.agents/plugins/marketplace.json
.claude-plugin/marketplace.json
.cursor-plugin/marketplace.json
.grok-plugin/marketplace.json
plugins/pingpong-cli/
  .codex-plugin/plugin.json
  .claude-plugin/plugin.json
  .cursor-plugin/plugin.json
  .mcp.json
  skills/
    list-account-balances/SKILL.md
    list-balance-movements/SKILL.md
    list-transaction-history/SKILL.md
README.md
README.zh-CN.md
SETUP.md
CLIENTS.md
docs/cli.md
docs/cli.zh-CN.md
```
