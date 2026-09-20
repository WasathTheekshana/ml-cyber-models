# Findings - Section 3: Anomaly Detection for Cyber Threats

This document records the results of running `anomaly_detection.ipynb` on the full CIC-IDS2018 `Wednesday-14-02-2018` capture in Google Colab, and what they mean.

## 1. Data loading and cleaning

`cic.csv` loaded successfully: 1,048,575 rows, 80 columns, matching the expected raw file.

Cleaning proceeded in this order (as designed): drop `Timestamp`, then remove duplicate rows, then remove rows with infinite/missing values.

- Dropping `Timestamp` before deduplicating means two flows that are identical in every feature except capture time now correctly count as duplicates. This produced a higher duplicate count than an earlier estimate made before `Timestamp` was excluded: **373,760 duplicate rows removed** (1,048,575 -> 674,815), notably more than the 225,628 estimated when duplicates were checked including the timestamp column. This is expected and correct, not a bug - excluding a non-predictive identifier column before deduplicating is exactly what should happen.
- A further 3,677 rows were removed for infinite/missing values (`Flow Byts/s`, `Flow Pkts/s`), leaving **671,138 rows**.

Class distribution after cleaning:

| Class | Count |
| --- | --- |
| Benign | 577,037 |
| SSH-Bruteforce | 94,048 |
| FTP-BruteForce | 53 |

**Notable finding:** `FTP-BruteForce` collapsed from 193,360 raw rows down to just **53 unique rows** once timestamp-exclusive deduplication was applied - a far more extreme drop than anticipated when the notebook was designed (which estimated roughly 39,000 unique rows for this class based on a preliminary check that still included timestamp in the duplicate comparison). This means the entire `FTP-BruteForce` attack in this capture, across all 193,360 recorded flows, only produced 53 distinct feature combinations - consistent with a single automated brute-force script repeatedly generating near-identical packets against the same target, differing only in timing.

Binary label distribution (0 = Normal, 1 = Anomaly): 577,037 normal, 94,101 anomaly.

## 2. Preprocessing and split

- Feature matrix: 77 numeric features + 3 one-hot `Protocol` columns (`Protocol_0.0`, `Protocol_6.0`, `Protocol_17.0`) = 80 columns total, matching the design.
- Train/test split: 536,910 train / 134,228 test (80/20, stratified). Of the training rows, 461,629 are normal and were used to fit both models. Test set: 115,408 normal, 18,820 anomaly.

## 3. Results

| Model | TPR | FPR | Precision | Recall | F1-score | ROC-AUC | Average Precision |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Isolation Forest | 0.0000 | 0.0506 | 0.00 | 0.00 | 0.00 | 0.886 | 0.398 |
| Autoencoder | 0.9999 | 0.0497 | 0.766 | 1.00 | 0.867 | 0.977 | 0.706 |

(TPR, FPR, Precision, Recall, F1-score are all measured at each model's own 95th-percentile threshold, derived from that model's anomaly score on normal-only training data. ROC-AUC and Average Precision are threshold-independent.)

### Autoencoder: strong, realistic result

The Autoencoder caught essentially all anomalies (TPR 99.99%) while keeping the false positive rate to about 5%, for a precision of 77% and recall of 100% on the anomaly class. This is a sensible, non-suspicious precision/recall trade-off for a semi-supervised anomaly detector - not perfect, not degenerate, exactly the profile expected from a model that has genuinely learned what normal traffic looks like.

### Isolation Forest: a real, explainable threshold failure

At its own 95th-percentile threshold, Isolation Forest caught **zero** anomalies (TPR = 0.0000), and its precision/recall/F1 on the anomaly class are all 0.00. This looks like a broken model at first glance, but its ROC-AUC is a respectable 0.886, meaning it does generally rank anomalous flows as more anomalous than normal ones across the full range of possible thresholds.

The problem is specifically that the fixed 95th-percentile cutoff, chosen from the distribution of normal training scores, happened to sit above nearly every anomaly's score too on this data, so nothing crossed it at that particular operating point. This is a textbook illustration of a point the notebook's Discussion section anticipated in general terms: **threshold choice can make or break a model that is not actually bad at ranking.** A lower threshold, chosen by inspecting the Isolation Forest's own ROC or precision-recall curve (already plotted in the notebook), would very likely recover meaningful TPR at some cost to FPR.

## 4. Takeaways for the report

- The Autoencoder clearly outperforms Isolation Forest here, both in raw discriminative power (ROC-AUC 0.977 vs 0.886) and dramatically so at the chosen operating point (TPR 99.99% vs 0%).
- Isolation Forest's failure at its threshold is a threshold-calibration problem, not a ranking problem. Report both the ROC-AUC (which shows it has real signal) and the threshold-based TPR/FPR (which shows the specific 95th-percentile choice does not work for this model on this data), since reporting only one of the two would be misleading in opposite directions.
- The extreme collapse of `FTP-BruteForce` to 53 unique rows after proper deduplication is worth citing as a concrete, dataset-specific example of why deduplication before evaluation matters: without it, this single repetitive attack pattern would have been counted 193,360 times over, disproportionately inflating its apparent presence in the data relative to how many genuinely distinct attack instances actually occurred.
- With only 53 total samples for `FTP-BruteForce`, any class-specific conclusions about that attack type specifically (as opposed to the combined binary anomaly class used for evaluation here) would not be statistically meaningful.
