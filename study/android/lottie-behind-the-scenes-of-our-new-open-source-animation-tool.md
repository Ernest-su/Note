---
title: Lottie,我们幕后新的开源动效工具
date: 2017-03-20 12:30:55
categories:
    - 开发效率
tags:
    - Lottie
    - 动效
    - AE
    - airbnb
    - 设计
    - 开源
---
> airbnb开源了一套适用于Android和iOS动效库。虽然尚在快速迭代中，但是已经忍不住在项目中小范围试用。无奈新开源的项目文档比较少。在airbnb设计部门的网站上看到一篇介绍Lottie的文章，试着翻一下。本文是第一次翻译英文文档，翻译不当或不专业的地方欢迎指教。

在过去，为Android，IOS,React Native应用创建复杂的动效，是一个困难又费时的过程。您要么为每种屏幕分辨率添加笨重的图像文件，要么写上千行难以扩展维护的代码。正因为如此，尽管动效是一个沟通的想法和创造引人注目的用户体验的强大工具，大多数应用程序并不使用动效。一年前，我们开始改变这种情况。

今天，我们很高兴介绍我们的解决方案。Lottie是一个可用于iOS，Android和React Native 的库，实时渲染 After Effects动效，并允许开发者在原生应用程序中使用动效像使用图片等静态资源一样简单。Lottie使用一个开源的After Effects插件(Bodymovin)导出的JSON文件形式的动效数据，这个插件捆绑了一个JavaScript播放器，可以在网络上渲染动画。从2015二月开始，Bodymovin的作者Hernan Torrisi，每月通过给插件增加新功能，改进现有功能，为我们的Lottie项目建立了一个坚实的基础。我们的团队（Brandon Withrow负责iOS，Gabriel Peal负责Android，Leland Richardson负责React Native，我负责体验设计）在Torrisi的杰出工作基础上开始Lottie开发之旅。

![](http://airbnb.design/wp-content/uploads/2017/02/icons.gif)

Lottie使得程序猿无痛实现丰富的动效。如上图所示的Nick Butcher跳跃动效，Bartek Lipinski的汉堡式菜单，Miroslaw Stanek的推特心等等，如果用代码从头开始实现四多么困难和费时。有了Lottie，诸如深入了解框架，预估动画时间，手动创建贝赛尔曲线，重新制作GIF参考动效将成为过去式。现在程序猿可以准确地使用设计师设计的动效，确切地说动效是如何制造的，我们实现的就是怎样。为了展示这一点，我们已经重新创建上面的动画，并在我们的示例应用程序提供After Effects和JSON文件。

我们的目标是尽我们所能支持尽可能多的After Effects 的功能，而不仅仅是简单的图标动画。我们还创建了一些其他示例来显示Lottie库的灵活性、丰富性和进阶功能集。在示例应用程序中，还有各种不同类型动画的源文件，包括基本线条艺术、基于字符的动效以及多角度的logo动效。

![](http://airbnb.design/wp-content/uploads/2017/01/screens_1.gif)

我们已经在屏幕的各个角落开始使用我们设计的Lottie动效，包括下图所示的应用程序的通知，全帧动画插图，回访评价页面等。我们计划通过有趣且有用的方式大幅度扩大动画使用。

![](http://airbnb.design/wp-content/uploads/2017/01/screens_2.gif)

## 灵活高效的解决方案

Airbnb是支持数以百万计的房东和房客的全球化公司，所以有一个灵活的动画格式，且可以在多种平台上使用，对我们而言非常重要。现在已经有与Lottie相似的动效库，如Marcus Eckert的Squall和Facebook的Keyframes。但我们的目标稍有不同。Facebook挑选了一小部分After Effects的功能提供支持，因为他们主要集中在交互，但我们想支持尽可能多。至于Squall，Airbnb的设计师用它和Lottie组合使用，因为它有一个神奇的After Effects预览APP，所以成为我们工作中必不可少的一部分。然而，它只支持iOS，但是我们的工程团队需要一个跨平台的解决方案。

Lottie有几个功能内置了它的API，使其更加灵活和高效。它支持从网络上加载JSON文件，这对A / B测试非常有用。它也有一个可选的缓存机制，所以经常使用的动画，如愿望列表的心型图标可以每次加载缓存的副本。Lottie也可以使用动效的进度功能实现用手势调整动画的进度，以及通过改变一个简单的值从而改变动画的速度。iOS版本甚至支持在运行时向动画添加额外的本地UI，可用于复杂的动画转换。

我们工作到目前为止，除了所有的After Effects功能和API的补充，也有很多未来的想法。这些包括映射View到Lottie动画，使用Lottie控制的View的转换，提供Battle Axe的 [RubberHose](http://www.battleaxe.co/rubberhose/)，梯度，类型和图像的支持（译者注：最新版已经支持图像）。最难的部分是下一步先挑选哪些功能实现。

![](http://airbnb.design/wp-content/uploads/2017/01/hardware.png)


# 构建社区

发布开源的东西不仅仅是把它放在那里供大家使用。它是连接人们和创造社区的桥梁。随着我们通过GitHub发布Lottie给设计师和程序猿而走得更近，我们希望确保与动画分支连接。（As we got closer to releasing Lottie to designers and engineers via GitHub, we wanted to be sure to connect with the animation folks as well.）

我们的灵感来自[9 Squares](http://9-squares.tumblr.com/),[ Motion Corpse](https://motioncorpse.tumblr.com/), 以及 Animography 创造的社区。这三个社区都汇集了来自世界各地的人，如果没有这些社区他们将不能在公共动画项目上合作。这些项目需要数月的工作以及组织各自的团队跟人们进行讨论，但这些毋容置疑作为一个整体为动画社区提供了巨大的价值。Motion Corpse 和Animography 公开分享After Effects 源文件，提供关于人们如何创作的不同见解。

追随他们的协作的领先脚步，我们达成协议，这三个团队贡献动画到我们的示例应用程序中。如下图所示，我们已经包括了由J.R. Canest创建一个动画（来自Motion Corpse），一个来自Al Boardman squares的动效 ，和一个使用animography Mobilo动画字体的动效键盘，包含20多个艺术家设计的字母。我们希望整合这些动画社区，强大的工程社区将擦除独特的火花。

![](http://airbnb.design/wp-content/uploads/2017/01/community.gif)

我们很乐意听到你如何使用Lottie-不管你是一名设计师，动画设计师还是程序猿。随时联系我们直接与你们的思想，lottie@airbnb.com反馈和见解。我们很高兴看到世界各地的社区以我们从未想过的方式开始使用Lottie。
>  译者注：airbnb已上线一个Lottie动效采集分享网站
[lottiefiles.com](http://www.lottiefiles.com)

下载 [Bodymovin](https://github.com/bodymovin/bodymovin), Lottie[iOS](https://github.com/airbnb/lottie-ios), [Android](https://github.com/airbnb/lottie-android) 以及 [React Native](https://github.com/airbnb/lottie-react-native).

这篇文章由  Brandon Withrow,  Gabriel Peal 以及 Salih Abdul-Karim 共同完成

[英文版原文链接](http://airbnb.design/introducing-lottie/)
[译文原文链接](http://www.suqishuo.cn/introducing-lottie/lottie-behind-the-scenes-of-our-new-open-source-animation-tool)