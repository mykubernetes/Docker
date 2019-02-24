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
6、登录web界面查看  
http://192.168.101.66:9090  

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

三、部署grafana  
--------------
1、拉取镜像  
``` # docker pull grafana/grafana ```  

2、运行grafana  
``` # docker run -d --name=grafana -p 3000:3000 grafana/grafana ```  

3、登录web  
http://192.168.101.66:3000  
1)添加数据源  
![image]https://github.com/mykubernetes/linux-install/blob/master/image/grafana1.png  
![image]https://github.com/mykubernetes/linux-install/blob/master/image/grafana2.png  

2)导入模板可去官网查找相关模板  
https://grafana.com/dashboards  
![image]https://github.com/mykubernetes/linux-install/blob/master/image/grafana4.png  

3)添加模板  
![image]https://github.com/mykubernetes/linux-install/blob/master/image/grafana3.png  
![image]https://github.com/mykubernetes/linux-install/blob/master/image/grafana5.png  


