# bash-completion

> 你还在为命令太长，子参数无法补全而苦恼吗？你还在为命令无法补全而打错而苦恼吗？现在我们隆重推出bash-completion~😎。

bash-completion是一个Linux上的自动补全工具，对bash补全功能的一个增强。

## 安装

一般源上都会有bash-completion，可以直接安装。

```bash
# CentOS / RHEL
yum install -y bash-completion
# Ubuntu / Debian
apt-get install -y bash-completion  
```

因为我们正在使用shell，使用shell在缓存里，所以下载完成后不会立即生效。

执行一下命令立即生效：

```bash
source /etc/profile.d/bash_completion.sh
```

接下来就可以使用bash-completion快速补全了，

## 使用bash-completion补全其他软件

系统自带的一些常用软件bash-completion都支持补全，但未集成有其他第三方软件。这时候便需要编辑补全脚本了。

例如k8s

```bash
#运行命令生成器
echo "source <(kubectl completion bash)" >> ~/.bashrc
#立即执行
source ~/.bashrc
```
