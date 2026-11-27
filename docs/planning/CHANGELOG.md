# Planning docs changelog

Versions follow `MAJOR.MINOR` (see [README](README.md)). Newest first.

## Roadmap & Paper Plan

### 1.1 — 27 Nov 2026
- **H1 made like-for-like.** H1 now names the benchmark: on Phototriage high-agreement test pairs, the student (<10M params) must reach ≥95% of the pair accuracy of **our own CLIP ViT-B/16 Siamese baseline (86M)** trained on the same split, with ≥8× fewer parameters. FGAesQ's 0.779 is measured on FGAesthetics' re-annotated pairs, so it is now quoted as context only (unless FGAesthetics is released and we evaluate on it directly). TL;DR claim and the "bar is now clear" evidence bullet updated to match.
- **New baseline in E1:** CLIP ViT-B/16 Siamese, frozen features + head first, full fine-tune on the lab 4090 if time.
- **Face-head labels specified** (Method section): pre-train on CEW + CelebA attributes; pseudo-label Phototriage face crops with eye-aspect ratio (threshold set on CEW), Laplacian variance on fixed-size crops, head-pose yaw/pitch for gaze, CelebA-trained smile head. No dependency on the VLM teacher, which only adds labels if it passes the flip test.
- **New check set:** ~200 hand-labelled Phototriage face crops (owner: B) to measure pseudo-label and face-head accuracy.
- Moved to Markdown in the repo; PDF snapshot `Keepr_Research_Roadmap_v1.1.pdf`.

### 1.0 — 27 Nov 2026
- First version (PDF only, now in `archive/Keepr_Research_Roadmap_v1.0.pdf`).

## Paper Summaries

### 1.1 — 27 Nov 2026
- Each of the 24 summaries expanded with three parts: *Concept explained*, *Limits to keep in mind*, *Try this*.
- Moved to Markdown in the repo; PDF snapshot `Keepr_Paper_Summaries_v1.1.pdf`.

### 1.0 — 27 Nov 2026
- First version: 24 papers, main idea / how it works / what we get (`archive/Keepr_Paper_Summaries_v1.0.pdf`).

## Feasibility Checklist

### 2.1 — 27 Nov 2026
- Moved to Markdown (`feasibility_checklist.md`) with a revision table; PDF snapshot `Keepr_Feasibility_Checklist_v2.1.pdf`. The v1 and v2 PDFs are in `archive/` as 1.0 and 2.0.
- **CLIP ViT-B/16 baseline added to the week:** A caches frozen CLIP features on Day 2 and trains a pairwise head on Day 3; full fine-tune only after the decision, if there is 4090 time.
- **Scorecard:** new rows for the CLIP baseline accuracy and the student ÷ CLIP ratio (≥ 0.95 is on track for the new H1). FGAesQ's 0.779 is not used for comparison.
- **Face-head labels section:** CEW, CelebA `Smiling`, head-pose gaze, Laplacian blur on 112×112 crops, CelebA `Eyeglasses` as the occlusion proxy; VLM only if it passes the flip test.
- **Face check set:** B hand-labels ~200 Phototriage face crops (Wed–Thu); A makes the first face pseudo-labels on Wednesday.
- File contracts gain `data/face_labels.csv` and `data/face_check.csv`.

### 2.0 — 27 Nov 2026
- Course-only scope, three roles ([A] Hamza GPU lead, [B] data + eval, [C] app), GPU plan, file contracts.

### 1.0 — 27 Nov 2026
- First 7-day checklist (`archive/Keepr_Feasibility_Checklist_v1.0.pdf`).
