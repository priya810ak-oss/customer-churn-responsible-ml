# Risk Register — Customer Churn ML

| ID | Risk | Evidence / trigger | Impact | Mitigation | Owner / review |
|---|---|---|---|---|---|
| R1 | Small/non-representative sample | Only 12 rows | High | Collect larger representative historical data; document sampling | Data/ML owner |
| R2 | Temporal leakage | No timestamps or event order | High | Define scoring cutoff and future outcome window; build time-based validation | ML + data owner |
| R3 | Plan-type separation | Basic: 5/5 churned; Standard/Pro: 0 churned | High | Treat as sample artifact until validated; inspect subgroup errors; avoid automated adverse action | ML + business owner |
| R4 | Identifier memorization/privacy | `customer_id` is unique for every row | Medium | Exclude identifier from model features; restrict access | Data owner |
| R5 | False positives | Unnecessary retention review/outreach | Medium | Human review, outreach caps, track precision/FP cost | Operations |
| R6 | False negatives | Churner is not flagged | High | Optimize/monitor recall; review FN cases; validate intervention threshold | Operations + ML |
| R7 | Proxy/commercial segmentation | `plan_type` and `monthly_spend_inr` are closely related in sample | Medium | Review business meaning; test redundancy/proxy effects; document intended use | Data/Responsible AI |
| R8 | Overconfidence from tiny validation set | Near-perfect sample-level patterns | High | Time-based external validation; confidence intervals where feasible; calibration on larger data | ML owner |
| R9 | Undocumented provenance/consent | Source file lacks provenance/permission details | High | Obtain documentation before real-customer use | Data governance |
| R10 | Misuse outside intended purpose | Score could be reused for punitive decisions | High | Restrict use to retention support; access controls; documented prohibited uses | Product/legal/governance |
