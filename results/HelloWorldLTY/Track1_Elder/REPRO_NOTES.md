# HelloWorldLTY · Track 1 (Elder) — honest from-scratch reproduction

**Date:** 2026-06-20  **Env:** conda `mpdd`  **Goal:** quantify the gap between the
player's claimed Track-1 score (**0.5881**) and what is honestly reproducible when you
re-extract features from raw and never reuse a frozen submission artifact.

## TL;DR
- The shipped `reprod_old/stage_a/run_stage_a.sh` is **not** a from-scratch run. It
  reproduces *classification byte-exact* only because it **copies the binary/ternary
  predictions from a frozen `class_dir/`** (the player's pre-computed test-set class
  predictions, themselves ensembled from many of their prior submissions — see
  `class_dir/class_phq_rank1_config_zh.md`). Only the **PHQ regression** is recomputed.
- The player's own `stage_a/STAGE_A_TEST.md` reports PHQ |Δ|=**0.195 mean** vs their
  reference — but that was achieved using **their own frozen features + frozen class_dir**.
- This reproduction instead **re-extracts all A/V features from raw** and **derives the
  classes from the recomputed PHQ** (`--class_mode phq_threshold`, thresholds learned on
  train CV only). Nothing frozen is reused. Result diverges materially from the frozen
  reference (below), confirming the claimed reproducibility was an artifact of reusing
  frozen products.

## What was self-extracted (from raw, 2026-06-20)
| modality | features | source |
|---|---|---|
| **Audio (A)** | mfcc (librosa n_mfcc=64), opensmile (ComParE_2016 LLD, 65-d), wav2vec2 (facebook/wav2vec2-base-960h last_hidden_state, 768-d) | **self-extracted from raw `.WAV`** |
| **Video (V)** | densenet121 (torchvision, 1000-d), openface (docker `algebr/openface`, 710-d from CSV cols 4:) | **self-extracted from raw `.avi/.mp4`** |
| **Gait (G)** | IMU sequence `.npy` | **official array** (raw sensor data, not a learned extractor) — documented caveat |
| **Personality (P)** | `descriptions_embeddings_with_ids.npy` (1024-d) | **official array** — the player does not specify the text-encoder recipe; re-deriving would add guesswork, not honesty — documented caveat |

Counts: 354 train + 88 test `.npy` per A/V feature, 0 extraction failures.
wav2vec2 model fetched via ModelScope mirror (`AI-ModelScope/wav2vec2-base-960h`).

## Run
`tmp/run_stage_a_honest_elder.sh` → `generate_sklearn_phqstack_modular_avgp_submission_stratmeta.py`
with the player's exact feature/regressor/stack config **plus `--class_mode phq_threshold`**
and the data tree pointed at the self-extracted features. 180 candidates produced;
ranking is by internal CV `candidate_score` (no leaderboard peeking).

- **Top candidate (`_r1`)**: `reg_rankensemble_top3_median`, CV ccc=0.402, CV f1=0.624,
  CV kappa=0.363, mae=0.704. → `submission_honest_top_r1.zip` (**primary honest submission**)
- **Player-analog (`_r3`)**: `reg_rankensemble_top2_trimmed`. → `submission_honest_playeranalog_r3.zip`

## Honest `_r1` vs player's frozen reference (`stage_a/expected_output/`)
| task | class flips (of 23) | PHQ \|Δ\| mean | PHQ \|Δ\| max |
|---|---|---|---|
| binary | 8 | 2.38 | 6.37 |
| ternary | 8 | 2.38 | 6.37 |

Contrast with the player's `STAGE_A_TEST.md` (0.195 mean, 0/23 class flips) — that low drift
**only exists when reusing their frozen features + frozen class_dir**. Re-extracting features
moves PHQ by ~2.4 on average and flips ~1/3 of class labels.

## Known weakness of the honest run
`phq_threshold` ternary never assigns **class-2 (severe)** — the learned high threshold
(11.5) exceeds the compressed from-scratch PHQ range, so severe cases {69,91,106} that the
player's dedicated classification ensemble caught are missed. This is exactly the score the
frozen `class_dir` was buying. A `_variant_frozenclass_myphq/` is included that grants the
player their frozen classification and substitutes only my-features PHQ, to isolate the PHQ
contribution from the classification contribution when comparing on Codabench.

## Files
- `submission_honest_top_r1.zip` — **submit this** (fully from-scratch).
- `submission_honest_playeranalog_r3.zip` — same pipeline, the player's hand-picked rank.
- `_variant_frozenclass_myphq/submission.zip` — frozen classes + my PHQ (isolates PHQ).
- `binary.csv` / `ternary.csv` — the `_r1` predictions (human-readable).
- `stage_a_honest_report.json` — full 180-candidate CV report.

## TODO (needs Codabench, blocked in this env)
Submit the three zips, record scores in `results/leaderboard_scores.md`, and compare to the
claimed **0.5881**. Expected: the fully-honest `_r1` lands well below 0.5881 (RUNBOOK
estimate ≈0.56–0.57 was for the *class-frozen* hybrid; the fully-honest number is lower
because severe-class ternary is missed). The gap = score the player obtained by reusing
frozen classification products on top of the from-scratch modeling.
