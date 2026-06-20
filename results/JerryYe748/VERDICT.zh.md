# JerryYe748 · Track 1 (Elder) — 复现裁决

**日期:** 2026-06-20  **状态:** ⛔ **无法从头复现 — 被阻止。**

本次复现按照用于其他选手的同一原则（从原始数据重新提取所有特征，不使用捆绑的特征/权重）来重新验证之前的审查（`docs/audit_JerryYe748.md`、`repro/JerryYe748/RUNBOOK.md`）。

## 为什么被阻止（于 2026-06-20 确认）
该仓库是 **预提取特征之上的推理/训练包** — 它包含 **没有** 原始→特征提取代码：
- `grep` 任何解码/提取调用（`librosa.load`、`torchaudio.load`、`cv2.VideoCapture`、`soundfile.read`、`whisper.load_model`、`av.open`）→ **零命中**。
- 唯一的 `from_pretrained` 是 `models/gated_multifeature_multitask.py:27`，一个 **可选的 `--wavlm_mode e2e` 分支** 加载检查点；默认是 `offline`，提交推理（`scripts/run_elder_submission_model.py`）仅消费预提取的 `.npy`。音频/视频永不被解码。
- `dataset.py` 通过 `np.load` 的 `*.npy` 独家读取特征。
- `MANIFEST.json` 以 Release 资产的形式发布特征（例如训练 `wavlm` = 1.76 GB、`whisper` = 2.2 GB、`openface3` = 219 MB）— 消费，不生成。

没有选手的捆绑 `.npy` + `.pth/.cbm`，原始→特征→训练链无法运行。

## 规则问题（非官方特征）
模型 **硬依赖** **非官方、更强** 的特征，官方基线不使用：**WavLM-Large、Whisper-large-v3、LCO-Omni-7B**（音频），**OpenFace3**（面部，vs 官方 OpenFace2），**YOLOv11** 骨骼（步态）。换回官方特征会需要重新架构 + 重新调整门控多特征模型。

## 未发现内容（已清除）
- **无测试标签泄漏**：`build_test_payload()` 将所有测试标签初始化为虚拟 0，仅从 `split_labels_test.csv` 读取 ID；辅助 CatBoost 仅在训练标签上拟合。
- **无逐 id 硬编码**：推理是标准前向；辅助 CatBoost 仅通过 **全局** `aux_threshold` 提升/降低不一致样本，不按测试 ID。

## 确定性风险 — 中等
`train_elder_submission_model.py` 固定 `BINARY_PHQ_EPOCH=43`、`TERNARY_EPOCH=45`、`PRESERVE_SHUFFLE_SEQUENCE=True`；CatBoost `random_seed=3407` 但 `iterations=2`。结果受限于特定的训练轨迹/环境，不太可能在重新训练时再现相同分数。

## 裁决
根据审查原则（无捆绑中间特征/预训练权重），本提交 **无法被认证为从头可重现**。为验证，选手必须提供完整的 **原始→特征** 提取代码并为非官方特征辩护；然后从官方原始数据重新提取，从随机初始化重新训练，并重新提交到 Codabench。
