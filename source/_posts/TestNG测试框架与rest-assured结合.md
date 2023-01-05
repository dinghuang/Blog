---
title: TestNG测试框架与rest-assured结合
date: 2019-01-24 09:47:00
tags:
    - JAVA
categories: JAVA
---
      
[TestNG官网](https://testng.org/doc/index.html)

# TestNG简介

![image](http://www.strongsickcat.com:7014/file/dinghuang-blog-picture/9bc4cb9fgy1g0apgqmzy2j206o06odhi.jpg)

TestNG是一个受JUnit和NUnit启发的测试框架，但引入了一些新功能，使其功能更强大，相对于JUnit来说，xml的配置使的testNG对于不同测试之间的依赖程度有更好的把控性。

# rest-assured简介
在Java中测试和验证REST服务比在Ruby和Groovy等动态语言中更难。REST [Assured](http://rest-assured.io/)将使用这些语言的简单性带入了Java域。

# TestNG测试框架与rest-assured结合

项目地址：
https://github.com/dinghuang/testNGExample

上面实现了模拟用户登录以及rest-assured的高级用法，同时可以通过命令行直接生成报告，报告中对http请求增加了过滤，会在报告中展示请求信息，可以通过xml解析直接获取，还实现了多个suit通过mvn命令直接运行。目前在Choerodon中已经增加了TestNG的支持，用户可以直接推到gitlab，gitlab中的runner会在ci中打包并运行测试jar包，把报告解析并提取请求信息生成测试用例。

项目的关键代码就不一一说了，项目中有注释，不懂的+我VX:742041978


