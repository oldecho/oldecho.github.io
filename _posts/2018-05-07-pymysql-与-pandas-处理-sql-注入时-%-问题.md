---
title: pymysql 与 pandas 处理 sql 注入时 % 问题
date: 2018-05-07 23:30:09
categories:
- python
tag:
- python
- pymysql
- pandas
---

使用 pymysql 和 pandas 插入数据库时，两者都可以匹配参数

**pymysql**

```python
cursor.execute(query, args=None)
```

**pandas**

```python
"""Read SQL query or database table into a DataFrame."""
pandas.read_sql(query, con, index_col=None, coerce_float=True, params=None, parse_dates=None, columns=None, chunksize=None)
```

**相同点**

会将 `args` 中参数转成 `string` ，也就是说 `query` 中应该接收 `%s` 

`query` 中不用加引号，会自动补上

**不同点**

使用 pandas 时，参数中包含 `%` 时需要转为 `%%` ，pymsql 则不需要

