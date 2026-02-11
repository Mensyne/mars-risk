# 🚀 MARS: High-Performance Risk Modeling Framework

**MARS** (Modeling Analysis Risk Score) 是一个面向信贷风控建模场景的 Python 工具库。它基于 **Polars** 构建数据处理逻辑，并遵循 **Scikit-learn** 的 API 设计规范，旨在为信贷风控大规模宽表场景下的数据画像、特征工程与模型评估提供更高效的解决方案。 

> **核心目标**：通过 Polars 的向量化执行提升数据处理效率，同时保持与 Scikit-learn 流水线（Pipeline）的兼容性。

## ✨ 核心特性 (Key Features)

### 1. 📊 高性能数据画像 (Data Profiling)
提供数据质量诊断与可视化报告，性能比传统 Pandas 方案快数倍。
* **全量指标概览**: 一次性计算 Missing, Zero, Unique, Top1 等基础 DQ 指标。
* **Unicode Sparklines**: 在终端或 Notebook 中直接生成迷你分布图 (如 ` ▂▅▇█`)，快速洞察数据分布。
* **多维趋势分析**: 支持按时间 (Month/Vintage) 或客群进行分组分析，自动计算初步的稳定性指标 (Var, CV)。
* **Excel 自动化报告**: 导出带有热力图、数据条和条件格式的精美 Excel 报表。

### 2. 🧮 快速分箱引擎 (High-Performance Binning)
针对风控评分卡场景深度优化的分箱器。
* **MarsNativeBinner**: 基于 Polars 表达式实现的快速分箱。
    * 支持 **Quantile** (等频), **Uniform** (等宽), **CART** (决策树) 三种模式。
    * **并行加速**: 决策树分箱利用 `joblib` 实现多核并行，内存占用低。
* **MarsOptimalBinner**: 混合动力最优分箱。
    * **Hybrid Engine**: 结合 Polars 的快速预分箱 (O(N)) 与 `optbinning` 的数学规划 (MIP/CP) 求解 (O(1))。
    * 支持**单调性约束** (Monotonic Trend) 和**特殊值/缺失值**的独立分层处理。

### 3. 🛠️ 工程化设计
* **Auto Polars**: 智能装饰器支持 Pandas DataFrame 无缝输入，内部自动转换为 Polars 计算，结果按需回退。
* **Pipeline Ready**: 所有组件均继承自 `MarsBaseEstimator` 和 `MarsTransformer`，兼容 Sklearn Pipeline。

---

## 📦 安装 (Installation)

```python
# 推荐使用 pip 安装
pip install mars-risk

# 或者从源码安装
git clone [https://github.com/leeesq/mars-risk.git](https://github.com/leeesq/mars-risk.git)
cd mars-risk
pip install -e .
```

依赖项: `polars`, `pandas`, `numpy`, `scikit-learn`, `scipy`, `xlsxwriter`, `colorlog`, `optbinning`

## ⚡️ 快速上手 (Quick Start)

### 场景 1：生成数据画像报告

```python

import polars as pl
from mars.analysis.profiler import MarsDataProfiler

# 1. 加载数据
df = pl.read_csv("your_data.csv")

# 2. 初始化分析器 (支持自定义缺失值，如 -999)
profiler = MarsDataProfiler(df, custom_missing_values=[-999, "unknown"])

# 3. 生成画像报告
report = profiler.generate_profile(
    profile_by="month",  # 可选：按月份分组分析趋势
    config_overrides={"enable_sparkline": True} # 开启迷你分布图
)

# 4. 展示与导出
report.show_overview()  # 在 Jupyter 中查看概览 (含热力图)
report.show_trend("mean") # 查看均值趋势
report.write_excel("data_profile_report.xlsx") # 导出为 Excel
```

### 场景 2：快速特征分箱
```python
from mars.feature.binner import MarsNativeBinner, MarsOptimalBinner

# --- 方式 A: 快速原生分箱 (适合大规模预处理) ---
binner = MarsNativeBinner(
    features=["age", "income"],
    method="quantile",  # 等频分箱
    n_bins=10,
    special_values=[-1] # 特殊值独立成箱
)
binner.fit(X_train, y_train)
X_train_binned = binner.transform(X_train)

# --- 方式 B: 最优分箱 (适合评分卡精细建模) ---
opt_binner = MarsOptimalBinner(
    features=["credit_score"],
    n_bins=5,
    solver="cp", # 使用约束编程求解
    monotonic_trend="ascending" # 强制单调递增
)
opt_binner.fit(X_train, y_train)
print(opt_binner.bin_cuts_) # 查看最优切点
```

## 📂 项目结构 (Project Structure)
```Plaintext
src/mars/
├── analysis/             # 数据分析与画像模块（已实现）
│   ├── config.py         # 分析配置类（已实现）
│   ├── evaluator.py      # 评估器（已实现）
│   ├── profiler.py       # MarsDataProfiler 核心逻辑（已实现）
│   └── report.py         # MarsProfileReport 报告容器（已实现）
├── core/                 # 核心基类与异常（已实现）
│   ├── base.py           # 兼容 Sklearn 的基类（已实现）
│   └── exceptions.py     # 自定义异常（已实现）
├── feature/              # 特征工程模块（已实现）
│   ├── binner.py         # Native/Optimal 分箱实现（已实现）
│   └── selector.py       # 特征筛选模块（已实现）
├── metrics/              # 指标计算模块（已实现）
│   └── calculation.py    # 指标计算函数（已实现）
├── modeling/             # 建模流水线模块（已实现）
│   ├── base.py           # 建模基础抽象（已实现）
│   ├── strategies.py     # 建模策略实现（已实现）
│   └── tuner.py          # 调参模块（已实现）
├── risk/                 # 风险策略模块（计划中，当前仅保留包结构）
├── scoring/              # 评分量化模块（计划中，当前仅保留包结构）
└── utils/                # 工具库（已实现）
    ├── date.py           # 日期处理工具（已实现）
    ├── decorators.py     # 装饰器工具（已实现）
    ├── logger.py         # 全局日志配置（已实现）
    └── plotter.py        # 绘图工具（已实现）
```

## 📄 许可证 (License)
MIT License
