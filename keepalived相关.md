<!--
{
    "title": "keepalived相关",
    "create": "2018-12-23 23:56:14",
    "modify": "2018-12-23 23:56:14",
    "tag": [
        "keepalived",
        "redis",
        "mysql",
        "nginx",
        "haproxy",
        "lvs"
    ],
    "info": []
}
-->

## 信息

keepalived/heartbeat

工作在layer3/4/7层，监控检查和vrrp冗余协议

layer3: 使用layer3工作方式，定期向服务器发送icmp数据包，若无法ping通则认为失效，以ip是否有效作为服务器正常与否的标准

layer4: 使用tcp端口状态确定服务器正常与否，检测到端口未启动则认为失效

layer7: 工作于应用层，依据用户设定的程序检测服务器程序的运行是否正常，以此判定是否失效

vrrp: 通过竞选协议实现虚拟路由器功能，通过组播224.0.0.18实现，虚拟路由器由vrid和ip组成，对外表现同一mac地址。master发送vrrp广播包，backup不会抢占master，除非开启抢占以及优先级高。master down时，多台backup中高优先级的成为master，获得vip

## 安装

```bash
## yum源安装
yum install keepalived

# 编译安装 http://www.keepalived.org/download.html
# 安装  依赖 libnfnetlink-devel zlib zlib-devel gcc gcc-c++ openssl openssl-devel openssh popt-devel
# download
# tar -zxvf keepalived.tgz
# ./configure
# make && make install

## 启动脚本 /usr/local/src/keepalived-1.3.5/keepalived/etc/init.d/keepalived
# 修改配置 Source configuration file (we set KEEPALIVED_OPTIONS there)
. /usr/local/keepalived/etc/sysconfig/keepalived
keepalived=/usr/local/keepalived/sbin/keepalived
keepalived_config=/usr/local/keepalived/etc/keepalived/keepalived.conf
keepalived_pid=/usr/local/keepalived/run/keepalived.pid
```

## 常用操作

### nginx+keepalived

```keepalived
## 安装nginx
## 安装keepalived

## 配置keepalived
# keepalived.conf
global_defs {
    notification_email {
        root@localhost
   }
   notification_email_from root@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL # 标识 本 节点 的名称
}
# 检测脚本
vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_check.sh"
    interval 2 # 每2秒检测一次nginx的运行状态
    weight -20 # 失败一次，将自己的优先级调整为-20
    fall 5 # 如果连续失败次数达到此值，则认为服务器已down
    rise 3 # 如果连续成功次数达到此值，则认为服务器已up，但不修改优先级
}
# VIP1
vrrp_instance VI_1 {
    state MASTER # 状态，主节点为MASTER
    interface ens33 # 绑定VIP的网络接口
    # track_interface {
    #     eth1
    # }
    lvs_sync_daemon_interface ens33 # 负载均衡器之间的监控借口，类似HA heartbeat 的心跳线
    virtual_router_id 51 # 虚拟路由的ID号，两个节点设置必须一样
    priority 100 # 节点优先级，值范围0~254，MASTER>BACKUP
    advert_int 5 # 组播信息发送时间间隔，两个节点必须设置一样，默认为1秒
    # nopreempt # 抢占模式
    # 设置验证信息，两个节点必须一致
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟IP，两个节点设置必须一致，可以设置多个
    virtual_ipaddress {
        192.168.1.1
    }
    # nginx存活状态检测脚本
    track_script {
        chk_nginx
    }
}

## nginx脚本 chk_nginx.sh 若nginx失败则stop keepalived实现切换
#!/bin/bash
killall -0 nginx
if [[ $? == 0 ]] ; then
    /etc/init.d/keepalived stop # ???????
fi

## nginx脚本
#!/bin/bash
count=`ps -ef|grep nginx|grep -v grep|wc -l`
if [ $count -gt 0 ];then
    exit 0
else
    exit 1
fi
```

### 双主/充分利用服务器

```keepalived
## master1 config file
global_defs {
    notification_email {
        root@localhost
   }
   notification_email_from root@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_check.sh"
    interval 2
    weight -20
    fall 5
    rise 3
}
# v1
vrrp_instance VI_1 {
    state MASTER # mst
    interface ens33
    lvs_sync_daemon_interface ens33
    virtual_router_id 51 # vrid
    priority 100 # pri
    advert_int 5
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.1 # ip
    }
    track_script {
        chk_nginx
    }
}
# v2
vrrp_instance VI_2 {
    state BACKUP # bak
    interface ens33
    lvs_sync_daemon_interface ens33
    virtual_router_id 52 # vrid
    priority 90 # pri
    advert_int 5
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.2 # ip
    }
    track_script {
        chk_nginx
    }
}
# ===========================================================
## master2 config file
global_defs {
    notification_email {
        root@localhost
   }
   notification_email_from root@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_check.sh"
    interval 2
    weight -20
    fall 5
    rise 3
}
# v1
vrrp_instance VI_1 {
    state BACKUP # bak
    interface ens33
    lvs_sync_daemon_interface ens33
    virtual_router_id 51 # vrid
    priority 90 # pri
    advert_int 5
    nopreempt ## 非抢占
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.1 # ip
    }
    track_script {
        chk_nginx
    }
}
# v2
vrrp_instance VI_2 {
    state MASTER # mst
    interface ens33
    lvs_sync_daemon_interface ens33
    virtual_router_id 52 # vrid
    priority 100 # pri
    advert_int 5
    nopreempt ## 非抢占
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.2 # ip
    }
    track_script {
        chk_nginx
    }
}
```

### 与其他服务组合

#### lvs+keepalived配置

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

# ========
## 更改tcp检测为http_get检测
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

#### redis/nfs/mysql等与keepalived组合

```keepalived
## 安装redis/nfs/mysql
## 安装keepalived

## 配置
# 同上nginx类似

## 脚本
# 同上nginx类似
```

### 发送邮件

```keepalived
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
vrrp_script chk_nginx {
    script "/etc/keepalived/ck_ng.sh"
    interval 2
    weight -5
    fall 3
    rise 2
}
vrrp_sync_group VG_1 {
    group {
        VI_1
    }
    # 节点变为master时执行
    notify_master /etc/keepalived/sendmail.pl
    # notify_fault /path/to/send_mail.sh fault
    # notify_master /path/to/send_mail.sh master
    # notify_backup /path/to/send_mail.sh backup
}
vrrp_instance VI_1 {
    state MASTER
    interface ens32
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_nginx
    }
    virtual_ipaddress {
        192.168.1.1/24
    }
}

## 脚本 权限a+x
> yum install perl-CPAN
> perl -MCPAN -e shell
> capn > install Net::SMTP_auth
#!/usr/bin/perl -w
use Net::SMTP_auth;
use strict;
my $mailhost = 'smtp.server.host';
my $mailfrom = 'sender@server.host';
my @mailto = ('receiver@server.host');
my $subject = 'title';
my $text = "content";
my $user = 'sender@server.host';
my $passwd = 'passwd';
&SendMail();
sub SendMail() {
    my $smtp = Net::SMTP_auth->new( $mailhost, Timeout => 120, Debug => 1 ) or die "Error.\n";
    $smtp->auth( 'LOGIN', $user, $passwd );
    foreach my $mailto (@mailto) {
        $smtp->mail($mailfrom);
        $smtp->to($mailto);
        $smtp->data();
        $smtp->datasend("To: $mailto\n");
        $smtp->datasend("From:$mailfrom\n");
        $smtp->datasend("Subject: $subject\n");
        $smtp->datasend("\n");
        $smtp->datasend("$text\n\n");
        $smtp->dataend();
    }
    $smtp->quit;
}
```
