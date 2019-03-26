alertmanager_cluster配置
=======================

1、将第一台的命令和配置拷贝到第二台，配置文件一样  

2、启动第一台  
``` # nohup alertmanager --config.file/etc/alertmanager/alertmanager.yml --cluster.listen-address 192.168.101.66:8001 > alertmanager.out 2>&1 & ```  

3、启动第二台  
``` # nohup alertmanager --config.file/etc/alertmanager/alertmanager.yml --cluster.listen-address 192.168.101.67:8001 --cluster.peer 192.168.101.66:8001 > alertmanager.out 2>&1 & ```  

#注意： 集群监听端口为 8001，默认为9094  

4、登录web地方访问可查看集群状态  

5、添加alertmanager主机  
```
# vim /etc/prometheus/prometheus.yml
……
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 192.168.101.66:9093
      - 192.168.101.67:9093
```  
