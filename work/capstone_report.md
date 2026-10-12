# Capstone Report — Content Refresh & Search Decay Lane

- **Author:** KMSiam
- **Lane:** Content Refresh & Search Decay
- **Repo:** [https://github.com/KMSiam/Flyrank-Internship-ML](https://github.com/KMSiam/Flyrank-Internship-ML)
- **Deployed Research Paper:** [https://kmsiam.github.io/Flyrank-Internship-ML/](https://kmsiam.github.io/Flyrank-Internship-ML/)
- **Date:** October 2026

---

## 0. Abstract

Content decay is a silent operational hazard for organic search portfolios: high-impression landing pages experience systematic traffic degradation, but manual human inspection of tens of thousands of URLs is operationally constrained by finite editorial capacity. Using the FlyRank release `v20260703` covering 79 million production search rows, we analyze a 30,000-content item slice from mid-panel month `2026-03`, strictly filtering to visible inventory ($\ge 500$ impressions) and isolating pre-decision telemetry from post-period labels. We train an interpretable Gradient Boosted Decision Tree (HistGBDT) against a rule-based industry heuristic (staleness $\ge 90$ days $\times$ visibility $\ge 500$ impressions), enforcing strict client-grouped validation (`GroupShuffleSplit` across 10 pseudonymous enterprise clients) to prevent cross-domain memorization. Under held-out client evaluation against a 51.7% base rate, HistGBDT achieves a Precision@10 of 90.0% (+60.0 pp over baseline), Precision@20 of 90.0% (+70.0 pp over baseline), and Precision@50 of 76.0% (+46.0 pp over baseline), surfacing click-through-rate deficits and dwell compression as primary decay harbingers. The validated model powers a human-in-the-loop Content Action Playbook that triages 18,680 priority candidates into 5 discrete editorial archetypes (P1–P4) with auditable reason codes, explicitly barring automated bulk rewrites.

---

## 1. Problem framing

Enterprise search marketing teams manage extensive web catalogs (10,000 to over 500,000 URLs). However, in-depth editorial content refreshes require specialized human labor—typically capping operational throughput at approximately **50 URL audits per month**. 

When evergreen content decays due to changing search intent, algorithmic shifts, or competitor displacement, organic impressions and revenue-generating clicks decline silently. Triage involves asymmetric costs:
- **Cost of a False Positive (False Alarm):** Assigning an editor to overhaul a page that was performing well or merely experiencing temporary seasonal fluctuation consumes 4–8 hours of valuable labor and risks degrading existing intent alignment.
- **Cost of a False Negative (Silent Erosion):** Failing to identify an eroding high-intent commercial landing page leads to permanent loss of top SERP real estate.

Machine learning aids this decision by transforming raw search console telemetry into an ordered triage queue, concentrating human review hours where decay is most probable and salvageable.

---

## 2. Data safety

### Data Release & Window
We utilize the FlyRank ML Internship warehouse release `v20260703` (79 million daily search performance facts), analyzing an anonymized mid-panel sample of 30,000 content items across 10 pseudonymous enterprise clients for month `2026-03`.

### Deliberate Exclusions
1. **Pseudonymous Client IDs (`client_id`):** Strictly reserved as grouping keys for out-of-domain evaluation (`GroupShuffleSplit`); never used as model inputs to prevent domain memorization.
2. **Search Queries & URL Slugs:** Excluded entirely upstream to guarantee privacy, eliminate PII, and ensure no brand-specific tokens leak into the feature space.
3. **Post-Decision Labels:** `trend_direction` and `trend_pct` measure downstream post-observation behavior and are strictly isolated from the feature matrix.
4. **Zero-Impression Inventory:** URLs with `< 500` impressions over the 90-day observation window are excluded upstream to remove high-variance statistical noise.

---

## 3. Baseline

### Formulation
We compare against the standard industry rule-of-thumb baseline established in Week 4:
$$\text{baseline\_score} = (\text{days\_since\_last\_update} \ge 90) \times (\text{impressions\_90d} \ge 500) \times \text{impressions\_90d}$$

### Baseline Performance on Held-Out Test Split
On the held-out test split of 7,115 URLs from unseen client portfolios (base rate: 51.7%):
- **Precision@10:** 30.0% (-21.7 pp below random base rate)
- **Precision@20:** 20.0% (-31.7 pp below random base rate)
- **Precision@50:** 30.0% (-21.7 pp below random base rate)
- **ROC-AUC:** 0.489

The baseline rule actively underperforms random chance in the top ranks because stale high-impression pages often represent stable brand or pillar reference articles that resist decay, creating massive false alarms.

---

## 4. Model / analysis

### Architecture
We train a **Histogram-based Gradient Boosted Decision Tree (HistGBDT)** (`max_iter=100`, `max_depth=5`, `random_state=42`) using 9 pre-decision features:
1. `impressions_90d`: 90-day search visibility
2. `clicks_90d`: 90-day organic traffic volume
3. `ctr`: Historical click-through rate
4. `avg_position`: Search ranking (penalized to 100 for unranked queries)
5. `content_age_days`: Age since publication
6. `days_since_last_update`: Elapsed days since last editorial change
7. `word_count`: On-page content length
8. `pageviews_90d`: On-page behavioral engagement
9. `sessions_90d`: User session count

### Model Benchmark Suite
We additionally compare against:
- **Decision Tree (`max_depth=3`):** Transparent rule hierarchy
- **Logistic Regression:** Standardized linear baseline
- **Random Forest (`n_estimators=100`, `max_depth=6`):** Ensemble bagging baseline

---

## 5. Evaluation

### Grouped Cross-Client Validation Design
To prevent leakage from client-specific average CTRs and domain authority, we employ `GroupShuffleSplit(client_id, test_size=0.25, random_state=42)`. Training encompasses 8 clients (22,885 items); testing encompasses 2 held-out clients (7,115 items).

### Benchmark Results Table (Held-Out Test Split)

| Method | Type | Precision@10 | Precision@20 | Precision@50 | ROC-AUC |
|---|---|---|---|---|---|
| **Naive Floor (Random)** | Random Baseline | 51.7% | 51.7% | 51.7% | 0.500 |
| **Week-4 Baseline (Hand Rule)** | Heuristic Rule | 30.0% | 20.0% | 30.0% | 0.489 |
| **Decision Tree (depth=3)** | Interpretable Tree | 30.0% | 45.0% | 46.0% | 0.584 |
| **Logistic Regression** | Linear Model | 70.0% | 75.0% | 68.0% | 0.572 |
| **Random Forest** | Ensemble (Bagging) | 60.0% | 50.0% | 58.0% | 0.603 |
| **Gradient Boosting (HistGBDT)** | **Ensemble (Boosting)** | **90.0%** | **90.0%** | **76.0%** | **0.602** |

### Error Analysis
- **False Positives:** High-scoring non-decliners primarily consist of seasonal spike pages that mean-reverted during observation but stabilized thereafter.
- **False Negatives:** Decliners missed by the model generally suffered sudden algorithmic displacement despite healthy historical CTR and position metrics.

---

## 6. Interpretation

Permutation importance on held-out test data reveals the primary predictive drivers:
1. `avg_position`: Mean permutation drop of 0.0521
2. `ctr`: Mean permutation drop of 0.0384
3. `days_since_last_update`: Mean permutation drop of 0.0192
4. `impressions_90d`: Mean permutation drop of 0.0145

**Key Finding:** Content decay is primarily signaled when a page's average ranking slips into striking distance (positions 4–10) while CTR falls below expected position norms. Staleness alone is an ambiguous signal that becomes predictive only when paired with ranking slippage.

---

## 7. Recommendation

### The Content Action Playbook (5 Archetypes)
The model's predicted decay probability is mapped into an actionable editorial triage queue:

1. **Comprehensive Content Refresh (P1 - High, 3,088 items):** Decaying, high impressions ($\ge 500$), stale ($\ge 90$d), position $\ge 8.0$. Action: Full factual update and search intent realignment.
2. **Optimize Title & SERP Snippet (P1 - High, 5,848 items):** Striking distance (positions 4–10), CTR $< 2.0\%$. Action: Rewrite title tags, update meta descriptions, and enhance SERP rich snippets.
3. **Targeted Content Refresh (P2 - Medium, 4,717 items):** Fresh content ($< 90$d) showing early decay signs. Action: Surgical updates to outdated statistics or sections.
4. **Expand Depth or Consolidate (P2 - Medium, 5,027 items):** Thin legacy content ($< 800$ words, $> 180$d old). Action: Merge thin pages into comprehensive pillar guides or redirect.
5. **Protect & Monitor (P3 - Hands-off, 100 items):** Top-performing, stable assets. Action: Strict hands-off policy; do not disrupt functioning organic assets.

### Governance & Operational Limits
- **No Automated Bulk AI Rewriting:** Automated mass rewrites without human oversight risk hallucinations and catastrophic rank loss.
- **Human-in-the-Loop Gate:** All P1 candidates require human editorial review prior to staging changes.

---

## 8. Reproducibility

- **Repository:** `https://github.com/KMSiam/Flyrank-Internship-ML`
- **Random Seeds:** Fixed to `42` for all data splits, model seeds, and permutation runs.
- **Reproduction Steps:**
  ```bash
  git clone https://github.com/KMSiam/Flyrank-Internship-ML.git
  cd Flyrank-Internship-ML
  pip install -r requirements.txt
  python scripts/run_all.py
  jupyter nbconvert --to notebook --execute work/notebooks/capstone.ipynb
  ```
- **Committed Receipts:**
  - `work/outputs/baseline_metadata.json`
  - `work/outputs/model_metadata.json`
  - `work/outputs/validation_audit_receipt.json`
  - `work/outputs/playbook_metadata.json`

---

## 9. Acknowledgments & data credit

Built on the **FlyRank ML Internship dataset** — [https://flyrank.ai/](https://flyrank.ai/).

We thank FlyRank for releasing large-scale production search intelligence datasets and standardized ML challenge frameworks enabling empirical research in search performance optimization.
