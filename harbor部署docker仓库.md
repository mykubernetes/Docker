harbor部署docker仓库
====================
1、下载地址  
链接：https://pan.baidu.com/s/1YEcx59DCE09tUVtRcV79-Q 提取码：oofm  
2、github下载地址  
https://github.com/goharbor/harbor/releases  
3、解压缩  
``` # tar xvf harbor-offline-installer-v1.4.0.tgz -C /op/module ```  
4、配置文件一般改这三项  
```
vim /opt/module/harbor/harbor.cfg
hostname = node01.com
harbor_admin_password = Harbor12345
db_password = root123
```  
5、修改docker-compose.myl配置仓库存储路径  

6、安装docker-compose命令  
``` yum install -y docker-compose ```  
7、执行安装脚本  
``` 
cd /opt/module/harbor/
./install.sh
```
