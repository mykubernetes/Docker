两台harbor安装keepalived  
1、主harbor端配置  
```
#yum -y install keepalived

# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   script_user root
   #需要制定脚本运行用户
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
vrrp_script check_harbor {
    script "/opt/harbor.sh"
    interval 2
    weight 20

}
#使用监控脚本来监控自身80端口，因为他是整个harbor的访问入口

vrrp_instance VI_1 {
    state MASTER
    interface ens32
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.111.100/32 dev ens32 label ens32:2
    }
    track_script {
    check_harbor
    } 
}
```

2、从harbord端配置  
```
# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   script_user root
   #需要制定脚本运行用户
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL1
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
vrrp_script check_harbor {
    script "/opt/harbor.sh"
    interval 2
    weight 20

}

vrrp_instance VI_1 {
    state BACKUP
    interface ens32
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.111.100/32 dev ens32 label ens32:2
    }
    track_script {
    check_harbor
    } 
}
```

3、检查脚本  
```
# vim /opt/harbor.sh 

#!/bin/bash

sum=`netstat -lnpt | grep -wo 80 | wc -l`

if [ $sum -eq 0 ]; then
        pkill -9 keepalived
fi

# chmod +x /opt/harbor.sh 
```
