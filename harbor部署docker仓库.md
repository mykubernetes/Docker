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
# vim /opt/module/harbor/harbor.cfg
hostname = node01.com
harbor_admin_password = Harbor12345
db_password = root123
```  
5、修改docker-compose.myl配置仓库存储路径  
```
vim docker-compose.yml
  log:
    volumes:
      - /app/dcos/harbor/data/log/harbor/:/var/log/docker/:z
  registry:
    volumes:
      - /app/dcos/harbor/data/registry:/storage:z
    ports:
      - 5000:5000
  postgresql:
    volumes:
      - /app/dcos/harbor/data/database:/var/lib/postgresql/data:z
  adminserver:
    volumes:
      - /app/dcos/harbor/data/config/:/etc/adminserver/config/:z
      - /app/dcos/harbor/data/secretkey:/etc/adminserver/key:z
      - /app/dcos/harbor/data/:/data/:z 
  ui:
    volumes:
      - /app/dcos/harbor/data/secretkey:/etc/ui/key:z
      - /app/dcos/harbor/data/ca_download/:/etc/ui/ca/:z
      - /app/dcos/harbor/data/psc/:/etc/ui/token/:z
  jobservice:
    volumes:
      - /app/dcos/harbor/data/job_logs:/var/log/jobs:z
  redis:
    volumes:
      - /app/dcos/harbor/data/redis:/var/lib/redis
```  
注意：/app/dcos/harbor/data/secretkey:/etc/adminserver/key:z，必须和harbor.cfg中设置的一样，不然启动adminserver的时候会报错："harbor failed to initialize the system: read /etc/adminserver/key: is a directory"  

修改docker-compose.chartmuseum.yml配置文件中数据存储目录  
```
volumes:
      - /app/dcos/harbor/data/chart_storage:/chart_storage:z
    ports:
      - 9999:9999
```  


6、安装docker-compose命令  
``` # yum install -y docker-compose ```  
7、执行安装脚本  
``` 
# cd /opt/module/harbor/
# ./install.sh
```  
8、启动停止  
```
# docker-compose start
# docker-compose stop
```  
