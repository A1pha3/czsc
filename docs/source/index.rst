.. czsc documentation master file
   CZSC - 缠中说禅技术分析工具

欢迎来到 CZSC 文档！
================================

CZSC 是一个基于缠中说禅理论的综合性量化交易 Python 库，提供技术分析、信号生成、回测和市场分析等功能。

.. note::
   **版本说明**：0.10.X 版本开始，核心功能逐步使用 rs-czsc（Rust 实现）替换 Python 实现，性能显著提升。

快速开始
--------

.. toctree::
   :maxdepth: 2
   :caption: 快速开始

   学习资料
   命令行工具
   开发日志

核心模块
--------

.. toctree::
   :maxdepth: 2
   :caption: 核心模块

   modules

主要功能
--------

* **缠论分析**：自动识别分型、笔、线段、中枢
* **多级别分析**：支持多时间周期联立分析
* **信号系统**：完整的信号-事件-交易体系
* **回测框架**：CTA 策略研究和参数优化
* **数据连接**：支持 Tushare、聚宽、天勤等多数据源
* **可视化工具**：丰富的图表和分析工具

文档导航
---------

.. list-table::
   :widths: 25 50 25
   :header-rows: 1

   * - 文档
     - 说明
     - 适用人群
   * - `快速开始指南 <../快速开始指南.html>`_
     - 15分钟快速上手
     - 所有用户
   * - `源码阅读指南 <../源码阅读指南.html>`_
     - 系统学习代码结构
     - 开发者
   * - `项目核心工具 <../项目核心工具.html>`_
     - 核心工具 API 参考
     - 所有用户
   * - `英文版指南 <../SOURCE_CODE_READING_GUIDE.html>`_
     - English Source Code Guide
     - English Users

在线资源
---------

* **API 文档**: https://czsc.readthedocs.io/en/latest/modules.html
* **飞书项目文档**: https://s0cqcxuy3p.feishu.cn/wiki/wikcn3gB1MKl3ClpLnboHM1QgKf
* **B 站视频教程**: https://space.bilibili.com/243682308/channel/series
* **GitHub 仓库**: https://github.com/waditu/czsc

社区交流
---------

* 飞书群: `加入链接 <https://applink.feishu.cn/client/chat/chatter/add_by_link?link_token=0bak668e-7617-452c-b935-94d2c209e6cf>`_
* GitHub Issues: https://github.com/waditu/czsc/issues

索引和表格
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
