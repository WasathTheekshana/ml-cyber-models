# Section 2 - Cyber Attack Detection

Classifies network traffic flows into normal (BENIGN) traffic versus multiple specific attack types, comparing a classic machine learning model against a deep learning model, as required by Section 2 (25 marks) of the assignment.

## What's here

| File | Purpose |
| --- | --- |
| `cyber_attack_detection.ipynb` | The notebook: data loading (Google Colab upload), cleaning, EDA, PCA feature selection, both models, evaluation, comparison, visualizations. |
| `requirements.txt` | Python dependencies if you want to run this outside Colab. Not needed in Colab itself - it ships with pandas/scikit-learn/TensorFlow preinstalled. |
| `data/` | The 8 CIC-IDS2017 CSV files used by the notebook. |
| `outputs/` | Created automatically the first time you run the notebook; holds generated figures (`figures/`), saved models (`models/`), and the model comparison table (`model_comparison.csv`). Not present until you run it. |

## Dataset

**CIC-IDS2017** (Canadian Institute for Cybersecurity), specifically the widely-used "MachineLearningCSV" flow-feature export: 8 CSV files, one per working day / attack scenario, each row a network flow summarized by CICFlowMeter into 78 numeric features plus a `Label` column. Combined: 2,830,743 flows across 15 classes (1 BENIGN class + 14 attack types).

Expected files (exact names, matching the official distribution including its inconsistent capitalization and the "Infilteration" typo):

- `Monday-WorkingHours.pcap_ISCX.csv`
- `Tuesday-WorkingHours.pcap_ISCX.csv`
- `Wednesday-workingHours.pcap_ISCX.csv`
- `Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv`
- `Thursday-WorkingHours-Afternoon-Infilteration.pcap_ISCX.csv`
- `Friday-WorkingHours-Morning.pcap_ISCX.csv`
- `Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv`
- `Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv`

**Data quality notes (handled by the notebook, Section 2 "Data Cleaning"):**
- Column names have inconsistent leading whitespace (e.g. `" Label"`).
- The column `Fwd Header Length` appears twice in the raw header; pandas auto-renames the duplicate to `Fwd Header Length.1` rather than erroring.
- Some `Web Attack` labels contain a Unicode replacement character (`�`) in place of a dash, baked into the dataset at the source; the notebook replaces it with a plain hyphen.
- Columns `Flow Bytes/s` and `Flow Packets/s` contain literal `Infinity`/`NaN` values for zero-duration flows; these are converted to `NaN` and dropped.
- This particular CSV export contains only numeric flow features (no `Protocol`, IP, or timestamp columns), so the only encoding needed is on the target `Label` itself.

**Class imbalance:** BENIGN traffic is about 80% of all rows, while some attack classes are extremely rare (Heartbleed: 11 samples, Web Attack - Sql Injection: 21 samples, Infiltration: 36 samples, across the *entire* dataset). This is a real, documented characteristic of CIC-IDS2017, not a bug, and it is discussed in the notebook's Discussion section along with what it means for evaluation (macro-averaged metrics and per-class precision-recall curves are used specifically because of this).

## Models implemented

1. **Classic ML:** PCA feature selection (95% variance retained) + Random Forest, multi-class, `class_weight="balanced"`.
2. **Deep Learning:** 1D CNN over the same PCA-reduced features, multi-class softmax output.

Both are evaluated with accuracy, macro-averaged precision/recall/F1, a confusion matrix (raw counts and row-normalized), and one-vs-rest precision-recall curves per class, then compared side by side.

## Running in Google Colab (recommended)

1. Have the 8 CSV files listed above ready on your computer (already present in this folder's `data/`).
2. Upload `cyber_attack_detection.ipynb` to Colab (**File -> Upload notebook**), or open it from Google Drive/GitHub.
3. Run cells from the top (**Runtime -> Run all**, or step through one at a time).
4. When the "Load Dataset" cell runs, it checks `data/<filename>`, `/content/<filename>`, and `/content/data/<filename>` for each of the 8 expected files. **For whichever ones it can't find, it automatically opens Colab's upload dialog** - you can select multiple files at once there, so you can upload all 8 in a single dialog.
   - If you'd rather upload ahead of time via the Files sidebar, either drop them directly into `/content` or create a `data` folder there first - both are auto-detected.
5. Everything else (cleaning, EDA, PCA, both models, evaluation, plots) runs automatically. All packages used (`pandas`, `scikit-learn`, `tensorflow`, `matplotlib`, `seaborn`) are preinstalled in Colab, so `requirements.txt` isn't needed there.

**Important:** anything you upload in Colab only lives for the current runtime/session. Since this dataset is about 847MB combined, re-uploading it after a runtime restart/disconnect will take a while - that is expected with the browser-upload workflow this notebook uses.

**Runtime expectations:** with about 2.8 million rows, loading and cleaning the data takes a few minutes, and training both models takes considerably longer than Section 1's notebook. In an actual test run on Colab's standard CPU runtime, the CNN alone took roughly 215-265 seconds per epoch across 14-15 epochs before early stopping triggered - about 50-65 minutes for CNN training alone, on top of data loading, cleaning, and Random Forest fitting. Budget at least an hour for a full run end to end on CPU; a GPU runtime (**Runtime -> Change runtime type**) will speed up the CNN step but not the Random Forest step, since scikit-learn does not use GPUs.

## Running locally instead

1. Install Python 3.10+ and dependencies: `pip install -r requirements.txt`.
2. Make sure all 8 CSV files exist under `data/` relative to the notebook (they already do in this folder).
3. `cd` into this folder and run `jupyter notebook cyber_attack_detection.ipynb`, then run all cells top to bottom.

## What the notebook does

- Loads and concatenates the 8 CSV files, printing the shape of each and the combined total.
- Cleans column names, fixes the corrupted label character, removes infinite/missing values.
- Shows a log-scale class distribution plot to make the severe imbalance visible.
- Encodes the multi-class label, does an 80/20 stratified split, standardizes features.
- Applies PCA (95% variance) as the feature-selection step, shared by both models.
- Trains and evaluates the Random Forest model, then the CNN model.
- Saves all figures and models to `outputs/`: `figures/*.png`, `models/scaler.joblib`, `models/pca.joblib`, `models/label_encoder.joblib`, `models/random_forest.joblib`, `models/cnn_model.keras`, and `model_comparison.csv`.
- Prints a final side-by-side comparison table and chart.

## Troubleshooting

- **Colab: the upload dialog didn't appear / files still missing after uploading:** re-run the "Load Dataset" cell - it only opens the dialog for files it couldn't find at that point. Check the Files sidebar to confirm exactly where an uploaded file landed (it should be directly under `content` or `content/data`) and that its filename matches the expected list exactly, including capitalization.
- **`MemoryError` or the Colab runtime crashes/restarts during loading:** this is a large dataset (about 847MB, 2.8 million rows). Make sure you're on a Colab runtime with enough RAM (the free tier's standard runtime is usually sufficient but can be tight); avoid running other memory-heavy cells in the same session.
- **Very slow Random Forest or CNN training:** this is expected with 2.8 million rows; consider switching to a Colab GPU runtime (**Runtime -> Change runtime type**) to speed up CNN training, though this will not speed up the Random Forest step (scikit-learn does not use GPUs).
- **Warnings about undefined precision/recall for a class:** this happens for the extremely rare classes (Heartbleed, Web Attack - Sql Injection, Infiltration) when a model predicts zero instances of that class; `zero_division=0` is set everywhere so this produces a `0.0` metric instead of crashing, but treat those specific numbers as unreliable given how few samples exist.
- **TensorFlow install issues (local only):** TensorFlow's official wheels support a specific range of Python versions/platforms - see https://www.tensorflow.org/install if `pip install tensorflow` fails on your system.
