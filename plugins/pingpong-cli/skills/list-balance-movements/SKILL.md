---
name: list-balance-movements
description: "Query PingPong balance movements for an order or a bounded time range. Use to trace changes in a funding account balance; this is not a trade-status query."
---

Use this plugin's configured pingpong-cli MCP connection. If the endpoint is not a PingPong production or approved sandbox endpoint, ask the maintainer before querying. Do not silently change connectors or environments.


# Balance movements

Read-only PingPong tool: list_balance_movements. This skill does not create invoices, subscriptions, payments, refunds or accounting entries.

## Inputs

| Argument | Required | Constraint |
| --- | --- | --- |
| page_no / page_size | yes | Integer ≥ 1 / integer 1–100 |
| order_id | no | Optional order lookup; takes precedence over dates |
| start_time | no | Optional offset ISO8601 lower bound; if omitted, API defaults to one month before now |
| end_time | no | Optional offset ISO8601 upper bound; if omitted, API defaults to now |
| account_type | no | BALANCE_ACCOUNT only |
| currency_list | no | Array of three-uppercase-letter currency codes |
| subaccount_id_list | no | Array of alphanumeric-only account IDs |
| dry_run | no | Boolean; request preview only |

## Workflow

1. Use `order_id` for a known order; when supplied, omit dates because the server ignores them.
2. Otherwise pass whichever time bounds the user specified. Each bound is optional; if omitted, the API defaults start to one month before now and end to now. If neither is specified, this default period is queried. Ask only when the requested period or timezone is ambiguous.
3. Query list_balance_movements. This tool has no source, status_list or order_type_list arguments.
4. Summarize the actual returned movements, currencies and matching identifiers. A movement is not necessarily a sale, expense, or settlement; infer direction only from documented response semantics.
5. A period exceeding 365 days requires a narrower range or an explicitly scoped segmentation plan. Boundary inclusivity and stable deduplication keys are not established here; do not promise gap-free or duplicate-free merging.

## Example

Replace sample identifiers with the user's actual values. The example is not a live result.

```json
{"order_id":"YOUR_ORDER_ID","page_no":1,"page_size":20}
```

If the user chooses CLI, replace demo with their selected configured environment:

```sh
pingpong --env demo balance-movements list --order_id YOUR_ORDER_ID --page_no 1 --page_size 20
```

For arrays or other filters, use a complete JSON object with --args or --args-file, not mixed with business parameter flags.

## Connection and execution

Use the user's selected PingPong environment and account. In a plugin session, use its authenticated MCP connection. If the user explicitly chooses the CLI, use the commands below with that environment; do not silently switch connectors or environments. Skill installation and CLI login do not authenticate another MCP client.

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
