# Receipt and verification fields

All values below are **synthetic examples**. Never copy them into an answer: use only the fields of the real tool result.

## A `recorded` receipt (register_event)

```json
{
  "outcome": "recorded",
  "recorded": true,
  "record_id": "00000000-0000-4000-8000-000000000000",
  "hash": "<64 lower-case hex characters>",
  "canonicalization": "RFC8785",
  "hash_algorithm": "SHA-256",
  "recorded_at": "2026-01-01T12:00:00.000Z",
  "status": "accepted",
  "receipt_version": 1,
  "registry_id": "clearedbadge",
  "environment": "production",
  "content_kind": "json_event",
  "verify_url": "https://clrbadge-web.vercel.app/verify?id=00000000-0000-4000-8000-000000000000",
  "attempt_id": "<uuid>",
  "quota": {"total": 10, "used": 1, "remaining": 9, "resets": false},
  "recipient_note": "…",
  "arweave_note": "…"
}
```

| Field | Present | Meaning |
|---|---|---|
| `record_id` | always on `recorded` | The record ID; also the `id` in the Verify link |
| `hash` | always | SHA-256 of the RFC 8785 canonical event |
| `canonicalization`, `hash_algorithm` | always | `RFC8785`, `SHA-256` |
| `recorded_at` | when the server returned it in a valid form | ClearedBadge's server time of registration, not when the content was made |
| `status` | idem | The record status (`accepted`) |
| `receipt_version`, `registry_id`, `environment`, `content_kind` | when the server knows its environment | Receipt metadata. `environment` is `production` or `test`: **a `test` receipt is not a production record.** |
| `verify_url` | always on `recorded` | Where the recipient can check the unchanged event |
| `attempt_id` | always | The attempt; quote it if anything is unclear |
| `quota` | in online mode | The account's lifetime registration quota after this record |
| `upstream_warning` | sometimes | An optional field had an unexpected format and was left out |
| `ledger_warning` | rarely | The attempt log could not be updated; a repeat of this event will be treated as uncertain |

What to give the user after `recorded`, in this order:
1. the record ID and the Verify link;
2. the hash and the recorded time, if present;
3. where the saved event and receipt are, and "Keep the event JSON unchanged: the record keeps only its hash.";
4. one short sentence on what a record does not show (more only if the user seems to read more into it);
5. the Arweave status, if known, as a separate fact. Registration does not wait for anchoring.

Keep it short: these five points, not a long disclaimer.

## A `verify` result

```json
{
  "record_id": "00000000-0000-4000-8000-000000000000",
  "content_result": "match",
  "hash": "<64 hex>",
  "note": "A match only means …",
  "arweave": {"arweave_status": "not_anchored", "not_started": {"reason": "submission_paused"}, "note": "…"},
  "verify_url": "https://clrbadge-web.vercel.app/verify?id=00000000-0000-4000-8000-000000000000"
}
```

On a `mismatch`, the result also has `submitted_hash` (the hash of the supplied event) and `message`. With a `txid`, `arweave` can also have `payload_sha256`, `submitted_at` and `confirmed_at`.

## The Verify link

`verify_url` opens ClearedBadge's Verify page. There, the recipient pastes the JSON or drops a file, and the page compares it **in the browser**: only the record ID is sent to ClearedBadge. The `id` query parameter is the `record_id`.
