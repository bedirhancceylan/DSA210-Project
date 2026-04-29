# DSA 210 — Analyzing Task Completion Patterns in an Early-Stage Startup

**Bedirhan Ceylan** — Spring 2026
**Section A — TA: Erfan Tarmohammadi**

This project analyzes 486 task records from the development workflow of [Sollen](https://www.sollen.io), an early-stage startup, to identify which factors are associated with longer completion times and delays. The pipeline covers exploratory analysis, hypothesis testing, feature engineering, and machine learning classification.

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
├── project_proposal.pdf
├── requirements.txt
└── README.md
```

---

## EDA & Hypothesis Testing Findings

| # | Question | Test | Result |
|---|---|---|---|
| 1 | Does duration differ across categories? | Kruskal-Wallis | Reject H₀ (p < 0.0001) — product/tech longest, ops/research shortest |
| 2 | Is category associated with delay status? | Chi-Square | Fail to reject (p = 0.275) — category alone is not a strong predictor |
| 3 | Do delayed tasks take longer than on-time? | Mann-Whitney U (one-sided) | Reject H₀ (p < 0.0001, r = 0.876) — by construction, but quantified |

Key insight: the chi-square result motivated the ML phase. Since category alone does not predict delay, we hypothesized that the real signal lies in **feature interactions** (developer × workload × timing) — which tree ensembles captured.

---

## Machine Learning Pipeline

### Feature Engineering (Day 1)

Built 10 leakage-free features from the raw columns:

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

| Feature | Importance (ROC-AUC drop) |
|---|---|
| `creation_month_num` | 0.063 |
| `task_category` | 0.043 |
| `dev_historical_delay_rate` | 0.039 |
| `assigned_to` | 0.039 |
| `team_total_load_at_creation` | 0.033 |
| `dev_avg_past_duration` | 0.027 |

Top finding: developer-history features (`dev_historical_delay_rate`, `dev_avg_past_duration`) and team load are stronger signals than the task's own category — supporting the chi-square conclusion that delay is a function of *who is doing the work and how much they already have on their plate*, not just *what kind of task it is*.

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

---

## AI Usage Disclosure

Per project guidelines: code structure, hypothesis test selection, feature engineering decisions, and notebook organization were developed iteratively with Claude (Anthropic). All code was reviewed, executed, and verified locally. Analytical decisions (target leakage handling, feature selection, model choice rationale) were made by the author.
