# Findings - Section 2: Cyber Attack Detection

This document records the results of running `cyber_attack_detection.ipynb` on the full CIC-IDS2017 dataset in Google Colab, and what they mean.

## 1. Data loading and cleaning

All 8 expected CSV files were found via the Colab upload dialog and loaded successfully:

| File | Rows |
| --- | --- |
| Monday-WorkingHours.pcap_ISCX.csv | 529,918 |
| Tuesday-WorkingHours.pcap_ISCX.csv | 445,909 |
| Wednesday-workingHours.pcap_ISCX.csv | 692,703 |
| Thursday-WorkingHours-Morning-WebAttacks.pcap_ISCX.csv | 170,366 |
| Thursday-WorkingHours-Afternoon-Infilteration.pcap_ISCX.csv | 288,602 |
| Friday-WorkingHours-Morning.pcap_ISCX.csv | 191,033 |
| Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv | 286,467 |
| Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv | 225,745 |
| **Combined** | **2,830,743** |

This matches the officially reported CIC-IDS2017 totals, confirming the raw files are the authentic, uncorrupted dataset (unlike the Enron Spam Data issue found in Section 1).

Cleaning removed 2,867 rows containing infinite or missing values (from `Flow Bytes/s` and `Flow Packets/s` on zero-duration flows), leaving 2,827,876 rows. The corrupted `Web Attack` labels (containing a Unicode replacement character baked into the source file) were correctly cleaned to `Web Attack - Brute Force`, `Web Attack - XSS`, and `Web Attack - Sql Injection`.

Class distribution after cleaning (15 classes total):

| Class | Count |
| --- | --- |
| BENIGN | 2,271,320 |
| DoS Hulk | 230,124 |
| PortScan | 158,804 |
| DDoS | 128,025 |
| DoS GoldenEye | 10,293 |
| FTP-Patator | 7,935 |
| SSH-Patator | 5,897 |
| DoS slowloris | 5,796 |
| DoS Slowhttptest | 5,499 |
| Bot | 1,956 |
| Web Attack - Brute Force | 1,507 |
| Web Attack - XSS | 652 |
| Infiltration | 36 |
| Web Attack - Sql Injection | 21 |
| Heartbleed | 11 |

BENIGN alone is about 80% of the dataset, while three attack types (Heartbleed, Web Attack - Sql Injection, Infiltration) have fewer than 40 samples in the *entire* 2.8 million row dataset. This extreme imbalance is a genuine, documented property of CIC-IDS2017, not a data quality problem.

## 2. Preprocessing and PCA

- Train/test split: 2,262,300 train / 565,576 test (80/20, stratified across all 15 classes).
- PCA reduced the 78 standardized numeric features down to 25 components, retaining 95.27% of the variance.

## 3. Results

| Model | Accuracy | Macro Precision | Macro Recall | Macro F1 | Mean Average Precision |
| --- | --- | --- | --- | --- | --- |
| Random Forest (PCA) | 99.25% | 0.861 | 0.859 | 0.831 | 0.861 |
| CNN (PCA) | 98.20% | 0.770 | 0.679 | 0.693 | 0.736 |

Both models score high on overall accuracy because BENIGN dominates the data, but they diverge sharply on macro-averaged metrics, which weight every class equally regardless of size. Random Forest clearly outperforms the CNN once rare classes are taken into account.

Per-class detail on the three rarest classes:

| Class (support) | Random Forest (P / R / F1) | CNN (P / R / F1) |
| --- | --- | --- |
| Heartbleed (2) | 1.00 / 1.00 / 1.00 | 1.00 / 1.00 / 1.00 |
| Web Attack - Sql Injection (4) | 1.00 / 0.50 / 0.67 | 0.00 / 0.00 / 0.00 |
| Infiltration (7) | 0.67 / 0.29 / 0.40 | 0.00 / 0.00 / 0.00 |
| Web Attack - XSS (130) | 0.43 / 0.48 / 0.45 | 0.00 / 0.00 / 0.00 |
| Web Attack - Brute Force (301) | 0.75 / 0.70 / 0.72 | 0.91 / 0.10 / 0.19 |
| Bot (391) | 0.18 / 0.96 / 0.31 | 0.99 / 0.36 / 0.53 |

The CNN completely failed to detect Web Attack - Sql Injection, Infiltration, and Web Attack - XSS (0.00 on all three). Random Forest still caught some of these, though imperfectly. Both models handled Heartbleed perfectly, but its support is only 2 test samples, so that result carries little statistical weight either way.

Bot shows an interesting split: Random Forest has very high recall (0.96) but low precision (0.18), meaning it flags many false positives as Bot; the CNN has the opposite pattern (0.99 precision, 0.36 recall), meaning it is conservative and misses most real Bot traffic. Neither is simply "better" here; the two models make different kinds of mistakes on this class.

## 4. A methodological asymmetry worth noting

Random Forest was trained with `class_weight="balanced"`, which reweights the loss to compensate for the class imbalance. The CNN's `model.fit()` call was **not** given any equivalent weighting. This is a plausible contributor to the CNN's much larger drop on rare-class metrics: part of the gap between the two models reflects "Random Forest got help with the imbalance and the CNN did not," not purely an intrinsic difference between tree ensembles and convolutional networks on this task. A fairer comparison would add `class_weight` (or per-sample weights) to the CNN's training call and re-run.

## 5. Runtime observations

- CNN training took roughly 215-265 seconds per epoch, across 14-15 epochs before `EarlyStopping` triggered - about 50-65 minutes for CNN training alone, on top of data loading, cleaning, and Random Forest fitting.
- This is notably longer than the original 15-30 minute estimate in the README, which should be revised to reflect this real timing (closer to an hour or more end to end, on Colab's standard CPU runtime).

## 6. Takeaways for the report

- Both models achieve very high overall accuracy (98-99%), but overall accuracy is a poor summary statistic for this task given the imbalance; macro-averaged metrics and per-class results tell a much more informative and quite different story.
- Random Forest is the stronger model overall here, especially on rare attack types, but part of that advantage may be attributable to `class_weight="balanced"` rather than model architecture alone.
- Three of the fifteen classes (Heartbleed, Web Attack - Sql Injection, Infiltration) have so few samples in the whole dataset that their metrics should be reported with an explicit caveat about statistical reliability, regardless of which model is being discussed.
- A natural next experiment, if time allows, is adding class weighting to the CNN and re-running the comparison to see how much of the gap closes.
