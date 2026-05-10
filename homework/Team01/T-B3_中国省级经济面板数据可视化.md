# T-B3：中国省级经济面板数据整理与可视化

> 难度：⭐⭐ 中等｜类别：宏观经济与国际数据  
> 核心工具：`akshare` · `pandas` · `matplotlib` · `folium` / `plotly`  
> 建议人数：7～8 人｜预计完成时间：1～2 周

---

## 一、项目背景

中国幅员辽阔，各省经济发展水平差异显著：东部沿海省份人均 GDP 比西部内陆高出数倍，而近年来西部地区增速反超东部的趋势也在加速。国家统计局每年公布各省的 GDP、工业增加值、社会消费品零售总额、固定资产投资等核心经济指标，是分析区域经济格局的基础数据。

本题获取并整理近 10～15 年的省级经济面板数据，通过地图、热力图、排名动态图等多种可视化形式，呈现中国区域经济格局的演变规律。

---

## 二、学习目标

完成本题后，你将能够：

- 使用 `akshare` 获取中国宏观和省级经济数据
- 整理「宽表 → 长表」的面板数据格式转换
- 处理省级数据的常见问题（直辖市与省的区分、数据口径变化）
- 用 `folium` 或 `plotly` 绘制中国地图，实现地理数据可视化
- 制作排名动态条形图（Bar Chart Race）

---

## 三、项目目录结构

```
T-B3_ProvincialEconomy/
├── readme.md
├── data_raw/
│   ├── gdp_province_raw.csv       ← 各省 GDP 原始数据
│   ├── industry_raw.csv           ← 工业增加值原始数据
│   └── retail_raw.csv             ← 社会消费品零售总额
├── data_clean/
│   ├── gdp_panel.csv              ← 整理后的 GDP 面板数据（长表）
│   └── latest_year.csv            ← 最新年度各省指标对比表
├── output/
│   ├── fig_gdp_map.html           ← 交互式地图（folium）
│   ├── fig_gdp_heatmap.png
│   ├── fig_growth_rank.png
│   └── fig_coastal_inland.png
├── 01_get_data.ipynb
├── 02_data_clean.ipynb
└── 03_analysis_visualization.ipynb
```

---

## 四、任务分解

### 任务 1：数据获取（`01_get_data.ipynb`）

`akshare` 提供了丰富的国内宏观数据接口，以下是常用的省级数据接口：

```python
import akshare as ak
import pandas as pd

# --- 方法一：akshare 宏观接口（推荐，免爬虫）---

# 查看可用的宏观数据接口
help(ak.macro_china_gdp)  # 全国 GDP

# 各省 GDP（部分接口）
# akshare 的省级数据接口命名规律：macro_china_xxx
# 建议在 Jupyter 中输入 ak.macro_china_ 然后按 Tab 查看所有接口

# 省级 GDP 数据（如有）
try:
    gdp_prov = ak.macro_china_gdp_province()
    print(gdp_prov.head())
    gdp_prov.to_csv('../data_raw/gdp_province_raw.csv', index=False)
except Exception as e:
    print(f'接口不可用：{e}，请改用方法二')

# --- 方法二：直接从国家统计局 CSV 下载（备选）---
# 访问 https://data.stats.gov.cn/
# 选择「分省年度数据」→「地区生产总值」→「下载 CSV」
# 将文件放入 data_raw/ 后读取：
# gdp_prov = pd.read_csv('../data_raw/gdp_province_stats_gov.csv',
#                         encoding='gbk')  # 注意编码
```

> **推荐数据获取策略**：
> 1. 先试 akshare 接口（`ak.macro_china_` 系列），能用就用
> 2. 若接口失效，从国家统计局数据库 <https://data.stats.gov.cn/> 手动下载 CSV
> 3. 两种方式获取的数据在 notebook 中都要记录来源和下载时间

#### 探索可用接口

```python
# 列出所有 akshare 中包含 'province' 或 'gdp' 的函数
import akshare as ak
funcs = [f for f in dir(ak) if 'province' in f.lower() or
         ('china' in f.lower() and 'gdp' in f.lower())]
print('\n'.join(funcs))
```

---

### 任务 2：数据清洗（`02_data_clean.ipynb`）

省级数据常见问题及处理方法：

```python
import pandas as pd
import numpy as np

# 假设已读入数据（按实际字段名调整）
df = pd.read_csv('../data_raw/gdp_province_raw.csv')
print(df.head())
print(df.dtypes)

# --- 问题 1：省份名称不统一 ---
# 统计局数据中可能出现「内蒙古自治区」「内蒙古」「Inner Mongolia」等
province_map = {
    '内蒙古自治区': '内蒙古',
    '西藏自治区': '西藏',
    '新疆维吾尔自治区': '新疆',
    '广西壮族自治区': '广西',
    '宁夏回族自治区': '宁夏',
    '北京市': '北京',
    '天津市': '天津',
    '上海市': '上海',
    '重庆市': '重庆',
    # ……根据实际数据补充
}
if '省份' in df.columns:
    df['province'] = df['省份'].replace(province_map)

# --- 问题 2：数值列含单位文字或逗号 ---
# 如「12,345.6 亿元」→ 需提取数字
def clean_numeric(s):
    if isinstance(s, str):
        s = s.replace(',', '').replace('亿元', '').replace(' ', '')
        try:
            return float(s)
        except:
            return np.nan
    return s

for col in df.select_dtypes(include='object').columns:
    if col != 'province':
        df[col] = df[col].apply(clean_numeric)

# --- 问题 3：宽表转长表（年份为列名）---
# 假设列名为：province, 2010, 2011, ..., 2023
year_cols = [c for c in df.columns if str(c).isdigit()]
df_long = df.melt(
    id_vars=['province'],
    value_vars=year_cols,
    var_name='year',
    value_name='gdp_100m'  # 单位：亿元
)
df_long['year'] = df_long['year'].astype(int)

# 计算同比增速
df_long = df_long.sort_values(['province', 'year'])
df_long['gdp_growth'] = df_long.groupby('province')['gdp_100m'].pct_change() * 100

df_long.to_csv('../data_clean/gdp_panel.csv', index=False)
print(df_long.head(20))
```

---

### 任务 3：分析与可视化（`03_analysis_visualization.ipynb`）

#### 图 1：各省 GDP 总量地图（folium 交互版）

```python
import folium
import pandas as pd
import json

df = pd.read_csv('../data_clean/gdp_panel.csv')
latest = df[df['year'] == df['year'].max()].copy()

# 获取中国省级 GeoJSON（需下载）
# 下载地址：https://github.com/longwosion/geojson-map-china
# 文件：china_provinces.geojson

with open('../data_raw/china_provinces.geojson', 'r', encoding='utf-8') as f:
    china_geo = json.load(f)

m = folium.Map(location=[35, 105], zoom_start=4,
               tiles='CartoDB positron')

folium.Choropleth(
    geo_data=china_geo,
    data=latest,
    columns=['province', 'gdp_100m'],
    key_on='feature.properties.name',
    fill_color='YlOrRd',
    fill_opacity=0.75,
    line_opacity=0.3,
    legend_name=f'GDP（亿元，{latest["year"].max()}年）',
    highlight=True,
).add_to(m)

m.save('../output/fig_gdp_map.html')
print('地图已保存，用浏览器打开 output/fig_gdp_map.html 查看')
```

> **GeoJSON 文件下载**：访问 <https://github.com/longwosion/geojson-map-china>，下载 `china_provinces.geojson` 放入 `data_raw/` 目录。

#### 图 2：GDP 增速热力图（省份 × 年份）

```python
import seaborn as sns
import matplotlib.pyplot as plt

df = pd.read_csv('../data_clean/gdp_panel.csv')

# 筛选近 10 年，构建宽表
growth_wide = df[df['year'].between(2013, 2023)].pivot_table(
    index='province', columns='year', values='gdp_growth'
)

# 按最近一年增速排序
growth_wide = growth_wide.sort_values(2023, ascending=False)

fig, ax = plt.subplots(figsize=(14, 12))
sns.heatmap(growth_wide, annot=True, fmt='.1f',
            cmap='RdYlGn', center=5, ax=ax,
            linewidths=0.3, cbar_kws={'label': 'GDP 增速 (%)'},
            annot_kws={'size': 7})
ax.set_title('各省 GDP 增速热力图（2013-2023，%）', fontsize=13)
ax.set_xlabel('年份')
ax.set_ylabel('')
plt.tight_layout()
plt.savefig('../output/fig_gdp_heatmap.png', dpi=150, bbox_inches='tight')
```

#### 图 3：东中西部 GDP 增速对比（分组折线图）

```python
# 按地区划分省份
east = ['北京','天津','河北','上海','江苏','浙江','福建',
        '山东','广东','海南','辽宁']
central = ['山西','吉林','黑龙江','安徽','江西','河南','湖北','湖南']
west = ['内蒙古','广西','重庆','四川','贵州','云南','西藏',
        '陕西','甘肃','青海','宁夏','新疆']

df['region'] = df['province'].apply(
    lambda x: '东部' if x in east else ('中部' if x in central else '西部')
)

region_avg = df.groupby(['region', 'year'])['gdp_growth'].mean().reset_index()
region_avg = region_avg[region_avg['year'].between(2005, 2023)]

fig, ax = plt.subplots(figsize=(12, 5))
colors = {'东部': '#2980b9', '中部': '#27ae60', '西部': '#e67e22'}
for region, group in region_avg.groupby('region'):
    ax.plot(group['year'], group['gdp_growth'],
            label=region, color=colors[region], linewidth=2, marker='o', markersize=4)

ax.set_title('东中西部 GDP 平均增速对比（2005-2023）', fontsize=13)
ax.set_xlabel('年份')
ax.set_ylabel('GDP 增速（%）')
ax.legend()
plt.tight_layout()
plt.savefig('../output/fig_coastal_inland.png', dpi=150)
```

---

## 五、结果解读指引

- 哪些省份的 GDP 增速在近 10 年持续高于全国平均？有何共同特征？
- 2020 年各省受 COVID-19 冲击是否均等？哪些省份受影响最大/最小？
- 东中西部的增速差距是在缩小还是扩大？这与「西部大开发」「中部崛起」政策的预期是否一致？
- 直辖市（北京、上海）的数据与同体量省份相比有何差异？是否需要特殊处理？

---

## 六、拓展方向（选做）

- 用 `bar_chart_race` 库制作 GDP 总量排名的动态条形图动画（`pip install bar-chart-race`）
- 加入人均 GDP 数据，制作「总量 vs 人均」的双维对比，找出「大而不富」的省份
- 结合工业增加值和消费数据，分析各省的经济结构（工业主导 vs 消费主导）

---

## 七、AI 辅助提示词

**akshare 接口探索：**
```
我想用 akshare 获取中国各省的年度 GDP 数据，
请帮我列出所有可能有用的接口名称，并说明每个接口的参数和返回字段。
如果 akshare 没有现成接口，请告诉我如何从国家统计局网站下载数据。
```

**GeoJSON 地图绘制：**
```
我有一个 DataFrame，包含 province（省份中文名）和 gdp_100m（GDP亿元）两列。
我已下载了中国省级 GeoJSON 文件，GeoJSON 中省份名字段是 feature.properties.name。
请帮我用 folium 绘制一张中国 GDP 热力地图，鼠标悬停时显示省份名和 GDP 数值。
```

**Bar Chart Race 动画：**
```
我有一个 DataFrame，行是年份，列是省份，值是 GDP 总量（亿元）。
请帮我用 bar-chart-race 库生成一个动态排名条形图视频，
显示 2000-2023 年各省 GDP 排名变化，输出为 MP4 格式。
```

---

## 八、参考资源

- akshare 宏观数据文档：<https://akshare.akfamily.xyz/data/macro/macro.html>
- 国家统计局数据库：<https://data.stats.gov.cn/>
- 中国省级 GeoJSON：<https://github.com/longwosion/geojson-map-china>
- bar-chart-race 文档：<https://www.dexplo.org/bar_chart_race/>
- 课程参考章节：ds2026 第 12 章（数据管理与组织）、第 13 章（数据管理格式与存储）

---

*最后更新：2026-05-10*
