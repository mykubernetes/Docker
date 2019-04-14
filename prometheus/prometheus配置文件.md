prometheus配置文件  
1、删除metrics  
```
  - job_name: 'docker'
    static_configs:
      - targets: ['192.168.20.172:8080', '192.168.20.173:8080', '192.168.20.174:8080']
    metric_relabel_configs:
      - source_labels: [__name__]               #源标签
        separator: ','                          #分隔符
        regex: '(container_tasks_state|container_memory_failures_total)'      #正则表达式，对名称是这两个的标签执行动作
        action: drop                            #动作删除
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
