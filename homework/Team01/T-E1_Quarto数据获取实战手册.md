# T-E1：用 Quarto book 写「Python 数据获取实战手册」

> 难度：⭐⭐⭐ 较难（管理难度高）｜类别：Quarto + AI Agent 在线书/报告  
> 核心工具：`Quarto` · `GitHub Actions` · `GitHub Pages` · 各数据获取工具  
> 建议人数：7～8 人｜预计完成时间：2 周

---

## 一、项目背景

Quarto 是新一代科学与技术出版系统，支持将 `.qmd` 文件（Markdown + 代码）渲染为精美的 HTML 网页、PDF、Word 文档甚至整本在线书籍。通过 GitHub Actions，每次提交代码后可自动触发构建并发布到 GitHub Pages，实现「写完即发布」的现代技术写作流程。

本题要求小组成员分工合作，撰写一本《Python 数据获取实战手册》，每章对应一种数据获取方法，形成完整的在线参考书，供全班同学在后续学习中使用。

---

## 二、学习目标

完成本题后，你将能够：

- 使用 Quarto 创建多章节在线书籍（`quarto book`）
- 配置 GitHub Actions 实现自动化构建与发布
- 以「主编 + 章节作者」模式协作完成技术文档
- 掌握 `.qmd` 格式：Markdown + Python 代码块 + 交叉引用
- 体验「代码即文档」的现代技术写作流程

---

## 三、书籍目录结构（建议）

小组可根据兴趣和分工调整各章主题：

| 章节 | 建议主题 | 负责人 |
|------|----------|--------|
| 第 1 章 | 数据获取概览：工具选择指南 | 全组合作 |
| 第 2 章 | akshare：A 股与宏观数据 | 成员 A |
| 第 3 章 | yfinance / OpenBB：美股与全球市场 | 成员 B |
| 第 4 章 | FRED API：美联储宏观数据 | 成员 C |
| 第 5 章 | World Bank API：跨国经济数据 | 成员 D |
| 第 6 章 | 网络爬虫入门：requests + BeautifulSoup | 成员 E |
| 第 7 章 | 数据清洗常见坑与解决方案 | 成员 F |
| 第 8 章 | 数据存储：CSV、SQLite、Parquet 的选择 | 成员 G |
| 附录 | 常用 AI 提示词模板 | 全组合作 |

---

## 四、项目目录结构

```
T-E1_DataAcquisitionBook/
├── readme.md                    ← 项目说明、分工表、预览链接
├── _quarto.yml                  ← Quarto 书籍配置文件（核心）
├── index.qmd                    ← 封面/前言
├── chapters/
│   ├── 01_overview.qmd
│   ├── 02_akshare.qmd
│   ├── 03_yfinance.qmd
│   ├── 04_fred.qmd
│   ├── 05_worldbank.qmd
│   ├── 06_crawler.qmd
│   ├── 07_data_cleaning.qmd
│   └── 08_data_storage.qmd
├── appendix/
│   └── ai_prompts.qmd
├── data/                        ← 示例数据（小文件）
├── .github/
│   └── workflows/
│       └── publish.yml          ← GitHub Actions 自动发布配置
└── _freeze/                     ← Quarto 缓存（自动生成，加入 .gitignore）
```

---

## 五、关键配置文件

### `_quarto.yml`（书籍配置）

```yaml
project:
  type: book
  output-dir: _book

book:
  title: "Python 数据获取实战手册"
  author: "第 X 组 · ds2026"
  date: "2026-05-10"
  chapters:
    - index.qmd
    - chapters/01_overview.qmd
    - chapters/02_akshare.qmd
    - chapters/03_yfinance.qmd
    - chapters/04_fred.qmd
    - chapters/05_worldbank.qmd
    - chapters/06_crawler.qmd
    - chapters/07_data_cleaning.qmd
    - chapters/08_data_storage.qmd
    - appendix/ai_prompts.qmd

format:
  html:
    theme: cosmo
    code-fold: false
    code-tools: true
    toc: true
    number-sections: true
  pdf:
    documentclass: scrreprt
    papersize: a4

execute:
  freeze: auto          # 只在代码变化时重新执行
  cache: true
  echo: true
  warning: false
```

### `.github/workflows/publish.yml`（自动发布）

```yaml
name: Quarto Publish

on:
  push:
    branches: [main]

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install akshare yfinance pandas matplotlib seaborn \
                      requests beautifulsoup4 fredapi wbdata

      - name: Install Quarto
        uses: quarto-dev/quarto-actions/setup@v2

      - name: Render and Publish
        uses: quarto-dev/quarto-actions/publish@v2
        with:
          target: gh-pages
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 六、章节写作规范（`.qmd` 文件模板）

每个章节作者须遵循以下结构：

````markdown
# 第 X 章：章节标题

::: {.callout-note}
**本章作者**：姓名（学号）  
**完成日期**：YYYY-MM-DD  
**主要内容**：一句话说明本章内容
:::

## 工具简介

介绍本章涉及的数据获取工具：是什么、解决什么问题、与其他工具相比的优劣。

## 安装与准备

```python
# 安装
# pip install xxx

# 导入
import xxx
```

## 基础用法

### 获取 XXX 数据

```{python}
#| label: fig-basic-demo
#| fig-cap: "示例图表标题"

import akshare as ak  # 替换为实际工具
import matplotlib.pyplot as plt

# 获取数据
data = ak.xxx_function(...)
print(data.head())

# 可视化
fig, ax = plt.subplots()
ax.plot(data.index, data.iloc[:, 0])
plt.show()
```

上图展示了……（2-3句解读）

## 进阶用法

……

## 常见问题与坑

| 问题 | 原因 | 解决方法 |
|------|------|----------|
| 接口返回空数据 | … | … |
| 中文字符乱码 | … | … |

## 本章小结

本章介绍了……，主要函数包括……，适用场景是……

## 参考资源

- 官方文档：
- 相关推文：
````

---

## 七、组内协作流程

### 分工建议

| 角色 | 职责 |
|------|------|
| 主编（1 人） | 维护 `_quarto.yml`、协调格式统一、最终合并审核 |
| 章节作者（6 人） | 各自负责 1～2 章，独立分支开发 |
| DevOps（可与主编重合） | 配置 GitHub Actions，确保自动发布正常 |

### Git 协作流程

```bash
# 1. 每位成员在自己的分支上工作
git checkout -b chapter/02-akshare

# 2. 写好章节后提交
git add chapters/02_akshare.qmd
git commit -m "feat: 完成第2章 akshare 基础用法"
git push origin chapter/02-akshare

# 3. 在 GitHub 上创建 Pull Request，@主编 进行审核
# 4. 主编审核通过后合并到 main，自动触发发布

# 本地预览（在合并前检查效果）
quarto preview chapters/02_akshare.qmd
# 或预览整本书
quarto preview
```

---

## 八、评分重点

| 维度 | 占比 | 说明 |
|------|------|------|
| 内容质量 | 35% | 每章代码可运行、说明清晰、有实质内容 |
| 格式规范 | 15% | 章节结构一致、图表有标注、引用正确 |
| 发布效果 | 20% | GitHub Pages 正常访问，目录导航正常 |
| 分工公平 | 15% | readme 中的分工说明与实际 commit 记录对应 |
| 拓展内容 | 15% | 附录完整，有超出基础要求的内容 |

---

## 九、AI 辅助提示词

**Quarto 配置问题：**
```
我在用 Quarto book 模式，执行 quarto render 时报错：
[粘贴错误信息]
我的 _quarto.yml 配置如下：
[粘贴配置]
请帮我找出问题并修复。
```

**章节内容生成：**
```
我在写一本关于 Python 数据获取的技术手册，第 X 章主题是「{工具名}」。
请帮我生成该章的大纲，包括：工具简介、安装方法、3 个由浅入深的示例、
常见错误、本章小结。要求面向有 Python 基础但不熟悉该工具的读者。
```

**GitHub Actions 调试：**
```
我的 GitHub Actions workflow 在 Quarto publish 步骤失败，
错误信息是：[粘贴错误]
我的 publish.yml 内容是：[粘贴配置]
请帮我找出原因。
```

---

## 十、参考资源

- Quarto 官方文档（Books）：<https://quarto.org/docs/books/>
- Quarto GitHub Actions 发布指南：<https://quarto.org/docs/publishing/github-pages.html>
- 优秀 Quarto book 示例：<https://r4ds.hadley.nz/>（R for Data Science，参考其写作风格）
- ds2026 课程网站本身就是 Quarto book，可参考其源码：<https://github.com/lianxhcn/ds2026>

---

*最后更新：2026-05-10*
