# Data Model

MongoDB local via Docker in dev, MongoDB Atlas in production. Swap via environment variable. The code does not change.

**Fundamental principle:** evolving document. A single transaction goes through the phases raw → classified → enriched without duplicating data. Each phase writes its fields and does not overwrite what the previous phase wrote.

---

## Collections

| Collection | Responsibility |
|------------|----------------|
| `transactions` | Main document — full transaction lifecycle |
| `enricher_registry` | Available enrichers, priority, module and configuration |
| `aggregates` | Materialized totals by category, period and merchant |
| `reports` | LLM-generated reports with history |
| `categories` | User-editable category and subcategory taxonomy |
| `goals` | Financial goals by category and period |
| `budgets` | Monthly budgets by category with alerts |
| `users` | User profile and preferences |

---

## Collection: transactions

The document has four layers, each immutable after write:

**raw** — original source data, never changed after ingestion.
- `text`: raw text as it arrived at the connector
- `source`: file_connector, telegram, voice, manual, image, mercadopago
- `source_metadata`: filename + row for CSV, chat_id + message_id for Telegram, etc.
- `ingested_at`: ingestion timestamp

**classified** — LLM extraction, immutable after classification.
- `amount`: numeric value in BRL
- `currency`: BRL (extensible)
- `date`: ISO 8601
- `merchant_raw`: raw establishment name extrapolated by the LLM
- `category`: main classified category
- `classified_at`: classification timestamp

**enriched** — filled by the winning enricher. Null until an enricher resolves it.
- `merchant_normalized`: standardized merchant name
- `subcategory`: resolved subcategory
- `description`: product or service description
- `enricher`: name of the enricher that resolved it
- `enriched_at`: enrichment timestamp
- `metadata`: free schema per enricher

**enrichment_log** — append-only array of all attempts.
- `enricher`, `result` (hit/miss), `at`

**status** — lifecycle: `raw` → `classified` → `enriched` | `skipped` | `manual_pending`

### Status states

| Status | When | Next action |
|--------|------|-------------|
| `raw` | Just ingested, waiting for LLM | LLM Facade classifies |
| `classified` | LLM classified, gate decided to enrich | Enrichment Orchestrator processes |
| `enriched` | Enricher resolved | Change Stream triggers PFM Aggregator |
| `skipped` | Gate decided not to enrich | Change Stream triggers PFM Aggregator |
| `manual_pending` | No enricher resolved | Manual queue — user completes via Telegram |

---

## Indexes

| Index | Fields | Purpose |
|-------|--------|---------|
| Main query | `user_id` + `classified.date` (desc) | User spend in a period |
| Status | `user_id` + `status` | Enrichment Orchestrator looks for pending |
| Category | `user_id` + `classified.category` + `classified.date` | Aggregator aggregates by category |
| Deduplication | `user_id` + `amount` + `date` + `merchant_raw` (unique) | Prevents duplicate ingestion |

---

## Change Streams

The PFM Aggregator does not use an external queue. It watches the `transactions` collection and reacts when status changes to `enriched` or `skipped`.

Change Streams require MongoDB with a **replica set** even in the local environment.

---

## Collection: enricher_registry

- `name`: enricher name
- `priority`: execution order (1 = highest)
- `enabled`: bool — enable/disable without redeploy
- `module`: Python module path
- `config`: free object with enricher-specific settings

Enable new enricher = `insertOne`. Disable = `updateOne` on the `enabled` field.

---

## Local → Atlas migration

The only difference between local and production is the `MONGO_URI` variable:

- Local: `mongodb://localhost:27017/?replicaSet=rs0`
- Production: `mongodb+srv://user:pass@cluster.mongodb.net/harpy`

The code does not change.
