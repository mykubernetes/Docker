docker部署prometheus
===================

一、部署prometheus
------------------
1、拉取镜像  
``` # docker pull prom/prometheus ```  

2、创建prometheus用户  
``` # useradd -u 65534 prometheus ```  

3、创建prometheus存储目录  
```
# mkdir -p /opt/prometheus/data
# chown -R prometheus.prometheus prometheus
```  

4、快速开始，拷贝容器内，配置文件模板  
```
# docker run -d --name=prometheus -p 9090:9090 prom/prometheus
# docker exec -it prometheus /bin/sh

之前创建的prometheus用户ID65534和容器内的nobody用户ID65534对应
$ cat /etc/passwd 
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false

保存并修改配置到本地磁盘
$ cat /etc/prometheus/prometheus.yml    复制
$ exit
vim /opt/prometheus/config/prometheus.yml   粘贴
注意：粘贴文件的时候按：set paste 否则格式显示不正常
# docker stop prometheus
# docker rm prometheus
```  

5、运行prometheus  
```
# docker run -d --name=prometheus -p 9090:9090 \
-v /opt/prometheus/data/:/data/ \
prom/prometheus \
--config.file=/data/prometheus.yml \
--storage.tsdb.path=/data/
```  

6、服务发现  
https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config  

7、登录web界面查看  
http://192.168.101.66:9090  

#安装 cadvisor
---------------
监控docker  
1、运行cadvisor  
```
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```  
http://192.168.101.66:8080  
2、修改prometheus配置文件  
```
# tail -n 3 /opt/prometheus/data/prometheus.yml
  - job_name: 'docker'
    static_configs:
      - targets: ['192.168.101.66:8080', '192.168.101.67:8080', '192.168.101.68:8080']
```  
二、部署node_exporter  
------------------
1、拉取镜像  
``` # docker pull prom/node-exporter ```  

2、运行exporter  
``` # docker run -d -p 9100:9100 prom/node-exporte ```  

3、修改prometheus配置文件添加exportz主机  
```
# tail -n 5 /opt/prometheus/data/prometheus.yml
- job_name: 'node01'
    static_configs:
    - targets: ['192.168.101.66:9100']
      labels:
        host: 'node01'
```  
4、重启prometheus  
``` # docker restart prometheus ```  

5、性能指标计算  
```
CPU使用率：  
100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance) * 100)  
内存使用率：  
100 - (node_memory_MemFree_bytes+node_memory_Cached_bytes+node_memory_Buffers_bytes) / node_memory_MemTotal_bytes * 100  
磁盘使用率：  
100 - (node_filesystem_free_bytes{mountpoint="/",fstype=~"ext4|xfs"} / node_filesystem_size_bytes{mountpoint="/",fstype=~"ext4|xfs"} * 100)  
```  

6、推荐node的grafana模板  
https://grafana.com/dashboards/9276  

三、添加myslq监控  
----------------
1、拉取镜像  
``` # docker pull prom/mysqld-exporter ```  

2、授权数据库用户  
``` 
mysql> grant process,replication client,select on *.* to 'exporter'@'%' identified by 'exporter'; 
mysql> flush privileges;
```  

3、运行mysql-exporter  
``` # docker run -d --name=prom_mysql -p 9104:9104 -e DATA_SOURCE_NAME="exporter:exporter@(192.168.101.67:3306)/" prom/mysqld-exporter ```  

4、编辑prometheus 服务端配置文件  
```
# tail -n 5 prometheus.yml 
  - job_name: 'mysql'
    static_configs:
    - targets: ['192.168.101.67:9104']
      labels:
        host: 'node02'
```  

5、重启服务端  
``` # docker restart prometheus ```  

6、推荐mysql的grafana模板  
https://grafana.com/dashboards/7362  

四、部署告警  
------------

https://github.com/prometheus/alertmanager/releases  
1、拉取镜像  
``` # docker pull prom/alertmanager ```  

2、运行alertmanager  
``` # docker run -d --name=prom_alertmanager -p 9093:9093 prom/alertmanager ```  

3、编辑prometheus 服务端配置文件  
```
vim /opt/prometheus/data/prometheus.yml
alerting:
  alertmanagers:
   - static_configs: targets:['192.168.101.66:9093']

rule_file:
	- "/data/rule.yml"
```  

```
# tail -n 3 prometheus.yml 
  - job_name: 'alertmanager'
    static_configs:
    - targets: ['192.168.101.66:9093']
```

4、定义告警出发条件  
```
# cat /opt/prometheus/data/rule.yml 
groups:
- name: test-rule
  rules:
  - alert: NodeMemoryUsage                     #metric名字
    expr: (node_memory_MemTotal_bytes - (node_memory_Buffers_bytes + node_memory_Cached_bytes + node_memory_MemFree_bytes )) / node_memory_MemTotal_bytes *100 > 80           #表达式
    for: 1m                                    #告警持续一分钟
    labels:
      severity: critical                       #添加自定义标签
    annotations:
      summary: "{{$labels.instance}}: High Memory useage detected"
      description: "{{$labels.instance}} Memory usage is above 80% (current value is: {{ $value }}"
      
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{$labels.instance}} down"
      description: "{{$labels.instance}} of job {{$labels.job}} has been down for more than 5 minutes."

  - alert: MysqlConnect
    expr: mysql_global_status_threads_connected > 1
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "mysql max connect is warnning"
      description: "mysql max connect is Over connections"
```  

五、配置邮件告警(支持钉钉、微信等）
--------------
``` 
# docker ps
# docker exec -it prom_alertmanager /bin/sh

#cat /etc/alertmanager/alertmanager.yml 
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.163.cn:587'
  smtp_from: 'user@163.com'
  smtp_auth_username: 'user@163.com'
  smtp_auth_password: '123456'
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'email'
  routes:
  - match:
      severity: critical
    receiver: pager
  - match_re:
      severity: ^(warning|critical)$
    receiver: support_team

receivers:
- name: 'email'
  email_configs:
  - to: '123456@qq.com'
- name: 'support_team'
  email_configs:
  - to: '654321@qq.com'
- name: 'pager'
  email_configs:
  - to: 'alert-pager@example.com'


- name: 'webhook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```  
- group_by: 根据 labael(标签)进行匹配，如果是多个，就要多个都匹配  
- group_wait: 30s 等待该组的报警，看有没有一起合伙搭车的  
- group_interval: 5m 下一次报警开车时间  
- repeat_interval: 3h 重复报警时间  
注意：  
新报警报警时间： 上一次报警之后的 group_interval 时间  
重复的报警，下次报警时间为：group_interval + repeat_interval  

钉钉，微信告警https://prometheus.io/docs/alerting/configuration/#webhook_config  
参考文档  
使用文档：https://prometheus.io/docs/guides/node-exporter/  
GitHub：https://github.com/prometheus/node_exporter  
exporter列表：https://prometheus.io/docs/instrumenting/exporters/ 

k8s集群中
```
cAdvisor（Container Advisor）用于收集正在运行的容器资源使用和性能信息
https://github.com/google/cadvisor  

kubelet的节点使用cAdvisor提供的metrics接口获取该节点所  
有容器相关的性能指标数据。  
暴露接口地址：  
https://NodeIP:10255/metrics/cadvisor  
https://NodeIP:10250/metrics/cadvisor  

kube-state-metrics采集了k8s中各种资源对象的状态信息：
kube_daemonset_*
kube_deployment_*
kube_job_*
kube_namespace_*
kube_node_*
kube_persistentvolumeclaim_*
kube_pod_container_*
kube_pod_*
kube_replicaset_*
kube_service_*
kube_statefulset_*

推荐模板：
• 集群资源监控：3119
• 资源状态监控 ：6417
• Node监控 ：9276
```
六、部署grafana  
--------------

官网下载  
https://grafana.com/grafana/download  
1、拉取镜像  
``` # docker pull grafana/grafana ```  

2、运行grafana  
``` # docker run -d --name=grafana -p 3000:3000 grafana/grafana ```  

3、登录web  
http://192.168.101.66:3000  
1)添加数据源  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/grafana1.png)  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/grafana2.png)  

2)导入模板可去官网查找相关模板  
https://grafana.com/dashboards  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/grafana4.png)  

3)添加模板  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/grafana3.png)  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/grafana5.png)  


