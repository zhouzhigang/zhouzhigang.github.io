---
title: "深入Kotlin协程"
linkTitle: "深入Kotlin协程"
date: 2018-06-21
description: >
  协程的起源、实现原理、不同语言实现的差异
---

## Why

尽管有不少关于Kotlin协程的文章，但是大家似乎对这个还是存在不少疑惑，不少东西好像也没讲得太透。

团队不少同学认为协程并不是所谓的"免费"的异步，主要原因是需要切换到Koltin。其实切换的成本并不高，Koltin和Java本身就是可以互操作（不过协程部分在Java中调用Kotlin会复杂些），真正的成本可能在于后续的维护以及出问题时的排查，因此需要对背后的原理有一定的了解。

## Objective

* 为什么使用协程？什么场景下适用？
* Kotlin协程实现原理
* Kotlin Coroutine 与 GoRoutine、Java11协程的比较
* 几种异步方式的比较：Futures、C o rou tinReactive的比较
