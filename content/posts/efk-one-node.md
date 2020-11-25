---
title: "统一日志平台：EFK单节点部署"
date: 2020-05-22T09:56:21+08:00
draft: false
categories: ["部署"]
tags: ["elasticsearch", "filebeat","kibana", "日志"]
---

## EFK单节点部署
本层版本选择7.7.
## 1. 安装es

### 1.1 下载es
https://www.elastic.co/cn/downloads/elasticsearch
推荐rpm
### 1.2 上传安装包安装es
```
rpm -ivh elasticsearch-7.7.0-x86_64.rpm
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
```
以下可选  
`vim /etc/sysctl.conf`  
添加或修改 
`vm.max_map_count=262144`
`sudo sysctl -p`

防止 es 启动时出现如下错误： max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
vim /etc/security/limits.conf
* soft memlock 100000
* hard memlock 100000
* soft nproc 100000
* hard nproc 100000
```
以上

### 1.3 配置es
```
#按需配置jvm.options, 默认-Xms1g -Xmx1g  
#推荐：如果足够的内存，也尽量不要 超过 32 GB。即每个节点内存分配不超过 32 GB。 因为它浪费了内存，降低了 CPU 的性能，还要让 GC 应对大内存。如果你想保证其安全可靠，设置堆内存为 31 GB 是一个安全的选择。

vim /etc/elasticsearch/jvm.options

vim /etc/elasticsearch/elasticsearch.yml
#修改如下基础配置
network.host: 10.119.31.209
discovery.seed_hosts: ["10.119.31.209"]

```

### 配置 TLS 和身份验证（推荐）
rpm安装es默认home路径：/usr/share/elasticsearch
```
#在 Elasticsearch 主节点上配置 TLS
/usr/share/elasticsearch/bin/elasticsearch-certutil cert -out /etc/elasticsearch/elastic-certificates.p12 -pass ""

chmod 640 /etc/elasticsearch/elastic-certificates.p12

vim /etc/elasticsearch/elasticsearch.yml
#添加安全配置
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12

#启动es  
systemctl start elasticsearch

#自动设置elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user的密码，记得保存，也可以把auto改成interactive手动设置密码
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```
浏览器打开http://10.119.31.209:9200/

## 2. 安装kibana
```
rpm -ivh kibana-7.7.0-x86_64.rpm
vim  
#修改如下配置
server.host: "10.119.31.209"
elasticsearch.hosts: ["http://10.119.31.209:9200"]
elasticsearch.username: "kibana"
elasticsearch.password: "password"
i18n.locale: "zh-CN"

#启动kibana
systemctl start kibana.service
```

## 3. 安装filebeat
filebeat默认支持几十种采集模块，如nginx, kafka, mysql等，可按需配置或完全自定义配置路径。
### 3.1 nginx日志收集
```
#nginx节点安装filebeat
rpm -ivh filebeat-7.7.0-x86_64.rpm
vim 
output.elasticsearch:
  hosts: ["<es_url>"]
  username: "elastic"
  password: "<password>"
setup.kibana:
  host: "<kibana_url>"
```
```
#启用nginx模块
sudo filebeat modules enable nginx
#修改采集路径
vim /etc/filebeat/modules.d/nginx.yml
- module: nginx
  # Access logs
  access:
    enabled: true
    var.paths: ["/app/openresty/nginx/logs/access*.log"]
```
```
#setup 命令加载 Kibana 仪表板。如果仪表板已设置，请省略此命令。
sudo filebeat setup
#启动filebeat
sudo systemctl start filebeat
```

### 3.2 配置索引生命周期管理
http://xxx/app/kibana#/management/elasticsearch/index_lifecycle_management/policies/edit/filebeat

按需配置保留的日志索引大小。

### 3.3 原日志文件清理
如接入EFK日志平台后，不想保留原始日志文件，最方便的方案如下：
1. 不分割日志，如nginx按天分割；
2. 编写个脚本把日志文件内容清空，设定每天或每周执行一次。
```
#!/bin/bash
# nginx-logs-flush.sh
logdir=/app/openresty/nginx/logs
echo "" > $logdir/error.log
echo "" > $logdir/access.log
```
定时任务，如下为每周日凌晨两点执行
```
crontab -l
0 2 * * 0 /app/openresty/nginx/logs/nginx-logs-flush.sh
```

---
参考官方文档：  
https://www.elastic.co/cn/blog/getting-started-with-elasticsearch-security  
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html  