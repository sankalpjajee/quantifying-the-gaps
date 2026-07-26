# PersonaEval / MatrAIx — Editorial Working Document
**Assembled from: 9 reviewer lenses → citation verification → 3-judge merit panel → 3-team red team.**
**Status: single decisive plan of record. Nothing below is a menu.**
**Date: 2026-07-26. Source read directly at `/home/user/quantifying-the-gaps/COLM_2026_SocialSim`.**

---

## 1. Bottom line

**No. This is not a top-tier main-track paper today, and it is not close.**

The single thing that decides it is not any of the ~120 filed defects individually. It is this:

> **The paper reports no evidence that any number it emits bears any relation to the application under test.**

Every quantity in the paper is a self-report by the same persona-conditioned model instance that produced the behavior (`body/99Appendix.tex:65-69`, live: *"the same persona-conditioned instance later completes the post-interaction form in character, so the evaluation reflects the assigned user's point of view rather than an external judge"*). The one metric offered as validation is at ceiling and its provenance is contradicted between the live text, the live appendix, and a commented TODO. There is no control arm, no repeat, no interval, no test, and no application whose quality is known independently of what the simulator says about it.

The remediation program the reviewers assembled — reruns, ICCs, control arms, judge decoupling, permutation tests, legible figures — establishes **reliability**. The paper's own worst result proves reliability and validity come apart: the Web arm is perfectly reliable (`body/99Appendix.tex:513` — Overall 0.79±0.03, Goal 1.00±0.00, steps 3.0±0.0) and completely uninformative. Optimizing reliability is optimizing the quantity the paper's own table shows is not the binding one.

**But the fix is reachable, cheap, and does not require a single human participant.** It requires one experiment nobody prioritized correctly and one result that is already sitting, unremarked, in a table the authors printed themselves:

**Across the four chatbot applications, mean overall satisfaction is a perfect rank function of interaction volume and a perfect rank *inverse* of task completion.** Computed directly from `body/99Appendix.tex:505–511`:

| Application | Overall | turns | grounded items | MaxTurn | Done |
|---|---|---|---|---|---|
| Chatbot-Finance | **0.64** (1st) | 7.9 (1st) | 28.6 (1st) | 0.92 (1st) | 0.08 (4th) |
| Chatbot-Movie | **0.60** (2nd) | 6.6 (2nd) | 15.2 (2nd) | 0.26 (2nd) | 0.74 (3rd) |
| Chatbot-Beauty | **0.59** (3rd) | 6.1 (3rd) | 8.3 (3rd) | 0.16 (3rd) | 0.84 (2nd) |
| Chatbot-Medical | **0.57** (4th) | 5.2 (4th) | 0.0 (4th) | 0.00 (4th) | 1.00 (1st) |

ρ(Overall, turns) = **+1.00**. ρ(Overall, grounded items) = **+1.00**. ρ(Overall, MaxTurn) = **+1.00**. ρ(Overall, Done) = **−1.00**. Exact two-tailed permutation *p* = 2/24 = **0.083** at n=4 — suggestive, not conclusive, and testable at the run level today with n=200.

The application that never terminates (Finance: 92% hit the 8-turn cap, 8% complete) receives the **highest** satisfaction. The application that always terminates by user decision (Medical: 100% Done, zero grounded items) receives the **lowest**. If that survives run-level testing, the instrument measures how much text the application emitted, not whether it worked — and that is a real, novel, publishable measurement-validity result about self-report user simulators, obtained for free.

**The decision:** run one experiment that can falsify the instrument (a known-groups quality manipulation, ~$20 and half a day of engineering), report what it says, and build the paper around admissibility conditions rather than around a wrapper. Do that and this is a defensible main-track submission. Skip it and no amount of writing, statistics, or figure work changes the outcome, and the honest destination is a system-demonstration track where this artifact is genuinely strong.

---

## 2. Recommended framing — one choice

### 2.1 The framing

**Title (commit to this):**

> **What Does a Simulated User Measure? Admissibility Conditions for Persona-Based Evaluation of Interactive Applications**

**The thesis, in one falsifiable sentence:**

> Persona conditioning shapes what a simulated user *says* and not what it *scores*; the score instead tracks interaction volume rather than task success; therefore simulated-user self-report is not admissible as evaluation evidence without the three checks we define, and the systems in current use do not pass them.

**Why this framing and not the alternatives.**

The judge panel unanimously nominated `headline-diversity-outcome-dissociation` as the reachable reframe, and it is the right *mechanism*. All three red teams attacked it as a *headline*, and they are right on three counts:

1. **N=1 model.** A phenomenon claim ("persona conditioning produces persona-shaped language with application-shaped scores") supported by one simulator, one prompt template, one corpus, one temperature is a claim about Claude Haiku 4.5. Meanwhile the record downgrades the model-swap ablation to merit 2.5 and "skip this cycle." That is an internal contradiction: it ships a phenomenon claim with the ablation phenomenon claims require explicitly deprioritized.
2. **It is already published.** `wang2024flatten` (within-group diversity flattening) and `west2025priceofformat` (format-induced diversity collapse) are in this paper's own `.bib` and `truncation-drops-all-failure-modes` correctly orders them restored to live text (`body/02related_work.tex:12`, currently commented). Implement both and Section 2 will state that Section 1's headline is a documented property of persona-conditioned LLMs.
3. **The compression has no denominator.** The dispersion leg rests on d̄ ≈ 0.82–0.83 mean pairwise TF-IDF cosine (`body/99Appendix.tex:866-867`), which `diversity-claim-no-null` correctly says is what any two short sparse documents give. The outcome leg rests on a normalization the authors' own commented TODO says they could not verify (`body/99Appendix.tex:796-797`) and on a chatbot rating scale the prompt never pins (`body/99Appendix.tex:327-328`: `"overallRating": "<integer score on the task-specific rating scale>"`).

The admissibility framing absorbs the dissociation as its mechanism, is not owned by prior work, is falsifiable, generalizes past this system, and — critically — is the frame under which **every currently-embarrassing table row becomes corroborating evidence**: the ceiling'd judge, the 92%-censored Finance arm, the zero-grounding Medical arm, and the 3.0±0.0-step Web arm all become instances of the same failure the paper is measuring.

### 2.2 The three admissibility checks (this is the contribution)

Commit to exactly three named checks, report pass/fail per application, and put the criterion check in the headline slot:

| Check | Question | Estimator | Status in current draft |
|---|---|---|---|
| **C1 Reliability** | How much of the observed spread is decoding noise? | Within-persona SD across R≥5 reruns, in **raw scale points**; variance partition only where both components are non-degenerate | Unestimable — one draw per cell at temperature 0.7 (`body/99Appendix.tex:437`) |
| **C2 Criterion sensitivity** | Does the score rank applications of *known-differing* quality? | Spearman ρ between simulated mean rating and constructed quality tier; effect expressed in C1 noise-floor units | **Never attempted. This is the paper.** |
| **C3 Persona grounding** | Is the assigned persona recoverable from the simulated user's own output, above a lexical-copy baseline? | Top-1 re-identification from stored `ratingReason` / `answerRationale`, with permutation null and profile-token-scrub ablation | Asserted six times by hand ("Trace signal:", `body/99Appendix.tex:554,593,637,682,720,768`), never measured |

**C2 is load-bearing. C1 is its denominator. C3 is what makes the failure interesting rather than merely a broken system.**

### 2.3 Ready-to-paste abstract

Two rules before you use it. **(a)** Every number is a placeholder marked with `\NUM{}`, which renders visibly in red so nothing ships unfilled. **(b)** Do **not** reinstate "92–100% of runs judged consistent with the assigned persona" from `abstract-overclaim-and-rewrite` — Judge 2 flagged this as an internal contradiction and is correct: it is the saturated, provenance-unknown metric that two other upheld findings say must not be reported as evidence. This abstract cannot be written until C2 returns a number.

```latex
% Put this in the preamble so unfilled numbers are impossible to miss:
% \newcommand{\NUM}[1]{\textcolor{red}{\textbf{[#1]}}}

\begin{abstract}
Simulated users are increasingly proposed as a cheap substitute for user
studies of interactive applications, but it is rarely established that a
simulated user's output measures the application rather than the simulator.
We treat persona-based user simulation as an instrument and ask what evidence
licenses its use. We define three admissibility conditions --- reliability
across reruns, sensitivity to application quality differences of known sign,
and recoverability of the assigned persona from the simulated user's own
output --- and we instantiate them in \method, a persona simulation harness
that drives one persona population through survey, multi-turn chatbot, and
browser applications under a shared trace and evaluation schema.
Across \NUM{K} applications, \NUM{M} simulator model families, and
\NUM{N} runs, we find a dissociation: the assigned persona is recoverable
from the simulated user's free text at \NUM{X}\% (permutation null
\NUM{Y}\%), while the same runs' numeric ratings are near-invariant to the
persona and, in the multi-turn setting, do not order deliberately degraded
applications above working ones
($\rho = \NUM{r}$, 95\% CI \NUM{[lo, hi]}; rerun noise floor
\NUM{s} scale points). Simulated satisfaction instead tracks interaction
volume rather than task success ($\beta_{\text{volume}} = \NUM{b}$,
$\beta_{\text{completion}} = \NUM{b2}$). We conclude that persona-conditioned
self-report is admissible for eliciting user-shaped \emph{language} and is
not admissible as a quality signal without an external outcome, and we
release the harness, the protocol, and all traces.
\end{abstract}
```

**Fallback abstract, only if C2 does not run.** This is the demo-track abstract. Label it that way internally and submit it to a demo track; do not dress it as main-track.

```latex
\begin{abstract}
Real user studies are important for understanding how people interact with
interactive systems, but they are costly, slow, and hard to scale during
iterative development. We present \method, a persona simulation harness that
connects simulated users instantiated from existing persona datasets to
heterogeneous applications through task adapters, and records interaction
trajectories, outcomes, and post-interaction evaluation forms in a shared
schema. We instantiate \method on six applications spanning three interaction
protocols: structured surveys, four multi-turn chatbots, and a browser-based
shopping task. We report 400 runs with full prompts, adapter schemas, and
traces released. We make no claim that the resulting scores approximate human
judgments; we report the trajectories and outcomes as produced, together with
the conditions under which they are and are not interpretable.
\end{abstract}
```

### 2.4 Ready-to-paste contribution bullets

Replace `body/01Introduction.tex:8-12` wholesale. The current two bullets are artifact statements — *"A plug-and-play persona simulation system"* and *"A multi-application demonstration"* — and neither is a proposition a reviewer can accept or reject.

```latex
Therefore, we ask what evidence licenses treating a simulated user's output as
evaluation evidence, and we introduce \method (\autoref{fig:personaeval}) as
the instrument for answering it. Our contributions are as follows:
\begin{itemize}
    \item \textbf{Three admissibility conditions for simulated-user
    evaluation.} We define reliability (variation across reruns of the same
    persona), criterion sensitivity (whether the simulated score orders
    applications whose relative quality is fixed by construction), and persona
    grounding (whether the assigned persona is recoverable from the simulated
    user's own output above a lexical-copy baseline), and we give an
    estimator, a null distribution, and a reporting standard for each.

    \item \textbf{A measured dissociation between simulated language and
    simulated scores.} Across \NUM{K} applications and \NUM{M} simulator model
    families, persona is recoverable from the simulated user's free text at
    \NUM{X}\% against a \NUM{Y}\% permutation null, while the same runs'
    numeric ratings vary by \NUM{Z} and do not separate deliberately degraded
    applications from working ones. Simulated satisfaction instead tracks
    interaction volume rather than task success.

    \item \textbf{An open harness and protocol.} \method\footnote{Code, traces,
    and adapter specifications: \url{<anonymized>}.} drives one persona
    population through survey, multi-turn chatbot, and browser applications
    under a shared trace and evaluation schema, with all prompts, adapter
    interfaces, and \NUM{N} raw run records released so that the admissibility
    conditions can be recomputed for other simulators.
\end{itemize}
```

Note the ordering: the protocol is the third bullet, not the first. Under this framing the harness is the method section, not the contribution.

---

## 3. Blocking issues — must fix before any top-tier submission

Ordered by execution sequence, not by severity. Items B0–B3 are day-1 gates: everything downstream fails or changes if they resolve badly.

| # | Issue | Anchor | Fix (concrete artifact) | Effort | Panel merit |
|---|---|---|---|---|---|
| **B0** | **De-anonymization risk in source.** Real author names, institutional emails, and a "Team Leader & Correspondence" line sit in comments directly above a template `\author{}` block that still reads "Antiquus S. Hippocampus." The compiled PDF *is* anonymous (`[submission]` option suppresses `\@author`), so this is latent, not live — but several other recommendations urge uploading the source tree as supplementary material, which would deanonymize the team. | `colm2026_conference.tex:85-94`, `:96-114` | Delete lines 85–94. Move the real author list *inside* `\author{}` where `[submission]` suppresses it. Strip the abandoned "Matraix" name (`colm2026_conference.tex:79`). Add a pre-upload gate: `grep -rniE '(yuexing\|xiaominli\|@mit\.edu\|@g\.harvard\.edu\|TODO\|Matraix)' *.tex body/ tables/` must return zero before any archive is built. | 5 min | 2.33 (mis-triaged; red team ranks it highest severity-per-minute) |
| **B1** | **Data inventory.** Roughly fifteen upheld recommendations are premised on "computable today from data already on disk." Nothing in this repo establishes that per-run records exist. This directory contains only `.tex`, `.bib`, `.sty` and nine baked figure PDFs. | n/a — external | Confirm per-run records carry: `persona_id`, `application`, **raw** item scores (pre-normalization), all free-text rationale fields (`ratingReason`, `answerRationale`, `constraintRationale`, `preferenceRationale`, `clarificationRationale`), `action_history`, `selectedProductId`, `orderId`, turn counts, grounded-item lists, per-run judge outputs if any, and per-persona selection similarity scores. Publish the answer to the team **before scheduling anything else.** If the raw scores and rationales are gone, the reframe is not carryable and the paper is frozen at its current numbers. | 2 h | — (red team addition) |
| **B2** | **Judge provenance contradiction.** `body/05results.tex:43` (live): *"We evaluate persona alignment with a human-judged 1--5 score."* Contradicted by live appendix `body/99Appendix.tex:65-69` (*"rather than an external judge"*) and by commented TODO `body/99Appendix.tex:821-823`: *"paste the exact LLM-judge rubric/prompt... The released app does not commit a fidelity judge, so confirm the per-axis scale... and the judge model."* Six lenses found this independently. It reads as misrepresentation, not sloppiness. | `body/05results.tex:43`; `body/99Appendix.tex:65-69`, `:810-825` | **Time-box to 2 hours, then act on a pre-committed rule.** If a dated judge model + rubric + stored per-run judge outputs are all three in hand by hour 2: keep, name the snapshot, paste the rubric live, report the full 1–5 histogram with ceiling rate. **Otherwise delete** the Persona/Task/SimQ columns from `tab:app-aggregate-metrics`, delete panel (c) of `fig_results_components`, delete the paragraph at `body/05results.tex:41-43`, and paste: *"We do not report a persona-alignment score. The judge assigns the maximum on all Chatbot-Finance runs (1.00±0.00) and varies only between 0.92 and 1.00 across all six entries; a metric with that dynamic range cannot support a fidelity claim."* **Do not run a human annotation study** — see §8. Strip every TODO from `.tex` regardless. | 2 h | 5 |
| **B3** | **Normalization undefined and demonstrably wrong.** "Normalized" appears 7× in live prose and on 4 axis labels; the only definition is commented (`body/99Appendix.tex:787-797`) and carries the authors' own TODO doubting it. Two independent reviewers recovered the plotted values from the vector paths: the 50 Chatbot-Beauty ratings land on exactly six values, all multiples of 1/7 (counts 10/13/8/7/3/9, weighted mean 0.591, SD 0.249 — reproducing `body/99Appendix.tex:507`'s "0.59±0.25" exactly). The map is **divide-by-max**, not the commented `(r-1)/9`. | `body/99Appendix.tex:472`, `:787-797`; `body/05results.tex:20,30` | **Do not hunt for the plotting script.** Adopt the recovered map as ground truth. Publish as a three-column live table (metric, raw scale, map to [0,1]) with the divide-by-max form written explicitly, plus one sentence: *"the worst attainable rating maps to 1/7 rather than 0, so reported normalized means are inflated relative to a floor-corrected map."* Blocks the dissociation ratio, the leniency gap, and every dispersion comparison. Delete both TODOs. | 30 min | 3.67 |
| **B3b** | **Chatbot rating scale never stated.** `body/99Appendix.tex:327-328` specifies `"overallRating": "<integer score on the task-specific rating scale>"` — the prompt never pins it. Traces show /7 (`:589` = 2/7, `:677` = 4/7, `:715` = 5/7); the web form is 1–10 (`:416-419`); survey is 1–5 (`:178`). If the scale differs **across** the four chatbot entries, those four rows are not mutually comparable and every cross-application dispersion claim collapses. | `body/99Appendix.tex:327-328`, `:416-419`, `:178` | Read the per-application `overallRating` scale out of the adapter config. Print it per application in `tab:settings`. Confirm the four chatbot entries share one scale. **This is a precondition for B3 and for the dissociation claim.** | 30 min | 3.67 (bundled) |
| **B4** | **Medical grounding contradiction — treat as a bug report, not prose.** `body/99Appendix.tex:511`: Chatbot-Medical reports `Task 1.00±0.00` alongside `grounded items 0.0±0.0` across all 50 runs. A grounding metric that returns the maximum on runs that grounded nothing is not reading the grounding record — and it returned values on the other five applications too, so the entire Task column is suspect. Compounding: Finance reports `Persona/Task/SimQ 1.00±0.00` on conversations that were cut off mid-task in 92% of runs. | `body/99Appendix.tex:509`, `:511` | Determine what computes Task grounding, on what input, and why it returns 1.00 on zero-grounding and on truncated runs. Report the finding whatever it is. **Sequence after B2**: if the judge columns are deleted, only the `grounded items = 0.0` question remains (likely a field-name mismatch against an adapter returning no item list) and it is a 30-minute grep. **Do not write the Ethics paragraph conceding "no clinician reviewed any transcript" until this is diagnosed** — the two side by side on one page are the worst pairing in the paper. | 1–3 h | 3.33 (underrated) |
| **B5** | **No criterion evidence — the C2 experiment.** No arm anywhere establishes that the score responds to application quality. | new work | See §4, item **H**. This is the paper. | 0.5 d eng + $20 | 4.67 (as `self-evaluation-circularity`) |
| **B6** | **No repeats.** One stochastic draw per (persona, application) cell at temperature 0.7; the persona effect is perfectly confounded with decoding noise, and every SD in the paper is uninterpretable. | `body/99Appendix.tex:437`, `:505-513` | R=5 on the four non-degenerate applications. **Note: "recorded seeds" is not executable** — the Anthropic Messages API exposes no seed parameter and OpenAI's is best-effort. Replace with: *"Neither provider guarantees deterministic decoding; we record request IDs, full decoding configuration, and API access dates, and report across-repeat resampling variance rather than seed-level reproducibility."* **Do not compute a variance ratio on Web** (both components < 0.001; the ratio is unstable and vacuous). Report within-persona SD in **raw scale points**. **Do not put an ICC in the abstract.** | overnight + $80–150 | 5 |
| **B7** | **No persona control.** Nothing distinguishes "persona conditioning drives this" from "a generic LLM user simulator produces this." The framework's founding premise is untested. | `body/05results.tex:5`; absent from `tab:app-aggregate-metrics` | Arms A (persona block removed from `body/99Appendix.tex:80`) and B (personas from a different application's panel). **Run on Chatbot-Beauty and Chatbot-Movie** (SD 0.25 and 0.25 — the two entries with estimable variance), **not on Web** (SD 0.03; cannot show an effect under any hypothesis and its inclusion guarantees a headline null). Report as bootstrap CI on the SD ratio, not a p-value. Record how many lines arm B required — free evidence for the plug-and-play claim. | overnight + $40 | 4.67 |
| **B8** | **WebArena misattribution.** Five live places attribute the web environment to WebArena and cite it; the adapter drives a bespoke store named "Northstar Home Goods" with invented product ids (`desk-002`). | `body/04application.tex:23`; `body/99Appendix.tex:380-381`, `:391`, `:452`, `:729`; `tables/application.tex:25` | Rename in all five live places to *"a sandboxed e-commerce site (\emph{Northstar Home Goods}, N products across M categories) modeled on WebArena's shopping domain."* Drop `\citep{zhou2024webarena}` from the `tab:settings` row; keep it as a design citation in prose. Delete every "-style" hedge — reviewers read it as an admission. **Do not run real WebArena** (2–3 weeks of infra, replaces a degenerate arm with an incompetence-confounded one, and does not fix the actual problem). **State catalog size and category count** — its absence is the leading unexplained cause of the 3-step degeneracy. | 30 min | 4.33 |
| **B9** | **No intervals, no tests, no correction.** ~61 quantities reported as bare mean±SD with explicit comparative claims drawn from them; binaries (Clarify/Done/MaxTurn/Goal/Complete/Schema) reported as mean±SD; `grounded items 28.6±29.6` reported as mean±SD. | `body/99Appendix.tex:503-516`; `body/05results.tex:37,40` | Declare **one** primary endpoint: **the C2 criterion effect** (Spearman ρ between simulated mean rating and constructed quality tier, pooled across manipulated applications, cluster-bootstrapped over personas). *Not* persona variance share — it is a reliability quantity, is unestimable on Web, and does not answer the reviewer's question. Secondary and labeled: variance partition, control contrasts, C3 accuracy. Wilson intervals on all binary columns; median[IQR] for skewed counts; error bars on every marker; Holm over a named handful. | 2 h (inside the figure pass) | 4 |
| **B10** | **No Limitations, no Ethics.** Zero occurrences of "limitation", "ethic", "IRB", "consent", "licen" in any `.tex`. | absent | §7 below, pasted verbatim. **Write it last**, after B2/B5/B6 resolve — a Limitations section written on day 1 will contradict the methods by day 10. | 2 h | 3.33 |

---

## 4. High-value additions — ordered by score-delta per unit effort

Cost estimates are order-of-magnitude and assume Haiku-4.5 / GPT-4o-mini pricing; chatbot runs dominate at roughly 30–40k persona-side input tokens plus ~3k output, i.e. ~$0.05–0.07/run all-in. **Money is not the constraint. Endpoint availability is** — the runs pin two model snapshots, and if either drifts or retires mid-cycle, every rerun-based item dies simultaneously and the old grid is no longer comparable. **Put all API work in one batch, early.**

### Tier 0 — Free, hours, no API. Do all of these first.

**A. Volume-not-success rank reversal.** *Highest delta-per-hour in the record.*
- **Anchor:** `body/99Appendix.tex:505-511`; claims it invalidates at `body/05results.tex:37`.
- **Recipe:** (i) Print the cross-application rank result exactly as in §1, with the exact permutation *p* = 0.083 and an explicit *n*=4 caveat. (ii) Test it where there is power: pool the 200 chatbot runs and fit `overallRating ~ turns + grounded_items + done + (1|persona) + (1|application)` on **raw** scores, reporting standardized coefficients with cluster-bootstrap CIs over personas. (iii) One sentence if it holds: *"Simulated satisfaction is a monotone function of interaction volume and is uncorrelated with, or inversely related to, task completion."*
- **Cost:** 3 hours. **Zero API.**
- **Why first:** this is a measurement-validity result about self-report user simulators generally, it is novel, and it is *better* than the panel's chosen headline because it is a direct instrument failure rather than a restatement of published diversity-collapse work.

**B. Survey singleChoice mode collapse.** *Panel ranked this near last; two red teams say promote to top-5. I side with the red teams.*
- **Anchor:** `body/99Appendix.tex:178-262` (six singleChoice items across three instruments, 5–6 options each: `purchase_driver` ∈ {comfort, style, price, brand_reputation, durability}; `adoption_barrier` ∈ {too_expensive, not_my_style, unsure_about_comfort, already_have_similar_shoes, prefer_other_brands}); trace at `:541,545`.
- **Recipe:** For each of the 6 items × 50 personas: option share, modal share, normalized entropy H/log₂(k), χ² against uniform. Put it in the headline figure beside the rating dispersion.
- **Cost:** 2 hours. **Zero API.**
- **Why:** this is the compression claim measured on a **discrete, encoder-free, assumption-free** outcome. It is immune to every objection that will be raised against embedding distances (scale-incomparability, length effects, encoder choice). If 50 maximally-dispersed personas concentrate on one `purchase_driver`, no reviewer can argue with it.

**C. Label-permutation test with k-selection inside the loop.**
- **Anchor:** `body/05results.tex:20,40`; `body/99Appendix.tex:889-892`, `:903`.
- **Recipe:** 10k permutations, re-running silhouette-based k-selection inside each shuffle, BH-FDR across the six applications. Pure numpy on 50 points — **this is one hour, not "medium effort."**
- **Cost:** 1 hour. **Zero API.**
- **Pre-commit to the outcome now.** With silhouette 0.09–0.21, groups as small as n=3 (Survey G4/G6) and n=6 (Beauty C2), and BH at m=6 requiring the smallest raw *p* < 0.0083 for the first rejection, the realistic outcome is that **all six die**. If fewer than two survive: delete Figure 2(b), delete the max–min annotation, delete every "because" sentence at `body/05results.tex:40`, demote `fig_beauty_persona_groups` to an appendix illustration with per-group *n* printed in the caption and an explicit note that group names were assigned after inspecting outcomes, and report one line: *"no application's persona-group spread exceeds its label-permutation null after correction (all p > X)."* That is a cleaner finding than the current claim. **Budget it as a deletion, not as a result.**

**D. Persona re-identification with lexical-copy control (C3).**
- **Anchor:** the six hand-written "Trace signal:" assertions at `body/99Appendix.tex:554,593,637,682,720,768`; stored fields at `:137`, `:317-335`, `:421-422`.
- **Recipe — implement as embedding retrieval, not LLM forced-choice.** Rank the true persona among the application's 50 by cosine between the rationale embedding and the profile embedding. ~350 embedding calls per application (minutes, cents), **not** 15,000 pairwise LLM calls (hours, $50–200). Report **three** numbers per application:
  1. raw top-1 accuracy;
  2. top-1 after scrubbing every content n-gram (n≥2) appearing verbatim in the persona profile from the rationale;
  3. a lexical-only baseline (TF-IDF overlap alone).
  The gap between (2) and (3) is the grounding claim. Report the **empirical within-application permutation null**, not 1/50 — the 50 personas were selected by similarity to the same description and are lexically clustered, so chance sits well above 2%. Wilson CIs throughout.
- **Cost:** half a day, cents. **This control is non-negotiable:** the persona profile is pasted verbatim into the system prompt (`body/99Appendix.tex:80` — `{persona_profile}`), so top-1 could hit 90%+ on pure string copying, and shipping the number without the ablation hands a reviewer a one-line kill. If (2) collapses to chance, **that is the stronger result** — persona conditioning produces surface copying, not behavioral variation, which *is* the thesis.

**E. Panel overlap matrix.**
- **Anchor:** `body/99Appendix.tex:24-26` ("336 curated profiles from four sources... the runs reported use the Nemotron-Personas-USA subset"); commented caption `:929-930` ("300 selected Nemotron personas"); overlap claim `:842-848`.
- **Recipe:** 6×6 persona-ID Jaccard matrix. 20 lines. Pre-commit both branches:
  - **High overlap** (likely — ~300 selections from a subset of a 336-profile bundle): you have a partially crossed design and can fit the crossed model at zero cost, **and** the inter-domain diversity claim at `:842-848` must be rewritten as a statement about **reuse**, and `fig1_overall_diversity` recaptioned or retired.
  - **Low overlap:** the cross-application dispersion sentences at `body/05results.tex:37` must be **withdrawn**, not conditioned — the panels are then different populations.
- **Cost:** 1 hour. Also settles `selection-not-recruitment` and `convenience-sample-not-population`, both of which currently assume a pool size the live paper never states.

**F. Web trace audits.**
- **Anchor:** `body/99Appendix.tex:513` (steps 3.0±0.0, Goal 1.00±0.00) vs. the four-step trace at `:742-757`; task prompt at `:399-401` ("compare at least two relevant options... add one product to the cart, complete the sandbox checkout, and reach the order confirmation page").
- **Recipe — two pure-counting audits only:** (i) distribution of `len(action_history)` and histogram of distinct action types — this resolves the 3-vs-4 step contradiction; (ii) cardinality and normalized entropy of `selectedProductId` across 50 runs. **Skip** the screenshot verification and the "compare at least two options" compliance rate — the latter requires hand-classifying an unknown action vocabulary and is half a day of undefendable heuristic.
- **Cost:** 1 hour, gated on B1. If `action_history` was not persisted, **cut the web arm from the results**, keep the web adapter in the system description as a capability, and reclaim ~0.5 pages.

**G. Survey 50×3 crossed decomposition — reported correctly.**
- **Anchor:** `body/99Appendix.tex:472` ("each persona's survey scores are averaged across the three instruments"), `:503` ("50 personas / 150 runs"), `:523` (three instruments: ChatGPT Images, Instagram Reels, Nike Air Max Dn).
- **The panel's framing is backwards and would put a self-harming number in the abstract.** These are three *unrelated products*. High cross-stimulus ICC does **not** mean "the persona is a reproducible respondent"; it means the persona's rating is invariant to the stimulus — a response-style artifact, which is the exact validity failure the paper is accused of. Low ICC is partly *correct behavior*: a real respondent should rate a running shoe and an image generator differently.
- **Recipe:** fit the crossed model on **raw** Likert values, report the three-component decomposition (persona / instrument / residual), and **lead with the instrument variance share**. Report it in the appendix as *"persona-level rating consistency across unrelated stimuli."* **Never call it reliability. Never put it in the abstract. Drop the D-study/Spearman-Brown extrapolation entirely** — projecting from k=3 heterogeneous stimuli treats stimulus as an exchangeable facet when it is the substantive variable. With 3 instrument levels, report the instrument component as a point estimate with an explicit caveat that 3 levels cannot estimate a variance component precisely, or fit instrument as a fixed factor.
- **Cost:** half a day. **Zero API.** If the persona share is high, use it as *supporting* evidence for compression: a persona that gives statistically indistinguishable ratings to a photo model, a short-video product, and a running shoe is exhibiting a leniency trait dominating stimulus content.

### Tier 1 — One API batch, days 2–3. Instrument it for telemetry before launching.

**H. C2 criterion-sensitivity known-groups experiment.** *Highest absolute value in the entire record. Do it regardless of where it ranks on delta-per-hour.*
- **Anchor:** new. Derived from `self-evaluation-circularity` (merit 4.67), promoted to primary by all three red teams.
- **Design — three tiers, not two.** A single binary contrast gives no ordering information, so a null is uninterpretable (was the manipulation too weak, or is the instrument dead?). Monotone dose–response is publishable even when a pairwise contrast misses significance.
  - **Chatbot-Beauty** (Overall SD 0.25 — best-powered): tier 1 = current RecAI retrieval; tier 2 = retrieval degraded to top-1; tier 3 = fixed generic catalog listing that ignores the query. *(If the RecAI retrieval path is not patchable in under 2 hours — check on day 1 — use the fixed-listing degradation for both degraded tiers rather than burning half a day in unfamiliar third-party code.)*
  - **Web** (SD 0.03 — the saturation case, and the environment the team built and therefore controls end to end): tier 1 = full catalog; tier 2 = persona-relevant category removed; tier 3 = search ranking randomized.
- **Scale:** 50 personas × 3 tiers × R=3 × 2 applications = 900 runs.
- **Pre-register**: the tier ordering and the primary endpoint (Spearman ρ between simulated mean `overallRating` and true tier; Cliff's δ for the tier-1-vs-tier-3 contrast; effect reported in **C1 noise-floor units**).
- **State the MDE honestly.** At n=50/arm, α=.05 two-tailed, 80% power, d = 2.80·√(2/50) = 0.56. Beauty: 0.56 × 0.25 = **0.14 normalized ≈ 1 raw point on a 7-point scale**. Web: 0.56 × 0.03 = **0.017 normalized ≈ 0.17 raw points on a 10-point scale** — a number that *sounds* precise and actually reflects a response distribution with 45/50 runs on one value, ≈0.47 bits of entropy out of an available 3.32.
- **Cost:** ~$20–60 and half a day of engineering. **Cheaper than every human tier in the reviewers' validation plans and worth more than all of them combined**, because it is the only design in the program that can falsify the paper's central proposition without recruiting anyone.
- **Publish the null loudly if it is a null:** *"A persona simulator whose satisfaction score cannot rank a working recommender above one that ignores the query is not an evaluation instrument."* That sentence is the paper.

**I. Persona controls (B7).** Arms A + B on Beauty and Movie. ~$40, overnight. See B7.

**J. Reruns (B6).** R=5 on the four non-degenerate applications. ~$80–150, overnight. See B6.

**K. Second simulator model family — promote from merit 2.5 to required.**
- **Anchor:** `body/99Appendix.tex:437` (Claude Haiku 4.5 throughout).
- **The panel and the red team disagree sharply here. I side with the red team.** The panel downgraded this to "skip this cycle"; two red teams point out that the panel simultaneously promoted a *phenomenon* claim to the headline, and a phenomenon claim established on one model, one prompt template, one corpus, and one temperature is a claim about Haiku 4.5. That is an internal contradiction.
- **Recipe:** one additional simulator family (a GPT-class model) on Chatbot-Beauty and Web, 50 runs each. **Pre-register the endpoint as the compression/criterion statistics, not the persona-group ordering** — the clusters are being deleted anyway (item C), so a Spearman on ~5 group means answers a question you have withdrawn. Report per family: outcome dispersion, C3 re-identification accuracy, and the C2 tier effect. If the dissociation replicates across families it is a property of persona simulation; if not it is a Haiku artifact and must be reported as one — still publishable, and far better than being told so by a reviewer.
- **Cost:** ~$10, in the same batch.

**L. T=16 Finance rerun.** 50 runs, ~$8. Settles whether the paper's *highest* satisfaction score (0.64) is an artifact of interruption — 92% of Finance runs hit the 8-turn cap (`body/99Appendix.tex:509`).

**M. Cost/latency telemetry — use-it-or-lose-it.**
- **Anchor:** `body/00Abstract.tex:2,10`; `body/01Introduction.tex:3`. The paper's motivating premise is that user studies are too costly; it reports no cost, token, wall-clock, or throughput number anywhere.
- **Instrument the batch before it launches:** per-run input/output tokens per provider, wall-clock, retry count, parse failures. This **cannot be reconstructed afterward**, and "include failed/retried runs in the cost" is impossible retroactively.
- **Report cost only adjacent to the C2 result, and make the argument about iteration latency and multiplicative scaling, not per-respondent price.** "$2 of simulation versus $120 of Prolific" is a 60× claim a reviewer answers in one line: *$120 buys real humans and settles the validity question your paper cannot.* The defensible form is: *"Detecting a quality-tier difference of size D costs $X and Y hours; the same panel across 6 applications × R reruns × k system variants costs the human study 6·R·k times more."* If you will not measure it, delete "scalable" and "parallelizable" from `body/00Abstract.tex:10`.

### Tier 2 — Free archival anchors (≤1 week, $0, no IRB)

**N. Cui+2024 effect-size recovery on the survey block.** `Cui+2024` is at `colm2026_conference.bib:11-17` and is cited live only in the appendix (`body/99Appendix.tex:8`). Run the survey block's pre-registered contrasts and report recovery against the published 73–81% band. **Frame it as the standard you have not yet met and are adopting as protocol — do not frame it as a benchmark to beat.** The survey instruments are 4-item marketing questionnaires with essentially no main effects to recover; reporting 55% against a published 73–81% band writes "our system underperforms 2024 work" into the paper as a number.

**O. PRISM — distributional only, not paired.** The reviewers' paired per-person design is **invalid as specified**: PRISM ratings are of the specific model endpoint each participant conversed with (a heterogeneous set, most now deprecated or drifted), so replaying prompts into GPT-4o-mini and comparing to that human's rating confounds simulator fidelity with system-under-test identity. It also requires a new persona renderer for PRISM self-descriptions. **Recipe:** compare *marginal* rating distributions and report the **variance ratio σ_sim/σ_human** — the diagnostic that survives the confound and the one the current zero-variance columns will fail. Frame as evidence for compression, not as fidelity validation. State the transfer assumption explicitly in text.

---

## 5. Correctness and consistency defects in the source

### 5.1 Internal contradictions (the paper contradicts itself, in text a reviewer will read)

| Defect | Location A | Location B | Resolution |
|---|---|---|---|
| **Human vs. LLM judge** | `body/05results.tex:43`: *"a human-judged 1--5 score"* | `body/99Appendix.tex:65-69` (live): *"rather than an external judge"*; `:821-823` (commented): *"TODO: paste the exact LLM-judge rubric... The released app does not commit a fidelity judge"* | **B2.** Delete-by-default after a 2-hour time box. |
| **WebArena vs. Northstar** | `body/04application.tex:23`, `body/99Appendix.tex:380-381`, `:452`, `:729`, `tables/application.tex:25` | `body/99Appendix.tex:391` (`"websiteName": "Northstar Home Goods"`), `:743-753` (`ModDesk Compact $249`, `FocusDesk Pro $429`, `product id desk-002`) | **B8.** Rename all five; state catalog size. |
| **3 steps vs. 4 steps** | `body/99Appendix.tex:513`: `steps 3.0$\pm$0.0` | `body/99Appendix.tex:741-757`: Step 1, Step 2, Step 3, **Step 4** ("completes sandbox checkout and reaches an order confirmation page") | Item **F** resolves it via the action-history histogram. If unresolvable, define "step" explicitly in the table caption and flag the discrepancy — a zero-variance step count contradicting the paper's own worked example reads as a logging bug and generalizes to distrust of every other number. |
| **Task grounding = 1.00 with zero grounded items** | `body/99Appendix.tex:511`: `Task 1.00$\pm$0.00` … `grounded items 0.0$\pm$0.0` | — | **B4.** Bug report. Scope to the whole column, not the Medical row. |
| **Highest satisfaction on the most-truncated application** | `body/99Appendix.tex:509`: Finance `Overall 0.64` (highest of four), `MaxTurn 0.92`, `Done 0.08`, and `Persona/Task/SimQ 1.00±0.00` | `body/05results.tex:37,40` discuss neither | Item **A** + turn-cap disclosure (§5.3). |
| **Normalization: commented `(r-1)/9` vs. plotted `r/max`** | `body/99Appendix.tex:791-795` (commented) | Recovered vector values on a k/7 grid | **B3.** |
| **`fig_results_components` panel (a) plots three incompatible ordinal grids on one "Normalized rating" axis** | `body/05results.tex:17-21` | Recovered: four chatbot columns on k/7; Survey on k/5 with **n=150** un-averaged instrument-level points; Web taking only {0.70, 0.80} with 45/50 at 0.80 | Replace with a 100%-stacked ordinal bar per application, raw levels printed, axis from 0, native scale under each name ("Chatbot (7-pt)", "Survey (5-pt)", "Web (10-pt)"). Resolve the *n* inconsistency by **deciding**: plot the 150 instrument-level points, state n=150 = 50 personas × 3 instruments in the caption, and note that the per-persona averaging at `body/99Appendix.tex:472` applies only to persona-level analyses. |
| **OASIS listed as a persona source and as a simulation framework in one paper** | `body/99Appendix.tex:24-26` (persona source) | `body/02related_work.tex:15` (simulation framework) | 30-second edit. Drop OASIS from the persona-source list or explain what an "OASIS persona" is. Reads as unfamiliarity with the cited systems. |

### 5.2 Undefined quantities shipped as results

- **Persona / Task / SimQ** appear in `tab:app-aggregate-metrics` for all six applications (`body/99Appendix.tex:503-513`) and are defined **nowhere** in the compiled paper. Their only rubric is commented (`:810-825`). **No live evaluation form contains a persona-alignment field**: `box:chat-eval` (`:313-338`) has only `constraintSatisfaction`/`preferenceSatisfaction`/`overallRating`/`clarificationUseful`; `box:survey-schema` (`:126-167`) has `answers`/`trajectory`/`validation`; `box:web-eval` (`:407-431`) has `goalCompleted`/`orderId`/`selectedProductId`/`selectedProductName`/`needSatisfaction`/`easeOfUse`/`informationQuality`/`overallRating`/`ratingReason`. So the numbers cannot come from the documented instruments. → **B2.** An undefined metric is worse than a missing one.
- **Embedding model, similarity metric, embedded field set, and candidate pool size** for persona selection — the step that determines every number in the paper — are named nowhere. `body/05results.tex:5` and `body/99Appendix.tex:32-36` say only "embedding similarity." Add to `tab:settings`; print the verbatim application-description query strings in a promptbox (**these queries are the test-set definition**); release the selected persona IDs per application.

### 5.3 Confounds that invalidate specific live sentences

- **Turn-cap censoring.** `body/05results.tex:5` caps chatbot conversations at 8 turns. Finance: `MaxTurn 0.92`, `Done 0.08`, `turns 7.9±0.3`. Medical: `MaxTurn 0.00`, `Done 1.00`, `turns 5.2±1.2`. The per-application means at `body/05results.tex:37` are therefore not commensurable — Finance users are rating an interrupted interaction.
  **Fix — report, do not subset.** Print MaxTurn beside every satisfaction number and add a censoring-status covariate. **Do not apply the reviewers' run-validity filter as written**: "terminates by user decision AND grounds ≥1 item" leaves Finance at n=4 and Medical at **n=0**, destroying two of six applications. Use *termination-by-user-decision alone* as an admissibility criterion, report filtered and unfiltered side by side with the exclusion rate per application, and state plainly that Finance is unanalyzable under it — which is itself the finding. **Drop the survival-analysis apparatus** (see §8).
  **Free by-product worth a sentence:** `Clarify` ranges 0.06 (Medical) to 0.88 (Finance) under the *identical* shared evaluation form (`body/99Appendix.tex:340-342`: *"The finance and medical tasks currently use the same shared chatbot evaluation form"*) — strong evidence the form measures application behavior rather than user experience.
- **Nested, not crossed.** Each application faces a different similarity-selected 50-persona panel (`body/99Appendix.tex:32-36`), so persona and application effects are structurally confounded and the cross-application comparisons at `body/05results.tex:37` compare populations, not systems. → item **E** decides which branch applies.
- **Selection is a relevance filter described as recruitment.** `body/99Appendix.tex:34`: *"This approximates recruiting users whose background matches the application."* `body/05results.tex:5`: *"to approximate the selection of real users."* Rewrite both to *"an application-relevant convenience panel"* and state what inference it does and does not license. **Verify the actual Nemotron-USA subset size before writing "near-exhaustive" into Limitations** — the live paper never states it, and conceding a near-exhaustive pool is devastating if true and gratuitous if the runs drew from the full corpus.
- **Diversity claim has no null.** d̄ ≈ 0.82–0.83 (`body/99Appendix.tex:866-867`) supports the live claim at `:869-870` that each application is *"evaluated by a diverse set of users rather than a single collapsed persona type."* Near-orthogonality is what any pair of short sparse TF-IDF documents gives. Bootstrap 1000 random 50-persona draws and report the percentile — **but note the collision with item E**: if the pool is ~336 and 50 are selected, a "random 50" draw and the "selected 50" are nearly the same set and the null is weak by construction. State the selection ratio explicitly and say so.

### 5.4 Bibliography and build

- **`colm2026_conference.bib:19` reads `@article@article{SSR+2025,`** — a malformed header. **Run a clean `latex + bibtex` build today as a smoke test.** No LaTeX toolchain is present in this environment and no built PDF exists in the repo; an unbuildable source discovered on the deadline is the only truly fatal item on this list, and nobody scheduled a build check. The entry is uncited so BibTeX may skip it silently, but fix it in 30 seconds regardless.
- **Five entries carry `author = {Anonymous}`** (lines 276, 342, 382, 408, 585: `tan2025populationaligned`, `west2025priceofformat`, `joshi2026beyondcooperative`, `zhou2025llmeconomist`, `horizonbench2026`), and reviewers report the keys name people who are not the papers' authors. All five are currently **uncited**, so none reaches the compiled PDF. **Rule: fix-or-delete before citing, not before submitting.** Several of these are exactly the entries the failure-mode and realism-gap fixes would promote into live text — verify author lists for those 4–8 entries only (~20 min).
- **`wang2025scope` (`colm2026_conference.bib:168-174`)** is `@inproceedings` with a `journal` field, key year 2025 vs. `year = {2026}`, `author = {{Salesforce Research}}`, and a title that does not match arXiv:2601.07110. **Verify what that paper actually claims before citing it** — reviewers report it shows Nemotron personas underperforming purpose-built representations on SimBench, which is a direct threat to this paper's persona source. If true, cite it and pre-empt; being first to name it is correct.
- **UXAgent is cited twice under two keys**: `Lu+2025a` (`:27`, arXiv 2502.12561, cited at `body/99Appendix.tex:8`) and `lu2025uxagent` (`:673`, arXiv 2504.09407, cited at `body/01Introduction.tex:3`). Collapse to one and cite consistently — **this is the only bibliography defect visible in the compiled PDF.**
- **47 of 81 entries are uncited**, including `Vaswani+2017` and an orphaned ~20-entry psychometrics/census block (`:432-597`). Invisible in the PDF. **Do not bulk-delete** — that range contains `appleprimex` (`:575-582`), which *is* cited live at `body/99Appendix.tex:26`. Low priority.
- **`\input{tables/application}` is commented out** at `body/04application.tex:78` — the only occurrence in the repo — so the main body contains **zero tables** and `\label{tab:application-coverage}` is dead. When restoring: delete ", game" at `tables/application.tex:15` (no live text supports a game recommender), delete or restore the commented forum row at `:27-29`, and fix `\multirow{2}{*}{Web}` at `:24`, which currently spans a single row.
- **`tab:settings` (`:456`), `tab:app-aggregate-metrics` (`:517`), and all six `app:*-results` labels are referenced from nowhere.** The paper's only quantitative table is unreachable by a reader following the text.
- **Literal digit cross-reference:** `body/99Appendix.tex:17` — *"the models and run settings behind the results in 5"*. Renders as "the results in 5." Replace with `\autoref{sec:results}` and grep for other bare section/figure digits.
- **Wrong figure reference:** `body/03system_description.tex:16` cites `\autoref{fig:within-clusters}` for *"persona attributes, such as background, preferences, and communication style"* — that label points to a TF-IDF/SVD cluster scatter at `body/99Appendix.tex:907`. Repoint to `Box~\ref{box:persona}` (`:38`) or delete the parenthetical.
- **Wrong appendix pointer:** `body/05results.tex:5` — *"Details persona selection are provided in \autoref{app:diversity}"* — ungrammatical, and the selection rule is in `app:personas` (`body/99Appendix.tex:20`), not `app:diversity` (`:829`).
- **Appendix A duplicates Section 2 byte-for-byte.** `body/99Appendix.tex:3` ≡ `body/02related_work.tex:8` and `:5` ≡ `:15`. Its only unique paragraph (`:8`, applications) is the commented-out Section 2 paragraph. Delete the two duplicates and promote `:8` into Section 2 — it is live-quality, fully cited, and the only prose positioning the three settings.
- **Duplicate `\usepackage[most]{tcolorbox}`** at `colm2026_conference.tex:16` and `:19`; `math_commands.tex` (12 KB) never `\input`. Harmless. **Do not touch the preamble this cycle** — dismissed by the panel, and editing a working build under deadline is pure downside.

---

## 6. Writing, figures, and structure

### 6.1 Structure and page budget

Live main-body prose is ~1,150 words (Intro 217, RW 130, System 220, Applications 220, Results 302, Conclusion 62) against COLM's 9 pages. **The budget does not close, and nobody said so.**

Estimated: prose ≈1.9 pp + four floats ≈1.4 pp + title/abstract/refs ≈1.5 pp = **≈4.8 of 9, leaving ≈4.2 pp free.** The union of upheld additions is ≈4.6 pp *before* any appendix promotion:

| Item | Pages |
|---|---|
| New results (C1/C2/C3, controls, dissociation, volume-not-success) | 1.8 |
| `tab:app-aggregate-metrics` promoted at readable size | 0.6 |
| Differentiation paragraph + comparison table | 0.7 |
| Limitations + Ethics | 1.0 |
| Net figure growth from legibility fixes | 0.5 |
| **Total** | **4.6** |

**Two recommendations bid against each other and nobody noticed:** fixing figure legibility makes figures *bigger*, so `figure-text-illegible` and `structural-imbalance-promotions` compete for the same 4.2 pages.

**Decisions:**
- **Promote exactly one thing: `tab:app-aggregate-metrics` into Section 5 as Table 1, cited from `body/05results.tex:37`.** It is the only numeric evidence in the paper and is currently unreachable.
- **Promote it LAST**, after the censoring rates are printed beside every satisfaction number, the Persona/Task/SimQ columns are deleted (B2), and the Medical grounding bug is diagnosed (B4). Promoting an unexplained internal contradiction into Table 1 converts a buried liability into the first thing a reviewer sees.
- **Drop the other promotions.** The persona corpus and run-settings table stay in the appendix; there is no room, and new content fills the body anyway.
- **Do not "lengthen the paper."** An honest 6-page main-track paper reads better than a padded 9-page one.

### 6.2 Figures — delete first, regenerate second

Every figure prints below the ~7pt floor. Independently measured twice, agreeing to within 0.1pt against COLM's 5.5in text block (`colm2026_conference.sty:66`):

| Figure | Native width | Included at | Effective smallest text |
|---|---|---|---|
| `fig_results_components.pdf` | 19.33 in | `0.80\linewidth` (`05results.tex:17`) | **2.73 pt** (ticks); 3.64 pt axis labels |
| `fig_appendix_interpersona_heatmaps.pdf` | 15.70 in | `0.98\textwidth` (`99Appendix.tex:486`) | **2.20 pt** × 246 printed cell numbers |
| `fig_appendix_overall_metric_distributions.pdf` | 15.39 in | `0.98\textwidth` (`:476`) | **2.80 pt** |
| `fig2_within_domain_diversity.pdf` | 17.44 in | `0.90\textwidth` (`:874`) | **3.58 pt** |
| `PersonaEval.pdf` | 10.58 in | `0.98\linewidth` (`03system_description.tex:8`) | **4.08 pt**; largest glyph 7.03 pt |
| `PersonaEval_demo.pdf` | **37.27 in**, zero embedded fonts, one 2772×1109 raster | `0.98\textwidth` (`04application.tex:8`) | ≈**1.5 pt**, reduced 6.9× |

**Nothing anywhere reaches 8 pt.**

**Sequence it LAST and scope it to survivors.** Four of the nine figures should be *deleted* under other upheld findings, and regenerating figures you are about to cut is wasted labor:

- **Delete** `fig_appendix_interpersona_heatmaps.pdf` — 246 cells at 2.20 pt cannot be rescued by resizing, and the same information is in `tab:app-aggregate-metrics`. **But carry the polarity fix forward:** the single sequential dark=high colormap renders Finance `MaxTurn 0.92` as the darkest, most success-looking cell while it means 92% of conversations hit the cap without resolution — the failure column reads as the success column. **Report `1 − MaxTurn` as "Resolved before cap"** wherever MaxTurn appears, in the table and in any regenerated figure. One line, removes a genuine misreading.
- **Delete** panel (c) of `fig_results_components` (B2). Five of its six boxes are degenerate flat lines at 5.000; only Chatbot-Medical has a real distribution.
- **Delete** `fig3_within_domain_clusters.pdf` from the argument (item C); keep at most an appendix illustration. Drop its shared 1–6 legend regardless — cluster indices are **panel-local** and the shared legend invites a cross-panel reading that is meaningless.
- **Move** `PersonaEval_demo.pdf` to the appendix unmodified with a caption noting its panel order differs from the paper's. Its lettering runs **(a) Chatbot, (b) Web app, (c) Survey** against the survey/chatbot/web ordering used in the title, abstract, Section 4, and every results figure. Do not spend time cropping and re-lettering a figure that is no longer load-bearing.

**Then regenerate only the 4–5 survivors:** `figsize=(5.5, h)` for full width or `(4.4, h)` for `0.80\linewidth`, 7–8 pt fonts, included at `width=\linewidth` with **no** further scaling. Okabe-Ito palette plus a redundant marker shape per application (currently color is the sole class channel across ~300 overlapping points in `fig1_overall_diversity`, and its gold Web series is near-white on white and vanishes in grayscale). **Skip the PyMuPDF CI check** — 15 lines that run once on a paper that ships once.

**Hero figure (`PersonaEval.pdf`):** do the five-minute half only. Move the float declaration to the top of the Introduction with `[t]` — it is cited from `body/01Introduction.tex:7` but declared at `body/03system_description.tex:6`, so it cannot float to page 1. Fix the four visible typos: **"questionaires"**, **"Rubris"**, **"their trails"** (for *trials*), and the "Real User Study / deployment" casing. **Skip the vector redraw and the added claim panel** — the C2 result should be its own results figure where it can be read, not a squeezed strip inside an architecture diagram.

**Captions.** Discard the reviewers' nine drafted replacement captions — they assert results (a 0.37 group gap, an 87% fidelity rate) that no test has established and write captions for figures that are being deleted. Apply a four-item rule to the 4–5 survivors on the final writing day: **state *n*; name where the metric is defined; state polarity wherever a column is lower-is-better; end with the claim the figure supports.** ~30 minutes. Put per-group *n* in the panel-(b) row labels — Survey G4/G6 are n=3 and Beauty C2 is n=6, currently plotted at the same visual weight as Finance G2 at n=26, and *n* appears only inside the figure images.

### 6.3 Line-level writing

Do these in **one 45-minute hygiene sitting on the final day**, not as separate tasks:

| Fix | Anchor |
|---|---|
| Dangling pronoun in the paper's opening sentence — *"applications can produce different outcomes across users because **they** bring different goals"* | `body/01Introduction.tex:3` |
| Uncited gap claim — *"many existing pipelines are designed for a single task format"* | `body/01Introduction.tex:5` |
| Two byte-identical URLs labeled "Codes" and "demo video" | `body/01Introduction.tex:9` |
| `\method(~\autoref{...})` → `\method{} (\autoref{...})`; renders as "PersonaEval ( Figure 1)". Same on the second parenthetical. | `body/03system_description.tex:16` |
| Space before `\footnote` | `body/01Introduction.tex:9` |
| Scope residue from a different paper — *"conditions **recommendation agents** on arbitrary persona profiles"* — in the paper's only positioning sentence, duplicated live at `body/99Appendix.tex:5` | `body/02related_work.tex:15` |
| Unsupported *"increasingly realistic simulations"* → *"packaged into simulation frameworks"* | `body/02related_work.tex:15` |
| 43-word run-on about the sandbox | `body/03system_description.tex:18` |
| *"persona model"* — occurs nowhere else in the paper | `body/03system_description.tex:21` |
| Section heading "Task API" vs. "task adapter" used everywhere else — **the one terminology fix worth making**; skip the full four-vocabulary audit | `body/03system_description.tex:24` |
| Causal *"because"* claims → explicitly labeled hypotheses, pending item C | `body/05results.tex:37,40` |
| *"expand to broader domain of applications"* → *"expand to a broader range of applications"* | `body/06conclusion.tex:4` |
| Delete unsupported closer *"These findings position \method as a useful tool for simulating real-user studies"* | `body/06conclusion.tex:4` |
| Delete *"approximates real-user behavior"* → *"generates persona-conditioned interaction trajectories"* | `body/00Abstract.tex:2` |
| Delete *"approximates recruiting users"* | `body/99Appendix.tex:34` |
| Delete *"to approximate the selection of real users"* | `body/05results.tex:5` |
| `\label[appendix]` on `app:settings` and `app:diversity` (renders "Section A.4" mid-sentence next to "Appendix A.3"); delete the six unreferenced `app:*-results` labels | `body/99Appendix.tex:435`, `:829`, `:521,561,600,644,689,727` |
| Purge "demo"/"demonstration" — 9 live occurrences | `00Abstract.tex:4`; `01Introduction.tex:9,11`; `04application.tex:4,10`; `99Appendix.tex:13,24,169`; `tables/application.tex:32` |

**Conditional on the demo purge:** do it **only alongside the contribution rewrite**. Deleting the word while the contributions remain artifact claims fools nobody, and a paper that scrubs "demo" and still reads as one has spent its credibility. If C2 does not run, **keep the word** and submit to the demo track, where it is simply accurate.

### 6.4 Related work

- **Restore the commented failure-mode paragraph** (`body/02related_work.tex:12`) as live text. `argyle2023oneofmany`, `santurkar2023whoseopinions`, `wang2024flatten`, `west2025priceofformat`, `li2024instability`, and `chen2024personasurvey` are currently cited **zero** times in live text. Omitting exactly the literature that predicts your own degenerate results reads as avoidance.
  **Restore it only in the same revision that ships a measurement it is tied to.** Tie `wang2024flatten`/`west2025priceofformat` to the mode-collapse numbers (item B, item F). **Replace the `li2024instability` "free fourth result"** — a per-turn *persona-consistency* curve needs the judge whose existence is in doubt; substitute **per-turn re-identification accuracy** from stored per-turn text and report the accuracy-vs-turn-index slope as the drift test. That needs no judge and is genuinely free.
  End with the positioning line: *"\method does not solve these failure modes; it makes them measurable"* — but only once at least two of them are, in fact, measured.
- **Restore the values-vs-demographics sentence** (`body/02related_work.tex:9`, commented) — it justifies choosing rich Nemotron narratives over demographic tuples.
- **Write the differentiation paragraph in prose** (half a day). It is the paper's positioning argument and is worth the time. `Liu+2025` ("Free Lunch for User Experience: Crowdsourcing Agents for Scalable User Studies", `colm2026_conference.bib:648-654`) — the closest competitor — is cited **zero** times.
- **Build the comparison matrix only after C2 returns a number, and cut it to three columns you can verify from each system's own abstract** (modalities covered, persona source, primary outcome). Drop the columns requiring *negative* claims about others' capabilities ("none of the four supports the same population across more than one modality" is checkable and probably false — TinyTroupe ships both survey-style and open-ended interaction). **Do not include a "validated against human data" column** — it reads "no" for \method and "yes" for three competitors, and a table where your only differentiating YES is "we computed an ICC" argues for rejection. Handle that in Limitations, where it reads as candor rather than as a scorecard you lost.
- **Replace the multi-modality assertion with a claim about reporting**, which is checkable and defensible: *"We are not aware of a prior system that reports the same persona panel driving survey, dialogue, and browser tasks under a shared trace and evaluation schema."*
- Add one sentence citing the canonical user-simulation survey to frame \method inside the system-evaluation branch. Ten minutes.
- **Cite one 2025–26 real-vs-simulated realism-gap paper at `body/05results.tex:43`** as the reason persona alignment is necessary but not sufficient — after verifying its author list and arXiv ID against the listing page. **Do not integrate an external benchmark** (MirrorBench or similar) this cycle unless it is a genuine drop-in over stored traces; unfamiliar benchmark integration is the classic deadline sinkhole.

---

## 7. Ready-to-paste LaTeX

Both sections are calibrated to the honest post-C2 state. **Write them last**, after B2/B4/B5/B6 resolve — and verify the two bracketed facts before shipping. Every limitation is paired with the measurement that bounds it, per the red team's hardening: an unmitigated item is a confession, a bounded item is scope.

### 7.1 Limitations

```latex
\section*{Limitations}
\label{sec:limitations}

\paragraph{No human ground truth.}
We report no comparison against human users. Our claim is not that \method
reproduces human behavior; it is that a simulator's output is admissible as
evaluation evidence only under conditions we define and measure, and the
criterion-sensitivity condition (\autoref{sec:results}) can be evaluated
without human data because the quality ordering of our application variants is
fixed by construction. Human concordance is nonetheless the property a
practitioner ultimately needs, and we do not establish it. The closest
available anchors are archival: \citet{Cui+2024} report 73--81\% recovery of
human main effects in scenario-based replication, a standard our survey
adapter has not been evaluated against.

\paragraph{The evaluator is the evaluated.}
For all applications, the persona-conditioned model instance that produces the
interaction also completes the post-interaction form
(\autoref{app:simulator}). The satisfaction scores are therefore self-reports,
not independent measurements. We bound this by reporting, for every
application, at least one outcome that the simulated user does not author ---
sandbox order records for the web task, and turn counts and grounded-item
counts for the chatbot tasks --- and by reporting the association between the
two. We do not claim the self-report is independent of the simulator's rating
disposition; \autoref{sec:results} is in part a measurement of that
disposition.

\paragraph{One persona corpus, \NUM{M} simulator families.}
All reported personas are drawn from Nemotron-Personas-USA, which is
US-centric and English-only, and are rendered by a single loader into one text
block. Selection keeps the 50 profiles nearest each application description
[VERIFY the size of the Nemotron-USA subset before shipping this sentence: if
the candidate pool is a few hundred, selection is near-exhaustive and orders
rather than selects]. We report \NUM{M} simulator model families on
\NUM{K} applications; where a result does not replicate across families we say
so explicitly, and results reported for a single family should be read as
properties of that model rather than of persona simulation in general.

\paragraph{Panels are nested, not crossed.}
Each application is evaluated by its own similarity-selected panel, so persona
and application effects are not separable in principle and no
persona\,$\times$\,application interaction is estimable. We report the
persona-identity overlap between panels (\autoref{app:diversity}) so that
readers can judge how far the design departs from a crossed one, and we
restrict cross-application comparisons to those the overlap supports.

\paragraph{Protocol effects dominate outcomes in two applications.}
The eight-turn cap truncates 92\% of Chatbot-Finance runs and 0\% of
Chatbot-Medical runs. We report the censoring rate beside every satisfaction
number and do not compare applications with materially different censoring
without saying so. The web environment is saturated: all 50 runs complete the
goal in the same number of actions and \NUM{45}/50 emit an identical overall
rating. We report the web arm as a saturation control rather than as a third
modality with an independent outcome measure, and we do not draw fidelity
conclusions from it.

\paragraph{Environments are bespoke.}
All six applications are hosted by us, including the shopping site, which is a
sandbox we built rather than a public benchmark deployment. No number in this
paper is directly comparable to a number in prior work, and reproduction
requires the released sandbox and catalog. Extending the admissibility
protocol to a public, versioned environment with a programmatic success metric
is the clearest next step and is the one we recommend to anyone adopting it.

\paragraph{Decoding is not deterministic.}
Neither model provider exposes a determinism guarantee: the Anthropic Messages
API has no seed parameter and OpenAI's is best-effort. We therefore record
request identifiers, full decoding configuration, and API access dates, and we
report across-repeat resampling variance ($R=\NUM{R}$) rather than
seed-level reproducibility. Exact re-execution after an endpoint update is not
possible.
```

### 7.2 Ethics Statement

Half a page is enough. **Do not write the medical paragraph until B4 is diagnosed** — "no clinician reviewed any transcript" printed beside a table showing `Task 1.00±0.00` on runs that grounded nothing is the worst pairing available.

```latex
\section*{Ethics Statement}
\label{sec:ethics}

\paragraph{Human subjects.}
No human subjects participated in this work. No interaction traces, ratings,
or annotations reported here were produced by a person, and no institutional
review was required. We state this explicitly because a persona-based
simulation of user research can be mistaken for user research; every score in
this paper is model output.

\paragraph{Simulation is pre-human triage, not a user study.}
\method is intended to surface candidate problems before a human study, not to
replace one. Our own results argue against substitution: the simulated
satisfaction scores we measure do not order applications of known-differing
quality, and they track interaction volume rather than task completion. We
explicitly recommend that \method output not be used to inform clinical,
regulatory, procurement, or deployment decisions, and that any such decision
rest on evidence from the people affected by it.

\paragraph{Demographic conditioning.}
Personas carry demographic attributes (age, gender, marital status, education,
occupation, and location), and our analyses group simulated users by persona
content. Any resulting statement is a statement about model behavior under a
prompt, not about the groups named in that prompt. Reporting that simulated
``nail professionals'' rate a recommender lower is a measurement of the
simulator; treating it as a finding about nail professionals would reproduce
exactly the silicon-sampling and within-group-flattening failures documented
by \citet{argyle2023oneofmany}, \citet{santurkar2023whoseopinions}, and
\citet{wang2024flatten}, which our results corroborate rather than resolve.

\paragraph{Sensitive application domains.}
Two adapters place simulated users in sensitive settings: a medical
consultation assistant and a financial research assistant. Neither produced
advice delivered to a person, no patient or health data of any kind was used,
and no clinician or financial professional reviewed any transcript. We report
these adapters as measurements of the harness, not as evaluations of the
underlying assistants, and we do not claim that the guidance they produced is
safe, accurate, or suitable for use.

\paragraph{Artifact and data licensing.}
The reported runs use only the Nemotron-Personas-USA subset (CC~BY~4.0). To
avoid redistributing corpora under incompatible terms --- PersonaHub is
CC~BY-NC-SA~4.0 and research-only --- the release ships persona identifiers
and per-source download scripts rather than a merged corpus, so each dataset
remains under its own license. Released interaction traces contain
model-generated text from two commercial providers; each trace turn is
labelled with the model that produced it so that downstream users can apply
the correct terms. Application code under test is used unmodified under its
own license, and the versions used are listed in \autoref{app:settings}.
```

---

## 8. Struck and dismissed critiques

Authors: these were checked and set aside. Do not resurrect them.

### 8.1 Struck by the verifier

**None.** The verifier struck no finding as factually unsound. Three findings were marked **MISCITED** on sub-claims while their cores were confirmed — the corrections matter and are folded into §5:

| Finding | Core status | What was wrong |
|---|---|---|
| `fig3a-invalid-cross-app-axis` | **Confirmed and worse than filed** | Three incompatible ordinal grids, not two: the four chatbot columns are k/7, **Survey is k/5 with n=150 un-averaged points** (not 50), and the shared 0.571 median belongs to the four chatbot columns, not Survey (median 0.600). |
| `saturated-alignment-metric` | **Ceiling confirmed** | Recovered per-application means were wrong. Actual: Survey 4.733, Movie 4.980, Beauty 4.780, Finance 5.000, Medical 4.600, Web 4.980. **Five** of six boxes are degenerate flat lines at 5.000, not four; only **Chatbot-Finance** is literally 5.00 with zero variance. Cite the table values, not the figure-derived ones. |
| `related-work-no-positioning` | **Positioning gap confirmed** | Its headline claim that UXAgent is never discussed is **wrong** — UXAgent is cited live under a second key (`lu2025uxagent`, `body/01Introduction.tex:3`). The accurate residue: no live sentence compares \method against any named system. |

Smaller corrections worth knowing: `heatmap-wrong-chart-and-wrong-polarity` asserted the companion figure "draws box plots over binary variables" — **false**, `fig_appendix_overall_metric_distributions.pdf` is a strip plot (2550 individual markers, zero box rectangles). `captions-not-self-contained` claimed no caption defines $r_{90}$ — **false**, and contradicted by its own quoted evidence at `body/99Appendix.tex:881-882`. `webarena-not-webarena` described "a bespoke four-product mock site" — the number four appears nowhere in the source. `no-limitations-section` cited `body/06conclusion.tex:3`; the sentence is on **:4**.

### 8.2 Dismissed by the judge panel (true but immaterial)

| Item | Why dismissed |
|---|---|
| `textfloatsep-template-deviation` — `\setlength{\textfloatsep}{5pt}` at `colm2026_conference.tex:126` | COLM's template warning concerns `geometry` and page margins, not float separation. The paper is pages under length so the tweak buys nothing; reverting one line is harmless but it has no bearing on review. |
| `preamble-hygiene` — hyperref loaded 3rd of 20, `tcolorbox` loaded twice with identical options, `math_commands.tex` (508 lines) never `\input`, three packages used only in commented code | Invisible in the compiled PDF, no observed build problem, and editing a working preamble under deadline is pure downside risk. |

### 8.3 Killed or hardened by the red team

Where the red team killed a recommendation, it is dropped. Where it hardened one, the hardened version is what appears above.

**KILLED — do not do these:**

| Recommendation | Why it is dead |
|---|---|
| **Human annotation study of persona alignment** (`human-judged-contradiction` option 1, `judge-saturation-and-self-report` fix 3): ≥150–200 traces × 2+ annotators, Krippendorff's α | Time estimate is 1.5–2× low: an 8-turn transcript plus a full Nemotron persona record plus an evaluation form is 5–8 min per careful ordinal judgement, i.e. **32–43 hours**, not 20–30. And α is 1 − D_o/D_e — with almost every unit rated 5, D_e collapses and α is numerically unstable and interpretively meaningless **regardless of how well the raters agree**. Forty hours to compute an undefined statistic on a ceilinged metric. |
| **`tiered-validation-plan` Tiers 1–3** ($600 / $2,500 / $7,000–9,000) | Calendar wrong by 2–6×. "1 week IRB exempt determination running in parallel" is not a thing — exempt determinations queue 2–6 weeks and you cannot recruit before one issues. Tier 2 puts real participants in front of a **medical consultation** and a **financial allocation** chatbot; neither is plausibly exempt, both trigger expedited or full review, realistic calendar 6–10 weeks. Its Tier-1 power calculation also answers the wrong question (60/arm for d=0.5 is a two-sample mean test; the deliverable is W₁ and distribution match, for which n=60 gives a CI wider than any effect you would claim). **Keep the internal cost discipline; put none of it in the paper** — a four-tier costed program in the manuscript documents that the authors priced the validating study and chose not to run it, and tells the reviewer exactly what to demand. |
| **`validation-metrics-undefined`** — six metric families (per-persona correlation + ICC, KS + W₁ + TVD + Cliff's δ with null bands, rank correlation with τ_b, effect-size recovery with four sub-statistics, calibration plots with ECE and a fitted isotonic map, variance ratio) | A specification document for an experiment with no data. Costs 1–2 days of the team's best analytical attention and produces text a reviewer reads as a **promise** — precisely the deferred-validation failure the paper is already punished for. Elaborating the promise makes it worse. Two ideas retained: always report ICC alongside Pearson (a simulator rating everything 1.5 points high still scores r=0.9), and always report a null band so readers know what "as close as humans are to themselves" looks like. |
| **`turn-cap-censoring-invalidates-cross-app-means`** — Kaplan-Meier, Cox PH, Tobit-adjusted means | **Statistically wrong.** Tobit is for a censored *outcome*; here the rating is not censored, the conversation *length* is. Applying it to ratings conditioned on censoring status is a misapplication a reviewer who knows survival analysis will name, converting a real problem into an own goal. KM/Cox is theater at this resolution: turn counts run 5–8 against a hard cap of 8, giving ~4 distinct event times; the log-rank comparing 92% to 0% censoring is trivially significant and restates two numbers already in the table. **Its Clarify observation is retained** in §5.3. |
| **`survey-best-roi`** ordering (survey > chatbot > web for human validation) | Optimizes for cheapness and is blind to what the survey arm can prove. The survey adapter is a **single JSON completion** — no interaction, no turn-taking, no application under test, no trajectory. It is a persona-conditioned survey responder, the most thoroughly studied artifact in this literature. Validating it well reproduces prior work; validating it badly is a bad result; neither supports the paper's distinctive claim about heterogeneous *interactive* applications. **Retained:** its correct warning not to instrument the web sandbox for human participants. **Reversed:** if human money is ever spent, it goes to **chatbot**, not survey. |
| **`webarena-not-webarena` option (a)** — actually run WebArena at a pinned tag with per-episode reset | 2–3 weeks of infrastructure minimum (multi-GB docker image, reset protocol, adapter rewritten against a real DOM), it produces a different action space so no existing web number carries over, and it does not fix the actual problem — that persona conditioning does not propagate through a constrained checkout task. Option (b), the honest rename, is what ships. |
| **`free-human-datasets`** PRISM **paired per-person** design | Three independent kill-shots: (i) a new persona renderer for PRISM self-descriptions is 2–3 days; (ii) PRISM humans conversed with a heterogeneous set of model endpoints, so replaying into GPT-4o-mini confounds simulator fidelity with system identity; (iii) PRISM conversations were steered turn-by-turn by the human, so the simulated user steers elsewhere and the "paired" comparison is paired in name only. **Distributional variance-ratio version retained** (item O). |
| **`free-human-datasets`** Cui+2024 **effect-size recovery as a benchmark to beat** | Re-fielding their instruments and obtaining per-contrast human effect sizes is a replication study, not a week. **Cite the 73–81% band as the standard not yet met and adopt it as protocol** (item N). |
| **`no-human-groundtruth`** optional N=50 Prolific arm on the Nike instrument | At N=50 on a 5-point Likert the bootstrap CI on per-item W₁ is ±0.25–0.30 Likert points and modal-agreement carries a Wilson interval of ~±14 pp. You cannot distinguish "matches humans" from "does not." Real money and a real determination for a result consistent with both hypotheses. **Its mandatory half — the four deletions — is unchanged and is a day-1 task.** |
| **`no-baselines-no-repeats`** allocation "10 personas × 10 seeds" | Wrong allocation and internally inconsistent with `underpowered-n50`: between-persona power is driven by the *number of personas*, not draws per persona. Collapsing to 10 personas throws away 80% of between-persona information; the resulting ICC has a 95% CI of roughly [0.1, 0.8]. **Use 50 personas × 5 draws** — same total runs, far better precision, and it preserves every persona-level analysis. |
| **`selection-not-recruitment`** items (2) and (3) — re-sample from the full 1M Nemotron corpus; add a census-quota sampling condition | Re-selecting from 1M means embedding a million records, new index infrastructure, and **re-running every experiment on entirely new panels, invalidating every existing number.** The quota condition is another full grid plus matching logic and makes the paper about sampling frames instead of about the instrument. **One Limitations sentence covers it** (§7). Next-paper decisions. |
| **`scale-heterogeneity-ordinal`** item (2) — harmonize all instruments to one scale | Priced at "~$25". The price is not $25; it is **discarding the paper's entire results section.** Item (3), the ordinal cumulative-link model on raw responses, is retained and is the correct model. |
| **`plug-and-play-asserted-not-shown`** LOC table | Likely to *refute* the claim it supports: `box:chat-apps` shows the four chatbot "applications" differ only in three text fields (`applicationKey`, `systemLabel`, `systemDescription`), and `body/99Appendix.tex:340-342` states finance and medical share one evaluation form. A per-adapter table will show six applications are three adapters, one instantiated four times. **Ship instead:** the abstract Task API signature in a code box in Section 3 — currently the interface a new application must implement exists only in prose, which is the genuine defect. Plus one free data point: how many lines the shuffled-persona arm (B7) required. |
| **`artifact-release-insufficient`** nine/ten-directory artifact spec with `results/expected`, tolerance-comparison scripts, and a mocked smoke test | Multi-week engineering for badge credit that does not move a review score. **Ship three things:** a README mapping each figure to what produces it; raw per-run traces; a pinned environment file. Swap the anonymous URL for an archival DOI at camera-ready, not now. |
| **`figure-text-illegible`** PyMuPDF CI check | 15 lines that run once on a paper that ships once. |
| **`new-figures-needed`** Figures A and C (human-agreement scatter; seed-reliability grid) | Depend on work that will not happen this cycle. Figure D (ordinal degeneracy) is the same stacked-bar rebuild `fig3a` already requires — build it once. Table B (cost) is item M. |
| **`terminology-audit`** full four-vocabulary sweep | 2–3 hours of find-and-replace across ~1,400 lines with real risk of breaking `\autoref` targets, for a score delta near zero. **One edit survives:** rename the "Task API" heading to match "task adapter." |
| **`bib-duplicate-and-dead-entries`** bulk deletion of `colm2026_conference.bib:432-597` | That range contains `appleprimex`, which **is** cited live at `body/99Appendix.tex:26`. An uncited entry costs nothing; a mistakenly deleted cited entry produces a `?` in the PDF. Collapse the two UXAgent keys and stop. |
| **`repro-checklist`** as a published appendix subsection | Every FAIL row is an item elsewhere. Use it as a 20-minute pre-submission self-audit; do not print it. Its one paper-facing row: **single-run vs. repeated is never disclosed**, which is why the SDs in `tab:app-aggregate-metrics` are uninterpretable to a reader even before the statistics are questioned. |

**Judge disagreements I am adjudicating, since the panel split:**

1. **`abstract-overclaim-and-rewrite`.** Judge 3 rated it merit 4 and "shippable today"; Judge 2 flagged that its replacement abstract headlines "92–100% of runs judged consistent with the assigned persona" — the exact saturated, provenance-unknown metric that `fidelity-metric-ceiling` and `human-judged-contradiction` say must not be reported as evidence. **I side with Judge 2.** Do not ship that number. Use §2.3's fallback instead.
2. **`survey-confidence-and-choice-fields-unused`.** Judges 1 and 2 called it understated at "minor"; Judge 3 downgraded it to polish. **I side with Judges 1 and 2 and with the red team:** promote to top-5. It is the compression claim measured on a discrete, encoder-free outcome, immune to every objection raised against the embedding-distance version. **Skip the `confidence`-field half** unless a quick check shows it is non-degenerate.
3. **`single-persona-model-no-pins`.** Judge 1 downgraded to 3, Judge 3 to 2; the red team argues it is blocking because the panel's own chosen headline is a phenomenon claim requiring N>1 models. **I side with the red team, with a scope reduction:** two families, two applications, ~$10 (item K). A first paper on a new instrument is allowed one model *if the scope is stated*; it is not allowed one model if the headline is a phenomenon claim.
4. **`free-human-datasets`.** Judges 1 and 2 upheld at 4; Judge 3 downgraded to 3. **I side with Judge 3 plus the red team hardening:** two archival anchors, distributional only, framed as calibration references rather than fidelity validation. Listing eight datasets you did not use is worse than reporting two you did.
5. **`eval-form-one-latent-factor`.** Judges 1 and 2 upheld at 3; Judge 3 downgraded to polish. **I side with Judge 3.** Polychoric correlations at n=200 with one binary item (`clarificationUseful`) and ω on four items is a fragile estimation problem yielding a suggestive result you cannot defend. **Do the descriptive version only:** print the 4×4 Spearman item matrix and the PC1 variance share, binary item excluded from the polychoric step. If PC1 > 80%, one sentence — *"the four-item form behaves as a single did-it-go-well dimension"* — is the whole contribution.
6. **`leniency-halo-overall-exceeds-components`.** All three judges upheld at 3. **The red team is right that it is probably a normalization artifact:** the components are 1–5 items and `overallRating` is on a different, unpinned scale; under divide-by-max a 1–5 midpoint maps to 0.60 and a 1–7 midpoint to 0.571, producing a systematic offset of the same sign and roughly the same magnitude as the reported +0.10 "halo." **Gate it behind a ten-minute recompute on raw scales** using a within-run rank statistic (Wilcoxon on rank of overall vs. mean component rank, Holm across applications, Cliff's δ). If it survives, one sentence and one small figure inside a combined "self-report artifacts" paragraph with item 5. If not, drop it entirely.
7. **`no-cis-no-tests-no-correction` primary endpoint.** The panel prescribed persona variance share; two red teams say wrong endpoint. **I side with the red team** — see B9.

---

## 9. Venue recommendation and plan of record

### 9.1 Venue

**Target: COLM 2026 main track, gated on one experiment.**

**Go/no-go gate — end of Week 1: has the C2 criterion-sensitivity experiment (item H) returned a number, of any sign?**

- **YES → COLM main track.** Reframe per §2, and the negative result is the contribution. A measured null on your own instrument is a stronger scientific position than not having measured; a reviewer who reads *"our simulated satisfaction score cannot rank a working recommender above one that ignores the query, and here is the protocol that established it"* is reading a measurement paper, not a wrapper paper.
- **NO → system-demonstration track** (EMNLP/ACL Demos or equivalent), using the fallback abstract in §2.3, keeping the word "demo," and shipping the B0–B4 correctness fixes plus the free Tier-0 analyses as an honest appendix. This artifact is genuinely strong as a demo — four chatbot backends, a browser environment, and a survey adapter under one persona interface is more integration work than most simulation papers do, and the 17 prompt boxes would let someone reimplement the simulator. **Do not submit the fallback to main track.** Scrubbing "demo" while the contributions remain artifact claims draws the harshest review available, because it signals the authors knew what they had.

**First, confirm which deadline is real.** This directory is named `COLM_2026_SocialSim`, which reads as a workshop track. The workshop and main-track calendars are different projects and this plan is sized differently for each. Resolve that before Day 1.

**Second-choice venues, ranked, if COLM main track is missed:** (1) NeurIPS Datasets & Benchmarks — the admissibility protocol plus released traces is squarely their remit, and the fix list is essentially their checklist; (2) CSCW — takes simulated-user work seriously and is the community most alert to silicon-sampling harms, so the failure-mode literature you are restoring is an asset there rather than a liability; (3) CHI — highest bar on ecological validity and the harshest audience for LLM-rates-LLM without human comparison. **Not** ACL/EMNLP main track, where the empirical bar for a claims-about-users paper now effectively requires human correlation.

### 9.2 Plan of record

**Sequencing principle: API work is the only work with an external expiry date.** The runs pin Claude Haiku 4.5 and GPT-4o mini; if either drifts or retires mid-cycle, every rerun-based item dies at once and the existing grid becomes incomparable. Verify both model IDs still resolve **before** scheduling anything else. Writing and re-analysis do not rot; endpoints do.

**Day 1 — gates and free analysis. No API calls.**
- Hour 0: **delete the identity comments** (B0). Run a clean `latex + bibtex` build as a smoke test; fix `@article@article` (`colm2026_conference.bib:19`).
- Hours 0–2: **data inventory** (B1). Publish the answer to the whole team before anything else is scheduled. Also confirm: is the embedding model recorded? Are per-persona similarity scores persisted? Are the plotting scripts alive?
- Hours 0–2, parallel: **judge time box** (B2). At hour 2, act on the pre-committed rule.
- **Normalization fix** (B3) + **chatbot scale audit** (B3b). 1 hour. This blocks two other items and must not slip.
- **Item A** (volume-not-success), **B** (choice-item entropy), **C** (permutation test), **E** (Jaccard), **F** (web audits). All free, all today.
- Check on the RecAI retrieval path: patchable in under 2 hours, or use the fixed-listing degradation?

**Days 2–3 — one API batch. Instrument telemetry before launch (item M).**
- **H** (C2, 900 runs, two applications, three tiers) — *the batch's reason for existing*
- **I** (persona controls, Beauty + Movie)
- **J** (R=5 reruns, four non-degenerate applications)
- **K** (second model family, two applications)
- **L** (T=16 Finance)
- Total ≈ $150–250 and one to two overnights. **If GPT-4o mini is already retired, that decision cascades:** freeze the current numbers, drop every rerun-based item, pivot entirely onto Tier 0, and go to the demo track.

**Days 4–5 — analysis and assets.**
- C2/C1/C3 analysis; **D** (re-identification with lexical scrub); **G** (survey crossed decomposition, reported as stimulus-invariance); item-6 self-report artifacts pass; **B4** diagnosis.
- **Delete four figures, then regenerate the 4–5 survivors** at print size with CIs, *n*, and corrected MaxTurn polarity. Build the stacked ordinal bar once — it discharges `fig3a-invalid-cross-app-axis`, `new-figures-needed`, `undefined-metrics-and-normalization`, and the deleted panel (c) simultaneously.
- Apply B9 inside the same pass; marginal cost ≈ 2 hours since you are rebuilding the plots anyway.

**Days 6–10 — writing. This is the unbudgeted bottleneck and nobody costed it.**
- Turning ~1,150 words of live body into a main-track paper with a new central thesis is **5–8 writing days for a small team**, on top of everything above. Minimum honest total for the whole program is ~3 weeks of near-full-time effort by two people.
- Order: differentiation paragraph → Related Work restoration → Results rewritten around C1/C2/C3 → Section 3 Task API code box → contribution bullets → abstract (**not before Day 8**) → Limitations → Ethics → captions → 45-minute hygiene sitting → 20-minute reproducibility self-audit → pre-upload anonymity grep.
- **Write a one-sentence thesis on Day 1** and test it against every result as it lands. Do not write any abstract early; anchoring on the fallback abstract on Day 2 is how the reframe quietly fails to happen.

**The page count fills itself under this plan** (≈3.5–4 new pages from C2, controls, the dissociation figure, the promoted table, Limitations, and Ethics). Do not schedule "lengthen the paper" as a task.

---

## 10. What the red team says is still fatal after all of the above

Six things. The first three are the ones that decide whether this becomes a main-track paper; the last three are risks to manage, not defects to fix.

**1. The instrument may fail C2, and there is no reframe below that.**
The persona null is survivable — reframed as compression, it becomes the finding. A **criterion** null is different. Sensitivity is the one property an evaluation instrument must have. If 50 personas rate a working recommender 0.59 and a query-ignoring one 0.55, the paper's honest conclusion is *"our evaluator cannot detect a deliberately broken system"* — and the paper is then a well-executed negative result about persona-based evaluation, not a system paper with a caveat. **That is still publishable and is still the right thing to do.** But the authors must decide *now*, before the run, that they will publish it. A team that runs C2, gets a null, and then quietly reports only C1 and C3 has produced a worse paper than one that never ran it. Pre-register the endpoint and pre-commit to publication in writing.

**2. Everything free depends on data nobody has confirmed exists.**
Roughly fifteen upheld recommendations are premised on "computable today from data already on disk." This repository contains only `.tex`, `.bib`, `.sty`, and nine baked figure PDFs. The authors' own commented TODOs show they had already lost the normalization map (`body/99Appendix.tex:796-797`) and could not identify the judge (`:821-823`). If the raw per-run scores, the free-text rationales, and `action_history` are not persisted, then item A degrades to a table-level n=4 observation, items B, D, F, G are impossible, the dissociation cannot be measured, and the reframe is not carryable by "the current data" at all. **B1 is a real gate, not a formality.** If it fails, go to the demo track on Day 1 and do not spend the three weeks.

**3. Comparability is zero after the WebArena rename, and that is not fixable this cycle.**
B8 is mandatory and honest. It is also the moment all six environments become private: three internal survey instruments, RecAI catalogs of unstated size, an OpenBB wrapper, a GitHub medical assistant at unstated commit, and a store the team built. **No number in the paper is comparable to any number in any prior work, and nobody outside the team can reproduce a row.** The admissibility framing partially rescues this — a protocol is more portable than a benchmark result — but a reviewer is entitled to ask why the protocol was never demonstrated on a public, versioned environment with a programmatic success metric, and the honest answer is "we ran out of time." **State it in Limitations rather than letting a reviewer state it.** For the next version, instantiating the three checks on one public environment (real WebArena at a pinned tag, τ-bench, WebShop) is worth more than any additional analysis on private ones, because those environments ship ground truth in the box.

**4. Restoring the failure-mode literature creates a collision the paper must handle explicitly.**
`truncation-drops-all-failure-modes` is a top-ROI item and is correctly upheld. But `wang2024flatten` and `west2025priceofformat` document diversity collapse **against human reference distributions**, and this paper has none. Restore the paragraph and lead with an unqualified compression claim and Section 2 will announce that Section 1's headline is known. The admissibility framing survives this — *"prior work documents diversity collapse; we show the collapse is channel-specific and quantify its consequence for evaluation"* is a real delta — but it must be written into the text as an explicit positioning move, not left for a reviewer to notice.

**5. The plan is ~3 weeks of near-full-time work for two people, and writing is the unbudgeted item.**
Three days of analysis, two of assets, five to eight of writing, plus a batch that must run early because of endpoint expiry. Every reviewer costing in the record priced experiments in dollars and none priced the rewrite in days. If the actual calendar is two weeks, this plan does not fit and something must be cut **before Day 1, not discovered on Day 12**. The correct cut, in that case, is not to trim the experiments — it is to go to the demo track and do B0–B4 plus Tier 0 properly.

**6. The re-identification result may be pure paraphrase, and the control is the whole measurement.**
`{persona_profile}` is pasted verbatim into the system prompt (`body/99Appendix.tex:80`), and the simulated user is instructed to stay in character. Top-1 re-identification from `ratingReason` could hit 90%+ on nothing but occupation, city, and hobby tokens copied out of the prompt. The panel's recommendation explicitly deprioritizes the lexical-overlap variant ("skip unless the top-1 result is ambiguous") — **that is exactly backwards.** The scrub ablation *is* the validity of C3. Ship the number without it and a reviewer kills the paper's grounding leg in one line; ship it with the ablation and either outcome is a result. If accuracy collapses to chance after scrubbing, persona conditioning produces surface copying rather than behavioral variation — which is a sharper and more damning version of the thesis, and the paper should say so.