# Install, test and remove

## A. Recommended: install from the marketplace (Claude Code 2.1.292 or later)

1. In your shell:
   ```bash
   claude plugin install clearedbadge --marketplace ClearedBadge/clearedbadge-agent-plugin
   ```
   This adds the marketplace `clearedbadge` and installs the plugin for your user account. Inside a session, `/plugin install clearedbadge --marketplace ClearedBadge/clearedbadge-agent-plugin` does the same and lets you pick a scope (Claude Code 2.1.275 or later).

   The same in two steps:
   ```bash
   claude plugin marketplace add ClearedBadge/clearedbadge-agent-plugin
   ```
   ```bash
   claude plugin install clearedbadge@clearedbadge
   ```
2. Start Claude Code. Type `/mcp`, choose `plugin:clearedbadge:clearedbadge`, and sign in in the browser.
3. Check the connection without recording anything: see [examples/first-check.md](examples/first-check.md).

## B. A local copy, for testing or changing the plugin

Get a copy:

```bash
git clone https://github.com/ClearedBadge/clearedbadge-agent-plugin
```

Then, from the folder that contains the clone:

- **One session, nothing installed:**
  ```bash
  claude --plugin-dir ./clearedbadge-agent-plugin
  ```
- **Installed from a local marketplace**, exactly as users will install it:
  ```bash
  claude plugin marketplace add ./clearedbadge-agent-plugin
  ```
  ```bash
  claude plugin install clearedbadge@clearedbadge
  ```
  Remove it again with `claude plugin marketplace remove clearedbadge`.

Check the files before you share them:

```bash
claude plugin validate ./clearedbadge-agent-plugin
```

## C. Developers: only the skill, server added yourself

If you manage MCP servers yourself, or use the skill in a project:

1. Add the server:
   ```bash
   claude mcp add --transport http --scope user clearedbadge https://clrbadge-mcp.vercel.app/mcp
   ```
   Then sign in through `/mcp` → `clearedbadge`.
2. Copy the folder `skills/clearedbadge/` from this package to `~/.claude/skills/clearedbadge/` (all projects) or to `<project>/.claude/skills/clearedbadge/` (one project).
3. Start a new session. With this route the server is called `clearedbadge`, and the skill is `/clearedbadge`.

Other MCP clients can connect to `https://clrbadge-mcp.vercel.app/mcp` (Streamable HTTP, OAuth sign-in). Only Claude Code has been tested with this package.

## More than one ClearedBadge connection

If you also added ClearedBadge as a connector on claude.ai, Claude Code loads that connector too. Name the server you want in your request, for example "use plugin:clearedbadge:clearedbadge". The skill asks which one to use before it records anything.

## Problems

| Symptom | Do |
|---|---|
| `Plugin "clearedbadge" not found in marketplace` | Check the marketplace source; `claude plugin marketplace list` shows what is added |
| No ClearedBadge tools in a session | `/mcp`: is `plugin:clearedbadge:clearedbadge` there and signed in? Restart the session after installing |
| The server shows "needs authentication" | `/mcp` → `plugin:clearedbadge:clearedbadge` → authenticate |
| `access_refused`, or HTTP 403 | This login has no (or no longer) ClearedBadge access |
| `quota_exhausted` | The registration quota is used up; `verify` still works |
| HTTP 429 | Too many requests: wait at least the Retry-After time, if shown |

## Remove

1. Uninstall the plugin:
   ```bash
   claude plugin uninstall clearedbadge@clearedbadge
   ```
2. Remove the marketplace (this also uninstalls its plugins):
   ```bash
   claude plugin marketplace remove clearedbadge
   ```
3. With route C, also remove the server, and delete the folder `~/.claude/skills/clearedbadge/`:
   ```bash
   claude mcp remove clearedbadge --scope user
   ```

Removing the plugin does not delete records or your ClearedBadge account; records cannot be removed. For your account and data, see the [Privacy Policy](https://clrbadge-web.vercel.app/privacy).
