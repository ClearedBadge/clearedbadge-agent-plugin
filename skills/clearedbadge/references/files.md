# Files

The ClearedBadge MCP server has two tools, `register_event` and `verify`, and both take **JSON objects**. It has **no file tool**: it cannot register or compare the exact bytes of a file.

**Do not pretend otherwise:**
- Wrapping a file's text or a file's hash in a JSON event registers **that event**, not the file. Verifying the file later against that record does not work.
- Only do it if the user explicitly wants an event that *describes* the file, for example `{"type": "report.delivered", "file_sha256": "<hash>", "name": "report-v2"}`. Say clearly that the record is of the event.

**ClearedBadge's file routes** (outside this MCP, public, no account; with daily limits per network):
- the **Record page**: https://clrbadge-web.vercel.app/record. **Private** computes the fingerprint in the browser, and only that is sent. **Standard** uploads the file, which is hashed in memory and not stored. Hand the user the link.
- the **API**, for developers: `POST https://clrbadge-seven.vercel.app/record-file-hash` with exactly `{"sha256":"<64 lower-case hex>"}`, or `POST /upload` (multipart, at most 4 MB). Use these only if the user asks you to and you have an HTTP tool. Report the real response; the receipt fields are `badge_id`, `hash`, `created_at`, `status` and `fingerprint_method`.
- **verifying a file:** the Verify page with the record ID (in the browser, byte for byte).

JSON is a special case. A JSON **file** is compared byte for byte; the same content as a JSON **event** is compared after RFC 8785. They are different records: register in the form in which the content will be handed over.
