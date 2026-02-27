# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

CZSC（缠中说禅技术分析工具）是基于缠中说禅理论的综合性量化交易Python库，提供技术分析、信号生成、回测和市场分析等功能。本项目专注于实现缠论的分型、笔、线段等核心概念的自动识别，以及基于此的多级别量化交易策略。

**重要版本说明**：0.10.X 版本开始，核心功能逐步使用 rs-czsc（Rust 实现）替换 Python 实现，性能显著提升。有需要了解对象或函数具体执行逻辑的，可以查看 [0.9.X](https://github.com/waditu/czsc/tree/v0.9.69) 版本。

## 常用开发命令

### UV 包管理 (项目使用UV管理依赖)
```bash
# 同步依赖并安装开发工具
uv sync --extra dev

# 安装所有依赖组合
uv sync --extra all

# 运行测试（注意：项目使用 test/ 目录，不是 tests/）
uv run pytest

# 运行指定测试文件
uv run pytest test/test_analyze.py -v

# 运行单个测试函数
uv run pytest test/test_analyze.py::test_czsc_basic -v

# 带覆盖率的测试
uv run pytest --cov=czsc

# 代码格式化和检查
uv run black czsc/ test/ --line-length 120
uv run isort czsc/ test/
uv run flake8 czsc/ test/

# 构建 Sphinx 文档（在 docs/ 目录下）
cd docs && ./make.bat html  # Windows
cd docs && make html        # Linux/Mac

# 类型检查
uv run mypy czsc/ --ignore-missing-imports

# 安全检查
uv add --dev bandit[toml]
uv run bandit -r czsc/
```

### 测试规范
- 所有测试文件位于 `test/` 目录，使用 pytest 格式
- **关键原则**：测试数据统一通过 `czsc.mock` 模块获取，不要在测试中硬编码模拟数据
- 测试文件命名模式：`test_*.py`
- 模拟数据使用 `generate_symbol_kines` 函数生成，支持多品种、多频率、可重现的随机数据
- CI 在 Python 3.10, 3.11, 3.12, 3.13 版本上运行测试


## 代码架构

### 核心组件

1. **`czsc/core.py`** - 混合架构核心模块，智能选择 Rust/Python 实现：
   - Rust 版本优先（rs-czsc），性能优化
   - Python 版本作为回退方案
   - 导入核心类：`CZSC`、`RawBar`、`NewBar`、`Signal`、`Event`、`Position` 等

2. **`czsc/py/`** - Python 实现的核心算法：
   - `analyze.py`: 缠论分析核心类，实现分型、笔的自动识别
   - `objects.py`: 核心数据结构定义
   - `bar_generator.py`: K线数据生成和重采样

3. **`czsc/traders/`** - 交易执行框架：
   - `base.py`: CzscSignals 和 CzscTrader 核心类
   - `cwc.py`: 权重交易客户端
   - `rwc.py`: Redis权重管理客户端
   - `dummy.py`: 模拟回测框架
   - `optimize.py`: 开仓平仓参数优化

4. **`czsc/signals/`** - 按类别组织的信号生成函数：
   - `bar.py`: K线级别信号
   - `pos.py`: 持仓相关信号  
   - `cxt.py`: 上下文信号
   - `tas.py`: 技术指标信号
   - `vol.py`: 成交量信号
   - 每个信号模块包含专业的技术分析函数

5. **`czsc/sensors/`** - 事件检测和特征分析：
   - `cta.py`: CTA研究框架
   - `feature.py`: 特征选择器和分析器
   - `event.py`: 事件匹配和检测

6. **`czsc/utils/`** - 工具模块：
   - `bar_generator.py`: K线数据生成和重采样
   - `cache.py`: 磁盘缓存工具
   - `st_components.py`: Streamlit仪表板组件
   - `ta.py`: 技术分析指标
   - `data/`: 数据工具子模块
     - `client.py`: 统一数据客户端接口
     - `cache.py`: 磁盘缓存实现
     - `converters.py`: 数据格式转换器
     - `validators.py`: 数据验证器
   - `plotting/`: **可视化工具子模块（重构优化）**
     - `backtest.py`: 回测图表（累计收益、回撤、月度热力图等）
     - `weight.py`: 权重分析图表（换手率、权重分布等）
     - `kline.py`: K线图表类 `KlineChart`
     - `common.py`: 通用常量和辅助函数
   - `analysis/`: **分析工具子模块（重构优化）**
     - `stats.py`: 统计分析（日收益表现、回撤分析、PSI等）
     - `corr.py`: 相关性分析（NMI矩阵、截面IC等）
     - `events.py`: 事件分析（重叠检测等）
   - `pdf_report_builder.py`: PDF 报告构建器（支持中文、图表嵌入、目录）
   - `html_report_builder.py`: HTML 报告构建器
   - `backtest_report.py`: 回测报告生成工具

7. **`czsc/svc/`** - 统计和可视化服务：
   - `backtest.py`: 回测分析工具
   - `factor.py`: 因子分析
   - `correlation.py`: 相关性分析
   - `statistics.py`: 统计分析工具
   - `forms.py`: Streamlit 表单组件

8. **`czsc/features/`** - 特征计算模块：
   - `ret.py`: 未来收益特征（仅用于研究/回测，含未来信息）
   - `vpf.py`: 价格波动特征
   - `utils.py`: 特征工程工具

9. **`czsc/fsa/`** - 飞书服务应用模块：
   - `base.py`: 飞书 API 封装
   - `bi_table.py`: 飞书多维表格接口
   - `im.py`: 飞书即时消息接口
   - `spreed_sheets.py`: 飞书电子表格接口

10. **`czsc/connectors/`** - 数据源连接器：
   - 支持天勤、Tushare、聚宽、CCXT等多个数据源
   - 统一的数据接口封装

### 信号-事件-交易体系

项目实现了系统化的量化交易方法：
- **信号（Signals）**: 基础技术指标和市场状态
- **事件（Events）**: 信号的逻辑组合，通过 signals_all/signals_any/signals_not 实现 AND/OR/NOT 逻辑
- **交易（Trading）**: 基于事件和风险管理的执行

### 多级别联立分析

CZSC 支持使用 `CzscTrader` 类进行多级别联立分析，可同时分析不同时间周期（如1分钟、5分钟、30分钟、日线）进行全面的市场决策。

## 开发指南

### 代码规范
- 行长度：120字符（在 pyproject.toml 中配置）
- 适当使用类型提示
- 遵循代码库中现有的命名约定
- 信号函数版本化命名（如 `V241013`）便于管理
- **代码质量原则**：
  - **DRY（Don't Repeat Yourself）**: 提取重复代码为辅助函数
  - **KISS（Keep It Simple）**: 保持函数简洁，职责单一
  - **使用模块级常量**: 避免魔法值，集中管理配置
  - **类型提示优先**: 使用 `Literal`、`Optional` 等提升代码可读性
  - **向后兼容性**: 公共 API 修改需谨慎，避免破坏现有代码
  - **文档完整**: 所有公共函数必须有完整的 docstring

### CI/CD 检查
项目在 GitHub Actions 中运行以下检查（推送和 PR 时自动触发）：
- **测试**: Python 3.10-3.13 多版本测试 + 覆盖率报告
- **代码检查**: flake8 语法检查 + mypy 类型检查（mypy 仅作提示，不会阻断 CI）
- **安全审计**: safety 依赖漏洞扫描 + bandit 代码安全检查
- **依赖分析**: 过期依赖检查 + 许可证审计

### 信号函数开发
- 信号函数应遵循飞书文档中的规范说明
- 所有信号函数必须经过适当测试
- 使用 `czsc/signals/` 中现有的信号模板作为参考
- 按类别组织信号函数（bar、pos、cxt、tas、vol等）

### 数据处理最佳实践
- 测试数据统一通过 `czsc.mock.generate_symbol_kines` 生成
- 使用 `format_standard_kline` 将DataFrame转换为RawBar对象列表
- 使用 `BarGenerator` 进行K线合成和多级别分析
- 通过 `czsc.utils.data.client.DataClient` 统一访问不同数据源
- 注意使用磁盘缓存提高重复计算效率

**数据客户端使用：**
```python
from czsc.utils.data import DataClient

# 统一接口访问不同数据源
client = DataClient(source="ts")  # tushare
# client = DataClient(source="jq")  # 聚宽
# client = DataClient(source="ccxt")  # 数字货币
df = client.get_kline(symbol="000001", freq="日线", start="20200101")
```

### 数据格式转换
```python
# 从mock数据生成CZSC对象的正确模式
from czsc.core import CZSC, format_standard_kline, Freq
from czsc.mock import generate_symbol_kines

# 生成K线数据
df = generate_symbol_kines('000001', '30分钟', '20240101', '20240105')

# 转换为RawBar对象列表
bars = format_standard_kline(df, freq=Freq.F30)

# 创建CZSC分析对象
czsc_obj = CZSC(bars)
```

### 回测可视化最佳实践
```python
# 推荐使用新的 plotting 模块（统一接口）
from czsc.utils.plotting import (
    plot_cumulative_returns,
    plot_backtest_stats,
    plot_monthly_heatmap,
    plot_drawdown_analysis,
    plot_daily_return_distribution,
    plot_colored_table,
    plot_long_short_comparison,
    KlineChart,
    COLOR_DRAWDOWN,
    COLOR_RETURN,
)

# 或者直接从子模块导入（向后兼容）
from czsc.utils.plotting.backtest import (
    plot_cumulative_returns,
    plot_backtest_stats,
    plot_monthly_heatmap
)

# 准备日收益数据（index为日期，columns为策略收益）
dret = ...  # DataFrame with datetime index and returns columns

# 1. 绘制累计收益曲线
fig = plot_cumulative_returns(
    dret,
    title="策略累计收益",
    template="plotly",  # 或 "plotly_dark"
    to_html=False  # 返回 Figure 对象，True 则返回 HTML 字符串
)
fig.show()

# 2. 绘制综合回测统计图（包含回撤分析、收益分布、月度热力图）
fig = plot_backtest_stats(
    dret,
    ret_col="total",  # 指定收益列
    title="回测统计概览",
    template="plotly"
)
fig.show()

# 3. 单独绘制月度收益热力图
fig = plot_monthly_heatmap(dret, ret_col="total")
fig.show()

# 4. 导出为 HTML（用于报告）
html_str = plot_cumulative_returns(dret, to_html=True)
with open("backtest_report.html", "w", encoding="utf-8") as f:
    f.write(html_str)
```

**模块级常量使用：**
```python
from czsc.utils.plotting import (
    COLOR_DRAWDOWN,
    COLOR_RETURN,
    QUANTILES_DRAWDOWN,
    MONTH_LABELS
)

# 使用预定义常量确保图表风格统一
# 避免硬编码颜色值和配置参数
```

**分析工具使用：**
```python
from czsc.utils.analysis import (
    daily_performance,
    top_drawdowns,
    rolling_daily_performance,
    nmi_matrix,
    cross_sectional_ic,
    psi,
)

# 日收益表现分析
stats = daily_performance(dret["total"])
drawdowns = top_drawdowns(dret["total"])

# 相关性分析
ic = cross_sectional_ic(factor_df, return_df)
```

**K线图表绘制：**
```python
from czsc.utils.plotting import KlineChart

# 创建K线图表（支持多子图）
kc = KlineChart(n_rows=3)
kc.add_kline(czsc_obj, name="K线")
kc.add_sma(czsc_obj, period=5, name="MA5")
kc.add_volume(czsc_obj, row=2)
kc.fig.show()
```

### 依赖管理（UV配置）
- 核心运行时依赖定义在 `pyproject.toml` 的 `[project.dependencies]` 中
- 开发依赖在 `[project.optional-dependencies.dev]` 中
- 测试依赖在 `[project.optional-dependencies.test]` 中
- 使用 UV 进行依赖管理和虚拟环境控制
- 参考 `docs/UV管理开源项目指南.md` 获取详细指导

## 关键环境变量和设置

- `CZSC_USE_PYTHON`: 强制使用 Python 版本实现（默认优先使用 Rust 版本）
- `czsc_min_bi_len`: 最小笔长度（来自 `czsc.envs`）
- `czsc_max_bi_num`: 最大笔数量（来自 `czsc.envs`）
- 缓存目录自动管理，具备大小监控功能

## 缓存管理

项目大量使用磁盘缓存：
- 缓存位置：`czsc.utils.cache.home_path`
- 清除缓存：`czsc.empty_cache_path()`
- 监控大小：`czsc.get_dir_size(home_path)`
- 当缓存超过1GB时会显示清理提示

## Streamlit集成

项目在 `czsc/utils/st_components.py` 中包含丰富的 Streamlit 分析组件，提供回测结果、相关性分析、因子分析等可视化工具。

## Rust/Python 混合架构

项目核心功能使用 Rust 重构以提升性能：
- **版本控制**: 通过环境变量 `CZSC_USE_PYTHON` 控制，默认优先使用 Rust 版本
- **回退机制**: Rust 版本不可用时自动回退到 Python 版本（见 `czsc/core.py`）
- **核心模块**: 已迁移的模块包括 `CZSC` 分析器、K线生成器、枚举类型等
- **版本检测**: 运行时自动检测 `rs-czsc` 库可用性和版本信息

## 数据连接器支持

项目集成多个数据源连接器（见 `czsc/connectors/`）：
- `tq_connector.py`: 天勤数据源
- `ts_connector.py`: Tushare数据源
- `jq_connector.py`: 聚宽数据源
- `ccxt_connector.py`: 数字货币数据源
- `research.py`: 研究数据接口
- `cooperation.py`: 合作数据接口

## 回测和策略研究框架

### 策略开发基础（`czsc/strategies.py`）
- `CzscStrategyBase`: 策略开发的抽象基类
- `CzscJsonStrategy`: JSON配置化的策略实现
- 策略要素：品种参数、K线周期、信号配置、持仓策略
- 支持策略序列化和反序列化

### CTA研究框架（`czsc/sensors/cta.py`）
- `CTAResearch` 类提供统一的策略回测接口
- 支持单品种回放、多品种优化、参数网格搜索
- 并行处理支持，提升大规模回测效率
- 自动保存回测结果和策略配置

### 信号函数体系（`czsc/signals/`）
- 信号函数按类别组织：`bar.py`（K线级别）、`pos.py`（持仓）、`cxt.py`（上下文）等
- 版本化命名规范（如 `V241013`）便于管理和兼容性
- 支持信号匹配、分类和决策逻辑
- 信号配置支持动态加载和解析

### 特征分析工具（`czsc/sensors/`）
- `feature.py`: 特征选择器和滚动特征分析
- `event.py`: 事件匹配和检测
- `utils.py`: 特征工程工具函数

### 探索性数据分析（`czsc/eda.py`）
- `vwap`/`twap`: 成交量和时间加权平均价
- `remove_beta_effects`: 去除beta对因子的影响
- `cross_sectional_strategy`: 横截面策略分析
- `judge_factor_direction`: 因子方向判断
- `monotonicity`: 单调性分析
- `turnover_rate`: 换手率计算
- `make_price_features`: 价格特征工程

### 特征计算（`czsc/features/`）
**注意**：`ret.py` 中的函数包含未来信息，仅用于模型训练和因子评价，不可用于实际交易
- `RET001`: 未来 N 根K线收益率（close）
- `RET002`: 未来 N 根K线收益率（open）
- `RET003`: 未来 N 根K线收益波动率

### 飞书服务集成（`czsc/fsa/`）
- 支持飞书多维表格、即时消息、电子表格等接口
- 用于策略推送、数据同步、协同研究等场景

## 重要文档和资源

- [项目文档](https://s0cqcxuy3p.feishu.cn/wiki/wikcn3gB1MKl3ClpLnboHM1QgKf)
- [信号函数编写规范](https://s0cqcxuy3p.feishu.cn/wiki/wikcnCFLLTNGbr2THqo7KtWfBkd)
- [API文档](https://czsc.readthedocs.io/en/latest/modules.html)
- [B站视频教程](https://space.bilibili.com/243682308/channel/series)
- [UV管理开源项目指南](./docs/UV管理开源项目指南.md)：详细的UV包管理和开发工作流程指南

## 示例代码和用例

项目提供丰富的示例代码（`examples/` 目录）：

### 核心功能示例
- `30分钟笔非多即空.py`: 基础缠论策略实现
- `use_cta_research.py`: CTA研究框架使用
- `use_optimize.py`: 参数优化工具使用
- `use_html_report_builder.py`: HTML 报告生成示例
- `策略持仓权重管理.ipynb`: 权重管理策略

### 报告与可视化
- `svc_demos.py`: Streamlit 交互式分析仪表板（运行：`streamlit run examples/svc_demos.py`）

### 数据源集成示例
- `TS数据源的形态选股.py`: Tushare数据源集成
- `test_offline/`: 离线测试和集成测试案例

### 开发工具示例
- `develop/`: 开发工具和测试脚本
  - `czsc_benchmark.py`: CZSC 分析性能基准测试
  - `test_trading_view_kline.py`: K 线图可视化测试
- `signals_dev/`: 信号函数开发和测试
- `test_offline/`: 各种连接器和功能测试

### Streamlit应用
- `animotion/`: Streamlit可视化应用
  - `czsc_app.py`: 主应用界面
  - `czsc_human_replay.py`: 人工回放工具
  - `czsc_stream.py`: 实时数据流展示

### 文档资源
- `docs/`: 项目文档，包括开发日志、学习资料等
- `docs/source/`: Sphinx文档源文件

## 项目特色和最佳实践

1. **混合架构设计**: Rust性能优化 + Python灵活性
2. **多级别联立分析**: 支持多时间周期综合决策
3. **系统化信号体系**: 信号→事件→交易的完整流程
4. **丰富的数据源**: 支持A股、期货、数字货币等多市场
5. **完善的测试框架**: 统一的模拟数据生成和测试规范
6. **可视化工具**: Streamlit组件库支持快速分析展示
7. **策略研究工具**: CTA框架、参数优化、回测分析一体化
8. **代码质量优化**: 遵循 DRY、KISS、SOLID 原则
   - 使用模块级常量消除魔法值
   - 提取辅助函数减少代码重复
   - 完善的类型提示（Type Hints）
   - 清晰的函数职责分离
   - 保持向后兼容性的 API 设计

### 代码优化案例

**`czsc/utils/plotting/` 模块**展示了代码重构的最佳实践：
- 将原来的 `plot_backtest.py` 重构为模块化结构
- 提取 `common.py` 统一管理常量和辅助函数
- 按功能分离 `backtest.py`、`weight.py`、`kline.py`
- 保持向后兼容性（可通过 `plotting` 模块统一导入）

**报告生成工具：**
- `pdf_report_builder.py`: 支持 PDF 报告生成，包含目录、中文支持、图表嵌入
- `html_report_builder.py`: 支持 HTML 报告生成，支持自定义样式