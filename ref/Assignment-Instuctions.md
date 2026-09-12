## Assignment Details

**Module Name:** MACHINE LEARNING IN CYBER

**Module Number:** COMP70049

**Title of Assignment:** Assignment

**Contribution to mark:** This assignment is worth 100% of the overall mark for this module.

**Hand in deadlines: Monday, 28 September 2026, 12:00 AM**

## Introduction

This assignment explores the intersection of **machine learning** and **cybersecurity**, focusing on key areas such as **phishing email detection, network intrusion classification, anomaly detection, and ransomware identification**. Students will implement **both classic machine learning models (e.g., Logistic Regression, Decision Trees, Random Forests, Isolation Forests, SVMs)** and **deep learning approaches (e.g., LSTMs, CNNs, Autoencoders, Transformers)** to tackle real-world security challenges.

The primary objectives of this assignment are:

- To **understand the application of machine learning** in cybersecurity.
- To **develop hands-on skills** in data preprocessing, feature engineering, and model training.
- To **compare classic ML vs. deep learning models** in different cybersecurity use cases.
- To **evaluate the performance of ML models** using relevant metrics such as **accuracy, precision, recall, and F1-score**.

The assignment is divided into **four sections**, each focusing on a different aspect of cybersecurity:

1. **Email Security & Phishing Detection (25 Marks)** – Detecting phishing emails using text-based ML and deep learning models.
2. **Cyber Attack Detection (25 Marks)** – Classifying different types of network intrusions using structured attack datasets.
3. **Anomaly Detection for Cyber Threats (25 Marks)** – Identifying unusual or suspicious network activity using unsupervised ML techniques.
4. **Ransomware Detection & Prevention (25 Marks)** – Detecting ransomware activity based on system behavior and API call sequences.

Each section carries 25 marks, making a total of 100 marks for the assignment.

## Instructions

### 1. **Implementation Requirements**

For each of the four sections in this assignment, students must:

1. **Choose an appropriate dataset** (introduced in the assignment or sourced online).
2. **Preprocess the data**, including cleaning, feature engineering, and normalization.
3. **Implement two machine learning models** for each section: **One classic machine learning algorithm** (e.g., Logistic Regression, Random Forest, SVM, Decision Tree) and **One deep learning model** (e.g., LSTM, CNN, Autoencoder, BERT).
4. **Train and evaluate the models** using appropriate performance metrics.
5. **Compare the results** of both models and analyze their effectiveness.
6. **Provide visualizations** (e.g., confusion matrix, ROC curve, loss/accuracy plots).

The implementation should be done in **Python** using appropriate libraries.

Each student must submit:

1. **Python script.** The code should be well-structured and commented. Ensure the notebook/script runs correctly without errors.
2. **Written Report (**up to 3000 words**)**
   - **Include in the report for each experiment**
     - Description of datasets used.
     - Explanation of preprocessing techniques.
     - Overview of implemented ML and deep learning models.
     - Performance comparison of models.
     - Visualizations (graphs, confusion matrices, accuracy/loss plots).
     - Discussion on findings and cybersecurity implications.
     - Summary of key takeaways and potential improvements.
3. A file containing the implemented code(s)
4. **README File**
   - Instructions on how to run the code.

Upload to the Blackboard via provided link by submission deadline.

## Marking Criteria

| **Criteria**                      | **Description**                                             | **Marks** |
| --------------------------------- | ----------------------------------------------------------- | --------- |
| **Correctness of Implementation** | Proper execution of ML and deep learning models             | **50**    |
| **Depth of Analysis**             | Discussion of results, insights, and performance comparison | **30**    |
| **Code Quality & Documentation**  | Readability, comments, structure, and efficiency            | **10**    |
| **Report Presentation**           | Clarity, organization, and completeness                     | **10**    |

## **Section 1: Email Security & Phishing Detection (25 Marks)**

### **Task:**

Phishing emails are a major cybersecurity threat. The goal is to build a model that can detect **phishing emails** based on textual features.

### **Steps to Follow:**

1. **Dataset Selection:** Use datasets such as:
   - **Enron Spam Dataset**
   - **SpamAssassin Dataset**
   - **Phishing Email Dataset from UCI Machine Learning Repository**
2. **Preprocessing:**
   - Convert email text into numerical features using **TF-IDF** or **word embeddings (Word2Vec, GloVe)**.
   - Remove stopwords, punctuation, and apply stemming/lemmatization.
3. **Classic Machine Learning Approach:**
   - Train a **Logistic Regression** or **Random Forest** model for classification.
4. **Deep Learning Approach:**
   - Use an **LSTM (Long Short-Term Memory)** or **BERT (Bidirectional Encoder Representations from Transformers)** model for email classification.
5. **Evaluation:**
   - Compare models using **Precision, Recall, F1-score, and ROC curve**.

## **Section 2: Cyber Attack Detection (25 Marks)**

### **Task:**

Network intrusion detection is critical for identifying cyber-attacks. In this section, students will develop a machine learning model to classify various attack types.

### **Steps to Follow:**

1. **Dataset Selection:** Use datasets such as:
   - **NSL-KDD Dataset**
   - **CIC-IDS2017 (Canadian Institute for Cybersecurity)**
2. **Preprocessing:**
   - Handle missing values, standardize numerical data, and encode categorical variables.
   - Perform **feature selection** using techniques like **PCA (Principal Component Analysis)**.
3. **Classic Machine Learning Approach:**
   - Train a **Decision Tree** or **Random Forest** model for multi-class attack classification.
4. **Deep Learning Approach:**
   - Use a **CNN (Convolutional Neural Network)** or **RNN (Recurrent Neural Network)** to process network traffic data.
5. **Evaluation:**
   - Compare model accuracy, precision-recall curves, and **Confusion Matrix**.

## **Section 3: Anomaly Detection for Cyber Threats (25 Marks)**

### **Task:**

Anomaly detection helps detect **unknown cyber threats** and **zero-day attacks** by identifying unusual network behavior. This section focuses on unsupervised learning techniques for cybersecurity.

### **Steps to Follow:**

1. **Dataset Selection:** Use datasets such as:
   - **UNSW-NB15 Dataset**
   - **CIC-IDS2018 (Anomaly & Intrusion Detection Dataset)**
2. **Preprocessing:**
   - Remove duplicates, normalize numerical features, and handle missing values.
   - Select **relevant network traffic features** (e.g., packet size, protocol type, connection duration).
3. **Classic Machine Learning Approach:**
   - Train an **Isolation Forest** or **One-Class SVM** to detect anomalous behavior.
4. **Deep Learning Approach:**
   - Use an **Autoencoder Neural Network** to learn normal traffic patterns and identify anomalies.
5. **Evaluation:**
   - Measure **True Positive Rate (TPR), False Positive Rate (FPR), and Precision-Recall curves**.

## **Section 4: Ransomware Detection & Prevention (25 Marks)**

### **Task:**

Ransomware is a significant cybersecurity threat. This section focuses on detecting ransomware activity based on system behavior.

### **Steps to Follow:**

1. **Dataset Selection:** Use datasets such as:
   - **Windows API Call Sequence Dataset** (from ransomware attacks)
   - **CIC-AndMal2017 (for Android Ransomware Detection)**
2. **Preprocessing:**
   - Convert system logs (API calls, file access patterns) into numerical feature representations.
   - Perform **feature extraction** to identify key ransomware behavior patterns.
3. **Classic Machine Learning Approach:**
   - Train an **SVM (Support Vector Machine)** or **Gradient Boosting Classifier** to classify ransomware vs. normal activity.
4. **Deep Learning Approach:**
   - Use **LSTM (Recurrent Neural Network)** to model system behavior over time and detect ransomware attacks.
5. **Evaluation:**
   - Compare models using **Precision, Recall, and F1-score**.
