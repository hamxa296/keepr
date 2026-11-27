# Planning docs

| Doc | Current version | Source of truth | Snapshot |
|---|---|---|---|
| Research Roadmap & Paper Plan | 1.2 | [`roadmap.md`](roadmap.md) | [`Keepr_Research_Roadmap_v1.2.pdf`](Keepr_Research_Roadmap_v1.2.pdf) |
| Paper Summaries (24 papers) | 1.2 | [`paper_summaries.md`](paper_summaries.md) | [`Keepr_Paper_Summaries_v1.2.pdf`](Keepr_Paper_Summaries_v1.2.pdf) |
| 7-Day Feasibility Checklist | 2.2 | [`feasibility_checklist.md`](feasibility_checklist.md) | [`Keepr_Feasibility_Checklist_v2.2.pdf`](Keepr_Feasibility_Checklist_v2.2.pdf) |

Each doc has exactly one latest version, and it contains everything: read only that one. Never keep changes in a separate amendments file; edit the doc and bump its version. Older versions live in [`archive/`](archive/). What changed between versions: [`CHANGELOG.md`](CHANGELOG.md).

## Versioning rules

- **Markdown is the source of truth** so changes show up as readable diffs in pull requests.
  The PDF next to it is a snapshot of the same version for reading and sharing.
- Version numbers are `MAJOR.MINOR`:
  - **MINOR** (1.1 → 1.2): a fix or addition that doesn't change the plan's direction
    (a corrected number, a new baseline, a clarified method).
  - **MAJOR** (1.x → 2.0): the plan changes direction (a different dataset, task or main claim).
- Every version bump updates three things in one commit: the version line at the top of the
  Markdown file, a new PDF snapshot named `..._vX.Y.pdf` (the old PDF moves to `archive/`),
  and an entry in `CHANGELOG.md`.
- Commit doc changes directly to `main` (no separate branches in this project).
  Tag the version-bump commit `docs-vX.Y`.
- The live editable versions are Claude Docs (links at the top of each Markdown file).
  Edit there or here, but re-export so both match before bumping the version.
