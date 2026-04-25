# Project Introduction and Run Order

This repository extracts behavioral features from CERT insider-threat datasets, then provides example scripts for:

- Supervised classification
- Unsupervised anomaly detection
- A research reproduction pipeline used in TNSM 2020 experiments

The project has two main stages:

1. Build extracted feature tables from raw CERT logs
2. Train/evaluate models on those extracted tables


## 1) What each file is for

- feature_extraction.py
  - Main ETL pipeline for raw CERT folders such as r4.2, r5.2, r6.2.
  - Reads action logs (logon, device, file, http, email), combines by week, engineers features, and writes CSV files to ExtractedData.
  - Also writes session and sub-session representations.

- temporal_data_representation.py
  - Builds temporal representations from extracted day/week CSV data:
    - concat
    - percentile
    - meandiff
    - meddiff
  - Outputs Pickle files (.pkl), used by the anomaly example.

- example_classification.py
  - Minimal RandomForest classification demo.
  - Expects a file named day-r5.2.csv.gz in the current directory.

- example_anomaly_detection.py
  - Minimal autoencoder-style anomaly demo (MLPRegressor reconstruction error).
  - Expects a file named week-r5.2-percentile30.pkl in the current directory.

- TNSM2020/run_classification.py
  - Reproduction-style classification experiment runner with configurable algorithms.
  - Uses helper logic in TNSM2020/clf_helpers.py.
  - Loads tuned parameters from TNSM2020/params.pkl.

- TNSM2020/clf_helpers.py
  - Data splitting, normalization, user-level metrics, confusion matrices, ROC/AUC helpers.

- README.md
  - Short project summary and links to papers/datasets.

- requirements.txt
  - Core dependencies for extraction and basic examples.
  - Note: xgboost is imported by TNSM2020/run_classification.py but not listed here.


## 2) Recommended run order

Use this order for a clean end-to-end run.

1. Install dependencies
2. Extract features from a raw CERT folder (feature_extraction.py)
3. (Optional) Build temporal representation (temporal_data_representation.py)
4. Run sample classification/anomaly demos
5. (Optional) Run TNSM2020 experimental pipeline


## 3) Step-by-step commands

Run from the repository root unless noted otherwise.

### Step A: Setup environment

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install xgboost
```

Why xgboost: TNSM2020/run_classification.py imports XGBClassifier.


### Step B: Prepare raw CERT dataset folder

feature_extraction.py must be executed inside a CERT dataset folder whose name is one of:

- r4.1, r4.2, r5.1, r5.2, r6.1, r6.2

That folder should contain files like:

- logon.csv, device.csv, file.csv, http.csv, email.csv
- LDAP/...

Then run extraction from inside that CERT folder.

Example (if your CERT folder is beside this repo):

```bash
cp /workspaces/feature-extraction-for-CERT-insider-threat-test-datasets/feature_extraction.py /path/to/r5.2/
cd /path/to/r5.2
python3 feature_extraction.py 8
```

Output (inside the CERT folder):

- ExtractedData/weekr5.2.csv
- ExtractedData/dayr5.2.csv
- ExtractedData/sessionr5.2.csv
- plus sub-session files (sessionnact25..., sessiontime120..., etc.)


### Step C: Run temporal representations (optional, needed for anomaly example)

From the same folder containing extracted CSV:

```bash
python3 /workspaces/feature-extraction-for-CERT-insider-threat-test-datasets/temporal_data_representation.py \
  --representation percentile \
  --file_input ExtractedData/weekr5.2.csv \
  --window_size 30
```

This creates a file like:

- weekr5.2-percentile30.pkl


### Step D: Run example scripts from this repository

The example scripts use hard-coded file names. Put expected files in repo root.

For example_classification.py, expected input is day-r5.2.csv.gz.

```bash
cd /workspaces/feature-extraction-for-CERT-insider-threat-test-datasets
gzip -c /path/to/r5.2/ExtractedData/dayr5.2.csv > day-r5.2.csv.gz
python3 example_classification.py
```

For example_anomaly_detection.py, expected input is week-r5.2-percentile30.pkl.

```bash
cp /path/to/r5.2/weekr5.2-percentile30.pkl /workspaces/feature-extraction-for-CERT-insider-threat-test-datasets/week-r5.2-percentile30.pkl
python3 example_anomaly_detection.py
```


### Step E: Run TNSM2020 experiments (optional)

This pipeline expects data files in TNSM2020/data and naming pattern:

- {dtype}{dataset}.csv.gz
- Example for week + r5.2: weekr5.2.csv.gz

Prepare data:

```bash
mkdir -p /workspaces/feature-extraction-for-CERT-insider-threat-test-datasets/TNSM2020/data
gzip -c /path/to/r5.2/ExtractedData/weekr5.2.csv > /workspaces/feature-extraction-for-CERT-insider-threat-test-datasets/TNSM2020/data/weekr5.2.csv.gz
```

Run:

```bash
cd /workspaces/feature-extraction-for-CERT-insider-threat-test-datasets/TNSM2020
python3 run_classification.py
```

Outputs include pickled result files and ROC image files.


## 4) When to run which script

- Run feature_extraction.py first whenever you start from raw CERT logs.
- Run temporal_data_representation.py when you need history-window features (especially for anomaly detection).
- Run example_classification.py for a quick supervised baseline demo.
- Run example_anomaly_detection.py for a quick unsupervised baseline demo.
- Run TNSM2020/run_classification.py when reproducing research-style classification experiments with richer reporting.


## 5) Important gotchas

- The extractor is strict about current working directory name (must look like r5.2, r4.2, etc.).
- Example script filenames use hyphenated names (day-r5.2.csv.gz, week-r5.2-percentile30.pkl), while extractor outputs non-hyphen names (dayr5.2.csv, weekr5.2.csv).
- requirements.txt does not include xgboost, but TNSM2020 script imports it.
- README mentions sample_classification.py/sample_anomaly_detection.py, but files in this repo are named example_classification.py/example_anomaly_detection.py.
