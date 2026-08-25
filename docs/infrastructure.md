# Infrastructure

## Component mapping

| Component | Service / Technology | Local (dev) | Production |
|-----------|----------------------|-------------|------------|
| Transaction storage | MongoDB | Docker local (replica set) | MongoDB Atlas (AWS) |
| Message Bus | SQS FIFO | LocalStack | AWS real |
| File staging (CSV/PDF) | S3 | LocalStack | AWS real |
| Workers | Lambda Python 3.12 | SAM CLI local | AWS real |
| Post-enrichment notification | MongoDB Change Stream | Docker local | Atlas |
| LLM config and registry | SSM Parameter Store | AWS real | AWS real |
| MCP and API credentials | Secrets Manager | AWS real | AWS real |
| Report scheduler | EventBridge | local cron | AWS real |
| Local LLM | Ollama | host.docker.internal:11434 | — |
| Production LLM | AWS Bedrock | — | Swap via SSM |

---

## SQS — Queues

| Queue | Type | Producer | Consumer | Retention |
|-------|------|----------|----------|-----------|
| `harpy-transaction-raw.fifo` | FIFO | Connectors | Lambda Ingestion Service | 4h |
| `harpy-transaction-enrich.fifo` | FIFO | Ingestion Service (gate) | Lambda Enrichment Orchestrator | 12h |
| `harpy-transaction-manual.fifo` | FIFO | Enrichment Orchestrator (fallback) | Manual process via Telegram | 7 days |

There is **no** `harpy-transaction-enriched` queue — the PFM Aggregator is triggered via MongoDB Change Stream directly, without an intermediate queue.

---

## Lambda — Functions

| Function | Trigger | Responsibility | Timeout |
|----------|---------|----------------|---------|
| `harpy-file-connector` | S3 Event | parse CSV and XLSX → publish to SQS raw | 30s |
| `harpy-ingestion-service` | SQS raw | normalize + LLM classify + enrichment gate | 60s |
| `harpy-enrichment-orchestrator` | SQS enrich | run enricher chain → UpdateOne in MongoDB | 120s |
| `harpy-pfm-aggregator` | MongoDB Change Stream | aggregate data + generate LLM report | 60s |

---

## SSM Parameter Store — Configuration

| Parameter | Default (local) | Default (prod) |
|-----------|-----------------|----------------|
| `/harpy/llm/provider` | ollama | bedrock |
| `/harpy/llm/ollama/host` | http://host.docker.internal:11434 | — |
| `/harpy/llm/ollama/model` | llama3.2 | — |
| `/harpy/llm/bedrock/model_id` | — | anthropic.claude-3-haiku-20240307-v1:0 |
| `/harpy/enrichment/threshold_amount` | 50.0 | 50.0 |

Enricher registry lives in the MongoDB `enricher_registry` collection — not in SSM. Enable/disable enricher = `updateOne`, zero redeploy.

---

## Secrets Manager — Credentials

| Secret | Content |
|--------|---------|
| `harpy/gmail-mcp-token` | Gmail MCP token for Email Enricher |
| `harpy/whatsapp-mcp-token` | WhatsApp MCP token for WhatsApp Enricher |
| `harpy/mercadolivre-api-key` | Mercado Livre API key |
| `harpy/ifood-api-key` | iFood API key |
| `harpy/mongodb-uri` | Empty locally, Atlas URI in prod |
| `harpy/telegram-bot-token` | Telegram Bot token |
| `harpy/steam-api-key` | Steam API key |
| `harpy/google-play-credentials` | Google Play API credentials |

---

## Local setup — docker-compose

MongoDB must run as a replica set even in the local environment because MongoDB Change Stream does not work in standalone mode. docker-compose starts MongoDB with the `--replSet rs0` flag and runs the replica set initialization script automatically via `docker-entrypoint-initdb.d`.

Services:

- **mongodb**: mongo:7 with `--replSet rs0`, replica set initialized via `rs-init.js`, healthcheck via mongosh ping, persistent volume on `mongo_data`
- **localstack**: simulates SQS and S3 locally with the same queue and bucket names as production
- **ollama**: local LLM on `host.docker.internal:11434` (or running directly on the host)

---

## Local → Atlas migration (zero diff)

The `MONGO_URI` variable is the only difference between local and production environments.

- Local: `mongodb://localhost:27017/?replicaSet=rs0`
- Production: `mongodb+srv://user:pass@cluster.mongodb.net/harpy`

Switching from local to Atlas = one environment variable. Zero code rewrite. Same logic for the LLM provider via `/harpy/llm/provider` in SSM.

---

## Technical decisions

| Decision | Choice | Justification |
|----------|--------|---------------|
| Storage | MongoDB Docker local → Atlas AWS | No LocalStack for MongoDB; Change Stream requires replica set |
| Message Bus | SQS FIFO via LocalStack | Decouples producers from consumers; LocalStack locally |
| Post-enrichment notification | MongoDB Change Stream | No extra queue between internal modules |
| Local LLM | Ollama via LLM Facade | Sensitive data never leaves the machine |
| Production LLM | AWS Bedrock | Swap via SSM — zero rewrite |
| File staging | S3 + S3 Event → SQS | Decouples upload from processing; supports retry |
| Enricher registry | MongoDB `enricher_registry` collection | Enable/disable without redeploy |
| Lambda runtime | Python 3.12 | Native async, asyncio support for parallel enrichers |
