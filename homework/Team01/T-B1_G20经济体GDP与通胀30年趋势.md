# T-B1：G20 经济体 GDP 增速与通胀 30 年趋势

> 难度：⭐⭐ 中等｜类别：宏观经济与国际数据  
> 核心工具：`wbdata` · `pandas` · `matplotlib` · `plotly`（可选）  
> 建议人数：7～8 人｜预计完成时间：1～2 周

---

## 一、项目背景

世界银行（World Bank）是全球最权威的宏观经济数据库之一，提供 200 多个国家、1000 多个指标的历史数据，涵盖 GDP、通胀、人口、贸易、教育等维度。`wbdata` 是其官方 Python 接口，可直接在代码中拉取数据，无需注册。

本题选取 G20 主要经济体，获取近 30 年的 GDP 增速、CPI 通胀率和失业率数据，清洗合并后制作多维度跨国对比可视化，探索不同经济体「增长—通胀—就业」三角关系的历史演变。

---

## 二、学习目标

完成本题后，你将能够：

- 使用 `wbdata` / `pandas_datareader` 批量获取多国多指标数据
- 处理典型的面板数据（国家 × 时间 × 指标）格式转换
- 识别并处理跨国数据中常见的缺失值和异常值
- 制作「热力图」「多国折线图」「散点图动画」等高信息密度可视化
- 撰写跨国比较分析的结构化报告

---

## 三、项目目录结构

```
T-B1_G20MacroTrends/
├── readme.md
├── data_raw/
│   ├── gdp_growth_raw.csv         ← GDP 增速原始数据
│   ├── inflation_raw.csv          ← CPI 通胀率原始数据
│   └── unemployment_raw.csv       ← 失业率原始数据
├── data_clean/
│   └── g20_macro_panel.csv        ← 合并后的面板数据（长表格式）
├── output/
│   ├── fig_gdp_heatmap.png
│   ├── fig_gdp_trend_major.png
│   ├── fig_inflation_trend.png
│   └── fig_growth_inflation_scatter.png
├── 01_get_data.ipynb
├── 02_data_clean.ipynb
└── 03_analysis_visualization.ipynb
```

---

## 四、G20 国家代码表

使用 World Bank API 时，国家须使用 ISO 2 位或 3 位代码：

| 国家 | 代码 | 国家 | 代码 |
|------|------|------|------|
| 中国 | CHN | 美国 | USA |
| 日本 | JPN | 德国 | DEU |
| 英国 | GBR | 法国 | FRA |
| 印度 | IND | 巴西 | BRA |
| 加拿大 | CAN | 韩国 | KOR |
| 澳大利亚 | AUS | 俄罗斯 | RUS |
| 意大利 | ITA | 墨西哥 | MEX |
| 印度尼西亚 | IDN | 土耳其 | TUR |
| 沙特阿拉伯 | SAU | 阿根廷 | ARG |
| 南非 | ZAF | 欧盟 | EUU |

---

## 五、任务分解

### 任务 1：数据获取（`01_get_data.ipynb`）

**常用 World Bank 指标代码**：

| 指标 | 代码 |
|------|------|
| GDP 增速（年度，%） | `NY.GDP.MKTP.KD.ZG` |
| CPI 通胀率（年度，%） | `FP.CPI.TOTL.ZG` |
| 失业率（%） | `SL.UEM.TOTL.ZS` |
| 人均 GDP（现价美元） | `NY.GDP.PCAP.CD` |
| 经常账户差额（占 GDP %） | `BN.CAB.XOKA.GD.ZS` |

```python
import wbdata
import pandas as pd
from datetime import datetime

# 定义国家列表
G20 = ['CHN','USA','JPN','DEU','GBR','FRA','IND','BRA',
       'CAN','KOR','AUS','RUS','ITA','MEX','IDN','TUR',
       'SAU','ARG','ZAF']

# 定义指标
indicators = {
    'NY.GDP.MKTP.KD.ZG': 'gdp_growth',
    'FP.CPI.TOTL.ZG':    'inflation',
    'SL.UEM.TOTL.ZS':    'unemployment',
    'NY.GDP.PCAP.CD':    'gdp_per_capita',
}

# 下载数据（1990-2023）
df = wbdata.get_dataframe(indicators, country=G20, parse_dates=True)
print(df.shape)
print(df.head(10))

# 保存原始数据
df.to_csv('../data_raw/wb_raw.csv')
```

> **如果 wbdata 安装失败**，可改用 `pandas_datareader`：
> ```python
> import pandas_datareader.wb as wb
> df = wb.download(indicator='NY.GDP.MKTP.KD.ZG',
>                  country=G20, start=1990, end=2023)
> ```

---

### 任务 2：数据清洗（`02_data_clean.ipynb`）

`wbdata` 返回的数据是「国家 × 年份」的 MultiIndex DataFrame，需要重塑为分析友好的格式。

```python
import pandas as pd
import numpy as np

df = pd.read_csv('../data_raw/wb_raw.csv', index_col=[0, 1])
print('原始格式：')
print(df.head())
print('\n索引层级：', df.index.names)

# 重置索引，转为长表
df_long = df.reset_index()
df_long.columns = ['country', 'date'] + list(df.columns)
df_long['date'] = pd.to_datetime(df_long['date'])
df_long['year'] = df_long['date'].dt.year

# 检查每个国家的缺失情况
missing_summary = df_long.groupby('country')[['gdp_growth', 'inflation']].apply(
    lambda x: x.isnull().sum()
)
print('\n各国缺失值统计：')
print(missing_summary)

# 筛选有效范围：1993-2023（部分转轨经济体早期数据缺失，合理截断）
df_clean = df_long[df_long['year'].between(1993, 2023)].copy()

# 对缺失值记录说明（不盲目填充）
# 极端值检查：通胀率 > 100% 或 < -10% 标记为异常
df_clean['inflation_flag'] = df_clean['inflation'].apply(
    lambda x: 'extreme' if pd.notnull(x) and abs(x) > 100 else 'normal'
)
print('\n极端通胀记录：')
print(df_clean[df_clean['inflation_flag'] == 'extreme'][
    ['country', 'year', 'inflation']
])

df_clean.to_csv('../data_clean/g20_macro_panel.csv', index=False)
print(f'\n清洗完成：{len(df_clean)} 条记录')
```

---

### 任务 3：分析与可视化（`03_analysis_visualization.ipynb`）

#### 图 1：G20 GDP 增速热力图（国家 × 年份）

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv('../data_clean/g20_macro_panel.csv')

# 构建宽表：行=国家，列=年份
gdp_wide = df.pivot_table(index='country', columns='year',
                           values='gdp_growth')

# 选取主要国家和近 20 年
major = ['CHN','USA','JPN','DEU','GBR','FRA','IND','BRA','KOR','AUS']
gdp_plot = gdp_wide.loc[major, 2003:2023]

# 国家代码映射为中文名（可选）
name_map = {'CHN':'中国','USA':'美国','JPN':'日本','DEU':'德国',
            'GBR':'英国','FRA':'法国','IND':'印度','BRA':'巴西',
            'KOR':'韩国','AUS':'澳大利亚'}
gdp_plot.index = [name_map.get(c, c) for c in gdp_plot.index]

fig, ax = plt.subplots(figsize=(14, 5))
sns.heatmap(gdp_plot, annot=True, fmt='.1f', cmap='RdYlGn',
            center=0, ax=ax, linewidths=0.3,
            cbar_kws={'label': 'GDP 增速 (%)'},
            annot_kws={'size': 8})
ax.set_title('主要经济体 GDP 增速热力图（2003-2023，%）', fontsize=13)
ax.set_xlabel('')
ax.set_ylabel('')
plt.tight_layout()
plt.savefig('../output/fig_gdp_heatmap.png', dpi=150)
```

#### 图 2：中美印三国 GDP 增速对比

```python
three = df[df['country'].isin(['CHN', 'USA', 'IND'])].copy()
three['name'] = three['country'].map({'CHN':'中国','USA':'美国','IND':'印度'})

fig, ax = plt.subplots(figsize=(12, 5))
colors = {'中国': '#c0392b', '美国': '#2980b9', '印度': '#27ae60'}
for name, group in three.groupby('name'):
    ax.plot(group['year'], group['gdp_growth'],
            label=name, color=colors[name], linewidth=2, marker='o', markersize=3)

ax.axhline(0, color='black', linewidth=0.6, linestyle='--', alpha=0.5)
ax.set_title('中美印 GDP 年增速对比（1993-2023）', fontsize=13)
ax.set_xlabel('年份')
ax.set_ylabel('GDP 增速（%）')
ax.legend()

# 标注金融危机和新冠
for year, label in [(2009, '全球金融危机'), (2020, 'COVID-19')]:
    ax.axvline(year, color='gray', linewidth=1, linestyle=':', alpha=0.7)
    ax.text(year + 0.1, ax.get_ylim()[1] * 0.9, label,
            fontsize=8, color='gray', rotation=90)

plt.tight_layout()
plt.savefig('../output/fig_gdp_trend_major.png', dpi=150)
```

#### 图 3：「菲利普斯曲线」——增速与通胀散点图

```python
# 选取近 10 年数据，G20 各国每年一个点
recent = df[df['year'].between(2013, 2023)].dropna(
    subset=['gdp_growth', 'inflation']
)
# 去除极端通胀（阿根廷等）
recent = recent[recent['inflation'].abs() < 50]

fig, ax = plt.subplots(figsize=(9, 6))
scatter = ax.scatter(recent['gdp_growth'], recent['inflation'],
                     c=recent['year'], cmap='viridis',
                     alpha=0.6, s=40)
plt.colorbar(scatter, ax=ax, label='年份')
ax.axhline(2, color='red', linewidth=1, linestyle='--',
           alpha=0.5, label='通胀目标 2%')
ax.axvline(0, color='gray', linewidth=0.5, linestyle='--')
ax.set_xlabel('GDP 增速（%）')
ax.set_ylabel('CPI 通胀率（%）')
ax.set_title('增长与通胀散点图（G20，2013-2023）', fontsize=13)
ax.legend()
plt.tight_layout()
plt.savefig('../output/fig_growth_inflation_scatter.png', dpi=150)
```

---

## 六、结果解读指引

- 从热力图中，哪些年份是全球经济的「共同低谷」？（2009、2020）各国受影响程度是否一致？
- 中国的增速为何在 2015 年后持续下台阶？与其他新兴市场相比如何？
- 「增长-通胀」散点图中是否能看到正相关？2022 年为何出现高通胀+低增速的「滞涨」象限？
- 哪些国家的数据缺失最严重？这对分析结论有什么影响？

---

## 七、拓展方向（选做）

- 使用 `plotly.express.scatter` 制作可交互的动态气泡图，横轴 GDP 增速、纵轴通胀率、气泡大小为 GDP 总量、颜色区分地区
- 加入人均 GDP 指标，制作不同收入水平国家的增速对比
- 计算各国 2020 年疫情冲击的「V 型复苏」速度（2020 跌幅 vs 2021 反弹幅度）

---

## 八、AI 辅助提示词

**数据获取时：**
```
我想用 wbdata 包获取以下 World Bank 指标：
- NY.GDP.MKTP.KD.ZG（GDP 增速）
- FP.CPI.TOTL.ZG（CPI 通胀率）
国家列表：['CHN','USA','JPN','DEU','GBR','FRA','IND','BRA']
时间范围：1990-2023
请给我完整代码，包括安装、导入、获取和保存为 CSV。
```

**宽长表转换：**
```
我的 DataFrame 是 MultiIndex 格式，第一层是国家代码，第二层是年份，
列名是指标名称。请帮我转换为长表格式（country, year, indicator, value），
并同时保留宽表版本（行=国家，列=年份）供热力图使用。
```

**热力图缺失值处理：**
```
我的热力图数据中有一些 NaN 值，sns.heatmap 会显示为空白格，
请帮我在空白格上标注「N/A」字样，并将颜色设为浅灰色。
```

---

## 九、参考资源

- wbdata 文档：<https://wbdata.readthedocs.io/>
- World Bank 指标搜索：<https://data.worldbank.org/indicator>
- Our World in Data（跨国数据可视化参考）：<https://ourworldindata.org/>
- 课程参考章节：ds2026 第 12 章（数据管理与组织）、第 14 章（数据清洗）

---

*最后更新：2026-05-10*
