# AI Usage Disclosure — DSA 210 Project

**Author:** Bedirhan Ceylan
**Course:** DSA 210 — Introduction to Data Science, Spring 2026
**Section:** A — Teaching Assistant: Erfan Tarmohammadi
**Date:** May 2026

---

## Overview

Per the DSA 210 project guidelines (Academic Integrity section), this document discloses my use of AI tools during this project. The use of AI was permitted but had to be "explicitly documented, including specific prompts used and outputs generated." This file fulfils that requirement.

I used **Claude (Anthropic, model: claude-opus-4-7)** as a coding and writing assistant throughout the project. All conversations were one-on-one. No code or text was used without my own review and verification. Below I describe what Claude was used for, organized by project phase, with representative examples of the prompts I used.

---

## 1. Project Setup Phase (March–April 2026)

### What Claude was used for
- Confirming the project guidelines after reading the official DOCX file.
- Discussing the data source options (using the Sollen Azure DevOps export I have access to vs. a public dataset).
- Checking which TA I was assigned to from the student list spreadsheet.

### Representative prompts
- *"Read the project guidelines and the student list, and tell me what's required and which TA I'm assigned to."*
- *"Is using my own startup's Azure DevOps task data acceptable for this project, or do I need to enrich it with another dataset?"*

### What I did with the output
I wrote the project proposal myself based on Claude's clarifications. The proposal PDF (`project_proposal.pdf` in the repo) is my own writing.

---

## 2. EDA & Hypothesis Testing Phase (April 2026)

### What Claude was used for
- Selecting which hypothesis tests fit the structure of my data (heavy right-skew, mixed categorical/numeric, imbalanced classes).
- Generating the initial scaffolding of the EDA notebook (imports, plot styling, consistent figure sizes).
- Discussing how to handle the heavy right-skew in duration (log transform for visualization, non-parametric tests for inference).

### Representative prompts
- *"My target is binary delayed/on-time and my predictor is task category (5 levels). I want to know if there's an association. Which hypothesis test should I use?"*
- *"Duration is heavily right-skewed (median 21 days, max 343 days). Which test compares duration across 5 categories without assuming normality?"*
- *"Help me set up a Jupyter notebook with consistent matplotlib styling for the EDA section."*

### What I did with the output
I ran every cell myself in Jupyter, verified the output, and interpreted the results. The interpretations of the three tests in the report (especially the chi-square negative result and what it implied for the ML phase) are my own analysis. Claude suggested the test choices; I chose how to present and use them.

---

## 3. ML Pipeline Phase (Late April 2026)

### What Claude was used for
- The single most important contribution: **flagging target leakage**. When I described the feature set I was planning to use, Claude pointed out that `actual_duration_days` and `completion_date` would leak the target (since the target is derived from duration). This led me to drop those columns and to design the historical features (`dev_historical_delay_rate`, `dev_avg_past_duration`) so they only use strictly past data.
- Suggesting the four-day ML notebook structure: feature engineering → baselines → tree ensembles → final model selection.
- Code structure for the scikit-learn `Pipeline` + `ColumnTransformer` setup, ensuring preprocessing happens inside cross-validation folds (preventing CV leakage).
- Generating per-notebook code scaffolds with my chosen models, hyperparameters, and metrics.

### Representative prompts
- *"My delay_label is computed as duration > 1.5 × category median. If I include actual_duration_days as a feature, what happens?"*
- *"How do I compute a 'developer historical delay rate' feature without leaking future information?"*
- *"Set up a stratified 5-fold cross-validation with ColumnTransformer for one-hot + standard-scale, reporting accuracy, precision, recall, F1, and ROC-AUC."*
- *"Compare scikit-learn's RandomForestClassifier vs GradientBoostingClassifier — does GBC accept class_weight?"*
- *"Why is permutation importance more reliable than impurity-based importance for high-cardinality categoricals?"*

### What I did with the output
I ran each notebook end-to-end on my own machine, debugged any environment issues myself (including a `zsh` quote-parsing problem on multi-line pasted commands), and verified that the metrics in the notebooks match the metrics quoted in the report. The model selection (Random Forest as final model) was my decision based on the comparison table; Claude only generated the comparison code.

---

## 4. Documentation & Reporting Phase (May 2026)

### What Claude was used for
- Drafting the README.md structure and content.
- Drafting this final report's structure and content.
- Drafting the project website (`docs/index.html`) for GitHub Pages.
- Suggesting wording improvements throughout (especially for the Limitations and Future Work sections).
- Git workflow guidance: how to create the `milestone1` and `milestone2` tags, how to push without overwriting history, how to write descriptive commit messages.

### Representative prompts
- *"Draft a README that covers project overview, dataset schema, EDA summary, model comparison table, and reproduction instructions."*
- *"Write a final report covering motivation, data source, data analysis, findings, limitations, and future work — academic tone, English, around 10 pages."*
- *"How do I create an annotated git tag and push it without affecting existing tags or commit timestamps?"*

### What I did with the output
I read every paragraph of the generated report, edited it for accuracy (especially numbers — every metric in the report was cross-checked against the actual notebook outputs), and rephrased sections that did not match my voice or the level of the audience. The final claims and interpretations are mine.

---

## 5. What Claude Was NOT Used For

For full transparency, here is what I did entirely without AI:
- All data extraction from Azure DevOps (manual SQL/UI queries).
- Anonymization of developer names.
- All decisions about which results to keep and which to discard.
- All decisions about which features to engineer and which to exclude.
- All interpretation of statistical results and their operational implications.
- All Jupyter cell execution, debugging, and result verification.
- All git operations (commits, tags, pushes) — Claude provided the commands; I ran them.
- The project topic and scope.

---

## Statement of Compliance

I confirm that:
1. All AI assistance is disclosed in this document.
2. The work submitted is my own in terms of decisions, interpretations, and verification.
3. I have personally executed and verified every code cell, every metric, and every claim in the report.
4. No part of the analytical findings was accepted from Claude without independent verification on my own machine.

— Bedirhan Ceylan, May 2026
