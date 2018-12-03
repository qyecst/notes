<!--
{
    "title": "nfs相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "nfs",
        "network file system"
    ],
    "info": [
        "待测//todo"
    ]
}
-->

## nfs信息

- 网络文件系统
- RPC协议远程调用
- 软件包：`rpcbind nfs-utils`
    - `rpcbind`支持安全NFS RPC服务连接
    - `nfs-utils`基本的NFS命令，监控程序.etc
- 守护进程
    - `nfsd`基本守护进程，管理登入.etc
    - `mountd`RPC安装守护进程，管理NFS文件系统.etc
    - `rpcbind`端口映射

## nfs配置

文件：`/etc/exports`

需注意目录权限

```nfs
/shared/folder 192.168.100.0/24(rw)
```

`showmount -e <ip> && mount <ip>:/shared/folder /local/mount/path`

NFS分享文件信息：`/var/lib/nfs/etab`
