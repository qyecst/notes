<!--
{
    "title": "mysql相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-12 19:26:25",
    "tag": [
        "mysql"
    ],
    "info": [
        "mysql/mariadb cluster//todo",
        "mysql proxy//todo"
    ]
}
-->

## mysql信息

### 初始化

```bash
systemctl start mysqld
systemctl start mariadb

# datadir数据文件路径 basedir安装路径
#mysql --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql

#mysql_install_db --user=mysql --datadir=/data/mysql
#mysqladmin -u root password 'new-password'
mysql_secure_installation
```

### 安装：

```bash
# https://dev.mysql.com/downloads/repo/yum/ # yum源安装mysql
# repo文件 yum安装mariadb
#[mariadb]
#name = MariaDB
#baseurl = http://yum.mariadb.org/10.1/centos7-amd64
#gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
#gpgcheck=1

yum install mysql-server mysql-devel mysql-libs # centos6
yum install mariadb mariadb-server mariadb-libs # centos7
```

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
# 开启binlog，值为日志前缀
log-bin=mysql_binlog

# 超时时间
wait_timeout = 28800
interactive_timeout = 28800

# 连接数
max_connections = 200

# ONLY_FULL_GROUP_BY SELECT中的列需要在GROUP BY中出现
# PIPES_AS_CONCAT 将"||"视为字符串的连接操作符而非或运算符
# ANSI_QUOTES 启用后不能使用双引号引用字符串，它被解释为识别符
sql_mode = "STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

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

## 主从配置

```bash
## 主
# /etc/my.cnf | my-large.cnf
[mysqld]
server-id = 1
log-bin = mysql-bin

> grant replication slave on *.* to 'replica'@'%' identified by 'pwd';
> flush privileges;
> show master status;
# file:mysql-bin.00002 position: 123

#> use database_name; # 数据库
> flush tables with read lock; # 锁定表/读锁
mysqldump -uroot -pxxx database_name > database_name.sql # 导出数据
> unlock tables; # 解锁表

## 从
# /etc/my.cnf | my-large.cnf
[mysqld]
server-id = 2
#replicate_wild_do_table=test.% #只同步test库下的表
#relay_log=mysqld-relay-bin #记录中继日志
#log-slave-updates=YES #从服务器同步后记录日志

> create database database_name;
> use database_name;
> source database_name.sql; # 导入数据

> change master to master_host='192.168.1.1',
                   master_user='replica',
                   master_password='pwd',
                   master_log_file='mysql-bin.00002',
                   master_log_pos=123;
> slave start;
> show slave status;
```

## binlog常用操作

```mysql
# 查看日志列表
> show master logs;
# 查看master状态，最新一个binlog日志编号，最后一个操作事件pos结束点
> show master status;
# 刷新log日志，开始产生一个新编号的binlog日志文件
> flush logs;
# 清空binlog日志
> reset master;
```

`mysqlbinlog`用于查看binlog

```bash
mysqlbinlog /path/mysql_binlog.00001
```

```mysql
# mysql命令行内; logfile_name 默认第一个日志文件; pos 默认第一个; offset默认0; row_count默认全部;
> show binlog events [IN 'logfile_name'] [FROM pos] [LIMIT [offset,] row_count];
```

## 数据操作

```bash
# 复制
> CREATE DATABASE `db_new` DEFAULT CHARACTER SET UTF8 COLLATE UTF8_GENERAL_CI;
mysqldump -u<user_name> -p<user_pwd> --add-drop-table db_old | mysql -h 192.168.1.1 db_new -u<user_name> -p<user_pwd>

# 备份
mysqldump -u<user_name> -p<user_pwd> --flush-logs[F] --lock-all-tables[l] --log-error=/path/dump.err --databases[B] db1 db2 > /path/db.bak.sql

# 恢复
mysql -u<user_name> -p<user_pwd> -v < /path/db.bak.sql;

# --start-position=[pos]  --stop-position=[pos]  --start-datetime="xxxx-xx-xx xx:xx:xx"  --stop-datetime="xxxx-xx-xx xx:xx:xx"  --database=[db_name]
mysqlbinlog  /path/mysql_binlog.00001 | mysql -u<user_name> -p<user_pwd> -v db_name
mysqlbinlog --stop-position=953 --database=db_name /path/mysql_binlog.00001 | mysql -u<user_name> -p<user_pwd> -v db_name
```

## 用户相关操作

```mysql
# mysql 5.x
CREATE USER 'user' IDENTIFIED BY 'u_passwd';
GRANT ALL PRIVILEGES ON db.* TO 'user'@'%' IDENTIFIED BY 'passwd';
REVOKE ALL PRIVILEGES ON db.* FROM 'user'@'%';
GRANT USAGE ON *.* TO 'user'@'%' WITH MAX_USER_CONNECTIONS 10;

# mysql 8.x
CREATE USER 'user' IDENTIFIED WITH mysql_native_password BY 'passwd';
ALTER USER 'root'@'%' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
```

## 其他操作

```mysql
# 索引，查询块更新慢
ALTER TABLE t1 ADD PRIMARY KEY('column'); # 主键
ALTER TABLE t1 ADD UNIQUE('column'); # 唯一索引
ALTER TABLE t1 ADD INDEX index_name('column'); # 普通索引
ALTER TABLE t1 ADD FULLTEXT('column'); # 全文索引 MyISAM
ALTER TABLE t1 ADD INDEX index_name('column','column2'); # 组合索引

DROP INDEX index_name ON t1; # 删除
ALTER TABLE t1 DROP INDEX index_name;
ALTER TABLE t1 DROP PRIMARY KEY;

SHOW INDEX FROM t1; # 查看
SHOW KEYS FROM t1;

# 慢查询
show variables like '%slow%';
show variables like '%long_query%';

set global slow_query_log = on;

## /etc/my.cnf
[mysqld]
slow_query_log = ON # 开启
slow_query_log_file = /usr/local/mysql/data/slow.log # log文件
long_query_time = 1 # '慢'时间定义
log_queries_not_using_indexes = OFF # 如果ON 记录所有没有利用索引的查询，前提slow_query_log为ON

mysqldumpslow -s t -t 5 /path/to/logfile
```
