# ARGUS Roadmap

ARGUS — Adaptive Reconnaissance and Gaze Understanding System — is an
early-stage, open-source research and engineering program investigating whether
**operator attention** (where a human operator looks, and the temporal structure
of that attention) can serve as a usable signal for autonomous systems. This
roadmap describes how the program is sequenced from its current foundation
through its longer-term research ambitions.

## How to read this roadmap

- **Phases are sequenced, not strictly serial.** Each phase builds on the
  artifacts of the ones before it, but the later research phases (Phase 4
  onward) are expected to run *in parallel* and to feed back into one another.
  Intent inference, sensor cueing, and human-machine teaming are interdependent
  research questions, not a tidy assembly line.
- **Dates are intentionally omitted.** ARGUS is early-stage and largely
  unbuilt. Committing to calendar dates now would be guesswork. Instead, each
  phase is defined by its *objectives*, its concrete *outputs*, and the
  *evaluation criteria* that would tell us the phase actually succeeded. Progress
  is measured against those criteria, not a schedule.
- **Status labels are load-bearing.** Every phase is tagged `[current]`,
  `[planned]`, or `[research]`. These labels separate what exists today from
  what is engineering intent and what is an open research hypothesis. See the
  [Status Legend](#status-legend) at the end.
- **The only concrete asset today is CASEset.** Everything beyond Phase 1 is
  planned engineering or a research hypothesis. This honesty is deliberate.

The conceptual direction the program is working toward — *not* a built
end-to-end system — is:

```
Operator -> Attention -> ARGUS -> Context Understanding -> Mission Support -> Autonomous System
```

For motivation and research questions, see
[PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md). For the dataset that grounds all
of this, see [DATASET.md](./DATASET.md). For metrics and protocol, see
[EVALUATION.md](./EVALUATION.md). For privacy and responsible-use commitments,
see [ETHICS.md](./ETHICS.md).

---

## Phase 1 — CASEset Dataset Foundation `[current]`

The empirical foundation of the program: a multi-stream dataset of synchronized
webcam frames, screenshots, gaze coordinates, and interaction logs produced by
prior work. CASEset is the only concrete asset ARGUS currently has, and this
phase is about making it documented, reproducible, and usable by others.

### Objectives

- Consolidate the CASEset multi-stream recordings (webcam imagery, screen
  context, gaze coordinates, interaction/temporal logs) into a single,
  documented dataset.
- Define a stable on-disk **schema** and synchronization model that aligns the
  streams to a common timeline.
- Establish privacy, consent, and data-handling practices before any modeling
  work begins (see [ETHICS.md](./ETHICS.md)).
- Document collection methodology with enough detail to reproduce or extend the
  capture process.

### Outputs

- `caseset/` tooling for collection, stream synchronization, and schema
  validation.
- [DATASET.md](./DATASET.md): a datasheet covering provenance, collection
  methodology, stream schema, sampling, known limitations, and privacy posture.
  Where a precise statistic (participant count, frame count, hours, resolution,
  fps) is not yet confirmed, it is recorded as `<TBD>` rather than invented.
- A documented directory layout and file naming convention for synchronized
  streams.
- Data-loading and integrity-check utilities (entry points under `scripts/`,
  with loaders later promoted into `argus/data/`).

### Evaluation Criteria

- A new contributor can load CASEset and reconstruct synchronized
  (webcam, screen, gaze, interaction) tuples from the documentation alone.
- Stream synchronization is verifiable: timestamps align within a stated
  tolerance, and the validation tooling flags drift or missing frames.
- The dataset schema is stable enough that downstream code can depend on it.
- Privacy and consent handling are documented and consistent with
  [ETHICS.md](./ETHICS.md).

---

## Phase 2 — Baseline Gaze Models `[planned]`

Establish honest, reproducible baselines for gaze estimation from CASEset
inputs. The goal is a measuring stick, not state of the art.

### Objectives

- Implement standard, well-understood gaze-estimation baselines that map webcam
  imagery to gaze coordinates.
- Define train / validation / test splits on CASEset that avoid subject and
  session leakage.
- Stand up the core Python package (`argus/`) so models, data loaders, and
  evaluation share common interfaces.

### Outputs

- Baseline model implementations under `argus/models/`, with data loaders in
  `argus/data/` consuming the CASEset schema from Phase 1.
- Reproducible training and evaluation entry points under `scripts/`, driven by
  versioned configs under `configs/`.
- An initial evaluation harness in `argus/evaluation/` reporting the core gaze
  metrics defined in [EVALUATION.md](./EVALUATION.md).
- A baseline results table with documented splits, seeds, and hardware.

### Evaluation Criteria

- Baselines reproduce to within a stated variance from a clean checkout using
  the published configs.
- Gaze error (e.g., angular / on-screen positional error per
  [EVALUATION.md](./EVALUATION.md)) is reported on held-out subjects, not just
  held-out frames.
- Results are comparable across runs because splits, seeds, and configs are
  version-controlled.
- The package interfaces (data, model, evaluation) are stable enough for Phase 3
  to extend without rewrites.

---

## Phase 3 — Context-Aware Attention Models `[planned]`

Move beyond geometry-only gaze by incorporating **screen context** — what the
operator is actually looking at — into attention estimation. This is where
ARGUS's central premise starts to be tested.

### Objectives

- Extract features from screen context (screenshots) that describe what content
  occupies regions of the display.
- Combine gaze with screen context and temporal behavior to model *attention*
  (sustained, structured looking) rather than instantaneous gaze points.
- Measure whether context improves attention estimates over the Phase 2
  geometry-only baselines.

### Outputs

- Screen/context feature extraction under `argus/context/`.
- Context-aware attention model implementations under `argus/attention/`
  (and `argus/models/` as appropriate).
- Configs and scripts for context-aware experiments, plus analysis notebooks in
  `notebooks/`.
- Extended evaluation reporting attention-level metrics (e.g., region-of-interest
  attention, temporal stability) alongside the Phase 2 baselines.

### Evaluation Criteria

- Context-aware models show a measurable, statistically defensible improvement
  over geometry-only baselines on held-out subjects — or the negative result is
  reported plainly.
- Attention estimates are temporally coherent, not frame-to-frame noise.
- Ablations isolate the contribution of screen context from gaze geometry and
  from temporal modeling.
- Improvements generalize across operators and sessions, not just within a
  single subject.

---

## Phase 4 — Intent Inference `[research]`

**Hypothesis:** human attention carries information about intent, threat
awareness, priorities, and situational understanding. This phase tests whether
any of that can be inferred from CASEset-style signals — an open research
question, not a deliverable promise.

### Objectives

- Define what "intent" concretely means in this setting (e.g., target of
  interest, next likely action, priority ordering) in a way that is *labelable*.
- Investigate whether attention patterns from Phase 3 predict those defined
  intent signals above chance.
- Characterize the limits: which intents are inferable, which are not, and how
  confident the system can honestly be.

### Outputs

- Intent representations and modeling code under `argus/attention/`.
- A documented labeling protocol and (where feasible) intent annotations layered
  onto CASEset.
- Evaluation reporting calibration and uncertainty, not just point accuracy.
- A written account of findings — including negative or inconclusive results —
  in [RESEARCH_DIRECTION.md](./RESEARCH_DIRECTION.md).

### Evaluation Criteria

- Defined intent signals are predicted meaningfully above a well-specified
  chance baseline on held-out subjects.
- Predictions are **calibrated**: stated confidence matches observed reliability.
- The scope of what is and is not inferable is documented honestly.
- No claim of intent inference is made beyond what the held-out evaluation
  supports.

---

## Phase 5 — Sensor Cueing `[research]`

**Hypothesis:** inferred operator attention and intent can usefully *cue*
sensing — for example, suggesting where an autonomous system might look or
allocate resources next. Studied here in simulation and replay, not on any
deployed platform.

### Objectives

- Formulate sensor cueing as a decision problem driven by attention/intent
  estimates.
- Evaluate, in simulation or replay over CASEset-style data, whether
  attention-derived cues improve a measurable downstream sensing objective.
- Understand failure modes: stale cues, over-trust, and distribution shift.

### Outputs

- A cueing experiment harness (under `argus/pipeline/` and `scripts/`) that turns
  attention/intent estimates into candidate cues.
- Simulation/replay evaluation comparing attention-cued strategies against
  uncued and naive baselines.
- Documentation of assumptions, the simulation environment, and limitations.

### Evaluation Criteria

- Attention-derived cueing improves a clearly defined sensing objective over
  uncued baselines in simulation/replay.
- Benefits are robust to realistic noise and latency in attention estimates.
- Failure modes are characterized, with guidance on when cueing should be
  ignored.
- No claim is made about real-world or platform deployment — all results are
  explicitly simulation/replay.

---

## Phase 6 — Human-Machine Teaming `[research]`

**Hypothesis:** attention-aware support can improve how autonomous systems and
human operators work together — keeping the human in control while reducing
workload. This phase studies the *interaction*, with the operator firmly in the
loop.

### Objectives

- Design interaction patterns where ARGUS-derived context supports, rather than
  overrides, the operator.
- Study the effect of attention-aware support on operator workload, trust, and
  task performance.
- Keep human oversight and authority central, consistent with
  [ETHICS.md](./ETHICS.md).

### Outputs

- Teaming interaction prototypes built on the Phase 5 cueing pipeline.
- A human-in-the-loop evaluation protocol (task design, workload and trust
  measures) documented in [EVALUATION.md](./EVALUATION.md).
- Reported study results, including conditions where attention-aware support did
  *not* help.

### Evaluation Criteria

- Attention-aware support measurably improves a teaming outcome (e.g., task
  performance or workload) without degrading operator situational awareness.
- Operators retain clear authority and can override or disengage support.
- Results are reported with appropriate caveats given study scale and
  population.
- Ethical and oversight requirements from [ETHICS.md](./ETHICS.md) are met.

---

## Phase 7 — Mission Support Systems `[research / long-term]`

The long-horizon direction: integrating the validated components above into
coherent **mission support** that helps operators direct autonomous systems.
This is a *direction*, not a committed build, and depends entirely on the
results of the preceding research phases.

### Objectives

- Explore how attention modeling, intent inference, cueing, and teaming compose
  into end-to-end mission-support workflows.
- Identify which components are mature enough to integrate and which remain open
  research.
- Define what responsible, operator-centered mission support would require to be
  trustworthy.

### Outputs

- Integration experiments wiring validated components through
  `argus/pipeline/`.
- Reference workflows and a candidate end-to-end architecture, documented in
  [ARCHITECTURE.md](./ARCHITECTURE.md).
- A frank assessment of the gap between current results and any mission-support
  ambition.

### Evaluation Criteria

- An integrated workflow demonstrates value over its individual components on a
  defined, realistic task.
- The system keeps the operator in control and degrades gracefully when
  attention/intent estimates are unreliable.
- Claims are bounded strictly by demonstrated evidence; aspiration is labeled as
  such.
- The path from research prototype to anything beyond it is described honestly,
  with open problems named.

---

## Status Legend

| Label | Meaning |
| --- | --- |
| `[current]` | Work that is active now and grounded in an existing asset. As of this roadmap, only **Phase 1 (CASEset Dataset Foundation)** is current. |
| `[planned]` | Engineering that is intended and reasonably well understood, but **not yet built**. Scope is committed in direction; timelines are not. |
| `[research]` | An open research **hypothesis**. Whether it works is genuinely unknown, and a negative or inconclusive result is a legitimate, reportable outcome. |

Throughout the wider documentation, individual claims are additionally labeled
`[current]`, `[planned]`, or `[hypothesis]` wherever a reader might otherwise
mistake aspiration for fact. ARGUS is early-stage: most components described
beyond Phase 1 do not exist yet, and the roadmap is written to make that
unambiguous.
