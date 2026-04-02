# zabbix安装(只看这个就行)

# 2025.5.15版本

## 服务版本

本实验基于Rocky Linux 9.2 最小化安装

zabbix 7.2

php 8

## 软件环境

- zabbix软件包
`zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent`
- 数据库
`mariadb-server`
- php
`php php-fpm php-mysqlnd php-gd php-bcmath php-mbstring php-xml php-ldap php-pecl-zip`
- 文字支持
`glibc-common langpacks-zh_CN glibc-all-langpacks`

## 可以预先准备的文件

```bash
curl http://ftp.gnu.org/gnu/glibc/glibc-2.35.tar.gz -o glibc-2.35.tar.gz
curl --proxy socks5://172.17.70.138:7897 (使用clash代理)(内含字符映射文件和中文区域定义文件)
```

## 服务器环境

- Rockylinux保留官方软件源文件

## 防火墙配置

- Zabbix Server

```bash
firewall-cmd --permanent --add-port=10050/tcp   # Agent被动检测firewall-cmd --permanent --add-port=10051/tcp   # Server端接收数据firewall-cmd --permanent --add-service=http     #Apache Webfirewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=ssh      #添加sshfirewall-cmd --permanent --add-port=22/tcp
firewall-cmd --permanent --add-service=mysql    #数据库连接（按实际类型选择）firewall-cmd --reload   #重新加载配置
```

## SELinux配置

```bash
setsebool -P httpd_can_network_connect 1   #允许Apache网络连接setsebool -P httpd_can_network_connect_db 1    #允许Apache连接数据库setsebool -P httpd_execmem 1   #允许PHP执行内存操作（PHP-FPM需要）setsebool -P zabbix_can_network 1  #允许Zabbix Server通信
```

## 安装步骤

**1. 配置zabbix的官方源**

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/7.2/release/rocky/9/noarch/zabbix-release-latest-7.2.el9.noarch.rpm
```

- 在此笔记写下时，官方源无需代理，可以直连，速度还行。
- 在此笔记写下时，笔者曾使用阿里源，但安装zabbix-server-mysql软件包时，没有
`/usr/share/zabbix/`路径下的`sql-scripts/mysql/server.sql.gz`，原因尚未查明。
完成后，应当在/etc/yum.repos.d/下存在多个zabbix配置。

```bash
dnf -y install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```

**2. 配置Rockylinux的网络镜像源或者本地软件仓库，需要保留系统自带的rocky官方源配置文件。**

```bash
# 将文件中链接替换sed -e 's|^mirrorlist=|#mirrorlist=|g' \    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g' \    -i.bak \    /etc/yum.repos.d/rocky-*.repo
```

```bash
dnf makecache && dnf clean all
```

**3. 配置数据库**

安装MariaDB数据库

```bash
dnf -y install mariadb-server
systemctl enable --now mariadb
```

```bash
mysql -u root -p# 密码随意mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;# 创建一个名为zabbix的数据库，用于存储监控数据；# character set utf8mb4使用utf8mb4字符集；# collate utf8mb4_bin指定二进制排序规则，严格区分大小写和重音符号mysql> create user zabbix@localhost identified by '123456';# 创建一个仅允许本地登录的MySQL/MariaDB用户，密码为123456mysql> grant all privileges on zabbix.* to zabbix@localhost;# 授予用户zabbix对zabbix数据库的完全控制权限mysql> set global log_bin_trust_function_creators = 1;# 允许MySQL/MariaDB信任存储函数和触发器的创建者mysql> quit;
# 退出就是退出
```

**4. 导入数据库初始模式和数据。会提示输入密码。**

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

**5. 导入完成后禁用选项**

```bash
mysql -u root -pset global log_bin_trust_function_creators = 0;quit;
```

**6. 编辑配置文件，如果你用了非默认用户名和密码，需要更改。**

```bash
vim /etc/zabbix/zabbix_server.conf
100 DBName=zabbix         #存储监控数据的库名116 DBUser=zabbix             #连接数据库的用户124 DBPassword=123456     #设置zabbix用户密码
```

**7. 安装PHP(你明明可以在一开始就全下好的baka)**

```bash
dnf install php php-fpm php-mysqlnd php-gd php-bcmath php-mbstring php-xml php-ldap php-pecl-zip
```

**8. 启动服务**

```bash
systemctl restart zabbix-server zabbix-agent httpd php-fpm
systemctl enable zabbix-server zabbix-agent httpd php-fpm
```

**10. 配置语言**

```bash
dnf install langpacks-zh_CN glibc-all-langpacks glibc-common
```

```bash
curl http://ftp.gnu.org/gnu/glibc/glibc-2.35.tar.gz -o glibc-2.35.tar.gz
# 你也可以在一开始准备好。tar -xzvf glibc-2.35.tar.gz
# 解压，在当前目录的glibc-2.35文件夹里cp glibc-2.35/localedata/locales/zh_CN /usr/share/i18n/locales/
# 复制中文区域定义文件cp glibc-2.35/localedata/charmaps/UTF-8 /usr/share/i18n/charmaps/
# 复制字符映射文件
```

**11. 登录zabbix web**

http://服务器ip/zabbix

**12. 在安装完成后，出现sshd无法启动的情况，这是因为zabbix顺带升级了OpenSSL，而旧版本OpenSSH与其版本不匹配。**

```bash
dnf update openssh -y
```

**13. secure CRT文件互传**

secure crt快捷键Alt P,连接到sftp

- pwd: 查询linux主机所在目录
- lpwd: 查询本地目录
- ls: 查询linux主机所在目录文件
- lls: 查询本地上传目录文件
- lcd: 改变本地上传目录的路径
- cd: 改变远程上传目录
- get: 将远程目录中文件下载到本地目录
- put: 将本地目录中文件上传到远程主机（linux）
- quit: 断开FTP连接