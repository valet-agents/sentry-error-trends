# Sentry Error Trends

On every Sentry trend it pulls the stack trace, searches the web for similar bugs, and posts a starting theory before you've opened your laptop.

## Prerequisites
- A [Sentry](https://sentry.io) organization where you can create an internal integration (API access + webhook)
- A Sentry project configured to send issue/alert webhooks to the agent's webhook URL after deploy
- A Slack workspace where you can install the agent's bot and invite it to one or more channels

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> · <code>sentry-webhook</code></td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>sentry-mcp</code> · <code>parallel-search-mcp</code></td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <br />
      <a href="https://valet.dev/deploy?from=github.com/valet-agents/sentry-error-trends">
        <img src="https://raw.githubusercontent.com/valet-agents/sentry-error-trends/main/.github/deploy-button.svg" alt="Deploy Agent →" height="40" />
      </a>
      <br /><br />
    </td>
  </tr>
</table>
