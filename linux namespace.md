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

pid namespace允许嵌套，存在上下级关系。新创建的namespace属于创建进程所在namespace的下级，上级namespace可以看到***所有下级***namespace的进程信息，无法看到上级或者平级namespace的进程信息。  

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
