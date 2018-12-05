<!--
{
    "title": "bind相关",
    "create": "2018-09-02 11:26:43",
    "modify": "2018-12-05 15:54:56",
    "tag": [
        "bind",
        "dns"
    ],
    "info": []
}
-->

## bind安装

安装:`yum install bind`

## bind配置

### bind配置文件

```bind
# /etc/named.conf

options {
    listen-on port 53 { any; }; // 监听端口/地址
    listen-on-v6 { none; }; // ipv6监听端口/地址
    pid-file "/var/run/named/named.pid"; // pid文件的路径名
    session-keyfile "/var/run/named/session.key"; // TSIG会话密钥的文件的路径名
    directory "/var/named"; // 工作目录
    dump-file "/var/named/data/cache_dump.db"; // rndc dumpdb命令时，服务器存放数据库文件的路径名
    statistics-file "/var/named/data/named_stats.txt"; // rndc stats命令时，服务器会将统计信息追加到的文件路径名
    memstatistics-file "/var/named/data/named_mem_stats.txt"; // 内存使用统计文件的路径名
    recursing-file "/var/named/data/named.recursing"; // rndc recursing命令指定转储当前递归请求到的文件路径
    secroots-file "/var/named/data/named.secroots"; // rndc secroots命令服务器转储安全根的目的文件的路径名
    recursion yes; // 开启递归查询
    allow-recursion { any; }; // 允许哪些主机可以通过本服务器进行递归查询
    allow-query { any; }; // 允许哪些主机可以通过本服务器进行普通dns查询
    allow-query-cache { any; }; // 对缓存的访问控制
    forwarders { // 转发查询的dns服务器
        223.5.5.5;
        223.6.6.6;
        1.1.1.1;
    };
    allow-transfer { slave_dns_ip; }; // 哪些地址允许和本地服务器进行域传输
    notify yes; // 区域变更发送通知
    also-notify { slave_dns_ip; }; // 有更新的区被加载，向表中的服务器和区中ns记录的服务器发送notify消息
};
logging { // 日志记录
    // 通道指定了应该向哪里发送日志数据
    channel default_debug {
            file "/var/named/data/named.run";
            severity dynamic; // 动态，使用主机的记录级别
    };
    channel query_log {
            file "/var/named/log/query.log" versions 1 size 100m; // versions指定存在几个版本log，size指定文件大小，不指定versions满大小后停止写入
            severity info;
            print-category yes;
            print-severity yes;
            print-time yes;
    };
    channel notify_log {
            file "/var/named/log/notify.log" versions 8 size 128m;
            severity info;
            print-category yes;
            print-severity yes;
            print-time yes;
    };
    channel xfer_in_log {
            file "/var/named/log/xfer_in.log" versions 100 size 10m;
            severity info;
            print-category yes;
            print-severity yes;
            print-time yes;
    };
    channel xfer_out_log {
        file "/var/named/log/xfer_out.log" versions 100 size 10m;
        severity info;
        print-category yes;
        print-severity yes;
        print-time yes;
    };
    // 类别则规定了哪些数据需要记录
    category notify { notify_log; };
    category queries { query_log; };
    category xfer-in { xfer_in_log; };
    category xfer-out { xfer_out_log; };
};
zone "." IN {
    type hint;
    file "named.ca";
};
zone "master_domain.com" IN {
    type master;
    file "pri_zone/master_domain.com.zone";
};
zone "10.in-addr.arpa" IN {
    type master;
    file "pri_zone/master_domain_arpa.arpa.zone";
};
```

### bind区域文件

```zone
# pri_zone/master_domain.com.zone

$TTL 1D
@       IN      SOA     test.do. root.localhost.  ( ; test.do. 域
                                    2002081601  ; serial 每次修改后需要增减serial值以便于bind主从同步
                                    28800       ; refresh
                                    14400       ; retry
                                    604800      ; expiry
                                    86400 )     ; minimum
@               IN      NS      ns              ; ns记录
ns              IN      A       10.1.1.1        ; ns解析记录
www             IN      A       10.1.1.2        ; 域名解析记录 www 添加.test.do => www.test.do 或 www.test.do.


# pri_zone/master_domain_arpa.arpa.zone

$TTL 1D
;主机记录 IN 权威记录开始 域（注意.的存在） 邮箱
@       IN      SOA     test.do. root.localhost. ( ; test.do. 域
                                    2002081601  ; serial 每次修改后需要增减serial值以便于bind主从同步
                                    28800       ; refresh
                                    14400       ; retry
                                    604800      ; expiry
                                    86400 )     ; minimum
@               IN      NS      ns.test.do.     ; ns反解记录
2.1.1           IN      PTR     www.test.do.    ; 域名反解地址 10.1.1.2 反着写ip
```

### bind主从配置

```bind
# 主服务器
options {
    listen-on port 53 { any; };
    listen-on-v6 { none; };
    pid-file "/var/run/named/named.pid";
    session-keyfile "/var/run/named/session.key";
    directory "/var/named";
    dump-file "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    recursing-file "/var/named/data/named.recursing";
    secroots-file "/var/named/data/named.secroots";
    recursion yes;
    allow-recursion { any; };
    allow-query { any; };
    allow-query-cache { any; };
    forwarders {
        223.5.5.5;
        223.6.6.6;
        1.1.1.1;
    };
    allow-transfer { slave_dns_ip; };  // 主服务器添加从服务器ip，允许zone文件传输。从服务器无需添加
    notify yes;
    also-notify { slave_dns_ip; }; // 主服务器添加从服务器ip，允许zone文件改变后发送notify。从服务器无需添加
};
zone "." IN {
    type hint;
    file "named.ca";
};
zone "master_domain.com" IN {
    type master;  // type为 master
    file "pri_zone/master_domain.com.zone";  // 需要创建zone文件
};
zone "10.in-addr.arpa" IN {
    type master;  // type为 master
    file "pri_zone/master_domain_arpa.arpa.zone";  // 需要创建zone文件
};

# 从服务器
options {
    listen-on port 53 { any; };
    listen-on-v6 { none; };
    pid-file "/var/run/named/named.pid";
    session-keyfile "/var/run/named/session.key";
    directory "/var/named";
    dump-file "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    recursing-file "/var/named/data/named.recursing";
    secroots-file "/var/named/data/named.secroots";
    recursion yes;
    allow-recursion { any; };
    allow-query { any; };
    allow-query-cache { any; };
    forwarders {
        223.5.5.5;
        223.6.6.6;
        1.1.1.1;
    };
    allow-transfer { none; };  // 从服务器无需添加transfer，无需设置notify
};
zone "." IN {
    type hint;
    file "named.ca";
};
zone "master_domain.com" IN {
    type slave;  // type为 slave
    masters { master_ip; };  // 添加主服务器ip
    file "slaves/master_domain.com.zone";  // 无需创建zone文件，从主服务器同步
};
zone "10.in-addr.arpa" IN {
    type slave;  // type为 slave
    masters { master_ip; };  // 添加主服务器ip
    file "slaves/master_domain_arpa.arpa.zone";  // 无需创建zone文件，从主服务器同步
};
```

## docker部署bind

### docker-bind配置

文件:`named.conf` or `/etc/bind/named.conf`

```bind
options {
    // zone文件目录
    directory "/var/bind";

    // 允许使用此dns服务器进行递归查询的地址
    allow-recursion {
        127.0.0.1/32;
        10.0.0.0/8;
    };

    // 允许查询dns的客户端ip地址
    allow-query { any; }
    allow-query-cache { any; }; // 对缓存的访问控制

    // 上级dns服务器地址，此服务器没有的域名会向上级查询，上级失败则自己向根dns查询
    forwarders {
        223.5.5.5;
        223.6.6.6;
    };

    // 监听地址，any为所有网卡
    listen-on { any; };
    listen-on-v6 { none; };

    // pid文件位置
    pid-file "/var/run/named/named.pid";

    // 允许zone信息传输的地址
    allow-transfer { none; };
};
// 根域
zone "." IN {
    type hint;
    file "named.ca";
};
// test.do域，10.0.0.0/8
zone "test.do" IN {
    type master;
    file "pri/test.zone";
    allow-update { none; }; // 允许某些客户端/服务器更新dns条目，需支持dns submit
};
// test.do反解域，10.0.0.0/8
zone "10.in-addr.arpa" IN {
    type master;
    file "pri/10.zone";
    allow-update { none; };
};
```

文件：`pri/test.zone` or `/var/bind/pri/test.zone`

```bind
$TTL 1D
@       IN      SOA     test.do. root.localhost.  (
                                    2002081601  ; serial 每次修改后需要增减serial值以便于bind主从同步
                                    28800       ; refresh
                                    14400       ; retry
                                    604800      ; expiry
                                    86400 )     ; minimum
@               IN      NS      ns              ; ns记录
ns              IN      A       10.1.1.1        ; ns解析记录
www             IN      A       10.1.1.2        ; 域名解析记录
```

文件：`pri/10.zone` or `/var/bind/pri/10.zone`

```bind
$TTL 1D
;主机记录 IN 权威记录开始 域（注意.的存在） 邮箱
@       IN      SOA     test.do. root.localhost. (
                                    2002081601  ; serial 每次修改后需要增减serial值以便于bind主从同步
                                    28800       ; refresh
                                    14400       ; retry
                                    604800      ; expiry
                                    86400 )     ; minimum
@               IN      NS      ns.test.do.     ; ns反解记录
2.1.1           IN      PTR     www.test.do.    ; 域名反解地址
```

### docker-bind主从配置

```bind
// 主 # named.conf
options {
    directory "/var/bind";
    allow-recursion { any; };
    allow-query { any; };
    allow-query-cache { any; };
    forwarders {
        223.5.5.5;
        223.6.6.6;
        1.1.1.1;
    };
    listen-on { any; };
    listen-on-v6 { none; };
    pid-file "/var/run/named/named.pid";
    allow-transfer { slave_ip; };
};
zone "." IN {
    type hint;
    file "named.ca";
};
zone "master_domain.com" IN {
    type master;
    file "bind_pri/master_domain.zone";
};
zone "10.in-addr.arpa" IN {
    type master;
    file "bind_pri/master_domain_arpa.zone";
};

// 从 # named.conf
options {
    directory "/var/bind";
    allow-recursion { any; };
    allow-query { any; };
    allow-query-cache { any; };
    forwarders {
        223.5.5.5;
        223.6.6.6;
        1.1.1.1;
    };
    listen-on { any; };
    listen-on-v6 { none; };
    pid-file "/var/run/named/named.pid";
    allow-transfer { none; };
};
zone "." IN {
    type hint;
    file "named.ca";
};
zone "master_domain.com" IN {
    type slave;
    masters { master_ip; };
    file "slaves/master_domain.zone";
};
zone "10.in-addr.arpa" IN {
    type slave;
    masters { master_ip; };
    file "slaves/master_domain_arpa.zone";
};
```

### docker-bind的Dockerfile

基于alpine：

```dockerfile
# 使用alpine
FROM alpine:3.8

# 换源，安装bind
RUN echo "http://mirrors.aliyun.com/alpine/v3.8/main/" > /etc/apk/repositories && apk add bind

# -f参数前台运行，防止docker退出
ENTRYPOINT [ "/usr/sbin/named", "-c", "/etc/bind/named.conf", "-u", "named", "-f" ]
```

### 使用docker-compose

启动：`docker-compose up -d`

```dockercompose
# docker-compose版本
version: "3"

# 服务列表
services:
    # 服务名
    bind_service:
        # 构建
        build: "./dockerfile/bind/"
        # 启动策略
        restart: "always"
        # 容器名
        container_name: "bind_alpine"
        # 端口
        ports:
            - "53:53/udp"
        # 挂载
        volumes:
            - "named.conf:/etc/bind/named.conf"
            - "bind_pri/master_domain.zone:/var/bind/bind_pri/master_domain.zone"
            - "bind_pri/master_domain_arpa.zone:/var/bind/bind_pri/master_domain_arpa.zone"
```
