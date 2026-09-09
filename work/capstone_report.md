# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Abdul Wasay
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/AbdulWasay65/flyrank-ml-internship
- **Date:** 2026-09-09

## 0. Abstract

This capstone asks whether page-level Google Search Console signals from March 2026 can prioritize pages that subsequently improve by at least two average-position places in April 2026. The analysis uses the FlyRank ML Internship warehouse, aggregating anonymized daily content-performance observations to the client/content-page level. A Random Forest uses March impressions, clicks, and average position, evaluated against a transparent Week-4 baseline with client-grouped validation. The Random Forest achieved 0.3685 Average Precision versus 0.2086 for the baseline, an observed gain of 0.1599 AP points, with grouped ROC AUC of 0.7947. The output is intended as a human-review prioritization signal for SEO/content teams, not as causal proof or an automatic content-publishing rule.

## 1. Problem framing

Content teams can have more pages to review than available editorial or SEO capacity allows. The decision supported here is which pages should be reviewed first. The unit of analysis is an anonymized client/content-page observation; the output is a model score/rank; the human action is to review the highest-priority pages using content, search-intent, technical, and SERP context. A wrong high-priority call costs reviewer time, while an incorrect low-priority call can delay attention to a page that might merit intervention. ML is useful here because a ranked signal can help allocate limited review capacity more consistently than an unranked page list.

## 2. Data safety

The analysis uses the FlyRank internship warehouse's daily `fact_content_daily_performance` data. March 2026 provides decision-time features: impressions, clicks, and average position. April 2026 provides the subsequent outcome used to define the target. Client identifiers are pseudonymous and used only for grouped validation; they are not model features. Label-derived fields such as trend direction or trend percentage are not used as features. Public artifacts exclude client names, domains, private URLs, search queries, credentials, and raw private exports.

Pages were included when usable Google Search Console data existed in both March and April and average position was non-null for both periods. Pages without measurable April outcomes were excluded because subsequent improvement could not be evaluated reliably.

## 3. Baseline

The Week-4 baseline provides a transparent comparison for the same prioritization task and evaluation metric. Its Average Precision was 0.2086. Comparing the Random Forest with this baseline on the same task makes the model's incremental ranking value explicit rather than presenting an isolated model score.

## 4. Model / analysis

The method is a Random Forest classifier because the lane requires a ranked prioritization signal and the model can capture nonlinear relationships without requiring a complex feature pipeline. The exact features are March impressions, March clicks, and March average position. The target is `1` when April average position is at least two positions better than March, formally `April average position <= March average position - 2`; otherwise it is `0`. The model uses 150 trees, maximum depth 6, minimum leaf size 50, balanced class weights, and random seed 42.

## 5. Evaluation

The train/test split is grouped by `client_hash_id`, with 80% of client groups used for training and 20% for testing, using `random_state=42`. Grouping reduces the risk of pages from the same client appearing in both sets. April outcome information is used only to construct the target and is not supplied as a feature.

The Random Forest achieved Average Precision of 0.3685 versus 0.2086 for the baseline, an observed improvement of +0.1599 AP points. The validation audit reported grouped ROC AUC of 0.7947. The model produced 8,506 predicted positives against 2,469 actual positives, including 6,469 false positives and 432 false negatives. This error profile supports using the score for prioritization and human review rather than automatic decisions.

## 6. Interpretation

The strongest model signal was March average position, which accounted for approximately 80.8% of feature importance. Impressions accounted for approximately 12.5% and clicks approximately 6.8%. The result suggests that the selected March search-performance signals contain useful directional information for ranking pages by subsequent improvement likelihood.

The main surprise and caution is the size of the false-positive set. A useful ranking model can still send many pages to review that do not meet the eventual outcome definition. Some pages also receive identical scores when their input features are identical, limiting the usefulness of the score alone as a complete ordering within tied groups.

## 7. Recommendation

1. **Review high-scoring pages first.** Use the score to allocate human review capacity; do not auto-publish changes.
2. **Validate search intent and page relevance.** Confirm that the page matches the intended query/topic and provides useful information.
3. **Inspect content quality and freshness.** Check outdated information, missing specificity, title/H1 quality, usefulness, evidence, and internal linking.
4. **Check technical and SERP context.** Review indexing/technical issues and competing search results before recommending an intervention.
5. **Record the human decision.** Use APPROVE, MODIFY, REJECT, or HOLD and record the reason for auditability.

Confidence is appropriate for directional prioritization on the analyzed dataset and evaluation window, not for causal or universal claims. The score must not be used alone to rewrite, delete, publish, or make irreversible SEO changes.

## 8. Reproducibility

The capstone notebook, validation audit, action playbook, metrics JSON, and ranked queue are committed under `work/`. The model uses random seed 42. The analysis follows the repository's DuckDB-over-remote-Parquet workflow: filter and aggregate the warehouse in SQL, bring only the modeling frame into the ML environment, and avoid committing datasets. The deployed paper mirrors the capstone narrative.

Key artifacts:

- `work/notebooks/capstone.ipynb`
- `work/notebooks/w06_validation_audit.ipynb`
- `work/notebooks/w07_action_playbook.ipynb`
- `work/outputs/ml10_playbook_metrics.json`
- `work/outputs/ml10_ranked_action_queue.csv`
- `submission/paper_url.txt`

## 9. Acknowledgments & data credit

Built on the [FlyRank ML Internship dataset](https://flyrank.ai). Public-facing material uses anonymized, public-safe language and does not identify clients, domains, private queries, or credentials.

---

> **Claims checklist:** findings are framed as observed, measured, directional, and decision-support evidence. The model does not predict Google's algorithm and does not establish causal refresh impact.
