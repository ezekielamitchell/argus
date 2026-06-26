# ARGUS

**Adaptive Reconnaissance and Gaze Understanding System**

![Status](https://img.shields.io/badge/status-Phase%201%20%E2%80%94%20Foundation-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Type](https://img.shields.io/badge/type-Research-purple)

ARGUS is an early-stage, open-source research and engineering program investigating whether **operator attention** — where a human operator looks, and the temporal structure of that attention — can serve as a usable signal for autonomous systems. It is **not** a finished product: today its only concrete asset is the **CASEset** dataset, and everything beyond Phase 1 is planned engineering or an open research hypothesis. This repository is deliberately transparent about that status — candor about what is and is not built is a core value of the project.

## Why ARGUS Exists

Traditional gaze estimation answers a narrow question: *where* on a screen is a person looking? That coordinate is useful, but it is only the surface of the signal. ARGUS asks a different and harder question: what does that attention **mean**, and how can an autonomous system use it?

**Core hypothesis** *[hypothesis]*: Human attention carries information about intent, threat awareness, priorities, and situational understanding. If that information can be reliably and ethically extracted from low-cost sensing — a webcam, screen context, and interaction logs — it could improve how autonomous systems support human operators.

The distinction matters because attention is not just a pointer; it is a window into reasoning. An operator's gaze pattern over time — what they fixate on, what they revisit, what they ignore, and when — encodes priorities that a raw coordinate cannot. ARGUS treats that temporal structure as the object of study. Whether the hypothesis holds is exactly what the program intends to test, not assert.

Why this problem space matters can be seen in the broader move toward human-machine teaming in defense autonomy, where companies such as Anduril, Shield AI, Saronic, and Scout AI illustrate the demand for systems that collaborate with human operators rather than replace them. ARGUS has **no** relationship with, endorsement from, or evaluation by any of these organizations; they are reference points for relevance, nothing more.

## Research Areas

- **Context-aware gaze estimation** — gaze prediction that incorporates on-screen context, not just eye geometry.
- **Human-machine teaming** — how attention can mediate collaboration between operators and autonomous systems.
- **Intent inference** — deriving operator goals and priorities from attention and interaction patterns.
- **Operator attention modeling** — representing the temporal structure of where and how operators attend.
- **Sensor cueing** — using inferred attention to direct or prioritize downstream sensing.
- **Autonomous reconnaissance** — applying attention-derived signals to reconnaissance tasks.
- **Defense AI** — responsible application of the above to defense autonomy problems.

## System Vision

```
        +------------------------+
        |        Operator        |
        +------------------------+
                    |
                    v
        +------------------------+
        |       Attention        |   where + temporal structure
        +------------------------+
                    |
                    v
        +------------------------+
        |         ARGUS          |   sensing + modeling
        +------------------------+
                    |
                    v
        +------------------------+
        | Context Understanding  |   what the attention means
        +------------------------+
                    |
                    v
        +------------------------+
        |    Mission Support     |
        +------------------------+
                    |
                    v
        +------------------------+
        |   Autonomous System    |
        +------------------------+
```

*Conceptual direction only — this is the program's intended trajectory, not a shipped end-to-end pipeline. Only the foundation layer (CASEset) currently exists.*

## Repository Structure

```
argus/
├── README.md                 # Project overview and entry point
├── LICENSE                   # MIT license
├── CONTRIBUTING.md           # How to contribute
├── .gitignore
├── docs/                     # Project documentation
│   ├── PROJECT_OVERVIEW.md   # Motivation, objectives, research questions
│   ├── ROADMAP.md            # Phased program plan (Phase 1–7)
│   ├── ARCHITECTURE.md       # Inputs, processing, outputs (conceptual)
│   ├── RESEARCH_DIRECTION.md # What makes ARGUS different; open questions
│   ├── DATASET.md            # CASEset: collection, schema, privacy, limits
│   ├── EVALUATION.md         # Metrics and evaluation protocol
│   └── ETHICS.md             # Privacy, consent, oversight, responsible use
├── argus/                    # Core Python package (planned; Phase 2+)
│   ├── data/                 # Dataset loaders and multi-stream synchronization
│   ├── context/              # Screen / context feature extraction
│   ├── models/               # Gaze and attention model implementations
│   ├── attention/            # Attention and intent modeling
│   ├── pipeline/             # End-to-end inference pipeline
│   └── evaluation/           # Metrics and evaluation harnesses
├── caseset/                  # CASEset dataset tooling (collection, sync, schema)
├── configs/                  # Experiment and model configurations
├── scripts/                  # Data prep, training, evaluation entry points
├── notebooks/                # Research and analysis notebooks
├── tests/                    # Unit and integration tests
└── data/                     # Local datasets and artifacts (git-ignored)
```

- **`docs/`** — All project documentation: motivation, roadmap, conceptual architecture, research direction, dataset description, evaluation protocol, and ethics. Start here to understand the program.
- **`argus/`** — The core Python package *[planned; Phase 2+]*. Sub-packages separate concerns: `data/` (loaders and multi-stream synchronization), `context/` (screen and context feature extraction), `models/` (gaze and attention models), `attention/` (attention and intent modeling), `pipeline/` (end-to-end inference), and `evaluation/` (metrics and harnesses). Most of this is not yet implemented.
- **`caseset/`** — Tooling for the CASEset dataset: collection, multi-stream synchronization, and schema. This directory is tracked in git and holds code, schema, and documentation only — it never contains raw session recordings. This is the most developed part of the program.
- **`configs/`** — Experiment and model configuration files.
- **`scripts/`** — Entry points for data preparation, training, and evaluation.
- **`notebooks/`** — Research and analysis notebooks.
- **`tests/`** — Unit and integration tests.
- **`data/`** — Local datasets and artifacts, including any CASEset session recordings on disk (webcam frames, screenshots, gaze, and interaction logs, e.g. under `data/caseset/sessions/...`). Git-ignored and not distributed with the repository; this is the sole home for raw recordings, which are never committed to version control. The `caseset/` tooling above operates on this data but does not store it.

## Current Status

**Phase 1 — CASEset Dataset Foundation** is the current, active phase. CASEset is a multi-stream dataset of synchronized webcam frames, screenshots, gaze coordinates, and interaction logs produced by prior work, and it is the empirical foundation everything else builds on. See [`docs/DATASET.md`](docs/DATASET.md) for collection methodology, schema, privacy posture, and limitations.

Every later phase below is **planned or research — not yet built**:

| Phase | Name | Status |
| ----- | ---- | ------ |
| **Phase 1** | CASEset Dataset Foundation | **Current / active** |
| Phase 2 | Baseline Gaze Models | Planned |
| Phase 3 | Context-Aware Attention Models | Planned |
| Phase 4 | Intent Inference | Research |
| Phase 5 | Sensor Cueing | Research |
| Phase 6 | Human-Machine Teaming | Research |
| Phase 7 | Mission Support Systems | Research / long-term |

No gaze model, attention model, intent-inference component, or end-to-end pipeline currently exists in this repository. Claims about ARGUS capabilities should be read against this table.

## Documentation

| Document | Description |
| -------- | ----------- |
| [`docs/PROJECT_OVERVIEW.md`](docs/PROJECT_OVERVIEW.md) | Motivation, objectives, and research questions. |
| [`docs/ROADMAP.md`](docs/ROADMAP.md) | Phased program plan covering Phase 1–7. |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Inputs, processing, and outputs (conceptual). |
| [`docs/RESEARCH_DIRECTION.md`](docs/RESEARCH_DIRECTION.md) | What makes ARGUS different and the open questions it pursues. |
| [`docs/DATASET.md`](docs/DATASET.md) | CASEset: collection, schema, privacy, and limitations. |
| [`docs/EVALUATION.md`](docs/EVALUATION.md) | Metrics and evaluation protocol. |
| [`docs/ETHICS.md`](docs/ETHICS.md) | Privacy, consent, oversight, and responsible use. |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to contribute to ARGUS. |

## Roadmap

ARGUS follows a seven-phase plan that begins with the CASEset dataset foundation (Phase 1) and progresses through baseline and context-aware modeling toward longer-horizon research in intent inference, sensor cueing, human-machine teaming, and mission support. See [`docs/ROADMAP.md`](docs/ROADMAP.md) for the full phased breakdown.

## License

Released under the MIT License. Copyright (c) 2026 Ezekiel A. Mitchell. See [`LICENSE`](LICENSE) for the full text.
