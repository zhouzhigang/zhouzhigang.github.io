---
title: "代码提交规范"
linkTitle: "代码提交规范"
date: 2016-06-08
description: >
  代码提交问题、原则、规范及模板
---

## 常见问题

日常的提交中经常可以看到一些不规范的`commit message`, 看完后完全不知道做了些什么。

如下所示:

```
"update" - 没有说明更新什么内容, 为什么更新
"update api version" - 没有说明更新什么api, 更新到哪个版本
"fix bug" - 没有说明修复了什么bug, 为什么修复, 相关问题的链接
还有一种很常见的问题是大部分 message 都相同, 而实际更新的内容和 message 描述的内容并不相同
```

## Why

一个好的`commit message`可以带来很多好处的, 最主要是使代码更加容易理解(通过`git log`就可以知道代码做了些什么),
还有一个就是提高代码的可维护性(查看历史并不是为了挖坟, 很多时候一些决策过了一段时间后我们很有可能忘掉当初为什么变更的,
换做其他人维护那就更加难以理解为什么会有这样的代码, 如果message中有相应说明、项目或ticket链接可以大大提高代码的可维护性, 同时可以节约沟通成本)。

具体好处如下:

+ 易理解、易维护
  - 查看log就可知道变更思路
  - 方便Review
  - 便于后期维护(非常重要)
+ 便于查找和过滤
  - 方便根据关键字查找过滤
  - 方便 cherry-pick 或 rebase 等操作
+ 可以直接从commit生成Change log

## Commit 基本原则

1. 经常提交
2. 一个commit只做一件事
  - 可以是一个小功能，小改进或者一个bug fix
3. 保持commit可读性
  - 提交时必须带上有意义的描述
4. 不相干的代码修改不要放在同一个commit
5. 有错误的半成品程序不能commit
  - 可通过 amend 或 rebase 命令合并 commit

注：commit前建议先pull代码，避免merge造成时间线混乱

## Commit Message 规范

主要包含 Subject, Body, Footer 三个部分(用空行隔开), 其中Subject为必填(最低要求: 做到准确描述提交的内容)。

```
一句话概述commit主题(必须)

详细描述What和Why(可选)

不兼容或关闭issue等说明(可选)
```

当然第一行的概述也有些相对复杂的写法, 如[AngularJS Git Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)
中的`<type>(<scope>): <subject>`, 我们这里采取相对简单点的写法, 但是一定要把commit的内容描述清楚。

### Commit Message 模板(包括具体规范和提示)

如果怕忘记, 或不知道如何写, 可以设置一个简单的模板。

模板里的内容如下(文件名随意, 比如`.gitmessage`):

```
# Subject 一句话概述commit主题(必须)

# <Body> 详细描述What和Why(可选)

# <Footer>不兼容或关闭issue等说明(可选)


# 主题(Subject)是 commit 的简短描述，不超过50个字符
# * 用一句话说明本次所作的提交, 如果一句话说不清楚，那有可能这个提交得拆分成多次
# * 主要采用 Verb + Object + Adverb 的形式描述，常见动词及示例如下
# 1. Add: 添加代码和逻辑, 如 Add xxx field/method/class
# 2. Change: 代码更新，如 Change xxx to yyy with reason
# 3. Remove: 删除旧特性/功能，如 Remove xxx which was deprecated
# 4. Fix: 修复bug，如 Fix #123, fix xxx error
# 5. Update/Release: maven 版本变更， Update/Release xxx version to 1.0.0
# 6. Refactor: 代码重构, 如rename, move, extract, inline等
# 7. Polishing: 代码打磨(代码格式化，不涉及逻辑调整，使代码更清晰易读等无错修改)

# 正文(Body)详细描述本次 commit 做了什么、为什么这样做(不是怎么做的)
# * 每行不要超过70字符
# 1. 这个改动解决了什么问题？
# 2. 这个改动为什么是必要的？
# 3. 会影响到哪些其他的代码？

# 尾注(Footer) 用于关闭 Issue 或存在不兼容时添加相关说明等
# 1. REAKING CHANGE: 与上一个版本不兼容的相关描述、理由及迁移办法
# 2. close #issue: 关闭相关问题（附链接）
# 3. revert: 撤销以前的commit
```

注: #开头的内容为注释, 起提示作用, 不会被提交

然后将该文件设置为`commit.template`

```
git config --global commit.template ~/.gitmessage
```

## Reference
+ [Git 最佳实践：commit msg](http://blog.jobbole.com/109197/)
+ [Commit message 和 Change log 编写指南 - 阮一峰](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
+ [如何写一个好的Commit Message - 雷卷](http://www.atatech.org/articles/67029)
+ [配置Git的commit message模板 - 雪卒](http://www.atatech.org/articles/26380)
+ [Git Commit 信息规范 - 云翮](http://www.atatech.org/articles/61665)
+ [合并整理 Git commit](http://www.atatech.org/articles/21621)
+ [Git commit message 的寫法](https://github.com/oracle-design/guides/wiki/Git-commit-message-%E7%9A%84%E5%AF%AB%E6%B3%95) 