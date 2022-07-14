开启转发功能
vim /etc/sysctl.conf

输入net.ipv4.ip_forward=1

使用命令sysctl -p使配置生效 


iptables -t nat -A PREROUTING -p udp --dport 123 -j DNAT --to 192.168.1.129:1234  
 
查看本机上有哪些已经配置好的iptables规则(如第三条就是我们使用上面命令添加的一条规则)
 iptables -t nat -xnvL PREROUTING   
 ```
 $ iptables -t nat -xnvL PREROUTING                                                              #查看iptables规则
Chain PREROUTING (policy ACCEPT 135 packets, 7956 bytes)
    pkts      bytes target     prot opt in     out     source               destination         
  640072 34743448 LOG        udp  --  *      *       0.0.0.0/0            0.0.0.0/0            multiport dports 5683,5682,5681 LOG flags 0 level 4 prefix "[DNT]-->1.2 "
  640072 34743448 DNAT       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            multiport dports 5683,5682,5681 to:192.168.1.2
       0        0 DNAT       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            multiport dports 5683 to:192.168.1.2
 ```
 
删除一条已经存在的iptables规则(我们删掉上面的第二条规则，并再次查看所、有规则)
 iptables -t nat -D PREROUTING 2
 
 iptables -F -t nat #删除所有规则
