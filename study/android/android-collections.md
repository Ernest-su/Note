---
title: 安卓开发相关框架文章收集（长期更新）
date: 2017-03-34 17:30:55
categories:
    - 开发效率
tags:
    - Android
---
> 在做项目初期，如果知道有哪些框架/工具可用，对项目的开发会有很大的帮助。这篇帖子收集项目开发用过的，或者有类似需求不知道有对应工具，后来发现这些工具有相见恨晚感觉的。作为一个类似于备忘录的帖子，根据个人判断在需要的时候进行增删改查。介绍时候尽量用一两个关键词。收集的文章可能会因各种原因失效，请以标题自行在搜索引擎搜索。帖子目的不是做到大而全且更新频繁。如果读者有这样的需求，良心推荐[干货集中营](http://gank.io),每日分享 **妹子图** 和 ** 技术干货**，还有供大家中午休息的休闲视频

最近更新于 2017-03-24 [原文才持续更新哦](http://www.suqishuo.cn/android-collections/)

# 工具类
## DiffUtil
> DiffUtil是一个查找集合变化的工具类，是搭配RecyclerView一起使用的。DiffUtil的作用，就是找出集合中每一个Item发生的变化，然后对每个变化给予对应的刷新。
[Android开发学习之路-DiffUtil使用教程](http://www.cnblogs.com/Fndroid/p/5789470.html)

## logger
> 简单，美观，功能强大的安卓打印工具。
[GitHub](https://github.com/orhanobut/logger)

## moco模拟服务器返回数据
>灵活配置参数，请求方法，URL，支持挂载文件，自带独立微型服务器，可以用json，java等语言配置。
[介绍文章](http://www.suqishuo.cn/how-to-build-api-mock-service/)
[官网](https://github.com/dreamhead/moco/)

# 快速开发
## AS插件
> ButterKnife Zelezny：ButterKnife 生成器
SelectorChapek：selector 自动生成XML
GsonFormat：json字符串生成相应的实体类
 [介绍文章](http://stormzhang.com/android/2015/05/26/android-tools/)

## bufferKnife
> JakeWharton大神出品。View,资源，事件注解方式初始化。
[官网](http://jakewharton.github.io/butterknife/)

# 网络框架

## Volley
> 2013年Google I/O大会上推出的一个新的网络通信框架，简化Http通信，提供异步回调。
个人更推荐使用retrofit`

## Retrofit
> Retrofit与okhttp共同出自于Square公司,retrofit就是对okhttp做了一层封装。把网络请求都交给给了Okhttp.配置简单灵活，接口返回对象，支持rxJava无缝对接
[GitHub](https://github.com/square/retrofit)

## Glide 图片加载
> 这个库被广泛的运用在google的开源项目中，包括2014年google I/O大会上发布的官方app.Glide和Picasso有90%的相似度

## ~~Picasso图片加载~~ 
> 推荐用Glide代替。查看对比文章 [Picasso VS Glide](http://www.cnblogs.com/sihaixuan/p/4853233.html)

# 数据库优化

## GreenDAO
> greenDao是一个将对象映射到SQLite数据库中的轻量且快速的ORM解决方案，跟EvenBus师出同门。
[介绍文章1](http://www.suqishuo.cn/greendao-project-practice.md)
> [介绍文章2](http://www.suqishuo.cn/greendao-project-practice-ToOne-ToMany.md)
[官网](http://greenrobot.org/greendao/)

## sqlbrite
> square出品。一个轻量级SQLiteOpenHelper包装类，引入reactive 流式语法,完美解决数据库和UI的同步更新！
 [GitHub](https://github.com/square/sqlbrite)

# 性能优化
## Hugo 打印方法参数执行时间
> JakeWharton大神出品。只要一行`@DebugLog`注解打印方法的传参和返回值，甚至方法的执行时间,只在debug版本打印，对release版本无任何影响。
[GitHub](https://github.com/JakeWharton/hugo.git)

## Scalpel 
> JakeWharton大神出品。查看界面层级，3D的效果。方便优化界面布局。
[GitHub](https://github.com/JakeWharton/scalpel.git)

## leakcanary
> square出品。内存泄漏检测工具
 [GitHub](https://github.com/square/leakcanary)

#  rxJava/rxAndroid 
##  rxJava/rxAndroid 精品文章
 [RxJava Essentials 中文翻译版](http://rxjava.yuxingxin.com/)
 [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
 [深入浅出RxJava（一：基础篇）](http://blog.csdn.net/lzyzsd/article/details/41833541)
 [RxJava 和 RxAndroid 五（线程调度]( http://www.cnblogs.com/zhaoyanjun/p/5624395.html)
## rxJava 示例代码
 [rxJava 示例代码集](https://github.com/rengwuxian/RxJavaSamples)

## rxJava框架 rxBinding
# 架构
## 精品文章
[Android架构探索](https://theseyears.gitbooks.io/android-architecture-journey/content/rxbus.html)
