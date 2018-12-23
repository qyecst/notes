<!--
{
    "title": "lvs相关",
    "create": "2018-12-23 23:56:14",
    "modify": "2018-12-23 23:56:14",
    "tag": [
        "lvs",
        "keepalived"
    ],
    "info": []
}
-->

## 信息

lvs/Linux virtual server。请求lvs vip，依据转发方式和算法，将请求转发给后端服务器，后端服务器处理后返回给用户。

### 负载均衡转发方式

RR(round-robin LC(least-connection WRR(weight-RR WLC(weight-LC

基于局部性的最少链接(Locality-Based Least Connections Scheduling/LBLC: 根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器。若服务器不存在，或者该服务器超载且有服务器处于其一半的工作负载，则用“最少链接”的原则选出一个可用的服务器，将请求发送到该服务器

带复制的基于局部性最少链接(Locality-Based Least Connections with Replication Scheduling/LBLCR: 根据请求的目标IP地址找出该目标IP地址对应的服务器组,按“最小连接”原则从该服务器组中选出一台服务器，若服务器没有超载，将请求发送到该服务器。若服务器超载；则按“最小连接”原则从整个集群中选出一台服务器，将该服务器加入到服务器组中，将请求发送到该服务器。当该服务器组有一段时间没有被修改，将最忙的服务器从服务器组中删除，以降低复制的程度

目标地址散列调度(Destination Hashing Scheduling/DH: 根据请求的目标IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空

源地址散列调度(Source Hashing Scheduling/SH: 根据请求的源IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空

### 工作原理

vs/nat(virtual server via network address translation 用户请求lvs到达director，director将请求的destIP改为后端realserverIP，同时修改port，最后数据发送到realserver，realserver处理后返回director，director将数据发给用户。两次传输经过director，director会成为瓶颈。

vs/dr(virtual server via direct routing 用户请求lvs到达director，director将请求mac修改为realserver的mac，destIP不变还是VIP，源ip为用户ip不变，director将数据发给realserver，realserver检测到destIP为自己本地VIP，处理后直接返回用户。若与用户不同网段则通过网关。

vs/tun(virtual server via IP tunneling 用户请求lvs到达director，director通过IP-TUN加密将请求dest的mac修改为后端realserver的mac，destVIP不变，源IP为用户ip不变，director将数据发给realserver，realserver基于IP-TUN解密，检测destIP为自身本地VIP，处理后直接返回用户，不同网段则过网关。

## 安装

```bash
# ipvs (ip virtual server)工作在内核空间，叫ipvs，生效实现调度的代码
# ipvsadm 工作在用户空间，叫ipvsadm，为ipvs内核框架编写规则，定义集群服务，后端真实的服务器(Real Server)

# DS：Director Server 指的是前端负载均衡器节点
# RS：Real Server 后端真实的工作服务器
# VIP：向外部直接面向用户请求，作为用户请求的目标的IP地址
# DIP：Director Server IP，主要用于和内部主机通讯的IP地址
# RIP：Real Server IP，后端服务器的IP地址
# CIP：Client IP，访问客户端的IP地址

## 编译 http://www.linuxvirtualserver.org/index.html
# download tar.gz file
# tar -zxvf ipvsadm.tgz
# ./configure && make && make install

## yum安装
yum install ipvsadm
```

## 常用操作

### lvs配置/启用

```bash
## 查看lvs信息
ipvsadm -Ln

-A --add-service # 增加一台新的虚拟服务器VIP
-E --edit-service # 编辑内核虚拟服务器表中的一条虚拟服务器记录
-D --delete-service # 删除内核虚拟服务器表中的一条虚拟服务器记录
-C --clear # 清除内核虚拟服务器表中的所有记录
-R --restore # 恢复虚拟服务器规则
-S --save # 保存虚拟服务器规则，输出为-R 选项可读的格式
-a --add-server # 在一个虚拟服务器中增加一台新的真实服务器RIP
-e --edit-server # 编辑一条虚拟服务器记录中的某条真实服务器记录
-d --delete-server # 删除一条虚拟服务器记录中的某条真实服务器记录
-L|-l --list # 显示内核虚拟服务器表
-Z --zero  #虚拟服务表计数器清零/清空当前的连接数量等
--set tcp tcpfin udp # 设置连接超时值
--start-daemon # 启动同步守护进程 master|backup 说明LVSRouter是master或backup可用keepalived的VRRP
--stop-daemon # 停止同步守护进程
-t --tcp-service service-address # 说明虚拟服务器提供的是tcp服务[vip:port] [real-server-ip:port]
-u --udp-service service-address # 说明虚拟服务器提供的是udp服务[vip:port] [real-server-ip:port]
-f --fwmark-service fwmark # 说明是经过iptables标记过的服务类型
-s --scheduler scheduler # 使用的调度算法 rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq 默认的调度算法是wlc
-p --persistent [timeout] #持久稳固的服务 来自同一个客户的多次请求，将被同一台真实的服务器处理 timeout默认值300秒
-M --netmask # 子网掩码
-r --real-server server-address # 真实的服务器[Real-Server:port]
-g --gatewaying 指定LVS的工作模式为直接路由模式/是LVS默认的模式
-i --ipip # 指定LVS的工作模式为隧道模式
-m --masquerading # 指定LVS的工作模式为NAT模式
-w --weight weight #真实服务器的权值
--mcast-interface interface # 指定组播的同步接口
-c --connection # 显示LVS目前的连接 ipvsadm -L -c
--timeout # 显示tcp tcpfin udp的timeout值 ipvsadm -L --timeout
--daemon # 显示同步守护进程状态
--stats # 显示统计信息
--rate # 显示速率信息
--sort # 对虚拟服务器和真实服务器排序输出
-n --numeric # 输出IP地址和端口的数字形式

## 添加VIP => 添加realserver => 启动lvs
ipvsadm --set 30 5 60
ifconfig eth0:0 1.1.1.1 netmask 255.255.255.255 broadcast 1.1.1.1 up
# ip addr add 1.1.1.1/32 dev eth0 label eth0:1
route add -host 1.1.1.1 dev eth0:0
# ip route add 1.1.1.1/32 dev eth0:1

ipvsadm -A -t 1.1.1.1:80 -s wlc -p 120
ipvsadm -a -t 1.1.1.1:80 -r 2.2.2.2:80 -g -w 2
ipvsadm -a -t 1.1.1.1:80 -r 3.3.3.3:80 -g -w 2

ipvsadm -Ln

## 关闭
ipvsadm -C
ipvsadm -Z
ifconfig eth0:0 down
# ip addr del 1.1.1.1/32 dev ens33:1
route del 1.1.1.1
# ip route del 1.1.1.1/32 dev ens33:1

# cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0:1
DEVICE="eth0:1"
BOOTPROTO="none"
IPADDR="1.1.1.1"
NETMASK="255.255.255.255"
NM_CONTROLLED="no"
ONBOOT="yes"
TYPE="Ethernet"

## realserver设置
# start
ifconfig lo:1 1.1.1.1 netmask 255.255.255.255 broadcast 1.1.1.1 up
route add -host 1.1.1.1 dev lo:1
# ip addr add 1.1.1.1/32 dev lo label lo:1
# ip route add 1.1.1.1/32 dev lo:1
echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
# vim /etc/sysctl.conf  sysctl -p
net.ipv4.conf.lo.arp_ignore=1
net.ipv4.conf.lo.arp_announce=2
net.ipv4.conf.all.arp_ignore=1
net.ipv4.conf.all.arp_announce=2

# stop
ifconfig lo:1 down
route del 1.1.1.1
# ip addr del 1.1.1.1/32 dev lo:1
# ip route del 1.1.1.1/32 dev lo:1
echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
# vim /etc/sysctl.conf  sysctl -p
net.ipv4.conf.lo.arp_ignore=0
net.ipv4.conf.lo.arp_announce=0
net.ipv4.conf.all.arp_ignore=0
net.ipv4.conf.all.arp_announce=0
```

### lvs+keepalived配置

```keepalived
# 使用了keepalived.conf配置后，无需ipvsadm -A，配置均在conf文件内
## 安装keepalived
## 安装lvs

## 配置
global_defs {
    notification_email {
        root@localhost
   }
   notification_email_from root@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    lvs_sync_daemon_interface ens33
    virtual_router_id 51
    priority 100
    advert_int 5
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        1.1.1.1
    }
}
virtual_server 1.1.1.1 80{ # 定义对外提供服务的LVS的VIP以及port
    delay_loop 6 # 设置健康检查时间，单位是秒
    lb_algo wrr # 负载均衡方式W-RR
    lb_kind DR # DR模式lvs
    # nat_mask 255.255.255.0
    persistence_timeout 60 # 持续60s
    protocol TCP
    real_server 2.2.2.2 80{
        weight 100
        TCP_CHECK{
            connect_timeout 10
            nb_get_retry 3 # 失败重试次数
            delay_before_retry 3 # 失败重试的间隔时间
            connect_port 80 # 连接的后端端口
        }
    }
    real_server 3.3.3.3 80{
        weight 100
        TCP_CHECK{
            connect_timeout 10
            nb_get_retry 3 # 失败重试次数
            delay_before_retry 3 # 失败重试的间隔时间
            connect_port 80 # 连接的后端端口
        }
    }
}
# ========
## backup的keepalived配置类似
```

### lvs-dr客户端配置vip

```bash
# 为了解决realserver收到destIP为VIP的数据包认为不是自己的而丢弃，所以将VIP配置于lo网卡上/防止都配置vip致使ip冲突/vip不会在物理机产生mac表

# start
ifconfig lo:1 1.1.1.1 netmask 255.255.255.255 broadcast 1.1.1.1 up
route add -host 1.1.1.1 dev lo:1
# ip addr add 1.1.1.1/32 dev lo label lo:1
# ip route add 1.1.1.1/32 dev lo:1
echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
# vim /etc/sysctl.conf  sysctl -p
net.ipv4.conf.lo.arp_ignore=1
net.ipv4.conf.lo.arp_announce=2
net.ipv4.conf.all.arp_ignore=1
net.ipv4.conf.all.arp_announce=2

# stop
ifconfig lo:1 down
route del 1.1.1.1
# ip addr del 1.1.1.1/32 dev lo:1
# ip route del 1.1.1.1/32 dev lo:1
echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
# vim /etc/sysctl.conf  sysctl -p
net.ipv4.conf.lo.arp_ignore=0
net.ipv4.conf.lo.arp_announce=0
net.ipv4.conf.all.arp_ignore=0
net.ipv4.conf.all.arp_announce=0
```

### lvs+keepalived添加url检测

```keepalived
real_server 2.2.2.2{
    weight 100
    HTTP_GET{
        connect_timeout 10
        nb_get_retry 3
        delay_before_retry 3
        url{
            path /path/to/monitor/check
            status_code 200
        }
    }
}
```
