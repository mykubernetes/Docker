1、overlay共享数据方式是通过硬连接  
2、overlay2是通过每层的 lower文件  

overlay  
```
root@no1:/var/lib/docker/overlay2# ll a6350774f0c5a4c89f850252180930a48ae28ca37c33ac3a2ba39585bb3c2c42/
total 8.0K
-rw-r--r--  1 root root   26 2017-12-13 22:03:57.063074124 -0800 link
drwxr-xr-x 21 root root 4.0K 2017-12-13 22:03:59.027035468 -0800 diff/ ##挂载点
root@no1:/var/lib/docker/overlay2# ll 43a794c7d4fc61414cceedb2914026f09c9228397a885fb2d040278d6a1b5856/
total 20K
drwx------ 2 root root 4.0K 2017-12-13 22:04:00.471007047 -0800 work/
drwx------ 2 root root 4.0K 2017-12-13 22:04:00.471007047 -0800 merged/
-rw-r--r-- 1 root root   28 2017-12-13 22:04:00.471007047 -0800 lower
-rw-r--r-- 1 root root   26 2017-12-13 22:04:00.471007047 -0800 link
drwxr-xr-x 6 root root 4.0K 2017-12-13 22:04:00.503006417 -0800 diff/

root@no1:/var/lib/docker/overlay2# ll 33a4e0217cb0342f024ee2f093ab7188433b2b231c75999848bd2d36eb501255/
total 20K
drwx------ 2 root root 4.0K 2017-12-13 22:04:00.511006259 -0800 work/
drwx------ 2 root root 4.0K 2017-12-13 22:04:00.511006259 -0800 merged/
-rw-r--r-- 1 root root   57 2017-12-13 22:04:00.511006259 -0800 lower
-rw-r--r-- 1 root root   26 2017-12-13 22:04:00.511006259 -0800 link
drwxr-xr-x 3 root root 4.0K 2017-12-13 22:04:00.531005866 -0800 diff/
```  

overlay2  
```
root@no1:/var/lib/docker/overlay2# 
root@no1:/var/lib/docker/overlay2# cat a6350774f0c5a4c89f850252180930a48ae28ca37c33ac3a2ba39585bb3c2c42/l^C
root@no1:/var/lib/docker/overlay2# cat 43a794c7d4fc61414cceedb2914026f09c9228397a885fb2d040278d6a1b5856/lower 
l/UZH33GFG3GMRJCYLAIHJQKEUIN
root@no1:/var/lib/docker/overlay2# cat 33a4e0217cb0342f024ee2f093ab7188433b2b231c75999848bd2d36eb501255/lower 
l/QQIYMV3GOYXUTDN6GAAKSM7ZFL:l/UZH33GFG3GMRJCYLAIHJQKEUIN
```  

挂载方式  
---
```  mount|grep overlay ```  

overlay: 只挂载一层,其他层通过最高层通过硬连接形式共享(增加了磁盘inode的负担)  
```
/var/lib/docker/overlay/ae6ca8bdaf74720c26b4d780d0c7837e487505c410efb5b9d891bb78796e8e0f/merged type overlay
(rw,relatime,
lowerdir=/var/lib/docker/overlay/632707d3098b737da98ada134fb2cdb8c18c6492dabc9fabbc08e664afc23b8e/root,
upperdir=/var/lib/docker/overlay/ae6ca8bdaf74720c26b4d780d0c7837e487505c410efb5b9d891bb78796e8e0f/upper,
workdir=/var/lib/docker/overlay/ae6ca8bdaf74720c26b4d780d0c7837e487505c410efb5b9d891bb78796e8e0f/work)
```  


overlay2: 逐层挂载  
```
(rw,relatime,
lowerdir=/var/lib/docker/overlay2/l/AQLAUEDWASUZFK6WMTNMV67AEF:/var/lib/docker/overlay2/l/KWJWIYWDTPZCGVRASSRIVREKMN:/var/lib/docker/overlay2/l/ZYQCFR4K5ZI5GDFJXHINZJTNF2:/var/lib/docker/overlay2/l/EY6ZNSFU3IGYHG3ALKBRVDWMX2:/var/lib/docker/overlay2/l/QQIYMV3GOYXUTDN6GAAKSM7ZFL:/var/lib/docker/overlay2/l/UZH33GFG3GMRJCYLAIHJQKEUIN,
upperdir=/var/lib/docker/overlay2/2d3b5711d26366b06283c1e8632d5065b9b6ba2e027b7cdd351a2d89b3810dfd/diff,
workdir=/var/lib/docker/overlay2/2d3b5711d26366b06283c1e8632d5065b9b6ba2e027b7cdd351a2d89b3810dfd/work)
```  
