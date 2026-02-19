---
title: "R语言学习2--数据变换"
date: 2026-02-19 09:00:00 +0800
excerpt_separator: "<!--more-->"
last_modified_at: 2026-02-19T09:00:00-05:00
categories:
  - Data Science
tags:
  - R
---

# 流水线（pipe）
数据变换也可以类似ggplot2那样逐步进行，每一步的输出作为下一步的输入。不一样的是，ggplot2的图层之间叠加用'+'号表示，而数据变换每步之间使用'|>'管道符表示。

例：
```
flights |> 
  filter(dest == "IAH") |> 
  mutate(speed = distance / air_time * 60) |> 
  select(year:day, dep_time, carrier, flight, speed) |> 
  arrange(desc(speed))
```

# 行操作
* `filter()`

  允许按照行的各个特性（列的值）筛选行。

  常用条件表：
  
| 条件符 | 条件|
|--------|-----|
| %in%  | 值在某个集合中 |
| & | 且 |
| 丨 | 或 |
|其余条件符和python一样| |

* `arrange()`

  允许按照一列或者多列的值进行行的排序。

  可配合`desc()`进行倒序排列。

* `distinct()`

  按列提取出数据集中所有与众不同的行。提取的一定是首次出现的行。

# 列操作
* `mutate()`

  按照输入的变量名和表达式，向表中加入指定的列。

  可以使用`.before`, `.after`参数确定位置，`.keep`参数决定是否留下其余的列

  比如：

  ```
  flights |> 
  mutate(
    gain = dep_delay - arr_delay,
    hours = air_time / 60,
    gain_per_hour = gain / hours
  )
  ```
  会留下所有的列，以及新加入的列：
  ```
    year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time arr_delay
   <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>     <dbl>
  1  2013     1     1      517            515         2      830            819        11
  2  2013     1     1      533            529         4      850            830        20
  3  2013     1     1      542            540         2      923            850        33
  4  2013     1     1      544            545        -1     1004           1022       -18
  5  2013     1     1      554            600        -6      812            837       -25
  6  2013     1     1      554            558        -4      740            728        12
  7  2013     1     1      555            600        -5      913            854        19
  8  2013     1     1      557            600        -3      709            723       -14
  9  2013     1     1      557            600        -3      838            846        -8
  10  2013     1     1      558            600        -2      753            745         8
  336,766 more rows
  ```
  而加上.keep 参数后：
  ```
  flights |> 
  mutate(
    gain = dep_delay - arr_delay,
    hours = air_time / 60,
    gain_per_hour = gain / hours,
    .keep = "used"
  )
  ```
  便会只留下计算过程中用到的行。
```
  dep_delay arr_delay air_time  gain hours gain_per_hour
       <dbl>     <dbl>    <dbl> <dbl> <dbl>         <dbl>
 1         2        11      227    -9 3.78          -2.38
 2         4        20      227   -16 3.78          -4.23
 3         2        33      160   -31 2.67         -11.6 
 4        -1       -18      183    17 3.05           5.57
 5        -6       -25      116    19 1.93           9.83
 6        -4        12      150   -16 2.5           -6.4 
 7        -5        19      158   -24 2.63          -9.11
 8        -3       -14       53    11 0.883         12.5 
 9        -3        -8      140     5 2.33           2.14
10        -2         8      138   -10 2.3           -4.35
```

* `select()`

按照列名进行选择，留下指定列。

* `rename()`

重命名列。`rename(new_name=oldname)`

* `relocate()`

将列移动到指定位置。默认会将其移动至最前方，可以使用`.before`和`.after`参数指定移动的具体位置。

移动目标可以是一行，也可以是使用`a:b`框选出的n行。R语言中，‘：’冒号一般用于表达区间。

# 组操作
* `group_by()`和`summarize()`

  两者通常配合使用。

  `group_by()`会为数据结构添加分组信息。如：
  ```
  flights |> 
  group_by(month)
  ```
  注意下面的`# Groups:   month [12]`，这表明该数据结构现在拥有了分组信息，分了12组，每组为一个月份：
```
  # A tibble: 336,776 × 19
# Groups:   month [12]
    year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time arr_delay
   <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>     <dbl>
 1  2013     1     1      517            515         2      830            819        11
 2  2013     1     1      533            529         4      850            830        20
 3  2013     1     1      542            540         2      923            850        33
 4  2013     1     1      544            545        -1     1004           1022       -18
 5  2013     1     1      554            600        -6      812            837       -25
 6  2013     1     1      554            558        -4      740            728        12
 7  2013     1     1      555            600        -5      913            854        19
 8  2013     1     1      557            600        -3      709            723       -14
 9  2013     1     1      557            600        -3      838            846        -8
10  2013     1     1      558            600        -2      753            745         8
# ℹ 336,766 more rows
```

`summarize()`的作用：将多行数据按照分组压缩成一行，压缩依据传递给该函数的表达式进行

```
flights |> 
  group_by(month) |> 
  summarize(
    avg_delay = mean(dep_delay,na.rm=TRUE)
  )
```
输出：
```
# A tibble: 12 × 2
   month avg_delay
   <int>     <dbl>
 1     1     10.0 
 2     2     10.8 
 3     3     13.2 
 4     4     13.9 
 5     5     13.0 
 6     6     20.8 
 7     7     21.7 
 8     8     12.6 
 9     9      6.72
10    10      6.24
11    11      5.44
12    12     16.6 
```

**slice_...()函数可以从每组中选出某些元素进行计算**

**当使用多列进行分组的时候，每次使用`summarize()`都会去除一次分组**

* `ungroup()`

  用于将分组信息从一个数据结构移除。

* `.by`参数

  新语法，用于替代`group_by()`函数。例如，下面两种写法等价：
```
flights |> 
  summarize(
    delay = mean(dep_delay, na.rm = TRUE), 
    n = n(),
    .by = month
  )
```
```
flights |> 
  group_by(month) |> 
  summarize(
    avg_delay = mean(dep_delay,na.rm=TRUE)
  )
```  

## Tips
当使用`summarize`时，可以同时使用`n()`函数打印每组的元素数。在统计实践中，此值可以避免使用了过小的样本导致统计出现偏差。