# Harpy Aerie

**Autonomous Financial Planner with behavioral architecture**

Harpy Aerie is a personal financial system designed for people who have emotional aversion to money. It does not educate. It acts.

The system comes to the user (Telegram), captures any financial signal, classifies and enriches it, and delivers analysis in natural language — without dashboards that need to be opened and without emotional friction.

> **Core principle:** It is not a spending control app. It is an intelligent personal financial planner built for people who feel aversion when looking at their money — not for those who already have financial discipline.

---

## Why it exists

Most people do not have an information problem. They have an **emotional aversion to money**. Traditional apps make this worse because money hurts exactly when you open them.

Harpy Aerie inverts the equation:
- Zero friction input (Telegram, email, CSV, photo, voice)
- LLM classifies intention and routes to the correct module
- Behavioral architecture (Kahneman, COM-B, EAST, MINDSPACE)
- Action and commitment devices instead of financial education

---

## Modules

| Module | Responsibility |
|--------|----------------|
| **PFM** — Personal Financial Manager | Capture, classify, enrich and analyze any financial event |
| **OPS** — Personal Organization | Capture actionable signals and keep Todoist as the source of truth |
| **FPL** — Financial Planner | Decision engine (portfolio recommendations with mandatory human confirmation) |
| **Market Analysis** | Macro context (BCB, IBGE, FEBRABAN) |
| **Investments** | B3 and Tesouro Direto tracking |

A **Signal Router** (LLM) classifies every incoming message and routes it to the correct module(s).

---

## Stack

- **Python** — orchestration, workers, pipeline
- **MongoDB** (Docker local → Atlas) — evolving document model
- **SQS FIFO** — event bus between modules
- **S3** — file staging (CSV, PDF, images)
- **Lambda (Python 3.12)** — workers
- **EventBridge** — scheduling (Daily Planner, reports)
- **Ollama** (local) / **AWS Bedrock** (prod) — LLM via Facade (swap by env var, zero rewrite)
- **Telegram Bot API** — primary interactive channel
- **Pluggy** — Open Finance Brazil
- **Todoist** — task source of truth (OPS)
- **Metabase** — self-hosted dashboard over MongoDB

**Local-first MVP:** financial data never leaves the machine. Production migration changes two environment variables. Zero code rewrite.

---

## Documentation

Full documentation lives under [`docs/`](docs/):

- [Vision & Behavioral Foundation](docs/vision.md)
- [Architecture Overview](docs/architecture/overview.md)
- [PFM Module](docs/modules/pfm.md)
- [OPS Module](docs/modules/ops.md)
- [FPL Module](docs/modules/fpl.md)
- [Data Model](docs/data-model.md)
- [Infrastructure](docs/infrastructure.md)

---

## Status

Architecture and detailed design are complete. Implementation of the core PFM pipeline is the current priority.

---

## License

Private project.
