1、overlay共享数据方式是通过硬连接  
2、overlay2是通过每层的lower文件  

查看存储驱动类型  
```
# docker info |grep -i driver
Storage Driver: overlay2       #使用的overlay2类型
Logging Driver: json-file
Cgroup Driver: cgroupfs

# docker info |grep -i driver
Storage Driver: overlay       #使用的overlay类型
Logging Driver: json-file
Cgroup Driver: cgroupfs

```  

overlay  
```
# docker pull ubuntu

# pull了5个目录包含了5个镜像层，每一层在/var/lib/docker/overlay/下都有自己的目录
# ls -l /var/lib/docker/overlay/
total 0
drwx------. 3 root root 18 Aug 18 03:59 4e543990b3d73415ba5be72315c288318cdc4abcaa94bcc1a4561cea34a83cc2
drwx------. 3 root root 18 Aug 18 03:59 a6591a40349dbfef3a151e34d40839a59b3734bc89296a89f391936a89947aef
drwx------. 3 root root 18 Aug 18 03:52 bed08bfdf435a55100e67638793d6c8a34c9541601e9c424c12f27eb213797dc
drwx------. 3 root root 18 Aug 18 03:59 c756cffbbf05b25b90c528f9672ac74673edd9d3d40b10f331f941a0e087db69
drwx------. 3 root root 18 Aug 18 04:00 dff5331f56789b84325b3028c503bb3d85d613886d24ff3dd6906112aaf0df5b

# 镜像层目录中，共享的数据使用的是硬链接，他们的inode号相同。这样做有效地利用了磁盘。
# ls -i /var/lib/docker/overlay/4e543990b3d73415ba5be72315c288318cdc4abcaa94bcc1a4561cea34a83cc2/root/bin/ls
51111560 /var/lib/docker/overlay/4e543990b3d73415ba5be72315c288318cdc4abcaa94bcc1a4561cea34a83cc2/root/bin/ls
# ls -i /var/lib/docker/overlay/a6591a40349dbfef3a151e34d40839a59b3734bc89296a89f391936a89947aef/root/bin/ls
51111560 /var/lib/docker/overlay/a6591a40349dbfef3a151e34d40839a59b3734bc89296a89f391936a89947aef/root/bin/ls


```  

overlay2  
```
# docker pull ubuntu

docker pull ubuntu下载了包含5层的镜像，可以看到在/var/lib/docker/overlay2中，有6个目录。
# ls -l /var/lib/docker/overlay2/
total 0
drwx------. 3 root root     30 Aug 18 03:56 13fa5d88d8f0cf8eda921035dd97ea76e0c453b26444de561ffd41d8feb3dfbb
drwx------. 4 root root     55 Aug 18 03:56 920ddf7826d40de7c94d2fc1a76e69ee7a3f13464d8eeb4923c1a14b04ace538
brw-------. 1 root root 253, 0 Aug 18 03:38 backingFsBlockDev
drwx------. 4 root root     55 Aug 18 03:56 f52f9b29acf403e41a99f092622695db368efc63f597f0d1b4e255518d4cb7c1
drwx------. 4 root root     55 Aug 18 03:56 fb855cabed1d6805247bcbb295707e152c1d571c07ff8241e16ef497fcedec60
drwx------. 2 root root    142 Aug 18 03:57 l

# l目录包含了很多软连接，使用短名称指向了其他层。短名称用于避免mount参数时达到页面大小的限制。
# ls -l /var/lib/docker/overlay2/l
total 0
lrwxrwxrwx. 1 root root 72 Aug 18 04:03 CPAT6TA7HDLDV2K2AUZ7PUZ7E4 -> ../cb2d502b283c2e6f295c633672412d4028d901c105ed7b09ae589201a7484996/diff
lrwxrwxrwx. 1 root root 72 Aug 18 04:03 MRI4KAFT6CVQCWIKJPBN3IJPPO -> ../f1ebf4b5fa481a5fed80c953b17fcfef26e99494f03f78cc5226cb6dee718c71/diff
lrwxrwxrwx. 1 root root 72 Aug 18 04:03 TVV7P55MQ3KWPKVDOMGCAFWN4F -> ../59cb718bbc3ade78aa47a91cd2d591ac998126773be5484e2bd57ca0d0af2643/diff
lrwxrwxrwx. 1 root root 72 Aug 18 04:03 ZWZ7HWFUGJXW6XXWYCGWQR5CV7 -> ../1c69087ac603c6d078a5d44de6e134feb6bc9e42406de887a145e865b32fe691/diff


在最低层中，有个link文件，包含了前面提到的这个层对应的短名称；还有个diff目录，包含了这个镜像的内容。
# ls /var/lib/docker/overlay2/cb2d502b283c2e6f295c633672412d4028d901c105ed7b09ae589201a7484996/
diff  link

# cat /var/lib/docker/overlay2/cb2d502b283c2e6f295c633672412d4028d901c105ed7b09ae589201a7484996/link 
CPAT6TA7HDLDV2K2AUZ7PUZ7E4

# ls /var/lib/docker/overlay2/cb2d502b283c2e6f295c633672412d4028d901c105ed7b09ae589201a7484996/diff/
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var


第二底层中，lower文件指出了该层的组成。
# ls /var/lib/docker/overlay2/1c69087ac603c6d078a5d44de6e134feb6bc9e42406de887a145e865b32fe691/
diff  link  lower  work

# cat /var/lib/docker/overlay2/1c69087ac603c6d078a5d44de6e134feb6bc9e42406de887a145e865b32fe691/lower 
l/TVV7P55MQ3KWPKVDOMGCAFWN4F:l/CPAT6TA7HDLDV2K2AUZ7PUZ7E4

# ls /var/lib/docker/overlay2/1c69087ac603c6d078a5d44de6e134feb6bc9e42406de887a145e865b32fe691/diff/
etc  sbin  usr  var
```  

挂载方式  
overlay: 只挂载一层,其他层通过最高层通过硬连接形式共享(增加了磁盘inode的负担)  
overlay2: 逐层挂载  

