<!--
{
    "title": "mysql相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "mysql"
    ],
    "info": []
}
-->

## mysql信息

初始化：`mysql --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql`

## mysql配置

`/etc/my.cnf`

```mysql
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql

# NULL 限制mysqld不允许导入/导出
# /tmp/ 限制mysqld导入/导出只能发生在/tmp/目录
# 没有具体值时，表示不做限制
secure-file-priv= NULL

symbolic-links=0

character-set-server=utf8
collation-server=utf8_general_ci

# 0 依靠InnoDB的主线程 每秒一次 将缓冲数据写入日志文件并刷新到磁盘[mysqld崩溃，1s日志丢失]
# 1 每次提交 将缓冲数据写入日志，并且立即刷到磁盘[1条日志丢失]
# 2 每次提交 将缓冲数据写入日志，每秒刷新一次日志文件，操作系统决定何时刷入磁盘[系统崩溃，1s日志丢失]
innodb_flush_log_at_trx_commit=2
# 0 提交后将二进制日志从缓冲写入磁盘，但是不进行刷新操作
# 1 提交后将二进制文件写入磁盘并立即执行刷新操作
# N 在N次提交后刷新到磁盘
sync_binlog=100

# 超时时间
wait_timeout = 28800
interactive_timeout = 28800

# 连接数
max_connections = 200

[mysql]
default-character-set=utf8

[mysql.server]
default-character-set=utf8

[mysqld_safe]
default-character-set=utf8

[client]
#default-character-set=utf8  # mysqlslap报错

# 引入配置
!includedir /etc/mysql/conf.d/
```