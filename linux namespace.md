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


3、ipc namespace
---
IPC（inter-process communication）namespace，主要用来隔离svipc - System V inter-process communication mechanisms和POSIX。IPC和POSIX都是用来实现进程之间的数据交换的。ipc mechanisms包括：message queues、semaphore sets、shared memory segments。POSIX与IPC的message queues功能类似，但实现方式不同。

- terminal-1
```
### 创建新的ipc namespace和uts namespace并运行bash ###
# unshare --uts --ipc bash

### 修改hostname以便于区分 ###
# hostname namespace-01 && exec bash

### 查看inode number，确保新namespace创建成功 ###
# readlink /proc/$$/ns/{uts,ipc}
uts:[4026532499]
ipc:[4026532500]

### 查看所有IPC对象信息 ###
# ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      

------ Semaphore Arrays --------
key        semid      owner      perms      nsems 

### 创建一个MQ ###
# ipcmk -Q
Message queue id: 0

### 查看MQ对象，-q参数表示仅显示MQ信息 ###
# ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x2e011e55 0          root       644        0            0 

### 显示当前进程pid ###
# echo $$
1459
```  

- terminal-2  
```
### 在另一个终端中创建新的ipc namespace和uts namespace ###
# unshare --uts --ipc bash

### 修改hostname便于区分 ###
# hostname namespace-02 && exec bash

### 确保不在同一个namespace ###
# readlink /proc/$$/ns/{uts,ipc}
uts:[4026532501]
ipc:[4026532502]

### 查看当前namespace的ipc对象，无法看到msqid为0的MQ ###
# ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      

------ Semaphore Arrays --------
key        semid      owner      perms      nsems     
```  

- terminal-3  
```
### 在namespace01中执行新的bash ###
# nsenter --target 1459 --uts --ipc bash

### 查看inode number，确保与terminal-1中一致 ###
# readlink /proc/$$/ns/{uts,ipc}
uts:[4026532499]
ipc:[4026532500]

### 查看ipc对象信息 ###
# ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x2e011e55 0          root       644        0            0           

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
```  
- ipc namespace与uts namespace都不存在嵌套或者上下级关系
- 这里使用了三个终端和两个ipc namespace，为便于观察，分别设置了两个hostname



4、pid namespace
---

pid namespace是用来隔离进程号的，也就是说，同一个系统中，在不同的namespace下可以存在相同pid的两个进程。比如两个pid都为1的两个进程。  

在Linux系统中，pid为大于等于1的整数，且不能重复，进程退出后，pid将回收利用。pid为1的进程是内核启动的第一个进程，通常为init或者systemd。后续启动的其他进程都是该进程的子进程，当init或者systemd进程退出时，系统也就退出了。  
```
### 查看当前系统状态的进程树信息，其实进程为systemd(1)，括号内为pid ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(657)
           ├─auditd(614)───{auditd}(615)
           ├─crond(653)
           ├─dbus-daemon(641)───{dbus-daemon}(646)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(490)
           ├─master(1259)─┬─pickup(1265)
           │              └─qmgr(1266)
           ├─polkitd(639)─┬─{polkitd}(648)
           │              ├─{polkitd}(649)
           │              ├─{polkitd}(652)
           │              ├─{polkitd}(658)
           │              └─{polkitd}(661)
           ├─rsyslogd(970)─┬─{rsyslogd}(974)
           │               └─{rsyslogd}(975)
           ├─sshd(969)───sshd(1421)───bash(1425)───pstree(1630)
           ├─systemd-journal(462)
           ├─systemd-logind(640)
           ├─systemd-udevd(498)
           └─tuned(968)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)

```  
在每个pid namespace中都存在着一颗类似的进程树，都有一个pid为1的进程。当某个进程（pid != 1）退出时，内核会指定pid为1的进程成为其子进程的父进程。当pid为1的进程停止时，内核会杀死该namespace中的所有的其他进程，namespace销毁。  

pid namespace允许嵌套，存在上下级关系。新创建的namespace属于创建进程所在namespace的下级，上级namespace可以看到所有下级namespace的进程信息，无法看到上级或者平级namespace的进程信息。  

与其他namespace不同，进程的pid namespace无法更改，即使调用setns(2)也只会影响当前进程的子进程。  

- terminal-1
```
### 创建新的pid namespace、uts namespace和mnt namespace，运行bash，--fork表示以（unshare的）子进程运行bash程序 ###
# unshare --pid --uts --mount --fork bash

### 设置hostname以便于区分 ###
# hostname namespace-01 && exec bash

### 查看进程树，依然可以看到完整的进程树，因为pstree读取/proc目录的信息，而/proc继承自unshare所在的mount namespace ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(657)
           ├─auditd(614)───{auditd}(615)
           ├─crond(653)
           ├─dbus-daemon(641)───{dbus-daemon}(646)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(490)
           ├─master(1259)─┬─pickup(1812)
           │              └─qmgr(1266)
           ├─polkitd(639)─┬─{polkitd}(648)
           │              ├─{polkitd}(649)
           │              ├─{polkitd}(652)
           │              ├─{polkitd}(658)
           │              └─{polkitd}(661)
           ├─rsyslogd(970)─┬─{rsyslogd}(974)
           │               └─{rsyslogd}(975)
           ├─sshd(969)───sshd(1421)───bash(1425)───unshare(1792)───bash(1793)───pstree(1816)    # bash进行的父进程是unshare(1792)，当前进程为bash(1793)
           ├─systemd-journal(462)
           ├─systemd-logind(640)
           ├─systemd-udevd(498)
           └─tuned(968)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)
### 查看进程号，当前进程号为1 ###
# echo $$
1
### 查看inode number，此时的$$读取的进程信息为unshare所在的mount namespace对应的inode信息 ###
# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026531836]
uts:[4026531838]
mnt:[4026531840]

# readlink /proc/1793/ns/{pid,uts,mnt}
pid:[4026532501]
uts:[4026532500]
mnt:[4026532499]

### 重新挂载proc ###
# mount --types proc proc /proc/

### 再次查看进程树，仅能看到当前namespace的进行信息 ###
# pstree -pl
bash(1)───pstree(45)

### 查看$$对应的namespace的inode，显示1793对应的信息 ###
# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026532501]
uts:[4026532500]
mnt:[4026532499]
```  

- terminal-2
```
### 查看进程树，依然显示完整的进程树信息，说明上级namespace可以看到夏季namespace的pid信息，且显示相对于当前namespace的pid ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(657)
           ├─auditd(614)───{auditd}(615)
           ├─crond(653)
           ├─dbus-daemon(641)───{dbus-daemon}(646)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(490)
           ├─master(1259)─┬─pickup(1812)
           │              └─qmgr(1266)
           ├─polkitd(639)─┬─{polkitd}(648)
           │              ├─{polkitd}(649)
           │              ├─{polkitd}(652)
           │              ├─{polkitd}(658)
           │              └─{polkitd}(661)
           ├─rsyslogd(970)─┬─{rsyslogd}(974)
           │               └─{rsyslogd}(975)
           ├─sshd(969)─┬─sshd(1421)───bash(1425)───unshare(1792)───bash(1793)
           │           └─sshd(1848)───bash(1852)───pstree(1867)
           ├─systemd-journal(462)
           ├─systemd-logind(640)
           ├─systemd-udevd(498)
           └─tuned(968)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)

### 查看inode number ###
# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026531836]
uts:[4026531838]
mnt:[4026531840]

### 创建namespace02并默认挂载proc ###
# unshare --pid --uts --mount --fork --mount-proc bash

### 设置hostname以便于区分 ###
# hostname namespace-02 && exec bash

### 查看inode number ###
# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026532504]
uts:[4026532503]
mnt:[4026532502]

### 查看当前namespace的进程树，无法看到namespace01的进行信息 ###
# pstree -pl
bash(1)───pstree(19)
```  

- terminal-1  
```
### 回到namespace01查看进行树信息，依然只有自身namespace下的进程信息，无法看到上级或者namespace02的进行 ###
# pstree -pl
bash(1)───pstree(50)

### 在namespace01的基础上创建namespace03 ###
# unshare --pid --uts --mount --fork --mount-proc bash

### 修改hostname以便于区分 ###
# hostname namespace-03 && exec bash

### 查看inode number，确保为新namespace ###
# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026532507]
uts:[4026532506]
mnt:[4026532505]

### 查看进程树信息 ###
# pstree -pl
bash(1)───pstree(20)
```  

- terminal-2
```
### 回到namespace02查看进程树信息 ###
# pstree -pl
bash(1)───pstree(21)
```  

- terminal-3
```
### 查看进程树信息 ###
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(657)
           ├─auditd(614)───{auditd}(615)
           ├─crond(653)
           ├─dbus-daemon(641)───{dbus-daemon}(646)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(490)
           ├─master(1259)─┬─pickup(1812)
           │              └─qmgr(1266)
           ├─polkitd(639)─┬─{polkitd}(648)
           │              ├─{polkitd}(649)
           │              ├─{polkitd}(652)
           │              ├─{polkitd}(658)
           │              └─{polkitd}(661)
           ├─rsyslogd(970)─┬─{rsyslogd}(974)
           │               └─{rsyslogd}(975)
           ├─sshd(969)─┬─sshd(1421)───bash(1425)───unshare(1792)───bash(1793)───unshare(1929)───bash(1930)
           │           ├─sshd(1848)───bash(1852)───unshare(1905)───bash(1906)
           │           └─sshd(1952)───bash(1956)───pstree(1971) # terminal-3对应的进程，当前进程
           ├─systemd-journal(462)
           ├─systemd-logind(640)
           ├─systemd-udevd(498)
           └─tuned(968)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)

### 查看进程1793的inode number，1793对应namespace01 ###
# readlink /proc/1793/ns/{pid,uts,mnt}
pid:[4026532501]
uts:[4026532500]
mnt:[4026532499]

### 查看进程1930的inode number，1930对应namespace03 ###
# readlink /proc/1930/ns/{pid,uts,mnt}
pid:[4026532507]
uts:[4026532506]
mnt:[4026532505]

### 查看进程1906的inode number，1906对应namespace02 ###
# readlink /proc/1906/ns/{pid,uts,mnt}
pid:[4026532504]
uts:[4026532503]
mnt:[4026532502]

### 在namespace01中启动新的bash ###
# nsenter --target 1793 --uts --mount --pid bash

### 查看namespace01中的进程树信息，多了一个unshare(51)和一个bash(52) ###
# pstree -pl
bash(1)───unshare(51)───bash(52)

### 查看当前进程的inode number，对应namespace01 ###
# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026532501]
uts:[4026532500]
mnt:[4026532499]

### 查看bash(52)进程的namespace的inode number，对应namespace03，因此证实namespace01可以查看namespace03中的进行信息，也就是上级可以查看下级的进行信息 ###
# readlink /proc/52/ns/{pid,uts,mnt}
pid:[4026532507]
uts:[4026532506]
mnt:[4026532505]
```  

目前的状态大概是这个样子  
```
                              +-------------------------------+
                              |hostname: localhost.localdomain|
                          +---+pid namespace:4026531836       +---+
                          |   +-------------------------------+   |
                          |                                       |
          +---------------v---------------+       +---------------v---------------+
          |hostname: namespace-01         |       |hostname: namespace-02         |
          |pid namespace:4026532501       |       |pid namespace:4026532504       |
          +---------------+---------------+       +-------------------------------+
                          |
          +---------------v---------------+
          |hostname: namespace-03         |
          |pid namespace:4026532507       |
          +-------------------------------+
```  

- terminal-2
```
### 查看进程树 ###
# pstree -pl
bash(1)───pstree(23)

### 执行bash产生子进程 ###
# bash

### 输出子进程号 ###
# echo $$
24

### 在子进程中执行sleep，产生孙进程，后台进程，当前bash依然为bash(24) ###
# sleep 3600 &
[1] 33

### 查看进程树 ###
# pstree -pl
bash(1)───bash(24)─┬─pstree(34)
                   └─sleep(33)
### 退出bash(24) ###
# exit
exit

### 查看进程树，由于bash(24)退出，sleep(33)的父进程变成了bash(1) ###
# pstree -pl
bash(1)─┬─pstree(35)
        └─sleep(33)

### 退出namespace02的bash(1) ###
# exit
exit

### 查看完整进程树，没有sleep进程，说明namespace02中的bash(1)退出后，namespace02中的其他进程都被kill掉了 ###
# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ├─agetty(657)
           ├─auditd(614)───{auditd}(615)
           ├─crond(653)
           ├─dbus-daemon(641)───{dbus-daemon}(646)
           ├─firewalld(663)───{firewalld}(791)
           ├─lvmetad(490)
           ├─master(1259)─┬─pickup(1812)
           │              └─qmgr(1266)
           ├─polkitd(639)─┬─{polkitd}(648)
           │              ├─{polkitd}(649)
           │              ├─{polkitd}(652)
           │              ├─{polkitd}(658)
           │              └─{polkitd}(661)
           ├─rsyslogd(970)─┬─{rsyslogd}(974)
           │               └─{rsyslogd}(975)
           ├─sshd(969)─┬─sshd(1421)───bash(1425)───unshare(1792)───bash(1793)
           │           ├─sshd(1848)───bash(1852)───pstree(2123)
           │           └─sshd(1952)───bash(1956)───nsenter(1996)───bash(1997)
           ├─systemd-journal(462)
           ├─systemd-logind(640)
           ├─systemd-udevd(498)
           └─tuned(968)─┬─{tuned}(1362)
                        ├─{tuned}(1363)
                        ├─{tuned}(1364)
                        └─{tuned}(1378)
```  

- terminal-1  
```
### 查看namespace01的进程树 ###
# pstree
bash───pstree

### 执行bash产生子进程 ###
# bash

### 输出进程号 ###
# echo $$
143

### 执行bash产生子进程 ###
# bash

### 输出进程号 ###
# echo $$
152

### 执行sleep，后台执行 ###
# sleep 3600 &
[1] 162

### 查看当前状态的进程树 ###
# pstree -pl
bash(1)───bash(143)───bash(152)─┬─pstree(163)
                                └─sleep(162)

### sleep为后台执行进程，当前bash进程为bash(152)，退出当前进程 ###
# exit
exit

### 查看当前状态进程树，sleep(162)的父进程变成bash(1)，而不是bash(143) ###
# pstree -pl
bash(1)─┬─bash(143)───pstree(164)
        └─sleep(162)
```  


5、net namespace  
---

net namespace是用来隔离网络空间的，包括：网络设备、路由表、iptables、sockets等。也就是说每个net namespace都拥有自己独立的网络环境。

一个网络设备只能出现在一个net namespace中，无论是物理设备还是虚拟设备。net namespace之间可以通过veth pair进行通信，需要手动配置，默认的net namespace只有一个状态为down的loop back设备。

- terminal-1
```
### 查看inode number ###
# readlink /proc/$$/ns/{net,uts}
net:[4026531956]
uts:[4026531838]

### 创建net namespace和uts namespace，运行bash ###
# unshare --net --uts bash

### 设置hostname ###
# hostname namespace-01 && exec bash

### 查看inode number ###
# readlink /proc/$$/ns/{net,uts}
net:[4026532501]
uts:[4026532499]

### 查看网络设备信息 ###
# ip addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
# ping 127.0.0.1
connect: Network is unreachable

### 启动loop back ###
# ip link set dev lo up

### 查看设备信息 ###
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever

### 再次测试loop back ###
# ping -c 5 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.036 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.060 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.061 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.060 ms
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.061 ms

--- 127.0.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 0.036/0.055/0.061/0.012 ms

### 查看当前进程的pid ###
# echo $$
1554
```  


- terminal-2  
```
### 在另一个终端中创建新的namespace并运行bash ###
# unshare --net --uts bash

### 设置hostname ###
# hostname namespace-02 && exec bash

### 查看inode ###
# readlink /proc/$$/ns/{net,uts}
net:[4026532571]
uts:[4026532569]

### 查看网络设备 ###
# ip addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

### 启动loop back ###
# ip link set dev lo up

### 测试loop back ###
# ping -c 5 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.032 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.047 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.060 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.060 ms
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.068 ms

--- 127.0.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4003ms
rtt min/avg/max/mdev = 0.032/0.053/0.068/0.014 ms

### 查看当前进程号 ###
# echo $$
1624
```  
此时，namespace-01和namespace-02是两个完全封闭的网络，彼此之间不能访问，也无法与外界连接。首先来看一下如何让两个net namespace之间可以相互访问，为此，Linux提供了veth pair，veth pair好比连接了网线的两块网卡，数据从一端流入就会从另一端流出，而我们需要做的就是将两块网卡分别安装到namespace-01和namespace-02中。  


- terminal-3
```
### 查看当前系统的网络设备 ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
    inet 172.16.7.4/24 brd 172.16.7.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:feb9:ee4a/64 scope link 
       valid_lft forever preferred_lft forever

### 创建一对veth，名称由系统生成 ###
# ip link add type veth

### 再次查看网络设备，veth0和veth1就是新创建的一对veth pair ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
    inet 172.16.7.4/24 brd 172.16.7.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:feb9:ee4a/64 scope link 
       valid_lft forever preferred_lft forever
3: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 6e:96:0f:78:bf:d6 brd ff:ff:ff:ff:ff:ff
4: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 22:a1:52:63:8e:ec brd ff:ff:ff:ff:ff:ff

### 将veth0安装到namespace-01，这里通过进程的pid来指定对应的net namespace ###
# ip link set dev veth0 netns 1554

### 再次查看网络设备，设备veth0从当前列表中消失 ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
    inet 172.16.7.4/24 brd 172.16.7.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:feb9:ee4a/64 scope link 
       valid_lft forever preferred_lft forever
4: veth1@if3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 22:a1:52:63:8e:ec brd ff:ff:ff:ff:ff:ff link-netnsid 0
```  

- terminal-1  
```
### 回到第一个终端，查看namespace-01的网络设备信息，veth0成功加入namespace-01，状态为down ###
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: veth0@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 6e:96:0f:78:bf:d6 brd ff:ff:ff:ff:ff:ff link-netnsid 0

```  

- terminal-3
```
### 回到第三个终端，继续将veth1加入namespace-02 ###
# ip link set dev veth1 netns 1624

### 查看网络设备，veth0和veth1均消失 ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
    inet 172.16.7.4/24 brd 172.16.7.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:feb9:ee4a/64 scope link 
       valid_lft forever preferred_lft forever
```  

- terminal-2  
```
### 查看namespace-02的网络设备信息，veth1成功加入namespace-02 ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
4: veth1@if3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 22:a1:52:63:8e:ec brd ff:ff:ff:ff:ff:ff link-netnsid 0

### 接下来为veth pair配置IP地址并启动，首先配置veth1 ###
# ip addr add 192.168.1.2/24 dev veth1

### 启动veth1 ###
# ip link set dev veth1 up 

### 查看设备信息，IP地址正确，状态为LOWERLAYERDOWN而非UP，因为此时veth0尚未启动 ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
4: veth1@if3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN group default qlen 1000
    link/ether 22:a1:52:63:8e:ec brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.2/24 scope global veth1
       valid_lft forever preferred_lft forever
### 查看路由表信息 ###
# ip route
192.168.1.0/24 dev veth1 proto kernel scope link src 192.168.1.2 
```  

- terminal-01  
```
### 回到第一个终端，为veth0配置IP地址 ###
# ip addr add 192.168.1.1/24 dev veth0

### 启动veth0 ###
# ip link set dev veth0 up

### 查看设备信息，IP地址正确，状态为UP ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: veth0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 6e:96:0f:78:bf:d6 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 192.168.1.1/24 scope global veth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6c96:fff:fe78:bfd6/64 scope link 
       valid_lft forever preferred_lft forever

### 测试 ###
# ping -c 5 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.048 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.059 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.073 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=64 time=0.070 ms
64 bytes from 192.168.1.2: icmp_seq=5 ttl=64 time=0.057 ms

--- 192.168.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4002ms
rtt min/avg/max/mdev = 0.048/0.061/0.073/0.011 ms
```  

- terminal-2  
```
### 回到第二个终端，检查veth1的状态 ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
4: veth1@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 22:a1:52:63:8e:ec brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.2/24 scope global veth1
       valid_lft forever preferred_lft forever
    inet6 fe80::20a1:52ff:fe63:8eec/64 scope link 
       valid_lft forever preferred_lft forever

### 测试 ###
# ping -c 5 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.040 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=0.057 ms
64 bytes from 192.168.1.1: icmp_seq=4 ttl=64 time=0.057 ms
64 bytes from 192.168.1.1: icmp_seq=5 ttl=64 time=0.050 ms

--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4002ms
rtt min/avg/max/mdev = 0.040/0.054/0.067/0.010 ms
```  

的网络拓扑结构大概是这个样子  
```
    +-------------------------------------+     +-------------------------------------+
    |   hostname: namespace-01            |     |         hostname: namespace-02      |
    |   ip: 192.168.1.1           veth0 <---------> veth1 ip: 192.168.1.2             |
    |   net namespace: 4026532501         |     |         net namespace: 4026532571   |
    +-------------------------------------+     +-------------------------------------+
```  
两个net namespace之间可以使用veth pair连接，多个net namespace之间就需要用到bridge，bridge实现交换机的功能。
退出所有namespace从头开始，namespace退出后，veth pair会被自动清理。

接下来的操作将使用 ip netns 命令来创建net namespace，这样可以在一个终端中完成所有操作。与上面重复的操作不在详细说明
```
### 创建三个net namespace，分别为：net-0、net-1、net-2 ###
# ip netns add net-0
# ip netns add net-1
# ip netns add net-2

### 查看创建结果 ###
# ip netns list
net-2
net-1
net-0

### 进入net-0查看网络设备，net-1和net-2操作方式类似 ###
# ip netns exec net-0 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

### 分别启动三个net namespace的loop back设备 ###
# ip netns exec net-0 ip link set dev lo up
# ip netns exec net-1 ip link set dev lo up
# ip netns exec net-2 ip link set dev lo up

### 创建bridge设备 ###
# ip link add type bridge

### 查看设备，bridge0为新创建的网桥，名称由系统生成 ###
# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
3: bridge0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 32:f1:74:a0:e0:d0 brd ff:ff:ff:ff:ff:ff

### 创建一对veth pair ###
# ip link add type veth
# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
3: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 3a:44:74:56:87:fb brd ff:ff:ff:ff:ff:ff
4: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether f6:4a:22:30:98:6d brd ff:ff:ff:ff:ff:ff
5: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether de:24:db:85:d4:57 brd ff:ff:ff:ff:ff:ff

### 使用veth pair连接net-0和bridge0 ###
# ip link set dev veth0 netns net-0
# ip link set dev veth1 master bridge0
# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
3: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 3a:44:74:56:87:fb brd ff:ff:ff:ff:ff:ff
5: veth1@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether de:24:db:85:d4:57 brd ff:ff:ff:ff:ff:ff link-netnsid 0

### 设置net-0中veth0的IP ###
# ip netns exec net-0 ip addr add 192.168.1.1/24 dev veth0

### 启动net-0中veth0 ###
# ip netns exec net-0 ip link set dev veth0 up

### 启动net-0中设备 ###
# ip netns exec net-0 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
6: veth0@if7: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN group default qlen 1000
    link/ether a6:34:37:f0:a3:28 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.1/24 scope global veth0
       valid_lft forever preferred_lft forever

### 启动net-0外的veth1 ###
# ip link set dev veth1 up

### 查看设备状态 ###
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
    inet 172.16.7.4/24 brd 172.16.7.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:feb9:ee4a/64 scope link 
       valid_lft forever preferred_lft forever
3: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 3a:44:74:56:87:fb brd ff:ff:ff:ff:ff:ff
5: veth1@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master bridge0 state UP group default qlen 1000
    link/ether de:24:db:85:d4:57 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::dc24:dbff:fe85:d457/64 scope link 
       valid_lft forever preferred_lft forever

### 使用相同的步骤为net-1和net-2分别创建veth pair并连接网桥，步骤略 ###

### 启动网桥 ###
# ip link set dev bridge0 up

### 查看网络设备 ###
# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:b9:ee:4a brd ff:ff:ff:ff:ff:ff
3: bridge0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 32:f1:74:a0:e0:d0 brd ff:ff:ff:ff:ff:ff
5: veth1@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master bridge0 state UP mode DEFAULT group default qlen 1000
    link/ether de:24:db:85:d4:57 brd ff:ff:ff:ff:ff:ff link-netnsid 0
7: veth3@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master bridge0 state UP mode DEFAULT group default qlen 1000
    link/ether 32:f1:74:a0:e0:d0 brd ff:ff:ff:ff:ff:ff link-netnsid 1
9: veth5@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master bridge0 state UP mode DEFAULT group default qlen 1000
    link/ether 6e:60:6d:c5:65:1f brd ff:ff:ff:ff:ff:ff link-netnsid 2

### 测试 ###
# ip netns exec net-0 ping -c 5 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.046 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.083 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.083 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=64 time=0.070 ms
64 bytes from 192.168.1.2: icmp_seq=5 ttl=64 time=0.071 ms

--- 192.168.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4003ms
rtt min/avg/max/mdev = 0.046/0.070/0.083/0.016 ms

[root@localhost ~]# ip netns exec net-0 ping -c 5 192.168.1.3
PING 192.168.1.3 (192.168.1.3) 56(84) bytes of data.
64 bytes from 192.168.1.3: icmp_seq=1 ttl=64 time=0.047 ms
64 bytes from 192.168.1.3: icmp_seq=2 ttl=64 time=0.087 ms
64 bytes from 192.168.1.3: icmp_seq=3 ttl=64 time=0.069 ms
64 bytes from 192.168.1.3: icmp_seq=4 ttl=64 time=0.085 ms
64 bytes from 192.168.1.3: icmp_seq=5 ttl=64 time=0.070 ms

--- 192.168.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 0.047/0.071/0.087/0.017 ms
```  

当前网络拓扑图大概是这个样子  
```
+----------------------------+ +----------------------------+ +----------------------------+
|                            | |                            | |                            |
|        netns: net-0        | |        netns: net-1        | |        netns: net-2        |
|  +----------------------+  | |  +----------------------+  | |  +----------------------+  |
|  |   lo: 127.0.0.1/8    |  | |  |   lo: 127.0.0.1/8    |  | |  |   lo: 127.0.0.1/8    |  |
|  +----------------------+  | |  +----------------------+  | |  +----------------------+  |
|  +----------------------+  | |  +----------------------+  | |  +----------------------+  |
|  | veth: 192.168.1.1/24 |  | |  | veth: 192.168.1.2/24 |  | |  | veth: 192.168.1.3/24 |  |
|  +-----------^----------+  | |  +-----------^----------+  | |  +-----------^----------+  |
|              |             | |              |             | |              |             |
+----------------------------+ +----------------------------+ +----------------------------+
               |                              |                              |
+--------------v------------------------------v------------------------------v-------------+
|                                          bridge0                                         |
+------------------------------------------------------------------------------------------+
```  

此时此刻，三个net namespace都还只能访问192.168.1.0/24网段，也就是只能彼此之间相互访问，无法访问ens33（物理网卡），业务发访问外部网络。要想访问外部网络，需要启用Linux的ip_forward功能并配置NAT转发规则。  
```
### 首先，临时启用Linux的ip_forward功能 ###
# sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1

### 为bridge0配置IP ###
# ip addr add 192.168.1.224/24 dev bridge0

### 将bridge0的IP设置为各个net namespace的默认路由 ###
# ip netns exec net-0 ip route add default via 192.168.1.224

### 查看路由表信息 ###
# ip netns exec net-0 ip route
default via 192.168.1.224 dev veth0 
192.168.1.0/24 dev veth0 proto kernel scope link src 192.168.1.1 

### 配置NAT，将所有请求伪装成ens33的IP ###
# iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE

### 测试外网（百度）IP，无法使用域名，因为没有DNS ###
# ip netns exec net-0 ping -c 5 180.97.33.108
PING 180.97.33.108 (180.97.33.108) 56(84) bytes of data.
64 bytes from 180.97.33.108: icmp_seq=1 ttl=127 time=12.3 ms
64 bytes from 180.97.33.108: icmp_seq=2 ttl=127 time=12.2 ms
64 bytes from 180.97.33.108: icmp_seq=3 ttl=127 time=12.8 ms
64 bytes from 180.97.33.108: icmp_seq=4 ttl=127 time=12.6 ms
64 bytes from 180.97.33.108: icmp_seq=5 ttl=127 time=12.3 ms

--- 180.97.33.108 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4012ms
rtt min/avg/max/mdev = 12.216/12.496/12.877/0.262 ms

### 能ping，说明网络通畅，但是目前还无法测试curl，比如 ###
# ip netns exec net-0 curl http://180.97.33.108
curl: (7) Failed connect to 180.97.33.108:80; No route to host

### 查看iptables规则，主要关系FORWARD ###
# iptables -n -L --line-numbers
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination  
<snip>
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1       20  4014 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
2        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
3       28  2232 FORWARD_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
4       28  2232 FORWARD_IN_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
5       28  2232 FORWARD_IN_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
6        5   300 FORWARD_OUT_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
7        5   300 FORWARD_OUT_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
8        0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate INVALID
9        4   240 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT 87 packets, 57504 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1     2232  556K OUTPUT_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
<snip>

### 为了让curl可以访问，需要在Chain FORWARD的第8条之前插入转发规则 ###
# iptables -t filter -I FORWARD 8 -i ens33 -o bridge0 -j ACCEPT
# iptables -t filter -I FORWARD 9 -o ens33 -i bridge0 -j ACCEPT

### 再次查看规则 ###
# iptables -L -n -v --line-numbers
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
<snip>

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1       20  4014 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
2        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
3       28  2232 FORWARD_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
4       28  2232 FORWARD_IN_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
5       28  2232 FORWARD_IN_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
6        5   300 FORWARD_OUT_ZONES_SOURCE  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
7        5   300 FORWARD_OUT_ZONES  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
8        0     0 ACCEPT     all  --  ens33  bridge0  0.0.0.0/0            0.0.0.0/0           
9        0     0 ACCEPT     all  --  bridge0 ens33   0.0.0.0/0            0.0.0.0/0           
10       0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate INVALID
11       4   240 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT 4 packets, 448 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1     2346  602K OUTPUT_direct  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
<snip>

### 测试curl ###
# ip netns exec net-0 curl http://180.97.33.108
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=http://s1.bdstatic.com/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn"></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=http://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');</script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```  
目前对ping和curl访问方式的差别尚未理解透彻，无法解释为什么ping不需要forward而curl需要，猜测和ICMP协议有关吧～  

当前的网络拓扑结构大概就是下面这个样子吧  
```
+-------------------------------------------------------------------------------------------------------------------------+
|                                                Newwork Protocol Stack                                                   |
+--------------^-------------------------------------------------------------^--------------------------------------------+
               |                                                             |
               |               +---------------------------------------------v--------------------------------------------+
               |               |                                          bridge0                                         |
               |               |                                     ip: 192.168.1.224                                    |
               |               +--------------^------------------------------^------------------------------^-------------+
               |                              |                              |                              |
+----------------------------+ +----------------------------+ +----------------------------+ +----------------------------+
|              |             | |              |             | |              |             | |              |             |
|  +-----------v----------+  | |  +-----------v----------+  | |  +-----------v----------+  | |  +-----------v----------+  |
|  | veth: 172.16.7.4/24  |  | |  | veth: 192.168.1.1/24 |  | |  | veth: 192.168.1.2/24 |  | |  | veth: 192.168.1.3/24 |  |
|  +----------------------+  | |  +----------------------+  | |  +----------------------+  | |  +----------------------+  |
|  +----------------------+  | |  +----------------------+  | |  +----------------------+  | |  +----------------------+  |
|  |   lo: 127.0.0.1/8    |  | |  |   lo: 127.0.0.1/8    |  | |  |   lo: 127.0.0.1/8    |  | |  |   lo: 127.0.0.1/8    |  |
|  +----------------------+  | |  +----------------------+  | |  +----------------------+  | |  +----------------------+  |
|      physical network      | |        netns: net-0        | |        netns: net-1        | |        netns: net-2        |
|                            | |                            | |                            | |                            |
+----------------------------+ +----------------------------+ +----------------------------+ +----------------------------+
```  

6、user namespace
---

user namespace用来隔离user和group，也就是用来隔离权限（而非文件系统）。和权限扯上关系的都比较复杂，大有有牵一发动全身的姿态。作为所有namespace中实现最复杂的一个，这里就节点介绍一下吧，点到即止  
```
### 尝试创建user namespace，出师不利，第一步就出状况 ###
# unshare --uts --user bash
/usr/bin/id: cannot find name for group ID 65534
/usr/bin/id: cannot find name for user ID 65534

### 这是由于在笔者的操作系统版本（CentOS7.5）中，user namespace还是预览版本的功能，默认并没有启用。文件/proc/sys/user/max_user_namespaces中设置了允许创建的user namespace的数量 ###
# cat /proc/sys/user/max_user_namespaces
0

### 修改max_user_namespaces文件 ###
# echo 2147483647 > /proc/sys/user/max_user_namespaces

### 再次创建user namespace，出现如下提示 ###
# unshare --user --uts bash
/usr/bin/id: cannot find name for group ID 65534
/usr/bin/id: cannot find name for user ID 65534

### 查看当前用户id，gid和uid都显示为65536，/proc/sys/kernel/overflowuid和/proc/sys/kernel/overflowgid文件中的默认值 ###
$ id
uid=65534 gid=65534 groups=65534 context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```  

之所以显示65536这个默认值，是因为在创建新的user namespace时，我们没有指定当前用户（root）在新的user namespace中的映射关系。也就是说，在创建新的user namespace时，并没有多出来新的user，新namespace中的user是从老的namespace中映射过去的。

老namespace中的普通user可以映射成新namespace中的root，该root的uid变成了1。该root拥有与映射前的用户相同的权限，但是并不具备映射前的capabilities
```
### 在顶层user namespace中，root用户可以访问其他用户的home目录并创建文件 ###
# touch /home/chenlei/a.txt

### 查看文件创建结果 ###
# ll /home/chenlei/
total 0
-rw-r--r--. 1 chenlei chenlei 0 Feb 24 22:54 a.txt

### 创建新的user namespace和uts namespace并运行bash，--map-root-user表示将当前用户映射为新namespace的root用户 ###
# unshare --user --uts --map-root-user bash

### 修改hostname便于观察 ###
# hostname namespace-01 && exec bash

### 查看当前用户id，依然是root用户 ###
# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

### 再次查看其他用户的home目录，发现无权访问 ###
# ll /home/chenlei/
ls: cannot open directory /home/chenlei/: Permission denied

### 查看root用户的home目录 ###
# pwd
/root
# touch a.txt && ll 
total 8
-rw-------. 1 root root 1510 Oct 17 21:19 anaconda-ks.cfg
-rw-r--r--. 1 root root    0 Feb 24 22:59 a.txt

### 退出当前namespace ###
# exit
exit

### 使用普通用户登录 ###
# su - chenlei
Last login: Sun Feb 24 22:51:26 CST 2019 on pts/0

### 以普通用户身份创建新的user namespace和uts namespace ###
$ unshare --user --uts --map-root-user bash

### 修改hostname便于观察 ###
# hostname namespace-02 && exec bash

### 查看当前用户的uid，发现chenlei用户变成了root用户 ###
# id
uid=0(root) gid=0(root) groups=0(root),65534 context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

### 查看chenlei用户的home目录的owner，owner变为root，也就是现在新user namespace的root ###
# ll /home/
total 0
drwx------. 2 root root 96 Feb 24 22:37 chenlei

### 查看根目录下文件的owner，所以的文件owner都变成了65534 ###
# ll /
total 16
lrwxrwxrwx.   1 65534 65534    7 Oct 17 21:15 bin -> usr/bin
dr-xr-xr-x.   5 65534 65534 4096 Oct 17 21:20 boot
drwxr-xr-x.  21 65534 65534 3260 Feb 24 21:03 dev
drwxr-xr-x.  74 65534 65534 8192 Feb 24 21:03 etc
drwxr-xr-x.   3 65534 65534   21 Oct 17 21:18 home
lrwxrwxrwx.   1 65534 65534    7 Oct 17 21:15 lib -> usr/lib
lrwxrwxrwx.   1 65534 65534    9 Oct 17 21:15 lib64 -> usr/lib64
drwxr-xr-x.   2 65534 65534    6 Apr 11  2018 media
drwxr-xr-x.   2 65534 65534    6 Apr 11  2018 mnt
drwxr-xr-x.   2 65534 65534    6 Apr 11  2018 opt
dr-xr-xr-x. 111 65534 65534    0 Feb 24 21:03 proc
dr-xr-x---.   5 65534 65534  226 Feb 24 22:59 root
drwxr-xr-x.  22 65534 65534  660 Feb 24 21:03 run
lrwxrwxrwx.   1 65534 65534    8 Oct 17 21:15 sbin -> usr/sbin
drwxr-xr-x.   2 65534 65534    6 Apr 11  2018 srv
dr-xr-xr-x.  13 65534 65534    0 Feb 24 21:03 sys
drwxrwxrwt.   7 65534 65534   93 Feb 24 21:04 tmp
drwxr-xr-x.  13 65534 65534  155 Oct 17 21:15 usr
drwxr-xr-x.  19 65534 65534  267 Oct 17 21:20 var

### 使用这个新root尝试访问/root目录，发现无权访问 ###
# ll /root/
ls: cannot open directory /root/: Permission denied
```  

既然权限与映射前的一致，那么这个user namespace到底有什么用呢。答案是 capabilities ，即使是普通用户，在映射为新user namespace之后，会拥有新user namespace所有的capabilites，关于capabilities的介绍。这些capabilities可以与其他namespace配合使用。  

每一个其他namespace都有一个指向user namespace的指针。也就是说，user namespace控制着其他namespace的权限和capabilites。这个user namespace指针指向namespace创建进程所属的user namespace。  

user namespace可以嵌套～


7、Cgroup namespace
---

略～
