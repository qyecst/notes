<!--
{
    "title": "puppet相关",
    "create": "2018-12-21 20:26:02",
    "modify": "2018-12-21 20:26:02",
    "tag": [
        "puppet"
    ],
    "info": []
}
-->

## 安装

```bash
# 软件 https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm
yum install puppet-server
yum install puppet

# 启动/防火墙/安全设置
# puppetmaster start | iptables/firewalld | selinux

# puppet证书,客户端向服务端第一次链接时会发起证书申请
puppet agent --server <server.host> --test

# 服务端审核证书
puppet cert --list [--all] # 查看申请证书客户端 [查看已发的所有证书]
puppet cert -s <agent.host.cert> # 为特定主机发证书
puppet cety -s and -a # 给所有发证书
```

## 配置

```bash
# /etc/puppet/manifests/

# 在agent创建test.txt文件,并写入内容
# vim /etc/puppet/manifests/site.pp
node default{ # node主机节点 default表所有 可为特定主机
    file{ # 基于file模块的管理
        "/tmp/test.txt": # 文件名
            content => "hello world"; # 内容
    }
}

# 客户端同步,获取服务配置
ntpdate <server.host.ntp> # 要求时间需要同步 ntp/chrony
puppet agent --server <server.host> --test
```

## 常见模块/资源

```puppet
# puppet describe -l # 查看支持的模块/资源
# puppet describe -s [mod_name] # 查看对应帮助信息
file # 文件模块
package # 软件包管理
service # 服务管理
cron # 自动任务计划
exec # 远程执行命令
```

### file

```puppet
ensure # 默认为文件/目录
backup # 通过filebucket备份
checksum # 检查文件是否修改
ctime/mtime # 只读 文件更新/修改时间
content # 内容/与source/target互斥
force # 强制执行删除
owner/group # 用户/用户组
link # 软连接
mode # 权限
path # 路径

## 服务端下载nginx.conf至客户端/tmp
# nginx.conf复制到/etc/puppet/files/
# vim /etc/puppet/fileserver.conf
[files]
path /etc/puppet/files/
allow *

# 创建site.pp
node default{
    file{
        "/tmp/nginx.conf":
            mode => "666",
            owner => "root",
            group => "root",
            source => "puppet://server.host/files/nginx.conf";
    }
}

# 客户端同步
puppet agent --server=server.host

## 服务端下载sysctl.conf,客户端存在则备份sysctl.conf.bak,后覆盖源文件
node default{
    file{
        "/etc/sysctl.conf":
            source => "puppet://server.host/files/sysctl.con",
            backup => ".bak_$uptime_seconds";
    }
}

## 创建/export/docker的软连接为/var/lib/docker/
node default{
    file{
        "/var/lib/docker":
            ensure => link,
            target => "/export/docker";
    }
}

## 创建目录/tmp/test
node default{
    file{
        "/tmp/test":
            ensure => directory;
    }
}
```

### package

```puppet
present # 检测软件存在/不存在则安装
installed # 安装软件
absent # 删除/无依赖,当被依赖时不删除
purged # 删除配置和依赖
latest # 升级最新
version # 指定安装版本

## 安装ntpdate screen
node default{
    package{
        ["ntpdate", "screen"]:
            ensure => "installed";
    }
}

## 卸载ntpdate screen
node default{
    package{
        ["ntpdate", "screen"]:
            ensure => "absent";
    }
}
```

### service

```puppet
enable # 指定服务开机自启 true|false
ensure # 是否运行服务 running|stopped
name # 守护进程名字
path # 启动脚本搜索路径
provider # 默认为init
hasrestart # 管理脚本是否支持restart,不支持则使用stop start实现restart
hasstatus # 是否支持status参数,利用status判断服务是否运行,不支持则查找进程列表

## 启动httpd服务/停止nfs服务
node default{
    service{
        "httpd":
            ensure => "running";
        "nfs":
            ensure => "stopped";
    }
}

## 启动httpd服务并开机自启/停止nfs并开机不自启
node default{
    service{
        "httpd":
            enable => true,
            ensure => running;
        "nfs":
            enable => false,
            ensure => stopped;
    }
}
```

### exec

```puppet
command # 执行的命令
creates # 执行命令生成的文件
cwd # 命令执行目录/不存在则失败
group # 命令执行的账户组
logoutput # 是否记录输出
onlyif # 只在onlyif设定的命令返回0时才执行exec
path # 命令执行的搜索路径
refresh # 刷新命令执行状态true|false
refreshonly # 可使得命令变成仅刷新触发
return # 指定返回代码
timeout # 运行命令最长时间
tries # 重试次数,默认1
try_sleep # 重试间隔 单位s 秒
user # 执行命令用户
provider # shell|windows
environment # 设定额外的环境变量,设定path时path会被覆盖

## tar解压nginx软件包
node default{
    exec{
        "tar zxvf nginx.ver.tgz":
            path => ["/usr/bin", "/bin"],
            user => "root",
            group => "root",
            timeout => "10",
            command => "tar -zxvf /tmp/nginx.ver.tgz";
    }
}

## 远程执行auto_install.sh
node default{
    file{
        "/tmp/auto_install.sh":
            source => "puppet://server.host/files/auto_install.sh",
            owner => "root",
            group => "root",
            mode => 755;
    }
    exec{
        "/tmp/auto_install.sh":
            cwd => "/tmp",
            user => "root",
            path => ["/usr/bin", "/usr/sbin", "/bin/sh"];
    }
}

## 更新sysctl.conf 文件发生变化则执行sysctl -p
node default{
    file{
        "/etc/sysctl.conf":
            source => "puppet://server.host/files/sysctl.conf",
            owner => "root",
            group => "root",
            mode => 644;
    }
    exec{
        "sysctl refresh kernel config":
            path => ["/usr/bin", "/usr/sbin", "/bin", "/sbin"],
            command => "sysctl -p",
            subscribe => File["/etc/sysctl.conf"],
            refreshonly => true;
    }
}
```

### cron

```puppet
user # 某个用户的crontab任务,默认为运行puppet的用户
command # 执行的命令,默认为title名称
ensure # 表示是否启用true|false
environment # 环境变量
hour # 设置crontab小时 0-23
minute # 设置crontab分钟 0-59
month # 设置crontab月份 1-12
monthday # 设置月天数 1-31
weekday # 设置星期 0-7
name # crontab名字,区分不同的crontab
provider # crontab默认的crontab程序
target # crontab作业存放位置

## 添加ntpdate任务
node default{
    cron{
        "ntpdate":
            command => "/usr/sbin/ntpdate ntp.server.host",
            user => "root",
            hour => 0,
            minute => 0;
    }
}

## 删除ntpdate任务
node default{
    cron{
        "ntpdate":
            command => "/usr/sbin/ntpdate net.server.host",
            user => "root",
            hour => 0,
            minute => 0,
            ensure => absent;
    }
}
```

## 常用操作

### 自动认证

```bash
vim /etc/puppet/puppet.conf

[main]
autosign = true

# 重启服务
# 清除证书
puppet cert --clean agent.cert.host
# agent清除证书
rm -rf /var/lib/puppet/ssl
# 获取
puppet agent --server server.host --test
```

### agent同步

```bash
# 启动服务,server配置了node,agent会30min同步
vim /etc/sysconfig/puppet

PUPPET_SERVER = server.host # server端主机名
PUPPET_PORT = 8140 # puppet server使用端口
PUPPET_LOG = /var/log/puppet/puppet.log # 日志文件路径
PUPPET_EXTRA_OPTS = --waitforcert=500 # server证书返回等待时间

# 修改同步时间
vim /etc/puppet/puppet.conf

[agent]
runinterval =60 # 60s同步一次配置
```

### 主动推送

```bash
# 修改客户端配置
vim /etc/puppet/puppet.conf
[agent]
listen = true

vim /etc/sysconfig/puppet
PUPPET_SERVER = server.host

vim namespaceauth.conf
[puppetrunner]
allow *

vim auth.conf
path /run
method save
allow *

# 重启puppet agent服务,之后服务端执行
puppet kick -d agent.host
# puppet kick -d `cat hosts.txt`
```

### 模块化

```bash
# vim /etc/puppet/modules/ntp/manifests/init.pp
class ntp{
    Exec { path => "/bin:/sbin:/bin/sh:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin"}
    exec {
        "auto config ntp crontab":
            command => "sed -i -e '/ntpdate/s/0/*\5/2' -e 's/pool.ntp.org/10.0.0.1/' /var/spool/cron/root";
    }
}

# /etc/puppet/manifests/
# modules.pp
import "ntp"
# nodes.pp
node default{
    include ntp
}

# vim site.pp
import "modules.pp"
import "nodes.pp"

## 正则节点名
node /^some_str\d+\.net { # 匹配some_str{0..10..100...}.net 的节点
    include ntp
}
```
