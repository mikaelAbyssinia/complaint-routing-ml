# FinResolve — AI-Powered Financial Complaint Routing System

> A production-grade machine learning system that automatically classifies and routes consumer financial complaints to the appropriate department and assigns priority levels for resolution.

**Live Demo:** [https://finresolve-app.onrender.com](https://finresolve-app.onrender.com)

---

## Overview

FinResolve is an end-to-end machine learning platform built on top of the Consumer Financial Protection Bureau (CFPB) complaint dataset. The system ingests a free-text complaint narrative, predicts the most appropriate department to handle it, and assigns a priority level — critical, high priority, or standard — enabling financial institutions to triage and route complaints efficiently at scale.

The platform serves three distinct user roles: customers who submit complaints, department operators who manage their queues, and reviewers who handle low-confidence edge cases that require human judgment.

---

## Academic Context

| | |
|---|---|
| **Institution** | George Washington University |
| **Course** | Applied Machine Learning for Analytics — EMSE 6575 |
| **Instructor** | Prof. Maksim Tsvetovat |
| **Term** | Spring 2026 |

**Team**

| Name | Role |
|---|---|
| Michael M. Demissie | Project Lead, ML Engineering, Backend & Frontend |
| Blessing | Team Member |
| Andy Gomez | Team Member |
| Tony | Team Member |
| Chidochashe | Team Member |
| Tino | Team Member |
| Ncee | Team Member |

---

## System Architecture

```
User Complaint (free text)
        │
        ▼
┌─────────────────────────┐
│   FastAPI REST API      │
│   JWT Authentication    │
└────────────┬────────────┘
             │
    ┌────────┴────────┐
    ▼                 ▼
┌─────────┐    ┌──────────────┐
│  BERT   │    │  LightGBM    │
│ Dept.   │    │  Priority    │
│ Model   │    │  Classifier  │
└────┬────┘    └──────┬───────┘
     │                │
     ▼                ▼
Department        Priority Level
(9 categories)   (critical / high / standard)
     │                │
     └────────┬────────┘
              ▼
     PostgreSQL (Neon)
     Complaint stored with
     predictions & confidence scores
              │
     ┌────────┴────────┐
     ▼                 ▼
Operator Dashboard  Reviewer Queue
(by department &    (low-confidence
 priority column)    complaints)
```

---

## Machine Learning Models

### Department Classifier — DistilBERT

A fine-tuned `distilbert-base-uncased` model that classifies complaint narratives into one of 9 financial departments.

| Department |
|---|
| Bank accounts |
| Card services |
| Consumer loans |
| Credit reporting |
| Debt collection |
| Money transfer services |
| Mortgage |
| Payday / personal loans |
| Student loan |

The model is hosted on AWS S3 and downloaded to the server at startup. It achieves strong performance across all 9 classes on the CFPB dataset.

### Priority Classifier — TF-IDF + LightGBM

A TF-IDF vectorizer (50,000 features, unigrams + bigrams) paired with a LightGBM gradient boosting classifier. Priority labels were engineered using a domain-specific rule-based scoring system developed for each department — incorporating signals such as fraud keywords, legal references, financial amounts, harassment indicators, and persistence signals.

| Priority | Description | Target Resolution |
|---|---|---|
| Critical | Identity theft, fraud, foreclosure, legal threats | 1–2 business days |
| High Priority | Disputed accounts, harassment, credit damage | 3–4 business days |
| Standard | Routine disputes, billing inquiries | 5–6 business days |

**Model Performance on held-out test set (203,180 samples):**

| Class | Precision | Recall | F1 |
|---|---|---|---|
| Critical | 0.90 | 0.90 | 0.90 |
| High Priority | 0.82 | 0.84 | 0.83 |
| Standard | 0.93 | 0.91 | 0.92 |
| **Overall Accuracy** | | | **0.88** |

---

## Dataset

**Source:** [Consumer Financial Protection Bureau (CFPB) Complaint Database](https://www.consumerfinance.gov/data-research/consumer-complaints/)

| Split | Samples |
|---|---|
| Training | 812,720 |
| Test | 203,180 |
| **Total** | **1,015,900** |

The dataset was filtered to include only complaints with a consumer narrative, deduplicated, cleaned of CFPB-style masking tokens (`XXXX`), and labeled using domain-specific priority scoring functions before model training.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **API** | FastAPI, Uvicorn |
| **Auth** | JWT (python-jose), bcrypt |
| **ML — Department** | DistilBERT (HuggingFace Transformers) |
| **ML — Priority** | TF-IDF + LightGBM (scikit-learn pipeline) |
| **Database** | PostgreSQL (Neon serverless) |
| **Model Storage** | AWS S3 |
| **Frontend** | Jinja2 templates, vanilla JS |
| **Deployment** | Render |
| **Language** | Python 3.11 |

---

## Project Structure

```
complaint-routing-ml/
├── app/
│   ├── main.py               # FastAPI entrypoint
│   ├── config.py             # Environment variable management
│   ├── database.py           # PostgreSQL connection
│   ├── dependencies.py       # JWT auth dependency
│   ├── logger.py             # Centralized logging
│   ├── preprocessing.py      # Text preprocessing
│   ├── ml/
│   │   ├── predictor.py      # Inference pipeline
│   │   ├── priority_rules.py # Rule-based label engineering
│   │   └── train_priority.py # LightGBM training script
│   ├── routes/
│   │   ├── auth.py           # Register & login endpoints
│   │   ├── complaints.py     # Complaint submission & retrieval
│   │   └── pages.py          # HTML page routes
│   └── schemas/
│       ├── auth.py           # Pydantic auth models
│       └── complaints.py     # Pydantic complaint models
├── models/
│   └── priority-lgbm/        # LightGBM pipeline & label encoder
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_baseline_model.ipynb
│   ├── 03_BERT_model_department.ipynb
│   └── distilbert_base_uncased_priority.ipynb
├── data/
│   ├── raw/                  # Original CFPB CSV
│   ├── interim/              # Cleaned parquet
│   └── processed/            # Train/test splits
├── docs/                     # Project documentation
├── tests/                    # Test suite
├── static/                   # Static assets
├── templates/                # Jinja2 HTML templates
├── requirements.txt
└── README.md
```

---

## API Endpoints

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/register` | None | Register a new user |
| `POST` | `/login` | None | Authenticate and receive JWT |
| `POST` | `/submit-complaint` | Customer | Submit a complaint for AI routing |
| `GET` | `/my-complaints` | Customer | Retrieve personal complaint history |
| `GET` | `/operator-complaints` | Operator | Get department complaint queue |
| `GET` | `/reviewer-complaints` | Reviewer | Get low-confidence complaint queue |

---

## User Roles

**Customer** — submits complaints through the portal and tracks resolution status.

**Operator** — views complaints routed to their department, organized by priority level (critical, high priority, standard) with AI confidence scores.

**Reviewer** — handles complaints where the model confidence fell below the threshold for either department or priority, requiring human judgment before routing.

---

## Local Development

**Prerequisites:** Python 3.11, PostgreSQL, AWS credentials (for department model S3 download)

```bash
# Clone the repository
git clone https://github.com/michael-demissie/complaint-routing-ml.git
cd complaint-routing-ml

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your DATABASE_URL, AWS credentials, SECRET_KEY

# Create database tables
python3 app/db_setup.py

# Start the development server
uvicorn app.main:app --reload
```

The app will be available at `http://127.0.0.1:8000`.

---

## Environment Variables

| Variable | Description |
|---|---|
| `SECRET_KEY` | JWT signing key (generate with `secrets.token_hex(32)`) |
| `DATABASE_URL` | PostgreSQL connection string |
| `AWS_ACCESS_KEY_ID` | AWS credentials for S3 model download |
| `AWS_SECRET_ACCESS_KEY` | AWS credentials for S3 model download |
| `AWS_REGION` | AWS region (e.g. `us-east-1`) |
| `S3_BUCKET_NAME` | S3 bucket containing the department model |
| `ALLOWED_ORIGINS` | Comma-separated list of allowed CORS origins |
| `CONFIDENCE_THRESHOLD` | Minimum confidence score before flagging for review (default: `0.5`) |

---

## Deployment

The application is deployed on **Render** with a **Neon** serverless PostgreSQL database. The DistilBERT department model (~250MB) is stored on **AWS S3** and downloaded to `/tmp` at startup, with caching to skip re-downloads on warm restarts. The LightGBM priority model (~14MB) is stored directly in the repository.

---

## Acknowledgements

This project was developed as part of the Applied Machine Learning for Analytics course (EMSE 6575) at the George Washington University under the supervision of Prof. Maksim Tsvetovat. The complaint data is sourced from the publicly available CFPB Consumer Complaint Database.
