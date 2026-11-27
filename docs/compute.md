# Compute

Status (28 Nov 2026): **Kaggle only.** No lab GPU is confirmed. A lab RTX A2000 (12 GB) may
become available on Mon 30 Nov; it is optional and nothing depends on it.

Full plan, quota split and budget:
[Feasibility Checklist, "How work flows on Kaggle only" and "GPU plan"](planning/feasibility_checklist.md).

## Kaggle setup

- Each teammate has 30 GPU-hours a week (90 in total). Phone verification unlocks GPUs and Internet.
- **Datasets (private, shared with the team):** Phototriage (uploaded 28 Nov) and
  `phototriage-256` (same images, short side 256 px; made once by the resize notebook).
  Inside a notebook they appear under `/kaggle/input/<slug>/`.
- **Starter notebook:** Accelerator GPU T4 ×2, Internet on, attach the datasets, then
  `git clone` the repo, `pip install -e .`, and call a script with a config. If the repo is
  private, keep a read-only GitHub token in Kaggle Secrets.
- **Two jobs per session:** quota counts session time, not GPUs, so run two jobs side by side
  with `CUDA_VISIBLE_DEVICES=0` and `CUDA_VISIBLE_DEVICES=1`.
- **Long runs:** "Save Version → Save & Run All" runs headless for up to 12 hours. Checkpoint
  every epoch.
- **Precision:** the T4 has no bf16; use fp16 autocast with a grad scaler.
- **Outputs:** written to `/kaggle/working` (up to 20 GB) and published as private datasets
  (features, predictions, teacher labels, checkpoints) so teammates attach them as inputs.
- **Quota tracking:** every row in the results sheet records the Kaggle account and GPU hours.

## Memory notes

| Job | Fits on |
|---|---|
| EfficientNet-B0 / MobileNetV3 student, 224 px, batch 32 | One T4, easily |
| CLIP ViT-B/16 frozen feature cache | One T4, minutes |
| CLIP ViT-B/16 full fine-tune (optional) | One T4, fp16, batch 16 or gradient checkpointing |
| Qwen2.5-VL-7B teacher | T4 ×2 in fp16 (`device_map="auto"`), or one T4 in 4-bit; 3B on one T4 as fallback |

## If the lab A2000 arrives

Use it only for long overnight jobs (no 12-hour limit, no quota). Record here:

- How to reach it (SSH host, VPN, or in person):
- Who else uses it, how time is booked, whether overnight runs are allowed:
- Python / CUDA / driver versions:
