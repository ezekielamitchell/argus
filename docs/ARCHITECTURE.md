# ARGUS Architecture

> **Status note.** This document describes the **intended** architecture of
> ARGUS (Adaptive Reconnaissance and Gaze Understanding System). It is a
> conceptual map of how the system is meant to fit together, not a description
> of working software. As of today, the only layer with concrete grounding is
> the **dataset and synchronization layer** built around the CASEset dataset
> ([Phase 1, current](ROADMAP.md)). Everything downstream — feature extraction,
> context modeling, temporal analysis, and all outputs — is `[planned]` or
> `[hypothesis]` and has not been implemented or demonstrated. Each component
> below is labeled accordingly. Where a claim could be mistaken for a
> demonstrated capability, it is marked `[planned]` or `[hypothesis]`.

ARGUS investigates a single research question: **can operator attention — where
a human operator looks, and the temporal structure of that looking — serve as a
usable signal for autonomous systems?** `[hypothesis]` The architecture
described here exists to make that question testable, reproducibly and
ethically, starting from low-cost sensing (webcam, screen context, and
interaction logs).

For the dataset this architecture consumes, see
[DATASET.md](DATASET.md). For how outputs are measured, see
[EVALUATION.md](EVALUATION.md). For the phased plan that schedules each
component, see [ROADMAP.md](ROADMAP.md).

---

## Overview

At the highest level, ARGUS is a pipeline that ingests synchronized
multi-stream operator data, extracts attention-related features, models the
operator's context over time, and emits attention and intent estimates that
could inform downstream autonomous systems.

The guiding conceptual direction — a *direction*, not a built end-to-end
system — is:

```
Operator -> Attention -> ARGUS -> Context Understanding -> Mission Support -> Autonomous System
```

Only the leftmost portion of this chain (capturing operator attention as data,
via CASEset) is concretely realized today. The architecture is deliberately
staged so that each layer can be built, validated, and replaced independently,
and so that honest evaluation is possible at every boundary.

Design principles:

- **Dataset-first.** Nothing downstream is trusted until it can be measured
  against CASEset. The synchronization layer is the foundation everything else
  depends on.
- **Stage isolation.** Each processing stage has a defined input/output
  contract so it can be swapped, ablated, or evaluated on its own.
- **Honest labeling.** Components are tagged `[current]`, `[planned]`, or
  `[hypothesis]`; the architecture does not imply capabilities that do not
  exist.
- **Privacy by construction.** Because inputs include webcam imagery and screen
  content, data handling and consent constraints (see
  [DATASET.md](DATASET.md)) shape the architecture rather than being bolted on.

---

## Inputs

ARGUS studies four synchronized input streams. Each is a distinct sensing
modality with its own rate, representation, and privacy profile. All four are
present in CASEset; the synchronization of these streams is the one part of the
system that exists today `[current]`.

### Webcam frames
Low-cost RGB imagery of the operator's face and upper body, captured from a
commodity webcam. Used as the raw signal for head pose, eye region, and
ultimately gaze direction. Frame rate, resolution, and color format are dataset
properties — see [DATASET.md](DATASET.md); no fixed values are asserted here.
This is the most privacy-sensitive stream and is governed by the consent and
handling rules in [ETHICS.md](ETHICS.md). `[current]` as raw capture;
downstream use of the imagery for gaze is `[planned]`.

### Screenshots (screen context)
Periodic captures of what the operator is looking *at* — the on-screen content
that gives gaze coordinates their meaning. A gaze point is only informative
relative to the scene it lands on, so screen context is treated as a
first-class input rather than metadata. Screenshots may contain sensitive
content and are subject to the same privacy controls as webcam imagery.
`[current]` as raw capture; semantic context extraction is `[planned]`.

### Gaze coordinates
Per-timestamp estimates of where on the screen the operator is looking,
expressed in screen or normalized coordinates. In CASEset these are recorded
alongside the other streams; within ARGUS they serve both as a target signal
for baseline gaze models and as an input feature for higher-level attention and
context modeling. `[current]` as recorded data; model-produced gaze is
`[planned]`.

### User interactions
Temporal logs of operator behavior — keypresses, mouse movement and clicks,
cursor trajectories, focus changes, and the timing between them. These carry
information about workload, hesitation, and task structure that is not visible
in gaze alone. They are lightweight, low-cost, and non-imaging, which makes
them attractive as a complementary signal. `[current]` as recorded logs;
behavioral feature extraction is `[planned]`.

---

## Processing

Processing is organized into four stages. Stages run roughly left-to-right, but
context modeling and temporal analysis are expected to interact (attention over
time depends on context, and context interpretation depends on temporal
patterns). Only **Synchronization** has concrete grounding today.

### Synchronization `[current]`
Aligns the four input streams onto a common, well-defined timeline so that a
gaze coordinate, the screenshot it refers to, the webcam frame that produced
it, and the interaction events around it can all be associated with the same
moment. Because the streams are captured at different rates and with different
clocks, this stage handles timestamp normalization, resampling/interpolation
policy, and clock-offset correction. This is the empirical foundation of the
whole system and the part of the architecture that exists. Collection and
synchronization tooling lives under `caseset/`; the schema and method are
documented in [DATASET.md](DATASET.md).

### Feature Extraction `[planned]`
Turns raw aligned streams into model-ready features: head pose and eye-region
features from webcam frames, gaze-relative scene features from screenshots, and
behavioral features (timing, velocity, dwell) from interaction logs. This stage
defines the boundary between raw sensing and learned representation. None of
these extractors are implemented yet; they are scheduled with the baseline and
context model work in [ROADMAP.md](ROADMAP.md).

### Context Modeling `[planned]`
Combines gaze with screen content to answer *what* the operator is attending
to, not just *where* the gaze landed — for example, associating a gaze point
with a region, object, or interface element on screen. This is what
distinguishes "context-aware" gaze estimation from raw gaze regression. It
depends on the screenshot stream as a semantic input. `[planned]`, targeted by
the context-aware attention work in [ROADMAP.md](ROADMAP.md).

### Temporal Analysis `[hypothesis]`
Models how attention and context evolve over time — fixations, transitions,
revisits, scan patterns, and how these relate to workload and intent. The core
hypothesis is that the *temporal structure* of attention carries information
about priorities, threat awareness, and situational understanding that a single
frame cannot. This is the most speculative stage and is explicitly a research
hypothesis, not a demonstrated result.

---

## Outputs

The intended outputs sit at increasing levels of abstraction and increasing
uncertainty. None are produced by working software today; all are `[planned]`
or `[hypothesis]`. How each would be measured is defined in
[EVALUATION.md](EVALUATION.md).

### Attention estimates `[planned]`
Where the operator is attending, in context — gaze and fixation estimates
grounded against on-screen content. This is the nearest-term, most concrete
output and the first that can be evaluated against CASEset.

### Intent signals `[hypothesis]`
Inferred indications of operator intent, priority, or threat awareness derived
from attention and its temporal structure. This is a research hypothesis:
ARGUS does not currently infer intent, and whether intent can be reliably and
ethically recovered from attention is an open question
([RESEARCH_DIRECTION.md](RESEARCH_DIRECTION.md)).

### Sensor-priority recommendations `[hypothesis]`
The longest-horizon output: suggestions about where sensing or autonomous
attention might be directed, informed by the operator's modeled attention and
intent. This is conceptual only and is tied to the later research phases in
[ROADMAP.md](ROADMAP.md). ARGUS makes no claim of performing sensor cueing
today, and asserts no relationship with any deployed system or organization.

---

## Data-Flow Diagram

Top-level flow from inputs through processing to outputs. Solid boxes upstream
are grounded in CASEset; everything from Feature Extraction onward is intended,
not built.

```
            INPUTS                         PROCESSING                          OUTPUTS
  (CASEset streams, [current])      (synchronization [current],
                                     remainder [planned]/[hypothesis])

  +-------------------+
  |  Webcam frames    |---------+
  +-------------------+         |
  +-------------------+         |     +------------------+    +------------------+
  |  Screenshots      |---------+---->| Synchronization  |--->| Feature          |
  +-------------------+         |     | [current]        |    | Extraction       |
  +-------------------+         |     +------------------+    | [planned]        |
  |  Gaze coordinates |---------+                            +---------+--------+
  +-------------------+         |                                      |
  +-------------------+         |                                      v
  |  User interactions|---------+                            +------------------+
  +-------------------+                                       | Context Modeling |
                                                             | [planned]        |
                                                             +---------+--------+
                                                                       |
                                                                       v
                                                             +------------------+
                                                             | Temporal Analysis|
                                                             | [hypothesis]     |
                                                             +---------+--------+
                                                                       |
                  +----------------------------+-----------------------+
                  v                            v                       v
        +-------------------+      +-------------------+    +----------------------------+
        | Attention         |      | Intent signals    |    | Sensor-priority            |
        | estimates         |      | [hypothesis]      |    | recommendations            |
        | [planned]         |      +-------------------+    | [hypothesis]               |
        +-------------------+                               +----------------------------+
```

---

## Component Diagram

Mapping of processing stages and outputs onto the canonical `argus/` Python
package subfolders. The package is `[planned]` (Phase 2+); the layout below is
the intended home for each stage, not existing modules.

```
  Stage / concern                         argus/ subpackage
  ----------------------------------      ---------------------------------
  Dataset loading & stream sync     -->   argus/data/        [planned]*
  (CASEset tooling)                       caseset/           [current]

  Screen / context features         -->   argus/context/     [planned]
  Gaze & attention model impls      -->   argus/models/      [planned]
  Attention + intent modeling       -->   argus/attention/   [planned]
  End-to-end inference orchestration-->   argus/pipeline/    [planned]
  Metrics & evaluation harnesses    -->   argus/evaluation/  [planned]

  * argus/data/ is the package-side loader for the synchronized streams that
    caseset/ produces; the synchronization concept is grounded today, but the
    argus/ package code that wraps it is still [planned].
```

```
  +----------------------------------------------------------------+
  |                         argus/  (package)                      |
  |                                                                |
  |   argus/data/        argus/context/      argus/models/         |
  |   load + sync   ---> screen/context  ---> gaze + attention     |
  |   streams            features            model implementations |
  |        |                  |                      |             |
  |        +---------+--------+----------+-----------+             |
  |                  v                   v                         |
  |          argus/attention/      argus/pipeline/                 |
  |          attention + intent --> end-to-end inference           |
  |          modeling              orchestration                   |
  |                  |                   |                         |
  |                  +--------+----------+                         |
  |                           v                                    |
  |                   argus/evaluation/                            |
  |                   metrics + harnesses  --> EVALUATION.md        |
  +----------------------------------------------------------------+
              ^
              |
        caseset/  (collection, sync, schema tooling)  [current]
        data/     (local datasets & artifacts, git-ignored)
```

---

## Status of Components

| Component | Phase | Status |
|-----------|-------|--------|
| CASEset capture & synchronization (`caseset/`) | Phase 1 — CASEset Dataset Foundation | `[current]` — exists; the system's empirical foundation |
| Dataset loaders / stream sync API (`argus/data/`) | Phase 1–2 | `[planned]` — package wrapper around existing sync concept |
| Baseline gaze models (`argus/models/`) | Phase 2 — Baseline Gaze Models | `[planned]` — not yet implemented |
| Context feature extraction (`argus/context/`) | Phase 3 — Context-Aware Attention Models | `[planned]` — not yet implemented |
| Context-aware attention modeling (`argus/attention/`) | Phase 3 — Context-Aware Attention Models | `[planned]` — not yet implemented |
| Temporal attention analysis | Phase 3–4 | `[hypothesis]` — core research question, unproven |
| Intent inference (`argus/attention/`) | Phase 4 — Intent Inference | `[hypothesis]` — research only |
| Sensor cueing / sensor-priority recommendations | Phase 5 — Sensor Cueing | `[hypothesis]` — research / long-term |
| Human-machine teaming integration | Phase 6 — Human-Machine Teaming | `[hypothesis]` — research / long-term |
| Mission support systems | Phase 7 — Mission Support Systems | `[hypothesis]` — long-term direction |
| Inference pipeline (`argus/pipeline/`) | Phase 2+ | `[planned]` — orchestration, built incrementally |
| Evaluation harnesses (`argus/evaluation/`) | Phase 2+ | `[planned]` — see [EVALUATION.md](EVALUATION.md) |

---

## Related Documents

- [DATASET.md](DATASET.md) — CASEset collection, schema, privacy, and limits;
  the grounding for the input and synchronization layers.
- [EVALUATION.md](EVALUATION.md) — metrics and protocol for measuring every
  output described above.
- [ROADMAP.md](ROADMAP.md) — phased program plan (Phase 1–7) that schedules
  each component in this architecture.
- [RESEARCH_DIRECTION.md](RESEARCH_DIRECTION.md) — what makes ARGUS distinct and
  the open questions behind the `[hypothesis]` components.
- [ETHICS.md](ETHICS.md) — privacy, consent, and responsible-use constraints
  that shape how the input streams are handled.
