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
