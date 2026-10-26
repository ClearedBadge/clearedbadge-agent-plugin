# Setup: connect ClearedBadge and install this skill

The skill only tells the agent how to use ClearedBadge. The tools come from the ClearedBadge MCP server, which must be connected and signed in separately.

- Server: `https://clrbadge-mcp.vercel.app/mcp` (Streamable HTTP, OAuth sign-in)
- Tools: `register_event` (records; counts against the account's registration quota) and `verify` (read only)
- Use is subject to the ClearedBadge Beta Terms (https://clrbadge-web.vercel.app/terms) and Privacy Policy (https://clrbadge-web.vercel.app/privacy).

The agent never handles passwords or keys: sign-in happens in the browser, through the client.

## Claude Code

### Route A: the plugin (server and skill together)

The plugin is the folder `plugin/clearedbadge` in the `clrbadge-mcp` repository.

```bash
claude --plugin-dir /path/to/clrbadge-mcp/plugin/clearedbadge
```

The plugin adds the MCP server `plugin:clearedbadge:clearedbadge` (tools `mcp__plugin_clearedbadge_clearedbadge__register_event` and `…__verify`) and the skill `clearedbadge:clearedbadge`. Then, in that session:
1. type `/mcp`;
2. choose `plugin:clearedbadge:clearedbadge`;
3. sign in in the browser.

### Route B: server and skill separately

1. Add the server once, for all your projects:
   ```bash
   claude mcp add --transport http --scope user clearedbadge https://clrbadge-mcp.vercel.app/mcp
   ```
2. Start `claude`, type `/mcp`, choose `clearedbadge`, sign in in the browser.
3. Copy the skill folder `plugin/clearedbadge/skills/clearedbadge` from the repository (the folder with `SKILL.md` and `references/`) to:
   - `~/.claude/skills/clearedbadge/` for all your projects, or
   - `<project>/.claude/skills/clearedbadge/` for one project.
4. Start a new session. The skill loads by itself when a request fits; you can also call it with `/clearedbadge`.

### Other ClearedBadge connections

If you also added ClearedBadge as a connector in claude.ai, Claude Code loads that connector too. With more than one ClearedBadge server, name the one to use in your request.

## First test: verify only (records nothing)

Ask, naming your server (with the plugin: `plugin:clearedbadge:clearedbadge`):

> Use ClearedBadge to verify record 00000000-0000-4000-8000-000000000000 against the event {"test": "connection"}.

Expected: the agent calls `verify` once and reports `not_found` ("no record with this ID"), not a mismatch. That shows the server is connected and signed in, without using any registration quota.

Register a first real event only when you want a record. Every `register_event` call that succeeds uses one registration of your quota, which does not reset.

## Other clients

Any MCP client that supports Streamable HTTP and OAuth can in principle connect; only the clients listed as tested have been tried. Skills are a separate feature per client; where a client has no skills, the tool descriptions of the server still apply. Status of each client: see the repository's `docs/AGENT-COMPATIBILITY.md`.

## Problems

| Symptom | Cause | Do |
|---|---|---|
| No `register_event` or `verify` tool | Server not added, or not loaded in this session | Check with `/mcp`; restart the session after adding |
| The server shows "needs authentication" in `/mcp` | Not signed in, or the sign-in expired | `/mcp` → the server → authenticate |
| `access_refused`, or HTTP 403 | This login has no (or no longer) ClearedBadge access | Sign in again with the right account |
| `quota_exhausted` | Registration quota used up | `verify` still works |
| HTTP 429 | Too many requests in a short time | Wait at least the Retry-After time, if shown |
