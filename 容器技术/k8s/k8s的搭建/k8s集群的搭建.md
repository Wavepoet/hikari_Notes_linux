# k8s集群的搭建

---

> 此我使用3台设备，4G/2U，系统都为CentOS-Stream-9-x86_64，安装k8s版本为v1.28.2，因为k8s在1.28版本后更改了源位置，所以这份安装指北并不完全通用。能代理则代理，阿里源还是太垃圾了，很多问题都是网络上的问题。默认在全部设置上配置，if线会有点多。

---

## 环境准备

先要配置IP，并保证能ping通对方

设置主机名，三台设备分别为k8s-master，k8s-worker1，k8s-worker2。

```bash
#k8s_master
 hostnamectl set-hostname k8s-master
 bash
#k8snode1
 hostnamectl set-hostname k8s-worker1
 bash
#k8snode2
 hostnamectl set-hostname k8s-worker2
 bash
```

编写/ete/hosts文件，请书写自己设备的IP。

```bash
 cat >> /etc/hosts <<EOF
192.168.24.174 k8s-master
192.168.24.175 k8s-worker1
192.168.24.176 k8s-worker2
192.168.24.174 cluster-master
EOF
```

关闭SElinux和防火墙，添加网桥过滤和地址转发功能。

```bash
#临时关闭，永久关闭还需自行修改配置文件。
setenforce 0
systemctl stop firewalld 
# 允许 iptables 检查桥接流量(所有节点)
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
EOF
 modprobe br_netfilter
 sysctl --system
```

k8s在v1.28中已经支持swap调用但还是有些漏洞，由于Pod可能通过Swap"超售"内存，需要关闭swap。

```bash
# 临时关闭
swapoff -a
# 永久关闭（注释掉 swap 行）
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
```

时间同步

```bash
systemctl start chronyd
systemctl enable chronyd

crontab -e
#写入同步时间的频率和服务器
0 */1 * * * ntpdate timel.aliyun.com
#保存退出
date  
```

修改源，这里使用阿里源。

```bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

老版本需要升级内核，最新版本可以不用升级内核。

---

## 安装IPVS（可选）

后续需要调整Service 负载均衡，策略时可能会用到。

```bash
 yum install -y ipvsadm
```

---

## 容器安装（containerd）

在此安装的是containerd，也可以安装docker（以后可能会写）

k8s于1.24版本移除了dockershim组件，dockershim的替代品是ric-dockerd，因此现在使用docker作为容器运行时需要安装ric-docekrd。

yum安装

```bash
#安装前置工具
yum install -y yum-utils device-mapper-persistent-data lvm2
#添加仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#安装 containerd.io
yum update
yum install -y containerd.io cri-tools
```

生成默认配置文件

```bash
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

k8s会检测一遍环境，需要连网，如果连接不上服务器可以修改源。

```bash
vim /etc/containerd/config.toml
#末行模式使用/可查找
#修改sandbox_image为阿里源
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
#不使用阿里源也推荐把版本改成3.9
sandbox_image = "registry.k8s.io/pause:3.9"
#修改 Cgroup 驱动
SystemdCgroup = true
```

设置自启动

```bash
systemctl daemon-reload
systemctl enable --now containerd
systemctl restart containerd
```

 检查crctl

```bash
[root@k8s-master qyc]# crictl info
{
  "status": {
    "conditions": [
      {
      ……
#反正会输出很多
```

如果报错说未指引和containerd通讯请执行以下命令，如果没有可以跳过。

```bash
crictl --runtime-endpoint unix:///run/containerd/containerd.sock images
tee /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

检查`runc`。

```bash
runc
```

如果runc报错？那就自己找方法吧，反正我没报错，累了，懒得再装一次验证报错了，如果没有可以跳过。

## Docker安装参考，使用containerd**直接跳过即可**

如需要使用docker作为容器请参考，使用containerd则直接跳过即可。

```bash
# 下载 cri-docker
wget https://ghproxy.com/https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.5/cri-dockerd-0.2.5.amd64.tgz 

# 解压并安装
tar xvf cri-dockerd-0.2.5.amd64.tgz 
cp cri-dockerd/cri-dockerd /usr/bin/

# 配置 cri-docker 服务
cat > /usr/lib/systemd/system/cri-docker.service <<EOF 
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket

[Service]
Type=notify
ExecStart=/usr/bin/cri-dockerd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.7
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
EOF

# 配置 cri-docker socket
cat > /usr/lib/systemd/system/cri-docker.socket <<EOF 
[Unit]
Description=CRI Docker Socket for the API
PartOf=cri-docker.service

[Socket]
ListenStream=%t/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
EOF

# 启动 cri-docker
systemctl daemon-reload 
systemctl enable cri-docker --now

```

---

## kubelet，kubeadm，kubectl的安装

所有设备都需要安装kubelet，kubeadm，kubectl。

```bash
yum install -y kubelet kubeadm kubectl
systemctl enable  --now kubelet
```

查看版本信息

```bash
kubectl version
```

编写kubelet文件

```bash
 vim /etc/sysconfig/kubelet
#写入
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"
#重启 kubelet 服务
systemctl daemon-reload
systemctl restart kubelet
systemctl status kubelet
```

ping测试, 查看节点是否连通

```bash
 ping cluster-master
```

---

## master的安装

在mster中操作。

可以提前下载镜像，使用阿里源。

```bash
kubeadm config images list  --image-repository registry.aliyuncs.com/google_containers
kubeadm config images pull  --image-repository registry.aliyuncs.com/google_containers
```

master初始化，只在master中运行。注意apiserver IP为master IP。

```bash
#使用containerd请运行
kubeadm init \
--apiserver-advertise-address=192.168.24.174 \
--control-plane-endpoint=cluster-master \
--image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
--kubernetes-version v1.28.2 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.169.0.0/16 \
#使用docker请在命令中加入指定为docker
--cri-socket unix:///var/run/cri-dockerd.sock

kubeadm init \
--apiserver-advertise-address=192.168.24.174 \
--control-plane-endpoint=cluster-master \
--kubernetes-version v1.28.2 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.169.0.0/16 \
```

初始化报错后退出，可能初始化进程没退导致无法第二次初始使用，如果没有可以跳过。

```bash
kubeadm reset
```

初始化完成后执行官方提供的命令，类似于下面的命令，可以在初始化完成后的输出看到，以实际输出命令的为准。

```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

检查是否能使用

```bash
[root@k8s-master qyc]# kubectl  get nodes
NAME         STATUS     ROLES           AGE    VERSION
k8s-master   NotReady   control-plane   8m7s   v1.28.2

```

---

## worker的配置

在master初始化完成后也会提供worker加入的命令，类似于下面的命令，一般位于初始化输出完成后的最后一排。以实际输出命令的为准。在worker中使用命令即可。

```bash
kubeadm join cluster-master:6443 --token f0yj10.1s5p42ou4ugi1hu7 \
        --discovery-token-ca-cert-hash sha256:91b82a6310bcfcc54d9283132af9eb52f86eba9f53b7c0d70c3b75323160b6ff

```

如果加入失败请使用命令重置修改错误再尝试加入，如果没有可以跳过。

```bash
sudo kubeadm reset
```

在master查看是否已经加入。可以看到worker在列表当。

```bash
[root@k8s-master qyc]# kubectl get nodes
NAME          STATUS   ROLES           AGE   VERSION
k8s-master    Ready    control-plane   26h   v1.28.2
k8s-worker1   Ready    <none>          25h   v1.28.2
k8s-worker2   Ready    <none>          23h   v1.28.2
```

---

## 网络插件calico的安装

初始化完成后还无法启动pod，需要部署一个网络插件。在这里部署的是calico。在master上操作。

其他可以更具版本可以参考官网

[Community-tested Kubernetes versions | Calico Documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/community-tested)

安装 Tigera Operator 和自定义资源定义。

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/operator-crds.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/tigera-operator.yaml
```

拉取Calico 所需的自定义资源。

```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/custom-resources.yaml -O
```

查看ConfigMap 。

```bash
kubectl get cm -n kube-system kubeadm-config -o yaml | grep podSubnet
```

修改custom-resources，根据自己需求修改

```bash
vim custom-resources.yaml
……
spec:
#指定使用阿里源
  registry: registry.aliyuncs.com/calico
  calicoNetwork:
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26
      #cidr修改成与ConfigMap一样的IP
      cidr: 192.169.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
……
```

安装装

```bash
kubectl create -f custom-resources.yaml
```

有错误可以使用以下命令删除

```bash
kubectl delete -f custom-resources.yaml
kubectl delete -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/operator-crds.yaml
kubectl delete -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/tigera-operator.yaml
```

验证验证（显示拉去错误可以等会，下载会比较慢）

```bash

[root@k8s-master qyc]#watch kubectl get pods -n calico-system
NAME                                       READY   STATUS              RESTARTS      AGE
calico-kube-controllers-577bccc84b-hvd8b   1/1     Running             0             23h
calico-node-dsbjn                          1/1     Running             0             23h
calico-node-gtz7l                          1/1     Running             0             23h
……
```

## 参考

[【k8s1.28版】基于Containerd容器部署K8S 1.28版本集群，全程干货无废话（k8s部署/k8s安装/k8s实战/k8s教程）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1cu4y1R78u/?spm_id_from=333.1007.top_right_bar_window_default_collection.content.click&vd_source=6a688cd548533c0b00d38cf95f81acbf)

[Containerd容器运行时安装与配置指南 - Leo-Yide - 博客园](https://www.cnblogs.com/leojazz/p/18824110)

[超详细，简单的Containerd的安装步骤_containerd安装-CSDN博客](https://blog.csdn.net/luck099/article/details/131233362)

[从零开始：Kubernetes 集群的搭建与配置指南，超详细，保姆级教程_kubernetes菜鸟教程-CSDN博客](https://blog.csdn.net/Lentr0py/article/details/141127252)

[juejin.cn](https://juejin.cn/post/7291570449334812735)

[containerd快速安装指南🚀-阿里云开发者社区](https://developer.aliyun.com/article/1471191)

[轻量级容器管理工具Containerd的两种安装方式 - 只为心情愉悦 - 博客园](https://www.cnblogs.com/liuzhonghua1/p/18010847)

使用AI：

Genimin 2.5 Pro

ChatGPT-5
