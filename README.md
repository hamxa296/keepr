# Keepr

Pick the best photo in a series of near-duplicates with a small, calibrated, on-device model
that says "not sure" on the pairs people themselves disagree on.

AI341 Deep Neural Networks semester project (due **18 Dec 2026**) and a paper
(working title: *Know When It's a Toss-Up: Distilling Vision-Language Models into Calibrated,
On-Device Series Photo Selection*), draft due **31 Jan 2027**.

The full plan lives in [`docs/planning/`](docs/planning/):
the Research Roadmap & Paper Plan, the Paper Summaries, and the 7-day Feasibility Checklist
(v2 is current; v1 is kept for reference).

## Repository layout

```
docs/planning/   planning PDFs (roadmap, paper summaries, feasibility checklists)
docs/            file contracts, compute notes, AI341 brief
configs/         one YAML per experiment run
src/keepr/       Python package: data, models, training, evaluation, teacher, export
scripts/         command-line entry points that call src/keepr
notebooks/       exploration and Kaggle reproducibility notebooks
app/             Next.js browser app (onnxruntime-web + MediaPipe)  [owner: C]
data/            local datasets, not committed (see data/README.md)
preds/ feats/ teacher/ models/   run outputs, shared via Drive, not committed
results/         results sheet export, small figures for the report
```

Large files (images, checkpoints, feature caches, ONNX models) are never committed. They live
in the shared Drive folder using the same folder names, so paths match on every machine.

## Team

| Role | Owns | Compute |
|---|---|---|
| [A] Hamza, GPU + training lead | training script, baseline, seeds, ablations, VLM probe, feature cache | lab RTX 4090 (access not yet confirmed) |
| [B] Data + evaluation | Phototriage splits, `pairs.csv`, `eval.py`, calibration, plots, shifted set | laptop CPU + Kaggle |
| [C] App + deployment | Next.js app, MediaPipe faces, burst grouping, ONNX/INT8 export, browser speed | laptop CPU + Kaggle |

## Conventions

- File formats between team members: [`docs/file_contracts.md`](docs/file_contracts.md).
- Run names: `YYYYMMDD_backbone_seed` (for example `20261129_effb0_s0`). Each run gets one
  config in `configs/` and one row in the shared results sheet
  (template: [`results/results_sheet_template.csv`](results/results_sheet_template.csv)).
- Split by **series**, never by photo. Test pairs are those with more than 70% rater agreement.
- The shifted phone-photo set never touches training or tuning. Tune on the Phototriage
  validation split only.
- At least one student training run must reproduce on a free Kaggle T4 (AI341 rule).

## Setup

```bash
python3.11 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pip install -e .
```

## Feasibility decision (Fri 4 Dec 2026)

_To be filled in from the scorecard: GO on the Phototriage plan, or SWITCH to the simpler
culler._

| Check | Go if | Number | Pass? |
|---|---|---|---|
| Data access | Phototriage on disk by Sat 28 Nov | | |
| Small model (val pair accuracy) | ≥ 0.68 | | |
| Stable over 3 seeds | std ≤ 0.02 | | |
| Browser speed (ms per photo) | < 100 ms | | |
| Trust (ECE before T / abstain gain) | ECE > 0.05 or gain ≥ 3 points | | |
| VLM teacher (bonus) | ≥ 60% on order-consistent pairs | | |
