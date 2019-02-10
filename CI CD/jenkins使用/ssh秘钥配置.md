ssh秘钥配置
=========
1、安装Publish  Over SSH、Git plugin插件  
2、生产秘钥对  
```
# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
/root/.ssh/id_rsa already exists.
Overwrite (y/n)? 
```  
3、配置Jenkins的SSH密钥  
点击系统管理-->系统设置，找到Publish Over SSH  
把私钥复制到key  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins7.jpg)
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins8.jpg)
将公钥内容（/root/.ssh/id_rsa.pub ）复制到其他节点/root/.ssh/authorized_keys  

这里添加了两个ssh：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins9.jpg)
点击：Test Configuration，均显示Sucess。OK，配置成功。  
