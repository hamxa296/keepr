# Keepr — 7-Day Feasibility Checklist

<!--
  Versioned planning doc. The Markdown file is the source of truth for diffs;
  the PDF next to it is a snapshot of the same version. See docs/planning/README.md.
-->
> **Version 2.4** · 28 Nov 2026 · PDF snapshot: [`Keepr_Feasibility_Checklist_v2.4.pdf`](Keepr_Feasibility_Checklist_v2.4.pdf)
> This is the only version to read. Older versions are in [`archive/`](archive/). Changes: [CHANGELOG](CHANGELOG.md).

| Version | Date | Changes |
|---|---|---|
| 2.4 | 28 Nov 2026 | Roles named Person A, B and C instead of by name; the team decides who takes which |
| 2.3 | 28 Nov 2026 | Kaggle-only: no lab GPU is confirmed (the A2000 may come on Mon 30 Nov and is optional). Phototriage is already a private Kaggle dataset shared with the team. Code runs from the GitHub repo inside thin Kaggle notebooks; images pre-resized to 256 px; T4 ×2 sessions run two jobs at once; run outputs shared as Kaggle datasets instead of Drive; quota split per person |
| 2.2 | 28 Nov 2026 | No 4090: compute is now the lab RTX A2000 (12 GB) plus Kaggle T4 (30 h/week per teammate, 90 h total). VLM teacher runs in 4-bit on the A2000 or fp16 across Kaggle T4 ×2. Phototriage shared as a private Kaggle dataset. Seeds and ablations spread across all three Kaggle quotas |
| 2.1 | 27 Nov 2026 | CLIP ViT-B/16 feature cache and head added (the H1 baseline); face-head label sources; ~200-crop face check set for B; face-label file contracts; CLIP-ratio row in the scorecard |
| 2.0 | 27 Nov 2026 | Three roles, one lab GPU, file contracts, GPU plan |
| 1.0 | 27 Nov 2026 | First 7-day checklist, single person |

**Fri 27 Nov → decision Fri 4 Dec 2026 · 3 teammates, 3 Kaggle accounts (30 GPU-hours/week each).**
Goal: prove, with real numbers, that the Phototriage version of Keepr works, or switch early to the simpler culler.

## Who does what

| Who | Owns | Compute |
|---|---|---|
| **[A] Person A** · GPU + training lead | Training script, baseline, seeds, backbone ablations, CLIP baseline, VLM probe, saved feature caches. Final call on the decision. | Kaggle T4 ×2 (30 h/week) |
| **[B] Person B** · Data + evaluation | Phototriage download and splits, `eval.py`, calibration and "not sure" analysis, all plots, the scorecard, the face check set, the shifted phone-photo set. | Laptop CPU + Kaggle T4 (30 h/week) |
| **[C] Person C** · App + deployment | Next.js app, MediaPipe faces in the browser, burst grouping, ONNX/INT8 export, browser speed test, app screenshots for the memo. | Laptop CPU + Kaggle T4 (30 h/week) |

## Start today: the first six steps

- [ ] **[All]** 30-minute kickoff call: confirm roles, pick a daily 15-minute standup time, open a group chat.
- [ ] **[A]** Create the GitHub repo `keepr` (folders `data/ src/ configs/ notebooks/ app/`) and invite B and C. Create the shared results sheet (Google Sheets).
- [ ] **[All]** Install Python 3.11 + PyTorch, clone the repo, and create a Kaggle account. Verify your phone number on Kaggle to unlock its free weekly GPU quota.
- [ ] **[A]** Make the Kaggle starter notebook: Accelerator **GPU T4 ×2**, Internet **on**, attach the Phototriage dataset, `git clone` the repo, `pip install -e .`, then run `nvidia-smi` and `python -c "import torch; print(torch.cuda.is_available(), torch.cuda.device_count())"`. You should see two T4s and `True 2`. Share it with B and C; everyone copies it for their own runs.
- [x] **[A]** Phototriage uploaded as a **private Kaggle dataset** and shared with B and C (done 28 Nov).
- [ ] **[B]** Read Phototriage's terms of use and record them in `data/README.md`. Also check the FGAesthetics paper (arXiv 2603.03907) for a data link.
- [ ] **[C]** Create the app skeleton: `npx create-next-app keepr-app`, then add `onnxruntime-web` and MediaPipe Tasks Vision.

## How work flows on Kaggle only

All GPU work runs on Kaggle. Each of the three accounts has 30 GPU-hours a week (90 in total), which is well above what this week needs.

- **Code lives in GitHub, runs on Kaggle.** Notebooks stay thin: turn Internet on, `git clone` the repo, `pip install -e .`, attach the datasets, call a script with a config. Everyone runs the same code. If the repo is private, store a read-only GitHub token in Kaggle **Secrets**, never in the notebook.
- **Pre-resize once.** Kaggle sessions have only a few CPU cores, so decoding full-size JPEGs starves the GPU. One notebook resizes every image to 256 px (short side) and saves the result as a second private dataset, `phototriage-256`. All training uses that.
- **Always pick GPU T4 ×2 and run two jobs per session.** Quota counts session time, not GPUs. Launch two runs side by side with `CUDA_VISIBLE_DEVICES=0` and `CUDA_VISIBLE_DEVICES=1` (for example, seeds 1 and 2).
- **Run long jobs headless.** "Save Version → Save & Run All" runs in the background for up to 12 hours with the browser closed. Checkpoint every epoch so a stopped session can resume.
- **fp16, not bf16.** The T4 has no bf16; use `torch.autocast(dtype=torch.float16)` with a grad scaler.
- **Share outputs as Kaggle datasets, not Drive.** Feature caches, predictions, teacher labels and checkpoints are notebook outputs (`/kaggle/working`, up to 20 GB). Publish each as a private dataset (or add the notebook's output as an input) so teammates attach it directly. B and C then do evaluation, calibration and head training on a laptop CPU with no GPU.
- **Log GPU hours.** Every run's row in the results sheet records its Kaggle account and GPU hours, so nobody runs out of quota mid-week.

**Optional lab RTX A2000 (12 GB).** It may become available on Mon 30 Nov. It is about as fast as a T4 for this work and has less memory, so nothing depends on it. If it arrives, use it only for long overnight jobs (no 12-hour limit, no quota).

## File contracts

Agree on these today so everyone can work in parallel. Full column specs: [`docs/file_contracts.md`](../file_contracts.md).

| File (shared as a Kaggle dataset) | Columns / contents | Made by | Used by |
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

- [ ] **[A]** Run the resize notebook once and publish `phototriage-256` (CPU session is enough; no GPU quota needed).
- [ ] **[B]** Build `pairs.csv`: split by series (never by photo), keep each pair's vote fraction, mark test pairs with over 70% agreement.
- [ ] **[B]** Write `eval.py`: pair accuracy, ECE (15 bins), reliability diagram, risk–coverage. Test it on random predictions (accuracy should be about 0.50).
- [ ] **[A]** Write the Siamese training script (EfficientNet-B0, BCE on sigmoid(sA − sB)). Debug on 200 pairs first.
- [ ] **[A]** Write the feature-cache script: run a frozen pretrained backbone once over every image and save `feats/*.npy`. It must handle both EfficientNet-B0 and the CLIP ViT-B/16 image encoder.
- [ ] **[C]** Run MediaPipe face detection in the browser on 20 of your own photos; draw boxes on the faces.
- [ ] **[C]** "Hello world" ONNX: export a pretrained MobileNet to ONNX and run it in the browser with onnxruntime-web.

**Done when:** `pairs.csv` is published as a Kaggle dataset, the `phototriage-256` dataset exists, `eval.py` passes the random test, the training loop runs on 200 pairs, and faces show up in the browser.

**If stuck:** Phototriage site down: check the auto-triage GitHub repo README for mirrors and email the Princeton authors today. Paste any error into Claude together with the code cell that caused it.

## Day 2 · Sun 29 Nov

First real result.

- [ ] **[A]** Train the baseline on Kaggle (one T4 of a T4 ×2 session): 224 px, batch 32, AdamW, learning rate 1e-4, about 10 epochs, seed 0 (`configs/baseline_effb0.yaml`). Save the checkpoint and `preds/val_seed0.csv`.
- [ ] **[A]** Run the feature cache for EfficientNet-B0 **and CLIP ViT-B/16**, on the other T4 of the same session, and publish `feats/` as a Kaggle dataset.
- [ ] **[B]** Run `eval.py` on A's predictions and record validation pair accuracy in the scorecard.
- [ ] **[B]** Write the protocol for the shifted phone-photo set: ask 5 friends for 20–30 burst series each, with consent; set up Label Studio.
- [ ] **[C]** Browser speed test with the placeholder model: average milliseconds per photo over 50 photos.

**Done when:** a real validation pair accuracy is in the scorecard. Go if ≥ 0.68 (Chang's much larger VGG got 0.73).

**If stuck:** accuracy near 0.50 usually means flipped labels or mismatched pairs, so check 10 pairs by eye. Otherwise try MobileNetV3-Large or a learning rate of 3e-5.

## Day 3 · Mon 30 Nov

Trust check and the large baseline, mostly on CPU from saved files.

- [ ] **[B]** From the predictions CSV: ECE + reliability diagram, model probability vs vote fraction (scatter + correlation), accuracy at 100/90/80/70% coverage.
- [ ] **[B]** Fit the temperature T on validation, recompute ECE, and log before/after.
- [ ] **[A, B]** Train seeds 1 and 2 (the brief needs mean ± std over 3 seeds). Run both in one T4 ×2 session on B's quota, one seed per GPU.
- [ ] **[A]** CLIP ViT-B/16 Siamese baseline, cheap version: train a small pairwise head on the cached frozen CLIP features (`configs/clip_vitb16_head.yaml`). Minutes, on GPU or CPU. Record its validation pair accuracy and the student ÷ CLIP ratio in the scorecard. A full CLIP fine-tune (`configs/clip_vitb16_finetune.yaml`) waits until after the decision and only runs if there is spare quota; on a 16 GB T4 use fp16 with batch 16 (or gradient checkpointing).
- [ ] **[C]** With A's cached EfficientNet-B0 features, train a tiny head on your laptop CPU (logistic regression or a small MLP on feature A − feature B). Compare with A's full model.
- [ ] **[C]** Export A's real checkpoint to ONNX + INT8 and rerun the browser speed test.

**Done when:** ECE before/after, the "abstain on 20%" gain, 3-seed accuracy (mean ± std), the CLIP baseline accuracy and real browser ms/photo are all in the scorecard.

**If stuck:** ONNX export fails: export with a fixed input size (1×3×224×224) and opset 17. Slow in the browser: try WebGPU, a 160 px input, or MobileNetV3-Small.

## Day 4 · Tue 1 Dec

VLM teacher probe (a bonus: if it fails, drop it, not the project).

- [ ] **[A]** On Kaggle T4 ×2: Qwen2.5-VL-7B in fp16 split across both GPUs (`device_map="auto"`, about 16 GB of weights over 32 GB), or in 4-bit on one GPU (about 6 GB). Time 20 pairs first. Take 300–500 validation pairs and ask "Two photos of the same scene. Which would most people keep? Answer A or B." Save P(A) and P(B).
- [ ] **[A]** Run every pair in both orders and save `teacher/qwen7b.csv` with the seconds per pair.
- [ ] **[B]** On CPU: teacher accuracy (orders averaged), flip rate (% of verdicts that follow position, not content), and accuracy on order-consistent pairs only.
- [ ] **[C]** In the app: group photos into bursts (cosine similarity of embeddings) and show a "Best" badge from the real model.

**Done when:** teacher numbers are in the scorecard, and the app picks a best photo per burst.

**If stuck:** out of memory, NaNs in fp16, or too slow: use 4-bit loading, or Qwen2.5-VL-3B in fp16 on one T4 (about 7 GB). A biased or weak teacher simply gets dropped. The face heads do not depend on it (see below).

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

## GPU plan: Kaggle only

| Task | Who | Kaggle quota | Where it runs |
|---|---|---|---|
| Resize images to `phototriage-256` | A | none | Kaggle CPU session |
| Main baseline (seed 0) + feature caches (EfficientNet-B0, CLIP ViT-B/16) | A | A | One T4 ×2 session: baseline on GPU 0, caches on GPU 1 |
| CLIP baseline head on cached features | A | none | Kaggle CPU or laptop |
| Seeds 1 and 2 | B | B | One T4 ×2 session, one seed per GPU |
| Backbone ablations, reproducibility run | C | C | T4 ×2 sessions, two runs each |
| VLM teacher probe and labels | A | A | T4 ×2: 7B in fp16 across both GPUs, or 4-bit on one; 3B as fallback |
| CLIP full fine-tune (optional, after the decision) | A | A or C | One T4, fp16, batch 16 |
| Face pseudo-labels | A | none or A | Kaggle CPU, or a T4 if MediaPipe on CPU is too slow |
| Head-only training, evaluation, calibration, plots, face check set | B, C | none | Laptop CPU |
| App, MediaPipe, ONNX export, browser test | C | none | Laptop + browser |

**Budget.** 90 GPU-hours a week across three accounts. Our rough estimate (replace with real timings after the first run) is under 15 GPU-hours for the whole feasibility week: tens of minutes per student run on pre-resized images, about an hour for the 500-pair VLM probe in both orders, and minutes for each feature cache. Running two jobs per T4 ×2 session roughly halves it.

The AI341 brief requires training on free-tier GPUs or the lab; Kaggle is free tier, so every run already meets the rule.

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
| 4-bit loading | Storing a big model's weights in 4 bits so a 7B VLM fits in about 6 GB, at a small cost in accuracy. |
| ONNX / INT8 | A portable model file format / storing weights as 8-bit integers to make it smaller and faster. |

## Read this week

- **[All]** Chang et al. 2016, *Automatic Triage for a Photo Series*: the dataset and baseline.
- **[A]** LEAF 2026 (arXiv 2601.20689) and Co-Instruct 2024: teacher and distillation.
- **[B]** Calibrated Selective Classification 2022 (arXiv 2208.12084): the "not sure" analysis.
- **[C]** Real-time Burst Photo Selection 2018 (arXiv 1803.07212): tiny on-device models.

All 24 summaries: [Paper Summaries](paper_summaries.md). Full plan: [Research Roadmap](roadmap.md).
