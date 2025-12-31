# MySQL

## MySQL是什么

MySQL 是目前世界上最流行的开源关系型数据库管理系统（RDBMS）。

MySQL1995年由瑞典公司MySQL AB开发，2008年被Sun收购，2010年随Sun被Oracle（甲骨文）收购。MySQL被Oracle收购后，社区因为担心MySQL闭源等原因,2009年由MySQL创始人Michael Widenius主导了MariaDB项目，MariaDB是MySQL的分支版本，完全兼容MySQL，但社区支持更加活跃。

## 安装

这里使用的是Linux发行版为Rocky Linux 9.7。付上MySQL和MariaDB的两个的安装方式。

### 安装MySQL

```bash
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum update
yum install mysql-server
```

启动MySQL

```bash
#设置权限
chown -R mysql:mysql /var/lib/mysql/
#始化
mysqld --initialize
#启动
systemctl start mysqld
systemctl enable mysqld
systemctl status mysqld
```

### 安装MariaDB

如果安装了MySQL，则需要卸载MySQL，然后安装MariaDB。

```bash
yum remove -y  mysql-server
rm -rf /etc/yum.repos.d/mysql*
```

安装MariaDB

```bash
yum install -y mariadb-server
```

启动MariaDB

```bash
systemctl start mariadb
systemctl enable mariadb
systemctl status mariadb
```

### 连接数据库

我装的是mariaDB，所以下面我用命令就是mariadb命令了。如果是MySQL，把mariadb换成mysql就行。

进入命令行：

```bash
# MariaDB
mariadb
```

输入密码后，进入MySQL命令行。

默认root密码为空，可以修改密码，注意：不是在mysql/mariadb命令行中修改。

```bash
mariadb -u root -p 
```

连接远程数据库：

```bash
mariadb -h <主机名或IP地址> -P <端口号> -u <用户名> -p
```

使用自动补全功能（这只是临时，退出会话后失效）：

```bash
mariadb -u root -p --auto-rehash
```

### 永久开启补全

```bash
vim /etc/my.cnf
#添加如下模块
[mysql]
auto-rehash
```

## MyCil

mycli 是一个给 MySQL / MariaDB 用的增强型命令行客户端
相比自带的 mysql，mycli 提供了以下额外功能：

- ✅ 自动补全（库名 / 表名 / 字段名 / 关键字）

- 🎨 语法高亮

- 🧠 历史记录（↑↓可查）

- 🧪 结果表格美化

- ⚡ 对 MySQL / MariaDB 完全兼容

### MyCli安装

Linux可以先尝试使用包管理器安装，我使用Rockey Linux安装时显示包冲突。

```bash
# Ubuntu/Debian
sudo apt install bash-completion

# CentOS/RHEL
sudo yum install bash-completion

#或者使用dnf安装
dnf install epel-release -y
dnf install mycli -y
```

也可以使用pip安装：（windows和linux通用）

```bash
pip install mycli
```

使用uv或者pipx安装:

```bash
#uv安装
uv install mycli
#pipx安装
pipx install mycli
```

### MyCli使用

## MySQL使用

```bash
# 显示所有数据库
show databases;
```
