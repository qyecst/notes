<!--
{
    "title": "redis相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "redis"
    ],
    "info": []
}
-->

## redis信息

- K-V型数据库
- 端口tcp6379

## redis配置

```redis
port 6379
daemonize yes # yes以守护进程形式运行，docker中应改为no避免容器退出
requirepass p@ssw0rd # 设置密码
```