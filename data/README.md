# data/ — Local Datasets & Artifacts (contents git-ignored)

Purpose: local home for real datasets (e.g., CASEset) and generated
artifacts. Contents live here on your machine but are NEVER committed: the
`.gitignore` ignores everything under `data/` (`/data/**`) regardless of file
extension, re-including only this `README.md` and a `.gitkeep` placeholder.
To share a small, reviewed, de-identified sample you must force-add the
specific file (`git add -f data/sample/<file>`) — there is no blanket
exception, by design.

Status: [current] empty placeholder — no data committed.
Becomes active in: Phase 1 — CASEset Dataset Foundation (local use).

[planned] Local storage layout per `docs/DATASET.md`. Do not commit raw
data, webcam frames, screenshots, or any personally identifiable content.
