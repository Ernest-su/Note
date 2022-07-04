---
title: curl批量下载并设置正确名称
date: 2022-01-05 14:46:55
categories:
    - linux
tags:
    - linux
---
我有一个文件，其中包含一个或多个链接。使用wget从URL列表中下载带有文件名的文件

http://example.com/00001 
http://example.com/00002 
http://example.com/00003 
http://example.com/00004 
当在浏览器中手动转到其中一个链接时，文件将自动下载。下载的文件具有该文件的唯一名称，与url路径不同。

我知道使用wget -i file-of-links.txt，但下载时，每个文件的标题将基于url，而不是基于文件名。

是否有一个命令可以让我下载给定文件中的链接并用文件名而不是url名保存下载的文件？

可以使用--trust-server-names和--content-disposition选项执行此操作。

--trust-server-names

如果设置为on，对重定向重定向URL的最后一部分将用作本地文件名。默认情况下，它被用来在原来的URL

--content-disposition

如果设置为on，Content-Disposition的头实验（不完全功能）支持启用的最后一个组件。这可能会导致额外往返服务器的HEAD请求，并且已知会遇到一些错误，这就是为什么默认情况下当前不启用它的原因。

此选项对于某些使用Content-Disposition标头描述下载文件名称的文件下载CGI程序很有用。