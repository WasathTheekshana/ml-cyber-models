# Section 3 - Anomaly Detection for Cyber Threats

Detects anomalous (attack) network traffic using unsupervised/semi-supervised learning, comparing a classic machine learning model against a deep learning model, as required by Section 3 (25 marks) of the assignment.

## What's here

| File | Purpose |
| --- | --- |
| `anomaly_detection.ipynb` | The notebook: data loading (Google Colab upload), cleaning, EDA, feature preparation, both models, evaluation, comparison, visualizations. |
| `requirements.txt` | Python dependencies if you want to run this outside Colab. Not needed in Colab itself - it ships with pandas/scikit-learn/TensorFlow preinstalled. |
| `data/cic.csv` | The dataset used by the notebook. |
| `results/` | Where you can save an executed copy of the notebook after running it, if you want a record of a specific run's output. |
| `outputs/` | Created automatically the first time you run the notebook; holds generated figures (`figures/`), saved models (`models/`), and the model comparison table (`model_comparison.csv`). Not present until you run it. |

## Dataset

**CIC-IDS2018** (Canadian Institute for Cybersecurity), specifically the `Wednesday-14-02-2018` capture (`cic.csv`, about 358MB): 1,048,575 network flows summarized by CICFlowMeter into numeric features, a `Protocol` field, a `Timestamp`, and a `Label` with 3 values: `Benign`, `FTP-BruteForce`, and `SSH-Bruteforce`.

For this anomaly-detection task, the two attack types are combined into a single `Anomaly` class, versus `Benign` as `Normal`. Both models are trained **only on normal traffic** and never see an attack example during training - they learn what normal looks like and flag deviations from it, which is the standard approach for detecting attack types a model has never seen before (the "unknown/zero-day" framing in the assignment brief).

**Data quality notes (handled by the notebook, Section 2 "Data Cleaning"):**
- `Timestamp` is dropped (not a numeric predictor).
- About 21.5% of rows are exact duplicates in the raw file (225,628 out of 1,048,575), concentrated heavily in the attack traffic (about 80% of `FTP-BruteForce` rows and about 37% of `SSH-Bruteforce` rows are duplicates, versus under 1% of `Benign` rows) - automated brute-force attempts tend to produce near-identical flow patterns. These are removed per the assignment's explicit preprocessing instruction.
- `Flow Byts/s` and `Flow Pkts/s` contain literal `Infinity`/`NaN` values for zero-duration flows, same issue as the CIC-IDS2017 data in Section 2; these rows are dropped.

## Models implemented

1. **Classic ML:** Isolation Forest, fit on normal-only training data.
2. **Deep Learning:** Dense Autoencoder, fit on normal-only training data, using reconstruction error as the anomaly score.

Both are evaluated with TPR, FPR, precision/recall/F1 (at a 95th-percentile threshold derived from normal training data), a confusion matrix, an ROC curve, and a precision-recall curve, then compared side by side.

## Running in Google Colab (recommended)

1. Have `cic.csv` ready on your computer (already present in this folder's `data/`).
2. Upload `anomaly_detection.ipynb` to Colab (**File -> Upload notebook**), or open it from Google Drive/GitHub.
3. Run cells from the top (**Runtime -> Run all**, or step through one at a time).
4. When the "Load Dataset" cell runs, it checks `data/cic.csv`, `/content/cic.csv`, and `/content/data/cic.csv`. If none of those exist, **it automatically opens Colab's file-upload dialog** - pick the CSV, and the notebook continues from there. At about 358MB, the upload itself may take a few minutes.
   - If you'd rather upload ahead of time via the Files sidebar, either drop it directly into `/content` or create a `data` folder there first - both are auto-detected.
5. Everything else (cleaning, EDA, feature preparation, both models, evaluation, plots) runs automatically. All packages used (`pandas`, `scikit-learn`, `tensorflow`, `matplotlib`, `seaborn`) are preinstalled in Colab, so `requirements.txt` isn't needed there.

**Important:** anything you upload in Colab only lives for the current runtime/session. If the runtime disconnects or restarts, you'll need to re-upload the CSV.

**Runtime expectations:** with about 1 million raw rows (roughly 820,000 after deduplication), loading and cleaning takes a few minutes. Isolation Forest fits on the normal-only subset (a few hundred thousand rows) in a couple of minutes; the autoencoder trains for up to 30 epochs with early stopping, which should take considerably less time per epoch than Section 2's CNN since both the row count and feature count here are smaller. Budget at least 15-20 minutes for a full run on Colab's standard CPU runtime.

## Running locally instead

1. Install Python 3.10+ and dependencies: `pip install -r requirements.txt`.
2. Make sure `data/cic.csv` exists relative to the notebook (it already does in this folder).
3. `cd` into this folder and run `jupyter notebook anomaly_detection.ipynb`, then run all cells top to bottom.

## What the notebook does

- Loads `cic.csv` and prints its shape.
- Drops `Timestamp`, removes exact duplicate rows, removes rows with infinite/missing values, and collapses the label to binary (`Normal` vs `Anomaly`).
- Shows exploratory plots (binary class balance, protocol type by class, connection duration and packet size by class).
- One-hot encodes `Protocol` and assembles the full numeric feature set.
- Splits into train/test (80/20, stratified), then further isolates the normal-only rows of the training split for fitting both models; scales features using only that normal-only subset.
- Trains and evaluates the Isolation Forest, then the Autoencoder, each using a threshold set from the 95th percentile of that model's own anomaly score on normal training data.
- Saves all figures and models to `outputs/`: `figures/*.png`, `models/scaler.joblib`, `models/isolation_forest.joblib`, `models/autoencoder.keras`, and `model_comparison.csv`.
- Prints a final side-by-side comparison table and chart.

## Troubleshooting

- **Colab: the upload dialog didn't appear / "Could not find the dataset" error:** re-run the "Load Dataset" cell - it only opens the dialog when the file isn't found. If the runtime restarted, a previously uploaded file is gone and needs re-uploading.
- **Colab: uploaded the file but it's still not found:** check exactly where it landed via the Files sidebar - it should be directly under `content` (i.e. `/content/cic.csv`) or `content/data`. A different filename is fine as long as it's the *only* `.csv` in that folder.
- **Local: `FileNotFoundError` even though the CSV is in `data/`:** the notebook's kernel working directory isn't this folder - a common VS Code Jupyter extension issue, since it runs kernels from the workspace root rather than the notebook's own folder. Launch Jupyter directly from this folder instead (`cd` here, then `jupyter notebook anomaly_detection.ipynb`).
- **Every prediction comes out "Normal" (or every prediction comes out "Anomaly"):** double-check that `X_train_normal_scaled` (used to fit both models and to compute the 95th-percentile threshold) actually only contains normal rows, and that the threshold was computed from that same normal-only score distribution, not from the mixed test set.
- **TensorFlow install issues (local only):** TensorFlow's official wheels support a specific range of Python versions/platforms - see https://www.tensorflow.org/install if `pip install tensorflow` fails on your system.
