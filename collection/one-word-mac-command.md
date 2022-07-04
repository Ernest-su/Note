---
title: 一句话笔记：常用mac命令
date: 2021-07-15 17:21:02
tags: linxu
---



# 双网卡路由配置

- 查看路由表  netstat -nr
- 删除路由 sudo route -v delete -net 172.20/24
- 添加路由 sudo route -n add -net 172.20.3.0 -netmask 255.255.255.0 192.168.123.1



# 文件系统远程挂载

SMB

//IP地址是windows地址，后面跟上windows上的共享目录，后面的路径是linux上的挂载路径
sudo mount -t cifs  //172.20.4.5/临时文件  /data/sync -o username=test,password=test,sec=ntlm,vers=1.0

NFS

mount -t nfs -o nolock 172.20.4.106:/home /data/sync

开机挂载



# 开机自动挂载文件

修改 /etc/fstab 文件， 挂载命令如：

SMB

//192.168.22.111/shared /home/www/ cifs username=xxx,password=xxx,rw,dir_mode=0777,file_mode=0777 0 0

NFS

172.20.4.106:/home /data/sync nfs username=test,password=test,rw,dir_mode=0777,file_mode=0777 0 0



# curl,apt 设置代理

curl -x socks5h://192.168.3.125:9081 -LO http://google.com

sudo apt-get -o Acquire::http::proxy="socks5h://192.168.3.125:9081/" update
apt-get -o Acquire::http::proxy="socks5h://192.168.3.125:9081/"  install xxx





# 批量替换文件内容



```bash
 sed -i 's/http:\/\/tracker\.m.com/https:\/\/tracker\.m\.com/' *.txt

```
