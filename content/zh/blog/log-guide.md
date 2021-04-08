---
title: "日志规范"
linkTitle: "日志规范"
date: 2016-05-23
description: >
  日志问题、原则、规范
---

## 目前主要存在的问题
+ 日志级别随便用
    - 很多ERROR, WARN级别的日志居然是起到信息记录作用的
+ 打印格式不规范，如缺少必要参数、未打印异常堆栈等
	- 缺少上下文信息，如traceId、问题定位必要的入参、出参
+ 日志中有很多干扰信息，如一些乱七八糟的错误异常
+ 日志文件未分类或分类不够

## 日志原则

* 清晰、可读
* 不允许影响系统正常运行
* 不允许产生安全问题
* 不允许输出公司机密信息
* 可供开发人员定位问题的真正原因
* 可供监控系统自动监控与分析

## 日志规范

### 日志命名及目录规范

### 框架规范
+ 应用中不可直接依赖使用日志系统（Log4j、Logback）中的API，而应依赖使用日志框架（SLF4J、JCL--Jakarta Commons Logging）中的API

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
private static final Logger logger = LoggerFactory.getLogger(Xxx.class);
```

### 格式规范
+ 通用格式规范
	+ 使用占位符填充参数，禁止字符串拼接，新版本无需加`isXxxEnabled()`判断
	+ 生产环境绝对不允许直接使用`System.out`或`System.err`输出日志或使用`e.printStackTrace()`打印异常堆栈
+ 对格式无特定要求（普通日志）：
	- 参数和值之间用等号(=)或冒号(:)连接，不同参数间用竖线(|)或逗号(,)分隔
+ 对格式有特定要求（如接入xflush或tlog）：
	- 参数和值、参数和参数优先用竖线(|)分隔，其次可以考虑&、_、-、$
	- 空字段用空字符串("")补齐，不能直接跳过

```java
logger.debug("Entering xxx(param1={}, param2={})", param1, param2);
logger.error("xxx error", e);
```

### 日志级别规范

<http://stackoverflow.com/questions/2031163/when-to-use-the-different-log-levels>

+ FATAL
    - 系统遭到严重的系统级错误而挂了，则应该记入FATAL日志
    - 最严重的日志级别，所以一般不用，一般一个系统只有一条FATAL日志：系统因为严重错误而宕机
+ ERROR
    - 看到ERROR的日志需要马上进行处理，只允许用于打印异常日志
    - ERROR级别的日志出现代表系统已经产生错误以致用户已经在使用上出现问题了，但是系统没挂
    - 对于用户自己操作不当，如请求参数错误等等，不应该记为ERROR日志
+ WARN
    - 记录潜在的可能威胁系统的状态，如网络的波动
    - 这些状态还没导致出问题，但是如果不及时处理且程度再高一些就要出问题
    - 参数错误等推荐记录为WARN，便于用户投诉时追踪问题和后续提示、体验优化
+ INFO
    - 记录系统重要的业务处理已经结束，方便确认状态
        - 如系统初始化、XX接口成功执行
    - 记录显著改变应用状态的每一个action
    	- 如数据库更新、外部系统请求
    - 为了更快地定位WARN、ERROR和FATAL的问题
    - INFO日志的量不宜过多，一般不大于DEBUG或TRACE量的10%
+ DEBUG
    - 用于跟踪程序流向的诊断信息
    - 重要方法入口，业务流程前后及处理的结果等，推荐记录log，并使用debug级别
+ TRACE
    - TRACE比DEBUG更细

#### 思考

* 异常是否都用error？error和warn的区别？
* info和debug的使用场景？
* 动态调整日志级别

### 日志内容规范

* 日志必要参数：traceId、方法签名（、机器ip）
* 必要的入参：如aliId等定位问题的必要参数，对于消息来讲要打印出messageId
* 必要的出参：如errorMsg、errorCode
* 异常日志只在service层打印，中间层都throws，除非必要
* try...catch时一定要打印堆栈
* 不打印无效日志

## 最佳实践

* 是否要使用公共类？是否使用AOP来打日志？
*   各种日志框架的关系、技术选型
    *   是否需要考虑`slf4j2`? 优缺点对比

## 延伸

*   如何设计一个日志框架？
*   常见日志框架如`slf4j`的设计思路、相关设计模式分析
	* 与自己设计的进行对比
    * 其他类似的小而美的代码/框架，如junit的分析

## 参考资料

*   [Optimal Logging - by Anthony Vallone from Google](https://testing.googleblog.com/2013/06/optimal-logging.html)
*   [The Art of Logging](https://www.codeproject.com/Articles/42354/The-Art-of-Logging)
*   [Clean code, clean logs](http://www.nurkiewicz.com/2010/05/clean-code-clean-logs-use-appropriate.html)
*   [优秀日志实践准则](http://tech.lede.com/2017/06/30/rd/server/loggingHabit/)
*   [王健：最佳日志实践](http://blog.jobbole.com/56574/)
*   [Java日志终极指南](http://www.importnew.com/16331.html)
*   [Java 日志框架解析：设计模式、性能](http://www.jianshu.com/p/9916de96cb99)
*   [Java日志框架slf4j API介绍及异常接口实现分析](https://ketao1989.github.io/2014/05/02/Java-slf4j-Introduce/)
*   [深入源码之SLF4J](http://www.blogjava.net/DLevin/archive/2012/11/08/390991.html)
*   [日志那点事儿——slf4j源码剖析](http://www.cnblogs.com/xing901022/p/4149524.html)
*   [log4j 1.x 概述、意义和源码分析](http://blog.leanote.com/post/stackess/log4j-1.x-%E6%A6%82%E8%BF%B0%E3%80%81%E6%84%8F%E4%B9%89%E5%92%8C%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90)
*   [日志工具现状调研](http://tech.lede.com/2017/02/06/rd/server/log4jSearch/) 