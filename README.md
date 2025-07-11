# Credit‑Scoring AI‑Act Demo

> Demo project to showcase EU AI Act compliance artifacts with ZenML.

## Project Structure

```bash
credit_scoring_ai_act/
├── data/
│   ├── raw/ # public synthetic loan data (CSV)
│   └── snapshots/ # auto‑versioned by data_versioning step
├── pipelines/
│   ├── credit_scoring.py # "happy path" train‑→‑deploy
│   ├── monitor.py # scheduled drift checker
│   └── retrain.py # triggered by incident label
├── steps/
│   ├── data_loader.py # load CSV → log SHA‑256, WhyLogs profile
│   ├── data_preprocessor.py # basic feature eng, outputs train/test splits
│   ├── data_splitter.py # split dataset into train/test
│   ├── generate_compliance_metadata.py # generate compliance metadata
│   ├── train.py # XGBoost / sklearn model
│   ├── evaluate.py # std. metrics + Fairlearn/Aequitas scan
│   ├── approve.py # human‑in‑loop gate (approve_deployment step)
│   ├── post_market_monitoring.py # post‑market monitoring
│   ├── post_run_annex.py # generate Annex IV documentation
│   ├── risk_assessment.py # risk assessment
│   └── deploy.py # push to Modal / local FastAPI
│
├── compliance/
│   ├── templates/
│   │   ├── annex_iv_template.j2 # annex iv template
│   │   └── fria_template.md # rights‑impact narrative
│   ├── records/ # automated compliance records
│   ├── manual_fills/ # manual compliance inputs
│   ├── monitoring/ # monitoring records
│   ├── deployment_records/ # deployment history
│   └── approval_records/ # approval history
│
├── reports/ # auto‑generated Docs, PDFs, JSON logs
│   ├── annex_iv_<run>.pdf
│   ├── model_card_<run>.json
│   └── incident_log.json
├── utils/ # shared utilities
├── configs/ # configuration files
└── README.md
```

### Pipeline graph

1. `ingest -> preprocess -> train -> evaluate -> approve_deploy -> deploy`
2. **Scheduled**: `monitor` (daily) → `report_incident` on drift.  
   Incident closed? → GitHub label **retrain** triggers `retrain` pipeline.

### Where each compliance artifact is produced

| Step / Hook             | AI‑Act Article(s) | Output artefact                                |
| ----------------------- | ----------------- | ---------------------------------------------- |
| **ingest**              | 10, 12            | `dataset_info` metadata, SHA‑256               |
| **preprocess**          | 10, 12            | `preprocessing_info_*.json`, data quality logs |
| **evaluate**            | 15                | `fairness_metrics`, `metrics.json`             |
| **approve_deploy**      | 14                | `approval.log` (approver, rationale)           |
| **post_run_annex.py**   | 11, 12            | `annex_iv_<run>.pdf`                           |
| **monitor.py**          | 17                | drift scores in `reports/drift_<date>.json`    |
| **incident_webhook.py** | 18                | `incident_log.json`, external ticket           |
| **risk_assessment.py**  | 9                 | risk scores, risk register updates             |

### Compliance Directory Structure

| Directory               | Purpose                                                 | Auto/Manual |
| ----------------------- | ------------------------------------------------------- | ----------- |
| **records/**            | Automated compliance records from pipeline runs         | Auto        |
| **manual_fills/**       | Manual compliance inputs and preprocessing info         | Manual      |
| **monitoring/**         | Post-market monitoring records and drift detection logs | Auto        |
| **deployment_records/** | Model deployment history and model cards                | Auto        |
| **approval_records/**   | Human approval records and rationales                   | Manual      |
| **templates/**          | Jinja templates for documentation generation            | Manual      |

### "Definition of Done"

```bash
zenml up && zenml run credit_scoring
# -> reports/annex_iv_<run>.pdf exists
# -> fairness_metrics logged
# -> approval gate records reviewer
# -> drift monitor cronjob scheduled
# -> compliance/records/ contains run artifacts
# -> compliance/manual_fills/ contains required inputs
```
