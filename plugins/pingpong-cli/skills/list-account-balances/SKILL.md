---
name: list-account-balances
description: "Query PingPong account balances by currency or funding account. Use for current balance lookups by currency or funding account."
---

Use this plugin's configured pingpong-cli MCP connection. If the endpoint is not a PingPong production or approved sandbox endpoint, ask the maintainer before querying. Do not silently change connectors or environments.


# Account balances

Read-only PingPong tool: list_account_balances. This skill does not create invoices, subscriptions, payments, refunds or accounting entries.

## Inputs

| Argument | Required | Constraint |
| --- | --- | --- |
| page_no | yes | Integer ≥ 1 |
| page_size | yes | Integer 1–100 |
| account_type | no | BALANCE_ACCOUNT only |
| currency_list | no | Array of three-uppercase-letter currency codes, e.g. ["USD","CNY"]; use actual known currencies |
| subaccount_id_list | no | Array of account IDs; each contains only letters/digits |
| dry_run | no | Boolean; request preview only |

## Workflow

1. Identify whether the request is for all visible balances, selected currencies, or specified funding accounts. Do not invent account IDs.
2. Query list_account_balances with the requested filters and pagination. Balances has no order, source, or date filter; do not send those fields.
3. Present balances by account and currency using only the fields actually returned. If only one page was requested, explicitly state that coverage.
4. For a historical balance or an explanation of a change, explain that this tool reads the current snapshot; use the movements tool for a supported period. Do not reconstruct a historical balance without a verified baseline and complete movement data.

## Example

Replace sample identifiers with the user's actual values. The example is not a live result.

```json
{"page_no":1,"page_size":1}
```

If the user chooses CLI, use the production MCP unified entry and select the business data environment in the browser authorization page:

```sh
pingpong --env production balances list --page_no 1 --page_size 1
```

For arrays or other filters, use a complete JSON object with --args or --args-file, not mixed with business parameter flags.

## Connection and execution

Use the user's selected PingPong environment and account. In a plugin session, use its authenticated MCP connection. If the user explicitly chooses the CLI, use the production MCP unified entry and the business environment selected during browser authorization; do not silently switch connectors. Skill installation and CLI login do not authenticate another MCP client.

Read the live tool schema before execution and use only supported snake_case arguments. If it differs from this guide, follow the live contract and report the mismatch. Do not pass tokens, app secrets, private keys, gateway URLs, or signatures as business arguments. The server supplies upstream credentials.

Respect an explicit page request exactly. For a simple lookup with no page preference, state that you are reading page 1 with 20 records; this is a skill choice, not a server default. For an explicitly requested full report, paginate only after identifying the actual response's list and termination semantics. Do not invent total/count/has_more fields or treat one page as the full account.

## Results and failures

- Read the actual business response, not just HTTP success or MCP isError=false. The server passes upstream JSON through; there is no guaranteed record or amount field schema. An empty object or unreadable result is not proof of a zero balance or no activity.
- Report filters, page coverage, retrieval time, and values with their actual currency and meaning. Preserve monetary precision; never add different currencies or label an unknown amount as available/frozen/net.
- MCP dry_run=true returns a validated request preview (output includes dryRun=true), not account data. CLI --dry-run is only a local request preview and does not verify authorization or full business rules.
- Tool missing: check the selected client's connection, authorization and enabled tools. Do not claim that running CLI login fixes that client's MCP session.
- TOOL_NOT_ALLOWED / SCOPE_INSUFFICIENT: report the denied tool/scope and stop that query. Do not bypass it through another connector or repeatedly reauthorize. Tool discovery is not proof of permission.
- 401: use the chosen connector's login flow if the user wants to renew the session. A 403 from downstream /v2/token/get indicates that upstream authentication or access was denied; check server credentials and access policies; report sanitized diagnostics to the maintainer instead of repeatedly logging in.
- Validation failure: correct only identified invalid inputs. For transient failures, use a bounded retry only if appropriate; keep partial results marked incomplete. Never report a failed query as an empty result.
