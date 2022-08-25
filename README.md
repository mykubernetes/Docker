| 官方教程 | 网址 |
|---------|------|
| 官方Dockerfile | https://github.com/docker-library |
| 官方Dockerfile详解 | https://docs.docker.com/engine/reference/builder/ |
| 官方Docker compose | https://docs.docker.com/compose/compose-file/compose-versioning/ |
| docker二进制包下载地址 | https://download.docker.com/linux/static/stable/x86_64/ |
| docker-book | https://github.com/frank6866/gitbook-docker |
| 博客 | https://blog.51cto.com/qiangsh/2165048 |
| 博客 | https://www.cnblogs.com/xuweiweiwoaini/category/1809813.html |
| 博客 | https://blog.csdn.net/m0_46090675/article/details/121846718?spm=1001.2014.3001.5502 |

| Docker网络模式 | 配置 | 说明 |
|---------------|------|------|
| host模式 | –net=host | 容器和宿主机共享Network namespace |
| container模式 | –net=container:NAME_or_ID | 容器和另外一个容器共享Network namespace。 kubernetes中的pod就是多个容器共享一个Network namespace |
| none模式 | –net=none | 容器有独立的Network namespace，但并没有对其进行任何网络设置，如分配veth pair 和网桥连接，配置IP等 |
| bridge模式 | –net=bridge | （默认为该模式） |



| Namespaces | 描述 |
|--------------|-------|
| Mount namespaces | 挂载点 
| UTS namespaces | 主机名与域名 
| IPC namespaces | 信号量、消息队列和共享内存 
| PID namespaces | 进程号 
| Network namespaces | 网络设备、网络栈、端口等 
| User namespaces | 用户和组 

| cgroups | 描述 |
|--------|---------|
| blkio | 块设备IO |
| cpu | CPU |
| cpuacct | CPU资源使用报告 |
| cpuset | 多处理器平台上的CPU集合 |
| devices | 设备访问 |
| freezer | 挂起或恢复任务 |
| memory | 内存用量及报告 |
| perf_event | 对cgroup中的任务进行统一性能测试 |
| net_cls | cgroup中的任务创建的数据报文的类别标识符 |

相关命令
---

1、docker image pull 拉取镜像
```
# 完整命令
docker image pull <Registry>/<Repository>:<Tag>

# 默认从 Docker Hub 上拉取镜像时
docker image pull <Repository>:<Repository>：<Tag>

# 从非官方的 Docker Hub 中拉取，仓库前面需要加上 Docker Hub 的用户名或者组织名。
docker image pull <YourDockerID>/<Repository>:<Tag>

# 从第三方仓库服务（非 docker hub）中拉取镜像
docker image pull gcr.io/demp/tu-demp:v2

docker image pull alpine:latest	# 从官方 Docker Hub 的 alpine 仓库中拉取标有 latest 标签的镜像
docker image pull ubuntu:latest
docker image pull redis:latest

docker image pull alpine 		# 默认拉取标签为 latest 的镜像

docker image pull mongo:3.3.11	# 拉取非 latest 标签的镜像

docker image pull dawnguo/docker-hexo:alpine  # 从非官方的 Docker Hub 中拉取，仓库前面需要加上 Docker Hub 的用户名或者组织名
```

2、docker image rm/prune 删除镜像
```
# 从 Docker 主机删除镜像，删除操作会在当前主机上删除该镜像以及相关的镜像层。但是，如果某个镜像层被多个镜像共享，那么只有当全部依赖该镜像层的镜像全都删除之后，该镜像层才会删除。
# 另外被删除的镜像上存在运行状态的容器，删除不会被允许，所以需要停止并删除该镜像相关的全部容器之后才能删除镜像。
# 输出内容中：每一个 Deleted：行都表示一个镜像层被删除
docker image rm <ImageID>	# 根据 image id 来删除镜像
docker rmi	# 也可以

docker image rm <ImageID1> <ImageID2> ...	# 删除多个

docker image rm <Repository>:<Tag>	# tag 同样可以省略

docker image rm $(docker image ls -q) -f # 删除本地系统中的全部镜像

docker image prune # 移除全部的 dangling 镜像

docker image prune -a # 额外移除没有被使用的镜像（没有被任何容器使用的镜像）
```

3、docker image ls 查看镜像
```
docker image ls
docker image ls <Repository>	# 显示名为 Repository 的镜像
docker image ls -a	# 显示所有镜像
docker image ls -q # 只返回系统本地拉取的全部镜像 ID 列表
docker image ls --digests # 查看镜像的 SHA256 签名

docker images # 等同于 docker image ls
```
可以使用 --filter 参数来过滤 docker image ls 命令返回的镜像列表内容，支持四种过滤器：
- dangling：可以指定为 true 或者 false，指定 true 仅返回悬虚镜像，指定 false 仅返回非悬虚镜像。
- before：需要镜像名称或者 ID 作为参数，返回在指定镜像之前被创建的全部镜像
- since：与 before 类似，不过返回的是指定镜像之后创建的全部镜像
- label：根据标注（label）的名称或者值，对镜像进行过滤。docker image ls 命令输出中将不显示标注内容。
```
docker image ls --filter dangling=true# dangling 镜像是指那些没有标签的镜像，在列表中显示 <none>：<none>。通常出现这种情况是因为构建了一个新的镜像，并且为这个镜像打上了已经存在的标签。那么旧镜像就变成了悬虚（dangling）镜像了。
```

还可以使用 reference 的方式过滤。
```
# 过滤并只显示标签带 latest 的
docker image ls --filter=reference="*:latest"
```

使用 --format 来通过 Go 模板对输出内容进行格式化
```
docker image ls --format "{{.Size}}"# 只返回 Docker 主机上的镜像的大小属性
docker image ls --format "{{.Reposity}}:{{.Tag}}:{{.Size}}"# 只显示仓库、标签和大小
```

4、docker search 查找镜像
```
# docker search 命令允许通过 CLI 的方式搜索 Docker Hub。可通过 "Repository" 字段（仓库名称）的内容进行匹配，并且对返回内容中任意列的值进行过滤。返回的镜像中既有官方的也有非官方的。并且默认情况下只显示 25 行
docker search alpine	# 查找 Repository 字段中带有 alpine 的

docker search apline --filter "is-official=true"
docker search apline --filter "is-automated=true"

docker search apline --limit 30 # 增加返回内容的行数，最多 100 行
```

5、docker image inspect
```
# 查看镜像的组成情况
docker image inspect <ImageID>||<Repository>:<Tag>
```

6、查看 Docker 情况
```
docker version		# 当命令的输出包含 Client 和 Server 内容的时候，表示 daemon 已经启动

docker info	# 返回所有容器和镜像的数量、Docker 使用的执行驱动和存储驱动，以及 Docker 的基本配置

service docker status	# 查看 docker 的状态

systemctl is-active docker	# 查看 docker 的状态
```

7、容器周期相关操作
```
# 启动容器最基础的格式，指定了启动所需的镜像以及要运行的应用
# 省略了 <tag>,那么默认是 latest
docker container run <Options> <Repository>:<Tag> <App>	
docker run <Options> <Repository>:<Tag> <App>	# 这个命令也可以

# 启动某个 Ubuntu Linux 容器，并在其中运行 bash shell 作为其应用。
# -it 参数表示将当前 shell 连接到容器的 shell 终端之上，并且与容器具有交互。具体为：-i 表示容器中的 STDIN 是开启的，-t 为要创建的容器分配一个伪 tty 终端
# 在启动的容器中运行某些指令，可能无法正常工作，这是因为大部分容器镜像都是经过高度优化的，有些指令并没有被打包进去。
docker container run -it ubuntu:latest	/bin/bash	

# -d 表示后台模式，告知容器在后台运行
docker container run -d ubuntu sleep 1m	

# 设置容器名字为 percy（合法的名字是可包含：大小写字母、数字、下划线、圆点、横线）
docker container run --name percy -it ubuntu:latest /bin/bash	

# 配置重启策略，采用 always 策略，--restar 标志会检查容器的退出代码
docker container run --restart -it always ubuntu:latest /bin/bash

# -p 80:8080 将 Docker 主机的 80 端口映射到容器内的 8080 端口。当有流量访问主机的 80 端口时
# 流量会直接映射到容器内的 8080 端口。
docker container run -d -p 80:8080 <Repository>:<Tag> <App>

# 在宿主机上随便选择一个位于 49153~65535 之间的一个端口来映射到容器的 80 端口，之后可以通过 docker ps -l 或者 docker port 来查看端口映射情况
docker container run -d -p 80 <Repository>:<Tag> <App>

# 将 Dockerfile 中 EXPOSE 指令的端口都随机映射到主机的端口上，注意是大写的 P
docker container run -d -P <Repository>:<Tag> <App>

# 将容器内的 80 端口绑定到本地宿主机 127.0.0.1 这个 IP 地址的 80 端口上
docker container run -d -p 127.0.0.1:80:80	

# 没有指定要绑定的宿主机端口号，随机绑定到宿主机 127.0.0.1 的一个端口上
docker container run -d -p 127.0.0.1::80	


# 有时不指定要运行的 app 也是可以的，这是因为构建镜像时指定了默认命令。可以通过 docker image inspect 查看，如果有设置了 Cmd 那么表示在基于该镜像启动容器时会默认运行 Cmd。当然，也可以自己指定，但是这种在构建时指定默认命令是一种很普遍的做法，可以简化容器的启动。
docker container run <Repository>:<Tag>

# 在这次运行时覆盖原 Dockerfile 中的 ENTRYPOINT 指令
docker container run --entrypoint <Command> <Repository>:<Tag> <Params>
# 停止运行中的容器，并将状态设置为 Exited(0)，发送 SIGTERM 信号
docker container stop <ContainerName>||<ContainerID>
docker stop <ContainerName>||<ContainerID> # 同理也可以

# 也可以使用 docker kill 命令停止容器，只是发出的是 SIGKILL 信号
docker kill <ContainerName>||<ContainerID>
```

```
# 重启处于停止（Exited）状态的容器
docker container start <ContainerName>||<ContainerID>
docker start <ContainerName>||<ContainerID>
```

```
# 删除停止运行的容器
docker container rm <ContainerName>||<ContainerID>
docker rm <ContainerName>||<ContainerID>

# 一次性删除运行中的容器，但是建议先停止之后再删除。
docker container rm <ContainerName>||<ContainerID> -f

# 清理掉 Docker 主机上全部运行的容器
docker container rm $(docker container ls -aq) -f
```

```
# Docker 1.3 之后允许使用 docker container exec（或者 docker exec）命令在运行状态的容器中，启动一个新进程，也就是会创建新进程。在将 Docker 主机的 Shell 连到一个正在运行中容器的终端十分有用。
docker container exec <options> <ContainerName>/<ContainerID> <app>
docker container exec -it ubuntu bash	# 会在容器内部启动一个新的 Bash Shell，并连接到该 bash
docker exec

# 重新附着到该容器的会话上，比如之前开了一个 shell，之后退出又重新 start 了，那么可以可以使用这个，重新连接之前的 shell
docker attach <ContainerName>||<ContainerID>
```

```
Ctrl-PQ 	# 退出容器，但并不终止容器运行。会切回 Docker 主机的 shell，并保持容器在后台运行
```

8、查看容器
```
docker container ls		# 观察当前系统正在运行(UP)状态的容器，如果有 port 选项的话，那么是 host-port:container-port 的格式
docker container ls -a	# 观察当前系统正在运行的容器容器，包括 stop 状态的
```

```
docker ps		# 查看当前系统中正在运行的容器
docker ps -a	# 查看当前系统中所有的容器（运行和停止）
dokcer ps -l	# 列出最后一次运行的容器（运行和停止）
docker ps -n x	# 显示最后 x 个容器（运行和停止）
```

```
# 查看指定容器的指定端口的映射情况
docker port <ContainerName>||<ContainerID> 80
```

9、其他
```
docker inspect <ContainerName>||<ContainerID>
docker container inspect <ContainerName>||<ContainerID> # 显示容器的配置信息（名称、命令、网络配置等）和运行时信息
docker inspect --format '{{.NetworkSettings.IPAddress}}'# 支持 -f 或者 --format 标志查看选定内容的结果，就跟 image ls 那个一样的

docker logs	# 获取守护式容器的日志
docker logs -f # 监控守护式容器的日志（跟 tail 命令使用差不多）
docker logs -t # 显示守护式容器日志的时间戳

docker top	# 查看容器内部运行的进程
```

10、Dockerfile
```
# Linux x64
FROM alpine

LABEL maintainer="multiparam@gmail.com"

# Install Node and NPM
RUN apk add --update nodejs nodejs-npm

# Copy app to /src
COPY . /src

WORKDIR /src

# Install dependencies
RUN  npm install

EXPOSE8080

ENTRYPOINT ["node", "./app.js"]
```

12、构建dockerfile
```
# 构建出一个叫 web:latest 的镜像，.（点）表示将当前目录作为构建上下文并且当前目录需要包含 Dockerfile
# 如果没有指定标签的话，那么会默认设置一个 latest 标签
docker image build -t web:latest .
# 可以通过 docker image build 的输出内容了解镜像的构建过程，而构建过程的最终结果是返回了新镜像的 ID。其实，构建的每一步都会返回一个镜像的 ID。
```

13、docker image build 构建镜像
```
# 构建出一个叫 web:latest 的镜像（-t 参数是为镜像打标签），. 表示将当前目录作为构建上下文并且当前目录需要包含 Dockerfile（当然也可以通过 -f 参数可以指定位于任意路径下的任意名称的 Dockfile）
docker image build -t web:latest .

# 强制忽略对缓存的使用。为什么需要这个功能呢？因为如果构建中有用到 apt-get update 的话，那么碰到了由带有 apt-get update 构建出来的缓存内容之后，Docker 将不会刷新 APT 包的缓存，但是此时你又需要每个包的最新版本。
docker image build --no-cache -t web:latest

docker image build -t web:latest --squash			# 创建一个合并的镜像

# 可以指定一个 git 仓库的源地址来指定 Dockerfile 的位置
docker image build -t web:latest git@github.com:test/test
```

14、docker commit 构建镜像
```
# 提交一个新的镜像层，增加在原来的镜像之上
docker commit <ContainerID>  <NewRepository>:<Tag>

# 提交时可以指定更多的数据
docker commit -m="A new custom image" --author="DawnGuo"  <ContainerID>  <NewRepository>:<Tag>
```

15、推送镜像相关
```
# 在推送之前，先使用 Docker ID 登录 Docker Hub。个人认证信息会保存到 $HOME/.docker/config.json
docker login

# 在推送之前，可能需要为镜像打上标签
docker imgae tag <CurrentRepositry>:<CurrentTag> <NewRepositry>:<NewTag>

# 下面这个例子会将镜像推送到 <Registry>/<Repositry> 这个仓库中
# 在推送过程中需要指定 Registry（镜像仓库服务）、Repository（镜像仓库）和 Tag（镜像标签）等信息
# 但是不指定 Registry 和 Tag 也可以，Docker 会默认 Registry 为 docker.io，Tag 为 latest，但是 Repository 没有默认值，而是从被推送的镜像的 Repositry 属性值获取（比如 web:latest 中的 web 值就作为 Repositry）
docker image push <Registry>/<Repositry>:<Tag>


# 下面展示如何向 Docker Hub 中推荐，在创建了自己的 Docker ID 并登录的前提下。
# 1. 首先是给镜像打上标签，因为假如直接将 web:latest 进行推送的话，其实是直接推送到 docker.io/web:latest 中，但是对于 docker.io/web 这个仓库我们是没有访问权限的，我们顶多是推送到 docker.io/<YourDockerID>/web 中，所以我们需要将其重新打标签，相当于将仓库重命名。
# 2. 之后推送打上新标签的仓库，那么将会将这个镜像 push 到 docker.io/<YourDockerID>/web 这个仓库中了
docker image tag web:latest <YourDockerID>/web:latest

docker image push <YourDockerID>/web:latest
```

16、其他
```
# 查看构建镜像的过程中都执行了哪些指令，每行显示一个指令，那些 SIZE 列对应的数值不为零的指令表示会新建镜像层
docekr image history <REPOSITORY>:<TAG>
docker history
docker inspect
```
