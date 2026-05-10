# T-E4：AI Agent + Quarto 撰写行业年度数据报告

> 难度：⭐⭐⭐ 中等偏难｜类别：Quarto + AI Agent 在线书/报告  
> 核心工具：`Quarto` · AI Agent（Claude / GPT）· 自选数据源  
> 建议人数：7～8 人｜预计完成时间：2 周

---

## 一、项目背景

行业研究报告是金融机构、咨询公司、政府部门最常见的数据产品之一。一份好的行业报告应当：有翔实的数据支撑、有清晰的可视化、有有见地的分析文字——而不是数据和图表的简单堆砌。

本题要求小组自选一个感兴趣的行业，综合运用数据获取、AI 辅助写作、Quarto 排版三种能力，产出一份结构完整、图文并茂的行业年度数据报告，发布为在线可访问的 HTML 书和可下载的 PDF。

---

## 二、学习目标

完成本题后，你将能够：

- 从零开始规划一份行业报告的框架和数据需求
- 综合使用多个数据源，构建支撑报告的数据集
- 用 AI 辅助撰写分析文字，并学会「审稿」AI 输出
- 使用 Quarto 制作专业的多格式报告（HTML + PDF）
- 理解「数据→洞见→叙事」的报告写作逻辑

---

## 三、行业选题建议

小组可从以下方向选择，也可自拟（需经老师确认）：

| 行业方向 | 可获取的核心数据 | 推荐数据源 |
|----------|------------------|------------|
| 新能源汽车 | 月度销量、渗透率、各省补贴、龙头股价 | 乘联会 / akshare |
| 半导体 / 芯片 | A 股半导体指数、进出口数据、全球厂商营收 | akshare / FRED / yfinance |
| 医药 / 创新药 | CDE 批件数量、集采中标价、医保目录 | 爬虫 / akshare |
| 消费品 | 社消零售分项、主要品牌营收、通胀传导 | 统计局 / akshare |
| 人工智能产业 | 融资数据、专利数量、算力指标、独角兽榜 | CB Insights / 爬虫 |
| 房地产 | 70 城房价、商品房销售面积、土地出让金 | 统计局 / akshare |
| 跨境电商 | 海关进出口、平台 GMV、汇率影响 | 海关总署 / akshare |
| 自选行业 | — | — |

---

## 四、项目目录结构

```
T-E4_IndustryReport_[行业名]/
├── readme.md                    ← 选题说明、数据来源声明、分工表
├── _quarto.yml                  ← Quarto 配置（book 或 article）
├── index.qmd                    ← 执行摘要（Executive Summary）
├── chapters/
│   ├── 01_industry_overview.qmd ← 行业基本面：定义、规模、格局
│   ├── 02_market_size.qmd       ← 市场规模与增速分析
│   ├── 03_competitive.qmd       ← 竞争格局与主要玩家
│   ├── 04_policy.qmd            ← 政策环境与监管动态
│   ├── 05_financials.qmd        ← 上市公司财务数据分析（如适用）
│   └── 06_outlook.qmd           ← 展望与关键变量
├── data_raw/                    ← 原始数据
├── data_clean/                  ← 清洗后的数据
├── scripts/
│   ├── fetch_data.py            ← 数据获取脚本
│   └── ai_draft.py              ← AI 辅助生成初稿脚本
├── output/
│   ├── report.html
│   └── report.pdf
└── .github/workflows/publish.yml
```

---

## 五、任务分解

### 任务 1：报告规划（`readme.md` + 第一次组会）

在开始写作前，全组须先达成以下共识（写入 readme）：

```markdown
## 报告规划

### 行业选题
行业名称：XXX  
选择理由：（1-2句，说明与本组背景或兴趣的关联）  
报告时间范围：2020-2024（或根据数据可用性调整）

### 核心研究问题
1. 该行业近 5 年的规模增长速度如何？增长驱动力是什么？
2. 市场竞争格局是集中还是分散？头部企业的优势如何？
3. 政策对行业的影响：最重要的 3 个政策节点是什么？
4. 未来 1-2 年，哪些变量最值得关注？

### 数据来源清单
| 数据类型 | 来源 | 获取方式 | 负责人 |
|---------|------|---------|--------|
| 行业销量数据 | 乘联会官网 | 手动下载 CSV | 成员 A |
| 上市公司股价 | akshare | Python API | 成员 B |
| 财务数据 | 同花顺/东方财富 | akshare API | 成员 C |
| 政策文件日期 | 政府网站 | 手动整理 | 成员 D |

### 最低图表要求（必须完成）
- [ ] 行业规模趋势图（折线图，近 5 年）
- [ ] 市场份额饼图或条形图（最新年度）
- [ ] 主要上市公司股价走势对比图
- [ ] 财务指标对比表（营收、净利润、毛利率）
- [ ] 政策事件时间轴
- [ ] 至少一张地图（地区分布）或热力图（季节性）
- [ ] 自选 2 张（体现行业特色的创意可视化）
```

---

### 任务 2：数据获取脚本（`scripts/fetch_data.py`）

```python
"""
行业数据获取脚本 — 以新能源汽车为例
其他行业请参考此结构，替换具体接口
"""
import akshare as ak
import yfinance as yf
import pandas as pd
import os

OUTPUT_DIR = '../data_raw'
os.makedirs(OUTPUT_DIR, exist_ok=True)

def fetch_ev_monthly_sales():
    """新能源汽车月度销量（乘联会数据，通过 akshare 获取）"""
    try:
        # 注意：实际接口名需在 akshare 文档中确认
        df = ak.car_sale_rank_mg()
        df.to_csv(f'{OUTPUT_DIR}/ev_sales_raw.csv', index=False)
        print(f'✓ 销量数据：{len(df)} 条')
    except Exception as e:
        print(f'✗ 销量数据获取失败：{e}')

def fetch_ev_stock_prices():
    """新能源汽车板块上市公司股价"""
    # 主要新能源汽车相关股票
    stocks = {
        '002594.SZ': '比亚迪',
        '600438.SH': '通威股份',
        '300750.SZ': '宁德时代',
        'TSLA':      'Tesla（美股）',
        'XPEV':      '小鹏（美股）',
        'LI':        '理想（美股）',
    }
    prices = {}
    for code, name in stocks.items():
        try:
            if '.' in code:  # A 股
                df = ak.stock_zh_a_hist(
                    symbol=code.split('.')[0],
                    period='monthly', adjust='hfq',
                    start_date='20200101'
                )
            else:  # 美股
                df = yf.download(code, start='2020-01-01',
                                  interval='1mo', progress=False)
            prices[name] = df
            print(f'✓ {name}：{len(df)} 条')
        except Exception as e:
            print(f'✗ {name} 失败：{e}')

    # 合并收盘价
    close_data = {}
    for name, df in prices.items():
        if '收盘' in df.columns:
            close_data[name] = df['收盘']
        elif 'Close' in df.columns:
            close_data[name] = df['Close']

    pd.DataFrame(close_data).to_csv(f'{OUTPUT_DIR}/ev_stock_prices.csv')

if __name__ == '__main__':
    fetch_ev_monthly_sales()
    fetch_ev_stock_prices()
    print('\n数据获取完成，文件保存至 data_raw/')
```

---

### 任务 3：AI 辅助写作（`scripts/ai_draft.py`）

```python
"""
使用 AI 生成报告各章节初稿
注意：AI 生成的内容必须经过人工审核和修改，不允许直接使用
"""
import anthropic
import json
import os

client = anthropic.Anthropic(api_key=os.environ.get('ANTHROPIC_API_KEY'))

def generate_section_draft(section_title: str,
                           data_summary: str,
                           charts_description: str) -> str:
    """根据数据摘要和图表描述，生成报告章节初稿"""

    prompt = f"""
你是一位专业的行业分析师，正在撰写一份行业年度数据报告。
请根据以下信息，为「{section_title}」这一章节生成 400-600 字的分析文字。

数据摘要：
{data_summary}

图表描述（你需要在文中引用这些图表的结论）：
{charts_description}

写作要求：
1. 语言专业客观，避免夸张或过度乐观
2. 每个重要结论须有数据支撑（直接引用上方数据摘要中的具体数字）
3. 段落结构：现状描述→原因分析→趋势判断
4. 不要捏造上方未提供的数据
5. 最后一段指出本章分析的局限性（如数据时效性、口径差异等）
6. 直接输出正文，不需要章节标题
"""

    response = client.messages.create(
        model='claude-sonnet-4-20250514',
        max_tokens=1000,
        messages=[{'role': 'user', 'content': prompt}]
    )
    return response.content[0].text


# 使用示例
if __name__ == '__main__':
    # 以市场规模章节为例
    data_summary = """
    - 2024 年中国新能源汽车销量为 XXX 万辆，同比增长 XX%
    - 市场渗透率从 2020 年的 5.8% 提升至 2024 年的 XX%
    - 纯电动（BEV）占比 XX%，插电混动（PHEV）占比 XX%
    """
    charts_description = """
    - 图 1：2018-2024 年月度销量折线图，2022 年后呈现加速增长趋势
    - 图 2：2024 年各省新能源汽车渗透率地图，广东、浙江、上海领先
    """
    draft = generate_section_draft('市场规模分析', data_summary, charts_description)
    print(draft)

    # 保存草稿供人工审核
    with open('../data_clean/ai_draft_market_size.txt', 'w', encoding='utf-8') as f:
        f.write(draft)
```

**AI 写作使用规范**（必须在 readme 中声明）：

```markdown
## AI 工具使用声明

本报告使用 Claude/GPT 辅助生成了以下内容的初稿：
- 第 X 章市场规模分析（初稿生成后经全组讨论修改）
- 第 X 章政策分析（AI 生成结构，数据由小组补充验证）

以下内容为小组独立完成：
- 所有数据获取和清洗代码
- 所有图表的设计和实现
- 所有具体数据的核实和修正
- 最终结论和展望（第 6 章）

AI 生成内容的审核流程：
1. 核对所有数字是否与原始数据一致
2. 删除 AI 捏造的具体案例或未经核实的说法
3. 补充小组自己的行业观察和分析
```

---

### 任务 4：Quarto 报告配置（支持中文 PDF）

````yaml
# _quarto.yml
project:
  type: book
  output-dir: _book

book:
  title: "XXX 行业年度数据报告（2024）"
  author: "ds2026 第 X 组"
  date: "2026-05"
  chapters:
    - index.qmd
    - chapters/01_industry_overview.qmd
    - chapters/02_market_size.qmd
    - chapters/03_competitive.qmd
    - chapters/04_policy.qmd
    - chapters/05_financials.qmd
    - chapters/06_outlook.qmd

format:
  html:
    theme: cosmo
    toc: true
    number-sections: true
    fig-width: 10
    fig-height: 6
  pdf:
    documentclass: ctexart    # 支持中文的文档类
    papersize: a4
    geometry: "margin=2.5cm"
    fig-pos: H
    number-sections: true
    colorlinks: true

execute:
  echo: false       # PDF/HTML 中隐藏代码，只显示输出
  warning: false
  freeze: auto
````

---

## 六、评分重点（与其他题目的差异）

本题特别强调**分析深度**，防止「数据多但分析少」：

| 维度 | 占比 | 评分说明 |
|------|------|----------|
| 数据完整性 | 20% | 图表数量 ≥ 8 张，覆盖报告承诺的核心问题 |
| 分析深度 | 30% | 每章有实质性结论，不止于描述数字 |
| 数据真实性 | 20% | AI 生成内容须经人工核实，无捏造数据 |
| 排版质量 | 15% | PDF 和 HTML 均可正常渲染，图表标签完整 |
| AI 使用规范 | 15% | 明确声明 AI 使用范围，审核流程有记录 |

---

## 七、AI 辅助提示词

**报告框架设计：**
```
我要写一份关于「中国新能源汽车行业」的年度数据报告（2020-2024）。
请帮我设计一个完整的报告框架，包括：
- 6-8 个章节的标题和主要内容
- 每章需要的核心数据（具体到指标名称）
- 每章建议的图表类型（不超过 2 张/章）
- 数据可能来自哪些公开来源
```

**分析文字审核：**
```
以下是 AI 生成的行业分析段落：
[粘贴 AI 输出]
以下是我核实后的实际数据：
[粘贴真实数据]
请帮我：
1. 找出 AI 输出中与真实数据不符的地方
2. 修改为符合真实数据的版本
3. 补充 AI 遗漏但数据中包含的重要信息
```

**Quarto 中文 PDF 问题：**
```
我用 Quarto 渲染 PDF，报告中的中文全部显示为方块字。
我的 _quarto.yml 使用了 documentclass: ctexart。
错误信息是：[粘贴错误]
请告诉我如何解决中文 PDF 渲染问题（macOS/Windows/Linux 环境）。
```

---

## 八、参考资源

- 行业报告参考（结构）：各大券商研究报告（在官网免费获取）
- Quarto 中文支持：<https://quarto.org/docs/output-formats/pdf-basics.html>
- CTeX 文档类（中文 PDF）：<https://ctan.org/pkg/ctex>
- 报告写作逻辑参考：麦肯锡「金字塔原理」（图书馆可借）
- 课程参考章节：ds2026 全部前 14 章

---

*最后更新：2026-05-10*
