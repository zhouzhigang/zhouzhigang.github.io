---
title: "异常处理规范"
linkTitle: "异常处理规范"
date: 2016-06-07
description: >
  异常设计及处理原则、规范
---

## Why

## Java异常体系

- 为什么需要异常？为什么要区分checked, unchecked?
- Java中异常设计分析，背后设计的一些思考
  - 如IOException的异常层次体系？
  - SQLException中的getErrorCode()？

## 异常设计原则

+ 接口中是否要抛异常？还是使用错误码？
	- 接口的分类？
	- 如果抛异常的话，使用哪种异常？
    - 新增异常的话是否会破坏接口的兼容性？

## 异常处理原则

+ 发生异常时如何处理？
	- 查不到数据时返回null还是异常？
	- 怎样判断某个类的能否处理某个异常？
    - 如何正确重试？
+ 异常发生时是否都要记录日志？对应的日志级别？
	- 比如NullException异常如何处理？IllegalArgumentException如何处理？

## 常见异常坏味道及重构

+ 有哪些常见的异常坏味道？如何重构？
	- 如忽略异常、异常中处理分支逻辑、忘记释放资源、整块try-catch等...

## 延伸

+ Code Complete, Clean Code, Refactoring等书中关于异常设计有哪些观点？
+ 从语言设计角度分析Java的异常设计？和其他语言的比较（如C++, C#, Kotlin, Ruby等）
	- 设计一门语言的异常时，需要考虑哪些问题？各种语言设计异常时的权衡？
    - 比如checked, unchecked的利弊？遇到异常时默认终止、恢复还是重试？...

## 参考资料
* [什么情况下使用异常处理？](https://www.zhihu.com/question/27122172)
* [Advantages of Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/advantages.html)
* [如何优雅的设计java异常](https://lrwinx.github.io/2016/04/28/%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E8%AE%BE%E8%AE%A1java%E5%BC%82%E5%B8%B8/)
* [如何优雅的处理异常（java）？](https://www.zhihu.com/question/28254987)
* [Java中的Checked Exception——美丽世界中潜藏的恶魔？](http://www.importnew.com/21117.html)
* [Java 中的异常和处理详解](http://www.importnew.com/26613.html)
* [有效处理Java异常三原则](http://www.importnew.com/1701.html) 
* [Java异常处理的10个最佳实践](http://www.importnew.com/20139.html)
* [Kotlin 和 Checked Exception](http://www.yinwang.org/blog-cn/2017/05/23/kotlin)
* [如何评价王垠的《Kotlin和Checked Exception》？](https://www.zhihu.com/question/60240474)
* [Java 异常处理的最佳实践Best Practices for Exception Handling](http://www.jcodecraeer.com/a/chengxusheji/java/2013/0724/1485.html)
* [Java 异常处理的误区和经验总结](https://www.ibm.com/developerworks/cn/java/j-lo-exception-misdirection/) 