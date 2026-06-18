# MPDD-AVG 2026 复现与审查 (Reproduction & Integrity Review)

本仓库是 **MPDD-bench** 团队对 *MPDD-AVG 2026 挑战赛* 前列选手提交进行
**从头复现 (from-scratch reproduction)** 与 **结果真实性 / 不正当手段审查
(integrity review)** 的工作区。

负责赛道：

| Codabench | 赛道 | 说明 |
| --------- | ---- | ---- |
| [16077](https://www.codabench.org/competitions/16077/) | Track 1 · Elder (老年) · AVG+P | 音频+视频+步态+人格 |
| [16079](https://www.codabench.org/competitions/16079/) | Track 2 · Young (青年) · AVG+P | 音频+视频+步态+人格 |

## 审查原则（核心）

> **一定要从头训练的全流程复现。** 不使用选手提供的中间特征 (.npy)、
> 不加载选手预训练好的权重 (.pth/.cbm)。否则只会拿到选手上传的高分，
> 无法验证结果真实性，给选手钻空子的机会。

一次"有效复现"必须满足：

1. **原始数据起步** —— 从 HuggingFace 官方数据集
   ([`chasonfff/MPDD-AVG-2026`](https://huggingface.co/datasets/chasonfff/MPDD-AVG-2026))
   下载，按官方特征流程**自己重新抽取特征**；
2. **从零训练** —— 用选手的训练脚本与超参，从随机初始化训练，不加载任何
   选手随包附带的权重；
3. **只在最后评测一次测试集** —— 训练 / 选模 / 调阈值阶段绝不接触测试集标签；
4. **核对分数** —— 把自己从头复现得到的提交，与选手在 Codabench
   排行榜上的分数对比，量化差距。

## 被审查的选手仓库

| # | 选手 / 仓库 | 覆盖赛道 | 状态 |
| - | ----------- | -------- | ---- |
| 1 | [HelloWorldLTY/depression_detection](https://github.com/HelloWorldLTY/depression_detection) | Track1 Elder + Track2 Young | 已克隆，代码在 zip 内 |
| 2 | [JerryYe748/MPDD-AVG-2026-Elder-submission](https://github.com/JerryYe748/MPDD-AVG-2026-Elder-submission) | Track1 Elder | 已克隆，权重/特征在 Release |
| 3 | [buptlyx/MPDD-AVG2026](https://github.com/buptlyx/MPDD-AVG2026) | Track1 + Track2（全 baseline 流程） | 已克隆 |
| 4 | [qwe-rty-uop/mpdd](https://github.com/qwe-rty-uop/mpdd) | — | **空仓库**，无可复现内容 |

> 前 5 名的第 5 个仓库链接待群里补充后加入本表。

## 目录结构

```
MPDD_codes/
├── README.md                  # 本文件
├── docs/
│   ├── 00_environment_and_data.md   # 环境搭建 + HF 数据下载（通用）
│   ├── audit_HelloWorldLTY.md       # 选手1 审查报告
│   ├── audit_JerryYe748.md          # 选手2 审查报告
│   ├── audit_buptlyx.md             # 选手3 审查报告
│   └── audit_qwe-rty-uop.md         # 选手4（空仓库）说明
├── repro/
│   ├── HelloWorldLTY/         # 选手1 从头复现 runbook + 脚本
│   ├── JerryYe748/            # 选手2 从头复现 runbook + 脚本
│   └── buptlyx/               # 选手3 从头复现 runbook + 脚本
└── results/                   # 复现产物与分数对比表（本地运行后填写）
```

## 本地运行环境（执行者）

- GPU: RTX 5080 Laptop / 32 GB RAM
- 可访问 HuggingFace（下载官方数据集与特征抽取所需的预训练编码器）
- 按 `docs/00_environment_and_data.md` 配置后，按各 `repro/<选手>/RUNBOOK.md`
  逐步从头复现

## 当前进度

- [x] 克隆全部选手仓库，解出 zip 内代码
- [x] 三个非空仓库静态代码审查（见 `docs/audit_*.md`）
- [ ] 本地从头复现（GPU 机器执行）
- [ ] 分数核对与最终结论
