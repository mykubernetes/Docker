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

一、Docker Swarm 配置集群节点  
```
$ docker swarm init --advertise-addr 192.168.99.100
  Swarm initialized: current node (n0ub7dpn90rxjq97dr0g8we0w) is now a manager.
  To add a worker to this swarm, run the following command:
        docker swarm join --token SWMTKN-1-5uwpqibnvmho1png8zmhcw8274yanohee32jyrcjlait9djhsk-envtxo4dl6df2ar3qldcccfdg 192.168.99.100:2377
  To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```  
二、加入集群
加入Swarm集群工作节点  
```
docker swarm join --token SWMTKN-1-5uwpqibnvmho1png8zmhcw8274yanohee32jyrcjlait9djhsk-envtxo4dl6df2ar3qldcccfdg 192.168.99.100:2377
This node joined a swarm as a worker.
```  
加入Swarm集群管理节点  
```
$ docker swarm join-token manager
    To add a manager to this swarm, run the following command:
    docker swarm join --token SWMTKN-1-5uwpqibnvmho1png8zmhcw8274yanohee32jyrcjlait9djhsk-0koz1b98sco8r5cn3g61eahnu 192.168.99.100:2377

$ docker-machine ssh manager2 "docker swarm join --token SWMTKN-1-5uwpqibnvmho1png8zmhcw8274yanohee32jyrcjlait9djhsk-0koz1b98sco8r5cn3g61eahnu 192.168.99.100:2377"
This node joined a swarm as a manager.
```