# Section 1 - Email Security & Phishing Detection

Detects phishing/malicious emails from their text content, comparing a classic machine learning model against a deep learning model, as required by Section 1 (25 marks) of the assignment.

## What's here

| File | Purpose |
| --- | --- |
| `phishing_detection.ipynb` | The notebook: data loading (Google Colab upload), deduplication, preprocessing, both models, evaluation, comparison, visualizations. |
| `requirements.txt` | Python dependencies if you want to run this outside Colab. Not needed in Colab itself - it ships with pandas/scikit-learn/TensorFlow/NLTK preinstalled. |
| `data/spam_assassin.csv` | The dataset used by the notebook. |
| `outputs/` | Created automatically the first time you run the notebook; holds generated figures (`figures/`), saved models (`models/`), and the model comparison table (`model_comparison.csv`). Not present until you run it. |

## Dataset

**SpamAssassin Public Corpus** (Kaggle: [`ganiyuolalekan/spam-assassin-email-classification-dataset`](https://www.kaggle.com/datasets/ganiyuolalekan/spam-assassin-email-classification-dataset)) - 5,796 raw emails with a `text` column (full email including headers) and a `target` column (`1` = spam, `0` = ham). The `spam`/`ham` labels are used as a proxy for `phishing`/`legitimate`, consistent with the assignment brief's own suggested dataset list. The notebook discusses this and other limitations in the final "Discussion" section.

**Why SpamAssassin and not Enron Spam Data.** This section originally used the `marcelwiechmann/enron-spam-data` Kaggle mirror instead. That dataset turned out to have a serious bug in its own build script: every row labeled `spam` actually contained duplicated, leftover `ham` text rather than real spam content (the script's spam-processing loop opens each file but never reads it, silently reusing whatever text was last read while processing ham files). This was confirmed by direct inspection - the first "spam" row's message was byte-for-byte identical to the ham row right before it. That made the Enron mirror unusable for this task, so the notebook was switched to SpamAssassin, which doesn't have this problem. The full story is documented in the notebook's Discussion section, since it's a good illustration of why you should sanity-check a dataset before trusting model results from it.

**Data quality note - duplicate emails.** Roughly 8% of SpamAssassin's rows are exact-duplicate emails. The notebook includes a **"Remove Duplicate Emails"** step (Section 3) that deduplicates on email text *before* the train/test split, so duplicates can't leak across the split and inflate metrics. Don't remove this step when adapting the notebook.

## Models implemented

1. **Classic ML:** TF-IDF vectorization + Logistic Regression.
2. **Deep Learning:** Keras Tokenizer + padded sequences + an LSTM network.

Both are evaluated with accuracy, precision, recall, F1-score, a confusion matrix, and an ROC curve, then compared side by side.

## Running in Google Colab (recommended)

1. Download `data/spam_assassin.csv` from this folder (or directly from Kaggle) to your computer, if you don't already have it.
2. Upload `phishing_detection.ipynb` to Colab (**File → Upload notebook**), or open it from Google Drive/GitHub.
3. Run cells from the top (**Runtime → Run all**, or step through one at a time).
4. When the "Load Dataset" cell runs, it checks `data/spam_assassin.csv`, `/content/spam_assassin.csv`, and `/content/data/spam_assassin.csv`. If none of those exist, **it automatically opens Colab's file-upload dialog** - pick the CSV, and the notebook continues from there.
   - If you'd rather upload ahead of time via the Files sidebar (folder icon on the left), either drop it directly into `/content` (the root shown by default) or create a `data` folder there first and upload into it - both are auto-detected.
5. Everything else (dedup, preprocessing, both models, evaluation, plots) runs automatically. All packages used (`pandas`, `scikit-learn`, `tensorflow`, `nltk`, `matplotlib`, `seaborn`) are preinstalled in Colab, so `requirements.txt` isn't needed there.

**Important:** anything you upload in Colab only lives for the current runtime/session. If the runtime disconnects or restarts (including from being idle too long), you'll need to re-upload the CSV - the notebook will prompt you again automatically when you re-run the "Load Dataset" cell.

## Running locally instead

1. Install Python 3.10+ and dependencies: `pip install -r requirements.txt`.
2. Make sure `data/spam_assassin.csv` exists relative to the notebook (it already does in this folder).
3. `cd` into this folder and run `jupyter notebook phishing_detection.ipynb`, then run all cells top to bottom.

On a typical laptop CPU, the full run (including LSTM training) should take a few minutes; it will be faster with a GPU-enabled TensorFlow install.

## What the notebook does

- Loads the dataset and prints its shape.
- Auto-detects the text/label columns and prints which ones it picked, then normalizes labels to `0`/`1`.
- Removes duplicate emails (Section 3) to prevent train/test leakage.
- Shows exploratory plots (class balance, email-length distribution).
- Cleans the text (lowercase, strip URLs/HTML, remove stopwords, lemmatize).
- Trains and evaluates the Logistic Regression model, then the LSTM model.
- Saves all figures and models to `outputs/`: `figures/*.png`, `models/tfidf_vectorizer.joblib`, `models/logistic_regression.joblib`, `models/lstm_model.keras`, `models/tokenizer_config.json`, and `model_comparison.csv`.
- Prints a final side-by-side comparison table and chart.

## Troubleshooting

- **Colab: the upload dialog didn't appear / "Could not find the dataset" error:** re-run the "Load Dataset" cell - the dialog only opens once per attempt when no file is found. If the runtime restarted, any previously uploaded file is gone and needs re-uploading.
- **Colab: uploaded the file but it's still not found:** check exactly where it landed via the Files sidebar - it should be directly under `content` (i.e. `/content/spam_assassin.csv`) or `content/data`. A different filename is fine as long as it's the *only* `.csv` in that folder.
- **Local: `FileNotFoundError` even though the CSV is in `data/`:** the notebook's kernel working directory isn't this folder - a common VS Code Jupyter extension issue, since it runs kernels from the workspace root rather than the notebook's own folder. Launch Jupyter directly from this folder instead (`cd` here, then `jupyter notebook phishing_detection.ipynb`).
- **`IndexError: single positional indexer is out-of-bounds` when printing a sample email:** this means one class (usually the "spam" one) has zero rows left after loading/deduplication - a sign the dataset itself has a labeling/content bug, not a notebook bug. This is exactly what happened with the old Enron mirror (see "Why SpamAssassin..." above); if you swap in a different dataset and see this, inspect it for the same kind of issue before trusting any results from it.
- **`nltk` download errors (e.g. behind a proxy/firewall):** the notebook calls `nltk.download(...)` for `stopwords`, `wordnet`, `omw-1.4`, and `punkt` on first run; these need internet access once (Colab has this by default).
- **Auto-detected column names look wrong:** the cell under "Column Detection & Label Normalization" prints the column names it picked. If they're wrong for your copy of the dataset, check `df.columns` from the previous cell and set `TEXT_COL` / `LABEL_COL` manually (commented-out lines are provided in that cell).
- **TensorFlow install issues (local only):** TensorFlow's official wheels support a specific range of Python versions/platforms - see https://www.tensorflow.org/install if `pip install tensorflow` fails on your system.
