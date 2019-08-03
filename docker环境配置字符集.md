1、进入docker镜像查看字符集  
```
# docker run -it --rm centos sh
sh-4.2# echo $LANG

sh-4.2# locale
LANG=
LC_CTYPE="POSIX"
LC_NUMERIC="POSIX"
LC_TIME="POSIX"
LC_COLLATE="POSIX"
LC_MONETARY="POSIX"
LC_MESSAGES="POSIX"
LC_PAPER="POSIX"
LC_NAME="POSIX"
LC_ADDRESS="POSIX"
LC_TELEPHONE="POSIX"
LC_MEASUREMENT="POSIX"
LC_IDENTIFICATION="POSIX"
LC_ALL=

sh-4.2# locale -a
C
POSIX
en_US.utf8
```  

2、以配置zh_CN.GB18030字符集为例  
```
yum install -y kde-l10n-Chinese
yum reinstall -y glibc-common
localedef -c -f GB18030 -i zh_CN zh_CN.GB18030

#验证成功加载中文语言包zh_CN.gb18030
# locale -a
C
POSIX
en_US.utf8
zh_CN.gb18030
```  

3、修改字符集配置  
```
$ cat /etc/locale.conf 
LANG="en_US.UTF-8"
$ echo 'LANG="zh_CN.GB18030"' > /etc/locale.conf && source /etc/locale.conf
$ echo "export LC_ALL=zh_CN.GB18030" >> /etc/profile && source /etc/profile

#验证配置生效
$ echo $LANG
zh_CN.GB18030
```  

4、Dockerfile示例：  
docker容器环境需要基于dockerfile制作对应字符集镜像，追加以下内容到自定义dockerfile中：  
```
# cat Dockerfile
FROM centos
LABEL Maintainer dockerhub.com
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
	&& yum -y install kde-l10n-Chinese \
	&& yum -y reinstall glibc-common \
	&& localedef -c -f GB18030 -i zh_CN zh_CN.GB18030 \
	&& echo 'LANG="zh_CN.GB18030"' > /etc/locale.conf \
	&& source /etc/locale.conf \
	&& yum clean all 
ENV LANG=zh_CN.GB18030 \
    LC_ALL=zh_CN.GB18030
```  
