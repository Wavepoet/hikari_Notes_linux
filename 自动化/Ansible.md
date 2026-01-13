# Ansible

Ansible 是一个开源的自动化运维工具，主要用于配置管理、应用部署、任务执行和编排。

现代数据中心中的

```bash
yum -y install epel-release
yum -y install ansible
```

关闭ssh指纹

```bash
vim /etc/ansible/ansible.cfg


[defaults]
host_key_checking = False
```

```bash
vim /etc/ansible/hosts
<ip> ansible_sshcommonargs='-oStrictHostkeychecking=no'
```

输入密码

```bash
ansible all -m ping -k
```

```bash
vim /etc/ansible/hosts
[k8s]
192.168.24.167 ansible_password=qyc261810
192.168.24.166 ansible_password=qyc261810
```
