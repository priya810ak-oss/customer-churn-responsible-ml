# Responsible Data Card — Customer Churn Training Data

## Dataset purpose
**Supported decision:** Use the dataset to support a human-reviewed customer-retention workflow: identify customers who may be at risk of churn so a retention/support team can decide whether an appropriate, non-coercive outreach action is useful.

**Decisions it must not make:** The dataset/model must not autonomously cancel accounts, deny service, change prices, remove benefits, or make other high-impact decisions about a customer. A model score should be treated as a risk signal, not as proof that a customer will churn.

**Unit of observation:** one customer record.

**Prediction target:** `churned` (1 = churned, 0 = not churned).

**Action window / scoring cutoff:** For a production version, all features must be measured at a clearly documented scoring cutoff and the model should predict churn during a defined future window (for example, the next 30 days). The supplied file contains no timestamp/window field, so this cannot be verified from the dataset alone.

**Non-ML baseline:** A simple human-readable review rule is proposed:
`review = last_login_days >= 14 OR support_tickets >= 4`.
This is only a baseline and must be validated against a real business-defined outcome window before deployment.

## Provenance and permission
The supplied file is a small training dataset named `customer-churn-training.csv`. The source file does not document who created the records, how they were collected, consent status, licensing terms, or permitted users.

Therefore:
- **Provenance:** not documented in the supplied dataset.
- **Consent/licensing:** not documented; must be confirmed before using real customer data.
- **Permitted use:** training/assessment use is assumed for this task; production use requires documented authorization and privacy review.
- `customer_id` should be treated as an identifier and should not be used as a predictive feature.

## Population and representation
The dataset contains **12 customer records**:
- Basic: 5
- Standard: 4
- Pro: 3

Target distribution:
- `churned = 1`: 5/12 (41.7%)
- `churned = 0`: 7/12 (58.3%)

All five Basic records are churned, while none of the Standard or Pro records are churned in this sample. This is a very small and potentially non-representative sample, so these observed rates must **not** be interpreted as population-level churn rates.

Missing groups cannot be determined because demographic/protected-group fields are not present. Sampling method is also undocumented.

## Features and target
| Feature | Meaning from supplied data | Use / risk |
|---|---|---|
| `customer_id` | Customer identifier | Exclude from modeling; can create memorization/privacy risk. |
| `tenure_months` | Customer tenure in months | Candidate predictor; exact measurement timing is not documented. |
| `support_tickets` | Number of support tickets | Candidate predictor; verify that the count is available before the scoring cutoff. |
| `monthly_spend_inr` | Monthly spend in INR | Candidate predictor; highly related to `plan_type` in this tiny sample, so redundancy/proxy risk should be checked. |
| `last_login_days` | Days since last login | Candidate predictor; can be highly informative but may become leakage if measured after the prediction cutoff. |
| `plan_type` | Basic / Standard / Pro | Candidate categorical predictor; may encode pricing/contract/business decisions and shows complete separation from the target in this sample. |
| `churned` | 1/0 churn outcome | Target label; must be defined with a clear time window in production. |

**Potential leakage:** No timestamp or event-order information is supplied. Therefore it is impossible to verify that every feature was known before the churn outcome. In particular, `last_login_days`, support activity, and plan information must be checked for temporal leakage.

**Sensitive proxies:** No explicit demographic/protected attributes are included. However, `plan_type` and `monthly_spend_inr` may proxy for customer segment or commercial treatment, so their use should be reviewed with domain owners.

## Quality checks
Observed in the supplied 12-row file:
- Missing values: **0** in every column.
- Duplicate rows: **0**.
- `customer_id` is unique: 12 unique IDs.
- Numeric ranges:
  - `tenure_months`: 1–30
  - `support_tickets`: 0–5
  - `monthly_spend_inr`: ₹499–₹1,499
  - `last_login_days`: 1–30
- No obvious numeric outlier was identified from the small sample using the IQR rule; however, n=12 is too small for a strong outlier conclusion.
- Class balance: 7 non-churned / 5 churned.
- Train/test separation: **not verifiable** from the supplied file because there is no split indicator or time field. A random split would be unstable for n=12; temporal separation should be used once timestamps are available.

## Risks and safeguards
1. **Sampling bias:** n=12 may not represent the customer population.
   - Safeguard: collect a larger, representative historical sample and document sampling.
2. **Target leakage:** feature timing is undocumented.
   - Safeguard: define a prediction cutoff and future outcome window; only use data available before the cutoff.
3. **Plan/segment bias:** the sample has 100% churn for Basic and 0% for Standard/Pro.
   - Safeguard: monitor subgroup error rates and do not make automated adverse decisions from the score.
4. **False positives:** unnecessary retention outreach can waste resources or annoy customers.
   - Safeguard: human review, outreach caps, and a measured false-positive cost.
5. **False negatives:** a customer who churns but is not flagged may be missed by retention efforts.
   - Safeguard: prioritize recall for the retention use case while tracking false-negative cost.
6. **Privacy/identifier risk:** `customer_id` is an identifier.
   - Safeguard: exclude it from features, minimize access, and use pseudonymous IDs where operationally possible.
7. **Model overconfidence:** perfect or near-perfect results on 12 rows may be an artifact of the sample.
   - Safeguard: use larger out-of-time validation data and calibrated probabilities before deployment.
8. **Misuse:** a churn score could be repurposed for punitive treatment.
   - Safeguard: restrict intended use to supportive retention decisions and document prohibited uses.

## Intended evaluation
**Business baseline:** Compare against the non-ML review rule and the majority-class baseline.

**Primary model metrics:** precision, recall, F1, confusion matrix, and PR-AUC where sample size permits.

**Calibration:** Brier score and calibration curve on sufficiently large validation data.

**Fairness:** If protected-group data becomes legitimately available, report group-level false-positive rate, false-negative rate, recall, and precision where sample sizes are adequate. Do not infer protected status from the current dataset.

**Error analysis:** Inspect false positives and false negatives by plan type and operational segment; investigate whether errors are concentrated in any group.

**Validation:** Prefer time-based train/validation/test separation for production data. The current 12-row dataset is suitable for demonstrating the workflow, not for establishing production performance.

## Bottom line
This dataset can demonstrate responsible churn-model framing and baseline analysis, but it is **not sufficient evidence for deployment**. The largest gaps are sample size, missing timestamps/outcome-window definition, undocumented provenance/consent, and the unusually strong relationship between `plan_type` and the target in this tiny sample.
