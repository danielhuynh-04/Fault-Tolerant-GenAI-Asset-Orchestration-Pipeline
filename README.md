<div align="center">

# ⚡ Fault-Tolerant GenAI Asset Orchestration Pipeline

### Idempotent, Resume-Safe Multimodal Generation Pipeline with Automated Multi-Provider Failover (`HTTP 429` Handling), 7-Layer Prompt Evaluation Rubric, OAuth 2.0 Security & Scheduled Telemetry

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img src="https://img.shields.io/badge/Google_Cloud_APIs-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white">
  <img src="https://img.shields.io/badge/Gemini_API-8E75B2?style=for-the-badge&logo=googlegemini&logoColor=white">
  <img src="https://img.shields.io/badge/OAuth_2.0-EB4335?style=for-the-badge&logo=google&logoColor=white">
  <img src="https://img.shields.io/badge/SQLAlchemy-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white">
  <img src="https://img.shields.io/badge/Slack_Block_Kit-4A154B?style=for-the-badge&logo=slack&logoColor=white">
  <img src="https://img.shields.io/badge/Plotly-3F4F75?style=for-the-badge&logo=plotly&logoColor=white">
  <img src="https://img.shields.io/badge/pytest-0A9EDC?style=for-the-badge&logo=pytest&logoColor=white">
</p>

_A production-grade, fault-tolerant multimodal GenAI orchestration pipeline in Python. Built with **SQLAlchemy/SQLite idempotent job-state tracking**, **concurrent worker pools (`ThreadPoolExecutor`)**, and **exponential-backoff retries** with automated failover across LLM and diffusion providers on **`HTTP 429` rate limits**. Backed by a systematically evaluated **7-layer prompt framework** (3 iterations × 5 asset classes scored on a 5-criterion Likert rubric), **OAuth 2.0** scoped security, real-time **Slack Block Kit** incident alerting, scheduled **Plotly** telemetry dashboards, and **`pytest`** unit test suites._

**Engineered by:** [Daniel Huynh (Huynh Le Thanh Hai)](https://github.com/danielhuynh-04) • [LinkedIn](https://linkedin.com/in/le-thanh-hai-huynh-8353913a1) • [Kaggle](https://kaggle.com/lthanhhihunh)

<p align="center">
  <a href="PROMPT_ENGINEERING_REPORT_EN.md"><b>📖 Read 7-Layer Prompt Engineering Report (English)</b></a> &nbsp;•&nbsp; 
  <a href="prompt_engineering_report.md"><b>📖 Đọc Báo Cáo Kỹ Nghệ Prompt 7 Tầng (Tiếng Việt)</b></a> &nbsp;•&nbsp;
  <a href="docs/System_Design.md"><b>🏛️ System Design & Architecture</b></a>
</p>

</div>

---

## 🏆 Key Engineering Highlights

| Capability | Engineering Implementation |
|:---|:---|
| **Idempotent & Resume-Safe State Machine** | Tracks job lifecycle (`PENDING → RUNNING → SUCCESS / FAILED`) in SQLite via SQLAlchemy ORM; rows marked `DONE` are automatically skipped on rerun to prevent duplicate API consumption. |
| **High-Availability Multi-Provider Failover** | Primary **Gemini Imagen 3** (`imagen-3.0-generate-002`) automatically fails over to **Flux** (`Pollinations.ai`) upon quota exhaustion (`HTTP 429`) or transient API timeouts. |
| **Concurrent Worker Pool & Retry Backoff** | Parallel I/O execution via `ThreadPoolExecutor` paired with a custom `@retry` decorator implementing exponential backoff (`2s → 4s → 8s`, 3 attempts). |
| **7-Layer Structured Prompt Framework** | Parameterized prompt architecture (`[SUBJECT]`, `[STYLE]`, `[PALETTE]`, `[LIGHTING]`, `[COMPOSITION]`, `[TECHNICAL]`, `[NEGATIVE]`) benchmarked across **3 iterations × 5 asset classes** using a **5-criterion Likert rubric** (+123% quality improvement from V1 to V3). |
| **OAuth 2.0 Scoped Security** | Eliminates raw SMTP app passwords in favor of **Gmail API OAuth 2.0 Web Consent** and scoped **Google Drive API v3** credentials. |
| **Real-Time Incident Alerting & Telemetry** | Dispatches structured **Slack Block Kit** webhooks on job completion/failure and schedules daily **Plotly HTML telemetry dashboards** (`APScheduler` at `23:00`). |
| **Automated Unit Testing (`pytest`)** | Dedicated unit test suite covering input schema validation edge cases (`test_validator.py`) and in-memory SQLAlchemy state transitions (`test_db_logger.py`). |

---

## 🖼️ Visual Showcase: 7-Layer Prompt Framework (V3 Production Outputs)

All 5 asset categories below were generated through the pipeline and evaluated across 3 controlled iterations (`V1 → V2 → V3`) on a 25-point (5-criterion × 5-point Likert) rubric. Full methodology and prompt templates are documented in [`PROMPT_ENGINEERING_REPORT_EN.md`](PROMPT_ENGINEERING_REPORT_EN.md).

| Character Asset | Bingo Ball Asset | UI Button Asset | Card Frame Asset | Stage Background |
|:---:|:---:|:---:|:---:|:---:|
| <img src="prompt_engineering/iterations/character/v3_character_final.png" width="150" alt="Character"/> | <img src="prompt_engineering/iterations/bingo_ball/v3_ball_blue_final.png" width="135" alt="Ball"/> | <img src="prompt_engineering/iterations/ui_button/v3_button_play_final.png" width="150" alt="Button"/> | <img src="prompt_engineering/iterations/card_frame/v3_frame_blue_final.png" width="135" alt="Frame"/> | <img src="prompt_engineering/iterations/background/v3_background_final.png" width="170" alt="Background"/> |
| **Game Show Host** (`1024×1024`) | **Glossy 3D Sphere** (`#7`) | **3D Bevel UI** (`PLAY`) | **Ornate Baroque Frame** | **Theatrical Stage** (`1080p`) |
| V1: `13/25` → **V3: `25/25`** | V1: `11/25` → **V3: `25/25`** | V1: `10/25` → **V3: `25/25`** | V1: `12/25` → **V3: `25/25`** | V1: `10/25` → **V3: `25/25`** |

---

## 🏗️ System Architecture

```mermaid
graph TD
    A["Google Sheets<br>Asset Request Input"] -->|Read PENDING Rows| B("src/sheets_reader.py<br>Google Sheets API v4")
    B -->|Raw Row Data| C{"src/validator.py<br>Schema Validation"}

    C -- Invalid Data --> D["src/db_logger.py<br>Mark Status = FAILED"]
    C -- Valid Data --> E("src/main.py<br>ThreadPoolExecutor<br>Concurrent Worker Pool")

    E -->|Dispatch Job| F["src/ai_generator.py"]

    subgraph "High-Availability AI Routing Engine (Multi-Provider Failover)"
       F -->|"Primary: Gemini Imagen 3<br>(imagen-3.0-generate-002)"| G["Google Gemini API"]
       G -.->|"HTTP 429 / Quota Exhaustion"| H["Flux Fallback Provider<br>(Pollinations.ai)"]
       G -.->|"Transient Timeout / Error"| I["src/retry_wrapper.py<br>Exponential Backoff<br>3 Retries (2s → 4s → 8s)"]
       F -->|"Audio Synthesis (MP3)"| J["Audio Synthesis Engine<br>(gTTS)"]
    end

    subgraph "LLM Prompt Optimization Layer"
       F -->|"Pre-Generation Step"| K["Gemini 2.0 Flash<br>(Structured Prompt Expansion)"]
       K -.->|"Fallback"| L["Fallback Text LLM<br>(OpenAI-Compatible Endpoint)"]
    end

    G --> M("Multimodal Asset Output<br>PNG / JPG / GIF / MP3")
    H --> M
    J --> M

    M -->|"Upload Binary"| N["src/drive_uploader.py<br>Google Drive API v3<br>OAuth 2.0 Scoped Token"]
    N -->|"Store & Return URL"| O[("Google Drive<br>Outputs/{date}/{format}/")]

    M -->|"Persist State Transition"| D
    D -->|"ACID Transaction"| P[("SQLite Database<br>Thread-Safe via SQLAlchemy<br>check_same_thread=False")]

    M -->|"Real-Time Incident & Job Alert"| Q["src/notifier.py"]
    Q -->|"Slack Block Kit"| R["Slack Channel<br>Incoming Webhook"]
    Q -->|"Gmail API OAuth 2.0"| S["Administrator Email<br>HTML + Telemetry Attachment"]

    T(("APScheduler<br>Daily CronTrigger")) -.->|"Scheduled Execution"| U["src/daily_report.py"]
    P -.->|"Query Aggregated Metrics"| U
    U -->|"Render Interactive Charts"| V["Plotly HTML Telemetry Dashboard<br>(Pie + Bar + Scatter + Distribution)"]
    V --> S

    style A fill:#003366,color:#fff,stroke:#002244
    style F fill:#336699,color:#fff,stroke:#113355
    style G fill:#6699cc,color:#fff,stroke:#224466
    style M fill:#003366,color:#fff,stroke:#002244
    style P fill:#336699,color:#fff,stroke:#113355
    style V fill:#6699cc,color:#fff,stroke:#224466
```

---

## 🔄 Orchestration Sequence (BPMN Flow)

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant System as Orchestrator (main.py)
    participant AI as Multi-Provider AI Gateway
    participant Storage as Persistence (Drive / SQLAlchemy)
    participant Notif as Observability (Slack / Gmail OAuth 2.0)

    User->>Storage: 1. Submit batch asset requests to Google Sheets
    System->>Storage: 2. Fetch PENDING rows (skip DONE rows for idempotency)
    Storage-->>System: Return normalized row payloads

    Note over System, AI: Concurrent Worker Execution (ThreadPoolExecutor)
    System->>System: 3. Validate input schema (format, description length, URL)

    alt Valid Request
        System->>AI: 4. Expand & optimize prompt via 7-Layer LLM template
        AI-->>System: Return structured 7-layer prompt
        System->>AI: 5. Generate multimodal asset (Primary Gemini → Flux Failover on HTTP 429)
        AI-->>System: Return binary asset payload
        System->>Storage: 6. Upload binary to Google Drive folder hierarchy
        Storage-->>System: Return shareable Drive URL
        System->>Storage: 7. Commit SQLite state transition → SUCCESS
        System->>Storage: 8. Sync DONE status back to Google Sheets
        System->>Notif: 9. Dispatch notify_success()
        Notif-->>User: Real-time Slack Block Kit + Gmail OAuth 2.0 notification
    else Invalid Input or Retries Exhausted
        AI-->>System: Exception raised after 3 exponential-backoff retries
        System->>Storage: 10. Commit SQLite state transition → FAILED (log error & duration)
        System->>Storage: 11. Sync FAILED status to Google Sheets
        System->>Notif: 12. Dispatch notify_failure()
        Notif-->>User: Real-time Slack Incident Block + Gmail error alert
    end

    Note over System, Notif: Scheduled Telemetry Pipeline (APScheduler)
    System-->>System: 13. Trigger scheduled analytics job
    System->>Storage: 14. Query daily job statistics & latency distribution from SQLite
    System->>System: 15. Render 4 interactive Plotly charts into standalone HTML dashboard
    System->>Notif: 16. Email HTML Telemetry Dashboard via Gmail OAuth 2.0
    Notif-->>User: Daily Operational Telemetry Summary
```

---

## 🧩 Repository Structure

```text
Fault-Tolerant-GenAI-Asset-Orchestration-Pipeline/
│
├── src/                                      # Core Orchestration & Pipeline Modules
│   ├── main.py                               # ⚙️  Pipeline Orchestrator (ThreadPoolExecutor, CLI entry point)
│   ├── sheets_reader.py                      # 📊 Google Sheets API v4 Ingestion & Idempotent Row Filtering
│   ├── validator.py                          # ✅ Input Schema Validation (format, URL, description constraints)
│   ├── ai_generator.py                       # 🤖 Multi-Provider AI Gateway (Gemini Imagen 3 → Flux Failover + Audio)
│   ├── retry_wrapper.py                      # 🔄 Exponential-Backoff Retry Decorator (configurable attempts & delay)
│   ├── drive_uploader.py                     # ☁️  Google Drive API v3 Uploader (OAuth 2.0 scoped tokens)
│   ├── db_logger.py                          # 💾 SQLAlchemy ORM State Machine (thread-safe SQLite transactions)
│   ├── notifier.py                           # 📢 Slack Block Kit Webhooks + Gmail API OAuth 2.0 Notifications
│   └── daily_report.py                       # 📈 Plotly Telemetry Dashboard Generator + APScheduler CronTrigger
│
├── prompt_engineering/                       # 7-Layer Prompt Framework & Empirical Evaluation
│   ├── PROMPT_ENGINEERING_REPORT_EN.md       # Full English Prompt Engineering Report (5 assets × 3 iterations)
│   ├── prompt_engineering_report.md          # Vietnamese Prompt Engineering Report
│   └── iterations/                           # Generated V3 showcase assets (character, ball, button, frame, stage)
│
├── tests/                                    # Automated Unit Test Suite (pytest)
│   ├── conftest.py                           # Shared pytest fixtures
│   ├── test_validator.py                     # Schema validation & normalization unit tests (10 test cases)
│   └── test_db_logger.py                     # In-memory SQLAlchemy state persistence & telemetry unit tests (6 test cases)
│
├── database/                                 # Relational Schema Definition
│   └── schema.sql                            # SQLite DDL for `jobs` and `reports` tables with composite indexes
│
├── docs/                                     # System Design & Requirements Specification
│   ├── Requirement_Analysis.md               # Functional & Non-Functional Requirements (FR/NFR matrix)
│   └── System_Design.md                      # High-level architecture & BPMN swimlane design principles
│
├── tools/
│   └── populate_sheet.py                     # Utility script to seed sample benchmark requests into Google Sheets
│
├── .env.example                              # Environment configuration template (zero hardcoded secrets)
├── requirements.txt                          # Pinned Python dependencies
├── PROMPT_ENGINEERING_REPORT_EN.md           # Root shortcut to English Prompt Engineering Report
└── README.md                                 # Project documentation
```

---

## 🛠️ Technology Stack & Engineering Methods

| Layer | Technologies & Patterns |
|:---|:---|
| **Language & Data** | Python 3.10+, SQL, SQLAlchemy 2.0 ORM, SQLite (`check_same_thread=False`) |
| **Generative AI Providers** | Google Gemini Imagen 3 (`imagen-3.0-generate-002`), Gemini 2.0 Flash (`gemini-2.0-flash`), Flux Diffusion (`Pollinations.ai`), `gTTS` |
| **Cloud & Authentication** | Google Sheets API v4, Google Drive API v3, Gmail API v1, **OAuth 2.0 Web Consent (Scoped Tokens)**, Google Service Accounts |
| **Fault Tolerance & Concurrency** | `ThreadPoolExecutor` concurrent worker pool, custom `@retry` decorator with exponential backoff (`2s → 4s → 8s`), automated `HTTP 429` provider failover, idempotent `DONE`-row filtering |
| **Prompt Engineering** | 7-Layer Modular Prompt Architecture evaluated across 3 iterations and 5 asset categories using a 5-criterion Likert scale (`25/25` final score) |
| **Observability & Alerting** | Slack Incoming Webhooks (**Slack Block Kit** JSON), scheduled **Plotly** HTML telemetry dashboards (`APScheduler`) |
| **Quality Assurance** | **`pytest`** unit testing for input validation and database state machine transitions |

---

## 🚀 Quick Start

### 1. Clone & Install Dependencies
```bash
git clone https://github.com/danielhuynh-04/Fault-Tolerant-GenAI-Asset-Orchestration-Pipeline.git
cd Fault-Tolerant-GenAI-Asset-Orchestration-Pipeline
python -m venv .venv
.venv\Scripts\activate       # Windows (use `source .venv/bin/activate` on Linux/macOS)
pip install -r requirements.txt
```

### 2. Configure Google Cloud & OAuth 2.0 Credentials
> **Architecture & Requirements Reference:** [`docs/System_Design.md`](docs/System_Design.md) • [`docs/Requirement_Analysis.md`](docs/Requirement_Analysis.md)

1. Create a [Google Cloud Project](https://console.cloud.google.com) and enable **Google Sheets API**, **Google Drive API**, **Generative Language API**, and **Gmail API**.
2. Create a Service Account for Sheets access → download JSON key as `credentials.json` in the project root.
3. Obtain a [Gemini API Key](https://aistudio.google.com/app/apikey).
4. Create an [OAuth 2.0 Client ID](https://console.cloud.google.com/apis/credentials/oauthclient) (Desktop App) for Drive & Gmail API consent → save as `client_secret.json` in the project root.
5. Configure a [Slack Incoming Webhook](https://api.slack.com/messaging/webhooks) for real-time Block Kit alerts.

### 3. Environment Variables
```bash
cp .env.example .env
# Populate GEMINI_API_KEY, GOOGLE_SHEET_ID, GOOGLE_DRIVE_FOLDER_ID, SLACK_WEBHOOK_URL, and EMAIL_ADMIN
```

### 4. Run Pipeline & Unit Tests
```bash
# Run automated unit test suite (16/16 passing tests)
python -m pytest tests/ -v

# Execute the end-to-end orchestration pipeline
python src/main.py

# Generate the interactive Plotly HTML telemetry report immediately
python src/main.py --report-only

# Run pipeline with background APScheduler daily telemetry daemon
python src/main.py --with-scheduler
```

---

## 📊 Idempotent Job State Machine

```text
PENDING ──► RUNNING ──► SUCCESS (Asset generated, uploaded to Drive, persisted in SQLite, Slack/Gmail notified)
               │
               └──► RETRY (Attempt 1 → 2 → 3 with Exponential Backoff & Provider Failover on HTTP 429)
                       ├──► SUCCESS
                       └──► FAILED (Retries exhausted; error & latency persisted in SQLite, incident alert sent)

PENDING ──► FAILED (Blocked at Schema Validator prior to AI API invocation — zero wasted tokens)
```

---

<div align="center">
  <b>Daniel Huynh (Huynh Le Thanh Hai)</b><br>
  Computer Science (Database & AI Engineering) • Ho Chi Minh City Open University<br>
  <a href="mailto:Haiworkai@gmail.com">Haiworkai@gmail.com</a> • <a href="https://linkedin.com/in/le-thanh-hai-huynh-8353913a1">LinkedIn</a> • <a href="https://kaggle.com/lthanhhihunh">Kaggle</a>
</div>
