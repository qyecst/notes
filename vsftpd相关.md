<!--
{
    "title": "vsftpd相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "vsftpd",
        "ftp"
    ],
    "info": []
}
-->

## ftp信息

- FTP/file transfer protocol文件传输协议
- OSI第七层，应用层
- 控制通道port 21，数据通道port 20
- 模式
    - 主动模式/PORT
        - 客户端发起请求，链接服务端port 21，客户端port N为大于1024的随机端口
        - 服务端port 21响应客户端
        - 服务端打开port 20链接客户端port N+1
        - 客户端响应，开始传输数据
    - 被动模式/PASV
        - 客户端发起请求，链接服务端port 21，客户端port N为大于1024的随机端口
        - 服务端port 21响应客户端
        - 服务端打开大于1024的随机端口，客户端使用port N+1链接服务端打开的端口
        - 服务端响应，开始传输数据
- 软件包：`vsftp`
    - 登入方式：
        - 匿名登入
        - 本地用户
        - 虚拟用户

## ftp配置

```ftp
# 禁用匿名登入
anonymous_enable=NO
# 允许本地用户/虚拟用户登入
local_enable=YES
# 允许写入
write_enable=YES
# 文件默认权限掩码
local_umask=022
# 显示该目录需要注意的内容，显示的档案默认是'.message'，`message_file=.message`
dirmessage_enable=YES
# 使用者上传与下载文件都会被纪录起来，`xferlog_file=/var/log/xferlog`
xferlog_enable=YES
# ftp-data的端口号
connect_from_port_20=YES
# 设定为wuftp相同的登录文件格式？NO因为登录文件会比较容易读，有使用wuftp登录文件的分析软件需要设定为YES
xferlog_std_format=YES
# standalone模式监听ipv4/6，不能同时YES，6YES包含4
listen=YES
listen_ipv6=NO
# pam使用文件名
pam_service_name=vsftpd
# 是否借助vsftpd处理某些账号
userlist_enable=YES
# 支持TCP Wrappers
tcp_wrappers=YES
# 禁止匿名权限
anon_upload_enable=NO
anon_mkdir_write_enable=NO
anon_other_write_enable=NO
# chroot到用户家目录
chroot_local_user=YES
#local_root=/some/path/to/chroot
# 启用chroot时默认主目录不能有w权限，否则保存，使用YES允许w权限
allow_writeable_chroot=YES
# 启用虚拟用户
guest_enable=YES
# 虚拟用户映射实际用户
guest_username=ftpvuser
# 虚拟用户config目录
user_config_dir=/etc/vsftpd/vconf
# 虚拟用户和本地用户有相同的权限
virtual_use_local_privs=YES
### pasv，处理连接了但是无法显示列表问题，开放端口/21 port_minnum~port_maxnum
    pasv_address=<outter_ip/internet_ip>
    pasv_addr_resolve=YES
    pasv_min_port=<port_minnum>
    pasv_max_port=<port_maxnum>
```

虚拟用户：
1.创建用户文件：`vusertxt`

```txt
username1
passwd1
username2
passwd2
```

生成用户验证数据库：`db_load -T -t hash -f vusertxt vuserdb.db`  
权限修改：`chmod 600 vuserdb.db`

2.修改：`/etc/pam.d/vsftpd`文件名为`vsftpd.conf`中的`pam_service_name`所设定

```ftp
auth required /lib[64]/security/pam_userdb.so db=/etc/vsftpd/vuserdb #无后缀
account required /lib[64]/security/pam_userdb.so db=/etc/vsftpd/vuserdb
```

3.创建虚拟用户映射用户：`useradd -d <vuser/path/home> ftpvuser`，用户名为`vsftpd.conf`中的`guest_username`所设定，注意目录以及子目录权限问题

4.虚拟用户设置：配置文件目录为`vsftpd.conf`中的`user_config_dir`所设定，配置文件名为虚拟用户名，一名一文件，配置项同`vsftpd.conf`

```ftp
# file-path:/etc/vsftpd/vconf/username1

local_root=/home/ftpvuser/username1
write_enable=YES
# .etc
```

注意：使用chroot时因vsftpd的安全设置，用户主目录不能有w权限，报错`500 OOPS: vsftpd: refusing to run with writable root inside chroot()`，可在`vsftpd.conf`使用`allow_writeable_chroot=YES`解决