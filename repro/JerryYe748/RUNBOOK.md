# 复现手册 · JerryYe748/MPDD-AVG-2026-Elder-submission

审查结论：**仅凭该仓库无法从头复现**（无特征抽取代码，依赖随包预抽特征 + 训练权重，
且用非官方特征）。本手册说明"诚实从头复现"需要什么、目前缺什么。

## ⛔ 复现红线
- **不加载**随 Release 提供的训练权重：`model/models/elder_binary_phq_model.pth`、
  `elder_ternary_model.pth`、`elder_auxiliary_binary_model.cbm`。
- **不使用**随包预抽特征 `.npy`（wavlm/whisper/lco/openface3_compact/skeleton 等）。
- 即排除 `run_elder_submission_model.py` 的"从权重直接推理"路径。

## 0. 取代码
```bash
git clone https://github.com/JerryYe748/MPDD-AVG-2026-Elder-submission.git
cd MPDD-AVG-2026-Elder-submission
conda create -n elder python=3.10 -y && conda activate elder
pip install -r requirements.txt   # numpy1.26.2 scipy torch2.1.2 catboost transformers4.36.2 ...
```

## 1. 关键障碍：缺特征抽取代码（需选手补交）
该模型训练消费以下**非官方**特征，但仓库**不含**其从原始数据的抽取代码：

| 模态 | 特征 | 抽取依赖（需自建/选手补交） |
| ---- | ---- | --------------------------- |
| 音频 | wavlm | `microsoft/wavlm-large`，16kHz、30s 分块，存 `last_hidden_state` (1024d) |
| 音频 | whisper | `openai/whisper-large-v3` 编码器隐状态 (1280d) |
| 音频 | lco | `LCO-Embedding/LCO-Embedding-Omni-7B`（SentenceTransformers），3584d/clip |
| 视频 | openface3_compact | OpenFace3 流水线（**非**官方 OpenFace2），222d/frame |
| 视频 | skeleton_relative | YOLOv11 骨架→相对坐标 |
| IMU  | gait | 官方 IMU 序列（12 通道） |
| 文本 | descriptions_(structured_)embeddings | 由官方描述文本编码 |

> 要诚实复现，必须先**自建上述抽取脚本**从 HuggingFace 官方原始音视频/IMU 重抽特征。
> 这部分选手未提供 → 建议通过群里/组委会**要求其补交特征抽取代码**，否则无法认定可复现。

## 2. 从头训练（特征齐备后）
```bash
cd source
python scripts/train_elder_submission_model.py \
  --binary-phq-config ../model/configs/elder_submission_binary_phq_model.json \
  --ternary-config   ../model/configs/elder_submission_ternary_model.json \
  --train-data-root  <你重抽特征后的 trainval/Elder> \
  --train-split-csv  <.../split_labels_train.csv> \
  --official-npy     <.../descriptions_embeddings_with_ids.npy> \
  --package-dir      ../outputs/trained_model
```
> 注意：`train_elder_submission_model.py` 钉死 `BINARY_PHQ_EPOCH=43`、`TERNARY_EPOCH=45`，
> 且 `PRESERVE_SHUFFLE_SEQUENCE=True` 注入随机 → 结果绑定特定轨迹，**重训难复现同分**。

## 3. 推理 + 提交（用你自训的权重，而非随包权重）
```bash
python scripts/run_elder_submission_model.py \
  --package-dir ../outputs/trained_model \
  --output-dir ../outputs/inference_details \
  --submission-dir ../outputs/submission
```

## 4. 核对与记录
- 比对自训分数与排行榜分数，填 `results/leaderboard_scores.md`。
- 记录两点供组委会：①缺特征抽取代码（不可独立复现）；②使用非官方特征（OpenFace3/
  YOLOv11/WavLM/Whisper/LCO）是否违规。证据拷到 `results/JerryYe748/`。
