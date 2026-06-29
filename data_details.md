# 📊 Phishing Dataset Details & Specifications

This document provides a deep dive into the properties, columns, data types, and statistical structure of the `phishing_email_detection_2026_dataset.csv` file.

---

## 1. Dataset Specifications & Structure

- **Total Records**: 1,500 emails
- **Total Columns**: 9
- **Missing Values**: 0 (100% complete)
- **Duplicate Rows**: 0 (100% unique)

### Feature Schema & Data Types

| Column Name | Data Type | Role | Description |
| :--- | :---: | :---: | :--- |
| `email_id` | Integer | ID | Unique identifier for each email |
| `sender_email` | Object (String) | Metadata | Full email address of the sender |
| `subject` | Object (String) | Metadata | Subject line of the email |
| `has_link` | Binary (0/1) | Feature | Indicates if the email body contains links (URLs) |
| `has_attachment` | Binary (0/1) | Feature | Indicates if the email contains file attachments |
| `urgency_score` | Integer (1-10) | Feature | Human-graded or heuristic score of semantic urgency |
| `spelling_errors` | Integer (0-10) | Feature | Count of spelling and grammatical errors detected |
| `email_length_words` | Integer | Feature | Total word count of the email body |
| `is_phishing` | Binary (0/1) | **Target** | Class label (0 = Legitimate, 1 = Phishing) |

---

## 2. Descriptive Statistics by Class

Due to the synthetic design of this dataset, there is a clear, non-overlapping boundary between legitimate and phishing email characteristics.

### Legitimate Emails (Class 0 - 760 samples)

| Numeric Feature | Mean | Std | Min | 25% | 50% | 75% | Max |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `urgency_score` | **3.03** | 1.39 | 1.00 | 2.00 | 3.00 | 4.00 | 5.00 |
| `spelling_errors` | **1.03** | 0.82 | 0.00 | 0.00 | 1.00 | 2.00 | 2.00 |
| `email_length_words` | **192.86** | 64.91 | 80.00 | 140.00 | 192.00 | 251.00 | 300.00 |
| `has_link` | **0.52** | 0.50 | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |
| `has_attachment` | **0.50** | 0.50 | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |

### Phishing Emails (Class 1 - 740 samples)

| Numeric Feature | Mean | Std | Min | 25% | 50% | 75% | Max |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `urgency_score` | **8.49** | 1.10 | 7.00 | 8.00 | 8.00 | 9.00 | 10.00 |
| `spelling_errors` | **6.46** | 2.29 | 3.00 | 4.00 | 6.00 | 9.00 | 10.00 |
| `email_length_words` | **70.63** | 29.22 | 20.00 | 45.00 | 71.00 | 96.00 | 120.00 |
| `has_link` | **0.74** | 0.44 | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |
| `has_attachment` | **0.52** | 0.50 | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |

---

## 3. Key Statistical Observations

> [!IMPORTANT]
> - **Urgency Score separation**: Legitimate scores are strictly $\le 5$ while phishing scores are strictly $\ge 7$.
> - **Spelling Errors separation**: Legitimate errors are strictly $\le 2$ while phishing errors are strictly $\ge 3$.
> - **Email Length boundary**: Legitimate emails are longer on average (193 words) than phishing emails (71 words), with minor overlap between 80 and 120 words.
> - **Links and Attachments**: While links are 22% more common in phishing emails, attachments show no variance (50% vs 52%) between classes, making attachments a poor standalone predictor.
