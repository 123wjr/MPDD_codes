# JerryYe748 · Track 1 (Elder) — reproduction verdict

**Date:** 2026-06-20  **Status:** ⛔ **Cannot be reproduced from scratch — BLOCKED.**

This re-verifies the earlier audit (`docs/audit_JerryYe748.md`, `repro/JerryYe748/RUNBOOK.md`)
by attempting a from-scratch reproduction under the same principle used for the other
players (re-extract all features from raw, no bundled features/weights).

## Why it's blocked (confirmed 2026-06-20)
The repo is an **inference/training package over pre-extracted features** — it contains **no
raw→feature extraction code**:
- `grep` for any decoding/extraction call (`librosa.load`, `torchaudio.load`,
  `cv2.VideoCapture`, `soundfile.read`, `whisper.load_model`, `av.open`) → **zero hits**.
- The only `from_pretrained` is `models/gated_multifeature_multitask.py:27`, an **optional
  `--wavlm_mode e2e` branch** that loads a checkpoint; the default is `offline` and the
  submission inference (`scripts/run_elder_submission_model.py`) consumes pre-extracted
  `.npy` only. No audio/video is ever decoded.
- `dataset.py` reads features exclusively via `np.load` of `*.npy`.
- `MANIFEST.json` ships the features as Release assets (e.g. train `wavlm` = 1.76 GB,
  `whisper` = 2.2 GB, `openface3` = 219 MB) — consumed, not generated.

Without the player's bundled `.npy` + `.pth/.cbm`, the raw→feature→train chain cannot run.

## Rules problem (non-official features)
The model **hard-depends** on **non-official, stronger** features the official baseline does
not use: **WavLM-Large, Whisper-large-v3, LCO-Omni-7B** (audio), **OpenFace3** (face, vs
official OpenFace2), **YOLOv11** skeleton (gait). Swapping back to official features would
require re-architecting + re-tuning the gated multi-feature model.

## What was NOT found (cleared)
- **No test-label leakage**: `build_test_payload()` initialises all test labels to dummy 0
  and reads only IDs from `split_labels_test.csv`; aux CatBoost fits on train labels only.
- **No per-id hardcoding**: inference is standard forward; the aux CatBoost only promotes/
  demotes inconsistent samples by a **global** `aux_threshold`, not by test ID.

## Determinism risk — medium
`train_elder_submission_model.py` pins `BINARY_PHQ_EPOCH=43`, `TERNARY_EPOCH=45`,
`PRESERVE_SHUFFLE_SEQUENCE=True`; CatBoost `random_seed=3407` but `iterations=2`. Results are
bound to a specific training trajectory/environment and unlikely to reproduce the same score
on retrain.

## Verdict
Per the audit principle (no bundled intermediate features / pretrained weights), this
submission **cannot be certified as from-scratch reproducible**. To verify, the player must
supply the full **raw→feature** extraction code and justify the non-official features; then
re-extract from official raw data, retrain from random init, and resubmit to Codabench.
