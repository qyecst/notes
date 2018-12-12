<!--
{
    "title": "redis相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "redis"
    ],
    "info": [
        "redis cluster//todo"
    ]
}
-->

## redis信息

- K-V型数据库
- 端口tcp6379

## redis安装

```bash
yum install epel-release
yum install redis

# https://redis.io/download
tar -zxvf redis.tgz
# ...
```

## redis配置

```redis
port 6379
daemonize yes # yes以守护进程形式运行，docker中应改为no避免容器退出
requirepass p@ssw0rd # 设置密码
```

## 主从配置

```redis
# 主
requirepass pwd

# 从
slaveof 92.168.1.1 6379
masterauth pwd
requirepass pwd
slave-read-only yes
slave-serve-stale-data yes
```

## 哨兵配置

```redis
# 主
## 配置 /etc/redis-sentinel.conf
daemonize yes
port 26379
sentinel monitor <mymaster> <master ip> 6379 2 # 当有2个及以上的sentinel服务检测到master宕机，执行主从切换的功能
sentinel auth-pass <mymaster> pwd

## 启动
# redis-sentinel /etc/redis-sentinel.conf
# redis-server /etc/redis-sentinel.conf --sentinel

## 确认
> INFO sentinel
> sentinel masters
> sentinel slaves mymaster

# 从
## 配置 /etc/redis-sentinel.conf
daemonize yes
port 26379
sentinel monitor <mymaster> <master ip> 6379 2
sentinel auth-pass <mymaster> pwd
# redis-sentinel /etc/redis-sentinel.conf

# 哨兵
## 确认
> SHUTDOWN # 主
tail /var/log/redis/sentinel.log
> info replication # 从 =>升主
> info replication # 主 =>变为从

redis-cli -p 26379
127.0.0.1:26379> sentinel masters
127.0.0.1:26379> sentinel slaves mymaster
127.0.0.1:26379> sentinel get-master-addr-by-name mymaster
127.0.0.1:26379> sentinel reset mymaster
127.0.0.1:26379> sentinel failover mymaster
127.0.0.1:26379> sentinel flushconfig mymaster
```
