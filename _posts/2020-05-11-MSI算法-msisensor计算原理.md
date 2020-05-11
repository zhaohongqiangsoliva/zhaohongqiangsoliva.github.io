---
layout:     post
title:      MSI算法
subtitle:   msisensor计算原理
date:       2020-5-11
author:     BY
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - MSI
---
# MSI
## msisensor 计算msi原理
 MSI突变中重复至少5次的1-5个碱基的序列。
 然后，使用MSIsensor msi命令，
 通过使用卡方检验比较肿瘤和正常样品之间的读取长度分布来估计每个微卫星位点
 （具有≥20个映射读数）和每个肿瘤/正常组织对的突变状态。
 P从MSIsensor输出中提取每个微卫星的值并用于定义微卫星突变状态
 （如果P <.05，则认为微卫星是突变的）
 msisensor-dis文件中标记详细信息
 ```
   T0000110000
   N0000110000
 ```
 T=1,1
 N=1,1
msi 计算每条reads 的 重复数量
对比T/N    (T-N)^2/N =X^2  得出 卡方值，结果对应表得出Pvalue
直接用python scipy包计算就可以
