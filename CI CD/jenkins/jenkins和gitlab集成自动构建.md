jenkins和gitlab集成自动构建
======
步骤瞬间不能错
------------
1、安装jenkins集成gitlab插件  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins54.png)  
2、生产gitlab的ipa Token验证  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins50.png)  
3、复制Token  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins51.png)  
4、jenkins添加gitab点击系统设置  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins49.png)  
5、添加gitlab  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins53.png)  
6、添加gitlab生成的api token  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins52.png)  
7、添加成功  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins55.png)  
8、登陆jenkins找到相应项目配置gitlab触发并复制web地址和token  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins56.png)  
9、将jenkins的web地址和token地址添加到gitlab里  
这样gitlab提交代码后会通知jenkins 并触发相应操作  
在对应的项目里添加次操作
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins57.png)  
