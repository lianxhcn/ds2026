# ex_P02b：CSMAR 上市公司财务特征清洗与分析

> **作业性质**：个人作业  
> **预计用时**：4-15 小时  
> **对应讲义**：第 12-13 章 (数据管理)、第 14 章 (数据清洗)、第 15 章 (描述统计与可视化)  
> **提交方式**：见文末「提交要求」

---

## 任务背景

上市公司财务数据是公司金融、资本结构、公司治理和行业比较研究中最常用的数据类型之一。拿到 CSMAR 数据之后，真正的难点并不只是「读入数据」，而是如何识别变量、统一口径、处理缺失值和离群值，并把分散在多个原始文件中的信息整理成可用于分析的公司—年度面板数据。

本次作业以 CSMAR(国泰安) 数据为基础，要求你完成一套完整的数据处理与分析流程：从原始压缩包解压、变量字典整理、公司—年度面板构造，到描述统计、时序图、行业比较和股权结构分析。作业重点不是简单生成几个表格和图形，而是训练你建立一套可复现的数据清洗和分析规范。

---

## 第一部分：数据准备与项目结构

### 1.1 下载 CSMAR 数据

下载 [CSMAR](https://pan.sysu.edu.cn/link/AA1F301427054F4DAAB7B034C91952C8AD) 文件夹到本地。

该文件夹中的 `data_raw_zip/` 子文件夹包含 7 个 `.zip` 文件，均来自 CSMAR(国泰安) 数据库。本次作业所需变量都可以在这 7 个文件中找到。

要求：

- 将 7 个 `.zip` 文件保留在 `data/data_raw_zip/`。
- 使用 Python 代码自动解压，不得手动逐个解压。
- 解压后的原始数据存放在 `data/raw/`。
- 在 `README.md` 中列出 7 个原始文件的名称、主要内容和你从中提取的变量。
- 原始数据文件体积通常较大，不建议上传 GitHub；但必须说明他人如何通过运行 Notebook 从头重建数据。

### 1.2 项目目录结构

按以下规范建立项目文件夹，**使用 Python 代码自动创建**，不得手动新建：

```text
dshw-p02b/
├── README.md                         ← 项目说明
├── report.html                       ← 分析报告(HTML 或 PDF，见第六部分)
├── requirements.txt                  ← 依赖库列表
├── .gitignore
├── 01_extract_raw_data.ipynb         ← 解压、读取和初步识别原始数据
├── 02_clean_construct_variables.ipynb← 数据清洗、变量构造和合并
├── 03_analysis.ipynb                 ← 描述统计、可视化和分析
├── data/
│   ├── data_raw_zip/                 ← CSMAR 原始压缩包，不上传 GitHub
│   ├── raw/                          ← 解压后的原始数据，不上传 GitHub
│   ├── dict/                         ← 变量字典和字段说明
│   │   └── variable_dictionary.csv
│   ├── clean/                        ← 清洗后的单表数据
│   │   └── firm_year_clean.csv
│   ├── combined/                     ← 合并后的分析数据
│   │   └── csmar_firm_year_panel.csv
│   └── temp/                         ← 临时文件，不上传 GitHub
├── output/                           ← 表格和图形输出
│   ├── tables/
│   └── figures/
└── process_log.txt                   ← 数据处理日志
```

文件命名规范：全部小写，用下划线分隔，不含空格和中文。

### 1.3 变量字典

在 `data/dict/variable_dictionary.csv` 中建立变量字典，至少包含以下字段：

| 字段 | 含义 |
|------|------|
| `source_file` | 变量所在的原始文件 |
| `raw_variable` | CSMAR 原始变量名 |
| `clean_variable` | 清洗后的变量名 |
| `definition` | 变量含义 |
| `unit` | 单位 |
| `note` | 口径说明或特殊处理说明 |

需要特别说明：

- 不同版本的 CSMAR 文件中，原始字段名可能不同。你需要根据数据文件中的变量标签、字段说明或 Excel 表头，确认变量来源。
- 如果某个指标不是直接给出的，而是由多个字段计算得到，必须在 `note` 中写明计算公式。
- 如果某个指标的可得年份晚于 2000 年，使用实际可得年份，并在报告中说明。

---

## 第二部分：数据清洗与变量构造

在 `02_clean_construct_variables.ipynb` 中，完成公司—年度面板数据的清洗和构造。每一步必须有 Markdown 单元格说明处理逻辑，不能只有代码。

### 2.1 基础清洗要求

对每个原始数据表，至少完成以下检查：

| 清洗项目 | 具体要求 |
|---------|---------|
| 编码与读取 | 尝试识别文件编码，例如 `utf-8-sig`、`gbk` 或 `gb18030`；若读取失败，说明原因和处理方式 |
| 主键统一 | 将股票代码统一为 6 位字符串，记为 `code`；将会计年度统一为整数型，记为 `year` |
| 日期处理 | 将报表日期、上市日期等日期变量统一为 `datetime64` 格式 |
| 年度样本 | 保留 2000 年至今的年度观测；如果原始数据起始年份较晚，以实际年份为准 |
| 重复值处理 | 检查 `code-year` 是否重复；若重复，说明保留规则 |
| 缺失值检测 | 统计核心变量的缺失数量和缺失比例，并解释可能原因 |
| 数值型转换 | 将金额、比例、股权比例等变量转换为数值型 |
| 异常值检查 | 检查分母为 0、负资产、负权益、比例异常等问题，并说明处理方式 |

### 2.2 样本口径

建议采用以下样本口径，并在 `README.md` 和报告中说明：

- 公司范围：A 股上市公司。
- 时间范围：2000 年至最新可得年份。
- 频率：年度。
- 报表口径：优先使用合并报表；如原始数据包含多种报表类型，需说明筛选规则。
- 行业分类：使用 CSMAR 行业代码，至少保留行业大类代码(如 `C`、`D`、`E`、`F`、`G`、`J`、`K`)。
- 金融业公司是否保留：本作业 C 部分要求分析金融业(`J`)，因此不要直接删除金融业；但在分析中需要说明金融业资产负债率口径与非金融企业可能不同。

### 2.3 指标构造

构造以下公司—年度变量。所有比率变量建议使用小数形式，例如 0.35 表示 35%。

| 变量 | 名称 | 建议计算方式 |
|------|------|--------------|
| `Lev` | 总负债率 | 总负债 / 总资产 |
| `SL` | 流动负债率 | 流动负债 / 总资产 |
| `LL` | 长期负债率 | 非流动负债 / 总资产；若无非流动负债字段，可用总负债 - 流动负债 |
| `SDR` | 短债比率 | 流动负债 / 总负债 |
| `Cash` | 现金比率 | 货币资金或现金及现金等价物 / 总资产 |
| `ROA` | 总资产收益率 | 净利润 / 总资产 |
| `ROE` | 净资产收益率 | 净利润 / 净资产 |
| `SLoan` | 短期银行借款率 | 短期银行借款 / 总资产 |
| `LLoan` | 长期银行借款率 | 长期银行借款 / 总资产 |
| `Top1` | 第一大股东持股比例 | 第一大股东持股比例 |
| `HHI5` | 前五大股东持股集中度 | 前五大股东持股比例平方和 |
| `Size` | 公司规模 | 总资产的自然对数 |
| `Age` | 上市年限 | 会计年度 - 上市年份 + 1 |

对应公式为：

$$
\begin{aligned}
\operatorname{Lev}_{i,t}
&= \frac{\operatorname{TotalLiab}_{i,t}}{\operatorname{TotalAsset}_{i,t}}, \\
\operatorname{SL}_{i,t}
&= \frac{\operatorname{CurrentLiab}_{i,t}}{\operatorname{TotalAsset}_{i,t}}, \\
\operatorname{LL}_{i,t}
&= \frac{\operatorname{NonCurrentLiab}_{i,t}}{\operatorname{TotalAsset}_{i,t}}, \\
\operatorname{SDR}_{i,t}
&= \frac{\operatorname{CurrentLiab}_{i,t}}{\operatorname{TotalLiab}_{i,t}}, \\
\operatorname{Cash}_{i,t}
&= \frac{\operatorname{CashEquivalent}_{i,t}}{\operatorname{TotalAsset}_{i,t}}, \\
\operatorname{ROA}_{i,t}
&= \frac{\operatorname{NetProfit}_{i,t}}{\operatorname{TotalAsset}_{i,t}}, \\
\operatorname{ROE}_{i,t}
&= \frac{\operatorname{NetProfit}_{i,t}}{\operatorname{Equity}_{i,t}}.
\end{aligned}
$$

股权集中度指标为：

$$
\operatorname{HHI5}_{i,t}
=
\sum_{k=1}^{5} s_{k,i,t}^{2},
$$

其中，$s_{k,i,t}$ 表示第 $k$ 大股东持股比例。若原始数据以百分数表示持股比例，例如 35 表示 35%，需先除以 100，再计算平方和。

### 2.4 离群值处理

对以下变量进行离群值处理：

```text
Lev, SL, LL, SDR, Cash, ROA, ROE, SLoan, LLoan, Top1, HHI5
```

要求：

- 默认按年度在 1% 和 99% 分位数进行缩尾处理(winsorize)。
- 可以选择其他缩尾比例，但必须说明理由。
- `Size` 和 `Age` 一般不需要缩尾；如处理，需说明理由。
- 同时保留原始变量和缩尾后变量。例如：
  - 原始变量：`Lev_raw`
  - 缩尾变量：`Lev`
- 在报告中说明缩尾前后均值、标准差和极值是否发生明显变化。

### 2.5 合并与输出

将资产负债表、利润表、股权结构、公司基本信息和行业信息合并为公司—年度面板数据。

要求：

- 合并主键为 `code-year`。
- 每次合并前后都记录行数变化。
- 在 `process_log.txt` 中记录关键处理步骤，例如读取成功、合并成功、删除重复值、处理缺失值、输出文件等。
- 最终保存以下文件：

```text
data/clean/firm_year_clean.csv
data/combined/csmar_firm_year_panel.csv
output/tables/missing_summary.csv
output/tables/winsor_summary.csv
```

---

## 第三部分：数据存储与管理

### 3.1 基础要求：CSV

所有清洗后的核心数据必须保存为 CSV 格式：

```text
data/clean/firm_year_clean.csv
data/combined/csmar_firm_year_panel.csv
```

在 `README.md` 中说明：

- CSV 格式的优点。
- CSV 在大规模财务数据库管理中的不足。
- 为什么本次作业仍然以 CSV 作为基础提交格式。

### 3.2 进阶要求：Parquet 或 SQLite 二选一

在完成 CSV 的基础上，从以下两种进阶格式中选择一种额外实现。

#### 方式 B：Parquet

将最终公司—年度面板数据额外保存为 Parquet 格式：

```text
data/combined/csmar_firm_year_panel.parquet
```

在 Notebook 中演示：

```python
import os
import time
import pandas as pd
import pyarrow.parquet as pq

# 只读取少数列，展示列式存储的优势
df_small = pd.read_parquet(
    "data/combined/csmar_firm_year_panel.parquet",
    columns=["code", "year", "Lev", "ROA", "Cash"]
)

# 查看 Schema
schema = pq.read_schema("data/combined/csmar_firm_year_panel.parquet")
print(schema)

# 比较 CSV 和 Parquet 的读取速度与文件体积
t0 = time.time()
pd.read_csv("data/combined/csmar_firm_year_panel.csv")
print(
    f"CSV 读取耗时: {time.time() - t0:.3f}s, "
    f"文件大小: {os.path.getsize('data/combined/csmar_firm_year_panel.csv') / 1024:.1f} KB"
)

t0 = time.time()
pd.read_parquet("data/combined/csmar_firm_year_panel.parquet")
print(
    f"Parquet 读取耗时: {time.time() - t0:.3f}s, "
    f"文件大小: {os.path.getsize('data/combined/csmar_firm_year_panel.parquet') / 1024:.1f} KB"
)
```

用文字回答：在本次数据规模下，两种格式的速度和体积差异是否明显？如果扩展到全部 A 股公司、季度数据或多个数据库合并，差异会如何变化？

#### 方式 C：SQLite

将最终公司—年度面板数据额外存入 SQLite 数据库：

```text
data/combined/csmar_finance.db
```

数据库至少包含 3 张表：

```sql
CREATE TABLE firm_year_finance (
    code TEXT,
    year INTEGER,
    Lev REAL,
    Cash REAL,
    ROA REAL,
    ROE REAL,
    Size REAL,
    Age REAL,
    PRIMARY KEY (code, year)
);

CREATE TABLE firm_info (
    code TEXT PRIMARY KEY,
    name TEXT,
    industry_code TEXT,
    industry_name TEXT,
    list_date TEXT
);

CREATE TABLE ownership (
    code TEXT,
    year INTEGER,
    Top1 REAL,
    HHI5 REAL,
    PRIMARY KEY (code, year)
);
```

在 Notebook 中演示至少 2 条具有实际业务含义的 SQL 查询，并说明用途。例如：

```sql
-- 查询各行业每年平均负债率最高的前 10 个行业—年份组合
SELECT industry_code, year, AVG(Lev) AS mean_lev
FROM firm_year_finance
GROUP BY industry_code, year
ORDER BY mean_lev DESC
LIMIT 10;
```

```sql
-- 查询第一大股东持股比例较高且现金比率较高的公司
SELECT f.code, i.name, f.year, f.Cash, o.Top1
FROM firm_year_finance f
LEFT JOIN ownership o
       ON f.code = o.code AND f.year = o.year
LEFT JOIN firm_info i
       ON f.code = i.code
WHERE f.Cash > 0.3 AND o.Top1 > 0.5
ORDER BY f.year, f.Cash DESC;
```

注意：`*.db` 文件不要上传 GitHub。在 `.gitignore` 中添加 `*.db`，并在 `README.md` 中说明如何从 Notebook 重建数据库。

---

## 第四部分：描述统计与可视化

在 `03_analysis.ipynb` 中完成以下分析。每张图后须有不少于 2 句文字解读，不能只展示图形。

### 4.1 年度描述统计

列表呈现以下指标在 2000 年至最新可得年份期间各年度的平均值、中位数、标准差、最小值和最大值，并作简要分析：

```text
Lev, SL, LL, SDR, Cash, ROA, ROE, SLoan, LLoan, Top1, HHI5, Size, Age
```

输出文件：

```text
output/tables/yearly_summary.csv
output/tables/yearly_summary.xlsx
```

建议表格结构为：

| 年份 | 变量 | 均值 | 中位数 | 标准差 | 最小值 | 最大值 | 样本量 |
|------|------|------|--------|--------|--------|--------|--------|
| 2000 | Lev  | ...  | ...    | ...    | ...    | ...    | ...    |

分析时至少回答：

- 哪些变量的年度均值变化较明显？
- 哪些变量的离散程度较大？
- 是否存在某些年份样本量明显偏少？可能原因是什么？
- 缩尾处理是否影响主要结论？

### 4.2 时序图

完成以下时序图，并保存至 `output/figures/`。

#### 图 1：Lev 的均值和中位数

- 横轴为年份。
- 纵轴为 `Lev`。
- 同一图中展示年度均值和年度中位数。
- 文件名：`fig01_lev_mean_median.png`。

分析要点：

- 均值和中位数之间是否存在稳定差距？
- 如果均值长期高于中位数，说明什么？
- 负债率是否存在明显的阶段性变化？

#### 图 2：ROA 和 Cash 的均值

- 横轴为年份。
- 纵轴为年度均值。
- 同一图中展示 `ROA` 和 `Cash` 的年度均值；如果量纲差异明显，可使用双纵坐标。
- 文件名：`fig02_roa_cash_mean.png`。

分析要点：

- 盈利能力和现金持有是否存在同步或反向变化？
- 哪些年份可能受到宏观冲击或市场环境变化影响？
- 仅凭该图能否推断因果关系？为什么？

---

## 第五部分：行业负债率特征分析

### 5.1 行业范围

分析以下行业：

| 行业代码 | 行业名称 |
|---------|---------|
| `C` | 制造业 |
| `D` | 电力、热力、燃气及水生产和供应业 |
| `G` | 交通运输、仓储和邮政业 |
| `E` | 建筑业 |
| `K` | 房地产业 |
| `F` | 批发和零售业 |
| `J` | 金融业 |

如果你的原始行业代码不是单字母行业大类，需要先提取行业大类代码。例如，`C39` 归入 `C`。

### 5.2 算术平均负债率

绘制上述行业在 2000 年至最新可得年份期间的年平均负债率时序图：

$$
\overline{\operatorname{Lev}}_{g,t}
=
\frac{1}{N_{g,t}}
\sum_{i \in g}
\operatorname{Lev}_{i,t}.
$$

要求：

- 横轴为年份。
- 纵轴为行业年度平均 `Lev`。
- 不同行业使用不同线型或颜色。
- 文件名：`fig03_industry_lev_equal_weight.png`。
- 图后作简要分析。

### 5.3 加权平均负债率

绘制上述行业在 2000 年至最新可得年份期间的年加权平均负债率时序图。默认使用总资产作为权重：

$$
\operatorname{WLev}_{g,t}
=
\sum_{i \in g}
w_{i,g,t}\operatorname{Lev}_{i,t},
\quad
w_{i,g,t}
=
\frac{\operatorname{TotalAsset}_{i,t}}{\sum_{j \in g}\operatorname{TotalAsset}_{j,t}}.
$$

要求：

- 权重可以选择公司总资产，也可以选择总市值；本作业推荐使用总资产。
- 如果使用总市值作为权重，需要说明总市值变量来源。
- 文件名：`fig04_industry_lev_asset_weighted.png`。
- 图后作简要分析。

### 5.4 两种算法比较

用文字回答：

- 算术平均和加权平均的经济含义分别是什么？
- 两张图中行业排序是否一致？
- 哪些行业的加权平均负债率明显高于算术平均负债率？这说明行业内大公司与小公司的杠杆结构有何差异？
- 在讨论行业整体债务风险时，哪一种算法更合理？为什么？

### 5.5 行业变量列表

呈现上述行业在 2001、2003、2005、2007、2009、2011、2013、2015、2017、2019、2021、2023 年度的以下变量均值，并作简要分析：

```text
SLoan, LLoan, Lev, Cash, ROA, ROE
```

输出文件：

```text
output/tables/industry_selected_years_summary.csv
output/tables/industry_selected_years_summary.xlsx
```

建议表格结构为：

| 年份 | 行业代码 | 行业名称 | 变量 | 均值 | 样本量 |
|------|---------|---------|------|------|--------|
| 2001 | C | 制造业 | Lev | ... | ... |

---

## 第六部分：股权结构分析

### 6.1 Top1 箱线图

绘制第一大股东持股比例 `Top1` 的年度箱线图。

要求：

- 横轴为年份。
- 纵轴为 `Top1`。
- 年份取值为：2001、2003、2005、2007、2009、2011、2013、2015、2017、2019、2021、2023。
- 文件名：`fig05_top1_boxplot_selected_years.png`。

### 6.2 分析问题

围绕 2005 年、2007 年和 2023 年的箱线图差异，回答：

- 三个年份的中位数、四分位距和极端值有何差异？
- 2005 年前后股权分置改革可能如何影响第一大股东持股比例分布？
- 2023 年与早期年份相比，上市公司股权结构是否更加分散？证据是什么？
- 仅根据箱线图能否判断控制权稳定性？还需要哪些补充指标？

---

## 第七部分：README.md 要求

`README.md` 须包含以下内容，可在此基础上扩展：

```markdown
## P02b：CSMAR 上市公司财务特征清洗与分析

### 数据来源

- 数据库：CSMAR(国泰安)
- 原始文件位置：data/data_raw_zip/
- 解压后文件位置：data/raw/
- 样本范围：A 股上市公司，2000 年至最新可得年份
- 数据频率：年度

### 原始文件说明

| 文件名 | 主要内容 | 使用变量 |
|--------|----------|----------|
| ...    | ...      | ...      |

### 变量构造说明

| 变量 | 含义 | 计算方式 | 原始变量来源 |
|------|------|----------|--------------|
| Lev  | 总负债率 | 总负债 / 总资产 | ... |
| ...  | ... | ... | ... |

### 数据清洗说明

- 样本筛选规则：……
- 重复值处理规则：……
- 缺失值处理规则：……
- 离群值处理规则：……
- 合并规则：……

### 存储方式

- 基础：CSV
- 进阶：Parquet / SQLite(二选一)
- 选择该进阶方式的理由：……

### GitHub 仓库

https://github.com/[你的用户名]/dshw-p02b

### 如何运行

1. 安装依赖：`pip install -r requirements.txt`
2. 将 CSMAR 原始压缩包放入 `data/data_raw_zip/`
3. 运行 `01_extract_raw_data.ipynb` 解压并识别原始数据
4. 运行 `02_clean_construct_variables.ipynb` 清洗数据并构造变量
5. 运行 `03_analysis.ipynb` 生成表格、图形和分析结果
6. 打开 `report.html` 阅读完整报告
```

---

## 第八部分：分析报告

除 Notebook 之外，须额外提交一份独立分析报告，格式为 `.html` 或 `.pdf`，文件名为 `report.html` 或 `report.pdf`，放置于项目根目录。

可由 Notebook 导出：

```bash
jupyter nbconvert --to html 03_analysis.ipynb --output report.html
```

或使用 Quarto 渲染：

```bash
quarto render report.qmd --to html
```

报告内容要求：

- 包含完整流程：数据来源、变量构造、清洗说明、年度统计、时序图、行业比较、股权结构分析和结论。
- 每个分析模块须有 Markdown 标题和解释文字，不能全是代码块。
- 每张图后须有不少于 2 句文字解读。
- 报告须能独立阅读：不打开 Notebook，读者也能理解数据来源、处理方法和主要发现。
- 关键表格和图形应嵌入报告正文，而不是只保存为外部文件。

---

## 第九部分：提交要求

本次作业须同时完成以下两种提交方式，缺一不可。

### 9.1 坚果云压缩包

将整个项目文件夹压缩为 `.zip`，命名格式：

```text
P02b_学号_姓名.zip
```

例如：

```text
P02b_20231001_张三.zip
```

上传至课程指定的坚果云共享文件夹。

### 9.2 GitHub 仓库

在个人 GitHub 账户下新建仓库，仓库名称建议为：

```text
dshw-p02b
```

使用 GitHub Desktop 将本地项目同步至该仓库，并将仓库地址写入 `README.md`。

### 9.3 `.gitignore` 建议配置

```gitignore
# CSMAR 原始数据和临时文件
data/data_raw_zip/
data/raw/
data/temp/

# 数据库文件
*.db

# Notebook 缓存
.ipynb_checkpoints/

# Python 缓存
__pycache__/

# 系统文件
.DS_Store
Thumbs.db
```

说明：是否上传 `data/clean/` 和 `data/combined/` 中的清洗后数据，由你自行决定；但必须在 `README.md` 中说明。

---

## 提交清单

提交前请逐项检查：

- [ ] 项目根目录名称为 `dshw-p02b`
- [ ] 目录结构由 Python 代码自动创建
- [ ] `README.md` 完整，含数据来源、原始文件说明、变量构造、清洗规则、存储方式、GitHub 仓库链接和运行步骤
- [ ] `requirements.txt` 存在，能够安装必要依赖
- [ ] `.gitignore` 配置正确，未上传 CSMAR 原始数据和数据库文件
- [ ] `01_extract_raw_data.ipynb` 能够自动解压并读取原始数据
- [ ] `02_clean_construct_variables.ipynb` 完成变量构造、缺失值处理、重复值处理、缩尾处理和多表合并
- [ ] `process_log.txt` 存在，记录关键处理步骤
- [ ] `data/dict/variable_dictionary.csv` 存在，变量来源说明清晰
- [ ] `data/combined/csmar_firm_year_panel.csv` 存在
- [ ] Parquet 或 SQLite 至少完成一种进阶存储方式
- [ ] 年度描述统计表已输出
- [ ] 图 1-5 均已完成并保存至 `output/figures/`
- [ ] 每张图后均有不少于 2 句文字解读
- [ ] 行业负债率的算术平均和加权平均均已分析
- [ ] 2005 年、2007 年和 2023 年的 `Top1` 箱线图差异已有文字解释
- [ ] `report.html` 或 `report.pdf` 存在于项目根目录，且可独立阅读
- [ ] GitHub 仓库已创建并同步，仓库地址已写入 `README.md`

---

## 评分标准

| 维度 | 分值 | 说明 |
|------|------|------|
| 数据准备与项目结构 | 15 分 | 目录规范，原始数据管理清晰，变量字典完整，日志记录规范 |
| 数据清洗与变量构造 | 30 分 | 主键统一、重复值处理、缺失值处理、变量公式、缩尾处理和多表合并均规范 |
| 描述统计与时序图 | 20 分 | 年度统计表完整，Lev、ROA、Cash 时序图规范，文字解读有实质内容 |
| 行业负债率分析 | 15 分 | 算术平均和加权平均负债率图形正确，行业比较和经济解释充分 |
| 股权结构分析 | 10 分 | Top1 箱线图规范，对 2005 年、2007 年和 2023 年差异解释合理 |
| 报告与可复现性 | 10 分 | 报告可独立阅读，Notebook 可从头运行，GitHub 和 README 说明清晰 |
| **加分项** | **+10 分** | 使用 Quarto 发布在线报告或电子书，排版整洁，链接可公开访问 |

> **核心提示**：本次作业的关键不是「把表格和图画出来」，而是把 CSMAR 原始数据整理成结构清晰、口径一致、可以复现的公司—年度面板。变量构造说明、清洗过程记录和图表后的解释文字，是评分最能拉开差距的地方。
