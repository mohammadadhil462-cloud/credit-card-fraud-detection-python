# Project Documentation: Credit Card Fraud Analysis

This document gives the detail behind the [README](../README.md): the dataset, the cleaning steps, how each analysis was done, the full results, and how to reproduce them.

## 1. Objective

Quantify the fraud problem in a credit card transaction dataset, find where and when fraud concentrates, and test whether simple, explainable business rules can reduce losses without creating too many false alerts.

## 2. Dataset

| Property | Value |
|---|---|
| File | `credit_card_fraud_detection_2026.csv` |
| Rows | 100,000 transactions |
| Columns | 43 |
| Period | 1 Jan 2026 to 31 Dec 2026 |
| Target | `is_fraud` (1 = fraud, 0 = legitimate) |
| Class balance | 2,055 fraud (2.05%), 97,945 legitimate |
| Duplicates | None (`transaction_id` is unique) |

### Data dictionary

Descriptions are based on column names and observed values in the notebook.

**Identifiers**

| Column | Description |
|---|---|
| `transaction_id` | Unique transaction ID (e.g. TXN-00000001) |
| `customer_id` | Customer ID |
| `card_id` | Card ID |
| `merchant_id` | Merchant ID |

**Transaction details**

| Column | Description |
|---|---|
| `transaction_datetime` | Date and time of the transaction |
| `transaction_amount` | Transaction value |
| `merchant_category` | Merchant type (e.g. Online Retail, Travel & Airlines, Gas Station) |
| `merchant_country` | Country of the merchant |
| `transaction_hour` | Hour of day (0-23) |
| `transaction_day_of_week` | Day name |
| `is_weekend` | 1 if on a weekend |
| `transaction_channel` | In-store, ATM, Mobile App or Online |
| `device_type` | Device used |
| `browser` | Browser used (blank for in-store and ATM) |

**Card security and location**

| Column | Description |
|---|---|
| `card_present` | 1 if the physical card was present |
| `used_chip` | 1 if chip was used |
| `used_pin` | 1 if PIN was used |
| `international_transaction` | 1 if cross-border |
| `high_risk_country` | 1 if the merchant country is flagged high risk |
| `distance_from_home_km` | Distance from the customer's home |
| `distance_from_last_tx_km` | Distance from the previous transaction |

**Customer and account**

| Column | Description |
|---|---|
| `customer_age` | Age in years |
| `customer_gender` | Gender |
| `customer_state` | US state of the customer |
| `account_age_days` | Age of the account in days |
| `credit_limit` | Credit limit |
| `credit_score` | Credit score |
| `num_cards_held` | Number of cards held |
| `email_domain` | Email provider group (gmail, yahoo, hotmail, corporate, other) |

**Behavioural history**

| Column | Description |
|---|---|
| `transactions_last_1h` / `_24h` / `_7d` | Transaction counts in recent windows |
| `amount_last_1h` / `_24h` | Amount spent in recent windows |
| `days_since_last_transaction` | Days since the previous transaction |
| `avg_transaction_amount_30d` | Customer's 30-day average amount |
| `std_transaction_amount_30d` | Customer's 30-day standard deviation of amount |
| `amount_vs_avg_ratio` | Transaction amount relative to the customer's average |
| `num_declined_last_30d` | Declined transactions in the last 30 days |
| `num_chargebacks_ever` | Lifetime chargebacks |

**Fraud labels**

| Column | Description |
|---|---|
| `is_fraud` | Fraud flag (target) |
| `fraud_type` | Legitimate, Card-Not-Present, Account-Takeover, Synthetic-Identity or Card-Present |
| `fraud_amount_lost` | Money lost on the transaction (0 for legitimate) |

**Columns added in the notebook:** `Month`, `Month_Name`, `age_band`, `account_age_band`.

## 3. Data Cleaning

| Issue | Finding | Action |
|---|---|---|
| Missing values | Only `browser` had nulls (41,709) | All were in-store (35,777) or ATM (5,932) transactions, so they are structural, not errors. Filled with `None (not online)` and kept every row |
| Duplicates | 0 duplicate `transaction_id` | None needed |
| Data types | `transaction_datetime` was text | Converted with `pd.to_datetime` |
| Feature engineering | Needed for time and segment analysis | Added `Month`, `Month_Name`, `age_band`, `account_age_band` |

## 4. Analysis and Detailed Findings

### 4.1 Size of the problem

| Metric | Value |
|---|---|
| Total transactions | 100,000 |
| Fraud transactions | 2,055 |
| Fraud rate | 2.05% |
| Total transaction value | 12,687,197.67 |
| Total fraud loss | 279,842.20 |
| Avg fraud amount | 169.93 |
| Avg legitimate amount | 125.97 |

Fraudulent transactions are on average about 35% larger than legitimate ones.

### 4.2 Loss by fraud type

| Fraud type | Cases | Amount lost | Share of loss |
|---|---|---|---|
| Card-Not-Present | 1,054 | 150,504.04 | 53.8% |
| Account-Takeover | 447 | 62,294.02 | 22.3% |
| Synthetic-Identity | 309 | 34,860.41 | 12.5% |
| Card-Present | 245 | 32,183.73 | 11.5% |

### 4.3 Where fraud concentrates

**Channel**

| Channel | Fraud rate |
|---|---|
| Online | 3.18% |
| Mobile App | 1.91% |
| ATM | 1.30% |
| In-store | 0.80% |

**Top merchant categories:** Online Retail (3.32%), Travel & Airlines (2.92%), Entertainment & Streaming (2.50%), Utilities & Telecom (2.32%), Hotel & Lodging (2.09%).

**Top merchant countries**

| Country | Fraud rate | Frauds | Transactions |
|---|---|---|---|
| Romania | 12.94% | 122 | 943 |
| Russia | 11.38% | 119 | 1,046 |
| Nigeria | 10.94% | 138 | 1,261 |
| China | 9.87% | 166 | 1,682 |

All other countries are at or below 3.3%.

**Card security and location flags**

| Flag | Value = 0 | Value = 1 |
|---|---|---|
| `international_transaction` | 0.35% | 3.42% |
| `high_risk_country` | 1.59% | 11.05% |
| `card_present` | 2.75% | 0.80% |
| `used_chip` | 2.65% | 0.75% |
| `used_pin` | 2.34% | 0.77% |

### 4.4 When fraud happens

- **Hour of day:** the highest rates are at 3am (11.30%), 1am (8.38%), 4am (7.81%) and 2am (7.48%). Midnight is 2.67%.
- **Overnight window (0-4am):** 2,456 transactions, 150 of them fraud.
- **Weekend vs weekday:** 2.09% vs 2.04%, essentially no difference.
- **Month:** the monthly rate moves within a narrow band (about 1.9% to 2.3%) with no strong seasonal pattern. October is the highest month and June the lowest.

### 4.5 Customers and accounts at risk

**Account age**

| Account age | Fraud rate | Frauds | Transactions |
|---|---|---|---|
| < 3 months | 4.85% | 515 | 10,608 |
| 3-12 months | 1.89% | 484 | 25,667 |
| 1-3 years | 1.78% | 688 | 38,549 |
| 3+ years | 1.46% | 368 | 25,176 |

**Prior chargebacks**

| Chargebacks | Fraud rate | Transactions |
|---|---|---|
| 0 | 1.76% | 77,032 |
| 1 | 2.07% | 13,649 |
| 2 | 4.34% | 6,039 |
| 3 | 4.83% | 2,154 |
| 4 | 4.26% | 1,126 |

**Little to no signal:** age band (1.88% to 2.17%), email domain (1.86% to 2.22%), and customer state (the top state, Iowa, is 2.89%, which is a modest spread across states of similar size).

### 4.6 Rule-based detection

Definitions used in the notebook:

| Rule | Logic |
|---|---|
| High-risk country | `high_risk_country == 1` |
| Overnight (0-4am) | `transaction_hour` between 0 and 4 |
| New account (<= 90 days) | `account_age_days <= 90` |
| 2+ chargebacks | `num_chargebacks_ever >= 2` |
| International + card not present | `international_transaction == 1` and `card_present == 0` |
| Amount > 3x customer average | `amount_vs_avg_ratio > 3` |
| Combined (4 rules) | High-risk country OR overnight OR new account OR 2+ chargebacks |

**Performance** (total fraud = 2,055)

| Rule | Flagged | Frauds caught | Recall | Precision | False alerts |
|---|---|---|---|---|---|
| High-risk country | 4,932 | 545 | 26.5% | 11.1% | 4,387 |
| Overnight (0-4am) | 2,456 | 150 | 7.3% | 6.1% | 2,306 |
| New account (<= 90 days) | 10,608 | 515 | 25.1% | 4.9% | 10,093 |
| 2+ chargebacks | 9,319 | 414 | 20.1% | 4.4% | 8,905 |
| International + card not present | 35,749 | 1,634 | 79.5% | 4.6% | 34,115 |
| Amount > 3x customer average | 14,645 | 447 | 21.8% | 3.1% | 14,198 |
| Combined (4 rules) | 24,730 | 1,228 | 59.8% | 5.0% | 23,502 |

**Cost model**

```
Net benefit = fraud_amount_lost on flagged transactions - (false alerts x cost per alert)
```

Base case uses a cost of **5 per false alert**.

| Rule | Money saved | Alert cost | Net benefit |
|---|---|---|---|
| Amount > 3x customer average | 186,515 | 70,990 | 115,525 |
| International + card not present | 227,760 | 170,575 | 57,185 |
| High-risk country | 76,226 | 21,935 | 54,291 |
| Combined (4 rules) | 163,181 | 117,510 | 45,671 |
| New account (<= 90 days) | 67,658 | 50,465 | 17,193 |
| 2+ chargebacks | 53,391 | 44,525 | 8,866 |
| Overnight (0-4am) | 17,545 | 11,530 | 6,015 |

**Sensitivity to the cost per false alert**

| Rule | Cost = 1 | Cost = 5 | Cost = 10 |
|---|---|---|---|
| Amount > 3x customer average | 172,317 | 115,525 | 44,535 |
| High-risk country | 71,839 | 54,291 | 32,356 |
| International + card not present | 193,645 | 57,185 | -113,390 |
| Combined (4 rules) | 139,679 | 45,671 | -71,839 |
| New account (<= 90 days) | 57,565 | 17,193 | -33,272 |
| 2+ chargebacks | 44,486 | 8,866 | -35,659 |
| Overnight (0-4am) | 15,239 | 6,015 | -5,515 |

Only two rules, **Amount > 3x customer average** and **High-risk country**, stay profitable across all three cost levels.

## 5. Conclusions

1. Fraud costs about 2.2% of total transaction value (279,842 of 12.69M) across 2.05% of transactions.
2. Card-Not-Present fraud and the online channel are the main exposure.
3. The strongest single predictors are **high-risk country**, **overnight timing**, **new accounts**, **prior chargebacks**, and **missing chip/PIN/card-present security**.
4. The most reliable rules are **amount > 3x customer average** and **high-risk country**. Broad rules that catch a lot of fraud (international + card not present) can lose money when review costs rise.

## 6. Limitations

- Savings assume every flagged fraud is stopped.
- The cost per false alert is an assumption, which is why sensitivity analysis is included.
- No train/test split was used, so rule performance is measured in-sample.
- Rules were evaluated individually. Only one combination (the 4-rule OR) was tested.
- The analysis is descriptive and rule-based. No predictive model was built.

## 7. Future Work

- Test layered rules such as `amount > 3x average` combined with `high-risk country`.
- Train a classification model (logistic regression, random forest, gradient boosting) with a train/test split and evaluate with precision-recall metrics suited to imbalanced data.
- Use `distance_from_home_km`, velocity features (`transactions_last_1h`) and `credit_score`, which were not analysed here.
- Build a Power BI dashboard from the findings.

## 8. Reproducing the Analysis

1. Install dependencies: `pip install -r requirements.txt`
2. Place `credit_card_fraud_detection_2026.csv` in the project folder and update the path in the first notebook cell.
3. Run `Fraud_detection.ipynb` from top to bottom.

## 9. Notebook Map

| Section | Content |
|---|---|
| Setup and cleaning | Imports, load data, null handling, datatype fix, month features |
| Business Question 1 | Overall fraud size and average amounts |
| Business Question 2 | Loss by fraud type |
| Business Question 3 | Channel, category, country and card security analysis |
| Business Question 4 | Hour, weekend and monthly patterns |
| Business Question 5 | Age, account age, chargebacks, state, email domain |
| Business Question 6 | Rule definitions, recall and precision, cost model, sensitivity analysis |
