<!--
{
    "title": "ansible相关",
    "create": "2018-12-21 23:14:14",
    "modify": "2018-12-21 23:14:14",
    "tag": [
        "ansible"
    ],
    "info": []
}
-->

## 安装

```bash
# https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

yun install epel-release
yum install ansible
```

## 配置

```bash
## /etc/ansible/
# hosts被管理机ip/主机名列表 ansible.cfg主配置文件 roles/角色或插件路径
# hosts
1.1.1.1
2.2.2.2
[webserver]
3.3.3.3
4.4.4.4

www.test.example.com

## 多模块管理
ansible-doc -l # 查看模块
ansible-doc <mod_name> # 查看模块帮助
```

## 常用操作

```bash
# /etc/ansible/hosts
[web]
1.1.1.1
2.2.2.2
[db]
3.3.3.3
4.4.4.4
# 分成web和db两组，本机也可以是被管理机

-v # 详细信息
-i <path> # 指定hosts路径
-f <num> # 指定fork开启同步进程数，默认5
-m <name> # 指定模块名称，默认command
-a <mod_arg> # 模块参数或命令
-k # 输入远程被管理端密码
-sudo # 基于sudo执行
-K # 与sudo使用，输入sudo密码
-u # 指定执行用户
-C # 不改变真实内容，相当于测试
-T # 执行超时时间，默认10s
--version # 版本信息
```

### ping模块

```bash
ansible all -k -m ping # 可使用ssh-copy-id -i .ssh/id_rsa.pub u_name@<u_ip> 方式ssh公钥验证，不输密码

# localhost不在all里
127.0.0.1 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

### command模块

```bash
chdir # 执行命令前切换目录
creates # 文件存在时不执行改步骤
executable # 用shell环境执行命令
free_form # 需要执行的脚本
removes # 文件不存在时，不执行该步骤
warn # 若在ansible.cfg存在告警，若设置false则不会警告此行

## 执行date命令
ansible -i /etc/ansible/hosts all -k -m command -a "date"

127.0.0.1 | CHANGED | rc=0 >>
Fri Dec 21 23:31:04 CST 2018

## 执行ping命令
ansible all -a "ping -c 3 www.baidu.com"

127.0.0.1 | CHANGED | rc=0 >>
PING www.a.shifen.com (61.135.169.125) 56(84) bytes of data.
64 bytes from 61.135.169.125 (61.135.169.125): icmp_seq=1 ttl=128 time=28.2 ms
64 bytes from 61.135.169.125 (61.135.169.125): icmp_seq=2 ttl=128 time=30.5 ms
64 bytes from 61.135.169.125 (61.135.169.125): icmp_seq=3 ttl=128 time=28.0 ms
--- www.a.shifen.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 28.058/28.945/30.578/1.172 ms

## 正则模式执行df -h
ansible 127.0.0.* -a "df -h"

127.0.0.1 | CHANGED | rc=0 >>
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   17G  1.9G   16G  11% /
devtmpfs                 475M     0  475M   0% /dev
tmpfs                    487M  124K  487M   1% /dev/shm
tmpfs                    487M   20M  467M   5% /run
tmpfs                    487M     0  487M   0% /sys/fs/cgroup
/dev/sda1               1014M  130M  885M  13% /boot
tmpfs                     98M     0   98M   0% /run/user/0
```

### copy模块

```bash
src # 源文件/目录，空目录不复制
content # 替代src，将指定文件内容复制到远程文件内
dest # 目标文件/目录，绝对路径
backup # 复制前，备份远程上的原始文件
directory_mode # 用于复制文件夹，新建的文件被复制，老旧的不被复制
follow # 支持link文件复制
force # 覆盖远程不一致的内容
group owner mode # 设置远程文件的用户组/用户/权限

### 复制文件
ansible all -k -m copy -a 'src=/etc/passwd dest=/tmp/ mode=755 owner=root'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "checksum": "b27d2f5924df8ec21f78ee14c076746d9d8b3838", 
    "dest": "/tmp/passwd", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "f262fa40a82026c3a646159f85d9391f", 
    "mode": "0755", 
    "owner": "root", 
    "secontext": "unconfined_u:object_r:admin_home_t:s0", 
    "size": 1118, 
    "src": "/root/.ansible/tmp/ansible-tmp-1545406728.22-10114933231732/source", 
    "state": "file", 
    "uid": 0
}

### 复制文件内容
ansible all -k -m copy -a 'content="hello world" dest=/tmp/passwd mode=744'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "checksum": "2aae6c35c94fcfb415dbe95f408b9ce91ee846ed", 
    "dest": "/tmp/passwd", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "5eb63bbbe01eeed093cb22bb8f5acdc3", 
    "mode": "0744", 
    "owner": "root", 
    "secontext": "unconfined_u:object_r:admin_home_t:s0", 
    "size": 11, 
    "src": "/root/.ansible/tmp/ansible-tmp-1545406826.61-2027016378642/source", 
    "state": "file", 
    "uid": 0
}

### 复制文件内容并备份
ansible all -k -m copy -a 'content="other words" dest=/tmp/passwd backup=yes'

127.0.0.1 | CHANGED => {
    "backup_file": "/tmp/passwd.20894.2018-12-21@23:42:32~", 
    "changed": true, 
    "checksum": "a805a4164f7b71cb87def3304723f651e4a10b40", 
    "dest": "/tmp/passwd", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "25c6d97753b3cb3fae751cc0faed7449", 
    "mode": "0744", 
    "owner": "root", 
    "secontext": "unconfined_u:object_r:admin_home_t:s0", 
    "size": 11, 
    "src": "/root/.ansible/tmp/ansible-tmp-1545406951.72-262319852208374/source", 
    "state": "file", 
    "uid": 0
}
```

### yum模块

```bash
conf_file # 远程yum执行所依赖的配置文件
disable_gpg_check # 是否检测gpg key
name # 安装的软件名称/软件组名称
update_cache # 安装前更新缓存
enablerepo # 指定repo源
skip_broken # 跳过异常软件节点
state # 软件包状态installed|present|latest|absent|removed

### 安装sysstat screen软件
ansible all -k -m yum -a 'name=sysstat,screen state=installed'

127.0.0.1 | CHANGED => {
    "ansible_facts": {
        "pkg_mgr": "yum"
    }, 
    "changed": true, 
    "msg": "", 
    "rc": 0, 
    "results": [
        "sysstat-10.1.5-17.el7.x86_64 providing sysstat is already installed", 
        "Loaded plugins: fastestmirror ... Complete!\n"
    ]
}

### 卸载screen软件
ansible all -k -m yum -a 'name=screen state=absent'

127.0.0.1 | CHANGED => {
    "ansible_facts": {
        "pkg_mgr": "yum"
    }, 
    "changed": true, 
    "msg": "", 
    "rc": 0, 
    "results": [
        "Loaded plugins: fastestmirror ... Complete!\n"
    ]
}
```

### file模块

```bash
src # 源文件/目录
follow # 支持link复制
force # 覆盖远程不一致的内容
group mode owner # 设置远程文件用户组/权限/用户
path # 目标路径，可用dest/name代替
state # 状态file|link|directory|hard|touch|absent
attributes # 文件特殊属性

### 创建目录
ansible all -k -m file -a 'path=/tmp/test state=directory mode=755'
127.0.0.1 | CHANGED => {
    "changed": true, 
    "gid": 0, 
    "group": "root", 
    "mode": "0755", 
    "owner": "root", 
    "path": "/tmp/test", 
    "secontext": "unconfined_u:object_r:user_tmp_t:s0", 
    "size": 6, 
    "state": "directory", 
    "uid": 0
}

### 创建文件
ansible all -k -m file -a 'path=/tmp/test.txt state=touch mode=600'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "dest": "/tmp/test.txt", 
    "gid": 0, 
    "group": "root", 
    "mode": "0600", 
    "owner": "root", 
    "secontext": "unconfined_u:object_r:user_tmp_t:s0", 
    "size": 0, 
    "state": "file", 
    "uid": 0
}
```

### user模块

```bash
system # yes为创建系统用户，默认no，即创建普通用户
append # 添加新的组
comment # 描述信息
createhome # 创建家目录
force # 强制删除用户
group groups # 创建用户主组 将用户加入组/添加附属组
home # 指定家目录
name # 用户名
password # 指定用户密码，为密文/加密后的字符 openssl passwd 'simple'
remove # 删除家目录
shell # 指定用户shell
uid # 设置uid
update_password # 修改用户密码
state # 状态，默认present表示新建

### 创建用户
ansible all -k -m user -a 'name=some_name state=present createhome=yes'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "comment": "", 
    "create_home": true, 
    "group": 1002, 
    "home": "/home/some_name", 
    "name": "some_name", 
    "shell": "/bin/bash", 
    "state": "present", 
    "system": false, 
    "uid": 1002
}

### 删除用户
ansible all -k -m user -a 'name=some_name state=absent remove=yes'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "force": false, 
    "name": "some_name", 
    "remove": true, 
    "state": "absent"
}
```

### cron模块

```bash
name # 计划任务名称
cron_file # 替换该用户的计划文件
minute hour day month weekday # 类似crontab的分 时 日 月 周
job # 计划执行的命令/state=present
backup # 是否备份之前的计划任务
user # 计划任务的用户
state # 状态present|absent

### 创建计划任务
ansible all -k -m cron -a 'minute=0 hour=0 day=* month=* weekday=* name="some_name" job="/usr/sbin/update ntp.server.host"'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "envs": [], 
    "jobs": [
        "some_name"
    ]
}
[root@localhost ~]# cat /var/spool/cron/root 
#Ansible: some_name
0 0 * * * /usr/sbin/update ntp.server.host

### 创建并备份
ansible all -k -m cron -a 'minute=1 hour=1 day=* month=* weekday=* name="some_name2" job="/usr/sbin/update ntp.server.host" backup=yes'

127.0.0.1 | CHANGED => {
    "backup_file": "/tmp/crontabMSuhoJ", 
    "changed": true, 
    "envs": [], 
    "jobs": [
        "some_name", 
        "some_name2"
    ]
}

### 删除计划任务
ansible all -k -m cron -a 'name="some_name" state=absent'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "envs": [], 
    "jobs": [
        "some_name2"
    ]
}
```

### synchronize模块

```bash
compress # 开启压缩，默认开启
archive # 是否归档模式同步，保证源/目标文件属性一致
checksum # 是否校验
dirs # 以非递归方式传输目录
links # 同步链接文件
recursive # 是否递归
rsync_opts # 使用rsync参数
copy_links # 同步时是否复制链接
delete # 删除源不存在而目标存在的文件
src # 源目录/文件
dest # 目标目录/文件
dest_port # 目标端口
rsync_path # rsync服务路径
rsync_timeout # 超时时间
set_remote_user # 远程用户
--exclude=.log # 忽略文件，以.log结尾
mode # 同步模式push|pull，默认推送push

### 同步
ansible all -k -m synchronize -a 'src=/etc/sysconfig/network-scripts/ifcfg-ens33 dest=/tmp/iface.cfg'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "cmd": "/usr/bin/rsync --delay-updates -F --compress --archive --out-format=<<CHANGED>>%i %n%L /etc/sysconfig/network-scripts/ifcfg-ens33 /tmp/iface.cfg", 
    "msg": ">f+++++++++ ifcfg-ens33\n", 
    "rc": 0, 
    "stdout_lines": [
        ">f+++++++++ ifcfg-ens33"
    ]
}

### 同步/忽略文件
ansible all -m synchronize -a 'src=/etc/sysconfig/network-scripts/ dest=/tmp/iface.d/ compress=yes delete=yes rsync_opts=--no-motd,--exclude=ifdown*,--exclude=ifup*,--delete-excluded'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "cmd": "/usr/bin/rsync --delay-updates -F --compress --delete-after --archive --no-motd --exclude=ifdown* --exclude=ifup* --delete-excluded --out-format=<<CHANGED>>%i %n%L /etc/sysconfig/network-scripts/ /tmp/iface.d/", 
    "msg": "cd+++++++++ ./\n>f+++++++++ ifcfg-ens33\n>f+++++++++ ifcfg-lo\n>f+++++++++ init.ipv6-global\n>f+++++++++ network-functions\n>f+++++++++ network-functions-ipv6\n", 
    "rc": 0, 
    "stdout_lines": [
        "cd+++++++++ ./", 
        ">f+++++++++ ifcfg-ens33", 
        ">f+++++++++ ifcfg-lo", 
        ">f+++++++++ init.ipv6-global", 
        ">f+++++++++ network-functions", 
        ">f+++++++++ network-functions-ipv6"
    ]
}
```

### shell模块

```bash
chdir # 执行命令前切换到目录
creates # 文件存在时，不执行此步骤
executable # 用shell环境执行
free_form # 需要执行的脚本
removes # 文件不存在，不执行此步骤
warn # 若在ansible.cfg存在告警，若设置false则不会警告此行

### 执行脚本
ansible all -m shell -a '/bin/sh /tmp/some.sh >> /tmp/some.log'

# some.sh
echo hello world

127.0.0.1 | CHANGED | rc=0 >>

[root@localhost ~]# cat /tmp/some.log 
hello world

### 创建目录
ansible all -m shell -a 'mkdir -p $(date +"%F") chdir=/tmp/ state=directory warn=no'

127.0.0.1 | CHANGED | rc=0 >>

### 检测进程启动
ansible all -m shell -a 'ps aux|grep nginx|grep -v grep'

127.0.0.1 | FAILED | rc=1 >>
non-zero return code
```

### service模块

```bash
enabled # 是否开机自启服务
name # 服务名称
runlevel # 服务启动级别
arguments # 服务命令行参数
state # 状态started|stopped|restarted|reloaded

### 重启httpd服务
ansible all -m service -a 'name=httpd state=restarted'

127.0.0.1 | CHANGED => {
    "changed": true, 
    "name": "httpd", 
    "state": "started", 
    "status": {
        "ActiveEnterTimestampMonotonic": "0", ...
}

### 重启网络服务
ansible all -m service -a 'name=network args=eth0 state=restarted'

 [WARNING]: Ignoring "args" as it is not used in "systemd"

127.0.0.1 | CHANGED => {
    "changed": true, 
    "name": "network", 
    "state": "started", 
    "status": {
}

### 开机自启
ansible all -m systemd -a 'name=crond enabled=yes'

127.0.0.1 | SUCCESS => {
    "changed": false, 
    "enabled": true, 
    "name": "crond", 
    "status": {
}
```
