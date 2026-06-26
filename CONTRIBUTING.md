# Contributing to ARGUS

Thanks for your interest in ARGUS — the **Adaptive Reconnaissance and Gaze
Understanding System**. ARGUS is an early-stage, open-source research and
engineering program investigating whether **operator attention** (where a human
operator looks, and the temporal structure of that attention) can serve as a
usable signal for autonomous systems.

This document explains how to contribute productively while the project is still
taking shape. Please read it before opening an issue or a pull request.

## Project Values

A few principles guide everything in this repository. We ask contributors to
share them.

- **Honesty over hype.** ARGUS is a research program, not a finished product.
  Its only current concrete asset is the **CASEset** dataset. We describe what
  exists plainly and label what does not yet exist as planned or hypothetical.
- **Evidence over adjectives.** Claims should be backed by data, code,
  citations, or clearly stated reasoning. Prefer specifics to superlatives.
- **Reproducibility first.** A result that cannot be reproduced is not yet a
  result. We would rather ship a documented `<TBD>` placeholder than a confident
  but unverified number.
- **Privacy and consent are non-negotiable.** ARGUS studies human attention
  using webcam, screen, and interaction data. Responsible handling of that data
  is a hard requirement, not an afterthought. See [`docs/ETHICS.md`](docs/ETHICS.md).
- **Early-stage is a feature.** Being candid that most components are not yet
  built lets us design them well. Good questions are as welcome as good code.

## Project Status

ARGUS follows a seven-phase plan. We are at the very beginning.

| Phase | Name | Status |
|-------|------|--------|
| 1 | CASEset Dataset Foundation | **current** |
| 2 | Baseline Gaze Models | planned |
| 3 | Context-Aware Attention Models | planned |
| 4 | Intent Inference | research |
| 5 | Sensor Cueing | research |
| 6 | Human-Machine Teaming | research |
| 7 | Mission Support Systems | research / long-term |

Because the core Python package (`argus/`) is still **planned (Phase 2+)**, the
most valuable contributions right now are in documentation, research notes,
dataset tooling, and evaluation design — not application code.

## Ways to Contribute

You do not need to write code to make a meaningful contribution. In rough order
of where help is most useful today:

### 1. Research and analysis

- Literature reviews and pointers to relevant prior work in context-aware gaze
  estimation, intent inference, operator attention modeling, sensor cueing, or
  human-machine teaming.
- Research notes that sharpen or challenge the core hypothesis (see below).
- Open-question write-ups for [`docs/RESEARCH_DIRECTION.md`](docs/RESEARCH_DIRECTION.md).

> **Core hypothesis** *[hypothesis]*: human attention carries information about
> intent, threat awareness, priorities, and situational understanding. If that
> information can be reliably and ethically extracted from low-cost sensing
> (webcam + screen context + interaction logs), it could improve how autonomous
> systems support human operators. Treat this as a hypothesis to be tested, not
> a fact to be assumed.

### 2. Documentation

- Improving clarity, structure, and accuracy of files in [`docs/`](docs/).
- Fixing broken links, typos, and inconsistent terminology.
- Drafting sections that are still stubs or marked `<TBD>`.

### 3. Dataset tooling

- Loaders, validators, and multi-stream synchronization utilities for CASEset
  under [`caseset/`](caseset/) (and later [`argus/data/`](argus/data/)).
- Schema documentation and privacy-preserving processing helpers.
- **Tooling only** — never the recordings themselves. The tracked `caseset/`
  directory holds collection, sync, and schema *tooling*; the recorded sessions
  it operates on live under the git-ignored `data/` directory
  (e.g. `data/caseset/sessions/...`) and are never committed. See
  [Data and Privacy for Contributions](#data-and-privacy-for-contributions).

### 4. Evaluation design

- Proposed metrics and evaluation protocols for [`docs/EVALUATION.md`](docs/EVALUATION.md).
- Baselines, ablations, and reporting conventions we should adopt before models
  exist, so that results are comparable from day one.

### 5. Code (later)

When Phase 2 begins, the core package will land under `argus/`. If you want to
prototype model or pipeline code now, please open an issue first so we can agree
on scope, interfaces, and conventions before you invest time.

## Getting Started

```bash
# 1. Fork the repository on GitHub, then clone your fork
git clone https://github.com/<your-username>/argus.git
cd argus

# 2. Add the upstream remote so you can keep your fork in sync
git remote add upstream https://github.com/<owner>/argus.git
```

The primary language is **Python**, but the package, dependency manifest, and
toolchain are **forthcoming (planned, Phase 2+)**. There is no build or test
command to run yet. When the toolchain lands, setup and contributor instructions
will be added here and in the [`docs/`](docs/) tree.

### Proposing work

1. Check existing [issues](https://github.com/<owner>/argus/issues) to avoid duplicate effort.
2. Open an issue describing what you want to do and why. For anything beyond a
   small docs fix, please wait for a quick round of discussion before starting,
   so we can align on scope and direction.
3. Small, self-contained typo or wording fixes can go straight to a pull request.

## Repository Layout

Contributions should fit the canonical structure. Please do not introduce new
top-level directories without discussion.

```
argus/
├── README.md                 # Project overview and entry point
├── LICENSE                   # MIT license
├── CONTRIBUTING.md           # How to contribute (this file)
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

## Development Workflow

### Branching

Work on a topic branch in your fork, never on `main`. Use a short, descriptive
branch name with a type prefix:

```
docs/clarify-dataset-schema
research/gaze-baseline-survey
data/caseset-sync-validator
feat/context-feature-extractor   # once code work begins
fix/broken-roadmap-links
```

### Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/) style. Keep the
subject line in the imperative mood and under ~72 characters; add a body when the
change needs explanation.

```
docs: add status labels to evaluation protocol draft
research: summarize prior work on context-aware gaze estimation
data: validate CASEset stream timestamps on load
fix: correct phase numbering in ROADMAP
```

Common types: `docs`, `research`, `data`, `feat`, `fix`, `test`, `refactor`,
`chore`.

### Pull requests

- Keep PRs **small and focused** — one logical change per PR. Large, sprawling
  PRs are hard to review and slow to merge.
- Reference the issue the PR addresses (e.g. `Closes #12`).
- Describe **what** changed and **why**, and note anything you are unsure about.
- Make sure documentation reflects code or process changes in the same PR.
- Be responsive to review comments; expect at least one round of feedback. A
  maintainer review is required before merge. Reviews focus on accuracy,
  honesty of claims, privacy compliance, and fit with the roadmap.

## Documentation Style

Documentation is currently the heart of the project, so we hold it to a clear
standard. These rules keep ARGUS honest.

- **Label aspiration explicitly.** Wherever a reader could mistake a goal for a
  fact, tag the statement as one of `[current]`, `[planned]`, or `[hypothesis]`.
- **No hype, no buzzword spam.** Write professionally and technically. Prefer
  precise, evidence-driven statements over marketing language.
- **No exaggerated capability claims.** Never state that ARGUS performs a task
  that has not actually been demonstrated. Do not imply deployment, evaluation,
  endorsement, partnership, or contract with any organization.
- **Reference points are not relationships.** Companies working in defense
  autonomy may be cited to explain *why this problem space matters*. Do not imply
  any affiliation with them.
- **Do not invent statistics.** If a precise figure (participant count, frame
  count, hours, resolution, fps) is not established, use an explicit `<TBD>`
  placeholder or describe the schema/methodology generically. Reproducibility
  matters more than impressive-looking numbers.
- **Format.** Plain GitHub-Flavored Markdown. Use fenced code blocks for
  ASCII/text diagrams. US English. Be concise. No emoji.

Example of acceptable phrasing:

> ARGUS *[planned]* will provide baseline gaze models in Phase 2. The CASEset
> dataset *[current]* supplies synchronized webcam, screenshot, gaze, and
> interaction streams for training and evaluation. Whether operator attention
> improves downstream autonomy *[hypothesis]* is an open research question.

## Data and Privacy for Contributions

ARGUS works with sensitive human data: webcam imagery, screen captures, gaze
coordinates, and interaction logs. Protecting that data is a hard requirement.

- **Never commit raw CASEset recordings.** Webcam frames, screenshots, and any
  data that could identify a participant must not enter the repository or its
  history. Recorded sessions live only under the git-ignored `data/` directory
  (e.g. `data/caseset/sessions/...`); the tracked `caseset/` directory holds
  collection, sync, and schema tooling only, never subject data.
- **Never commit PII** of any kind — names, faces, emails, account identifiers,
  or anything that could re-identify a person.
- **Large artifacts stay out of git.** Datasets, model weights, and bulky
  intermediate outputs belong in the git-ignored [`data/`](data/) directory or
  external storage, never tracked in the repo.
- **Contribute tooling, not data.** Loaders, validators, schema docs, and
  anonymization utilities are welcome; the underlying recordings are not.
- If you believe you have accidentally committed sensitive data, stop and flag
  it in an issue (or privately to the maintainer) immediately so history can be
  scrubbed before anything is published.

Before contributing anything that touches CASEset, read
[`docs/DATASET.md`](docs/DATASET.md) for the schema, collection methodology, and
limitations, and [`docs/ETHICS.md`](docs/ETHICS.md) for privacy, consent, and
responsible-use expectations.

## Reporting Issues

Use GitHub [issues](https://github.com/<owner>/argus/issues) for bugs, documentation problems, dataset
tooling requests, and research proposals. A helpful issue includes:

- A clear, specific title.
- What you expected versus what you found (for bugs), or the question/idea you
  are raising (for proposals).
- Steps to reproduce, relevant file paths, and environment details where
  applicable.
- For anything privacy-sensitive, **do not paste raw data or PII** into the
  issue. Describe the problem abstractly and link to the relevant tooling.

If you are unsure whether something is worth an issue, open one anyway —
well-scoped questions help shape an early-stage project.

## Code of Conduct

We are committed to a welcoming, harassment-free environment for everyone,
regardless of background or experience level. In short:

- Be respectful, constructive, and patient.
- Critique ideas and code, not people.
- Assume good faith and help newcomers get oriented.
- No harassment, discrimination, or personal attacks.

A formal Code of Conduct (likely the
[Contributor Covenant](https://www.contributor-covenant.org/)) will be adopted
as the contributor community grows. Until then, the standard above applies.
Report conduct concerns to the maintainer.

## Licensing

ARGUS is released under the **MIT License**, © 2026 Ezekiel A. Mitchell. By
contributing, you agree that your contributions are licensed under the same MIT
License that covers the project (see [`LICENSE`](LICENSE)). Only contribute work
you have the right to license this way, and do not introduce code, data, or text
that is incompatible with MIT.

---

Thank you for helping build ARGUS carefully and honestly. Questions, critiques,
and well-scoped proposals are all genuinely welcome.
