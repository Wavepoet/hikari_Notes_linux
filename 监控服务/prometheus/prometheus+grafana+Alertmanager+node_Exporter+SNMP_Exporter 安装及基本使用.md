# prometheus+grafana+Alertmanager+**node_Exporter+**SNMP_Exporter 安装及基本使用

大致的每个程序的作用

![8c3b43af122cc001009acb18a0b987e7.png](attachment:7149db5f-77ee-4c20-a399-c3bc912adb0e:8c3b43af122cc001009acb18a0b987e7.png)

> 普罗米修斯监控并不难装，但官方仓库和GitHub仓库都在国外，需要科学上网，麻烦，我官网下的包scp上服务器，解压会报错…...，在此用到的OS版本为Rockey Linux 9.4  x86。此教程较为繁琐，但有利于了解安装步骤，方便今后安装及使用
> 

github：https://github.com/prometheus/

官网：https://prometheus.io/download/

grafana：[https://grafana.com/](https://grafana.com/grafana/download/12.0.0)

/直接拉取

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
wget https://github.com/prometheus/alertmanager/releases/download/v0.28.1/alertmanager-0.28.1.linux-amd64.tar.gz
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-12.0.0-1.x86_64.rpm
#按需求选择
wget https://github.com/prometheus/snmp_exporter/releases/download/v0.29.0/snmp_exporter-0.29.0.linux-amd64.tar.gz
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
```

基本的搭建需要拉取以下安装包

```bash
[root@localhost prometheus]# ll
总用量 316376
-rw-r--r--. 1 root root  33436897  3月  7 23:09 alertmanager-0.28.1.linux-amd64.tar.gz
-rw-r--r--. 1 root root 174753651  5月  6 01:41 grafana-12.0.0-1.x86_64.rpm
-rw-r--r--. 1 root root 104200835  3月 18 23:12 prometheus-2.53.4.linux-amd64.tar.gz
```

---

## 安装普罗米修斯

```bash
#解压文件 
 tar zvxf  prometheus-2.53.4.linux-amd64.tar.gz
 mv prometheus-2.53.4.linux-amd64 /usr/local/prometheus
```

### 配置系统服务

```bash
cd /usr/lib/systemd/system
vim prometheus.service 
#粘贴以西内容
[Unit]
  Description=https://prometheus.io
  
  [Service]
  Restart=on-failure
  ExecStart=/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml --web.listen-address=:9090

  [Install]                      
  WantedBy=multi-user.target
```

生效文件，启动服务

```bash
systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus
```

### 配置解析

prometheus的配置文件在/usr/local/prometheus/prometheus.yml

需要注意yaml文件为严格缩进，不能使用tab键补全部，注意”：“后面要加空格。

末行键入`:set listc`查看隐藏符号

```bash
#prometheus的全局配置
# my global config
global:
  scrape_interval: 15s # 拉取的时间间隔
  evaluation_interval: 15s # 告警对等时间间隔
  # scrape_timeout is set to the global default (10s).
  
#prometheus的Alertmanager报警模块配置
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.

rule_files:  #规则文件位置
  # - "first_rules.yml"
  # - "second_rules.yml"
  
#prometheus的监控配置
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus" #定义监控任务名为prometheus

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"] #从哪获取数据
```

prometheus默认使用9090端口

防火墙放行端口（貌似不放行也可以访问）

```bash
setenforce 0
firewall-cmd --add-port=9090/tcp --permanent
firewall-cmd --reload
```

访问[http://localhost:9090](http://localhost:9090/)查看

---

## **Grafana 安装**

回到wget拉取文件的目录

```bash
yum install -y initscripts fontconfig
yum install -y  grafana-12.0.0-1.x86_64.rpm
systemctl start grafana-server.service
systemctl enable grafana-server.service
systemctl status grafana-server.service

firewall-cmd --add-port=3000/tcp --permanent
firewall-cmd --reload
```

访问[http://](http://172.17.70.237:9100/)[localhost](http://localhost:9090/)[:3000/](http://172.17.70.237:9100/)查看,初始用户，密码为admin，admin，登录后修改密码。

### 汉化

登录后点用户（就这头像）—-》点prodfil

![image.png](attachment:26f8005b-076c-4241-be26-cbc2f1e75e85:image.png)

找到Language就可以改了，记得保存

![image.png](attachment:5eaadf96-8751-4ebf-83ec-67d76b906318:image.png)

官方的汉化不全，但也基本够用了.（官网的汉化也是半中文，半英文，就挺抽象）

### 配置

搜索**Add new connection或**prometheus

![image.png](attachment:011891c0-5d44-470a-a77c-5b9774ed3c60:image.png)

找到prometheus（可搜索），添加连接

![image.png](attachment:dc10c148-c895-49e7-bf5c-b12188cae042:image.png)

回到首页点+ —→new dashboard，当然有表盘可以自己用本地自己的表盘。

![image.png](attachment:131f682f-63ea-456c-843e-a694c53e537a:image.png)

选择prometheus，

![image.png](attachment:232583b5-44f2-4cb9-94a5-3a8782f4beca:image.png)

接下来就开始你的设计了！！！

找到Metric选择你需要的指标

![image.png](attachment:91a0e33b-f4d0-4821-9df5-4a865aab7393:image.png)

找到Kick start yuer query选择查询方式

![image.png](attachment:c9370538-3eea-4125-a031-f0efd0666262:image.png)

点Run qyeries运行查询

![image.png](attachment:585c9721-0915-45df-a46c-a220e94f17c4:image.png)

其他方面就不多赘述了，慢慢摸索，点save dashboard保存

![image.png](attachment:fbd89912-2939-46d6-a275-f3e53b17a021:image.png)

---

## 安装Alertmanager

```bash
tar zvxf alertmanager-0.28.1.linux-amd64.tar.gz
mv alertmanager-0.28.1.linux-amd64 /usr/local/alertmanager
```

### 配置系统服务

```bash
cd /usr/lib/systemd/system
vim alertmanager.service
#粘贴以西内容
[Unit]
Description=https://prometheus.io

[Service]
Restart=on-failure
ExecStart=/usr/local/alertmanager/alertmanager --config.file=/usr/local/alertmanager/alertmanager.yml

[Install]
WantedBy=multi-user.target
```

### 修改prometheus文件

添加- localhost:9093和 - "alert.yml"

```bash
vim /usr/local/prometheus/prometheus.yml
#粘贴以西内容
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093
          - localhost:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "alert.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"
```

新建alert.yml文件

```bash
vim /usr/local/prometheus/alert.yml
#粘贴以西内容
  groups:
- name: Prometheus alert
  rules:
#对任何实例超过30s无法联系的情况发出警报
  - alert: 服务告警
    expr: up == 0
    for: 30s
    labels:
      severity: critica1
    annotations:
      instance: "{{ $labels.instance }}"
      description: "{{ $labels.job }}服务已关闭"
```

加权限

```jsx
chmod +x /usr/local/prometheus/alert.yml
```

### 启动

```bash
systemctl daemon-reload
systemctl stop prometheus
systemctl start prometheus
systemctl start alertmanager.service
systemctl enable alertmanager.service
```

放行端口

```bash
firewall-cmd --add-port=9093/tcp --permanent
firewall-cmd --reload
```

访问[http://](http://172.17.70.237:9100/)[localhost](http://localhost:9090/)[:9093/](http://172.17.70.237:9100/)查看

---

## **安装node_exporter**

```bash
#解压文件
tar zvxf node_exporter-1.9.0.linux-amd64.tar.gz 
mv node_exporter-1.9.0.linux-amd64 /usr/local/node_exporter
```

### 添加系统服务

```bash
vim /usr/lib/systemd/system/node_exporter.service
#粘贴以西内容
[Unit]
Description=node_exporter
After=network.target 

[Service]
ExecStart=/usr/local/node_exporter/node_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target 
```

详细的配置

```jsx
cat > /etc/systemd/system/node_exporter.service << 'EOF'
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/local/node_exporter/node_exporter \
    --web.listen-address=:9100 \
    --path.rootfs=/ \
    --collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)
Restart=always
RestartSec=5
StartLimitInterval=0

[Install]
WantedBy=multi-user.target
EOF
```

### 添加监控

```bash
vim /usr/local/prometheus/prometheus.yml
#粘贴以西内容
  - job_name: 'linux'
    static_configs:
    - targets: ['192.168.223.130:9100','172.17.70.237:9100']
    #node_exporter服务器的IP，可添加多个
#再次说明yaml文件不可以使用tab补全部
```

### 启动

```bash
systemctl daemon-reload
systemctl start node_exporter
systemctl start prometheus
```

放行端口

```bash
firewall-cmd --add-port=9100/tcp --permanent
firewall-cmd --reload
```

访问[http://](http://172.17.70.237:9100/)[localhost](http://localhost:9090/)[:9100/](http://172.17.70.237:9100/)查看

访问[http://](http://172.17.70.237:9090/targets?search=)[localhost](http://localhost:9090/)[:9090/targets?search=](http://172.17.70.237:9090/targets?search=)可以看到出现node_exporter服务器

web端点击status—>Targets查看

![image.png](attachment:15526884-5bb3-4c00-8938-48d4bf8df95b:image.png)

---

## 安装SNMP_Exporter

```bash
#解压文件
tar zxvf snmp_exporter-0.29.0.linux-amd64.tar.gz  -C /usr/local/
cd  /usr/local/
mv snmp_exporter-0.29.0.linux-amd64 snmp_exporter
```

### 配置系统服务

```bash
vim  /etc/systemd/system/snmp_exporter.service
#粘贴以西内容
[Unit]
Description=node_exporter
After=network.target

[Service]
ExecStart=/usr/local/snmp_exporter/snmp_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### 配置监控

```bash
vim /usr/local/prometheus/prometheus.yml
#粘贴以西内容
  - job_name: 'snmp'
    static_configs:
      - targets: ['192.168.224.130:9116','172.17.70.237:9116']  # snmpn设备IP，可以添加多个设备
    metrics_path: /snmp
    params:
      module: [if_mib]  # 使用上面定义的模块名称
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9116  # 重定向到的SNMP Exporter 地址

```

### 重启服务

```bash
systemctl daemon-reload
systemctl start snmp_exporter.service
systemctl stop prometheus.service
systemctl start prometheus
systemctl enable snmp_exporter
```

### 放行端口

```bash

firewall-cmd --add-port=9116/tcp --permanent
firewall-cmd --reload
```

访问访问[http://](http://172.17.70.237:9100/)[localhost](http://localhost:9090/)[:9116/](http://172.17.70.237:9100/)查看

访问访问[http://](http://172.17.70.237:9100/)[localhost](http://localhost:9090/)[:9090/targets?search=/](http://172.17.70.237:9100/)查看

---

## 参考

https://developer.aliyun.com/article/1379529

https://blog.csdn.net/qq_48721706/article/details/135943577

https://blog.csdn.net/qq_48721706/article/details/135943577

https://blog.csdn.net/weixin_41004518/article/details/141963162

创建时间：

2025.5.18     春风