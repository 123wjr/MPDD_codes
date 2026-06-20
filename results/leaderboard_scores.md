# 排行榜分数核对表

在本地浏览器打开 Codabench 排行榜，把每位选手的**最终官方分数**填入下表，
作为从头复现核对的"真值"。

- Track 1 · Elder · AVG+P：<https://www.codabench.org/competitions/16077/>
- Track 2 · Young · AVG+P：<https://www.codabench.org/competitions/16079/>

> 评分通常综合二分类 / 三分类的 Macro-F1、Accuracy、Kappa 与 PHQ-9 回归的
> CCC/RMSE/MAE（以官方 evaluation 脚本为准）。

## Track 1 · Elder (16077)

| 排名 | 选手 | 排行榜分数 | 从头复现分数 | 差距 | 结论 |
| ---- | ---- | ---------- | ------------ | ---- | ---- |
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

## Track 2 · Young (16079)

| 排名 | 选手 | 排行榜分数 | 从头复现分数 | 差距 | 结论 |
| ---- | ---- | ---------- | ------------ | ---- | ---- |
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

## 已知线索（来自选手 README，待官方核对）

- HelloWorldLTY · Track1 Elder：README 自称提交分数 **0.5881**（来自冻结产物再生成，非从头训练）。
- HelloWorldLTY · Track2 Young：README 自称 base **0.566**，最终 `submission_final_0601`
  经"MPDD-2025 标签泄漏 + leaderboard probing"链条得到（详见审查报告）。

## 已完成的从头复现产物（待传 Codabench 取真值分）

| 选手 | 赛道 | 产物 | 备注 |
| ---- | ---- | ---- | ---- |
| buptlyx | Track1 Elder (16077) | `results/buptlyx/Track1_Elder/submission.zip` | 干净从头复现；23 test id；binary 17×0/6×1，ternary 21×0/1×1/1×2 |
| buptlyx | Track2 Young (16079) | `results/buptlyx/Track2_Young/submission.zip` | 干净从头复现；22 test id；binary 9×0/13×1，ternary 20×0/1×1/1×2；**自抽视频特征非空，不受官方空特征问题影响** |
| HelloWorldLTY | Track1 Elder (16077) | `results/HelloWorldLTY/Track1_Elder/submission_honest_top_r1.zip` | 诚实从头复现（自抽 5 特征 + `phq_threshold`，**不复用冻结 class_dir**）；vs 选手冻结参考 binary/ternary 各翻转 8/23，PHQ \|Δ\| 均值 2.38；诚实三分类**从不命中重度(class2)**。自称 **0.5881** 来自复用冻结分类产物。 |
| HelloWorldLTY | Track2 Young (16079) | `results/HelloWorldLTY/Track2_Young/submission_honest_top_r1.zip` | 诚实从头复现（**仅 Stage A**，5 自抽特征 + `phq_threshold`，**无 Stage B / 无 mpdd_2025 泄漏标签 / 无重组**）；vs 选手冻结参考 0.566211：binary 翻转 12/22、ternary 13/22，PHQ \|Δ\| 均值 3.16；诚实 binary 退化为全正类（备选 `_r2` 阈值 3.5 较均衡）。自称 **0.566211** 来自 Stage-B 重组等被排除的产物。 |

> 上述两个 `submission.zip` 均为从原始数据自抽特征 + 随机初始化训练 + 仅评测一次测试集生成，
> 使用选手自己的代码。把它们分别上传到对应 Codabench 赛道后，将官方分数填入上方对照表的
> "从头复现分数"列即可完成核对。
