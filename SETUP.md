# OpenClaw Docker Setup

> 🔒 Never commit real tokens. Use placeholders in docs.

## Prerequisites

- Docker Desktop with Compose v2+
- Gemini API key
- Slack workspace admin access

---

## 1. Get API Keys

### Gemini API
1. Visit https://makersuite.google.com/app/apikey
2. Create API Key → Copy as `GEMINI_API_KEY`

### Slack App Setup
1. **Create App:** https://api.slack.com/apps → "From scratch"
2. **Socket Mode:** Settings → Socket Mode → Enable → Generate Token (`xapp-...`) → Save as `SLACK_APP_TOKEN`
3. **Bot Scopes:** OAuth & Permissions → Add scopes:
   - `app_mentions:read`, `channels:history`, `channels:read`, `chat:write`
   - `im:history`, `im:read`, `im:write`, `users:read`
4. **Install:** Click "Install to Workspace" → Copy Bot Token (`xoxb-...`) → Save as `SLACK_BOT_TOKEN`
5. **Events:** Event Subscriptions → Enable → Add: `app_mention`, `message.channels`, `message.im`
6. **App Home:** (Optional) Enable Messages Tab

---

## 2. Configure Environment

Create `.env`:
```bash
cp .env.example .env
```

Edit with your keys:
```bash
OPENCLAW_IMAGE=openclaw:local
GEMINI_API_KEY=<your-key>
SLACK_BOT_TOKEN=xoxb-<your-token>
SLACK_APP_TOKEN=xapp-<your-token>
```

Create `openclaw.json`:

```json
{
  "gateway": {
    "mode": "local",
    "bind": "lan",
    "port": 18789
  },
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": {
        "primary": "google/gemini-3-pro-preview"
      }
    },
    "list": [
      {
        "id": "main",
        "identity": {
          "name": "OpenClaw",
          "theme": "helpful AI assistant",
          "emoji": "🦞"
        }
      }
    ]
  },
  "channels": {
    "slack": {
      "enabled": true,
      "botToken": "${SLACK_BOT_TOKEN}",
      "appToken": "${SLACK_APP_TOKEN}",
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist"
    }
  },
  "env": {
    "vars": {
      "GEMINI_API_KEY": "${GEMINI_API_KEY}",
      "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
      "SLACK_APP_TOKEN": "${SLACK_APP_TOKEN}"
    }
  }
}
```

---

## 3. Deploy

```bash
# Build
docker build -f ./Dockerfile -t openclaw:local .

# Copy config
mkdir -p ~/.openclaw
cp openclaw.json ~/.openclaw/openclaw.json

# Start
docker compose up -d openclaw-gateway

# Get dashboard URL
docker compose run --rm openclaw-cli dashboard --no-open
```

---

## 4. Usage

**Slack Pairing:**
1. Message bot in Slack
2. Bot sends pairing code
3. Approve: `docker exec openclaw-openclaw-gateway-1 node dist/index.js pairing approve slack <CODE>`

**Invite to channels:** `/invite @bot-name`

---

## Quick Commands

```bash
# Start/Stop
docker compose up -d openclaw-gateway
docker compose down

# Logs
docker compose logs -f openclaw-gateway

# Pairing
docker exec openclaw-openclaw-gateway-1 node dist/index.js pairing approve slack <CODE>

# Dashboard
docker compose run --rm openclaw-cli dashboard --no-open
```

---

## Troubleshooting

**Container issues:** `docker compose logs openclaw-gateway`
**Slack not connecting:** Check tokens in `.env` and Socket Mode enabled
**Pairing fails:** Codes expire in 1 hour, request new one

---

**Docs:** https://docs.openclaw.ai | **Setup Complete!** 🦞
