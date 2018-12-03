<!--
{
    "title": "samba相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "smb",
        "nmb",
        "samba"
    ],
    "info": [
        "待测//todo"
    ]
}
-->

## samba信息

- 基于C/S
- 服务smb/nmb
    - smb监听tcp139/445，nmb解析<工作组名-IP>，监听udp137
- 安装：`yum install samba`
- 配置：`/etc/samba/*`

## samba配置

1.创建目录：`mkdir /test && chmod -R 755 /test`

2.用户：`useradd usertest && smbpasswd -a usertest && pdbedit -L`

3.配置：`vim /etc/samba/smb.conf`

```samba
[test]
    comment = 描述信息
    path = /test
    allow hosts = 123.123.123.0/24
    user = usertest
    writable = yes
```

4.测试配置文件：`testparm`

5.使用：`yum install samba-clent` `smbclient -L //<ip addr> -U usertest`
