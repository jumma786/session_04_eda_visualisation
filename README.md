# Session 04 — Exploratory Data Analysis and Visualisation

[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/notebook-jupyter-orange.svg)](https://jupyter.org/)

Exploratory data analysis on a workplace training dataset to surface modelling hypotheses for predicting course completion, and to communicate findings to a non-technical audience.

## Objective

Use EDA to generate evidence-based modelling hypotheses for predicting `completed_training`, and to deliver a stakeholder-ready story for a learning manager.

## Dataset

`workplace_eda_training_dataset.csv` — 720 learner records, 16 columns, no missing values. Each row is one learner enrolled on an internal training course.

- **Target:** `completed_training` (1 = completed, 0 = did not)
- **Base completion rate:** 87.5% (heavily imbalanced — accuracy alone is a poor model metric)
- **Features cover:**
  - Learner background — `region`, `department`, `role_family`, `experience_band`, `prior_python_score`
  - Course delivery — `training_mode`, `course_minutes`, `completion_days`
  - In-course signals — `engagement_score`, `manager_support_score`, `support_tickets_last_30d`
  - Outcome-adjacent (leakage candidates) — `assessment_score`, `satisfaction_score`, `productivity_gain_estimate_pct`

## Key findings

1. **Engagement is the strongest legitimate pre-completion signal.** Completers have a median engagement score of ~80 versus ~69 for non-completers — a clear 11-point gap with limited inter-quartile overlap.
2. **Department matters; delivery mode does not.** Completion ranges from 81.8% (Customer Support) to 92.7% (IT) — an ~11 percentage point gap. Training mode (self-paced / blended / instructor-led) is essentially flat at ~87–88%, with only 1.3 pp spread.
3. **Support tickets are non-linear.** Completion sits at ~89% for 0–2 tickets, then drops to 78% at 3+. A linear correlation (-0.08) hides this — the risk is concentrated at the tail.

## Modelling hypotheses

- **Feature likely useful:** `engagement_score` — strong, monotonic relationship with the target, plausibly measurable mid-course.
- **Feature likely useful:** `manager_support_score` — modest correlation (0.20), captured before course start, reflects a real business lever.
- **Feature likely useful:** a derived `high_friction_flag` (`support_tickets_last_30d >= 3`) — converts the non-linear pattern into a clean binary signal.
- **Potential leakage risk:** `productivity_gain_estimate_pct` — has the highest correlation with the target (0.50), but productivity *gain* can only be measured *after* training. The same caution applies to `assessment_score` and `satisfaction_score` if they are recorded post-course.

## Charts

| # | Chart | Question it answers |
|---|---|---|
| 1 | [Assessment score distribution](charts/distribution_assessment_score.png) | Are assessment scores separated by completion status? |
| 2 | [Completion rate by training mode](charts/completion_by_training_mode.png) | Does delivery format affect completion? |
| 3 | [Correlation heatmap](charts/correlation_heatmap.png) | Which numeric features move together, and which correlate with the target? |
| 4 | [Engagement by completion status](charts/engagement_vs_completion.png) | Is engagement different for completers vs non-completers? |
| 5 | [Completion rate by department](charts/stakeholder_summary_chart.png) | Where should a learning manager focus intervention? |

The full 300-word data story is in [[`reports/five_plot_insight_report.md`](reports/five_plot_insight_report.md).](https://github.com/jumma786/session_04_eda_visualisation/blob/main/report/five_plot_insight_report.md)

## Repository layout

```
session_04_eda_visualisation/
├── README.md
├── data/
│   └── workplace_eda_training_dataset.csv
├── notebooks/
│   └── Session_04_Practical_Activity.ipynb
├── charts/
│   ├── distribution_assessment_score.png
│   ├── completion_by_training_mode.png
│   ├── correlation_heatmap.png
│   ├── engagement_vs_completion.png
│   └── stakeholder_summary_chart.png
└── reports/
    └── five_plot_insight_report.md
```

## How to run

**Requirements:** Python 3.10+ and Jupyter.

```bash
# 1. Clone
git clone https://github.com/YOUR-USERNAME/session_04_eda_visualisation.git
cd session_04_eda_visualisation

# 2. (Optional but recommended) create a virtual environment
python -m venv .venv
source .venv/bin/activate          # macOS / Linux
# .venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install pandas matplotlib seaborn jupyter

# 4. Launch the notebook
jupyter notebook notebooks/Session_04_Practical_Activity.ipynb
```

Run all cells from top to bottom. The notebook reads the CSV from `../data/` and writes regenerated charts to a local `notebooks/charts/` folder; pre-rendered annotated copies live in `charts/` at the repo root.

## Method

The notebook follows a consistent EDA pattern across every plot:

1. **Ask a question** — every chart answers a specific analytical question, not just decorates the notebook.
2. **Summarise** — group, aggregate, or describe the data needed to answer the question.
3. **Visualise** — pick a chart type that matches the question (histogram for distributions, bar for categorical comparisons, heatmap for correlations, boxplot for feature-target relationships).
4. **Annotate** — every chart carries a written observation plus a modelling implication.
5. **Caveat** — flag any leakage risk, sample-size limit, or bias concern.

## Caveats

- **Class imbalance** (87.5% / 12.5%) — accuracy will be a misleading model metric. Use precision-recall, F1, or ROC-AUC.
- **Small tail samples** — only 91 learners have 3+ support tickets, and just 5 have 5+. Tail-band estimates carry wide confidence intervals.
- **Timing leakage** — three high-correlation features (`productivity_gain_estimate_pct`, `assessment_score`, `satisfaction_score`) are likely measured at or after course end and should be excluded from a pre-completion prediction model.
- **Single-snapshot data** — there is no temporal split, so a held-out test set will leak information unless time-based validation is used.
