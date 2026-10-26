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
