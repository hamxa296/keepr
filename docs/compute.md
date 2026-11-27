# Compute

Status (27 Nov 2026): **lab RTX 4090 access not yet confirmed.**

## Plan

| Task | Who | Where |
|---|---|---|
| Baseline, seeds, backbone ablations | A | lab RTX 4090 (24 GB) |
| VLM teacher probe and labels | A | lab RTX 4090 (Qwen2.5-VL-7B in bf16) |
| Feature cache | A | lab RTX 4090, output shared on Drive |
| Head-only training on cached features | B, C | laptop CPU |
| Evaluation, calibration, plots | B | laptop CPU |
| Reproducibility rerun (one each) | B, C | Kaggle T4 (free weekly quota) |
| App, ONNX export, browser test | C | laptop CPU + browser |

Fallback if the 4090 is unavailable or busy: Kaggle T4 x2 for training, Qwen2.5-VL-3B in
4-bit for the teacher.

## To fill in once access is confirmed

- How to reach the machine remotely (SSH host, VPN or Tailscale, port):
- Who else uses it and how GPU time is booked:
- Where datasets and outputs live on it:
- Python / CUDA / driver versions (`nvidia-smi`, `python -c "import torch; print(torch.cuda.is_available())"`):
