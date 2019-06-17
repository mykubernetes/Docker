
1、通过docker pull 获取官方提供的镜像发现ubuntu镜像比较小  
```
# docker images |grep ubuntu
ubuntu           latest              7698f282e524        4 weeks ago         69.9MB
# docker images |grep centos
centos           latest              9f38484d220f        3 months ago        202MB
```  

2、查看每一层使用了多少空间  
```
# docker history ubuntu
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
7698f282e524        4 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           4 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B                  
<missing>           4 weeks ago         /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
<missing>           4 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   745B                
<missing>           4 weeks ago         /bin/sh -c #(nop) ADD file:1f4fdc61e133d2f90…   69.9MB              
# docker history centos
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9f38484d220f        3 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           3 months ago        /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
<missing>           3 months ago        /bin/sh -c #(nop) ADD file:074f2c974463ab38c…   202MB 
```  

https://github.com/GoogleContainerTools/distroless  
"distroless"镜像只包含应用程序及其运行时依赖项，不包含程序包管理器、shell 以及在标准 Linux 发行版中可以找到的任何其他程序。  
