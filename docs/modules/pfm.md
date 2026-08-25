# PFM — Personal Financial Manager

## Mission

The PFM module is the heart of Harpy Aerie. Its mission is to capture any financial event — regardless of origin, format or channel — and turn it into structured, enriched data ready for analysis and decision.

**Central premise:** the user does not change behavior to feed the system. The system adapts to the user. If there is a receipt in email, PFM reads it. If a message is sent on Telegram, PFM understands it. If a bank CSV was exported, PFM processes it. Input is chaotic by design — resolving that is the system's responsibility, not the user's.

---

## Design Principles

- **Zero friction on input** — any format, any channel, no instructions to the user
- **Immediate persistence** — data is saved before any enrichment. Enrichment failure does not lose the transaction
- **Evolving document** — raw → classified → enriched. One single transaction, no JOINs, no data duplication
- **Pluggable enrichers** — a new enricher does not touch the core. Implements interface + registers in the database
- **Local-first MVP** — financial data never leaves the machine. Cloud migration changes 2 environment variables, not the code
- **LLM as infrastructure** — pipeline component, not the product. Abstracted via Facade, replaceable without rewrite
- **Event-driven first** — each step publishes/consumes events via SQS. Components do not call each other directly

---

## Input Sources

Each connector has a single responsibility: normalize the input and publish to the bus. From there the pipeline is identical regardless of origin.

| Connector | Input | Transformation |
|-----------|-------|----------------|
| File Connector | xlsx, csv, pdf | parse → lines → payload per line |
| Image Connector | receipt photo | LLM Vision → extract fields → payload |
| Telegram Connector | free-text messages | intent filter → payload if transaction |
| Voice Connector | audio | Whisper STT → text → payload |
| Manual Input | free text | direct → payload |
| Mercado Pago Connector | real-time API | webhook → payload |
| Nota Paulista Connector | portal CSV | parse → payload per line |
| Open Banking Connector | Open Finance Brazil API | automatic capture without file export |
| PIX Connector | PIX receipts from email or Telegram | extract payee, amount, PIX key |

---

## Pipeline

**Flow**

Input → Connector → SQS FIFO → Lambda Classifier → MongoDB PENDING → Enrichment Gate → SQS Enrichment → Chain of Responsibility → MongoDB ENRICHED → Change Stream → Aggregator → Report Generator

**Steps**

1. **Normalization** — Connector turns raw input into a canonical TransactionPayload: amount, date, raw text, source, source metadata, unique tracking id.
2. **LLM Classification** — Lambda Classifier uses LLM Facade to extract amount, date, merchant_raw, category, subcategory. Output is structured JSON. Document is created in MongoDB immediately after classification.
3. **Immediate persistence** — Document created with status PENDING before any enrichment attempt. Enrichment failure does not imply data loss.
4. **Enrichment Gate** — decides whether the transaction enters the enrichment queue based on: null merchant_normalized, empty subcategory, or amount above configurable threshold (default R$ 50).
5. **Chain of Responsibility** — enrichers run in decreasing confidence order. The first positive result ends the chain. Chain is built dynamically from the enricher_registry in MongoDB.
6. **Status resolution** — document updated to ENRICHED if an enricher resolved it, or MANUAL_REVIEW if none did. In the second case, a Telegram notification is sent with the option of a direct reply.
7. **Aggregation** — MongoDB Change Stream detects the status update without an extra queue between modules. Aggregator computes balances, categories, trends and anomalies.
8. **Report Generation** — LLM generates natural-language analysis based on aggregates. On demand or scheduled.

---

## Enrichment Chain

| Priority | Enricher | Source | How it resolves | Confidence |
|----------|----------|--------|-----------------|------------|
| 1 | Email Enricher | Gmail MCP | email with same amount + date | High — receipts are exact |
| 2 | WhatsApp Enricher | WhatsApp MCP | behavioral context from conversations | Medium |
| 3 | Mercado Livre | public API | orders by amount | Medium |
| 4 | Amazon | API and email | order history | Medium |
| 5 | iFood | API and email | order history | High for food |
| 6 | Steam / Google Play | public API | purchase history | High for games |
| 999 | Manual Queue | — | user resolves on Telegram | — |

**Invariants**

The enricher registry lives in the `enricher_registry` collection in MongoDB and is read by the orchestrator at startup. Enabling or disabling an enricher is an `updateOne` in the database, no redeploy. Adding a new enricher means implementing the `BaseEnricher` interface and registering the module in the collection. The core is not touched.

---

## Aggregation and Reports

**What the Aggregator computes**

- Total balance by period (daily, weekly, monthly, yearly)
- Spend by category and subcategory with historical evolution
- Spend by merchant with ranking and trend
- Moving average of spend by category (last 3 and 6 months)
- Anomaly detection when a category exceeds historical standard deviation
- Month-end projection based on current pace
- Month-over-month comparison by category
- Fixed vs variable spend identification
- Recurring subscription identification
- Actual spend vs goal/budget comparison
- Monthly financial health score

**What the Report Generator produces**

- Automatic weekly report (every Sunday via EventBridge)
- Automatic monthly report (first of the month via EventBridge)
- Ad-hoc query — user asks on Telegram and gets the answer
- Proactive alert — PFM detects anomaly and notifies without the user asking
- Cut suggestion — based on historical pattern and category variance
- Future spend forecast by category with a simple trend model

---

## Where it runs

### MVP — local

Everything on the user's machine. Financial data never leaves the machine.

- MongoDB with replica set via Docker (Change Stream requires replica set even locally)
- SQS FIFO and S3 via LocalStack in Docker
- Lambda Handlers running locally via SAM CLI or as direct Python scripts
- LLM via Ollama

### Production — AWS

Two endpoints change. The code does not change.

- MongoDB Atlas (hosted on AWS)
- AWS SQS FIFO
- AWS Lambda with Python 3.12
- AWS Bedrock as LLM provider
- AWS S3 for CSV and PDF file staging
- AWS Secrets Manager for credentials
- AWS SSM Parameter Store for configuration
- AWS EventBridge for report scheduling
