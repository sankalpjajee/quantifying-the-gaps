# EXPERIMENTS.md — ordered execution protocol

**Companion to:** `draft_evaluation.tex` (the empty tables these experiments fill)
**Parent plan:** `REVISION_PLAN.md` (Track B items B1–B12, plus prerequisites P1/P2)
**Target manuscript:** `/home/user/quantifying-the-gaps/AAAI_27_Rodent/AnonymousSubmission2027.tex`

---

## 0. How to read this document

Experiments are ordered by **execution dependency**, not by value. Running them out of
order wastes work: five of the thirteen produce numbers that become invalid the moment
E3 changes the model.

Every entry has the same five fields:

- **Procedure** — what to run, precisely enough to run it.
- **Code changes** — what has to be edited, and what must *not* be.
- **Runtime** — compute time, then realistic wall-clock including plumbing.
- **Objection killed** — the reviewer sentence this makes unwriteable.
- **If the result is unfavourable** — what to do. Read this field *before* running the
  experiment, not after. Several of these results are more likely than not to come back
  worse than the manuscript currently implies, and the difference between a paper that
  survives that and one that does not is entirely whether the authors decided in advance
  how to report it.

### Runtime estimates: where they come from

The manuscript states that training completes in **under four minutes on a single GPU**
(Limitations, item 7). Every compute estimate below is derived from that stated figure by
multiplying out the number of fits. These are *arithmetic projections from a number in the
paper*, not measurements. Verify the per-fit time once, on E3, and rescale everything
here by the observed ratio.

### One rule that overrides everything else

**No number enters the manuscript that was not produced by a run recorded in this
document.** If an experiment does not run, its table cells stay marked and its sentence
does not get written. There is no third option where a plausible value goes in "for now."

---

## 1. Dependency graph

```
E0 (provenance audit)  ── must complete before anything else
  │
  ├─► E1 (per-class, confusion, κ) ──┐
  ├─► E2 (class distribution)      ──┤
  │                                  │
  └─► E3 (drop subject-ID) ── THE GATE
        │                            │
        ├─► E4 (engineered-only)     │
        │     │                      │
        │     ├─► E5 (block split P2)│
        │     ├─► E6 (LOAO P3) ──────┤──► E10 (seeds/CIs/Wilcoxon)
        │     ├─► E7 (baselines)     │
        │     ├─► E8 (calibration)   │
        │     └─► E9 (ablation)      │
        │                            │
        └─► E12 (temporal smoothing) │
                                     │
E11 (SAGER on this cohort) ──────────┘   (independent; start early, long lead time)
E13 (edge benchmark)                     (independent; can run any time)
```

**E1 and E2 are re-run after E3.** They are listed first because they cost almost nothing
and because their output determines whether the manuscript's existing claims can stand
even for one more day. Their *final* versions come from the post-E3 model.

**Start E11 first in wall-clock terms.** It is the only item with third-party integration
risk and it sits off the critical path, so begin it on day 1 and let it run in parallel.

## 2. Budget

| Set | Experiments | Compute | Wall clock | What it buys |
|---|---|---|---|---|
| **Minimum honest** | E0, E1, E2, E3 | ~1 h | 2 days | The manuscript stops asserting things it has not measured. Below this, do not submit anywhere. |
| **Competitive** | + E4, E5, E6, E7, E8 | ~4 h | 5–6 days | A subject-independent number, a real baseline set, and calibration that is not measured on a leaky split. This is the set the parent plan estimates at 10–15% acceptance. |
| **Complete** | + E9, E10, E11, E12, E13 | ~30 h | 3–4 weeks | Ablation, variance, significance, an external published comparator, temporal context, deployment cost. |

If only one week exists, run **Minimum honest + E6 + E7**. E6 is the objection every
reviewer raises; E7 is the one the frozen abstract already promises ("outperforming all
baseline methods") and currently does not support.

---

# E0 — Provenance audit (prerequisite, not an experiment)

Nothing below is a measurement. All of it is reading the code and the data files. It is
listed as E0 because three later experiments are undefined until it is done, and because
two of its four questions determine whether the submitted abstract is accurate.

### Procedure

Answer these, in writing, from the code — not from the figures and not from memory:

1. **Does the feature-extraction code compute any cross-frequency coupling quantity?**
   Search for: modulation index, phase-amplitude, PAC, PLV, comodulogram, `hilbert`,
   `angle(`, `analytic`. Record the answer as yes-with-location or no.
2. **What are the exact columns of the design matrix that produced the reported model?**
   Dump `list(X.columns)` (or the array shape and the column-construction code) and its
   length. Reconcile with the "5,009 features" figure in the manuscript. Note which
   columns are raw signal samples and whether an animal identifier is present.
3. **Which fitted model and which split produced each of the three figures?** The feature
   importance figure and the SHAP figure cannot both describe the same design matrix —
   one contains a single `eeg_data` column and the other contains nine bare integer
   sample indices. Establish which is which.
4. **What `importance_type` does the feature-importance figure use**, and what is its
   normalization denominator? XGBoost exposes `weight`, `gain`, `total_gain`, `cover`,
   and `total_cover`, which rank differently.

Also transcribe, because later sections need them and none can be guessed: Welch
parameters (`nperseg`, `noverlap`, `window`, `nfft`, `detrend`, `scaling`); MMD window
length, stride, and aggregation; whether any filtering, notch, re-referencing, or artifact
rejection was applied; the hyperparameter grid actually searched and the fold unit;
`max_depth` and `objective` of the fitted model; library versions and hardware.

### Code changes

None. Add a `PROVENANCE.md` to the repository recording the answers. This file becomes the
source for the `% REQUIRES_EXPERIMENT` slots in the Method and Experimental Setup sections
that are transcription rather than measurement.

### Runtime

2–4 hours of reading.

### Objection killed

*"The paper never specifies what the model was trained on."* Also, and more urgently: it
determines whether the title and the frozen abstract's "cross-frequency coupling metrics"
describe something that exists.

### If the result is unfavourable

**If question 1 comes back "no CFC is computed":** this is the one problem in the whole
revision that writing cannot fix, because the abstract is frozen and claims it. The
options, in order of preference:

- **(a)** Compute a real CFC feature now and retrain. The Tort modulation index over a
  theta-phase / gamma-amplitude pair is standard, is a few dozen lines, and would make the
  abstract retroactively accurate. This is the only clean outcome. It costs roughly one
  day including feature extraction over the cohort and one retrain, and it should be
  inserted between E0 and E3. Budget for it now rather than deciding later.
- **(b)** Ship with the CFC claim confined to the frozen abstract and the title, delete
  every body claim that repeats it, and add nothing that implies it was computed. The
  exposure is then one sentence in an abstract that cannot be edited. This is survivable
  but it is not honest *enough* — a reviewer comparing abstract to Method will notice.
- **(c)** Do not, under any circumstance, write a CFC formula into the Method section
  describing a computation that was not performed. That converts a submission error into
  fabrication.

**If question 2 confirms an animal identifier and raw sample columns:** proceed to E3,
which exists to fix exactly that. Disclose both in the Method section regardless of
whether E3 runs.

---

# E1 — Per-class metrics, confusion matrix, κ, balanced accuracy

Fills: `tab:perclass`, `tab:confusion`. Parent plan item B3. **Cheapest high-value item in
the entire revision.**

### Procedure

1. Load the saved test-set predictions. If they were not saved, refit the selected
   configuration with the recorded seed (42) and the same split — one fit.
2. Compute, on the P1 test partition:
   - `sklearn.metrics.classification_report(y_true, y_pred, target_names=['Wake','SWS','REM'], digits=4)`
   - `sklearn.metrics.confusion_matrix(y_true, y_pred)` — **raw counts, not normalized**
   - `sklearn.metrics.cohen_kappa_score(y_true, y_pred)`
   - `sklearn.metrics.balanced_accuracy_score(y_true, y_pred)`
3. Record both macro and weighted averages of precision/recall/F1, separately and
   labelled. Do not mix them.
4. **Consistency check, mandatory:** the macro row must reproduce 86.8 / 81.2 / 83.5 and
   accuracy must be 91.5%. If it does not, stop and find out why before writing anything.
   Do not adjust the computation until it matches the abstract.

### Code changes

Add a `report_metrics(y_true, y_pred, y_prob, name)` helper that emits all of the above to
a CSV keyed by protocol and configuration name, and call it from every evaluation path.
Everything downstream (E3–E12) reuses it. Ten lines; write it once and never compute a
metric ad hoc again.

Also: **fix the metric-averaging inconsistency** while you are here. The manuscript reports
the network's accuracy and recall as the same number (83.54%), which is the signature of a
weighted recall for one model and a macro recall for the other. Recompute every cell in the
existing performance table under one convention.

### Runtime

Compute: minutes. Wall clock: half a day, most of it writing the helper.

### Objection killed

*"On a three-class problem where one class is 8% of the data and the entire narrative is
about that class, the authors report only macro aggregates."* Also: a reviewer can already
derive an approximate REM recall from the four published numbers by algebra. Being seen to
have withheld it is far worse than the number itself.

### If the result is unfavourable

It very likely will be. The paper's own four numbers, combined with the stated ~8% REM
prior, imply a REM recall in the neighbourhood of 50–60%. Assume the measured value lands
somewhere in that range.

- **Report it in the table, plainly, with no hedging adjective.**
- **Delete these two sentences today**, before the number is even known, because both are
  claims about this table made before the table existed:
  - Introduction: *"with balanced precision, recall, and F1-scores across states"*
  - Conclusion: *"The robust performance across all vigilance states, particularly
    challenging REM sleep detection, establishes…"*
- **Reframe rather than defend.** A low REM recall on a single-channel EEG scorer with no
  EMG is an expected, explicable result — REM is defined partly by muscle atonia, which
  this recording setup cannot observe. Say that. It converts a bad number into a
  methodological observation about the modality, which is a legitimate finding and which
  the Limitations section already half-states.
- **The frozen abstract survives.** 86.8 / 81.2 / 83.5 are macro averages and remain
  literally true whatever the per-class breakdown shows. Only the body's *interpretation*
  of them was false.
- **Add the corresponding Limitations item** rather than deleting the old one: single-channel
  EEG bounds achievable REM sensitivity, and the confusion matrix shows where the missed
  REM epochs go.

---

# E2 — Class distribution audit

Fills: `tab:classdist`. Part of parent plan item B3.

### Procedure

`df.groupby(['animal','label']).size().unstack()` on the label file. Report counts and
percentages, overall and per animal. Also report the majority-class share, which is the
floor row of `tab:baselines`.

Check while you are there: do the per-animal totals sum to 138,240, and are they equal? An
exactly-uniform 17,280 per animal means no epoch was excluded from any 48-hour recording of
a freely behaving animal, which in turn means no artifact rejection was performed. Whichever
the answer, it must be stated in the Method section.

### Code changes

None beyond a script.

### Runtime

10 minutes.

### Objection killed

*"91.5% accuracy against what baseline rate?"* and *"how much does the class prior vary
between animals?"* — the second of which determines how to read the fold spread in E6.

### If the result is unfavourable

If the majority class alone exceeds ~60%, the headline accuracy is much less impressive
than it reads and the reviewer will compute this themselves. Pre-empt it: report the
majority-class floor in the same table as the model, and shift the emphasis of the Results
section onto κ and macro-F1, both of which are chance-corrected or class-balanced and
neither of which is inflated by the prior. This costs nothing and is strictly more
informative.

If the prior varies a lot across animals, say so — it is the explanation for fold variance
in E6 and it is better introduced before that table than after it.

---

# E3 — Retrain without the subject-identity column — **THE GATE**

Parent plan item B1. Nothing downstream is honest until this is done.

### Procedure

1. Drop the animal-index column from the feature matrix.
2. Retrain the optimized configuration with **identical** hyperparameters (learning rate
   0.1, 500 estimators, subsample 0.8, `colsample_bytree` 0.8, `gamma` 0, `reg_lambda` 1,
   seed 42) on the **identical** epochs and the **identical** P1 partition. Change one
   thing only.
3. Regenerate: all four headline metrics, `tab:perclass`, `tab:confusion`, the feature
   importance figure, the SHAP figure, and the calibration curve.
4. Record **both** configurations. The difference between them is a measurement of how
   much of the reported accuracy was subject memorization, and it is worth reporting as
   such.

### Code changes

One line at the feature-matrix construction site, plus a configuration flag so both
variants can be produced from one script. Keep the flag — E9's ablation harness will
reuse it.

### Runtime

Compute: ~4 minutes per fit, so under 15 minutes including figure regeneration. Wall clock:
half a day.

### Objection killed

*"The model takes subject identity as an input, which is visible in the paper's own
feature-importance figure, so it has undefined behaviour on a ninth animal and the
generalization claim is vacuous."* This objection is currently available to any reviewer
who looks at Figure 3 for ten seconds.

### If the result is unfavourable

This is the one fix that can collide with the frozen abstract. The subject-ID column ranks
above several named features by mean |SHAP|, so a measurable drop is plausible.

- **If the metrics are essentially unchanged** (within rounding of 91.5 / 86.8 / 81.2 /
  83.5): report the corrected model as the headline, note in one sentence that removing the
  subject index did not change the figures, and move on. Best case, and it also disposes of
  the objection permanently.
- **If the metrics drop:** report *both*, labelled by configuration, and state the
  relationship in one sentence in the Results section — something of the form "the
  originally reported configuration included an integer animal index among its inputs and
  achieved 91.5% under P1; with that column removed the same configuration achieves [x]%,
  and all subsequent analyses use the latter model." The abstract remains literally true as
  a statement about a specific configuration, and the paper is visibly the one that found
  and fixed the problem rather than the one that shipped it.
- **What not to do:** do not keep the subject-ID model as the headline because it scores
  higher. Do not quietly relabel. Do not omit the corrected number. A reviewer who works out
  that the reported model conditions on subject identity, and finds no acknowledgement,
  will not accept the paper on any other grounds.
- **If the drop is large** (more than a few points): that is itself the paper's most
  interesting early result, and it feeds directly into the E5/E6 framing — the model was
  substantially identifying animals rather than states, which is precisely what the
  protocol ladder is designed to expose.

---

# E4 — Engineered-features-only model

Parent plan item B4. Run only if E0 confirms raw sample columns in the design matrix.

### Procedure

1. Build the design matrix from the engineered descriptors alone — band powers, relative
   powers, peak frequencies, spectral entropy, Hjorth activity/mobility/complexity, MMD,
   and CFC if it exists. No raw sample columns, no subject index.
2. Train the same configuration on it. This is the model the paper claims to be about, and
   at present it is evaluated nowhere.
3. Retrain the feed-forward network on the **identical** vector so the model comparison is
   finally matched.

### Code changes

A column-selection step at matrix construction, driven by the same flag introduced in E3.

### Runtime

Compute: ~10 minutes for both fits (the engineered matrix is far smaller than 5,009
columns, so expect faster than 4 minutes each). Wall clock: 1 day.

### Objection killed

Two at once. *"The contribution is claimed to be a compact physiologically grounded
feature basis, but the reported model consumed five thousand raw signal samples, so the
claimed contribution is never evaluated."* And: *"The XGBoost-versus-network comparison
varies the inputs and the model simultaneously and therefore measures neither."*

### If the result is unfavourable

If the engineered-only model scores materially below the raw-inclusive one, the paper's
framing has to change but the paper does not die:

- The honest headline becomes the engineered-only model, because that is the one the
  contribution is about and the one that is interpretable, small, and deployable. Report
  the raw-inclusive figure alongside it as an upper reference.
- Then say what the gap means: it quantifies how much of the signal is *not* captured by
  the canonical feature basis, which is a real and citable observation about the adequacy
  of standard spectral descriptors for this task, and one no cited scorer reports.
- If the drop is severe, the accuracy-versus-interpretability trade-off becomes the
  paper's actual subject. That is a better paper than "XGBoost gets 91.5%," and it is
  fully consistent with the frozen abstract, which claims a model built on engineered
  features and does not claim the model contained nothing else.

---

# E5 — Contiguous-block split (protocol P2)

Fills the P2 row of `tab:ladder`. Parent plan item B9.

### Procedure

1. Choose the block unit and fix it: whole light/dark phases (preferable — it also breaks
   the circadian confound) or whole hours (more blocks, finer granularity). Pick one, state
   it, use it everywhere.
2. Assign whole blocks to train or test, per animal, holding out ~20% of blocks.
3. Train and evaluate exactly as in P1. Report the same metric set.

### Code changes

A block-assignment function replacing the stratified epoch shuffle. It needs the epoch
timestamps or the within-recording epoch index, which must therefore be preserved through
feature extraction — check that it is; a pipeline that shuffles before extraction has
already destroyed the information needed here.

### Runtime

Compute: ~4 minutes. Wall clock: half a day, mostly recovering epoch ordering if it was
not retained.

### Objection killed

*"Temporally adjacent epochs from the same sleep bout appear on both sides of the split,
so the test epochs are near-duplicates of training epochs."*

### If the result is unfavourable

P2 below P1 is the *expected* outcome and is the point of running it — the gap is the
measurement. Report it as such. The only genuinely awkward case is P2 ≈ P3 ≪ P1, which
would mean temporal adjacency, not subject identity, was doing the work; that is still a
reportable and interesting decomposition, and it is more informative than either number
alone. There is no outcome of E5 that is bad news, only outcomes that change which sentence
gets written.

---

# E6 — Leave-one-animal-out cross-validation (protocol P3)

Fills `tab:loao` and the P3 row of `tab:ladder`. Parent plan item B2. **This is the
experiment most likely to return a number below 91.5%, and the one the authors most need a
plan for.**

### Procedure

1. `GroupKFold(n_splits=8)` grouped on the animal identifier — or simply loop over the
   eight animals, holding each out in turn.
2. **The subject-identity column must already be removed** (E3). Under P3 the held-out
   animal's identifier never appears in training, so the model has no fitted behaviour for
   it; leaving the column in makes the fold result uninterpretable rather than merely
   pessimistic.
3. For each fold: refit from scratch on the seven training animals with unchanged
   hyperparameters and seed; evaluate on the held-out animal.
4. Report **per fold**: n_test, accuracy, macro-F1, κ, REM recall. Then mean, standard
   deviation, and **minimum** across folds.
5. Do **not** re-tune hyperparameters per fold and then report those folds. If
   hyperparameters are re-selected, do it in a nested inner loop over the seven training
   animals only.

### Code changes

The grouping and the loop. If the feature pipeline currently fits any transform (scaler,
imputer, encoder) on the full dataset, that fit must move inside the fold — otherwise P3
leaks and its whole purpose is defeated. Check this specifically; it is the most common way
a correctly-designed LOAO is silently invalidated.

### Runtime

Compute: 8 fits ≈ 35 minutes at the stated per-fit time. Wall clock: half a day to one day.

### Objection killed

*"There is no subject-independent performance estimate anywhere in this paper."* Every
comparable accepted evaluation in this space partitions by subject or by dataset. No
published rodent scorer that this paper positions itself against omits it.

### If the result is unfavourable

**Expect it to be.** Plan for the drop now, in writing, before the number exists. Decide
the reporting sentence in advance and let the measurement fill in the value.

**Scenario A — LOAO within ~2 points of P1.** Unexpected and excellent. Report it, and note
that the small gap indicates the feature basis transfers across animals in this cohort.
Resist over-claiming from n=8: eight animals from one colony is not evidence of
cross-laboratory transfer, and the Limitations section must continue to say so.

**Scenario B — LOAO 5–15 points below P1.** The most likely outcome, and the one the parent
plan identifies as the paper's best available opportunity. Do this:

1. Make the protocol ladder (`tab:ladder`) the central result of the Results section, not a
   robustness check buried at the end.
2. Reframe the paper's contribution around the gap: *how much subject-dependent evaluation
   overstates single-channel rodent vigilance scoring, measured on the same cohort with
   everything else held fixed.* This is transferable to every other group doing this work,
   which "our model gets 91.5%" is not.
3. The frozen abstract survives intact. 91.5% remains true as the P1 figure; the abstract
   never states a protocol. Label it as the P1 figure everywhere in the body.
4. Delete the generalization claims the manuscript currently makes and its own Limitations
   section already retracts — do not soften them, delete them.
5. Note that this reframing is the structure of the accepted work in this area: the
   contribution of generalizable-sleep-staging papers is the *task*, not a better
   classifier.

**Scenario C — LOAO more than ~20 points below P1, or individual folds collapsing (REM
recall near zero on some animals).** Still report it, unchanged. Then spend the remaining
effort turning it into a finding rather than leaving it as a failure. Three cheap follow-ups,
in order of value:

1. **Diagnose the shift.** Correlate per-fold accuracy against per-animal properties: class
   prior (from E2), signal amplitude scale, band-power distribution, electrode impedance if
   recorded. If a single property explains most of the fold variance, that is a concrete,
   citable statement about what changes between animals in single-channel rodent EEG.
2. **Test per-animal normalization.** Z-score or quantile-normalize the features within each
   animal before training. If this recovers most of the gap, it is a genuinely useful result
   with a one-line implementation, and it becomes a contribution in its own right — the
   mixture z-scoring idea has precedent in this literature, so it is a natural thing to
   evaluate and a natural thing to cite.
3. **Measure the adaptation curve.** For each held-out animal, add the first N minutes of
   its own labelled epochs to training, for N in {5, 15, 30, 60}, and plot recovery. The
   deliverable — "a new animal needs roughly N minutes of manual scoring before the scorer
   is usable on it" — is the single most practically valuable number this paper could
   produce for a working sleep lab, and it exists only because the LOAO number was bad.

**What not to do, under any scenario:**

- Do not report only the mean across folds. Report the spread and the worst fold; with
  n=8 the mean alone is not a summary.
- Do not drop a bad fold. If one animal is an outlier, report it as an outlier and
  investigate why in the Discussion.
- Do not substitute 4-fold or 2-fold grouped CV to shrink the variance and present it as
  subject-independent evaluation. A reviewer will ask why the fold count is not eight.
- Do not present P1 as the headline and P3 in an appendix. If the two are reported at
  different prominence, the reader concludes the weaker one was hidden — and they will be
  right.
- Do not add the word "significantly" anywhere until E10 has run.

---

# E7 — Repair the baseline set

Fills `tab:baselines` (all rows except SAGER). Parent plan item B7.

### Procedure

1. **Majority-class floor.** Predict the most frequent state everywhere; report the same
   metric set. Follows directly from E2.
2. **Multinomial logistic regression.** Same design matrix, same partition, same
   standardization (fitted on training only). Record regularization strength, solver,
   maximum iterations, and whether the solver converged.
3. **Feed-forward network.** Retrained on the identical feature vector (from E4).
4. Report every row under the same protocol, with the protocol named in the caption.

### Code changes

A `run_baseline(model, X, y, split)` wrapper calling the E1 metric helper. All three
baselines go through it so no row can differ in convention from another.

### Runtime

Compute: under 30 minutes total. Wall clock: 1 day.

### Objection killed

*"The abstract claims the method outperforms all baseline methods; the body compares
against two models the authors wrote themselves, one of which appears at 54.9% only in the
Conclusion and nowhere else."* And: *"Accuracy is reported without a trivial floor on a
class-imbalanced problem."*

### If the result is unfavourable

- **If logistic regression reproduces near 54.9%:** investigate before reporting it. On a
  three-class problem whose majority class alone plausibly exceeds 50%, 54.9% is close to
  the trivial rate, and a published logistic-regression baseline on a comparable rodent
  task reaches 93.3%. A baseline that weak reads as broken, not as evidence, and reviewers
  penalize strawman baselines heavily. Check for unconverged solver, unscaled features, or
  raw sample columns swamping the design matrix. Fix and re-report.
- **If logistic regression scores close to the boosted ensemble** (entirely possible on
  well-separated engineered spectral features): report it, and adjust the paper's framing
  rather than the number. The honest contribution then rests on interpretability,
  calibration, and the protocol analysis rather than on accuracy — which is where the
  parent plan already recommends putting it, and which is fully consistent with the frozen
  abstract's "outperforming all baseline methods" as long as the ensemble is in fact ahead
  by some margin. If it is *not* ahead, that sentence in the abstract becomes a problem
  and must be addressed explicitly in the body rather than left to be discovered.
- **If the network beats the ensemble on the matched feature vector:** report that too. It
  is a legitimate result and the paper's engineered-feature argument is unaffected — the
  claim is about the representation, not about which learner consumes it.

---

# E8 — Calibration under every protocol

Fills `tab:calib`. Parent plan item B5.

### Procedure

1. Ensure the model emits probabilities. **The manuscript specifies `multi:softmax`, which
   returns hard class labels** — a reliability diagram, an ECE, and a Brier score cannot be
   computed from those. Either the reported objective is wrong or the calibration figure
   came from a different model. Resolve this in E0 and use `multi:softprob`.
2. For each of P1, P2, P3: compute one-vs-rest reliability curves, ECE, MCE, per-class and
   multiclass Brier score. State the bin count and the binning scheme, and state whether
   the analysis is one-vs-rest or top-label.
3. Regenerate the reliability figure with the class legend relabelled Wake/SWS/REM rather
   than 0/1/2, and state the label-encoder mapping in the Method section.

### Code changes

Objective change plus a `calibration_report()` function. `sklearn.calibration.calibration_curve`
plus a hand-rolled ECE.

### Runtime

Compute: ~1 hour including refits under all protocols. Wall clock: 1 day.

### Objection killed

*"The text claims maximum deviation from the diagonal never exceeds 3 percentage points;
the figure printed directly beneath that sentence shows deviations several times larger for
one class."* This is currently the most immediately checkable false statement in the
manuscript — it is refuted by the paper's own figure.

### If the result is unfavourable

- **The current sentence must be deleted regardless of what E8 returns**, because it is
  already contradicted by the figure in the submitted PDF. Delete it today; do not wait for
  the measurement.
- **If calibration is poor under P1:** report the ECE and name the miscalibrated class.
  Then either apply a standard recalibration (temperature scaling or isotonic regression,
  fitted on a held-out calibration split, never on the test set) and report both, or state
  plainly that the raw scores are not suitable for thresholding. Both are useful; the
  second is honest and costs nothing.
- **If calibration holds under P1 but degrades under P3:** this is the most likely outcome
  and it is the *strongest* result in this experiment, not the weakest. "Confidence scores
  from a rodent scorer are well calibrated within a cohort and degrade on unseen animals"
  is a measured, transferable warning about closed-loop protocols that threshold on those
  scores. No cited rodent scorer reports it either way. Lead with it.
- **Before writing the claim that no published rodent scorer reports calibration**, check it
  against each system individually. It is a strong negative claim, it is being promoted to
  the paper's central contribution, and it has to survive one search by one reviewer.

---

# E9 — Feature-family ablation

Fills `tab:ablation`. Parent plan item B6.

### Procedure

Two blocks, both under P3 on all eight folds, with at least five seeds per cell:

- **Cumulative addition:** spectral only → + Hjorth → + MMD → + CFC (full).
- **Leave-one-family-out from the full set:** full − spectral, full − Hjorth, full − MMD,
  full − CFC.

Both are needed. The families are strongly collinear — the Hjorth descriptors are functions
of the second and fourth spectral moments, and an amplitude-range statistic covaries with
low-frequency power — so a family can look essential in one block and redundant in the
other. That divergence is itself a result worth reporting.

Also run the "standard XGBoost configuration" with default hyperparameters that the Method
section promises and never reports.

### Code changes

A feature-group registry mapping each descriptor to a family, plus a variant loop. Reuses
the E3/E4 column-selection flag.

### Runtime

Compute: 8 variants × 8 folds × 5 seeds = 320 fits ≈ 21 hours at the stated per-fit time —
though the ablated matrices are much smaller than the full one, so expect faster. Parallelize
across folds. Wall clock: 1–2 days.

### Objection killed

*"Nothing is removed and re-measured anywhere in this paper. The only evidence offered for
the feature contributions is a gain-based importance score, which is a property of one
fitted model and is known to distribute credit arbitrarily among correlated predictors."*

### If the result is unfavourable

- **If MMD contributes nothing** (full − MMD indistinguishable from full): **report it.**
  This is the case the authors most need to have decided about in advance. The frozen
  abstract lists MMD as a feature that was used, which remains true whatever the ablation
  shows; the abstract does not claim MMD was important, and it does not claim the feature
  is novel. A measured negative result on one feature is entirely publishable and is far
  less damaging than a reviewer inferring that the feature was never tested. **Delete the
  Conclusion's existing claim that MMD "successfully captured unique temporal dynamics…
  enhancing discrimination" now**, before the ablation runs — it is a claim about a table
  that does not yet exist.
- **If spectral-only nearly matches the full set:** that is a clean, quotable result about
  how far canonical band powers alone carry this task, and it strengthens rather than
  weakens the paper's compactness argument.
- **If the two blocks disagree** (a family looks essential cumulatively and redundant in
  leave-one-out, or vice versa): report both and attribute the difference to collinearity
  in one sentence. Do not report only the block that looks better; a reviewer who runs the
  other one will find it.
- **If the CFC arm cannot be run because no CFC feature exists:** delete those two rows
  rather than filling them, and handle the abstract discrepancy under E0. Do not construct a
  coupling feature after the fact and present it as the one the abstract described.

---

# E10 — Seeds, confidence intervals, significance

Parent plan item B8. Adds ± columns throughout and one significance statement.

### Procedure

1. At least five seeds per model per protocol; report mean ± standard deviation on every
   metric.
2. **Bootstrap confidence intervals by resampling held-out animals, not epochs.** Epochs
   within a recording are strongly autocorrelated; bootstrapping over them yields intervals
   that are meaninglessly tight and would be a worse error than reporting no interval at all.
3. Wilcoxon signed-rank across the eight paired LOAO folds, for the ensemble against the
   network and against logistic regression. Report the statistic and the p-value.

### Code changes

Seed loop and an aggregation step over the per-fold CSVs from E1's helper.

### Runtime

Compute: absorbed into E6 and E9 if the seed loop is added there rather than run separately.
Wall clock: half a day for the analysis.

### Objection killed

*"The Results section says the model 'significantly' outperforms the alternatives; the
submitted reproducibility checklist answers 'no' to number of runs, to any measure of
variation, and to statistical significance testing."* Reviewers read the checklist against
the paper.

### If the result is unfavourable

- **If the difference is not significant:** delete the word "significantly" — which should
  be deleted immediately regardless, since no test has been run — and report the p-value
  anyway. With eight folds, a non-significant result is a statement about the power of an
  eight-animal cohort, not about the method. Say that.
- **If the fold variance is large:** report it. A wide interval on n=8 is the truth about
  this cohort, and stating it is what makes the Limitations section's existing claim about
  "wider confidence intervals" for REM defensible rather than asserted — at present that
  sentence claims an interval nothing in the paper computes.

---

# E11 — Run SAGER on this cohort

Fills the SAGER row of `tab:baselines`. Parent plan item B11. **Start this on day 1** — it
is off the critical path and carries the most integration risk.

### Procedure

1. Obtain SAGER (Saevskiy et al., *Sensors* 25(3):921, 2025; open source at
   `github.com/sykesva/SAGER`). It is a Python single-channel rodent scorer using artifact
   processing, multi-band spectral analysis, and Gaussian-mixture clustering.
2. Convert these eight recordings into its expected input format.
3. Run it under the same protocols where applicable and score its output against the same
   reference labels with the same E1 metric helper.
4. Note that SAGER's published headline figures are **sleep–wake** detection accuracies, not
   three-state accuracies. Do not compare against those; compare against what it produces on
   this data.

Systems requiring paired EMG — AccuSleep and IntelliSleepScorer — cannot be run on this
cohort at all. State that explicitly rather than omitting them silently; it explains why the
external comparison set has one member.

### Code changes

Format conversion and a label-alignment step (its state naming and epoch length may differ
from this pipeline's).

### Runtime

Compute: low. Wall clock: 2–3 days, dominated by integration.

### Objection killed

*"The entire baseline set consists of two models the authors wrote themselves."* A head-to-head
against an independently developed, independently published, openly available method on the
same epochs is worth more than any additional in-house ablation.

### If the result is unfavourable

- **If SAGER outperforms the proposed model on this cohort:** report it. That is a real
  finding, it is what running the experiment was for, and the paper's contribution can rest
  on interpretability, calibration, and the protocol analysis — none of which SAGER
  provides. What cannot survive is a reviewer discovering that the obvious comparison was
  available, open-source, applicable, and not run.
- **If integration fails:** say so, in the paper, in one sentence naming the specific
  obstacle. An honest "we could not run it because X" is publishable. A number transcribed
  from SAGER's own paper and placed in a table of results on this cohort is not — the
  cohorts, protocols, label sets, and in one case the task definition all differ.
- **Report no comparison number that was not produced by actually running the method on
  this data.**

---

# E12 — Temporal smoothing

Parent plan item B10. Adds two rows to `tab:baselines`.

### Procedure

Apply an HMM over the predicted state sequence, or a minimum-bout-duration / median-filter
rule, to the per-epoch predictions. Report with and without, as an ablation, under every
protocol. Requires epoch ordering to have been preserved (see E5).

### Code changes

A post-processing function over the predicted sequence. If an HMM is used, fit the
transition matrix on training animals only, never on the test animal.

### Runtime

Compute: minutes (post-processing only, no refit). Wall clock: 1 day, mostly recovering
epoch ordering and choosing the smoothing parameter honestly — on training folds, not on
test.

### Objection killed

*"The classifier treats epochs as i.i.d. and the paper never mentions that sleep states are
temporally structured. Essentially every serious published rodent scorer applies temporal
context."* It is simultaneously a missing baseline and the cheapest available accuracy gain.

### If the result is unfavourable

If smoothing does not help, that is informative on its own — it suggests the errors are not
isolated single-epoch flips but structured confusions, which the confusion matrix from E1
can corroborate. Report it as a negative ablation. If smoothing helps a lot, report the
gain and state clearly that the smoothing parameter was selected on training folds; a gain
obtained by tuning a smoother on the test set is worse than no gain.

---

# E13 — Edge inference benchmark

Fills `tab:compute`. Parent plan item B12.

### Procedure

Measure per-epoch wall-clock time for **feature extraction** and **model inference
separately**, on a workstation CPU and on an embedded-class device. Report single-epoch
latency and batch throughput. Include feature extraction — for this pipeline the spectral
transform likely dominates, and reporting tree-traversal time alone would overstate the
deployability advantage.

### Code changes

Timing instrumentation; a serialized model and extraction script that run on the target
device.

### Runtime

Compute: minutes. Wall clock: half a day, plus device access.

### Objection killed

Converts Limitations item 7 — which currently concedes that inference latency on edge
hardware is unmeasured — into a measured result. Deployability is the only axis on which a
few hundred shallow trees beat a residual CNN, so it is the axis worth measuring.

### If the result is unfavourable

If the pipeline cannot run in real time on the target device, report the numbers and state
the constraint. A measured "this does not yet run in real time on a Raspberry-Pi-class
device, and feature extraction rather than inference is the bottleneck" is a useful
engineering finding and points at the obvious fix. An unmeasured claim of edge-readiness is
not defensible and the current Limitations section already declines to make it.

---

## 3. The reporting contract — agree to this before running anything

The purpose of this section is to remove the temptation that appears the moment a number
comes back worse than hoped. Sign off on these five commitments **now**, while none of the
results exist.

1. **Every experiment that is run gets reported**, in the paper, whatever it returns. No
   experiment is "exploratory" retroactively.
2. **The protocol is chosen before the result is seen.** Fold count, block unit, seed count,
   metric-averaging convention, and bin count are fixed in advance and written down. They
   are not adjusted after seeing a number.
3. **The strictest protocol gets equal prominence.** P3 appears in the same table as P1, in
   the main body, at the same font size. If a number is in an appendix and its more
   flattering counterpart is not, the reader is entitled to conclude it was hidden.
4. **Deletions happen first.** The seven claims listed below are unmeasured assertions about
   tables that do not yet exist. They come out today, before any experiment runs, so that no
   result is ever written to match a sentence that was already there:
   - "allowing the model to generalize across individual animals and diverse recording conditions"
   - "with balanced precision, recall, and F1-scores across states"
   - "yielding a state-of-the-art 91.5% accuracy"
   - "significantly outperforming the other models"
   - "the maximum deviation from the diagonal never exceeds 3 percentage points"
   - "The MMD feature successfully captured unique temporal dynamics… enhancing discrimination"
   - "The robust performance across all vigilance states, particularly challenging REM sleep detection, establishes…"
5. **The reproducibility checklist is re-answered after the experiments, not before**, and an
   unsupported "yes" is treated as worse than an honest "no". Reviewers cross-check it
   against the paper.

## 4. What the abstract does and does not constrain

Worth having in front of you while writing up, because it is less restrictive than it feels:

- **Constrained:** the four numbers 91.5 / 86.8 / 81.2 / 83.5 must remain attributable to
  *some* configuration and protocol reported in the paper. Label them as the P1 figures for
  the originally submitted configuration and they stay true.
- **Constrained:** the paper must use XGBoost, three states, spectral band power, MMD, and
  cross-frequency coupling metrics, and must release code.
- **Not constrained:** which protocol is the headline. The abstract states no protocol.
- **Not constrained:** the per-class breakdown. Macro averages remain true whatever it shows.
- **Not constrained:** whether MMD turned out to matter. The abstract says it was used, not
  that it was decisive, and it does not claim the feature is new.
- **Not constrained:** "outperforming all baseline methods" is scoped to *this paper's own*
  baselines. It is not a state-of-the-art claim and must not be written as one anywhere in
  the body.
- **The one real collision:** cross-frequency coupling, if E0 finds none in the code. See E0.
