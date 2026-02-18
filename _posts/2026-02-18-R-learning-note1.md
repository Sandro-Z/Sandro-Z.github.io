---
title: "R语言学习1--数据可视化"
date: 2026-02-15 10:17:00 +0800
excerpt_separator: "<!--more-->"
last_modified_at: 2026-02-18T17:13:02-05:00
categories:
  - Data Science
tags:
  - R
---

# ggplot2绘图基本概念

ggplot2绘图是以“图层”的形式逐层进行绘制的。一般的，绘图时用`ggplot()`函数定义所用数据集和坐标，确定底图；`geom_point()`或者其他类型图对应的函数绘制图的主体；`geom_smooth`绘制趋势线；`labs`更改图的风格，颜色，坐标名，图例等。

示例：
```
ggplot(
  data = penguins,
  mapping = aes(x = flipper_length_mm, y = body_mass_g)
) +
  geom_point(aes(color = species, shape = species)) +
  geom_smooth(method = "lm") +
  labs(
    title = "Body mass and flipper length",
    subtitle = "Dimensions for Adelie, Chinstrap, and Gentoo Penguins",
    x = "Flipper length (mm)", y = "Body mass (g)",
    color = "Species", shape = "Species"
  ) +
  scale_color_colorblind()
```

可绘制出下面的图像。

![fig1](/assets/images/unnamed-chunk-14-1.png)

* 注意mapping的定义域：若在最开始的ggplot中定义某些参数，则会全局mapping该维度，可能会影响趋势线等。

例：

在全局map了species：

```
ggplot(
  data = penguins,
  mapping = aes(x = flipper_length_mm, y = body_mass_g, color = species)
) +
  geom_point() +
  geom_smooth(method = "lm")
```

geom_smooth会在全局产生三条趋势线。

![fig2](/assets/images/unnamed-chunk-11-1.png)

而如果只在point图层map species：

```
ggplot(
  data = penguins,
  mapping = aes(x = flipper_length_mm, y = body_mass_g)
) +
  geom_point(mapping = aes(color = species)) +
  geom_smooth(method = "lm")
```

![fig3](/assets/images/unnamed-chunk-12-1.png)

由于species这个维度没在全局map，因此smooth方法没有按照此维度平滑，最终只形成一条曲线。

# 绘制常见图形的方法

`geom_bar()`绘制柱形图，坐标使用`fct_infreq()`可以按照频率大小排序

`geom_histogram()`直方图，`geom_density()`频率分布图

`geom_boxplot()`箱线图，`geom_bar()`堆栈条形图看比例

`facet_wrap()`可以将散点图按照某个维度拆分

# 保存图形
刚刚生成在工作区的图形，可以用`ggsave()`保存到工作目录。（查看工作目录可以使用`getwd()`）