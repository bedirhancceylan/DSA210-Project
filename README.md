# DSA 210 — Analyzing Task Completion Patterns in an Early-Stage Startup

**Bedirhan Ceylan** — Spring 2026
**Section A — TA: Erfan Tarmohammadi**

This project analyzes 486 task records from the development workflow of [Sollen](https://www.sollen.io), an early-stage startup, to identify which factors are associated with longer completion times and delays. The pipeline covers exploratory analysis, hypothesis testing, feature engineering, and machine learning classification.

📄 **[Final Report (PDF)](DSA210_Final_Report_Bedirhan_Ceylan.pdf)** · **[Final Report (DOCX)](DSA210_Final_Report_Bedirhan_Ceylan.docx)**
🌐 **[Project Website](https://bedirhancceylan.github.io/DSA210-Project/)**
🤖 **[AI Usage Disclosure](AI_USAGE.md)**

---

## Dataset

Source: Azure DevOps export from Sollen's task tracking system, October 2024 – April 2026.

| Column | Type | Description |
|---|---|---|
| `task_id` | int | Unique task identifier |
| `title` | str | Free-text task title |
| `task_category` | str | tech, design, research, product, operations |
| `priority_level` | str | low, medium, high |
| `assigned_to` | str | Anonymized developer ID (Developer_A...N) |
| `creation_date` | datetime | When the task was opened |
| `completion_date` | datetime | When the task was closed |
| `actual_duration_days` | float | Days between creation and completion |
| `delay_label` | str | `delayed` if duration > 1.5 × category median, else `on_time` |
| `has_urgent_tag` | int | Binary urgent flag |

486 tasks across 15 developers, 5 categories. Overall delay rate: 34.8%.

---

## Project Structure

```
dsa210-project/
├── data/
│   ├── sollen_tasks.csv               # Raw export
│   ├── sollen_tasks_features.csv      # Engineered features + target (Day 1)
│   ├── model_comparison.csv           # Final summary table (Day 4)
│   └── final_model_random_forest.pkl  # Persisted final model (Day 4)
├── notebooks/
│   ├── eda.ipynb                      # EDA + 3 hypothesis tests
│   ├── ml.ipynb                       # ML Day 1: feature engineering
│   ├── ml_day2.ipynb                  # ML Day 2: baseline models
│   ├── ml_day3.ipynb                  # ML Day 3: tree ensembles
│   └── ml_day4.ipynb                  # ML Day 4: final model
├── plots/                             # 22 generated figures
├── docs/
│   └── index.html                     # Project website (GitHub Pages)
├── DSA210_Final_Report_Bedirhan_Ceylan.pdf
├── DSA210_Final_Report_Bedirhan_Ceylan.docx
├── AI_USAGE.md
├── project_proposal.pdf
├── requirements.txt
└── README.md
```

---

## EDA & Hypothesis Testing Findings

| # | Question | Test | Result |
|---|---|---|---|
| 1 | Does duration differ across categories? | Kruskal-Wallis | Reject H₀ (H = 39.97, p < 0.0001) — product/tech longest, ops/research shortest |
| 2 | Is category associated with delay status? | Chi-Square | Fail to reject (χ² = 5.13, df = 4, p = 0.275) — category alone is not a strong predictor |
| 3 | Do delayed tasks take longer than on-time? | Mann-Whitney U (one-sided) | Reject H₀ (U = 50,252, p < 0.0001, r = 0.876) — by construction, but quantified |

**Key insight:** the chi-square negative result motivated the ML phase. Since category alone does not predict delay, we hypothesized that the real signal lies in **feature interactions** (developer × workload × timing) — which tree ensembles captured.

---

## Machine Learning Pipeline

### Feature Engineering

10 leakage-free features were constructed from the raw columns:

| Feature | Source |
|---|---|
| `task_category`, `assigned_to`, `creation_day` | Direct categorical |
| `creation_month_num`, `is_weekend_creation`, `title_word_count` | Temporal / lexical |
| `team_total_load_at_creation` | Open tasks across team at task creation time |
| `developer_workload_at_creation` | Open tasks for the same developer |
| `dev_historical_delay_rate` | Developer's prior delay rate (using only tasks created before current) |
| `dev_avg_past_duration` | Developer's prior mean duration (same constraint) |

Critical leakage decisions:
- Dropped `actual_duration_days`, `completion_date`, `delay_label` (target was constructed from these)
- Dropped `priority_level` (99.2% one value) and `has_urgent_tag` (99.4% zero) — no variance
- Historical features computed using strictly past data only (no future leakage)

### Model Comparison

All models evaluated with stratified 5-fold cross-validation and an 80/20 stratified test split (random_state=42).

| Model | CV ROC-AUC | Test ROC-AUC | Test Accuracy | Test F1 (delayed) |
|---|---|---|---|---|
| Logistic Regression | 0.577 ± 0.039 | 0.639 | 0.62 | 0.53 |
| Decision Tree | 0.582 ± 0.030 | 0.573 | 0.60 | 0.47 |
| kNN (k=5) | 0.662 ± 0.050 | 0.710 | 0.70 | 0.47 |
| Gradient Boosting | 0.684 ± 0.033 | 0.725 | 0.71 | 0.48 |
| **Random Forest** | **0.708 ± 0.040** | **0.741** | **0.71** | **0.55** |

Random Forest selected as the final model — best on every metric.

### Feature Importance (Permutation, Random Forest)

| Feature | Importance (mean ROC-AUC drop) |
|---|---|
| `creation_month_num` | 0.063 |
| `task_category` | 0.043 |
| `dev_historical_delay_rate` | 0.039 |
| `assigned_to` | 0.039 |
| `team_total_load_at_creation` | 0.033 |
| `dev_avg_past_duration` | 0.027 |

**Top finding:** developer-history features (`dev_historical_delay_rate`, `dev_avg_past_duration`) and team load are stronger signals than the task's own category — supporting the chi-square conclusion that delay is a function of *who is doing the work and how much they already have on their plate*, not just *what kind of task it is*.

---

## Limitations

- **Sample size.** 486 tasks is small for ML; F1 std is ±0.045 across folds.
- **Single-team scope.** All data from one startup; patterns may not generalize.
- **Threshold definition.** 1.5× category median is conventional but not validated against external business definitions of delay.
- **Missing context features.** Story points, dependencies, vacation days, external blockers — none captured in Azure DevOps export.
- **Two zero-variance fields** (`priority_level`, `has_urgent_tag`) reflect the team not actively using these workflow fields — a process-level finding.
- **Interpretability trade-off.** Random Forest is harder to explain than logistic regression for stakeholder-facing deployment.

## Future Work

- **Threshold tuning** — current default of 0.5 produces a conservative model; precision-recall analysis with a domain cost matrix would identify a more useful operating point.
- **Hyperparameter tuning** — `GridSearchCV` over `n_estimators`, `max_depth`, `min_samples_leaf`.
- **Time-series cross-validation** — current shuffled CV does not respect temporal order over the 18-month window.
- **Richer feature set** — sprint identifiers, parent/child task links, story points, national holidays, team velocity rolling averages.
- **Multi-target prediction** — extend from binary classification to duration regression.
- **Deployment** — lightweight web interface accepting task category, assignee, current load → returns delay probability.
- **Benchmarking** — compare model against human intuition and trivial heuristics (assignee's prior delay rate alone) before any deployment decision.

---

## How to Reproduce

```bash
git clone https://github.com/bedirhancceylan/DSA210-Project.git
cd DSA210-Project
python3 -m venv venv
source venv/bin/activate          # macOS/Linux
pip install -r requirements.txt
jupyter notebook
```

Then run notebooks in order: `eda.ipynb` → `ml.ipynb` → `ml_day2.ipynb` → `ml_day3.ipynb` → `ml_day4.ipynb`.

All notebooks use `random_state=42` for any stochastic operation, so results are exactly reproducible.

### Milestone tags

- **`milestone1`** (cc73f8a) — EDA + hypothesis testing + project proposal (April 14)
- **`milestone2`** (722b23b) — Complete ML pipeline + final Random Forest model (April 29)
- **`milestone3`** (final) — Final report + website + AI usage documentation (May 18)

---

## Tools & Technologies

- Python 3.10+
- pandas, numpy, matplotlib, seaborn, scipy
- scikit-learn (preprocessing, models, evaluation, permutation importance)
- joblib (model persistence)
- Jupyter Notebook
- Azure DevOps (data source)

---

## AI Usage Disclosure

Per project guidelines, all use of AI tools is documented. See **[AI_USAGE.md](AI_USAGE.md)** for the full disclosure, including categorized prompts and division of contributions between Claude and the author.
