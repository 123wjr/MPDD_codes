# 审查报告 · HelloWorldLTY/depression_detection

- 仓库：<https://github.com/HelloWorldLTY/depression_detection>
- 覆盖：Track 1 (Elder, AVG+P) + Track 2 (Young, AVG+P)
- 代码位置：随包 `reprod_old.zip` / `reprod_young.zip`（已解压审查）
- 审查日期：2026-06-18

## 总判定

**从头不可复现自称分数；Track 2 存在明确不正当手段。**

| 赛道 | 自称分数 | 从头复现可达 | 判定 |
| ---- | -------- | ------------ | ---- |
| Track 1 Elder | 0.5881 | ~0.56–0.57 | ⚠️ 临界：成绩依赖冻结产物再阈值化，非从头训练；无泄漏 |
| Track 2 Young | ~0.62 | ~0.566 | 🔴 不正当：往届标签泄漏 + 排行榜试探 + 手工调参 |

---

## Track 1 (Elder)

### 流水线
两阶段：Stage A（训练 sklearn/torch PHQ 栈，产出冻结参考 `reference/submission_elderly0529aemulti.zip` ≈0.5714）→ Stage B（对参考的 PHQ 按 2.5/5.0 阈值重切三分类，得 `outputs/submission_tv02_thr_25_50_best.zip` ≈0.5881）。

### 证据
- `reproduce_best.py`：读取冻结参考 zip 的 `binary_pred`/`phq9_pred`（**逐字节复制不变**），仅按 PHQ 阈值重导出 `ternary_pred`。README 自述此路径"只需 Python 标准库 + unzip，无需 GPU、无需 ML 库，约 1 秒"——**即重新吐出已存好的提交，而非训练**。
- `verify_best.sh`：校验再生成结果与"标准最佳"**逐字节一致**。
- `stage_a/STAGE_A_TEST.md`：真正 Stage A 从头重训**无法字节级复现**——分类标签一致，但 PHQ 平均 |Δ|≈0.195（id92 最大 0.633），原因是 numpy/sklearn/torch 及集群版本漂移。漂移导致阈值边界样本 **id51** 翻转（PHQ 2.4765→0 变 2.7890→1），三分类分布从 `{0:18,1:2,2:3}` 变为 `{0:17,1:3,2:3}`。

### 结论
方法透明、无标签泄漏；但**自称 0.5881 依赖冻结参考产物**，从随机初始化 + 当前环境从头训练只能到约 0.56–0.57。是否接受取决于赛规是否允许"提交冻结产物"。

---

## Track 2 (Young) —— 不正当手段（取证）

### 流水线
Stage A（3 个 sklearn 任务头重组，得 base `reference/0566211_submission.zip` ≈0.566，本身干净、不读测试标签）→ **Stage B 六轮链条**（`code_stage_b/`，逐步把 ≈0.566 抬到 ≈0.62）。问题全在 Stage B。

### 不正当手段 #1：往届(MPDD-2025)标签泄漏
22 个测试 ID 中 18 个的真实标签来自往届数据集，被硬编码并用于候选打分排序。

- `code_stage_b/build_t2_ens30_0530.py:44`
  ```python
  LEAK = [1, 7, 13, 28, 33, 34, 40, 42, 44, 58, 74, 83, 85, 89, 90, 93, 105, 110]
  ```
- 同文件 ~48 行加载往届真值：
  ```python
  _gt = json.load(open(ROOT / "mpdd_2025/MPDD-2025/MPDD-Young/Training/labels/personalized_train.json"))
  GTB = np.array([_gt[str(i)]["binary_depression"] for i in LEAK])
  GTT = np.array([_gt[str(i)]["tri_depression"] for i in LEAK])
  GTP = np.array([float(_gt[str(i)]["PHQ_9"]) for i in LEAK])
  ```
- 同文件 ~210–236 行：对 30 个候选集成用**泄漏标签**算 F1/CCC 并据此排序选最优。
- 相同模式在每一轮重复：`build_t2_ens30b_0530.py`、`build_t2_ens20c_0530.py`、`build_t2_ens_0601.py`、`build_t2_ens_round5_0601.py`、`build_t2_final_0601.py`。

### 不正当手段 #2：排行榜试探 (leaderboard probing)
未在往届出现的 4 个 ID `{5,15,22,47}` 通过反复提交看分数反推真值。

- `build_t2_ens_0601.py` docstring：
  > "CodaBench-confirmed facts now: ref=0.566, ens17=0.59, ens92=0.6233(BEST), ens95(id47→mild)=0.5836(FAIL ⇒ id47 is N), ens102(id5 phq 2.26→2.0)=0.6220(⇒ don't disturb)… all unseen CLASSES resolved: id5=N,id15=M,id22=N,id47=N."
- `build_t2_ens_round5_0601.py` docstring：用泄漏-18 统计建未知 ID 的类别先验，并按已测得的排行榜分数"校准"。

### 不正当手段 #3：手工硬调测试预测
- `build_t2_final_0601.py:74–92`：对 id47 的 PHQ 在 `{7.22,6.0,5.0,4.0,3.5,3.0,2.5,2.0}` 上做一维 maximin 扫描挑值。

### 选手自认
- `code_stage_b/run_stage_b.sh` 注释 + young README：自述"uses the MPDD-2025 label leak + leaderboard-derived decisions"、Stage B"不是泛化模型，只是用泄漏标签编辑少数测试单元格"、"可迁移的科学只有 Stage A"。

### 结论
**Stage B 完全无法从头复现**（需往届标签 + 反复排行榜提交 + 手调）。去掉泄漏后从头只能到 ≈0.566，与自称 ≈0.62 差 0.05+。**建议取消该赛道资格。**

---

## 证据复核与可靠性评级（2026-06-20 二次核验）

下列每条 Track 2 不正当手段均**直接来自选手自己提交的源码与注释**（已于 2026-06-20
在 `reprod_young/code_stage_b/` 逐行重新核对，行号与内容一字不差），不依赖本团队的
复现结果或任何建模判断，因此**事实层面可靠性极高（自证型证据）**。

| 结论 | 证据（选手本人文件） | 可靠性 |
| ---- | -------------------- | ------ |
| 往届(MPDD-2025)标签泄漏 | `LEAK = [1, 7, …, 110]` 硬编码于 **6 个** stage-B 脚本（`build_t2_ens_0601.py:44`、`build_t2_ens_round5_0601.py:39`、`build_t2_final_0601.py:31`、`build_t2_ens30_0530.py:44`、`build_t2_ens20c_0530.py:38`、`build_t2_ens30b_0530.py:36`），各脚本均加载 `mpdd_2025/.../personalized_train.json` 真值并据此给候选打分排序 | **极高（逐字自证）** |
| 排行榜试探 | `build_t2_final_0601.py:4-5` 注释："CodaBench-confirmed facts now: ref=0.566, ens17=0.59, ens92=0.6233(BEST), ens95(id47→mild)=0.5836(FAIL ⇒ id47 is N)…"，由提交分数反推未出现的 4 个 ID `{5,15,22,47}`；`build_t2_ens_round5_0601.py:52` "22: …CodaBench-confirmed" | **极高（明示）** |
| 手工硬调测试预测 | `build_t2_final_0601.py` 对 id47 的 PHQ 做一维扫描挑值；同文件 `:114` "binary & ternary are LEFT EXACTLY as the proven ens92 (0.6233)" | **极高** |
| 选手自认 | `run_stage_b.sh:2-3` "Stage B: ensemble / leak-exploitation / leaderboard-probing chain that lifts the 0.566 base to the 0.6233 best"；README "the only transferable science is Stage A" | **极高** |

### 仍未达到"官方核实"的部分（如实声明）
1. **具体分数尚未在 Codabench 实测**：自称 ≈0.62/0.6233 来自选手注释中对排行榜的自述读数，
   0.566211 为其冻结 base 产物；本环境无法提交，故 `leaderboard_scores.md` 的"真值分"列仍为空。
   "差距 ≈0.05+"在方向上有力支撑，但尚非官方实测值。
2. **本团队诚实 Stage-A 复现自身有局限**（见 `results/HelloWorldLTY/Track2_Young/REPRO_NOTES.md`）：
   步态(G)与人格(P)沿用官方数组未重抽；诚实二分类退化为全正类。故"≈0.566"是善意估计而非竞赛级分数。
3. **"建议取消资格"为判断性结论**：泄漏 + 试探的**事实**铁证如山，但是否构成取消资格取决于赛规
   （本团队尚未通读赛规），属于在可靠事实之上的编辑性建议。

### 一句话
> Track 2 的**事实核心**——头部分数由"往届标签泄漏 + 排行榜试探 + 手工编辑"取得——**可靠**，
> 由选手自己的源码证明；**具体分数差距与取消资格建议**为有据推断，尚待一次真实 Codabench 提交
> 与赛规对照后才能正式定案。

---

## 复现要点（详见 `repro/HelloWorldLTY/RUNBOOK.md`）
- **只跑 Stage A**（从头训练），**禁用** `reference/`、`outputs/` 内任何冻结 zip 与 `reproduce_best.py`/`verify_*.sh`。
- **Track 2 严禁**任何 `code_stage_b/*` 脚本与 `mpdd_2025/` 标签文件。
- 把 Stage A 从头产出的提交上传 Codabench，与自称分数对比、记录差距。
