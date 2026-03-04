---
name: directline
description: Test Azure Bot Service and Copilot Studio bots via DirectLine API
argument-hint: <secret_or_token_endpoint> [region]
---

# DirectLine Testing Skill

Test Azure Bot Service and Copilot Studio bots via DirectLine API.

## Usage
```
/directline <secret_or_token_endpoint> [region]
```

- `secret_or_token_endpoint`: Either a DirectLine secret OR a Copilot Studio token endpoint URL
- `region`: Optional. One of: `scratch`, `europe`, `india`. Defaults to global production endpoint.

## Instructions

When this skill is invoked:

### 1. Determine Token Source

**If arg is a URL (Copilot Studio token endpoint):**
```bash
TOKEN_RESP=$(curl -s -X GET "{TOKEN_ENDPOINT}")
TOKEN=$(echo "$TOKEN_RESP" | jq -r '.token')
# User ID is embedded in the token - extract it:
USER_ID=$(echo "$TOKEN" | cut -d'.' -f2 | base64 -d 2>/dev/null | jq -r '.user')
```

**If arg is a secret (Azure Bot DirectLine):**
```bash
USER_ID="claude_$(date +%s)"
# Generate token with user ID
TOKEN_RESP=$(curl -s -X POST "https://{endpoint}/v3/directline/tokens/generate" \
  -H "Authorization: Bearer {SECRET}" \
  -H "Content-Type: application/json" \
  -d '{"user":{"id":"'"$USER_ID"'"}}')
TOKEN=$(echo "$TOKEN_RESP" | jq -r '.token')
```

### 2. Select Endpoint

**By Environment:**

| Source | Environment | Endpoint |
|--------|-------------|----------|
| Copilot Studio | Production | `directline.botframework.com` |
| Azure Bot | Production | `directline.botframework.com` |
| Azure Bot | Scratch | `webchat.scratch.botframework.com` |

**Regional Endpoints (Production):**

| Region | Endpoint |
|--------|----------|
| Global (default) | `directline.botframework.com` |
| Europe | `europe.directline.botframework.com` |
| India | `india.directline.botframework.com` |

Use regional endpoints when the Azure Bot is deployed in a specific region for lower latency and data residency compliance.

### 3. Start Conversation
```bash
CONV_RESP=$(curl -s -X POST "https://{endpoint}/v3/directline/conversations" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json")
CONV_TOKEN=$(echo "$CONV_RESP" | jq -r '.token')
CONV_ID=$(echo "$CONV_RESP" | jq -r '.conversationId')
```

### 4. Interactive Mode

**To send a user message:**
```bash
curl -s -X POST "https://{endpoint}/v3/directline/conversations/{CONV_ID}/activities" \
  -H "Authorization: Bearer {CONV_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "message",
    "text": "<user message>",
    "from": {
      "id": "'"$USER_ID"'",
      "role": "user"
    }
  }'
```

**To upload a file (no text):**
```bash
curl -s -X POST "https://{endpoint}/v3/directline/conversations/{CONV_ID}/upload?userId=$USER_ID" \
  -H "Authorization: Bearer {CONV_TOKEN}" \
  -H "Content-Type: <mime_type>" \
  -H 'Content-Disposition: name="file"; filename="<filename>"' \
  --data-binary @/path/to/file
```

**To upload a file with a text message (multipart):**
```bash
curl -s -X POST "https://{endpoint}/v3/directline/conversations/{CONV_ID}/upload?userId=$USER_ID" \
  -H "Authorization: Bearer {CONV_TOKEN}" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/file;type=<mime_type>;filename=<filename>" \
  -F "activity={\"type\":\"message\",\"from\":{\"id\":\"$USER_ID\",\"role\":\"user\"},\"text\":\"<user message>\"};type=application/vnd.microsoft.activity"
```

Common MIME types: `application/pdf`, `image/png`, `image/jpeg`, `text/plain`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document` (docx), `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` (xlsx).

**To get bot responses:**
```bash
curl -s "https://{endpoint}/v3/directline/conversations/{CONV_ID}/activities" \
  -H "Authorization: Bearer {CONV_TOKEN}"
```

### 5. Display Results
- Show bot messages to user
- Track watermark for incremental updates

## Critical Requirements

1. **User ID consistency**: The `from.id` in sent activities MUST exactly match the user ID from the token. For the upload endpoint, the `userId` query parameter must also match.
2. **Role field**: Always include `"role": "user"` in the from object
3. **Token usage**: Use the conversation token (from Start Conversation response) for subsequent requests
4. **File uploads**: Use the `/upload` endpoint with `--data-binary` (not `-d`) to preserve binary content. Uploaded files are deleted by DirectLine after 24 hours.

## Tested Configurations

### Azure Bot (Production)
- Endpoint: `directline.botframework.com`
- Secret: From Azure Bot → Channels → DirectLine (format: `<siteId>.<secretKey>`)
- Token generation required with user ID
- Bot response: Depends on bot implementation

### Azure Bot (Scratch)
- Endpoint: `webchat.scratch.botframework.com`
- Secret: From Azure Bot → Channels → DirectLine (format: `<siteId>.<secretKey>`)
- Token generation required with user ID
- Bot response: Depends on bot implementation

### Copilot Studio (Production)
- Token endpoint: `https://<environmentId>.environment.api.powerplatform.com/powervirtualagents/botsbyschema/<botSchemaName>/directline/token?api-version=2022-03-01-preview`
- Endpoint: `directline.botframework.com`
- Token already contains user ID (extract from JWT payload)
- Bot response: Depends on bot implementation

## Example Sessions

**Azure Bot (production):**
```
User: /directline <YOUR_DIRECTLINE_SECRET>

Claude: Connecting to directline.botframework.com...
        Conv ID: aBcDeFgHiJkLmNoPqRsTuV-xx
        Bot: "Hello and Welcome!"

User: Hello!
Claude: Bot: "You said: Hello!"
```

**Azure Bot (scratch):**
```
User: /directline <YOUR_DIRECTLINE_SECRET> scratch

Claude: Connecting to webchat.scratch.botframework.com...
        Conv ID: aBcDeFgHiJkLmNoPqRsTuV-xx
        Bot: "Hello and Welcome!"

User: What is 2+2?
Claude: Bot: "You said: What is 2+2?"
```

**Copilot Studio (production):**
```
User: /directline https://<env>.environment.api.powerplatform.com/.../token?api-version=2022-03-01-preview

Claude: Getting token from Copilot Studio...
        Conv ID: aBcDeFgHiJkLmNoPqRsTuV-xx
        Connected to directline.botframework.com

User: Find books about Python
Claude: Bot: "**Books About Python Programming**
        Here are some highly recommended books..."
```

**File upload (during a session):**
```
User: upload ~/Documents/report.pdf and ask "summarize this document"

Claude: Uploading report.pdf (application/pdf) with message...
        Activity ID: 0003
        [polls for response]
        Bot: "This document covers..."
```

## References

- [Direct Line API 3.0 Reference](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-api-reference) - Official API documentation with regional endpoints
- [Send Activity & Upload Attachments](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-send-activity) - Send activities and file uploads via DirectLine
- [About Direct Line Channel](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-channel-directline) - Channel overview and configuration
