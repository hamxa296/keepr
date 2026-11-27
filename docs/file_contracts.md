# File contracts

Agreed formats for files passed between team members, from Feasibility Checklist v2.
All of these live in the shared Drive folder under the same paths; only small samples are
ever committed. Change a contract only after telling the whole team.

## `data/pairs.csv` (made by B, used by A and B)

One row per labelled photo pair.

| Column | Type | Meaning |
|---|---|---|
| `pair_id` | str | unique id for the pair |
| `series_id` | str | Phototriage series id; the unit of splitting |
| `img_a` | str | image path relative to `data/raw/phototriage/` |
| `img_b` | str | image path relative to `data/raw/phototriage/` |
| `label` | int | 1 if A is preferred by the majority, else 0 |
| `vote_frac` | float | fraction of raters preferring A, in [0, 1] |
| `split` | str | `train`, `val` or `test` (assigned per series) |

Extra columns we expect to need (add when building the file):
`n_votes` (int, raters for this pair) and `high_agreement` (bool, `vote_frac` > 0.7 or < 0.3,
the Chang 2016 test protocol).

## `preds/<run>.csv` (made by A, used by B)

| Column | Type | Meaning |
|---|---|---|
| `pair_id` | str | joins to `pairs.csv` |
| `p_a` | float | model's P(A preferred), before temperature scaling |
| `label` | int | copied from `pairs.csv` |
| `vote_frac` | float | copied from `pairs.csv` |

## `feats/<backbone>.npy` (made by A, used by A, B, C)

One feature vector per image from a frozen pretrained backbone, shape `(n_images, dim)`.
Saved with a companion `feats/<backbone>_index.csv` holding `row, img_path` so rows can be
matched to images.

## `teacher/<model>.csv` (made by A, used by B)

| Column | Type | Meaning |
|---|---|---|
| `pair_id` | str | joins to `pairs.csv` |
| `p_ab` | float | teacher's P(A better) with A shown first |
| `p_ba` | float | teacher's P(A better) with B shown first |
| `seconds` | float | wall-clock seconds for both orders |

Order-averaged teacher probability: `(p_ab + p_ba) / 2`.

## `models/<run>.onnx` (made by A, handed to C)

The exported per-photo scorer, one input `image` of shape `1x3x224x224` (float32, ImageNet
normalisation), one output `score`. Export with opset 17. Ship both `<run>.onnx` (FP32) and
`<run>_int8.onnx`.

## Run naming

`YYYYMMDD_backbone_seed`, for example `20261129_effb0_s0`. The same name is used for the config
(`configs/<run>.yaml`), predictions, checkpoint, ONNX file and the results sheet row.
