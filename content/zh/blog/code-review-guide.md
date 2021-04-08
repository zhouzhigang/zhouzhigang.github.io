---
title: "代码Review指南"
linkTitle: "代码Review指南"
date: 2016-07-05
description: >
  代码Review原则、关注点
---

## 目标

* 当你知道你的每一行代码都有另外一个人看，会促进自己写更优雅的代码
*   及早发现潜在缺陷与BUG，降低事故成本
*   经验传承，促进团队内部知识共享，提高团队整体水平
*   可以保证至少有两个人都理解任何一份代码

## How

思路: 鼓励Code Review文化，利用工具提高效率；

*   利用AK客户端进行事前Review
    * Google，Facebook都严格执行`pre-commit code review`
* 尽量分批Review
*   结合`Task`让`Commit Message`规范化(也利于Review)

### 操作指南

Review 工具 / IDEA 插件安装（略）

### 主要Review的内容/关注点

+ 功能性
  - 为什么变更？是否实现了相关需求？
  - 是否有测试来保证代码功能的正确性？
+ 设计
  - 主要接口的设计(接口要有注释)
  - 是否在合适的位置？
  - 逻辑是否正确
  - 复用性？是否复用了老代码？或提供新的基础设施？
+ 可维护性
  - 命名是否反应了要做的事？
  - 代码意图是否清晰？
  - 是否有巨大的类或复杂的实现？
  - 测试代码是否覆盖了主要case？包括正常和异常路径
+ 健壮性
  - 验证
  - 异常处理等
+ 性能、安全等
+ 编码风格
  - 一般交给代码检测工具


## 参考
* [Java Code Review清单](http://www.importnew.com/12511.html)
* [What to look for in a Code Review](https://blog.jetbrains.com/upsource/2015/07/23/what-to-look-for-in-a-code-review/) 