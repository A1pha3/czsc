# CZSC 文档

## 📚 文档导航

| 文档 | 说明 | 适用人群 |
|------|------|----------|
| [文档索引](文档索引.md) | 完整文档索引和学习路径 | 所有用户 |
| [快速开始指南](快速开始指南.md) | 15分钟快速上手 | 所有用户 |
| [源码阅读指南](源码阅读指南.md) | 系统学习代码结构 | 开发者 |
| [英文源码指南](SOURCE_CODE_READING_GUIDE.md) | English Source Code Guide | English Users |
| [项目核心工具](项目核心工具.md) | 核心工具API参考 | 所有用户 |
| [CWC 使用指南](instructions/cwc_usage_guide.md) | 策略持仓权重管理 | 策略开发者 |
| [日收益指标计算](instructions/日收益指标计算.md) | 回测指标详解 | 研究员 |
| [策略回测原理](instructions/策略回测原理.md) | 权重回测原理 | 研究员 |

## Sphinx 文档

### 安装依赖

```bash
pip install recommonmark sphinx_rtd_theme sphinx_automodapi
```

### 生成文档

在 base 环境下，执行以下命令：

```shell
# Windows
./make.bat clean
./make.bat html

# Linux/Mac
make clean
make html
```

生成的文档在 `build/html` 目录。

### 在线文档

- API 文档：https://czsc.readthedocs.io
- 飞书项目文档：https://s0cqcxuy3p.feishu.cn

## 参考资料

* [前复权、后复权、不复权价格区别与计算](https://liguoqinjim.cn/post/quant/fq_price/)
* [使用飞书创建自己的通知机器人](https://liguoqinjim.cn/post/tool/%E4%BD%BF%E7%94%A8%E9%A3%9E%E4%B9%A6%E5%88%9B%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E9%80%9A%E7%9F%A5%E5%9C%BA%E6%99%BA%E4%BA%BA/)
* [CLAUDE.md](../CLAUDE.md) - 开发者指南

## 文档结构

```
docs/
├── README.md                      # 本文件
├── 快速开始指南.md                 # 新手入门
├── 源码阅读指南.md                 # 代码结构详解
├── 项目核心工具.md                 # 工具API参考
├── instructions/                  # 详细说明文档
│   ├── cwc_usage_guide.md        # CWC 使用指南
│   ├── 日收益指标计算.md          # 指标计算说明
│   └── 策略回测原理.md            # 回测原理说明
├── source/                        # Sphinx 文档源文件
│   ├── conf.py
│   └── index.rst
└── make.bat / Makefile            # 文档生成脚本
```
