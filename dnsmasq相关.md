<!--
{
    "title": "dnsmasq相关",
    "create": "2018-05-16 14:30:06",
    "modify": "2018-12-16 22:38:25",
    "tag": [
        "dnsmasq",
        "dns"
    ],
    "info": []
}
-->

## dnsmasq安装

```bash
yum install dnsmasq # centos
apk add dnsmasq # alpine
```

## dnsmasq配置

```conf
# /etc/dnsmasq.conf

# 用户
user=root
# 不使用主机hosts作解析配置
no-hosts
# 解析域名文件，格式类似 /etc/hosts
addn-hosts=/dns/hosts
# 配置文件目录
conf-dir=/etc/dnsmasq.d/,*.conf
# 使用全部的上级dns服务器，选响应最快的
all-servers
# 上级dns服务器
server=223.5.5.5
server=223.6.6.6
server=1.1.1.1
# 缓存大小
cache-size=32768
# 缓存时间 秒
min-cache-ttl=600
```

## docker中使用dnsmasq

基于alpine：

```dockerfile
# 使用alpine，体积小
FROM alpine:3.8

# 换速度快的源，安装dnsmasq
RUN echo "http://mirrors.aliyun.com/alpine/v3.8/main/" > /etc/apk/repositories && \
    apk --update add dnsmasq

# 暴露端口
EXPOSE 53 53/udp

# 启动参数
ENTRYPOINT ["dnsmasq", "-k"]
```

使用docker-compose启动：`docker-compose up -d`

```dockercompose
# 版本
version: "3.5"

# 服务
services:
    dnsmasq:
        build: "."
        restart: "always"
        container_name: "dnsmasq"
        ports:
            - "53:53/tcp"
            - "53:53/udp"
        volumes:
            - "./dns/:/dns/:rw"
            - "./dnsmasq.conf:/etc/dnsmasq.conf:ro"
```

目录结构：

```file
dns_dnsmasq
├── dns
│   └── hosts
├── dnsmasq.conf
├── docker-compose.yaml
└── Dockerfile
```
