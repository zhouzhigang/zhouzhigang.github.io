---
title: "自动化周刊设想及实现"
linkTitle: "自动化周刊设想及实现"
date: 2017-05-23
description: >
  自动化周刊设想及核心实现
---

## 需求
1. 自动化收集整理一些初始的文章
2. 众人投稿的方式，并自动收集稿件
3. 生成markdown、发邮件、钉钉智能机器人

### 目的
1. 初步过滤、避免人工收集
2. 防止一个人收集整理太辛苦
3. 自动整理成果并发送，节约时间

---

## 需求分析

1. 从相关数据源抓取信息
  - 哪些数据源？
  - 如何抓取？
      - 简单为美；无需考虑分布式、反爬等
2. 简单的整理
  - 过滤的策略？
3. 存储/发送邮件
  - 可否不占用独立的服务器，如与gitlab集成

---

## 文章来源

+ 国内
  - [码农周刊](http://weekly.manong.io/issues/)
  - [掘金](https://gold.xitu.io/welcome/backend)
  - [推酷 - 编程狂人](http://www.tuicool.com/mags/)
  - [推酷 - Guru Weekly](http://www.tuicool.com/mags/guru)
  - [PM周刊](http://pmweekly.com/issues/)
+ 国外
  - [Dzone](https://dzone.com/), [Hack News](https://news.ycombinator.com/), [reddit](https://www.reddit.com/)

---

## 技术选型

+ Java
  - [httpclient](http://hc.apache.org/httpcomponents-client-ga/)/[HtmlParser](http://htmlparser.sourceforge.net/)[jsoup](https://jsoup.org/)
  - [Crawler4j](https://hao.jobbole.com/crawler4j/)/[WebMagic](https://github.com/code4craft/webmagic)
  - [Nutch](https://github.com/apache/nutch)
+ Python
  - Requests/urllib/urllib2/httplib2
  - [pyquery](https://pythonhosted.org/pyquery/); [Scrapy](https://scrapy.org/)
+ Ruby
  - [httprb](https://github.com/httprb/http),[faraday](https://github.com/lostisland/faraday)
  - [nokogiri](http://www.nokogiri.org/)
+ Node.js
  - request, [cheerio](https://github.com/cheeriojs/cheerio), [nodemailer](https://github.com/nodemailer/nodemailer), [showdown](https://github.com/showdownjs/showdown)

---

## 内容解析与过滤

+ 内容解析
  + xpath
  + 正则
  + jquery：容错处理

+ 内容过滤
  + 标题关键字

---

## 邮件发送/文件生成

+ 分组/去重
+ 邮件发送
+ markdown与html转换

---

## 钉钉机器人

+ [创建钉钉机器人教程](http://one-voice-platform.alibaba-inc.com/ai/help/Tutorials/T1_Dingding.html)
+ [创建意图API](http://one-voice-platform.alibaba-inc.com/ai/help/Port/Api.html)
  - 抓取到相关数据后，可直接创建相关意图，这样机器人就可实现关键字返回具体内容
  - 避免使用到数据库及回调查询
+ 问题: 钉钉`markdown`的渲染有问题

---

## CI

### 基本步骤

- 设置地址在 project -> settings -> runners ,左侧是自定义runner
- 注册special runner 的方法 参考上面的设置页面，的左侧说明
- runner的具体行为 对应项目中的 .gitlab-ci.yml
- 编写规则：1）before_script :在后续的job前执行，保证job依赖的插件已安装
- 编写规则：2）jobxx : 相互独立的任务，script是具体执行的构建脚本，only是制定分支

---

### .gitlab-ci.yml

+ [官方文档](https://docs.gitlab.com/ce/ci/quick_start/README.html)
+ [定时触发](http://cloudlady911.com/index.php/2016/11/02/how-to-schedule-a-job-in-gitlab-8-13/)

```
before_script:
  - echo $PATH
  - time enclose install tnpm
  - tnpm -v
node-6:
  image: reg.docker.alibaba-inc.com/dockerlab/node-ci:1.0.0
  script:
    - time tnpm i --install-node=6 --no-cache
    - time tnpm run ci
  tags:
    - swarm
```

---

## 附: 具体技术选型

---

### Java

+ [Nutch](https://github.com/apache/nutch) 分布式、高度可扩展、可伸缩的网络爬虫
+ [Heritrix](https://github.com/internetarchive/heritrix3)
+ [Crawler4j](https://hao.jobbole.com/crawler4j/)简单的轻量级网络爬虫
+ [WebMagic](https://github.com/code4craft/webmagic)
+ HttpURLConnection/[httpclient](http://hc.apache.org/httpcomponents-client-ga/)/[HtmlParser](http://htmlparser.sourceforge.net/)[jsoup](https://jsoup.org/)

---

### Python

+ Requests
+ pyQuery
+ [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/)
+ selenium
+ Scrapy
+ [Python入门网络爬虫之精华版](https://github.com/lining0806/PythonSpiderNotes)

---

### Ruby

+ [httprb](https://github.com/httprb/http)
+ [faraday](https://github.com/lostisland/faraday)
+ [nokogiri](http://www.nokogiri.org/)
+ [redcarpet](https://github.com/vmg/redcarpet)
+ [kramdown](https://github.com/gettalong/kramdown)
+[Markdown Processing in Ruby](https://www.sitepoint.com/markdown-processing-ruby/)

---

### Node.js

+ [superagent](http://visionmedia.github.io/superagent/) http库
+ [cheerio](https://github.com/cheeriojs/cheerio)  Node.js 版的 jquery
+ [showdown](https://github.com/showdownjs/showdown) Markdown 与 HTML 转换
+ [nodemailer](https://github.com/nodemailer/nodemailer) 发送邮件
