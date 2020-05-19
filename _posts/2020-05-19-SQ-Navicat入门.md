---
layout:     post
title:      SQL-入门教程
subtitle:   SQL 入门教程-自用
date:       2020-05-19
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - SQL
---
### 创建数据库文件
文件-新建连接-sqllite-新建sqllite3-数据库文件-选择存放位置
### 导入数据
下载文件
这是数据分析师的excel实战数据的下载地址：

https://pan.baidu.com/s/1eUjcGaI

提取密码：g5xa

![](https://upload-images.jianshu.io/upload_images/15500891-9477d45a1c797271.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选择编码
gb2312-windows电脑数据文件
utf-8 linux电脑数据文件
![image.png](https://upload-images.jianshu.io/upload_images/15500891-18381028b0550186.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/15500891-1e4dfaf953a55fc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/15500891-75844065213089f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### table是表名，column是我们想要查询的字段／列，column可以用 * 代替，指代全部字段，意为从table表查询所有数据。
```
select column from table
```
### where 是基础查询语法，用于条件判断。
```
select * from DataAnalyst
where city = '上海'
```
### 上图是最简化的查询语句，将所有城市为上海的职位数据过滤出来。我们也可以用 and 进行多条件判断。
```
select * from DataAnalyst
where city = '上海' and positionName = '数据分析师'
```

当我们涉及到非常复杂的与或逻辑判断，应该怎么办？比如即满足条件AB，又要满足条件C，或者是满足条件DE。此时需要用括号明确逻辑判断的优先级。


```
select * from DataAnalyst

where (city = '上海' and positionName = '数据分析师') or (city = '北京' and positionName = '数据产品经理')
```

### 接下来的问题来了，当我们要查询多个条件，比如北京上海广州深圳南京这些城市，难道一个个用and关联起来？这太麻烦了，我们可以使用 in 。

```
select * from DataAnalyst
where city in ('北京','上海','广州','深圳','南京')
```

### 当我们遇到字段数据类型是数值时，也可以使用符号> 、>=、< 、<=、!= 进行逻辑判断，!= 指的是不等于，等价于 <> 。


```
select * from DataAnalyst
where companyId >= 10000
```

### 当我们需要取区间数值时，使用 between and


```
select * from DataAnalyst
where companyId between 10000 and 20000
```

between and 包括数值两端的边界，等同于 companyId >=10000 and companyId <= 20000。



### 如果要模糊查找，能用like。
```
select * from DataAnalyst
where positionName like '%数据分析%'
```
语句的含义是在positionName列查找包含「数据分析」字段的数据，%代表的是通配符，含义是无所谓「数据分析」前面后面是什么内容。如果是 '数据分析%' ，则代表字段必须以数据分析开头，无所谓后面是什么。

### group by，它是数据分析中常见的语法，目的是将数据按组／维度划分。类似于Excel中的数据透视表，我们以city为例。


```
select * from DataAnalyst
group by city
```
![image.png](https://upload-images.jianshu.io/upload_images/15500891-f5211522cf114a2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 它将城市划分成几组，通过group by 可以快速的浏览数据有哪些城市。我们看一下它的高阶用法。


```
select city,count(1) from DataAnalyst
group by city
```
![image.png](https://upload-images.jianshu.io/upload_images/15500891-4b6f33ead3832b98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 上述语句，使用count函数，统计计数了每个城市拥有的职位数量。括号里面的1代表以第一列为计数标准。这里出现新的问题，当我们遇到重复数据怎么办？在DataAnalyst 这张表中，北京职位包含重复的职位ID，我们需要去重。


```
select city,count(distinct positionId) from DataAnalyst
group by city
```
![image.png](https://upload-images.jianshu.io/upload_images/15500891-557d1ffbae282827.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
北京的数据一下子少了2000，多余的重复值被排除在外。distinct 是去重函数，distinct positionId 会只计算唯一的positionId个数。日常工作中，活跃用户数、文章UV，都是用distinct 计算获得，这是唯一标示符ID的重要作用。
### 除了count，还有max，min，sum，avg等函数，也叫做聚合函数。用法和Excel没什么区别。



当我们在group by 添加多个字段，它将以多维的形式进行数据聚合。


```
select city,workYear,count(distinct positionId) from DataAnalyst
group by city,workYear
```

### 接下来是新的问题，如果我想找出各个城市，数据分析师岗位数量在500以上的城市有哪些，应该怎么计算？有两种方法，第一种，是使用having语句，它对聚合后的数据结果进行过滤。


```
select city,count(distinct positionId) from DataAnalyst
group by city having count(distinct positionId) >= 500
```
![image.png](https://upload-images.jianshu.io/upload_images/15500891-cf19ee6749a57169.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二种，是利用嵌套子查询。

![image](https://upload-images.jianshu.io/upload_images/15500891-8e5a53f2a0f6f841.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们将第一次查询获得的城市职位数的结果，看作一张新的表，利用as 将它命名为t1( table1 的简写)，将职位数命名为一个新的字段counts。然后外面再套一层select 过滤出counts >=500。

这种查询方式就叫嵌套子查询，使用场景比较广泛，where 后面也能跟子查询。

很多时候，数据是凌乱的，我们希望结果能够呈现一定的顺序，这时候就用到order by语句。
```
select city,count(distinct positionId) as counts from DataAnalyst
group by city
order by counts
```
