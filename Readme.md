# RealRecon — Real-time Transaction Reconciliation Engine

**Detect mismatches, auto-resolve common errors, and get full observability of payments — in seconds, not hours.**

RealRecon is an event-driven reconciliation engine designed for modern payment systems (UPI, cards, wallets). It provides stream ingestion, deterministic matching, alerts & auto-remediation, and a live dashboard for full observability.

**Primary CTAs:**
* [View on GitHub](#)
* [Demo (Live Dashboard)](#)

---

## 🚀 Key Benefits

* **Instant detection:** Identify missing/duplicate/amount-mismatch events within seconds.
* **Actionable alerts:** Prioritised alerts with context and automated retry paths.
* **Reproducible audits:** Full immutable event log and audit trail for compliance.
* **Developer friendly:** Small codebase, docker compose, and sample event producers for fast demos.
* **Extensible:** Add new payment rails, custom matching rules, and ML anomaly detectors.

---

## 🎯 Who it’s for

* Hackathon teams building payment features
* Fintech startups validating reconciliation ideas
* Engineers preparing interviews or demos with a live system

---

## ✨ Feature Highlights

* **Event Ingestion:** Simulate/integrate gateway and bank via webhooks, Kafka, or direct API.
* **Normalization:** Convert diverse event shapes into a unified transaction model.
* **Matching Engine:** Deterministic + fuzzy rules (id, amount, time-window).
* **State Machine:** Tracks lifecycle (INIT → GATEWAY_RECEIVED → BANK_RECEIVED → MATCHED/MISMATCHED → RESOLVED).
* **Timers & SLA Rules:** Per-txn timers to escalate missing events.
* **Auto-resolve Flows:** Retry, replay, or automatic ledger correction hooks.
* **Dashboard & Logs:** Live stream, filters, per-txn timeline, exportable reports.
* **Audit + Replay:** Replay events from stream for forensic debugging.

---

## 🛠 Tech Stack

* **Backend:** Python (FastAPI)
* **Stream:** Kafka / Redpanda (or Redis Streams for lightweight)
* **Data Store:** PostgreSQL + Redis for real-time state
* **Frontend:** React + Vite + WebSockets (SSE)
* **Deployment:** Docker Compose / Kubernetes
* **Observability:** Prometheus + Grafana (metrics), ELK for logs

---

## 🏗 System Architecture

### Overview
1. **Gateway Service:** Sends "Payment Captured" events.
2. **Bank Service:** Sends "Ledger Posted" events.
3. **Recon Engine:** Listens to both streams, matches pairs, starts SLA timers, identifies mismatches, and updates the state.
4. **Dashboard:** Displays real-time status via WebSockets.

### High-Level Diagram

```
[Gateway Service] --> topic: gateway_txns  \
                                            --> [Event Stream (Kafka/Redpanda)] --> [Reconciliation Worker(s)] --> [Postgres (events + audit)]
[Bank Service]    --> topic: bank_txns     /                                       \--> [Redis (state + timers)]
                                                                                      \--> [Alerting / Ops UI (WebSockets)]
                                                                                      \--> [Dashboard (React)]
```

---

## 💻 Quickstart

### Prerequisites
* Docker & Docker Compose
* Python 3.9+ (for local scripts)

### Running Locally

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/real-recon.git
   cd real-recon
   ```

2. **Start the infrastructure:**
   ```bash
   cd infra
   docker compose up --build
   ```

3. **Open the Dashboard:**
   Visit `http://localhost:3000` to view the live stream.

### Running Demo Scenarios

We provide simulation scripts in `producers/` to verify the logic.

**Happy Path (Match):**
```bash
# Gateway event followed closely by Bank event
python producers/gateway_sim.py --txn TXN1001 --status SUCCESS
python producers/bank_sim.py --txn TXN1001 --status SUCCESS
```
*Result: Dashboard shows `MATCHED`.*

**Missing Bank Event (SLA Breach):**
```bash
# Gateway sent, Bank never responds
python producers/gateway_sim.py --txn TXN1002 --status SUCCESS
# Wait 60s...
```
*Result: Dashboard shows `BANK_MISSING` alert.*

**Amount Mismatch:**
```bash
python producers/gateway_sim.py --txn TXN1003 --amount 5000
python producers/bank_sim.py --txn TXN1003 --amount 4500
```
*Result: Dashboard shows `AMOUNT_MISMATCH`.*

---

## 🔌 API Examples

You can manually trigger events via the REST API.

**Emit Gateway Event:**
```bash
curl -X POST http://localhost:8000/events \
  -H "Content-Type: application/json" \
  -d '{
    "txn_id": "TXN1001",
    "amount": 50000,
    "currency": "INR",
    "source": "GATEWAY",
    "event_type": "PAYMENT_CAPTURED",
    "status": "SUCCESS",
    "timestamp": "2025-12-18T10:15:30Z"
  }'
```

**Query Transaction State:**
```bash
curl http://localhost:8000/txns/TXN1001
```

---

## 📊 Database Schema

### `events` (Immutable Log)
Stores every incoming raw event for auditability.
```sql
CREATE TABLE events (
  id BIGSERIAL PRIMARY KEY,
  txn_id TEXT,
  source TEXT, -- GATEWAY, BANK
  event_type TEXT,
  status TEXT,
  amount BIGINT,
  raw_payload JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### `transactions` (Current State)
Stores the reconciled state of unique transactions.
```sql
CREATE TABLE transactions (
  txn_id TEXT PRIMARY KEY,
  state TEXT, -- INIT, MATCHED, MISMATCHED, BANK_MISSING
  gateway_event_id BIGINT,
  bank_event_id BIGINT,
  last_updated TIMESTAMPTZ DEFAULT now()
);
```

---

## 🛡 Security & Compliance

* **HMAC Signatures:** Verify webhook sources.
* **Encryption:** Sensitive fields encrypted at rest (PG) and in transit (TLS).
* **Audit Logs:** Full immutable history of all automated and manual actions.
* **Limit PII:** Avoid storing full PAN/CVV; rely on tokenized references.

---

## 🚀 Roadmap

* [x] Core Reconciliation Engine (Exact Match)
* [x] Real-time Dashboard (WebSockets)
* [x] SLA Timers & Alerts
* [ ] Fuzzy Matching (ML-based)
* [ ] Automated Ledger Correction
* [ ] Slack/PagerDuty Integrations

---

## 🤝 Contributing

1. Fork the repo
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request
