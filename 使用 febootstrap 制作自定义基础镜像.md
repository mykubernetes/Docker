febootstrap
===
febootstrap 是一个方便我们制作原生 OS 基础镜像的工具，例如 Centos、Ubuntu 等操作系统，同时还可以指定安装一些特定软件到环境镜像里面，可以使我们更方便的了解和控制基础镜像的构成，最后，通过该基础镜像在扩展成应用镜像，最终来部署服务。  

febootstrap 安装  
---

Centos6 操作系统安装  
因为在 Centos6 系统，默认源中存在该包，可以直接使用 yum 安装。  
``` # yum install febootstrap -y ```  

Centos7 操作系统安装  
由于在 Centos7 系统中，默认源中不存在该包，无法用 yum 直接安装  
```
# 下载 rmp 包
wget http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/febootstrap-3.21-4.el6.x86_64.rpm
wget http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/fakechroot-2.9-24.5.el6_1.1.x86_64.rpm
wget https://www.dwhd.org/wp-content/uploads/2016/06/febootstrap-supermin-helper-3.21-4.el6_.x86_64.rpm
wget http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/fakechroot-libs-2.9-24.5.el6_1.1.x86_64.rpm
wget http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/fakeroot-1.12.2-22.2.el6.x86_64.rpm
wget http://rpmfind.net/linux/centos/6.10/os/x86_64/Packages/fakeroot-libs-1.12.2-22.2.el6.x86_64.rpm

# 安装 rpm 包
rpm -ivh *.rpm
warning: fakechroot-2.9-24.5.el6_1.1.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:febootstrap-supermin-helper-3.21-################################# [ 17%]
   2:fakeroot-libs-1.12.2-22.2.el6    ################################# [ 33%]
   3:fakeroot-1.12.2-22.2.el6         ################################# [ 50%]
   4:fakechroot-libs-2.9-24.5.el6_1.1 ################################# [ 67%]
   5:fakechroot-2.9-24.5.el6_1.1      ################################# [ 83%]
   6:febootstrap-3.21-4.el6           ################################# [100%]
```  
安装完毕，本地就可以使用 Febootstrap 命令制作原生操作系统基础镜像了。  

制作自定义基础镜像  
---

制作镜像说明  
``` febootstrap -i bash -i wget -i apt-get centos72 centos72-image http://mirrors.163.com/centos/7.2.1511/os/x86_64/ ```  
- -i 需要安装的 package，例如这里安装 bash、wget、apt-get。  
- centos72 指定 Image 操作系统版本  
- centos72-image 生成的镜像目录，会在当前目录下生成 centos72-image 目录。  
- http://mirrors.163.com/centos/7.2.1511/os/x86_64/ 镜像 OS 源地址，可以修改为其它源地址  

这里以Centos7.6操作系统制作基于原生Centos7.6 OS操作系统，并添加一些必要的软件工具  
```
# febootstrap -i bash -i wget -i yum -i telnet -i iputils -i iproute -i vim -i gzip -i tar centos76 centos76-extend http://mirrors.aliyun.com/centos/7/os/x86_64/

# 下载安装完毕后目录（相当于本地一个完整的 centos7.6 的系统目录）
# ll centos76-extend/
total 8
lrwxrwxrwx.  1 root root    7 Jun  9 15:50 bin -> usr/bin
dr-xr-xr-x.  2 root root    6 Apr 11  2018 boot
drwxr-xr-x.  5 root root  186 Apr 11  2018 dev
drwxr-xr-x. 49 root root 4096 Jun  9 15:51 etc
drwxr-xr-x.  2 root root    6 Apr 11  2018 home
lrwxrwxrwx.  1 root root    7 Jun  9 15:50 lib -> usr/lib
lrwxrwxrwx.  1 root root    9 Jun  9 15:50 lib64 -> usr/lib64
drwxr-xr-x.  2 root root    6 Apr 11  2018 media
drwxr-xr-x.  2 root root    6 Apr 11  2018 mnt
drwxr-xr-x.  2 root root    6 Apr 11  2018 opt
dr-xr-xr-x.  2 root root    6 Apr 11  2018 proc
dr-xr-x---.  2 root root    6 Apr 11  2018 root
drwxr-xr-x. 12 root root  149 Jun  9 15:51 run
lrwxrwxrwx.  1 root root    8 Jun  9 15:50 sbin -> usr/sbin
drwxr-xr-x.  2 root root    6 Apr 11  2018 srv
dr-xr-xr-x.  2 root root    6 Apr 11  2018 sys
drwxrwxrwt.  7 root root   93 Jun  9 15:51 tmp
drwxr-xr-x. 13 root root  155 Jun  9 15:50 usr
drwxr-xr-x. 18 root root  238 Jun  9 15:50 var

# 将当前目录打包并导入到镜像
$ tar -c .|docker import - centos7.6:base-extend
sha256:495a4b1d9b504cb23b1ae89b5d5187eebd6163639384f5cd2605b620c488c3be

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos7.6           base-extend         495a4b1d9b50        21 seconds ago      442MB
```  
