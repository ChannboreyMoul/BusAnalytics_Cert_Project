# 🔑 Key Security & Data Analytics Findings

This document summarizes the core indicators that separate legitimate business correspondence from phishing attempts based on our data modeling.

---

## 1. Top Threat Signals (Ranked by Predictive Power)

Our machine learning model coefficients and SHAP value importance rankings show which email signals are the most predictive of phishing:

```
Rank  Feature                 Predictive Strength   Diagnostic Direction
---------------------------------------------------------------------------------
1st   has_urgency_keywords    Extremely High        Phishing has threat keywords
2nd   subject_length          Very High             Phishing subjects are longer
3rd   urgency_score           Very High             Phishing is highly urgent (7+)
4th   spelling_errors         High                  Phishing contains 3+ typos
5th   email_length_words      Medium-High (Inverse) Phishing is short (< 120 words)
6th   has_link                Low-Medium            Phishing is more likely to link
7th   is_suspicious_domain    None (Uniform Data)   Logical check for spoofed domains
8th   has_attachment          None (Uniform Data)   No statistical class variance
```

---

## 2. In-Depth Signal Breakdown

### 🚨 Signal A: Urgency and Emotional Pressure (`urgency_score`)
Phishing emails are highly urgent. The average urgency score is **8.49 out of 10** for phishing emails, compared to **3.03 out of 10** for legitimate emails. 
- Attackers manipulate users by creating fake emergencies (e.g., account lockouts, immediate password expirations) to prompt quick actions before verification.

### ✍️ Signal B: Spelling & Grammar Errors (`spelling_errors`)
Legitimate emails average **1.03 spelling errors**, whereas phishing emails average **6.46 errors**.
- High counts of spelling mistakes are common in phishing. This occurs because attackers may use poor translations, bypass automated email security rules using visual similarity homoglyphs, or filter out highly cautious users by target selection.

### 📏 Signal C: Email Body Word Count (`email_length_words`)
There is a strong inverse relationship between word count and phishing:
- **Legitimate emails** tend to provide detailed context and average **193 words**.
- **Phishing emails** are kept short, averaging **71 words**, to drive the user to click a call-to-action link before they can read and analyze the context.

### 🏷️ Signal D: Subject Keywords (`has_urgency_keywords`)
- **100.00%** of phishing email subjects in the dataset contain threat/urgency keywords (e.g., *Verify, Action, Account Locked, Suspicious Login, Claim, Reward, Expire*).
- **0.00%** of legitimate subjects in the dataset contain these terms (which instead focus on topics like *Meeting Reminder, Interview Schedule, Invoice Attached, Project Update*).
- Filtering subjects for these key threat phrases acts as an immediate and highly effective first-line defense.
