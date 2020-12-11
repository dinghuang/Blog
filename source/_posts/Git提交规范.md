---
title: Git提交规范
date: 2020-11-08 15:47:00
tags:
    - GIT
categories: GIT
---

# 设置用户
首次拉代码后，在目录下用命令行设置用户信息

```
git config core.ignorecase false
git config --global user.name "张三"
git config --global user.email san.zhang@gmail.com
```


# Git Commit规范
规定格式如下：
```
<type>(<scope>): <subject> #issue_number

<description>
```
其中，type、scope、subject是必需的，description 可以省略。不管是哪一个部分，任何一行都不得超过72个字符（或100个字符）。这是为了避免自动换行影响美观。

## type(必须)取值说明
type用于说明 commit 的类别:

- feat: (feature)增加新功能
- fix: 修补bug
- docs: 文档（documentation）， 只改动了文档相关的内容
- style: 不影响代码含义的改动，例如去掉空格、改变缩进、增删分号
- build: 构造工具的或者外部依赖的改动，例如webpack，npm
- refactor: 代码重构时使用
- revert：回滚到上一个版本,执行git revert打印的message
- test:  添加测试或者修改现有测试
- pref: 提高性能的改动
- chore: 不修改src或者test的其余修改，例如构建过程或辅助工具的变动
- merge：代码合并
- sync：同步主线或分支的Bug
- ci: 与CI（持续集成服务）有关的改动

## scope(可选)取值说明
scope用于说明 commit影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。例如：
node-pc/common rrd-h5/activity，而we-sdk不需指定模块名。如果一次commit修改多个模块，建议拆分成多次commit，以便更好追踪和维护。后加入项目的新成员应遵循已有的 scope 约定（通过 git log可以查看某个文件的提交历史），不要自己编造。使用首字母小写的驼峰命名。除具体的模块、组件名之外，可以使用 base 表示基础结构、框架相关的改动，用 misc 表示杂项改动，用 all 表示大范围重构。


例如在Angular，可以是location，browser，compile，compile，rootScope， ngHref，ngClick，ngView等。如果你的修改影响了不止一个scope，你可以使用*代替。

## subject(必须)
subject是 commit 目的的简短描述，50 个字符左右的简要说明。

## 中文
- 结尾不加句号或其他标点符号。
- 根据以上规范git commit message将是如下的格式：
```
fix(DAO):用户查询缺少username属性 
feat(Controller):用户查询接口开发
```
以上就是我们梳理的git commit规范，那么我们这样规范git commit到底有哪些好处呢？

- 便于程序员对提交历史进行追溯，了解发生了什么情况。
- 一旦约束了commit message，意味着我们将慎重的进行每一次提交，不能再一股脑的把各种各样的改动都放在一个git commit里面，这样一来整个代码改动的历史也将更加清晰。
- 格式化的commit message才可以用于自动化输出Change log。

## 英文
首字母小写，通常是动宾结构，描述做了什么事情，动词用一般现在时，禁止出现 update code ， fix bug 等无实际意义的描述，好的例子： select connector by sorting free memory （不需要形如 update about how to select connector … 的啰嗦写法）, fix success tip can not show on IE8 （不需要形如 fix bug of … 的啰嗦写法）

- 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
- 尽量使用简单句保证简洁性
- 第一个字母小写
- 结尾不加句号（.）
- 通过翻译检测工具确认英文的正确性和可读性

## body
body填写详细描述，主要描述改动之前的情况及修改动机，对于小的修改不作要求，但是重大需求、更新等必须添加body来作说明。

## break changes
break changes指明是否产生了破坏性修改，涉及break changes的改动必须指明该项，类似版本升级、接口参数减少、接口删除、迁移等。

## affect issues
affect issues指明是否影响了某个问题。例如我们使用jira时，我们在commit message中可以填写其影响的JIRA_ID，若要开启该功能需要先打通jira与gitlab。

## 使用Commitizen工具
安装commitizen
```
npm install -g commitizen 
```
安装完成后，使用
```
(base) [@dinghuangMacPro:test (master)]$git add .
(base) [@dinghuangMacPro:test (master)]$git cz
#会出现下面的，按照提示写入
(base) [@dinghuangMacPro:test (master)]$ git cz
cz-cli@4.2.2, cz-conventional-changelog@3.3.0

? Select the type of change that you're committing: (Use arrow keys)
❯ feat:     A new feature
  fix:      A bug fix
  docs:     Documentation only changes
  style:    Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
  refactor: A code change that neither fixes a bug nor adds a feature
  perf:     A code change that improves performance
  test:     Adding missing tests or correcting existing tests
(Move up and down to reveal more choices)

#选择类型后
? Select the type of change that you're committing: feat:     A new feature
? What is the scope of this change (e.g. component or file name): (press enter to skip) add aa.txt
? Write a short, imperative tense description of the change (max 82 chars):
 (4) 新增文件
? Provide a longer description of the change: (press enter to skip)
 长描述：这是我第一次的提交
? Are there any breaking changes? No
? Does this change affect any open issues? Yes
? Add issue references (e.g. "fix #123", "re #123".):
 feat #123

#查看生成的git commit信息
(base) [@dinghuangMacPro:test (master)]$ git log
commit 00d63b2c54c502bb75725e5a461d6ce9499910bc (HEAD -> master)
Author: 丁煌 <dinghuang123@gmail.com>
Date:   Wed Nov 25 13:39:13 2020 +0800

    feat(add aa.txt): 新增文件

    长描述：这是我第一次的提交

    feat #123
```
自动生成Change log
```
npm install -g conventional-changelog-cli
conventional-changelog -p angular -i CHANGELOG.md -s
```
这个插件是根据当前分支提交的所有commit信息来生成对应的changelog，一般是在开发完成后，封代码后去进行生成。如果自动生成的有问题，可以根据自己的需求进行修改。

# 分支规范
## 分支基本原则
基本原则：master为保护分支，不直接在master上进行代码修改和提交。


开发日常需求或者项目时，从master分支上checkout一个feature分支进行开发或者bugfix分支进行bug修复，功能测试完毕并且项目发布上线后，将feature分支合并到主干master，并且打Tag发布，最后删除开发分支。


## 分支命名规范
例如：
- 分支版本命名规则：分支类型 分支发布时间 分支功能。比如：feature_20200401_#issueNum
    - 分支类型包括：feature、bugfix、refactor三种类型，即新功能开发、bug修复和代码重构
    - 时间使用年月日进行命名，不足2位补0
    - 分支功能命名使用snake case命名法，即下划线命名。
- Tag包括3位版本，前缀使用v。比如v1.2.31。Tag命名规范：
    - 新功能开发使用第2位版本号，bug修复使用第3位版本号
    - 核心基础库或者Node中间价可以在大版本发布请使用灰度版本号，在版本后面加上后缀，用中划线分隔。alpha或者belta后面加上次数，即第几次alpha(具体可以查看语义化版本)：
        - v2.0.0-alpha-1
        - v2.0.0-belta-1
