# HelloWorldLTY · Track 1 (Elder) — 诚实的从头复现

**日期:** 2026-06-20  **环境:** conda `mpdd`  **目标:** 量化选手宣称的 Track-1 成绩（**0.5881**）与诚实从头复现之间的差距，即从原始数据重新提取特征且不重复使用任何冻结的提交物。

## TL;DR
- 已发布的 `reprod_old/stage_a/run_stage_a.sh` **不是** 从头运行。它仅因为 **从冻结的 `class_dir/` 复制二分/三分预测**（选手的预计算测试集类别预测，本身是从他们许多之前的提交集成的 — 见 `class_dir/class_phq_rank1_config_zh.md`）才使分类逐字节精确再现。仅重新计算 **PHQ 回归**。
- 选手自己的 `stage_a/STAGE_A_TEST.md` 报告 PHQ |Δ|=**0.195 平均值** vs 他们的参考 — 但那是通过 **使用他们自己的冻结特征 + 冻结 class_dir** 实现的。
- 本复现改为 **从原始数据重新提取所有 A/V 特征** 并 **从重新计算的 PHQ 导出类别**（`--class_mode phq_threshold`，阈值仅在训练 CV 上学习）。不重复使用任何冻结产品。结果与冻结参考有实质性差异（下文），证实宣称的可重现性是重复使用冻结产品的产物。

## 自提取内容（从原始数据，2026-06-20）
| 模态 | 特征 | 来源 |
|---|---|---|
| **音频 (A)** | mfcc (librosa n_mfcc=64), opensmile (ComParE_2016 LLD, 65-d), wav2vec2 (facebook/wav2vec2-base-960h last_hidden_state, 768-d) | **从原始 `.WAV` 自提取** |
| **视频 (V)** | densenet121 (torchvision, 1000-d), openface (docker `algebr/openface`, 710-d from CSV cols 4:) | **从原始 `.avi/.mp4` 自提取** |
| **步态 (G)** | IMU sequence `.npy` | **官方数组**（原始传感器数据，非学习提取器）— 文档化的注意事项 |
| **性格 (P)** | `descriptions_embeddings_with_ids.npy` (1024-d) | **官方数组** — 选手未指定文本编码器配方；重新推导会增加猜测而非诚实性 — 文档化的注意事项 |

数量：354 个训练 + 88 个测试 `.npy` 每个 A/V 特征，0 个提取失败。
wav2vec2 模型通过 ModelScope 镜像获取（`AI-ModelScope/wav2vec2-base-960h`）。

## 运行
`tmp/run_stage_a_honest_elder.sh` → `generate_sklearn_phqstack_modular_avgp_submission_stratmeta.py`
配合选手的确切特征/回归器/堆叠配置 **加上 `--class_mode phq_threshold`**
和指向自提取特征的数据树。生成 180 个候选；排名按内部 CV `candidate_score`（无排行榜偷窥）。

- **顶部候选（`_r1`）**: `reg_rankensemble_top3_median`，CV ccc=0.402，CV f1=0.624，CV kappa=0.363，mae=0.704。→ `submission_honest_top_r1.zip`（**主要诚实提交**）
- **选手模拟（`_r3`）**: `reg_rankensemble_top2_trimmed`。→ `submission_honest_playeranalog_r3.zip`

## 诚实的 `_r1` vs 选手的冻结参考（`stage_a/expected_output/`）
| 任务 | 类别翻转（共 23） | PHQ \|Δ\| 平均 | PHQ \|Δ\| 最大 |
|---|---|---|---|
| 二分 | 8 | 2.38 | 6.37 |
| 三分 | 8 | 2.38 | 6.37 |

对比选手的 `STAGE_A_TEST.md`（0.195 平均值，0/23 类别翻转）— 那种低漂移 **仅在重复使用其冻结特征 + 冻结 class_dir 时存在**。重新提取特征将 PHQ 平均移动约 2.4，并翻转约 1/3 的类别标签。

## 诚实运行的已知弱点
`phq_threshold` 三分永不分配 **class-2（重度）** — 学习的高阈值 (11.5) 超过压缩的从头 PHQ 范围，因此选手的专用分类集成捕获的重度案例 {69,91,106} 被遗漏。这恰好是冻结的 `class_dir` 所带来的分数。包含了一个 `_variant_frozenclass_myphq/` 版本，它给予选手冻结的分类并仅替换我的特征 PHQ，以在 Codabench 上比较时隔离 PHQ 贡献与分类贡献。

## 文件
- `submission_honest_top_r1.zip` — **提交此文件**（完全从头）。
- `submission_honest_playeranalog_r3.zip` — 同样的管道，选手的精心挑选排名。
- `_variant_frozenclass_myphq/submission.zip` — 冻结的类别 + 我的 PHQ（隔离 PHQ）。
- `binary.csv` / `ternary.csv` — `_r1` 预测（人类可读）。
- `stage_a_honest_report.json` — 完整的 180 候选 CV 报告。

## TODO（需要 Codabench，在此环境中被阻止）
提交三个 zip，记录分数到 `results/leaderboard_scores.md`，并与宣称的 **0.5881** 比较。预期：完全诚实的 `_r1` 远低于 0.5881（RUNBOOK 估计 ≈0.56–0.57 是针对 *类别冻结* 混合；完全诚实的数字更低因为重度类别三分被遗漏）。差距 = 选手通过在从头建模之上重复使用冻结分类产品所获得的分数。
