# 审查报告 · JerryYe748/MPDD-AVG-2026-Elder-submission

- 仓库：<https://github.com/JerryYe748/MPDD-AVG-2026-Elder-submission>
- 覆盖：Track 1 (Elder, AVG+P)
- 权重(.pth/.cbm) 与预抽特征(.npy) 以 GitHub Release 分卷资产提供（不在 clone 内）
- 审查日期：2026-06-18

## 总判定

**仅凭本仓库无法从头复现：代码只消费预打包特征 + 随包训练权重，且使用非官方更强特征。**
未发现测试集标签泄漏或 per-id 硬编码。

---

## 1. 从头复现可行性 —— 不可行
- `dataset.py` 的 `_load_feature_array()` 只从磁盘读 `.npy`，**不做任何特征抽取**。
- 仓库内**没有**任何从原始音视频/IMU 抽取 WavLM / Whisper / LCO / OpenFace3 /
  YOLOv11 骨架 / 结构化描述特征的代码。
- README 自述：特征"以预打包 `.npy` 随 Release 提供……因此无需重抽特征的依赖"。
- MANIFEST.json 列出随包特征体积：train wavlm 1.76GB、whisper 2.2GB、openface3 219MB、
  test 同理——全部供消费而非生成。
- ⇒ 脱离选手随包文件，本仓库**无法**完成"原始数据→特征→训练"的从头流程。

## 2. 非官方特征（规则问题）
| 特征 | 选手所用 | 官方/问题 |
| ---- | -------- | --------- |
| 面部 | OpenFace3 (`openface3_compact`) | 官方为 OpenFace2，非官方流水线 |
| 骨架 | YOLOv11 提取 | 非官方 |
| 音频 | WavLM-Large / Whisper-large-v3 / LCO-Omni-7B | 均非官方音频特征集 |

- `dataset.py:30–32` 定义这些别名；`scripts/train_gated_multifeature_multitask.py:109–110`
  默认 `--binary_audio_features=wavlm,lco,whisper`。模型（门控多特征多任务架构）**强依赖**
  这些非官方特征，换回官方特征需改架构 + 重调参。

## 3. 测试集泄漏 —— 未发现
- `scripts/run_elder_submission_model.py` 的 `build_test_payload()`：测试标签全部初始化为
  dummy 0，只从 `split_labels_test.csv` 取 **ID**，不读标签列。
- 辅助 `prepare_elder_aux_binary_model.py`：CatBoost 仅用 train 标签 `fit`。
- `dump_checkpoint_phq_into_route_cache.py` 名字可疑，但仅用于训练/选模阶段，未被推理主流程
  `run_elder_submission_model.py` 调用。

## 4. 硬编码 / per-id 覆盖 —— 未发现
- 推理为标准前向；辅助 CatBoost 仅对"binary=0 且 ternary>0 的不一致样本"按
  **全局阈值** `aux_threshold` 统一 promote/demote，**非按测试 ID 覆盖**。

## 5. 确定性 / 过拟合风险 —— 中
- `scripts/train_elder_submission_model.py:10–12`：`BINARY_PHQ_EPOCH=43`、`TERNARY_EPOCH=45`、
  `PRESERVE_SHUFFLE_SEQUENCE=True`（钉死 epoch 且注入 shuffle 随机）。
- ⇒ 结果绑定到特定训练轨迹/环境，**重训不易复现同分**。CatBoost 用 `random_seed=3407`
  但 `iterations=2`，跨平台/版本亦不稳定。

## 结论与复现要点
- 该提交本质是**推理包**：吃预打包非官方特征 + 随包权重。按审查原则（禁用选手中间特征与
  预训练权重），**本仓库无法被认定为可从头复现**。
- 若要真正核查：需选手补交**从原始数据起**的全部特征抽取代码，并说明非官方特征是否合规；
  随后用官方原始数据重抽、从随机初始化重训，再上传 Codabench 比对。
- 详见 `repro/JerryYe748/RUNBOOK.md`（含"缺什么、需要选手补交什么"清单）。
