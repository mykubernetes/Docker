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

4、运行prometheus  
```
docker run -d --name=prometheus \
	-p 9090:9090 \
	-v /opt/prometheus/data/:/data/ \
	prom/prometheus \
	--config.file=/data/prometheus.yml \
	--storage.tsdb.path=/data/
```  

