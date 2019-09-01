1、修改prometheus配置文件  
```
# cat prometheus.yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "rules/node_rules.yml"        #此处添加记录规则文件
```  

2、创建文件  
```
进入prometheus工作目录
# cd prometheus
# mkdir -p rules
# touch node_rules.yml
```  

3、添加记录规则  
```
# vim node_rules.yml
groups:
- name: node_rules
  interval: 10s
  rules:
  - record: instance:node_cpu:avg_rate5m
    expr: 100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance) * 100)
    labels:
      metric_type: aggregation
  - record: instance:node_memory_usage:percentage
    expr: (node_memory_MemTotal_bytes - (node_memory_MemFree_bytes + node_memory_Cached_bytes + node_memory_Buffers_bytes)) / node_memory_MemTotal_bytes * 100
  - record: instance:root:node_filesystem_usage:percentage
    expr: (node_filesystem_size_bytes{mountpoint="/"} - node_filesystem_free_bytes{mountpoint="/"}) /node_filesystem_size_bytes{mountpoint="/"} * 100
```  
