概述  
overlay存储驱动层数过多时会导致文件链接数过多可能会耗尽inode

内核要求  
---
需要一个高版本的内核推荐4.9以上，我们用的是4.14，如果使用低内核可能一些FROM别的基础镜像就跑不了  

使用xfs文件系统  
---
不使用xfs就无法做到给每个容器限制10G的大小，就可能出现一个容器的误操作导致把机器盘全占完，我们使用了lvm去弄个分区出来做xfs文件系统，当然你也可以不用lvm  
```
if which lvs &>/dev/null; then
  echo ""; echo -e "Remove last docker lv and mount ......"
  lvremove k8s/docker -y
  lvcreate -y -n docker k8s -L 100G
  mkfs.xfs -n ftype=1 -f /dev/mapper/k8s-docker
  mkdir -p /var/lib/docker
  mount -o pquota,uqnoenforce /dev/mapper/k8s-docker /var/lib/docker
  echo -e "/dev/mapper/k8s-docker                                  /var/lib/docker         xfs     defaults,pquota        0 0" >> /etc/fstab
fi
```  

配置使用overlay2  
---
```
# cat /etc/docker/daemon.json
{
  "storage-opts": [
    "overlay2.override_kernel_check=true",
    "overlay2.size=10G"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m"
  }
}
```  

重启  
```
systemctl daemon-reload 
systemctl restart docker
```  
这样就可以把每个容器磁盘大小限制在10G了  
