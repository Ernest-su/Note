---
title: GitHub上star超过2k的安卓MVP架构指南
date: 2017-03-28 19:06:55
categories:
    - 架构
tags:
    - 架构
    - MVP
    - Android
    - 安卓
---
> 近日在搜MVP模式作为安卓项目架构时候，发现GitHub上一篇介绍MVP架构的文章。一看star已经超过2K了。通读全文发现作者用实际项目将MVP架构要怎么分职责讲得十分通俗易懂。为了方便大家查阅，把原文翻译了一下。作者还有一篇文章讲安卓项目和代码风格指南的。欢迎查看[相关翻译](http://www.suqishuo.cn/android-project-and-code-guidelines/)。

# 架构指南
我们的安卓APP架构基于 [MVP](https://en.wikipedia.org/wiki/Model–view–presenter) (Model View Presenter) 模式。
* __View (UI 层)__:这是 Activities, Fragments 和其他标准安卓控件所在层。它的职责是呈现从Presenter收到的数据给用户。它也处理用户的交互和输入（点击监听等）以及在需要时触发Presenter中对应的动作。

* __Presenter__: Presenter 订阅由 `DataManager` 提供的 RxJava Observables。他们负责处理订阅的生命周期，分析/修改`DataManager`返回的数据，调用View中的合适方法以便展示数据。

* __Model (Data 层)__: 它们的职责是取回，保存，缓存，以及通知数据。它可以与本地数据库以及其他的数据仓库通信，使用restful API或者其他第三方SDK等方式。它分为两部分：一组帮助工具类以及一个  `DataManager` 。工具类的数量因项目而异，每一个工具类有特定的功能，比如与API通信或在 `SharedPreferences` 中保存数据.  `DataManager`  把不同工具类的输出使用rx操作符进行组合变换，所以它可以： 1) 提供有意义的数据给 Presenter，2) 把经常一起出现的动作组合在一起。这一层也包含实际的实体类，定义数据结构是怎样的。

![](/images/mvp_architecture_diagram.png)

从右到左看示意图:

* __Helpers (Model)__: 一组类的集合，每个类有特定的职责，他们的功能可以从与API或数据库通信，到实现一些特定的业务逻辑等。每个项目会有不同的工具类，但最常见的几个是：
	- __DatabaseHelper__: 它处理插入，更新，以及获取来自本地SQLite数据库的数据。它的方法返回 Rx Observables ，发射(译者注：Rx术语)简单java对象 (models)。
	- __PreferencesHelper__: 它保存以及获取来自 `SharedPreferences` 的数据，他可以返回 Observables 或者直接返回简单java对象。
	- __Retrofit services__ : [Retrofit](http://square.github.io/retrofit) 接口 使用Restful API 通信，每个不同的API拥有自己的 Retrofit 服务. 它们返回 Rx Observables.

* __Data Manager (Model)__: 这是MVP架构的关键部分。它持有每一个工具类的引用，使用这些工具类满足来自Presenter的请求。 它的方法广泛的使用 Rx 操作符来组合，转换或过滤来自工具类的输出，以便生成Presenter想要的输出。它返回发射数据模型（data models）的 observables。

* __Presenters__: 订阅由  `DataManager`  提供的observables，处理这些数据以便调用View中合适的方法。

* __Activities, Fragments, ViewGroups (View)__: 实现了一组Presenter可以调用的方法的标准安卓组件。它们也处理用户的交互如点击等，然后调用Presenter中合适的方法来采取相应的行动。这些控件也实现框架相关的任务，如管理安卓生命周期，渲染视图（Views）等。

* __Event Bus__: 它使得View控件得到发生在Model的特定事件通知。通常  `DataManager` 发出事件，然后这些事件可以被Activities 和 Fragments 订阅。  event bus  **仅仅** 用于非常特别的动作——这些事件不是仅与一个界面相关的，要通知多方，如：用户已经退出登录。
[原文链接](https://github.com/ribot/android-guidelines/blob/master/architecture_guidelines/android_architecture.md)
[译文链接](http://www.suqishuo.cn/android-architecture-guideline-for-mvp/)