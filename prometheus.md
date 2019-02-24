docker部署prometheus
===================

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

