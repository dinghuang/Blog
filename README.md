# 如何在使用GitHub Page搭建HEXO博客

## 准备工作
1. 安装Git
2. 安装Node.js
3. 注册Github账号
4. 创建新的仓库 名称必须为 用户名.github.io
![此处输入图片的描述][1]
5. 添加本地电脑的SSH认证，如图所示
![此处输入图片的描述][2]
在本地命令工具中输入
``ssh-keygen -t rsa -C "Github的注册邮箱地址"``
一路回车之后会得到两个文件：id_rsa和id_rsa.pub，然后
用带格式的编辑器（比方说notepad++或者sublime）打开id_rsa.pub，复制里面的所有内容，然粘贴到KEY里面。

## 安装HEXO

1. 打开命令行工具
``npm install -g hexo-cli``
2. 安装 Hexo 完成后，分步执行（即输入代码之后敲回车）下列命令，Hexo 将会在指定文件夹中新建所需要的文件。
``hexo init <folder>``
在文件夹内执行
``npm install``
3. 成功后，博客项目成功初始化

## 配置NEXT主题

1. 安装NEXT主题
```mkdir themes/next
$ curl -s https://api.github.com/repos/iissnan/hexo-theme-next/releases/latest | grep tarball_url | cut -d '"' -f 4 | wget -i - -O- | tar -zx -C themes/next --strip-components=1```
修改站点配置文件``_config.yml``
```
title: 一只病猫
subtitle: 静坐常思己过，闲谈莫论人非
description: 学习、生活、闲谈、足球
author: 强壮的病猫
language: zh-Hans
timezone: Asia/Shanghai
theme: next
url: https://dinghuang.github.io/
deploy:
  type: git
  repo: https://github.com/dinghuang/dinghuang.github.io.git
  branch: master
#开启swiftype搜索，后面的id在swiftype官网申请，[具体操作][3]
swiftype_key: HcRPHRrBuwozvgUoLNyX
#要放到仓库的静态资源都放在source文件夹中，.md会被转换成HTML，所以这里要忽略
skip_render: README.md
```
2. 配置NEXT主题
设置主题配置文件``./themes/next/_config.yml``，可以参考本项目的配置。
NEXT强大之处在于继承了很多第三方服务插件，不过类似评论搜索功能的插件被墙了，外网是可以用的。[具体参考][4]
3. 设置标签分类
参考[链接][5]，在文章开头，引入：
```
---
title: 设计模式
date: 2018-07-18 09:43:00
tags:
    - JAVA
categories: JAVA
---
```
更多配置请参考官方文档

## 提交
输入命令
``npm install hexo-deployer-git --save``
创建文章
``hexo new "你想要的文章标题填在这个双引号里"``
文章会生成在
``./source/_posts/设计模式.md``
进行修改后
``hexo clean ; hexo genarate``
然后输入
`` hexo deploy``
此时文章已经成功部署。

  [1]: https://note.youdao.com/yws/public/resource/cf7c41010a7c5769daf36ed28209eda3/xmlnote/A621EF7ED1CA4E3F9249E04FC033831B/3282
  [2]: https://note.youdao.com/yws/public/resource/cf7c41010a7c5769daf36ed28209eda3/xmlnote/2EF0BE5C89D649E49278592AE806FD65/3285
  [3]: https://theme-next.iissnan.com/third-party-services.html#search-system
  [4]: https://theme-next.iissnan.com/third-party-services.html
  [5]: https://github.com/iissnan/hexo-theme-next