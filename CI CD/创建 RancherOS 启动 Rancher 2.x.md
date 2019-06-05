1、安装 Docker machine  
```
$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```  

2、查看安装后的版本  
```
$ docker-machine -v
docker-machine version 0.16.0, build 702c267f  
```  

3、创建 RancherOS  
可以使用 docker-machine 命令来加载 VM 虚拟机来创建 RancherOS 了，目前支持虚拟机类型有 VirtualBox、VMWare(VMWare VSphere, VMWare Fusion)、AWS，注意在创建前，请选择以上一种 VM 并安装到本地，这里我本机已安装 virtualbox 虚拟机了。  
```
$ docker-machine create -d virtualbox \
        --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso \
        --virtualbox-memory 3072 \
        rancher-machine        
```  

4、稍等一会，它会自动创建一个名称为 rancher-machine ，内存为 3072M，系统为 Linux 4.14.85 的虚拟机 RancherOS 系统。  
```
# 查看创建的虚拟机列表
$ docker-machine ls
NAME              ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
rancher-machine   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.06.1-ce

# 查看虚拟机 IP
$ docker-machine ip rancher-machine
192.168.99.101

# 进入到 rancher-machine 虚拟机内
$ docker-machine ssh rancher-machine
[docker@rancher-machine ~]$ docker -v
Docker version 18.06.1-ce, build e68fc7a
```  

5、RacherOS 系统内有两种 Docker 进程，一种就是常用的 Docker Daemon，也就是我们说的 Docker 容器，另一种就是 System Docker，这个是 RancherOS 运行系统服务的进程，例如：syslog、ntp、console 等系统服务。  
```
$ sudo system-docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
rancher/os                  v1.5.1              d63929d3b7e7        4 weeks ago         155MB
rancher/os-logrotate        v1.5.1              75feef2a843e        4 weeks ago         46.9MB
rancher/os-syslog           v1.5.1              4fd39422c03a        4 weeks ago         46.9MB
rancher/os-console          v1.5.1              533f86f2ce17        4 weeks ago         46.9MB
rancher/os-acpid            v1.5.1              513cc99c250c        4 weeks ago         46.9MB
rancher/os-base             v1.5.1              6aec1b999629        4 weeks ago         46.9MB
rancher/os-docker           18.06.1-1           5097564f0920        5 weeks ago         148MB
rancher/container-crontab   v0.4.0              7a2485d285d9        14 months ago       12.9MB

$ sudo system-docker ps
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS               NAMES
9bf9855e2132        rancher/os-console:v1.5.1          "/usr/bin/ros entr..."   1 days ago          Up 1 days                               console
89e69b66af05        rancher/os-docker:18.06.1-1        "ros user-docker"        1 days ago          Up 1 days                               docker
764787b89066        rancher/os-base:v1.5.1             "/usr/bin/ros entr..."   1 days ago          Up 1 hours                              ntp
c6c6520de701        rancher/os-base:v1.5.1             "/usr/bin/ros entr..."   1 days ago          Up 1 days                               network
96a22ba5b10c        rancher/os-base:v1.5.1             "/usr/bin/ros entr..."   1 days ago          Up 1 days                               udev
4226d2939e7e        rancher/container-crontab:v0.4.0   "container-crontab"      1 days ago          Up 1 days                               system-cron
34aaaeec7b00        rancher/os-acpid:v1.5.1            "/usr/bin/ros entr..."   1 days ago          Up 1 days                               acpid
87d675f308eb        rancher/os-syslog:v1.5.1           "/usr/bin/entrypoi..."   1 days ago          Up 1 days                               syslog
```  

6、启动 Rancher 2.x  
```
$ sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```  
