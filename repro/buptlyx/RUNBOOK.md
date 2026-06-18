# 复现手册 · buptlyx/MPDD-AVG2026

审查结论：**干净、可从头复现**。本手册让你在本地 RTX 5080 上从原始数据起跑通全流程。

## 0. 取代码与环境
```bash
git clone https://github.com/buptlyx/MPDD-AVG2026.git
cd MPDD-AVG2026
conda create -n mpddavg python=3.10 -y && conda activate mpddavg
pip install --upgrade pip
pip install torch torchvision torchaudio        # 选与 CUDA 匹配的版本
pip install numpy scikit-learn librosa opensmile opencv-python tqdm
```

## 1. 数据
按 `docs/00_environment_and_data.md` 从 HuggingFace 下载官方数据，软链/放到仓库根：
```
MPDD-AVG2026/MPDD-AVG2026-trainval/{Elder,Young}/...
MPDD-AVG2026/MPDD-AVG2026-test/{Elder,Young}/...
```

## 2. 从原始数据重抽特征（关键：不要直接用官方/选手预抽特征）
```bash
# 示例：音频 MFCC（其余 opensmile/wav2vec、视频 densenet/resnet/openface 同目录）
python feature_extract/audio/mfcc/extract_mfcc_embedding.py    # 按脚本内路径参数调整
python feature_extract/visual/densenet/extract_densenet_embedding.py
# ... 逐个模态跑完，产出 .npy 落到对应 Audio/Video 子目录
```
> 视频特征抽取较慢，优先用 GPU。OpenFace 需要 OpenFace2 工具链。

## 3. 从头训练（验证集只从 train 切，测试集最后才碰）
```bash
# Track1 / Elder / A-V-G-P 二分类（脚本自动遍历 3×3 音视频组合）
bash scripts/Track1/A-V-G-P/run_binary.sh
bash scripts/Track1/A-V-G-P/run_ternary.sh
# Track2 / Young 同理
bash scripts/Track2/A-V-G-P/run_binary.sh
```
可用环境变量覆盖：`DEVICE/SEED/EPOCHS/BATCH_SIZE/LR/...`。

## 4. 测试集评测（只跑一次）+ 生成提交
```bash
python test.py --checkpoint checkpoints/.../best_model_*.pth \
  --data_root MPDD-AVG2026/MPDD-AVG2026-test/Young \
  --split_csv MPDD-AVG2026/MPDD-AVG2026-test/Young/split_labels_test.csv \
  --personality_npy MPDD-AVG2026/MPDD-AVG2026-trainval/Young/descriptions_embeddings_with_ids.npy
```

## 5. 核对与记录
- 把生成的提交传 Codabench，与排行榜分数比对，填入 `results/leaderboard_scores.md`。
- ⚠️ **Track2 A-V-P / A-V-G-P**：官方数据 Young 测试集视频特征为 0 字节空文件，视频分支
  被零填充——记录分数时务必注明此前提；**G-P 子赛道不受影响**，可作为干净对照。
- 把训练日志/指标/提交拷到 `results/buptlyx/<赛道>/`（勿提交大文件/数据）。
