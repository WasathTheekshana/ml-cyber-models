# Section 1 — Email Security & Phishing Detection

Detects phishing/malicious emails from their text content, comparing a classic machine learning model against a deep learning model, as required by Section 1 (25 marks) of the assignment.

There are **two notebooks** with identical modeling logic — pick whichever matches how you want to get the dataset:

| Notebook | Use this if... |
| --- | --- |
| `phishing_detection.ipynb` | You have (or are willing to set up) Kaggle API credentials and internet access. It downloads the dataset automatically via `kagglehub`, with an automatic fallback to a local CSV if that fails. Meant for a local Jupyter/VS Code setup. |
| `phishing_detection_colab_upload.ipynb` | You're running in **Google Colab** and want to upload the dataset CSV yourself, with no Kaggle API/credential setup at all. |

## What's here

| File | Purpose |
| --- | --- |
| `phishing_detection.ipynb` | Kaggle-download notebook: data loading via `kagglehub` (local fallback), preprocessing, both models, evaluation, comparison, visualizations. Saves outputs to `outputs/`. |
| `phishing_detection_colab_upload.ipynb` | Google Colab notebook: same preprocessing/modeling/evaluation, but loads the CSV you upload directly in Colab (auto-detects it, or pops up Colab's upload dialog if it can't find it). Saves outputs to `outputs_local/` (kept separate so running both notebooks doesn't overwrite each other's figures/models). |
| `requirements.txt` | Python dependencies needed for `phishing_detection.ipynb` on a local machine. Not needed for Colab — it ships with pandas/scikit-learn/TensorFlow/NLTK preinstalled. |
| `data/` | Local fallback location for `phishing_detection.ipynb` (place a manually downloaded CSV here). Not used when running in Colab — see below. |
| `outputs/` / `outputs_local/` | Created automatically the first time you run the corresponding notebook; hold generated figures (`figures/`), saved models (`models/`), and the model comparison table (`model_comparison.csv`). Not present until you run a notebook. |

## Dataset

**Enron Spam Data** (Kaggle: [`marcelwiechmann/enron-spam-data`](https://www.kaggle.com/datasets/marcelwiechmann/enron-spam-data)). The dataset's `spam`/`ham` labels are used as a proxy for `phishing`/`legitimate` — this is a standard stand-in used in coursework and much of the phishing-detection literature, and is one of the datasets explicitly suggested by the assignment brief. The notebook discusses this choice and its limitations in the final "Discussion" section.

**Data quality note — duplicate emails.** The raw CSV contains a large number of exact-duplicate emails (over half of its ~33.7k rows are copies of just a few hundred unique messages — some single automated/bulk emails appear 1,500–4,500 times). Both notebooks include a **"Remove Duplicate Emails"** step (Section 3) that deduplicates on email text *before* the train/test split. Without this step, duplicates leak across the split and models can score a suspicious, meaningless ~99.9%+ on every metric by memorizing duplicates rather than generalizing — this was confirmed by an earlier run of this pipeline before the fix was added. Don't remove this step when adapting the notebooks.

## Models implemented

1. **Classic ML:** TF-IDF vectorization + Logistic Regression.
2. **Deep Learning:** Keras Tokenizer + padded sequences + an LSTM network.

Both are evaluated with accuracy, precision, recall, F1-score, a confusion matrix, and an ROC curve, then compared side by side.

## Running in Google Colab (`phishing_detection_colab_upload.ipynb`)

1. Download the dataset CSV from https://www.kaggle.com/datasets/marcelwiechmann/enron-spam-data to your computer (you do **not** need to upload it to Colab yourself first).
2. Upload `phishing_detection_colab_upload.ipynb` to Colab (**File → Upload notebook**), or open it from Google Drive/GitHub.
3. Run cells from the top (**Runtime → Run all**, or step through one at a time).
4. When the "Load Dataset" cell runs, it checks `data/enron_spam_data.csv`, `/content/enron_spam_data.csv`, and `/content/data/enron_spam_data.csv`. If none of those exist, **it automatically opens Colab's file-upload dialog** — pick the CSV you downloaded in step 1, and the notebook continues from there.
   - If you'd rather upload ahead of time via the Files sidebar (folder icon on the left), either drop it directly into `/content` (the root shown by default) or create a `data` folder there first and upload into it — both are auto-detected.
5. Everything else (preprocessing, both models, evaluation, plots) runs the same as the local notebook. All packages used (`pandas`, `scikit-learn`, `tensorflow`, `nltk`, `matplotlib`, `seaborn`) are preinstalled in Colab, so `requirements.txt` isn't needed there.

**Important:** anything you upload in Colab only lives for the current runtime/session. If the runtime disconnects or restarts (including from being idle too long), you'll need to re-upload the CSV — the notebook will prompt you again automatically when you re-run the "Load Dataset" cell.

## Running locally (`phishing_detection.ipynb`)

### Setup

1. Install Python 3.10+ (any recent Python 3 should work).
2. From this folder, install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

### Getting the dataset

**Option A — automatic download.** The first code cell tries to download the dataset automatically via `kagglehub`. This needs a Kaggle account and API credentials:

1. Create a Kaggle account if you don't have one, then go to **Account Settings → API → Create New Token** to download `kaggle.json`.
2. Place it at `~/.kaggle/kaggle.json` (on Windows: `C:\Users\<you>\.kaggle\kaggle.json`), **or** set the environment variables `KAGGLE_USERNAME` and `KAGGLE_KEY` before launching Jupyter.

If the download fails (no credentials/no internet), it automatically falls back to Option B.

**Option B — manual download (fallback).**

1. Go to https://www.kaggle.com/datasets/marcelwiechmann/enron-spam-data and download the CSV.
2. Save it as `data/enron_spam_data.csv` in this folder.

### Running

```bash
cd "01-email-security-phishing-detection"
jupyter notebook phishing_detection.ipynb
```

Run all cells top to bottom (**Kernel → Restart & Run All**). On a typical laptop CPU, the full run (including LSTM training) should take a few minutes; it will be faster with a GPU-enabled TensorFlow install.

## What either notebook does once the data is loaded

- Prints dataset shape, detected column names, and label distribution.
- Shows exploratory plots (class balance, email-length distribution).
- Trains and evaluates the Logistic Regression model, then the LSTM model.
- Saves all figures and models to `outputs/` (`phishing_detection.ipynb`) or `outputs_local/` (`phishing_detection_colab_upload.ipynb`): `figures/*.png`, `models/tfidf_vectorizer.joblib`, `models/logistic_regression.joblib`, `models/lstm_model.keras`, `models/tokenizer_config.json`, and `model_comparison.csv`.
- Prints a final side-by-side comparison table and chart.

## Troubleshooting

- **Colab: the upload dialog didn't appear / "Could not find the dataset" error:** re-run the "Load Dataset" cell — the dialog only opens once per attempt when no file is found. If the runtime restarted, any previously uploaded file is gone and needs re-uploading.
- **Colab: uploaded the file but it's still not found:** check exactly where it landed via the Files sidebar — it should be directly under `content` (i.e. `/content/enron_spam_data.csv`) or `content/data`. A different filename is fine as long as it's the *only* `.csv` in that folder.
- **Local: `FileNotFoundError: No dataset found in 'data/'` even though you placed the CSV there:** the notebook's kernel working directory isn't this folder — a common VS Code Jupyter extension issue, since it runs kernels from the workspace root rather than the notebook's own folder. Launch Jupyter directly from this folder instead (`cd` here, then `jupyter notebook phishing_detection.ipynb`) so the working directory is already correct.
- **Local: `kagglehub` download fails with a 401/403 error:** your Kaggle credentials aren't set up correctly — double check `kaggle.json`'s location/permissions, or use the manual-download fallback (Option B) instead.
- **`nltk` download errors (e.g. behind a proxy/firewall):** both notebooks call `nltk.download(...)` for `stopwords`, `wordnet`, `omw-1.4`, and `punkt` on first run; these need internet access once (Colab has this by default). If it keeps failing locally, download these NLTK corpora on another machine and copy the `nltk_data` folder over, or set the `NLTK_DATA` environment variable to point at a pre-populated copy.
- **Auto-detected column names look wrong:** the cell under "Column Detection & Label Normalization" prints the column names it picked. If they're wrong for your copy of the dataset, check `df.columns` from the previous cell and set `TEXT_COL` / `LABEL_COL` manually (commented-out lines are provided in that cell).
- **TensorFlow install issues (local only):** TensorFlow's official wheels support a specific range of Python versions/platforms — see https://www.tensorflow.org/install if `pip install tensorflow` fails on your system.
