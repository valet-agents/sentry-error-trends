# Sentry Webhook Received

The JSON webhook payload is appended directly after these
instructions in the user message. Parse it inline — do not fetch,
list, or search for the payload elsewhere. Do NOT use tools to
read the payload.

## Scope

This invocation is about **one specific Sentry issue** — the one
named in the payload. Do not scan, list, or comment on other
issues during this turn.

## What to extract from the payload

Pull these fields from the inline JSON:

- **issue id** (`data.issue.id` or top-level `id` depending on
  resource type) — used as the de-dup key.
- **fingerprint / culprit** — used to disambiguate similar errors.
- **title** and **level** (e.g. `error`, `warning`).
- **project slug** and **organization slug** — used to compose
  the Sentry issue URL.
- **event_count** (count of events in the spike window).
- **users_count** (number of distinct affected users).
- **first_seen** / **last_seen** timestamps — used for "started
  X ago" framing.
- **release** (if present) — used to correlate with deploys.

## Steps

1. **Parse** the inline payload and extract the fields above.
2. **De-dup check**: open `MEMORY.md` and look for this issue id
   in the recently-posted list. If it was posted within the last
   24 hours, **stop silently — do not post again.** Otherwise,
   continue.
3. **Enrich via sentry-mcp**: pull the most recent event for the
   issue (stack trace, breadcrumbs, request context, release tag),
   plus the 24-hour event/user counts to confirm the trend.
4. **Web-search via parallel-search-mcp**: run 1–2 targeted
   queries (exception class + first line of message + framework /
   library + version). Read the top results; pick the 3 strongest
   matches. Discard SEO-spam, listicles, and unrelated hits.
5. **Compose the hypothesis post** per the SOUL "Phase 4: Post
   the hypothesis" template — Sentry URL first, starting theory
   marked as a theory, up to 3 cited sources with short quotes.
6. **Resolve target channels**: list every Slack channel the bot
   is a member of and post once to each. If the bot is in zero
   channels, DM the workspace install user with the same
   hypothesis plus the one-line invite hint.
7. **Record the post**: append the issue id and the current
   timestamp to `MEMORY.md` under a `recently-posted` list so the
   24-hour de-dup check works on the next webhook.
8. Do not send any follow-ups, reactions, or thread replies after
   the initial post. Your turn ends after the posts complete.

## Skip conditions

Skip posting (and stop silently) if any of these are true:

- The issue id was posted within the last 24 hours (de-dup).
- The payload's resource is not an issue/event alert (e.g.
  installation lifecycle webhooks, comment events).
- The issue has fewer than 2 events in the last 24 hours **and**
  is not flagged as a regression — too quiet to be a "trend".
