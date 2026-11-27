# Compute

Status (28 Nov 2026): **no RTX 4090.** We have the lab **RTX A2000 (12 GB)** and **Kaggle T4**
notebooks (30 h/week per account, 3 teammates = 90 h/week). Full plan and budget:
[Feasibility Checklist, "GPU plan"](planning/feasibility_checklist.md#gpu-plan-lab-a2000-for-long-jobs-kaggle-for-parallel-ones).

## What runs where

| Task | Who | Where |
|---|---|---|
| Long jobs: main baseline, feature caches, CLIP head, CLIP fine-tune (optional) | A | Lab A2000 |
| VLM teacher (Qwen2.5-VL-7B) | A | A2000 in 4-bit (~6 GB), or Kaggle T4 ×2 in fp16; 3B in bf16 as fallback |
| Extra seeds, backbone ablations | A, B, C | Kaggle T4, spread over the three quotas |
| Head-only training on cached features, evaluation, plots | B, C | Laptop CPU |
| Reproducibility rerun (one each, AI341 rule) | B, C | Kaggle T4 |

## GPU notes

- **A2000 (Ampere, 12 GB):** supports bf16. Small CNN students fit easily. A full CLIP ViT-B/16
  fine-tune needs bf16 with batch 16 or gradient checkpointing. A 7B VLM only fits in 4-bit.
- **Kaggle T4 (Turing, 16 GB; T4 ×2 = 32 GB):** no bf16, so use fp16 mixed precision. Sessions
  stop after 12 hours: checkpoint every epoch and save outputs before the session ends.
  Attach Phototriage as a private Kaggle dataset shared with the team.
- Log GPU hours per run in the results sheet so the weekly Kaggle quota can be tracked.

## To fill in for the A2000

- How to reach it (SSH host, VPN, or in person):
- Who else uses it, how time is booked, whether overnight runs are allowed:
- Where datasets and outputs live on it:
- Python / CUDA / driver versions (`nvidia-smi`, `python -c "import torch; print(torch.cuda.is_available())"`):
