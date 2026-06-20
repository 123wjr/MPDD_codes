# HelloWorldLTY · Track 2 (Young) — honest from-scratch reproduction

**Date:** 2026-06-20  **Env:** conda `mpdd`  **Goal:** quantify the gap between the
player's claimed Track-2 score (final **0.566211**, dir `reprod_young/young.566211/`) and
what is honestly reproducible when you re-extract every feature from raw and never reuse a
frozen submission artifact, leaked labels, or the Stage-B recombination.

## TL;DR
- The player's headline Track-2 number (0.566211) comes from
  `young.566211/final_recombination_from_local/Track2_submit/` — a **Stage-B recombination**
  of many hand-picked candidate submissions, not a single from-scratch run.
- This reproduction runs **Stage A only**: `generate_sklearn_phqstack_modular_avgp_submission.py`
  with `--feature_profile balanced_pca_compact_eventwide` (uses **only the 5 self-extracted
  A/V features** — no whisper/resnet/qwenvl/osdn extras) and `--class_mode phq_threshold`
  (classes derived from regenerated PHQ, thresholds learned on train CV only). **No Stage B,
  no `mpdd_2025` leaked labels, no recombination of hand-picked candidates.**
- Result diverges materially from the player's frozen 0.566211 reference (below), confirming
  the published score depends on products this from-scratch run deliberately excludes.

## What was self-extracted (from raw, 2026-06-20)
All from raw `event_N.wav` / `event_N.avi`, renamed to the event scheme `dataset.py` expects
(audio `E1/E2/E3.npy`, video `event_1/2/3.npy`). 264 train (88 ids × 3 events) + 66 test
(22 ids × 3) `.npy` per feature, 0 extraction failures.

| modality | features | source |
|---|---|---|
| **Audio (A)** | mfcc (librosa n_mfcc=64, FRAME), opensmile (ComParE_2016 LLD, 65-d), wav2vec2 (facebook/wav2vec2-base-960h last_hidden_state, 768-d) | **self-extracted from raw `.wav`** |
| **Video (V)** | densenet121 (torchvision, 1000-d), openface (docker `algebr/openface`, 710-d from CSV cols 4:) | **self-extracted from raw `.avi`** |
| **Gait (G)** | IMU sequence `.npy` | **official array** (raw sensor data, not a learned extractor) — documented caveat |
| **Personality (P)** | `descriptions_embeddings_with_ids.npy` (1024-d) | **official array** — player does not specify the text-encoder recipe; re-deriving would add guesswork, not honesty — documented caveat |

OpenFace was extracted with K=3 shards and a continuous bmp/hog janitor to keep disk bounded
(the docker build writes per-frame bmp/hog and self-cleans only at shard end).

## Run
`tmp/run_stage_a_honest_young.sh` → `generate_sklearn_phqstack_modular_avgp_submission.py`
(`--track Track2 --class_mode phq_threshold --feature_profile balanced_pca_compact_eventwide
--candidate_family stack_only --n_folds 10 --max_outputs 180`), data tree pointed at the
self-extracted features. 180 candidates produced; ranking is by internal CV `candidate_score`
(no leaderboard peeking).

Two code-packaging fixes were needed (neither alters the modeling):
- The Young `code_stage_a/` shares its (byte-identical) `.py` codebase with
  `reprod_old/stage_a/code` but does not ship the `models/` package; it was copied from there.
- `balanced_pca_compact_eventwide` requires **mfcc** (`("mfcc","densenet","stats_pz")` +
  `event_stats_pz`); mfcc had not been extracted for Young, so it was self-extracted from raw
  before the run.

## Selected candidates
- **Top (`_r1`)**: `reg_rankensemble_top5_median`, CV cand_score=0.2593, ccc=0.2848,
  mae=0.631, class_f1=0.560, kappa=0.312. → `submission_honest_top_r1.zip`
  (**primary honest submission**).
- **Alternate (`_r2`)**: `reg_rankensemble_top3_median`, CV cand_score=0.2576 (effectively
  tied with `_r1`). Learned binary threshold 3.5 instead of 1.5, giving a less degenerate
  binary split (see below). → `submission_honest_alt_r2.zip`. Provided as the next-ranked
  alternate (selected by CV rank, **not** by test-set peeking).

## Honest `_r1` vs player's frozen reference (`reference/0566211_submission.zip`)
| task | class flips (of 22) | PHQ \|Δ\| mean | PHQ \|Δ\| max |
|---|---|---|---|
| binary | 12 | 3.16 | 9.07 |
| ternary | 13 | 3.16 | 9.07 |

Class distributions (test, 22 ids):

| | binary 0 / 1 | ternary 0 / 1 / 2 |
|---|---|---|
| player frozen ref (0.566211) | 12 / 10 | 11 / 10 / 1 |
| honest `_r1` | 0 / 22 | 5 / 15 / 2 |
| honest `_r2` (alt) | 5 / 17 | 5 / 14 / 3 |

## Known weakness of the honest run
The from-scratch binary collapses to **all-positive** in `_r1` (the CV-learned threshold 1.5
sits below the compressed from-scratch PHQ range, so every test id lands positive). The
player's frozen reference is balanced (12/10) — that balance is exactly what the Stage-B
recombination of hand-picked candidates was buying. Unlike the Elder run, the honest Young
ternary **does** reach severe (class 2): 2 cases in `_r1`, 3 in `_r2`. The `_r2` alternate
(threshold 3.5) is included precisely because its binary split is less degenerate while still
being a CV-ranked candidate, not a test-peeked one.

## Files
- `submission_honest_top_r1.zip` — **submit this** (fully from-scratch, Stage A only).
- `submission_honest_alt_r2.zip` — next-ranked CV alternate, less degenerate binary.
- `binary.csv` / `ternary.csv` — the `_r1` predictions (human-readable).
- `stage_a_honest_report.json` — full 180-candidate CV report.

## TODO (needs Codabench, blocked in this env)
Submit the two zips, record scores in `results/leaderboard_scores.md`, and compare to the
claimed **0.566211**. Expected: the fully-honest Stage-A-only number lands below 0.566211 —
the gap = the score the player obtained from the Stage-B recombination / hand-picked candidate
products that this from-scratch run excludes.
