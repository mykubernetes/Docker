prometheus配置文件  
1、删除metrics  
```
  - job_name: 'docker'
    static_configs:
      - targets: ['192.168.20.172:8080', '192.168.20.173:8080', '192.168.20.174:8080']
    metric_relabel_configs:
      - source_labels: [__name__]               #源标签__name__为内置标签
        separator: ','                          #指定分隔符，默认为","
        regex: '(container_tasks_state|container_memory_failures_total)'  #正则表达式，key,对名称是这两个的标签执行动作
        action: drop                            #动作删除
```  
2、更换标签  
```
  - job_name: 'docker'
    static_configs:
      - targets: ['192.168.20.172:8080', '192.168.20.173:8080', '192.168.20.174:8080']
    metric_relabel_configs:
      - source_labels: [id]                     #源标签key为id的
        regex: '/kubepods/([a-z0-9]+)'         #为id标签里值为kubepods.*的value传递给$1
        replacement: '$1'                       #$1为匹配到的值value，输出的值为原值，也可以不使用$1，则使用的是自定义的值
        target_label: container_id              #将匹配到符合正则表达式的value的标签名，赋予一个新的标签名
```  
- target_label 为key
- replacement 为value
- regex 正则表达式匹配
- source_lables 源标签
- separator 分隔符，正则表达式多个的分隔符

3、删除标签  
```
  - job_name: 'docker'
    static_configs:
      - targets: ['192.168.20.172:8080', '192.168.20.173:8080', '192.168.20.174:8080']
    metric_relabel_configs:
      - regex: 'kernelVersion'                 #正则表达式，匹配的标签key
        action: labeldrop                      #执行的动作删除标签
```  

参考地址：
https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Crelabel_config%3E
