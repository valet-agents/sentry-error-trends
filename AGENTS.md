This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **sentry-mcp**: The Sentry MCP server, authenticated with an internal integration token. The agent uses it to pull issue details, stack traces, breadcrumbs, recent events, release context, and affected-user counts. Add it from the catalog at the org level so other Sentry-powered agents can share it.
- **parallel-search-mcp**: Parallel search, the web-research tool. Used to find similar bugs in GitHub issues, Stack Overflow, and engineering blogs after the Sentry context is pulled. No API key required.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts trend hypotheses to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **sentry-webhook** (sentry-webhook): The real-time trigger. Verifies the `Sentry-Hook-Signature` (HMAC-SHA256) on incoming Sentry webhooks using `SENTRY_CLIENT_SECRET`, then hands the payload to the agent. After deploy, you'll get a webhook URL — paste it into Sentry → Settings → Developer Settings → your internal integration → Webhook URL.

### Secrets

- **SENTRY_ACCESS_TOKEN** (on the `sentry-mcp` connector): Sentry internal integration token. Create one at Sentry → Settings → Developer Settings → New Internal Integration. Required scopes: `event:read`, `project:read`, `org:read`. Optionally add `event:write` and `issue:write` if you plan to use the confirmed-write flow (resolve/comment) from Slack.
- **SENTRY_CLIENT_SECRET** (on the `sentry-webhook` channel): Auto-managed by the channel, but you'll need to copy the same value into your Sentry internal integration's Client Secret field so signatures verify on both sides. Sentry shows this once when you create the integration; rotate via Sentry if it leaks.

### External Setup

1. **Create a Sentry internal integration**: Sentry → Settings → Developer Settings → New Internal Integration. Give it `event:read`, `project:read`, `org:read` (and optionally `event:write` + `issue:write` for the confirmed-write flow). Copy the **token** into `SENTRY_ACCESS_TOKEN` and the **client secret** into `SENTRY_CLIENT_SECRET`.
2. **Wire the webhook**: After deploy, copy the agent's webhook URL into the integration's **Webhook URL** field, enable the `issue` and `event_alert` resources, and save.
3. **Invite the Slack bot**: Invite the agent's bot to whichever channel(s) you want hypotheses to land in. The agent posts to every channel it's a member of — invite it to one focused channel, or several. If the bot has not been invited anywhere, hypotheses go as a DM to the workspace install user with a one-line nudge.
4. **Smoke-test**: trigger a test webhook from your Sentry integration's UI, or @mention the bot in Slack with a question like *"what's the noisiest issue today?"* — that exercises the Slack + Sentry path without waiting for a real spike.

## Customizing

- **Change which Sentry events trigger the agent**: edit the resources enabled on your Sentry internal integration (issue, event_alert, error, etc.). The channel passes through whatever Sentry sends.
- **Control where hypotheses post**: invite or remove the bot from channels in Slack — that's the only signal the agent uses. There is no channel name in the configuration.
- **Tune the de-dup window**: the agent skips re-posting any Sentry issue id it has posted in the last 24h, tracked in `MEMORY.md`. Adjust the window in `SOUL.md` if your team prefers more or fewer repeats on regressions.
- **Bound web-search citations**: the SOUL caps citations at 3 per post. Lower or raise the cap there if your team wants tighter or richer hypothesis posts.
