# 数据字典

生成时间：2026-05-24 02:12

## stock_daily
- 描述：个股日度行情
- 粒度：firm-date
- 主键：code + date
- 行数：15,100
- 字段：['date', 'open', 'high', 'low', 'close', 'volume', 'amount', 'turnover', 'code', 'name', 'pct_chg']

## index_daily
- 描述：市场指数日度
- 粒度：index-date
- 主键：index_code + date
- 行数：4,533
- 字段：['date', 'open', 'high', 'low', 'close', 'volume', 'amount', 'pct_chg', 'code']

## fin_annual
- 描述：年度财务指标
- 粒度：firm-year
- 主键：code + year
- 行数：60
- 字段：['code', 'name', 'year', 'roe', 'net_profit_margin', 'revenue_yoy', 'profit_yoy', 'debt_ratio', 'current_ratio', 'asset_turnover']

## company_info
- 描述：公司基本信息
- 粒度：firm
- 主键：code
- 行数：10
- 字段：['code', 'name']

## shibor_3m
- 描述：Shibor 3 个月期
- 粒度：monthly
- 主键：date
- 行数：75
- 字段：['date', 'shibor_3m']

## usd_cny
- 描述：人民币兑美元
- 粒度：monthly
- 主键：date
- 行数：74
- 字段：['date', 'usd_cny']

## cpi_monthly
- 描述：CPI 月度同比
- 粒度：monthly
- 主键：date
- 行数：74
- 字段：['date', 'cpi_yoy']
