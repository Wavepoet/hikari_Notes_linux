# prometheus+grafana+Alertmanager简化安装方式

> OS为Rockey 9.4 x86，时间2025.5.19，适合已了解配置或无脑安装的人。
> 

拉取压缩包

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
wget https://github.com/prometheus/alertmanager/releases/download/v0.28.1/alertmanager-0.28.1.linux-amd64.tar.gz
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-12.0.0-1.x86_64.rpm
```

解压安装包并移动

```bash
tar zvxf  prometheus-2.53.4.linux-amd64.tar.gz
mv prometheus-2.53.4.linux-amd64 /usr/local/prometheus
tar zvxf alertmanager-0.28.1.linux-amd64.tar.gz
mv alertmanager-0.28.1.linux-amd64.tar.gz /usr/local/alertmanager
```

安装rpm

```bash
yum install -y initscripts fontconfig
yum install -y  grafana-12.0.0-1.x86_64.rpm
```

配置系统为服务

---

```bash
vim /usr/lib/systemd/system/prometheus.service 
#粘贴以西内容
[Unit]
  Description=https://prometheus.io
  
  [Service]
  Restart=on-failure
  ExecStart=/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml --web.listen-address=:9090

  [Install]                      
  WantedBy=multi-user.target

```

---

```bash
vim /usr/lib/systemd/system/alertmanager.service
#粘贴以西内容
[Unit]
Description=https://prometheus.io

[Service]
Restart=on-failure
ExecStart=/usr/local/alertmanager/alertmanager --config.file=/usr/local/alertmanager/alertmanager.yml

[Install]
WantedBy=multi-user.target
```

配置服务

```bash
rm -rf  vim /usr/local/prometheus/prometheus.yml
```

```bash
vim /usr/local/prometheus/prometheus.yml

# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

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

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
      
  - job_name: 'linux'
    static_configs:
      - targets: ['192.168.223.130:9100','172.17.70.237:9100']

  - job_name: 'snmp'
    static_configs:
      - targets: ['192.168.224.130:9116','172.17.70.237:9116']  # snmpn服>务器IP.可以添加多个设备
    metrics_path: /snmp
    params:
      module: [if_mib]  # 使用上面定义的模块名称
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9116  # SNMP Exporter 地址
```

```bash
chmod +x /usr/local/prometheus/prometheus.yml
```

---

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

启动服务

```bash
systemctl daemon-reload
systemctl start prometheus
systemctl start grafana-server.service
systemctl start alertmanager.service
```

放行

```bash
firewall-cmd --add-port=9090/tcp --permanent
firewall-cmd --add-port=3000/tcp --permanent
firewall-cmd --add-port=9093/tcp --permanent
firewall-cmd --reload
```

访问[http://](http://172.17.70.237:9100/)[localhost](http://localhost:9090/)[:3000/](http://172.17.70.237:9100/)，访问[http://](http://172.17.70.237:9100/)[localhost](http://localhost:9090/)[:9090/](http://172.17.70.237:9100/)，访问[http://](http://172.17.70.237:9100/)[localhost](http://localhost:9090/)[:9093/](http://172.17.70.237:9100/)查看