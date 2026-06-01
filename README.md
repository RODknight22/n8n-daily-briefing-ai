# 📰 n8n Daily Briefing AI

> Automated morning newsletter with **Cybersecurity, AI & Finance** news — runs Mon–Fri at 9:00 AM (Mexico City time), summarized by Google Gemini and delivered via SMTP.

![n8n](https://img.shields.io/badge/n8n-workflow-orange?logo=n8n) ![Gemini](https://img.shields.io/badge/Google_Gemini-free_tier-blue?logo=google) ![License](https://img.shields.io/badge/license-MIT-green)

---

## 📌 What it does

Every weekday morning this workflow:

1. Pulls fresh articles from **6 RSS feeds** in parallel
2. Labels each article by category (`Cybersecurity` / `AI` / `Finance`)
3. Sends the labeled batch to **Google Gemini 2.5 Flash** (free tier)
4. Gemini compiles a clean, structured **HTML newsletter in Spanish** — preserving English tech terms like `malware`, `zero-day`, `ETFs`, `blockchain`, etc.
5. Delivers the styled email via **SMTP** — no OAuth, no redirects, perfect for Home Labs

---

## ✨ Features

| Category | Sources |
|---|---|
| 🔐 Cybersecurity | BleepingComputer, Wired Security |
| 🤖 AI | TechCrunch AI, VentureBeat |
| 📈 Finance & Crypto | MarketWatch, CoinTelegraph |

- **Home Lab friendly** — SMTP instead of OAuth2 (no callback URL issues on private IPs)
- **Zero cost** — Google Gemini 2.5 Flash free tier (1,500 requests/day)
- **Bilingual output** — Spanish prose, English technical terms preserved
- **Compact format** — 3 headlines per section, one sentence each

---

## 📋 Prerequisites

- **n8n** self-hosted instance (Docker, npm, or cloud)
- **Google Account** with a [Gemini API Key](https://aistudio.google.com/app/apikey) (free)
- **Gmail account** with [2-Step Verification](https://myaccount.google.com/security) enabled + an [App Password](https://myaccount.google.com/apppasswords)

---

## 🚀 Deployment Guide

### 1. Import the workflow

1. Open your n8n instance
2. Go to **Workflows → Add Workflow → Import from File**  
   *(or press `Ctrl+V` on an empty canvas to paste JSON directly)*
3. Upload / paste the contents of `workflow.json`

### 2. Set the Timezone

> **Settings → n8n settings → Timezone → `America/Mexico_City` → Save**

Or set it at the Docker container level (recommended):

```yaml
environment:
  - GENERIC_TIMEZONE=America/Mexico_City
  - TZ=America/Mexico_City
```

### 3. Create the Google Gemini credential

1. Visit [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey) and generate a free API key
2. In n8n: **Credentials → New → search `Google PaLm API`**
3. Paste your API key and save as `Google Gemini API`
4. Open the **"Google Gemini 2.5 Flash"** subnode and link this credential

### 4. Create the SMTP credential

1. Go to [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
2. Create a new App Password — name it `n8n SMTP`
3. Copy the 16-character password (no spaces when pasting)
4. In n8n: **Credentials → New → search `SMTP`** and fill in:

| Field | Value |
|---|---|
| **Host** | `smtp.gmail.com` |
| **Port** | `465` |
| **SSL/TLS** | `SSL/TLS` |
| **User** | `your-sender@gmail.com` |
| **Password** | The 16-char App Password |
| **Client Host Name** | `localhost` |

5. Open the **"Enviar Boletin por SMTP"** node and:
   - Set `fromEmail` → your sender Gmail address
   - Set `toEmail` → your recipient address
   - Link the SMTP credential

### 5. Activate

Toggle **Inactive → Active** in the top-right corner. Done — the workflow runs automatically every weekday at 9:00 AM.

---

## 🛠️ Optimization Tips (Home Lab)

Add these environment variables to your n8n Docker container to prevent execution history from filling your disk:

```env
EXECUTIONS_DATA_PRUNE=true
EXECUTIONS_DATA_MAX_AGE=168
EXECUTIONS_DATA_PRUNE_MAX_COUNT=500
```

Full `docker-compose.yml` snippet:

```yaml
services:
  n8n:
    image: n8nio/n8n
    environment:
      - GENERIC_TIMEZONE=America/Mexico_City
      - TZ=America/Mexico_City
      - EXECUTIONS_DATA_PRUNE=true
      - EXECUTIONS_DATA_MAX_AGE=168
      - EXECUTIONS_DATA_PRUNE_MAX_COUNT=500
    volumes:
      - n8n_data:/home/node/.n8n
    ports:
      - "5678:5678"
```

> 💡 Setting `GENERIC_TIMEZONE` at the container level means you skip the timezone config in n8n's UI entirely.

---

## 🗂️ Workflow Architecture

```
[Schedule Trigger: Mon–Fri 9AM]
         |
   +-----+-----+----------+-------------+------------+---------------+
   ↓           ↓          ↓             ↓            ↓               ↓
[BleepingCmp] [Wired] [TechCrunch] [VentureBeat] [MarketWatch] [CoinTelegraph]
   +-----+-----+----------+-------------+------------+---------------+
         |
   [Merge: Append x6]  ← waits for all 6 feeds
         |
   [Code: Label & slice 4 articles per feed → 24 total]
         |
   [AI Agent] ← [Google Gemini 2.5 Flash subnode]
         |
   [Send Email via SMTP]
```

---

## 🔄 Customization

**Change the schedule** — Edit the Schedule Trigger cron. Default: `0 9 * * 1-5`

**Add more RSS sources** — Duplicate an RSS node, connect it to the Merge node, increase `numberInputs`, and add the new category to the `cats` array in the Code node.

**Swap the AI model** — Replace the Gemini subnode with any LangChain-compatible model: OpenAI GPT, Anthropic Claude, or a local Ollama model for fully offline operation.

**Change output language** — Update the system prompt in the AI Agent node.

---

## 📄 License

MIT — free to use, modify and share.

---

*Built with [n8n](https://n8n.io) + [Google Gemini](https://ai.google.dev) • Made for the Home Lab community*
