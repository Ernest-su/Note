---
title: openwrt多拨笔记
date: 2019-08-01 22:12:37
tags: openwrt
---



公司对每个ip限速300k，平时下载文件甚至打开网页都很慢，之前斐讯0元购撸了个k2路由器，刷了openwrt，配了多拨，但是后来重新刷路由器固件后多拨配置丢了，年久未搞，配了很久都单线多拨失败。今日经过百度多篇文章，最终根据文章的图文操作，弄清楚多拨会用到哪些配置文件，特此记录下来，往后搞单线多拨只需按照笔记无脑敲命令就行了，甚至连luci都不需要。



首先更新openwrt的软件源

opkg update

然后按照必要的文件

opkg install luci kmod-macvlan luci-app-mwan3

在/etc/rc.local中exit上一行加入一下配置

sleep 6
ip link add link eth0.2 name vth1 type macvlan
ifconfig vth1 up

ip link add link eth0.2 name vth2 type macvlan
ifconfig vth2 up

ip link add link eth0.2 name vth3 type macvlan
ifconfig vth3 up

ip link add link eth0.2 name vth4 type macvlan
ifconfig vth4 up

ip link add link eth0.2 name vth5 type macvlan
ifconfig vth5 up

ip link add link eth0.2 name vth6 type macvlan
ifconfig vth6 up



在/etc/config/network中，把原有的config interface各项去掉， 修改成一下版本：
//这个wan修改一下

config interface 'wan'       
        option proto 'dhcp'    
        option ifname 'eth0.2'   
        option metric '20'

config interface 'vwan1'       
        option proto 'dhcp'    
        option ifname 'vth1'   
        option metric '21'

config interface 'vwan2'       
        option proto 'dhcp'    
        option ifname 'vth2'   
        option metric '22'   

config interface 'vwan3'       
        option proto 'dhcp'    
        option ifname 'vth3'   
        option metric '23'
        
config interface 'vwan4'       
        option proto 'dhcp'    
        option ifname 'vth4'   
        option metric '24'        
        
config interface 'vwan5'       
        option proto 'dhcp'    
        option ifname 'vth5'   
        option metric '25'
        
config interface 'vwan6'       
        option proto 'dhcp'    
        option ifname 'vth6'   
        option metric '26'        

在 /etc/config/firewall 中

config zone                                     
option name 'wan'                       
option input 'REJECT'                   
option output 'ACCEPT'                  
option forward 'REJECT'                 
option masq '1'                         
option mtu_fix '1'                      
option network 'wan wan6 vwan1 vwan2 vwan3 vwan4 vwan5 vwan6'



在 /etc/config/mwan3 中

config interface 'vwan1'
        option enabled '1'
        option initial_state 'online'
        option family 'ipv4'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option check_quality '0'
        option timeout '2'
        option interval '5'
        option failure_interval '5'
        option recovery_interval '5'
        option down '3'
        option up '3'
        list track_ip '8.8.8.8'

config interface 'vwan2'
        option enabled '1'
        option initial_state 'online'
        option family 'ipv4'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option check_quality '0'
        option timeout '2'
        option interval '5'
        option failure_interval '5'
        option recovery_interval '5'
        option down '3'
        option up '3'
        list track_ip '8.8.8.8'

config interface 'vwan3'
        option enabled '1'
        option initial_state 'online'
        option family 'ipv4'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option check_quality '0'
        option timeout '2'
        option interval '5'
        option failure_interval '5'
        option recovery_interval '5'
        option down '3'
        option up '3'
        list track_ip '8.8.8.8'

config interface 'vwan4'
        option enabled '1'
        option initial_state 'online'
        option family 'ipv4'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option check_quality '0'
        option timeout '2'
        option interval '5'
        option failure_interval '5'
        option recovery_interval '5'
        option down '3'
        option up '3'
        list track_ip '8.8.8.8'

config interface 'vwan5'
        option enabled '1'
        option initial_state 'online'
        option family 'ipv4'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option check_quality '0'
        option timeout '2'
        option interval '5'
        option failure_interval '5'
        option recovery_interval '5'
        option down '3'
        option up '3'
        list track_ip '8.8.8.8'

config interface 'vwan6'
        option enabled '1'
        option initial_state 'online'
        option family 'ipv4'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option check_quality '0'
        option timeout '2'
        option interval '5'
        option failure_interval '5'
        option recovery_interval '5'
        option down '3'
        option up '3'
        list track_ip '8.8.8.8'

config member 'mwan'
        option interface 'wan'

config member 'mvwan1'
        option interface 'vwan1'

config member 'mvwan2'
        option interface 'vwan2'     

config member 'mvwan3'
        option interface 'vwan3'

config member 'mvwan4'
        option interface 'vwan4'   

config member 'mvwan5'
        option interface 'vwan5'

config member 'mvwan6'
        option interface 'vwan6'              

config policy 'balanced'
        option last_resort 'default'
        list use_member 'mwan'
        list use_member 'mvwan1'
        list use_member 'mvwan2'
        list use_member 'mvwan3'
        list use_member 'mvwan4'
        list use_member 'mvwan5'
        list use_member 'mvwan6'


 最后重启一下路由器

 reboot


​        
使用大文件测速专用下载测试一下

100g   http://repos.mia.lax-noc.com/speedtests/100gb.bin
10t       http://repos.mia.lax-noc.com/speedtests/10tb.bin        