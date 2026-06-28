<div align="center">

# 🟢 Distributed AI Content Moderation System

### Real-time content moderation at scale — processing 1000+ posts/second using AI

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=flat-square&logo=apachekafka&logoColor=white)](https://kafka.apache.org)
[![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=flat-square&logo=docker&logoColor=white)](https://docker.com)
[![HuggingFace](https://img.shields.io/badge/HuggingFace-FFD21E?style=flat-square&logo=huggingface&logoColor=black)](https://huggingface.co)

**[🌐 Live Demo](#)** · **[📹 Demo Video](#)** · **[🐛 Report Bug](../../issues)**

</div>

---

## 🎯 What is This?

Platforms like Instagram, Twitter, and Reddit receive millions of posts every day. Checking each piece of content for hate speech, spam, and toxic content manually is impossible.

This project builds an automated content moderation pipeline that:
- Receives posts at high volume (simulating 1000+/second)
- Processes each through multiple AI models simultaneously
- Makes approve/reject/flag decisions automatically
- Shows everything live on a monitoring dashboard

---

## ✨ Features

- ⚡ **High throughput** — Kafka queue handles traffic spikes without crashes
- 🤖 **3 AI models** — hate speech, spam, toxicity detection simultaneously
- 📊 **Live monitoring** — Prometheus + Grafana real-time dashboard
- 🔄 **Horizontal scaling** — add more workers with one command
- 👤 **Human review queue** — flagged content sent to moderators
- 🐳 **One command setup** — `docker-compose up` runs everything
- 📈 **Performance metrics** — decisions/second, accuracy, response time

---

## 🏗️ System Architecture

```
                     INCOMING CONTENT
                          │
                          ▼
              ┌─────────────────────┐
              │   FastAPI Server    │
              │   (Port 8000)      │
              │   Receives posts   │
              │   Returns instantly │
              └──────────┬──────────┘
                         │ Sends to queue
                         ▼
              ┌─────────────────────┐
              │    Apache Kafka     │
              │  (Message Queue)   │
              │  Holds all posts   │
              │  Nothing gets lost │
              └──────────┬──────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
    ┌─────────┐    ┌─────────┐    ┌─────────┐
    │ Worker1 │    │ Worker2 │    │ Worker3 │
    │         │    │         │    │         │
    │ AI      │    │ AI      │    │ AI      │
    │ Models  │    │ Models  │    │ Models  │
    └────┬────┘    └────┬────┘    └────┬────┘
         └──────────────┼──────────────┘
                        │ All save results
                        ▼
              ┌─────────────────────┐
              │    PostgreSQL       │
              │  Stores decisions   │
              └─────────────────────┘
                        │
                        ▼
              ┌─────────────────────┐
              │ Prometheus+Grafana  │
              │  Live monitoring    │
              └─────────────────────┘
```

---

## 🤖 AI Models Used

| Model | Task | Source | Cost |
|-------|------|--------|------|
| `cardiffnlp/twitter-roberta-base-hate-latest` | Hate speech detection | HuggingFace | Free (local) |
| `mrm8488/bert-tiny-finetuned-sms-spam-detection` | Spam detection | HuggingFace | Free (local) |
| `unitary/toxic-bert` | Toxicity scoring | HuggingFace | Free (local) |

All models run **locally** — no API costs, no data leaving your server.

---

## ⚡ Decision Engine Rules

```
Score > 90% hate speech  →  AUTO REMOVE  ❌
Score > 85% toxicity     →  AUTO REMOVE  ❌
Score > 95% spam         →  AUTO REMOVE  ❌

Score 60-90% hate speech →  FLAG for human review  🚩
Score 60-85% toxicity    →  FLAG for human review  🚩

All scores low           →  APPROVE  ✅
```

---

## 🚀 Getting Started

### Prerequisites

```
Docker & Docker Compose (required)
Python 3.11+ (for local development)
8GB RAM minimum (AI models need memory)
```

### Run With Docker (Recommended)

```bash
# 1. Clone
git clone https://github.com/Pushkarpant/content-moderation-system.git
cd content-moderation-system

# 2. Start everything
docker-compose up

# This starts:
# → Kafka (message queue)
# → 3 AI worker containers
# → FastAPI API server
# → PostgreSQL database
# → Prometheus (metrics)
# → Grafana (dashboard)
```

### Access the Services

| Service | URL | Purpose |
|---------|-----|---------|
| API | `localhost:8000` | Submit content |
| API Docs | `localhost:8000/docs` | Test endpoints |
| Grafana | `localhost:3001` | Live monitoring |
| Prometheus | `localhost:9090` | Raw metrics |

### Test It

```bash
# Submit a post
curl -X POST http://localhost:8000/submit-content \
  -H "Content-Type: application/json" \
  -d '{"text": "This is a normal post", "user_id": "user123"}'

# Response (instant):
{
  "status": "submitted",
  "content_id": "post_abc123",
  "message": "Content queued for moderation"
}

# Check the decision (after ~2 seconds):
curl http://localhost:8000/decisions/post_abc123
{
  "decision": "APPROVED",
  "hate_score": 0.02,
  "spam_score": 0.01,
  "toxicity_score": 0.05,
  "processing_ms": 145
}
```

### Scale Workers

```bash
# Need more processing power? Add workers instantly:
docker-compose up --scale worker=10

# 10 workers now processing simultaneously
# 10x throughput
```

---

## 📡 API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/submit-content` | Submit content for moderation |
| `GET` | `/decisions/{id}` | Get moderation decision |
| `GET` | `/queue/stats` | Queue size and status |
| `GET` | `/metrics/today` | Today's moderation stats |
| `GET` | `/review-queue` | Content flagged for human review |

---

## 📊 Monitoring Dashboard

The Grafana dashboard shows in real-time:

- **Posts/second** — throughput over time
- **Decision breakdown** — Approved vs Removed vs Flagged
- **Processing time** — average milliseconds per post
- **Worker health** — which workers are active
- **Queue depth** — how many posts waiting

---

## 🗂️ Project Structure

```
content-moderation-system/
│
├── api/
│   ├── main.py               # FastAPI server
│   └── kafka_producer.py     # Sends to Kafka
│
├── worker/
│   ├── consumer.py           # Reads from Kafka
│   ├── ai_models.py          # HuggingFace models
│   └── decision_engine.py    # Rules engine
│
├── monitoring/
│   ├── prometheus.yml        # Metrics config
│   └── grafana/
│       └── dashboard.json    # Pre-built dashboard
│
├── frontend/                 # React dashboard
├── docker-compose.yml        # Full system setup
└── requirements.txt
```

---

## 🛠️ Tech Stack

| Component | Technology | Why |
|-----------|-----------|-----|
| Message Queue | Apache Kafka | Handles traffic spikes, nothing lost |
| AI Models | HuggingFace Transformers | Free, runs locally |
| API | FastAPI (Python) | Fast, async capable |
| Database | PostgreSQL | Store all decisions |
| Monitoring | Prometheus + Grafana | Production-grade observability |
| Containers | Docker + Compose | Easy deployment, scalable |

---

## 🎓 Key Learnings

- **Why Kafka?** Decouples content ingestion from processing. 100k posts can arrive simultaneously without crashing — they queue safely and workers process at their own pace.

- **Why local AI models?** API costs would be prohibitive at scale. Local HuggingFace models run free after download, with <200ms inference.

- **Why multiple workers?** Each worker processes independently. Adding more workers = linear throughput increase. This is horizontal scaling.

---

## 📝 License

MIT License — see [LICENSE](LICENSE)

---

<div align="center">
  Built with ❤️ by <a href="https://github.com/Pushkarpant">Pushkar</a>
</div>
