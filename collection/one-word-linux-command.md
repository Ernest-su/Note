---
title: 一句话笔记：常用linux命令
date: 2021-07-15 17:21:02
tags: linux
---

# 文件系统远程挂载

SMB

//IP地址是windows地址，后面跟上windows上的共享目录，后面的路径是linux上的挂载路径
sudo mount -t cifs  //172.20.4.5/临时文件  /data/sync -o username=test,password=123456,sec=ntlm,vers=1.0

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

# 软连接

sudo ln -s "/data_20t/media/霍元甲港版导演剪辑版" "/data_20t/media/Fearless Hong Kong Version Director's Cut 2006 Blu-Ray 1080p AVC LPCM 7.1"



# 配置 ipv6



```
cat /etc/network/interfaces

# The loopback network interface
auto lo
iface lo inet loopback
auto enp3s0
iface enp3s0 inet dhcp
iface enp3s0 inet6 dhcp
autoconf 1
accept_ra 2
auto enp4s0
iface enp4s0 inet dhcp

```

# 重启网卡

`sudo /etc/init.d/networking restart`
