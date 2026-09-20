# Section 4 - Ransomware Detection and Prevention

Classifies Windows executables as malicious or benign from the sequence of Windows API calls each one made, comparing a classic machine learning model against a deep learning model, as required by Section 4 (25 marks) of the assignment.

## What's here

| File | Purpose |
| --- | --- |
| `ransomware_detection.ipynb` | The notebook: data loading (Google Colab upload), cleaning, deduplication, EDA, feature extraction, both models, evaluation, comparison, visualizations. |
| `requirements.txt` | Python dependencies if you want to run this outside Colab. Not needed in Colab itself - it ships with pandas/scikit-learn/TensorFlow preinstalled. |
| `data/new_dataset.csv` | The dataset used by the notebook. |
| `results/` | Where you can save an executed copy of the notebook after running it, if you want a record of a specific run's output. |
| `outputs/` | Created automatically the first time you run the notebook; holds generated figures (`figures/`), saved models (`models/`), and the model comparison table (`model_comparison.csv`). Not present until you run it. |

## Dataset

**Windows API Call Sequence Dataset** (`new_dataset.csv`, about 1.5MB): 3,940 samples, each recording up to the first 152 Windows API calls made by an executable during dynamic analysis, already integer-encoded into 308 API call categories (IDs 0-307), plus a `hash` identifier and a binary `malware` label (`1` = malicious, `0` = benign).

**Data quality notes (handled by the notebook, Sections 2-3):**
- The raw CSV has 177 columns; 22 trailing columns are empty for every row except one (that one row has a longer recorded sequence than every other sample). Only the columns `hash`, `malware`, and the fixed 152 sequence positions (`"0"` through `"151"`) are used, so this doesn't need special handling beyond selecting the right columns.
- **Most samples do not use all 152 sequence positions.** Only 3 of the 3,940 rows have a value in every one of them; the real recorded sequence averages 62.9 calls (range 2-152), and the CSV pads every row out to 152 columns with empty cells afterward - 350,911 empty cells in total across the dataset. Pandas reads these as `NaN`, which cannot be cast directly to an integer dtype (attempting to do so raises `IntCastingNaNError`). The notebook fills them with a dedicated padding sentinel (`PAD_VALUE = 308`, one past the highest real API call ID - `0` cannot be reused for this, since `0` is itself a common real API call ID) before converting to integers. This sentinel is excluded from the SVM's bag-of-calls counts and included in the LSTM's embedding vocabulary size (`VOCAB_SIZE = 309`).
- Checking the raw data directly showed only 2,298 of the 3,940 rows have a unique 152-column sequence (including its padding) - about 49% are exact duplicates of another row's full sequence (one sequence alone repeats 164 times), which is expected for malware datasets since many samples are variants of the same underlying family. These are removed before splitting into train/test, for the same train/test-leakage reason as every other section in this assignment. A useful side effect: after deduplication the two classes end up almost perfectly balanced (about 50/50), down from about 67/33 in the raw file.

## Feature extraction and models

Two different feature representations are built from the same underlying sequence, matched to what each model needs:

1. **Classic ML (SVM):** a 308-dimensional "bag of API calls" - how many times each API category appears anywhere in a sample's *real* (non-padded) sequence, regardless of order - plus two engineered features: the number of distinct calls used, and the real sequence length. This is the standard feature-extraction technique for this kind of categorical sequence data with classic ML models.
2. **Deep Learning (LSTM):** the raw, ordered, padded sequence of 152 API call IDs, fed through an `Embedding` layer into an `LSTM`, which can pick up on call-order patterns the bag-of-calls representation discards entirely. No explicit masking is used, so the model has to learn on its own that the padding sentinel carries no signal - see the notebook's Discussion section.

Both are evaluated with accuracy, precision, recall, F1-score (the specific metrics the assignment names for this section), a confusion matrix, and an ROC curve, then compared side by side.

## Running in Google Colab (recommended)

1. Have `new_dataset.csv` ready on your computer (already present in this folder's `data/`).
2. Upload `ransomware_detection.ipynb` to Colab (**File -> Upload notebook**), or open it from Google Drive/GitHub.
3. Run cells from the top (**Runtime -> Run all**, or step through one at a time).
4. When the "Load Dataset" cell runs, it checks `data/new_dataset.csv`, `/content/new_dataset.csv`, and `/content/data/new_dataset.csv`. If none of those exist, **it automatically opens Colab's file-upload dialog** - pick the CSV, and the notebook continues from there. At only about 1.5MB, this uploads almost instantly.
   - If you'd rather upload ahead of time via the Files sidebar, either drop it directly into `/content` or create a `data` folder there first - both are auto-detected.
5. Everything else (cleaning, deduplication, EDA, feature extraction, both models, evaluation, plots) runs automatically. All packages used (`pandas`, `scikit-learn`, `tensorflow`, `matplotlib`, `seaborn`) are preinstalled in Colab, so `requirements.txt` isn't needed there.

**Runtime expectations:** this is by far the smallest dataset of the four sections (under 2,300 rows after deduplication). The whole notebook, including both models, should run in well under 10 minutes on Colab's standard CPU runtime.

## Running locally instead

1. Install Python 3.10+ and dependencies: `pip install -r requirements.txt`.
2. Make sure `data/new_dataset.csv` exists relative to the notebook (it already does in this folder).
3. `cd` into this folder and run `jupyter notebook ransomware_detection.ipynb`, then run all cells top to bottom.

## What the notebook does

- Loads `new_dataset.csv` and prints its shape.
- Selects only the relevant columns (152 sequence positions plus the label), removes duplicate sequences, and prints the resulting class balance.
- Shows exploratory plots (class balance, API call diversity per sample by class).
- Builds the two feature representations: a bag-of-API-calls count vector for the SVM, and the raw ordered sequence for the LSTM.
- Splits into train/test (80/20, stratified) once, applied consistently to both representations.
- Trains and evaluates the SVM, then the LSTM.
- Saves all figures and models to `outputs/`: `figures/*.png`, `models/scaler.joblib`, `models/svm.joblib`, `models/lstm_model.keras`, and `model_comparison.csv`.
- Prints a final side-by-side comparison table and chart.

## Troubleshooting

- **Colab: the upload dialog didn't appear / "Could not find the dataset" error:** re-run the "Load Dataset" cell - it only opens the dialog when the file isn't found. If the runtime restarted, a previously uploaded file is gone and needs re-uploading.
- **Colab: uploaded the file but it's still not found:** check exactly where it landed via the Files sidebar - it should be directly under `content` (i.e. `/content/new_dataset.csv`) or `content/data`. A different filename is fine as long as it's the *only* `.csv` in that folder.
- **Local: `FileNotFoundError` even though the CSV is in `data/`:** the notebook's kernel working directory isn't this folder - a common VS Code Jupyter extension issue, since it runs kernels from the workspace root rather than the notebook's own folder. Launch Jupyter directly from this folder instead (`cd` here, then `jupyter notebook ransomware_detection.ipynb`).
- **`KeyError` on a sequence column:** if you swap in a different CSV, make sure it still has columns literally named `"0"` through `"151"` and a `malware` column; a differently-shaped API call sequence dataset will need `SEQ_LEN` and `NUM_API_CATEGORIES` updated in the Data Overview cell to match.
- **`IntCastingNaNError: Cannot convert non-finite values (NA or inf) to integer`:** this means some sequence columns have missing (padded) values that aren't being filled before the cast to `int32`. The Data Overview cell (Section 2) already handles this via `df[SEQ_COLS].fillna(PAD_VALUE)`; if you see this error, check you haven't reordered or removed that `fillna` call, and that `PAD_VALUE` is set higher than the highest real API call ID in whatever data you're using.
- **TensorFlow install issues (local only):** TensorFlow's official wheels support a specific range of Python versions/platforms - see https://www.tensorflow.org/install if `pip install tensorflow` fails on your system.
