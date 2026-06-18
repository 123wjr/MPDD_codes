# 审查报告 · buptlyx/MPDD-AVG2026

- 仓库：<https://github.com/buptlyx/MPDD-AVG2026>
- 覆盖：Track 1 + Track 2，子赛道 A-V-P / A-V-G-P / G-P
- 审查日期：2026-06-18

## 总判定

**干净、可从头复现。** 唯一注意：Track 2 Young 测试集视频特征为空文件，影响其
A-V-P / A-V-G-P 分数解释（数据集问题，非作弊，选手已透明说明）。

---

## 1. 从头完整性 —— 完备
- `feature_extract/` 含完整抽取链：音频 mfcc/opensmile/wav2vec、视频 densenet/resnet/openface、
  人格 `gen_describtion.py`/`extrapersonality.py`，均**读原始媒体**用公开库（librosa、
  torchvision 等）生成 `.npy`。如 `feature_extract/audio/mfcc/extract_mfcc_embedding.py:49-85`
  从原始 wav 抽 64 维 MFCC。
- 训练→测试端到端可跑：`dataset.py` 从磁盘读特征、线性插值到 `target_t=128`；缺失/空文件
  优雅回退为零填充。

## 2. 测试集泄漏 —— 干净
- `train_val_split.py:28` 仅接受 `split ∈ {"", train, val}` 的行，**从不加载测试集**；
  用 `StratifiedShuffleSplit` 按 ID 分层切验证集，train/val ID 集合互斥（无受试者跨集泄漏）。
- `train.py`：早停与 best ckpt 选择**只用验证集指标**（`current_score = get_selection_score(val_metrics, ...)`），测试集训练期从不读。
- `test.py`：`build_unlabeled_task_maps()` 用全 0 占位标签，预测仅 `argmax(logits)`，
  只输出 `id/pred/phq9_pred`，**不读测试标签、无 per-id 硬编码**。
- ✅ 印证 README 宣称："测试集仅在训练结束后评测一次""验证集只从 train 内按 ID 切分"。

## 3. 模型选择 —— 干净
- best ckpt 仅在验证指标严格提升时覆盖（分类按 F1/Kappa，回归按 CCC）。

## 4. 权重 / 种子
- `.gitignore` 排除 `*.pth`，clone 内**无**已训练权重（README 的"权重稍后上传"为承诺，
  当前不存在）。⇒ 必须从头训练，无捷径。
- `train.py:93-98` `setup_seed()` 覆盖 random/numpy/torch/cuda；config 默认 `seed=3407`。
  脚本固定超参，利于复现（GPU 非确定性致非字节级一致属正常）。

## 5. 数据质量注意（影响分数解释，非作弊）
- README 自述：发布数据中 `MPDD-AVG2026-test/Young/Video/{densenet,resnet,openface}` 为
  **0 字节空文件**，故 Track2 A-V-P / A-V-G-P 测试时视频分支被**零填充**（`dataset.py`
  对空文件回退零矩阵）。⇒ 这两个子赛道的测试分数**不反映视频贡献**，存在训练(真视频)/
  测试(零视频)分布偏移；**Track2 G-P 不受影响**。此外 Young trainval 的 openface 也"不完全有效"。
- 这是**数据集发布问题**，选手已在 README 透明说明并提示交叉核对训练日志中的有效样本数。

## 结论与复现要点
- 通过审查；可作为"诚实从头复现"的正面参照（甚至可用其官方特征抽取为其他选手补齐缺失环节）。
- 复现见 `repro/buptlyx/RUNBOOK.md`。核对 Track2 A-V-P/A-V-G-P 分数时务必注明视频特征为空的前提。
