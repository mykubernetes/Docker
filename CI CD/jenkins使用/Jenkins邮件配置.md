Jenkins邮件配置  
==============
1、Jenkins邮件配置  
依次点击：系统管理-->系统设置-->Jenkins Location，向下拉找到邮件通知，然后设置。  
系统管理员邮件地址和发邮件的邮箱要保持一致。  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins18.jpg)  
设置好之后，点击测试一下。提示：Email was successfully sent，OK，配置成功。  

配置好之后，回到test_php工程里配置：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins19.jpg)  
选择：构建后操作--> E-mail  Notification：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins20.jpg)

重新构建项目，看看会不会收到邮件通知：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins21.jpg)  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins22.jpg)
不过：这个邮件通知，只有在构建失败的时候才会发送。如果构建成功，则不会发送邮件。  


2、Extension E-mail Notification配置  
插件名称：Email Extension Plugin
依次点击：系统管理-->系统设置-->Extension E-mail Notification。  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins23.jpg)  
往下拉，点击：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins24.jpg)
勾选：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins25.jpg)
最后，把前面设置的：邮件通知，删除，然后保存退出即可。  

（2）修改test_php的配置：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins26.jpg)
删除E-mail Notification的配置：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins27.jpg)
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins28.jpg)
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins29.jpg)
（3）测试  
