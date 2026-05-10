# T-A3：中美利率与汇率联动关系探索

> 难度：⭐⭐ 中等｜类别：金融与资本市场数据  
> 核心工具：`akshare` · `fredapi` · `pandas` · `matplotlib` / `seaborn`  
> 建议人数：7～8 人｜预计完成时间：1～2 周

---

## 一、项目背景

中美利差（中国 10 年期国债收益率 − 美国 10 年期国债收益率）是影响人民币汇率、跨境资本流动的重要因素之一。2022 年以来，随着美联储持续加息而中国维持宽松货币政策，中美利差出现罕见倒挂，人民币兑美元汇率承压。

本题通过同时获取中美两国的利率数据与人民币汇率数据，计算利差序列，分析「利差—汇率」关系在不同宏观周期下的变化规律，是练习多数据源合并与时序分析的中等难度项目。

---

## 二、学习目标

完成本题后，你将能够：

- 使用 `akshare` 获取中国债券收益率和人民币汇率数据
- 使用 `fredapi` 获取美联储 FRED 数据库的宏观数据
- 处理不同频率数据的对齐（日度 vs 月度）
- 计算滚动相关系数并可视化动态关系
- 理解「利差」与「汇率」的经济逻辑

---

## 三、项目目录结构

```
T-A3_RateExchangeAnalysis/
├── readme.md
├── data_raw/
│   ├── cn_bond_yield_raw.csv      ← 中国 10Y 国债收益率
│   ├── us_bond_yield_raw.csv      ← 美国 10Y 国债收益率（FRED）
│   └── usdcny_rate_raw.csv        ← 人民币兑美元汇率
├── data_clean/
│   └── merged_monthly.csv         ← 对齐后的月度合并数据
├── output/
│   ├── fig_yield_trend.png
│   ├── fig_spread.png
│   ├── fig_exchange_rate.png
│   └── fig_rolling_corr.png
├── 01_get_data.ipynb
├── 02_data_clean_merge.ipynb
└── 03_analysis_visualization.ipynb
```

---

## 四、前置准备：FRED API Key

FRED（Federal Reserve Economic Data）是美联储旗下的免费经济数据库，提供超过 80 万条数据序列。使用 `fredapi` 前需申请 API Key：

1. 访问 <https://fred.stlouisfed.org/> 并注册账号（支持国内邮箱）
2. 登录后访问「My Account → API Keys」申请 Key（即时生效）
3. 将 Key 保存为环境变量或写入 `config.py`（**不要直接写进 notebook 再上传到 GitHub**）

```python
# config.py（不提交到 GitHub，加入 .gitignore）
FRED_API_KEY = 'your_key_here'
```

> **注意**：老师已统一申请教学用 Key，如无法自行注册可联系老师获取。

---

## 五、任务分解

### 任务 1：获取数据（`01_get_data.ipynb`）

#### 1a：用 akshare 获取中国国债收益率和汇率

```python
import akshare as ak
import pandas as pd

# 中国 10 年期国债收益率（日度）
cn_bond = ak.bond_zh_us_rate(start_date='20150101')
# 查看字段
print(cn_bond.head())
print(cn_bond.columns.tolist())

# 保留中国 10Y 列，重命名
cn_yield = cn_bond[['日期', '中国国债收益率10年']].copy()
cn_yield.columns = ['date', 'cn_yield_10y']
cn_yield['date'] = pd.to_datetime(cn_yield['date'])
cn_yield.to_csv('../data_raw/cn_bond_yield_raw.csv', index=False)

# 人民币兑美元即期汇率（日度，中间价）
fx = ak.currency_boc_sina(symbol='美元')
print(fx.head())
fx_clean = fx[['日期', '中间价']].copy()
fx_clean.columns = ['date', 'usdcny']
fx_clean['date'] = pd.to_datetime(fx_clean['date'])
fx_clean.to_csv('../data_raw/usdcny_rate_raw.csv', index=False)
```

#### 1b：用 fredapi 获取美国国债收益率

```python
from fredapi import Fred
import sys
sys.path.append('..')
from config import FRED_API_KEY

fred = Fred(api_key=FRED_API_KEY)

# 美国 10Y 国债收益率（日度）
# 系列代码：DGS10（10-Year Treasury Constant Maturity Rate）
us_yield = fred.get_series('DGS10', observation_start='2015-01-01')
us_yield = us_yield.reset_index()
us_yield.columns = ['date', 'us_yield_10y']
us_yield.to_csv('../data_raw/us_bond_yield_raw.csv', index=False)
print(f'美国国债收益率数据：{len(us_yield)} 条')
print(us_yield.tail())
```

> **FRED 常用系列代码参考**：
> - `DGS10`：美国 10 年期国债收益率（日度）
> - `FEDFUNDS`：联邦基金利率（月度）
> - `DEXCHUS`：人民币兑美元汇率（可与 akshare 对比验证）

---

### 任务 2：数据清洗与合并（`02_data_clean_merge.ipynb`）

**核心挑战**：三个数据集的日期不完全一致（中美交易日历不同、汇率有缺失），需要统一到月度频率后合并。

```python
import pandas as pd
import numpy as np

cn_yield = pd.read_csv('../data_raw/cn_bond_yield_raw.csv', parse_dates=['date'])
us_yield = pd.read_csv('../data_raw/us_bond_yield_raw.csv', parse_dates=['date'])
usdcny   = pd.read_csv('../data_raw/usdcny_rate_raw.csv',   parse_dates=['date'])

# 检查缺失值
for name, df in [('中国收益率', cn_yield), ('美国收益率', us_yield), ('汇率', usdcny)]:
    print(f'{name} 缺失值：{df.isnull().sum().sum()} 条，共 {len(df)} 条')

# 各序列设日期为索引
cn_yield = cn_yield.set_index('date').sort_index()
us_yield = us_yield.set_index('date').sort_index()
usdcny   = usdcny.set_index('date').sort_index()

# 统一转为月末频率（取月内最后一个有效值）
cn_m  = cn_yield.resample('ME').last()
us_m  = us_yield.resample('ME').last()
cny_m = usdcny.resample('ME').last()

# 合并三个序列
merged = pd.concat([cn_m, us_m, cny_m], axis=1)
merged.columns = ['cn_yield_10y', 'us_yield_10y', 'usdcny']

# 计算利差（中国 - 美国，单位：百分点）
merged['spread'] = merged['cn_yield_10y'] - merged['us_yield_10y']

# 删除任何列有缺失的行，记录删除数量
n_before = len(merged)
merged = merged.dropna()
print(f'删除 {n_before - len(merged)} 行含缺失值的记录，剩余 {len(merged)} 条')

merged.to_csv('../data_clean/merged_monthly.csv')
print(merged.tail())
```

---

### 任务 3：分析与可视化（`03_analysis_visualization.ipynb`）

#### 图 1：中美 10 年期国债收益率走势对比

```python
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

merged = pd.read_csv('../data_clean/merged_monthly.csv', parse_dates=['date'], index_col='date')

fig, ax = plt.subplots(figsize=(12, 5))
ax.plot(merged.index, merged['cn_yield_10y'], label='中国 10Y 国债', color='#c0392b', linewidth=2)
ax.plot(merged.index, merged['us_yield_10y'], label='美国 10Y 国债', color='#2980b9', linewidth=2)
ax.fill_between(merged.index, merged['cn_yield_10y'], merged['us_yield_10y'],
                where=merged['cn_yield_10y'] >= merged['us_yield_10y'],
                alpha=0.15, color='red', label='中国 > 美国')
ax.fill_between(merged.index, merged['cn_yield_10y'], merged['us_yield_10y'],
                where=merged['cn_yield_10y'] < merged['us_yield_10y'],
                alpha=0.15, color='blue', label='利差倒挂')
ax.set_title('中美 10 年期国债收益率（%）', fontsize=13)
ax.set_ylabel('收益率（%）')
ax.legend()
ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y'))
plt.tight_layout()
plt.savefig('../output/fig_yield_trend.png', dpi=150)
```

#### 图 2：中美利差时序图（标注关键事件）

```python
fig, ax = plt.subplots(figsize=(12, 4))
ax.plot(merged.index, merged['spread'], color='#27ae60', linewidth=1.8)
ax.axhline(0, color='black', linewidth=0.8, linestyle='--', alpha=0.6)
ax.fill_between(merged.index, merged['spread'], 0,
                where=merged['spread'] >= 0, alpha=0.2, color='green')
ax.fill_between(merged.index, merged['spread'], 0,
                where=merged['spread'] < 0,  alpha=0.2, color='red')

# 标注关键事件（手动填写）
events = {
    '2022-03': '美联储\n开始加息',
    '2023-10': '利差\n最深倒挂',
}
for date_str, label in events.items():
    date = pd.Timestamp(date_str)
    if date in merged.index:
        ax.annotate(label, xy=(date, merged.loc[date, 'spread']),
                    xytext=(15, 20), textcoords='offset points',
                    arrowprops=dict(arrowstyle='->', color='gray'),
                    fontsize=9, color='gray')

ax.set_title('中美 10 年期国债利差（中国 - 美国，%）', fontsize=13)
ax.set_ylabel('利差（百分点）')
plt.tight_layout()
plt.savefig('../output/fig_spread.png', dpi=150)
```

#### 图 3：利差与汇率的散点图及相关性

```python
import seaborn as sns

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# 左：散点图
axes[0].scatter(merged['spread'], merged['usdcny'], alpha=0.6, s=25)
axes[0].set_xlabel('中美利差（%）')
axes[0].set_ylabel('美元兑人民币（USD/CNY）')
axes[0].set_title('利差 vs 汇率散点图')

# 加线性拟合线
import numpy as np
z = np.polyfit(merged['spread'], merged['usdcny'], 1)
p = np.poly1d(z)
x_line = np.linspace(merged['spread'].min(), merged['spread'].max(), 100)
axes[0].plot(x_line, p(x_line), 'r--', linewidth=1.5)

# 右：滚动相关（12 个月）
rolling_corr = merged['spread'].rolling(12).corr(merged['usdcny'])
axes[1].plot(merged.index, rolling_corr, color='purple', linewidth=1.8)
axes[1].axhline(0, color='gray', linewidth=0.5, linestyle='--')
axes[1].set_title('利差与汇率的滚动相关系数（12 个月窗口）')
axes[1].set_ylabel('相关系数')

plt.tight_layout()
plt.savefig('../output/fig_rolling_corr.png', dpi=150)
```

---

## 六、结果解读指引

- 中美利差在哪些时段为正（中国利率高于美国）？哪些时段倒挂？
- 利差倒挂期间，人民币汇率是否出现贬值？这与教科书上「利率平价理论」的预测是否一致？
- 滚动相关系数是否稳定？在哪些时段相关性较弱？可能的解释是什么？
- 利差与汇率的关系是「同期」还是「有时滞」？（可以尝试滞后 1～3 个月的相关分析）

---

## 七、拓展方向（选做）

- 加入中美 CPI 对比，探讨「实际利差」（名义利差 - 通胀差）与汇率的关系
- 用 FRED 的 `FEDFUNDS` 数据标注加息/降息周期，观察周期切换前后利差的变化
- 将分析扩展至其他货币对（欧元、日元）与美国利差的关系

---

## 八、AI 辅助提示词

**日期对齐问题：**
```
我有三个 pandas DataFrame，date 列格式都是 datetime，
但频率不同（日度、月度）且有缺失。
请帮我将它们统一转换为月末频率（取每月最后一个有效值），然后 merge 到一起。
```

**绘图时中文乱码：**
```
我用 matplotlib 画图，图上的中文显示为方块。
我的环境是 Windows/macOS/Linux（请选择），请帮我解决中文字体问题。
```

**滚动相关计算：**
```
我有两列时间序列数据（spread 和 usdcny），
请帮我计算 12 个月的滚动 Pearson 相关系数，并用折线图可视化，
同时在图上标注相关系数为负的区域（红色背景）。
```

---

## 九、参考资源

- akshare 债券数据文档：<https://akshare.akfamily.xyz/data/bond/bond.html>
- FRED API Python 包：<https://github.com/mortada/fredapi>
- FRED 系列代码查询：<https://fred.stlouisfed.org/>
- 人民币汇率形成机制简介：<https://www.pbc.gov.cn>
- 课程参考章节：ds2026 第 9 章（金融数据获取）、第 13 章（数据管理）

---

*最后更新：2026-05-10*
