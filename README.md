# Precision-Health Omics + LLM


**Purpose.** A realistic portfolio project for roles in bioinformatics, precision public health, and AI platforms. It shows: (i) omics data QC + modeling, (ii) LLM-assisted literature structuring, (iii) microservices/API + Docker.


### Highlights
- **Omics pipeline**: missingness, batch checks, PCA; longitudinal/cross-sectional stats; simple classifier/regressor with XGBoost.
- **Literature mining**: PubMed E-utilities ingest → summarization + keyphrase/NER → Postgres tables.
- **API**: FastAPI endpoints for predictions and literature lookup.
- **Infra**: Docker + docker-compose; PostgreSQL + optional MongoDB; GitHub Actions CI.


> Use any public proteomics dataset (e.g., PRIDE/CPTAC) or your own CSVs with columns: `sample_id`, `subject_id`, `batch`, `timepoint`, features `prot_*`, and labels like `case_control` or a numeric phenotype.


---
## Quickstart


```bash
# 0) Clone & env
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env # set DB creds & API keys if any


# 1) Run API + DB with Docker
docker compose up -d # starts postgres + api service


# 2) Prepare demo data (creates small synthetic proteomics + a few PubMed abstracts)
python scripts/demo_data_prep.py
python scripts/fetch_pubmed.py --query "aging proteomics biomarkers" --retmax 50


# 3) Run pipelines
python -m src.pipelines.omics_qc --in data/raw/proteomics.csv --out data/processed/qc_report.json
python -m src.models.train_classifier --in data/processed/train.csv --target case_control --out artifacts/model.joblib


# 4) Start API (if not using compose)
uvicorn src.app.main:app --reload


# 5) Hit endpoints
curl localhost:8000/health
curl -X POST localhost:8000/omics/predict -H "Content-Type: application/json" -d '{"features": {"prot_A": 0.5, "prot_B": 1.2}}'
curl "localhost:8000/literature/search?term=proteomics+aging"
