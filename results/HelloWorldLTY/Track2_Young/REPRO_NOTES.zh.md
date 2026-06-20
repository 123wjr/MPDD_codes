# HelloWorldLTY · Track 2 (Young) — 诚实的从头复现

**日期:** 2026-06-20  **环境:** conda `mpdd`  **目标:** 量化选手宣称的 Track-2 成绩（最终 **0.566211**，目录 `reprod_young/young.566211/`）与诚实从头复现之间的差距，即从原始数据重新提取所有特征，且不重复使用任何冻结的提交物、泄露的标签或 Stage-B 重组。

## TL;DR
- 选手的头条 Track-2 数字 (0.566211) 来自 `young.566211/final_recombination_from_local/Track2_submit/` — 这是多个精心挑选的候选提交的 **Stage-B 重组**，而非单一的从头运行。
- 本次复现仅运行 **Stage A**: `generate_sklearn_phqstack_modular_avgp_submission.py`，配合 `--feature_profile balanced_pca_compact_eventwide`（仅使用 **5 个自提取的 A/V 特征** — 无 whisper/resnet/qwenvl/osdn 额外功能）和 `--class_mode phq_threshold`（类别从重新生成的 PHQ 导出，阈值仅在训练 CV 上学习）。**无 Stage B，无 `mpdd_2025` 泄露标签，无精心挑选候选的重组。**
- 结果与选手冻结的 0.566211 参考值有实质性差异（见下文），证实发布的成绩取决于本从头运行刻意排除的产品。

## 自提取内容（从原始数据，2026-06-20）
所有来自原始 `event_N.wav` / `event_N.avi`，重命名为 `dataset.py` 所需的事件方案（音频 `E1/E2/E3.npy`，视频 `event_1/2/3.npy`）。264 个训练 (88 个 id × 3 个事件) + 66 个测试 (22 个 id × 3 个) `.npy` 每个特征，0 个提取失败。

| 模态 | 特征 | 来源 |
|---|---|---|
| **音频 (A)** | mfcc (librosa n_mfcc=64, FRAME), opensmile (ComParE_2016 LLD, 65-d), wav2vec2 (facebook/wav2vec2-base-960h last_hidden_state, 768-d) | **从原始 `.wav` 自提取** |
| **视频 (V)** | densenet121 (torchvision, 1000-d), openface (docker `algebr/openface`, 710-d from CSV cols 4:) | **从原始 `.avi` 自提取** |
| **步态 (G)** | IMU sequence `.npy` | **官方数组**（原始传感器数据，非学习提取器）— 文档化的注意事项 |
| **性格 (P)** | `descriptions_embeddings_with_ids.npy` (1024-d) | **官方数组** — 选手未指定文本编码器配方；重新推导会增加猜测而非诚实性 — 文档化的注意事项 |

OpenFace 以 K=3 分片和连续 bmp/hog 清理器提取，以保持磁盘使用量有界（docker 构建逐帧写入 bmp/hog，仅在分片末端自清理）。

## 运行
`tmp/run_stage_a_honest_young.sh` → `generate_sklearn_phqstack_modular_avgp_submission.py`
（`--track Track2 --class_mode phq_threshold --feature_profile balanced_pca_compact_eventwide
--candidate_family stack_only --n_folds 10 --max_outputs 180`），数据树指向自提取的特征。生成 180 个候选；排名按内部 CV `candidate_score`（无排行榜偷窥）。

需要两个代码打包修复（均不改变建模）：
- Young `code_stage_a/` 与 `reprod_old/stage_a/code` 共享其（字节相同）`.py` 代码库但不包含 `models/` 包；包已从那里复制。
- `balanced_pca_compact_eventwide` 需要 **mfcc**（`("mfcc","densenet","stats_pz")` + `event_stats_pz`）；Young 未提取 mfcc，因此在运行前从原始数据自提取。

## 选定的候选
- **顶部（`_r1`）**: `reg_rankensemble_top5_median`，CV cand_score=0.2593，ccc=0.2848，mae=0.631，class_f1=0.560，kappa=0.312。→ `submission_honest_top_r1.zip`（**主要诚实提交**）。
- **替代（`_r2`）**: `reg_rankensemble_top3_median`，CV cand_score=0.2576（实际与 `_r1` 并列）。学习的二分阈值 3.5 而非 1.5，给出更少退化的二分划分（见下文）。→ `submission_honest_alt_r2.zip`。作为下一排名的替代提供（由 CV 排名选择，**非** 通过测试集偷窥）。

## 诚实的 `_r1` vs 选手的冻结参考（`reference/0566211_submission.zip`）
| 任务 | 类别翻转（共 22） | PHQ \|Δ\| 平均 | PHQ \|Δ\| 最大 |
|---|---|---|---|
| 二分 | 12 | 3.16 | 9.07 |
| 三分 | 13 | 3.16 | 9.07 |

类别分布（测试，22 个 id）：

| | 二分 0 / 1 | 三分 0 / 1 / 2 |
|---|---|---|
| 选手冻结参考 (0.566211) | 12 / 10 | 11 / 10 / 1 |
| 诚实 `_r1` | 0 / 22 | 5 / 15 / 2 |
| 诚实 `_r2`（替代） | 5 / 17 | 5 / 14 / 3 |

## 诚实运行的已知弱点
从头二分在 `_r1` 中坍缩为 **全正** （CV 学习的阈值 1.5 低于压缩的从头 PHQ 范围，因此每个测试 id 都落入正类）。选手的冻结参考是平衡的 (12/10) — 那个平衡恰好是 Stage-B 精心挑选候选重组所带来的。与 Elder 运行不同，诚实的 Young 三分 **确实** 达到了重度（类别 2）：在 `_r1` 中 2 个案例，在 `_r2` 中 3 个。`_r2` 替代（阈值 3.5）被包含正是因为其二分划分不那么退化，同时仍是 CV 排名的候选，而非测试偷窥的候选。

## 文件
- `submission_honest_top_r1.zip` — **提交此文件**（完全从头，仅 Stage A）。
- `submission_honest_alt_r2.zip` — 下一排名 CV 替代，二分不那么退化。
- `binary.csv` / `ternary.csv` — `_r1` 预测（人类可读）。
- `stage_a_honest_report.json` — 完整的 180 候选 CV 报告。

## TODO（需要 Codabench，在此环境中被阻止）
提交两个 zip，记录分数到 `results/leaderboard_scores.md`，并与宣称的 **0.566211** 比较。预期：完全诚实的 Stage-A-only 数字低于 0.566211 — 差距 = 选手从 Stage-B 重组/精心挑选候选产品中获得的分数，本从头运行刻意排除的产品。
