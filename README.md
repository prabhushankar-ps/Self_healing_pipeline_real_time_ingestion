# Self-Healing Real-Time Yelp Review Sentiment Pipeline (Airflow + Kafka + Ollama)

A production-style, self-healing data/ML pipeline that ingests **real-time Yelp reviews**, curates them through a **Medallion (Bronze/Silver/Gold)** architecture, and performs **sentiment classification (positive/negative)** using an **Ollama-hosted LLM**. The system includes a **monitoring + remediation layer** to detect failures, isolate bad data, and restore healthy operation with minimal manual intervention.

---

## What this repository delivers

### Core capabilities
- **Real-time ingestion**: Kafka consumer processing Yelp review events.
- **Medallion architecture**:
  - **Bronze**: raw immutable events
  - **Silver**: cleaned + validated + deduplicated
  - **Gold**: analytics-ready, sentiment-enriched outputs
- **LLM inference**: sentiment classification via **Ollama** (Python SDK).
- **Self-healing pipeline**:
  - automated retries + backoff
  - circuit breaker around inference
  - DLQ for poison messages
  - replay/reprocess when downstream recovers (idempotent)
  - health checks to pinpoint where failures occur (ingestion, transform, inference, write)
- **Monitoring**: pipeline health, model call health, data quality, and remediation tracking.

---

## Architecture

```mermaid
flowchart LR
  A[Kafka Topic: yelp_reviews] --> B[Ingestion]
  B --> C[(Bronze Store)]
  C --> D[Validation + Cleaning]
  D --> E[(Silver Store)]
  E --> F[Gold Builder]
  F --> G[(Gold Store)]
  G --> H[Ollama Sentiment Inference]
  H --> I[(Sentiment Output Store)]

  B --> M[Monitoring + Self-Healing Controller]
  D --> M
  H --> M
  M --> N[Alerts / Dashboard / Logs]
  B -->|Poison messages| Q[(DLQ)]
  D -->|Bad schema / invalid| Q
  H -->|Timeout/invalid output| Q
