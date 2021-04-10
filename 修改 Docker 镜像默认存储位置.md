使用软链接
===

默认情况下 Docker 容器的存放位置在 /var/lib/docker 目录下面，可以通过下面命令查看具体位置。

1、查看默认存放位置
```
$ sudo docker info | grep "Docker Root Dir"
```

2、 停掉Docker服务
```
$ systemctl stop docker
```

3、移动原有的内容  
移动整个 /var/lib/docker 目录到空间不较大的目的路径。这时候启动 Docker 时发现存储目录依旧是 /var/lib/docker 目录，但是实际上是存储在数据盘 /data/docker 上。
```
$ mv /var/lib/docker /data/docker
```
4、进行链接
```
$ ln -sf /data/docker /var/lib/docker
```

指定容器启动参数
===
在配置文件中指定容器启动的参数 --graph=/var/lib/docker 来指定镜像和容器存放路径。Docker 的配置文件可以设置大部分的后台进程参数，在各个操作系统中的存放位置不一致。在 Ubuntu 中的位置是 /etc/default/docker 文件，在 CentOS 中的位置是 /etc/sysconfig/docker 文件。

1、CentOS6 因为Ubuntu默认开启了selinux机制
```
OPTIONS=--graph="/data/docker" --selinux-enabled -H fd://
```

2、CentOS7 修改docker.service文件，使用-g参数指定存储位置
```
$ vi /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --graph /new-path/docker
```

3、Ubuntu 因为Ubuntu默认没开启selinux机制
```
OPTIONS=--graph="/data/docker" -H fd://
```
重新启动之后，Docker 的路径就改成 /data/docker 了。

4、重新reload配置文件
```
$ sudo systemctl daemon-reload
```

5、重启docker服务
```
$ sudo systemctl restart docker.service
```
如果 Docker 的版本是 1.12 或以上的，可以修改或新建 daemon.json 文件。修改后会立即生效，不需重启 Docker 服务。

6、修改配置文件
```
$ vim /etc/docker/daemon.json
{
    "registry-mirrors":
        ["http://www.daocloud.io"],
    "graph": "/new-path/docker"
}
```

System 下创建配置文件
===
在 /etc/systemd/system/docker.service.d 目录下创建一个 Drop-In 文件 docker.conf，默认 docker.service.d 文件夹不存在，必须先创建它。创建 Drop-In 文件的原因，是我们希望 Docker服务使用 docker.conf 文件中提到的特定参数，将默认服务所使用的位于 /lib/systemd/system/docker.service 文件中的参数进行覆盖。

1、定义新的存储位置
```
$ sudo vi /etc/systemd/system/docker.service.d/docker.conf
[Service]
ExecStart=/usr/bin/dockerd --graph="/data/docker" --storage-driver=devicemapper
```
保存并退出 vim 编辑器 /data/docker 就是新的存储位置，而 devicemapper 是当前 Docker 所使用的存储驱动。如果你的存储驱动有所不同，请输入之前第一步查看并记下的值。现在，你可以重新加载服务守护程序，并启动 Docker 服务了，这将改变新的镜像和容器的存储位置。为了确认一切顺利，运行 docker info 命令检查 Docker 的根目录。

2、重新reload配置文件
```
$ sudo systemctl daemon-reload
```

3、重启docker服务
```
$ sudo systemctl start docker
```
