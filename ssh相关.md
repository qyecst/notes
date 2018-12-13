<!--
{
    "title": "ssh相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-13 22:49:07",
    "tag": [
        "ssh"
    ],
    "info": []
}
-->

## ssh配置

```ssh
/etc/ssh/sshd_config

# 参数详解：https://man.openbsd.org/sshd_config

HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key #私钥地址
HostKey /etc/ssh/ssh_host_ed25519_key
AuthorizedKeysFile .ssh/authorized_keys #设置公钥验证文件的路径
ChallengeResponseAuthentication no #是否允许质疑-应答(challenge-response)认证
GSSAPIAuthentication yes # 使用基于GSSAPI的用户认证
GSSAPICleanupCredentials no # 用户退出登录后自动销毁用户凭证缓存
UsePAM yes # yes的话非对称密钥验证失败，仍然可用口令登录(???)
UsePrivilegeSeparation sandbox # Default for new installations.
Banner none # banner信息
AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
AcceptEnv XMODIFIERS # 指定客户端发送的哪些环境变量将会被传递到会话环境中，应当小心使用，默认是不传递任何环境变量
Subsystem sftp /usr/libexec/openssh/sftp-server # 配置一个外部子系统(例如一个文件传输守护进程)
UseDNS no # 不使用DNS查询客户端
AddressFamily inet # 使用哪种地址族。any(默认)、inet(仅IPv4)、inet6(仅IPv6)
SyslogFacility AUTHPRIV # log记录，包括特权信息如用户名在内的认证活动
PasswordAuthentication yes # 使用口令认证
Port 22 # 端口
Protocol 2 # 协议
PermitRootLogin no # root登入
X11Forwarding no # 禁止用户运行远程主机上的X程序
PermitEmptyPasswords no # 不允许使用空密码的用户登录
MaxStartups 5 # 设置同时发生的未验证的并发量
LoginGraceTime 15s # 15秒内客户端不能登录即登录超时，切断连接
AllowUsers u_user # 仅允许这些用户登录，而拒绝其它所有用户。处理顺序DenyUsers, AllowUsers, DenyGroups, AllowGroups
# AllowGroups <xxx> # 用空格分隔的组名列表，可用*和?通配符，仅允许这些组中的成员登录
```

## 使用公钥/禁用密码

```ssh
RSAAuthentication yes # 允许RSA验证(文档中找不到此设置，是否废弃存疑)
PubkeyAuthentication yes # 允许公钥验证
AuthorizedKeysFile .ssh/authorized_keys # 公钥验证文件的路径

ChallengeResponseAuthentication no # 禁用质疑-应答认证
PasswordAuthentication no # 禁用密码认证
UsePAM no # 禁用pam模块
```
