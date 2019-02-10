
四、发布php代码  
（1）
（2）新建一个任务  
首页点击新建任务。  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins10.jpg)  
填写相关信息：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins11.jpg)  
选择构建自由风格的软件项目，然后点确定。  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins12.jpg)  
源码管理选择git，地址自己注册一个。  

构建：  

构建：选择Send files  or execute command over SSH  

``` **/**：表示全部 ```     
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins13.jpg)
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins14.jpg)
根据实际情况设置  

最后，点击保存。  

构建：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins15.jpg)
查看控制台输出：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins16.jpg)
最后显示：SUCESS。  

到lb01、lb02查看一下：  
```
[root@lb01 ~]# ll -d /tmp/jenkins_test/
drwxr-xr-x 2 nobody nobody 151 Sep 11 19:54 /tmp/jenkins_test/
[root@lb01 ~]# 

lb02：

[root@lb02 ~]# ll -d /tmp/lb02/
drwxr-xr-x 2 nobody nobody 151 Sep 11 19:54 /tmp/lb02/
[root@lb02 ~]# 
```
（3）测试  

登录到自己的https://github.com/yanyuzm/mytest。比如在修改README.MD文件内容：  

    # mytest  
    测试项目  
    #  

提交后。  

回到jenkins点击立即构建  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins17.jpg)
构建后，到lb01查看一下：  

[root@lb01 ~]# cat /tmp/jenkins_test/README.md   
# mytest  
测试项目  
#  
[root@lb01 ~]#   

在lb02查看：  

[root@lb02 ~]# cat /tmp/lb02/README.md  
# mytest  
测试项目  
#  
[root@lb02 ~]#  

均同步成功。  

也就是说在jenkins上修改项目的文件，重新构建后，会同步到相关的机子上。  



八、部署java项目-创建私有仓库  

java的项目需要编译和打包，编译和打包可以使用maven。  

本次实验使用git私有仓库的形式。请到https://github.com/注册一个私有仓库，并且设置ssh密钥登陆。  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins32.jpg)  
将上面的仓库克隆到/home目录中  
```
[root@lb01 ~]# cd /home/
[root@lb01 home]# git clone git@github.com:yanyuzm/test_java
Cloning into 'test_java'...
warning: You appear to have cloned an empty repository.
[root@lb01 home]#
```  
初始化及创建推送一个测试文件：  
```
[root@lb01 home]# cd test_java/
[root@lb01 test_java]# git init
Reinitialized existing Git repository in /home/test_java/.git/
[root@lb01 test_java]# 
```  
推送测试文件：  
```
[root@lb01 test_java]# echo haha > README.md
[root@lb01 test_java]# git add README.md
[root@lb01 test_java]# git commit -m "first commit"
[master (root-commit) 4fb58db] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
[root@lb01 test_java]# git remote add origin https://github.com/yanyuzm/test_java.git
fatal: remote origin already exists.
[root@lb01 test_java]# git push -u origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 206 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: 
remote: Create a pull request for 'master' on GitHub by visiting:
remote:      https://github.com/yanyuzm/test_java/pull/new/master
remote: 
To git@github.com:yanyuzm/test_java
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
[root@lb01 test_java]#
```  
查看一下：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins33.jpg)  
OK，创建成功。  
九、部署java项目-下载zrlog源码  

下载地址：https://codeload.github.com/94fzb/zrlog/zip/master  

下载到/home/目录  
```
[root@lb01 home]# curl -O https://codeload.github.com/94fzb/zrlog/zip/master
```  
解压，解压后的目录为：zrlog-master  
```
[root@lb01 home]# ls
git  master  mytest  test  test_java  zrlog-master
[root@lb01 home]# 
```  
将zrlog-master目录的全部文件复制到到test_java目录中。  
```
[root@lb01 home]# mv zrlog-master/* test_java/
mv: overwrite ‘test_java/README.md’? r
[root@lb01 home]# 
```  
进入test_java目录推送：  
```
[root@lb01 home]# cd test_java/
[root@lb01 test_java]# git add .
[root@lb01 test_java]# git commit -m "add zrlog" 
[root@lb01 test_java]# git push
Counting objects: 568, done.
Compressing objects: 100% (517/517), done.
Writing objects: 100% (567/567), 3.96 MiB | 1.07 MiB/s, done.
Total 567 (delta 61), reused 0 (delta 0)
remote: Resolving deltas: 100% (61/61), done.
To git@github.com:yanyuzm/test_java
   4fb58db..67c89f3  master -> master
[root@lb01 test_java]# 
```  

十一、部署java项目-安装maven  

1、下载、安装maven  

maven要安装在jenkins所在的机子上，前面中，jenkins安装在192.168.10.101机子上。  

下载地址：http://maven.apache.org/download.cgi  

下载并解压到/usr/local/目录中  
```
[root@lb01 ~]# curl -O https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
[root@lb01 ~]# tar xf apache-maven-3.5.4-bin.tar.gz -C /usr/local/
[root@lb01 ~]#
```  
2、配置环境变量  
```
[root@lb01 ~]# vim /etc/profile.d/maven.sh
export MAVEN_PATH=/usr/local/apache-maven-3.5.4
export PATH=$MAVEN_PATH/bin:$PATH
[root@lb01 ~]# chmod +x /etc/profile.d/maven.sh
[root@lb01 ~]# source /etc/profile.d/maven.sh
[root@lb01 ~]# 
```  
查看maven版本信息：  
```
[root@lb01 ~]# mvn --version
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-18T02:33:14+08:00)
Maven home: /usr/local/apache-maven-3.5.4
Java version: 1.8.0_181, vendor: Oracle Corporation, runtime: /usr/local/jdk1.8/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-862.el7.x86_64", arch: "amd64", family: "unix"
[root@lb01 ~]# 
```  
至此，maven安装完成。  

3、jenkins安装maven插件  

浏览器打开：192.168.10.101:8080，登录成功后。  

点击：系统管理-->全局工具配置  

设置全局文件路径，如下图：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins34.jpg)  
新增maven，如下图：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins35.jpg)  
配置jdk：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins36.jpg)  
我的jdk安装路径是：/usr/local/jdk1.8  

配置好maven，点击保存  
十二、部署java项目-安装插件  

登录jenkins之后，点击系统管理-->管理插件  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins37.jpg)  
检查是否安装了Maven Integration plugin和Deploy to container Plugin这两个插件。如果没有安装，则需要安装。  

安装成功后，重启jenkins服务。  

登录jenkins后点击新建，可以看到有maven的项目选项了：  

至此，插件安装成功。  
十三、部署java项目-构建job  

1、创建项目  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins38.jpg)  
创建一个maven项目，名称为：test-java  

创建成功后，页面跳转到：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins39.jpg)  
填写源码管理中的url仓库地址。  

如果仓库是私有的，必须配置用户密码等信息，如下：    
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins40.jpg)  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins41.jpg)  
配置好ssh私钥信息后，保存，下图中选择git。  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins42.jpg)  
Build设置：  

Goals and options可以设置成：clean install -D maven.test.skip=true，也可以默认不设置；Root POM保持默认。  

构建后操作：  

添加Editable  Email Notification设置：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins43.jpg)  
设置好之后，保存。  

2、构建项目  

点击立即构建：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins44.jpg)  
十四、部署java项目-手动安装jdk  

1、下载jdk，并解压到/usr/local/目录，然后重命名为：/usr/local/jdk1.8/  

jdk下载地址：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html  

2、配置环境变量：  
```
[root@lb01 ~]# vim /etc/profile.d/jdk.sh 
export JAVA_HOME=/usr/local/jdk1.8
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=$JAVA_HOME/lib

[root@lb01 ~]# chmod +x /etc/profile.d/jdk.sh
[root@lb01 ~]# source /etc/profile.d/jdk.sh
```  
十五、部署java项目-发布war包  

配置test-java工程的构建后操作：
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins45.jpg)  
选择Deploy war/ear to a container：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins46.jpg)  
设置如下：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins47.jpg)  
设置访问tomcat的用户名和密码（不支持tomcat9）：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins48.jpg)  
点击添加，然后设置如下：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins49.jpg)  
保存退出即可。  

最后，立即构建。  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins50.jpg)  
到tomcat所在的机子（192.168.10.102）查看一下：  
```
[root@lb02 ~]# ls /usr/local/tomcat9.0/webapps/
docs  examples  host-manager  manager  ROOT  zrlog-2.0.6  zrlog-2.0.6.war
[root@lb02 ~]# 
```  
war包已经发布过去了。  

浏览器打开：http://192.168.10.102:8080/zrlog-2.0.6。  

Jenkins中的工程构建tomcat只有8.x版本，而192.168.10.102机子装的是9.0，所以最终会报错。  

换回8.x版本之后，重新构建即可。  

tomcat8.5下载地址：https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.34/bin/apache-tomcat-8.5.34.tar.gz

192.168.10.102安装tomcat8.5成功，并配置好之后  
```
[root@lb02 local]# startup.sh 
Using CATALINA_BASE:   /usr/local/tomcat8.5
Using CATALINA_HOME:   /usr/local/tomcat8.5
Using CATALINA_TMPDIR: /usr/local/tomcat8.5/temp
Using JRE_HOME:        /usr/local/jdk1.8
Using CLASSPATH:       /usr/local/tomcat8.5/bin/bootstrap.jar:/usr/local/tomcat8.5/bin/tomcat-juli.jar
Tomcat started.
[root@lb02 local]# 
```  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins51.jpg)
重新构建test-java工程。

部分构建信息如下：

[INFO] Reactor Summary:
[INFO] 
[INFO] zrlog 2.0.6 ........................................ SUCCESS [  1.461 s]
[INFO] common ............................................. SUCCESS [  6.646 s]
[INFO] data ............................................... SUCCESS [  3.105 s]
[INFO] service ............................................ SUCCESS [  3.740 s]
[INFO] web 2.0.6 .......................................... SUCCESS [  9.116 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 27.644 s
[INFO] Finished at: 2018-09-12T00:42:39+08:00
[INFO] ------------------------------------------------------------------------
Waiting for Jenkins to finish collecting data
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/data/pom.xml to com.zrlog/data/2.0.6/data-2.0.6.pom
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/data/target/data-2.0.6.jar to com.zrlog/data/2.0.6/data-2.0.6.jar
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/common/pom.xml to com.zrlog/common/2.0.6/common-2.0.6.pom
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/common/target/common-2.0.6.jar to com.zrlog/common/2.0.6/common-2.0.6.jar
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/pom.xml to com.zrlog/zrlog/2.0.6/zrlog-2.0.6.pom
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/service/pom.xml to com.zrlog/service/2.0.6/service-2.0.6.pom
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/service/target/service-2.0.6.jar to com.zrlog/service/2.0.6/service-2.0.6.jar
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/web/pom.xml to com.zrlog/web/2.0.6/web-2.0.6.pom
[JENKINS] Archiving /var/lib/jenkins/workspace/test-java/web/target/../../target/zrlog-2.0.6.war to com.zrlog/web/2.0.6/web-2.0.6.war
/var/lib/jenkins/workspace/test-java/target/zrlog-2.0.6.war is not inside /var/lib/jenkins/workspace/test-java/web/; will archive in a separate pass
channel stopped
Deploying /var/lib/jenkins/workspace/test-java/target/zrlog-2.0.6.war to container Tomcat 8.x Remote with context 

  [/var/lib/jenkins/workspace/test-java/target/zrlog-2.0.6.war] is not deployed. Doing a fresh deployment.
  Deploying [/var/lib/jenkins/workspace/test-java/target/zrlog-2.0.6.war]

Email was triggered for: Always
Sending email for trigger: Always
Sending email to: xzm280@163.com

Finished: SUCCESS

OK，成功。

浏览器再次打开：http://192.168.10.102:8080/zrlog-2.0.6/
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins52.jpg)
OK，发布成功。
