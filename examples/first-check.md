# First check: verify only, records nothing

After installing and signing in (`/mcp` → `plugin:clearedbadge:clearedbadge`), ask Claude:

> Use ClearedBadge (the plugin:clearedbadge:clearedbadge server) to verify record 00000000-0000-4000-8000-000000000000 against the event {"test": "connection"}.

Expected:
- Claude calls `verify` once.
- The answer says there is **no record with this ID** (`not_found`). This is not a mismatch: nothing was compared.
- The Arweave status is `not_checked`, because there is no record.

That shows the plugin, the sign-in and the server work. It uses no registration quota.

## Recording a result (uses one unit of quota)

> Save this final quote as quote.event.json and register it with ClearedBadge: (paste the JSON, or use [sample-event.json](sample-event.json))

Claude saves the exact JSON first, calls `register_event` once, and gives you:
- the record ID;
- the fingerprint (hash);
- the recorded time;
- the Verify link.

Keep the JSON file unchanged: ClearedBadge keeps only its fingerprint.

## Checking a result you received

> Does quote.event.json match ClearedBadge record <record ID>?

The answer is `match` or `mismatch`. If the check could not be made, the answer says that instead, for example `not_found` or `unavailable`; that is neither a match nor a mismatch. A match only shows that this JSON is the recorded version. It shows nothing about who sent it, whether it arrived, or whether it is true.
