ssh秘钥配置
=========
1、安装Publish  Over SSH、Git plugin插件  
安装Publish  Over SSH、Git plugin插件。  
安装完成后要重启jenkins服务。  
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
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins7.jpg)
Passphrase：留空，Path  to key：留空，把101的私钥复制黏贴到key  


![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins8.jpg)




同时将101机子的公钥内容（/root/.ssh/id_rsa.pub ）复制到102的/root/.ssh/authorized_keys  

这里添加了两个ssh：  
![image](https://github.com/mykubernetes/linux-install/blob/master/image/jenkins9.jpg)
点击：Test Configuration，均显示Sucess。OK，配置成功。  

SSH密钥设置思路：  

1、jenkins端生成SSH密钥
2、将jenkins生成的私钥内容（/.ssh/id_rsa）复制到Publish Over SSH中的key
3、将jenkins端的公钥（/root/.ssh/id_rsa.pub）内容复制黏贴到客户端的/root/.ssh/authorized_keys文件中。
