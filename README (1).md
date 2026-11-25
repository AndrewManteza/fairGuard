# FairGuard AI

A human-in-the-loop fraud investigation assistant that prioritizes suspicious transactions, explains why they were flagged, drafts investigation reports for review, and **measures whether detection errors differ across customer groups**.

> **Status:** Class project (2-week scope). Uses public/synthetic data only. Results demonstrate the *method*, not real-world findings about any population.

---

## Why This Project

| Impact area | What FairGuard AI does | Why it matters |
|---|---|---|
| Financial protection | Scores transactions and prioritizes cases for investigation | Helps investigators focus on potentially costly fraud |
| Operational efficiency | Gathers transaction evidence and drafts investigation reports for review | Reduces repetitive research and documentation work |
| Customer experience | Analyzes the false-alert trade-off at different thresholds | Fewer legitimate transactions incorrectly flagged |
| Fairness assessment | Compares false-positive and false-negative rates across groups | Makes disparities visible instead of hiding them behind overall accuracy |
| Accountability | Logs scores, explanations, drafts, and reviewer decisions | Lets reviewers verify how conclusions were reached |

**Design principle:** the system *assists* investigators. It ranks and explains; a human decides.

---

## Architecture

```
 Transactions (CSV) ──► Feature pipeline ──► Detection model (risk score)
                                                     │
                                                     ▼
                                         Ranked case queue (score × amount)
                                                     │
                    ┌────────────────────────────────┼───────────────────────┐
                    ▼                                ▼                       ▼
          Evidence + report drafter          SHAP explanations        Fairness monitor
          (template, optional LLM)           (top factors per case)   (Fairlearn, by group)
                    │                                │                       │
                    └────────────────────────────────┼───────────────────────┘
                                                     ▼
                                      Audit log (CSV / SQLite)
```

| Component | Role |
|---|---|
| Feature pipeline | Cleans data; builds amount, hour, distance, merchant category, age features |
| Detection model | Logistic regression baseline + XGBoost; time-based train/test split |
| Case queue | Ranks flagged transactions by risk score × amount (expected-loss proxy) |
| Explanations | SHAP top contributing factors for each flagged case |
| Report drafter | Fills an evidence-based template; LLM polish is optional, and every claim cites a transaction ID |
| Fairness monitor | Group-sliced false-positive rate, false-negative rate, and flag rate |
| Audit log | Case ID, timestamp, score, factors, draft, reviewer decision |

---

## Data

| Dataset | Use |
|---|---|
| Sparkov synthetic credit card fraud (Kaggle) | Primary: includes gender and date of birth, enabling group analysis |
| IEEE-CIS / ULB (Kaggle) | Optional benchmarks (limited or no demographics) |

Datasets are not committed to this repo. See `data/README.md` for download steps. Check each dataset's license on Kaggle and cite accordingly.

**Limitation:** demographic fields in synthetic data are not real people, so fairness findings validate the *approach*.

---

## Tech Stack (all free)

Python · pandas · scikit-learn · XGBoost · SHAP · Fairlearn · SQLite/DuckDB · Google Colab · (optional) Streamlit · (optional) Ollama or a free LLM API tier

---

## Evaluation

- **Detection:** PR-AUC and recall at a fixed false-positive rate (not accuracy, since fraud is highly imbalanced)
- **Fairness:** false-positive-rate gap across gender and age bands (primary metric), plus false-negative-rate gap and flag-rate gap
- **Customer experience:** threshold analysis showing false positives vs. fraud caught
- **Optional stretch:** inject a known synthetic bias and confirm the fairness monitor detects it

---

## Repository Structure

```
fairguard-ai/
├── README.md
├── data/               # download instructions only (no raw data committed)
├── notebooks/          # exploration, modeling, fairness analysis
├── src/                # feature pipeline, scoring, report drafter, audit log
├── app/                # optional Streamlit demo
├── docs/               # system design doc, diagrams
└── outputs/            # metrics, charts, sample case reports, audit log
```

---

## Two-Week Plan

| Days | Focus |
|---|---|
| 1–2 | Setup, data exploration, features, time-based split |
| 3 | Baseline and XGBoost models, evaluation |
| 4 | Fairness analysis with Fairlearn |
| 5 | Case ranking and threshold trade-off analysis |
| 8 | SHAP explanations |
| 9 | Evidence gathering and report drafting |
| 10 | Audit log |
| 11–12 | System design doc, slides, optional Streamlit page |
| 13–14 | Rehearsal and buffer |

---

## Known Limitations

- Public/synthetic data; no real customers or investigators
- Synthetic demographics: results demonstrate methodology only
- No measured time savings (would require a real investigator study)
- Batch scoring only; no real-time streaming
- Audit log is a simple table, not an immutable store

## Production Path (out of scope for this project)

Streaming ingestion and feature store · graph features for fraud rings · champion/challenger model testing · continuous drift and fairness monitoring · append-only audit storage · legal/compliance review of how group data is collected and used · third-party model audits

---

## Fairness Notes

- Group attributes are used **only for measurement**, not as model inputs.
- Proxy features (e.g., location, merchant type) can still encode group information. Review them.
- Historical fraud labels can carry past bias, and only investigated cases produce labels, which can create feedback loops.
- Fairness metrics can conflict; this project prioritizes false-positive-rate parity and documents that choice.

---

## Team

_Add names and course info here._

## License

_Add a license (e.g., MIT) and dataset citations here._
