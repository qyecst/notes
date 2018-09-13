[info]: # ({"title":"docker内使用alpine做dns服务器", "create":"2018-09-02 11:26:43", "modify":"2018-09-02 11:26:43", "category":"笔记", "tag_list":["docker", "bind", "dns", "alpine"], "info_list":[]})

docker内使用alpine做dns服务器，~~好像没啥用？~~

[preview]: # (end preview)

## alpine的dockerfile

```dockerfile
# 使用alpine
FROM alpine:3.8

# 换源，安装bind
RUN echo "http://mirrors.aliyun.com/alpine/v3.8/main/" > /etc/apk/repositories && apk add bind

# -f参数前台运行，防止docker退出
ENTRYPOINT [ "/usr/sbin/named", "-c", "/etc/bind/named.conf", "-u", "named", "-f" ]
```

## docker-compose文件

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
            - "pri/10.zone:/var/bind/pri/10.zone"
            - "pri/test.zone:/var/bind/pri/test.zone"
```

## bind配置

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
    allow-update { none; };
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
                                    2002081601  ; serial
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
                                    2002081601  ; serial
                                    28800       ; refresh
                                    14400       ; retry
                                    604800      ; expiry
                                    86400 )     ; minimum
@               IN      NS      ns.test.do.     ; ns反解记录
2.1.1           IN      PTR     www.test.do.    ; 域名反解地址
```

## 启动

`docker-compose up -d`