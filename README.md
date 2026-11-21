# AI Sales Automation Pipeline

> **Multi-agent autonomous pipeline that prospects, diagnoses and closes the sales cycle — from Google Maps to WhatsApp, with no human intervention before final approval.**

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=node.js&logoColor=white"/>
  <img src="https://img.shields.io/badge/TypeScript-5.4-3178C6?style=for-the-badge&logo=typescript&logoColor=white"/>
  <img src="https://img.shields.io/badge/Claude_AI-Anthropic-D97706?style=for-the-badge&logo=anthropic&logoColor=white"/>
  <img src="https://img.shields.io/badge/WhatsApp-API-25D366?style=for-the-badge&logo=whatsapp&logoColor=white"/>
  <img src="https://img.shields.io/badge/Gmail-OAuth_2.0-EA4335?style=for-the-badge&logo=gmail&logoColor=white"/>
  <img src="https://img.shields.io/badge/PM2-Process_Manager-2B037A?style=for-the-badge&logo=pm2&logoColor=white"/>
</p>

---

## What it is

B2B prospecting automation system built as a multi-agent architecture reference. Runs automatically at 08h, executes the entire sales funnel and delivers results to an approval dashboard — the salesperson only decides what to send.

Each agent has a single responsibility, well-defined input and output, and can be replaced or scaled independently.

---

## Architecture — 7-Agent Pipeline

```
  Google Maps
      │
      ▼
  ╔═══════════╗   Searches companies by industry and city
  ║  AGENT 1  ║   Filters by ratings and sales potential
  ║ Prospector║   AI-based priority score
  ╚═════╤═════╝
        │ leads.json
        ▼
  ╔═══════════╗   Analyzes the business with Claude AI
  ║  AGENT 2  ║   Identifies weaknesses and opportunities
  ║ Diagnosis ║   Calculates score 0–100
  ╚═════╤═════╝
        │ diagnostics.json
        ▼
  ╔═══════════╗   Generates a personalized HTML landing page
  ║  AGENT 3  ║   based on the business diagnosis
  ║  Builder  ║   Automatic deploy via Surge.sh
  ╚═════╤═════╝
        │ pages/{slug}/index.html
        ▼
  ╔═══════════╗   Captures mobile video of the landing page
  ║  AGENT 4  ║   via headless Puppeteer
  ║   Video   ║
  ╚═════╤═════╝
        │
        ▼
  ╔═══════════╗   Generates personalized prospecting message
  ║  AGENT 5  ║   referencing diagnosis and landing page
  ║  Channel  ║   adapts tone by industry
  ╚═════╤═════╝
        │ messages.json
        ▼
  ╔═══════════╗   AI review — quality score
  ║  AGENT 6  ║   Removes generic AI patterns
  ║  Reviewer ║   Rewrites if score < 80
  ╚═════╤═════╝
        │ approved message
        ▼
  ┌──────────────────────────────────────┐
  │          APPROVAL DASHBOARD          │  ← only manual step
  │  Landing page preview                │
  │  Quality score · history             │
  └─────────────────┬────────────────────┘
                    │ approved
                    ▼
  ╔═══════════╗   Sends via WhatsApp or Gmail
  ║  AGENT 7  ║   Monitors replies 24/7
  ║  Handler  ║   Responds to leads automatically with AI
  ╚═══════════╝
```

---

## Design Decisions

**Why 7 separate agents instead of a single orchestrator?** Each agent has its own failure cycle, latency and state. Separating them allows restarting a single stage, monitoring per-step metrics and replacing one agent's implementation without touching the others.

**Why PM2 instead of a job scheduler?** The system uses `node-cron` for daily scheduling + PM2 for auto-restart and boot on server startup. The combination eliminates the need for an external queue server for a single pipeline.

**Deliberate human-in-the-loop**: the approval dashboard is not a workaround — it is part of the architecture. Automation without review generates spam. Manual approval before sending is the only step that cannot safely be automated in this domain.

---

## Stack

| Layer | Technology |
|---|---|
| Language | TypeScript / Node.js 18+ |
| AI | Anthropic Claude API |
| Prospecting | Google Places API |
| WhatsApp | whatsapp-web.js |
| Email | Gmail API (OAuth 2.0) |
| Landing Pages | AI-generated HTML + Surge.sh |
| Screenshots/Video | Puppeteer |
| Server | Express.js |
| Processes | PM2 (auto-restart + boot) |
| Scheduling | node-cron |

---

## Structure

```
sales-automation/
├── agents/
│   ├── agent1_prospector.ts    # Google Maps → leads
│   ├── agent2_diagnostico.ts   # Claude AI → diagnosis + score
│   ├── agent3_builder.ts       # personalized HTML + deploy
│   ├── agent4_video.ts         # headless mobile capture
│   ├── agent5_canal.ts         # prospecting message
│   ├── agent6_revisor.ts       # AI quality review
│   └── agent7_handler.ts       # send + automatic replies
├── api/
│   └── server.ts               # Express API — approval dashboard
├── aprovacao.html              # web dashboard
├── orchestrator.ts             # daily cron — coordinates agents
└── ecosystem.config.js         # PM2 config
```

---

## Setup

```bash
git clone https://github.com/victor-germano/agente-de-automacao.git
cd agente-de-automacao
npm install
cp .env.example .env   # fill in the keys
```

**Configure Gmail OAuth:**
```bash
npx ts-node scripts/gerar-oauth.ts
```

**Start with PM2:**
```bash
pm2 start ecosystem.config.js
pm2 save
# dashboard: http://localhost:3000
```

**Required environment variables:**
```env
ANTHROPIC_API_KEY=
GOOGLE_PLACES_API_KEY=
OWNER_WHATSAPP=5511999999999
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REFRESH_TOKEN=
```

---

## Monitoring

```bash
pm2 status       # running processes
pm2 logs         # real-time logs
pm2 monit        # interactive dashboard
```

---

## License

MIT
