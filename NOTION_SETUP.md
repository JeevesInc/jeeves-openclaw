# 🦞 OpenClaw Docker Setup - Jeeves Internal

> 💡 **Quick Setup Guide** - Get OpenClaw running with Docker, Gemini AI, and Slack integration

---

## 📋 Prerequisites

Before starting, ensure you have:

- ✅ Docker Desktop with Compose v2+
- ✅ Admin access to Google Cloud (for Gemini)
- ✅ Admin access to Slack workspace

---

## 🔑 Step 1: API Keys Setup

### Google Gemini API Key

1. Navigate to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Click **"Create API Key"**
3. Copy the key and save as `GEMINI_API_KEY`

---

### Slack Bot Configuration

**🔗 Start here:** [Slack API Apps](https://api.slack.com/apps)

| Step | Action | Details |
|------|--------|---------|
| 1️⃣ | **Create App** | "From scratch" → Name it → Select workspace |
| 2️⃣ | **Socket Mode** | Settings → Socket Mode → Enable → Generate token with `connections:write` scope |
| 3️⃣ | **Bot Scopes** | OAuth & Permissions → Add these scopes ⬇️ |
| 4️⃣ | **Install** | "Install to Workspace" → Copy Bot Token (`xoxb-...`) |
| 5️⃣ | **Events** | Event Subscriptions → Enable → Add events ⬇️ |

**Required Bot Scopes:**
```
app_mentions:read
channels:history
channels:read
chat:write
im:history
im:read
im:write
users:read
```

**Required Events:**
```
app_mention
message.channels
message.im
```

> 💡 **Save these tokens:**
> - `SLACK_BOT_TOKEN` → starts with `xoxb-`
> - `SLACK_APP_TOKEN` → starts with `xapp-`

---

## ⚙️ Step 2: Configuration

### Create Environment File

```bash
# Copy template
cp .env.example .env
```

**Edit `.env` with your credentials:**

```bash
OPENCLAW_IMAGE=openclaw:local
GEMINI_API_KEY=<your-gemini-key>
SLACK_BOT_TOKEN=xoxb-<your-token>
SLACK_APP_TOKEN=xapp-<your-token>
```

---

### Create OpenClaw Config

**Save as `openclaw.json` in repo root:**

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

## 🚀 Step 3: Deployment

### One-Time Setup

```bash
# 1. Build Docker image (~2-3 minutes)
docker build -f ./Dockerfile -t openclaw:local .

# 2. Copy configuration
mkdir -p ~/.openclaw
cp openclaw.json ~/.openclaw/openclaw.json

# 3. Start gateway
docker compose up -d openclaw-gateway

# 4. Get dashboard URL
docker compose run --rm openclaw-cli dashboard --no-open
```

> ✅ **Success:** Gateway is now running on `http://127.0.0.1:18789`

---

## 💬 Step 4: Slack Integration

### First User Setup (Pairing)

1. **User messages bot** in Slack DM
2. **Bot responds** with pairing code (e.g., `ABC12DEF`)
3. **Approve pairing:**

```bash
docker exec openclaw-openclaw-gateway-1 node dist/index.js pairing approve slack <CODE>
```

### Add Bot to Channels

In Slack channel:
```
/invite @your-bot-name
```

---

## 🛠️ Common Operations

### Daily Commands

| Task | Command |
|------|---------|
| **Start Gateway** | `docker compose up -d openclaw-gateway` |
| **Stop Gateway** | `docker compose down` |
| **View Logs** | `docker compose logs -f openclaw-gateway` |
| **Restart** | `docker compose restart openclaw-gateway` |
| **Check Status** | `docker compose ps` |

### Pairing Management

```bash
# List pending pairing requests
docker compose run --rm openclaw-cli pairing list

# Approve user
docker exec openclaw-openclaw-gateway-1 node dist/index.js pairing approve slack <CODE>
```

### Access Dashboard

```bash
# Get dashboard URL with auth token
docker compose run --rm openclaw-cli dashboard --no-open
```

---

## 🐛 Troubleshooting

### Gateway Won't Start

```bash
# Check logs
docker compose logs openclaw-gateway

# Common fixes:
# - Check .env has all required keys
# - Verify no port conflicts on 18789
# - Restart Docker Desktop
```

### Slack Not Connecting

**Checklist:**
- [ ] Socket Mode enabled in Slack app settings
- [ ] Both tokens (`xoxb-` and `xapp-`) are correct
- [ ] All required scopes are added
- [ ] Bot is installed to workspace

```bash
# Verify tokens
cat .env | grep SLACK

# Check connection logs
docker compose logs openclaw-gateway | grep -i slack
```

### Pairing Code Issues

- ⏰ Codes expire after **1 hour**
- 📊 Max **3 pending** requests per channel
- 🔄 Request new code: message bot again

---

## 🔒 Security Notes

### Protected Files

| File | Status | Note |
|------|--------|------|
| `.env` | 🔴 Private | In `.gitignore`, never commit |
| `openclaw.json` | 🟡 Internal | Contains token references |
| `.env.example` | 🟢 Public | Safe template file |

### Token Management

**If tokens are exposed:**

1. **Slack:** Regenerate in app settings
2. **Gemini:** Delete key in AI Studio → Create new
3. Update `.env` and restart gateway

---

## 📁 Repository Structure

```
jeeves-openclaw/
├── .env                    # 🔴 Your credentials (gitignored)
├── .env.example            # 🟢 Safe template
├── openclaw.json           # ⚙️ Bot configuration
├── docker-compose.yml      # 🐳 Docker services
├── Dockerfile              # 🐳 Image definition
└── SETUP.md               # 📖 This guide
```

---

## 🔗 Quick Links

| Resource | URL |
|----------|-----|
| **OpenClaw Docs** | https://docs.openclaw.ai |
| **Docker Guide** | https://docs.openclaw.ai/install/docker |
| **Slack Integration** | https://docs.openclaw.ai/channels/slack |
| **GitHub (Original)** | https://github.com/openclaw/openclaw |
| **Jeeves Fork** | https://github.com/JeevesInc/jeeves-openclaw |

---

## ✅ Setup Checklist

Use this to verify your setup:

- [ ] Gemini API key obtained
- [ ] Slack app created with Socket Mode
- [ ] All bot scopes added
- [ ] Event subscriptions configured
- [ ] `.env` file created with all tokens
- [ ] `openclaw.json` created
- [ ] Docker image built
- [ ] Gateway started successfully
- [ ] Dashboard URL obtained
- [ ] First user paired successfully
- [ ] Bot invited to test channel

---

## 👥 Team Notes

**Maintainer:** Jeeves Inc Internal Team
**Last Updated:** February 2026
**Support:** Contact DevOps team for deployment issues

> 🎉 **You're all set!** OpenClaw is now running with Gemini and integrated with Slack.

---

*This is an internal Jeeves documentation. Do not share externally.*
