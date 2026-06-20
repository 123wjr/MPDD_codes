# buptlyx · Track2 Young · A-V-G-P — 从头复现记录

日期：2026-06-19　执行环境：RTX 5080 Laptop / conda env `mpdd` (torch 2.12+cu130)

## 结论
**干净、可从头复现成功**。从官方原始音视频/IMU 起步，自抽特征 → 随机初始化训练 →
仅评测一次测试集 → 生成 Codabench `submission.zip`，全流程跑通，使用选手自己的代码。

> 重要更正（针对 README/RUNBOOK 的"Young 测试视频为空"提示）：
> 该提示针对的是**官方预抽取的视频特征 .npy**（部分为 0 字节）。本次复现是
> **从原始 `.avi` 自行重抽 densenet 特征**，22 个测试 id 的原始视频文件全部非空
> （2–19 MB/段，66 段 0 失败），自抽 densenet 特征也全部非空（2–7 MB/段，66 个 .npy）。
> 因此本次 **A-V-G-P 复现使用的是真实视频特征**，不受官方空特征问题影响。

## 流程（全部从原始数据起）
1. 数据：HF `chasonfff/MPDD-AVG-2026`，解压 trainval/test 的 Young + privacy-constrained-raw-Young(-test)。
2. 重抽特征（**不用**官方/选手预抽 .npy）：
   - `mfcc`（librosa, n_mfcc=64）：train 264 + test 66 段，0 失败。
   - `densenet`（densenet121 ImageNet 预训练，逐帧）：train 264 + test 66 段，0 失败。
   - `gait`=官方原始 IMU 序列，`personality`=官方 descriptions_embeddings（原始/官方允许项）。
3. 训练：`scripts/Track2/A-V-G-P` run_binary + run_ternary，seed=42，val 从 train 切（79/9），
   bilstm_mean，selection=kappa，early-stop patience=90，weighted_sampler。
   binary best@27（早停），ternary best@66（早停）。
4. 评测：`test.py` 对 22 个测试 id 推理，`make/make.py` 打包 → `submission.zip`。

## 从头复现的验证集指标（val=9，样本少，仅供参考）
| 任务 | best epoch | val_kappa | val_f1 | val_acc | scoretrack |
| ---- | ---------- | --------- | ------ | ------- | ---------- |
| binary  | 27 | 0.372 | 0.649 | 0.667 | 0.341 |
| ternary | 66 | 0.289 | 0.423 | 0.667 | 0.228 |

测试集预测分布（22 id）：binary 9×0 / 13×1；ternary 20×0 / 1×1 / 1×2（+PHQ-9 回归）。

## 复现所需修正（与 Track1 Elder 一致 + Young 专属 1 处）
- `test.py` 的 3 处 glue 修正（split_csv 回退、`ids`→`pid` 回退、forward 关键字调用）
  与 Elder 完全相同，见 `../Track1_Elder/REPRO_NOTES.md`。本次直接复用同一份已改 `test.py`。
- **Young 专属数据归一化**：官方 Young trainval CSV 的 PHQ 列名为 `phq9_score`，
  但选手 `dataset.py` 硬编码 `PHQ-9`（Elder 用 `PHQ-9`）。在 Young CSV 上补一列
  `PHQ-9`（= `phq9_score` 拷贝），属数据归一化、非逻辑改动。
  > 该列名漂移再次说明选手代码早于当前数据发布版本，但不影响"可从头复现"判定。

## 待办
- 真实排行榜分数需在本地浏览器把 `submission.zip` 传 Codabench(16079) 后填入
  `results/leaderboard_scores.md`（远程环境屏蔽 codabench.org）。
- 产物：`submission/{binary,ternary}.csv,submission.zip`；本目录存指标/历史/提交 csv（不含大特征/数据）。
