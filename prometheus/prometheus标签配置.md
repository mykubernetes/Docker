prometheus配置文件

1、丢弃不需要的度量

它将使带有名称`container_tasks_state`和`container_memory_failures_total`的metric完全删除，并且不会存储在数据库中.而`__name__`针对metric名称的保留的
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
- relabel_configs # 抓取之前使用
- metrics_relabel_configs # 抓取之后使用


2. 丢弃不需要的时间序列

丢弃所有含有标签对`id="/system.slice/var-lib-docker-containers.*-shm.mount"`或者标签`container_label_JenkinsId`的时间序列。如果它属于一个单一的度量，这就没有必要了。它将适用于所有具有预定义标签集的指标。这可能有助于避免不必要的垃圾对指标数据的污染。对于container_label_JenkinsId，当您让Jenkins在Docker容器中运行从服务器，并且不希望它们打乱底层主机容器指标(例如Jenkins服务器容器本身、系统容器等)时，这一点特别有用。

```
- job_name: cadvisor
  ...
  metric_relabel_configs:
  - source_labels: [id]
    regex: '/system.slice/var-lib-docker-containers.*-shm.mount'
    action: drop
  - source_labels: [container_label_JenkinsId]
    regex: '.+'
    action: drop
```

3、丢弃敏感或者不想要的某些标签的度量

将中删除名称为`container_label_com_amazonaws_ecs_task_arn`的标签。出于安全原因不希望普罗米修斯记录敏感数据时。注意，在删除标签时，需要确保标签删除后的最终指标仍然是唯一标记的，并且不会导致具有不同值的重复时间序列。

```
- job_name: cadvisor
  ...
  metric_relabel_configs:
  - regex: 'container_label_com_amazonaws_ecs_task_arn'    #正则表达式，匹配的标签名
    action: labeldrop                     #执行的动作删除标签
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
      - source_labels: [image]
        regex: '.*/(.*)'
        replacement: '$1'
        target_label: id
      - source_labels: [service]
        regex: 'ecs-.*:ecs-([a-z]+-*[a-z]*).*:[0-9]+'
        replacement: '$1'
        target_label: service
```  
- target_label 为key
- replacement 为value
- regex 正则表达式匹配
- source_lables 源标签
- separator 分隔符，正则表达式多个的分隔符

参考地址：
https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Crelabel_config%3E



允许在采集之前对任何目标及其标签进行修改  
• 重命名标签名  
• 删除标签  
• 过滤目标  
action：重新标签动作
- replace：将regex与source_labels进行匹配，如果匹配，那么设置target_label为replacement，如果正则表达式不匹配，那么就不执行动作。
- keep：针对正则表达式不匹配source_labels的目标，就丢弃
- drop：针对正则表达式匹配的source_labels的目标，就丢弃
- labeldrop：将regex与所有标签名匹配。任何匹配的标签都将从标签集中删除
- labelkeep：将regex与所有标签名匹配。任何不匹配的标签将从标签集中删除
- hashmod：设置target_label为modulus连接的哈希值source_labels
- labelmap：匹配regex所有标签名称。然后复制匹配标签的值进行分组，replacement分组引用（${1},${2},…）替代

如果没有定义action，那么默认的就是replace.

```
relable_configs:
  # 源标签
  [ source_labels: '[' <labelname> [, ...] ']' ]
  
  # 多个源标签时连接的分隔符
  [ separator: <string> | default = ; ]
  
  # 重新标记的标签
  [ target_label: <labelname> ]
  
  # 整则表达式匹配源标签的值
  [ regex: <regex> | default = (.*) ]
  
  # work节点个数
  [ modulus: <uint64> ]
  
  # 替换正则表达式匹配的分组，分组引用 $1,$2,$3,....
  [ replacement: <string> | default = $1 ]
  
  # 基于正则表达式匹配执行的操作
  [ action: <relabel_action> | default = replace ]
```  
