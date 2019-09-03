![image](https://github.com/mykubernetes/linux-install/blob/master/image/alter.png)  

slave配置说明：
```
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  external_labels:
    worker: 1                                #从节点需要修改1,默认第一台从0开始，

rule_files:
  - "rules/*_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    dns_sd_configs:
      - names: ['_prometheus._tcp.xiodi.cn']
    relabel_configs:
    - source_labels: [__address__]          #把以前的address标签重新打标签，为tmp_hash,
      modulus: 2                            #slave节点个数
      target_label: __tmp_hash
      action: hashmod                       #设置hash值
    - source_labels: [__tmp_hash]           #对tmp_hash标签进行保留操作
      regex: ^1$                            #和worker值保持一致
      action: keep                          #对标签进行保留
注：
各个 slave（worker）节点,唯一的区别在于 prometheus.yml 的定义
worker 0 代表第一个slave
worker 1 代表第二个slave
modulus 代表总共几个slave节点，会根据这个做hash分配
regex ^0$ 代表匹配 worker: 0
```  
