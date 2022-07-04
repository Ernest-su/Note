---
title: Linux 命令手册安卓版
date: 2017-03-22 19:30:55
categories:
    - 工具
tags:
    - 工具
    - Linux
    - Android
    - 安卓
---

> 前两天逛GitHub时候发现了一个命令行手册项目[jaywcjlove/linux-command](https://github.com/jaywcjlove/linux-command.git) [手册网站](https://jaywcjlove.github.io/linux-command/)，觉得非常实用。GitHub的Pages很方便，但是国内访问有时候比较慢。虽然网上类似的在线手册很多了，但是我觉得一个手册应该是随手可用，而且即使在没有网时候，也可随时翻阅的。于是打算用一天半天把手册做成个离线版简易手册APP。

# 准备工作
翻了一下GitHub上项目的README，发现作者很贴心的把爬虫抓到的数据保存成MarkDown格式了。不得不说一下，MarkDown是程序员最喜欢的文档格式之一。把项目clone到本地，手册的数据来源就准备好了。再次 感谢作者的辛勤付出。
## MarkDown解析
安卓没有原生支持MarkDown的控件，在GitHub上用markdown为关键字搜了一下，[pegdown](https://github.com/sirthias/pegdown) 这个MarkDownn解析库是最多star的。遗憾的是，尽管目前仍然是最流行的MarkDown解析库之一，作者在三个多月前声明这个项目停止维护了。不过作者推荐了[@vsch](https://github.com/vsch)的[flex-mark-java](https://github.com/vsch/flexmark-java),另一个性能优越的markdown解析库。

`flexmark-java`解析架构基于[commonmark-java](https://github.com/atlassian/commonmark-java),另一款解析库(开源的世界就是这么任性啊),想到罗永浩的那句生命不息，折腾不止，决定先用`commonmark-java`解析，后面再切换到`flexmark-java`。尽管是最简单的引用，并没有怎么折腾。
## 搜索功能
手册的数据，解析都准备好了，还有一个手册必不可少的功能——搜索。以前做项目的时候都是用EditText+Button来实现搜索功能的。虽然一直知道安卓有个原生的searchView，但是项目做视觉设计的小伙伴设计出来的视觉不给searchView用武之地。自从进TV应用开发的坑后，也好久没有开发手机APP了。基于这两个原因，决定在这个小应用中学习一下searchView的使用。
计划实现的是带搜索建议的searchView,这样可以尽量的简化在手机上的输入。searchView的配置很简单，但是搜索建议需要有一定的学习。
安卓现在推荐的是ActionBar中搜索，除了要增加一点actionView的配置，其他的跟searchView大同小异。正所谓磨刀不误砍柴工，何况之前没有系统的学习过searchView，于是开始找百度，找博客补习功课。
以searchView搜索看了几篇配置博文后，发现一个博客把谷歌的searchView介绍做了一个专题的介绍，于是把这几篇文章细读了一遍。在读这几篇译文的时候想起很多大牛的教导，要读就读一手的文档，最好是出自代码作者之手的，再次一点就是读一手文档的翻译。除了这两种文章干货较多，其他类型的文章往往介绍得不够全面。
[英文原文（需要翻墙）](http://developer.android.com/guide/topics/search/search-dialog.html)
[译文第一篇：[Searchable(一)]](http://blog.csdn.net/hudashi/article/details/7052815)(剩下的几篇链接就不贴了，在文末有上下篇的链接跳转)

依照译文的介绍，我对搜索功能有了大概的方向。主要做了以下几点：
- 把前面获取到的MD文件放到assets目录下，用`AssetsManager`进行管理，这样就可以不用数据库了。展示命令详情的时候，也可以直接读取到命令对应文件流，再方便不过了。
- `searchView`的搜索建议需要有一个`ContentProvider`来提供搜索建议的数据源，没有数据库怎么用ContentProvider? 答案是`MatrixCursor`。`MatrixCursor`可以做到模拟数据库的作用，也提供方法方便插入数据等。
- 有了实现`ContentProvider`的思路,只需把`AssetsManager`获取到的命令文件名作为数据，提供给`searchView`来查询就可以了。

# 总结
手册的实现并没有什么难度，在补充了相关的知识后，只需要不到半天的时间就可以搞定了。项目虽小，但是还是给我不少收获的。之前看到有一篇文章，名字叫《提问的智慧》。搜索其实就是向机器提问的一种方式，科学的搜索关键词，合适的网站搜索事半功倍。搜索还是首选 [Google.com](http://www.google.com) 和 [stackoverflow.com](http://stackoverflow.com)

项目地址：
[GitHub](https://github.com/Ernest-su/LinuxCmd) 
[coding.net](https://git.coding.net/ernest/LinuxCmd.git)





