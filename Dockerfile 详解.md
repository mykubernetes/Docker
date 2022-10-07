# 一、Dockerfile（制作镜像脚本化）

## 1.1 什么是Dockerfile
Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，用于构建镜像。每一条指令构建一层镜像，因此每一条指令的内容，就是描述该层镜像应当如何构建。
- dockerfile 用于指示 docker image build 命令自动构建Image的源代码
- 是纯文本文件

示例：
```
docker build -f /path/Dockerfile
```

## 1.2 为什么要使用Dockerfile

问题:在dockerhub中官方提供很多镜像已经能满足我们的所有服务了,为什么还需要自定义镜像

核心作用:日后用户可以将自己应用打包成镜像,这样就可以让我们应用进行容器运行.还可以对官方镜像做扩展，以打包成我们生产应用的镜像。


Dockerfile的格式,两种类型的行
- 以# 开头的注释行
- 由专用“指令（Instruction）”开头的指令行

由Image Builder顺序执行各指令，从而完成Image构建


# 三、 Dockerfile常用指令

[官方build参考](https://docs.docker.com/engine/reference/builder/)

| 指令 | 功能介绍 |
|-----|----------|
| FROM | 指定构建新Image时使用的基础Image，通常必须是Dockerfile的第一个有效指令，但其前面也可以出现ARG指令 |
| LABEL | 附加到Image之上的元数据，键值格式 |
| ENV | 以键值格式设定环境变量，可被其后的指令所调用，且基于新生成的Image运行的Container中也会存在这些变量 |
| RUN | 以FROM中定义的Image为基础环境运行指定命令，生成结果将作为新Image的一个镜像层，并可由后续指令所使用 |
| CMD | 基于该Dockerfile生成的Image运行Container时，CMD能够指定容器中默认运行的程序，因而其只应该定义一次 |
| ENTRYPOINT | 类似于CMD指令的功能，但不能被命令行指定要运行的应用程序覆盖，且与CMD共存时，CMD的内容将作为该指令中定义的程序的参数 |
| WORKDIR | 为RUN、CMD、ENTRPOINT、COPY和ADD等指令设定工作目录 |
| COPY | 复制主机上或者前一阶段构建结果中（需要使用–from选项）文件或目录生成新的镜像层 |
| ADD | 与COPY指令的功能相似，但ADD额外也支持使用URL指定的资源作为源文件 |
| VOLUME | 指定基于新生成的Image运行Container时期望作为Volume使用的目录 |
| EXPOSE | 指定基于新生成的Image运行Container时期望暴露的端口，但实际暴露与否取决于“docker run”命令的选项，支持TCP和UDP协议 |
| USER | 为Dockerfile中该指令后面的RUN、CMD和ENTRYPOING指令中要运行的应用程序指定运行者身份UID，以及一个可选的GID |
| ARG | 定义专用于build过程中的变量，但仅对该指标之后的调用生效，其值可由命令行选项“–build-arg”进行传递 |
| ONBUILD | 触发器，生效于由该Dockerfile构建出的新Image被用于另一个Dockerfile中的FROM指令作为基础镜像时 |
| STOPSIGNAL | 用于通知Container终止的系统调用信号 |
| HEALTHCHECK | 定义检测容器应用的健康状态的具体方法 |
| SHELL | 为容器定义运行时使用的默认shell程序，Linux系统默认使用 [“/bin/sh”,”-c”]，Windows默认使用 [“cmd”, “/S”, “/C”] |


## 2.1 FROM

- 指定基础镜像，必须为第一个命令
```
格式：
　　FROM <image>
　　FROM <image>:<tag>
　　FROM <image>@<digest>

示例：　　
	FROM mysql:5.6
注：
   tag或digest是可选的，如果不使用这两个值时，会使用latest版本的基础镜像
```

## 2.2 MAINTAINER(新版即将废弃)

维护者信息
```
格式：
    MAINTAINER <name>
示例：
    MAINTAINER bertwu
    MAINTAINER xxx@163.com
    MAINTAINER bertwu <xxx@163.com>
```
  
## 2.3 RUN

构建镜像时执行的命令
```
RUN用于在构建镜像时执行命令，其有以下两种命令执行方式：
shell执行
格式：
    RUN <command>
exec执行
格式：
    RUN ["executable", "param1", "param2"]
示例：
    RUN ["executable", "param1", "param2"]
    RUN apk update
    RUN ["/etc/execfile", "arg1", "arg1"]
注：RUN指令创建的中间镜像会被缓存，并会在下次构建中使用。如果不想使用这些缓存镜像，
可以在构建时指定--no-cache参数，如：docker build --no-cache
```

## 2.4 ADD

将本地文件添加到容器中，tar类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似wget
```
格式：
    ADD <src>... <dest>
    ADD ["<src>",... "<dest>"] 用于支持包含空格的路径
示例：
    ADD hom* /mydir/          # 添加所有以"hom"开头的文件
    ADD hom?.txt /mydir/      # ? 替代一个单字符,例如："home.txt"
    ADD test relativeDir/     # 添加 "test" 到 `WORKDIR`/relativeDir/
    ADD test /absoluteDir/    # 添加 "test" 到 /absoluteDir/
```
  
## 2.5 COPY

功能类似ADD，但是是不会自动解压文件，也不能访问网络资源

## 2.6 CMD

构建镜像后调用，也就是在容器启动时才进行调用。
```
格式：
    CMD ["executable","param1","param2"] (执行可执行文件，优先)
    CMD ["param1","param2"] (设置了ENTRYPOINT，则直接调用ENTRYPOINT添加参数)
    CMD command param1 param2 (执行shell内部命令)
示例：
    CMD echo "This is a test." | wc -l
    CMD ["/usr/bin/wc","--help"]

注：CMD不同于RUN，CMD用于指定在容器启动时所要执行的命令，而RUN用于指定镜像构建时所要执行的命令。
```

## 2.7 ENTRYPOINT
配置容器，使其可执行化。配合CMD可省去"application"，只使用参数。
```
格式：
    ENTRYPOINT ["executable", "param1", "param2"] (可执行文件, 优先)
    ENTRYPOINT command param1 param2 (shell内部命令)
示例：
    FROM ubuntu
    ENTRYPOINT ["ls", "/usr/local"]
    CMD ["/usr/local/tomcat"]
  之后，docker run 传递的参数，都会先覆盖cmd,然后由cmd 传递给entrypoint ,做到灵活应用

注：ENTRYPOINT与CMD非常类似，不同的是通过docker run执行的命令不会覆盖ENTRYPOINT，
 而docker run命令中指定的任何参数，都会被当做参数再次传递给CMD。
 Dockerfile中只允许有一个ENTRYPOINT命令，多指定时会覆盖前面的设置，
 而只执行最后的ENTRYPOINT指令。
 通常情况下，	ENTRYPOINT 与CMD一起使用，ENTRYPOINT 写默认命令，当需要参数时候 使用CMD传参
```

## 2.8 LABEL

用于为镜像添加元数据
```
格式：
    LABEL <key>=<value> <key>=<value> <key>=<value> ...
示例：
　　LABEL version="1.0" description="这是一个Web服务器" by="IT笔录"
注：
　　使用LABEL指定元数据时，一条LABEL指定可以指定一或多条元数据，指定多条元数据时不同元数据
　　之间通过空格分隔。推荐将所有的元数据通过一条LABEL指令指定，以免生成过多的中间镜像。
```

## 2.9 ENV
设置环境变量
```
格式：
    ENV <key> <value>  #<key>之后的所有内容均会被视为其<value>的组成部分，因此，一次只能设置一个变量
    ENV <key>=<value> ...  #可以设置多个变量，每个变量为一个"<key>=<value>"的键值对，如果<key>中包含空格，可以使用\来进行转义，也可以通过""来进行标示；另外，反斜线也可以用于续行
示例：
    ENV myName John Doe
    ENV myDog Rex The Dog	
    ENV myCat=fluffy
```

## 2.10 EXPOSE
指定于外界交互的端口
```
格式：
    EXPOSE <port> [<port>...]
示例：
    EXPOSE 80 443
    EXPOSE 8080    
    EXPOSE 11211/tcp 11211/udp
注：　　EXPOSE并不会让容器的端口访问到主机。要使其可访问，需要在docker run运行容器时通过-p来发布这些端口，或通过-P参数来发布EXPOSE导出的所有端口

如果没有暴露端口，后期也可以通过-p 8080:80方式映射端口，但是不能通过-P形式映射
```

## 2.11 VOLUME
用于指定持久化目录（指定此目录可以被挂载出去）
```
格式：
    VOLUME ["/path/to/dir"]
示例：
    VOLUME ["/data"]
    VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"
注：一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：
1 卷可以容器间共享和重用
2 容器并不一定要和其它容器共享卷
3 修改卷后会立即生效
4 对卷的修改不会对镜像产生影响
5 卷会一直存在，直到没有任何容器在使用它
```

## 2.12 WORKDIR
工作目录，类似于cd命令
```
格式：
    WORKDIR /path/to/workdir
示例：
    WORKDIR /a  (这时工作目录为/a)
    WORKDIR b  (这时工作目录为/a/b)
    WORKDIR c  (这时工作目录为/a/b/c)
注：　
  通过WORKDIR设置工作目录后，Dockerfile中其后的命令RUN、CMD、ENTRYPOINT、ADD、COPY
  等命令都会在该目录下执行。在使用docker run运行容器时，可以通过-w参数覆盖构建时所设置的工作目录。
```

## 2.13 USER

指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。使用USER指定用户时，可以使用用户名、UID或GID，或是两者的组合。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户
```
格式:　　
USER user　　
USER user:group　　
USER uid　　
USER uid:gid　　
USER user:gid　　
USER uid:group
 
示例：    　　
     USER www
 注：
　　使用USER指定用户后，Dockerfile中其后的命令RUN、CMD、ENTRYPOINT都将使用该用户。
　　镜像构建完成后，通过docker run运行容器时，可以通过-u参数来覆盖所指定的用户。
```

## 2.14 ARG

用于指定传递给构建运行时的变量(给dockerfile传参)，相当于构建镜像时可以在外部为里面传参
```
格式：
    ARG <name>[=<default value>]
示例：
    ARG site
    ARG build_user=www
```

```
From centos:7
ARG parameter
VOLUME /usr/share/nginx
RUN yum -y install $parameter
EXPOSE 80 443
CMD nginx -g "daemon off;"

# 可以这如下这样灵活传参
docker build --build-arg=parameter=net-tools -t nginx:01 . 
```

## 2.15 ONBUILD
用于设置镜像触发器
```
格式：　
	ONBUILD [INSTRUCTION]
示例：
　　ONBUILD ADD . /app/src
　　ONBUILD RUN /usr/local/bin/python-build --dir /app/src
注：
　　NNBUID后面跟指令，当当前的镜像被用做其它镜像的基础镜像，该镜像中的触发器将会被钥触发
```

# 三、制作镜像

- 如果有多个RUN,自上而下依次运行，每次运行都会形成新的层，建议&& 放入一行运行
- 如果有多个CMD,只有最后一个运行
- 如果有多个Entrypoint，只有最后一个运行
- 如果CMD和entrypoint共存，只有entrypoint运行，且最后的CMD会当做entrypoint的参数

镜像制作分为两个阶段
- docker build阶段 基于dockerfile制作镜像 （RUN,用于此阶段的运行命令）
- docker run阶段 基于镜像运行容器 （CMD,基于image run容器时候，需要运行的命令）
- docker build 基于第一阶段的镜像被别人from制作新镜像 （entrypoint 或onbuild 基于镜像重新构建新镜像时候在此阶段运行的命令）

## 3.1 源码编译制作nginx镜像
```
# This my first nginx Dockerfile
# Version 1.0

# Base images 基础镜像
FROM centos

#MAINTAINER 维护者信息
MAINTAINER bertwu 

#ENV 设置环境变量
ENV PATH /usr/local/nginx/sbin:$PATH

#ADD  文件放在当前目录下，拷过去会自动解压
ADD nginx-1.8.0.tar.gz /usr/local/  
ADD epel-release-latest-7.noarch.rpm /usr/local/  

#RUN 执行以下命令 
RUN rpm -ivh /usr/local/epel-release-latest-7.noarch.rpm
RUN yum install -y wget lftp gcc gcc-c++ make openssl-devel pcre-devel pcre && yum clean all
RUN useradd -s /sbin/nologin -M www

#WORKDIR 相当于cd
WORKDIR /usr/local/nginx-1.8.0 

RUN ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_ssl_module --with-pcre && make && make install

RUN echo "daemon off;" >> /etc/nginx.conf

#EXPOSE 映射端口
EXPOSE 80

#CMD 运行以下命令
CMD ["nginx"]
```

## 3.2 制作简单镜像
```
root@ubuntu:~# mkdir myapp
root@ubuntu:~/myapp# vim Dockerfile # 编写Dockerfile
FROM alpine:3.15
LABEL Maintainer="bertwu <bertwu6688@edu.com>"
ADD hosts /etc/hosts

root@ubuntu:~/myapp# vim hosts # 编写文件
root@ubuntu:~/myapp# cat hosts 
127.0.0.1 localhost
127.0.0.1 localhost.localdomain
172.100.100.100 xxx.com


root@ubuntu:~/myapp# docker image build .  # 构建镜像
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM alpine:3.15
 ---> c059bfaa849c
Step 2/3 : LABEL Maintainer="bertwu <bertwu6688@edu.com>"
 ---> Running in 63324216f4ec
Removing intermediate container 63324216f4ec
 ---> bb69e6b659a2
Step 3/3 : ADD hosts /etc/hosts
 ---> 0d6e00e31ce6
Successfully built 0d6e00e31ce6
root@ubuntu:~/myapp# docker image tag 0d6e00e31ce6 myapp:1.0 # 添加标签
root@ubuntu:~/myapp# docker image ls # 查看镜像
REPOSITORY          TAG             IMAGE ID       CREATED              SIZE
myapp               1.0             0d6e00e31ce6   About a minute ago   5.59MB


root@ubuntu:~/myapp# docker image inspect myapp:1.0 | grep -i maint
                "Maintainer": "bertwu <bertwu6688@edu.com>"
                "Maintainer": "bertwu <bertwu6688@edu.com>"
```

## 3.3 自作1.1版本

先编辑apk下载文件
```
root@ubuntu:~/myapp# vim repositories
https://mirrors.aliyun.com/alpine/v3.15/main
https://mirrors.aliyun.com/alpine/v3.15/community

root@ubuntu:~/myapp# cat Dockerfile 
FROM alpine:3.15
LABEL Maintainer="bertwu <bertwu6688@edu.com>"
ADD repositories /etc/apk/repositories # 添加自己指定的repositories
RUN apk update && \
    apk add nginx bash # 安装nginx与bash



root@ubuntu:~/myapp# docker run --name myapp -it --rm myapp:1.1 /bin/bash # 制作镜像
bash-5.1# which nginx
/usr/sbin/nginx
bash-5.1# 
bash-5.1# nginx -v
nginx version: nginx/1.20.2
bash-5.1# 
bash-5.1# cat /etc/apk/repositories 
https://mirrors.aliyun.com/alpine/v3.15/main
https://mirrors.aliyun.com/alpine/v3.15/community
```

## 3.4 构建centos镜像

下面通过编写Dockerfile文件来制作Centos镜像，并在官方镜像的基础上添加vim和net-tools工具。首先在/home/dockfile 目录下新建文件Dockerfile。然后使用上述指令编写该文件。
```
Dockerfile
[root@localhost dockerfile]# cat Dockerfile 
FROM centos:7
MAINTAINER bertwu <1258398543@qq.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim   net-tools
EXPOSE 80
CMD /bin/bash
```

逐行解释该Dockerfile文件的指令：
- FROM centos:7 该image文件继承官方的centos7
- ENV MYPATH /usr/local：设置环境变量MYPATH
- WORKDIR $MYPATH：直接使用上面设置的环境变量，指定/usr/local为工作目录
- RUN yum -y install vim && net-tools：在/usr/local目录下，运行yum -y install vim和yum -y install net-tools命令安装工具，注意安装后的所有依赖和工具都会打包到image文件中
- EXPOSE 80：将容器80端口暴露出来，允许外部连接这个端口
- CMD：指定容器启动的时候运行命令

下面执行build命令生成image文件，如果执行成功，可以通过docker images来查看新生成的镜像文件。
```
[root@localhost dockerfile]# docker build -t mycentos:1.0 . 

[root@localhost dockerfile]# docker images
REPOSITORY    TAG             IMAGE ID       CREATED              SIZE
mycentos      1.0             e0316e2ed3a5   About a minute ago   409MB
```

可以使用 docker history 镜像id 查看镜像构建过程
```
[root@localhost dockerfile]# docker history  e0316e2ed3a5 
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
e0316e2ed3a5   2 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B        
79738577ded0   2 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
f10acdc62daf   2 minutes ago   /bin/sh -c yum -y install vim   net-tools       205MB     
40b0252c02c7   3 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0B        
d38940eb3b75   3 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B        
b23dc50b92b4   3 minutes ago   /bin/sh -c #(nop)  MAINTAINER bertwu <125839…   0B        
eeb6ee3f44bd   2 months ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      2 months ago    /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      2 months ago    /bin/sh -c #(nop) ADD file:b3ebbe8bd304723d4…   204MB    
```

进入容器，看看是否能够执行ifconfig 及vim命令
```
root@localhost dockerfile]# docker run -it mycentos:1.0

[root@3143cb46b8c4 local]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 7  bytes 586 (586.0 B)
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

## 3.5 构建springboot应用
```
cat Dockerfile

FROM openjdk:8-jre # jar包基于jdk ,war包基于tomcat
WORKDIR /app
ADD demo-0.0.1-SNAPSHOT.jar app.jar # 将上下文中 jar包复制到 /app目录下，并且重命名为app.jar
EXPOSE 8081 # 暴露端口
ENTRYPOINT[ "java" , "-jar" ] # 启动应用固定命令
CMD["app.jar"] # 动态传递jar包名
```
