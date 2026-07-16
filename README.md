# NOC Bot

An automated alert-response bot for operations teams (NOC) built on n8n. It monitors a Slack channel for incoming alerts, enriches them with observability data from Datadog, and generates AI-powered analysis using Claude — responding directly in the alert thread.

---

## What it does

The bot runs on a schedule and executes the following flow automatically:

1. **Slack reading** — Reads the NOC channel history looking for unanswered alerts.
2. **Alert parsing** — Identifies and structures active alerts into clean data objects.
3. **Datadog enrichment** — Queries in parallel:
   - Metrics from the last 30 minutes
   - Logs from the last 15 minutes
   - Events from the last hour
4. **Slack correlation** — Searches for related messages in the last hour.
5. **AWS Health** *(optional)* — Checks the status of AWS services.
6. **Claude AI analysis** — Sends all enriched context to Claude to generate a diagnosis and recommended action.
7. **Thread reply** — Posts the analysis directly in the original alert thread on Slack.
8. **Google Sheets logging** — Records every processed alert for auditing and metrics.

---

## Architecture

See [`docs/architecture.md`](docs/architecture.md) for the detailed flow diagram.

---

## Tech stack

- **n8n** — Workflow orchestration (self-hosted or cloud)
- **Slack API** — Alert ingestion and response
- **Datadog API** — Metrics, logs, and event enrichment
- **Anthropic Claude API** — AI-powered root cause analysis
- **Google Sheets API** — Audit logging
- **AWS Health API** *(optional)* — Infrastructure health context

---

## Installation

### Prerequisites

- [n8n](https://n8n.io/) (self-hosted or cloud)
- Datadog account with API access
- Slack bot with read/write permissions on the NOC channel
- Google Sheets API enabled
- Anthropic API key (Claude)

### Steps

1. Clone this repository:
   ```bash
   git clone https://github.com/juansandoval-arch/noc-bot.git
   cd noc-bot
   ```

2. Copy the environment variables template and fill it in:
   ```bash
   cp .env.example .env
   # Edit .env with your real credentials
   ```

3. Import the workflow into n8n:
   - Go to **Workflows → Import from file**
   - Select `workflows/NOC_bot.json`

4. Configure credentials in n8n:
   - **Slack**: OAuth2 or Bot Token with `channels:history`, `chat:write` scopes
   - **Datadog**: API Key + Application Key
   - **Google Sheets**: Service Account or OAuth2
   - **Anthropic (Claude)**: API Key via HTTP Header Auth

5. Activate the workflow.

---

## Environment variables

| Variable | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key for Claude |
| `DD_API_KEY` | Datadog API key |
| `DD_APP_KEY` | Datadog Application key |
| `DD_SITE` | Datadog site (e.g. `datadoghq.com`) |
| `SLACK_CHANNEL_ID` | ID of the Slack channel to monitor |
| `GOOGLE_SHEET_ID` | ID of the Google Sheet for alert logging |

---

## Repository structure

```
noc-bot/
├── workflows/
│   └── NOC_bot.json      # n8n workflow (import directly)
├── docs/
│   └── architecture.md   # Flow diagram and architecture description
├── .env.example          # Environment variables template
├── .gitignore
└── README.md
```

---

## Use case

Built for operations teams that receive high volumes of infrastructure alerts in Slack and need to triage them quickly. The bot eliminates manual enrichment — instead of an engineer spending 8+ minutes gathering metrics, logs, and context, the bot does it in seconds and presents a structured AI diagnosis in the alert thread.

**Result:** Near-zero manual response time for routine alerts. Engineers focus only on alerts that require human judgment.
