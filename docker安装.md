docker安装
===========

配置yum源  
阿里云镜像站：https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo  

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

