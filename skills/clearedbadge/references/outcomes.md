# Tool results and what to do

Source: the ClearedBadge MCP server as live since 25 October 2026.

## register_event: `outcome`

| `outcome` | `recorded` | Meaning | Do |
|---|---|---|---|
| `recorded` | `true` | ClearedBadge accepted the event and returned a receipt. | Give the receipt fields from the result (see receipt.md). Save the receipt next to the saved event. Say what a record does not show. |
| `already_recorded` | `true` | This exact event was recorded by an earlier call of this account; **nothing was sent now**. | Give its `record_id` and `verify_url`. Do not set `record_again` unless the user explicitly wants a second, separate record. |
| `unknown` | `null` | The request may have been processed (time-out, server error, broken answer). | **Do not call again.** Tell the user the record may exist, give the `attempt_id`, and keep the saved event. |
| `blocked_unknown_earlier_attempt` | `null` | An earlier call for this exact event ended unknown; nothing was sent now. | Same as `unknown`: do not call again on your own. Set `record_again: true` only after the user explicitly accepts a possible second, separate record. |
| `in_flight_or_interrupted` | — | Only on a local (stdio) server: an earlier attempt for this event was not finished. | Treat as unknown. |
| `not_sent` | `false` | No connection to ClearedBadge; nothing was sent or recorded. | You may call once more later; tell the user if it keeps failing. |
| `refused` | `false` | ClearedBadge refused the event (`http_status`, `error`); nothing was recorded. | Report the `error`. On `401`/`403` the server's own credential is wrong: do not retry, tell the operator. |
| `invalid_input` | `false` | The event is not valid for registration (not an object, NaN/Infinity, lone surrogate, too large: more than 1 MiB, a `__proto__` key, too deeply nested). Nothing was sent. | Report the `detail`. Do not alter the content silently; let the user decide. |
| `quota_exhausted` | `false` | The account's total registration quota is used up (it does not reset). Nothing was sent. | Tell the user; `verify` still works. |
| `access_refused` | `false` | The sign-in or key is missing, unknown or revoked. Nothing was sent. | Ask the user to sign in again (`/mcp`). |
| `service_unavailable` | `false` | The account database or the server's credential is not available. Nothing was sent. | Tell the user; try later. Do not loop. |
| `not_configured` | `false` | This MCP server has no ClearedBadge destination. | Tell the user the connector is not set up. |

## No tool result: an error from the client

Sometimes the client reports an error instead of a tool result. Where the error comes from decides what it means. The first four rows apply **only** to a recognisable refusal by ClearedBadge's access check: the status code **and** one of the error codes shown. A status code alone does not show its source (a proxy, the hosting platform or the client can answer with the same code); then the last row applies.

| Error | Where it comes from | What is known | Do |
|---|---|---|---|
| HTTP **401** (`unauthorized`, `invalid_token`) | The server's sign-in check, which answers before the MCP tools run | The call was refused before it ran: nothing was sent to the registry. | Ask the user to sign in again (`/mcp`). |
| HTTP **403** (`access_refused`, `not_linked`, `signup_closed`) | The same check: the login is valid but has no (or no longer) ClearedBadge access | Refused before it ran. | Tell the user; do not retry. |
| HTTP **429** (`rate_limited`, `signup_busy`) | The server's request limits, in the same check | Refused before it ran. | Wait at least the `Retry-After` header or `retry_after` value, if shown. If no value is shown, do not invent a waiting time: tell the user and let them try later. |
| HTTP **503** with `busy` or `service_unavailable` | The same check | Refused before it ran. | Try later; do not loop. |
| Any other error: another 5xx, a time-out, a lost connection, a 401/403/429/503 without one of the error codes above, or an error that does not show its source | Unknown: possibly after the tool started and the event was sent | **The event may be recorded.** | Treat it like `unknown`. Do not call `register_event` again on your own; tell the user, with the saved event. |

After an unclear error, do **not** retry on your own: a repeat is the user's decision. If they want one, call **once**, under all of these conditions:
- the **same server**;
- signed in to the **same account**;
- **exactly the same event**;
- `record_again: false`.

The server is built to recognise an earlier attempt for that event under those conditions. It then answers `already_recorded` (the first call recorded it) or `blocked_unknown_earlier_attempt` (still unclear) instead of making a second record. That is how the server is designed. Its production configuration has not been checked against that design, so this is **not a guarantee**. A different server or account, a changed event, or `record_again: true` can make a new record. On a local (stdio) server, a `ledger_warning` in an earlier result means this protection may be missing.

`verify` is read only, so repeating it after any of these errors is harmless. After a 429, wait as above.

## verify: `content_result`

| `content_result` | Meaning | Say |
|---|---|---|
| `match` | The supplied event, canonicalised with RFC 8785, has the SHA-256 recorded under this record ID. | "The content matches the recorded SHA-256 fingerprint." Add what that does not show. |
| `mismatch` | The supplied event has a different hash (`hash` = recorded, `submitted_hash` = supplied). | "It does not match this record." Not by itself evidence of tampering. |
| `not_found` | No record with this record ID. | "No record with this ID." Check the ID; this is not a mismatch. |
| `unavailable` | Network or service problem, or a contradictory answer. Nothing was compared. | "The check could not be made." **Not a match, not a mismatch.** |
| `refused` | ClearedBadge refused the comparison request. | Report the `error`. Not a mismatch. |
| `invalid_input` | The record ID is not a lower-case UUID, or the event is not valid JSON for comparison. Nothing was sent. | Report the `detail`. |
| `not_configured` | The server has no ClearedBadge destination. | The connector is not set up. |

JSON events are compared after RFC 8785: key order and whitespace do not matter, every value does. Duplicate keys or lone surrogates cannot be compared.

## verify: `arweave.arweave_status`

Reported separately from the content check, by the ClearedBadge anchor service.

| Status | Meaning |
|---|---|
| `not_anchored` | Not published on Arweave. `not_started.reason` gives why, for example `submission_paused`. |
| `submitted` | Sent to Arweave, **not yet confirmed**. |
| `confirmed` | At least 20 block confirmations and a byte-exact re-read of the anchored payload. ClearedBadge's own rule; there is no "final" status. |
| `failed` | The anchor service recorded the attempt as failed. |
| `unavailable` | The status could not be read. Says nothing about the content check. |
| `not_configured` / `not_checked` | Not looked up (no anchor service on this server, or the record was not found). |

Registration does not wait for anchoring. A `recorded` receipt with `not_anchored` or `submitted` is normal.
