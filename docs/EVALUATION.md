# Evaluation Protocol

**Status: [planned].** No benchmark results exist yet. ARGUS is in
**Phase 1 — CASEset Dataset Foundation** (see [ROADMAP.md](ROADMAP.md)).
The models this protocol measures are not yet built, so this document does
**not** report numbers. It defines *how* ARGUS work will be measured — the
metrics, units, data splits, baselines, and reporting standards — so that when
results do exist they are reproducible, comparable, and honest about
uncertainty.

Treat every metric below as a contract for *future* reporting. Any commit that
claims a result must (a) follow the splits and reporting standards here, or
(b) explicitly document and justify the deviation.

Related documents:

- [DATASET.md](DATASET.md) — CASEset collection, schema, privacy, and limits;
  the source of the subjects, sessions, and streams referenced here.
- [ARCHITECTURE.md](ARCHITECTURE.md) — inputs, processing stages, and outputs;
  defines the stages that latency and per-stage metrics decompose into.
- [ROADMAP.md](ROADMAP.md) — phased plan (Phase 1–7); ties each metric to the
  phase that first makes it measurable.

---

## 1. Evaluation Philosophy

The point of this protocol is to make it hard to fool ourselves. The following
principles are non-negotiable for any reported result.

- **Report uncertainty, not point estimates.** A single mean is not a result.
  Every headline metric is reported with a dispersion estimate (confidence
  interval or per-subject spread). See [Reporting Standards](#5-reporting-standards).
- **Held-out subjects are the default.** The primary evaluation regime is
  **subject-independent**: subjects in the test set never appear in training or
  validation. Gaze and attention signals are highly person-specific; a model
  that memorizes individuals will look strong and generalize poorly.
- **No leakage across sessions or subjects.** Frames, sessions, and subjects
  are partitioned so that no temporally adjacent or same-session data crosses
  the train/test boundary. Calibration data, when used, is drawn only from the
  test subject's *own* held-out calibration block, never from training.
- **Distinguish demonstrated from aspirational.** Results are labeled
  **[current]** only for tasks actually run on held-out data. Anything not yet
  measured stays **[planned]** or **[hypothesis]**. This document is almost
  entirely [planned] by design.
- **Reproducibility over optics.** A worse number computed correctly is more
  valuable than a better number computed with leakage. Configs, split
  definitions, and random seeds are versioned alongside results (see
  [configs/](../configs/)).
- **Early and candid.** ARGUS is early-stage. Where a metric cannot yet be
  computed because the data or model does not exist, that is stated plainly
  rather than papered over.

---

## 2. Core Metrics

Each metric below specifies: what it measures, its **units**, a **formula**
where useful, and the **phase** at which it first becomes measurable. Metrics
are grouped by what they evaluate.

### 2.1 Angular Gaze Error

- **What:** Angular difference between the predicted gaze direction and the
  **reference (measured) gaze direction**, as seen from the eye/camera origin.
  CASEset provides no gold-standard "ground-truth gaze"; the reference is a
  measured input whose own accuracy is bounded by calibration and the sensing
  device (see [DATASET.md](DATASET.md) Sections 2.3 and 5.1). This metric
  therefore measures agreement with a calibration-bounded reference, not error
  against a perfect label.
- **Units:** degrees of visual angle.
- **Formula:** for a predicted unit gaze vector `ĝ` and reference unit
  vector `g`,

  ```text
  error_angular = arccos( clamp( ĝ · g, -1, 1 ) )   # radians
  error_deg     = error_angular * 180 / pi
  ```

- **Aggregation:** report the per-frame mean and median across the test set,
  plus per-subject means (Section 5). Median is reported alongside the mean
  because angular error distributions are right-skewed. Because the reference
  itself carries calibration error, subject-level results inherit that error;
  sessions with poor calibration validation should be filtered or flagged.
- **Notes:** Angular error is the device-independent gaze metric and the
  primary headline for **Phase 2 — Baseline Gaze Models**. It does not depend
  on screen geometry, so it is comparable across capture setups when the
  camera intrinsics are known. The reference gaze is a measured input, not a
  gold standard; how it is captured and the accuracy limits it inherits from
  calibration and the sensing device are documented in
  [DATASET.md](DATASET.md) Sections 2.3 (gaze as a measured input) and 5.1
  (no ground-truth gaze claim).
- **Phase:** [planned] Phase 2.

### 2.2 Screen-Coordinate Error (Point-of-Regard)

- **What:** Distance between the predicted on-screen gaze point (point-of-regard)
  and the **reference (measured) gaze target** on the displayed screen content.
  As in Section 2.1, this reference is a calibration-bounded measured input, not
  a gold-standard label (see [DATASET.md](DATASET.md) Sections 2.3 and 5.1), so
  reported error is agreement with that reference and inherits its calibration
  error.
- **Units:** reported in **two** forms:
  - **pixels** — Euclidean distance in display pixels.
  - **normalized** — the same distance divided by a stated reference (screen
    diagonal, or width/height), so results compare across resolutions. State
    which normalization is used; default is fraction of screen diagonal.
- **Formula:** for predicted point `(x̂, ŷ)` and target `(x, y)` in pixels,

  ```text
  error_px   = sqrt( (x̂ - x)^2 + (ŷ - y)^2 )
  error_norm = error_px / diag_px      # diag_px = sqrt(W^2 + H^2)
  ```

  Optionally also report error in physical units (mm) or degrees when the
  screen size and viewing distance are recorded, to connect screen error back
  to angular error.
- **Aggregation:** per-frame mean and median; per-subject means (Section 5). Always report
  the display resolution and physical size used, since pixel error is
  meaningless without them.
- **Notes:** This is the user-facing accuracy of where ARGUS thinks the
  operator is looking. It couples gaze estimation with screen geometry, so it
  is sensitive to calibration quality and head pose.
- **Phase:** [planned] Phase 2; refined under Phase 3 with screen context.

### 2.3 Context-Awareness Accuracy

- **Task definition (must be fixed before reporting):** Given the webcam frame,
  the synchronized screen context, and recent interaction history, predict the
  **screen region / context element the operator is attending to** — i.e.,
  whether the gaze point falls on the semantically correct element of the
  current screen (for example, the active panel, the alert, or the target
  under inspection) rather than merely a correct pixel.
- **Label source:** region/element labels derived from screen context plus the
  task protocol of CASEset (see [DATASET.md](DATASET.md)). The exact label
  taxonomy is **[planned]** and will be frozen and versioned before any number
  is reported; until then this metric has no defined value.
- **Units:** accuracy (fraction in `[0, 1]`) or percentage. Because element
  classes are typically imbalanced, **also** report macro-averaged
  precision/recall/F1 and a confusion matrix over context classes.
- **Formula (top-1 accuracy):**

  ```text
  accuracy = (# frames where predicted region == labeled region) / (# frames)
  ```

- **Notes:** This metric is what separates ARGUS's **context-aware** direction
  (Phase 3) from raw gaze regression (Phase 2). A model can have low
  screen-coordinate error yet poor context accuracy near element boundaries,
  and vice versa; both are reported.
- **Phase:** [planned] Phase 3 — Context-Aware Attention Models.

### 2.4 Latency

- **What:** Wall-clock time to produce an output, measured both **end-to-end**
  and **per-stage** along the pipeline described in
  [ARCHITECTURE.md](ARCHITECTURE.md) (e.g., capture/ingest → context feature
  extraction → gaze/attention model → post-processing/output).
- **Units:** milliseconds (ms). Throughput may additionally be reported in
  frames per second (FPS).
- **What to report:**
  - End-to-end latency distribution: mean, median, **p95**, and **p99**.
  - Per-stage median and p95, so bottlenecks are visible.
  - The hardware, batch size, input resolution, and whether timing is
    warm (steady-state) or cold (includes load) — latency is meaningless
    without them.
- **Formula:** `latency_e2e = t_output - t_input_available`, measured per
  sample at steady state; per-stage times sum to (approximately) end-to-end
  minus queueing.
- **Notes:** Tail latency (p95/p99), not the mean, governs whether attention
  signals are usable in an interactive loop. Report tails.
- **Phase:** [planned] Phase 2 onward; meaningful once an inference pipeline
  exists (Phase 2+).

### 2.5 Robustness

- **What:** How much accuracy degrades under conditions that differ from the
  easy/average case. Robustness is reported as **stratified** metrics, not a
  single scalar: compute the Section 2.1–2.3 metrics within each slice and
  report the spread and worst slice.
- **Slices (at minimum):**
  - **Lighting** — e.g., low-light vs. well-lit vs. backlit/uneven.
  - **Head pose** — frontal vs. yawed/pitched beyond stated thresholds.
  - **Users** — per-subject breakdown, including the worst-performing subject
    (not just the mean), and demographic/appearance slices where consent and
    [ETHICS.md](ETHICS.md) permit.
  - **Distribution shift** — unseen sessions, unseen capture conditions, and
    (when available) data outside CASEset's collection protocol.
- **Units:** same units as the underlying metric, plus a **degradation**
  figure: `Δ = metric(slice) - metric(overall)` (or a ratio), and the
  worst-slice value.
- **Notes:** A model is only as trustworthy as its worst realistic slice.
  Headline numbers must always be accompanied by the worst-slice number.
- **Phase:** [planned] Phase 2 onward; expands with each phase.

### 2.6 Failure Cases (Qualitative Taxonomy)

Quantitative metrics hide *how* a model fails. Every evaluation includes a
qualitative failure review that bins errors into a taxonomy. The taxonomy is
**[planned]** and will be refined with real data, but the initial bins are:

- **Sensing failures** — face/eye not detected, occlusion (hand, glasses
  glare), motion blur, dropped or desynchronized streams.
- **Geometric failures** — extreme head pose, off-axis seating, miscalibration,
  unrecorded display change.
- **Context failures** — gaze near element boundaries, ambiguous screen layout,
  rapid context switches, stale screen context.
- **Temporal failures** — saccade vs. fixation confusion, latency-induced lag,
  blink artifacts.
- **Distribution failures** — lighting/appearance/hardware unlike training data.

For each bin, report frequency (share of errors) and representative,
**consent-compliant** examples (see [ETHICS.md](ETHICS.md)). Privacy
constraints in [DATASET.md](DATASET.md) govern what imagery may be shown.

- **Phase:** [planned] Phase 2 onward.

---

## 3. Data Splits & Protocol

Splits are defined over CASEset subjects and sessions (schema in
[DATASET.md](DATASET.md)). Exact subject/session counts are **`<TBD>`** until
the dataset is finalized; the *protocol* below is fixed regardless of counts.

### 3.1 Subject-Independent (primary)

- Partition **by subject**: each subject appears in exactly one of
  {train, validation, test}.
- This measures generalization to **new people** and is the headline regime for
  all gaze/attention metrics.
- No calibration on test subjects unless explicitly reported as a separate
  "calibrated" condition (Section 3.4).

### 3.2 Subject-Dependent (secondary)

- The same subjects appear in train and test, but split **by session or by
  time** within subject so no session crosses the boundary.
- This measures the achievable accuracy when the system has seen the user
  before (a realistic deployment assumption for a personal operator station).
- Always reported **alongside**, never instead of, subject-independent results,
  and clearly labeled — subject-dependent numbers are optimistic for new users.

### 3.3 Cross-Session

- Train and test on **different sessions** of the (possibly same) subjects,
  separated in time, to measure stability across sessions (appearance,
  lighting, seating, hardware drift).
- Reported as a robustness slice (Section 2.5) and/or a standalone condition.

### 3.4 Calibration Conditions

When a model supports per-user calibration, report **both**:

- **Uncalibrated** — no test-subject data used at all.
- **Calibrated** — a small, explicitly sized block of the test subject's *own*
  held-out calibration data is used; report how many calibration samples/points
  were used. Calibration data is never shared with training and never
  overlaps the test evaluation frames.

### 3.5 Anti-Leakage Rules

- No frame, session, or subject appears in more than one split.
- Temporally adjacent frames from one session never straddle the split boundary
  (avoid near-duplicate leakage from high frame rates).
- Preprocessing statistics (normalization, PCA, etc.) are fit on training data
  only and applied to validation/test.
- Hyperparameters and model selection use the **validation** split only; the
  **test** split is touched once, for the final reported number.
- Split definitions are versioned files in [configs/](../configs/) so a result
  can be reproduced exactly.

---

## 4. Baselines

A result is only meaningful relative to a credible baseline. For each task,
report ARGUS against at least:

- **Trivial / chance baselines** — for screen-coordinate error, predicting the
  screen center or the per-subject mean gaze location; for classification
  (context-awareness), the majority-class and stratified-random predictors.
  These set the floor.
- **Geometric / classical baselines** — a standard appearance- or
  model-based gaze estimator (e.g., a published open-source gaze pipeline) run
  under the *same* splits. This shows whether learned context helps at all.
- **Ablations of ARGUS itself** — gaze-only vs. gaze+screen-context vs.
  gaze+context+interaction history, so the contribution of each input stream
  (see [ARCHITECTURE.md](ARCHITECTURE.md)) is isolated.
- **Reported baselines, re-run not quoted** — external numbers are only
  comparable when re-run on CASEset under this protocol. Numbers lifted from
  other papers/datasets are cited for context but **not** presented as
  head-to-head comparisons.

Every baseline runs under the identical split, preprocessing, and metric code
as ARGUS. Differences in data or protocol are documented.

---

## 5. Reporting Standards

These standards apply to any document, README, table, or notebook that reports
an ARGUS result.

- **Means with confidence intervals.** Report the mean **and** a 95% confidence
  interval (or bootstrap CI) for every headline metric. State the method used
  to compute the interval and the unit of resampling (per-subject is preferred
  over per-frame, since frames within a subject are correlated).
- **Per-subject breakdowns.** Include a per-subject table for the primary
  metrics, and explicitly report the **worst** subject, not only the average.
- **Distributions, not just centers.** Report median alongside mean for skewed
  metrics (angular error, latency) and p95/p99 for latency.
- **Full context.** State dataset version, split definition (with config path),
  number of subjects/sessions/frames evaluated, hardware, model/config version,
  and random seed(s).
- **Honest labeling.** Tag each number **[current]**, **[planned]**, or
  **[hypothesis]**. Only numbers actually computed on held-out CASEset data may
  be **[current]**.
- **No cherry-picking.** Report the pre-registered primary metric on the test
  split as the headline; additional favorable cuts are clearly secondary.
- **Reproducibility footer.** Each results table links the config and the
  commit that produced it.

### Suggested results-table shape (illustrative; values are placeholders)

```text
Task: Angular Gaze Error (deg), subject-independent, CASEset v<TBD>
Model              | Mean ± 95% CI | Median | Worst subj | n_subj | n_frames
-------------------+---------------+--------+------------+--------+---------
Center baseline    |   <TBD>       | <TBD>  |  <TBD>     | <TBD>  | <TBD>
Classical gaze     |   <TBD>       | <TBD>  |  <TBD>     | <TBD>  | <TBD>
ARGUS gaze-only    |   <TBD>       | <TBD>  |  <TBD>     | <TBD>  | <TBD>
ARGUS +context     |   <TBD>       | <TBD>  |  <TBD>     | <TBD>  | <TBD>
```

---

## 6. Future Evaluation Goals

The metrics below are **[planned]/[hypothesis]** and **not yet defined in
detail**. They are listed so the protocol can grow with the program, and they
are tied to the relevant phases in [ROADMAP.md](ROADMAP.md). None should be
treated as a committed metric until it has its own specification in this
document.

- **Intent inference (Phase 4 — [research]).** Evaluating whether attention
  patterns predict operator intent will require an intent label schema, a
  ground-truth elicitation method, and a baseline. All three are **[planned]**
  and undefined here. Likely directions: classification metrics against
  task-derived intent labels, plus calibration of predicted confidence. This is
  a **[hypothesis]** until data shows intent is recoverable at all.
- **Sensor cueing (Phase 5 — [research]).** Evaluation would measure the value
  of attention-derived cues to a downstream consumer (e.g., did the cue point
  at the right region earlier/more reliably than a default policy). Metric
  definitions are **not yet defined**.
- **Human-machine teaming (Phase 6 — [research]).** Would require
  human-in-the-loop studies with task-level outcome metrics (e.g., operator
  workload, time-to-decision, error rate) under appropriate oversight and
  consent per [ETHICS.md](ETHICS.md). Study design is **[planned]**.
- **Mission support systems (Phase 7 — [research / long-term]).** System-level
  evaluation is out of scope for the current protocol and intentionally left
  undefined.

When any of these graduates from idea to measurable task, it gets a full
metric definition in Section 2 and a split protocol in Section 3 before any
number is reported.

---

## 7. Summary

- No results exist yet; this document defines the **protocol**, not outcomes.
- Headline regime is **subject-independent** with **no cross-session/subject
  leakage**.
- Core metrics: **angular gaze error (deg)**, **screen-coordinate error
  (px / normalized)**, **context-awareness accuracy**, **latency (ms,
  per-stage + tails)**, **robustness (stratified, worst-slice)**, and a
  **qualitative failure taxonomy**.
- Every number ships with **uncertainty**, **per-subject breakdowns**, full
  **context**, and an honest **[current]/[planned]/[hypothesis]** label.
- See [DATASET.md](DATASET.md), [ARCHITECTURE.md](ARCHITECTURE.md), and
  [ROADMAP.md](ROADMAP.md) for the data, system stages, and phasing this
  protocol depends on.
