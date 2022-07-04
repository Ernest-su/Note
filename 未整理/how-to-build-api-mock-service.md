---
title: 如何快速搭建模拟接口服务
date: 2017-03-03 17:30:55
categories:
    - 开发效率
tags:
    - moco
    - ngrok
    - 安卓
    - api
    - 模拟数据
---


> 相信大家在移动APP开发中，无论是做安卓还是iOS开发，都会遇到一个很尴尬的问题,接口没有开发好。而请求网络接口数据是开发中开发流程的第一步，因此方便快捷的模拟接口返回数据给APP，有助于提高开发效率。

下面介绍2个工具来搭建模拟接口服务器。

- moco
- ngrok


# moco 
> 在过去，我们通常部署一个war包到应用服务器，如Jetty或Tomcat等。我们都知道，开发一个war包然后部署到应用程序服务器太无聊，即使我们使用内嵌服务器，只要我们改变一点点，war包就要重新部署。  

 **moco是一款开源的专注于模拟接口数据的开源工具。它有以下优势：**
- [x] 使用简单：提供下载一个jar文件外加简单的json配置就可运行的方式
- [x] 配置灵活：可以用独立jar包加载json方式配置，也可以写java代码配置
- [x] 高度可配置：可以对url，参数,请求方法，返回内容等进行配置
- [x] 扩展强大：HTTPS，Socket，JUnit 集成，Maven 插件，Gradle 插件，Shell，Scala，满足你不同爱好

## 例子
- 下载 [moco独立运行程序](https://repo1.maven.org/maven2/com/github/dreamhead/moco-runner/0.11.0/moco-runner-0.11.0-standalone.jar)

- 新建moco配置文件如下(foo.json):
```json
[
  {
    "response" :
      {
        "text" : "Hello, Moco"
      }
  }
]
```


- 使用配置文件运行moco独立运行文件.
```shell
java -jar moco-runner-<version>-standalone.jar http -p 12306 -c foo.json
```

- 现在，打开浏览器访问 http://localhost:12306 ，你会看见配置的返回内容"Hello, Moco"

更多使用方法，请参考[moco的github说明文档](https://github.com/dreamhead/moco/blob/master/moco-doc/apis.md#content)

---

# ngrok
如果是个人开发，一个简单灵活的接口服务器就已经搭建成功了。APP只要把上文的`localhost`换成运行moco电脑的IP就能请求到模拟的数据。但是，手机和电脑需要同一个局域网。如果是团队开发，或者手机和电脑不在同一个局域网怎么办？ngrok就是用来解决这个问题的。
> ngrok提供了一个能够在公网安全访问内网Web主机的工具，能捕获所有HTTP请求的内容，也支持TCP端口映射，支持Linux、Windows、Mac OS X 等平台。

**注意**
ngrok V1.X的版本是可以免费支持将一个固定的二级域名指向本机的，不过作者已经把 V2.X的版本商业化,我们使用一个国内免费提供的服务。网址 http://qydev.com

1. 下载windows/linux/mac版本的客户端，解压到你喜欢的目录

1. 在命令行下进入到解压目录下

1. 执行 ngrok -config=ngrok.cfg -subdomain xxx 12306 //(xxx 是你自定义的域名前缀,12306是moco的端口)

1. 如果开启成功 你就可以使用 xxx.tunnel.qydev.com 来访问你本机的 `127.0.0.1:12306`的服务啦

# 小技巧

 打开 http://127.0.0.1:4040 可以查看ngrok访问情况。

 在linux下，ngrok 不能用 & 实现后台运行，我们可以使用使用screen这个命令，步骤如下：

 安装screen apt-get install screen 运行 screen -S

 任意名字（例如：ngork） 然后运行ngrok启动命令

 最后按快捷键`ctrl+A+D`即可保持ngrok后台运行
