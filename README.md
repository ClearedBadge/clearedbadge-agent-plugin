# ClearedBadge for Claude Code

Record a fingerprint of a final JSON result, so that anyone with an unchanged copy can later check it is the recorded version.

## Start in three steps

1. **Install** (Claude Code 2.1.292 or later), in your shell:
   ```bash
   claude plugin install clearedbadge --marketplace ClearedBadge/clearedbadge-agent-plugin
   ```
   Or inside a session (2.1.275 or later):
   ```text
   /plugin install clearedbadge --marketplace ClearedBadge/clearedbadge-agent-plugin
   ```
2. **Sign in once:** start Claude Code, type `/mcp`, choose `plugin:clearedbadge:clearedbadge` and sign in in the browser. Claude never sees a password or key.
3. **Try it safely** (records nothing, uses no quota). Ask Claude:
   > Use ClearedBadge (the plugin:clearedbadge:clearedbadge server) to verify record 00000000-0000-4000-8000-000000000000 against the event {"test": "connection"}.

   The answer should say there is **no record with this ID** (`not_found`). That is not a mismatch: nothing was compared. It shows the connection works.

To record something, ask Claude to register a final JSON result with ClearedBadge. Claude saves the exact JSON first and gives you a record ID and a Verify link. Keep the JSON unchanged: ClearedBadge keeps only its fingerprint. More examples: [examples/first-check.md](examples/first-check.md). Other install routes and removal: [INSTALL.md](INSTALL.md).

## What a record does and does not show

A record shows that this exact JSON matched this fingerprint at the server's recorded time. It does **not** show:
- that the content is true;
- who made or sent it;
- that anyone received it;
- that the described action happened;
- that it has legal effect.

The plugin records **JSON events**, not the exact bytes of a file such as a PDF. For files, use the Record page at https://clrbadge-web.vercel.app/record.

## When it is useful

- You hand a final JSON result to someone else (a quote, an export, an agent's final output) and want that exact version on record, so that a copy shown later can be compared with it.
- You receive a JSON result with a ClearedBadge record ID or Verify link and want to check whether it is the recorded version.

## What the three parts do

- **The MCP server** (`https://clrbadge-mcp.vercel.app/mcp`, hosted by ClearedBadge) provides two tools. `register_event` makes a record; `verify` compares a JSON event with a record and changes nothing.
- **The skill** (`skills/clearedbadge/`) tells Claude when to register, verify, propose or do nothing, how to read every result, and what a record does not show.
- **The plugin** installs both in one step. It runs no code on your machine.

## What is sent to ClearedBadge

- **When you register or verify:** the **whole JSON event** goes to the ClearedBadge server, which computes the fingerprint there. The record keeps the fingerprint and the time, not the event. Do not put secrets in an event. Put in personal data only if you may share it.
- **When you verify:** the record ID goes along with the event.
- **When you sign in:** the sign-in identifies your ClearedBadge account.

What ClearedBadge processes and keeps: [Privacy Policy](https://clrbadge-web.vercel.app/privacy). Use of the service: [Beta Terms](https://clrbadge-web.vercel.app/terms).

## Permissions, cost and quota

- **What it adds:** the plugin adds one skill and one MCP server, and nothing else. The `/plugin` panel shows this before you install; after a shell install, `claude plugin details clearedbadge` lists it.
- **Tool calls:** depending on your permission settings, Claude Code asks before it calls a ClearedBadge tool. `register_event` creates a record that cannot be removed; `verify` only reads.
- **Cost:** per the Beta Terms, the beta is free of charge.
- **Quota:** each account has a small, fixed registration quota that does not reset. A successful `register_event` uses one unit; `verify` uses none.

## Remove

```bash
claude plugin uninstall clearedbadge@clearedbadge
claude plugin marketplace remove clearedbadge
```

## Known limits

- **Tested:** Claude Code only, on Windows. Other MCP clients may work with the server, but are not tested with this plugin.
- **Activation:** Claude loads the skill reliably when you name ClearedBadge. It does not reliably do so for a need described without the name.
- **Production:** sign-in and a `verify` call through the plugin have been checked in production. A registration through the plugin has not been checked there yet.

## Licence

MIT, for the files in this package only: [LICENSE](LICENSE). The name ClearedBadge and the hosted service are not covered; see [NOTICE.md](NOTICE.md).
