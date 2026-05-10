# T-A2：美股「七姐妹」财务与股价全景对比

> 难度：⭐ 入门偏易｜类别：金融与资本市场数据  
> 核心工具：`yfinance` · `pandas` · `matplotlib` / `plotly`  
> 建议人数：7～8 人｜预计完成时间：1～2 周

---

## 一、项目背景

「科技七姐妹」（Magnificent 7）是指苹果（AAPL）、微软（MSFT）、英伟达（NVDA）、Alphabet（GOOGL）、亚马逊（AMZN）、Meta（META）、特斯拉（TSLA）七家美国科技巨头。它们合计市值一度超过全球多数国家的 GDP，是理解美国股市乃至全球资本市场的关键标的。

本题通过 `yfinance` 获取这七家公司的历史股价与财务数据，进行多维度的对比分析与可视化，是练习 API 调用、数据合并、可视化的理想入门项目。

---

## 二、学习目标

完成本题后，你将能够：

- 使用 `yfinance` 批量下载股价与财务数据
- 对多股票数据进行对齐合并（`pd.concat`、`merge`）
- 计算常用财务与风险指标（年化收益、最大回撤、夏普比率、PE/PB）
- 绘制折线图、相关性热力图、箱线图、雷达图等多种图表
- 用 Markdown 规范撰写数据分析报告

---

## 三、项目目录结构

```
T-A2_Magnificent7/
├── readme.md
├── data_raw/
│   ├── prices_raw.csv          ← 原始股价数据
│   └── financials_raw.csv      ← 原始财务数据
├── data_clean/
│   ├── prices_clean.csv        ← 清洗后的价格数据（复权、对齐）
│   └── metrics.csv             ← 计算好的各项指标
├── output/
│   ├── fig_price_trend.png
│   ├── fig_return_heatmap.png
│   ├── fig_risk_return.png
│   └── fig_valuation.png
├── 01_get_data.ipynb
├── 02_data_clean.ipynb
└── 03_analysis_visualization.ipynb
```

---

## 四、任务分解

### 任务 1：数据获取（`01_get_data.ipynb`）

**目标**：获取七家公司近 5 年的日度股价数据和最新财务指标。

**说明文字示例**：
> 使用 `yfinance` 的 `download()` 函数批量下载股价数据，使用 `Ticker.info` 获取 PE、PB、市值等财务摘要。注意指定 `auto_adjust=True` 以获取复权价格。

**参考代码框架**：

```python
import yfinance as yf
import pandas as pd

tickers = ['AAPL', 'MSFT', 'NVDA', 'GOOGL', 'AMZN', 'META', 'TSLA']

# 批量下载股价（近 5 年，日度，复权）
prices = yf.download(tickers, period='5y', auto_adjust=True)['Close']
prices.to_csv('../data_raw/prices_raw.csv')

# 获取财务摘要
records = []
for tk in tickers:
    info = yf.Ticker(tk).info
    records.append({
        'ticker': tk,
        'name':        info.get('shortName'),
        'marketCap':   info.get('marketCap'),
        'trailingPE':  info.get('trailingPE'),
        'priceToBook': info.get('priceToBook'),
        'revenueGrowth': info.get('revenueGrowth'),
        'grossMargins':  info.get('grossMargins'),
        'returnOnEquity': info.get('returnOnEquity'),
    })
pd.DataFrame(records).to_csv('../data_raw/financials_raw.csv', index=False)
print('数据获取完成')
```

**常见问题**：
- `yfinance` 需要网络访问，校园网若有限制可使用 VPN 或在课后完成
- `Ticker.info` 返回的字段因公司而异，用 `.get()` 而非直接取键可避免 KeyError
- 如遇到数据缺失（如 Tesla 的 PB 有时为空），记录在 notebook 中说明原因

---

### 任务 2：数据清洗（`02_data_clean.ipynb`）

**目标**：处理缺失值、计算收益率序列、整理财务指标表。

**参考代码框架**：

```python
import pandas as pd
import numpy as np

prices = pd.read_csv('../data_raw/prices_raw.csv', index_col=0, parse_dates=True)

# 检查缺失值
print(prices.isnull().sum())

# 前向填充（节假日停市产生的缺失）
prices = prices.ffill()

# 计算日度收益率
returns = prices.pct_change().dropna()

# 计算年化收益率（假设 252 个交易日）
annual_return = returns.mean() * 252

# 计算年化波动率
annual_vol = returns.std() * np.sqrt(252)

# 计算最大回撤
def max_drawdown(series):
    cumulative = (1 + series).cumprod()
    rolling_max = cumulative.cummax()
    drawdown = (cumulative - rolling_max) / rolling_max
    return drawdown.min()

max_dd = returns.apply(max_drawdown)

# 计算夏普比率（假设无风险利率 4.5%，当前美国短期国债水平）
rf = 0.045 / 252
sharpe = (returns.mean() - rf) / returns.std() * np.sqrt(252)

metrics = pd.DataFrame({
    '年化收益率': annual_return,
    '年化波动率': annual_vol,
    '最大回撤':   max_dd,
    '夏普比率':   sharpe,
})
metrics.to_csv('../data_clean/metrics.csv')
```

---

### 任务 3：分析与可视化（`03_analysis_visualization.ipynb`）

建议完成以下 **4 张核心图表**，每张图后附 3～5 句结果解读：

#### 图 1：标准化股价走势（基期 = 1）

```python
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

prices_norm = prices / prices.iloc[0]  # 基期归一化

fig, ax = plt.subplots(figsize=(12, 5))
for col in prices_norm.columns:
    ax.plot(prices_norm.index, prices_norm[col], label=col, linewidth=1.5)

ax.set_title('七姐妹股价走势（近 5 年，基期 = 1）', fontsize=14)
ax.set_xlabel('日期')
ax.set_ylabel('相对价格')
ax.legend(loc='upper left', ncol=2)
ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m'))
plt.tight_layout()
plt.savefig('../output/fig_price_trend.png', dpi=150)
plt.show()
```

#### 图 2：年度收益率热力图

```python
import seaborn as sns

annual_returns_by_year = returns.resample('YE').apply(
    lambda x: (1 + x).prod() - 1
) * 100

fig, ax = plt.subplots(figsize=(10, 4))
sns.heatmap(annual_returns_by_year.T, annot=True, fmt='.1f',
            cmap='RdYlGn', center=0, ax=ax,
            linewidths=0.5, cbar_kws={'label': '年度收益率 (%)'})
ax.set_title('七姐妹年度收益率热力图 (%)', fontsize=13)
plt.tight_layout()
plt.savefig('../output/fig_return_heatmap.png', dpi=150)
```

#### 图 3：风险-收益散点图

```python
fig, ax = plt.subplots(figsize=(7, 6))
for tk in metrics.index:
    ax.scatter(metrics.loc[tk, '年化波动率'] * 100,
               metrics.loc[tk, '年化收益率'] * 100,
               s=100, zorder=5)
    ax.annotate(tk,
                xy=(metrics.loc[tk, '年化波动率'] * 100,
                    metrics.loc[tk, '年化收益率'] * 100),
                xytext=(5, 5), textcoords='offset points', fontsize=10)

ax.set_xlabel('年化波动率 (%)')
ax.set_ylabel('年化收益率 (%)')
ax.set_title('风险-收益分布（近 5 年）', fontsize=13)
ax.axhline(0, color='gray', linewidth=0.5, linestyle='--')
plt.tight_layout()
plt.savefig('../output/fig_risk_return.png', dpi=150)
```

#### 图 4：估值指标对比（PE、PB 双轴柱状图）

```python
financials = pd.read_csv('../data_raw/financials_raw.csv', index_col='ticker')

fig, ax1 = plt.subplots(figsize=(9, 5))
x = range(len(financials))
bars1 = ax1.bar([i - 0.2 for i in x], financials['trailingPE'],
                width=0.35, label='PE（市盈率）', color='steelblue', alpha=0.8)
ax1.set_ylabel('市盈率（PE）')
ax1.set_xticks(list(x))
ax1.set_xticklabels(financials.index)

ax2 = ax1.twinx()
bars2 = ax2.bar([i + 0.2 for i in x], financials['priceToBook'],
                width=0.35, label='PB（市净率）', color='darkorange', alpha=0.8)
ax2.set_ylabel('市净率（PB）')

lines = [bars1, bars2]
ax1.legend(lines, ['PE（市盈率）', 'PB（市净率）'], loc='upper left')
ax1.set_title('七姐妹估值对比：PE 与 PB', fontsize=13)
plt.tight_layout()
plt.savefig('../output/fig_valuation.png', dpi=150)
```

---

## 五、结果解读指引

完成每张图后，在下方的 Markdown Cell 中回答以下问题（不限于此）：

- 过去 5 年涨幅最大/最小的是哪家公司？主要受什么事件驱动？
- 哪家公司的风险调整后收益（夏普比率）最高？
- 七家公司的估值是否处于合理区间？相互之间的差异如何解释？
- 你观察到哪些你认为值得深入研究的规律或异常？

---

## 六、拓展方向（选做）

- 加入标普 500 指数（`^GSPC`）作为基准，计算各股的超额收益（α）
- 计算两两相关系数矩阵，用热力图展示，分析「七姐妹」内部的分散化效果
- 用 `plotly` 替换 `matplotlib`，制作可交互的图表
- 尝试用 `OpenBB` 获取机构持仓变化数据

---

## 七、AI 辅助提示词

**获取数据时：**
```
我想用 yfinance 批量下载 ['AAPL','MSFT','NVDA','GOOGL','AMZN','META','TSLA'] 
近 5 年的日度收盘价（复权），并获取每家公司的 PE、PB、市值、营收增速等财务指标。
请给我完整的 Python 代码，包括错误处理和数据保存。
```

**计算最大回撤时：**
```
我有一个 pandas DataFrame，每列是一只股票的日度收益率序列，
请帮我写一个函数计算每只股票的最大回撤，并解释计算逻辑。
```

**图表美化时：**
```
我用 matplotlib 画了一张多股票的折线图，请帮我改进：
1. 使用色盲友好配色（colorblind-safe palette）
2. 添加鼠标悬停提示（改用 plotly）
3. x 轴只显示年份，y 轴添加百分号格式
```

---

## 八、参考资源

- yfinance 文档：<https://ranaroussi.github.io/yfinance/>
- 夏普比率计算：<https://en.wikipedia.org/wiki/Sharpe_ratio>
- 最大回撤可视化示例：<https://plotly.com/python/time-series/>
- 课程参考章节：ds2026 第 9 章（金融数据获取）、第 14 章（数据清洗）

---

*最后更新：2026-05-10*
