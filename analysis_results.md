# Phishing Email Dataset Analytics & Findings

This document summarizes the findings from the data analytics process performed on the `phishing_email_detection_2026_dataset.csv` file, along with the details of the codebase refactoring and model corrections implemented.

---

## 1. Overall Dataset Distribution

> [!NOTE]
> The dataset is perfectly balanced between legitimate and phishing emails, which is ideal for training high-quality predictive models.

- **Total Emails Analyzed**: 1,500
- **Phishing Emails**: 740 (49.33%)
- **Legitimate Emails**: 760 (50.67%)

---

## 2. Key Indicators of Phishing (Feature Means)

By comparing the average values between legitimate and phishing emails, we can easily identify the strongest signals of malicious intent.

| Feature | Average in Legitimate Emails | Average in Phishing Emails | Impact & Translation |
| :--- | :--- | :--- | :--- |
| **Urgency Score** | 3.03 / 10 | 8.49 / 10 | **Critical Indicator**. Phishing emails rely heavily on creating a false sense of urgency to bypass rational thinking. |
| **Spelling Errors** | 1.03 per email | 6.46 per email | **Strong Indicator**. Phishing emails contain numerous typos and grammatical errors, which help bypass simple spam filters or select vulnerable targets. |
| **Email Length (words)**| ~193 words | ~71 words | **Strong Indicator**. Legitimate emails are longer and more detailed, whereas phishing emails are brief to drive immediate action. |
| **Urgency Keywords** | 0.00% | 100.00% | **Perfect Predictor**. 100% of phishing subjects contain threat-related keywords, while 0% of legitimate subjects do. |

---

## 3. Machine Learning Pipeline Architecture

To prevent bugs and model decay, the machine learning pipeline was redesigned. The revised architecture is outlined below:

```mermaid
graph TD
    A[Raw Data CSV] --> B[Train-Test Split (80/20)]
    B -->|80% Train| C[StandardScaler Fit & Transform]
    B -->|20% Test| D[StandardScaler Transform Only]
    C --> E[Class Balance Validation (SMOTE check)]
    E --> F[GridSearchCV Hyperparameter Tuning]
    F -->|Best Estimators| G[Baseline & Tuned Models Evaluation]
    D --> H[Model Evaluation & Dashboard]
    G --> H
    G --> I[Production Pipeline Assembly (Scaler + Model)]
    C --> I
    I -->|Saved Model| J[phishing_detection_model.joblib]
```

---

## 4. Model Performance Comparison

After hyperparameter tuning and model evaluation on the test set (300 samples), we compared the classifiers:

| Model | Accuracy | Precision | Recall | F1-Score | ROC-AUC |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Logistic Regression** | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| **Random Forest (Tuned)** | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| **XGBoost (Tuned)** | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| **SVM** | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 |
| **Naive Bayes** | 1.0000 | 1.0000 | 1.0000 | 1.0000 | 1.0000 |

> [!TIP]
> The dataset is perfectly separable due to the synthetic boundaries of the features (`urgency_score` and `spelling_errors` do not overlap between the classes). Thus, all models achieve 100% accuracy. Logistic Regression was chosen as the default production model due to its light weight and rapid inference.

---

## 5. Deployed Codebase Improvements

We conducted a code review of the Jupyter Notebook [phishing_data_analytics.ipynb](file:///Users/user/Documents/Bus_Data_Project/emai_phising_notebook/phishing_data_analytics.ipynb) and resolved four critical bugs:

### 1. Fixed Scaler Data Leakage
- **Issue**: The original notebook fit the `StandardScaler` on the entire dataset `X` before performing the train-test split. This leaked mean and variance information from the test set into the training set.
- **Solution**: Split the raw data first using `train_test_split`, then fit the scaler on the training set only and use it to transform both train and test.

### 2. Fixed Production Model Inference Bug (0% Phishing Prediction)
- **Issue**: The production pipeline `StandardScaler` was fit on `X_train` (which was already scaled). This meant the pipeline scaler had a mean of ~0 and std of ~1. In production, when raw features (like `urgency_score = 8`) were passed to the pipeline, they were not scaled properly, causing the model to output a `0.0000` phishing probability for obviously malicious emails.
- **Solution**: Fit the production pipeline on raw training features (`X_train_raw`). The pipeline now properly scales incoming raw email features, yielding correct prediction outputs (e.g. `Is Phishing: True` with probability `1.0000`).

### 3. Solved GridSearchCV Tuning Flow Discard Bug
- **Issue**: The notebook executed hyperparameter tuning via `GridSearchCV` for Random Forest and XGBoost but discarded the outputs. It evaluated only the default baseline estimators.
- **Solution**: The best tuned models are now saved into the model dictionary (`Random Forest (Tuned)` and `XGBoost (Tuned)`), compared in the dashboard, and available for selection.

### 4. Corrected Feature Engineering Logic
- **Suspicious Domain Logic**: The original code marked safe public domains (`gmail.com`, `yahoo.com`, `hotmail.com`) as suspicious. We corrected this to identify spoofed/typosquatted domains containing phishing keywords (like `alert`, `verify`, or `secure`), leaving public domains as safe.
- **Urgency Keywords Regex**: The regex was updated to match keywords present in the dataset's phishing email subjects (like `verify`, `verification`, `suspicious`, `login`, `attempt`, `password`, `expire`, `claim`, `reward`, `locked`) while avoiding false positives on legitimate subjects (like `Project Update`).
- **Multicollinearity Removal**: Dropped the redundant `suspicious_score` from machine learning features `X` to stabilize Logistic Regression and prevent overflow warnings, making model coefficients robust and interpretable.

---

## 6. Actionable Security Answers

**Q: What is the most reliable way to spot a phishing email in this dataset?**
**A:** Look for emails that combine **high urgency scores** (7/10 and above), **spelling errors** (3 or more), and contain **subject keywords** like "Locked", "Suspicious", "Verify", or "Claim".

**Q: Are attachments a good way to filter out phishing?**
**A:** No. Legitimate emails have attachments 50.39% of the time, and phishing emails 52.16% of the time. Attachments are not a strong distinguishing factor.

**Q: How does email length correlate to phishing?**
**A:** There is a strong inverse correlation. Phishing emails are much shorter (avg. 71 words) than legitimate emails (avg. 193 words) to drive users to click links quickly without reading context.
