# 复现手册 · HelloWorldLTY/depression_detection

审查结论：**自称分数从头不可复现；Track 2 有不正当手段。** 本手册的目的是做**诚实的
从头复现**以量化真实差距 —— 因此**刻意绕开**选手的冻结产物、往届泄漏标签与排行榜试探链路。

## ⛔ 复现红线（务必遵守）
- **禁用**任何冻结提交产物与"再生成"脚本：`reference/*.zip`、`outputs/*.zip`、
  `reproduce_best.py`、`verify_best.sh`、`verify_chain.sh`。它们只是把存好的提交重吐一遍。
- **Track 2 禁用整条 Stage B**：`code_stage_b/*`、`run_stage_b.sh`，以及任何
  `mpdd_2025/.../personalized_train.json`（往届真值泄漏）。这正是作弊来源。
- 只允许从 **Stage A** 的训练代码、用**官方 trainval** 从随机初始化训练。

## 0. 取代码
```bash
git clone https://github.com/HelloWorldLTY/depression_detection.git
cd depression_detection
unzip -q reprod_old.zip   -d reprod_old      # Track1 Elder
unzip -q reprod_young.zip -d reprod_young    # Track2 Young
conda env create -f evo_new.yml || conda create -n evo python=3.10 -y   # 见 README
conda activate evo
pip install numpy scipy scikit-learn torch
```

## 1. 数据
按 `docs/00_environment_and_data.md` 准备官方 trainval/test，并按其 README 把
`MPDD-AVG2026/` 放到包内期望路径。**自己从原始数据重抽特征**，勿用选手随包的中间特征。

## 2. Track 1 (Elder) —— 只跑 Stage A 从头训练
```bash
cd reprod_old
PY=<你的python> ./stage_a/run_stage_a.sh        # 训练 3 个任务头并重组
# 产物在 stage_a/_work/...，取其中的 binary.csv / ternary.csv 组成提交
```
> 预期：分类标签与参考接近，但 PHQ 因版本漂移 ±0.2，阈值边界样本(如 id51)可能翻转；
> **从头分数约 0.56–0.57，低于自称 0.5881**。把 Stage A 产出（不要喂回 Path A）上传比对。

## 3. Track 2 (Young) —— 只跑 Stage A 从头训练
```bash
cd reprod_young
# 严格/原始路径：训练三个 A-V-G+P 任务头再重组（不接触 Stage B / 泄漏）
PY=<你的python> ./code_stage_a/run_stage_a.sh
# 产物 base_recombination/Track2/{binary,ternary}.csv,submission.zip
```
> 预期：从头得到的 base ≈0.566（PHQ |Δ|≈0.67、binary 约 14/22 与其参考不同）。
> **严禁**继续 Stage B。最终诚实分数 ≈0.566，与自称 ≈0.62 差 0.05+，差额正是泄漏+试探所得。

## 4. 核对与记录
- 把 Stage A 从头产出的 zip 传 Codabench，记录分数到 `results/leaderboard_scores.md`，
  与"自称分数"对比，量化"靠不正当手段多拿了多少分"。
- 证据日志（哪些脚本读了 `mpdd_2025` 标签、哪些 docstring 写了 leaderboard 试探）拷到
  `results/HelloWorldLTY/`，作为组委会判定材料。
