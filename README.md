# Formula Racing Prediction System

A machine learning system that predicts driver qualifying and race finishing
positions across formula-style racing series (F1, F2, F3, Formula E, and
beyond), using free public data sources only.

**Status:** Architecture & planning complete. Data foundation in progress.
Model implementation not started.

---

## What this project does

- Predicts qualifying results before they happen
- Predicts race finishing order, both before and after qualifying
- Supports any formula-style series via a configurable format adapter
- Uses historical data as far back as it exists (F1 back to 1950)
- Normalizes performance across regulation eras so drivers/cars from
  different decades are comparable
- Ingests real-time data: weather forecasts, FIA penalty documents, news
- Tracks season state: standings, championship gaps, calendar
- Outputs, per prediction: predicted finish order, a confidence score,
  championship points impact, and a **per-prediction factor breakdown**
  (which model/factor drove *this specific* predicted result, via SHAP)
- Computes model weights dynamically from validation accuracy — weights
  are never hand-set

Non-commercial project. Every data source used is free.

---

## Architecture at a glance

```
Data Layer (historical DB + real-time API polling + LLM extraction)
        │
Preprocessing (M1: Era Adapter — normalizes historical data into
                era-agnostic scores before anything else touches it)
        │
Model Layer (M2–M20: 18 trained models + M11/M12 rules-based layers)
        │
M21: Per-Prediction Attribution (SHAP — explains one prediction after
     the fact; does not feed back into model weights)
        │
Weighted Ensemble + Context Modifiers (weights from RMSE, shifted by
     rain, street circuits, championship-deciding races, etc.)
        │
Final Output: predicted order, confidence, points impact, attribution
```

21 models total: 18 are trained ML models, 2 (`M11` Season State,
`M12` Series Format Adapter) are rules-based/config layers with no
training step, and 1 (`M21`) is a post-hoc explainability layer.

154 individually numbered, permanent factor IDs (`F1`–`F154`) feed the
models. IDs are never renumbered or reused once assigned.

Full model definitions, factor list, dataset mappings, and design
decisions live in
[`docs/formula_racing_prediction_system.txt`](docs/formula_racing_prediction_system.txt) —
that file is the canonical architecture reference for this project.

---

## Repo structure

```
formula-racing-prediction-system/
├── data/
│   └── primary/                  # Downloaded source datasets (tracked in git)
│       ├── kaggle_rohanrao/      # 14 CSVs, F1 results 1950–present
│       ├── f1db/                 # ~45 CSVs, most comprehensive free F1 DB
│       └── tracinginsights/      # Optional backup/cross-reference mirror
├── docs/
│   ├── formula_racing_prediction_system.txt   # Full architecture reference
│   ├── formula_racing_full_execution_plan.pdf # Numbered step-by-step build plan
│   ├── session_log.txt                        # Environment/tooling setup log
│   ├── data_sources_reference.md              # All 18 data sources, categorized
│   └── data_sources_manifest.md               # Log of what's actually downloaded
└── README.md
```

Not yet created (deferred until the code skeleton phase begins):
`data/derived/`, `data/cache/`, `models/`, `pipelines/`, `ensemble/`,
`llm/`, `api/`, `tests/`, `environment.yml`.

`data/raw/`, `data/derived/`, `data/cache/`, `fastf1_cache/`, virtual
environments, and secrets/`.env` files are gitignored — everything
under `data/primary/` is intentionally tracked.

---

## Getting started (once environment setup is done)

This project uses **Anaconda/conda**, not a plain venv, and **Spyder**
as the primary IDE (not VS Code).

```bash
conda create -n formula_predictor python=3.11 -y
conda activate formula_predictor
conda install pandas numpy scipy scikit-learn xgboost lightgbm psycopg2 \
    sqlalchemy beautifulsoup4 requests fastapi uvicorn pytest spyder-kernels -y
pip install fastf1 shap google-generativeai groq openai-whisper python-dotenv
```

Then point Spyder's interpreter at the new environment
(**Tools → Preferences → Python Interpreter**) and restart the kernel.

See `docs/formula_racing_full_execution_plan.pdf` for the exact,
numbered (`Phase.Step.Substep`) build order, including which steps
need a git commit and what commit message to use.

---

## Data sources

18 primary sources, all free — Kaggle's Rohanrao F1 dataset, F1DB,
Jolpica (Ergast-compatible live API), OpenF1 (2023+ telemetry),
FastF1 (2018+ telemetry package), Open-Meteo (historical weather),
FIA documents (via LLM extraction), Wikipedia, and more.

Full categorization — what's downloaded vs. a live API vs. scraped vs.
an LLM service — is in `docs/data_sources_reference.md`. What has
actually been pulled down so far is tracked in
`docs/data_sources_manifest.md`.

**Known link corrections** (differ from the original architecture doc —
see `session_log.txt` for details):
- Open-Elevation is unreliable → use `api.opentopodata.org` instead
- Pirelli's motorsport URL has moved
- Kaggle/HuggingFace links are prone to copy-paste corruption
  (stray unicode in the URL) — retype manually if a link 404s

---

## Key design decisions

- **Dynamic weights only.** No model's importance is hand-set. Weights
  come from `1/RMSE` on validation data, renormalized, and shift with
  context (rain, street circuits, title-deciding races).
- **Era normalization first.** `M1` converts raw stats into
  era-agnostic scores before any other model trains on them.
- **Qualifying has two modes.** Pre-qualifying (predicted) and
  post-qualifying (actual times), the latter always taking priority
  when available.
- **LLMs extract, they don't predict.** Gemini/Groq turn PDFs and news
  into structured data; they never make a race prediction themselves.
- **Per-prediction attribution is separate from ensemble weighting.**
  Section 5-style weights set the *priors* before a race; `M21` (SHAP)
  measures what the trained model *actually* relied on for one specific
  driver/race, after the fact. One is global/pre-race, the other is
  per-prediction/post-hoc — they're complementary, not duplicates.

Full rationale for all ten-plus design decisions is in Section 3 of
the architecture reference.

---

## Roadmap

1. ~~Environment + repo + primary datasets~~ ✅
2. Conda environment, PostgreSQL, load primary data
3. Build the 15 derived datasets (wet-race flags, circuit classification,
   corner speed profiles, etc.)
4. Train `M1` (Era Adapter), then core models (`M2`–`M5`, `M13`, `M18`, `M19`)
5. Train supporting and complex models (`M6`–`M10`, `M14`–`M17`, `M20`)
6. Build the weighted ensemble, `M11`/`M12` context layers, and `M21`
   attribution logging
7. Real-time race-weekend pipeline
8. F2/F3 expansion

See `docs/formula_racing_full_execution_plan.pdf` for the full numbered
plan with per-step git checkpoints.

---

## License

Non-commercial personal project. All upstream data sources retain their
own licenses (CC0 for Kaggle Rohanrao, LGPL-3.0 for FastF1, etc.) —
see `docs/data_sources_reference.md` for details per source.
