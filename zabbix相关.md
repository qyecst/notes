<!--
{
    "title": "zabbix相关",
    "create": "2018-12-04 00:32:28",
    "modify": "2018-12-15 15:28:49",
    "tag": [
        "zabbix"
    ],
    "info": [
        "自动发现//todo",
        "邮件报警//todo",
        "web监控//todo",
        "远程命令&脚本//todo",
        "mysql主从监控等//todo"
    ]
}
-->

## 安装zabbix

### server/agent/proxy安装

```bash
# yum源安装
# https://www.zabbix.com/download

rpm -i https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent # 或者 PostgreSQL

# 创建zabbix数据库
mysql -uroot -p<password>
> create database zabbix character set utf8 collate utf8_bin;
> grant all privileges on zabbix.* to zabbix@localhost identified by 'password';
> quit;

# 导入数据库信息
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

# 编辑配置文件
vim /etc/zabbix/zabbix_server.conf
> DBHost=localhost
> DBName=zabbix
> DBUser=zabbix
> DBPassword=password

vim /etc/httpd/conf.d/zabbix.conf
> php_value date.timezone Asia/Shanghai

# 启动/开机自启
systemctl restart zabbix-server zabbix-agent httpd
systemctl enable zabbix-server zabbix-agent httpd

# 编译安装
# https://www.zabbix.com/download_sources

tar -zxvf zabbix-4.0.0.tar.gz
groupadd zabbix
useradd -g zabbix zabbix

# 创建zabbix数据库
mysql -uroot -p<password>
> create database zabbix character set utf8 collate utf8_bin;
> grant all privileges on zabbix.* to zabbix@localhost identified by '<password>';
> quit;

# 导入数据信息
cd database/mysql
mysql -uzabbix -p<password> zabbix < schema.sql
# 如果是创建proxy的数据库，不用执行下述sql
mysql -uzabbix -p<password> zabbix < images.sql
mysql -uzabbix -p<password> zabbix < data.sql

# 编译安装 Zabbix server and agent
./configure --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2 # curl 版本7.20.0以上
make && make install

# 选择指定的mysql客户端/主机上有多个版本的mysql/mariadb时很有用
./configure --with-mysql=/<path_to_the_file>/mysql_config
# 使用PostgreSQL而不是mysql/mariadb
./configure --enable-server --with-postgresql --with-net-snmp
# proxy以及使用sqlite文件/sqlite文件需要绝对路径
./configure --prefix=/usr --enable-proxy --with-net-snmp --with-sqlite3 --with-ssh2
# agent
./configure --enable-agent

# 编辑配置
vim /usr/local/etc/zabbix_agentd.conf
> Server=192.168.1.1 # 允许来抓取数据
> ServerActive=192.168.1.1 # 发送监控数据
> Hostname=some_name/ip_address # agent名，要与server配置的主机名相同

vim /usr/local/etc/zabbix_server.conf
> DBHost=localhost
> DBName=zabbix
> DBUser=zabbix
> DBPassword=password

vim /usr/local/etc/zabbix_proxy.conf
> Server=192.168.1.1 # 指向Zabbix Server
> Hostname=some_name/ip_address # 主机名
> DBHost=localhost # 指定数据库
> DBName=zabbix_proxy # 数据库名
> DBUser=zbuser # 数据库用户
> DBPassword=zbpass # 数据库密码
> Timeout=6
> LogSlowQueries=3000
> DataSenderFrequency=30
> HistoryCacheSize=128MB
> CacheSize=128MB

# 启动
zabbix_server
zabbix_agentd
zabbix_proxy
```

### 安装web界面

```bash
# 复制php文件
mkdir <htdocs>/zabbix
cd frontends/php
cp -a . <htdocs>/zabbix
# locale/make_mo.sh # svn安装并且想改语言 # `msgfmt` 需要被安装

# 打开网页 http://<server_ip_or_name>/zabbix
# 按照界面步骤设置zabbix
# 下载配置文件放在先前拷贝的php文件中子目录 conf/ 下
# 配置完成后进入web界面 默认用户名 Admin 密码 zabbix
```

### 安装Java gateway

```bash
# https://www.zabbix.com/documentation/4.0/manual/concepts/java/from_sources

# 安装java gateway会创建许多目录，建议指定prefix
./configure --enable-java --prefix=$PREFIX
make && make install

# 配置
## settings.sh
> setting.sh文件内设置javagateway
## vim zabbix_server.conf / zabbix_proxy.conf
> JavaGateway=192.168.3.14
> JavaGatewayPort=10052
> StartJavaPollers=5 # 默认server不会启用相关进程
# 需要重启server/proxy服务
```

## docker-compose部署zabbix

### mysql配置文件

```mysql
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

character-set-server=utf8
collation-server=utf8_general_ci
innodb_flush_log_at_trx_commit=2

[mysql]
default-character-set=utf8

[mysql.server]
default-character-set=utf8

[mysqld_safe]
default-character-set=utf8

[client]
#default-character-set=utf8

# Custom config should go here
!includedir /etc/mysql/conf.d/
```

### docker-compose文件

```dockercompose
version: "3.5"

services:
    zabbix-mysql:
        image: mysql
        restart: always
        container_name: zabbix-mysql
        hostname: zabbix-mysql
        environment:
            MYSQL_ROOT_PASSWORD: <u_db_root_passwd>
            MYSQL_DATABASE: zabbix
            MYSQL_USER: zabbix
            MYSQL_PASSWORD: <u_db_passwd>
        volumes:
            - ./mysql.cnf:/etc/mysql/my.cnf:ro
            - ./mysqldata/:/var/lib/mysql/:rw
    zabbix-server:
        image: zabbix/zabbix-server-mysql
        restart: always
        container_name: zabbix-server
        hostname: zabbix-server
        ports:
            - "10051:10051"
        environment:
            DB_SERVER_HOST: zabbix-mysql
            MYSQL_DATABASE: zabbix
            MYSQL_USER: zabbix
            MYSQL_PASSWORD: <u_db_passwd>
    zabbix-web:
        image: zabbix/zabbix-web-nginx-mysql
        restart: always
        container_name: zabbix-web
        hostname: zabbix-web
        ports:
            - "80:80"
        environment:
            DB_SERVER_HOST: zabbix-mysql
            MYSQL_DATABASE: zabbix
            MYSQL_USER: zabbix
            MYSQL_PASSWORD: <u_db_passwd>
            ZBX_SERVER_HOST: zabbix-server
            PHP_TZ: Asia/Shanghai
    zabbix-agent:
        image: zabbix/zabbix-agent
        restart: always
        container_name: zabbix-agent
        hostname: zabbix-agent
        ports:
            - "10050:10050"
        environment:
            ZBX_HOSTNAME: <zbx_host_name|zabbix -> 添加主机 的主机名>
            ZBX_SERVER_HOST: <172.24.0.1|zabbix服务器的ip>
        privileged: true # docker内的zbx-agent，需要权限读取宿主机的数据/不如直接部署zbx-agent至主机
```

### 目录结构

```txt
zabbix
├── docker-compose.yaml
├── mysql.cnf
└── mysqldata
```

## 其他问题

### 中文乱码

```bash
# 选择中文字体 '楷体' '黑体' .etc
cp font.ttf /var/www/html/zabbix/fonts/DejaVuSans.ttf # 拷贝至web相关目录并命名为DejaVuSans.ttf
```

### 触发命令/执行脚本

```bash
# agent设置开启支持
vim zabbix_agentd.conf
> EnableRemoteCommands=1

# 配置zabbix用户sudo并且无需密码登录
vim sudoers.d/zabbix
> Defaults:zabbix !requiretty
> zabbix ALL=(ALL) NOPASSWD: ALL

zabbix_get -s 192.168.1.2 -k "system.run[sudo systemctl restart httpd]" # server端重启agent端的httpd服务
# 服务端Configuration-Actions-Triggers -- Remote command进行设置
```

### 添加自定脚本监控

```bash
vim zabbix_agentd.conf
> UserParameter=some.keyvalue,sh /path/to/script.sh

# server端
zabbix_get -s 192.168.1.2 -k some.keyvalue
# 服务端添加监控项 Key中填写 some.keyvalue 即可，Type为zabbix agent
```
