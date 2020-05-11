---
layout:     post
title:      Strand bias 链偏好
subtitle:   Strand bias 链偏好
date:       2020-05-11
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - 生信算法
---
# Strand bias 链偏好
## 背景
![GATK.png](https://upload-images.jianshu.io/upload_images/15500891-64fe39367e89e6bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 造成链偏好的原因：
1.Local realignment
目的就是将比对到indel附近的reads进行局部重新比对，将比对的错误率降到最低。一般来说，绝大部分需要进行重新比对的基因组区域，都是因为插入/缺失的存在，因为在indel附近的比对会出现大量的碱基错配，这些碱基的错配很容易被误认为SNP。还有，在比对过程中，比对算法对于每一条read的处理都是独立的，不可能同时把多条reads与参考基因组比对来排错。因此，即使有一些reads能够正确的比对到indel，但那些恰恰比对到indel开始或者结束位置的read也会有很高的比对错误率，这都是需要重新比对的。Local realignment就是将由indel导致错配的区域进行重新比对，将indel附近的比对错误率降到最低。

![GATK-realignment.png](https://upload-images.jianshu.io/upload_images/15500891-34bd5e2375fb8fb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



2.BAQ（gatk BQSR）
对bam文件里reads的碱基质量值进行重新校正，使最后输出的bam文件中reads中碱基的质量值能够更加接近真实的与参考基因组之间错配的概率
在reads碱基质量值被校正之前，我们要保留质量值在Q25以上的碱基，但是实际上质量值在Q25的这些碱基的错误率在1%，也就是说质量值只有Q20，这样就会对后续的变异检测的可信度造成影响。还有，在边合成边测序的测序过程中，在reads末端碱基的错误率往往要比起始部位更高。另外，AC的质量值往往要低于TG。BQSR的就是要对这些质量值进行校正。

当两者同时出现的时候，就会产生大量的链偏好性

```
Chr	Pos		Depth	a1	b2	c3	d4	Forward Strand Genotype	Reverse Strand Genotype
6	32975014	21	5	5	10	1	Heterzygous		Homozygous
1	81967962	38	20	11	7	0	Heterzygous		Homozygous
12	10215654	31	15	9	7	0	Heterzygous		Homozygous
1. Forward strand reference allele.
2. Forward strand non reference allele.
3. Reverse strand reference allele.
4. Reverse strand non reference allele.
```
### 算法
#### SB算法

![SB.png](https://upload-images.jianshu.io/upload_images/15500891-2ca4f246e643975f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### gatk-SB
![GATK-SB.png](https://upload-images.jianshu.io/upload_images/15500891-78517a29149056c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### fisher检验算法范围（0-1）
![FISHER.png](https://upload-images.jianshu.io/upload_images/15500891-9edbc217e261bb10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Pvalue=1-p



#### 三者结果越大越好