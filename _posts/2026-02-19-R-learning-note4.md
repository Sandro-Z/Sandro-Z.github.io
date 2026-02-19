---
title: "R语言学习4--数据导入"
date: 2026-02-19 22:00:00 +0800
excerpt_separator: "<!--more-->"
last_modified_at: 2026-02-19T22:00:00+08:00
categories:
  - Data Science
tags:
  - R
  - Data tidying
---

# 基本导入及问题修复
用`read_csv()`函数可以将csv文件*粗略地*转换为R中的数据框格式。但是，有很多问题R不能自动解决，需要人工分析。

* 多种格式书写的缺失值无法自动判别->显式指定`na=c()`

* 列名不规范:跳过（`skip=n`）或首行不做列名（`col_names=FALSE`）,或者`janitor::clean_names()`进行启发式命名格式整理

* 列名推断：

`read_csv()`默认的推断逻辑是：

> Does it contain only F, T, FALSE, or TRUE (ignoring case)? If so, it’s a logical.
> Does it contain only numbers (e.g., 1, -4.5, 5e6, Inf)? If so, it’s a number.
> Does it match the ISO8601 standard? If so, it’s a date or date-time. 
> Otherwise, it must be a string.

如：
```
read_csv("
  logical,numeric,date,string
  TRUE,1,2021-01-15,abc
  false,4.5,2021-02-15,def
  T,Inf,2021-02-16,ghi
")
#> # A tibble: 3 × 4
#>   logical numeric date       string
#>   <lgl>     <dbl> <date>     <chr> 
#> 1 TRUE        1   2021-01-15 abc   
#> 2 FALSE       4.5 2021-02-15 def   
#> 3 TRUE      Inf   2021-02-16 ghi
```
如此逻辑导入的数据出现问题，用`col_types`强制指定。

# 写数据
`write_csv()`会将数据写成纯文本的csv格式。这个过程会损失所有的R数据结构中的列类别等信息。

可以写成rds或者parquet格式，通用且能够保留数据结构内部信息。

*注：由于AI编程的趋势，无必要死记细节的语法，故本文较为简洁。重点在于处理**非正常**数据时能够发现问题并随机应变。文件导入过程的调整和数据清洗过程相辅相成，实践时应该注意。*