# Revision Plan — AAAI-27 Rodent EEG Vigilance State Classification

**Target file:** `/home/user/quantifying-the-gaps/AAAI_27_Rodent/AnonymousSubmission2027.tex`
**Bibliography:** `/home/user/quantifying-the-gaps/AAAI_27_Rodent/references.bib`
**Constraint:** the abstract (line 103) is frozen. Every change below is checked against it in §6.
**Plan date:** 2026-07-26

---

## 0. Read this first — three things that are not ordinary paper weaknesses

Before the prioritized list, three findings need to be stated separately because they are not
"the reviewers might object" problems. They are problems with the manuscript's factual record.

### 0.1 Two citations describe things that do not exist

- **`henderson2025_sleepinvestigator`** (line 135). The paper says SleepInvestigatoR "couples
  Hidden-Markov smoothing with a modest MLP and requires only 30 min of labeled data for
  adaptation." The real work — **Gamble, Mackenzie C.; Williams, Benjamin R.; McKenna, James T.;
  Logan, Ryan W., "SleepInvestigatoR: a flexible R function for analyzing scored sleep in
  rodents," *SLEEP Advances* 6(2):zpaf032, 2025** (verified: academic.oup.com/sleepadvances/
  article/6/2/zpaf032/8138399; PubMed 38659801; PMC12146841) — is an R function that ingests
  **already-scored** sleep files and computes ~22–25 summary measures of sleep architecture. It
  has no HMM, no MLP, no classifier, and no adaptation procedure. The sentence is an invented
  capability attributed to a named published tool.
- **`garcia2021_dlmouse`** (line 132). The bib entry pairs an invented author ("García, F. and
  others") and an invented title with a **real DOI that belongs to a different paper**. I resolved
  `10.1016/j.jneumeth.2021.109324` myself: it is *"Topological signal processing and inference of
  event-related potential response"* (Wang, Behroozmand, Johnson, Bonilha, Fridriksson,
  *J. Neurosci. Methods* 363:109324, 2021) — human ERP topology, nothing to do with mice, sleep,
  CNNs, or LSTMs.

These are integrity issues, not competitiveness issues. They must be fixed before this manuscript
is sent anywhere, independently of whether the authors care about AAAI-27. A single reviewer
resolving one DOI finds the second one in under a minute.

### 0.2 The paper's own figures show two model inputs the text never mentions

I opened `images/feature_importance.png` and `images/Shap.png` directly. Both confirm:

1. **`animal` is a predictor.** It is ranked 9th of 12 in Fig. 3 and 7th of 11 named features in
   Fig. 4 — above `beta_power`, `activity`, `entropy`, and above the paper's own headline MMD
   feature. Subject identity is in the design matrix. Nothing in the Methodology says so.
2. **Raw time-domain samples are predictors.** Fig. 4 plots nine columns named with bare integers
   (2626, 2929, 2137, 5, 2044, 4841, 1959, 1968, 2627) — all in [0, 4999], i.e. sample indices of
   the 5,000-point epoch. This matches the "5,009 features" at line 201.
3. **The two figures were computed on different design matrices.** Fig. 3 contains a single column
   named `eeg_data`; Fig. 4 does not, and instead expands raw samples. They cannot both describe
   the model that produced 91.5%.

There is a further inference the critics did not draw, and the authors should check it against
their arrays before they do anything else. XGBoost's `feature_importances_` is normalized to sum
to 1.0 across **all** features. Measuring Fig. 3's bars against its own axis ticks, the twelve
plotted features sum to roughly **0.026**. If the plotted quantity is that normalized importance,
then ~97% of the model's total gain sits in columns that are not plotted — i.e. in the raw
samples. That would mean the paper's entire "physiologically grounded feature basis" framing
describes ~2.6% of the model. *Do not write this number into the paper.* It is a measurement off a
PNG, and the renormalization convention is unstated. But it is the first thing to verify in the
code, because if it is right, the paper's contribution claim is inverted, and if it is wrong the
paper needs to say what convention Fig. 3 uses.

### 0.3 The paper's own four numbers imply REM recall of roughly 54%

Line 248 defines recall as macro-averaged. Accuracy is exactly the prior-weighted mean of per-class
recalls. With accuracy = 0.915, macro recall = 0.812, and REM prior ≈ 0.08 (Limitations item 2):

```
(R_wake + R_sws + R_rem)/3 = 0.812
0.92·R_maj + 0.08·R_rem     = 0.915     (R_wake = R_sws = R_maj)
  →  R_maj = 94.8%,  R_rem = 54.1%
```

Relaxing the equal-majority assumption barely moves it. This is algebra on numbers already in the
paper, not a new measurement — but **it must not be written into the paper as a result.** It must
be measured (Track B3). What it does tell you now, for free, is that two body claims are almost
certainly false and must be deleted today: "balanced precision, recall, and F1-scores across
states" (line 117) and "robust performance across all vigilance states, particularly challenging
REM sleep detection" (line 379).

---

## 1. De-duplication: 86 weaknesses → 18 distinct defects

| # | Defect | Merged from | Severity | Track |
|---|---|---|---|---|
| D1 | Subject ID (`animal`) is a model input, undisclosed | W17, W38 | fatal | A (disclose) + B (retrain) |
| D2 | Cross-frequency coupling claimed in title/abstract/intro/related work, computed nowhere | W2, W18, W65, W80 | fatal | **see §6.1 — may be unfixable** |
| D3 | Epoch-wise stratified split; no subject-independent estimate; three generalization claims asserted and retracted by own Limitations | W9, W41, W52, W77 | fatal | A (disclose+delete) + B (LOAO) |
| D4 | No per-class metrics, no confusion matrix, no kappa; own numbers imply REM recall ≈54% | W40, W31, W47 | fatal | A (delete claims) + B (emit) |
| D5 | Six broken/fabricated domain citations; two fabricated descriptions | W55, W56, W57, W59, W66, W67 | fatal | **A** |
| D6 | Nearest prior art uncited (IntelliSleepScorer); closest-of-all prior work cited only as a bare URL and mischaracterized (SleepyRat → NPP 2025) | W60, W61, W62 | fatal | **A** |
| D7 | "State-of-the-art 91.5%" is false against the published record | W3, W53, W58 | fatal | **A** |
| D8 | Calibration claim contradicted by the paper's own figure | W20, W39 | fatal | **A** (restate) + B (recompute) |
| D9 | XGBoost design matrix never specified; contains raw samples; Figs. 3/4 disagree; NN-vs-XGB comparison confounded | W11, W19, W23, W36, W44 | fatal | A (disclose) + B (rerun) |
| D10 | SHAP paragraph describes a beeswarm; figure is a mean-\|SHAP\| bar chart; SHAP never cited | W21, W45, W63 | major | **A** |
| D11 | Feature-importance percentages unrecoverable from the figure under any normalization | W22, W43 | major | **A** |
| D12 | MMD called "custom"/"novel" while citing its originator; never formally defined | W1, W25, W37, W67 | fatal | **A** |
| D13 | Hjorth parameters (top 2 features) never defined, cited, or introduced in Methods | W10, W24, W64 | major | **A** |
| D14 | No preprocessing/filtering/artifact rejection; no Welch parameters; no montage, species, strain, sex; no data provenance or ethics; no label provenance | W26, W27, W31, W70, W73, W14 | major | **A** |
| D15 | Numeric inconsistencies (86.6/86.8, 83.42/83.54, 81.8/81.76); mixed metric-averaging; four mutually inconsistent split protocols; logistic-regression baseline appears only in the Conclusion at an implausible 54.9% | W15, W28, W29, W35, W42, W46, W50, W54, W81 | major | **A** (+ small B for the LR row) |
| D16 | No contribution statement; no problem formalization; no equations; no algorithm box; no Discussion; framing built on an accuracy number that four published tools beat; the one distinguishing result (calibration) is buried | W4, W5, W6, W7, W12, W13, W30, W68, W71, W72, W74, W75, W76, W78, W79, W82, W83, W84, W85, W86, W69 | major | **A** |
| D17 | No ablation: nothing removed and re-measured; no variance, no seeds, no statistical test, yet "significantly outperforming" is asserted | W8, W33, W37, W48, W49, W51 | major | **B** |
| D18 | Promissory code claim with no archive uploaded | W16 | minor | **A** (upload) |

### 1.1 Disagreements between critics, resolved

**(a) Is `garcia2021_dlmouse` merely wrong, or fabricated?**
Recon 1 flagged it only as "could not verify." Recon 3 / W56 said the DOI resolves to an unrelated
topology paper. **Recon 3 is right** — I resolved the DOI independently. Treat as fabricated:
delete the entry, do not "fix" it.

**(b) What is `sleepyrat2025_platform`?**
Recon 1: "no independent evidence it exists; treat as unverified." Recon 3 / W60: it corresponds to
a real published paper. **Recon 3 is right, and the consequence is larger than either critic
stated.** I verified: **Smith, Andrew; Milosavljevic, Snezana; Wright, Courtney J.; Grant,
Charlie A.; Pocivavsek, Ana; Valafar, Homayoun, "A deep learning software tool for automated sleep
staging in rats via single channel EEG," *NPP—Digital Psychiatry and Neuroscience*, 2025,
doi 10.1038/s44277-025-00035-y, PMID 40656054** — an LSTM scorer, rats, single-channel EEG, three
states, mean F1 87.6% over cross-validated test sets. Two consequences: (i) it is the **closest
prior work in existence** to this submission on every axis and is currently dismissed in half a
sentence as "classical scoring plus simple neural nets"; (ii) **Homayoun Valafar is also the senior
author of the arXiv preprint of this submission** (arXiv:2507.14166). This is the group's own prior
system, cited as an anonymous URL. That is simultaneously an under-positioned competitor, an
undeclared self-citation, and an anonymity exposure.

**(c) What does Fig. 3's axis actually mean?**
W22 measured the twelve bars as summing to ~0.027 (→ complexity = 52% renormalized); W43 measured
~0.0364 (→ 37.5%). My own measurement gives ~0.026 (→ ~52%). **The disagreement is immaterial and
both critics reached the right conclusion**: 14.1% is not recoverable under *any* normalization.
The material point neither critic drew is in §0.2 above — a normalized importance whose top bar is
0.0136 implies thousands of unplotted columns.

**(d) Is the SOTA claim survivable by softening?**
All three critics say delete. Agreed, and the reason is worth stating precisely: the frozen
abstract says only "outperforming all baseline methods," which is scoped to *this paper's own*
baselines. The body's "state-of-the-art" is a strictly stronger claim the abstract never made, and
it is falsified by SlumberNet (97%), IntelliSleepScorer (95.2%), MC-SleepNet (96.6%), SPINDLE
(93–99%), and by Saevskiy et al. (92%/93%) — a paper already in this manuscript's own bibliography.
Deleting it costs nothing and contradicts nothing.

**(e) Should the paper move to AISI, or to IAAI?**
Recon 1 argued AISI is a trap (its rubric scores deployment, stakeholders, field evaluation, social
impact — all of which this paper lacks) and IAAI-27 Emerging Applications is the real fit. I agree,
and the abstract deadlines for AAAI/AISI have passed anyway. See §7.3.

**(f) Is retitling sufficient?**
W12 says retitle to drop "Cross-Frequency." W80 says that does not resolve the problem because the
abstract still claims CFC metrics. **W80 is right on the constraint.** Retitling is cosmetic relief;
the CFC problem is §6.1 and is not solvable by writing.

---

## 2. Priority ranking — impact on acceptance ÷ cost to fix

Ordered by ratio, highest first. Track in brackets. Time estimates are for one competent author.

| Rank | Item | Track | Cost | Why the ratio is high |
|---|---|---|---|---|
| 1 | Fix six citations + delete two fabricated descriptions (D5) | A | 2 h | Integrity. Each is independently checkable by a reviewer in 30 s. |
| 2 | Delete every "state-of-the-art" self-description (D7) | A | 10 min | Two-word deletion removes a provably false central claim. |
| 3 | Delete the three unmeasured generalization claims (D3, part) | A | 10 min | The paper currently contradicts itself; deletion is free. |
| 4 | Rewrite the calibration sentence to match the figure (D8) | A | 30 min | The paper's *strongest* claim is refuted by the figure beneath it. |
| 5 | Rewrite the SHAP paragraph to match the figure, cite SHAP (D10) | A | 45 min | Text describes a plot that is not in the paper. |
| 6 | Delete "custom"/"novel" on MMD; attribute to Aboalayon (D12, part) | A | 15 min | The disproof is in the same sentence's own citation. |
| 7 | Disclose the split honestly + `animal` + raw samples (D1, D3, D9 — disclosure only) | A | 1 h | Converts "authors don't understand their evaluation" into "authors are honest about scope." |
| 8 | Reconcile every number; add LR row; one split protocol (D15) | A | 2 h | Cheap, checkable errors are what reviewers cite for low soundness. |
| 9 | **Retrain without `animal`** (D1) | **B** | 1 h compute | Gate. Nothing downstream is honest until this is done. |
| 10 | **Per-class table + 3×3 confusion matrix + kappa + balanced acc** (D4) | **B** | 30 min | Recompute from saved predictions. Near-free, removes the concealment reading. |
| 11 | Add IntelliSleepScorer + the NPP 2025 rat LSTM tool + AAAI citations (D6) | A | 2 h | Makes Related Work look researched; fixes an undeclared self-citation. |
| 12 | Define MMD, Hjorth, Welch params; add feature table (D12, D13, D14 part) | A | 4 h | The paper's headline features are currently unreproducible. |
| 13 | **Leave-one-animal-out, 8 folds** (D3) | **B** | 3–6 h | The objection every reviewer raises; no accepted comparable paper omits it. |
| 14 | Contributions list + reframe onto calibration/interpretability (D16 part) | A | 4 h | Answers the reviewer's literal first question. |
| 15 | Preprocessing, provenance, ethics, label provenance (D14) | A | 3 h | AAAI scores responsible research as a named criterion. |
| 16 | **Engineered-features-only XGBoost** (D9) | **B** | 1 h | The paper's actual claim is about *this* model, which is currently never evaluated. |
| 17 | Problem setting, notation, equations, algorithm box (D16 part) | A | 4 h | Flips three reproducibility-checklist items. |
| 18 | Structural rebuild: cuts, Experimental Setup, Discussion, page budget (D16) | A | 1–2 d | Removes the "ported from IEEE" signal. |
| 19 | **Recompute calibration under the clean protocol** (D8) | **B** | 1 h | If calibration is the contribution, it cannot rest on a leaky split. |
| 20 | **Feature-group ablation** (D17) | **B** | 1 d | Nothing in the paper is currently removed and re-measured. |
| 21 | **Seeds, CIs, Wilcoxon over folds** (D17) | **B** | 4 h | Fixes three self-reported "no" answers on the submitted checklist. |
| 22 | Upload anonymized code+data archive (D18) | A | 3 h | The one thing addable *after* the paper deadline. |
| 23 | **Contiguous-block split (leakage ladder)** (D3) | **B** | 3 h | Turns the biggest vulnerability into a controlled analysis. |
| 24 | **Temporal smoothing (HMM / min-bout)** (D17) | **B** | 1 d | Cheapest accuracy gain; closes "no temporal model." |
| 25 | **Re-run SAGER on this cohort** (D6) | **B** | 2–3 d | The only way to make a real comparative claim. |
| 26 | **Edge inference benchmark** | **B** | 4 h | Converts a conceded limitation into a contribution. |

---

# TRACK A — implementable now, zero new experiments

Everything in this track is writing, restructuring, formalization, honest reframing, or corrected
citations. A drafting agent can execute it from this section alone. **Two prerequisites the authors
must supply before drafting** (they are facts, not judgments, and cannot be invented):

- **P1.** What, if anything, the code computes as a cross-frequency coupling quantity. See §6.1.
- **P2.** What the `animal` column contains, what the XGBoost design matrix actually is, and which
  fitted model / split produced Figs. 3, 4, and 5.

Items A1–A8 can be drafted without P1/P2. Items A9 onward need them.

---

## A1. Bibliography surgery (D5, D6)

Edit `references.bib`. Every replacement below was verified by me against a publisher or PubMed
record during this session — I resolved each DOI/record independently rather than trusting the
recon summaries.

### A1.1 DELETE outright

```bibtex
% DELETE — DOI 10.1016/j.jneumeth.2021.109324 belongs to
% "Topological signal processing and inference of event-related potential response"
% (Wang, Behroozmand, Johnson, Bonilha, Fridriksson, J Neurosci Methods 363:109324, 2021).
% No paper matching the cited title/author/venue exists.
@article{garcia2021_dlmouse{...}}
```

Then delete the clause "and a CNN--LSTM scorer that integrates EEG and EMG streams
\citep{garcia2021_dlmouse}" from line 132.

### A1.2 REPLACE — SlumberNet

```bibtex
@article{jha2024_slumbernet,
  author  = {Jha, Pawan K. and Valekunja, Utham K. and Reddy, Akhilesh B.},
  title   = {{SlumberNet}: Deep Learning Classification of Sleep Stages Using Residual Neural Networks},
  journal = {Scientific Reports},
  volume  = {14},
  pages   = {4797},
  year    = {2024},
  doi     = {10.1038/s41598-024-54727-0}
}
```
Verified: nature.com/articles/s41598-024-54727-0; PMC10899258. **97% accuracy, 96% F1**, EEG+EMG,
plain ResNet — **no attention pooling**, and the "92.2%" in the manuscript appears nowhere in the
source.

### A1.3 REPLACE — SleepInvestigatoR

```bibtex
@article{gamble2025_sleepinvestigator,
  author  = {Gamble, Mackenzie C. and Williams, Benjamin R. and McKenna, James T. and Logan, Ryan W.},
  title   = {{SleepInvestigatoR}: A Flexible {R} Function for Analyzing Scored Sleep in Rodents},
  journal = {Sleep Advances},
  volume  = {6},
  number  = {2},
  pages   = {zpaf032},
  year    = {2025},
  doi     = {10.1093/sleepadvances/zpaf032}
}
```
Verified: academic.oup.com/sleepadvances/article/6/2/zpaf032/8138399; PubMed 38659801.

### A1.4 REPLACE — the "SleepyRat" entry (this is the important one)

```bibtex
@article{smith2025_ratlstm,
  author  = {Smith, Andrew and Milosavljevic, Snezana and Wright, Courtney J. and
             Grant, Charlie A. and Pocivavsek, Ana and Valafar, Homayoun},
  title   = {A Deep Learning Software Tool for Automated Sleep Staging in Rats via Single Channel {EEG}},
  journal = {NPP---Digital Psychiatry and Neuroscience},
  year    = {2025},
  doi     = {10.1038/s44277-025-00035-y}
}
```
Verified: PubMed 40656054; PMC12245713. LSTM, rats, single-channel EEG, three states, mean F1 87.6%,
>700 h expert-scored data. **Anonymity note:** this shares a senior author with arXiv:2507.14166.
Cite it in strict third person ("Smith et al. report…"), never as "our previous work."

### A1.5 REPLACE — multiple-classifier-systems entry

```bibtex
@article{gao2016_multi_cls,
  author  = {Gao, Vance and Turek, Fred and Vitaterna, Martha},
  title   = {Multiple Classifier Systems for Automatic Sleep Scoring in Mice},
  journal = {Journal of Neuroscience Methods},
  volume  = {264},
  pages   = {33--39},
  year    = {2016},
  doi     = {10.1016/j.jneumeth.2016.02.016}
}
```
Verified: PubMed 26928255. The bib currently says "McSweeney, D. and others" in journal "Sleep" —
both wrong. Describe it as a **multiple-classifier ensemble**, not a random forest.

### A1.6 RESOLVE — Yaghouby entry

`yaghouby2016_randomforest` ("Automatic Sleep Scoring in Rodents Using Random Forest Classifiers,"
EMBC 2016) could not be found by any critic or by me. **Delete it** and, if a citation is wanted for
that sentence, substitute the verified Yaghouby & Sunderam work — but note it is an **unsupervised
graphical model**, not a random forest, so the sentence must be rewritten, not just re-cited.
Authors must confirm the exact record before adding.

### A1.7 ADD — required new entries

```bibtex
@article{wang2023_intellisleepscorer,
  author  = {Wang, Lei A. and Kern, Ryan and Yu, Eunah and Choi, Soonwook and Pan, Jen Q.},
  title   = {{IntelliSleepScorer}, a Software Package with a Graphic User Interface for Automated
             Sleep Stage Scoring in Mice Based on a Light Gradient Boosting Machine Algorithm},
  journal = {Scientific Reports},
  volume  = {13},
  year    = {2023},
  doi     = {10.1038/s41598-023-31288-2}
}

@article{hjorth1970,
  author  = {Hjorth, Bo},
  title   = {{EEG} Analysis Based on Time Domain Properties},
  journal = {Electroencephalography and Clinical Neurophysiology},
  volume  = {29},
  number  = {3},
  pages   = {306--310},
  year    = {1970},
  doi     = {10.1016/0013-4694(70)90143-4}
}

@inproceedings{lundberg2017_shap,
  author    = {Lundberg, Scott M. and Lee, Su-In},
  title     = {A Unified Approach to Interpreting Model Predictions},
  booktitle = {Advances in Neural Information Processing Systems 30 (NIPS 2017)},
  pages     = {4765--4774},
  year      = {2017}
}

@article{wang2024_sleepdg,
  author  = {Wang, Jiquan and Zhao, Sha and Jiang, Haiteng and Li, Shijian and Li, Tao and Pan, Gang},
  title   = {Generalizable Sleep Staging via Multi-Level Domain Alignment},
  journal = {Proceedings of the AAAI Conference on Artificial Intelligence},
  volume  = {38},
  number  = {1},
  pages   = {265--273},
  year    = {2024},
  doi     = {10.1609/aaai.v38i1.27779}
}
```
Verified this session: IntelliSleepScorer (nature.com/articles/s41598-023-31288-2 — 95.2% accuracy,
Cohen's κ = 0.91, 5,776 h / 519 recordings / 124 mice; LR baseline 93.3%/κ 0.88, RF 94.3%/κ 0.89).
Hjorth 1970 (Semantic Scholar / PubMed 4195653). SleepDG (ojs.aaai.org/index.php/AAAI/article/
view/27779 + official repo BibTeX at github.com/wjq-learning/SleepDG).

### A1.8 OPTIONAL additions — verify at the publisher before adding

The critics also recommend AccuSleep (Barger et al., *PLOS ONE* 14(12):e0224642, 2019), SPINDLE
(Miladinović et al., *PLOS Comput Biol* 15(4):e1006968, 2019), MC-SleepNet (*Sci Rep* 9:15793, 2019),
Somnotate (Brodersen et al., *PLOS Comput Biol* 20(1):e1011793, 2024), the Rayan et al. rodent
sleep-scoring review (*Eur J Neurosci* 59(4), 2024), REST (*SLEEP Advances* 7(2):zpag044, 2026),
Rolnick et al. "Position: Application-Driven Innovation in Machine Learning" (ICML 2024, PMLR v235),
and the Xu et al. temporal-autocorrelation study (arXiv:2405.17024). **I did not personally open
publisher records for these eight.** Given that six of seven existing domain citations were wrong,
open each one before adding it. One more bad citation is unrecoverable.

### A1.9 De-identify the competition citation

Delete `bdhsc2024case` and the clause "Validated through a national big data health science case
competition \citep{bdhsc2024case}," from line 117. The full competition name in the bib title field
is uniquely searchable and identifies the host institution and 2024 participants. Restore in
camera-ready only.

---

## A2. Kill the false claims (D3, D7) — surgical deletions

Do these first; they take ten minutes and each removes a claim a reviewer can disprove.

| Line | Delete / replace |
|---|---|
| 115 | Delete ", allowing the model to generalize across individual animals and diverse recording conditions." End the sentence at "vigilance states." |
| 117 | Delete "Validated through a national big data health science case competition \citep{bdhsc2024case}," and "with balanced precision, recall, and F1-scores across states". |
| 129 | Rewrite whole sentence (see A3). Delete "yielding a state-of-the-art 91.5\% accuracy" and "by adding cross-frequency coupling and the MMD feature". |
| 142 | Delete "mitigates (ii) through boosting's inherent feature subsampling,". Replace with an explicit statement that cross-lab robustness is untested here. |
| 253 | Delete the entire sentence "The evaluation framework prioritized models demonstrating consistent performance across different animals and recording conditions to ensure robust generalization capabilities." |
| 303 | Delete "significantly" (no test was run; the submitted checklist says so). |
| 361 | Delete "Although our framework delivers state-of-the-art performance for automated rodent sleep staging," → "Several factors limit the generality of the current results:" |
| 379 | Delete "The robust performance across all vigilance states, particularly challenging REM sleep detection, establishes…" — rewrite once B3 gives the real per-class numbers. |
| 377 | Delete "The MMD feature successfully captured unique temporal dynamics of neural oscillations, enhancing discrimination between…" — nothing in the paper removes MMD and re-measures. Replace with a magnitude statement only, or delete pending the B6 ablation. |

---

## A3. Rewrite Related Work (D5, D6, D7) — exact replacement text

Replace the "Gradient-Boosted Trees" subsection (lines 128–129) with:

> **Gradient-Boosted Trees.** Boosting algorithms such as XGBoost \citep{Chen_2016} exploit
> non-linear feature interactions over engineered descriptors. The closest prior system to ours is
> IntelliSleepScorer \citep{wang2023_intellisleepscorer}, a LightGBM scorer for mice trained on
> 5{,}776\,h across 519 recordings from 124 animals, reporting 95.2\% accuracy and Cohen's
> $\kappa=0.91$ against logistic-regression (93.3\%) and random-forest (94.3\%) baselines. It uses
> EEG together with EMG and reports agreement statistics but not probability calibration or
> per-feature attribution. Our work occupies a different point in the same design space:
> single-channel EEG without EMG on a smaller cohort, with an interpretability and calibration audit
> that this line of work does not report.

Move `saevskiy2025_sensors` **out of this subsection** into "Classical Feature-Based Approaches" and
describe it correctly:

> \citet{saevskiy2025_sensors} combine artifact processing, multi-band spectral analysis, and
> Gaussian-mixture clustering, reporting 92\% (10 rats) and 93\% (10 mice) sleep--wake accuracy with
> REM-detection accuracy of 91\% and 89\% respectively.

(The manuscript currently quotes the mouse **REM-specific** 89% as if it were overall accuracy, in a
subsection that misdescribes the method family. Both must go.)

Replace the SlumberNet sentence at line 132 with:

> Recent mouse-specific examples include SlumberNet \citep{jha2024_slumbernet}, a residual network
> over paired EEG and EMG reporting 97\% accuracy and 96\% F1. Such networks require multi-modal
> recording and GPU-scale training that many preclinical labs cannot provision.

Replace the "Lightweight and Single-Channel Solutions" subsection (lines 134–135) with:

> **Lightweight and Single-Channel Solutions.** To lower cost and tethering burden, EMG-free
> pipelines have emerged. Closest to the present setting, \citet{smith2025_ratlstm} score rat
> vigilance states from single-channel EEG with a recurrent deep network, reporting a mean F1 of
> 87.6\% under cross-validation on 16 animals. Complementary tooling exists for downstream analysis:
> \citet{gamble2025_sleepinvestigator} provide an R function that computes standard sleep-architecture
> measures from already-scored recordings. Our work differs from \citet{smith2025_ratlstm} in
> representation---explicitly engineered, physiologically interpretable features rather than a learned
> deep representation---and in what is reported: per-feature attribution and probability calibration,
> neither of which that system provides.

Rewrite "Research Gaps" (lines 137–142). Keep gap (i) only if restated as a **cost/access** gap;
restate gap (ii) as an **open problem this paper does not solve**; keep gap (iii); **add gap (iv)**:

> (iv) \emph{Calibration}---no published rodent scorer reports whether its confidence scores are
> calibrated, yet closed-loop stimulation protocols threshold on exactly those scores.
>
> Our pipeline addresses (i) with a low-dimensional, physiologically grounded feature basis, (iii)
> via SHAP-based attribution \citep{lundberg2017_shap}, and (iv) via a reliability analysis of the
> predicted class probabilities. We do not address (ii): cross-laboratory and cross-strain
> robustness is untested here, and domain-generalization methods such as \citet{wang2024_sleepdg}
> set the standard the field is converging on.

**Before writing the "no published rodent scorer reports calibration" sentence, verify it** against
each work you cite. It is the paper's new central claim; it must survive one search.

---

## A4. Fix the interpretability text so it matches the figures (D8, D10, D11)

### A4.1 Calibration paragraph (line 347) — REPLACE

I inspected `images/calibration_curve.png` directly. It shows approximately **20** bins at 0.05
spacing, not ten, and the Class-0 curve is systematically **above** the diagonal across the entire
mid-to-upper range — deviations on the order of 8–10 percentage points at predicted probabilities
of ~0.32, ~0.48, ~0.73 and ~0.83. The claim "the maximum deviation from the diagonal never exceeds
3 percentage points" is refuted by the figure printed directly beneath it.

Replacement text (fill the bracketed values from the arrays that produced the plot — **do not read
them off the PNG**):

> Figure~\ref{fig:calib} reports one-vs-rest reliability curves over [N] equal-width probability
> bins. Two of the three classes track the diagonal closely (maximum deviation [x] and [y]
> percentage points), while [class] is systematically over-confident above a predicted probability
> of 0.3, with a maximum deviation of [z] percentage points. The expected calibration error is
> [ECE] and the multiclass Brier score is [B] under the [state convention] normalization, computed
> on [named split].

Also: **relabel Class 0/1/2 → Wake/SWS/REM** in the figure legend and state the `LabelEncoder`
mapping in Methods. As it stands a reader cannot tell whether the badly-calibrated class is the
minority class. Fix the clipped "Fraction of Positives" y-axis label.

### A4.2 SHAP paragraph (line 338) — REPLACE

`images/Shap.png` is a **stacked mean(|SHAP value|) bar chart**, x-axis labelled
"mean(|SHAP value|) (average impact on model output magnitude)". Absolute values discard sign; a bar
chart plots one aggregate per feature per class. Every directional and distributional claim in the
current paragraph — "points clustered at positive SHAP values," "pushes toward," "drives predictions
away from," "bimodal pattern" — is unsupported by the figure it cites.

Two options; pick one and do not mix them.

- **Option 1 (writing only, recommended for Track A).** Keep the figure, rewrite the paragraph to
  make magnitude claims only:
  > Figure~\ref{fig:shap} reports mean absolute SHAP values \citep{lundberg2017_shap} per feature and
  > per class, computed with TreeSHAP on [named split]. The ranking is dominated by the two Hjorth
  > descriptors and by \emph{delta\_power}, whose attribution mass is concentrated on the [class]
  > output. Absolute values discard sign, so this figure supports statements about which features the
  > model relies on, not about the direction of their effect.
  Then **delete** the sentence "These trends are physiologically plausible and give confidence that
  the classifier is leveraging meaningful neural signatures rather than spurious artifacts." With a
  subject-ID feature ranked 7th and raw sample indices carrying attribution, the SHAP evidence
  currently points the other way.
- **Option 2 (Track B).** Generate the actual per-class beeswarm and keep the directional narrative,
  stating which class is shown.

Either way, add the SHAP citation (currently absent entirely), state the estimator (TreeSHAP
interventional vs. `tree_path_dependent`), and state which model and which split produced the values.

### A4.3 Feature-importance paragraph (line 329) — REPLACE

The quoted percentages (14.1%, 10.3%, ~2%, "top five = 35%") are not recoverable from Fig. 3 under
any normalization. Re-emit them from the arrays, state `importance_type` explicitly (XGBoost exposes
`weight`, `gain`, `total_gain`, `cover`, `total_cover`, which rank differently), state the
denominator, and put them in the feature table (A6) as a column rather than in prose. Two further
required fixes:

- The text lists `delta_power, alpha_power, gamma_power`; Fig. 3 orders them `alpha, delta, gamma`.
- The gloss "Both belong to the Hjorth family and quantify high-frequency variance" is imprecise.
  Mobility is $\sqrt{\mathrm{Var}(x')/\mathrm{Var}(x)}$, a normalized mean frequency; complexity is
  the ratio of the mobility of the derivative to the mobility of the signal, a bandwidth measure.
  Neither "quantifies high-frequency variance."

Add one sentence noting that Hjorth parameters are functions of the second and fourth spectral
moments and are therefore strongly collinear with the band-power features — which is exactly the
regime in which gain-based importance is unreliable.

---

## A5. Honest disclosure of the evaluation protocol (D1, D3, D9)

Add this paragraph to Methodology, replacing the second half of line 242. **This is required whether
or not any Track B experiment is run.**

> Epochs from all eight animals were pooled and split at random with stratification by vigilance
> state into 80\% training and 20\% test. The split is therefore \emph{epoch-wise}, not
> \emph{animal-wise}: every animal contributes epochs to both partitions, and because the recordings
> are continuous, temporally adjacent 10\,s epochs from the same sleep bout can fall on opposite
> sides of the split. The figures reported here are consequently within-cohort, subject-dependent
> estimates that upper-bound cross-animal performance, and they are not directly comparable to the
> subject-independent figures reported by scorers evaluated with held-out animals or held-out
> datasets.

Also **fix line 242's account of stratification**, which is currently backwards: "Stratified sampling
was critical to maintain the natural distribution of vigilance states, preventing class imbalance
issues" — stratification *preserves* imbalance by construction and does nothing about majority-class
bias. Replace with: "Stratification guarantees that the test partition carries the same class
proportions as the population; no class reweighting or resampling was applied." (Confirm the last
clause: neither model config lists `scale_pos_weight` or class weights.)

Add to the same section, pending P2:

> The design matrix supplied to the classifier comprises [d] columns: [enumerate]. It includes an
> integer animal index, and [statement about raw samples]. [If retained:] Because the animal index
> is a categorical identifier of the training cohort, the model is not applicable to animals outside
> it; see Limitations.

And delete the fourth, orphaned protocol at line 257 ("the validation set (30\% of the training
data)"), which matches none of the three splits described in Methods.

---

## A6. New Methods content — what to write and where

### A6.1 Problem Setting (new subsection, insert before line 173)

> **Problem setting.** Let $x \in \mathbb{R}^{5000}$ denote one 10\,s epoch sampled at 500\,Hz,
> recorded from animal $a \in \{1,\dots,8\}$ and labelled $y \in \{\text{Wake}, \text{SWS},
> \text{REM}\}$. A feature map $\phi: \mathbb{R}^{5000} \to \mathbb{R}^{d}$ produces the descriptors
> of \S[Feature Engineering], and a classifier $f: \mathbb{R}^{d} \to \Delta^{2}$ outputs a
> distribution over the three states. Empirical risk minimization over a randomly partitioned pool of
> epochs assumes those epochs are i.i.d.\ draws. That assumption is violated twice here: epochs are
> nested within eight animals, and consecutive epochs within a sleep bout are strongly
> autocorrelated. We state the assumption explicitly because it determines how the reported figures
> should be read (see \S[Experimental Setup]).

### A6.2 Preprocessing (new subsection, insert after Data Collection)

Currently there is **no** preprocessing description anywhere: the words *filter, bandpass, notch,
detrend, re-reference, montage, artifact rejection* appear nowhere. And the arithmetic shows none
was applied — 8 animals × 48 h × 360 epochs/h = 138,240 exactly, so not one epoch was excluded from
a two-day recording of freely-behaving animals. State: filter cascade (type, cutoffs, order,
zero-phase or not); whether a mains notch was applied and at what frequency; reference scheme;
electrode placement and channel count; artifact-rejection rule and the number/percentage of epochs
excluded. **If none was performed, say so explicitly** and add to Limitations:

> No filtering or artifact rejection was applied. The gamma band as defined here (30--100\,Hz)
> contains the mains line frequency, and in single-channel EEG without an EMG reference, 30--100\,Hz
> power during wake is not separable from muscle activity. This bounds how far the attribution
> analysis can be read as neurophysiological rather than artifactual.

### A6.3 Welch parameters (rewrite line 176)

"Welch's method with appropriate windowing" specifies nothing. State `nperseg`, `noverlap`, `window`,
`nfft`, `detrend`, `scaling`; give the resulting frequency resolution in Hz; state the band-edge
convention (is a bin at exactly 4 Hz delta or theta?); state whether band power is integrated or
mean PSD; state whether a log transform was applied. This matters concretely: delta is defined
0.5–4 Hz, and a Welch segment shorter than 2 s cannot resolve its lower edge at all.

### A6.4 MMD as an equation (rewrite lines 188–191)

Currently: "the statistical distance between maximum and minimum signal values over specified time
windows" with "sliding window analysis." No window length, no stride, no aggregation, no formula, no
dimensionality — and Results later calls it "a single scalar per epoch," which is inconsistent with
an unaggregated sliding-window analysis. Write it as a displayed equation with explicit window
length $W$, stride $s$, per-window statistic, and aggregation across windows, transcribed from the
code. **And fix the attribution:**

> We adopt the Maximum--Minimum Distance descriptor introduced for human sleep staging by
> \citet{e18090272}, which computes the distance between the maximum and minimum sample of each
> sliding sub-window. To our knowledge it has not previously been evaluated for rodent vigilance
> staging. [Verify this last claim before writing it.]

Delete "custom" (line 189) and "novel" (line 201). The word "novel" occurs exactly once in the
manuscript and is attached to a feature defined in the reference cited in the same sentence.

### A6.5 Hjorth parameters (new subsection)

Define activity, mobility, complexity as displayed equations; cite `hjorth1970`; state whether
derivatives were computed by finite differences or in the frequency domain (this changes values
materially at 500 Hz); state whether they were computed on raw or filtered signal. These two
features are the model's top-ranked predictors and currently appear for the first time in Results.
`activity` appears in both figures and is not mentioned in the text at all.

### A6.6 Feature table (new Table 1)

One row per column of the design matrix: name, definition, per-epoch dimensionality, running total,
and (once A4.3 is done) importance share. End with the exact total fed to XGBoost. Reconcile with
the "5,009 features" at line 201 and state explicitly whether the neural network and XGBoost
received the same inputs. **If they did not, reframe the Table-1 comparison in the text, table
caption and figure caption as engineered-features-vs-raw-samples, not as a model comparison** — the
conclusion "XGBoost is effective in handling the engineered feature set" (line 310) does not
otherwise follow.

Note the arithmetic does not close under any reading: 5,000 raw + the 11 named columns in Fig. 4 is
5,011, not 5,009.

### A6.7 Algorithm box

One `\begin{algorithm}` for the end-to-end pipeline: preprocessing → Welch PSD → five-band
absolute/relative power, peak frequency, spectral entropy → Hjorth → MMD → [CFC, pending P1] →
concatenate → XGBoost. Flips reproducibility-checklist item 1.1 from "partial" to "yes" at zero
experimental cost. Figure 1's four boxes do not substitute.

### A6.8 Data provenance and ethics (new subsection)

AAAI-27 scores adherence to responsible research practices as a named additional criterion, and the
reproducibility checklist requires a motivation for the dataset choice. State: who collected the
recordings and under what animal-use protocol/approval; **species** (the paper says only "8
laboratory rodents" and never says mouse or rat, while comparing against mouse- and rat-specific
published numbers); strain, sex, age; housing and light cycle; licence and redistribution terms;
**who produced the ground-truth labels, how many scorers, under what criteria, and how disagreements
were resolved.** The paper cites human inter-rater κ = 0.76–0.85 twice but never says how its own
reference standard was produced — so 91.5% is currently measured against an unspecified target.

Move the "single-channel EEG" fact out of Limitations item 3 (page 5) into this subsection.

---

## A7. Numeric reconciliation (D15) — a checklist

| Where | Current | Action |
|---|---|---|
| Table 1, line 320 | precision 86.6% | → **86.8%** (abstract is frozen at 86.8%; the table must move). |
| Line 307 vs. line 319 | NN recall 83.42% vs. 83.54% | Return to source arrays; pick one; use everywhere. |
| Fig. 2, line 287 vs. line 319 | NN F1 81.8 vs. 81.76 | Regenerate figure from the same array as the table — or delete the figure (A8). |
| Table 1, line 319 | NN accuracy = NN recall = 83.54% | Diagnostic of a **weighted** recall for the NN against a **macro** recall for XGBoost. Recompute all cells under one stated convention; state it in the caption. If both are wanted, use separate labelled columns. |
| Line 377 | LR 54.9%, appears nowhere else | Add an LR row to Table 1 with the full metric set on the same held-out epochs. State its feature vector, regularization strength, solver, iterations. Add a **majority-class floor row** so 91.5% has a reference point. If 54.9% cannot be reproduced or explained, delete the claim. |
| Line 365 | "fewer than 11,000 training samples" | 8% of 138,240 = 11,059, which is the **full-set** count, not the 80% training portion. Correct. |
| Lines 241–242, 248, 257 | four inconsistent protocols | Collapse to one named protocol; state it once; use it for every number. Delete the unused splits or report results for them. |
| Line 251 | "grid search evaluation with cross-validation" | State the grid (parameter, values tried, count), fold count, fold-assignment unit, selection criterion, and whether search folds were animal-disjoint. Checklist item 4.1 asks for exactly this. |
| Lines 226–235 | optimized config | Missing `max_depth` (named in the tuning sentence), `objective`, `tree_method`, `min_child_weight`, `eval_metric`. Note that `multi:softmax` at line 221 returns **hard labels** and is incompatible with the reliability diagram and Brier score — state the objective actually used. Add software/library versions and hardware. |
| Line 245 | `StandardScaler` | State that it was fitted on training data only. As written, nothing rules out fitting on the full dataset — an additional leakage path for the NN row. |

---

## A8. Structural rebuild and page budget (D16)

The submission is ~5.3 body pages against a 7-page limit (9 total, pages 8–9 references only).
There is roughly **1.7 free pages**, and the cuts below recover roughly **another 1.0**.

### A8.1 CUT (recovers ~735 words + ~0.6 column of floats ≈ 1.0 page)

| Lines | What | Recovered |
|---|---|---|
| 146–166 | **Figure 1** (four-box TikZ pipeline). It is the universal supervised-learning diagram and is net-negative: the "5000 points" label is the only place the raw-sample count appears and it conflicts with "5,009 features." | ~0.25 col |
| 259–300 | **Figure 2** (bar chart). Plots six numbers that are a strict subset of Table 1's eight and disagrees with it on two. | ~0.35 col |
| 212–214 | Adam / dropout / sparse-categorical-cross-entropy / batch-size prose | ~90 w |
| 219–224 | "Standard XGBoost configuration" bullets — this variant's results are never reported | ~40 w |
| 244–245 | StandardScaler / LabelEncoder subsection → compress to one clause each | ~75 w |
| 237 | The results sentence embedded in Methodology (results do not belong in Methods) | ~30 w |
| 257, 305–308, 310 | Redundant results bullets and filler ("These results affirm the effectiveness of…") | ~90 w |
| 109 | Duplicated physiology (kept once, in Related Work line 123) | ~70 w |
| 107–113 | Compress Introduction paras 1–4 from 290 words to ~120 | ~170 w |
| 178–184 | Five band bullets → two rows of the feature table; delete "linked to conscious awareness" (unsupported anthropomorphism in a rodent methods section) | ~110 w |
| 169, 171 | Data-collection filler ("optimal balance between temporal granularity and statistical reliability", "minimizing environmental disturbances") | ~60 w |

### A8.2 Target section budget (7 content pages, references from page 8)

| Section | Budget | Contents |
|---|---|---|
| Introduction | 0.75 pg | Compressed motivation; research question; **four-bullet contributions list** (A8.3) |
| Related Work | 0.60 pg | Deduplicated physiology; corrected citations; positioning sentence per subsection; four gaps incl. calibration |
| Problem Setting & Method | 1.30 pg | Notation; preprocessing; feature table; MMD/Hjorth/entropy/[CFC] equations; algorithm box |
| Experimental Setup *(new section)* | 0.70 pg | Data provenance & ethics; species/strain/sex; one named split protocol; formal metric definitions; full baseline set; hyperparameter grid + selection criterion; compute |
| Results | 1.60 pg | Main table (incl. LR row, majority-class floor, κ, balanced accuracy); per-class table; 3×3 confusion matrix; class-distribution table; calibration table; Figs. 3 and 5 |
| Discussion *(new section)* | 0.50 pg | Three named-claim subsections (A8.4) |
| Limitations | 0.35 pg | 4–5 items (A8.5) |
| Conclusion | 0.20 pg | Three sentences |

Net new writing ≈ 2,000 words, most of which is specification the authors already possess. If space
is tight, cut Discussion to 0.35 pg and fold the confusion matrix into the per-class table.

### A8.3 Contributions list (append to Introduction, style-matched to `/home/user/quantifying-the-gaps/AAAI_27/AnonymousSubmission2027.tex` lines 176–182)

Put a research question above it:

> How much of the accuracy of end-to-end deep rodent sleep scorers is recoverable from a small set
> of physiologically grounded single-channel features, and what is gained in interpretability and
> probability calibration by giving that accuracy up?

> **Our contributions are:**
> - **A fully specified, low-dimensional feature basis** for single-channel rodent vigilance staging,
>   with every descriptor defined, its dimensionality stated, and the extraction code released.
> - **An attribution audit** showing which descriptors the classifier's decisions rest on, and which
>   they do not.
> - **A calibration analysis** of the predicted class probabilities---a reliability analysis and
>   proper scoring rule that, to our knowledge, no published rodent scorer reports---establishing
>   what the confidence scores can and cannot be thresholded on for closed-loop use.
> - **An explicit account of evaluation protocol**, including the subject-dependence of the
>   epoch-wise split standard in this application literature and what it does and does not license.

Every bullet is substantiated by material already in the paper; none requires a new experiment; none
contradicts the abstract. Note bullet 4 is honest rather than triumphant — that is deliberate, and
it is what converts the leakage problem from a hidden defect into a stated scope condition.

### A8.4 Discussion section (new, between Results and Limitations)

Three subsections, each a claim rather than a category — matching the companion paper's style
(lines 251–311 of `/home/user/quantifying-the-gaps/AAAI_27/AnonymousSubmission2027.tex`, where
subsections are propositions like "A Task Monoculture"):

1. *Handcrafted Features Recover Much of the Deep-Model Accuracy at Negligible Compute* — the
   accuracy/compute trade-off, stated honestly against published scorers, never as a SOTA claim.
2. *What the Attribution Analysis Does and Does Not Establish* — which markers the model uses; and,
   candidly, what the presence of a subject-ID feature and raw-sample columns means for that reading.
3. *Calibrated Probabilities, Not Accuracy, Are the Binding Requirement for Closed-Loop Use* — the
   reliability result, why no cited rodent scorer reports it, and what a laboratory can actually do
   with it.

Also: cut the Conclusion's second paragraph, which restates the Introduction almost verbatim.

### A8.5 Limitations rewrite (lines 358–373)

- Delete the "state-of-the-art" opening clause.
- Promote three facts out of Limitations and into where they belong: the class prior (item 2) → a
  Results class-distribution table; single-channel modality (item 3) → Data Collection; training time
  (item 7) → an Experimental Setup compute paragraph.
- Either compute the confidence intervals or soften item 2 — it currently asserts that "performance
  estimates for REM carry wider confidence intervals" when no interval is computed anywhere.
- **Add:** the protocol limitation (epoch-wise split; all animals in both partitions; temporally
  adjacent near-duplicates on both sides).
- **Add:** the ceiling limitation — accuracy measured against a reference standard with κ = 0.76–0.85
  internal disagreement cannot be read as approaching a true ceiling; agreement-with-consensus or κ
  against the label set is the more informative metric. Written well this reads as methodological
  sophistication, not concession.
- **Add** (pending P2): if the `animal` feature is retained, that the classifier is not applicable to
  unseen animals.

### A8.6 Register and headings

- Convert Methodology from agentless passive ("was performed," "were computed" — 24 instances) to
  active first-person plural, matching the companion paper.
- Delete evaluative adjectives standing in for evidence: "comprehensive" (×6), "robust" (×8) as
  self-description, "critical" (×7), "systematic" (×5), "optimal", "superior", "state-of-the-art"
  (×3). A two-layer 128/64 MLP is not "a comprehensive deep learning architecture."
- Split Methodology into **Method** and **Experimental Setup**; flatten the IEEE-style
  subsubsection nesting; rename analysis subsections to claims.

### A8.7 Title

Optional and lower-priority than everything above. "XGBoost-Based Automated Vigilance State
Classification from Rodent EEG Using Spectral and Cross-Frequency Features" names an off-the-shelf
library and a feature family the paper does not compute. Something like *"Interpretable, Calibrated
Vigilance State Classification from Single-Channel Rodent EEG"* carries a claim. **But note the
ordering constraint: dropping "Cross-Frequency" from the title does not resolve §6.1**, because the
frozen abstract still claims CFC metrics. And if the paper is already submitted, changing the title
on the submission form may not be possible.

---

## A9. Submission-form and compliance items

- **Primary keyword:** set to `APP: Healthcare & Bioinformatics Applications` (or `APP: AI for
  Science (Natural & Physical Sciences)`). One primary keyword is mandatory and it determines the
  reviewer pool. A core-ML keyword routes this to reviewers whose default prior is that engineered
  features plus XGBoost is not a contribution.
- **Code archive (A/D18):** the abstract claims "The code is released as a public resource." AAAI-27
  supplementary policy reportedly instructs reviewers not to credit promised-on-acceptance materials.
  Upload a real anonymized archive by the supplementary deadline (three days after the paper
  deadline — verify the exact date). It must contain the feature-extraction code (which is also the
  authoritative answer to P1 and the MMD parameterization), the exact XGBoost config, and the
  split-generation script. This is the one item addable **after** the paper deadline.
- **Reproducibility checklist:** re-answer honestly *after* the above, not before. Items 1.1
  (pseudocode), 4.1 (hyperparameter ranges + selection criterion) and 4.9 (formal metric definitions)
  become defensible "yes" from Track A alone. Items 4.10/4.11/4.12 (runs, variation, significance
  tests) only become "yes" after Track B. **An unsupported "yes" is worse than an honest "no"** —
  reviewers cross-check the checklist against the paper.
- **arXiv:** arXiv:2507.14166 carries a near-identical abstract under real author names. AAAI-27
  permits preprints, but the submission must not cite or point to it, and the preprint must not
  mention AAAI-27. The current `.tex` is compliant. Verify the arXiv page and the GitHub README
  contain no AAAI-27 mention, and do not post about the submission during review.

---

# TRACK B — requires running code on the authors' data

Ordered by marginal effect on acceptance probability. **Write no number for any of these before it
is computed.** Time estimates assume the existing pipeline and a model that trains in minutes.

### B1. Retrain without the `animal` column — **the gate**
- **Run:** drop the subject-ID column from the feature matrix; retrain the optimized XGBoost with
  identical hyperparameters (lr 0.1, 500 estimators, subsample 0.8, colsample_bytree 0.8, gamma 0,
  reg_lambda 1, seed 42) on identical epochs. Regenerate accuracy/precision/recall/F1, both
  importance figures, the SHAP analysis, and the calibration curve.
- **Why a reviewer needs it:** the subject identifier is visible in the paper's own Figure 3. A model
  conditioned on an integer animal ID has undefined behaviour on a ninth animal, which vacates both
  the generalization claim and the deployment framing. Report **both** numbers — with and without —
  because the gap is itself a quantitative measure of how much of 91.5% was subject memorization.
- **Cost:** ~1 h compute.
- **Produces:** corrected Table 1; regenerated Figs. 3, 4, 5; one sentence reporting the delta.
- **Abstract risk:** see §6.2. This is the one required fix that may collide with 91.5%.

### B2. Leave-one-animal-out cross-validation (8 folds)
- **Run:** `GroupKFold` on the animal index, identical feature pipeline and hyperparameters. Report
  per-fold and mean ± sd accuracy, macro-F1, Cohen's κ, per-class REM recall.
- **Why:** every accepted comparable evaluation partitions by subject or dataset — SleepDG
  (leave-one-dataset-out), MASS (subject-partitioned folds), CCIL (cross-person), Golany et al.
  (AAMI inter-patient DS1/DS2). No accepted AAAI paper in this space uses a stratified random split
  over pooled epochs. Present as an **additional stricter protocol** alongside the epoch-wise 91.5%,
  never as a replacement; the *gap between the two* is the most scientifically interesting result the
  paper can report.
- **Cost:** 3–6 h. No new data, no new code beyond the grouping.
- **Produces:** a per-fold table (8 rows × 4 metrics + mean/sd row); reframes the whole Results section.

### B3. Per-class metrics, confusion matrix, κ, balanced accuracy — **cheapest high-value item**
- **Run:** recompute from saved predictions. No retraining. 3×3 confusion matrix (rows = ground
  truth, Wake/SWS/REM); per-class precision/recall/F1/support; Cohen's κ; balanced accuracy. For
  both the epoch-wise split and every new protocol. Plus a class-count table (counts and percentages
  overall and per animal).
- **Why:** on an 8%-minority problem where every narrative claim concerns the minority class, macro
  aggregates alone read as concealment — and §0.3 shows a reviewer can *derive* REM recall ≈54% from
  the four numbers already published. Being caught hiding it is far worse than reporting it. κ is
  additionally the only quantity comparable to the human inter-rater κ = 0.76–0.85 the paper itself
  cites twice.
- **Cost:** ~30 min.
- **Produces:** confusion-matrix figure; per-class table; class-distribution table; the numbers that
  let lines 117 and 379 be rewritten truthfully.

### B4. Engineered-features-only XGBoost
- **Run:** train on the engineered descriptors only, with raw sample columns removed. Report as the
  headline model.
- **Why:** the paper's entire argument is that a compact physiologically grounded feature basis
  suffices. If the reported model consumed 5,000 raw samples plus ~9 descriptors, that model is never
  evaluated anywhere in the paper, and the NN-vs-XGBoost comparison is confounded (different inputs,
  not different models). Also retrain the same feed-forward network on the identical engineered
  vector and report it as a third Table 1 row so the comparison is finally matched.
- **Cost:** ~1–2 h.
- **Produces:** the model the paper claims to be about; a valid Table 1.

### B5. Recompute calibration under the clean protocol
- **Run:** reliability diagram, ECE, MCE, per-class and multiclass Brier under B1+B2. State bin count
  and binning scheme, one-vs-rest vs. top-label, and the normalization convention for Brier.
- **Why:** calibration is being promoted to the paper's primary contribution, and calibration is
  exactly what degrades under subject shift. Measured on a split whose test epochs' neighbours were in
  training, it is not evidence. Also produces something no cited rodent scorer reports.
- **Cost:** ~1 h.
- **Produces:** a calibration table (ECE/MCE/Brier per class per protocol); corrected Fig. 5.

### B6. Feature-group ablation
- **Run:** named variants with everything else fixed — spectral-only / +Hjorth / +MMD / [+CFC] / full
  — under LOAO on every fold, ≥5 seeds each. Report accuracy, macro-F1, per-class REM recall per cell.
  Also report the "Standard XGBoost" default-hyperparameter variant that Methods promises and never shows.
- **Why:** nothing is currently removed and re-measured. A gain share is an attribution statistic on
  one fitted model, unreliable under correlated features — and band powers, Hjorth parameters and an
  amplitude-range statistic are manifestly correlated. If MMD does not survive, **report that**: a
  negative result on a feature the abstract merely *lists* is publishable and far less damaging than a
  reviewer suspecting it was never tested.
- **Cost:** ~1 day.
- **Produces:** an ablation grid (5 variants × 3 metrics); the only evidence that can support or
  refute the Conclusion's MMD claim.
- **Caveat:** the CFC arm presupposes a CFC feature exists. See §6.1.

### B7. Repair the baseline set
- **Run:** logistic regression on the *same* design matrix and split as XGBoost, with stated
  regularization/solver/iterations; plus a majority-class floor.
- **Why:** 54.9% appears only in the Conclusion and sits at roughly the trivial rate on a three-class
  problem whose majority class alone likely exceeds 50%. IntelliSleepScorer's LR baseline reaches
  93.3%, which makes 54.9% look broken rather than informative. Reviewers penalize strawman baselines
  heavily, and the abstract's "outperforming all baseline methods" is currently unsubstantiated in
  the body.
- **Cost:** ~2 h.
- **Produces:** Table 1 rows that substantiate the abstract's comparative claim.

### B8. Variance and significance
- **Run:** ≥5 seeds per model, mean ± sd on every metric. Bootstrap CIs by resampling **held-out
  animals**, not epochs (bootstrapping autocorrelated epochs yields meaninglessly tight intervals).
  Wilcoxon signed-rank across the 8 LOAO folds for XGBoost vs. NN and vs. LR.
- **Why:** the submitted checklist answers "no" to number of runs, to any measure of variation, and to
  statistical significance testing — while the Results say "significantly outperforming." Reviewers
  see the checklist. Eight paired folds is enough for the test; one 20% split is not.
- **Cost:** ~4 h.
- **Produces:** ± columns throughout Table 1; a significance row; three checklist "no"s → "yes".

### B9. Contiguous-block split (the leakage ladder)
- **Run:** within-animal contiguous-block split (whole hours or whole light/dark phases) as the
  intermediate rung between random-epoch and LOAO.
- **Why:** reporting all three side by side converts the paper's largest vulnerability into a
  controlled analysis and quantifies, for the rodent-EEG community, how much the field-standard random
  split inflates results. That is a contribution in its own right.
- **Cost:** ~3 h.
- **Produces:** a three-row protocol-comparison table — arguably the paper's most interesting figure.

### B10. Temporal smoothing
- **Run:** HMM over the predicted state sequence, or a minimum-bout-duration / median-filter rule.
  Report with and without as an ablation.
- **Why:** essentially every serious published rodent scorer applies temporal context (Somnotate:
  LDA+HMM; SPINDLE: CNN+HMM). This paper classifies each epoch i.i.d. and does not mention the issue.
  It is both a missing baseline and the cheapest available accuracy gain.
- **Cost:** ~1 day.
- **Produces:** two extra Table 1 rows; closes the "no temporal model" criticism.

### B11. Run a published scorer on this cohort
- **Run:** SAGER (Saevskiy et al., open source at github.com/sykesva/SAGER) on these eight animals
  under the identical protocol. AccuSleep requires EMG and is not applicable to this dataset.
- **Why:** the entire current baseline set is two models the authors wrote themselves. No surveyed
  accepted AAAI paper does that. A head-to-head against a real published method on the same epochs is
  worth more than any additional in-house ablation. Report no comparison number not produced by
  actually running the method on this data.
- **Cost:** 2–3 days (integration risk).
- **Produces:** the only legitimate cross-method comparison in the paper.

### B12. Edge inference benchmark
- **Run:** throughput and per-epoch latency on CPU and on a Raspberry-Pi-class device.
- **Why:** Limitations item 7 concedes this is unmeasured, and deployability is the only axis on
  which a 500-tree ensemble beats a residual CNN. Converts a conceded limitation into a contribution.
- **Cost:** ~4 h.
- **Produces:** one compute table; strengthens the accuracy/compute trade-off claim.

---

## 6. Abstract-consistency audit

The frozen abstract commits to: 91.5% accuracy, 86.8% precision, 81.2% recall, 83.5% F1; three
states (REM/SWS/wake); spectral band power (delta–gamma); MMD; **cross-frequency coupling metrics**;
XGBoost; "outperforming all baseline methods"; "the code is released as a public resource."

### 6.1 Track A items checked against the abstract — all safe except one

| Track A item | Abstract claim touched | Verdict |
|---|---|---|
| Delete "state-of-the-art" (A2) | none — abstract never claims SOTA | **SAFE**. "Outperforming all baseline methods" is scoped to this paper's own baselines. |
| Delete "custom"/"novel" on MMD (A6.4) | "temporal dynamics via Maximum-Minimum Distance (MMD)" | **SAFE**. The abstract lists MMD as a feature used; it never claims the feature is new. |
| Table 1 precision 86.6 → 86.8 (A7) | 86.8% precision | **SAFE and required** — it makes the body agree with the abstract. |
| Disclose epoch-wise split (A5) | none — abstract states no protocol | **SAFE**. |
| Disclose `animal` as an input (A5) | "including spectral power…, MMD, and CFC" | **SAFE**. "Including" is non-exhaustive; disclosing an additional input does not contradict it. Rhetorically awkward, factually compatible. |
| Disclose raw-sample columns (A5, A6.6) | same | **SAFE** by the same reading. |
| Delete generalization claims (A2) | none | **SAFE**. |
| Rewrite calibration sentence (A4.1) | not in abstract | **SAFE**. |
| Rewrite SHAP paragraph (A4.2) | not in abstract | **SAFE**. |
| Rewrite Related Work / add citations (A1, A3) | none | **SAFE**. |
| Delete "balanced … across states" (line 117) and the REM claim (line 379) | 81.2% macro recall | **SAFE**. The macro figures remain literally true whatever the per-class breakdown shows. Only the *body's* interpretation of them is false. |
| Add LR row + majority floor (A7) | "outperforming all baseline methods" | **SAFE and helpful** — it substantiates a claim the body currently leaves unsupported. |
| Contributions list, Discussion, restructure (A8) | none | **SAFE**. |
| Retitle (A8.7) | — | Safe in itself, but does **not** resolve the item below. |

### 6.2 THE TWO THINGS THAT CANNOT BE FIXED BY WRITING — stated plainly

**(1) Cross-frequency coupling.** The frozen abstract claims "cross-frequency coupling metrics" as
one of three headline feature families. The title claims it. Lines 115 and 129 claim it. The
Methodology contains two feature subsections and defines no coupling quantity. And — decisively —
the paper's own Figures 3 and 4 **enumerate the model's inputs exhaustively** (complexity, mobility,
alpha/delta/gamma/beta/theta_power, activity, animal, mmd, entropy, eeg_data, plus raw sample
indices), and **not one of them is a coupling measure**. No modulation index, no phase-amplitude
coupling, no PLV, no comodulogram statistic.

The evidence in the submission indicates that a headline feature claimed in the abstract was never
in the model. If that is right, **there is no writing-only fix.** The three options are:

- **(a)** The authors find a CFC computation in their code that simply never made it into the
  figures. Then: document it in Methods (phase band, amplitude band, coupling statistic, filter,
  bin count, per-epoch dimensionality), regenerate the figures from the model that contains it, and
  the abstract becomes accurate. **This is the only fully clean outcome.**
- **(b)** No CFC was computed, and the authors compute it and retrain (a Track B item that did not
  make the ranked list because it is contingent). The abstract becomes retroactively accurate.
- **(c)** No CFC was computed and none is added. Then the honest body contradicts the submitted
  abstract, and the title advertises a method component that does not exist. Deleting the body
  claims at lines 115 and 129 reduces the exposure to one sentence in a frozen abstract and one
  word in a title — but it does not eliminate it.

**This must be resolved before anything else in this plan is drafted**, because it determines
whether the submitted abstract is accurate. Do not invent a formula under any circumstance.

**(2) Removing the `animal` feature may move the headline number.** The abstract is frozen at 91.5%
/ 86.8% / 81.2% / 83.5%. If dropping the subject-ID column changes those figures — and it plausibly
will, since `animal` ranks 7th of 11 named features by mean |SHAP| — then the abstract no longer
describes the model the corrected paper reports.

The least-bad honest presentation: report 91.5% explicitly labelled as the figure obtained by the
originally submitted configuration (which included an animal-index feature), and the retrained
figure as the corrected result, and say so in one sentence. That keeps the abstract literally true
as a statement about a specific configuration. It is not comfortable, and a reviewer may still note
the discrepancy — but it is far better than shipping a headline number produced by a model that
conditions on subject identity.

---

## 7. Realistic ceiling — my honest assessment

### 7.1 With Track A alone: no, this is not competitive at AAAI-27

Track A is worth every hour it costs, but it will not get this paper accepted, and I would rather say
so than imply otherwise.

What Track A achieves is real: it converts a manuscript that would be **desk-rejected at Phase 1 by
any reviewer who resolves one DOI or opens one of its own figures** into an honest, well-written,
correctly-cited applied paper. That is a large change in kind. It is not a large change in acceptance
probability, because after Track A the paper still has:

- no subject-independent performance number of any kind;
- no per-class results on a three-class problem whose entire narrative concerns the 8% class;
- a baseline set consisting of two models the authors wrote themselves;
- one small private cohort against a field norm of three to five public datasets;
- no ablation, no variance, no significance test;
- and a contribution that, stated honestly, is "standard engineered features plus an off-the-shelf
  boosted-tree ensemble, applied competently to one dataset."

Those map directly onto the two AAAI reviewer questions that decide applied papers — *"Do the
empirical results really support the claims of the paper?"* and *"Does the empirical evaluation
include appropriate baselines and comparisons?"* — and Track A cannot answer either. Add the
structural context: AAAI-26 ran ~29,000 submissions at a 17.6% accept rate with a Phase-1 kill and
**no rebuttal for papers that fail it**. Every fix has to land in the submitted PDF. Under those
conditions I would put Track A alone at **low single-digit acceptance probability**.

### 7.2 With Track A + B1 + B2 + B3: near, probably slightly below, the base rate

Those three experiments — drop the subject-ID feature, run leave-one-animal-out, emit per-class
metrics and κ — cost perhaps two days combined and change the paper's character completely. The
protocol ladder (random / block / LOAO) is genuinely interesting to the rodent-EEG community, and
the honest gap between epoch-wise and subject-wise numbers is a more publishable finding than 91.5%
ever was. Add B5 (calibration under the clean protocol) and the calibration contribution becomes
defensible rather than decorative.

Even so, the reviewer's first question still gets answered with "a careful application of standard
components," and AAAI-27's CFP explicitly prefers papers that "explore new territory or point out
new directions" over ones that advance the state of the art incrementally within a narrow area. I
would estimate **10–15%** — around or slightly below the venue base rate. Worth submitting; not
worth expecting.

### 7.3 The strategic recommendation

**Do Track A regardless of venue.** Items A1 and A4 are not competitiveness work — they are
correcting a fabricated description of a real published tool and a citation whose DOI points to
someone else's paper. Those must be fixed before this manuscript is sent anywhere, including to
arXiv in revised form.

**Treat AAAI-27 as the free option and IAAI-27 as the real target.** IAAI-27's deadline is
September 8, 2026 — six weeks after the AAAI main-track deadline — with an 8-page limit and no limit
on references or appendices. Its Emerging Applications category and its stated criteria ("design
rationale — the design decisions made and the alternatives considered," "measurable improvements,"
"deployment details," "lessons learned," with the explicit note that accounts reporting failures and
abandoned approaches are evaluated *more* favorably than success-only narratives) are close to a
description of what this paper becomes after Track A and B1–B3. That is the opposite of the current
manuscript's success-only voice, and the six-week gap is exactly enough to run B1, B2, B3, B5 and
B7. Both submissions benefit from the same work.

**Do not move to the AI for Social Impact track.** Its rubric is friendlier on novelty but scores
"quality of evaluation," "facilitation of follow-up work," and "overall scope and promise for social
impact," and it explicitly invites data-collection methodology, stakeholder modelling, and field
tests. A preclinical scorer with no deployment, no lab partner, and no field evaluation would trade
one rejection reason for a worse one. (Its abstract deadline has passed anyway.)

### 7.4 The one thing that would change my answer

If the LOAO number comes back materially below 91.5% and the authors report it prominently — framing
the paper around *the gap between subject-dependent and subject-independent evaluation in rodent
sleep scoring*, with the three-rung leakage ladder as the central figure and calibration-under-shift
as the second result — that is a different paper with a genuine, transferable finding, and it would
be competitive on its own terms rather than as an application of XGBoost. That reframing is fully
consistent with the frozen abstract (91.5% remains true as the epoch-wise figure) and it is the
single highest-return path available. It is also, notably, exactly the structure of the accepted
AAAI sleep-staging work: SleepDG's contribution is not a better classifier, it is the *task* of
generalizable sleep staging.
