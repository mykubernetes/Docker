1、harbor高可用架构
```
node01 192.168.101.69 docker nginx
node02 192.168.101.70 docker harbor
node03 192.168.101.71 docker harbor
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

6、node01端配置测试，所有需要上传下载镜像都需要配置此操作  
```
# docker tag nginx:1.13.12 192.168.101.69/kubernetes/nginx:1.13.12

# cat /etc/docker/daemon.json 
{
     "insecure-registries": ["192.168.101.69"]
}

图形化配置，创建仓库，创建用户，将仓库添加成员

# docker login 192.168.101.69
# docker push 192.168.101.69/kubernetes/nginx:1.13.12
The push refers to repository [192.168.101.69/kubernetes/nginx]
7ab428981537: Pushed 
82b81d779f83: Pushed 
d626a8ad97a1: Pushed 
1.13.12: digest: sha256:e4f0474a75c510f40b37b6b7dc2516241ffa8bde5a442bde3d372c9519c84d90 size: 948
```  

