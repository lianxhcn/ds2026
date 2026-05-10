# T-C3：全球 AI 独角兽公司数据整理与可视化

> 难度：⭐⭐ 中等｜类别：产业与企业数据  
> 核心工具：爬虫 / CSV 手动整理 · `pandas` · `plotly` · `folium`  
> 建议人数：7～8 人｜预计完成时间：1～2 周

---

## 一、项目背景

「AI 独角兽」指估值超过 10 亿美元、主营业务以人工智能为核心的未上市科技公司。2023-2025 年以来，全球 AI 独角兽数量激增，中美两国占据主导，但印度、以色列、英国等也涌现出一批新兴力量。

本题通过整理公开来源的全球 AI 独角兽数据，构建结构化数据库，分析估值分布、国家格局、赛道分布和融资趋势，是一道兼顾数据获取、清洗和可视化的综合题目。

---

## 二、学习目标

完成本题后，你将能够：

- 从多个公开渠道获取半结构化数据（网页、PDF、CSV 榜单）
- 处理估值等金融数值的格式清洗（「$1.2B」→ `1200`）
- 构建简单的 SQLite 关联数据库（公司表 + 融资轮次表）
- 制作地图、气泡图、树状图等高信息密度可视化

---

## 三、项目目录结构

```
T-C3_AIUnicorns/
├── readme.md
├── data_raw/
│   ├── cb_insights_raw.csv        ← CB Insights 榜单（手动下载或爬取）
│   ├── hurun_ai_raw.csv           ← 胡润 AI 独角兽榜（中国补充）
│   └── crunchbase_sample.csv      ← Crunchbase 融资数据（可选）
├── data_clean/
│   ├── unicorns_clean.csv         ← 清洗后的主数据
│   └── unicorns.db                ← SQLite 数据库
├── output/
│   ├── fig_country_distribution.html   ← 地图
│   ├── fig_valuation_bubble.png
│   ├── fig_sector_treemap.png
│   └── fig_funding_trend.png
├── 01_get_data.ipynb
├── 02_data_clean_db.ipynb
└── 03_analysis_visualization.ipynb
```

---

## 四、数据来源说明（重要）

由于 CB Insights 等榜单网站反爬虫较强，本题允许以下三种方式获取数据，**选其中一种即可**，在 notebook 中说明获取方式：

### 方式 A：手动下载 CSV（推荐，最稳定）

1. 访问 <https://www.cbinsights.com/research-unicorn-companies>（需免费注册）
2. 点击页面上的 「Download」或截图后手动整理为 CSV
3. 补充中国独角兽数据：胡润百富 AI 独角兽榜 <https://www.hurun.net/>

### 方式 B：使用现有 GitHub 数据集

已有研究者整理并公开的独角兽数据集：
- <https://github.com/datasets/unicorns>（部分，可能非最新）
- Kaggle 搜索「unicorn companies dataset」可找到 2020-2024 年多个版本

```python
# 用 pandas 直接读取 GitHub 上的 CSV
import pandas as pd
url = 'https://raw.githubusercontent.com/datasets/unicorns/main/data/unicorns.csv'
df = pd.read_csv(url)
print(df.head())
```

### 方式 C：爬取（技术挑战，选做）

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import time

# 部分信息技术类媒体会发布可爬取的 AI 独角兽列表
# 示例：爬取某新闻页面的表格
headers = {'User-Agent': 'Mozilla/5.0 (compatible; research bot)'}
url = 'https://example-tech-media.com/ai-unicorn-list'  # 替换为实际 URL
resp = requests.get(url, headers=headers, timeout=10)
soup = BeautifulSoup(resp.text, 'html.parser')

tables = pd.read_html(resp.text)  # 自动解析页面中所有 HTML 表格
if tables:
    df = tables[0]
    print(df.head())
```

---

## 五、任务分解

### 任务 1：数据获取与初步整理（`01_get_data.ipynb`）

无论使用哪种方式，目标是得到包含以下字段的数据表：

| 字段 | 说明 | 示例 |
|------|------|------|
| `company` | 公司名称 | OpenAI |
| `country` | 所在国家 | United States |
| `city` | 所在城市 | San Francisco |
| `valuation_b` | 估值（十亿美元） | 157.0 |
| `founded_year` | 成立年份 | 2015 |
| `sector` | 细分赛道 | Foundation Model |
| `investors` | 主要投资方 | Microsoft, Sequoia |
| `last_funding_date` | 最新融资时间 | 2024-03 |

```python
import pandas as pd

# 读入原始数据（按实际文件格式调整）
df = pd.read_csv('../data_raw/cb_insights_raw.csv')
print(df.shape)
print(df.dtypes)
print(df.head())

# 记录原始数据的基本信息
print(f'\n公司总数：{len(df)}')
print(f'来源国家数：{df["country"].nunique()}')
print(f'数据覆盖年份：{df["founded_year"].min()} - {df["founded_year"].max()}')
```

---

### 任务 2：数据清洗与 SQLite 数据库（`02_data_clean_db.ipynb`）

#### 2a：估值字段清洗

```python
import re
import numpy as np

def parse_valuation(s):
    """将 '$1.2B'、'1,200M'、'$12.5 billion' 等格式转为 float（单位：十亿美元）"""
    if pd.isnull(s):
        return np.nan
    s = str(s).strip().replace(',', '').replace('$', '').replace(' ', '')
    s = s.lower()
    try:
        if 'b' in s or 'billion' in s:
            return float(re.sub(r'[^\d.]', '', s))
        elif 'm' in s or 'million' in s:
            return float(re.sub(r'[^\d.]', '', s)) / 1000
        else:
            return float(re.sub(r'[^\d.]', '', s))
    except:
        return np.nan

df['valuation_b'] = df['valuation'].apply(parse_valuation)
print(f'估值解析失败：{df["valuation_b"].isnull().sum()} 条')
print(df[['company', 'valuation', 'valuation_b']].head(10))
```

#### 2b：国家名称标准化

```python
country_map = {
    'US': 'United States',
    'USA': 'United States',
    'U.S.': 'United States',
    'UK': 'United Kingdom',
    'U.K.': 'United Kingdom',
    '中国': 'China',
    # ……根据实际数据补充
}
df['country_std'] = df['country'].replace(country_map)
```

#### 2c：构建 SQLite 数据库

```python
import sqlite3

df_clean = df.dropna(subset=['company', 'valuation_b']).copy()
df_clean.to_csv('../data_clean/unicorns_clean.csv', index=False)

# 建立 SQLite 数据库
conn = sqlite3.connect('../data_clean/unicorns.db')

# 主表：公司信息
df_companies = df_clean[[
    'company', 'country_std', 'city', 'valuation_b',
    'founded_year', 'sector', 'last_funding_date'
]].copy()
df_companies.to_sql('companies', conn,
                    if_exists='replace', index=False)

# 简单查询验证
result = pd.read_sql('''
    SELECT country_std, COUNT(*) as count,
           ROUND(AVG(valuation_b), 2) as avg_valuation_b
    FROM companies
    GROUP BY country_std
    ORDER BY count DESC
    LIMIT 10
''', conn)
print(result)
conn.close()
```

---

### 任务 3：分析与可视化（`03_analysis_visualization.ipynb`）

#### 图 1：各国 AI 独角兽数量地图（plotly）

```python
import plotly.express as px
import pandas as pd

df = pd.read_csv('../data_clean/unicorns_clean.csv')

country_stats = df.groupby('country_std').agg(
    count=('company', 'count'),
    total_val=('valuation_b', 'sum'),
    avg_val=('valuation_b', 'mean')
).reset_index()

fig = px.choropleth(
    country_stats,
    locations='country_std',
    locationmode='country names',
    color='count',
    hover_name='country_std',
    hover_data={'total_val': ':.1f', 'avg_val': ':.2f'},
    color_continuous_scale='Blues',
    title='全球 AI 独角兽数量分布',
    labels={'count': '独角兽数量', 'total_val': '估值总计（B$）'}
)
fig.write_html('../output/fig_country_distribution.html')
fig.show()
```

#### 图 2：Top 20 公司估值气泡图

```python
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker

top20 = df.nlargest(20, 'valuation_b').sort_values('valuation_b', ascending=True)

fig, ax = plt.subplots(figsize=(10, 8))
colors = {'United States': '#2980b9', 'China': '#c0392b',
          'United Kingdom': '#27ae60'}
bar_colors = [colors.get(c, '#95a5a6') for c in top20['country_std']]

bars = ax.barh(top20['company'], top20['valuation_b'],
               color=bar_colors, alpha=0.85)

ax.set_xlabel('估值（十亿美元）')
ax.set_title('全球 AI 独角兽估值 Top 20', fontsize=13)

# 添加数值标签
for bar, val in zip(bars, top20['valuation_b']):
    ax.text(bar.get_width() + 1, bar.get_y() + bar.get_height() / 2,
            f'${val:.1f}B', va='center', fontsize=9)

from matplotlib.patches import Patch
legend_elements = [Patch(color=c, label=n) for n, c in colors.items()]
ax.legend(handles=legend_elements, loc='lower right')
plt.tight_layout()
plt.savefig('../output/fig_valuation_bubble.png', dpi=150)
```

#### 图 3：赛道分布树状图（Treemap）

```python
sector_stats = df.groupby('sector').agg(
    count=('company', 'count'),
    total_val=('valuation_b', 'sum')
).reset_index()

fig = px.treemap(
    sector_stats,
    path=['sector'],
    values='count',
    color='total_val',
    color_continuous_scale='RdYlBu',
    title='AI 独角兽赛道分布（按公司数量，颜色=估值总量）',
    labels={'total_val': '估值合计（B$）', 'count': '公司数量'}
)
fig.write_html('../output/fig_sector_treemap.html')
fig.show()
```

---

## 六、结果解读指引

- 中美两国各占全球 AI 独角兽的多少比例？估值占比与数量占比是否一致？
- 哪个细分赛道的独角兽最多？哪个赛道平均估值最高？
- 近 5 年成立的 AI 独角兽与更早成立的相比，估值中位数有何差异？
- 如果数据存在缺失或偏差（如 CB Insights 对非英语市场覆盖不足），这对结论有什么影响？

---

## 七、拓展方向（选做）

- 加入融资轮次数据，分析「天使轮→A 轮→B 轮→……」各阶段的估值分布变化
- 对投资方字段做文本拆分，统计最活跃的 VC/PE 机构，制作投资机构排名
- 将中国独角兽数据与胡润榜单交叉比对，分析两个来源的覆盖差异

---

## 八、AI 辅助提示词

**估值字段清洗：**
```
我有一列估值数据，格式混乱，包括 '$1.2B'、'1,200M'、'12.5 billion'、
'$12,500,000,000' 等多种写法，还有一些是 NaN。
请帮我写一个 Python 函数，将所有格式统一转换为浮点数（单位：十亿美元），
无法解析的返回 NaN，并说明每种格式的处理逻辑。
```

**SQLite 建表：**
```
我有一个关于 AI 独角兽公司的 DataFrame，字段有：
company, country, valuation_b, founded_year, sector, investors（逗号分隔字符串）
请帮我设计一个规范的 SQLite 数据库结构：
1. companies 表（主表）
2. investors 表（投资机构表）
3. company_investor 表（关联表，多对多关系）
并给出建表和数据写入的完整代码。
```

---

## 九、参考资源

- CB Insights 独角兽榜单：<https://www.cbinsights.com/research-unicorn-companies>
- 胡润百富：<https://www.hurun.net/zh-CN/Rank/HuList>
- Kaggle 独角兽数据集搜索：<https://www.kaggle.com/search?q=unicorn+companies>
- Plotly Treemap 文档：<https://plotly.com/python/treemaps/>
- 课程参考章节：ds2026 第 13 章（数据管理格式与存储）、第 14 章（数据清洗）

---

*最后更新：2026-05-10*
