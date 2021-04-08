---
title: "数据驱动买家增长与转化"
linkTitle: "数据驱动买家增长与转化"
date: 2020-10-31
description: >
  为了更好的从全局角度来考虑，避免单点/非核心路径/顾此失彼之类的错误，本来将从流程的视角来对O-P进行具体的分析。
---

## 大纲

> 核心思路：从数据现象出发，分析内部组成结构和外部影响因素，提出假设，再用数据验证假设；迭代；最后根据影响因素预测趋势及优化建议。

+ 核心数据总览：明确分析及优化方向
    - 核心指标、趋势、对比
    - 组成结构: 人及核心行为、终端、国家、行业…
+ 分析影响因素：确定驱动因子/增长动力
    - 买家：新用户、需求不符（价格、时效、服务）
    - 平台：通知召回率、支付能力、物流能力、体验
    - 卖家：接单能力、需求不匹配
    - 取消原因及影响
+ 优化方案及落地规划
    - 业务（What->Why->Predict->How）、技术（手工->自动->智能）

## 一、数据总览：明确分析及优化方向

看数据现状的两个基本视角：

+ 数据是什么，好 or 坏？评价标准/参考体系（如均值）
+ 变化趋势(时间)：整体是趋势，变好 or 变坏？

为进一步分析数据为什么会这样，可以从静态组成结构和动态过程进一步分析

+ 静态**结构**视角：不同维度（Dim，如订单、买家、业务类型...）汇总；结构占比变化会影响整体数值（第一序改变？）
    + 宏观上明确优化方向，及具体影响权重占比；避免抓不到重点，在无关大局的部份花过多时间
        + 比如某种类型订单占比很小，是否应该直接提升其比例，还是当前阶段无关紧要，直接忽略
    + 订单或买家等粒度太细，对个例进行分析时可以采用；大多数时候可以按业务类型进一步细分
+ 动态**过程**视角：入口UV逐步转化（ConverationRate）为最终结果；有哪些步骤或影响因素导致了最终的结果（第二序改变）


![一、数据总览：明确分析及优化方向](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/f6e4498c89f06b341c05d009b990ec19.png)

同GMV，转化率同样可以按类型来拆分看具体的组成结构及影响权重。

可以从人、货、场多个维度进行分析

![人及其核心行为](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/5f8edaa8803f7fc3547946423ef7da24.png)

![终端类型](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/41c2c9411e4a18911ccda343ae44991e.png)

![国家维度](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/3361bc72c8c7958108d13c30c0cbb0f8.png)

![行业维度](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/4f9ae8b374caf27a8af6e003e2b12a8e.png)

## 二、分析影响因素：确定驱动因子/增长动力

![二、分析影响因素：确定驱动因子/增长动力](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/f0f5a7b5c4257276aedac1c9e59bbdc3.png)

![新用户占比](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/5c782907707f79bdb1539dcfb312621c.png)

![通知召回率](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/84cd546b81c78fc0054cb3af78a7848b.png)

![人均订单太多](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/62ade2b274ded97441f338713f79215f.png)

![订单金额及匹配度](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/688695e0c70af4180937e1e0df7ced4b.png)

## 三、优化方案及落地规划

有不少人把业务和技术混杂在一起谈发展阶段与方向。但在我看来，至少可以有业务和技术两个维度，然后结合具体的业务组合来看具体需要落地的事情。

+ 业务：What（描述数据是什么） -> Why（描述原因是什么、影响因素及影响权重） -> Predict（预测未来趋势） -> How（优化或改变）
    + 前提：数据指标体系；如果能量化关系或公式最好
+ 技术：手工 -> 自动 -> 智能
    + 前提：数据模型的建立、数据的清洗等

回到具体的O-P转化上，

...
