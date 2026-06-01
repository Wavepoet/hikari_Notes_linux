# sudo 无密码

需要使用 root 用户

使用 `visudo` 命令编辑 sudoers 文件。

```bash
[qyc@localhost ~]$ sudo visudo
```

在最后一行添加

```bash
<用户名> ALL= (ALL)NOPASSWD:ALL
```
