<!--
{
    "title": "openvpn相关",
    "create": "2018-09-02 11:11:34",
    "modify": "2018-12-02 16:07:34",
    "tag": [
        "openvpn",
        "vpn"
    ],
    "info": [
        "windows下代理全部流量未做//todo"
    ]
}
-->

## 安装

下载地址：`https://openvpn.net/index.php/download/community-downloads.html`

CentOS安装：

```yum
yum install -y epel-release
yum update -y
yum install -y openvpn easy-rsa
```

## 软件配置

```bash
mkdir -p /etc/openvpn/easy-rsa/
cp -r /usr/share/easy-rsa/<3.0>/* /etc/openvpn/easy-rsa/  # <3.0>为版本号，视情况而定
```

修改软件配置文件

`cp /usr/share/doc/openvpn-<2.4.6>/sample/sample-config-files/server.conf /etc/openvpn/  # <2.4.6>为版本号，视情况而定`

修改`vim /etc/openvpn/server.conf`文件：

```conf
# 端口
port 1194
# 协议 tcp|udp
proto tcp
# 允许模式 tap|tun
dev tun
# ca证书
ca ca.crt
# server证书
cert server.crt
# server密匙
key server.key
# diffie hellman文件
dh dh2048.pem
# 分配给客户端的ip地址池
server 10.80.0.0 255.255.255.0
push "route 10.80.0.0 255.255.255.0"
# 推送给客户端的路由条目
push "route 10.10.0.0 255.255.254.0"
push "route 10.20.0.0 255.255.254.0"
push "route 10.40.0.0 255.255.254.0"
# push "redirect-gateway def1 bypass-dhcp"  # 可以重定向客户端的网关
# push "dhcp-option DNS 8.8.4.4"  #向客户端推送的DNS信息
# push "dhcp-option DNS 223.5.5.5"  #向客户端推送的DNS信息
# 连接保持时限
keepalive 10 120
# tls验证文件，server端0，客户端1
tls-auth ta.key 0
# 加密方式
cipher AES-256-CBC
# 运行时用户
user openvpn
# 运行时组
group openvpn
# keepalive超时后，重新启动VPN，不重新读取keys，保留第一次使用的keys
persist-key
# keepalive检测超时后，重新启动VPN，一直保持tun或者tap是linkup。否则会先linkdown然后linkup
persist-tun
# 当前状态文件
status log/openvpn-status.log
# log文件
#log        log/openvpn.log
# log追加
log-append  log/openvpn.log
# 日志冗余级别 0-9
verb 3
# explicit-exit-notify 1  #此选项开启只能使用udp协议
# 加入脚本处理，如密码验证
script-security 3
# 指定认证脚本
auth-user-pass-verify /etc/openvpn/checkpwd.sh via-env
# client-cert-not-required  # 不请求客户端的CA证书
# 使用客户端提供的username作为commonname
username-as-common-name
```

`vim /etc/openvpn/checkpwd.sh`文件：

```bash
#!/bin/sh

PWD_FILE="/etc/openvpn/pwd-file"
LOG_FILE="/etc/openvpn/log/open-password.log"
TIME_TAG=`date "+%Y-%m-%d %T"`

if [ ! -r "${PWD_FILE}" ]; then
  echo "${TIME_TAG}: Permission Denied- ${PWD_FILE}." >> ${LOG_FILE}
  exit 1
fi

ACCESS_PWD=`awk '!/^;/ && !/^#/ && !/^$/ && $1=="'${username}'" { print $2; exit; }' ${PWD_FILE}`

if [ "${ACCESS_PWD}" = "" ]; then
  echo "${TIME_TAG}: Invalid User- username= ${username}  password= ${password}" >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${ACCESS_PWD}" ]; then
  echo "${TIME_TAG}: Login Success- username= ${username}" >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_TAG}: Login Failure- username= ${username}  password= ${password}" >> ${LOG_FILE}
exit 1
```

`vim /etc/openvpn/pwd-file`文件：

```txt
test    test
test2   test2
```

## 生成证书等文件

添加`vars`文件：`vim /etc/openvpn/easy-rsa/vars`

或是`cp /usr/share/doc/easy-rsa-<3.0>/vars.example /etc/openvpn/easy-rsa/vars`

```bash
export KEY_DIR="$EASY_RSA/keys"  # 定义keys的生成目录
export KEY_SIZE=2048             # 定义生成私钥的大小，默认2048，执行build-dh命令生成dh2048文件的依据
export CA_EXPIRE=3650            # ca证书有效期，默认为3650天，即十年
export KEY_EXPIRE=3650           # 定义秘钥的有效期，默认为3650天，即十年
export KEY_COUNTRY="CN"          # 定义所在的国家
export KEY_PROVINCE="ZJ"         # 定义所在的省
export KEY_CITY="Hangzhou"       # 定义所在的城市
export KEY_ORG="xx"              # 定义所在的组织
export KEY_EMAIL="xx@xx.com"     # 定义邮箱
export KEY_OU="xx"               # 定义所在单位
export KEY_NAME="xx"             # 定义服务器名称
# =================================
set_var EASYRSA_REQ_COUNTRY    "US"
set_var EASYRSA_REQ_PROVINCE   "California"
set_var EASYRSA_REQ_CITY       "San Francisco"
set_var EASYRSA_REQ_ORG        "Copyleft Certificate Co"
set_var EASYRSA_REQ_EMAIL      "me@example.net"
set_var EASYRSA_REQ_OU         "My Organizational Unit"
# ... etc.
```

生成相应文件：

```bash
# 命令详解
init-pki  # 建立一个空的pki结构，生成一系列的文件和目录
build-ca [ cmd-opts ]  # 创建ca
gen-dh  # 创建diffie hellman文件
gen-req <base_filename> [ cmd-opts ]  # 创建证书
sign-req <type> <base_filename>  # 签约证书
```

```bash
# 操作
cd /etc/openvpn/easy-rsa/
./easyrsa init-pki  # 初始化
./easyrsa build-ca  # 创建ca，输入密码
./easyrsa gen-req server nopass  # 创建server证书，无密码
./easyrsa sign server server  # 签名server证书，输入ca密码
./easyrsa gen-dh  # 生成dh文件
./easyrsa gen-req client nopass  # 创建client证书，无密码
./easyrsa sign client client  # 签名client证书，输入ca密码
```

生成`ta.key`文件，用于`tls-auth`：

```bash
openvpn --genkey --secret ta.key
```

## 系统设置

开启路由转发功能：

```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
syctl -p  # 生效
```

设置`iptables`规则：

```bash
iptables -I INPUT -p [tcp|udp] -m [tcp|udp] --dport [port-number|1194] -j ACCEPT
iptables -t nat -I POSTROUTING -s [network|10.80.0.0/24] -j MASQUERADE
service iptables save  # 保存防火墙配置，若报错 `yum install iptables-services` 或 `iptables-save > /etc/sysconfig/iptables`
#iptables -I FORWARD -d 10.80.0.0/24 -j ACCEPT
#iptables -I FORWARD -s 10.80.0.0/24 -j ACCEPT
```

设置`firewalld`规则：

```bash
firewall-cmd --add-masquerade
firewall-cmd --add-masquerade --permanent
firewall-cmd --add-port=[port_number|1194]/[tcp|udp]
firewall-cmd --add-port=[port_number|1194]/[tcp|udp] --permanent
```

## 客户端连接

`client.ovpn`配置文件：

```conf
client
dev tun
remote [remote_addr_ip|0.0.0.0] [remote_port_number|1194]
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
verb 3
auth-nocache
proto tcp
auth-user-pass

#ca ca.crt
#cert client.crt
#key client.key
#tls-auth ta.key 1

<ca>
    ca.crt的内容
</ca>

<cert>
    client.crt的内容
</cert>

<key>
    client.key的内容
</key>

key-direction 1

<tls-auth>
    ta.key的内容
</tls-auth>
```

linux连接：`openvpn --daemon --cd [配置文件路径] --config [配置文件名称|client.ovpn] --log-append [日志路径|/var/log/openvpn.log]`

windows连接：下载`https://openvpn.net/index.php/download/community-downloads.html`

mac连接：下载`https://tunnelblick.net/downloads.html`