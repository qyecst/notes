<!--
{
    "title": "zabbix相关",
    "create": "2018-12-04 00:32:28",
    "modify": "2018-12-04 00:32:28",
    "tag": [
        "zabbix"
    ],
    "info": []
}
-->

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
