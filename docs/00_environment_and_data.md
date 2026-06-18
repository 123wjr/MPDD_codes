# 00 · 环境搭建与数据准备（通用）

本页是所有选手复现的共同前置步骤：拿到官方原始数据、准备特征抽取所需的预训练编码器。
**每个选手的 conda 环境不同**，各自的环境装在对应 `repro/<选手>/RUNBOOK.md` 里。

---

## 1. 下载官方数据集（HuggingFace）

官方数据集：<https://huggingface.co/datasets/chasonfff/MPDD-AVG-2026>

```bash
pip install -U "huggingface_hub[cli]"

# 建议放在本仓库外的大容量目录，避免误提交进 git
export MPDD_DATA=~/datasets/MPDD-AVG-2026
huggingface-cli download chasonfff/MPDD-AVG-2026 \
  --repo-type dataset \
  --local-dir "$MPDD_DATA"
```

期望目录结构（官方 trainval + test）：

```
MPDD-AVG-2026/
├── MPDD-AVG2026-trainval/
│   ├── Elder/   { Audio/ Video/ IMU/ split_labels_train.csv  descriptions_embeddings_with_ids.npy }
│   └── Young/   { Audio/ Video/ IMU/ split_labels_train.csv  descriptions_embeddings_with_ids.npy }
└── MPDD-AVG2026-test/
    ├── Elder/   { Audio/ Video/ IMU/ split_labels_test.csv }
    └── Young/   { Audio/ Video/ IMU/ split_labels_test.csv }
```

标签列：`ID, split, label2(二分类), label3(三分类), PHQ-9`。

> ⚠️ **从头复现的关键**：官方包里已经带了一部分预抽取特征 (.npy)。审查要求是
> **不直接信任选手随包附带的特征**，而是用官方原始 Audio/Video/IMU 配合
> *官方 baseline 的特征抽取脚本* 重新生成。若某选手用了**非官方**的更强特征
> （见其审查报告），需单独标注为规则问题，并在"按其方法复现"与"按官方特征复现"
> 两条线分别记录分数。

---

## 2. 特征抽取所需的预训练编码器

官方 baseline (buptlyx 仓库的 `feature_extract/`) 复现需要：

| 模态 | 特征 | 依赖 |
| ---- | ---- | ---- |
| Audio | mfcc / opensmile | `librosa`, `opensmile` |
| Audio | wav2vec | `transformers` + wav2vec2 权重 (HF) |
| Video | densenet / resnet | `torchvision` 预训练权重 |
| Video | openface | OpenFace 2.x 工具链 |
| IMU | gait | 直接读取官方 IMU 序列 |
| Personality | descriptions_embeddings | 官方已提供 .npy |

> 选手 HelloWorldLTY / JerryYe748 使用了**额外/更新**的编码器（WavLM、Whisper-large-v3、
> LCO-Embedding-Omni-7B、OpenFace3、YOLOv11 骨架等）。这些在各自 RUNBOOK 中单列，
> 下载体积大、且属于"非官方特征"，复现时需特别留意是否符合赛规。

---

## 3. 通用约定

- 数据放仓库外（用环境变量 `MPDD_DATA` 指向），`results/` 只存指标 / csv / 日志，
  **不要把数据或大权重提交进 git**。
- 复现产物统一写到 `results/<选手>/<赛道>/`，文件名带时间戳。
- 每条复现记录三件事：**(a) 跑的命令、(b) 得到的提交文件、(c) 与排行榜分数的差距**。

---

## 4. 排行榜分数获取

Codabench 排行榜（[16077](https://www.codabench.org/competitions/16077/) /
[16079](https://www.codabench.org/competitions/16079/)）记录各选手提交分数，
作为"真值"用于核对。请在本地浏览器查看并把每位选手的最终分数填入
`results/leaderboard_scores.md`。

> 备注：本远程环境网络策略屏蔽了 codabench.org，故分数核对需在你本地完成。
