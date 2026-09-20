# Findings - Section 1: Email Security & Phishing Detection

This document records what was found while building and validating the phishing/spam email classification pipeline, including a serious data quality bug that was caught before it could produce misleading results.

## 1. Initial run on Enron Spam Data looked suspicious

The pipeline was first built against the Enron Spam Data dataset (Kaggle: `marcelwiechmann/enron-spam-data`). The first full run produced near-perfect results for both models:

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
| --- | --- | --- | --- | --- | --- |
| Logistic Regression (TF-IDF) | 99.96% | 99.91% | 100.0% | 99.96% | 99.99% |
| LSTM | 99.97% | 99.94% | 100.0% | 99.97% | 99.99% |

These numbers are not realistic for a text classification task like this. Real spam/phishing classifiers on this kind of data typically land around 97-99%, not 99.95%+ with 100% recall. The LSTM's training curve was also a red flag: accuracy sat at a coin flip (~58%) for epochs 1-2, then jumped abruptly to 99.8%+ by epoch 3-4, which is a pattern typical of memorization rather than genuine learning.

## 2. Root cause 1: massive duplicate rows causing train/test leakage

Direct inspection of the raw CSV showed that out of 33,716 rows, only 15,793 were unique messages. 601 distinct email texts were exact duplicates, accounting for 18,472 rows (over half the dataset). Some single emails appeared 1,500-4,500 times identically.

Because the train/test split was a random row-level split with no deduplication, copies of the same exact email ended up in both the training and test sets. The models were not learning to generalize; they were memorizing duplicate emails in training and then "predicting" the identical copy in test. This inflated every metric to near-100%.

**Fix applied:** added a deduplication step (`df.drop_duplicates(subset="text")`) immediately after label normalization and before the train/test split, in both notebook variants that existed at the time.

## 3. Root cause 2: the Enron Spam Data Kaggle mirror is corrupted

Applying the deduplication fix did not just lower the accuracy, it broke the pipeline entirely: every row labeled "spam" disappeared, leaving zero spam rows and an `IndexError` when trying to print a sample spam email.

Investigating why led to a bug in the dataset's own build script (`build_data_file.py`, bundled with the Kaggle download). The script processes each Enron mail directory in two loops:

```python
# Process ham messages in directory
for entry in os.scandir(ham_folder):
    file = open(entry, encoding="latin_1")
    content = file.read().split("\n", 1)   # content is set here
    subject = content[0].replace("Subject: ", "")
    message = content[1]
    ...

# Process spam messages in directory
for entry in os.scandir(spam_folder):
    file = open(entry)
    file = open(entry, encoding="latin_1")   # opens the file, but never calls .read()
    subject = content[0].replace("Subject: ", "")   # reuses stale `content` from the ham loop
    message = content[1]
```

The spam-processing loop opens each spam file but never reads it. It reuses whatever `content` was left over from the last ham email processed in that directory. This was confirmed directly on the CSV: the first "spam" row's `Message` field was byte-for-byte identical to the ham row immediately before it, and every subsequent spam row in that batch repeated the same frozen text (in one case, an identical block repeated 4,501 times).

**Conclusion:** the `Message` and `Subject` columns for every row labeled "spam" in this specific Kaggle CSV contain no real spam content at all, just leftover ham text. This is not fixable from within the notebook; the real spam email text was never captured in this file. This also fully explains finding 1: the classifiers were not learning spam-vs-ham content, they were trivially recognizing a handful of frozen duplicate blocks.

## 4. Decision: switch dataset to SpamAssassin

Given the Enron mirror was unusable, the dataset was switched to the SpamAssassin Public Corpus (Kaggle: `ganiyuolalekan/spam-assassin-email-classification-dataset`), which was already listed as an alternative in the assignment's own suggested dataset list. Verification before switching:

- 5,796 rows, columns `text` (raw email including headers) and `target` (`1` = spam, `0` = ham).
- Target distribution: 3,900 ham, 1,896 spam.
- Duplicate check: 467 duplicate rows (about 8%), a normal level for a real email corpus, not evidence of corruption.
- Simulated dedup (keep first occurrence): 3,638 ham, 1,691 spam remained. Both classes survive deduplication, unlike Enron.
- Spot-checked several spam rows against neighbouring ham rows to confirm message content is genuinely different per row (no frozen/repeated-block pattern like Enron).

The Kaggle-download notebook variant (which used `kagglehub` for Enron) was removed. Only one notebook remains, designed for Google Colab: the dataset CSV is uploaded manually (or picked via an automatic Colab upload dialog if not found), with no Kaggle API credentials required.

## 5. Final results on SpamAssassin (with deduplication applied)

Pipeline run end to end in Google Colab, no errors:

- Loaded: 5,796 rows, columns `text` / `target` auto-detected correctly.
- Deduplication: removed 467 duplicate emails (5,796 to 5,329 rows). Label distribution after dedup: 3,638 ham, 1,691 spam - exactly matching the pre-run verification in step 4.
- Train/test split: 4,263 train / 1,066 test (80/20, stratified).

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
| --- | --- | --- | --- | --- | --- |
| Logistic Regression (TF-IDF) | 98.2% | 99.7% | 94.7% | 97.1% | 0.9984 |
| LSTM | 99.1% | 99.1% | 97.9% | 98.5% | 0.9998 |

These results look genuine, not leaked:

- Logistic Regression's recall on the spam class is 94.7%, not 100%. An imperfect, realistic precision/recall trade-off is what a linear bag-of-words model should produce, unlike the suspicious 1.000 recall seen on the corrupted Enron run.
- The LSTM modestly outperforms the TF-IDF + Logistic Regression baseline (better accuracy and recall), which is the expected pattern rather than both models mysteriously converging on identical perfect scores.
- Accuracy is still high (98-99%). This is expected and reported in the literature: SpamAssassin is considered a relatively easy benchmark because the `text` column includes full raw email headers (`Received:`, sender IPs, mail server chains), which carry strong spam signal on their own, separate from the message body content. This is documented as a limitation in the notebook, not hidden.

## 6. Takeaways for the report

- Always sanity-check a dataset before trusting model results from it, especially when results look "too good." A 99.9%+ score across every metric for two independently trained model types is a signal to investigate the data, not a result to report as-is.
- Deduplicate before splitting into train/test whenever a dataset might contain exact or near-exact duplicate records; otherwise apparent generalization performance can be inflated by simple memorization.
- A bug can live in a dataset's own preparation script, not just in your own code. The corrupted Enron mirror was a bug in a third-party CSV-building script (`build_data_file.py` on Kaggle), not in anything built for this assignment, but it would have silently produced a fraudulent-looking "phishing detector" if not caught.
- The reported SpamAssassin results (Logistic Regression ~98.2% accuracy, LSTM ~99.1% accuracy) are the numbers that should go into the written report and comparison discussion for this section.
