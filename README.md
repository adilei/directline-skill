# DirectLine Testing Skill for Claude Code

A [Claude Code custom skill](https://docs.anthropic.com/en/docs/claude-code/skills) that lets you interactively test Azure Bot Service and Microsoft Copilot Studio bots directly from the terminal using the [Direct Line API 3.0](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-api-reference).

## What it does

- Connects to any bot that supports the DirectLine channel (Azure Bot Service, Copilot Studio)
- Starts a conversation and enters an interactive chat loop
- Sends text messages and displays bot responses
- Uploads files (PDFs, images, documents) to the bot with optional text
- Supports regional endpoints (global, Europe, India) and scratch environments
- Handles both DirectLine secrets and Copilot Studio token endpoints

## Installation

Copy `SKILL.md` to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/directline
cp SKILL.md ~/.claude/skills/directline/
```

## Usage

### Connect to an Azure Bot

```
/directline <YOUR_DIRECTLINE_SECRET>
```

### Connect to a Copilot Studio agent

```
/directline <TOKEN_ENDPOINT_URL>
```

### Use a regional or scratch endpoint

```
/directline <SECRET> europe
/directline <SECRET> scratch
```

### Upload a file during a session

Once connected, ask Claude to upload a file:

```
upload ~/Documents/report.pdf and ask "what is this document about?"
```

## Supported bot sources

| Source | Auth |
|--------|------|
| **Azure Bot Service** | DirectLine secret (from Azure Portal > Bot > Channels > DirectLine) |
| **Copilot Studio** | Token endpoint URL (from Copilot Studio > Settings > Channels > Mobile app) |

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) CLI
- `curl` and `jq` available in your shell

## References

- [Direct Line API 3.0](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-api-reference)
- [Send Activity & Upload Attachments](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-send-activity)
- [About Direct Line Channel](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-channel-directline)
