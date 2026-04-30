# RetailSense AI

**Demand Forecasting & Customer Intelligence Platform**

[![Python](https://img.shields.io/badge/Python-3.11-blue)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.95+-green)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-18-61DAFB)](https://react.dev)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)](https://docker.com)
[![MLflow](https://img.shields.io/badge/MLflow-3.11-orange)](https://mlflow.org)
[![EC2](https://img.shields.io/badge/AWS-EC2-FF9900)](http://16.171.161.243:3000)

**Live Demo:** [http://16.171.161.243:3000](http://16.171.161.243:3000)  
**MLflow Tracking:** [http://16.171.161.243:5000](http://16.171.161.243:5000)  
**API Docs:** [http://16.171.161.243:8000/docs](http://16.171.161.243:8000/docs)

---

## Problem Statement

Retail chains operating across dozens of stores and hundreds of product families face a persistent forecasting problem: **demand is highly nonlinear**. Sales are shaped simultaneously by promotions, national holidays, oil prices (a proxy for economic conditions in Ecuador), store location, seasonal cycles, and historical lag patterns — none of which linear models handle well in combination.

Specifically, Corporación Favorita — one of Ecuador's largest grocery retailers — operates **54 stores** across multiple cities, carrying **33 product families**, with daily sales records spanning years. Their existing forecasting approach treated each store-family combination independently and relied on simple moving averages, producing errors too large to optimize inventory effectively. The consequences were real:

- **Overstocking** in slow periods locked up capital and increased waste
- **Understocking** during promotions and holidays caused lost sales and poor customer experience
- **No model visibility** — store managers could not understand why a forecast was high or low
- **No customer segmentation** — stores were treated as identical despite wildly different sales profiles
- **No model governance** — multiple model versions existed with no systematic comparison or tracking

---

## Solution: End-to-End ML Pipeline

RetailSense AI converts this problem into a **production-grade machine learning system** with full experiment tracking, model explainability, drift monitoring, and a live web dashboard.

### Pipeline Overview

```
Raw Data (CSV)
     │
     ▼
┌─────────────────────────────────┐
│  Feature Engineering            │
│  - Lag features (7, 14, 28 day) │
│  - Rolling statistics           │
│  - Date decomposition           │
│  - Holiday & oil encoding       │
│  - Store cluster assignment     │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│  Model Training (MLflow)        │
│  - XGBoost                      │
│  - LightGBM                     │
│  - ARIMA (baseline)             │
│  - K-Means / GMM segmentation   │
│  21 tracked runs, versioned     │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│  Model Selection (A/B Test)     │
│  Paired t-test on MAE           │
│  → LightGBM selected as best    │
│    (RMSE 336, MAE 229, MAPE 8%) │
│  → Registered as v3 in MLflow   │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│  FastAPI Backend                │
│  - /predict/forecast            │
│  - /predict/segment             │
│  - /explain/{store}  (SHAP)     │
│  - /ab-test                     │
│  - /drift                       │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│  React Dashboard                │
│  Dashboard / Forecast /         │
│  Segmentation / Explainability  │
│  A/B Test / Drift               │
└─────────────────────────────────┘
                 │
                 ▼
     Docker Compose on AWS EC2
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Data & Features | Pandas, NumPy, SQL window functions |
| ML Models | XGBoost, LightGBM, scikit-learn, statsmodels (ARIMA) |
| Clustering | K-Means, Gaussian Mixture Model (GMM) |
| Explainability | SHAP TreeExplainer |
| Experiment Tracking | MLflow 3.11 (21 tracked runs, model registry) |
| Backend API | FastAPI + Uvicorn |
| Frontend | React 18 + Recharts + TailwindCSS |
| Serving | Nginx (reverse proxy + static serving) |
| Storage | MinIO (S3-compatible), PostgreSQL |
| Deployment | Docker Compose on AWS EC2 |
| Orchestration (next) | Kubernetes |

---

## Model Results

| Model | RMSE | MAE | MAPE |
|-------|------|-----|------|
| **LightGBM** | **336.33** | **229.67** | **8.78%** |
| XGBoost | 360.22 | 245.97 | 9.46% |
| ARIMA (baseline) | 740.71 | 517.14 | 30.58% |

LightGBM selected as production model via paired t-test A/B comparison (p=0.0036, statistically significant).

**Top predictive features (SHAP):** `month`, `cluster`, `transactions`, `rolling_14_mean`, `family_enc`, `dayofweek`, `onpromotion`

---

## Screenshots

### Live Dashboard — EC2
![Live EC2 Dashboard](data/Screenshot%202026-04-30%20050222.jpg)

### Dashboard — Model Comparison
![Dashboard](data/Screenshot%202026-04-29%20044820.jpg)

### Forecast — Input & Prediction
![Forecast Input](data/Screenshot%202026-04-29%20044932.jpg)

![Forecast Result](data/Screenshot%202026-04-29%20045007.jpg)

> Prediction: **2486 units** with 95% confidence interval [1827, 3146] using LightGBM

### Explainability — SHAP Feature Importance
![SHAP Explainability](data/Screenshot%202026-04-29%20045026.jpg)

### A/B Test — Model Comparison
![AB Test](data/Screenshot%202026-04-29%20045042.jpg)

> T-stat: -3.32 · P-value: 0.0036 · Winner: XGBoost (on this sample)

### MLflow — Experiment Tracking
![MLflow Runs](data/Screenshot%202026-04-29%20035433.jpg)

![MLflow Best Model](data/Screenshot%202026-04-29%20035455.jpg)

> 21 tracked runs across XGBoost, LightGBM, ARIMA, Segmentation, A/B Test · Best model registered as `RetailSense_Forecaster v3`

---

## Project Structure

```
retailsense-ai/
├── backend/
│   ├── main.py            # FastAPI app, lifespan, CORS
│   ├── predict.py         # Forecast, segment, explain, A/B, drift endpoints
│   ├── model_loader.py    # Loads .pkl models from /app/models
│   ├── drift_monitor.py   # KS-test drift detection
│   ├── schemas.py         # Pydantic request/response models
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── pages/         # Dashboard, Forecast, Segmentation, Explainability, ABTest, Drift
│   │   ├── components/    # Navbar, Sidebar, StatCard
│   │   └── api/api.js     # Axios client (baseURL: /api)
│   ├── nginx/default.conf # Proxies /api → backend:8000, /mlflow → mlflow:5000
│   └── Dockerfile
├── models/                # Trained .pkl files (gitignored — transfer via scp)
│   ├── lgbm_model.pkl
│   ├── xgboost_model.pkl
│   ├── kmeans_model.pkl
│   ├── gmm_model.pkl
│   ├── scaler.pkl
│   ├── le_family.pkl
│   ├── le_type.pkl
│   └── features.json
├── notebooks/             # Training notebook + MLflow runs
├── data/                  # Raw CSVs (train, stores, oil, holidays)
├── kubernetes/            # K8s manifests (next phase)
├── scripts/               # Data loading + SQL feature engineering
├── mlflow/                # MLflow Dockerfile
├── docker-compose.yml
└── .env.example
```

---

## Local Development

```bash
# 1. Clone
git clone https://github.com/alyrraza/retailsense-ai.git
cd retailsense-ai

# 2. Copy env
cp .env.example .env

# 3. Copy model files (not in git)
# Place your .pkl files in ./models/

# 4. Run all services
docker compose up -d --build

# 5. Access
# Frontend:  http://localhost:3000
# Backend:   http://localhost:8000/docs
# MLflow:    http://localhost:5000
# MinIO:     http://localhost:9000
```

---

## EC2 Deployment

```bash
# On your local machine — copy models
scp -i "your-key.pem" -r ./models ubuntu@<EC2-IP>:~/retailsense-ai/

# On EC2
cd ~/retailsense-ai
cp .env.example .env
docker compose up -d --build

# Verify
docker compose ps
curl http://localhost:8000/
```

**Required EC2 Security Group inbound ports:** 22, 3000, 8000, 5000, 9000

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Health check + loaded models |
| GET | `/metrics` | RMSE/MAE/MAPE for all models |
| POST | `/predict/forecast` | Sales forecast with CI |
| POST | `/predict/segment` | Store cluster (K-Means + GMM) |
| GET | `/explain/{store_nbr}` | SHAP feature importances |
| POST | `/ab-test` | Paired t-test: LightGBM vs XGBoost |
| GET | `/drift` | KS-test drift detection |

Full interactive docs: [http://16.171.161.243:8000/docs](http://16.171.161.243:8000/docs)

---

## Dataset

[Corporación Favorita Grocery Sales — Kaggle](https://www.kaggle.com/competitions/store-sales-time-series-forecasting)

- 54 stores · 33 product families · 4+ years of daily sales
- Supplementary: oil prices, national holidays, store metadata, transactions
