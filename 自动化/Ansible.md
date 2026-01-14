# Ansible

现代数据中心中的服务器数量众多。在部署和运维时，不可能每个设备都使用SSH连接进行手动配置。为了更好地管理这些服务器，需要一种自动化的工具来对这些设备进行批量的配置。Ansible便运用于此。Ansible 是一个开源的自动化运维工具，主要用于配置管理、应用部署、任务执行和编排。

- 无代理，使用SSH管理设备
- 模块化设计，支持API自定义模块
- 使用Playbook编排任务（YAML格式）
- 幂等性

Ansible 的工作原理其实十分简单，就是使用SSH登录到设备上，然后在设备上编写执行python脚本。

Ansible有两种使用方式：

- Ad-Hoc：命令行方式的执行命令，一次执行一个任务

- Playbook：通过编写Playbook文件，批量执行多个任务。（类似于Docker Compose，shell脚本）

---

## 安装

```bash
#centos/redhat
yum -y install epel-release
yum -y install ansible
```

---

## 配置文件

使用``ansible --version``，可以看到ansible的配置文件在``/etc/ansible/ansible.cfg``。

```bash
[root@localhost ~]# ansible  --version
ansible [core 2.15.13]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.9/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.9.21 (main, Feb 10 2025, 00:00:00) [GCC 11.5.0 20240719 (Red Hat 11.5.0-5)] (/usr/bin/python3)
  jinja version = 3.1.6
  libyaml = True
```

查看配置文件，可以发现配置都被注释掉或者就没有配置项。ansible会使用默认配置。如果想要修改ansinle的配置，需要自行添加配置项。

```bash
[root@localhost ~]# cat /etc/ansible/ansible.cfg
# Since Ansible 2.12 (core):
# To generate an example config file (a "disabled" one with all default settings, commented out):
#               $ ansible-config init --disabled > ansible.cfg
#
# Also you can now have a more complete file by including existing plugins:
# ansible-config init --disabled -t all > ansible.cfg

# For previous versions of Ansible you can check for examples in the 'stable' branches of each version
# Note that this file was always incomplete  and lagging changes to configuration settings

# for example, for 2.9: https://github.com/ansible/ansible/blob/stable-2.9/examples/ansible.cfg
```

---

## hosts文件

在Ansible中也有像Linux中用户组类似的``主机组``这一概念。主机组代表一组主机。可以使用主机组对一组设备进行批量的操作。

同时Ansible中还一``父组``和``子组``这一概念，即一个主机组可以被另一个主机组包含。被包含的这个组就是子组，包含的组便是父组。（嵌套关系）

Ansible的配置文件中有一个hosts文件，用于定义Ansible管理的主机。默认的路径为``/etc/ansible/hosts``。

在hosts文件中:

- 主机组用``[]``定义
- 主机名可以使用``IP``或``主机名``定义。
- 没有主机组的主机需要写在文件的最前边，以避免被其他主机组覆盖。

hosts文件示例：

```bash
#没有主机组的主机写在文件的最前边
192.168.24.1
192.168.24.2
#组的名称，这里是k8s_server
[k8s_server]
#主机组中的主机
192.168.24.167
192.168.24.166
[web_server]
192.168.24.201
192.168.24.202
```

使用``ansible-inventory --graph``可以查看主机组的关系图。

```bash
[root@localhost ~]#  ansible-inventory --graph
@all:
  |--@ungrouped:
  |  |--192.168.24.1
  |  |--192.168.24.2
  |--@k8s_server:
  |  |--192.168.24.167
  |  |--192.168.24.166
  |--@web_server:
  |  |--192.168.24.201
  |  |--192.168.24.202
```

可以看见除了我们配置的主机组外，还有``@ungrouped``和``@all``这两个默认的主机组。``all``组包含了所有主机，``ungrouped``组包含了没有归属到任何主机组的主机。

---

## Ansible Ad-Hoc的使用

Ad-Hoc的命令格式如下：

```bash
ansible [pattern] -m [module] -a "[module options]"
# pattern：主机模式（对谁操作？）。可以是组机组，IP，ALL……
# module：模块名。比如：command、shell、yum、apt等。
# module options：模块参数。比如：yum的-y参数表示自动回答yes。
```

可以使用``ansible-doc -s <模块>``来查看可以使用的模块参数，或者使用``ansible-doc  <模块>``查看详细的模块说明。

```bash
[root@localhost ~]# ansible k8s_server -m yum -a "name=vim state=latest"
192.168.24.166 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: root@192.168.24.166: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).",
    "unreachable": true
}
192.168.24.167 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: root@192.168.24.167: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).",
    "unreachable": true
}
```

---

### 加入Key

可以发现Ansible无法连接到主机，因为没有配置Key。在此提供2种方法解决该问题。

1. 在命令后加入``-k``参数，Ansible会要求输入密码。

```bash
[root@localhost ~]# ansible k8s_server -m yum -a "name=vim state=latest" -k
SSH password:
```

2. 可以在hosts文件主机后直接配置key。格式为``ansible_password=<key>``。

```bash
vim /etc/ansible/hosts

[k8s_server]
192.168.24.167 ansible_password=qyc261810
192.168.24.166 ansible_password=qyc261810
```

再次尝试连接

```bash
[root@localhost ~]# ansible k8s_server -m yum -a "name=vim state=latest"
192.168.24.167 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
192.168.24.166 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Installed: vim-common-2:8.2.2637-23.el9_7.x86_64",
        "Installed: vim-enhanced-2:8.2.2637-23.el9_7.x86_64",
        "Removed: vim-common-2:8.2.2637-22.el9_6.x86_64",
        "Removed: vim-enhanced-2:8.2.2637-22.el9_6.x86_64"
    ]
}
```

### 配置ssh指纹

```bash
vim /etc/ansible/ansible.cfg

[defaults]
host_key_checking = False
```

```bash
vim /etc/ansible/hosts
<ip> ansible_sshcommonargs='-oStrictHostkeychecking=no'
```
