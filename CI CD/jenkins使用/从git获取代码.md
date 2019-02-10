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


回到jenkins点击立即构建  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins17.jpg)
构建后，到lb01查看一下：  

