<!--
{
    "title": "inotify&rsync相关",
    "create": "2018-12-21 01:37:33",
    "modify": "2018-12-21 01:37:33",
    "tag": [
        "inotify",
        "inotify-tools",
        "rsync",
        "rsyncd",
        "rsync daemon"
    ],
    "info": []
}
-->

## inotify

### inotify安装

```bash
# https://github.com/rvoicilas/inotify-tools/wiki
# https://github.com/rvoicilas/inotify-tools

yum install inotify-tools
```

### inotify配置

```bash
echo "655350" > /proc/sys/fs/inotify/max_user_watches
echo "655350" > /proc/sys/fs/inotify/max_queued_events

# vim /etc/sysctl.conf  `sysctl -p`
fs.inotify.max_user_watches = 655350
fs.inotify.max_queued_events = 655350

max_user_watches # 设置inotifywait或inotifywatch命令可以监视的文件数量(单进程)
max_user_instances # 设置每个用户可以运行的inotifywait或inotifywatch命令的进程数
max_queued_events # 设置inotify实例事件(event)队列可容纳的事件数量
```

### inotify常用操作

```bash
# inotifywait 等待特定文件系统事件发生，执行后处于阻塞状态
# inotifywatch 收集被监视的文件系统使用度统计数据

# 测试
inotifywait -mrq -timefmt '%d/%m/%y %H:%M'-format '%T %w%f %e' -e close_write,modify,delete,create,attrib,move /data

–timefmt # 时间格式 配合 -format 的 %T 使用 # %y年 %m月 %d日 %H小时 %M分钟
–format # 输出格式
-m # 始终保持监听状态，默认触发事件即退出’
-r # 递归查询目录
-q # 仅打印监控相关的信息
-e # 定义监控的事件 open 打开文件 access 访问文件 modify 修改文件 delete 删除文件 create 新建文件 attrb 属性变更 .etc
```

### inotify&rsync实时同步

```bash
## 同步修改的文件

#!/bin/bash
# 先cd  inotify再监听 ./ 才能rsync同步后目录结构一致
cd /path/data/ && inotifywait -mrq --format  '%Xe %w%f' -e modify,create,delete,attrib,close_write,move ./ | while read file
do
    INO_EVENT=$(echo $file | awk '{print $1}') # 事件类型部分赋值INO_EVENT
    INO_FILE=$(echo $file | awk '{print $2}') # 文件路径部分赋值INO_FILE
    echo "-------------------------------$(date +'%F %T'): $file------------------------------------"
    # 增加、修改、写入完成、移动进事件
    if [[ $INO_EVENT =~ ('CREATE'|'MODIFY'|'CLOSE_WRITE'|'MOVED_TO') ]] ; then
        echo 'CREATE or MODIFY or CLOSE_WRITE or MOVED_TO'
        rsync -avzR --password-file=/path/to/pwd_file $(dirname ${INO_FILE}) <u_name>@<ip>::<mod>
        # INO_FILE表路径 每次同步发生改变的文件的目录 -R把源的目录结构递归到目标后,保证目录结构一致性
    fi
    # 删除、移动出事件
    if [[ $INO_EVENT =~ ('DELETE'|'MOVED_FROM') ]] ; then
        echo 'DELETE or MOVED_FROM'
        rsync -avzR --delete --password-file=/path/to/pwd_file $(dirname ${INO_FILE}) <u_name>@<ip>::<mod>
        # 同步已删除路径会报错 所以同步被删文件目录上一级 --delete删除目标有而源没有的文件
    fi
    #修改属性 touch chgrp chmod chown等
    if [[ $INO_EVENT =~ 'ATTRIB' ]] ; then
        echo 'ATTRIB'
        if [ ! -d "$INO_FILE" ] ; then # 目录不同步，目录下文件同步时，rsync会更新目录
            rsync -avzR --password-file=/path/to/pwd_file $(dirname ${INO_FILE}) <u_name>@<ip>::<mod>
        fi
    fi
done

## 定时全量同步
# crond: * */2 * * * 每2h同步一次
rsync -avz --password-file=/path/to/pwd_file /data/ <u_name>@<ip>::<mod>
```

## rsync

### rsync安装

```bash
# 安装
yum install rsync
```

## rsync daemon

### daemon信息

远程shell连接的两端是通过管道完成通信和数据传输的，即使连接的一端是远程主机，当连接到目标端时，将在目标端上根据远程shell进程fork出rsync进程使其成为rsync server
rsync daemon是事先在server端上运行好的rsync后台进程(根据启动选项，也可以设置为非后台进程)，它监听套接字等待client端的连接，连接建立后所有通信方式都是通过套接字完成

```bash
# rsync user@host::src dest
# rsync://user@host:port/src dest
rsync --no-motd -r -v -f "+ */" -f "+ linux-4.19*.tar.gz" -f "- *" -m rsync://rsync.kernel.org/pub/ # 查看内核
rsync --no-motd -avzP rsync://rsync.kernel.org/pub/linux/kernel/v4.x/linux-4.19.11.tar.gz /tmp # 下载
```

### rsync配置文件

```rsync
# vim /etc/rsyncd.conf

# 全局配置参数
port=873 # 指定rsync端口。默认873
uid = rsync # rsync服务的运行用户，默认是nobody，文件传输成功后属主将是这个uid
gid = rsync # rsync服务的运行组，默认是nobody，文件传输成功后属组将是这个gid
use chroot = no # rsync daemon在传输前是否切换到指定的path目录下，并将其监禁在内
max connections = 200 # 指定最大连接数量，0表示没有限制
timeout = 300 # 确保rsync服务器不会永远等待一个崩溃的客户端，0表示永远等待
motd file = /var/rsyncd/rsync.motd # 客户端连接过来显示的消息
pid file = /var/run/rsyncd.pid # 指定rsync daemon的pid文件
lock file = /var/run/rsync.lock # 指定锁文件
log file = /var/log/rsyncd.log # 指定rsync的日志文件，而不把日志发送给syslog
dont compress = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2 # 指定哪些文件不用进行压缩传输

# 下面指定模块，并设定模块配置参数，可以创建多个模块
[mod1] # 模块ID
path = /path/to/data/ # 指定该模块的路径，该参数必须指定。启动rsync服务前该目录必须存在。rsync请求访问模块本质就是访问该路径。
ignore errors # 忽略某些IO错误信息
read only = false # 指定该模块是否可读写，即能否上传文件，false表示可读写，true表示可读不可写。所有模块默认不可上传
write only = false # 指定该模式是否支持下载，设置为true表示客户端不能下载。所有模块默认可下载
list = false # 客户端请求显示模块列表时，该模块是否显示出来，设置为false则该模块为隐藏模块。默认true
hosts allow = 10.0.0.0/24 # 指定允许连接到该模块的机器，多个ip用空格隔开或者设置区间
hosts deny = 0.0.0.0/32 # 指定不允许连接到该模块的机器
auth users = rsync_backup # 只有列表里的用户才能连接到模块，用户名密码保存在secrts file中,虚拟用户。默认所有用户都能连接，匿名连接。文件权限600
secrets file = /etc/rsyncd.passwd # 用户名和密码，每行一个username:passwd。由于"strict modes" 默认true，此文件非rsync daemon用户不可读写。文件权限600
[mod2] # 以下定义的是第二个模块
path = /path/to/data2/
read only = false
ignore errors
comment = anyone can access
```

### rsync常用操作

```bash
# 启动
rsync --daemon
systemctl start rsyncd

# 链接,pwd文件第一行认为是密码,其他忽略
rsync --list-only --port 873 <u_name>@<ip>::<mod>/path/file --password-file=/tmp/rsync.conn.passwd
rsync --list-only rsync://<u_name>@<ip>:<873|port>/mod/path/file --password-file=/tmp/rsync.conn.passwd

# 源路径如果是一个目录，不带尾随斜线表示的是整个目录包括目录本身，带上尾随斜线表示的是目录中的文件，不包括目录本身
rsync -av /data root@192.168.1.1:/data/bak/dir/

-a # 归档模式，表示递归传输并保持文件属性。等同于"-rtopgDl"
-v # 显示信息 -vvvv更详细
-r # 递归
-t # 保持mtime属性
-o # 保持owner属性
-p # 保持prem属性/不包括特殊属性
-g # 保持group属性
-D # 也拷贝设备文件和特殊文件
-l # 拷贝软链接本身而非软链接所指向的对象
-n # 仅测试，不实际传输，用于测试 和 -vvvv一起使用
-b # 目标上已存在的文件做一个备份，备份的文件名后默认使用"~"做后缀
-e # 指定远程要使用的程序，默认ssh
--port # 端口号，默认873
-R # 使用相对路径 rsync -R /etc/cron.d /tmp => 备份处：/tmp/etc/cron.d
--delete # 以SRC为准，多的删，少的补。exclude/include规则生效之后才执行
-z # 传输时进行压缩提高效率

## /etc/rsync.conf
[tmpdir]
path=/tmp
auth users=u_user
secrets file=/tmp/tmp_passwd
## 链接 shell + rsync daemon
rsync [options] --rsh=ssh auth_user@host::module
rsync [options] --rsh="ssh -l ssh_user" auth_user@host::module
rsync [options] -e "ssh -l ssh_user" auth_user@host::module
rsync [options] -e "ssh -l ssh_user" rsync://auth_user@host/module
rsync --list-only -e "ssh -l root" u_user@<ip>::tmpdir
```
