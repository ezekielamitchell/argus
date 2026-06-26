# ARGUS — Project Overview

**ARGUS** — Adaptive Reconnaissance and Gaze Understanding System.

ARGUS is an early-stage, open-source research and engineering program
investigating whether **operator attention** — where a human operator looks, and
the temporal structure of that looking — can serve as a usable signal for
autonomous systems.

This document briefs an engineering audience on *why* the program exists, *what
gap* it targets, *what it intends to build and learn*, and *the questions that
will decide whether the core idea holds up*. It is deliberately candid about
maturity: ARGUS is **not** a finished product. Its only current concrete asset
is the **CASEset** dataset (see [DATASET.md](DATASET.md)). Everything beyond
Phase 1 is planned engineering or a research hypothesis.

Throughout this repository, every claim that could be mistaken for a result is
labeled as one of **[current]**, **[planned]**, or **[hypothesis]**.

**Related reading:**
[ROADMAP.md](ROADMAP.md) (phased plan) ·
[RESEARCH_DIRECTION.md](RESEARCH_DIRECTION.md) (what makes ARGUS different) ·
[ARCHITECTURE.md](ARCHITECTURE.md) (inputs, processing, outputs) ·
[EVALUATION.md](EVALUATION.md) ·
[ETHICS.md](ETHICS.md)

---

## 1. Motivation

Autonomous systems that support human operators face a recurring bottleneck:
the machine has abundant access to *its own* sensor feeds but very little access
to the operator's *internal state* — what the human has noticed, what they are
prioritizing, what they are worried about, and what they have not yet seen. The
operator's intent is usually communicated through a thin, lossy channel of
explicit commands (clicks, keystrokes, voice). Most of the operator's
situational understanding never reaches the system at all.

Human visual attention is an unusually rich and underused side channel for that
understanding. A few reasons it is worth studying as a signal:

- **Attention precedes action.** People typically look at a thing before they
  act on it. The temporal gap between fixation and command is a window in which
  a supporting system could pre-stage information, surface context, or flag a
  conflict — *if* it could read attention reliably. *[hypothesis]*
- **Attention encodes priority.** Where an operator dwells, returns, and
  re-checks is a proxy for what they consider important under time pressure.
  Dwell time, revisit frequency, and scan-path structure are quantitative and
  loggable. *[hypothesis]*
- **Attention reveals gaps.** What an operator has *not* looked at is as
  informative as what they have. A region of the screen that never receives
  fixation during a critical interval is a candidate for machine cueing.
  *[hypothesis]*
- **The sensing is cheap and amenable to privacy-by-construction.** A commodity
  webcam, screen context, and interaction logs are low-cost and already present
  at most operator workstations. This lowers the barrier to study the signal
  without specialized eye-tracking hardware. *[current — the CASEset capture
  tooling already records these streams; downstream inferences remain
  hypotheses.]*

**Core hypothesis (stated as a hypothesis, not a fact):** human attention
carries information about intent, threat awareness, priorities, and situational
understanding. If that information can be reliably and ethically extracted from
low-cost sensing, it could improve how autonomous systems support human
operators.

Why this matters now, and why for a defense-autonomy audience specifically:
the field is investing heavily in autonomy that *teams with* humans rather than
replacing them. Companies such as Anduril, Shield AI, Saronic, and Scout AI are
useful **reference points** for why operator-facing autonomy is a live problem
space. ARGUS claims **no** relationship, endorsement, evaluation, or deployment
with any of them; they are cited only to locate the problem, not the project.
The teaming problem they all share — keeping a human meaningfully in the loop as
autonomy scales — is precisely where an attention signal, if it works, would be
useful.

---

## 2. Problem Statement

There is a gap between **gaze estimation** and **attention understanding**, and
that gap is where the hard, interesting, and unsolved problems live.

**Gaze estimation** answers a geometric question: *given an image of a face or
eyes (and optionally screen geometry), where on the screen is this person
looking right now?* It is an active research area with established benchmarks
and reasonable webcam-based accuracy. It produces a point (or a distribution
over points) per frame.

**Attention understanding** answers a cognitive and temporal question: *given a
stream of gaze over time, plus what was actually on the screen, what does this
operator's looking behavior mean?* That includes:

- Segmenting raw gaze into fixations, saccades, smooth pursuit, and noise.
- Binding fixations to **semantic** screen content (a track, an alert, a map
  region, a control), not just pixel coordinates — which requires the screen
  *context*, not only the gaze point.
- Characterizing temporal structure: dwell, revisits, scan paths, search vs.
  monitoring vs. confirmation behavior.
- Inferring higher-order state — priority, threat awareness, confusion,
  workload — from that structure.

A point estimate of gaze, however accurate, does not answer any of the questions
in the second list. **Coordinates are not understanding.** A system that knows
the operator's gaze landed at pixel `(x, y)` still does not know whether they
*recognized* the object there, whether they consider it a threat, whether they
are still thinking about it, or whether they have moved on. Closing that gap
requires fusing gaze with **screen context** and **interaction history** over
**time** — which is a modeling problem, not a calibration problem.

**Why this matters for human-machine teaming.** Effective teaming depends on
*shared situational awareness*: the machine and the human each having a usable
model of what the other knows and is doing. Today the machine's model of the
human is almost empty. If attention understanding can populate even a coarse,
calibrated, uncertainty-aware model of operator state, a supporting system could:

- cue sensors or analytics toward regions the operator is attending to (or
  conspicuously *not* attending to);
- suppress redundant alerts for content the operator has already visually
  acknowledged;
- detect divergence between operator focus and mission-critical events;
- hand off and escalate more gracefully because it has a temporal record of
  operator engagement.

Every one of those applications is **[hypothesis]** or **[planned]** today. The
problem statement is not "we built this"; it is "here is the specific gap, here
is why it is worth closing, and here is the empirical foundation we are starting
from." That foundation is CASEset.

---

## 3. Objectives

Objectives are split into **near-term** work that is current or concretely
planned, and **long-term** work that remains a research hypothesis. The phase
labels match [ROADMAP.md](ROADMAP.md) exactly.

### 3.1 Near-term objectives — [current] and [planned]

- **N1. Solidify the CASEset foundation — [current], Phase 1.**
  Document collection methodology, the multi-stream schema (synchronized webcam
  frames, screenshots, gaze coordinates, interaction logs), synchronization
  guarantees, privacy/consent posture, and known limitations. Provide tooling to
  load and align the streams reproducibly. See [DATASET.md](DATASET.md).

- **N2. Establish reproducible baseline gaze models — [planned], Phase 2.**
  Implement and evaluate webcam-based gaze estimation baselines against CASEset
  with a fixed, documented protocol so later context-aware models have an honest
  reference point. Metrics and protocol live in [EVALUATION.md](EVALUATION.md).

- **N3. Build the multi-stream data and context layer — [planned], Phase 2.**
  Deliver dataset loaders, temporal synchronization utilities, and screen/context
  feature extraction (the `argus/data` and `argus/context` packages in the
  [ARCHITECTURE.md](ARCHITECTURE.md) layout) as the substrate everything else
  depends on.

- **N4. Define evaluation before claiming results — [planned], Phase 2.**
  Specify metrics, splits, and reporting conventions up front, including how
  uncertainty and failure cases are reported, so that no capability is asserted
  without a measurement behind it.

- **N5. Make ethics and privacy a first-class engineering constraint —
  [current/ongoing].**
  Treat consent, data minimization, oversight, and responsible-use boundaries as
  design inputs, not afterthoughts. See [ETHICS.md](ETHICS.md).

### 3.2 Long-term objectives — [research]

- **L1. Context-aware attention models — [research], Phase 3.**
  Move from "where is the gaze point" to "what screen content is being attended,
  and how" by jointly modeling gaze, screen context, and time.

- **L2. Intent inference — [research], Phase 4.**
  Investigate whether operator intent and priority can be inferred from attention
  structure with calibrated, useful uncertainty — and characterize where it
  cannot.

- **L3. Sensor cueing — [research], Phase 5.**
  Study whether attention-derived signals can usefully cue downstream sensing or
  analytics, and under what conditions cueing helps versus harms.

- **L4. Human-machine teaming — [research], Phase 6.**
  Examine how an attention-aware support system changes shared situational
  awareness, trust, and operator workload.

- **L5. Mission support systems — [research / long-term], Phase 7.**
  The conceptual end direction, not a committed deliverable:
  `Operator -> Attention -> ARGUS -> Context Understanding -> Mission Support ->
  Autonomous System`. This is a *direction*, not a built end-to-end system.

---

## 4. Expected Outcomes

What the program aims to produce, staged honestly. Producing these artifacts is
the goal; *demonstrating* the downstream capabilities is a research bet, not a
promise.

**Artifacts — near-term ([current] / [planned]):**

- A documented, reproducible **CASEset** dataset with a published schema, sync
  methodology, and explicit limitations. *[current — Phase 1.]*
- A **baseline gaze benchmark** on CASEset with a frozen evaluation protocol and
  reported uncertainty. *[planned — Phase 2.]*
- A reusable **multi-stream data + context-extraction library**
  (`argus/data`, `argus/context`) and supporting `scripts/`, `configs/`, and
  `tests/`. *[planned — Phase 2.]*
- Clear **evaluation and ethics documentation** that others can audit and
  reproduce. *[current/ongoing.]*

**Artifacts — long-term ([research]):**

- Context-aware attention model implementations and their evaluations.
  *[research — Phase 3.]*
- Intent-inference experiments with calibration analysis and documented failure
  modes. *[research — Phase 4.]*
- Sensor-cueing and teaming studies, including negative results where the signal
  does not generalize. *[research — Phases 5–6.]*

**Knowledge — the more durable output:**

- Evidence on whether low-cost sensing yields an attention signal that is stable
  enough to act on, and at what spatial/temporal resolution.
- A characterization of *where the gap between gaze and attention can actually be
  closed* and where it cannot.
- Honest **negative results.** If attention does not reliably carry actionable
  intent under these conditions, that is a publishable, valuable outcome and will
  be reported as such.

The most likely realistic outcome at this stage is a well-documented dataset, a
set of reproducible baselines, and a sharper understanding of which research
questions below are tractable. That is the bar this program is held to.

---

## 5. Research Questions

These are the questions that decide whether ARGUS's premise holds. They are
written to be sharp and, where possible, falsifiable. None is answered yet; each
points at planned or research-stage work.

1. **Signal existence.** Can webcam-based gaze, fused with screen context,
   recover *semantic* attention (which on-screen entity is being attended) at an
   accuracy materially better than chance and better than gaze coordinates
   alone? *Falsifiable against a labeled CASEset benchmark.*

2. **Context contribution.** How much does adding screen context improve
   attention attribution over a context-free gaze baseline? *Measurable as a
   delta on a fixed metric; a null or negative delta falsifies the premise of
   Phase 3.*

3. **Temporal structure.** Do temporal features (dwell, revisit rate, scan-path
   shape) carry information beyond instantaneous gaze for distinguishing
   operator behaviors such as searching vs. monitoring vs. confirming?

4. **Intent predictiveness.** Does attention structure predict the operator's
   *next* action earlier and/or more accurately than interaction logs alone?
   *Falsifiable: compare lead time and accuracy of an attention-based predictor
   against an interaction-only predictor.*

5. **Calibration.** Can intent/priority inferences be produced with *calibrated*
   uncertainty, such that the system reliably knows when it does not know?

6. **Negative-space value.** Is "what the operator did *not* attend to" a useful
   predictor of missed events, and can it cue a support system without
   unacceptable false-alarm rates?

7. **Robustness and generalization.** How do gaze and attention estimates degrade
   under realistic conditions (lighting, head pose, occlusion, fatigue, off-axis
   cameras), and do models trained on CASEset generalize beyond its collection
   conditions?

8. **Individual variation.** How much do attention-to-intent mappings vary across
   operators, and how much per-operator calibration is required before the signal
   is usable?

9. **Cueing utility.** When attention-derived cues are fed to a downstream
   system, do they measurably improve a task outcome, leave it unchanged, or
   degrade it — and under what conditions does each occur?

10. **Teaming effect.** Does an attention-aware support layer improve shared
    situational awareness and operator workload without inducing over-trust or
    automation complacency?

11. **Ethical envelope.** Which uses of an operator-attention signal are
    consistent with informed consent, proportionality, and human oversight, and
    which fall outside a responsible-use boundary regardless of technical
    feasibility? (See [ETHICS.md](ETHICS.md).)

---

## 6. Where to go next

- **The plan and phasing:** [ROADMAP.md](ROADMAP.md) — Phases 1–7 in detail.
- **What makes ARGUS distinct and the open problems:**
  [RESEARCH_DIRECTION.md](RESEARCH_DIRECTION.md).
- **How the pieces fit (inputs, processing, outputs):**
  [ARCHITECTURE.md](ARCHITECTURE.md).
- **The empirical foundation:** [DATASET.md](DATASET.md) (CASEset).
- **How claims will be measured:** [EVALUATION.md](EVALUATION.md).
- **Privacy, consent, oversight, responsible use:** [ETHICS.md](ETHICS.md).

---

*ARGUS is licensed under the MIT License, © 2026 Ezekiel A. Mitchell.
Primary language: Python. This repository is early-stage; most components
described here are planned or research-stage, and are labeled accordingly.*
