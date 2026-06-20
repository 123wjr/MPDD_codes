# buptlyx · Track1 Elder · A-V-G-P — 从头复现记录

日期：2026-06-19　执行环境：RTX 5080 Laptop / conda env `mpdd` (torch 2.12+cu130)

## 结论
**干净、可从头复现成功**。从官方原始音视频/IMU 起步，自抽特征 → 随机初始化训练 →
仅评测一次测试集 → 生成 Codabench `submission.zip`，全流程跑通，使用选手自己的代码。

## 流程（全部从原始数据起）
1. 数据：HF `chasonfff/MPDD-AVG-2026`，解压 Train/Test-MPDD-Elder + privacy-constrained-raw-Elder。
2. 重抽特征（**不用**官方/选手预抽 .npy）：
   - `mfcc`（librosa, n_mfcc=64）：train 354 + test 88 段，0 失败，维度 (T,64) 与官方一致。
   - `densenet`（densenet121 ImageNet 预训练，逐帧）：train 354 + test 88，0 失败，维度 (T,1000)。
   - `gait`=官方原始 IMU 序列(12ch)，`personality`=官方 descriptions_embeddings（均为原始/官方允许项）。
   - 特征抽取用自写的 per-video 队列驱动并行（import 选手原函数，数值不变，仅调度更快）。
3. 训练：`scripts/Track1/A-V-G-P/run_binary.sh` + `run_ternary.sh`，seed=42，val 从 train 切（78/9），
   bilstm_mean，selection=kappa，early-stop patience=90。binary 第111轮停(best@21)，ternary 第103轮停(best@13)。
4. 评测：`test.py` 对 23 个测试 id 推理，`make/make.py` 打包 → `submission/submission.zip`。

## 从头复现的验证集指标（val=9，样本少，仅供参考）
| 任务 | best val_kappa | val_f1 | val_acc |
| ---- | -------------- | ------ | ------- |
| binary  | 0.308 | 0.649 | 0.667 |
| ternary | 0.129 | 0.286 | 0.667 |

测试集预测分布：binary 17×0 / 6×1；ternary 21×0 / 1×1 / 1×2（+PHQ-9 回归）。

## 复现 test.py 所需的 3 处 glue 修正（选手代码与模型/数据已漂移，记录备查）
1. `--split_csv` 未传时回退到 checkpoint 里训练用的 split_csv（只有 split=train 行）→ 测试 test_map 为空、
   报 "No valid samples"。解决：用测试 id 生成 `split_labels_test.csv`(split=test, 占位标签) 显式传入。
2. `predict()` 取 `batch["ids"]`，但 dataset 实际输出键为 `pid` → KeyError。改为回退到 `pid`。
3. `predict()` 调 `model(batch, only_modality=None)`，但 `TorchcatBaseline.forward` 只收
   `audio/video/gait/personality/pair_mask` 关键字并返回 (logits, reg)。改为按 train.py 的方式调用。
   > 这些是选手 test.py 自身的 bug（与其训练代码/模型不同步），不影响"可从头复现"判定，
   > 但说明其测试脚本未经端到端验证。

## 待办
- 真实排行榜分数需在本地浏览器把 `submission.zip` 传 Codabench(16077) 后填入
  `results/leaderboard_scores.md`（远程环境屏蔽 codabench.org）。
- 产物：`submission/{binary,ternary}.csv,submission.zip`；本目录存指标/历史/提交 csv（不含大特征/数据）。
