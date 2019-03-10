docker安装
===========

配置yum源  
https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo  

一、查看docker各个版本  
```
yum list docker-ce --showduplicates
Loaded plugins: product-id, search-disabled-repos, subscription-manager
This system is not registered with an entitlement server. You can use subscription-manager to register.
Installed Packages
docker-ce.x86_64                     18.06.0.ce-3.el7                             @docker-ce-stable
Available Packages
docker-ce.x86_64                     17.03.0.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.03.1.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.03.2.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.03.3.ce-1.el7                             docker-ce-stable 
docker-ce.x86_64                     17.06.0.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.06.1.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.06.2.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.09.0.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.09.1.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.12.0.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     17.12.1.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     18.03.0.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     18.03.1.ce-1.el7.centos                      docker-ce-stable 
docker-ce.x86_64                     18.06.0.ce-3.el7                             docker-ce-stable 
docker-ce.x86_64                     18.06.1.ce-3.el7                             docker-ce-stable 
```  
二、安装指定版本  

``` yum install -y docker-ce-17.06.0.ce  ```  

三、配置docker加速器  
```
$ touch /etc/docker/daemon.json
$ cat /etc/docker/daemon.json

{
 "registry-mirrors": ["https://registry.docker-cn.com"]
}
或者：
{
 "registry-mirrors": ["https://registry.docker-cn.com","https://dhq9bx4f.mirror.aliyuncs.com"]
}
```  

四、镜像的导入导出  
1、将镜像文件导出为tar文件:  
``` docker save jenkins:v1.1 -o jenkins.tar.gz  ```  
2、从tar文件导入镜像 ：  
``` docker load -i jenkins.tar.gz ```  

五、修改启动文件  
```
# cat /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
Environment="HTTPS_PROXY=http://www.ik8s.io:10070"
Environment="NO_proxy=127.0.0.0/8,192.168.101.0/24"
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target



#systemctl daemon-reload
#systemctl restart  docker
```
