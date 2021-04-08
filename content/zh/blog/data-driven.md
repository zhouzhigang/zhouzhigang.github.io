---
title: "从流程角度看交易数据"
linkTitle: "从流程角度看交易数据"
date: 2020-06-18
description: >
  通过梳理交易流程，建立合理指标体系；并进一步分析现状及问题；最后提出可落地的业务/技术方案
---

## 目的及思路

### 目的

- 梳理交易流程，建立合理指标体系
- 通过数据分析现状、问题
- 提供可落地的业务/技术方案

### 思路

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/9148/1572245337095-bbe8611f-8305-49d1-b567-8606891bc599.png#align=left&display=inline&height=158&name=image.png&originHeight=158&originWidth=716&size=29361&status=done&width=716)

## 指标体系构建

### 战略地图：宏观角度看网站核心数据
> 网站核心指标大盘以及内在逻辑关系


- 对企业而言，是需要获取**利润**的，而利润的来源，可以分为营收的增长，和成本的降低；
   - **营收的增长**，可以从人货场服务多个角度去考虑，比如提供更丰富的商品，吸引更多用户，延展到全球的市场，深入到行业，优化现有服务的质量与效率，或者提供新的服务等；
   - **成本的降低**则可以通过一些生产力的提升来达到降本提效的效果。
- 不过，我们提供了这些人货场以及相关的服务，客户也不一定会来用，所以从本质上来讲，我们一定要创造**客户价值**，这个也是我们应该长期关注的指标。
   - 买家作为需求的发起方和第一客户，基本的诉求有资金的安全、货品的质量、按时履约等。此外，和竞争对手，我们需要提供一定的差异化的价值，比如精准寻源
   - 类似的，对卖家而言。。。
- 关于数据指标，我认为有三个基本的维度，第一个维度是规模相关（做的多大，还有多少空间），第二个是质量相关（做的好不好，效率如何，客户满不满意），第三个则是转化相关（我们具体哪一步做的不好）
   - Note: 也有其他的指标分类维度，如基于用户留存的AARRR等。我们这里偏整体而非用户

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/9148/1572229435904-339cc51e-21bd-4d5f-9ee0-92c7696d148d.png#align=left&display=inline&height=387&name=image.png&originHeight=387&originWidth=835&size=93398&status=done&width=835)

Note: 各个指标存在一些动态的平衡（各个维度兼顾），**第一关键指标（One Metric That Matter，OMTM）**并不是绝对的，需要看**发展阶段及当前重点**。例如之前由于立足中国，不熟悉买家，所以更多的是从卖家价值出发，发展前期，更注重规模，所以重点在GMV；当前阶段更注重买家NPS等。

### 业务流程：微观角度看交易相关指标
> 流程的展开(全景可视&可缩放)：与指标关联（数据从哪里来，到哪里去，关注什么）

- 从交易流程来看，整个流程有准入、下单、支付、交付、售后等流程
   - 流程域关注的是**整体信息流、资金流、物流的编排与协同**，比如什么时候可以支付、什么时候可以交付，先货后款 or 先款后货，有没有多个批次等
- 各子域有各自的子流程

- 每个域同样可以从规模、质量、转化三个角度去思考关注哪些指标
   - 对于流程域来说，关注的更多的是上下游及各域间的协同，重点关注**履行质量（完美订单率、订单周期）**和**各环节转化率**
      - 交易规模是一个结果指标，并不是流程最关注的，大概了解下即可；不过**影响因素和结构**需要关注下（不然无法确保同一维度比较以及分析，后面细讲）

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/9148/1572404120719-e7c76471-1c2d-4a7e-a056-00708040e78f.png#align=left&display=inline&height=383&name=image.png&originHeight=383&originWidth=862&size=86746&status=done&width=862)

接下来就是流程更细独立的展开，描述**每一步骤应该做什么**（**后面具体分析的时候需要**），以及对应的数据，同样可以分为三个层次

- 第一个层次是偏整体流程的描述
- 第二个层次是流程相关的变体，方便扩展和调整
- 第三个层次是具体的步骤；涉及到子流程的，可按上述思路继续拆分

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/9148/1572404137610-374c3d59-321f-4fa0-bce8-e12f25ba1808.png#align=left&display=inline&height=449&name=image.png&originHeight=449&originWidth=921&size=112525&status=done&width=921)

注：其实这里两张图想并成一张图来画的，不过第二张图如果所有细节画出来的话会比较复杂；所以拆成两个视角，第一个偏宏观流程的规划（域间协同），第二个偏微观层次的流程（具体的步骤，分析转化等具体问题用这个视角）。

## 数据解读

### 交易规模解读

- 现状：标准，对比参照物
   - 目标对比：完成进度
   - 自身对比：增长趋势，年/月
   - 外部对比：集团内，集团外，增长空间
- 原因分析：
   - e.g GMV $62.3亿：好 or 坏？原因是什么？对策是什么？
   - e.g 小单化：好 or 坏？原因是什么？对策是什么？

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/9148/1572404914798-417c60e3-6eca-4cf4-a6c9-e71dd79903e5.png#align=left&display=inline&height=448&name=image.png&originHeight=448&originWidth=869&size=134982&status=done&width=869)

#### 组成结构分析

- Why
   - 实际是期望找到背后的各种影响因素
      - 找出背后原因后，寻找对应的策略，如拉新、留存、重点选择哪些品和市场等（非我们的重点）
   - 数据分析时，**需要同一维度，或者固定因变量进行对比**（**我们的重点**）
      - 存在差异性与共性，总体的平均值不一定可靠
         - 我和马老师平均工资
         - BuyNow与询盘订单差异较大
      - 对比时区分出哪些是相同因素，哪些是不同因素
         - 美国和印度订单转化率对比，直接比较的化有可能两者用户群体或者习惯都不一定导致错误的结论
- 如何拆分
   - 人货场各个维度 **MECE**
   - 这里只列举出部分我们常用的一些维度
- 多维度如何处理
   - 每个维度多个数值，2*2*...*256*300，组合爆炸
   - 人工智能算法
      - 不是万能的，可以做一些聚类、关联分析，大部分时候**无法解释原因**，还是需要结合业务特性去分析
   - 降维、剪枝（**找本质与重点**）
      - 影响不大的维度直接剔除
         - 这个需要看场景（分析什么内容），很多场景
      - 类似的维度合并
         - 

      - 非重点分支移除
         - e.g. 国家和行业选取规模最大和增速最快的分析（或者分类为成熟国家、新兴国家等）

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/9148/1572404156484-ca6fcd32-eb20-44de-a46e-7f66e5ba529a.png#align=left&display=inline&height=439&name=image.png&originHeight=439&originWidth=881&size=93756&status=done&width=881)

- 交易组成结构数据解读
   - 


![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/9148/1572414462137-5a564047-848d-4fff-b939-d8269d3e1b40.png#align=left&display=inline&height=434&name=image.png&originHeight=434&originWidth=951&size=230989&status=done&width=951)

### 交易质量解读
Note: 选取的人群、市场依赖于前面的结构分析

#### 完美订单率分析
完美订单率 = 无求助率*无支付失败率*按时发货率*无纠纷率

#### 履约周期分析
履约周期 = 下单周期 + 支付周期 + 履约周期

### 交易转化解读
L-D-C-F-O-P
交易核心: AB-O-P

#### AB-O分析

#### O-P分析

## 未来规划

- 核心问题对应的策略
- 流程规划
   - 透明度：全景可视（分层、粒度、解耦）；建立与数据的关联；当前存在较多问题导致难以可视化
      - [https://yuque.antfin-inc.com/chenmu.zzg/cmgqlw/yf36v8](https://yuque.antfin-inc.com/chenmu.zzg/cmgqlw/yf36v8)
- 数据应用矩阵
   - 横坐标，交易核心能力（流程）
   - 纵坐标：原因（影响因素分析）；监控预警；推荐；预测；