# Phishing Email Dataset Analytics & Findings

This document summarizes the exhaustive findings from the data analytics process performed on the `phishing_email_detection_2026_dataset.csv` file. The data has been translated into plain language to answer all possible questions about what makes an email in this dataset likely to be a phishing attempt.

## 1. Overall Dataset Distribution
> [!NOTE]
> The dataset is perfectly balanced between legitimate and phishing emails, which is ideal for predictive modeling.

- **Total Emails Analyzed**: 1,500
- **Phishing Emails**: 740 (49.33%)
- **Legitimate Emails**: 760 (50.67%)

## 2. Key Indicators of Phishing (Feature Means)

By comparing the average values between legitimate and phishing emails, we can easily identify the strongest signals of malicious intent.

| Feature | Average in Legitimate Emails | Average in Phishing Emails | Impact / Translation |
|---------|-----------------------------|----------------------------|----------------------|
| **Urgency Score** | 3.03 / 10 | 8.49 / 10 | **Critical Indicator**. Phishing emails rely heavily on creating a false sense of urgency (e.g., "Account suspended!", "Act now!"). |
| **Spelling Errors** | 1.03 per email | 6.46 per email | **Strong Indicator**. Phishing emails often contain numerous typos and grammatical errors, either by mistake or deliberately to bypass simple spam filters. |
| **Email Length (words)** | ~193 words | ~71 words | **Strong Indicator**. Legitimate communications tend to be longer and more detailed, whereas phishing emails are kept brief to quickly drive the user to click a link. |

## 3. Links and Attachments

> [!TIP]
> Always be suspicious of short, highly urgent emails that contain links. This combination is the hallmark of a phishing attack.

- **Presence of Links**:
  - **74.19%** of phishing emails contain links.
  - **51.97%** of legitimate emails contain links.
  - *Translation*: A link alone doesn't guarantee an email is phishing, but it is a primary vector for attacks. Phishing emails are significantly more likely to include links to redirect users to malicious landing pages.

- **Presence of Attachments**:
  - **52.16%** of phishing emails contain attachments.
  - **50.39%** of legitimate emails contain attachments.
  - *Translation*: Attachments are used almost equally across both types of emails in this dataset. Thus, an attachment by itself is **not** a strong distinguishing factor for this specific environment. 

## 4. Sender Domain Analysis

We analyzed the domains of the sender emails to see if certain domains are more prone to sending phishing attacks.

| Domain | Total Emails | Phishing Count | Phishing Rate |
|--------|--------------|----------------|---------------|
| `outlook.com` | 306 | 157 | **51.31%** |
| `bankverify.net` | 277 | 141 | **50.90%** |
| `company-secure.com` | 299 | 147 | **49.16%** |
| `gmail.com` | 314 | 153 | **48.73%** |
| `paypal-alert.com` | 304 | 142 | **46.71%** |

> [!WARNING]
> Attackers frequently use spoofed domain names (like `bankverify.net` or `paypal-alert.com`) to mimic trusted entities. They also heavily abuse free email providers (`outlook.com`, `gmail.com`).

## 5. Summary & Actionable Answers

Based on the data, here are the answers to the most critical questions regarding this dataset:

**Q: What is the most reliable way to spot a phishing email in this dataset?**
**A:** Look for a combination of **high urgency**, **short length**, and **numerous spelling errors**. If an email has 6+ spelling errors, is under 100 words, and is extremely urgent, it is almost certainly a phishing attempt.

**Q: Are attachments a good way to filter out phishing?**
**A:** No. In this dataset, attachments are equally prevalent in legitimate and malicious emails.

**Q: Should we automatically block specific domains?**
**A:** Automatically blocking domains like `bankverify.net` or `paypal-alert.com` might be effective since they appear to be dedicated phishing/spoofing domains. However, blocking `gmail.com` or `outlook.com` would result in massive false positives due to the high volume of legitimate traffic they also send.

**Q: How does email length correlate to phishing?**
**A:** There is an inverse correlation. Phishing emails are generally much shorter (average 71 words) compared to legitimate emails (average 193 words). Attackers want the user to take action quickly without reading too much context.
