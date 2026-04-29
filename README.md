# retailsense-ai

Scaffold for the RetailSense AI demand forecasting project.

Structure:

- backend/: FastAPI backend serving the model and monitoring utilities.
- frontend/: React application (to be added).
- notebooks/: Kaggle-style notebooks and experiments.
- data/: CSV datasets (put your CSVs here).
- kubernetes/: Kubernetes manifests for deployment.
- mlflow/: MLflow config and artifacts.
- scripts/: Utility scripts for data loading and SQL feature queries.

Get started:

1. Create a virtualenv and install backend requirements:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r backend/requirements.txt
```

2. Run the backend locally:

```bash
uvicorn backend.main:app --reload
```
