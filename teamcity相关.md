<!--
{
    "title": "teamcity相关",
    "create": "2018-12-04 14:28:43",
    "modify": "2018-12-04 14:28:43",
    "tag": [
        "teamcity",
        "jetbrains",
        "continuous integration",
        "continuous deployment",
        "continuous delivery",
        "ci/cd"
    ],
    "info": []
}
-->

## teamcity

JetBrains的CI/CD软件

## docker-compose部署方式

### docker-compose文件内容

```dockercompose
version: "3.5"
services:
    teamcity-mysql:
        image: mysql
        restart: always
        container_name: teamcity-mysql
        hostname: teamcity-mysql
        environment:
            MYSQL_ROOT_PASSWORD: <u_db_root_passwd>
            MYSQL_DATABASE: server1
            MYSQL_USER: server1
            MYSQL_PASSWORD: <u_db_passwd>
        volumes:
            - ./mysql.cnf:/etc/mysql/my.cnf:ro
            - ./mysqldata/:/var/lib/mysql/:rw
            - ./outter_dir/mysql/:/opt/outter_dir/:rw # 外部目录，可添加自定义软件/工具/依赖

    teamcity-server-s1: # mysql: db name is `server1`, db user is `server1`, db password is `<u_db_passwd>`
        image: jetbrains/teamcity-server
        restart: always
        container_name: teamcity-server-s1
        hostname: teamcity-server-s1
        depends_on:
            - teamcity-mysql
        ports:
            - "8111:8111"
        volumes:
            - ./ssh_file/:/root/.ssh/:rw # ssh公钥/私钥等
            - ./server1/datadir/:/data/teamcity_server/datadir/:rw # teamcity_server数据目录
            - ./server1/logs/:/opt/teamcity/logs/:rw # teamcity_server日志目录
            - ./outter_dir/server1/:/opt/outter_dir/:rw # 外部目录，可添加自定义软件/工具/依赖

    teamcity-agent-s1-1:
        image: jetbrains/teamcity-agent
        restart: always
        container_name: teamcity-agent-s1-1
        hostname: teamcity-agent-s1-1
        depends_on:
            - teamcity-server-s1
        environment:
            SERVER_URL: teamcity-server-s1:8111
        volumes:
            - ./ssh_file/:/root/.ssh/:rw # ssh公钥/私钥等
            - ./server1/agent1/conf/:/data/teamcity_agent/conf/:rw # teamcity_agent配置文件目录
            - ./outter_dir/s1_agent1/:/opt/outter_dir/:rw # 外部目录，可添加自定义软件/工具/依赖

    teamcity-agent-s1-2:
        image: jetbrains/teamcity-agent
        restart: always
        container_name: teamcity-agent-s1-2
        hostname: teamcity-agent-s1-2
        depends_on:
            - teamcity-server-s1
        environment:
            SERVER_URL: teamcity-server-s1:8111
        #devices:
        volumes:
            - ./ssh_file/:/root/.ssh/:rw # ssh公钥/私钥等
            - ./server1/agent2/conf/:/data/teamcity_agent/conf/:rw # teamcity_agent配置文件目录
            - ./outter_dir/s1_agent2/:/opt/outter_dir/:rw # 外部目录，可添加自定义软件/工具/依赖
            - /dev/bus/usb:/dev/bus/usb:rw # 可以添加宿主机的usb总线，需要`privileged: true`
        privileged: true # 添加权限用于使用宿主机的usb总线

    teamcity-agent-s1-3:
        image: jetbrains/teamcity-agent
        restart: always
        container_name: teamcity-agent-s1-3
        hostname: teamcity-agent-s1-3
        depends_on:
            - teamcity-server-s1
        environment:
            SERVER_URL: teamcity-server-s1:8111
        volumes:
            - ./ssh_file/:/root/.ssh/:rw # ssh公钥/私钥等
            - ./server1/agent3/conf/:/data/teamcity_agent/conf/:rw # teamcity_agent配置文件目录
            - ./outter_dir/s1_agent3/:/opt/outter_dir/:rw # 外部目录，可添加自定义软件/工具/依赖
```

### mysql配置内容

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

### 目录结构

```dir
teamcity
├── docker-compose.yaml
├── mysql.cnf
├── mysqldata
├── outter_dir
├── scripts
├── server1
└── ssh_file
```
