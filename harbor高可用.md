1、harbor高可用架构
```
    nginx
    |    |
harbor  harbor    #harobor双主复制模型     
```  

2、部署docker和docker-compose  
```
# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# wget https://github.com/docker/compose/releases/download/1.24.0-rc1/docker-compose-Linux-x86_64
# chmod +x docker-compose-Linux-x86_64
# mv docker-compose-Linux-x86_64 /usr/bin/docker-compose
```  

3、两台主机分别部署harbor  
```
# wget https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-offline-installer-v1.8.1.tgz
# tar xvf harbor-offline-installer-v1.8.1.tgz
# cd harbor
# vim harbor.cfg
hostname: 192.168.101.70
# ./install.sh
```  

4、部署nginx  
```
部署docker
# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

编辑配置文件
# cat nginx.conf 
user nginx;
workdf_processes 1;

error_log /var/log/nginx/error.log warn;

pid /var/run/nginx.pid;

events {

    worker_connections 1024;
}

stream {

   upstream hub {
        server 192.168.101.70:80;
        server 192.168.101.71:80;
   }
   server {
        listen 80;
        proxy_pass hub;
        proxy_timeout 300s;
        proxy_connect_timeout 5s;
   }
}

运行nginx
# docker run -idt --net=host --name harbornginx -v /root/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.13.12
```  

5、图形配置双主复制  

6、测试  
```
# docker tag nginx:1.13.12 192.168.101.69/kubernetes/nginx:1.13.12

# cat /etc/docker/daemon.json 
{
     "insecure-registries": ["192.168.101.69"]
}


# docker push 192.168.101.69/kubernetes/nginx:1.13.12

```  

