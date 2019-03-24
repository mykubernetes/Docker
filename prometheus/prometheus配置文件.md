prometheus配置文件  
1、删除metrics  
```
  - job_name: 'docker'
    static_configs:
      - targets: ['192.168.20.172:8080', '192.168.20.173:8080', '192.168.20.174:8080']
    metric_relabel_configs:
      - source_labels: [__name__]
        separator: ','
        regex: '(container_tasks_state|container_memory_failures_total)'
        action: drop
```  
2、更换标签  
```
  - job_name: 'docker'
    static_configs:
      - targets: ['192.168.20.172:8080', '192.168.20.173:8080', '192.168.20.174:8080']
    metric_relabel_configs:
      - source_labels: [id]
        regex: '/kubepods/([a-z0-9]+);'
        replacement: '$1'
        target_label: container_id
```  
3、删除标签  
```
  - job_name: 'docker'
    static_configs:
      - targets: ['192.168.20.172:8080', '192.168.20.173:8080', '192.168.20.174:8080']
    metric_relabel_configs:
      - regex: 'kernelVersion'
        action: labeldrop
```  
