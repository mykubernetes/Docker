jenkins安装
==========
1、 获取镜像  
``` docker pull jenkins/jenkins:lts ```  
2、修改目录权限否则启动失败  
```
useradd jenkins
mkdir /opt/jenkins
chown -R  jenkins.jenkins /opt/jenkins
```  
3、启动jenkins服务  
``` docker run -d --name jenkins \
-v /opt/jenkins:/var/jenkins_home \
--restart always \
-p 8080:8080 \
-p 50000:50000 \
-p 45000:45000 \
jenkins/jenkins:lts
```  
4、初始化  
http://192.168.101.66:8080/  
1、输入密码
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins1.png)  
2、复制密码，因为是docker映射所以目录在/opt/jenkins/下
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins2.png)  
3、安装插件
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins3.png)  
5、常用目录
```
plugins        #存储插件目录
jobs             #存储作业目录
config.xml   #主配置文件
```
