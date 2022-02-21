统称来说，容器是一种工具，指的是可以装下其它物品的工具，以方便人类归纳放置物品、存储和异地运输，具体来说比如人类使用的衣柜、行李箱、背包等可以成为容器，但今天我们所说的容器是一种 IT 技术。

容器技术是虚拟化、云计算、大数据之后的一门新兴的并且是炙手可热的新技术，容器技术提高了硬件资源利用率、方便了企业的业务快速横向扩容、实现了业务宕机自愈功能，因此未来数年会是一个容器愈发流行的时代，这是一个对于 IT 行业来说非常有影响和价值的技术，而对于 IT 行业的从业者来说，熟练掌握容器技术无疑是一个很有前景的行业工作机会。

容器技术最早出现在 freebsd 叫做 jail

# 一、Docker 简介

## 1.1 Docker 简介

### 1.1.1 Docker 是什么

首先 Docker是一个在 2013年开源的应用程序并且是一个基于 go 语言编写是 一个开源的 PAAS 服务(Platform as a Service，平台即服务的缩写)，go 语言是由 google 开发，docker 公司最早叫 dotCloud 后由于 Docker 开源后大受欢迎 就将公司改名为 Docker Inc，总部位于美国加州的旧金山，Docker 是基于 linux内核实现，Docker 最早采用 LXC 技术(LinuX Container 的简写，LXC 是 Linux 原 生支持的容器技术，可以提供轻量级的虚拟化，可以说 docker 就是基于 LXC 发展起来的(0.1.5 (2013-04-17)，提供 LXC 的高级封装，发展标准的配置方法)， 而虚拟化技术 KVM(Kernel-based Virtual Machine) 基于模块实现，Docker 后改为自己研发并开源的 runc 技术运行容器(1.11.0 (2016-04-13)。

`Docker now relies on containerd and runc to spawn containers`

Docker 相比虚拟机的交付速度更快，资源消耗更低，Docker 采用客户端/服务端架构，使用远程 API 来管理和创建 Docker 容器，其可以轻松的创建一个轻量级的、可移植的、自给自足的容器，docker 的三大理念是 build(构建)、ship(运输)、 run(运行)，Docker 遵从 apache 2.0 协议，并通过（namespace 及 cgroup 等）来提供容器的资源隔离与安全保障等，所以 Docke 容器在运行时不需要类似虚拟机（空运行的虚拟机占用物理机的一定性能开销）的额外资源开销，因此可以大幅提高资源利用率,总而言之 Docker 是一种用了新颖方式实现的轻量级虚拟机。类似于 VM 但是在原理和应用上和 VM 的差别还是很大的，并且 docker 的专业叫法是应用容器(Application Container)。

### 1.1.2 Docker 的组成

https://docs.docker.com/engine/docker-overview/

- Docker 主机(Host)：一个物理机或虚拟机，用于运行 Docker 服务进程和容器。
- Docker 服务端(Server)：Docker 守护进程，运行 docker 容器。
- Docker 客户端(Client)：客户端使用 docker 命令或其他工具调用 docker API。
- Docker 仓库(Registry): 保存镜像的仓库，类似于 git 或 svn 这样的版本控制系统。
- Docker 镜像(Images)：镜像可以理解为创建实例使用的模板。
- Docker 容器(Container): 容器是从镜像生成对外提供服务的一个或一组服务。

官方仓库: https://hub.docker.com/




### 1.1.3 Docker 对比虚拟机

- **资源利用率更高：** 一台物理机可以运行数百个容器，但是一般只能运行数十个虚拟机。
- **开销更小：** 不需要启动单独的虚拟机占用硬件资源。
- **启动速度更快：** 可以在数秒内完成启动。



使用虚拟机是为了更好的实现服务运行环境隔离，每个虚拟机都有独立的内核， 虚拟化可以实现不同操作系统的虚拟机，但是通常一个虚拟机只运行一个服务， 很明显资源利用率比较低且造成不必要的性能损耗，我们创建虚拟机的目的是为了运行应用程序，比如 Nginx、PHP、Tomcat 等 web 程序，使用虚拟机无疑带来了一些不必要的资源开销，但是容器技术则基于减少中间运行环节带来较大的 性能提升。



但是，如上图一个宿主机运行了 N 个容器，多个容器带来的以下问题怎么解决:

1.怎么样保证每个容器都有不同的文件系统并且能互不影响？  
2.一个 docker 主进程内的各个容器都是其子进程，那么实现同一个主进程下不同类型的子进程？  
3.各个进程间通信能相互访问(内存数据)吗？  
4.每个容器怎么解决 IP 及端口分配的问题？  
5。多个容器的主机名能一样吗？  
6.每个容器都要不要有 root 用户？怎么解决账户重名问题？

以上问题怎么解决？

### 1.1.4 Linux Namespace 技术

namespace 是 Linux 系统的底层概念，在内核层实现，即有一些不同类型的命名空间被部署在核内，各个 docker 容器运行在同一个 docker 主进程并且共用同一个宿主机系统内核，各 docker 容器运行在宿主机的用户空间，每个容器都要有类似于虚拟机一样的相互隔离的运行空间，但是容器技术是在一个进程内实现运行指定服务的运行环境，并且还可以保护宿主机内核不受其他进程的干扰和影响，如文件系统空间、网络空间、进程空间等，目前主要通过以下技术实现容 器运行空间的相互隔离：

| 隔离类型 | 功能 | 系统调用参数 | 内核版本 |
|---------|------|-------------|----------|
| MNT Namespace(mount) | 提供磁盘挂载点和文件系统的隔离能力 | CLONE_NEWNS | Linux 2.4.19 |
| IPC Namespace(Inter-Process Communication) | 提供进程间通信的隔离能力 | CLONE_NEWIPC | Linux 2.6.19 |
| UTS Namespace(UNIX Timesharing System) | 提供主机名隔离能力 | CLONE_NEWUTS | Linux 2.6.19 |
| PID Namespace(Process Identification) | 提供进程隔离能力 | CLONE_NEWPID | Linux 2.6.24 |
| Net Namespace(network) | 提供网络隔离能力 | CLONE_NEWNET | Linux 2.6.29 |
| User Namespace(user) | 提供用户隔离能力 | CLONE_NEWUSER | Linux 3.8 |

#### 1.4.1.1 MNT Namespace
每个容器都要有独立的根文件系统有独立的用户空间，以实现在容器里面启动服务并且使用容器的运行环境，即一个宿主机是 ubuntu 的服务器，可以在里面启动一个 centos 运行环境的容器并且在容器里面启动一个 Nginx 服务，此 Nginx 运行时使用的运行环境就是 centos 系统目录的运行环境，但是在容器里面是不能访问宿主机的资源，宿主机是使用了 chroot 技术把容器锁定到一个指定的运行目录里面。

#### 1.4.1.2 IPC Namespace
一个容器内的进程间通信，允许一个容器内的不同进程的(内存、缓存等)数据访问，但是不能夸容器访问其他容器的数据。

#### 1.4.1.3 UTS Namespace
UTS namespace（UNIX Timesharing System 包含了运行内核的名称、版本、 底层体系结构类型等信息）用于系统标识，其中包含了 hostname 和域名 domainname ，它使得一个容器拥有属于自己 hostname 标识，这个主机名标识独立于宿主机系统和其上的其他容器。

#### 1.4.1.4 PID Namespac
Linux 系统中，有一个 PID 为 1 的进程(init/systemd)是其他所有进程的父进程，那么在每个容器内 也要有一个父进程来管理其下属的子进程，那么多个容器的进程通过 PID namespace 进程隔离(比如 PID 编号重复、容器内的主进程生成与回收子进程等)。

#### 1.4.1.5 Net Namespace
每一个容器都类似于虚拟机一样有自己的网卡、监听端口、TCP/IP 协议栈等， Docker 使用 network namespace 启动一个 vethX 接口，这样你的容器将拥有它自己的桥接 ip 地址，通常是 docker0，而 docker0 实质就是 Linux 的虚拟网桥, 网桥是在 OSI 七层模型的数据链路层的网络设备，通过 mac 地址对网络进行划分，并且在不同网络直接传递数据。

#### 1.4.1.6 User Names
各个容器内可能会出现重名的用户和用户组名称，或重复的用户 UID 或者 GID， 那么怎么隔离各个容器内的用户空间呢？

User Namespace 允许在各个宿主机的各个容器空间内创建相同的用户名以及相同的用户 UID 和 GID，只是会把用户的作用范围限制在每个容器内，即 A 容 器和 B 容器可以有相同的用户名称和 ID 的账户，但是此用户的有效范围仅是当前容器内，不能访问另外一个容器内的文件系统，即相互隔离、互补影响、永不相见。

### 1.1.5 Linux control groups
在一个容器，如果不对其做任何资源限制，则宿主机会允许其占用无限大的内存空间，有时候会因为代码 bug 程序会一直申请内存，直到把宿主机内存占完， 为了避免此类的问题出现，宿主机有必要对容器进行资源分配限制，比如 CPU、 内存等，Linux Cgroups 的全称是 Linux Control Groups，它最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。此外，还能够对进程进行优先级设置，以及将进程挂起和恢复等操作。

#### 1.1.5.1 验证系统 cgroups

Cgroups 在内核层默认已经开启，从 centos 和 ubuntu 对比结果来看，显然内 核较新的 ubuntu 支持的功能更多

#### Centos cgroups
```
[19:57:18 root@centos7 ~]#uname -r
3.10.0-1127.el7.x86_64
[19:58:43 root@centos7 ~]#cat /boot/config-3.10.0-1127.el7.x86_64 | grep CGROUP
CONFIG_CGROUPS=y
# CONFIG_CGROUP_DEBUG is not set
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_SCHED=y
CONFIG_BLK_CGROUP=y
# CONFIG_DEBUG_BLK_CGROUP is not set
CONFIG_NETFILTER_XT_MATCH_CGROUP=m
CONFIG_NET_CLS_CGROUP=y
CONFIG_NETPRIO_CGROUP=y
```

#### ubuntu cgroups
```
[19:57:12 root@ubuntu18-04 ~]#uname -r
4.15.0-76-generic
[19:59:30 root@ubuntu18-04 ~]#cat /boot/config-4.15.0-76-generic | grep CGROUP
CONFIG_CGROUPS=y
CONFIG_BLK_CGROUP=y
# CONFIG_DEBUG_BLK_CGROUP is not set
CONFIG_CGROUP_WRITEBACK=y
CONFIG_CGROUP_SCHED=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_RDMA=y
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_BPF=y
# CONFIG_CGROUP_DEBUG is not set
CONFIG_SOCK_CGROUP_DATA=y
CONFIG_NETFILTER_XT_MATCH_CGROUP=m
CONFIG_NET_CLS_CGROUP=m
CONFIG_CGROUP_NET_PRIO=y
CONFIG_CGROUP_NET_CLASSID=y
```

#### 1.1.5.2 cgroups 中内存模块
```
[19:59:50 root@ubuntu18-04 ~]#cat /boot/config-4.15.0-76-generic | grep MEM | grep CG
CONFIG_MEMCG=y
CONFIG_MEMCG_SWAP=y
# CONFIG_MEMCG_SWAP_ENABLED is not set
CONFIG_SLUB_MEMCG_SYSFS_ON=y
```

#### 1.1.5.3 cgroups 具体实现
```
blkio：块设备 IO 限制。
cpu：使用调度程序为 cgroup 任务提供 cpu 的访问。
cpuacct：产生 cgroup 任务的 cpu 资源报告。
cpuset：如果是多核心的 cpu，这个子系统会为 cgroup 任务分配单独的 cpu和内存。
devices：允许或拒绝 cgroup 任务对设备的访问。
freezer：暂停和恢复 cgroup 任务。
memory：设置每个 cgroup 的内存限制以及产生内存资源报告。
net_cls：标记每个网络包以供 cgroup 方便使用。
ns：命名空间子系统。
perf_event：增加了对每 group 的监测跟踪的能力，可以监测属于某个特定的
group 的所有线程以及运行在特定 CPU 上的线程。
```

#### 1.1.5.4 查看系统 cgroups
```
[20:00:55 root@ubuntu18-04 ~]#ll /sys/fs/cgroup/
total 0
drwxr-xr-x 15 root root 380 Apr 12 19:56 ./
drwxr-xr-x  9 root root   0 Apr 12 19:56 ../
dr-xr-xr-x  4 root root   0 Apr 12 19:56 blkio/
lrwxrwxrwx  1 root root  11 Apr 12 19:56 cpu -> cpu,cpuacct/
lrwxrwxrwx  1 root root  11 Apr 12 19:56 cpuacct -> cpu,cpuacct/
dr-xr-xr-x  4 root root   0 Apr 12 19:56 cpu,cpuacct/
dr-xr-xr-x  2 root root   0 Apr 12 19:56 cpuset/
dr-xr-xr-x  4 root root   0 Apr 12 19:56 devices/
dr-xr-xr-x  2 root root   0 Apr 12 19:56 freezer/
dr-xr-xr-x  2 root root   0 Apr 12 19:56 hugetlb/
dr-xr-xr-x  4 root root   0 Apr 12 19:56 memory/
lrwxrwxrwx  1 root root  16 Apr 12 19:56 net_cls -> net_cls,net_prio/
dr-xr-xr-x  2 root root   0 Apr 12 19:56 net_cls,net_prio/
lrwxrwxrwx  1 root root  16 Apr 12 19:56 net_prio -> net_cls,net_prio/
dr-xr-xr-x  2 root root   0 Apr 12 19:56 perf_event/
dr-xr-xr-x  4 root root   0 Apr 12 19:56 pids/
dr-xr-xr-x  2 root root   0 Apr 12 19:56 rdma/
dr-xr-xr-x  5 root root   0 Apr 12 19:56 systemd/
dr-xr-xr-x  5 root root   0 Apr 12 19:56 unified/
```
有了以上的 chroot、namespace、cgroups 就具备了基础的容器运行环境，但是还需要有相应的容器创建与删除的管理工具、以及怎么样把容器运行起来、容器数据怎么处理、怎么进行启动与关闭等问题需要解决，于是容器管理技术出现了。

### 1.1.6 容器管理工具

目前主要是使用 docker，早期有使用 lxc。

#### 1.1.6.1 lxc

LXC：LXC 为 Linux Container 的简写。可以提供轻量级的虚拟化，以便隔离进程和资源，官方网站：https://linuxcontainers.org/

lxc 启动容器依赖于模板，清华模板源：

https://mirrors.tuna.tsinghua.edu.cn/help/lxc-images/，但是做模板相对较难， 需要手动一步步创构建文件系统、准备基础目录及可执行程序等，而且在大规模使用容器的场景很难横向扩展，另外后期代码升级也需要重新从头构建模板，基于以上种种原因便有了docker。

#### 1.1.6.2 docker

Docker 启动一个容器也需要一个外部模板但是较多镜像，docke 的镜像可以保 存在一个公共的地方共享使用，只要把镜像下载下来就可以使用，最主要的是可 以在镜像基础之上做自定义配置并且可以再把其提交为一个镜像，一个镜像可以被启动为多个容器。

Docker 的镜像是分层的，镜像底层为库文件且只读层即不能写入也不能删除数据，从镜像加载启动为一个容器后会生成一个可写层，其写入的数据会复制到容器目录，但是容器内的数据在删除容器后也会被随之删除。



#### 1.1.6.3 pouch

一个阿里巴巴自研以及开源的容器技术

https://www.infoq.cn/article/alibaba-pouch

https://github.com/alibaba/pouch

### 1.1.7 Docker 的优势

- **快速部署：** 短时间内可以部署成百上千个应用，更快速交付到线上。
- **高效虚拟化：** 不需要额外的 hypervisor 支持，直接基于 linux 实现应用虚拟化， 相比虚拟机大幅提高性能和效率。
- **节省开支：** 提高服务器利用率，降低 IT 支出。
- **简化配置：** 将运行环境打包保存至容器，使用时直接启动即可。
- **快速迁移和扩展：** 可夸平台运行在物理机、虚拟机、公有云等环境，良好的兼容 性可以方便将应用从 A 宿主机迁移到 B 宿主机，甚至是 A 平台迁移到 B 平台。

### 1.1.8：Docker 的缺点

隔离性：各应用之间的隔离不如虚拟机彻底。

### 1.1.9 docker(容器)的核心技术

#### 1.1.9.1 容器规范
容器技术除了的 docker 之外，还有 coreOS 的 rkt，还有阿里的 Pouch，为了保证容器生态的标准性和健康可持续发展，包括 Linux 基金会、Docker、微软、 红帽谷歌和、IBM、等公司在 2015 年 6 月共同成立了一个叫 open container （OCI）的组织，其目的就是制定开放的标准的容器规范，目前 OCI 一共发布了两个规范，分别是 runtime spec 和 image format spec，有了这两个规范，不同的容器公司开发的容器只要兼容这两个规范，就可以保证容器的可移植性和相互可操作性。

##### 1.1.9.1.1 容器 runtime(runtime spec)
runtime 是真正运行容器的地方，因此为了运行不同的容器 runtime 需要和操作系统内核紧密合作相互支持，以便为容器提供相应的运行环境。

### 目前主流的三种 runtime：

- **Lxc：** linux 上早期的 runtime，Docker 早期就是采用 lxc 作为 runtime。
- **runc：** 目前 Docker 默认的 runtime，runc 遵守 OCI 规范，因此可以兼容 lxc。
- **rkt：** 是 CoreOS 开发的容器 runtime，也符合 OCI 规范，所以使用 rktruntime 也可以运行 Docker 容器

runtime 主要定义了以下规范 ， 并以 json 格 式 保 存 在 /run/docker/runtime-runc/moby/容器 ID/state.json 文件，此文件会根据容器的状态实时更新内容：
```
版本信息：存放 OCI 标准的具体版本号。
容器 ID：通常是一个哈希值，可以在所有 state.json 文件中提取出容器 ID 对容器进行批量操作(关闭、删除等)，此文件在容器关闭后会被删除，容器启动后会自动生成。

PID：在容器中运行的首个进程在宿主机上的进程号，即将宿主机的那个进程设置为容器的守护进程。

容器文件目录：存放容器 rootfs 及相应配置的目录，外部程序只需读取 state.json 就可以定位到宿主机上的容器文件目录。

容器创建：创建包括文件系统、namespaces、cgroups、用户权限在内的各项内容。
容器进程的启动：运行容器启动进程，该文件在
/run/containerd/io.containerd.runtime.v1.linux/moby/容器 ID/config.json。

容器生命周期：容器进程可以被外部程序关停，runtime 规范定义了对容器操作信号的捕获，并做相应资源回收的处理，避免僵尸进程的出现。
```

##### 1.1.9.1.2 容器镜像(image format spec）
OCI 容器镜像主要包含以下内容：
```
文件系统:定义以 layer 保存的文件系统,在镜像里面是 layer.tar,每个 layer 保存了和上层之间变化的部分,image format spec 定义了 layer 应该保存哪些文件,怎么表示增加、修改和删除的文件等操作。

manifest文件：描述有哪些 layer，tag 标签及 config 文件名称。
config文件：是一个以 hash 命名的 json 文件，保存了镜像平台，容器运行时容器运行时需要的一些信息，比如环境变量、工作目录、命令参数等。
index 文件：可选的文件,指向不同平台的 manifest 文件,这个文件能保证一个镜像可以跨平台使用，每个平台拥有不同的 manifest 文件使用 index 作为索引

父镜像：大多数层的元信息结构都包含一个 parent 字段，指向该镜像的父镜

参数：
ID：镜像 ID，每一层都有 ID
tag 标签：标签用于将用户指定的、具有描述性的名称对应到镜像ID
仓库：Repository 镜像仓库
os：定义类型
architecture ：定义 CPU 架构
author：作者信息
create：镜像创建日期
```

#### 1.1.9.2 容器管理工具
管理工具连接 runtime 与用户，对用户提供图形或命令方式操作，然后管理工具将用户操作传递给 runtime 执行。

- xc 是 lxd 的管理工具。
- Runc 的管理工具是 docker engine，docker engine 包含后台 deamon 和 cli 两部分，大家经常提到的 Docker 就是指的 docker engine。
- Rkt 的管理工具是 rkt cl

#### 1.1.9.3 容器定义工具
容器定义工具允许用户定义容器的属性和内容，以方便容器能够被保存、共享和重建。

Docker image：是 docker 容器的模板，runtime 依据 docker image 创建容器。

Dockerfile：包含 N 个命令的文本文件，通过 dockerfile 创建出 docker image。

ACI(App container image)：与 docker image 类似，是 CoreOS 开发的 rkt 容器 的镜像格式。

#### 1.1.9.4 Registry
统一保存镜像而且是多个不同镜像版本的地方，叫做镜像仓库。

- Image registry：docker 官方提供的私有仓库部署工具。
- Docker hub：docker 官方的公共仓库，已经保存了大量的常用镜像，可以方便大家直接使用。
- Harbor：vmware 提供的自带 web 界面自带认证功能的镜像仓库，目前有很多公司使用。

### 1.1.9.5 编排工具
当多个容器在多个主机运行的时候，单独管理容器是相当复杂而且很容易出错，而且也无法实现某一台主机宕机后容器自动迁移到其他主机从而实现高可用的目的，也无法实现动态伸缩的功能，因此需要有一种工具可以实现统一管理、 动态伸缩、故障自愈、批量执行等功能，这就是容器编排引擎。

容器编排通常包括容器管理、调度、集群定义和服务发现等功能。

- Docker swarm：docker 开发的容器编排引擎。
- Kubernetes：google 领导开发的容器编排引擎，内部项目为 Borg，且其同时支持 docker 和 CoreOS。
- Mesos+Marathon：通用的集群组员调度平台，mesos(资源分配)与 marathon(容器编排平台)一起提供容器编排引擎功能。

### 1.1.10 docker(容器)的依赖技术
**容器网络**

docker 自带的网络 docker network 仅支持管理单机上的容器网络，当多主机运行的时候需要使用第三方开源网络，例如 calico、flannel 等。

**服务发现**

容器的动态扩容特性决定了容器 IP 也会随之变化，因此需要有一种机制可以自动识别并将用户请求动态转发到新创建的容器上，kubernetes 自带服务发现功能，需要结合 kube-dns 服务解析内部域名。

**容器监控**

可以通过原生命令 docker ps/top/stats 查看容器运行状态，另外也可以使 heapster/ Prometheus 等第三方监控工具监控容器的运行状态。

**数据管理**

容器的动态迁移会导致其在不同的 Host 之间迁移，因此如何保证与容器相关的数据也能随之迁移或随时访问，可以使用逻辑卷/存储挂载等方式解决。

**日志收集**

docker 原生的日志查看工具 docker logs，但是容器内部的日志需要通过 ELK 等专门的日志收集分析和展示工具进行处理。

## 1.2 Docker 安装及基础命令介绍

官方网址：https://www.docker.com/

**系统版本选择**

Docker 目前已经支持多种操作系统的安装运行，比如 Ubuntu、CentOS、 Redhat、Debian、Fedora，甚至是还支持了 Mac 和 Windows，在 linux 系统上需要内核版本在 3.10 或以上，docker 版本号之前一直是 0.X 版本或 1.X 版本， 但是从 2017 年 3 月 1 号开始改为每个季度发布一次稳版，其版本号规则也统一 变更为 YY.MM，例如 17.09 表示是 2017 年 9 月份发布的。

**Docker 版本选择**

Docker 之前没有区分版本，但是 2017 年初推出(将 docker 更名为)新的项 目 Moby，github 地址：https://github.com/moby/moby，Moby 项目属于 Docker 项目的全新上游，Docker 将是一个隶属于的 Moby 的子产品，而且之后的版本之后开始区分为 CE 版本（社区版本）和 EE（企业收费版），CE 社区版本和 EE 企业版本都是每个季度发布一个新版本，但是 EE 版本提供后期安全维护 1 年，而 CE 版本是 4 个月，本次演示的 Docker 版本为 18.03。

### 1.2.1 下载 rpm 包安装

官方 rpm 包下载地址
- https://download.docker.com/linux/centos/7/x86_64/stable/Packages/

二进制下载地址
- https://download.docker.com/
- https://mirrors.aliyun.com/docker-ce/linux/static/stable/x86_64/

阿里镜像下载地址
- https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/

### 1.2.2 通过修改 yum 源安装
```
[11:16:23 root@centos7 ~]#wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[11:16:50 root@centos7 ~]#wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[11:17:14 root@centos7 ~]#wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[11:17:40 root@centos7 ~]#yum install docker-ce
[11:21:23 root@centos7 ~]#yum info docker-ce
Name        : docker-ce
Arch        : x86_64
Epoch       : 3
Version     : 20.10.6
Release     : 3.el7
```

### 1.2.3 ubuntu 安装 docker、启动并验证服务
```
清华大学源示例：https://mirror.tuna.tsinghua.edu.cn/help/docker-ce/

#如果你过去安装过 docker，先删掉
[11:32:52 root@ubuntu18-04 ~]#sudo apt-get remove docker docker-engine docker.io

#首先安装依赖
[11:35:24 root@ubuntu18-04 ~]#sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common

#信任 Docker 的 GPG 公钥
[11:35:58 root@ubuntu18-04 ~]#curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

#对于 amd64 架构的计算机，添加软件仓库
[11:36:02 root@ubuntu18-04 ~]#sudo add-apt-repository \
>    "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \>    $(lsb_release -cs) \
>    stable"

#最后安装
[11:38:26 root@ubuntu18-04 ~]#apt-cache madison docker-ce   #查看docker-ce所有版本
[11:38:29 root@ubuntu18-04 ~]#apt-get install docker-ce=5:18.09.9~3-0~ubuntu-bionic #选择版本安装

#启动
[11:45:13 root@ubuntu18-04 ~]#systemctl enable --now docker

#验证服务是否启动
[11:49:48 root@ubuntu18-04 ~]#systemctl is-active docker
active
[11:50:20 root@ubuntu18-04 ~]#systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabl
   Active: active (running) since Tue 2021-04-13 11:45:08 CST; 5min ago
```

### 1.2.4 二进制安装
```
#下载tar包
[18:53:14 root@ubuntu18-04 ~]#wget https://mirrors.aliyun.com/docker-ce/linux/static/stable/x86_64/docker-19.03.15.tgz
#解压
[18:54:09 root@ubuntu18-04 ~]#tar xf docker-19.03.15.tgz
#拷贝命令
[18:54:53 root@ubuntu18-04 ~]#cp docker/* /usr/bin/
#创建containerd的service文件
[19:06:10 root@ubuntu18-04 ~]#vim /lib/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/bin/containerd
Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=1048576
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
#启动containerd
[19:10:32 root@ubuntu18-04 ~]#systemctl enable --now containerd.service 

#准备docker的service文件
[19:06:10 root@ubuntu18-04 ~]#vim /lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket containerd.service

[Service]
Type=notify
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target
#准备docker的socket文件
[19:11:11 root@ubuntu18-04 ~]#vim /lib/systemd/system/docker.socket
[Unit]
Description=Docker Socket for the API

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
#创建docker组
[19:17:03 root@ubuntu18-04 ~]#groupadd docker
#启动docker
[19:17:43 root@ubuntu18-04 ~]#systemctl start docker.socket 
[19:17:46 root@ubuntu18-04 ~]#systemctl start docker.service
```

### 1.2.5 验证 docker 版本
```
[11:52:16 root@ubuntu18-04 ~]#docker version 
Client: Docker Engine - Community  #客户端工具
 Version:           20.10.6
 API version:       1.39
 Go version:        go1.13.15
 Git commit:        370c289
 Built:             Fri Apr  9 22:46:01 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community #服务端
 Engine:
  Version:          18.09.9
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.11.13
  Git commit:       039a7df
  Built:            Wed Sep  4 16:19:38 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```

### 1.2.6 验证 docker0 网卡
在 docker 安装启动之后，默认会生成一个名称为 docker0 的网卡并且默认 IP 地址为 172.17.0.1 的网卡
```
[11:52:21 root@ubuntu18-04 ~]#ifconfig 
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:50:24:b6:35  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 1.2.7 验证 docker
```
[11:54:01 root@ubuntu18-04 ~]#docker info
Server:
 Containers: 0 #当前主机运行的容器总数
  Running: 0   #有几个容器是正在运行的
  Paused: 0   #有几个容器是暂停的
  Stopped: 0  #有几个容器是停止的
 Images: 0    #当前服务器的镜像数
 Server Version: 18.09.9  #服务端版本
 Storage Driver: overlay2 #正在使用的存储引擎
  Backing Filesystem: extfs #后端文件系统，即服务器的磁盘文件系统
  Supports d_type: true     #是否支持 d_type
  Native Overlay Diff: true #是否支持差异数据存储
 Logging Driver: json-file  #日志类型
 Cgroup Driver: cgroupfs    #Cgroups类型
 Plugins:  #插件
  Volume: local #卷
  Network: bridge host macvlan null overlay  #overlay跨主机通信
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog #日志类型
 Swarm: inactive #是否支持 swarm
 Runtimes: runc  #已安装的容器运行时
 Default Runtime: runc #默认使用的容器运行时
 Init Binary: docker-init  #初始化容器的守护进程，即 pid 为 1 的进程
 containerd version: 05f951a3781f4f2c1911b05e61c160e9c30eaa8e #版本
 runc version: N/A     # runc 版本
 init version: fec3683  #init 版本
 Security Options: #安全选项
  apparmor
  seccomp
   Profile: default  #默认的配置文件
 Kernel Version: 4.15.0-76-generic #宿主机内核版本
 Operating System: Ubuntu 18.04.4 LTS #宿主机操作系统
 OSType: linux       #宿主机操作系统类型
 Architecture: x86_64  #宿主机架构
 CPUs: 2      #宿主机 CPU 数量
 Total Memory: 962.2MiB  #宿主机总内存
 Name: ubuntu18-04   #宿主机 hostname
 ID: VMAK:UWKC:KCRT:FRR7:3FSF:XKJK:UX2L:O2I2:PYRJ:2IQZ:QBDR:ZEDJ  #宿主机ID
 Docker Root Dir: /var/lib/docker  #宿主机数据保存目录
 Debug Mode: false    #是否开启 debu
 Registry: https://index.docker.io/v1/  #镜像仓库
 Labels:  #其他标签
 Experimental: false  #是否测试版
 Insecure Registries:  #非安全的镜像仓库
  127.0.0.0/8
 Live Restore Enabled: false  #是否开启活动重启(重启 docker-daemon 不关闭容器)
 Product License: Community Engine   #产品许可信息

WARNING: No swap limit support  #系统警告信息(没有开启 swap 资源限制)
```

### 1.2.8 解决不支持 swap 限制警告
```
[13:39:41 root@ubuntu18-04 ~]#vim /etc/default/grub
#在GRUB_CMDLINE_LINUX中追加cgroup_enable=memory swapaccount=1
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0 cgroup_enable=memory swapaccount=1"
[13:39:49 root@ubuntu18-04 ~]#update-grub
[13:40:34 root@ubuntu18-04 ~]#reboot
```

### 1.2.9 docker 存储引擎
目前 docker 的默认存储引擎为 overlay2，不同的存储引擎需要相应的系统支持， 如需要磁盘分区的时候传递 d-type 文件分层功能，即需要传递内核参数开启格 式化磁盘的时候的指定功能。

**历史更新信息：** https://github.com/moby/moby/blob/master/CHANGELOG.md

**官方文档关于存储引擎的选择文档：** https://docs.docker.com/storage/storagedriver/select-storage-driver/

**存储驱动类型：**
```
AUFS（AnotherUnionFS）是一种 Union FS，是文件级的存储驱动。所谓 UnionFS就是把不同物理位置的目录合并 mount 到同一个目录中。简单来说就是支持将不同目录挂载到同一个虚拟文件系统下的文件系统。这种文件系统可以一层一层地叠加修改文件。无论底下有多少层都是只读的，只有最上层的文件系统是可写的。当需要修改一个文件时，AUFS 创建该文件的一个副本，使用 CoW 将文件从只读层复制到可写层进行修改，结果也保存在可写层。在 Docker 中，底下的只读层就是 image，可写层就是Container，是 Docker 18.06 及更早版本的首选存储驱动程序，在内核 3.13 上运行 Ubuntu 14.04 时不支持overlay2。

Overlay：一种 Union FS 文件系统，Linux 内核 3.18 后支持。

#overlay2: Overlay 的升级版，到目前为止，所有 Linux 发行版推荐使用的存储类型。

devicemapper：是 CentOS 和 RHEL 的推荐存储驱动程序，因为之前的内核版本不支持 overlay2，但是当前较新版本的 CentOS 和 RHEL 现在已经支持overlay2，因此推荐使用 overlay2。

ZFS(Sun-2005)/btrfs(Oracle-2007)：目前没有广泛使用。

vfs：用于测试环境，适用于无法使用 copy-on-write 文件系统的情况。 此存储驱动程序的性能很差，通常不建议用于生产。
```

Docker 官方推荐首选存储引擎为 overlay2，devicemapper 存在使用空间方面的 一些限制，虽然可以通过后期配置解决，但是官方依然推荐使用 overlay2，以下是网上查到的部分资料：
```
[11:02:24 root@centos8 ~]#xfs_info /
meta-data=/dev/mapper/cl-root    isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1 #这里需要是1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
如果 docker 数据目录是一块单独的磁盘分区而且是 xfs 格式的，那么需要在格 式化的时候加上参数-n ftype=1，否则后期在启动容器的时候会报错不支持 d-type。

### 1.2.10 docker 服务进程
通过查看 docker 进程，了解 docker 的运行及工作方式

#### 1.2.10.1 查看宿主机进程树
```
[14:12:36 root@ubuntu18-04 ~]#pstree -p 1
systemd(1)─┬─VGAuthService(649)
           ├─accounts-daemon(958)─┬─{accounts-daemon}(981)
           │                      └─{accounts-daemon}(1022)
           ├─agetty(1035)
           ├─atd(888)
           ├─blkmapd(538)
           ├─chronyd(1034)
           ├─containerd(965)─┬─containerd-shim(2180)─┬─nginx(2207)───nginx(2277)
           │                 │                       ├─{containerd-shim}(2181)
           │                 │                       ├─{containerd-shim}(2182)
           │                 │                       ├─{containerd-shim}(2183)
           │                 │                       ├─{containerd-shim}(2184)
           │                 │                       ├─{containerd-shim}(2185)
           │                 │                       ├─{containerd-shim}(2186)
           │                 │                       ├─{containerd-shim}(2187)
           │                 │                       ├─{containerd-shim}(2188)
           │                 │                       └─{containerd-shim}(2243)
           │                 ├─{containerd}(1095)
           │                 ├─{containerd}(1096)
           │                 ├─{containerd}(1097)
           │                 ├─{containerd}(1098)
           │                 ├─{containerd}(1099)
           │                 ├─{containerd}(1123)
           │                 ├─{containerd}(1126)
           │                 ├─{containerd}(1167)
           │                 ├─{containerd}(1176)
           │                 └─{containerd}(1180)
           ├─cron(919)
           ├─dbus-daemon(891)
           ├─dockerd(1182)─┬─docker-proxy(2173)─┬─{docker-proxy}(2174)
           │               │                    ├─{docker-proxy}(2175)
           │               │                    ├─{docker-proxy}(2176)
           │               │                    ├─{docker-proxy}(2177)
           │               │                    ├─{docker-proxy}(2178)
           │               │                    └─{docker-proxy}(2179)
           │               ├─{dockerd}(1216)
           │               ├─{dockerd}(1217)
           │               ├─{dockerd}(1218)
           │               ├─{dockerd}(1220)
           │               ├─{dockerd}(1229)
           │               ├─{dockerd}(1242)
           │               ├─{dockerd}(1258)
           │               ├─{dockerd}(1259)
           │               ├─{dockerd}(1267)
           │               ├─{dockerd}(1849)
           │               ├─{dockerd}(1856)
           │               └─{dockerd}(1857)
           ├─irqbalance(931)───{irqbalance}(935)
           ├─lvmetad(515)
           ├─lxcfs(932)─┬─{lxcfs}(971)
           │            └─{lxcfs}(972)
           ├─networkd-dispat(886)───{networkd-dispat}(1132)
           ├─polkitd(1038)─┬─{polkitd}(1074)
           │               └─{polkitd}(1079)
           ├─rpc.idmapd(569)
           ├─rpc.mountd(769)
           ├─rpcbind(617)
           ├─rsyslogd(934)─┬─{rsyslogd}(973)
           │               ├─{rsyslogd}(974)
           │               └─{rsyslogd}(975)
           ├─snapd(964)─┬─{snapd}(1211)
           │            ├─{snapd}(1213)
           │            ├─{snapd}(1214)
           │            ├─{snapd}(1215)
           │            ├─{snapd}(1243)
           │            ├─{snapd}(1247)
           │            ├─{snapd}(1248)
           │            ├─{snapd}(1308)
           │            ├─{snapd}(1309)
           │            ├─{snapd}(1350)
           │            └─{snapd}(1351)
           ├─sshd(1001)─┬─sshd(1088)───bash(1504)───pstree(2279)
           │            └─sshd(1505)───sftp-server(1715)
           ├─systemd(1136)───(sd-pam)(1145)
           ├─systemd-journal(511)
           ├─systemd-logind(913)
           ├─systemd-network(764)
           ├─systemd-resolve(765)
           ├─systemd-udevd(530)
           ├─unattended-upgr(967)───{unattended-upgr}(1120)
           └─vmtoolsd(650)───{vmtoolsd}(658)
```

18.06 及之前的 docker 版本，进程关系
```

```

#### 1.2.10.2 查看 containerd 进程关系

有四个进程：
- **dockerd：** 被 client 直接访问，其父进程为宿主机的 systemd 守护进程。
- **docker-proxy：** 实现容器通信，其父进程为 dockerd
- **containerd：** 被 dockerd 进程调用以实现与 runc 交互。
- **containerd-shim：** 真正运行容器的载体，其父进程为 containerd。

```
[14:49:27 root@ubuntu18-04 ~]#ps -ef | grep containerd
root        965      1  0 13:41 ?        00:00:01 /usr/bin/containerd
root       1182      1  0 13:41 ?        00:00:05 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root       2180    965  0 14:12 ?        00:00:00 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/d58046a1ec859766cef2cac45efaa74fd7832318366eea0dd6fd43f60064ee6f -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc

[14:49:33 root@ubuntu18-04 ~]#ps -ef | grep docker-proxy
root       2173   1182  0 14:12 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 80 -container-ip 172.17.0.2 -container-port 80
```

#### 1.2.10.3 containerd-shim 命令使用
```
[14:50:35 root@ubuntu18-04 ~]#containerd-shim -h
Usage of containerd-shim:
  -address string
    	grpc address back to main containerd
  -containerd-binary containerd publish
    	path to containerd binary (used for containerd publish) (default "containerd")
  -criu string
    	path to criu binary
  -debug
    	enable debug output in logs
  -namespace string
    	namespace that owns the shim
  -runtime-root string
    	root directory for the runtime (default "/run/containerd/runc")
  -socket string
    	socket path to serve
  -systemd-cgroup
    	set runtime to use systemd-cgroup
  -workdir string
    	path used to storge large temporary data
```

#### 1.2.10.4 容器的创建与管理过程
```
通信流程：
1.dockerd 通过 grpc 和 containerd 模块通信(runc)交换，dockerd 和 containerd通信的 socket文件：/run/containerd/containerd.sock。

2. containerd 在 dockerd 启动时被启动，然后 containerd 启动 grpc 请求监听，containerd 处理 grpc 请求，根据请求做相应动作。/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

3. 若是创建容器，containerd 拉起一个 container-shim 容器进程 , 并进行相应的创建操作。

4. container-shim 被拉起后，start/exec/create 拉起 runC 进程，通过 exit、control文件和 containerd 通信，通过父子进程关系和 SIGCHLD(信号)监控容器中进程状态。

5. 在整个容器生命周期中，containerd 通过 epoll 监控容器文件，监控容器事件。
```

#### 1.2.10.5 grpc 简介

gRPC 是 Google 开发的一款高性能、开源和通用的 RPC 框架，支持众多语言客户端。

https://www.grpc.io/



## 1.3 docker 镜像加速

国内下载国外的镜像有时候会很慢，因此可以更改 docker 配置文件添加一个加 速器，可以通过加速器达到加速下载镜像的目的。

### 1.3.1 获取加速地址

浏览器打开 http://cr.console.aliyun.com，注册或登录阿里云账号，点击左侧的镜像加速器，将会得到一个专属的加速地址，而且下面有使用配置说明：
```
[15:02:51 root@ubuntu18-04 ~]#mkdir -p /etc/docker
[15:03:01 root@ubuntu18-04 ~]#vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://qai5ut9z.mirror.aliyuncs.com"]
}
[15:04:14 root@ubuntu18-04 ~]#systemctl restart docker
[15:05:26 root@ubuntu18-04 ~]#docker info | grep -n1 Registry
51-  127.0.0.0/8
52: Registry Mirrors:
53-  https://qai5ut9z.mirror.aliyuncs.com/
```

## 1.4 Docker 镜像管理

Docker 镜像含有启动容器所需要的文件系统及所需要的内容，因此镜像主要 用于创建并启动 docker 容器。

Docker 镜像含里面是一层层文件系统,叫做 Union File System（Union FS 联合文件系统），2004 年由纽约州立大学石溪分校开发，联合文件系统可以将多 个目录挂载到一起从而形成一整个虚拟文件系统，该虚拟文件系统的目录结构就 像普通 linux 的目录结构一样，docker 通过这些文件再加上宿主机的内核提供 了一个 linux 的虚拟环境,每一层文件系统我们叫做一层 layer，联合文件系统可 以对每一层文件系统设置三种权限，只读（readonly）、读写（readwrite）和写 出（whiteout-able），但是 docker 镜像中每一层文件系统都是只读的,构建镜 像的时候,从一个最基本的操作系统开始,每个构建的操作都相当于做一层的修改, 增加了一层文件系统,一层层往上叠加,上层的修改会覆盖底层该位置的可见性， 这也很容易理解，就像上层把底层遮住了一样,当使用镜像的时候，我们只会看 到一个完全的整体，不知道里面有几层也不需要知道里面有几层，结构如下：


```
[15:10:30 root@ubuntu18-04 ~]#pwd
/root
[15:10:32 root@ubuntu18-04 ~]#mkdir a b system
[15:10:44 root@ubuntu18-04 ~]#touch a/a.txt b/b.txt
[15:10:58 root@ubuntu18-04 ~]#mount -t aufs -o dirs=./a:./b none ./system/
[15:11:38 root@ubuntu18-04 ~]#tree system/
system/
├── a.txt
└── b.txt

0 directories, 2 files
```
一个典型的 Linux 文件系统由 bootfs 和 rootfs 两部分组成，bootfs(boot file system) 主要包含 bootloader 和 kernel，bootloader 主要用于引导加载 kernel， 当 kernel 被加载到内存中后 bootfs 会被 umount 掉，rootfs (root file system) 包含的就是典型 Linux 系统中的/dev，/proc，/bin，/etc 等标准目录和文件， 下图就是 docker image 中最基础的两层结构，不同的 linux 发行版（如 ubuntu 和 CentOS ) 在 rootfs 这一层会有所区别。

但是对于 docker 镜像通常都比较小，官方提供的 centos 基础镜像在 200MB 左右，一些其他版本的镜像甚至只有几 MB，docker 镜像直接调用宿主机的内核， 镜像中只提供 rootfs，也就是只需要包括最基本的命令、工具和程序库就可以了，比如 alpine 镜像，在 5M 左右。

下图就是有两个不同的镜像在一个宿主机内核上实现不同的 rootfs。



容器、镜像父镜像



docker 命令是最常使用的 docker 客户端命令，其后面可以加不同的参数以实 现相应的功能，常用的命令如下：

### 1.4.1 搜索镜像
在官方的 docker 仓库中搜索指定名称的 docker 镜像，也会有很多镜
```
[15:15:22 root@ubuntu18-04 ~]#docker search centos:7.2.1511   #带指定版本
[15:15:56 root@ubuntu18-04 ~]#docker search centos   #不带版本号默认latest
```

### 1.4.2 下载镜像
从 docker 仓库将镜像下载到本地，命令格式如下：
```
#docker pull 仓库服务器:端口/项目名称/镜像名称:tag(版本)号
#不写仓库表示去默认仓库查找，不写tag表示默认latest
[15:16:00 root@ubuntu18-04 ~]#docker pull alpine
[15:18:38 root@ubuntu18-04 ~]#docker pull nginx
[15:19:05 root@ubuntu18-04 ~]#docker pull hello-world
[15:19:41 root@ubuntu18-04 ~]#docker pull centos
```

### 1.4.3 查看本地镜像
下载完成的镜像比下载的大，因为下载完成后会解压

[15:21:34 root@ubuntu18-04 ~]#docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    519e12e2a84a   3 days ago     133MB
alpine        latest    49f356fa4513   12 days ago    5.61MB
hello-world   latest    d1165f221234   5 weeks ago    13.3kB
centos        latest    300e315adb2f   4 months ago   209MB

REPOSITORY #镜像所属的仓库名称
TAG        #镜像版本号（标识符），默认为 latest
IMAGE ID   #镜像唯一 ID 标示
CREATED    #镜像创建时间
VIRTUAL SIZE #镜像的大小
```

### 1.4.4 镜像导出
可以将镜像从本地导出问为一个压缩文件，然后复制到其他服务器进行导入使用。
```
#导出方法1
[15:21:45 root@ubuntu18-04 ~]#docker save centos -o centos.tar.gz
[15:23:14 root@ubuntu18-04 ~]#ll centos.tar.gz 
-rw------- 1 root root 216535040 Apr 13 15:23 centos.tar.gz
#导出方法2
[15:23:17 root@ubuntu18-04 ~]#docker save centos >centos-1.tar.gz
[15:23:59 root@ubuntu18-04 ~]#ll centos-1.tar.gz 
-rw-r--r-- 1 root root 216535040 Apr 13 15:23 centos-1.tar.gz

#查看镜像内容
[15:24:36 root@ubuntu18-04 ~]#mkdir centos
[15:24:44 root@ubuntu18-04 ~]#mv centos-1.tar.gz centos/
[15:24:51 root@ubuntu18-04 ~]#cd centos/
[15:25:07 root@ubuntu18-04 centos]#tar xf centos-1.tar.gz 
[15:25:15 root@ubuntu18-04 centos]#cat manifest.json  #包含了镜像的相关配置，配置文件、分层
[{"Config":"300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55.json","RepoTags":["centos:latest"],"Layers":["f1037aee492bd55e63970da6da22c5c974c84e5ce4cb967104f2666c6abf7f98/layer.tar"]}]

#分层为了方便文件的共用，即相同的文件可以共用
[{"Config":" 配置文件.json","RepoTags":["docker.io/nginx:latest"],"Layers":[" 分层 1/layer.tar","分层 2 /layer.tar","分层 3 /layer.tar"]}]
```

### 1.4.5 镜像导入
```
#将镜像导入到 docker
[15:25:27 root@ubuntu18-04 centos]#scp centos-1.tar.gz 192.168.10.71:
[15:32:43 root@centos7 ~]#docker load < centos-1.tar.gz
[15:32:43 root@centos7 ~]#docker load -i centos-1.tar.gz
[15:32:59 root@centos7 ~]#docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
centos       latest    300e315adb2f   4 months ago   209MB
```

### 1.4.6 删除镜像
```
[15:33:09 root@centos7 ~]#docker rmi centos
Untagged: centos:latest
Deleted: sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55
Deleted: sha256:2653d992f4ef2bfd27f94db643815aa567240c37732cae1405ad1c1309ee9859
```
### 1.4.7 镜像命令总结
```
#获取运行参数帮助
[15:31:31 root@ubuntu18-04 centos]#docker daemon -help

#总结：企业使用镜像及常见操作： 搜索、下载、导出、导入、删除

#命令总结
docker pull 镜像名称:tags                #下载镜像
docker search 镜像名称:tags              #搜索镜像
docker load -i centos-latest.tar.xz     #导入本地镜像
docker save > /opt/centos.tar           #centos  #导出镜像
docker rmi  镜像ID/镜像名称              #删除指定 ID 的镜像，通过镜像启动容器的时候镜像不能被删除，除非将容器全部关闭
```

## 1.5 容器操作基础命令
```
#命令格式：
docker run [选项] [镜像名] [shell 命令] [参数]
docker run [参数选项] [镜像名称，必须在所有选项的后面] [/bin/echo 'hellowold']  #单次执行，没有自定义容器名称
docker run centos /bin/echo 'hello wold' #启动的容器在执行完 shel命令就退出了
```

### 1.5.1 从镜像启动一个容器
```
#会直接进入到容器，并随机生成容器 ID 和名称
[15:34:32 root@ubuntu18-04 centos]#docker run -it docker.io/centos bash
[root@07e1ea06d8eb /]# 
#退出容器不注销
ctrl+p+q
```

### 1.5.2 显示正在运行的容器
```
[15:40:38 root@ubuntu18-04 centos]#docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS              PORTS     NAMES
07e1ea06d8eb   centos    "bash"    About a minute ago   Up About a minute             gallant_tereshkova
```

### 1.5.3 显示所有容器
```
#包括当前正在运行以及已经关闭的所有容器
[15:40:40 root@ubuntu18-04 centos]#docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS              PORTS     NAMES
07e1ea06d8eb   centos    "bash"    About a minute ago   Up About a minute             gallant_tereshkova
```

### 1.5.4 删除运行中的容器
```
#即使容正在运行当中，也会被强制删除掉
[15:41:20 root@ubuntu18-04 centos]#docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
07e1ea06d8eb   centos    "bash"    2 minutes ago   Up 2 minutes             gallant_tereshkova
[15:42:01 root@ubuntu18-04 centos]#docker rm -f 07e1ea06d8eb
07e1ea06d8eb
[15:42:20 root@ubuntu18-04 centos]#docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 1.5.5 随机映射端口
```
[15:42:28 root@ubuntu18-04 centos]#docker pull nginx  #下载nginx镜像
[15:43:35 root@ubuntu18-04 centos]#docker run -P docker.io/nginx  #前台启动并随机映射本地端口到容器的 80
#前台启动的会话窗口无法进行其他操作，除非退出，但是退出后容器也会退出
#随机端口映射，其实是默认从 32768 开始
[15:45:22 root@ubuntu18-04 /]#ss -ntl
State    Recv-Q    Send-Q        Local Address:Port        Peer Address:Port       
LISTEN   0         128                       *:32768                  *:*
#访问端口
[15:47:05 root@ubuntu18-04 /]#curl 192.168.10.181:32768 -I
HTTP/1.1 200 OK
```

### 1.5.6 指定端口映射
```
#方式1：本地端口81映射到容器80
[15:52:07 root@ubuntu18-04 ~]#docker run -p 81:80 --name nginx-test-port1 nginx

#方式2：本地IP:本地端口:容器端口
[15:53:18 root@ubuntu18-04 ~]#docker run -p 192.168.10.181:81:80 --name nginx-test-port2 nginx

#方式3：本地IP:本地随机端口:容器端口
[15:54:23 root@ubuntu18-04 ~]#docker run -p 192.168.10.181::80 --name nginx-test-port3 nginx

#方式4：本机ip:本地端口:容器端口/协议，默认为tcp协议
[15:55:31 root@ubuntu18-04 ~]#docker run -p 192.168.10.181:83:80/udp --name nginx-test-port4 nginx

#方式5:一次性映射多个端口+协议：
[16:04:51 root@ubuntu18-04 ~]#docker run -p 86:80/tcp -p 443:443/tcp -p 56:56/udp --name nginx-test-port5 nginx

#查看 Nginx 容器访问日志
```

### 1.5.7 查看容器已经映射的端口
```
[16:05:58 root@ubuntu18-04 /]#docker port nginx-test-port5 
80/tcp -> 0.0.0.0:86
443/tcp -> 0.0.0.0:443
56/udp -> 0.0.0.0:56
```

### 1.5.8 自定义容器名称
```
[16:39:32 root@ubuntu18-04 ~]#docker run -it --name nginx-test nginx
```

### 1.5.9 后台启动容器
```
[16:40:38 root@ubuntu18-04 ~]#docker run -d --name nginx-test nginx
```

### 1.5.10 创建并进入容器
```
[16:41:58 root@ubuntu18-04 ~]#docker run -it --name test-centos centos /bin/bash
[root@8fb6b4596f1c /]# ps aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  3.0  0.3  12024  3332 pts/0    Ss   08:43   0:00 /bin/bash
root         14  0.0  0.3  44632  3320 pts/0    R+   08:43   0:00 ps aux
```

### 1.5.11 单次运行
容器退出后自动删除：
```
[16:45:52 root@ubuntu18-04 ~]#docker run -it --rm --name nginx-delete nginx
^C[16:46:53 root@ubuntu18-04 ~]#docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 1.5.12 传递运行命令
器需要有一个前台运行的进程才能保持容器的运行，通过传递运行参数是一 种方式，另外也可以在构建镜像的时候指定容器启动时运行的前台命令。
```
[16:46:59 root@ubuntu18-04 ~]#docker run -d centos tail -f '/etc/hosts'
10642e07d52004c3eb795118896f740ad2eb99d46438027c5be611bde296c043
[16:49:39 root@ubuntu18-04 ~]#docker ps
CONTAINER ID   IMAGE     COMMAND                CREATED          STATUS         PORTS     NAMES
10642e07d520   centos    "tail -f /etc/hosts"   11 seconds ago   Up 8 seconds             epic_pike
```

### 1.5.13 容器的启动和关闭
[16:49:47 root@ubuntu18-04 ~]#docker stop 10642e07d520  #停止容器
[16:51:05 root@ubuntu18-04 ~]#docker start  10642e07d520 #启动容器
```

### 1.5.14 进入到正在运行的容器

#### 1.5.14.1 使用 attach 命令
使用方式为 docker attach 容器名，attach 类似于 vnc，操作会在各个容器界面显示，所有使用此方式进入容器的操作都是同步显示的且 exit 后容器将被关闭，且使用 exit 退出后容器关闭，不推荐使用，需要进入到有 shell 环境的容器， 比如 centos 为例：
```
[16:57:49 root@ubuntu18-04 ~]#docker run -it centos bash
[root@d8112fd2dbcd /]# 
[root@s1 ~]# docker attach 63fbc2d5a3ec
[root@63fbc2d5a3ec /]#     
#在另外一个窗口启动测试页面是否同步
```

#### 1.5.14.2 使用 exec 命令
执行单次命令与进入容器，不是很推荐此方式，虽然 exit 退出容器还在运行
```
[17:02:19 root@ubuntu18-04 /]#docker exec -it  e4ef9a1b1c4b bash
[root@e4ef9a1b1c4b /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
[root@e4ef9a1b1c4b /]# exit
exit
[17:02:53 root@ubuntu18-04 /]#docker ps 
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
e4ef9a1b1c4b   centos    "bash"    3 minutes ago   Up 3 minutes             confident_poitras
```
#### 1.5.14.3 使用 nsenter 命令
推荐使用此方式，nsenter 命令需要通过 PID 进入到容器内部，不过可以使用 docker inspect 获取到容器的 PID
```
#ubuntu安装nsenter命令
[17:03:16 root@ubuntu18-04 /]#apt install util-linux
#centos安装nsenter命令
[15:52:18 root@centos7 ~]#yum install util-linux

[17:06:42 root@ubuntu18-04 /]#docker inspect -f "{{.NetworkSettings.IPAddress}}" e4ef9a1b1c4b
172.17.0.2

[17:06:50 root@ubuntu18-04 /]#docker inspect -f "{{.State.Pid}}" e4ef9a1b1c4b
7804  #获取到某个 docker 容器的 PID，可以通过 PID 进入到容器内

[17:08:49 root@ubuntu18-04 /]#nsenter -t 7804 -m -u -i -n -p
```

#### 1.5.14.4 脚本方式
```
#将 nsenter 命令写入到脚本进行调用，如下
docker_in(){
    PID=`docker inspect -f "{{.State.Pid}}" ${NAME_ID}`
    nsenter -t ${PID} -m -u -i -n -p
}

man(){
echo $1
NAME_ID=$1
if [ -z "${NAME_ID}" ];then
    echo "docker-in.sh CONTAINER ID|NAMES"
else
    docker inspect $1 &>/dev/null
    if [ $? -eq 0 ];then
        docker_in
    else
        echo "Please enter the correct Docker ID or name!"
    fi
fi
}
man $1
```

### 1.5.15 查看容器内部的 hosts 文件
```
[17:40:41 root@ubuntu18-04 /]#docker run -i -t --name test-centos centos bash
[root@ef5cbea46c93 /]# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	ef5cbea46c93 #默认会将实例的 ID 添加到自己的 hosts 文件

#ping容器ID
[root@ef5cbea46c93 /]# ping ef5cbea46c93
PING ef5cbea46c93 (172.17.0.3) 56(84) bytes of data.
64 bytes from ef5cbea46c93 (172.17.0.3): icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from ef5cbea46c93 (172.17.0.3): icmp_seq=2 ttl=64 time=0.064 ms
```

### 1.5.16 批量关闭正在运行的容器
```
[17:42:20 root@ubuntu18-04 /]#docker stop `docker ps -a -q`  #正常关闭所有运行中的容器
[17:42:20 root@ubuntu18-04 /]#docker stop $(docker ps -a -q)
```

### 1.5.17 批量强制关闭正在运行的容器
```
[17:43:08 root@ubuntu18-04 /]#docker kill $(docker ps -a -q)
```

### 1.5.18 批量删除已退出容器
```
[17:44:12 root@ubuntu18-04 /]#docker rm -f `docker ps -aq -f status=exited`
ef5cbea46c93
e4ef9a1b1c4b
```

### 1.5.19 批量删除所有容器
```
[17:46:45 root@ubuntu18-04 /]#docker rm -f $(docker ps -aq)
28ac8317c578
```

### 1.5.20 指定容器 DNS
```
Dns 服务，默认采用宿主机的 dns 地址
一是将 dns 地址配置在宿主机
二是将参数配置在 docker 启动脚本里面 –dns=1.1.1.1
```

### 1.5.21 拷贝容器中文件到宿主机
```
[17:40:41 root@ubuntu18-04 /]#docker cp test-centos:/etc/hosts /opt/   #拷贝容器中文件到宿主机
[17:40:41 root@ubuntu18-04 /]#docker cp /opt/hosts test-centos:/etc/hosts   #拷贝宿主机文件到容器
```

### 1.5.22 给容器传递环境变量
```
#有些容器启动时需要使用一些其他的环境变量，比如mysql
[17:40:41 root@ubuntu18-04 /]#docker run -it -p 3306:3306 --env "MYSQL_ROOT_PASSWORD=123456" --name mysql mysql
```

### 1.5.23 映射宿主机目录到容器
```
#挂载本地目录到容器-v
[11:51:52 root@docker ~]#docker run -it -p 3306:3306 --name mysql -v /data/mysql:/var/lib/mysql --env "MYSQL_ROOT_PASSWORD=123456" mysql
```

### 1.5.23 其他命令
```
[17:50:10 root@ubuntu18-04 /]#docker update bc529b3d97d6 --cpus 2 #更新容器配置信息
[17:50:37 root@ubuntu18-04 /]#docker events  #获取 dockerd 的实时事件
[17:54:52 root@ubuntu18-04 /]#docker stop bc529b3d97d6
bc529b3d97d6
[17:55:02 root@ubuntu18-04 /]#docker wait bc529b3d97d6 #显示容器的退出状态码
0
[17:56:07 root@ubuntu18-04 /]#docker diff bc529b3d97d6  #检查容器更改过的文件或目录
[17:57:00 root@ubuntu18-04 /]#docker logs bc529b3d97d6 #查看容器日志
[17:57:00 root@ubuntu18-04 /]#docker logs -f bc529b3d97d6 #跟踪容器日志
```

# 二、Docker 镜像与制作

**Docker 镜像有没有内核？**

从镜像大小上面来说，一个比较小的镜像只有十几 MB，而内核文件需要一百多兆， 因此镜像里面是没有内核的，镜像在被启动为容器后将直接使用宿主机的内核，而镜像本身则只提供相应的 rootfs，即系统正常运行所必须的用户空间的文件系统，比如/dev/，/proc，/bin，/etc 等目录，所以容器当中基本是没有 /boot 目录的，而/boot 当中保存的就是与内核相关的文件和目录。
```
[18:55:22 root@docker ~]#uname -r
4.15.0-140-generic
[18:55:25 root@docker ~]#docker run -it --rm --name centos7 centos:7
[root@e592cd81d362 /]# uname -r
4.15.0-140-generic
[root@e592cd81d362 /]# ll /boot
ls: cannot access /boot: No such file or directory
```

**为什么没有内核？**

由于容器启动和运行过程中是直接使用了宿主机的内核，所以没有直接调用过 物理硬件，所以也不会涉及到硬件驱动，因此也用不上内核和驱动，另外有内核的那是虚拟机。

## 2.1 手动制作 yum 版 nginx 镜像

Docker 制作类似于虚拟机的模板制作，即按照公司的实际业务务求将需要安装的软件、相关配置等基础环境配置完成，然后将虚拟机再提交为模板，最后再批量从模板批量创建新的虚拟机，这样可以极大的简化业务中相同环境的虚拟机运行环境的部署工作，Docker 的镜像制作分为手动制作和自动制作(基于DockerFile)，企业通常都是基于 Dockerfile 制作镜像，其中手动制作镜像步骤具体如下：

### 2.1.1 下载镜像并初始化系统
```
#基于某个基础镜像之上重新制作，因此需要先有一个基础镜像，本次使用官方提供的 centos 镜像为基础：
[18:56:35 root@docker ~]#docker pull centos:7
[18:58:58 root@docker ~]#docker run -it centos:7 
[root@be7aa1d10156 /]# yum install -y epel-release #由于是centos7官方包没有nginx先安装epel源
[root@be7aa1d10156 /]# yum install -y nginx    #安装nginx
[root@be7aa1d10156 /]# vi /etc/nginx/nginx.conf   
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
daemon off;  #关闭后台运行
[root@be7aa1d10156 /]# nginx -t  #测试配置文件
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@be7aa1d10156 /]# echo "<h1>zhangzhuo</h1>" >/usr/share/nginx/html/index.html    #准备自定义web页面
```

### 2.1.2 提交为镜像
在宿主机基于容器 ID 提交为镜像
```
[19:13:34 root@docker ~]#docker commit -m "nginx image" be7aa1d10156 web/centos7-nginx   #提交
sha256:18bd35ac63cf63a83a368e857f0807f9f49936a53d863d43f6ee95b72d747983
[19:13:57 root@docker ~]#docker images    #查看镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
web/centos7-nginx   latest              18bd35ac63cf        12 seconds ago      436MB
centos              7                   8652b9f0cb4c        5 months ago        204MB
[19:14:09 root@docker ~]#docker commit -m "nginx image" be7aa1d10156 web/centos7-nginx:v1   #提交带tag的镜像
sha256:fb60658fad06100600b009fd248edf41a712e034ce33180a097db6d844388d08
[19:15:11 root@docker ~]#docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
web/centos7-nginx   v1                  fb60658fad06        27 seconds ago       436MB
web/centos7-nginx   latest              18bd35ac63cf        About a minute ago   436MB
centos              7                   8652b9f0cb4c        5 months ago         204MB
```

#### 2.1.2.1 docker commit命令
```
[19:09:57 root@docker ~]#docker commit --help
Usage:	docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
Create a new image from a container's changes
Options:
  -a, --author string    Author (e.g., "John Hannibal Smith
                         <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
```

### 2.1.3 从自己镜像启动容器
```
[19:15:37 root@docker ~]#docker run -d -p 80:80 --name nginx web/centos7-nginx:v1 /usr/sbin/nginx  #启动
f6149017f11f7fe153471957ce278789ada30a2fa41bb6c0ef1ee3becc798dce
[19:16:56 root@docker ~]#ss -ntl | grep 80
LISTEN   0         128                       *:80                     *:* 
[19:17:02 root@docker ~]#curl 192.168.10.181  #测试访问
<h1>zhangzhuo</h1>
```

## 2.2 DockerFile 制作编译版 nginx 镜像
DockerFile 可以说是一种可以被 Docker 程序解释的脚本，DockerFile 是由一 条条的命令组成的，每条命令对应 linux 下面的一条命令，Docker 程序将这些 DockerFile 指令再翻译成真正的 linux 命令，其有自己的书写方式和支持的命令， Docker 程序读取 DockerFile 并根据指令生成 Docker 镜像，相比手动制作镜像的方式，DockerFile 更能直观的展示镜像是怎么产生的，有了写好的各种各样 DockerFile 文件，当后期某个镜像有额外的需求时，只要在之前的 DockerFile 添加或者修改相应的操作即可重新生成新的 Docke 镜像，避免了重复手动制作镜像的麻烦，具体如下：

https://docs.docker.com/engine/reference/builder/
```
FROM    #在整个dockfile文件中，除了注释之外的第一行，要是from，用于指定父镜像
ADD     #用于添加宿主机本地的文件、目录、压缩等资源到镜像里面去，会自动解压tar.gz格式的压缩包，不会自动解压zip
MAINTAINER   #(镜像的作者信息)
LABEL    #设置镜像的属性标签
COPY     #用于添加宿主机本地的文件、目录、压缩等资源到镜像里面去，不会解压任何压缩包
ENV      #设置容器环境变量
EXPOSE   #声明要把容器的某些端口映射到宿主机
STOPSIGNAL #设置信号
USER     #指定运行操作的用户
VOLUME   #定义volume，也就是创建目录
WORKDIR  #用于定义工作目录
RUN      #执行shell命令，但是一定要以非交互式的方式执行
CMD      #镜像启动为一个容器时候的默认命令或脚本， CMD ["/bin/bash"] 
ENTRYPOINT #也可以用于定义容器在启动时候默认执行的命令或者脚本，如果是和CMD命令混合使用的时候，会将CMD的命令当做参数传递给ENTRYPOINT后面的脚本，可以在脚本中对参数做判断并相应的容器初始化操作。
```

### 2.2.1 下载镜像并初始化系统
```
[18:56:35 root@docker ~]#docker pull centos:7
[19:26:40 root@docker opt]#mkdir dockerfile/{web/{nginx,tomcat,apache},system/{centos,ubuntu,redhat}} -pv  #创建目录环境
mkdir: created directory 'dockerfile'
mkdir: created directory 'dockerfile/web'
mkdir: created directory 'dockerfile/web/nginx'
mkdir: created directory 'dockerfile/web/tomcat'
mkdir: created directory 'dockerfile/web/apache'
mkdir: created directory 'dockerfile/system'
mkdir: created directory 'dockerfile/system/centos'
mkdir: created directory 'dockerfile/system/ubuntu'
mkdir: created directory 'dockerfile/system/redhat
#目录结构按照业务类型或系统类型等方式划分，方便后期镜像比较多的时候进行分类。
[19:27:59 root@docker opt]#cd dockerfile/web/nginx/  #进入到指定的 Dockerfile 目录
[19:28:35 root@docker nginx]#pwd
/opt/dockerfile/web/nginx
```

### 2.2.2 编写 Dockerfile
```
[19:28:54 root@docker nginx]#vim Dockerfile   #生成的镜像的时候会在执行命令的当前目录查找 Dockerfile 文件，所以名称不可写错，而且D必须大写
#My Dockerfile
#'#'为注释，等于shell脚本中#
#除了注释行以外的第一行，必须是From开始
From centos:7     #第一行先定义基础镜像，后面的本地有效的镜像名，如果本地没有会从远程仓库下载，第一行很重要
MAINTAINER Zhang.Zhuo 1191400158@qq.com  #镜像维护者的信息
RUN yum install -y epel-release && yum install -y gcc gcc-g++ pcre pcre-devel zlib zlib-devel openssl openssl-devel automake perl make  #安装编译环境
ADD nginx-1.18.0.tar.gz /usr/local/src/   #自动解压压缩包
RUN cd /usr/local/src/nginx-1.18.0 && ./configure --prefix=/usr/local/src/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module && make && make install #编译安装
RUN cd /usr/local/src/nginx
ADD nginx.conf /usr/local/src/nginx/conf/nginx.conf
RUN useradd nginx -s /sbin/nologin
RUN ln -sv /usr/local/nginx/sbin/nginx /usr/sbin/nginx
ADD index.html /usr/local/nginx/html/index.html
EXPOSE 80 443 #向外开放的端口，多个端口用空格做间隔，启动容器时候-p需要使用此端向外映射，如： -p 8081:80，则 80 就是这里的 80
CMD ["/usr/local/src/nginx/sbin/nginx"]  #运行的命令，每个 Dockerfile 只能有一条，如果有多条则只有最后一条被执行
```

如果在从该镜像启动容器的时候也指定了命令，那么指定的命令会覆盖Dockerfile 构建的镜像里面的 CMD 命令，即指定的命令优先级更高，Dockerfile的优先级较低一些，重新指定的指令优先级要高一些。

### 2.2.3 准备源码包与配置文件
```
[20:21:19 root@docker nginx]#ll nginx-1.18.0.tar.gz 
-rw-r--r-- 1 root root 1039530 Mar 23 12:03 nginx-1.18.0.tar.gz
[20:21:30 root@docker nginx]#ll index.html 
-rw-r--r-- 1 root root 19 Apr 14 20:05 index.html
[20:21:35 root@docker nginx]#ll nginx.conf 
-rw-r--r-- 1 root root 3786 Apr 14 20:10 nginx.conf
```

### 2.2.4 执行镜像构建
```
[20:21:42 root@docker nginx]#cat docker_build.sh 
#!/bin/bash
docker build -t nginx:v1 .
[20:22:15 root@docker nginx]#bash docker_build.sh
```

### 2.2.5 查看是否生成本地镜像
```
[20:23:43 root@docker nginx]#docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               v1                  761babea4d47        13 minutes ago      516MB
centos              7                   8652b9f0cb4c        5 months ago        204MB
```

### 2.2.6 从镜像启动容器
```
[20:25:03 root@docker nginx]#docker run -it -d --rm -p 80:80 nginx:v1
33c987262d6d882d77d02952a74bbccbb30aff5385d9693e5fbbcc96edd5794b
#测试访问
[20:25:06 root@docker nginx]#curl 192.168.10.181
<h1>zhangzhuo</h1>
```

### 2.2.7 镜像编译制作程中
```
制作镜像过程中，需要启动一个临时的容器进行相应的指令操作，操作完成后再把此容器转换为 image。

[20:36:44 root@docker ~]#docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
57ba27ee44de        8652b9f0cb4c        "/bin/sh -c 'yum ins…"   14 seconds ago      Up 14 seconds                                     confident_saha
```

## 2.3 自定义 Tomcat 业务镜像
基于官方提供的 centos、debain、ubuntu、alpine 等基础镜像构建 JDK(Java 环境)，然后再基于自定义的 JDK 镜像构建出业务需要的 tomcat 镜像

### 2.3.1 构建 JDK 镜像
先基于官方提供的基础镜像，制作出安装了常用命令的自定义基础镜像，然后在基础镜像的基础之上，再制作 JDK 镜像、Tomcat 镜像等。

#### 2.3.1.1 自定义 Centos 基础镜像
```
[20:30:55 root@docker ~]#docker pull centos:7
[20:19:26 root@docker ~]#cd /opt/dockerfile/system/centos/
[20:32:56 root@docker centos]#cat Dockerfile
FROM centos:7
RUN yum install -y epel-release vim wget tree lrzsz gcc gcc-c++ auto make pcre  pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop
RUN groupadd www -g 2020 && useradd www -u 2020 -g www #添加系统账户
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  #设置时区
[20:35:57 root@docker centos]#cat build-command.sh  #通过脚本构建镜像
#!/bin/bash
docker build -t centos-base:v1 .
[20:38:21 root@docker centos]#bash build-command.sh
```

#### 2.3.1.2 执行构建 JDK 镜像
```
[20:38:21 root@docker centos]#mkdir /opt/dockerfile/web/jdk
[20:39:17 root@docker centos]#cd /opt/dockerfile/web/jdk
[20:39:29 root@docker jdk]#mkdir jdk-8u281
[20:40:28 root@docker jdk-8u281]#ls
jdk-8u281-linux-x64.tar.gz   #上传jdk
[20:56:19 root@docker jdk-8u281]#tail profile  #准备环境变量
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib/:$JRE_HOME/lib/
[20:57:34 root@docker jdk-8u281]#cat Dockerfile  #准备Dockerfile文件
FROM centos-base:v1
MAINTAINER zhangzhuo "1191400158@qq.com"
ADD jdk-8u281-linux-x64.tar.gz /usr/local/src/
RUN ln -s /usr/local/src/jdk1.8.0_281 /usr/local/jdk
ADD profile /etc/profile
#ENV用来给刚启动容器的root用户使用
ENV JAVA_HOME=/usr/local/jdk
ENV PATH=$PATH:$JAVA_HOME/bin
ENV JRE_HOME=$JAVA_HOME/jre
ENV CLASSPATH=$JAVA_HOME/lib/:$JRE_HOME/lib/
[20:58:35 root@docker jdk-8u281]#cat build-jdk.sh  #创建脚本
#!/bin/bash
docker build -t jdk-8u281:v1 .
[20:58:35 root@docker jdk-8u281]#cat build-jdk.sh #执行
[20:59:20 root@docker jdk-8u281]#docker images   #查看镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jdk-8u281           v1                  cd9e5a353563        10 seconds ago      838MB
centos-base         v1                  b43d61fb8f07        3 minutes ago       482MB
nginx               v1                  761babea4d47        48 minutes ago      516MB
centos              7                   8652b9f0cb4c        5 months ago        204MB
[20:59:30 root@docker jdk-8u281]#docker run -it --rm jdk-8u281:v1 #测试
[root@8481d8d01c27 /]# java -version
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 1.8.0_281-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.281-b09, mixed mode)
```

### 2.3.2 从 JDK 镜像构建 tomcat 8 Base 镜像

基于自定义的 JDK 基础镜像，构建出通用的自定义 Tomcat 基础镜像，此镜像后期会被多个业务的多个服务共同引用(相同的 JDK 版本和 Tomcat 版本)。

#### 2.3.2.1 编辑 Dockerfile
```
[09:24:38 root@docker tomcat]#cd /opt/dockerfile/web/tomcat/
[09:32:27 root@docker tomcat]#pwd
/opt/dockerfile/web/tomcat
[09:39:26 root@docker tomcat]#cat Dockerfile 
FROM jdk-8u281:v1
ADD apache-tomcat-8.5.64.tar.gz /apps/
RUN ln -s /apps/apache-tomcat-8.5.64 /apps/tomcat
#准备脚本
[09:40:54 root@docker tomcat]#cat build_tomcat.sh 
#!/bin/bash
docker build -t tomcat-base:v8.5.64 .
#上传tomcat压缩包
[09:40:16 root@docker tomcat]#tree 
.
├── apache-tomcat-8.5.64.tar.gz
├── build_tomcat.sh
└── Dockerfile
#执行构建
[09:41:07 root@docker tomcat]#bash build_tomcat.sh 
[09:41:40 root@docker tomcat]#docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat-base         v8.5.64             be9d31f76c92        4 minutes ago       853MB
jdk-8u281           v1                  cd9e5a353563        13 hours ago        838MB
centos-base         v1                  b43d61fb8f07        13 hours ago        482MB
nginx               v1                  761babea4d47        14 hours ago        516MB
centos              7                   8652b9f0cb4c        5 months ago        204MB
```

### 2.3.3 构建业务镜像 1
创建 tomcat app1 和 tomcat app2 两个目录，表示基于 tomcat 自定义基础镜像构建出不同业务的 tomcat app 镜像

#### 2.3.3.1 准备 Dockerfile
```
[09:43:53 root@docker app1]#pwd
/opt/dockerfile/web/app1
[10:23:34 root@docker app1]#cat Dockerfile 
FROM tomcat-base:v8.5.64
ADD run_tomcat.sh /apps/tomcat/bin/run_tomcat.sh
RUN mkdir /apps/apache-tomcat-8.5.64/webapps/myapp
COPY myapp /apps/apache-tomcat-8.5.64/webapps/myapp
RUN chown www.www /apps -R
EXPOSE 8080
CMD ["/apps/tomcat/bin/run_tomcat.sh"]
#准备测试代码
[10:23:50 root@docker app1]#ls myapp/
META-INF  robots.txt  static  templates  WEB-INF
#准备容器启动执行脚本
[09:52:28 root@docker app1]#cat run_tomcat.sh 
#!/bin/bash
su - www -c "/apps/tomcat/bin/catalina.sh start"
su - www -c "tail -f /etc/hosts"
[09:53:57 root@docker app1]#chmod +x run_tomcat.sh
#准备构建脚本
[10:24:07 root@docker app1]#bash build-command.sh
#启动
[10:24:37 root@docker app1]#docker run -it -p 8080:8080 --rm tomcat-web:app1
#测试
[10:25:35 root@docker ~]#curl -I 192.168.10.181:8080/myapp/install
HTTP/1.1 200 
Content-Type: text/html;charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 15 Apr 2021 02:25:44 GMT
```

### 2.3.4. 构建业务镜像 2·
```
[10:29:43 root@docker app2]#pwd
/opt/dockerfile/web/app2
[10:32:46 root@docker app2]#cat Dockerfile 
FROM tomcat-base:v8.5.64
ADD run_tomcat.sh /apps/tomcat/bin/run_tomcat.sh
ADD myapp/* /apps/tomcat/webapps/myapp/
RUN chown www.www -R /apps/
EXPOSE 8080
CMD ["/apps/tomcat/bin/run_tomcat.sh"]
[10:39:54 root@docker app2]#mkdir myapp
[10:33:07 root@docker app2]#echo "Tomcat Web Page2" >myapp/index.html
[10:40:12 root@docker app2]#cat build_command.sh 
#!/bin/bash
docker build -t tomcat-web:app2 .
[10:40:19 root@docker app2]#cat run_tomcat.sh 
#!/bin/bash
su - www -c "/apps/tomcat/bin/catalina.sh start"
su - www -c "tail -f /etc/hosts"
[10:40:27 root@docker app2]#bash build_command.sh
```

## 2.5 构建 haproxy 镜像
构建出 haproxy 镜像，将 haproxy 通过容器的方式运行

#### 2.5.1 准备 Dockerfile
```
[10:42:46 root@docker haproxy]#pwd
/opt/dockerfile/web/haproxy
[11:06:31 root@docker haproxy]#vim Dockerfile
FROM centos-base:v1
MAINTAINER zhangzhuo "1191400158@qq.com"
RUN yum install -y libtermcap-devel ncurses-devel libevent-devel readline-devel gcc gcc-c++ glibc glibc-devel pcre pcre-devel openssl openssl-devel systemd-devel net-tools vim iotop bc zip unzip zlib-devel
ADD lua-5.4.3.tar.gz /usr/local/src/
RUN cd /usr/local/src/lua-5.4.3 && make linux && ln -s /usr/local/src/lua-5.4.3 /usr/local/lua && ln -s /usr/local/lua/src/lua /sbin/
ADD haproxy-2.0.21.tar.gz /usr/local/src/
RUN cd  /usr/local/src/haproxy-2.0.21 && make ARCH=x86_64 TARGET=linux-glibc USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1 USE_SYSTEMD=1 USE_CPU_AFFINITY=1 USE_LUA=1 LUA_INC=/usr/local/lua/src/ LUA_LIB=/usr/local/lua/src/ PREFIX=/usr/local/haproxy &&make install PREFIX=/usr/local/haproxy
RUN rm -rf /usr/local/src/haproxy-2.0.21
#这里只有装haproxy，由于启动需要配置文件需根据环境设置所以后面再装
[11:09:30 root@docker haproxy]#cat build_command.sh
#!/bin/bash
docker build -t haproxy-base:v1 .
[11:09:06 root@docker haproxy]#tree 
.
├── build_command.sh
├── Dockerfile
├── haproxy-2.0.21.tar.gz
└── lua-5.4.3.tar.gz

0 directories, 4 files
[11:09:32 root@docker haproxy]#bash build_command.sh 
[11:09:32 root@docker haproxy]#docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
haproxy-base        v1                  e4112201087b        5 minutes ago       741MB
tomcat-web          app2                bca8e74e4ec1        31 minutes ago      867MB
tomcat-web          app1                90f5ffdf147c        33 minutes ago      867MB
tomcat-base         v8.5.64             be9d31f76c92        2 hours ago         853MB
jdk-8u281           v1                  cd9e5a353563        14 hours ago        838MB
centos-base         v1                  b43d61fb8f07        14 hours ago        482MB
nginx               v1                  761babea4d47        15 hours ago        516MB
centos              7                   8652b9f0cb4c        5 months ago        204MB
```

### 2.5.2 准备部署haproxy-web:v1
```
[11:11:21 root@docker haproxy-web1]#pwd
/opt/dockerfile/web/haproxy-web1
#先启动俩个web服务，使用之前生成的镜像tomcat-web:v1和tomcat-web:v2
[11:12:45 root@docker haproxy]#docker run -it -d --rm --name web1 tomcat-web:app1
f17ab5f9b572a23d09b4e6e377067db56d7756120b40febd3bf0f3861776a5e2
[11:13:21 root@docker haproxy]#docker run -it -d --rm --name web2 tomcat-web:app2
cccec395c492d6059488df832c8740f4415632cb4aea589a62d6e0c8b17eef61
#获取容器的ip地址
[11:15:00 root@docker haproxy]#docker inspect web1 | grep IPAddress | tail -1
                    "IPAddress": "172.17.0.2",
[11:15:31 root@docker haproxy]#docker inspect web2 | grep IPAddress | tail -1
                    "IPAddress": "172.17.0.3",
#测试网页
[11:16:14 root@docker haproxy]#curl 172.17.0.2:8080/myapp/
Tomcat Web Page1
[11:16:21 root@docker haproxy]#curl 172.17.0.3:8080/myapp/
Tomcat Web Page2
#准备haproxy配置文件
[11:24:20 root@docker haproxy-web1]#cat haproxy.cfg 
global             
	maxconn 100000       
	chroot /usr/local/haproxy    
	stats socket /usr/local/haproxy/run/haproxy.sock
	uid 99            
	gid 99
	daemon
	pidfile /usr/local/haproxy/run/haproxy.pid
	log 127.0.0.1 local3 info


defaults
	option http-keep-alive 
	option forwardfor     
	maxconn 100000
mode http
	timeout connect 300000ms
	timeout client 300000ms
	timeout server 300000ms

#haproxy状态页
listen stats
	mode http
	bind 0.0.0.0:9999
	stats enable      
	log global
    stats hide-version
	stats uri /haproxy-status
	stats auth haadmin:123456
    stats refresh 5s
    stats admin if TRUE


listen web_host
    bind 0.0.0.0:80
    mode http
    log global
    balance static-rr
    option forwardfor
    server web1 172.17.0.2:8080 weight 1 check inter 3000 fall 3 rise 5
    server web2 172.17.0.3:8080 weight 1 check inter 3000 fall 3 rise 5
#准备启动脚本
[11:31:54 root@docker haproxy-web1]#cat run_haproxy.sh 
#!/bin/bash
/usr/local/haproxy/sbin/haproxy -f /etc/haproxy/haproxy.cfg
tail -f /etc/hosts
[11:31:59 root@docker haproxy-web1]#chmod +x run_haproxy.sh
#准备构建镜像脚本
[11:40:57 root@docker haproxy-web1]#cat build_command.sh 
#!/bin/bash
docker build -t haproxy-web1:v1 .
#构建
[11:41:57 root@docker haproxy-web1]#bash build_command.sh 
[11:42:59 root@docker haproxy-web1]#docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
haproxy-web1        v1                  237fa8d6495e        7 minutes ago       741MB
haproxy-base        v1                  e4112201087b        38 minutes ago      741MB
tomcat-web          app2                bca8e74e4ec1        About an hour ago   867MB
tomcat-web          app1                90f5ffdf147c        About an hour ago   867MB
tomcat-base         v8.5.64             be9d31f76c92        2 hours ago         853MB
jdk-8u281           v1                  cd9e5a353563        15 hours ago        838MB
centos-base         v1                  b43d61fb8f07        15 hours ago        482MB
nginx               v1                  761babea4d47        16 hours ago        516MB
centos              7                   8652b9f0cb4c        5 months ago        204MB
#测试
[11:43:00 root@docker haproxy-web1]#curl 192.168.10.181/myapp/
Tomcat Web Page1
[11:43:27 root@docker haproxy-web1]#curl 192.168.10.181/myapp/
Tomcat Web Page2
```

### 2.5.3 访问 haproxy 控制端


## 2.6 基于官方 alpine 基础镜像制作自定义镜像
```
[14:06:17 root@docker ~]#docker pull alpine
[14:07:34 root@docker alpine]#pwd
/opt/dockerfile/system/alpine
#Dockerfile文件
[14:33:55 root@docker alpine]#cat Dockerfile 
FROM alpine
MAINTAINER zhangzhuo "1191400158@qq.com"
ADD repositories /etc/apk/repositories   #拷贝国内镜像源
RUN apk update && apk add iotop gcc libgcc libc-dev libcurl libc-utils pcre-dev zlib-dev libnfs make pcre pcre2 zip unzip net-tools pstree wget libevent-dev iproute2 tzdata
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime #设置时区
#国内镜像源文件
[14:33:56 root@docker alpine]#cat repositories 
https://mirror.tuna.tsinghua.edu.cn/alpine/v3.13/main
https://mirror.tuna.tsinghua.edu.cn/alpine/v3.13/community
#执行脚本
[14:34:59 root@docker alpine]#cat build_command.sh 
#!/bin/bash
docker build -t alpine-base:v1 .
#执行
[14:35:17 root@docker alpine]#bash build_command.sh
```

**制作nginx镜像**
```
[16:08:00 root@docker alpine]#pwd
/opt/dockerfile/web/nginx/alpine
[16:08:18 root@docker alpine]#cat Dockerfile 
#My Dockerfile
#'#'为注释，等于shell脚本中#
#除了注释行以外的第一行，必须是From开始
From alpine-base:v1
MAINTAINER Zhang.Zhuo 1191400158@qq.com
RUN apk add openssl openssl-dev
ADD nginx-1.18.0.tar.gz /usr/local/src/
RUN cd /usr/local/src/nginx-1.18.0 && ./configure --prefix=/usr/local/src/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module && make && make install
RUN cd /usr/local/src/nginx
ADD nginx.conf /usr/local/src/nginx/conf/nginx.conf
RUN useradd nginx -s /sbin/nologin
RUN ln -sv /usr/local/nginx/sbin/nginx /usr/sbin/nginx
ADD index.html /usr/local/nginx/html/index.html
EXPOSE 80 443
CMD ["/usr/local/src/nginx/sbin/nginx"]
[16:08:20 root@docker alpine]#cat index.html 
<h1>zhangzhuo</h1>
[16:09:07 root@docker alpine]#cat build_command.sh 
#!/bin/bash
docker build -t alpine_nginx:v1 .
[16:09:12 root@docker alpine]#bash build_command.sh 
[16:09:38 root@docker alpine]#docker run -it -p 80:80 --rm alpine_nginx:v1
[16:00:44 root@docker ~]#curl 192.168.10.181
<h1>zhangzhuo</h1>
```

## 2.7 本地镜像上传至官方 docker 仓库
```
#创建docker账户，填写账户信息之后登录
[16:19:51 root@docker alpine]#docker login --username zhangzhuo0705
#输入密码登录
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
#查看认证信息
[16:20:30 root@docker alpine]#cat /root/.docker/config.json 
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "emhhbmd6aHVvMDcwNTpmNTE3YjhlOC1kMzEyLTRlODItOTQ0Zi1hNzA4ZmFmMzAzOGI="
		}
	},
	"HttpHeaders": {
		"User-Agent": "Docker-Client/19.03.15 (linux)"
	}
#上传镜像
[16:25:21 root@docker alpine]#docker tag alpine:latest zhangzhuo0705/alpine:latest  #先打标记
[16:26:30 root@docker alpine]#docker push zhangzhuo0705/alpine #上传
The push refers to repository [docker.io/zhangzhuo0705/alpine]
b2d5eeeaba3a: Mounted from library/alpine 
latest: digest: sha256:def822f9851ca422481ec6fee59a9966f12b351c62ccb9aca841526ffaa9f748 size: 528
#上传成功
```

## 2.8 本地镜像上传到阿里云
将本地镜像上传至阿里云，实现镜像备份与统一分发的功能。

https://cr.console.aliyun.com 注册账户、创建 namespace、创建仓库、修改镜像 tag 及上传镜像
```
#登录
[16:30:45 root@docker alpine]#docker login --username=zz1191400158 registry.cn-hangzhou.aliyuncs.com
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[17:00:02 root@docker alpine]#docker tag 6dbb9cc54074 registry.cn-beijing.aliyuncs.com/zhangzhuo/images:v1
[17:00:02 root@docker alpine]#docker push registry.cn-beijing.aliyuncs.com/zhangzhuo/images:v1
```

# 三、Docker 仓库之单机 Docker Registry
Docker Registry 作为 Docker 的核心组件之一负责镜像内容的存储与分发，客 户端的 docker pull 以及 push 命令都将直接与 registry 进行交互,最初版本的 registry 由 Python 实现,由于设计初期在安全性，性能以及 API 的设计上有着诸多的缺陷，该版本在 0.9 之后停止了开发，由新的项目 distribution（新的 docker register 被称为 Distribution）来重新设计并开发下一代 registry，新的项目由 go 语言开发，所有的 API，底层存储方式，系统架构都进行了全面的重新设计已解决上一代 registry 中存在的问题，2016 年 4 月份 rgistry 2.0 正式发布，docker 1.6 版本开始支持 registry 2.0，而八月份随着 docker 1.8 发布，docker hub 正式启用 2.1 版本 registry 全面替代之前版本 registry，新版 registry 对镜像存储格式 进行了重新设计并和旧版不兼容，docker 1.5和之前的版本无法读取2.0的镜像， 另外，Registry 2.4 版本之后支持了回收站机制，也就是可以删除镜像了，在 2.4 版本之前是无法支持删除镜像的，所以如果你要使用最好是大于 Registry 2.4 版 本的，目前最新版本为 2.7.x。

**官方文档地址：** https://docs.docker.com/registry/

**官方 github 地址：** https://github.com/docker/distribution

本部分将介绍通过官方提供的 docker registry 镜像来简单搭建一套本地私有仓库环境。

## 3.1 下载 docker registry 镜像
```
[18:59:11 root@ubuntu18-04 ~]#docker pull registry
Using default tag: latest
latest: Pulling from library/registry
ddad3d7c1e96: Pull complete 
6eda6749503f: Pull complete 
363ab70c2143: Pull complete 
5b94580856e6: Pull complete 
12008541203a: Pull complete 
Digest: sha256:bac2d7050dc4826516650267fe7dc6627e9e11ad653daca0641437abdf18df27
Status: Downloaded newer image for registry:latest
docker.io/library/registry:latest
```

## 3.2 搭建单机仓库

### 3.2.1 创建授权使用目录
```
[19:02:58 root@ubuntu18-04 ~]#mkdir /docker/auth -p
```

### 3.2.2 创建用户
```
[19:16:50 root@ubuntu18-04 docker]#apt install apache2-utils  #安装工具
[19:16:50 root@ubuntu18-04 docker]#htpasswd -Bbn jack 123456 >auth/htpasswd
[19:17:38 root@ubuntu18-04 docker]#cat auth/htpasswd  #验证用户名密码
jack:$2y$05$xQDLNYNYdX7Hw/2lwztbcuKsWIIn2TG3lP9KSUYBNI9E5LVN9nf2q
```

### 3.2.3 启动 docker registy
```
[19:19:03 root@ubuntu18-04 docker]#docker run -d -p 5000:5000 --restart=always --name registrt1 -v /docker/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" registry
f41a956b775c6922edfab57a3edc3ac5a35a9a325d34e5c12b898c124cee9205
```

### 3.2.4 验证端口和容器
```
[19:22:20 root@ubuntu18-04 docker]#ss -ntl | grep 5000
LISTEN   0         20480                     *:5000                   *:*
```

### 3.2.5 测试登录仓库

#### 3.2.5.1 报错如下
```
[19:22:31 root@ubuntu18-04 docker]#docker login 192.168.10.185:5000
Username: jack
Password: 
Error response from daemon: Get https://192.168.10.185:5000/v2/: http: server gave HTTP response to HTTPS client
```

#### 3.2.5.2 解决方法
编辑各 docker 服务器docker.service 配置文件如下：
```
[19:23:43 root@ubuntu18-04 docker]#vim /lib/systemd/system/docker.service
#添加--insecure-registry 192.168.10.185:5000
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/contain    erd.sock --insecure-registry 192.168.10.185:5000
#重启服务
[19:26:16 root@ubuntu18-04 docker]#systemctl daemon-reload 
[19:26:22 root@ubuntu18-04 docker]#systemctl restart docker
```

#### 3.2.5.3 验证各 docker 服务器登录
```
[19:26:53 root@ubuntu18-04 docker]#docker login 192.168.10.185:5000
Username: jack
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

### 3.2.6 在 Server1 登录后上传镜像

#### 3.2.6.1 镜像打 tag
```
[19:31:04 root@docker local]#docker images
REPOSITORY            TAG                 IMAGE ID            CREATED        
centos-tomcat-web     app1                e80ffa680dc7        26 hours ago 
#打标记格式为
主机IP或域名:端口/仓库/镜像名称:版本
[19:31:05 root@docker local]#docker tag centos-tomcat-web:app1 192.168.10.185:5000/jack/centos-tomcat-web:app1
```

#### 3.2.6.2 上传镜像
```
[19:32:28 root@docker local]#docker push 192.168.10.185:5000/jack/centos-tomcat-web:app1
The push refers to repository [192.168.10.185:5000/jack/centos-tomcat-web]
4e2c2a176fc0: Pushed 
23f6df3a62b0: Pushed 
1d7577df3b6f: Pushed 
881b5c2ddce2: Pushed 
a764977e3e3a: Pushed 
d1aa2af9bc2c: Pushed 
b275f11bf306: Pushed 
ff52868b9321: Pushed 
014cdbcb5a22: Pushed 
f317d1b8cc66: Pushed 
996be175afdf: Pushed 
b2175d74c6e4: Pushed 
174f56854903: Pushed 
app1: digest: sha256:27eaa52e436c76aae7204e5616ffe7831153ec285558258cfc907c43c5d88691 size: 3037
```

### 3.2.7 Server 2 下载镜像并启动容器

#### 3.2.7.1 登录并从 docker registry 下载镜像并且启动
```
[19:29:36 root@ubuntu18-04 docker]#docker login 192.168.10.185:5000
[19:35:46 root@ubuntu18-04 docker]#docker pull 192.168.10.185:5000/jack/centos-tomcat-web:app1
#验证镜像
[19:37:01 root@ubuntu18-04 docker]#docker images
REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
192.168.10.185:5000/jack/centos-tomcat-web   app1                e80ffa680dc7        26 hours ago        867MB
[19:38:00 root@ubuntu18-04 docker]#docker run -it -p 8080:8080 --rm 192.168.10.185:5000/jack/centos-tomcat-web:app1
#测试访问
[19:38:52 root@ubuntu18-04 ~]#curl 192.168.10.185:8080/myapp/
Tomcat Web Page1
```

# 四、docker 仓库之分布式 Harbor
Harbor 是一个用于存储和分发 Docker 镜像的企业级 Registry 服务器，由 vmware 开源，其通过添加一些企业必需的功能特性，例如安全、标识和管理等， 扩展了开源 Docker Distribution。作为一个企业级私有 Registry 服务器，Harbor 提供了更好的性能和安全。提升用户使用 Registry 构建和运行环境传输镜像的效率。Harbor 支持安装在多个 Registry 节点的镜像资源复制，镜像全部保存在私有 Registry 中， 确保数据和知识产权在公司内部网络中管控，另外，Harbor 也提供了高级的安全特性，诸如用户管理，访问控制和活动审计等。

- **vmware 官方开源服务列表地址：** https://vmware.github.io/harbor/cn/
- **harbor 官方 github 地址：** https://github.com/vmware/harbor
- **harbor 官方网址：** https://goharbor.io/



**基于角色的访问控制：** 用户与 Docker 镜像仓库通过“项目”进行组织管理， 一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限。 镜像复制：镜像可以在多个 Registry 实例中复制（同步）。尤其适合于负载均衡，高可用，混合云和多云的场景。

**图形化用户界面：** 用户可以通过浏览器来浏览，检索当前 Docker 镜像仓库，管理项目和命名空间。

**AD/LDAP 支：** Harbor 可以集成企业内部已有的 AD/LDAP，用于鉴权认证管理。 审计管理：所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。

**国际化：** 已拥有英文、中文、德文、日文和俄文的本地化版本。更多的语言将会添加进来。

**RESTful API - RESTful API ：** 提供给管理员对于 Harbor更多的操控, 使得与其它管理软件集成变得更容易。

部署简单：提供在线和离线两种安装工具， 也可以安装到 vSphere 平台(OVA 方式)虚拟设备。
```
nginx：harbor 的一个反向代理组件，代理 registry、ui、token 等服务。这个代理会转发 harbor web 和 docker client 的各种请求到后端服务上。
harbor-adminserver：harbor 系统管理接口，可以修改系统配置以及获取系统信息。
harbor-db：存储项目的元数据、用户、规则、复制策略等信息。
harbor-jobservice：harbor 里面主要是为了镜像仓库之前同步使用的。
harbor-log：收集其他 harbor 的日志信息。
harbor-ui：一个用户界面模块，用来管理 registry。
registry：存储 docker images 的服务，并且提供 pull/push 服务。
redis：存储缓存信息
webhook：当 registry 中的 image 状态发生变化的时候去记录更新日志、复制等操作。
token service：在 docker client 进行 pull/push 的时候负责 token 的发放。
```

## 4.1 安装 Harbor

**下载地址：** https://github.com/vmware/harbor/releases

**安装文档：** https://goharbor.io/docs/2.2.0/

### 4.1.1 安装 docker

本次使当前harbor最新的稳定版本2.2.1离线安装包 ，具体名称为 harbor-offline-installer-v2.2.1.tgz
```
[19:42:52 root@ubuntu18-04 ~]#hostnamectl set-hostname harbor1.zhangzhuo.org
[19:51:24 root@harbor ~]#hostname -I
192.168.10.185
#安装docker并且启动
[19:53:14 root@harbor ~]#systemctl is-active docker
active
```

### 4.1.2 下载 Harbor 安装包
```
#推荐使用离线完整安装包
[19:56:10 root@harbor1 ~]#cd /usr/local/src/
[19:56:54 root@harbor1 src]#ls
harbor-offline-installer-v2.2.1.tgz
```

## 4.2 配置 Harbor

### 4.2.1 解压并编辑 harbor.yml配置文件
之前的版本配置文件名称为harbor.cfg
```
[19:56:56 root@harbor1 src]#tar xf harbor-offline-installer-v2.2.1.tgz 
[19:58:15 root@harbor1 src]#ln -s /usr/local/src/harbor /usr/local/
[19:59:00 root@harbor1 harbor]#ls
common.sh  harbor.v2.2.1.tar.gz  harbor.yml.tmpl  install.sh  LICENSE  prepare
[19:59:01 root@harbor1 harbor]#cp harbor.yml.tmpl harbor.yml
#安装python环境
[20:00:00 root@harbor1 harbor]#apt install python-pip -y
#编辑配置文件
[20:05:35 root@harbor1 harbor]#grep -vE "#|^$" harbor.yml
hostname: harbor1.zhangzhuo.org   #这里修改
http:
  port: 80
harbor_admin_password: 123456     #admin管理员登录密码
database:
  password: root123
  max_idle_conns: 50
  max_open_conns: 1000
data_volume: /docker/images       #镜像存放位置
trivy:
  ignore_unfixed: false
  skip_update: false
  insecure: false
jobservice:
  max_job_workers: 10
notification:
  webhook_job_max_retry: 10
chart:
  absolute_url: disabled
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor
_version: 2.2.0
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy
```

## 4.3 启动 Harbor

### 4.3.1 安装docker-compose

启动之前先确保，本机已经安装docker-compose命令，如未安装会报错，请先安装。

docker-compose下载位置：
```
#安装
[20:12:41 root@docker ~]#ls
docker-compose-Linux-x86_64
[20:12:45 root@docker ~]#chmod +x docker-compose-Linux-x86_64 
[20:12:53 root@docker ~]#cp docker-compose-Linux-x86_64 /usr/bin/docker-compose
[20:13:11 root@docker ~]#docker-compose version
docker-compose version 1.29.1, build c34c88b2
docker-py version: 5.0.0
CPython version: 3.7.10
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```

### 4.3.2 安装
```
[20:10:33 root@harbor1 harbor]#pwd
/usr/local/harbor
[20:13:51 root@harbor1 harbor]#./install.sh 
[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating harbor-portal ... done
Creating registryctl   ... done
Creating harbor-db     ... done
Creating registry      ... done
Creating redis         ... done
Creating harbor-core   ... done
Creating harbor-jobservice ... done
Creating nginx             ... done
✔ ----Harbor has been installed and started successfully.----
```

### 4.3.3 登录web页面


win电脑需要在hosts文件添加域名才可以使用域名访问，也可以直接使用IP地址访问

## 4.4 停止与启动和重启Harbor
```
[20:40:40 root@harbor1 ~]#cd /usr/local/harbor
[20:43:20 root@harbor1 harbor]#pwd
/usr/local/harbor
#停止harbor
[20:43:44 root@harbor1 harbor]#docker-compose stop
[20:45:46 root@harbor1 harbor]#docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
#启动harbor
[20:45:50 root@harbor1 harbor]#docker-compose start
#重启harbor
[20:50:34 root@harbor1 harbor]#docker-compose restart
```

## 4.5 更新harbor配置文件
```
[20:43:20 root@harbor1 harbor]#pwd
/usr/local/harbor
[20:47:45 root@harbor1 harbor]#./prepare --help
prepare base dir is set to /usr/local/src/harbor
Usage: main.py prepare [OPTIONS]

Options:
  --conf TEXT         the path of Harbor configuration file
  --with-notary       the Harbor instance is to be deployed with notary
  --with-trivy        the Harbor instance is to be deployed with Trivy
  --with-chartmuseum  the Harbor instance is to be deployed with chart
                      repository supporting

  --help              Show this message and exit.
Clean up the input dir
#如果配置文件修改需要直接执行./prepare，后执行
```
## 4.6 开启harbor镜像扫描功能
```
[20:43:44 root@harbor1 harbor]#docker-compose stop
#删除所有镜像
[20:43:44 root@harbor1 harbor]#docker-compose rm
#重新配置
[21:04:52 root@harbor1 harbor]#./install.sh --with-trivy
```

## 4.7 harbor https 配置
```
[13:40:49 root@harbor1 harbor]#pwd
/etc
[13:41:09 root@harbor1 harbor]#mkdir certs
#生成自签证书
[13:41:09 root@harbor1 harbor]#cd certs
[13:41:22 root@harbor1 harbor]#openssl genrsa -out harbor-ca.key 2048
[13:47:07 root@harbor1 harbor]#openssl req -x509 -new -nodes -key harbor-ca.key -subj "/CN=harbor1.zhangzhuo.org" -days 7120 -out harbor-ca.crt
#修改harbor配置文件
[13:50:27 root@harbor1 harbor]#vim harbor.yml
hostname: harbor1.zhangzhuo.org
https:
  port: 443 
  certificate: /etc/certs/harbor-ca.crt                        
  private_key: /etc/certs/harbor-ca.key
#停止并删除harbor
[13:51:56 root@harbor1 harbor]#docker-compose stop 
[13:52:40 root@harbor1 harbor]#docker-compose rm
#重新执行安装脚本
[13:53:31 root@harbor1 harbor]#./install.sh --with-trivy #--with-trivy这个参数为开启扫描功能
#注意自签名证书是不被信任的所以，还需要配置 --insecure-registry harbor1.zhangzhuo.org才可以使用，或者直接使用授权证书
```

## 4.8 配置 docker 使用 harbor 仓库上传下载镜像

### 4.8.1 编辑 docker 配置文件
注意：如果我们配置的是 https 的话，本地 docker 就不需要有任何操作就可以访问harbor了
```
[20:13:24 root@docker ~]#vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/contain    erd.sock --insecure-registry harbor1.zhangzhuo.org
#配置dns解析
[20:20:25 root@docker ~]#vim /etc/hosts
192.168.10.185 harbor1.zhangzhuo.org
#重启服务
[20:21:31 root@docker ~]#systemctl daemon-reload 
[20:21:53 root@docker ~]#systemctl restart docker.service
```

### 4.8.2 验证能否登录 harbor
```
[20:22:06 root@docker ~]#docker login harbor1.zhangzhuo.org
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded  #出现这个表示登录成功
```

### 4.8.3 测试上传和下载镜像
将之前单机仓库构构建的 Nginx 镜像上传到 harbor 服务器用于测试

#### 4.8.3.1 导入镜像
```
#先给镜像打tag
[20:26:09 root@docker ~]#docker tag centos-base:v1 harbor1.zhangzhuo.org/imagesbase/centos-base:v1

#上传如果提示project imagesbase not found表示没有这个项目需要先去web端创建项目
[20:28:15 root@docker ~]#docker push harbor1.zhangzhuo.org/imagesbase/centos-base:v1
The push refers to repository [harbor1.zhangzhuo.org/imagesbase/centos-base]
f317d1b8cc66: Preparing 
996be175afdf: Preparing 
b2175d74c6e4: Preparing 
174f56854903: Preparing 
unauthorized: project imagesbase not found: project imagesbase not found

#创建后再次上传
[20:28:35 root@docker ~]#docker push harbor1.zhangzhuo.org/imagesbase/centos-base:v1
The push refers to repository [harbor1.zhangzhuo.org/imagesbase/centos-base]
f317d1b8cc66: Pushed 
996be175afdf: Pushed 
b2175d74c6e4: Pushed 
174f56854903: Pushed 
v1: digest: sha256:3048cfd8534a9cb73a9b9fdd17f8567294831d768e4a5675f01fd850db314ef0 size: 1157
```

新建项目时，勾选公开表示可以匿名下载本项目中镜像，但是上传必须登录，如果不勾选表示上传和下载都得登录

#### 4.8.3.2 下载镜像

下载镜像得配置docker.server文件，如果启用https及不用配置
```
#下载时可以写IP地址，如写域名需要配置dns
[20:36:41 root@harbor1 harbor]#docker pull 192.168.10.185/imagesbase/centos-base:v1
v1: Pulling from imagesbase/centos-base
2d473b07cdd5: Pull complete 
6c5db08bbd44: Pull complete 
058588353ab7: Pull complete 
0484442ee62e: Pull complete 
Digest: sha256:3048cfd8534a9cb73a9b9fdd17f8567294831d768e4a5675f01fd850db314ef0
Status: Downloaded newer image for 192.168.10.185/imagesbase/centos-base:v1
192.168.10.185/imagesbase/centos-base:v1
[20:37:18 root@harbor1 harbor]#docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
192.168.10.185/imagesbase/centos-base   v1                  b43d61fb8f07        2 days ago          482MB
#启动验证
[20:39:25 root@harbor1 harbor]#docker run -it --rm 192.168.10.185/imagesbase/centos-base:v1
[root@59937c2417e4 /]# ls
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

## 4.9 实现 harbor 高可用
高可用实现方式



Harbor 支持基于策略的 Docker 镜像复制功能，这类似于 MySQL 的主从同步， 其可以实现不同的数据中心、不同的运行环境之间同步镜像，并提供友好的管理界面，大大简化了实际运维中的镜像管理工作，已经有用很多互联网公司使用 harbor 搭建内网 docker 仓库的案例，并且还有实现了双向复制的案列，本文将实现单向复制的部署

### 4.9.1 实现双向同步复制的高可用
架构图



#### 4.9.1.1 部署harbor的同步集群
```
#安装俩台harbor主机
#首先安装好docker-ce，这里过程略过
#配置文件harbor.yml
[19:50:26 root@ubuntu18-04 harbor]#vim harbor.yml
hostname: 192.168.10.184   #这里只能写这个机器的IP地址，写域名会导致创建目标仓库测试失败
#其余默认，如开启https需要创建目标仓库时需要写https
#启动
[19:57:52 root@ubuntu18-04 harbor]#./install.sh
```


在仓库管理中点新建

这里目标名称可以随便写，目标URL如使用https写https，访问ID访问密码按自己的情况填写，验证证书不要勾选，之后点测试连接，然后点确定

**我这里192.168.10.185有镜像192.168.10.184没有镜像第一次需要同步一次**


名称：随便写

复制模式：第一次选Pull-based，表示从远端拉取

触发模式：手动

其他，默认

**之后选中，手动复制下，复制完之后删除这个复制规则新建实时同步规则**

俩端都设置

触发模式改为：事件驱动

复制模式：push-based，表示推送

**测试**
```
#测试1
[20:29:50 root@docker ~]#cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 zhang
192.168.10.184 harbor1.zhangzhuo.org
192.168.10.185 harbor2.zhangzhuo.org
[20:30:24 root@docker ~]#docker login harbor1.zhangzhuo.org
Login Succeeded
[20:30:33 root@docker ~]#docker tag mysql:latest harbor1.zhangzhuo.org/mysql/mysql:v1
#上传
[20:31:18 root@docker ~]#docker push harbor1.zhangzhuo.org/mysql/mysql:v1
#删除
[20:32:55 root@docker ~]#docker rmi harbor1.zhangzhuo.org/mysql/mysql:v1 mysql:latest
#去另一台拉取
[20:33:17 root@docker ~]#docker pull harbor2.zhangzhuo.org/mysql/mysql:v1
#测试2
[20:34:14 root@docker ~]#docker login harbor2.zhangzhuo.org
Login Succeeded
[20:34:39 root@docker ~]#docker tag harbor2.zhangzhuo.org/mysql/mysql:v1 harbor2.zhangzhuo.org/mysql/mysql:v2
[20:36:23 root@docker ~]#docker push harbor2.zhangzhuo.org/mysql/mysql:v2
#去另一台拉取
[20:37:10 root@docker ~]#docker pull harbor2.zhangzhuo.org/mysql/mysql:v2
#如果全部成功说明同步没有问题
```

#### 4.9.1.2 安装haproxy
```
#俩太都执行
[21:07:08 root@haproxy1 ~]#yum install haproxy
[21:08:01 root@haproxy1 ~]#vim /etc/sysctl.conf
net.ipv4.ip_nonlocal_bind = 1
[21:10:00 root@haproxy1 ~]#sysctl -p
[21:08:01 root@haproxy1 ~]#vim /etc/haproxy/haproxy.cfg
listen web_host_staticrr
    bind 192.168.10.100:80,192.168.10.100:443
    mode tcp
    log global
    balance source    #使用源地址hash
    option forwardfor
    server harbor1 192.168.10.184:443 weight 1 check inter 3000 fall 3 rise 5
    server harbor2 192.168.10.185:443 weight 2 check inter 3000 fall 3 rise 5
[21:10:36 root@haproxy1 ~]#systemctl enable --now haproxy.service
#测试
```

#### 4.9.1.3 安装keepalive
```
[21:38:02 root@haproxy1 keepalived]#yum install keepalived
#ke1配置
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 80
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    unicast_src_ip 192.168.10.71
    unicast_peer{
        192.168.10.72
    }
    virtual_ipaddress {
        192.168.10.100/24
    }
}
#ke2配置
vrrp_instance VI_1 {
    state BACKUP   
    interface eth0  
    virtual_router_id 80 
    priority 80    
    advert_int 1    
    authentication { 
        auth_type PASS
        auth_pass 1111   
    }
    unicast_src_ip 192.168.10.72
    unicast_peer{
        192.168.10.71
    }  
    virtual_ipaddress { 
        192.168.10.100/24 
    }
}
#启动全部
[21:39:59 root@haproxy2 keepalived]#systemctl enable --now  keepalived.servic
[21:40:27 root@haproxy1 keepalived]#ip a
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.10.100/24 scope global secondary eth0
       valid_lft forever preferred_lft foreve
```

#### 4.9.1.4 docker测试
```
[21:41:36 root@docker ~]#cat /etc/hosts
192.168.10.100 harbor.zhangzhuo.org
[21:42:19 root@docker ~]#cat /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry harbor.zhangzhuo.org
[21:42:21 root@docker ~]#systemctl daemon-reload
#登录
[21:43:11 root@docker ~]#docker login harbor.zhangzhuo.org
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
#上传
[21:44:09 root@docker ~]#docker tag nginx:latest harbor.zhangzhuo.org/nginx/nginx:latest
[21:46:23 root@docker ~]#docker push harbor.zhangzhuo.org/nginx/nginx:latest
#下载
[21:47:45 root@docker ~]#docker pull harbor.zhangzhuo.org/nginx/nginx:latest
#测试如果没有问题就可以正常使用了
```

# 五、Docker 数据管理
如果正在运行中的容器如果生成了新的数据或者修改了现有的一个已经存在的文件内容，那么新产生的数据将会被复制到读写层进行持久化保存，这个读写层也就是容器的工作目录，此即“写时复制(COW) copy on write”机制。

如下图是将对根的数据写入到了容器的可写层，但是把/data 中的数据写入到 了一个另外的 volume 中用于数据持久化。



5.1 数据类型
Docker 的镜像是分层设计的，镜像层是只读的，通过镜像启动的容器添加了 一层可读写的文件系统，用户写入的数据都保存在这一层当中。

如果要将写入到容器的数据永久保存，则需要将容器中的数据保存到宿主机的指定目录，目前 Docker 的数据类型分为两种:

一是数据卷(data volume)，数据卷类似于挂载的一块磁盘，数据容器是将数据保存在一个容器上。

二是数据卷容器(Data volume container), 数据卷容器是将宿主机的目录挂载至一个专门的数据卷容器，然后让其他容器通过数据卷容器读写宿主机的数据。


```
[17:13:46 root@docker centos]#docker inspect fbc376847a52  #查看指定PID的容器信息
"Data": {
                "LowerDir": "/var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87-init/diff:/var/lib/docker/overlay2/fbd8bf0facabebe6a58c40d185c7cfb45de8dfa92c5780826d405a744ccffaf3/diff:/var/lib/docker/overlay2/3b2fd7d1c147d15ebfe2413a6c9409cb9fb63c23c9294dddbca671de279c02e0/diff:/var/lib/docker/overlay2/25b14dc5795af105b89e75704c8e50f7d2cb2c6a8a1638d02ec64c305a57a386/diff:/var/lib/docker/overlay2/4689f5e9600fedc55dfca4fc36b2e0e5f06e2dbcdfac806c59915ed523ac541f/diff:/var/lib/docker/overlay2/3bb7ce46cdaf22d75e98a4297800dcab0000c01446e0b6dda45db9c3b38fc8d4/diff:/var/lib/docker/overlay2/790e3ed4a87f044e6be647dc177f96db71c85424f15340da1f4c862323bb7e6d/diff:/var/lib/docker/overlay2/93912a2163973ba847eb54ae940578d02b5f65df0d5d8198126fe888f87832f7/diff:/var/lib/docker/overlay2/1cb38822435dfe857960b53409a064678c73959dd72b5f800438d1850f01dd8d/diff:/var/lib/docker/overlay2/64198d25c1618b57963d9660e8c7d3d0d80136727ab35274e7470242fdec96d7/diff:/var/lib/docker/overlay2/124713fdec8e3abf67e6f33cc1abe17a897fd094e63c71e88e061a6f2e108ec6/diff:/var/lib/docker/overlay2/139308dc520f6ce67f52af09ddf2c04b3557002996ac7debbeea1ad0d32d3e65/diff:/var/lib/docker/overlay2/3931472b2f005a753d51d428f98563707d0a392b2c7e192f1702b4d5649a1ec5/diff:/var/lib/docker/overlay2/af7e16973910ee4e83806651b8a30b538beacb875160ec54d9ac9efa36d7e099/diff",
                "MergedDir": "/var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87/merged",
                "UpperDir": "/var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87/diff",
                "WorkDir": "/var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87/work"
            },

#说明
#Lower Dir：image 镜像层(镜像本身，只读)
#Upper Dir：容器的上层(读写)
#Merged Dir：容器的文件系统，使用 Union FS（联合文件系统）将 lowerdir 和 upper Dir：合并给容器使用。
#Work Dir：容器在宿主机的工作目录
```

在容器生成数据
```
[17:18:32 root@docker centos]#docker-in.sh fbc376847a52
[root@fbc376847a52 /]# dd if=/dev/zero of=file bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB) copied, 0.212477 s, 494 MB/s
[root@fbc376847a52 /]# md5sum file
2f282b84e7e608d5852449ed940bfc51  file
```

数据在宿主机哪里？
```
[17:20:21 root@docker centos]#ll /var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87/diff
total 102432
drwxr-xr-x 6 root root      4096 Apr 15 17:19 ./
drwx-----x 5 root root      4096 Apr 15 17:12 ../
drwxr-xr-x 3 2020 2020      4096 Apr 15 09:37 apps/
-rw-r--r-- 1 root root 104857600 Apr 15 17:19 file
dr-xr-x--- 2 root root      4096 Apr 15 17:20 root/
drwxrwxrwt 3 root root      4096 Apr 15 17:13 tmp/
drwxr-xr-x 3 root root      4096 Nov 13 09:54 var/
[17:20:56 root@docker centos]#md5sum /var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87/diff/file 
2f282b84e7e608d5852449ed940bfc51  /var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87/diff/file  #验证file文件的md5值是否和容器中一样
```

当容器被删除后，宿主机的数据还在吗？
```
[17:21:29 root@docker centos]#docker rm -f fbc376847a52
fbc376847a52
[17:25:21 root@docker centos]#ll /var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87/diff
ls: cannot access '/var/lib/docker/overlay2/5e1e19f7281ab1c42dbeb2343bc98e30ef2926d852ad062ddcc76950d0d38e87/diff': No such file or directory
#不存在了
```

### 5.1.1 什么是数据卷(data volume）
数据卷实际上就是宿主机上的目录或者是文件，可以被直接 mount 到容器当中使用。

实际生产环境中，需要针对不同类型的服务、不同类型的数据存储要求做相应的规划，最终保证服务的可扩展性、稳定性以及数据的安全性。

如下图：

左侧是无状态的 http 请求服务，右侧为有状态。 下层为不需要存储的服务，上层为需要存储的部分服务。



#### 5.1.1.1 创建 APP 目录并生成 web 页面
此 app 以数据卷的方式，提供给容器使用，比如容器可以直接宿主机本地的 web app，而需要将代码提前添加到容器中，此方式适用于小型 web 站点。
```
[16:00:42 root@docker ~]#mkdir /data/testapp -p
[16:00:58 root@docker ~]#echo "testapp page" >/data/testapp/index.html
[16:01:32 root@docker ~]#cat /data/testapp/index.html
testapp page
```

#### 5.1.1.2 启动容器并验证数据
启动两个容器，web1 容器和 web2 容器，分别测试能否在宿主机访问到宿主机的数据。
```
#注意使用-v 参数，将宿主机目录映射到容器内部，web2 的 ro 标示在容器内对该目录只读，默认是可读写的
#下载之前制作的tomcat镜像
[16:05:03 root@docker ~]#docker pull harbor.zhangzhuo.org/web1/tomcat-web:app1
[16:06:40 root@docker ~]#docker run -it -d --name web1 -v /data/testapp/:/apps/tomcat/webapps/testapp -p 8080:8080 harbor.zhangzhuo.org/web1/tomcat-web:app1 
afad227a55533604cc72a806d99aaf972e3f12a204b56761d742ea9e2ccf8a3f
#设置目录为只读
[16:08:02 root@docker ~]#docker run -it -d --name web2 -v /data/testapp/:/apps/tomcat/webapps/testapp:ro -p 8081:8080 harbor.zhangzhuo.org/web1/tomcat-web:app1 
641b10dcd2772113d1e9733226885ef3a7067aec03a35cd7a0b0ef66f48bf21d
```

#### 5.1.1.3 进入到容器内测试写入数据
```
[16:08:21 root@docker ~]#docker exec -it web1 bash
[root@afad227a5553 /]# cat /apps/tomcat/webapps/testapp/index.html 
testapp page
[root@afad227a5553 /]# exit
exit
[16:09:46 root@docker ~]#docker exec -it web2 bash
[root@641b10dcd277 /]# cat /apps/tomcat/webapps/testapp/index.html 
testapp page
[root@641b10dcd277 /]# exit
exit
```

#### 5.1.1.4 宿主机验证
验证宿主机的数据是否正常
```
[16:10:03 root@docker ~]#cat /data/testapp/index.html 
testapp page
````

#### 5.1.1.5 web 界面访问
```
[16:10:58 root@docker ~]#curl 192.168.10.181:8080/testapp/
testapp page
[16:11:43 root@docker ~]#curl 192.168.10.181:8081/testapp/
testapp page
```

#### 5.1.1.6 在宿主机或容器修改数据
```
[16:11:47 root@docker ~]#echo "web v2" >> /data/testapp/index.html 
[16:12:27 root@docker ~]#cat /data/testapp/index.html
testapp page
web v2
```

#### 5.1.1.7 web 端访问验证数据
```
[16:12:32 root@docker ~]#curl 192.168.10.181:8080/testapp/
testapp page
web v2
[16:13:12 root@docker ~]#curl 192.168.10.181:8081/testapp/
testapp page
web v2
```

#### 5.1.1.8 删除容器
```
#创建容器的时候指定参数-v，可以删除/var/lib/docker/containers/的容器数据目录，但是不会删除数据卷的内容，如下：
[16:13:15 root@docker ~]#docker ps
CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
641b10dcd277        harbor.zhangzhuo.org/web1/tomcat-web:app1   "/apps/tomcat/bin/ru…"   5 minutes ago       Up 5 minutes        0.0.0.0:8081->8080/tcp   web2
afad227a5553        harbor.zhangzhuo.org/web1/tomcat-web:app1   "/apps/tomcat/bin/ru…"   6 minutes ago       Up 6 minutes        0.0.0.0:8080->8080/tcp   web1
[16:14:14 root@docker ~]#docker rm -f `docker ps -qa`
641b10dcd277
afad227a5553
```

#### 5.1.1.9 验证宿主机的数据
```
[16:14:40 root@docker ~]#cat /data/testapp/index.html 
testapp page
web v2
```

#### 5.1.1.10 数据卷的特点及使用
```
#特点
1. 数据卷是宿主机的目录或者文件，并且可以在多个容器之间共同使用。
2. 在宿主机对数据卷更改数据后会在所有容器里面会立即更新。
3. 数据卷的数据可以持久保存，即使删除使用使用该容器卷的容器也不影响。
4. 在容器里面的写入数据不会影响到镜像本身。
5. 文件权限是直接继承宿主机上面的文件权限，指定的读写权限是指容器是否由读写权限
#使用场景
1. 日志输出
2. 静态 web 页面
3. 应用配置文件
4. 多容器间目录或文件共享
```

#### 5.1.1.11 文件挂载
文件挂载用于很少更改文件内容的场景，比如 nginx 的配置文件、tomcat 的配置文件等。

**创建容器并挂载配置文件**
```
[16:29:00 root@docker ~]#vim /data/testapp/catalina.sh
#添加下面参数
JAVA_OPTS="-server -Xms512m -Xmx512m -Xss512k -Xmn512m -XX:CMSInitiatingOccupancyFraction=65 -XX:+AggressiveOpts -XX:+UseBiasedLocking -XX:+DisableExplicitGC -XX:MaxTenuringThreshold=10 -XX:NewRatio=2 -XX:PermSize=64m -XX:MaxPermSize=128m -XX:CMSFullGCsBeforeCompaction=5 -XX:+ExplicitGCInvokesConcurrent -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=64m -XX:+UseFastAccessorMethods"
[16:29:00 root@docker ~]#ll /data/testapp/
total 40
drwxr-xr-x 2 root root  4096 Apr 18 16:29 ./
drwxr-xr-x 4 root root  4096 Apr 18 16:00 ../
-rwxr-xr-x 1 root root 25769 Apr 18 16:28 catalina.sh*
-rw-r--r-- 1 root root    20 Apr 18 16:12 index.html
```

**创建容器：**
```
[16:29:40 root@docker ~]#docker run -it -d -p 8080:8080 -v /data/testapp/catalina.sh:/apps/tomcat/bin/catalina.sh:ro harbor.zhangzhuo.org/web1/tomcat-web:app1 
a87be633f858bb56029924fb93b11bd525e8b5f16ac4526979e25b1865e150df
```

**验证参数生效**
```
[16:31:49 root@docker ~]#docker ps
CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
a87be633f858        harbor.zhangzhuo.org/web1/tomcat-web:app1   "/apps/tomcat/bin/ru…"   59 seconds ago      Up 57 seconds       0.0.0.0:8080->8080/tcp   dreamy_bell
[16:57:27 root@docker testapp]#ps -ef | grep tomcat
root       9208   2057  0 17:02 pts/0    00:00:00 docker run -it -p 8080:8080 -v /data/testapp/catalina.sh:/apps/tomcat/bin/catalina.sh:ro harbor.zhangzhuo.org/web1/tomcat-web:app1
root       9259   9238  0 17:02 pts/0    00:00:00 /bin/bash /apps/tomcat/bin/run_tomcat.sh
2020       9339   9259 14 17:02 ?        00:00:03 /usr/local/jdk/jre/bin/java -Djava.util.logging.config.file=/apps/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms512m -Xmx512m -Xss512k -Xmn512m -XX:CMSInitiatingOccupancyFraction=65 -XX:+AggressiveOpts -XX:+UseBiasedLocking -XX:+DisableExplicitGC -XX:MaxTenuringThreshold=10 -XX:NewRatio=2 -XX:PermSize=64m -XX:MaxPermSize=128m -XX:CMSFullGCsBeforeCompaction=5 -XX:+ExplicitGCInvokesConcurrent -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=64m -XX:+UseFastAccessorMethods -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /apps/tomcat/bin/bootstrap.jar:/apps/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/apps/tomcat -Dcatalina.home=/apps/tomcat -Djava.io.tmpdir=/apps/tomcat/temp org.apache.catalina.startup.Bootstrap start
root       9394   8389  0 17:03 pts/1    00:00:00 grep --color=auto tomcat
```

**进入容器测试文件读写**
```
[17:03:05 root@docker testapp]#docker exec -it silly_solomon bash
[root@2ac6e21470a0 /]# echo "test" >> /apps/tomcat/bin/catalina.sh 
bash: /apps/tomcat/bin/catalina.sh: Read-only file system
[root@2ac6e21470a0 /]# ll /apps/tomcat/bin/catalina.sh
-rwxr-xr-x 1 root root 25763 Apr 18 16:57 /apps/tomcat/bin/catalina.sh
```

#### 5.1.1.12 如何一次挂载多个目录或文件
多个目录可以位于不同的目录下
```
[17:07:30 root@docker testapp]#mkdir /data/zhangzhuo
[17:08:17 root@docker testapp]#echo "zhangzhuo" >> /data/zhangzhuo/index.html
[17:08:32 root@docker testapp]#docker run -it -d --name web1 -p 8080:8080 -v /data/testapp/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /data/zhangzhuo/:/apps/tomcat/webapps/testapp harbor.zhangzhuo.org/web1/tomcat-web:app1 
aa0440ff8aa77063b9da835aa59b526e5e3914ef242adc79e8ef06fe1b20b9f2
#再启动一个容器，验证宿主机目录或文件是否共享
[17:10:09 root@docker testapp]#docker run -it -d --name web2 -p 8081:8080 -v /data/testapp/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /data/zhangzhuo/:/apps/tomcat/webapps/testapp harbor.zhangzhuo.org/web1/tomcat-web:app1 
31b7e98f8a978352fc6dd1241d430ceffdd91a2017c800ddc9234bef023ef248
#测试
[17:11:05 root@docker testapp]#curl 192.168.10.181:8080/testapp/
zhangzhuo
[17:11:17 root@docker testapp]#curl 192.168.10.181:8081/testapp/
zhangzhuo
```

### 5.1.2 数据卷容器
数据卷容器功能是可以让数据在多个docker容器之间共享，即可以让 B 容器访问 A 容器的内容，而容器 C 也可以访问 A 容器的内容，即先要创建一个后台运行的容器作为 Server，用于卷提供，这个卷可以为其他容器提供数据存储服务，其他使用此卷的容器作为client端

#### 5.1.2.1 启动一个卷容器 Server
先启动一个容器，并挂载宿主机的数据目录：

将宿主机的 catalina.sh 启动脚本和 zhangzhuo 的 web 页面，分别挂载到卷容器 server 端，然后通过 server 端共享给 client 端使用。
```
[17:19:55 root@docker testapp]#docker rm -f `docker ps -qa`

[17:26:17 root@docker testapp]#docker run -d --name volume-server -v /data/testapp/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /data/zhangzhuo/:/apps/tomcat/webapps/zhangzhuo/ harbor.zhangzhuo.org/web1/tomcat-web:app1 
6557b41a72fee9f7d139cd392e13425d889bc14905337b3893b8affcabcf4e96
```

#### 5.1.2.2 启动两个端容器 Client
```
[17:21:42 root@docker testapp]#docker run -d --name web1 -p 8080:8080 --volumes-from volume-server harbor.zhangzhuo.org/web1/tomcat-web:app1 
053a071e53feb20f3af63b752f72d0a110ab5f9cf08090bf3aaa90f470f48f8f
[17:23:04 root@docker testapp]#docker run -d --name web2 -p 8081:8080 --volumes-from volume-server harbor.zhangzhuo.org/web1/tomcat-web:app1 
f79d7552e6be9b1ea51eb67bf77fbd2914a2d4e0f9b572516be0b5c380844747
```

#### 5.1.2.3 分别进入容器测试读写
读写权限依赖于源数据卷 Server 容器
```
[17:27:02 root@docker testapp]#docker exec -it volume-server bash
[root@6557b41a72fe /]# cat /apps/tomcat/webapps/zhangzhuo/index.html 
zhangzhuo
[root@6557b41a72fe /]# echo "zhangzhuo" >> /apps/tomcat/webapps/zhangzhuo/index.html    #可写文件权限
[root@6557b41a72fe /]# echo "zhangzhuo" >> /apps/tomcat/bin/catalina.sh 
bash: /apps/tomcat/bin/catalina.sh: Read-only file system  #只读文件权限
```

#### 5.1.2.4 测试访问 web 页面
```
[17:28:37 root@docker testapp]#curl  192.168.10.181:8080/zhangzhuo/
zhangzhuo
zhangzhuo
[17:29:42 root@docker testapp]#curl  192.168.10.181:8081/zhangzhuo/
zhangzhuo
zhangzhuo
```

#### 5.1.2.5 验证宿主机数据
```
[17:29:45 root@docker testapp]#cat /data/zhangzhuo/index.html 
zhangzhuo
zhangzhuo
```

#### 5.1.2.6 关闭卷容器 Server 测试能否启动新容器
```
[17:30:18 root@docker testapp]#docker stop volume-server 
volume-server
[17:32:08 root@docker testapp]#docker run -d --name web3 -p 8082:8080 --volumes-from volume-server harbor.zhangzhuo.org/web1/tomcat-web:app1 
5289c972a521b93c29db55104b809e7326d88675d29b9d7bd22f0f4ba3920d8
#测试停止完成之后能否创建新的容器，停止 volume server 是可以创建新容器的。
```

#### 5.1.2.7 测试删除源卷容器 Server 创建容器
```
[17:32:27 root@docker testapp]#docker rm -f volume-server 
volume-server
[17:33:18 root@docker testapp]#docker run -d --name web4 -p 8083:8080 --volumes-from volume-server harbor.zhangzhuo.org/web1/tomcat-web:app1 
Unable to find image 'harbor.zhangzhuo.org/web1/tomcat-web:app1' locally
app1: Pulling from web1/tomcat-web
Digest: sha256:27eaa52e436c76aae7204e5616ffe7831153ec285558258cfc907c43c5d88691
Status: Image is up to date for harbor.zhangzhuo.org/web1/tomcat-web:app1
docker: Error response from daemon: No such container: volume-server.
See 'docker run --help'.
#删除后无法在创建新容器
#测试之前容器是否正常，是正常的
[17:33:29 root@docker testapp]#curl  192.168.10.181:8080/zhangzhuo/
zhangzhuo
zhangzhuo
[17:34:41 root@docker testapp]#curl  192.168.10.181:8081/zhangzhuo/
zhangzhuo
zhangzhuo
[17:34:45 root@docker testapp]#curl  192.168.10.181:8082/zhangzhuo/
zhangzhuo
zhangzhuo
```

#### 5.1.2.8 卷容器说明
```
1.在当前环境下，即使把提供卷的容器 Server 删除，已经运行的容器 Client 依然可以使用挂载的卷，因为容器是通过挂载访问数据的，但是无法创建新的卷容器客户端，但是再把卷容器 Server 创建后即可正常创建卷容器 Client，此方式可以用于线上共享数据目录等环境，因为即使数据卷容器被删除了，其他已经运行的容器依然可以挂载使用
2.数据卷容器可以作为共享的方式为其他容器提供文件共享，类似于 NFS 共享，可以在生产中启动一个实例挂载本地的目录，然后其他的容器分别挂载此容器的目录，即可保证各容器之间的数据一致性。
```

# 六、网络部分
主要介绍 docker 网络相关知识。 Docker 服务安装完成之后，默认在每个宿主机会生成一个名称为 docker0 的网卡其 IP 地址都是 172.17.0.1/16，并且会生成三种类型的网络
```
[17:40:39 root@docker ~]#ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:b9ff:fe3a:f343  prefixlen 64  scopeid 0x20<link>
        ether 02:42:b9:3a:f3:43  txqueuelen 0  (Ethernet)
        RX packets 149  bytes 152648 (152.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 241  bytes 21842 (21.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[17:40:57 root@docker ~]#docker network list
NETWORK ID          NAME                DRIVER              SCOPE
e171790c42e3        bridge              bridge              local
e541256bb224        host                host                local
245cd90d385d        none                null                local
```

## 6.1 容器之间的互联

### 6.1.1 通过容器名称互联
即在同一个宿主机上的容器之间可以通过自定义的容器名称相互访问，比如一 个业务前端静态页面是使用 nginx，动态页面使用的是 tomcat，由于容器在启动 的时候其内部 IP 地址是 DHCP 随机分配的，所以如果通过内部访问的话，自定义名称是相对比较固定的，因此比较适用于此场景。
```
#此方式最少需要两个容器之间操作
#先创建第一个容器，后续会使用到这个容器的名称
[19:12:03 root@docker ~]#docker run -it -d --name tomcat-web1 -p 8081:8080 harbor.zhangzhuo.org/web1/tomcat-web:app1 
a010add5581056084fbd893c65e055a53405daead9960e2306a9b2e67644f2b8
#查看当前 hosts 文件内容
[19:12:34 root@docker ~]#docker exec -it tomcat-web1 bash
[root@a010add55810 /]# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	a010add55810
#创建第二个容器
[19:13:59 root@docker ~]#docker run -it -d --name tomcat-web2 -p 8082:8080 harbor.zhangzhuo.org/web1/tomcat-web:app1 
30d311d31b2ddf3d72f59048ec65127647543d2c1c46f8b932103de5b2eeeb2c
#查看第二个容器的 hosts 文件内容
[root@30d311d31b2d /]# vim /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      30d311d31b2d
172.17.0.2      a010add55810 tomcat-web1
#检测
[root@30d311d31b2d /]# ping tomcat-web1
PING a010add55810 (172.17.0.2) 56(84) bytes of data.
64 bytes from a010add55810 (172.17.0.2): icmp_seq=1 ttl=64 time=0.105 ms
64 bytes from a010add55810 (172.17.0.2): icmp_seq=2 ttl=64 time=0.057 ms
```

### 6.1.2 通过自定义容器别名互联
上一步骤中，自定义的容器名称可能后期会发生变化，那么一旦名称发生变化， 程序之间也要随之发生变化，比如程序通过容器名称进行服务调用，但是容器名 称发生变化之后再使用之前的名称肯定是无法成功调用，每次都进行更改的话又 比较麻烦，因此可以使用自定义别名的方式解决，即容器名称可以随意更，只要 不更改别名即可，具体如下：

命令格式：
```
docker run -d --name 新容器名称 --link 目标容器名称:自定义的名称 -p 本地端口:容器端口 镜像名称 shell 命令
```

示例
```
[19:25:02 root@docker ~]#docker run -it -d -p 8083 --name tomcat-web3 --link tomcat-web1:java_server harbor.zhangzhuo.org/web1/tomcat-web:app1  
f07e792e1b5831c5f9bbd28e49e62588fdb5ae7c2947f2c607367d742758140c
#查看当前容器的 hosts 文件
[19:25:02 root@docker ~]#docker run -it -d -p 8083 --name tomcat-web3 --link tomcat-web1:java_server harbor.zhangzhuo.org/web1/tomcat-web:app1  
f07e792e1b5831c5f9bbd28e49e62588fdb5ae7c2947f2c607367d742758140c
[19:26:13 root@docker ~]#docker exec -it tomcat-web3 bash
[root@f07e792e1b58 /]# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	java_server a010add55810 tomcat-web1
172.17.0.4	f07e792e1b58
#检查别名通信
[root@f07e792e1b58 /]# ping java_server
PING java_server (172.17.0.2) 56(84) bytes of data.
64 bytes from java_server (172.17.0.2): icmp_seq=1 ttl=64 time=0.357 ms
```

## 6.2 docker 网络类型
Docker 的网络使用 docker network ls 命令看到有三种类型，下面将介绍每一种类型的具体工作方式

**Bridge 模式：** 使用参数 –net=bridge 指定，不指定默认就是 bridge 模式。
```
#查看当前docker的网卡信息
[19:30:03 root@docker ~]#docker network list
NETWORK ID          NAME                DRIVER              SCOPE
e171790c42e3        bridge              bridge              local
e541256bb224        host                host                local
245cd90d385d        none                null                local

Bridge：  #桥接，使用自定义 IP
Host：    #不获取 IP 直接使用物理机 IP，并监听物理机 IP 监听端口
None:     #没有网络
```

### 6.2.1 Host 模式
**Host 模式：**使用参数 –net=host 指定

启动的容器如果指定了使用 host 模式，那么新创建的容器不会创建自己的虚拟网卡，而是直接使用宿主机的网卡和 IP 地址，因此在容器里面查看到的 IP 信息就是宿主机的信息，访问容器的时候直接使用宿主机 IP+容器端口即可，不过容器的其他资源文件系统、系统进程等还是和宿主机保持隔离。

此模式的网络性能最高，但是各容器之间端口不能相同，适用于运行容器端口比 较固定的业务。

为避免端口冲突，先删除所有的容器

示例：
```
#启动一个容器并且指定网络模式为host
[19:34:01 root@docker ~]#docker run -it -d --name tomcat-web --net=host harbor.zhangzhuo.org/web1/tomcat-web:app1 
6b99a0a7b81a7bd32e60d4371736ec7dc2ae12b491166d8133ab14c4f7bce338
#验证网络信息
[19:34:11 root@docker ~]#docker exec -it tomcat-web bash
[root@docker /]# ifconfig 
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:b9ff:fe3a:f343  prefixlen 64  scopeid 0x20<link>
        ether 02:42:b9:3a:f3:43  txqueuelen 0  (Ethernet)
        RX packets 151  bytes 152704 (149.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 243  bytes 22022 (21.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.181  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fe17:38ab  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:17:38:ab  txqueuelen 1000  (Ethernet)
        RX packets 844756  bytes 1190154312 (1.1 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 81717  bytes 7172940 (6.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 192  bytes 19556 (19.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 192  bytes 19556 (19.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[19:35:39 root@docker ~]#ss -ntl | grep 8080
LISTEN  0        100                          *:8080                   *:*   
#测试访问
[19:35:48 root@docker ~]#curl 192.168.10.181:8080/myapp/
Tomcat Web Page1
```

Host 模式不支持端口映射，当指定端口映射的时候会提示如下警告信息：
```
[19:36:51 root@docker ~]#docker run -it -d --name tomcat-web --net=host -p 80:8080 harbor.zhangzhuo.org/web1/tomcat-web:app1 
WARNING: Published ports are discarded when using host network mode #警告信息
3c85490b3dc7ca9a18c3b7b2de1afeadb5c158629a61a33b7e420a09801939a4
#使用主机网络模式时，将丢弃已指定的端口
```

### 6.2.2 none 模式

**None 模式：** 使用参数 –net=none 指定

在使用 none 模式后，Docker 容器不会进行任何网络配置，其没有网卡、没有 IP 也没有路由，因此默认无法与外界通信，需要手动添加网卡配置 IP 等，所以极少使用

**命令使用方式**
```
[19:37:51 root@docker ~]#docker run -it -d --name net_none -p 8081:8080 --net=none harbor.zhangzhuo.org/web1/tomcat-web:app1 
338aa16d09f9e67128c707d7d2d1967185ec3837376fb1b6fac5a81d0bd407ee
[19:40:48 root@docker ~]#docker ps
CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS              PORTS               NAMES
338aa16d09f9        harbor.zhangzhuo.org/web1/tomcat-web:app1   "/apps/tomcat/bin/ru…"   21 seconds ago      Up 20 seconds                           net_none
[19:40:51 root@docker ~]#docker exec -it net_none bash
[root@338aa16d09f9 /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

### 6.2.3 Container 模式
**Container 模式：** 使用参数 –-net=container:名称或 ID

使用此模式创建的容器需指定和一个已经存在的容器共享一个网络，而不是和宿主机共享网，新创建的容器不会创建自己的网卡也不会配置自己的 IP，而是和 一个已经存在的被指定的容器的 IP 和端口范围，因此这个容器的端口不能和被指定的端口冲突，除了网络之外的文件系统、进程信息等仍然保持相互隔离， 两个容器的进程可以通过 lo 网卡及容器 IP 进行通信。
```
[19:41:46 root@docker ~]#docker run -it -d --name tomcat-web1 -p 8081:8080 --net=bridge harbor.zhangzhuo.org/web1/tomcat-web:app1 
9b4402dba4611692bd108b7be94f0f20f07108ec9ab7fb788c7f625a5c9896a0
[19:45:13 root@docker ~]#docker run -it -d --name tomcat-web2 --net=container:tomcat-web1 harbor.zhangzhuo.org/web1/tomcat-web:app1  
0f500e8117acabdd20e1f32879e12cc6036577836921dc61dbabfffa6428ef26
#直接使用对方的网络，此方式较少使用
#查看
[19:45:13 root@docker ~]#docker run -it -d --name tomcat-web2 --net=container:tomcat-web1 harbor.zhangzhuo.org/web1/tomcat-web:app1 
0f500e8117acabdd20e1f32879e12cc6036577836921dc61dbabfffa6428ef26
[19:46:11 root@docker ~]#docker exec -it tomcat-web2 bash
[root@9b4402dba461 /]# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	9b4402dba461
[root@9b4402dba461 /]# ss -ntl
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      1      127.0.0.1:8005                       *:*                  
LISTEN     0      100            *:8080                       *:*                  
[root@9b4402dba461 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 13  bytes 1046 (1.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 6.2.4 bridge 模式
docker 的默认模式即不指定任何模式就是 bridge 模式，也是使用比较多的模式， 此模式创建的容器会为每一个容器分配自己的网络 IP 等信息，并将容器连接到 一个虚拟网桥与外界通信。

```
#查看bridge详细信息
[19:50:53 root@docker ~]#docker inspect bridge 
[
    {
        "Name": "bridge",
        "Id": "e171790c42e3bdeda0d399f780e18935775cce0a09f931fabb136abceb1cfce3",
        "Created": "2021-04-18T12:06:19.629642167+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

## 6.3 docker 夸主机互联之简单实现
夸主机互联是说 A 宿主机的容器可以访问 B 主机上的容器，但是前提是保证各宿主机之间的网络是可以相互通信的，然后各容器才可以通过宿主机访问到对方的容器，实现原理是在宿主机做一个网络路由就可以实现 A 宿主机的容器访问 B 主机的容器的目的，复杂的网络或者大型的网络可以使用 google 开源的 k8s 进行互联。

### 6.3.1 修改各宿主机网段
Docker 的默认网段是 172.17.0.x/24,而且每个宿主机都是一样的，因此要做路由的前提就是各个主机的网络不能一致，具体如下：
```
#修改方式docker.service文件ExecStart中添加--bip=IP/掩码

#服务器docker1更改网段
[20:04:25 root@docker1 ~]#vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock     --bip=10.10.0.1/24 --insecure-registry harbor.zhangzhuo.org

#服务器docker2更改网段
[20:04:25 root@docker2 ~]#vim /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock     --bip=10.20.0.1/24 --insecure-registry harbor.zhangzhuo.org

#重启服务
[20:04:07 root@docker1 ~]#systemctl daemon-reload 
[20:04:13 root@docker1 ~]#systemctl restart docker.service 

#检查
[20:08:13 root@docker1 ~]#ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.10.0.1  netmask 255.255.255.0  broadcast 10.10.0.255
        inet6 fe80::42:b9ff:fe3a:f343  prefixlen 64  scopeid 0x20<link>
        ether 02:42:b9:3a:f3:43  txqueuelen 0  (Ethernet)
        RX packets 151  bytes 152704 (152.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 245  bytes 22202 (22.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[20:07:38 root@docker2 ~]#ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.20.0.1  netmask 255.255.255.0  broadcast 10.20.0.255
        ether 02:42:72:52:5e:6f  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 6.3.2 在两个宿主机分别启动一个容器
```
#docker1
[20:08:15 root@docker1 ~]#docker run -it -d --name tomcat-web1 harbor.zhangzhuo.org/web1/tomcat-web:app1 
fe08dbee85dee86a825c12e0e008dbe0cb824b6afff745b0159cd2f9e0107cd9
[20:10:07 root@docker1 ~]#docker exec -it tomcat-web1 bash
[root@fe08dbee85de /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.0.2  netmask 255.255.255.0  broadcast 10.10.0.255
        ether 02:42:0a:0a:00:02  txqueuelen 0  (Ethernet)
        RX packets 11  bytes 906 (906.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

#docker2
[20:08:26 root@docker2 ~]#docker run -it -d --name tomcat-web2 harbor.zhangzhuo.org/web1/tomcat-web:app1 
39ac048ae4203a836c381f19fc9245027e42561636ebb8b67eb0e23478118ede
[20:10:59 root@docker2 ~]#docker exec -it tomcat-web1 bash
[root@39ac048ae420 /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.20.0.2  netmask 255.255.255.0  broadcast 10.20.0.255
        ether 02:42:0a:14:00:02  txqueuelen 0  (Ethernet)
        RX packets 12  bytes 1032 (1.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 6.3.3 添加静态路由
在各宿主机添加静态路由，网关指向对方的 IP
```
#docker1
[20:10:20 root@docker1 ~]#route add -net 10.20.0.0/24 gw 192.168.10.182
#这个可以不配模式就是允许
[20:12:27 root@docker1 ~]#iptables -A FORWARD -s 192.168.0.0/21 -j ACCEPT
#测试ping是否通信
[20:13:02 root@docker1 ~]#ping 10.20.0.2
PING 10.20.0.2 (10.20.0.2) 56(84) bytes of data.
64 bytes from 10.20.0.2: icmp_seq=1 ttl=63 time=0.525 ms
64 bytes from 10.20.0.2: icmp_seq=2 ttl=63 time=0.339 ms

#docker2
[20:11:57 root@docker2 ~]#route add -net 10.10.0.0/24 gw 192.168.10.181
#默认允许
[20:14:28 root@docker2 ~]#iptables -A FORWARD -s 192.168.0.0/24 -j ACCEPT
#测试
[09:17:08 root@docker2 ~]#ping 10.10.0.2
PING 10.10.0.2 (10.10.0.2) 56(84) bytes of data.
64 bytes from 10.10.0.2: icmp_seq=1 ttl=63 time=0.790 ms
64 bytes from 10.10.0.2: icmp_seq=2 ttl=63 time=0.463 ms
```

### 6.3.4 测试容器间互联
```
#docker1
[09:15:44 root@docker1 ~]#docker exec -it tomcat-web1 bash
[root@8581838de178 /]# ping 10.20.0.2
PING 10.20.0.2 (10.20.0.2) 56(84) bytes of data.
64 bytes from 10.20.0.2: icmp_seq=1 ttl=62 time=0.538 ms
[root@8581838de178 /]# curl 10.20.0.2:8080/myapp/
Tomcat Web Page1
#docker2
[09:17:49 root@docker2 ~]#docker exec -it tomcat-web1 bash
[root@573fcea29e0c /]# ping 10.10.0.2
PING 10.10.0.2 (10.10.0.2) 56(84) bytes of data.
64 bytes from 10.10.0.2: icmp_seq=1 ttl=62 time=0.487 ms
[root@573fcea29e0c /]# curl 10.10.0.2:8080/myapp/
Tomcat Web Page1
```

## 6.4 创建自定义网络
可以基于 docker 命令创建自定义网络，自定义网络可以自定义 IP 地范围和网关等信息。

### 6.4.1 创建自定义 docker 网络
```
#创建自定义网络
[09:26:29 root@docker1 ~]#docker network create -d bridge --subnet 10.100.0.0/24 --gateway 10.100.0.1 zhang-net 
79b124db33e9e161055c427912ba09fe2072aaf9ccc3f09a4e551f2bf05ac05e
#验证网络：
[09:28:05 root@docker1 ~]#docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
374e67213b1e        bridge              bridge              local
e541256bb224        host                host                local
245cd90d385d        none                null                local
79b124db33e9        zhang-net           bridge              local
```

### 6.4.2 创建不同网络的容器测试通信
```
#使用自定义网络创建容器
[09:28:59 root@docker1 ~]#docker run -it -d --name tomcat-zhang-net --net=zhang-net harbor.zhangzhuo.org/web1/tomcat-web:app1 
a04fedd83a6ef1bbdddee24401fc852d4e9f5ee8c88ebec7e27e4a6fcc490ba9
#查看IP
[09:30:35 root@docker1 ~]#docker exec -it tomcat-zhang-net bash
[root@a04fedd83a6e /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.100.0.2  netmask 255.255.255.0  broadcast 10.100.0.255
        ether 02:42:0a:64:00:02  txqueuelen 0  (Ethernet)
        RX packets 14  bytes 1172 (1.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        
#创建默认网络容器
[09:31:55 root@docker1 ~]#docker run -it -d --name tomcar-net harbor.zhangzhuo.org/web1/tomcat-web:app1 
c983363231db6dfe08efe183fa42307381455ebb2d331c4fc494056992405155
#查看网络
[09:32:28 root@docker1 ~]#docker exec -it tomcar-net bash
[root@c983363231db /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
9: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:0a:0a:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.10.0.3/24 brd 10.10.0.255 scope global eth0
       valid_lft forever preferred_lft forever

#如果俩个网络需要通信需要修改iptables规则
#先导出现在的规则
[09:35:08 root@docker1 ~]#iptables-save >iptables.sh
#需要注释这俩行
 23 #-A DOCKER-ISOLATION-STAGE-2 -o br-79b124db33e9 -j DROP
 24 #-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP 
 
#在从新导入
[09:37:39 root@docker1 ~]#iptables-restore <iptables.sh

#测试1
[09:38:22 root@docker1 ~]#docker exec -it tomcar-net bash
[root@c983363231db /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
9: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:0a:0a:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.10.0.3/24 brd 10.10.0.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@c983363231db /]# ping 10.100.0.2
PING 10.100.0.2 (10.100.0.2) 56(84) bytes of data.
64 bytes from 10.100.0.2: icmp_seq=1 ttl=63 time=0.309 ms
64 bytes from 10.100.0.2: icmp_seq=2 ttl=63 time=0.067 ms

#测试2
[09:38:41 root@docker1 /]#docker exec -it tomcat-zhang-net bash
[root@a04fedd83a6e /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:0a:64:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.100.0.2/24 brd 10.100.0.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@a04fedd83a6e /]# ping 10.10.0.3
PING 10.10.0.3 (10.10.0.3) 56(84) bytes of data.
64 bytes from 10.10.0.3: icmp_seq=1 ttl=63 time=0.091 ms
64 bytes from 10.10.0.3: icmp_seq=2 ttl=63 time=0.073 ms
```

# 七、Dokcer资源限制
资源限制介绍：https://docs.docker.com/config/containers/resource_constraints/

默认情况下，容器没有资源限制，可以使用主机内核调度程序允许的尽可能多的给定资源，Docker 提供了控制容器可以限制容器使用多少内存或 CPU 的方法， 设置 docker run 命令的运行时配置标志。

其中许多功能都要求宿主机的内核支持Linux 功能，要检查支持，可以使用docker info 命令，如果内核中禁用了某项功能，可能会在输出结尾处看到警告
```
[19:59:26 root@docker2 ~]#docker info
WARNING: No swap limit support
```
对于 Linux 主机，如果没有足够的内容来执行其他重要的系统任务，将会抛出 OOM (Out of Memory Exception,内存溢出、内存泄漏、内存异常), 随后系统会开始杀死进程以释放内存，凡是运行在宿主机的进程都有可能被 kill，包括 Dockerd 和其它的应用程序，如果重要的系统进程被 Kill,会导致和该进程相关 的服务全部宕机。

产生 OOM 异常时，Dockerd 尝试通过调整 Docker 守护程序上的 OOM 优先 级来减轻这些风险，以便它比系统上的其他进程更不可能被杀死，但是容器的 OOM 优先级未调整，这使得单个容器被杀死的可能性比 Docker 守护程序或其 他系统进程被杀死的可能性更大，不推荐通过在守护程序或容器上手动设置 --oom-score-adj 为极端负数，或通过在容器上设置--oom-kill-disable 来绕过这些安全措施。

## 7.1 OOM 优先级机制
linux 会为每个进程算一个分数，最终他会将分数最高的进程 kill。
```
/proc/PID/oom_score_adj #范围为-1000 到 1000，值越高越容易被宿主机 kill 掉，如果将该值设置为-1000，则进程永远不会被宿主机 kernel kill。
/proc/PID/oom_adj  #范围为-17 到+15，取值越高越容易被干掉，如果是-17，则表示不能被 kill，该设置参数的存在是为了和旧版本的 Linux 内核兼容。
/proc/PID/oom_score #这个值是系统综合进程的内存消耗量、CPU 时间(utime + stime)、存活时间(uptime - start time)和 oom_adj 计算出的进程得分，消耗内存越多得分越高，越容易被宿主机 kernel 强制杀死
```

## 7.2 容器的内存限制
Docker 可以强制执行硬性内存限制，即只允许容器使用给定的内存大小。

Docker 也可以执行非硬性内存限制，即容器可以使用尽可能多的内存，除非内核检测到主机上的内存不够用了。

### 7.2.1 内存限制参数
```
-m or --memory  #容器可以使用的最大内存量，如果设置此选项，则允许的最小存值为 4m （4 兆字节）
--memory-swap   #容器可以使用的交换分区大小，必须要在设置了物理内存限制的前提才能设置交换分区的限制
--memory-swappiness #设置容器使用交换分区的倾向性，值越高表示越倾向于使用 swap 分区，范围为 0-100，0 为能不用就不用，100 为能用就用。
--kernel-memory   #容器可以使用的最大内核内存量，最小为 4m，由于内核内存与用户空间内存在隔离，因此无法与用户空间内存直接交换，因此内核内存不足的容器可能会阻塞宿主主机资源，这会对主机和其他容器或者其他服务进程产生影响，因此不要设置内核内存大小。
--memory-reservation #允许指定小于--memory 的软限制，当 Docker检测到主机上的争用或内存不足时会激活该限制，如果使用--memory-reservation，则必须将其设置为低于--memory 才能使其优先。 因为它是软限制，所以不能保证容器不超过限制。
--oom-kill-disable #默认情况下，发生 OOM 时，kernel 会杀死容器内进程，但是可以使用--oom-kill-disable 参数，可以禁止 oom 发生在指定的容器上，即仅在已设置-m / - memory 选项的容器上禁用 OOM，如果-m 参数未配置，产生 OOM 时，主机为了释放内存还会杀死系统进程。
```

### 7.2.2 swap 限制
```
--memory-swap  #只有在设置了 --memory 后才会有意义。使用 Swap,可以让容器将超出限制部分的内存置换到磁盘上，WARNING：经常将内存交换到磁盘的应用程序会降低性能。
```

不同的--memory-swap 设置会产生不同的效果：
```
--memory-swap #值为正数， 那么--memory 和--memory-swap 都必须要设置，--memory-swap 表示你能使用的内存和 swap 分区大小的总和，例如：--memory=300m, --memory-swap=1g, 那么该容器能够使用 300m 内存和700m swap，即--memory 是实际物理内存大小值不变，而 swap 的实际大小计算方式为(--memory-swap)-(--memory)=容器可用 swap。
--memory-swap #如果设置为 0，则忽略该设置，并将该值视为未设置，即未设置交换分区。
--memory-swap #如果等于--memory 的值，并且--memory 设置为正整数，容器无权访问 swap 即也没有设置交换分区。
--memory-swap #如果设置为 unset，如果宿主机开启了 swap，则实际容器的 swap 值为 2x( --memory)，即两倍于物理内存大小，但是并不准确(在容器中使用 free 命令所看到的 swap 空间并不精确，毕竟每个容器都可以看到具体大小，但是宿主机的 swap 是有上限而且不是所有容器看到的累计大小)。
--memory-swap #如果设置为-1，如果宿主机开启了 swap，则容器可以使用主机上 swap 的最大空间。
```

## 7.3 内存限制示例
假如一个容器未做内存使用限制，则该容器可以利用到系统内存最大空间，默认创建的容器没有做内存资源限制。
```
#下载测试镜像
[20:14:36 root@docker1 ~]#docker pull harbor.zhangzhuo.org/web1/tomcat-web:app1
[20:15:50 root@docker1 ~]#docker run -it --rm --help  #查看帮助
```

### 7.3.1 内存大小硬限制
--memory 大小
```
#启动两个工作进程，每个工作进程最大允许使用内存256M，且宿主机不限制当前容器最大内存
#不限制
[20:18:14 root@docker1 ~]#docker run  -it -d --name tomcat-c1 harbor.zhangzhuo.org/web1/tomcat-web:app1 
67515f5f62d40617adccd0993b3e9cfc91ce1844416508cc53e2202dbacfca5c
[20:18:45 root@docker1 ~]#docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
67515f5f62d4        tomcat-c1           0.02%               88.54MiB / 2.908GiB   2.97%               836B / 0B           0B / 664kB          33
#硬限制256M
[20:20:33 root@docker1 ~]#docker run  -it -d --name tomcat-c1 --memory 256M harbor.zhangzhuo.org/web1/tomcat-web:app1 
68b86fdfcca3cd6f8ba6e2b60834e23afff029da7b5f2c6c90518a038afb804c
[20:21:08 root@docker1 ~]#docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
68b86fdfcca3        tomcat-c1           0.02%               70.49MiB / 256MiB   27.54%              836B / 0B           0B / 664kB          31
#宿主机 cgroup 验证
[20:21:13 root@docker1 ~]#cat /sys/fs/cgroup/memory/docker/68b86fdfcca3cd6f8ba6e2b60834e23afff029da7b5f2c6c90518a038afb804c/memory.limit_in_bytes 
268435456
[20:22:05 root@docker1 ~]#echo "268435456/1024/1024" | bc
256
```

注：通过 echo 命令可以改内存限制的值，但是可以在原基础之上增大内存限制， 缩小内存限制会报错 write error: Device or resource busy
```
#增加
[20:23:51 root@docker1 ~]#echo 536870912 >/sys/fs/cgroup/memory/docker/68b86fdfcca3cd6f8ba6e2b60834e23afff029da7b5f2c6c90518a038afb804c/memory.limit_in_bytes 
[20:24:32 root@docker1 ~]#docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
68b86fdfcca3        tomcat-c1           0.02%               68.05MiB / 512MiB   13.29%              1.05kB / 0B         0B / 664kB          31
#减少
[20:25:16 root@docker1 ~]#echo 2684354 >/sys/fs/cgroup/memory/docker/68b86fdfcca3cd6f8ba6e2b60834e23afff029da7b5f2c6c90518a038afb804c/memory.limit_in_bytes 
-bash: echo: write error: Device or resource busy #减少到启动容器规定的界限会报错
```

### 7.3.2 内存大小软限制
--memory-reservation 大小
```
[20:27:02 root@docker1 ~]#docker run  -it -d --name tomcat-c1 --memory 256M --memory-reservation 128m harbor.zhangzhuo.org/web1/tomcat-web:app1 
cdb3b276622d18f0415f5e9871e821b97647ebaa2a4acd066d902e79bf7bcdee
#宿主机验证
[20:27:27 root@docker1 ~]#cat /sys/fs/cgroup/memory/docker/cdb3b276622d18f0415f5e9871e821b97647ebaa2a4acd066d902e79bf7bcdee/memory.soft_limit_in_bytes 
134217728
```

### 7.3.3 关闭 OOM 机制
--oom-kill-disable
```
[20:28:47 root@docker1 ~]#docker run  -it -d --name tomcat-c1 --memory 256M --oom-kill-disable harbor.zhangzhuo.org/web1/tomcat-web:app1 
9a7e4d2a5b3a123a2fad5a12228492c242f850a2baebaf018e4e33944df38f69
#宿主机验证
[20:29:02 root@docker1 ~]#cat /sys/fs/cgroup/memory/docker/9a7e4d2a5b3a123a2fad5a12228492c242f850a2baebaf018e4e33944df38f69/memory.oom_control 
oom_kill_disable 1
under_oom 0
oom_kill 0
```

### 7.3.4 交换分区限制
--memory-swap 大小
```
[20:31:11 root@docker1 ~]#docker run  -it -d --name tomcat-c1 --memory 256M --memory-swap 512m harbor.zhangzhuo.org/web1/tomcat-web:app1 
3e9b8da0afe6daf645931830ebbd22e46b5ead24b2d640e8679ac8bdc1d07d9e
#宿主机验证
[20:31:30 root@docker1 ~]#cat /sys/fs/cgroup/memory/docker/3e9b8da0afe6daf645931830ebbd22e46b5ead24b2d640e8679ac8bdc1d07d9e/memory.memsw.limit_in_bytes 
536870912
```

### 7.3.5 K8s 1.8.3 更新日志
宿主机开启交换分区，会在安装之前的预检查环节提示相应错误信息：

https://github.com/kubernetes/kubernetes/blob/release-1.8/CHANGELOG-1.8.md



## 7.4 容器的 CPU 限制
https://docs.docker.com/config/containers/resource_constraints/

一个宿主机，有几十个核心的 CPU，但是宿主机上可以同时运行成百上千个不 同的进程用以处理不同的任务，多进程共用一个 CPU 的核心依赖技术就是为可压缩资源，即一个核心的 CPU 可以通过调度而运行多个进程，但是同一个单位 时间内只能有一个进程在 CPU 上运行，那么这么多的进程怎么在 CPU上执行和调度的呢？

**实时优先级：** 0-99

**非实时优先级(nice)：** -20-19，对应 100-139 的进程优先级

Linux kernel 进程的调度基于 CFS(Completely Fair Scheduler)，完全公平调度

CPU 密集型的场景：优先级越低越好，计算密集型任务的特点是要进行大量的计算，消耗 CPU 资源，比如计算圆周率、数据处理、对视频进行高清解码等等， 全靠 CPU 的运算能力。

IO 密集型的场景：优先级值高点，涉及到网络、磁盘 IO 的任务都是 IO 密集型任务，这类任务的特点是 CPU 消耗很少，任务的大部分时间都在等待 IO 操作完成（因为 IO 的速度远远低于 CPU 和内存的速度），比如 Web 应用，高并发， 数据量大的动态网站来说，数据库应该为 IO 密集型。

**磁盘的调度算法**
```
[20:32:19 root@docker1 ~]#cat /sys/block/sda/queue/scheduler 
noop deadline [cfq]
```
默认情况下，每个容器对主机 CPU 周期的访问权限是不受限制的，但是我们可 以设置各种约束来限制给定容器访问主机的 CPU 周期，大多数用户使用的是默 认的 CFS 调度方式，在 Docker 1.13 及更高版本中，还可以配置实时优先级。

参数：
```
--cpus #指定容器可以使用多少可用 CPU 资源，例如，如果主机有两个 CPU，并且设置了--cpus =“1.5”，那么该容器将保证最多可以访问 1.5 个的 CPU(如果是4核CPU，那么还可以是4核心上每核用一点，但是总计是1.5核心的CPU)，这相当于设置--cpu-period =“100000”(CPU 调度周期)和--cpu-quota = “150000”(CPU 调度限制)，--cpus 主要在 Docker 1.13 和更高版本中使用，目的是替代--cpu-period 和--cpu-quota 两个参数，从而使配置更简单，但是最大不能超出宿主机的 CPU 总核心数(在操作系统看到的 CPU 超线程后的数值)

[20:48:04 root@docker1 ~]#docker run -it -d --cpus 3 harbor.zhangzhuo.org/web1/tomcat-web:app1 
docker: Error response from daemon: Range of CPUs is from 0.01 to 2.00, as there are only 2 CPUs available.
See 'docker run --help'.   #分配给容器的 CPU 超出了宿主机 CPU


--cpu-period #(CPU 调度周期)设置 CPU 的 CFS 调度程序周期，必须与--cpu-quota 一起使用，默认周期为 100 微秒(1Second=1000Millisecond=1000000Microsecond)。
--cpu-quota #在容器上添加 CPU CFS 配额，计算方式为 cpu-quota /cpu-period 的结果值，早期的 docker(1.12 及之前)使用此方式设置对容器的CPU 限制值，新版本 docker(1.13 及以上版本)通常使用--cpus 设置此值。
--cpuset-cpus #用于指定容器运行的 CPU 编号，也就是我们所谓的绑核
--cpuset-mem #设置使用哪个 cpu 的内存，仅对非统一内存访问(NUMA)架构有效。
--cpu-shares #用于设置cfs中调度的相对最大比例权重,cpu-share的值越高的容器，将会分得更多的时间片(宿主机多核 CPU 总数为 100%，假如容器 A 为1024，容器 B 为 2048，那么容器 B 将最大是容器 A 的可用 CPU 的两倍 )，默认的时间片 1024，最大 262144。
```

## 7.5 测试CPU限制

### 7.5.1 未限制容器 CPU
对于一台四核的服务器，如果不做限制，容器会把宿主机的 CPU 全部占
```
[20:48:24 root@docker1 ~]#docker run -it -d harbor.zhangzhuo.org/web1/tomcat-web:app1 
cb94badc7839981e5657110c8a81670ab0f45d9d0063a9d8dc7bc5addcb091d5
#在宿主机查看CPU限制参数
[20:53:23 root@docker1 ~]#cat /sys/fs/cgroup/cpuset/docker/cb94badc7839981e5657110c8a81670ab0f45d9d0063a9d8dc7bc5addcb091d5/cpuset.cpus
0-1
```

### 7.5.2 限制容器 CPU
--cpus 1
```
#只给容器分配最多一核宿主机 CPU 利用率
[20:55:34 root@docker1 ~]#docker run -it -d --cpus 1 harbor.zhangzhuo.org/web1/tomcat-web:app1 
875e44ec9bcf4bb4006a412472107e72a65c3798e702d6ce4b127438ccc41f95
#宿主机验证
[20:56:20 root@docker1 ~]#cat /sys/fs/cgroup/cpu,cpuacct/docker/875e44ec9bcf4bb4006a412472107e72a65c3798e702d6ce4b127438ccc41f95/cpu.cfs_quota_us 
100000  #每核心 CPU 会按照 1000 为单位转换成百分比进行资源划分，1 个核心的 CPU 就是 100000/1000=100%，4 个核心 400000/1000=400%，以此类推。
```
注：CPU 资源限制是将分配给容器的 1 核心分配到了宿主机每一核心 CPU 上， 也就是容器的总 CPU 值是在宿主机的每一个核心 CPU分配了部分比例。

### 7.5.3 将容器运行到指定的 CPU
--cpuset-cpus 1
```
[20:59:38 root@docker1 ~]#docker run -it -d --cpus 1 --cpuset-cpus 1 harbor.zhangzhuo.org/web1/tomcat-web:app1 
e88be0def5054d681e7baa550940d7ba46a5f3ce8d1e9e1370edc979418bb6f8
#宿主机验证
[21:00:12 root@docker1 ~]#cat /sys/fs/cgroup/cpuset/docker/e88be0def5054d681e7baa550940d7ba46a5f3ce8d1e9e1370edc979418bb6f8/cpuset.cpus 
1
```

### 7.5.4 基于 cpu—shares 对 CPU进行切分
启动两个容器，zhang-c1 的--cpu-shares 值为 1000，zhang-c2 的 --cpu-shares 为 500，观察最终效果，--cpu-shares 值为 1000 的 zhang-c1 的 CPU 利用率基本是--cpu-shares 为 500 的 zhang-c2 的两倍：
```
[21:02:42 root@docker1 ~]#docker run -it -d --cpu-shares 1000 --name zhang-c1 harbor.zhangzhuo.org/web1/tomcat-web:app1 
89fdb35a9cd1421634c1326c8b28e3a6a78bc36788398cbcef087ee7f37f51d7
[21:03:20 root@docker1 ~]#docker run -it -d --cpu-shares 500 --name zhang-c2 harbor.zhangzhuo.org/web1/tomcat-web:app1 
12fa5ba204b1d95c5f975dcb62cf25dd3eca2aa00da62e54114a0af118b82cfe

#宿主机验证
[21:03:33 root@docker1 ~]#cat /sys/fs/cgroup/cpu,cpuacct/docker/12fa5ba204b1d95c5f975dcb62cf25dd3eca2aa00da62e54114a0af118b82cfe/cpu.shares 
500
[21:04:37 root@docker1 ~]#cat /sys/fs/cgroup/cpu,cpuacct/docker/89fdb35a9cd1421634c1326c8b28e3a6a78bc36788398cbcef087ee7f37f51d7/cpu.shares 
1000
```

**动态修改 CPU shares值**

--cpu-shares 的值可以在宿主机 cgroup 动态修改，修改完成后立即生效，其 值可以调大也可以减小。
```
[21:05:29 root@docker1 ~]#echo 2000 >/sys/fs/cgroup/cpu,cpuacct/docker/12fa5ba204b1d95c5f975dcb62cf25dd3eca2aa00da62e54114a0af118b82cfe/cpu.shares
```

# 八、单机编排之 Docker Compose
当在宿主机启动较多的容器时候，如果都是手动操作会觉得比较麻烦而且容器出错 ， 这个时候推 荐使用 docker 单机编排工具 docker-compose ， docker-compose 是 docker 容器的一种单机编排服务，docker-compose 是一 个管理多个容器的工具，比如可以解决容器之间的依赖关系，就像启动一个 nginx 前端服务的时候会调用后端的 tomcat，那就得先启动 tomcat，但是启动 tomcat 容器还需要依赖数据库，那就还得先启动数据库，docker-compose 就可以解决 这样的嵌套依赖关系，其完全可以替代 docker run 对容器进行创建、启动和停止。

docker-compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器 集群的快速编排，docker-compose 将所管理的容器分为三层，分别是工程 （project），服务（service）以及容器（container）。

github 地址： https://github.com/docker/compose

## 8.1 基础环境准备

### 8.1.1 安装 python-pip 软件
python-pip 包将安装一个 pip 的命令，pip 命令是一个 pyhton 安装包的安装 工具，其类似于 ubuntu 的 apt 或者 redhat 的 yum，但是 pip 只安装 python 相 关的安装包，可以在多种操作系统安装和使用 pip。
```
#ubuntu安装
[09:41:48 root@ubuntu ~]#apt update
[09:49:29 root@ubuntu ~]#apt install python-pip
#centos安装
[09:50:14 root@centos ~]#yum install epel-release
[09:50:14 root@centos ~]#yum install python-pip
[09:50:14 root@centos ~]#pip install --upgrade pip
```

官方二进制下载地址：https://github.com/docker/compose/releases

### 8.1.2 安装 docker compose
```
#安装
[09:49:29 root@ubuntu ~]#pip install docker-compose
#验证版本
[09:50:14 root@docker2 ~]#docker-compose version
docker-compose version 1.29.1, build c34c88b2
docker-py version: 5.0.0
CPython version: 3.7.10
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```

### 8.1.3 查看 docker-compose帮助
官方文档：https://docs.docker.com/compose/reference/
```
[09:58:29 root@docker1 ~]#docker-compose --help
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [--profile <name>...] [options] [--] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             #指定 Compose 模板文件，默认为 docker-compose.yml
  -p, --project-name NAME     #指定项目名称，默认将使用当前所在目录名称作为项目名。
  --verbose                   #显示更多输出信息
  --log-level LEVEL           #定义日志级别 (DEBUG,INFO,WARNING, ERROR CRITICAL)
  --ansi (never|always|auto)  Control when to print ANSI control characters
  --no-ansi                   #不显示 ANSI 控制字符
  -v, --version               #显示版本
Commands:
  build              #通过 docker-compose 构建镜像
  config             #查看当前配置，没有错误不输出任何信息
  create             #创建服务
  down               #停止和删除所有容器、网络、镜像和卷
  events             #从容器接收实时事件，可以指定json日志格式，如docker-compose events --json
  exec               #进入指定容器进行操作
  help               #显示帮助信息
  images             #显示当前服务器的docker镜像信息
  kill               #强制终止运行中的容器
  logs               #查看容器的日志
  pause              #暂停服务
  port               #查看端口
  ps                 #列出容器
  pull               #从新拉取镜像
  push               #上传镜像
  restart            #重启服务
  rm                 #删除已经停止的服务
  run                #一次性运行容器，等于docker run --rm
  scale              #设置指定服务运行的容器个数，如docker-compose scale nginx=2
  start              #启动服务
  stop               #停止服务
  top                #显示容器运行状态
  unpause            #取消暂停
  up                 #创建容器并运行
  version            #显示docker-compose版本信息
```

## 8.2 从 docker compose 启动单个容器
目录可以在任意目录，推荐放在有意义的位置。
```
#创建目录
[10:19:08 root@docker1 ~]#mkdir /docker/zhangzhuo -p
[10:19:10 root@docker1 ~]#cd /docker/zhangzhuo/
[10:19:16 root@docker1 zhangzhuo]#pwd
/docker/zhangzhuo

#编写一个 yml 格式的配置 docker-compose 文件，启动一个 tomcat 服务，由于格式为 yml 格式，因此要注意前后的缩进及上下行的等级关系。
[10:21:47 root@docker1 zhangzhuo]#vim docker-compose.yml
services-tomcat-web:
    image: harbor.zhangzhuo.org/web1/tomcat-web:app1
    expose:
      - 8080
    ports:
      - "8080:8080"

#启动容器
#必须要在 docker compose 文件所在的目录执行
#docker-compose up -d #不加是 d 前台启
[10:24:58 root@docker1 zhangzhuo]#docker-compose up -d
Creating zhangzhuo_services-tomcat-web_1 ... done


#测试
[10:25:07 root@docker1 zhangzhuo]#curl 192.168.10.181:8080/myapp/
Tomcat Web Page1
#容器的在启动的时候，会给容器自定义一个名称，在 service name 后面加_1
[10:24:58 root@docker1 zhangzhuo]#docker-compose up -d
Creating zhangzhuo_services-tomcat-web_1 ... done
[10:28:07 root@docker1 zhangzhuo]#docker-compose ps
             Name                            Command               State           Ports         
-------------------------------------------------------------------------------------------------
zhangzhuo_services-tomcat-web_1   /apps/tomcat/bin/run_tomcat.sh   Up      0.0.0.0:8080->8080/tcp
```

### 8.2.1 自定义容器名称
```
[10:30:07 root@docker1 zhangzhuo]#vim docker-compose.yml 
services-tomcat-web:
    image: harbor.zhangzhuo.org/web1/tomcat-web:app1
    container_name: tomcat-web1    #定义容器名称
    expose:
      - 8080
    ports:
      - "8080:8080"
[10:29:49 root@docker1 zhangzhuo]#docker-compose up -d
Recreating zhangzhuo_services-tomcat-web_1 ... done
[10:30:00 root@docker1 zhangzhuo]#docker-compose ps
   Name                  Command               State           Ports         
-----------------------------------------------------------------------------
tomcat-web1   /apps/tomcat/bin/run_tomcat.sh   Up      0.0.0.0:8080->8080/tcp
```

## 8.3 从 docker compose 启动多个容器
```
#编辑docker-compose文件
[10:35:52 root@docker1 zhangzhuo]#cat docker-compose.yml 
services-tomcat-web1:
    image: harbor.zhangzhuo.org/web1/tomcat-web:app1
    container_name: tomcat-web1
    expose:
      - 8080
    ports:
      - "8080:8080"
services-nginx-web1:
    image: harbor.zhangzhuo.org/nginx/centos-nginx:v1
    container_name: nginx-web1
    expose:
      - 80
    ports:
      - "80:80"
#重新启动容器
[10:36:24 root@docker1 zhangzhuo]#docker-compose up -d

#测试
[10:37:20 root@docker1 zhangzhuo]#ss -ntl | grep 80
LISTEN   0         20480                     *:80                     *:*     
LISTEN   0         20480                     *:8080                   *:* 
[10:37:28 root@docker1 zhangzhuo]#curl 192.168.10.181
<h1>zhangzhuo</h1>
[10:37:51 root@docker1 zhangzhuo]#curl 192.168.10.181:8080/myapp/
Tomcat Web Page1
```

## 8.4 定义数据卷挂载
```
#创建数据目录和文件
[10:39:09 root@docker1 zhangzhuo]#echo "docker.zhangzhuo.org" >/data/testapp/index.html 

#编辑 compose 配置文件
[10:43:47 root@docker1 zhangzhuo]#cat docker-compose.yml
services-tomcat-web1:
    image: harbor.zhangzhuo.org/web1/tomcat-web:app1
    container_name: tomcat-web1
    volumes:   #挂载文件夹
      - /data/testapp:/apps/tomcat/webapps/myapp
    expose:
      - 8080
    ports:
      - "8080:8080"
services-nginx-web1:
    image: harbor.zhangzhuo.org/nginx/centos-nginx:v1
    container_name: nginx-web1
    expose:
      - 80
    ports:
      - "80:80"
#重启容器
[10:43:50 root@docker1 zhangzhuo]#docker-compose stop
Stopping nginx-web1  ... done
Stopping tomcat-web1 ... done
[10:44:16 root@docker1 zhangzhuo]#docker-compose up -d
Starting nginx-web1    ... done
Recreating tomcat-web1 ... done

#测试
[10:44:23 root@docker1 zhangzhuo]#curl 192.168.10.181:8080/myapp/
docker.zhangzhuo.org
```

## 8.5 容器启动时依赖
```
[11:12:43 root@docker1 zhangzhuo]#cat docker-compose.yml 
service-tomcat-web1:
    image: harbor.zhangzhuo.org/web1/tomcat-web:app1
    container_name: tomcat-web1
    volumes:
      - /data/testapp:/apps/tomcat/webapps/myapp
    expose:
      - 8080
    ports:
      - "8080:8080"
    links:
      - service-nginx-web1     #这里如果写，表示service-tomcat-web启动时是依赖nginx的，并且会在本机hosts中添加解析记录
#添加links时如果tomcat添加nginx后，nginx就不可以添加tomcat了
service-nginx-web1:
    image: harbor.zhangzhuo.org/nginx/centos-nginx:v1
    container_name: nginx-web1
    volumes:
      - /data/testapp:/usr/local/nginx/html
    expose:
      - 80
    ports:
      - "80:80"
#启动
#启动时会先启动依赖的容器
[11:15:11 root@docker1 zhangzhuo]#docker-compose up -d
Creating nginx-web1 ... done
Creating tomcat-web1 ... done

#查看tomcat容器hosts
[11:15:39 root@docker1 zhangzhuo]#docker exec -it tomcat-web1 bash
[root@6ee9ffdbdf88 /]# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
10.10.0.2	nginx-web1 608b7e459d17
10.10.0.2	service-nginx-web1 608b7e459d17 nginx-web1
10.10.0.3	6ee9ffdbdf88
```

## 8.6 其他常用命令

### 8.6.1 重启单个指定容器
```
[10:48:32 root@docker1 zhangzhuo]#cat docker-compose.yml
services-tomcat-web1:
    image: harbor.zhangzhuo.org/web1/tomcat-web:app1
    container_name: tomcat-web1
    volumes:
      - /data/testapp:/apps/tomcat/webapps/myapp
    expose:
      - 8080
    ports:
      - "8080:8080"
services-nginx-web1:
    image: harbor.zhangzhuo.org/nginx/centos-nginx:v1
    container_name: nginx-web1
    volumes:
      - /data/testapp:/usr/local/nginx/html
    expose:
      - 80
    ports:
      - "80:80"
#重启services-nginx-web1
[10:49:20 root@docker1 zhangzhuo]#docker-compose restart services-nginx-web1
Restarting nginx-web1 ... done
```

### 8.6.2 重启所有容器
```
[10:51:06 root@docker1 zhangzhuo]#docker-compose restart
Restarting nginx-web1  ... done
Restarting tomcat-web1 ... done
```

### 8.6.3 停止和启动单个容器
```
[10:51:56 root@docker1 zhangzhuo]#docker-compose stop services-tomcat-web1
Stopping tomcat-web1 ... done
[10:52:51 root@docker1 zhangzhuo]#docker-compose start services-tomcat-web1
Starting services-tomcat-web1 ... done
```

## 8.7 实现单机版的 Nginx+Tomcat
编写 docker-compose.yml 文件，实现单机版本的 nginx+tomcat 的动静分离 web 站点，要求从 nginx 作为访问入口，当访问指定 URL 的时候转发至 tomcat 服务器响应。

### 8.7.1 架构图

```
说明:
当用户访问www.zhangzhuo.org/html目录下的文件时有nginx进行处理
当用户访问www.zhangzhuo.org/app目录下的文件时由tomcat进行处理
haproxy服务器进行代理http请求转发到nginx服务器
nginx服务器接收到请求后进行分析，如访问的是html文件夹由自己直接回复报文，如访问的是app文件夹就转发到tomcat服务器
```

镜像使用之前制作的镜像
```
tomcat使用 harbor.zhangzhuo.org/web/tomcat:app1
nginx使用 harbor.zhangzhuo.org/nginx/centos-nginx:v1
haproxy使用 harbor.zhangzhuo.org/web1/haproxy-web1:v1
```

### 8.7.2 准备服务配置文件与web文件

#### 8.7.2.1 准备haproxy配置文件
```
#准备haproxy配置文件
[11:51:59 root@docker1 ~]#mkdir /data/haproxy/ -p
[11:52:23 root@docker1 data]#cd /data/haproxy/
[11:54:49 root@docker1 haproxy]#cat haproxy.cfg 
global             
	maxconn 100000       
	chroot /usr/local/haproxy    
	stats socket /usr/local/haproxy/run/haproxy.sock
	uid 99            
	gid 99
	daemon
	pidfile /usr/local/haproxy/run/haproxy.pid
	log 127.0.0.1 local3 info


defaults
	option http-keep-alive 
	option forwardfor     
	maxconn 100000
mode http
	timeout connect 300000ms
	timeout client 300000ms
	timeout server 300000ms

#haproxy状态页
listen stats
	mode http
	bind 0.0.0.0:9999
	stats enable      
	log global
    stats hide-version
	stats uri /haproxy-status
	stats auth haadmin:123456
    stats refresh 5s
    stats admin if TRUE


listen web_host
    bind 0.0.0.0:80
    mode http
    log global
    balance static-rr
    option forwardfor
    server web1 nginx-server1:80 weight 1 check inter 3000 fall 3 rise 5
    server web2 nginx-server2:80 weight 1 check inter 3000 fall 3 rise 5
```

#### 8.7.2.2 准备nginx配置文件及web文件
```
[11:56:56 root@docker1 ~]#mkdir /data/nginx/{web,conf} -p
#配置文件
[13:40:18 root@docker1 nginx]#cat conf/nginx.conf 

user  nginx; #运行进程的身份

worker_processes  auto; #开启工作进程数量
worker_cpu_affinity auto; #工作进程绑定cpu

error_log  logs/error.log;  #错误日志

pid        logs/nginx.pid;  #pid文件生成位置

worker_priority 0;          #工作进程的优先级-20~19

worker_rlimit_nofile 65536; #nginx所有连接数
daemon off;

events {
    worker_connections  65536;       #每个工作进程的最大连接数
    use epoll;                                    #使用epoll事件驱动
    accept_mutex on;                       #优化只有有个请求时避免多个进程被唤醒
    multi_accept on;                        #每个工作进程可以接受多个新的网络连接
}

#优化性能
#thread_pool pool1 threads=16;     #配置线程池
#thread_pool pool2 threads=32;


http {
    include       mime.types;                             #导入支持的文件类型
    default_type  application/octet-stream;     #设置默认的类型，会提示下载不匹配的类型文件



#默认访问日志相关
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"'
    #                   '$server_name:$server_port';
    #access_log  logs/access.log  main;

#访问日志json格式
    log_format access_json '{"@timestamp":"$time_iso8601",'
        '"host":"$server_addr",'
        '"clientip":"$remote_addr",'
        '"size":$body_bytes_sent,'
        '"responsetime":$request_time,'
        '"upstreamtime":"$upstream_response_time",'
        '"upstreamhost":"$upstream_addr",'
        '"http_host":"$host",'
        '"uri":"$uri",'
        '"domain":"$host",'
        '"xff":"$http_x_forwarded_for",'
        '"referer":"$http_referer",'
        '"tcp_xff":"$proxy_protocol_addr",'
        '"http_user_agent":"$http_user_agent",'
        '"status":"$status"}';
    access_log logs/access.log access_json;

    sendfile        on; #实现文件零拷贝
    tcp_nopush     on; #在开启sendfile的情况下，合并请求统一发送给客户端

    keepalive_timeout  65 65; #会话保持时间

    gzip  on;              #开启文件压缩
    gzip_comp_level 5;     #压缩级别1-9
    gzip_min_length 1K;   #压缩的最小文件，低于这个值不会压缩
    gzip_types text/plain application/javascript application/x-javascript text/cssapplication/xml text/javascript application/x-httpd-php  image/jpeg image/gif image/png;                 #需要压缩的文件类型
    gzip_vary on;             #压缩后在响应报文首部插入Vary: Accept-Encoding

    server_tokens off;     #隐藏nginx版本信息

    upstream tomcat-webserver {
        server tomcat-web1:8080;
        server tomcat-web2:8080;
    }
    server {
        listen       80;                     
        server_name  localhost;     
        charset utf-8;                 
        location /html {                            
            root   /usr/local/nginx/html;          
            index  index.html index.htm;    
        }
        location /app {
            index index.jps;
            proxy_pass http://tomcat-webserver;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
#web文件
[13:41:15 root@docker1 nginx]#cat web/index.html
<h1>www.zhangzhuo.org nginx</h1>
```

### 8.7.2.3 准备tomcat的web文件
```
[13:41:45 root@docker1 data]#mkdir tomcat/web -p
[13:46:00 root@docker1 data]#cat /data/tomcat/web/index.jsp 
<%@page import="java.util.Enumeration"%>
<br />
host:
<%try{out.println(""+java.net.InetAddress.getLocalHost().getHostName());}catch(Exception e){}%>
<br />
remoteAddr: <%=request.getRemoteAddr()%>
<br />
remoteHost: <%=request.getRemoteHost()%>
<br />
sessionId: <%=request.getSession().getId()%>
<br />
serverName:<%=request.getServerName()%>
<br />
scheme:<%=request.getScheme()%>
<br />
<%request.getSession().setAttribute("t1","t2");%>
<%
Enumeration en = request.getHeaderNames();
while(en.hasMoreElements()){
String hd = en.nextElement().toString();
out.println(hd+" : "+request.getHeader(hd));
out.println("<br />");
}
%>
``` 

### 8.7.3 准备docker-compose.yml文件
```
[13:47:24 root@docker1 data]#mkdir /docker/web-server
[14:23:56 root@docker1 web-server]#cat docker-compose.yml 
service-tomcat-web1:
    image: harbor.zhangzhuo.org/web1/tomcat-web:app1
    container_name: tomcat-web1
    volumes:
      - /data/tomcat/web:/apps/tomcat/webapps/app
    expose:
      - 8080
service-tomcat-web2:
    image: harbor.zhangzhuo.org/web1/tomcat-web:app1
    container_name: tomcat-web2
    volumes:
      - /data/tomcat/web:/apps/tomcat/webapps/app
    expose:
      - 8080
service-nginx-server1:
    image: harbor.zhangzhuo.org/nginx/centos-nginx:v1
    container_name: nginx-server1
    volumes:
      - /data/nginx/conf/nginx.conf:/usr/local/src/nginx/conf/nginx.conf
      - /data/nginx/web:/usr/local/nginx/html/html
    expose:
      - 80
    links:                                                                          
      - service-tomcat-web1
      - service-tomcat-web2
service-nginx-server2:
    image: harbor.zhangzhuo.org/nginx/centos-nginx:v1
    container_name: nginx-server2
    volumes:
      - /data/nginx/conf/nginx.conf:/usr/local/src/nginx/conf/nginx.conf
      - /data/nginx/web:/usr/local/nginx/html/html
    expose:
      - 80
    links:                                                                          
      - service-tomcat-web1
      - service-tomcat-web2
service-haproxy:
    image: harbor.zhangzhuo.org/web1/haproxy-web1:v1
    container_name: haproxy-server
    volumes:
      - /data/haproxy/haproxy.cfg:/etc/haproxy/haproxy.cfg
    expose:
      - 80
      - 9999
    ports:
      - "80:80"
      - "9999:9999"
    links:
      - service-nginx-server1
      - service-nginx-server2
```

### 8.7.4启动并测试
```
[14:24:45 root@docker1 web-server]#docker-compose up -d
Creating tomcat-web1 ... done
Creating tomcat-web2 ... done
Creating nginx-server2 ... done
Creating nginx-server1 ... done
Creating haproxy-server ... done
#测试
[14:25:20 root@docker1 web-server]#curl 192.168.10.181/html/
<h1>www.zhangzhuo.org nginx</h1>
[14:25:22 root@docker1 web-server]#curl 192.168.10.181/html/
<h1>www.zhangzhuo.org nginx</h1>
[14:25:22 root@docker1 web-server]#curl 192.168.10.181/app/

<br />
host:
f60a44b97221

<br />
remoteAddr: 10.10.0.4
<br />
remoteHost: 10.10.0.4
<br />
sessionId: 99F437EEFF682BE9A0ADD9818E813398
<br />
serverName:tomcat-webserver
<br />
scheme:http
<br />

host : tomcat-webserver
<br />
connection : close
<br />
user-agent : curl/7.58.0
<br />
accept : */*
<br />
x-forwarded-for : 192.168.10.181
<br />

[14:25:28 root@docker1 web-server]#curl 192.168.10.181/app/

<br />
host:
f60a44b97221

<br />
remoteAddr: 10.10.0.5
<br />
remoteHost: 10.10.0.5
<br />
sessionId: E971CF9571D72126FFE4A49D26C3E381
<br />
serverName:tomcat-webserver
<br />
scheme:http
<br />

host : tomcat-webserver
<br />
connection : close
<br />
user-agent : curl/7.58.0
<br />
accept : */*
<br />
x-forwarded-for : 192.168.10.181
<br />
```

# 九、Docker swarm集群

## 9.1 Docker swarm安装
docker swarm不需要任何组件，安装完docker就可以直接使用

### 9.1.1 初始化集群
```
#初始化集群--advertise-addr声明当前机器IP，初始化集群之后这台主机就是管理节点
[15:28:36 root@docker1 ~]#docker swarm init --advertise-addr 192.168.10.181
Swarm initialized: current node (37npwq1hkr28vfhpjlx49xd5r) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0bsi3faeyks8qubj2luyc89zbxmqkwqn0hz3lyhprwkb9b20lo-1kaadrziqvn4ci6kagisokv0q 192.168.10.181:2377   #在其他节点运行这个命令就可以加入到swarm集群

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

#二次获取token
[15:31:43 root@docker1 ~]#docker swarm join-token worker 
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0bsi3faeyks8qubj2luyc89zbxmqkwqn0hz3lyhprwkb9b20lo-1kaadrziqvn4ci6kagisokv0q 192.168.10.181:2377
```

### 9.1.2 添加节点
```
#在其他节点加入swarm
[15:24:02 root@docker2 ~]# docker swarm join --token SWMTKN-1-0bsi3faeyks8qubj2luyc89zbxmqkwqn0hz3lyhprwkb9b20lo-1kaadrziqvn4ci6kagisokv0q 192.168.10.181:2377
This node joined a swarm as a worker.
[15:33:48 root@docker3 ~]#docker swarm join --token SWMTKN-1-0bsi3faeyks8qubj2luyc89zbxmqkwqn0hz3lyhprwkb9b20lo-1kaadrziqvn4ci6kagisokv0q 192.168.10.181:2377
This node joined a swarm as a worker.
#验证，必须在管理节点验证
[15:41:52 root@docker1 ~]#docker node ls
ID                            HOSTNAME                STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
37npwq1hkr28vfhpjlx49xd5r *   docker1.zhangzhuo.org   Ready               Active              Leader              19.03.15
t5c3nh0zmo8lm06dw0roihig9     docker2.zhangzhuo.org   Ready               Active                                  19.03.6
z6xak45b7uwhj6rox7arp2n67     docker3.zhangzhuo.org   Ready               Active                                  19.03.15
```

### 9.1.3 添加label
初始化完成，添加label，在管理节点
```
[15:45:57 root@docker1 ~]#docker node update --label-add name=swarm-master1 docker1.zhangzhuo.org
docker1.zhangzhuo.org
[15:45:59 root@docker1 ~]#docker node update --label-add name=swarm-master2 docker2.zhangzhuo.org
docker2.zhangzhuo.org
[15:46:05 root@docker1 ~]#docker node update --label-add name=swarm-master3 docker3.zhangzhuo.org
docker3.zhangzhuo.org
```

### 9.1.4 将其他节点提升为manager角色实现高可用
```
[15:46:29 root@docker1 ~]#docker node  promote docker2.zhangzhuo.org
Node docker2.zhangzhuo.org promoted to a manager in the swarm.
[15:48:15 root@docker1 ~]#docker node  promote docker3.zhangzhuo.org
Node docker3.zhangzhuo.org promoted to a manager in the swarm.
#验证
[15:48:20 root@docker1 ~]#docker node ls
ID                            HOSTNAME                STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
37npwq1hkr28vfhpjlx49xd5r *   docker1.zhangzhuo.org   Ready               Active              Leader              19.03.15
t5c3nh0zmo8lm06dw0roihig9     docker2.zhangzhuo.org   Ready               Active              Reachable           19.03.6
z6xak45b7uwhj6rox7arp2n67     docker3.zhangzhuo.org   Ready               Active              Reachable           19.03.15
```

查看node信息
```
[15:50:02 root@docker1 ~]#docker node inspect docker2.zhangzhuo.org 
[15:50:02 root@docker1 ~]#docker node inspect docker3.zhangzhuo.org
```

### 9.1.5 创建网络
```
#查看帮助
[15:50:28 root@docker1 ~]#docker network create --help
#创建overlay网络
[15:50:28 root@docker1 ~]#docker network create -d overlay --subnet=172.16.0.0/16 --gateway=172.16.0.1 --attachable jie-net
w7mc360xn2yrgsvaxelub0mn1
#验证
[15:52:17 root@docker1 ~]#docker network inspect jie-net 
[
    {
        "Name": "jie-net",
        "Id": "w7mc360xn2yrgsvaxelub0mn1",
        "Created": "2021-04-20T07:52:17.69187683Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.16.0.0/16",
                    "Gateway": "172.16.0.1"
                }
            ]
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

### 9.1.6 创建容器测试
```
#帮助查看
[15:57:00 root@docker1 ~]#docker service --help

Usage:	docker service COMMAND

Manage services

Commands:
  create      Create a new service
  inspect     Display detailed information on one or more services
  logs        Fetch the logs of a service or task
  ls          List services
  ps          List the tasks of one or more services
  rm          Remove one or more services
  rollback    Revert changes to a service's configuration
  scale       Scale one or multiple replicated services
  update      Update a service

Run 'docker service COMMAND --help' for more information on a command.
#具体命令的帮助
[15:57:49 root@docker1 ~]#docker service create --help
#创建--replicas副本数
[15:55:41 root@docker1 ~]#docker service create --replicas 2 -p 8080:8080 --network jie-net --name tomcat-web harbor.zhangzhuo.org/web1/tomcat-web:app1 
y7y84gmm2i6yyvc2g7khw5mbj
overall progress: 2 out of 2 tasks 
1/2: running   
2/2: running   
verify: Service converged 
#验证
[15:58:42 root@docker1 ~]#docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                                       PORTS
y7y84gmm2i6y        tomcat-web          replicated          2/2                 harbor.zhangzhuo.org/web1/tomcat-web:app1   *:8080->8080/tcp
[16:34:43 root@docker1 ~]#docker service ps tomcat-web 
ID                  NAME                IMAGE                                       NODE                    DESIRED STATE       CURRENT STATE            ERROR               PORTS
tbnfzx6iyh9a        tomcat-web.1        harbor.zhangzhuo.org/web1/tomcat-web:app1   docker1.zhangzhuo.org   Running             Running 37 minutes ago                       
z7dkd5lr7ume        tomcat-web.2        harbor.zhangzhuo.org/web1/tomcat-web:app1   docker2.zhangzhuo.org   Running             Running 37 minutes ago 
#测试访问，由于副本是俩个肯定有一个主机没有容器，但是所有节点只要访问8080端口都是可以的
[16:37:18 root@docker3 ~]#curl 192.168.10.183:8080/myapp/
Tomcat Web Page1
```

#### 9.1.6.1 测试管理节点宕机
```
[16:40:58 root@docker1 ~]#docker node ls
ID                            HOSTNAME                STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
37npwq1hkr28vfhpjlx49xd5r *   docker1.zhangzhuo.org   Ready               Active              Leader              19.03.15
t5c3nh0zmo8lm06dw0roihig9     docker2.zhangzhuo.org   Ready               Active              Reachable           19.03.6
z6xak45b7uwhj6rox7arp2n67     docker3.zhangzhuo.org   Ready               Active              Reachable           19.03.15
#关闭docker服务
[16:42:34 root@docker1 ~]#systemctl stop docker.service 
[16:43:06 root@docker1 ~]#systemctl stop docker.socket 
#其他节点查看
[16:43:24 root@docker2 ~]#docker node ls
ID                            HOSTNAME                STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
37npwq1hkr28vfhpjlx49xd5r     docker1.zhangzhuo.org   Down                Active              Unreachable         19.03.15
t5c3nh0zmo8lm06dw0roihig9 *   docker2.zhangzhuo.org   Ready               Active              Reachable           19.03.6
z6xak45b7uwhj6rox7arp2n67     docker3.zhangzhuo.org   Ready               Active              Leader              19.03.15
#查看刚刚启动的容器运行状况，原本docker1主机上的容器已经迁移到docker3
[16:44:41 root@docker2 ~]#docker service ps tomcat-web 
ID                  NAME                IMAGE                                       NODE                    DESIRED STATE       CURRENT STATE            ERROR               PORTS
3b0hhjeed7g0        tomcat-web.1        harbor.zhangzhuo.org/web1/tomcat-web:app1   docker3.zhangzhuo.org   Running             Running 2 minutes ago                        
tbnfzx6iyh9a         \_ tomcat-web.1    harbor.zhangzhuo.org/web1/tomcat-web:app1   docker1.zhangzhuo.org   Shutdown            Shutdown 2 minutes ago                       
z7dkd5lr7ume        tomcat-web.2        harbor.zhangzhuo.org/web1/tomcat-web:app1   docker2.zhangzhuo.org   Running             Running 2 minutes ago    
[16:44:43 root@docker2 ~]#docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                                       PORTS
y7y84gmm2i6y        tomcat-web          replicated          2/2                 harbor.zhangzhuo.org/web1/tomcat-web:app1   *:8080->8080/tcp
```

**恢复docker1节点**
```
[16:43:11 root@docker1 ~]#systemctl start docker.socket 
[16:46:51 root@docker1 ~]#systemctl start docker.service 
#验证，恢复docker1节点会从新加入集群但是刚刚接管的管理节点会一直为管理节点
[16:47:37 root@docker1 ~]#docker node ls
ID                            HOSTNAME                STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
37npwq1hkr28vfhpjlx49xd5r *   docker1.zhangzhuo.org   Ready               Active              Reachable           19.03.15
t5c3nh0zmo8lm06dw0roihig9     docker2.zhangzhuo.org   Ready               Active              Reachable           19.03.6
z6xak45b7uwhj6rox7arp2n67     docker3.zhangzhuo.org   Ready               Active              Leader              19.03.15
#验证容器
[16:48:05 root@docker1 ~]#docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                                       PORTS
y7y84gmm2i6y        tomcat-web          replicated          2/2                 harbor.zhangzhuo.org/web1/tomcat-web:app1   *:8080->8080/tcp
[16:48:07 root@docker1 ~]#docker service ps tomcat-web 
ID                  NAME                IMAGE                                       NODE                    DESIRED STATE       CURRENT STATE            ERROR               PORTS
3b0hhjeed7g0        tomcat-web.1        harbor.zhangzhuo.org/web1/tomcat-web:app1   docker3.zhangzhuo.org   Running             Running 5 minutes ago                        
tbnfzx6iyh9a         \_ tomcat-web.1    harbor.zhangzhuo.org/web1/tomcat-web:app1   docker1.zhangzhuo.org   Shutdown            Shutdown 5 minutes ago                       
z7dkd5lr7ume        tomcat-web.2        harbor.zhangzhuo.org/web1/tomcat-web:app1   docker2.zhangzhuo.org   Running             Running 6 minutes ago
```
## 9.2 docker swarm集群管理

### 9.2.1 移除集群节点
```
#离开集群，在要移除的节点执行，主节点切记不要执行
[17:22:16 root@docker1 ~]#docker swarm leave --force
Node left the swarm.
#以下在管理节点执行
#降级
[17:15:19 root@docker3 ~]#docker node demote docker1.zhangzhuo.org 
Manager docker2.zhangzhuo.org demoted in the swarm.
#移除
[17:23:56 root@docker3 ~]#docker node rm docker1.zhangzhuo.org 
docker1.zhangzhuo.org
```
