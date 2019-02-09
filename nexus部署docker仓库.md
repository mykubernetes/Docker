使用容器安装Nexus3
=================
1.下载nexus3的镜像：  
``` docker pull sonatype/nexus3 ```  
2.使用镜像启动一个容器：  
``` docker run -d --name nexus  --restart=always -p 5000:5000 -p 8081:8081 sonatype/nexus3 ```  
注：5000端口是用于镜像仓库的服务端口   8081 端口是nexus的服务端口  
3.启动之后我们就可以通过http://服务器IP:8081访问。  
默认账号密码为admin/admin123  

创建Docker私有仓库   

通过浏览器访问Nexus：  
http://服务器IP:8081  

点击右上角进行登录，通过初始用户名和密码进行登录（admin/admin123）：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
 

点击设置界面，选择Repositories，点击Create repository，如下图所示： 
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
 

选择仓库类型，这里Docker有三种类型，分别是group、hosted、proxy。这里只演示hosted类型，所以选择docker(hosted)，如下图：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
注：Docker镜像仓库类型含义解释如下：  
　　hosted : 本地存储，即同docker官方仓库一样提供本地私服功能  
　　proxy : 提供代理其他仓库的类型，如docker中央仓库  
　　group : 组类型，实质作用是组合多个仓库为一个地址  

指定docker仓库的名称、指定一个端口用来通过http的方式进行访问仓库、勾选是否支持docker API V1，然后create repository；  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
因为我们测试的时候不是使用加密的HTTPS进行访问，所以这里需要增加一个docker的启动参数，给他指定私库的地址，如下：  

编辑/etc/docker/daemon.json 增加如下内容，当然也可通过启动参数增加  
```
{

   "insecure-registries":["http://172.17.9.81:5000"]

}
```  
重启docker进程： systemctl restart docker  

查看docker信息： docker info ，有如下输出即正常  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
 

登录私库  

要使用私库进行上传下载需要进行登录连接到Nexus  

　　docker login http://172.17.9.81:5000/repository/docker-assoft/  

Docker上传镜像到私库  

使用docker tag 对镜像进行管理（必须进行此项操作）  

　　docker tag使用格式：  

　  docker tag SOURCE_IMAGE[:TAG]  TARGET_IMAGE[:TAG]  

　　docker tag portainer-temlates-new:latest 172.17.9.81:5000/portainer-templates:v1  

　　docker push 172.17.9.81:5000/portainer-templates:v1  

图例：使用tag进行打标，正常上传的结果  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
 

图例：不进行tag打标，会出现denied: requested access to the resource is denied报错  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
 


上传完成后，在nexus中对应的docker库中，即可看到此镜像
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
 
下载私库中的镜像  
1、删除本地上例实验中的镜像（docker rmi 172.17.9.81:5000/portainer-templates:v1）  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
2、docker pull 172.17.9.81:5000/portainer-templates:v1  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/nexus1.png)  
 
