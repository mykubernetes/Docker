Linux namespace是一种由内核提供的进程访问控制机制，与Cgroup的资源配额限制不同，namespace的主要功能是实现进程隔离，为进程提供独立的运行环境  

Linux内核实现了以下七种类型的namespace，其中Cgroup namespace仅在4.6或更高版本的内核中实现  

| 名称 | 内核版本 | 隔离内容 | 
| :---: | :--------: | :-------: |
| UTS | 3.0 | 主机名和域名 |
| MNT | 3.8 | 挂载点(文件系统) |
| IPC| 3.0 | 信号量、消息队列和共享内存 |
| PID | 3.8 | 进程编号 |
| NET | 3.0 | 网络设备、网络栈、端口等 |
| USER | 3.8 | 用户和用户组 |
| Cgroup | 4.6 | Cgroup root directory |

查看当前进程的namespace  
```
# ll /proc/$$/ns
total 0
lrwxrwxrwx. 1 root root 0 Feb 19 08:58 ipc -> ipc:[4026531839]
lrwxrwxrwx. 1 root root 0 Feb 19 08:58 mnt -> mnt:[4026531840]
lrwxrwxrwx. 1 root root 0 Feb 19 08:58 net -> net:[4026531956]
lrwxrwxrwx. 1 root root 0 Feb 19 08:58 pid -> pid:[4026531836]
lrwxrwxrwx. 1 root root 0 Feb 19 08:58 user -> user:[4026531837]
lrwxrwxrwx. 1 root root 0 Feb 19 08:58 uts -> uts:[4026531838]
```  
- 我的操作系统内核版本是3.10，仅支持六种类型的namespace
- 上述六个文件分别代表对应namespace的文件描述符，中括号内对应其inode number
- 如果两个进程的inode number一致，说明这两个进程属于同一个namespace


本文主要介绍各个namespace的主要功能，不涉及实现方式与数据结构。以下是这篇文章会用到的命令行工具介绍  
- readlink：查看namespace的inode number
- unshare：在新创建的namespace中运行指定的程序（新创建的namespace，新进程）
- nsenter：在指定的进程所在的namespace中运行新的程序（已存在的namespace，新进程）
- pstree：显示进程树

本文的实验环境  
```
# cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
# uname -a
Linux localhost.localdomain 3.10.0-862.el7.x86_64 #1 SMP Fri Apr 20 16:44:24 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```  

1、uts namespace 
---
UTS（UNIX Time-sharing System），用来隔离hostname和NIS domain name，每个uts namespace中运行的进程拥有独立的hostname和NIS，互不影响。  

- terminal-1  
```
### 查看当前状态的进程树 ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(653)
           ├─anacron(1490)
           ├─auditd(614)───{auditd}(615)
           ├─crond(649)
           ├─dbus-daemon(638)───{dbus-daemon}(643)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(489)
           ├─master(1261)─┬─pickup(1267)
           │              └─qmgr(1268)
           ├─polkitd(644)─┬─{polkitd}(655)
           │              ├─{polkitd}(656)
           │              ├─{polkitd}(657)
           │              ├─{polkitd}(660)
           │              └─{polkitd}(661)
           ├─rsyslogd(968)─┬─{rsyslogd}(973)
           │               └─{rsyslogd}(982)
           ├─sshd(970)───sshd(1421)───bash(1425)───pstree(1709)    # 主要关注进程，因为是通过ssh连接虚拟机，所以会有sshd父进程
           ├─systemd-journal(462)
           ├─systemd-logind(646)
           ├─systemd-udevd(498)
           └─tuned(967)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)

### 查看当前进程的uts，$$表示当前进程的PID，这里等同于1425 ###
# readlink /proc/$$/ns/uts /proc/1425/ns/uts
uts:[4026531838]
uts:[4026531838]

### 查看当前进程的hostname ###
# hostname
localhost.localdomain

### 创建新的uts namespace并运行bash ###
# unshare --uts bash

### 查看当前状态进程树 ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(653)
           ├─auditd(614)───{auditd}(615)
           ├─crond(649)
           ├─dbus-daemon(638)───{dbus-daemon}(643)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(489)
           ├─master(1261)─┬─pickup(1267)
           │              └─qmgr(1268)
           ├─polkitd(644)─┬─{polkitd}(655)
           │              ├─{polkitd}(656)
           │              ├─{polkitd}(657)
           │              ├─{polkitd}(660)
           │              └─{polkitd}(661)
           ├─rsyslogd(968)─┬─{rsyslogd}(973)
           │               └─{rsyslogd}(982)
           ├─sshd(970)───sshd(1421)───bash(1425)───bash(1717)───pstree(1726)   # 比上面多出一个bash(1717)
           ├─systemd-journal(462)
           ├─systemd-logind(646)
           ├─systemd-udevd(498)
           └─tuned(967)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)

### 1717与1425的uts inode number不同，说明这两个线程处在不同的namespace ###
# readlink /proc/$$/ns/uts /proc/1717/ns/uts
uts:[4026532418]
uts:[4026532418]

### 设置当前进程的hostname ###
# hostname namespace-01

### 新的hostname不会立即生效，执行exec bash，使用新bash取代当前bash，使hostname生效 ###
# exec bash

### 查看hostname ###
# hostname
namespace-01
```  

- terminal-2  
```
### 在另一个终端中查看进程树 ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(653)
           ├─auditd(614)───{auditd}(615)
           ├─crond(649)
           ├─dbus-daemon(638)───{dbus-daemon}(643)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(489)
           ├─master(1261)─┬─pickup(1267)
           │              └─qmgr(1268)
           ├─polkitd(644)─┬─{polkitd}(655)
           │              ├─{polkitd}(656)
           │              ├─{polkitd}(657)
           │              ├─{polkitd}(660)
           │              └─{polkitd}(661)
           ├─rsyslogd(968)─┬─{rsyslogd}(973)
           │               └─{rsyslogd}(982)
           ├─sshd(970)─┬─sshd(1421)───bash(1425)───bash(1717)      # terminal-1对应的进程信息
           │           └─sshd(1740)───bash(1744)───pstree(1759)    # 当前进程对应的信息
           ├─systemd-journal(462)
           ├─systemd-logind(646)
           ├─systemd-udevd(498)
           └─tuned(967)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)

### 查看当前进程的uts ###
# readlink /proc/$$/ns/uts /proc/1744/ns/uts
uts:[4026531838]
uts:[4026531838]

### 查看当前进程的hostname ###
# hostname
localhost.localdomain

### 在termenal-2中创建新的uts namespace ###
# unshare --uts bash
### 查看进程树 ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(653)
           ├─auditd(614)───{auditd}(615)
           ├─crond(649)
           ├─dbus-daemon(638)───{dbus-daemon}(643)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(489)
           ├─master(1261)─┬─pickup(1267)
           │              └─qmgr(1268)
           ├─polkitd(644)─┬─{polkitd}(655)
           │              ├─{polkitd}(656)
           │              ├─{polkitd}(657)
           │              ├─{polkitd}(660)
           │              └─{polkitd}(661)
           ├─rsyslogd(968)─┬─{rsyslogd}(973)
           │               └─{rsyslogd}(982)
           ├─sshd(970)─┬─sshd(1421)───bash(1425)───bash(1717)
           │           └─sshd(1740)───bash(1744)───bash(1763)───pstree(1772)    # 新进程1763
           ├─systemd-journal(462)
           ├─systemd-logind(646)
           ├─systemd-udevd(498)
           └─tuned(967)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)

### 查看当前进程的uts ###
# readlink /proc/$$/ns/uts /proc/1763/ns/uts 
uts:[4026532499]
uts:[4026532499]
### 设置hostname ###
# hostname namespace-02
# exec bash
### 查看hostname ###
# hostname
namespace-02
```  

目前的状态大概是这个样子  
```
                            +-----------------------------------+
                            |  hostname: localhost.localdomain  |
                        +---+  uts: 4026531838                  +---+
                        |   |  pid: 1425, 1744                  |   |
                        |   +-----------------------------------+   |
                        v                                           v
     +-----------------------------------+        +-----------------------------------+
     |  hostname: namespace-01           |        |  hostname: namespace-02           |
     |  uts: 4026532418                  |        |  uts: 4026532499                  |
     |  pid: 1717                        |        |  pid: 1763                        |
     +-----------------------------------+        +-----------------------------------+
```  
三个namespace之间设置的hostname互不影响，所有进程退出之后，namespace将被自动删除，inode number回收利用  

- terminal-1  
```
### 退出namespace-01对应的bash ###
# exit
exit

### 查看当前状态的进程树 ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(653)
           ├─auditd(614)───{auditd}(615)
           ├─crond(649)
           ├─dbus-daemon(638)───{dbus-daemon}(643)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(489)
           ├─master(1261)─┬─pickup(1785)
           │              └─qmgr(1268)
           ├─polkitd(644)─┬─{polkitd}(655)
           │              ├─{polkitd}(656)
           │              ├─{polkitd}(657)
           │              ├─{polkitd}(660)
           │              └─{polkitd}(661)
           ├─rsyslogd(968)─┬─{rsyslogd}(973)
           │               └─{rsyslogd}(982)
           ├─sshd(970)─┬─sshd(1421)───bash(1425)───pstree(1792)    # 当前进程1425，1717进程已退出
           │           └─sshd(1740)───bash(1744)───bash(1763)
           ├─systemd-journal(462)
           ├─systemd-logind(646)
           ├─systemd-udevd(498)
           └─tuned(967)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)
                        
### 查看hostname ###
# hostname
localhost.localdomain

### 查看当前进程和1717进程的uts，仅显示localhost.localdomain的uts，且1717对应目录以被删除 ###
# readlink /proc/$$/ns/uts /proc/1717/ns/uts
uts:[4026531838]
```  
Uts namespace不存在嵌套和上下级关系，namespace本身并不是一个集合或者容器，而是进程的属性，属性值相同的进程同属于一个namespace～  

- terminal-1  
```
### 在namespace-02中执行新的bash，1763为目标的进程pid，--uts表示与1763进程使用相同的uts namespace ###
# nsenter --target 1763 --uts bash
### 新bash对应的uts namespace inode number与namespace-02相同 ###
# readlink /proc/$$/ns/uts
uts:[4026532499]
# hostname
namespace-02
```  


2、mnt namespace
---
Mount namespace的作用是隔离mount point，每个mnt namespace内的文件结构可以单独修改，互不影响。  
```
### 当前进程所在的mnt namespace的所有挂载点信息记录在以下三个文件中 ###
# ll /proc/$$/mount*
-r--r--r--. 1 root root 0 Feb 20 12:24 /proc/1425/mountinfo
-r--r--r--. 1 root root 0 Feb 20 12:24 /proc/1425/mounts
-r--------. 1 root root 0 Feb 20 12:24 /proc/1425/mountstats

### 为接下来的操作准备两个目录，每个目录下一个文件 ###
# mkdir namespace01 && touch namespace01/namespace01.txt
# mkdir namespace02 && touch namespace02/namespace02.txt
```  

- terminal-1  
```
### 创建新的mount namespace和uts namespace并运行bash ###
# unshare --mount --uts bash

### 设置hostname以便于观察 ###
# hostname namespace-01 && exec bash

### 查看对应namespace的inode number ###
# readlink /proc/$$/ns/{uts,mnt}
uts:[4026532499]
mnt:[4026532418]

### 将上面创建的namespace01目录挂在到/mnt ###
# mount --bind namespace01/ /mnt/

### 查看/mnt目录下的文件 ###
# ll /mnt/
total 0
-rw-r--r--. 1 root root 0 Feb 20 12:46 namespace01.txt
```  


- terminal-2
```
### 在另一个终端上创建新的mount namespace和uts namespace，执行bash ###
# unshare --mount --uts bash

### 设置对应的hostname以便于区别 ###
# hostname namespace-02 && exec bash

### 查看对应的inode number，确保与terminal-1中的进程不在同一个namespace ###
# readlink /proc/$$/ns/{uts,mnt}
uts:[4026532501]
mnt:[4026532500]

### 挂载namespace02之前，首先查看/mnt目录，证实namespace02无法看到与namespace01相同的内容 ###
# ll /mnt/
total 0

### 将namespace02目录挂在到/mnt ###
#  mount --bind namespace02/ /mnt/

### 查看/mnt目录下的文件 ###
# ll /mnt/
total 0
-rw-r--r--. 1 root root 0 Feb 20 12:46 namespace02.txt
```  
当新的mount namespace创建时，会从创建进程所在的namespace复制挂载点信息，所以namespace01和namespace02都可以看到相同的文件视图  

- terminal-1  
```
### 在terminal-1下继续创建namespace03 ###
# unshare --mount --uts bash

### 修改hostname以便于区分 ###
# hostname namespace-03 && exec bash

### 查看inode number，与上述namespace01和namespace02均不同 ###
# readlink /proc/$$/ns/{uts,mnt}
uts:[4026532503]
mnt:[4026532502]

### 查看/mnt目录，视图与namespace01相同，无法看到namespace02的内容 ###
# ll /mnt/
total 0
-rw-r--r--. 1 root root 0 Feb 20 12:46 namespace01.txt
```  
关于shared subtrees：由上可知，在同一个namespace中挂载新设备时，其他namespace不受影响。但是，如果需要在所有namespace中挂着相同的设备时，就需要在每个namespace中执行相同的mount操作，非常麻烦，shared subtrees解决了这个问题。关于shared subtrees的更多内容https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt  



