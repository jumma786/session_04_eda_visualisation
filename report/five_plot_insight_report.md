# Five-Plot Insight Report

**Dataset:** `workplace_eda_training_dataset.csv` (720 learners, 16 columns)
**Target:** `completed_training` (base rate 87.5%)

## Data story

The headline question is why some learners finish workplace training and others do not. The five-plot exploration points to a clear, actionable answer: **engagement and friction matter; delivery format and prior skill barely do.**

The **assessment-score distribution** shows completers concentrated between 75 and 95 with a heavy peak at 100, while the smaller non-completer group sits lower and broader, centred around 70-80. The two groups overlap but separate visibly — assessment_score discriminates, though the ceiling spike at 100 hints that scoring may have been recorded after training, raising a leakage concern.

The **completion-rate-by-training-mode** bar chart is, deliberately, a flat one. Self-paced (88.2%), blended (87.2%) and instructor-led (86.9%) deliver almost identical completion rates. This is a useful negative result: the business should not expect to lift completion by changing delivery format alone.

The **correlation heatmap** ranks the candidate features. The strongest correlation with the target is `productivity_gain_estimate_pct` (r = 0.50), followed by `engagement_score` (0.27), `satisfaction_score` (0.22), and `manager_support_score` (0.20). Prior Python score and total course minutes show essentially no signal.

The **engagement-vs-completion** boxplot makes the strongest case for a single pre-completion feature. The completer median is ~80, the non-completer median ~69 — a 10-point gap with limited overlap in the inter-quartile boxes.

The **stakeholder chart** translates this for a learning manager: completion ranges from 82% in Customer Support to 93% in IT. The 11-point gap is large enough to justify targeted intervention rather than a uniform rollout.

**Bottom line for modelling:** engagement and manager-support are the most defensible features; productivity-gain looks tempting but is almost certainly outcome-contaminated.

*(Word count: ~300)*

## Hypotheses for the next modelling sprint

- **H1 — Engagement is predictive.** `engagement_score` shows clear separation between completers and non-completers; expected to be a top feature.
- **H2 — Manager support is predictive.** `manager_support_score` correlates 0.20 with completion and is available before training begins.
- **H3 — High friction signals risk.** A derived `high_friction_flag` (`support_tickets_last_30d >= 3`) captures a non-linear drop from ~89% to 78% completion.
- **Leakage caveat — productivity_gain_estimate_pct, assessment_score, and satisfaction_score** all correlate strongly with the target but are measured at or after course end. Excluded from any pre-completion prediction model.
- **Sample-size caveat —** only 91 learners sit in the 3+ ticket band and just 5 have 5 or more tickets; tail-band estimates are noisy and should be reported with confidence intervals.
