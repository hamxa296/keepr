# Keepr — 7-Day Feasibility Checklist

<!--
  Versioned planning doc. The Markdown file is the source of truth for diffs;
  the PDF next to it is a snapshot of the same version. See docs/planning/README.md.
-->
> **Version 2.1** · 27 Nov 2026 · PDF snapshot: [`Keepr_Feasibility_Checklist_v2.1.pdf`](Keepr_Feasibility_Checklist_v2.1.pdf)
> This is the only version to read. Older versions are in [`archive/`](archive/). Changes: [CHANGELOG](CHANGELOG.md).

| Version | Date | Changes |
|---|---|---|
| 2.1 | 27 Nov 2026 | CLIP ViT-B/16 feature cache and head added (the H1 baseline); face-head label sources; ~200-crop face check set for B; face-label file contracts; CLIP-ratio row in the scorecard |
| 2.0 | 27 Nov 2026 | Three roles, one lab GPU, file contracts, GPU plan |
| 1.0 | 27 Nov 2026 | First 7-day checklist, single person |

**Fri 27 Nov → decision Fri 4 Dec 2026 · 3 teammates, 1 lab GPU.**
Goal: prove, with real numbers, that the Phototriage version of Keepr works, or switch early to the simpler culler.

## Who does what

| Who | Owns | Compute |
|---|---|---|
| **[A] Hamza** · GPU + training lead | Training script, baseline, seeds, backbone ablations, CLIP baseline, VLM probe, saved feature caches. Final call on the decision. | Lab RTX 4090 (24 GB) |
| **[B] Teammate** · Data + evaluation | Phototriage download and splits, `eval.py`, calibration and "not sure" analysis, all plots, the scorecard, the face check set, the shifted phone-photo set. | Laptop CPU + free Kaggle GPU |
| **[C] Teammate** · App + deployment | Next.js app, MediaPipe faces in the browser, burst grouping, ONNX/INT8 export, browser speed test, app screenshots for the memo. | Laptop CPU + free Kaggle GPU |

## Start today: the first six steps

- [ ] **[All]** 30-minute kickoff call: confirm roles, pick a daily 15-minute standup time, open a group chat.
- [ ] **[A]** Create the GitHub repo `keepr` (folders `data/ src/ configs/ notebooks/ app/`) and invite B and C. Create the shared Drive folder and the results sheet.
- [ ] **[All]** Install Python 3.11 + PyTorch, clone the repo, and create a Kaggle account. Verify your phone number on Kaggle to unlock its free weekly GPU quota.
- [ ] **[A]** On the lab machine run `nvidia-smi` and `python -c "import torch; print(torch.cuda.is_available())"`. You should see the RTX 4090 and `True`. Write down how you reach it remotely in `docs/compute.md`.
- [ ] **[B]** Download Phototriage from phototriage.cs.princeton.edu and read its terms. Also check the FGAesthetics paper (arXiv 2603.03907) for a data link.
- [ ] **[C]** Create the app skeleton: `npx create-next-app keepr-app`, then add `onnxruntime-web` and MediaPipe Tasks Vision.

## How work flows without three GPUs

[A] runs everything heavy on the 4090 and saves the outputs as files to the shared Drive folder: model checkpoints, prediction CSVs, cached image features and teacher labels. [B] and [C] work on those files on their laptops. Analysis, calibration, plots, the app and even small "head-only" training on cached features all run on a normal CPU. Nobody waits for the GPU except [A].

## File contracts

Agree on these today so everyone can work in parallel. Full column specs: [`docs/file_contracts.md`](../file_contracts.md).

| File (in shared Drive) | Columns / contents | Made by | Used by |
|---|---|---|---|
| `data/pairs.csv` | pair_id, series_id, img_a, img_b, label (1 if A preferred), vote_frac, split | B | A, B |
| `preds/<run>.csv` | pair_id, p_a (model's P(A preferred)), label, vote_frac | A | B |
| `feats/<backbone>.npy` | one feature vector per image (cached backbone output); includes `feats/clip_vitb16.npy` | A | A, B, C |
| `teacher/<model>.csv` | pair_id, p_ab, p_ba (both orders), seconds | A | B |
| `models/<run>.onnx` | exported model, FP32 and INT8 | A → C | C |
| `data/face_labels.csv` | face_id, img_path, dataset, bbox, eye/smile/gaze/sharp labels or pseudo-labels, ear, yaw, pitch, blur_var, eyeglasses | A | A, B |
| `data/face_check.csv` | ~200 hand-labelled Phototriage face crops; evaluation only | B | A, B |

Name every run `YYYYMMDD_backbone_seed` and log it as one row in the shared results sheet (date, config, seed, metrics). That sheet becomes the report's tables.

## Day 1 · Sat 28 Nov

Data in shape, scripts ready, everything tested on small or dummy inputs.

- [ ] **[B]** Build `pairs.csv`: split by series (never by photo), keep each pair's vote fraction, mark test pairs with over 70% agreement.
- [ ] **[B]** Write `eval.py`: pair accuracy, ECE (15 bins), reliability diagram, risk–coverage. Test it on random predictions (accuracy should be about 0.50).
- [ ] **[A]** Write the Siamese training script (EfficientNet-B0, BCE on sigmoid(sA − sB)). Debug on 200 pairs first.
- [ ] **[A]** Write the feature-cache script: run a frozen pretrained backbone once over every image and save `feats/*.npy`. It must handle both EfficientNet-B0 and the CLIP ViT-B/16 image encoder.
- [ ] **[C]** Run MediaPipe face detection in the browser on 20 of your own photos; draw boxes on the faces.
- [ ] **[C]** "Hello world" ONNX: export a pretrained MobileNet to ONNX and run it in the browser with onnxruntime-web.

**Done when:** `pairs.csv` is in Drive, `eval.py` passes the random test, the training loop runs on 200 pairs, and faces show up in the browser.

**If stuck:** Phototriage site down: check the auto-triage GitHub repo README for mirrors and email the Princeton authors today. Paste any error into Claude together with the code cell that caused it.

## Day 2 · Sun 29 Nov

First real result.

- [ ] **[A]** Train the baseline on the 4090: 224 px, batch 32, AdamW, learning rate 1e-4, about 10 epochs, seed 0 (`configs/baseline_effb0.yaml`). Save the checkpoint and `preds/val_seed0.csv`.
- [ ] **[A]** Run the feature cache for EfficientNet-B0 **and CLIP ViT-B/16**, and upload `feats/` to Drive.
- [ ] **[B]** Run `eval.py` on A's predictions and record validation pair accuracy in the scorecard.
- [ ] **[B]** Write the protocol for the shifted phone-photo set: ask 5 friends for 20–30 burst series each, with consent; set up Label Studio.
- [ ] **[C]** Browser speed test with the placeholder model: average milliseconds per photo over 50 photos.

**Done when:** a real validation pair accuracy is in the scorecard. Go if ≥ 0.68 (Chang's much larger VGG got 0.73).

**If stuck:** accuracy near 0.50 usually means flipped labels or mismatched pairs, so check 10 pairs by eye. Otherwise try MobileNetV3-Large or a learning rate of 3e-5.

## Day 3 · Mon 30 Nov

Trust check and the large baseline, mostly on CPU from saved files.

- [ ] **[B]** From the predictions CSV: ECE + reliability diagram, model probability vs vote fraction (scatter + correlation), accuracy at 100/90/80/70% coverage.
- [ ] **[B]** Fit the temperature T on validation, recompute ECE, and log before/after.
- [ ] **[A]** Train seeds 1 and 2 (the brief needs mean ± std over 3 seeds).
- [ ] **[A]** CLIP ViT-B/16 Siamese baseline, cheap version: train a small pairwise head on the cached frozen CLIP features (`configs/clip_vitb16_head.yaml`). Minutes, on GPU or CPU. Record its validation pair accuracy and the student ÷ CLIP ratio in the scorecard. A full CLIP fine-tune (`configs/clip_vitb16_finetune.yaml`) waits until after the decision and only runs if there is spare 4090 time.
- [ ] **[C]** With A's cached EfficientNet-B0 features, train a tiny head on your laptop CPU (logistic regression or a small MLP on feature A − feature B). Compare with A's full model.
- [ ] **[C]** Export A's real checkpoint to ONNX + INT8 and rerun the browser speed test.

**Done when:** ECE before/after, the "abstain on 20%" gain, 3-seed accuracy (mean ± std), the CLIP baseline accuracy and real browser ms/photo are all in the scorecard.

**If stuck:** ONNX export fails: export with a fixed input size (1×3×224×224) and opset 17. Slow in the browser: try WebGPU, a 160 px input, or MobileNetV3-Small.

## Day 4 · Tue 1 Dec

VLM teacher probe (a bonus: if it fails, drop it, not the project).

- [ ] **[A]** On the 4090: Qwen2.5-VL-7B in bf16. Take 300–500 validation pairs and ask "Two photos of the same scene. Which would most people keep? Answer A or B." Save P(A) and P(B).
- [ ] **[A]** Run every pair in both orders and save `teacher/qwen7b.csv` with the seconds per pair.
- [ ] **[B]** On CPU: teacher accuracy (orders averaged), flip rate (% of verdicts that follow position, not content), and accuracy on order-consistent pairs only.
- [ ] **[C]** In the app: group photos into bursts (cosine similarity of embeddings) and show a "Best" badge from the real model.

**Done when:** teacher numbers are in the scorecard, and the app picks a best photo per burst.

**If stuck:** out of memory: load the 7B model in 4-bit, or use the 3B model. A biased or weak teacher simply gets dropped. The face heads do not depend on it (see below).

## Days 5–7 · Wed 2 – Fri 4 Dec

Buffer, face labels, memo, decision.

- [ ] **[All]** Wednesday is a buffer: fix whatever failed. B and C each rerun one training job on Kaggle so all three can explain training in the viva.
- [ ] **[A]** Wednesday: first pass of face pseudo-labels into `data/face_labels.csv` (see "Face-head labels" below).
- [ ] **[B]** Wednesday–Thursday: hand-label the face check set, about 200 Phototriage face crops across portrait, group and selfie series (a few hours). Save as `data/face_check.csv`.
- [ ] **[B]** Thursday: fill the scorecard and make 2 plots (reliability diagram, risk–coverage).
- [ ] **[C]** Thursday: 3 app screenshots plus the browser speed number.
- [ ] **[A]** Thursday: write the 1-page memo with a recommendation and show it to your supervisor.
- [ ] **[All]** Friday: apply the decision rule and record the decision in the repo README.

**Done when:** a written go / switch decision by Friday.

**If stuck:** borderline? Keep the app and face heads, and use Phototriage only for the ranking head.

## Face-head labels

No dependency on the VLM teacher, and almost no hand labelling.

| Head | Label source | Watch out for |
|---|---|---|
| Eyes open | CEW (open/closed) for pre-training; on Phototriage, eye aspect ratio (EAR) from MediaPipe landmarks | Set the EAR threshold on CEW, don't guess it. EAR shifts with head angle and glasses, so log yaw and eyeglasses with it |
| Smile | CelebA `Smiling` attribute | |
| Gaze (looking at camera) | Head pose: yaw and pitch from MediaPipe landmarks | Set thresholds on training data, never on the check set |
| Face sharp | Laplacian variance on the face crop | Resize crops to a fixed size (112×112) **before** measuring, since blur scores depend on face size |
| Occlusion | CelebA `Eyeglasses` as a proxy | Proxy only; report it as such |

- The VLM teacher adds smile and gaze labels **only if** it passes the flip test (≥ 60% on order-consistent pairs).
- The face check set (~200 crops, owner B) is used only to report face-head and pseudo-label accuracy, which the rubric needs. It never trains anything or sets a threshold.

## Scorecard

| Check | Go if | Switch if | Number | Pass? |
|---|---|---|---|---|
| Data access | Phototriage on disk by Sat | No Phototriage and no FGAesthetics | | |
| Small model (val pair accuracy) | ≥ 0.68 | ≤ 0.62 after tuning | | |
| Stable over 3 seeds | std ≤ 0.02 | Results jump around a lot | | |
| Browser speed (ms per photo) | < 100 ms | > 300 ms even at INT8 | | |
| Trust (ECE before T / abstain gain) | ECE > 0.05 or gain ≥ 3 points | Not a switch: a weaker story only | | |
| CLIP ViT-B/16 head baseline (val pair accuracy) | — (reference for H1) | — | | |
| Student ÷ CLIP accuracy | ≥ 0.95 is on track for H1 | Not a switch: shrink the H1 claim | | |
| VLM teacher (bonus) | ≥ 60% on order-consistent pairs | Not a switch: drop the teacher | | |

**Decision rule:** stay on the Phototriage plan if the first four rows pass. Otherwise switch to the simpler culler (face-state heads trained on CEW/CelebA + an image-quality head). It's the same app and team, and nothing built this week is wasted.

Compare against our own CLIP baseline, not FGAesQ's 0.779: that number is on FGAesthetics' re-annotated pairs, a different test set.

## GPU plan: only [A] needs the 4090

| Task | Who | Where it runs |
|---|---|---|
| Baseline, seeds, backbone ablations | A | Lab 4090 |
| Feature caches (EfficientNet-B0, CLIP ViT-B/16; one pass each) | A | Lab 4090; files shared on Drive |
| CLIP baseline head on cached features | A | Lab 4090 or laptop CPU |
| VLM teacher probe and labels | A | Lab 4090 (7B in bf16, about 16 GB of weights) |
| Face pseudo-labels | A | Lab 4090 or laptop CPU |
| Head-only training on cached features | B, C | Laptop CPU: seconds to minutes |
| Evaluation, calibration, plots, face check set | B | Laptop CPU |
| One reproducibility rerun each (viva prep) | B, C | Kaggle free GPU (T4), weekly quota |
| App, MediaPipe, ONNX export, browser test | C | Laptop CPU + browser |

The brief allows training on free-tier GPUs or the lab, so the lab 4090 is fine. Book GPU time in a shared calendar: long training runs overnight, VLM runs in the day. If the lab machine is busy or access isn't confirmed yet, [A] falls back to Kaggle too.

## Mini glossary

| Term | Plain meaning |
|---|---|
| Pair accuracy | Out of all photo pairs, how often the model picks the one most people preferred. |
| Leakage | The same scene appears in training and testing, so the score looks better than it really is. |
| Feature cache | Saving each image's backbone output once, so small heads can be trained later without a GPU. |
| ECE | Average gap between how sure the model says it is and how often it's right. 0 = perfectly honest. |
| Temperature scaling | Divide the model's raw outputs by one number T so its confidence becomes honest. |
| Risk–coverage | How accuracy changes as the model is allowed to say "not sure" on its least confident cases. |
| Flip rate | How often a VLM's answer depends on which photo was shown first rather than on the photos. |
| EAR | Eye aspect ratio: eye height over width from face landmarks; low means closed. |
| ONNX / INT8 | A portable model file format / storing weights as 8-bit integers to make it smaller and faster. |

## Read this week

- **[All]** Chang et al. 2016, *Automatic Triage for a Photo Series*: the dataset and baseline.
- **[A]** LEAF 2026 (arXiv 2601.20689) and Co-Instruct 2024: teacher and distillation.
- **[B]** Calibrated Selective Classification 2022 (arXiv 2208.12084): the "not sure" analysis.
- **[C]** Real-time Burst Photo Selection 2018 (arXiv 1803.07212): tiny on-device models.

All 24 summaries: [Paper Summaries](paper_summaries.md). Full plan: [Research Roadmap](roadmap.md).
