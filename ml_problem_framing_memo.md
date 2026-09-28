# ML Problem-Framing Memo — Customer Churn

## 1. Decision
The proposed ML system supports a **human-reviewed retention workflow**. It flags customers who may be at risk of churn so a retention/support team can decide whether to contact them with an appropriate support or retention option.

### Out of scope
The model must not:
- automatically cancel or suspend accounts;
- automatically deny service;
- automatically change prices or benefits;
- treat a high score as proof of future churn.

## 2. Prediction target
- **Unit of observation:** customer.
- **Target:** `churned`.
- **Target meaning in supplied data:** 1 = churned, 0 = not churned.
- **Required production definition:** churn within a documented future window after a scoring cutoff (for example, next 30 days). The supplied data does not contain timestamps, so this definition cannot currently be verified.

## 3. Action window
For a production workflow, define:
1. scoring cutoff date/time;
2. feature-availability cutoff;
3. prediction window;
4. intervention window;
5. label-finalization date.

Without these dates, leakage and realistic evaluation cannot be established.

## 4. Non-ML baseline
Use the following transparent baseline for review:

`flag = (last_login_days >= 14) OR (support_tickets >= 4)`

On the supplied 12 rows, this rule produces:
- True negatives: 7
- False positives: 0
- True positives: 4
- False negatives: 1
- Accuracy: 91.7%
- Precision: 100%
- Recall: 80.0%
- F1: 88.9%

These are descriptive results on the training file only, not an estimate of future production performance.

A second baseline is the majority-class predictor (`churned = 0` for every customer), which achieves 58.3% accuracy on this dataset but has 0% recall for churners.

## 5. Feature plan
Candidate predictors:
- `tenure_months`
- `support_tickets`
- `monthly_spend_inr`
- `last_login_days`
- `plan_type`

Exclude:
- `customer_id` (identifier/memorization risk).

Before training, verify that every feature is available at the scoring cutoff and is not generated after or from the churn outcome.

## 6. Model strategy
A simple interpretable model such as logistic regression with one-hot encoding for `plan_type` is a reasonable demonstration model. However, the current dataset is only 12 rows, so any fitted model should be treated as exploratory.

The sample shows complete separation by `plan_type`:
- Basic: 5/5 churned
- Standard: 0/4 churned
- Pro: 0/3 churned

This pattern is too strong to generalize from only 12 records.

## 7. Error costs
Initial working assumption for evaluation:
- **False negative cost = 5 units**: missing a customer who churns is more costly to the retention workflow.
- **False positive cost = 1 unit**: unnecessary review/outreach has a smaller but real operational cost.

These are **proposed assumptions, not measured business costs**. A business owner should validate them.

Expected cost:
`cost = 5 × FN + 1 × FP`

For the proposed rule: `5×1 + 1×0 = 5` cost units on the supplied sample.

## 8. Abstention / human review
The model should abstain when:
- probability is close to the intervention threshold;
- required features are missing;
- the customer is outside the population represented by the training data;
- monitoring detects drift or unusual input values.

Abstained cases go to human review rather than an automatic action.

## 9. Monitoring and rollback
Monitor:
- feature missingness;
- feature distributions;
- churn prevalence;
- precision/recall when labels mature;
- calibration;
- false-positive and false-negative rates;
- subgroup error rates where legitimate group data exists.

Rollback/disable the model if:
- leakage is discovered;
- performance materially degrades against the agreed baseline;
- fairness/error disparities exceed a pre-agreed threshold;
- data quality checks fail;
- intended-use controls are bypassed.

## 10. Go/no-go conditions
The current file is suitable for a **workflow demonstration only**.

Before production:
- document provenance and permissions;
- obtain a larger representative dataset;
- add timestamps and a clearly defined outcome window;
- perform time-based validation;
- validate the cost assumptions;
- review privacy and proxy risks;
- test calibration and subgroup errors;
- establish monitoring and rollback ownership.
