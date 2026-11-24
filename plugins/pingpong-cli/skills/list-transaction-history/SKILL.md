---
name: list-transaction-history
description: "Query PingPong transaction history by business source, order or time range. Use for transaction records and returned statuses; do not confuse these records with balance movements."
---

Use this plugin's configured pingpong-cli MCP connection. If the endpoint is not a PingPong production or approved sandbox endpoint, ask the maintainer before querying. Do not silently change connectors or environments.


# Transaction history

Read-only PingPong tool: list_transaction_history. This skill does not create invoices, subscriptions, payments, refunds or accounting entries.

## Inputs

| Argument | Required | Constraint |
| --- | --- | --- |
| source | yes | ECOMMERCE_TRADE, OFFLINE_TRADE or GENERAL_ACCOUNT |
| page_no / page_size | yes | Integer ≥ 1 / integer 1–100 |
| order_id | no | Optional order lookup; takes precedence over dates |
| start_time | no | Optional offset ISO8601 lower bound; if omitted, API defaults to one month before now |
| end_time | no | Optional offset ISO8601 upper bound; if omitted, API defaults to now |
| order_type_list | no | Nonempty list allowed only for GENERAL_ACCOUNT; RECEIVE, RETURN, PAYOUT, WITHDRAW, REFUND, TRANSFER, ADJUSTMENT |
| status_list | no | String array; valid status values must come from verified current docs, not guessed enums |
| dry_run | no | Boolean; request preview only |

## Workflow

1. Resolve source from explicit user context. If missing or ambiguous, ask which business source to query; do not silently default to GENERAL_ACCOUNT or query all sources.
2. Use `order_id` for an order lookup; when supplied, omit dates because the server ignores them. Otherwise pass whichever time bounds the user specified. If neither is provided, the API defaults to one month before now through now; a single omitted bound uses its documented default.
3. Only provide order_type_list for GENERAL_ACCOUNT. A nonempty list with another source is rejected, not ignored.
4. Query list_transaction_history. This tool does not support currency_list, subaccount_id_list or account_type; do not pass them. Any client-side filter must be labeled and cannot establish full coverage without full pagination.
5. Report statuses exactly as returned; do not translate an unknown status into success or settlement. For a matching balance effect, consult balance movements and label unmatched records as unresolved, not as proof that money is missing.

## Example

Replace sample identifiers with the user's actual values. The example is not a live result.

```json
{"source":"GENERAL_ACCOUNT","order_id":"YOUR_ORDER_ID","page_no":1,"page_size":20}
```

If the user chooses CLI, use the production MCP unified entry and select the business data environment in the browser authorization page:

```sh
pingpong --env production transactions list --source GENERAL_ACCOUNT --order_id YOUR_ORDER_ID --page_no 1 --page_size 20
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
- 429 / rate limited or tool_rate: stop parallel pagination. Do not retry the same wide-range page loop. Report the reported wait time, then narrow the period to day or week segments and use `page_size=100` with sequential pages. A monthly report with many rows may not be retrievable page-by-page in one run.
