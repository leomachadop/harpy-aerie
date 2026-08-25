# OPS — Personal Organization

## Mission

The OPS module is the nervous system of Harpy Aerie. Its mission is to capture any intention signal — a Telegram message, an email with a deadline, a voice note, a document — classify what is actionable, create the right tasks in the right places, and deliver a consolidated action plan to the user every day.

**Central premise:** the user does not need to manage the task system. They speak, write or forward — and OPS creates, organizes and prioritizes. The interface is Telegram. The source of truth for tasks is Todoist. OPS is the intelligent middleware between the two.

---

## Design Principles

- **Telegram as input and output hub** — text, voice, photo, document. Everything enters through the same channel. The daily plan also exits through the same channel
- **LLM as intention classifier** — not regex, not keyword heuristics. The LLM reads the message in context and decides: task, appointment, deadline or noise
- **Todoist as source of truth** — OPS does not maintain its own task list. It writes to Todoist, and the user manages from there with the apps they already use
- **Gmail as passive reading** — not interactive. Periodically scans the inbox for emails with deadlines, appointments and action signals
- **Event-driven first** — each step publishes/consumes events via SQS. No direct calls between components
- **Daily Planner with cross context** — the day plan aggregates Todoist tasks + day signals + financial context from PFM. It is not a task list — it is a situational analysis

---

## Input Sources

| Source | Role |
|--------|------|
| Telegram Bot (text) | Direct user message. Each message goes through the Signal Router to decide OPS, PFM or both |
| Telegram Bot (voice) | Voice note converted to text via Whisper STT then processed as text |
| Telegram Bot (photo) | Photo of document, ticket or note. LLM Vision extracts text and identifies tasks, deadlines and appointments |
| Telegram Bot (document) | CSV, PDF, DOCX forwarded by the user. Automatic upload to S3 and routing to the appropriate connector |
| Gmail MCP (passive scan) | Scheduled Lambda scans inbox for deadlines, meeting invites, bill notifications, follow-ups |
| WhatsApp MCP (passive) | Scan recent conversations for commitments made with contacts |
| Slack / Google Calendar / Apple Reminders | Desired / backlog integrations |

---

## Pipeline

**Flow**

Input → Adapter → Signal Router (LLM) → SQS FIFO (ops-signal-raw) → Lambda Signal Processor → Normalizer → LLM Intent Classifier → Enrichment Gate → SQS FIFO (ops-signal-processed) → Task Writer (Todoist + MongoDB) → EventBridge → Daily Planner → Telegram

### Signal Router

Classifies general intention before any processing:

- **OPS** — task, appointment, deadline, reminder
- **PFM** — financial transaction, receipt, expense
- **Both** — the same message can generate events for both modules
- **Noise** — no actionable intention; logged for context but discarded from the active pipeline

Examples:

- "Remind me to pay the condo bill tomorrow" → OPS
- "Your order #MLB-123 was confirmed — R$ 347" → PFM
- "Team meeting at 3pm tomorrow" → OPS
- "I paid the rent, need the invoice" → PFM + OPS

### Task Writer

Consumes the processed queue and executes two writes in parallel:

- Creates the task in Todoist via REST API (content, due_string, priority, project_id, labels, description)
- Writes the document in MongoDB with `todoist_task_id` and status `synced`

### Daily Planner

Lambda triggered via EventBridge at a configurable fixed time. Aggregates three sources:

- Todoist API: overdue tasks + due today + relevant backlog
- MongoDB ops_signals: current day signals with status synced
- MongoDB transactions (PFM): relevant recent expenses, bills due, uncategorized transactions

LLM generates the plan in natural language and sends it via Telegram Bot. Stored in `ops_daily_plans` in MongoDB.

---

## Outputs

- **Daily plan via Telegram** — delivered at the configured time (default 07:00). Includes day priorities, relevant backlog, financial context crossed with PFM
- **Proactive alerts** — notification when a high-priority task is due in less than X hours without completion
- **Creation confirmation** — Telegram replies confirming the created task
- **Weekly review (desired)** — LLM analyzes the week: completed tasks, accumulated pending, procrastination patterns
- **Nightly summary (desired)** — at 22:00, brief day analysis
- **Backlog analysis (desired)** — identification of tasks that stay in backlog without completion

---

## Where it runs

### MVP — local

- MongoDB via Docker with replica set
- SQS via LocalStack
- Lambda Handlers as Python scripts via SAM CLI
- LLM via Ollama
- EventBridge simulated via local cron
- Telegram Bot against the official Bot API

### Production — AWS

- MongoDB Atlas
- AWS SQS FIFO
- AWS Lambda Python 3.12
- AWS Bedrock
- AWS EventBridge for scheduling
- AWS Secrets Manager for credentials
- AWS SSM Parameter Store for planner schedule and thresholds
