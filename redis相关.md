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

## redis持久化

```bash
# rdb模式 半持久化 丢失最后一次快照后的所有操作
# 使用fork复制父进程的副本，操作系统使用copy-on-write策略，故而rdb文件为fork时的内存数据
# 快照时不会修改rdb文件，快照结束后替换旧文件。可以手动 `save` `bgsave`执行快照，前者主进程/阻塞其他请求，后者fork
# redis启动后会读取rdb文件
save 900 1 # 900s内至少1个键被更改则快照
save 300 10 # 300s内至少10个
# save之间是 或 的关系
dir /path/to/snapshotdir/
dbfilename dump.rdb
rdbcompression yes

# aof模式 全持久化 开启后每执行更改redis数据的命令，都会将该命令写入aof文件
# redis启动时逐个执行aof文件的命令以加载硬盘数据到内存，相对rdb文件较慢
dir /path/to/snapshotdir/
appendonly yes
appendfilename appendonly.aof
appendfsync always # 每次执行写入都执行同步，最安全 最慢 # everysec 每秒 no 操作系统决定/30s
auto-aof-rewrite-percentage 100 # 当aof文件超过上一次重写时的大小的多少百分比时再次进行重写
# 在aof文件在一定大小之后，重新将整个内存写到aof文件当中，以反映最新的状态 相当于`bgsave`
auto-aof-rewrite-min-size 64MB # 允许重写的最小大小 初始化启动redis有效 后续依据aof文件大小

## redis可同时开启rdb和aof，重新启动后使用aof文件恢复数据 rdb可用于备份
```
