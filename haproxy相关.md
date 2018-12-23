<!--
{
    "title": "haproxy相关",
    "create": "2018-12-23 23:56:14",
    "modify": "2018-12-23 23:56:14",
    "tag": [
        "haproxy"
    ],
    "info": []
}
-->

## 信息

lvs基于四层，haproxy支持虚拟主机，工作4/7层，支持session保持，cookie引导，支持url检测后端，负载均衡软件/多种算法

## 安装

```bash
## yum安装

yum install haproxy

## 编译 http://www.haproxy.org/

tar -zxvf haproxy-1.8.9.tar.gz
cd haproxy-1.8.9/
make TARGET=linux310 PREFIX=/usr/local/haproxy
# [root@localhost ~]# uname -r
# 3.10.0-957.1.3.el7.x86_64 => linux310
make install PREFIX=/usr/local/haproxy
```

## 常用操作

```bash
## 启动
haproxy -f /path/to/etc/conf.cfg
```

## 配置

```haproxy
# /etc/haproxy.cgf | /usr/local/haproxy/etc/haproxy.cfg

global # 全局配置
    maxconn 20480 # 默认最大连接数
    log 127.0.0.1 local3 # err warning info debug 日志
    chroot /usr/local/haproxy # chroot路径
    uid 99
    gid 99
    daemon # 后台形式运行haproxy
    nbproc 8 # 进程数/多进程提高性能/报错src/cfgparse.c修改if(nbproc>1){...etc
    pidfile /path/to/file.pid
    ulimit -n 65535 # ulimit限制
defaults # 可被用于frontend backend listen组件
    log global
    mode http # 基于7层，处理的类别 4层tcp
    maxconn 20480
    option httplog # 日志类别http格式
    option httpclose # 每次请求完成主动关闭连接
    option dontlognull # 不记录健康检查日志信息
    option forwardfor # 后端server需要真实ip时配置，从http header获得real ip
    option redispatch # serverid对应服务down后强制定向到其他服务器
    option abortonclose # 服务器负载高时，结束当前队列久的连接
    stats refresh 30 # 统计页面刷新时间
    retries 3 # 3次连接失败认为服务不可用
    balance roundrobin # 负载均衡方式 roundrobin/source/leastconn 轮询/iphash/最小连接
    contimeout 5000 # 连接超时
    clitimeout 50000 # 客户端超时
    srvtimeout 50000 # 服务器超时
    timeout check 2000 # 心跳检测超时
listen admin_status # 监控设置名称/自定义/frontend backend组合体
    bind 0.0.0.0:65532 # 监听ip port
    mode http
    log 127.0.0.1 local3 err
    stats refresh 5s # 5s刷新监控页面
    stats uri /admin?stats # 监控页面url
    stats realm some\ notice # 监控页面提示信息
    stats auth admin:admin # 用户名密码，可设置多个
    stats hide-version # 隐藏版本信息
    stats admin if TRUE # 手动启用/禁用后端服务器
listen site_status # 监控后端服务器状态
    bind 0.0.0.0:1081
    mode http
    log 127.0.0.1 local3 err
    monitor-uri /site_status # 网站健康检测url，正常200，异常503
    acl site_dead nbsrv(server_web) lt 2 # 定义网站down策略，当负载均衡上指定后端有效机器数小于1返回true
    monitor fail if site_dead # 满足策略时返回503/500
    monitor-net 192.168.1.1/32 # 来自此ip的日志信息不会被记录/转发
    monitor-net 192.168.1.2/32
frontend http_80_in # frontend配置，可定义多个acl
    bind 0.0.0.0:80
    mode http
    log global
    option httplog
    option httpclose
    option forwardfor
    acl some_web hdr_reg(host) -i ^(www.test.com|www3.test.com)$ # 请求域名满足返回true，-i忽略大小写
    # acl some hdr(host) -i test.com # 满足test.com返回true
    # acl file_req url_sub -i killall= # 在url包含killall= 则返回true
    # acl dir_req url_dir -i allow # 在url存在allow作为部分地址路径 则返回true
    # acl missing_cl hdr_cnt(Content-length) eq 0 # header的content-length为0 则true
    # block if missing_cl # 阻止length=0的请求，返回403
    # block if !file_req || dir_req # 不满足file_req或满足dir_req 返回403
    use_backend server_web if some_web # 满足some_web使用server_web的backend
backend server_web # backend设置
    mode http
    balance roundrobin
    cookie SERVERID # 允许插入serverid到cookie/serverid后续定义
    option httpchk GET /index.html # 心跳检测文件
    server web1 1.1.1.1:80 cookie web1 check inter 1500 rise 3 fall 3 weight 1 # cookie表serverid为web1 check inter 1500检测心跳频率 rise 3 为3此正确可使用 fall 3为3次失败认为不可用 weight权重
    server web2 2.2.2.2:80 cookie web2 check inter 1500 rise 3 fall 3 weight 2
```
