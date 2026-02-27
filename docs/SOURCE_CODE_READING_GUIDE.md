# CZSC Source Code Reading Guide

> A comprehensive guide to understanding the CZSC (Chan Theory Technical Analysis) quantitative trading library

## 🎯 Quick Start

### What is CZSC?

CZSC is a comprehensive Python library for quantitative trading based on Chan Theory (缠中说禅), featuring:

- ✅ Automated identification of patterns (分型), strokes (笔), segments (线段), and pivots (中枢)
- ✅ Multi-timeframe joint analysis framework
- ✅ Signal-Event-Trading system
- ✅ Strategy backtesting and optimization
- ✅ Multiple data source connectors
- ✅ Rust/Python hybrid architecture for performance
- ✅ Comprehensive analysis and visualization tools

## 📚 Reading Order

### Phase 1: Core Data Structures (Must Read)

| File | Key Content | Focus |
|------|-------------|-------|
| `czsc/py/enum.py` | Enumerations | `Freq`, `Mark`, `Direction`, `Operate` |
| `czsc/py/objects.py` | Data structures | `RawBar`, `NewBar`, `FX`, `BI`, `ZS`, `Signal`, `Event`, `Position` |
| `czsc/core.py` | Hybrid architecture | Rust/Python smart import |

### Phase 2: Core Algorithms (Must Read)

| File | Key Content | Focus |
|------|-------------|-------|
| `czsc/py/analyze.py` | CZSC analysis class | `remove_include`, `check_fx`, `check_bi` |
| `czsc/py/bar_generator.py` | Bar generator | Multi-timeframe synthesis |

### Phase 3: Signal System (Important)

| File | Key Content |
|------|-------------|
| `czsc/signals/bar.py` | Bar-level signals |
| `czsc/signals/cxt.py` | Context signals |
| `czsc/signals/tas.py` | Technical indicator signals |
| `czsc/signals/vol.py` | Volume signals |
| `czsc/signals/pos.py` | Position-related signals |

### Phase 4: Trading Framework (Core Application)

| File | Key Content |
|------|-------------|
| `czsc/traders/base.py` | `CzscSignals`, `CzscTrader` |
| `czsc/strategies.py` | Strategy templates |
| `czsc/sensors/cta.py` | CTA research framework |
| `czsc/traders/dummy.py` | Simple backtest |

### Phase 5: Tools & Services (Optional)

| File | Key Content |
|------|-------------|
| `czsc/utils/plotting/` | Visualization tools (refactored) |
| `czsc/utils/analysis/` | Statistical analysis tools (new) |
| `czsc/utils/ta.py` | Technical indicators |
| `czsc/svc/backtest.py` | Backtest analysis |
| `czsc/connectors/` | Data source connectors |

## 🔑 Core Concepts

### Signal-Event-Trading System

```
Raw Bars → Signals → Events → Trading Decisions
```

- **Signal**: Basic technical indicator or market state
- **Event**: Logical combination of signals (AND/OR/NOT) representing trading conditions
  - Contains `signals_all` (must satisfy all), `signals_any` (satisfy any), `signals_not` (must not appear)
- **Position**: Complete trading strategy with entry/exit rules

### Chan Theory Objects

```python
RawBar   # Raw K-line with OHLCV
  ↓ Remove inclusion relationship
NewBar   # Processed K-line without inclusion
  ↓ Identify patterns
FX       # Pattern (top/bottom)
  ↓ Identify strokes
BI       # Stroke (connection between adjacent patterns)
  ↓ Identify segments and pivots
ZS       # Pivot (price oscillation zone)
```

## 💻 Quick Examples

### Example 1: Basic Chan Analysis

```python
from czsc.core import CZSC, format_standard_kline, Freq
from czsc.mock import generate_symbol_kines

# Generate K-line data
df = generate_symbol_kines('000001', '30分钟', '20240101', '20240131')
bars = format_standard_kline(df, freq=Freq.F30)

# Create CZSC analysis object
czsc_obj = CZSC(bars)

# View results
print(f"Raw bars: {len(czsc_obj.bars_raw)}")
print(f"Patterns: {len(czsc_obj.fx_list)}")
print(f"Strokes: {len(czsc_obj.bi_list)}")
```

### Example 2: Multi-Timeframe Synthesis

```python
from czsc.core import BarGenerator, format_standard_kline, Freq
from czsc.mock import generate_symbol_kines

# Prepare 1-minute bars
df = generate_symbol_kines('000001', '1分钟', '20240101', '20240105')
bars = format_standard_kline(df, freq=Freq.F1)

# Create bar generator
bg = BarGenerator(
    base_freq='1分钟',
    freqs=['5分钟', '15分钟', '30分钟', '日线']
)

# Update bars
for bar in bars:
    bg.update(bar)

# Check results
for freq, freq_bars in bg.bars.items():
    print(f"{freq}: {len(freq_bars)} bars")
```

### Example 3: Calculate Signals

```python
from czsc.traders import CzscSignals

# Define signal configuration
signals_config = [
    {'name': 'czsc.signals.tas_ma_base_V221101',
     'freq': '日线', 'di': 1, 'ma_type': 'SMA', 'timeperiod': 5},
]

# Create signal calculator
cs = CzscSignals(bg, signals_config=signals_config)

# View current signals
print("Current signals:")
for k, v in cs.s.items():
    print(f"  {k}: {v}")
```

### Example 4: Visualization (New Unified Interface)

```python
# Recommended: Use the new unified plotting interface
from czsc.utils.plotting import (
    plot_cumulative_returns,
    plot_backtest_stats,
    plot_monthly_heatmap,
    KlineChart,
)

# Create K-line chart
kc = KlineChart(n_rows=2)
kc.add_kline(czsc_obj, name="K-line")
kc.add_sma(czsc_obj, period=5, name="MA5")
kc.add_volume(czsc_obj, row=2)
kc.fig.show()

# Plot backtest statistics
fig = plot_backtest_stats(dret, ret_col='total')
fig.show()
```

### Example 5: Statistical Analysis (New Module)

```python
# New analysis tools module
from czsc.utils.analysis import (
    daily_performance,
    top_drawdowns,
    cross_sectional_ic,
    nmi_matrix,
)

# Calculate performance metrics
stats = daily_performance(daily_returns)
print(f"Annual Return: {stats['年化']:.2%}")
print(f"Sharpe Ratio: {stats['夏普']:.2f}")
print(f"Max Drawdown: {stats['最大回撤']:.2%}")

# Calculate cross-sectional IC
ic = cross_sectional_ic(factor_df, return_df)
print(f"IC Mean: {ic.mean():.4f}")
print(f"IC IR: {ic.mean()/ic.std():.4f}")
```

## 📖 Learning Path

### Level 1: Beginner (1-2 weeks)
- Understand basic concepts
- Run example strategies
- Modify strategy parameters

### Level 2: Intermediate (2-4 weeks)
- Read signal functions
- Write custom signals
- Understand naming conventions

### Level 3: Advanced (4-8 weeks)
- Develop complete strategies
- Use CTAResearch framework
- Parameter optimization

### Level 4: Expert (Ongoing)
- Understand Rust implementation
- Contribute code
- Develop custom connectors

## 📦 Key Modules

### Core Modules (Must Read)
```
czsc/
├── core.py                    # Entry point
├── py/
│   ├── enum.py                # Enumerations
│   ├── objects.py             # Data structures
│   ├── analyze.py             # CZSC class
│   └── bar_generator.py       # Bar generator
├── signals/                   # Signal library
├── traders/                   # Trading framework
└── strategies.py              # Strategy templates
```

### Tools & Services (Optional)
```
czsc/
├── utils/
│   ├── ta.py                  # Technical indicators
│   ├── plotting/              # Visualization (refactored)
│   │   ├── backtest.py        # Backtest charts
│   │   ├── weight.py          # Weight analysis
│   │   ├── kline.py           # K-line charts
│   │   └── common.py          # Common utilities
│   └── analysis/              # Statistical analysis (new)
│       ├── stats.py           # Performance statistics
│       ├── corr.py            # Correlation analysis
│       └── events.py          # Event analysis
├── svc/                       # Services
│   └── backtest.py            # Backtest analysis
└── connectors/                # Data connectors
```

## 🎨 New Modules (v0.10.8+)

### Visualization Tools (`utils/plotting/`)

The plotting module has been refactored into a modular structure with a unified import interface:

```python
# Unified import (recommended)
from czsc.utils.plotting import (
    # Backtest plotting
    plot_cumulative_returns,
    plot_backtest_stats,
    plot_monthly_heatmap,
    plot_drawdown_analysis,
    plot_daily_return_distribution,

    # K-line charts
    KlineChart,

    # Constants
    COLOR_DRAWDOWN,
    COLOR_RETURN,
)
```

### Analysis Tools (`utils/analysis/`)

New statistical analysis module for comprehensive performance evaluation:

```python
from czsc.utils.analysis import (
    # Performance statistics
    daily_performance,
    top_drawdowns,
    rolling_daily_performance,
    holds_performance,
    psi,

    # Correlation analysis
    nmi_matrix,
    single_linear,
    cross_sectional_ic,

    # Event analysis
    overlap,
)
```

## 🔗 Resources

### Documentation
- [API Documentation](https://czsc.readthedocs.io/en/latest/modules.html)
- [Quick Start Guide](快速开始指南.md) (Chinese)
- [Source Code Guide](源码阅读指南.md) (Chinese)
- [Core Tools Reference](项目核心工具.md) (Chinese)
- [Feishu Wiki](https://s0cqcxuy3p.feishu.cn/wiki/wikcn3gB1MKl3ClpLnboHM1QgKf) (Chinese)
- [Video Tutorials](https://space.bilibili.com/243682308/channel/series) (Chinese)

### Community
- [Feishu Group](https://applink.feishu.cn/client/chat/chatter/add_by_link?link_token=0bak668e-7617-452c-b935-94d2c209e6cf)
- GitHub Issues
- WeChat: zengbin93

## ❓ FAQ

**Q: Python or Rust version?**
- Beginners: Read Python version (`czsc/py/`)
- Set `CZSC_USE_PYTHON=1` to force Python
- Default: Rust version for better performance

**Q: What is `di` parameter?**
- `di`: Distance Index (倒数第几根K线)
- `di=1`: Last bar (real-time)
- `di=2`: Second to last bar

**Q: How to understand "remove inclusion"?**
- Core Chan Theory concept
- Merges bars with inclusion relationship
- See `remove_include` in `czsc/py/analyze.py`

**Q: What's the difference between old and new plotting modules?**
- Old: `czsc.utils.plotly_plot` (monolithic)
- New: `czsc.utils.plotting` (modular, recommended)
- Both work, but new module provides better organization

## 📊 Strategy Structure

```python
from czsc import CzscStrategyBase, Position, Event

class MyStrategy(CzscStrategyBase):
    @property
    def positions(self):
        # Define entry events
        opens = [{
            "operate": "开多",
            "signals_all": ["signal1", "signal2"],  # Must satisfy all
            "signals_any": ["signal3"],              # Satisfy any
            "signals_not": ["signal5"],              # Must not appear
        }]

        # Define exit events
        exits = [...]

        # Create position object
        pos = Position(
            name="Strategy Name",
            symbol=self.symbol,
            opens=[Event.load(x) for x in opens],
            exits=[Event.load(x) for x in exits],
            interval=3600,  # Entry interval (seconds)
            timeout=100,    # Timeout (bars)
            stop_loss=500   # Stop loss (basis points)
        )
        return [pos]
```

## 🎓 Recommended Reading Order

1. ✅ **Day 1**: Core concepts (this document)
2. ✅ **Day 2-3**: Data structures (`objects.py`, `enum.py`)
3. ✅ **Day 4-6**: Core algorithms (`analyze.py`)
4. ✅ **Week 2**: Signal system (`signals/`)
5. ✅ **Week 3-4**: Trading framework (`traders/`, `strategies.py`)
6. ✅ **Optional**: Visualization (`utils/plotting/`) and Analysis (`utils/analysis/`)

**Remember**: Practice while learning! Run examples and modify code.

---

**Version**: v1.1.0
**Last Updated**: 2026-02-26
**Maintainer**: CZSC Community
