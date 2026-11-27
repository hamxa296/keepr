# Plan amendments

Changes to the planning PDFs in `docs/planning/`. Where this file and the PDFs disagree,
this file wins. Newest change first.

## A2 · 27 Nov 2026 · Face-head labels no longer depend on the VLM

**Problem.** The roadmap's loss has a face term `L_face`, but Phototriage has no face labels,
so it was unclear what the face heads train on. Leaning on the VLM teacher for them would tie
a core part of the model to a stretch goal.

**Change.**

| Head | Label source | Notes |
|---|---|---|
| Eyes open | CEW (closed/open eyes) for pre-training; pseudo-labels on Phototriage from eye aspect ratio (EAR) on MediaPipe landmarks | EAR threshold is fitted on CEW, not guessed. EAR shifts with head angle and glasses, so record yaw and eyeglasses alongside it |
| Smile | CelebA `Smiling` attribute | |
| Gaze (looking at camera) | Head pose: yaw and pitch from MediaPipe face landmarks | Thresholds set on training data, not on the check set |
| Face sharp | Variance of the Laplacian on the face crop | Crops are resized to a fixed size (112×112) **before** measuring blur, since raw blur scores depend on face size |
| Occlusion | Open: no source agreed yet | Decide before Day 4; drop the head if nothing clean exists |

- The VLM teacher is **optional** for face labels. Use it only as an extra signal, and only if
  it passes the flip test (≥ 60% accuracy on order-consistent pairs, Feasibility Checklist v2,
  Day 4).
- **Hand-labelled check set (owner: B).** About 200 face crops sampled from Phototriage,
  across portrait, group and selfie series, labelled by hand for every head. It is used only
  to report the accuracy of the face heads and of the pseudo-labels (the rubric needs this
  number). It is never used to train or to set thresholds. Format: `data/face_check.csv` in
  `docs/file_contracts.md`.

## A1 · 27 Nov 2026 · Like-for-like comparison for H1

**Problem.** The roadmap compared our student with FGAesQ's 0.779. That number is on
FGAesthetics natural series, which re-labelled Chang's photos with 10 votes per pair and
removed pairs people couldn't decide. It is not the same test set as Phototriage's
high-agreement pairs, so "within 3 points of 0.779" was not a valid claim.

**Change.**

- **New large baseline in E1: CLIP ViT-B/16 as a Siamese network.** This is the backbone
  FGAesQ uses (about 86M parameters), so it is the fairest stand-in for the state of the art
  that we can run on our own split.
  1. Cheap version first: cache frozen CLIP ViT-B/16 image features once
     (`feats/clip_vitb16.npy`) and train only a small pairwise head on top. Minutes on any
     GPU, or on CPU from the cache. Config: `configs/clip_vitb16_head.yaml`.
  2. Full fine-tune only if there is spare time on the lab 4090.
     Config: `configs/clip_vitb16_finetune.yaml`.
- **H1 rewritten** (see below). 0.779 is quoted only as context, or compared against directly
  only if the FGAesthetics data is released and we evaluate on its natural-series test set.

## Current hypotheses

These replace H1–H4 in the roadmap. H2–H4 are unchanged apart from cross-references.

- **H1 (efficiency).** On Phototriage high-agreement test pairs, a student under 10M
  parameters reaches at least 95% of the pair accuracy of our CLIP ViT-B/16 Siamese baseline,
  with at least 8× fewer parameters, and runs in under 100 ms per photo in the browser.
- **H2 (teacher).** Order-debiased, confidence-filtered VLM labels improve the student over
  human labels alone, even though the raw VLM scores lower than the student's target.
- **H3 (trust).** Training on soft vote fractions plus temperature scaling lowers ECE compared
  with hard 0/1 labels. Abstaining on the 20% least-confident pairs raises accuracy on the
  rest by at least 5 points, and this still holds on the shifted phone-photo set.
- **H4 (faces).** Face-state heads trained from CEW, CelebA and rule-based pseudo-labels
  (A2) improve accuracy on portrait, group and selfie series, with no loss elsewhere. Face-head
  accuracy is reported on the hand-labelled check set.

## Updated E1 baselines

Random · single-image quality score · Siamese ResNet (reproduction) · **CLIP ViT-B/16 Siamese,
frozen features + head** · CLIP ViT-B/16 Siamese, full fine-tune (if time) · reported numbers
from Chang 2016 (0.73) and SPSNet (75.36%), and FGAesQ (0.779) and Q-Align (0.711) labelled
as "different test set, context only".
