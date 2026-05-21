# VisualPortfolio 新人上手指南

## 1. 项目定位
VisualPortfolio 是一个**量化策略/资产组合表现可视化**工具，参考了 pyfolio 的指标与图表组织方式。典型入口是 `createPerformanceTearSheet`：输入价格序列或收益序列，输出绩效指标、逐日数据表、滚动风险指标，并可直接绘图。

## 2. 代码库整体结构

```text
VisualPortfolio/
├─ __init__.py          # 对外 API 导出（最常用入口）
├─ Tears.py             # tear sheet 组装层（流程编排 + 作图调用）
├─ Timeseries.py        # 指标与时序计算（收益聚合、回撤、滚动 beta/sharpe 等）
├─ Plottings.py         # 图表函数与绘图上下文
├─ Transactions.py      # 换手率计算
├─ Miscellaneous.py     # 组合分析辅助入口
└─ Env.py               # 数据源枚举与全局设置

requirements/
├─ py2.txt
└─ py3.txt

notebooks/
└─ Overview of  VisualPortfolio.ipynb  # 示例笔记本
```

## 3. 你需要优先理解的关键模块

### 3.1 `Tears.py`：核心业务入口
- `createPerformanceTearSheet` 是最关键函数：
  - 输入：`prices` 或 `returns`（二选一），可选基准 `benchmark`。
  - 逻辑：统一转对数收益 -> 聚合 -> 计算回撤与绩效指标 -> 生成可视化。
  - 输出：`perf_metric`、`perf_df`、`rollingRisk`。
- 同文件还包含 position / transaction 的 tear sheet（仓位暴露、持仓结构、换手等）。

### 3.2 `Timeseries.py`：计算层
- 负责“算什么”：
  - `aggregateReturns`：日/月/年收益聚合。
  - `drawDown`：回撤序列与峰/谷/恢复点。
  - `annualReturn` / `annualVolatility` / `sortinoRatio` / `sharpRatio`。
  - `RollingBeta` / `RollingSharp`：滚动风险指标。
- 大部分上层模块依赖这里，调试问题时通常要先看这层输入输出是否正确。

### 3.3 `Plottings.py`：展示层
- 负责“怎么画”：累计收益、回撤区间、月度热力图、年收益分布、rolling beta/sharpe 等。
- `plotting_context` 装饰器统一 seaborn/matplotlib 上下文，保证图风一致。

### 3.4 `Env.py` + `Miscellaneous.py`：数据接入与外部集成
- `Env.py` 通过 `Settings.data_source` 切换数据源（DataYes / DXDataCenter）。
- `Miscellaneous.py` 里的 `portfolioAnalysis` 演示从持仓出发拉行情再调用 tear sheet 的完整流程。

## 4. 对新人最重要的“理解顺序”

建议按下面顺序阅读，效率最高：
1. `README.rst`：先理解功能边界与最小使用样例。
2. `VisualPortfolio/__init__.py`：识别公开 API。
3. `VisualPortfolio/Tears.py`：掌握主流程（输入 -> 计算 -> 输出 -> 绘图）。
4. `VisualPortfolio/Timeseries.py`：逐个对照核心指标计算。
5. `VisualPortfolio/Plottings.py`：理解每张图和对应字段关系。
6. `notebooks/Overview of  VisualPortfolio.ipynb`：从示例把数据流跑通。

## 5. 开发与维护时要注意的点

- 项目中大量使用**对数收益**并在展示时再转累计收益，新增指标时要统一口径。
- 仓位/交易相关函数中存在早期 pandas 写法，升级依赖前应先补兼容测试。
- `PyFin` 与数据源 API 是关键外部依赖，建议先确认本地环境可用再做功能改造。
- 代码中部分命名有历史拼写（如 `createPostionTearSheet`, `Transcation`），对外兼容时不要随意重命名。

## 6. 后续学习建议（按 2~4 周节奏）

### 第一阶段（1 周）
- 目标：能独立跑通示例并解释每个输出字段。
- 任务：
  - 跑 `README` 示例。
  - 对 `perf_metric` 每个指标写出“定义 + 在代码中的计算位置”。

### 第二阶段（第 2 周）
- 目标：能定位常见数据问题。
- 任务：
  - 造几组异常输入（缺失日期、跳空、基准缺失）并观察输出。
  - 跟踪 `aggregateReturns -> drawDown -> plotting` 的链路日志。

### 第三阶段（第 3~4 周）
- 目标：可独立做小改动并回归验证。
- 任务：
  - 新增一个简单指标（如 Calmar Ratio）并在 `perf_metric` 暴露。
  - 在图表层新增一个对应可视化或注释。
  - 补最小单元测试/示例 notebook 结果截图，保证不破坏现有 API。

---

如果你是 mentor，可以直接把这份文档作为 onboarding checklist：
- [ ] 跑通示例
- [ ] 解释 7 个核心绩效指标
- [ ] 解释 4 张核心图（累计收益、回撤、月热力、年收益）
- [ ] 独立完成 1 个小指标改造并演示
