# CZSC Rust/Python 混合架构说明

> 本指南介绍 CZSC 的 Rust/Python 混合架构设计和使用方法

## 学习目标

完成本章学习后，你将能够：

### 基础目标（必掌握）

- [ ] 理解混合架构的设计思想
- [ ] 掌握版本选择方法
- [ ] 理解自动回退机制
- [ ] 能够根据需要选择版本

### 进阶目标（建议掌握）

- [ ] 理解性能差异和适用场景
- [ ] 掌握环境变量配置
- [ ] 了解版本兼容性问题

### 专家目标（挑战）

- [ ] 理解 Rust 绑定实现原理
- [ ] 能够开发自定义 Rust 扩展
- [ ] 参与项目贡献

---

## 一、架构概述

### 1.1 设计目标

```python
"""
混合架构设计目标：

1. 性能优先
   - 核心计算使用 Rust 实现
   - 充分利用 Rust 的零成本抽象

2. 灵活性
   - Python 版本作为回退方案
   - 保持 API 兼容性

3. 渐进式迁移
   - 逐步迁移核心模块
   - 用户无感知切换
"""
```

### 1.2 架构图

```
┌─────────────────────────────────────────────────────────┐
│                    CZSC 用户代码                         │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│              czsc.core (统一接口层)                      │
│           CZSC, BarGenerator, RawBar, ...                │
└─────────────────────────────────────────────────────────┘
                           │
            ┌──────────────┴──────────────┐
            ▼                             ▼
┌─────────────────────┐     ┌─────────────────────┐
│   rs-czsc (Rust)    │     │  czsc.py (Python)   │
│   ────────────────  │     │  ─────────────────  │
│   • 高性能          │     │  • 完整功能         │
│   • 核心算法        │     │  • 灵活扩展         │
│   • 内存安全        │     │  • 易于调试         │
└─────────────────────┘     └─────────────────────┘
            ▲                             ▲
            └──────────────┬──────────────┘
                           │
                  自动回退机制
            (Rust 不可用时使用 Python)
```

### 1.3 模块迁移状态

| 模块 | Rust 版本 | Python 版本 | 说明 |
|------|----------|------------|------|
| CZSC 分析器 | ✅ | ✅ | 核心，完全兼容 |
| BarGenerator | ✅ | ✅ | 合成算法 |
| 数据对象 | ✅ | ✅ | RawBar, NewBar, FX, BI, ZS |
| 包含关系处理 | ❌ | ✅ | 仅 Python |
| 分型/笔检查 | ❌ | ✅ | 仅 Python |
| 缓存管理 | ✅ | ✅ | 两个版本都支持 |
| 回测框架 | ✅ | ✅ | WeightBacktest |

---

## 二、版本选择机制

### 2.1 自动选择逻辑

```python
"""
默认优先级：Rust > Python

选择逻辑：
1. 检查环境变量 CZSC_USE_PYTHON
2. 尝试导入 rs_czsc
3. 失败则回退到 Python 版本

代码实现（czsc/core.py）：
"""

import os
from czsc.core import check_rs_czsc

installed, rs_czsc_version = check_rs_czsc()

if os.getenv('CZSC_USE_PYTHON', False) or not installed:
    # 使用 Python 版本
    from czsc.py import CZSC, BarGenerator, ...
else:
    # 使用 Rust 版本
    from rs_czsc import CZSC, BarGenerator, ...
```

### 2.2 强制使用 Python 版本

```python
# 方法 1：设置环境变量
import os
os.environ['CZSC_USE_PYTHON'] = '1'

# 方法 2：在启动脚本中设置
# 终端命令：
# export CZSC_USE_PYTHON=1
# 或 Windows:
# set CZSC_USE_PYTHON=1

# 导入（会使用 Python 版本）
from czsc.core import CZSC
```

### 2.3 检测当前版本

```python
from czsc.core import check_rs_czsc

# 检查 Rust 版本是否可用
installed, version = check_rs_czsc()

if installed:
    print(f"Rust 版本可用: {version}")
else:
    print("Rust 版本不可用，使用 Python 版本")

# 检查当前使用的版本
from czsc import CZSC
print(f"CZSC 模块来源: {CZSC.__module__}")
# 输出：rs_czsc 或 czsc.py
```

---

## 三、性能对比

### 3.1 基准测试

```python
"""
性能对比（处理 10000 根K线）：

操作                    | Python   | Rust     | 提升
------------------------|----------|----------|------
CZSC 初始化             | 850ms    | 45ms     | 19x
包含关系处理            | 320ms    | 18ms     | 18x
分型识别                | 180ms    | 12ms     | 15x
笔识别                  | 420ms    | 28ms     | 15x
中枢识别                | 250ms    | 20ms     | 12x
------------------------|----------|----------|------
总计                    | 2020ms   | 123ms    | 16x

注：实际性能取决于数据特征和硬件配置
"""
```

### 3.2 使用场景建议

```python
"""
版本选择建议：

使用 Rust 版本（默认）：
✓ 大批量数据处理
✓ 实时计算场景
✓ 回测和优化
✓ 生产环境

使用 Python 版本：
✓ 调试和开发
✓ 需要修改算法
✓ 学习源码
✓ 不满足 Rust 依赖
"""
```

---

## 四、环境配置

### 4.1 安装 Rust 版本

```bash
# 方法 1：自动安装（推荐）
pip install czsc

# rs-czsc 会作为依赖自动安装

# 方法 2：手动指定版本
pip install rs-czsc>=0.1.17

# 验证安装
python -c "from czsc.core import check_rs_czsc; print(check_rs_czsc())"
# 输出：(True, '0.1.17')
```

### 4.2 配置环境变量

```bash
# Linux/macOS
export CZSC_USE_PYTHON=0  # 使用 Rust 版本（默认）
export CZSC_USE_PYTHON=1  # 使用 Python 版本

# Windows (CMD)
set CZSC_USE_PYTHON=0

# Windows (PowerShell)
$env:CZSC_USE_PYTHON="0"

# Python 代码中设置
import os
os.environ['CZSC_USE_PYTHON'] = '1'
```

### 4.3 依赖管理

```bash
# 查看 Rust 版本
pip show rs-czsc

# 升级 Rust 版本
pip install --upgrade rs-czsc

# 卸载 Rust 版本（强制使用 Python）
pip uninstall rs-czsc

# 检查兼容性
# 确保 czsc 和 rs-czsc 版本匹配
# CZSC 0.10.x 需要 rs-czsc >= 0.1.17
```

---

## 五、API 兼容性

### 5.1 完全兼容的 API

```python
# 以下 API 在两个版本中完全兼容

from czsc.core import (
    # 枚举类型
    Freq, Operate, Mark, Direction,

    # 核心类
    CZSC, BarGenerator,

    # 数据对象
    RawBar, NewBar, FX, BI, FakeBI, ZS, Signal, Event, Position,

    # 函数
    format_standard_kline,
)

# 使用方式完全一致
czsc_obj = CZSC(bars)
bg = BarGenerator(base_freq='1分钟', freqs=['5分钟', '日线'])
```

### 5.2 始终使用 Python 实现的功能

```python
# 以下独立函数始终使用 Python 实现（无论是否安装 rs-czsc）
# Rust CZSC 分析器内部已集成等价实现，但这些独立函数可用于自定义分析流程

from czsc.core import (
    # 分析函数
    remove_include,    # 包含关系处理
    check_bi,          # 笔检查
    check_fx,          # 分型检查
    check_fxs,         # 分型序列检查

    # K线生成器函数
    freq_end_time,     # 周期结束时间
    is_trading_time,   # 交易时间判断
)

# 使用 Python 版本时可用
from czsc.py import remove_include, check_bi, check_fx, check_fxs
```

### 5.3 兼容性处理

```python
# 编写兼容两个版本的代码

try:
    # 尝试使用 Python 版本的函数
    from czsc.py import check_fx
    has_py_func = True
except ImportError:
    # Rust 版本不可用时的处理
    has_py_func = False
    print("注意：check_fx 函数仅在 Python 版本可用")

if has_py_func:
    fx = check_fx(k1, k2, k3)
else:
    # 使用替代方法或跳过
    pass
```

---

## 六、常见问题

### Q1: Rust 版本安装失败？

A: 尝试以下方法：

```bash
# 1. 更新 pip
pip install --upgrade pip

# 2. 使用预编译包
pip install --only-binary :all: czsc

# 3. 强制使用 Python 版本
export CZSC_USE_PYTHON=1
pip install czsc
```

### Q2: 版本切换后行为不一致？

A: 检查以下事项：

```python
# 1. 确认当前使用的版本
from czsc.core import CZSC
print(CZSC.__module__)

# 2. 检查数据输入
# 两个版本对数据格式要求可能略有不同

# 3. 检查环境变量
import os
print(os.getenv('CZSC_USE_PYTHON'))

# 4. 重新导入
import importlib
import czsc
importlib.reload(czsc)
```

### Q3: 性能不如预期？

A: 检查以下因素：

```python
"""
性能影响因素：

1. 数据量
   - 小数据量：差异不明显
   - 大数据量：Rust 优势明显

2. 操作类型
   - 简单操作：差异小
   - 复杂计算：Rust 优势大

3. I/O 密集型
   - 数据读取可能是瓶颈
   - 使用缓存提高性能

4. Python 调用开销
   - 频繁调用 Rust 函数有开销
   - 批量处理减少调用次数
"""
```

---

## 七、最佳实践

### 7.1 开发建议

```python
"""
开发建议：

1. 开发阶段
   - 使用 Python 版本
   - 便于调试和修改

2. 测试阶段
   - 使用 Rust 版本
   - 验证性能需求

3. 生产环境
   - 默认使用 Rust 版本
   - 设置 Python 版本为回退

4. 持续集成
   - 测试两个版本
   - 确保兼容性
"""
```

### 7.2 迁移指南

```python
"""
从纯 Python 迁移到混合架构：

1. 安装 Rust 版本
   pip install rs-czsc

2. 测试现有代码
   - 运行测试用例
   - 验证结果一致性

3. 性能对比
   - 测量性能提升
   - 识别瓶颈

4. 逐步切换
   - 非关键路径先切换
   - 关键路径充分测试
"""
```

---

## 八、练习与巩固

### 练习 1：版本检测

编写代码检测当前使用的版本，并打印详细信息。

### 练习 2：性能对比

对比处理相同数据时，两个版本的性能差异。

### 练习 3：兼容代码

编写兼容两个版本的代码，处理 Python 独有功能。

---

## 九、进阶学习

完成本章后，建议继续学习：

1. **[核心概念详解](./核心概念详解.md)** - 理解核心功能
2. **[性能优化指南](./性能优化指南.md)** - 学习性能优化
3. **[策略开发指南](./策略开发指南.md)** - 学习策略开发

---

**文档版本**: v1.0.0
**最后更新**: 2026-02-28
**维护**: CZSC Community
