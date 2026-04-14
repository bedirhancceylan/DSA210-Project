# DSA 210 - Analyzing Task Completion Patterns in an Early-Stage Startup

## Project Overview
This project analyzes task completion patterns within the development workflow of an early-stage startup currently in the product development phase. The goal is to identify which factors are associated with longer completion times and delays in startup task execution.

## Dataset
The dataset contains **486 task records** collected from the team's Azure DevOps workflow, covering October 2024 to early 2026. Each observation corresponds to a single completed task.

| Variable | Description |
|----------|-------------|
| `task_id` | Unique task identifier (Azure DevOps work item ID) |
| `title` | Task title/description |
| `task_category` | Department (tech, design, research, product, operations) |
| `priority_level` | Task priority (low, medium, high) |
| `assigned_to` | Anonymized developer label (Developer_A through Developer_N) |
| `creation_date` | Date the task was created |
| `completion_date` | Date the task was completed |
| `actual_duration_days` | Time from creation to completion (days) |
| `delay_label` | delayed / on_time (based on 1.5x category median threshold) |
| `has_urgent_tag` | Whether the task was tagged as URGENT (0/1) |

Team members are anonymized with non-identifiable labels (Developer_A, Developer_B, etc.) to protect privacy.

## Project Structure
```
├── README.md
├── requirements.txt
├── data/
│   └── sollen_tasks.csv
├── notebooks/
│   └── eda.ipynb
└── plots/
    └── (generated plots)
```

## How to Run
1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Navigate to the notebooks directory:
   ```bash
   cd notebooks
   ```
3. Launch Jupyter Notebook:
   ```bash
   jupyter notebook eda.ipynb
   ```
4. Run all cells sequentially (Kernel → Restart & Run All).

## Current Progress
- [x] Data collection from Azure DevOps (486 tasks)
- [x] Exploratory Data Analysis (EDA)
- [x] Hypothesis Testing (Kruskal-Wallis, Chi-Square, Mann-Whitney U)
- [ ] Machine Learning models (due May 5)
- [ ] Final report (due May 18)

## Tools & Technologies
- Python 3.10+
- pandas, numpy, matplotlib, seaborn, scipy
- Jupyter Notebook
- Azure DevOps (data source)
