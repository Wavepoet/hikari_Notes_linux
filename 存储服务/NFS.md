# NFS

NFS（Network File System）是一种允许不同系统之间共享文件的协议。

系统为`` Fedora Linux 43 ``

安装并启动服务

```bash
yum -y install rpcbind nfs-utils
systemctl enable --now nfs-server
```

创建共享目录并设置权限

```bash
mkdir -p /srv/nfs
chmod 777 /srv/nfs
```

编辑`` /etc/exports``文件，如果没有文件则创建文件，添加以下内容:

```txet
/srv/nfs *(rw,sync,no_root_squash,no_subtree_check)

MOUNTD_PORT=892          # mountd 服务端口，常见设置为892[reference:8]
LOCKD_TCPPORT=32803      # TCP 锁服务端口[reference:9]
LOCKD_UDPPORT=32769      # UDP 锁服务端口[reference:10]
STATD_PORT=662           # statd 服务端口[reference:11]
RQUOTAD_PORT=875         # quotad 服务端口[reference:12]
```

导出共享

```bash
sudo exportfs -rav
```

放行端口

```bash
sudo iptables -A INPUT -p tcp --dport 111 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 111 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 2049 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 2049 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 892 -j ACCEPT    # mountd
sudo iptables -A INPUT -p tcp --dport 32803 -j ACCEPT  # lockd
sudo iptables -A INPUT -p udp --dport 32769 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 662 -j ACCEPT    # statd
sudo iptables -A INPUT -p tcp --dport 875 -j ACCEPT    # quotad

sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

启动服务

```bash
 systemctl start rpcbind
sudo systemctl start nfs-server
systemctl enable rpcbind
systemctl enable nfs-server
```

尝试挂载

```bash
 sudo mount -t nfs 127.0.0.1:/srv/nfs <本地的挂载目录> -o nolock,vers=3
```

验证挂载

```bash
mount | grep nfs
df -h <本地的挂载目录>
```

取消挂载

```bash
umount <本地的挂载目录>
```
