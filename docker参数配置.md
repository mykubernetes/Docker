

| 参数 | 介绍 |
|-----|-----|
| –api-enable-cors=false | 远程API调用。 |
| -b, --bridge="" | 桥接一个系统上的网桥设备到 Docker 容器里，当使用 none 可以停用容器里的网络 |
| –bip="" | 使用 CIDR 地址来设定网络桥的 IP。此参数和 -b 不能一起使用 |
| -D, --debug=false | 开启Debug模式。例如：docker -d -D |
| -d, --daemon=false | 开启Daemon模式。 |
| –dns=[] | 设置容器使用DNS服务器。例如： docker -d --dns 8.8.8.8 |
| -dns-search=[] | 设置容器使用指定的DNS搜索域名。如： docker -d --dns-search example.com |
| –exec-driver=“native” | 设置容器使用指定的运行时驱动。如：docker -d -e lxc |
| -G, --group=“docker” | 在后台运行模式下，赋予指定的Group到相应的unix socket上。注意，当此参数 --group 赋予空字符串时，将去除组信息 |
| -g, --graph="/var/lib/docker" | 设置Docker运行时根目录 |
| -H, --host=[] | 设置后台模式下指定socket绑定，可以绑定一个或多个 tcp://host:port, unix:///path/to/socket, fd://* 或 fd://socketfd。如：$ docker -H tcp://0.0.0.0:2375 ps 或者$ export DOCKER_HOST=“tcp://0.0.0.0:2375”$ docker ps |
| -icc=true | 设置启用内联容器的通信。 |
| –ip=“0.0.0.0” | 设置容器绑定IP时使用的默认IP地址 |
| –ip-forward=true | 设置启动容器的 net.ipv4.ip_forward |
| –iptables=true | 设置启动Docker容器自定义的iptable规则 |
| –mtu=0 | 设置容器网络的MTU值，如果没有这个参数，选用默认 route MTU，如果没有默认route，就设置成常量值 1500。 |
| -p, --pidfile="/var/run/docker.pid" | 设置后台进程PID文件路径。 |
| -r, --restart=true | 设置重启之前运行中的容器 |
| -s, --storage-driver="" | 设置容器运行时使用指定的存储驱动，如,指定使用devicemapper,可以这样：docker -d -s devicemapper |
| –selinux-enabled=false | 设置启用selinux支持 |
| –storage-opt=[] | 设置存储驱动的参数 |


# Docker 配置文件位置

Docker 的配置文件可以设置大部分的后台进程参数，在各个操作系统中的存放位置不一致
- 在 ubuntu 中的位置是：/etc/default/docker
- 在 centos6 中的位置是：/etc/sysconfig/docker
- 在 centos7 中的位置是：/etc/docker/

# Centos7更改Docker运行根目录配置：
```
/etc/docker/daemon.json
{
    "graph": "/app/docker"
}
```

# 其他参数参考
```
{
    "authorization-plugins": [],               #访问授权插件
    "data-root": "",                           #docker数据持久化存储的根目录
    "dns": [],                                 #DNS服务器
    "dns-opts": [],                            #DNS配置选项，如端口等
    "dns-search": [],                          #DNS搜索域名
    "exec-opts": [],                           #执行选项
    "exec-root": "",                           #执行状态的文件的根目录
    "experimental": false,                     #是否开启试验性特性
    "storage-driver": "",                      #存储驱动器
    "storage-opts": [],                        #存储选项
    "labels": [],                              #键值对式标记docker元数据
    "live-restore": true,                      #dockerd挂掉是否保活容器（避免了docker服务异常而造成容器退出）
    "log-driver": "",                          #容器日志的驱动器
    "log-opts": {},                            #容器日志的选项
    "mtu": 0,                                  #设置容器网络MTU（最大传输单元）
    "pidfile": "",                             #daemon PID文件的位置
    "cluster-store": "",                       #集群存储系统的URL
    "cluster-store-opts": {},                  #配置集群存储
    "cluster-advertise": "",                   #对外的地址名称
    "max-concurrent-downloads": 3,             #设置每个pull进程的最大并发
    "max-concurrent-uploads": 5,               #设置每个push进程的最大并发
    "default-shm-size": "64M",                 #设置默认共享内存的大小
    "shutdown-timeout": 15,                    #设置关闭的超时时限(who?)
    "debug": true,                             #开启调试模式
    "hosts": [],                               #监听地址(?)
    "log-level": "",                           #日志级别
    "tls": true,                               #开启传输层安全协议TLS
    "tlsverify": true,                         #开启输层安全协议并验证远程地址
    "tlscacert": "",                           #CA签名文件路径
    "tlscert": "",                             #TLS证书文件路径
    "tlskey": "",                              #TLS密钥文件路径
    "swarm-default-advertise-addr": "",        #swarm对外地址
    "api-cors-header": "",                     #设置CORS（跨域资源共享-Cross-origin resource sharing）头
    "selinux-enabled": false,                  #开启selinux(用户、进程、应用、文件的强制访问控制)
    "userns-remap": "",                        #给用户命名空间设置 用户/组
    "group": "",                               #docker所在组
    "cgroup-parent": "",                       #设置所有容器的cgroup的父类(?)
    "default-ulimits": {},                     #设置所有容器的ulimit
    "init": false,                             #容器执行初始化，来转发信号或控制(reap)进程
    "init-path": "/usr/libexec/docker-init",   #docker-init文件的路径
    "ipv6": false,                             #开启IPV6网络
    "iptables": false,                         #开启防火墙规则
    "ip-forward": false,                       #开启net.ipv4.ip_forward
    "ip-masq": false,                          #开启ip掩蔽(IP封包通过路由器或防火墙时重写源IP地址或目的IP地址的技术)
    "userland-proxy": false,                   #用户空间代理
    "userland-proxy-path": "/usr/libexec/docker-proxy",                 #用户空间代理路径
    "ip": "0.0.0.0",                           #默认IP
    "bridge": "",                              #将容器依附(attach)到桥接网络上的桥标识
    "bip": "",                                 #指定桥接ip
    "fixed-cidr": "",                          #(ipv4)子网划分，即限制ip地址分配范围，用以控制容器所属网段实现容器间(同一主机或不同主机间)的网络访问
    "fixed-cidr-v6": "",                       #(ipv6）子网划分
    "default-gateway": "",                     #默认网关
    "default-gateway-v6": "",                  #默认ipv6网关
    "icc": false,                              #容器间通信
    "raw-logs": false,                         #原始日志(无颜色、全时间戳)
    "allow-nondistributable-artifacts": [],    #不对外分发的产品提交的registry仓库
    "registry-mirrors": [],                    #registry仓库镜像
    "seccomp-profile": "",                     #seccomp配置文件
    "insecure-registries": [],                 #非https的registry地址
    "no-new-privileges": false,                #禁止新优先级(??)
    "default-runtime": "runc",                 #OCI联盟(The Open Container Initiative)默认运行时环境
    "oom-score-adjust": -500,                  #内存溢出被杀死的优先级(-1000~1000)
    "node-generic-resources": ["NVIDIA-GPU=UUID1", "NVIDIA-GPU=UUID2"],           #对外公布的资源节点
    "runtimes": {                              #运行时
        "cc-runtime": {
            "path": "/usr/bin/cc-runtime"
        },
        "custom": {
            "path": "/usr/local/bin/my-runc-replacement",
            "runtimeArgs": [
                "--debug"
            ]
        }
    }
}
```
