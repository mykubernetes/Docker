docker swarm 常用命令  
```
命令                                 说明  
docker swarm init                    初始化集群
docker swarm join-token worker       查看工作节点的 token
docker swarm join-token manager      查看管理节点的 token
docker swarm join                    加入集群中
```  
docker node 常用命令  
```
命令                                 说明
docker node ls                       查看所有集群节点
docker node rm                       删除某个节点（-f强制删除）
docker node inspect                  查看节点详情
docker node demote                   节点降级，由管理节点降级为工作节点
docker node promote                  节点升级，由工作节点升级为管理节点
docker node update                   更新节点
docker node ps                       查看节点中的 Task 任务
```  
docker service 常用命令  
```
命令                                 说明
docker service create                部署服务
docker service inspect               查看服务详情
docker service logs                  产看某个服务日志
docker service ls                    查看所有服务详情
docker service rm                    删除某个服务（-f强制删除）
docker service scale                 设置某个服务个数
docker service update                更新某个服务
```  
docker stack 常用命令  
```
命令                                 说明
docker stack deploy                  部署新的堆栈或更新现有堆栈
docker stack ls                      列出现有堆栈
docker stack ps                      列出堆栈中的任务
docker stack rm                      删除堆栈
docker stack services                列出堆栈中的服务
docker stack down                    移除某个堆栈（不会删除数据）
```  

一、Docker Swarm
---

1、Docker Swarm 配置集群节点  
```
$ docker swarm init --advertise-addr 192.168.99.100
  Swarm initialized: current node (n0ub7dpn90rxjq97dr0g8we0w) is now a manager.
  To add a worker to this swarm, run the following command:
        docker swarm join --token SWMTKN-1-5uwpqibnvmho1png8zmhcw8274yanohee32jyrcjlait9djhsk-envtxo4dl6df2ar3qldcccfdg 192.168.99.100:2377
  To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```  

2、加入集群
加入Swarm集群工作节点  
```
docker swarm join --token SWMTKN-1-5uwpqibnvmho1png8zmhcw8274yanohee32jyrcjlait9djhsk-envtxo4dl6df2ar3qldcccfdg 192.168.99.100:2377
This node joined a swarm as a worker.
```

3、加入Swarm集群管理节点  
```
$ docker swarm join-token manager
    To add a manager to this swarm, run the following command:
    docker swarm join --token SWMTKN-1-5uwpqibnvmho1png8zmhcw8274yanohee32jyrcjlait9djhsk-0koz1b98sco8r5cn3g61eahnu 192.168.99.100:2377

$ docker swarm join --token SWMTKN-1-5uwpqibnvmho1png8zmhcw8274yanohee32jyrcjlait9djhsk-0koz1b98sco8r5cn3g61eahnu 192.168.99.100:2377
This node joined a swarm as a manager.
```  

4、当前swarm有哪些节点
```
# docker node ls
ID                            HOSTNAME  STATUS   AVAILABILITY  MANAGER STATUS  ENGINE VERSION
2pfwllgxpajx5aitlvcih9vsq     mini01    Ready    Active                        17.09.0-ce
zho14u85itt5l2i6cpg8fcd6t     mini02    Ready    Active                        17.09.0-ce
yo5f7qb28gf6g38ve4xhcis17 *   mini03    Ready    Active        Leader          17.09.0-ce
```

5、退出当前的swarm节点
```
# 在swarm管理机mini03上的操作

# 其中 2pfwllgxpajx5aitlvcih9vsq 是mini01在swarm机器上的ID，根据docker node ls 获取
# docker node rm --force 2pfwllgxpajx5aitlvcih9vsq           # 如果mini01上的docker没有停止服务，那么就需要使用 --force 选项
2pfwllgxpajx5aitlvcih9vsq

# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
zho14u85itt5l2i6cpg8fcd6t     mini02              Ready               Active                                  17.09.0-ce
yo5f7qb28gf6g38ve4xhcis17 *   mini03              Ready               Active              Leader              17.09.0-ce
##########################################

# 需要在mini01上执行的命令，这样mini01才能彻底退出swarm管理
# docker swarm leave
Node left the swarm.
```

6、swarm管理机退出swarm
- 首先需要删除所有节点，然后强制退出swarm即可
```
# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
yo5f7qb28gf6g38ve4xhcis17 *   mini03              Ready               Active              Leader              17.09.0-ce

# docker swarm leave --force          # swarm管理机退出swarm，需要 --force 参数
Node left the swarm. 

# docker node ls
Error response from daemon: This node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
```

二、docker service
---
0、创建一个overlay网络  
```
# docker network create -d overlay demo

# 查看网络
# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
fa8a244c6bd5        bridge              bridge              local
51c95dea1e5c        docker_gwbridge     bridge              local
7a7e31f4bce8        host                host                local
5hgg372xwxbl        ingress             overlay             swarm
lmt3pjswf7l0        demo                overlay             swarm
5ea08e9a282f        none                null                local

# 查看网络信息 
# docker network inspect demo
[
    {
        "Name": "demo",
        "Id": "xiykborz8hn2td40ykhi20dck",
        "Created": "0001-01-01T00:00:00Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": null,
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": null
    }
]
```  
1、创建一个service  
```
#docker service create --name mysql --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE=wordpress --network demo -p 3306:3306 -d --mount type=volume,source=mysqldata,destination=/var/lib/mysql mysql
```  
2、查看创建的service  
``` 
# docker service ls 
ID                   NAME           MODE         REPLICAS           IMAGE             PORTS
xf2s18b0ib90         mysql        replicated         1/1           busybox:latest    
```  
3、查看service的详细情况  
```
# docker service ps demo
ID                NAME         IMAGE              NODE         DESIRED  STATE       CURRENT STATE          ERROR
b7feqwz2c029     mysql.1    busybox:latest    swarm-manager       Running          Running 2 minute ago
```  
4、水平扩展service  
```
# docker service scale mysql=5
demo scale to 5
overall progress: 5 out of 5 tasks
1/5: running
2/5: running
3/5: running
4/5: running
5/5: running
verify: Service converged
```  
5、删除service  
```
# docker service rm mysql
demo
```  

6、更新  
``` 
# docker service update --image mysql:v2.0 mysql
```

7、删除网络【慎用】
```
# docker network rm demo
demo

# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
fa8a244c6bd5        bridge              bridge              local
51c95dea1e5c        docker_gwbridge     bridge              local
7a7e31f4bce8        host                host                local
5hgg372xwxbl        ingress             overlay             swarm
5ea08e9a282f        none                null                local
```

三、swarm标签管理
---

1、标签添加

根据最开始的主机和组件部署规划，标签规划如下：在swarm管理机mini03上执行
```
# 给mini01机器的标签
docker node update --label-add tomcat=true mini01
docker node update --label-add datanode=true mini01
docker node update --label-add hbase-regionserver-1=true mini01

# 给mini02机器的标签
docker node update --label-add tomcat=true mini02
docker node update --label-add datanode=true mini02
docker node update --label-add hbase-regionserver-2=true mini02

# 给mini03机器的标签
docker node update --label-add spark=true mini03
docker node update --label-add zookeeper=true mini03
docker node update --label-add namenode=true mini03
docker node update --label-add hbase-master=true mini03
```

2、删除标签

在swarm管理机mini03上执行，示例如下：
```
docker node update --label-rm zookeeper mini03
```

3、查看swarm当前的标签

在swarm管理机mini03上执行：
```
[root@mini03 ~]# docker node ls -q | xargs docker node inspect -f '{{.ID}}[{{.Description.Hostname}}]:{{.Spec.Labels}}'
6f7dwt47y6qvgs3yc6l00nmjd[mini01]:map[tomcat:true datanode:true hbase-regionserver-1:true]
5q2nmm2xaexhkn20z8f8ezglr[mini02]:map[tomcat:true datanode:true hbase-regionserver-2:true]
ncppwjknhcwbegmliafut0718[mini03]:map[hbase-master:true namenode:true spark:true zookeeper:true]
```

四、查看日志
---
启动容器时，查看相关日志，例如如下：
```
docker stack ps hadoop
docker stack ps hadoop --format "{{.Name}}: {{.Error}}"
docker stack ps hadoop --format "{{.Name}}: {{.Error}}" --no-trunc
docker stack ps hadoop --no-trunc
```

五、docker stack
---
1、编写配置文件  
```
# cat docker-compose.yml 
version: '3'
services:
  web:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST:  mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-network
    depends_on:
      - mysql
    deploy:
      mode: replicated                            #副本模式
      replicas: 3                                       #有三个副本
      restart_policy:                                 #重启策略
        condition: on-failure                    #有失败就重启
        delay: 5s                                       #间隔时间
        max_attempts: 3                           #最多尝试次数
      update_config:                               #更新策略
        parallelism: 1                                #同一时间只能更新一个
        delay: 10s                                     #间隔时间
      resources:                                      #资源限制
        limits:                                            #限制最大多少资源
          cpus: '0.50'
          memory: 50M
        reservations:                                #启动保留多少资源
          cpus: '0.25'
          memory: 20M

  mysql:
    image: mysql
    enviroment:
      MYSQL_ROOT_PASSWORD:  root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks: 
      -my-network
    deplay:
      mode: global                                  #global只能有一个，不能做横向扩展
      placement:
        constraints:
          - node.role == manager           #运行在manager节点

  volumes:
    mysql-data:
  networks:
    my-network:
      driver: overlay
```  

2、启动stack文件  
```
# docker stack deploy wordpress --compose-file=docker-compose.yml
Creating  network  wordpress_my-network
Creating  service    wordpress_web
Creating  service    wordpress_mysql
```  

3、查看有接stack  
```
# # docker stack ls
NAME          SERVICES   
wordpress        2
```  

4、查看stack里跑了哪些服务和状态  
``` # docker stack ps wordpress ```  
 
5、查看stack的service  
```  # docker stack services wordpress ```  

6、删除stack  
``` # docker stack rm wordpress ```  

7、stack的横向扩展  
``` # docker service scale wordpress_web=5 ```  

8、更新和创建一样  
``` # docker stack deploy wordpress --compose-file=docker-compose.yml ```  


六、Routing Mesh
---
1、internal---Container和Container之间的访问通过overlay网络（通过VIP虚拟IP）  
2、ingress---如果服务有绑定接口，则此服务可通过任意swarm节点的相应接口访问  

internal方式  
```
创建一个需要被dns解析的容器
# docker service create --name whoami -p 8000:8000 --network demo -d jwilder/whoami
# curl 127.0.0.1:8000
I'm 3242961c028a

创建一个客户端进行dns解析
# docker service create --name client -d --network demo busybox sh -c "while true; do sleep 3600; done"

#进入客户端容器进行解析
# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                   PORTS
6u2kg07ltixb        client              replicated          1/1                 busybox:latest          
sxn7j1c5qwvl        whomai              replicated          1/1                 jwilder/whoami:latest   *:8000->8000/tcp

# docker service ps client
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
lrrkoa5rjrmd        client.1            busybox:latest      node02              Running             Running 5 minutes ago                    

# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS               NAMES
5e6c6436a096        busybox:latest          "sh -c 'while true; …"   7 minutes ago       Up 7 minutes                            client.1.lrrkoa5rjrmdolme6srsy5ti9
3242961c028a        jwilder/whoami:latest   "/app/http"              10 minutes ago      Up 10 minutes       8000/tcp            whomai.1.8y2ipblxkxzx6usutqkxifrxd

# docker exec -it 5e6c sh
/ # 

#ping mysql返回的地址为service地址
/ # ping whoami
PING whomai (10.0.0.5): 56 data bytes
64 bytes from 10.0.0.5: seq=0 ttl=64 time=0.085 ms
64 bytes from 10.0.0.5: seq=1 ttl=64 time=1.721 ms


#解析mysql地址，前加tasks才能解析到容器地址
/ # nslookup whoami
Server:		127.0.0.11
Address:	127.0.0.11:53

Non-authoritative answer:
Name:	whomai
Address: 10.0.0.7
Address: 10.0.0.15

*** Can't find whomai: No answer
```  

ingress方式  
```
查看工作节点防火墙规则
# iptables -nL -t nat
在有部署的whomai机器访问
# curl 127.0.0.1:8000
I'm 49c082b99daa
在没有部署的whomai机器访问
# curl 127.0.0.1:8000
I'm 49c082b99daa
发现都可以反问，是因为防火墙规则配置了转发功能，并且docker_gwbride和防火墙规则在一个网络里

进入ingress
#  ls /var/run/docker/netns/
1-bt8dce6g1o  1-pbatc0ldzr  3b3ff00afea8  70db15bf500b  d12e84ce9ba1  ingress_sbox
   
# nsenter --net=/var/run/docker/netns/ingress_sbox 
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
25: eth0@if26: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP 
    link/ether 02:42:0a:ff:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.255.0.2/16 brd 10.255.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 10.255.0.45/32 brd 10.255.0.45 scope global eth0
       valid_lft forever preferred_lft forever
28: eth1@if29: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:13:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 172.19.0.2/16 brd 172.19.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```  
