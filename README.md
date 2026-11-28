# ChurnIQ

ChurnIQ is a Streamlit portfolio project that explores customer churn in IBM's fictional telecom sample. It compares three classification models, scores individual or batch profiles, and shows how model scores respond to edits.


**Live demo** https://churn-iq-adnan-hamid-wani.streamlit.app/

## What you can do

| Page | Workflow |
| --- | --- |
| Overview | Compare churn and monthly charges across customer tenure groups, then inspect outcomes and contract churn. |
| Data Explorer | Filter customers and compare data distributions, category churn rates, and the outcome mix. |
| Predict Churn | Score a fictional customer and inspect the model's response to specific edits. |
| Model Performance | Compare cross-validation scores, inspect held-out results, and see global importance. |
| Batch Predictions | Upload fictional profiles and download customer IDs, predicted classes and probabilities. |

## Quick start

Use Python 3.12. Install the runtime dependencies and launch the app from the repository root.

```bash
python -m venv .venv
```

Activate it on Windows PowerShell with `.\.venv\Scripts\Activate.ps1` or on macOS and Linux with `source .venv/bin/activate`.

```bash
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

The app uses the released pipeline and metrics in `models/`. Overview and Data Explorer download and verify the IBM sample on first use if it is absent locally. Sample download requires network access. The prediction and performance pages can run from the released artifacts without it.

## Model results

The training data covers **4,225** customers, with **1,409** for validation and **1,409** held out for testing. A five-fold stratified grid search on the training split chose each model's settings. Mean validation-fold average precision selected the winner. Average precision summarizes how precision changes as recall increases and focuses on the minority churn class.

| Model | Mean 5-fold average precision |
| --- | ---: |
| Random forest | 0.6756 |
| Logistic regression | 0.6673 |
| Decision tree | 0.6181 |

The random forest with 120 trees, maximum depth 12, and at least 15 training samples per leaf ranked highest. The scores were close for the top two models. Validation F1 selected the **36.49%** classification threshold. On validation, that cutoff caught **260** churners and raised **176** false alerts, compared with **169** caught and **87** false alerts at 50%. The lower cutoff traded more outreach for fewer missed churners.

| Held-out metric | Value |
| --- | ---: |
| Accuracy | 77.71% |
| Precision | 56.38% |
| Recall | 70.86% |
| F1 | 0.6280 |
| ROC-AUC | 0.8427 |
| Average precision | 0.6520 |
| Brier score | 0.1364 |

| Held-out outcome | Predicted retained | Predicted churn |
| --- | ---: | ---: |
| Actual retained | 830 | 205 |
| Actual churn | 109 | 265 |

The model uses 15 service, billing and account-history features. Customer IDs, demographics and the churn label are excluded from fitted inputs. Missing values and categories are handled by a pipeline fitted within each training fold. Risk bands use half the selected threshold and the full threshold as their boundaries. Low is below **18.24%**, Medium starts there, and High begins at **36.49%**. Classification uses the unrounded threshold. [Machine-readable results](models/metrics.json)

## Reproduce the analysis and model

```bash
python -m pip install -r requirements-dev.txt
python -m scripts.download_data
python -m jupyter nbconvert --to notebook --execute --inplace notebooks/01_exploratory_data_analysis.ipynb
python -m src.train
python -m pytest -q
```

Training writes the fitted pipeline and evaluation JSON together. Two runs in the same Python 3.12 environment produced identical artifacts. For an isolated reproduction, run `python -m src.train --output path/to/reproduced-models`. The app uses compatible pinned runtime packages to load the bundled artifact. Load Joblib files only from a trusted project source.

The executed [EDA notebook](notebooks/01_exploratory_data_analysis.ipynb) checks the data types, missing values, duplicates, churn balance and associations. The primary dataset is a pinned 7,043-row, 21-column IBM sample. See the [dataset source, checksum and manual instructions](docs/DATASET.md).

## Batch CSVs

Download the fictional example from Batch Predictions for the exact header and category spelling. Include one unique, nonempty `customerID` per row and the 15 feature headers. Fields may be blank if each row has at least one populated model input. The app checks malformed rows, unknown categories, invalid numbers, and contradicting service options. Extra columns are ignored. The preview shows up to 50 rows and the export follows the original row order. Formula-like IDs receive a leading apostrophe in the exported CSV for spreadsheet use.

## Screenshots

![ChurnIQ overview with tenure comparison, customer outcomes and contract churn](docs/screenshots/overview.png)

[Filtered outcome chart](docs/screenshots/explorer.png) · [Single-customer prediction](docs/screenshots/prediction.png) · [Model performance](docs/screenshots/performance.png)

## Project files

```text
app.py                      Streamlit entry point and navigation
views/                      Five Streamlit pages
assets/                     Shared dashboard styles
src/                        Data, schema, training, inference and explanations
scripts/download_data.py    Reproducible IBM sample download
notebooks/                  Executed exploratory analysis
models/                     Trained pipeline and machine-readable metrics
tests/                      Pytest and Streamlit app checks
docs/                       Requirements, technical design, dataset and release guide
tasks/                      Implementation checklist
```

## Explanations and limits

Global permutation importance measures the average drop in held-out average precision when a feature is shuffled. The single-customer page reports the changes in model score after replacing specific input groups with observed training reference values. These responses depend on the chosen edits, and correlated inputs can influence both forms of explanation.

The IBM sample comes from a fictional provider, with one snapshot and no chronological test. Results may vary across datasets, time periods or providers. Probability scores are model estimates without a separate calibration step. Removing demographic inputs leaves open the possibility that other fields act as proxies. The model is for learning and demonstration, and the dashboard uses fictional inputs.

## Deployment and portfolio notes

The [release guide](docs/DEPLOYMENT.md) covers publishing to GitHub, configuring Streamlit Community Cloud, and smoke-testing the public app.

- [Beginner-friendly project explanation](PROJECT_EXPLANATION.md)
- [20 technical interview questions and answers](INTERVIEW_PREPARATION.md)
- [Product requirements](docs/PRD.md) · [Technical design](docs/TECHNICAL_DESIGN.md)

Original ChurnIQ code uses the [MIT license](LICENSE). IBM's sample is attributed separately in [DATASET.md](docs/DATASET.md).
