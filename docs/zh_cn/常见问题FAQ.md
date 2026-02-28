# CZSC 常见问题 FAQ

> 本文档收集了 CZSC 使用过程中的常见问题和解决方案

## 目录

- [安装与环境](#安装与环境)
- [数据获取](#数据获取)
- [信号计算](#信号计算)
- [回测相关问题](#回测相关问题)
- [性能相关](#性能相关)
- [可视化相关](#可视化相关)
- [缠论概念](#缠论概念)
- [错误处理](#错误处理)

---

## 安装与环境

### Q1: 如何安装 CZSC？

```bash
# 方法 1：使用 pip 安装（推荐）
pip install czsc -U

# 方法 2：使用 UV 安装（开发推荐）
git clone https://github.com/waditu/czsc.git
cd czsc
uv sync --extra dev

# 方法 3：从源码安装
pip install git+https://github.com/waditu/czsc.git -U
```

### Q2: 安装时 rs-czsc 失败怎么办？

```bash
# 如果 rs-czsc 预编译包不可用，可以：

# 1. 强制使用 Python 版本
export CZSC_USE_PYTHON=1
pip install czsc

# 2. 或稍后重试
pip install czsc --upgrade --no-cache-dir
```

### Q3: Python 版本要求？

```text
CZSC 支持 Python 3.10 及以上版本

推荐版本：
- Python 3.10（稳定）
- Python 3.11（性能更好）
- Python 3.12（最新）
- Python 3.13（最新）

不推荐：
- Python 3.9 及以下（不支持）
```

### Q4: 依赖安装冲突？

```bash
# 使用虚拟环境隔离依赖

# venv
python -m venv czsc_env
source czsc_env/bin/activate  # Linux/macOS
# czsc_env\Scripts\activate   # Windows

# conda
conda create -n czsc python=3.11
conda activate czsc

# 然后安装
pip install czsc
```

---

## 数据获取

### Q5: 如何获取真实市场数据？

```python
from czsc.utils import DataClient

# Tushare 数据源（需要 token）
dc = DataClient(token="your_tushare_token")
df = dc.get_kline(symbol="000001.SZ", freq="日线", sdt="20230101", edt="20231231")

# 聚宽数据源（需要账户）
from czsc.connectors.jq_connector import JqDataClient
jq_dc = JqDataClient(username="your_username", password="your_password")
df = jq_dc.get_kline(symbol="000001.XSHG", freq="1d")

# CCXT 数字货币（免费）
from czsc.connectors.ccxt_connector import CCXTConnector
ccxt_dc = CCXTConnector(exchange="binance", symbol="BTC/USDT")
df = ccxt_dc.get_kline(freq="1h", sdt="20240101", edt="20240131")
```

### Q6: 如何使用模拟数据？

```python
from czsc.mock import generate_symbol_kines

# 生成随机K线数据（用于测试）
df = generate_symbol_kines(
    symbol='000001',
    freq='30分钟',
    sdt='20240101',
    edt='20240131',
    seed=42  # 随机种子，保证可重现
)

# 转换为 CZSC 格式
from czsc.core import format_standard_kline, Freq
bars = format_standard_kline(df, freq=Freq.F30)
```

### Q7: 数据格式不正确怎么办？

```python
# 检查必需的列
required_columns = ['dt', 'open', 'close', 'high', 'low', 'vol', 'amount']

# 重命名列
df = df.rename(columns={
    'trade_date': 'dt',
    'volume': 'vol',
    'amount': 'amount'
})

# 确保时间格式正确
df['dt'] = pd.to_datetime(df['dt'])

# 排序
df = df.sort_values('dt').reset_index(drop=True)
```

---

## 信号计算

### Q8: 如何计算自定义信号？

```python
from collections import OrderedDict
from czsc.core import CZSC

def my_signal_V240228(c: CZSC, **kwargs) -> OrderedDict:
    """自定义信号函数"""
    di = kwargs.get('di', 1)

    # 数据校验
    if len(c.bars_raw) < di + 10:
        return OrderedDict()

    # 信号逻辑
    # ...

    # 返回信号
    s = {f"{c.freq.value}_my_signal_满足_0": "满足"}
    return s

# 使用
signals_config = [
    {'name': 'my_module.my_signal_V240228',
     'freq': '日线', 'di': 1},
]
```

### Q9: 信号函数命名规范？

```python
"""
信号命名格式：{频率}_{模块}_{特征}_{版本}

示例：
- "日线_tas_ma_base_满足_0"
- "30分钟_cxt_fx_power_强_0"
- "5分钟_vol_double_ma_金叉_0"

命名规则：
1. 频率：1分钟/5分钟/30分钟/日线等
2. 模块：tas/cxt/bar/vol/pos/zdy
3. 特征：具体信号名称
4. 版本：VYYMMDD 格式
"""
```

### Q10: 如何获取当前所有信号？

```python
from czsc.traders import CzscSignals

cs = CzscSignals(bg, signals_config=signals_config)

# 查看所有信号
for key, value in cs.s.items():
    print(f"{key}: {value}")

# 只查看信号相关
signals = {k: v for k, v in cs.s.items() if '信号' in k}
```

---

## 回测相关问题

### Q11: 如何进行策略回测？

```python
from czsc.traders import CzscTrader
from czsc.strategies import CzscStrategyBase

# 创建策略
strategy = MyStrategy(symbol='000001')

# 创建交易员
trader = CzscTrader(strategy=strategy, init_n=100)

# 执行回测
trader.replay(bars, sdt='20230101', res_path='./results')

# 查看结果
print(f"交易次数: {len(trader.positions[0].pairs)}")
print(f"胜率: {trader.positions[0].win_rate:.2%}")
```

### Q12: 如何处理手续费和滑点？

```python
# 在策略中定义手续费
class StrategyWithFee(CzscStrategyBase):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.commission_rate = 0.0003  # 万三手续费
        self.slippage_rate = 0.0001    # 万一滑点

    def calculate_pnl(self, open_price, close_price, amount):
        """计算扣除费用后的收益"""
        gross_pnl = (close_price - open_price) * amount
        commission = (open_price + close_price) * amount * self.commission_rate
        slippage = (open_price + close_price) * amount * self.slippage_rate
        return gross_pnl - commission - slippage
```

### Q13: 回测结果与实盘差异大？

```python
"""
常见原因：

1. 未来函数
   - 使用了未来数据
   - 检查信号计算时机

2. 理想化成交
   - 未考虑滑点
   - 未考虑流动性

3. 数据质量
   - 复权处理不当
   - 数据缺失

4. 过度拟合
   - 参数优化过度
   - 样本外验证不足
"""
```

### Q14: 如何进行参数优化？

```python
from itertools import product

param_grid = {
    'ma_period': [5, 10, 20],
    'timeout': [20, 30, 40],
}

# 生成所有组合
combinations = [dict(zip(param_grid.keys(), v))
                for v in product(*param_grid.values())]

# 测试每组参数
results = []
for params in combinations:
    strategy = MyStrategy(symbol='000001', **params)
    trader = CzscTrader(strategy=strategy)
    trader.replay(bars, sdt='20230101', res_path='./temp')

    pnl = sum(p.pnl for p in trader.positions[0].pairs)
    results.append({'params': params, 'pnl': pnl})

# 找出最佳参数
best = max(results, key=lambda x: x['pnl'])
print(f"最佳参数: {best['params']}, 收益: {best['pnl']}")
```

---

## 性能相关

### Q15: 如何提高计算速度？

```python
# 1. 使用 Rust 版本（默认）
# pip install rs-czsc

# 2. 使用缓存
from czsc.utils import DiskCache
dc = DataClient(cache_path="./cache")

# 3. 并行处理
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(backtest, symbols))
```

### Q16: 内存占用过高？

```python
from czsc.utils import home_path, get_dir_size, empty_cache_path

# 查看缓存大小
size_mb = get_dir_size(home_path) / 1024 / 1024
print(f"缓存大小: {size_mb:.2f}MB")

# 清理缓存
if size_mb > 1000:
    empty_cache_path()
```

### Q17: 多品种回测太慢？

```python
from czsc.sensors import CTAResearch

# 使用 CTAResearch 并行回测
cta = CTAResearch(
    strategy=MyStrategy,
    read_bars=read_bars,
    results_path='./results'
)

cta.dummy(
    symbols=symbols,
    sdt='20230101',
    edt='20231231',
    max_workers=8  # 并行进程数
)
```

---

## 可视化相关

### Q18: 如何绘制K线图？

```python
from czsc.utils.plotting import KlineChart

kc = KlineChart(n_rows=2, height=600)

# 添加K线
kc.add_kline(czsc_obj, name="000001", row=1)

# 添加笔
kc.add_bi(czsc_obj, row=1)

# 添加成交量
kc.add_volume(czsc_obj, row=2)

kc.fig.show()
```

### Q19: 如何导出图表？

```python
# 导出为HTML
html_str = plot_cumulative_returns(dret, to_html=True)
with open("backtest.html", "w") as f:
    f.write(html_str)

# 导出为图片（需要 kaleido）
fig = plot_cumulative_returns(dret, to_html=False)
fig.write_image("backtest.png")
```

### Q20: 中文显示乱码？

```python
import plotly.graph_objects as go

fig = plot_cumulative_returns(dret)
fig.update_layout(
    font=dict(
        family="SimHei",  # 或 "Microsoft YaHei"
        size=12
    )
)
```

---

## 缠论概念

### Q21: 什么是分型？

```python
"""
分型（FX）是K线组合形态：

顶分型：中间K线的高点和低点都是三根中最高的
   K1.high < K2.high > K3.high
   K1.low < K2.low > K3.low

底分型：中间K线的高点和低点都是三根中最低的
   K1.low > K2.low < K3.low
   K1.high > K2.high < K3.high

        顶分型            底分型
           /\              /\
          /  \            /  \
    K1   /    \ K3   K3  /    \ K1
        /      \          /      \
       K2      K        K2      K
"""
```

### Q22: 什么是笔？

```python
"""
笔（BI）是连接相邻分型的线段：

向上笔：底分型 → 顶分型，且顶分型高点 > 底分型低点
向下笔：顶分型 → 底分型，且底分型低点 < 顶分型高点

识别条件：
1. 至少跨越一根K线
2. 满足最小笔长度
3. 顶底分型交替

    D(底) ──────────────────────── G(顶)  向上笔
              └── 新的笔 ──→ G'(顶)
"""

from czsc.core import CZSC

czsc_obj = CZSC(bars)

# 查看笔
for bi in czsc_obj.bi_list:
    print(f"{bi.direction}: {bi.fx_a.dt} -> {bi.fx_b.dt}")
```

### Q23: 什么是中枢？

```python
"""
中枢（ZS）是价格震荡区间：

定义：至少三笔构成的区间重叠

向上中枢：下-上-下-上...
向下中枢：上-下-上-下...

    ZG（中枢高点）
        │
   ┌─────┴─────┐
   │   中枢    │
   │           │
   └─────┬─────┘
        │
     ZD（中枢低点）
"""

# 查看中枢
for zs in czsc_obj.zs_list:
    print(f"中枢: [{zs.zd}, {zs.zg}], 笔数: {len(zs.lines)}")
```

---

## 错误处理

### Q24: ImportError: No module named 'rs_czsc'

```python
"""
解决方案：

1. 安装 Rust 版本
   pip install rs-czsc

2. 或使用 Python 版本
   export CZSC_USE_PYTHON=1

3. 重新安装 czsc
   pip install --force-reinstall czsc
"""
```

### Q25: ValueError: 数据不足

```python
# 数据量不足时增加初始化数据

# 不推荐：数据刚好
# bg = BarGenerator(base_freq='1分钟', freqs=['日线'])
# for bar in bars_1m[:100]:  # 数据太少
#     bg.update(bar)

# 推荐：提供足够的初始化数据
bg = BarGenerator(base_freq='1分钟', freqs=['日线'])
for bar in bars_1m[:1000]:  # 提供充足数据
    bg.update(bar)
```

### Q26: 信号计算结果为空？

```python
# 检查以下几点

# 1. 数据是否充足
print(f"K线数量: {len(czsc_obj.bars_raw)}")
print(f"笔数量: {len(czsc_obj.bi_list)}")

# 2. 参数是否正确
print(f"di={di}, 需要至少 {di+1} 根K线")

# 3. 信号逻辑是否正确
# 打印调试信息
logger.debug(f"信号计算状态: ...")
```

### Q27: 缓存文件损坏？

```python
from czsc.utils import empty_cache_path

# 清空缓存重新生成
empty_cache_path()

# 重新运行
# 缓存会自动重建
```

---

## 其他问题

### Q28: 如何贡献代码？

```python
"""
贡献流程：

1. Fork 项目仓库
   https://github.com/waditu/czsc

2. 创建功能分支
   git checkout -b feature/your-feature

3. 编写代码和测试
   - 遵循代码规范
   - 添加单元测试

4. 提交 Pull Request
   - 描述变更内容
   - 等待审查
"""
```

### Q29: 如何获取帮助？

```python
"""
获取帮助的途径：

1. 文档
   - 项目文档：docs/zh_cn/
   - API 文档：https://czsc.readthedocs.io

2. 社区
   - GitHub Issues
   - 飞书群

3. 示例代码
   - examples/ 目录
   - 单元测试：test/ 目录
"""
```

### Q30: 版本更新注意事项？

```python
"""
版本更新建议：

1. 查看更新日志
   - GitHub Releases

2. 测试环境先升级
   - 验证兼容性

3. 备份重要数据
   - 策略配置
   - 缓存数据

4. 渐进式升级
   - 不要跨多个大版本
"""
```

---

## 获取更多帮助

如果以上问题无法解决您的疑问，请：

1. 查看 [完整文档](./文档索引.md)
2. 提交 [GitHub Issue](https://github.com/waditu/czsc/issues)
3. 加入社区讨论

---

**文档版本**: v1.0.0
**最后更新**: 2026-02-28
**维护**: CZSC Community

---

**相关文档**：
- [快速开始指南](./快速开始指南.md) - 快速上手
- [核心概念详解](./核心概念详解.md) - 深入理解
- [文档索引](./文档索引.md) - 查看所有文档
