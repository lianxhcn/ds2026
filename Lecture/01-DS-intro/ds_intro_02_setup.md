---
title: "数据分析与经济决策"
subtitle: "第一讲（下）：怎么做，用什么做"
author: 连玉君
date: 2026-05-10
format:
  html:
    toc: true
    toc-depth: 3
    number-sections: true
---

---

> **本章定位**：这不是一章需要精读的讲义，而是一份贯穿整个学期的**参考地图**。
> 建议课上快速浏览，建立整体印象；遇到具体问题时再回来查阅对应章节和链接。

---

## 数据分析的基本流程

一个完整的数据分析项目，通常经历以下阶段。这张图既是本章的结构，也是整门课程的骨架——后续每一讲都对应其中的某个环节。

```
问题定义 → 数据获取 → 数据清洗 → 探索性分析 → 建模分析 → 结果解读 → 报告与沟通
```

| 阶段 | 核心任务 | 最常见的陷阱 | 对应课程内容 |
|------|----------|-------------|-------------|
| **问题定义** | 把模糊问题转化为可用数据回答的具体问题 | 跳过这步直接找数据 | 第一讲 |
| **数据获取** | API、爬虫、数据库、调查 | 不记录数据来源和版本 | 数据获取专讲 |
| **数据清洗** | 缺失值、异常值、格式、合并 | 低估所需时间（实际占 60-80%）| 数据清洗专讲 |
| **探索性分析** | 分布、相关性、异常、可视化 | 用 EDA 图表替代正式分析就结束 | 可视化专讲 |
| **建模分析** | 统计推断、因果识别、预测建模 | 选模型先于想问题 | 建模模块 |
| **结果解读** | 区分统计显著与经济显著；承认局限 | 只报 p < 0.05，不报效应量 | 各建模讲均涉及 |
| **报告与沟通** | 可复现报告、可视化、叙事 | 结论对受众不可理解 | Quarto 专讲 |

::: {.callout-note}
**"80% 的时间在清洗数据"——认真的**

这不是夸张。真实项目中，数据获取和清洗加在一起，通常占整个分析时间的 60-80%。
建模部分反而最快。接受这个现实，会让你在遇到脏数据时少一些挫败感，多一些耐心。
:::

---

## 数据类型与数据结构

### 按结构化程度

| 类型 | 特征 | 典型例子 | 主要分析工具 |
|------|------|----------|-------------|
| **结构化数据** | 行列格式，有明确字段定义 | 股价日数据、宏观指标、问卷数据 | pandas、Excel、SQL |
| **半结构化数据** | 有嵌套层级，但非规则表格 | JSON、XML、HTML、财报 XBRL | json、BeautifulSoup、lxml |
| **非结构化数据** | 无预定义格式 | 新闻文本、图片、音频、视频 | NLP 工具、CV 库、ASR 模型 |

本课程以结构化数据为主，兼顾文本（半/非结构化）。图像、音频等数据类型在「多模态」一节简介。

### 按时间维度

| 类型 | 定义 | 例子 | 分析优势 |
|------|------|------|---------|
| **截面数据** | 某一时间点，多个个体 | 2024 年各省 GDP | 描述横截面差异 |
| **时间序列** | 同一个体，多个时间点 | 上证指数日收益率 | 刻画动态演化 |
| **面板数据** | 多个个体 × 多个时间点 | 31 省 × 20 年宏观数据 | 控制个体固定效应，增强因果识别 |

面板数据兼具截面和时序信息，是经济学实证研究中最常用的数据结构，也是本课程建模模块的重点。

### 按生成方式

这对区分**非常重要**，直接决定你能否做因果推断：

| 类型 | 定义 | 因果推断难度 | 例子 |
|------|------|-------------|------|
| **观测数据** | 自然发生，研究者不干预 | 高（存在混淆变量）| 企业财务数据、宏观统计数据 |
| **实验数据** | 研究者随机分配干预 | 低（随机化消除混淆）| 肯尼亚驱虫药 RCT |
| **准实验数据** | 利用外生冲击"模拟"随机化 | 中（需要论证外生性）| 广场协议作为汇率冲击 |

绝大多数经济数据是观测数据。因果推断的核心挑战，就是从观测数据中"还原"出类似实验的逻辑结构。

### 多模态数据（新兴方向）

**多模态（Multimodal）** 指同时结合多种数据类型的分析方法。随着大语言模型和多模态 AI 的发展，这个领域在金融和经济研究中正在快速扩张。

| 数据模态 | 金融/经济中的应用 | 典型来源 |
|----------|-----------------|---------|
| **文本** | 上市公司公告情感分析、货币政策声明解读、财报风险因素提取 | Wind、SEC EDGAR、Reuters |
| **图像** | 卫星遥感测算夜间灯光经济活动、停车场/港口车船数量 | NASA、Planet Labs、Google Earth |
| **表格 + 文本** | 财报 PDF（数字与文字混合）| 年报 PDF、招股书 |
| **音频/视频** | 分析师电话会议记录、CEO 声音语气与情绪识别 | Earnings call 录音 |
| **传感器/IoT** | 高频交易数据、供应链传感器数据 | 交易所 Level-2 数据 |

::: {.callout-note}
**多模态分析目前还在快速演化中**

本课程不会系统讲授多模态分析方法（超出课程范围），但有意识地了解它的存在很重要——它代表着数据分析边界的扩展方向。文本分析作为最成熟的非结构化数据方法，在本课程中有专讲。
:::

---

## 数据来源与平台

### 学术与商业数据库

这类数据库覆盖全、质量高，但通常需要机构授权：

| 数据库 | 覆盖范围 | 获取方式 |
|--------|----------|---------|
| [**Wind（万得）**](https://www.wind.com.cn) | 中国股票、债券、宏观、基金、期货，最全面 | 高校/机构授权 |
| [**CSMAR（国泰安）**](https://www.gtarsc.com) | A 股上市公司财务、公司治理、IPO | 高校授权 |
| [**RESSET（锐思）**](http://www.resset.cn) | A 股数据，部分指标与 CSMAR 互补 | 高校授权 |
| [**WRDS**](https://wrds-www.wharton.upenn.edu) | Compustat（全球财务）、CRSP（美股）、TAQ（高频）| 高校授权 |
| [**BvD Orbis**](https://www.bvdinfo.com/en-gb/our-products/data/international/orbis) | 全球私营企业财务数据 | 高校授权 |

### 公开数据平台

免费可用，适合教学、研究和入门练习：

| 平台 | 特点 | 链接 |
|------|------|------|
| **FRED**（美联储）| 美国及全球宏观经济数据，80 万+ 时间序列，有 Python API | [fred.stlouisfed.org](https://fred.stlouisfed.org) |
| **World Bank Open Data** | 全球发展指标，覆盖 200+ 国家 | [data.worldbank.org](https://data.worldbank.org) |
| **IMF Data** | 国际货币基金组织经济统计 | [imf.org/en/Data](https://www.imf.org/en/Data) |
| **Our World in Data** | 社会、经济、健康、环境议题的高质量可视化数据，全部可下载 | [ourworldindata.org](https://ourworldindata.org) |
| **国家统计局** | 中国官方宏观统计 | [stats.gov.cn](http://www.stats.gov.cn) |
| **BIS Statistics** | 国际清算银行：银行、债务、房价、汇率 | [bis.org/statistics](https://www.bis.org/statistics/) |

### 数据科学竞赛与数据集平台

学习和练手的首选，也是毕业项目找数据的好起点：

**Kaggle** — [kaggle.com](https://www.kaggle.com)

- 全球最大数据科学竞赛平台，托管 10 万+ 公开数据集
- 涵盖金融、医疗、社会科学、NLP、图像等几乎所有领域
- 内置免费 GPU/TPU 计算环境和 Notebook，无需本地配置即可运行代码
- 有活跃的社区讨论，大量优质开源 Notebook 可以直接参考

**OpenBB Terminal** — [openbb.co](https://openbb.co) | [GitHub](https://github.com/OpenBB-finance/OpenBBTerminal) ⭐ ~35k

- 开源金融数据终端，对标彭博的免费替代品
- 整合 50+ 数据源：股票、债券、加密货币、宏观经济、另类数据
- 支持 Python API，可直接在 Jupyter Notebook 中调用
- 内置 AI Copilot，可用自然语言查询数据

```python
# OpenBB 使用示例
from openbb import obb

# 获取股票历史数据
data = obb.equity.price.historical("600519.SS", start_date="2015-01-01")

# 获取宏观指标
gdp = obb.economy.gdp(countries=["china", "japan", "united_states"])
```

### API 直接获取

适合需要定制化数据获取的场景：

| 工具 | 覆盖范围 | 安装 |
|------|----------|------|
| [**yfinance**](https://github.com/ranaroussi/yfinance) | 全球股票、指数、ETF、期货（Yahoo Finance）| `pip install yfinance` |
| [**akshare**](https://akshare.akfamily.xyz) | A 股、港股、期货、宏观，国内最全免费库 | `pip install akshare` |
| [**pandas-datareader**](https://pandas-datareader.readthedocs.io) | FRED、World Bank、OECD 等多源封装 | `pip install pandas-datareader` |
| [**fredapi**](https://github.com/mortada/fredapi) | FRED 专用 Python 客户端 | `pip install fredapi` |

---

## 数据分析工具生态

### 编程语言的选择

| | Python | R | Stata |
|-|--------|---|-------|
| **定位** | 通用数据科学，AI 集成最佳 | 统计分析与可视化 | 计量经济学，学术研究 |
| **学习曲线** | 中 | 中 | 低（命令式）|
| **生态系统** | 最全，持续扩张 | 统计类最强 | 计量专用包最成熟 |
| **AI 集成** | 最好（Copilot、Agent 框架均以 Python 为核心）| 一般 | 弱 |
| **可复现性** | Jupyter/Quarto | R Markdown/Quarto | do 文件 |
| **本课程** | ✅ 主要语言 | 可选 | 可选 |

::: {.callout-tip}
已经熟悉 Stata 或 R 的同学，**不需要推倒重来**。本课程允许用任意语言提交作业。
但建议逐渐向 Python 迁移——不是因为 Python"更好"，而是因为当你需要接入 AI 工具、处理非结构化数据或构建 Agent 时，Python 的生态优势是压倒性的。
:::

### 开发环境

| 工具 | 角色 | 链接 |
|------|------|------|
| [**VS Code**](https://code.visualstudio.com) | 主编辑器，支持 Python、R、Stata、Markdown，AI 扩展丰富 | 免费 |
| [**Jupyter Notebook**](https://jupyter.org) | 交互式分析环境，代码 + 文字 + 图表三位一体 | 随 Anaconda 附带 |
| [**Quarto**](https://quarto.org) | 学术级可复现报告，支持多语言，输出 HTML/PDF/Slides | 免费，推荐 |
| [**Anaconda**](https://www.anaconda.com) | Python 发行版 + 包管理器，科学计算首选 | 免费 |

### 核心 Python 分析库（概览）

下表仅供建立整体印象，每个库在对应讲次会有专门介绍：

| 类别 | 库 | 一句话描述 |
|------|-----|----------|
| 数据处理 | [pandas](https://pandas.pydata.org) | 数据框操作的事实标准 |
| 数值计算 | [numpy](https://numpy.org) | 高性能数组运算 |
| 静态可视化 | [matplotlib](https://matplotlib.org) / [seaborn](https://seaborn.pydata.org) | 图表绘制基础库 |
| 交互可视化 | [plotly](https://plotly.com/python/) | 动态图表，适合报告展示 |
| 统计建模 | [statsmodels](https://www.statsmodels.org) | OLS、时间序列、检验等 |
| 机器学习 | [scikit-learn](https://scikit-learn.org) | 分类、回归、聚类的工业标准 |
| 因果推断 | [linearmodels](https://bashtage.github.io/linearmodels/) | 面板数据、IV 估计 |
| 金融数据 | [yfinance](https://github.com/ranaroussi/yfinance) / [akshare](https://akshare.akfamily.xyz) | 行情数据获取 |

### 版本控制：Git + GitHub

**为什么数据分析必须用版本控制？**

- **可复现性**：六个月后你能找回当时用的是哪个版本的代码和数据吗？
- **协作**：多人同时修改同一个分析文件，没有版本控制是灾难
- **回退**：改出问题时能恢复到上一个可用状态

本课程要求所有小组作业通过 GitHub 提交。不熟悉 Git 的同学请参考课程配套材料：[A2. 项目规划与版本控制](../appendix_python/a2_project.html)。

---

## AI 与 Agent：分析工具的新维度

### 两个需要理解的概念：对齐

在讨论如何使用 AI 工具之前，有两个"对齐"概念值得先理解清楚。理解它们，能让你更清醒地知道 AI 能做什么、为什么会犯某些错误、以及如何更有效地使用它。

#### 概念一：AI 对齐（AI Alignment）

**AI 对齐**指的是：确保 AI 系统的行为符合人类真正的意图和价值观，而不只是在字面上满足训练目标。

这听起来像技术问题，但对你使用 AI 做数据分析有直接的实践含义：

| AI 的常见"失对齐"表现 | 背后原因 | 对数据分析的影响 |
|----------------------|---------|----------------|
| **幻觉（Hallucination）** | 模型优化的是"输出看起来合理"，而非"输出是真的" | AI 会自信地给出错误数据、编造文献引用 |
| **迎合（Sycophancy）** | 训练数据中人类倾向于给"同意自己"的回答打高分 | AI 会倾向于支持你的假设，而非客观评估 |
| **表面完成任务** | 模型优化的是任务完成的"外观"，而非任务的真实目标 | AI 写的代码运行不报错，但逻辑是错的 |
| **过度简化** | 清晰、简洁的回答在训练中得分更高 | 复杂的统计问题被给出过度简化的答案 |

::: {.callout-important}
**关键结论：永远不要把 AI 的输出当作最终答案。**

这不是说 AI 没用——它极其有用。但它是一个需要你主动验证的**高效草稿生成器**，而不是一个可以盲目信任的权威来源。
理解对齐问题，能让你预判 AI 在哪些类型的任务上更可靠（代码格式转换、文本润色），在哪些类型上需要格外谨慎（数据事实核查、统计逻辑判断）。
:::

#### 概念二：目标对齐（Goal Alignment）

这个概念直接连接第一章的内容。

第一章强调"目标是第一位的"——但当你使用 AI 辅助分析时，会面临一个额外的挑战：**你给 AI 的指令，和你真正想解决的问题，很可能并不一致。**

例子：

> 你真正的问题：*"政府补贴政策有没有提高企业的研发投入？"*
> 你给 AI 的指令：*"帮我写一段 OLS 回归代码，因变量是研发投入，自变量是补贴金额"*

AI 会完美地完成你的指令，但你的指令本身就存在严重的内生性问题（获得补贴的企业本来就更愿意研发）。AI 不会主动提醒你这个问题——除非你明确要求它评估方法的因果识别逻辑。

**目标对齐的实践含义**：在向 AI 提问时，不只是说"做什么"，还要说"为什么做"、"用来回答什么问题"。这样 AI 才有可能在方法层面给出更有价值的建议。

### 对话型 AI vs. AI Agent

| | 对话型 AI（ChatGPT / Claude）| AI Agent |
|-|-------------------------------|----------|
| **类比** | 博学的顾问，你问他，他答你 | 能自己动手干活的助手 |
| **行为方式** | 单轮或多轮对话，每次生成文本 | 感知环境 → 规划步骤 → 调用工具 → 观察结果 → 再规划 |
| **能做什么** | 写代码、解释概念、润色文字 | 自动搜索数据、运行代码、调试错误、读取文件、生成报告 |
| **典型场景** | "帮我解释这段报错" | "分析某公司近5年财报，和同行比较，生成投资备忘录" |
| **自主程度** | 低，每步都需要人指导 | 高，可以自主分解任务并执行多步骤 |

### AI 嵌入数据分析全流程

| 分析阶段 | AI 能做什么 | 需要你把关什么 |
|----------|------------|--------------|
| **问题定义** | 帮你梳理问题框架、列出可能的分析路径、提示遗漏的变量 | 判断 AI 提供的框架是否符合你的实际场景 |
| **数据获取** | 生成 API 调用代码、爬虫脚本、数据库查询语句 | 验证代码逻辑，核实数据来源可信度 |
| **数据清洗** | 识别数据问题、生成清洗代码、解释异常值 | 确认清洗逻辑符合领域知识（如极端值是噪声还是真实信号）|
| **探索性分析** | 快速生成可视化代码、描述统计表格 | 核实图表是否准确反映数据，避免误导性可视化 |
| **建模分析** | 推荐合适模型、生成估计代码、解释系数含义 | **重点核查**：因果识别逻辑、模型假设是否成立 |
| **结果解读** | 生成结论摘要初稿、翻译术语 | 用自己的判断评估结论是否过度或不足 |
| **报告撰写** | 润色文字、格式化输出、生成摘要 | 确保最终报告是你自己理解和认可的 |

### 课程 AI 使用规范

::: {.callout-important}
**✅ 鼓励**

- 用 AI 辅助写代码、调试报错、理解概念
- 用 AI 检索文献、梳理分析思路
- 用 AI 草拟报告文字，然后人工修改

**⚠️ 必须做**

- 提交作业时附上关键提示词的原文或链接（例如：[示例格式](https://www.doubao.com/thread/w9d7da7ee6fa0bc32)）
- 对 AI 生成的代码和结论进行独立验证，再使用

**❌ 不允许**

- 不加理解地将 AI 输出直接提交
- 不标注 AI 使用情况
- 用 AI 代替对核心方法的理解（方法论部分必须自己搞懂）
:::

### 重要工具与资源

#### 日常编程助手

| 工具 | 定位 | 链接 |
|------|------|------|
| **GitHub Copilot** | IDE 内嵌代码补全，VS Code 集成最佳 | [github.com/features/copilot](https://github.com/features/copilot) |
| **Cursor** | 以 AI 为核心的代码编辑器，可对话式修改代码 | [cursor.com](https://www.cursor.com) |
| **Claude / ChatGPT** | 通用对话 AI，解释概念、写代码、调试均可 | [claude.ai](https://claude.ai) / [chatgpt.com](https://chatgpt.com) |

#### Agent 框架（由浅入深）

以下按学习曲线从低到高排列，建议按顺序探索：

**① smolagents（HuggingFace）**
[github.com/huggingface/smolagents](https://github.com/huggingface/smolagents) ⭐ ~15k

最轻量的 Agent 框架入门首选。核心理念是**代码型 Agent**：让 AI 通过写 Python 代码来解决问题，而不是调用预定义工具。接口极简，5 分钟可以跑起第一个 Agent。

```python
from smolagents import CodeAgent, HfApiModel

agent = CodeAgent(
    tools=[],
    model=HfApiModel("Qwen/Qwen2.5-Coder-32B-Instruct")
)

agent.run(
    "用 akshare 获取贵州茅台过去3年的日收益率数据，"
    "计算年化波动率，并与同期沪深300比较，画出走势对比图"
)
```

**② LangChain**
[github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain) ⭐ ~100k

最流行的 LLM 应用开发框架。适合构建需要检索外部文档（RAG）或调用多种 API 工具的复杂 Agent。学习曲线比 smolagents 陡，但生态最成熟。

适合任务：把公司年报 PDF 喂给 AI，然后做问答式分析；构建持续追踪某个数据源的自动化分析流水线。

**③ AutoGen（微软）**
[github.com/microsoft/autogen](https://github.com/microsoft/autogen) ⭐ ~40k

多 Agent 协作框架：多个 AI 角色（分析师、代码员、审核员）相互协作，分工完成复杂任务。适合需要多轮推理和交叉验证的研究任务。

**④ OpenBB Copilot**
[docs.openbb.co/copilot](https://docs.openbb.co/copilot)

金融数据分析专用 Agent，直接集成在 OpenBB Terminal 中。可以用自然语言查询金融数据、生成图表、构建分析报告，无需写代码。

#### 经济学 AI 资源列表

**Awesome AI for Economists**
[github.com/hanlulong/awesome-ai-for-economists](https://github.com/hanlulong/awesome-ai-for-economists)

专为经济学研究者策划的 AI 工具清单，覆盖：文献检索、数据获取、计量辅助、论文写作。是进入"AI 辅助经济学研究"的最佳入口，建议收藏并定期查阅。

---

## 延伸阅读与资源

### AI 与数据分析

| 资源 | 描述 | 链接 |
|------|------|------|
| Békés (2026). *Doing Data Analysis with AI* | 专门讲 AI 辅助数据分析的教材，与本课程高度相关 | [gabors-data-analysis.com/ai-course](https://gabors-data-analysis.com/ai-course/) |
| HuggingFace smolagents 文档 | Agent 框架入门，有大量示例 | [huggingface.co/docs/smolagents](https://huggingface.co/docs/smolagents) |
| Anthropic Prompt Engineering 指南 | 如何写出高质量提示词 | [docs.anthropic.com/…/prompt-engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview) |

### Python 数据分析

| 资源 | 描述 | 链接 |
|------|------|------|
| VanderPlas (2023). *Python Data Science Handbook* | 数据分析 + 可视化 + 机器学习，免费在线阅读 | [jakevdp.github.io/PythonDataScienceHandbook](https://jakevdp.github.io/PythonDataScienceHandbook/) |
| McKinney (2022). *Python for Data Analysis* (3e) | pandas 作者写的 pandas 教材 | [wesmckinney.com/book](https://wesmckinney.com/book/) |
| QuantEcon | 面向经济学家的 Python/Julia 计算经济学 | [quantecon.org/lectures](https://quantecon.org/lectures/) |

### 金融数据分析

| 资源 | 描述 | 链接 |
|------|------|------|
| Scheuch et al. (2024). *Tidy Finance with Python* | 股票回报、CAPM、Fama-French、投资组合 | [tidy-finance.org/python](https://www.tidy-finance.org/python/index.html) |
| Hilpisch (2019). *Python for Finance* | 金融建模和量化策略 | [GitHub](https://github.com/yhilpisch/py4fi2nd) |

### 因果推断

| 资源 | 描述 | 链接 |
|------|------|------|
| Huntington-Klein. *The Effect* | 因果图和反事实框架，最易读的因果推断入门 | [theeffectbook.net](https://theeffectbook.net/) |
| Facure (2022). *Causal Inference for the Brave and True* | Python 实现，含 DID、RDD、IV、匹配 | [matheusfacure.github.io/python-causality-handbook](https://matheusfacure.github.io/python-causality-handbook/landing-page.html) |

---

*上一章：[数据分析与经济决策（上）：是什么，为什么重要](ds_intro_01_intro.md)*
