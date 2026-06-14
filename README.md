# worldcup-forecast

**简体中文** | [English](README-EN.md)

一个面向 FIFA 世界杯的结构化 + 贝叶斯预测器；同样重要的是，它也提供了一份
**诚实、经过严格验证**的说明：到底哪些方法真的能改进国家队比赛预测。

![python](https://img.shields.io/badge/python-3.10%2B-blue)
![license](https://img.shields.io/badge/license-MIT-green)
![tests](https://img.shields.io/badge/tests-passing-brightgreen)

大多数世界杯模型会发布一份冠军榜和一个看起来很漂亮的准确率数字。这个项目则提供了
**滚动向前验证框架**和一份**发现台账**，用显著性检验说明哪些想法有效、哪些无效，
包括那些不太令人舒服的结论。核心结论本身也保持诚实：经过穷尽式搜索后，模型已经位于
*结构效率前沿*，而博彩市场非常难以击败。

---

## 它能做什么

- **球队强度**：基于冻结的赛前 FIFA 快照、Transfermarkt 阵容身价、无泄漏 Elo，
  并用 Klement 风格的结构先验包裹起来（GDP、人口、气候、主场优势、足球文化）。
- **比赛模型**：带部分池化的分层贝叶斯 Poisson 模型（PyMC）——数据丰富的球队更多由
  比赛结果牵引，数据稀缺的球队则收缩到结构先验。
- **赛事模拟**：对官方 2026 赛制进行 Monte-Carlo 模拟（12 个小组、8 个成绩最好的
  小组第三、73-104 淘汰赛映射），输出冠军和阶段概率。
- **验证**：样本外滚动向前评分（log-loss / RPS / Brier），并使用配对 bootstrap
  显著性检验；从不只用少量比赛来判断模型。
- **市场基准**：Polymarket 和传统博彩公司赔率仅用于比较。根据设计中的独立性原则，
  **市场数据绝不会作为模型输入**。

## 快速开始

```bash
git clone <repo-url> && cd worldcup-forecast
pip install -e .                 # 或者：make install

wcforecast forecast              # 2026 冠军概率（双轨）
wcforecast predict Brazil Morocco
wcforecast validate              # 带显著性检验的样本外评分表
ODDS_API_KEY=xxxx wcforecast odds   # 实时博彩公司共识（可选，需要免费 key）
```

需要 Python >= 3.10。首次运行会自动下载公开比赛和排名数据集；小型的冻结 2026 输入
已随项目放在 `data/snapshots/` 中。

## 示例输出

```
2026 World Cup — champion probability (Monte Carlo)

  team                      accuracy %  independent %
  France                          13.8           13.3
  Spain                           12.9           11.4
  England                         11.4           11.3
  Argentina                        8.6            9.9
  United States                    6.7            7.7
  ...
```

项目会并排报告两条轨道：
- **accuracy**：完整模型（FIFA + 阵容身价锚点 + 比赛数据）；
- **independent**：仅结构化（“Klement”）先验，不含市场数据，且可解释。

二者之间的差异是*需要解释的特征*，不是需要隐藏的错误。

## 诚实发现：这个项目最独特的部分

每个候选想法都在一个**锁定的 305 场比赛窗口（2024-2026）**上做过样本外测试，
并使用配对 bootstrap 检验显著性。完整台账见：
[`docs/FINDINGS.md`](docs/FINDINGS.md)。

| 想法 | 结论 |
|---|---|
| 无泄漏 Elo（K=40）+ 结构先验 | ✅ 稳健基线 |
| 温度校准 + 中立场平局提升 | ✅ 小幅但显著 |
| 模型平均（Bayesian + Elo-logit） | ✅ 降低方差 |
| 阵容身价作为强度锚点 | ✅ 有助于 2026 预测（在强弱悬殊比赛上，与敏锐市场的距离约缩小 30%） |
| FIFA vs Elo 锚点 | ➖ 平手（FIFA 的价值在于数据质量，而不是额外信号） |
| 额外特征（状态、休息、洲际足联） | ➖ 没有显著收益 |
| 历史交手记录 | ❌ 控制强度后几乎为零 |
| Dixon-Coles 低比分修正 | ❌ 在这份数据上 rho 约为 0 |
| 梯度提升（LightGBM）集成与 isotonic 校准 | ❌ 对稀疏数据过拟合，显著更差 |

**一句话结论：** 简单、低方差的改动有帮助；灵活、复杂的方法容易在国家队数据上过拟合。
模型已经到达结构前沿——单场世界杯比赛结果大约有一半来自运气，而市场很难击败。
这个结论本身就是产品的一部分。

## 工作原理

完整方法论和设计取舍见 [`docs/DESIGN.md`](docs/DESIGN.md)
（为什么使用部分池化、为什么市场只作为基准、如何避免数据泄漏）。

## 项目结构

```
worldcup-forecast/
├── src/wcforecast/
│   ├── teams.py        # 48 支球队、官方抽签、洲际足联
│   ├── data.py         # 无泄漏加载器（martj42、FIFA、阵容、World Bank）
│   ├── ratings.py      # Elo + 结构化（Klement）先验指数
│   ├── model.py        # 分层贝叶斯 Poisson（PyMC）
│   ├── predict.py      # 比分网格 1X2 + 校准
│   ├── simulate.py     # 官方赛程结构 + Monte Carlo 冠军赔率
│   ├── validate.py     # 滚动向前样本外框架 + 指标 + 显著性
│   ├── markets.py      # Polymarket + 博彩公司赔率（仅作基准）
│   └── cli.py          # `wcforecast` 命令行接口
├── tests/              # 快速、离线的单元测试
├── docs/               # DESIGN.md（方法论）+ FINDINGS.md（已验证台账）
└── data/               # 冻结的 2026 快照（已提交）+ 缓存（git 忽略）
```

## 作为库使用

```python
from wcforecast import data, ratings, model, simulate, predict

results = data.load_results()
s = ratings.structural_index("2026-06-11", results,
                             fifa=data.load_fifa_snapshot(),
                             squad=data.load_squad_values())
matches = data.load_matches(start="2006-01-01", cutoff="2026-06-11")
m = model.fit(matches, s, weights=ratings.recency_weights(matches, "2026-06-11"),
              dixon_coles=True)

simulate.champion_probabilities(m, s, n_sims=20000).head()
predict.calibrate(m.match_probs("Brazil", "Morocco", home_advantage=0.0))
```

## 设计原则：独立于市场

模型从不读取博彩赔率（无论是作为特征、校准目标，还是集成成员）。它的目的在于提供一个
*独立、可解释的结构化信号*；市场只用于基准比较。击败市场**不是**目标——验证结果也确认，
这件事本来就非常困难。

## 局限

- 单场比赛预测的上限较低（很多结果来自运气）；详见 FINDINGS。
- 阵容身价和 FIFA 快照都是单一的 2026 快照——用于向前预测，而不是历史回测
  （因为没有可用历史）。
- 免费博彩公司赔率层级只覆盖实时赔率，因此用于*历史*验证的市场基准只能比较结果，
  不能比较历史价格。

## 许可证

MIT — 见 [`LICENSE`](LICENSE)。

## 致谢

受 Joachim Klement 的结构化世界杯模型启发。数据来源：martj42/international_results、
Dato-Futbol/fifa-ranking、FIFA、Transfermarkt、World Bank、Polymarket、The Odds API。
