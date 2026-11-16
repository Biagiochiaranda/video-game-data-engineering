# Video Game Data Engineering | ETL, REST API & NoSQL

A Python-based data engineering project focused on building a reproducible ETL workflow to integrate video game sales data with optional external metadata, validate data quality, and support analytical exploration.

Developed by Biagio Chiarandà and Simone Fabio Taurisano for the Data Management course in the Master's Degree in Data Science at the University of Milano-Bicocca.

## Project overview

The project explores how video game sales relate to critical reception and how release coverage changes over time. Its primary engineering goal is to process heterogeneous sources through a structured workflow:

1. Ingest a historical CSV dataset of video game sales.
2. Validate its schema, standardize release dates, and profile missing values.
3. Optionally retrieve game metadata from the RAWG REST API and cache API responses.
4. Integrate structured and semi-structured records into a document-oriented JSON schema.
5. Optionally persist the documents to MongoDB Atlas.
6. Produce data-quality reports, statistical summaries, and visualizations.

The local CSV analysis runs without API or database credentials. RAWG enrichment and MongoDB persistence are optional extensions.

## Tech stack

- Python and Jupyter Notebook
- Pandas and NumPy for ingestion, transformation, and data validation
- Requests and REST APIs for optional metadata enrichment
- JSON for document-oriented data integration
- PyMongo and MongoDB Atlas for optional NoSQL persistence
- Matplotlib for data visualization

## Architecture

```text
                    +----------------------+      +----------------------+
                    |  Sales CSV (Kaggle)  |      |  RAWG REST API       |
                    +----------+-----------+      +-----------+----------+
                               |                              |
                               v                              v
                    +----------------------+      +----------------------+
                    | CSV ingestion and    |      | Optional API lookup, |
                    | data validation      |      | retries and caching  |
                    +----------+-----------+      +-----------+----------+
                               |                              |
                               +---------------+--------------+
                                               |
                                               v
                                +-----------------------------+
                                | Python integration and      |
                                | JSON document construction  |
                                +--------------+--------------+
                                               |
                              +----------------+----------------+
                              |                                 |
                              v                                 v
                   +---------------------+           +---------------------+
                   | Local JSON outputs  |           | MongoDB Atlas       |
                   | and quality reports |           | (optional)          |
                   +----------+----------+           +---------------------+
                              |
                              v
                   +---------------------+
                   | Analysis and charts |
                   +---------------------+
```

## Dataset

The primary source is the [Video Game Sales 2024 dataset on Kaggle](https://www.kaggle.com/datasets/asaniczka/video-game-sales-2024), which contains:

- 64,016 title-platform records and 39,798 distinct game titles.
- Information about platforms, publishers, genres, release dates, sales, and critic scores.
- Sales recorded in millions of units and critic scores measured on a 0–10 scale.

The optional [RAWG Video Games Database API](https://rawg.io/apidocs) provides additional metadata such as user ratings, rating counts, genres, and playtime. RAWG user ratings use a different 0–5 scale and must not be treated as equivalent to CSV critic scores.

The source CSV is not included in the supplied project files. Download it from Kaggle and place it at `Data/Source/video_game_sales.csv` before running the notebook. Check the dataset's license and usage terms before redistributing it.

## Repository structure

Arrange the project locally as follows. The root directory must be named `video_game_market` for the current notebook's path discovery logic to work without modification.

```text
video_game_market/
├── README.md
├── notebooks/
│   └── 01_sales_reception_and_integration.ipynb
├── docs/
│   ├── Research_Report.pdf
│   └── Project_Presentation.pdf
└── Data/
    ├── Source/
    │   └── video_game_sales.csv       # Download separately from Kaggle
    ├── RAWG_Cache/                    # Created automatically when needed
    └── Results/                       # Generated reports, CSVs and charts
```

## Getting started

### 1. Set up the local project

Download or clone this repository into a directory named `video_game_market`, then create the `Data/Source` folder if necessary and place `video_game_sales.csv` inside it.

### 2. Install dependencies

Python 3.10 or newer is recommended. From the project directory:

```bash
python -m venv .venv
source .venv/bin/activate          # On Windows: .venv\Scripts\activate
python -m pip install "pandas>=2.0" numpy "matplotlib>=3.6" requests pymongo jupyter ipython
```

### 3. Run the notebook offline (default)

From the project root:

```bash
jupyter notebook notebooks/01_sales_reception_and_integration.ipynb
```

Run the notebook cells in order. By default, `RUN_RAWG=0` and `WRITE_MONGODB=0`, so external API requests and database writes are disabled. Results are saved under `Data/Results/`.

### 4. Enable optional RAWG enrichment

Obtain your own RAWG API key, review the provider's terms and rate limits, and set the following environment variables before starting Jupyter:

```bash
export RAWG_API_KEY="your_api_key"
export RUN_RAWG=1
```

With this setting, the notebook may request API data for its selected records and cache responses locally. Exact normalized-title matches are accepted conservatively; editions and platform-specific releases may still require manual review.

### 5. Enable optional MongoDB persistence

Configure a MongoDB Atlas connection and enable writing:

```bash
export MONGODB_URI="your_mongodb_connection_string"
export WRITE_MONGODB=1
```

The notebook uses the `GamingDB` database and `games_integrated_revised` collection. Documents are written with deterministic identifiers and upsert semantics to reduce duplicate writes across repeated executions.

On Windows PowerShell, use `$env:VARIABLE_NAME = "value"` instead of `export`.

Never commit API keys, database credentials, `.env` files, or private API caches to a public repository.

## Data engineering highlights

- Data ingestion and schema validation for structured CSV records.
- Cleaning and type standardization while preserving unknown values rather than replacing them with zero.
- HTTP retry handling, timeouts, response caching, and explicit API match statuses.
- Integration of CSV data and optional JSON metadata into a flexible document schema.
- Deterministic record IDs based on title, platform, and release date.
- Optional idempotent writes to MongoDB using upserts.
- Quality audits for null values, missing release dates, and duplicate rows.

## Results

The completed static-data analysis reported:

| Metric | Result |
| --- | ---: |
| Source records | 64,016 |
| Distinct titles | 39,798 |
| Records with both sales and critic scores | 4,126 |
| Spearman correlation between sales and critic scores | 0.338 |
| Records selected for optional API enrichment | 200 |

The observed sales–critic-score association is positive but modest. It is descriptive, not evidence that reviews cause sales. Missing observations, non-random review coverage, and multiple platform releases of the same game limit generalization.

In the documented offline run, all 200 selected enrichment records remained partial because RAWG cache data was unavailable. Therefore, API-based rating trends and fully integrated MongoDB query results are not claimed as completed findings. Live API execution can produce different results over time.

## Generated outputs

The notebook writes files into `Data/Results/`, including:

- `source_quality.json` — source-level data quality summary.
- `enrichment_sample.csv` — deterministic sample selected for optional enrichment.
- `integrated_records.json` — merged records with explicit integration statuses.
- `static_analysis.json` — complete-case count and Spearman correlation.
- `release_coverage.png` — source release coverage by year.
- `sales_and_critic_scores.png` — sales versus critic-score density.
- `source_genre_coverage.png` — genre distribution in the source dataset.

Additional RAWG-based charts are generated only when matching API metadata is available.

## Documentation

- [Research report](docs/Research_Report.pdf) — methodology, data architecture, validation, analysis, and limitations.
- [Project presentation](docs/Project_Presentation.pdf) — overview of the workflow and key findings.
- [Jupyter notebook](notebooks/01_sales_reception_and_integration.ipynb) — implementation and reproducible analysis.

## Limitations and future work

The project is an academic data-engineering workflow, not a production-deployed service. The CSV analysis is reproducible with the source data; external enrichment requires valid credentials, network access, and fresh API responses. Suggested future improvements include edition-aware matching, timestamped API snapshots, automated tests, orchestration, and text-based review sentiment analysis.

## Authors

- Biagio Chiarandà
- Simone Fabio Taurisano

University of Milano-Bicocca — Master's Degree in Data Science, Data Management course.
