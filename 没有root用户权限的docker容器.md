问题描述  
运行没有root用户权限的docker容器，将面临挂载本地卷无权限的问题，示例如下：  
```
docker run -d --name postgresql-master \
    -p 5432:5432 \
    -e POSTGRESQL_REPLICATION_MODE=master \
    -e POSTGRESQL_USERNAME=postgres \
    -e POSTGRESQL_PASSWORD=password123 \
    -e POSTGRESQL_DATABASE=my_database \
    -e POSTGRESQL_REPLICATION_USER=my_repl_user \
    -e POSTGRESQL_REPLICATION_PASSWORD=my_repl_password \
    -v /postgresql-master/bitnami/:/bitnami \
    bitnami/postgresql:latest
```  


当配置-v /postgresql-master/bitnami/:/bitnami参数，尝试挂载容器目录到主机目录时容器将启动失败：  
```
# docker logs -f postgresql-master 

Welcome to the Bitnami postgresql container
Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-postgresql
Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-postgresql/issues

nami    INFO  Initializing postgresql
postgre INFO  ==> No injected postgresql.conf file found. Creating default postgresql.conf file...
postgre INFO  ==> No injected pg_hba.conf file found. Creating default pg_hba.conf file...
postgre INFO  ==> Deploying PostgreSQL from scratch...
Error executing 'postInstallation': EACCES: permission denied, mkdir '/bitnami/postgresql'
```  

报错信息如下：  
``` Error executing 'postInstallation': EACCES: permission denied, mkdir '/bitnami/postgresql' ```  

解决方法：
docker run添加以下参数：  
--user $(id -u):$(id -g)

重新执行：
```
docker run -d --name postgresql-master \
    -p 5432:5432 \
    --user $(id -u):$(id -g) \
    -e POSTGRESQL_REPLICATION_MODE=master \
    -e POSTGRESQL_USERNAME=postgres \
    -e POSTGRESQL_PASSWORD=password123 \
    -e POSTGRESQL_DATABASE=my_database \
    -e POSTGRESQL_REPLICATION_USER=my_repl_user \
    -e POSTGRESQL_REPLICATION_PASSWORD=my_repl_password \
    -v /postgresql-master/bitnami/:/bitnami \
    bitnami/postgresql:latest	
```  
