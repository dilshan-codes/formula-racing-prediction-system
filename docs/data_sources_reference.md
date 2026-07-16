# Data Sources Reference — Master Categorization

All 18 primary data sources from the project design doc (Section 8),
categorized by HOW they're used. This is the single source of truth
for "what do we do with each source." Keep updated as sources are added.

Categories:
  📦 Downloadable file  — get once, store in data/primary/, commit to repo
  🔗 Live API           — queried by code at runtime, never stored as a file
  🕸️ Scraping target     — no API/bulk download, scraped by code later
  🐍 Python package      — installed via pip, not a data file
  🔑 LLM service         — needs free account + API key, not a dataset itself

---

## 📦 DOWNLOADABLE FILES
(Download once → extract → store in data/primary/{source_name}/ → commit)

### 1. Kaggle Rohanrao F1 Championship Dataset
- Link: https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020
- Status: DOWNLOADED (see data_sources_manifest.md for date/version)
- Target folder: data/primary/kaggle_rohanrao/
- Requires: free Kaggle account login to download

### 2. F1DB
- Link: https://github.com/f1db/f1db/releases
- File to grab: f1db-csv.zip (NOT the JSON/SQL/SMILE variants — CSV matches our pipeline)
- Status: DOWNLOADED (see manifest)
- Target folder: data/primary/f1db/

### 13. TracingInsights / HuggingFace F1 mirrors
- Link: https://huggingface.co/datasets (search "F1")
- Status: NOT YET CHECKED — verify if direct CSV download exists before treating as API
- Target folder (if downloadable): data/primary/tracinginsights/

---

## 🔗 LIVE APIs
(Never downloaded/stored — called by pipeline/ingestion or pipeline/realtime code later)

### 3. Jolpica API
- URL: https://api.jolpi.ca/ergast/f1/
- Live + historical F1 results (Ergast-compatible). No key required.
- Key endpoints: /f1/{year}/{round}/results, /f1/current/driverStandings, /f1/circuits
- Used by: M11 Season State tracker, realtime pipeline (Phase 8)

### 4. OpenF1 API
- URL: https://api.openf1.org
- Real-time + historical telemetry/session data, 2023+. No key required.
- Rate limit: 3 req/s, 30 req/min
- Used by: M9 Incident/SC Probability, M13 Race Start/Lap 1, realtime pipeline

### 6. Open-Meteo Archive API
- URL: https://archive-api.open-meteo.com/v1/archive
- Historical weather, any GPS location, 1940-present. No key required.
- MUST include query params (lat, lng, dates, hourly fields) — bare URL errors, that's expected
- Used by: DERIVED-1 wet_race_flags.csv, M6 Weather Impact

### 7. Open-Elevation API → REPLACED with Open Topo Data
- Original (unreliable): api.open-elevation.com
- USE INSTEAD: https://api.opentopodata.org/v1/srtm30m?locations={lat},{lng}
- Used by: F25, F98 (circuit altitude)

### 8. TimeZoneDB
- URL: https://timezonedb.com/api
- Requires free registration for API key. Rate limit: 1 req/sec, 1M req/month.
- Used by: DERIVED-2 circuit_type_classification.csv (timezone field)

---

## 🕸️ SCRAPING TARGETS
(No API, no bulk download — accessed via BeautifulSoup/scraping code later, per-page/per-document)

### 9. Wikipedia
- No single URL — per-page: en.wikipedia.org/wiki/{Page_Name}
- Used for: driver height/weight, circuit infoboxes, season history, TD history

### 10. FIA Documents
- URL: https://www.fia.com/documents
- PDFs (stewards decisions, technical directives, penalty bulletins) — extracted via
  Gemini/Groq LLM, not scraped as plain text
- Used by: DERIVED-7 penalty_events_history.csv, M7 Penalty Risk, M17 Technical Directive

### 11. Motorsport Stats
- URL: https://motorsportstats.com
- No official API — covers F2/F3/Formula E results not available via Jolpica/OpenF1
- Used by: DERIVED-14 f2_f3_results_database.csv (Phase 9)

### 12. UK Companies House
- URL: https://find-and-update.company-information.service.gov.uk
- Free public filings for UK-registered teams (Mercedes, McLaren, Williams, Red Bull, Haas)
- Used by: F63, F64 (budget proxy)

### 17. Pirelli Press Releases
- URL: https://www.pirelli.com/tyres/en-ww/motorsport/car/formula-1
- (Corrected URL — original doc's pirelli.com/global/en-ww/race/formula1 has moved)
- Used by: F127, F128 (tire compound allocation)

### 18. FIA Formula 2 / Formula 3 Official Sites
- Links: https://www.fiaformula2.com / https://www.fiaformula3.com
- (Both recently relaunched with new site designs — expect layout changes if scraping)
- Used by: F2/F3 equivalents of all driver/team/race factors (Phase 9)

---

## 🐍 PYTHON PACKAGES
(Installed via pip when setting up the Anaconda environment — not manually downloaded)

### 5. FastF1
- Install: pip install fastf1
- Docs: https://docs.fastf1.dev
- Provides: lap timing, telemetry, weather, session results, 2018+

### 16. Whisper (Speech-to-Text)
- Install: pip install openai-whisper
- Model: https://huggingface.co/openai/whisper-large-v3 (downloads automatically on first use)
- Used for: team radio transcription (F61, F68)

---

## 🔑 LLM SERVICES (signup required, not datasets)

### 14. Google Gemini API
- Sign up: https://aistudio.google.com
- Free tier: 15 req/min, 1M tokens/day
- Store key in .env as GEMINI_API_KEY (never commit)

### 15. Groq API
- Sign up: https://console.groq.com
- Free tier: 14,400 req/day (Llama 3.3 70B)
- Store key in .env as GROQ_API_KEY (never commit)

---

## Quick Count
- Downloadable files: 3 (2 confirmed done, 1 to verify)
- Live APIs: 5
- Scraping targets: 6
- Python packages: 2
- LLM services: 2
- **Total: 18** (matches Section 8 of the design doc exactly)
