# Play This Life plugins

Distribution packages for the Play This Life assistant connector
(`https://www.playthislife.com/api/mcp`). The server itself lives in the app repository; see its
`docs/mcp-connector.md`.

- `claude-code/`: Claude Code plugin. This repository is also a marketplace:
  `/plugin marketplace add Kindorbit/play-this-life-plugins`, then
  `/plugin install play-this-life@play-this-life-plugins`.
- `chatgpt/`: ChatGPT plugin package (Agent Plugins format). Build the upload with
  `cd chatgpt && zip -r ../play-this-life-chatgpt.zip .` after replacing `demo_recording_url`.

Validate the Claude Code plugin with `claude plugin validate .`.

## Connect in Claude

1. Open Claude's **Settings → Connectors**.
2. Choose **Add connector → Add custom connector**.
3. Enter **Play This Life** and `https://www.playthislife.com/api/mcp`.
4. Choose **Sign in now** and **Register automatically**, then add the connector.
5. Select **Connect**, sign in to Play This Life with Google if prompted, and review **Allow**.

Allow gives Claude access to read and change the connected account's tasks, progress and wins.
Disconnect from Claude's connector settings when you no longer want it connected.

Try these prompts:

- "What do I have today?"
- "Add a habit: stretch 10 minutes every weekday morning."
- "I finished my stretching today."
- "What did I complete this week?"

Changes appear in the Play This Life app. The connector can access only the account you connect.
Reading requires sign-in. Creating or changing tasks requires an active plan, trial or onboarding
access. Each write is a separate operation; repeating a request can repeat the change.

## Other clients

The same Streamable HTTP MCP endpoint supports OAuth with Dynamic Client Registration and S256
PKCE. Claude Code can use the plugin above, or add the server directly:

```sh
claude mcp add --transport http play-this-life https://www.playthislife.com/api/mcp
```

The ChatGPT package is prepared for directory review; it is not yet published in the directory.

## Support and privacy

Play This Life is operated by Kindorbit Software. See
[support](https://www.playthislife.com/contact),
[privacy](https://www.playthislife.com/privacy), and
[terms](https://www.playthislife.com/terms).
