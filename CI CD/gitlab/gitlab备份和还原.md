
1、gitlab查看版本  
```
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
```  

2、查看gitlab服务状态命令  
```
gitlab-ctl status
```  

3、启动、停止服务
```
# 启动Gitlab所有组件
sudo gitlab-ctl start

# 停止Gitlab所有组件
sudo gitlab-ctl stop

# 重启Gitlab所有组件
sudo gitlab-ctl restart
```

备份与恢复  
===
备份配置  
---
配置文件再/etc/gitlab/ 下面，将所有的配置用tar备份即可  
```
[root@localhost init.d]# cd /etc/gitlab/
[root@localhost gitlab]# ls
gitlab.rb  gitlab-secrets.json  trusted-certs
```  
 
备份数据  
---
Gitlab的备份目录路径设置  
```
[root@code-server ~]# vim /etc/gitlab/gitlab.rb
gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/data/gitlab/backups"    //gitlab备份目录
gitlab_rails['backup_archive_permissions'] = 0644       //生成的备份文件权限
gitlab_rails['backup_keep_time'] = 7776000              //备份保留天数为3个月（即90天，这里是7776000秒）
 
[root@code-server ~]# mkdir -p /data/gitlab/backups
[root@code-server ~]# chown -R git.git /data/gitlab/backups
[root@code-server ~]# chmod -R 777 /data/gitlab/backups

如上设置了gitlab备份目录路径为/data/gitlab/backups，最后使用下面命令重载gitlab配置文件，是上述修改生效！
root@code-server ~]# gitlab-ctl reconfigure
```  


GItlab备份操作（使用备份命令"gitlab-rake gitlab:backup:create"）  
```
手动备份gitlab
[root@code-server backups]# gitlab-rake gitlab:backup:create
Dumping database ...
Dumping PostgreSQL database gitlabhq_production ... [DONE]
done
Dumping repositories ...
 * treesign/treesign ... [DONE]
 * gateway/gateway ... [DONE]
 * treesign/treesign-doc ... [SKIPPED]
 * qwsign/qwsign ... [DONE]
 * qwsign/qwsign-doc ... [DONE]
 * test/test ... [DONE]
done
Dumping uploads ...
done
Dumping builds ...
done
Dumping artifacts ...
done
Dumping pages ...
done
Dumping lfs objects ...
done
Dumping container registry images ...
[DISABLED]
Creating backup archive: 1510471890_2017_11_12_9.4.5_gitlab_backup.tar ... done
Uploading backup archive to remote storage  ... skipped
Deleting tmp directories ... done
done
done
done
done
done
done
done
Deleting old backups ... done. (0 removed)
 
然后查看下备份文件（文件权限是设定好的644）
[root@code-server backups]# ll
total 244
-rw-r--r-- 1 git git 245760 Nov 12 15:33 1510472027_2017_11_12_9.4.5_gitlab_backup.tar
 
编写备份脚本，结合crontab实施自动定时备份，比如每天0点、6点、12点、18点各备份一次
[root@code-server backups]# pwd
/data/gitlab/backups
[root@code-server backups]# vim gitlab_backup.sh
#!/bin/bash
/usr/bin/gitlab-rake gitlab:backup:create CRON=1
 
注意：环境变量CRON=1的作用是如果没有任何错误发生时， 抑制备份脚本的所有进度输出
 
[root@code-server backups]# crontab -l
0 0,6,12,18 * * * /bin/bash -x /data/gitlab/backups/gitlab_backup.sh > /dev/null 2>&1
```


3）Gitlab恢复操作  
---
```
GItlab只能还原到与备份文件相同的gitlab版本。
假设在上面gitlab备份之前创建了test项目，然后不小心误删了test项目，现在就进行gitlab恢复操作：

1）停止相关数据连接服务
[root@code-server backups]# gitlab-ctl stop unicorn
ok: down: unicorn: 0s, normally up
[root@code-server backups]# gitlab-ctl stop sidekiq
ok: down: sidekiq: 1s, normally up
[root@code-server backups]# gitlab-ctl status
run: gitaly: (pid 98087) 1883s; run: log: (pid 194202) 163003s
run: gitlab-monitor: (pid 98101) 1883s; run: log: (pid 194363) 163002s
run: gitlab-workhorse: (pid 98104) 1882s; run: log: (pid 194362) 163002s
run: logrotate: (pid 98117) 1882s; run: log: (pid 5793) 160832s
run: nginx: (pid 98123) 1881s; run: log: (pid 194359) 163002s
run: node-exporter: (pid 98167) 1881s; run: log: (pid 194360) 163002s
run: postgres-exporter: (pid 98173) 1881s; run: log: (pid 194204) 163003s
run: postgresql: (pid 98179) 1880s; run: log: (pid 194365) 163002s
run: prometheus: (pid 98187) 1880s; run: log: (pid 194364) 163002s
run: redis: (pid 98230) 1879s; run: log: (pid 194358) 163002s
run: redis-exporter: (pid 98234) 1879s; run: log: (pid 194208) 163003s
down: sidekiq: 8s, normally up; run: log: (pid 194437) 163001s
down: unicorn: 21s, normally up; run: log: (pid 194443) 163001s
  
2）现在通过之前的备份文件进行恢复（必须要备份文件放到备份路径下，这里备份路径我自定义的/data/gitlab/backups，默认的是/var/opt/gitlab/backups）
[root@code-server backups]# pwd
/data/gitlab/backups
[root@code-server backups]# ll
total 244
-rw-r--r-- 1 git git 245760 Nov 12 15:33 1510472027_2017_11_12_9.4.5_gitlab_backup.tar
  
Gitlab的恢复操作会先将当前所有的数据清空，然后再根据备份数据进行恢复
[root@code-server backups]# gitlab-rake gitlab:backup:restore BACKUP=1510472027_2017_11_12_9.4.5
Unpacking backup ... done
Before restoring the database we recommend removing all existing
tables to avoid future upgrade problems. Be aware that if you have
custom tables in the GitLab database these tables and all data will be
removed.
  
Do you want to continue (yes/no)?
........
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
WARNING:  no privileges were granted for "public"
GRANT
[DONE]
done
Restoring repositories ...
 * treesign/treesign ... [DONE]
 * gateway/gateway ... [DONE]
 * treesign/treesign-doc ... [DONE]
 * qwsign/qwsign ... [DONE]
 * qwsign/qwsign-doc ... [DONE]
 * test/test ... [DONE]
Put GitLab hooks in repositories dirs [DONE]
done
Restoring uploads ...
done
Restoring builds ...
done
Restoring artifacts ...
done
Restoring pages ...
done
Restoring lfs objects ...
done
This will rebuild an authorized_keys file.
You will lose any data stored in authorized_keys file.
Do you want to continue (yes/no)? yes
  
  
Deleting tmp directories ... done
done
done
done
done
done
done
done
[root@code-server backups]#
  
最后再次启动Gitlab
[root@code-server backups]# gitlab-ctl start
ok: run: gitaly: (pid 98087) 2138s
ok: run: gitlab-monitor: (pid 98101) 2138s
ok: run: gitlab-workhorse: (pid 98104) 2137s
ok: run: logrotate: (pid 98117) 2137s
ok: run: nginx: (pid 98123) 2136s
ok: run: node-exporter: (pid 98167) 2136s
ok: run: postgres-exporter: (pid 98173) 2136s
ok: run: postgresql: (pid 98179) 2135s
ok: run: prometheus: (pid 98187) 2135s
ok: run: redis: (pid 98230) 2134s
ok: run: redis-exporter: (pid 98234) 2134s
ok: run: sidekiq: (pid 104494) 0s
ok: run: unicorn: (pid 104497) 1s
[root@code-server backups]# gitlab-ctl status
run: gitaly: (pid 98087) 2142s; run: log: (pid 194202) 163262s
run: gitlab-monitor: (pid 98101) 2142s; run: log: (pid 194363) 163261s
run: gitlab-workhorse: (pid 98104) 2141s; run: log: (pid 194362) 163261s
run: logrotate: (pid 98117) 2141s; run: log: (pid 5793) 161091s
run: nginx: (pid 98123) 2140s; run: log: (pid 194359) 163261s
run: node-exporter: (pid 98167) 2140s; run: log: (pid 194360) 163261s
run: postgres-exporter: (pid 98173) 2140s; run: log: (pid 194204) 163262s
run: postgresql: (pid 98179) 2139s; run: log: (pid 194365) 163261s
run: prometheus: (pid 98187) 2139s; run: log: (pid 194364) 163261s
run: redis: (pid 98230) 2138s; run: log: (pid 194358) 163261s
run: redis-exporter: (pid 98234) 2138s; run: log: (pid 194208) 163262s
run: sidekiq: (pid 104494) 4s; run: log: (pid 194437) 163260s
run: unicorn: (pid 104497) 4s; run: log: (pid 194443) 163260s
 
恢复命令完成后，可以check检查一下恢复情况
[root@code-server backups]# gitlab-rake gitlab:check SANITIZE=true
Checking GitLab Shell ...
 
GitLab Shell version >= 5.3.1 ? ... OK (5.3.1)
Repo base directory exists?
default... yes
Repo storage directories are symlinks?
default... no
Repo paths owned by git:root, or git:git?
default... yes
Repo paths access is drwxrws---?
default... yes
hooks directories in repos are links: ...
5/1 ... ok
6/2 ... ok
5/3 ... repository is empty
12/4 ... ok
12/5 ... ok
Running /opt/gitlab/embedded/service/gitlab-shell/bin/check
Check GitLab API access: OK
Access to /var/opt/gitlab/.ssh/authorized_keys: OK
Send ping to redis server: OK
gitlab-shell self-check successful
 
Checking GitLab Shell ... Finished
 
Checking Sidekiq ...
 
Running? ... yes
Number of Sidekiq processes ... 1
 
Checking Sidekiq ... Finished
 
Checking Reply by email ...
 
Reply by email is disabled in config/gitlab.yml
 
Checking Reply by email ... Finished
 
Checking LDAP ...
 
LDAP is disabled in config/gitlab.yml
 
Checking LDAP ... Finished
 
Checking GitLab ...
 
Git configured correctly? ... yes
Database config exists? ... yes
All migrations up? ... yes
Database contains orphaned GroupMembers? ... no
GitLab config exists? ... yes
GitLab config up to date? ... yes
Log directory writable? ... yes
Tmp directory writable? ... yes
Uploads directory exists? ... yes
Uploads directory has correct permissions? ... yes
Uploads directory tmp has correct permissions? ... yes
Init script exists? ... skipped (omnibus-gitlab has no init script)
Init script up-to-date? ... skipped (omnibus-gitlab has no init script)
Projects have namespace: ...
5/1 ... yes
6/2 ... yes
5/3 ... yes
12/4 ... yes
12/5 ... yes
Redis version >= 2.8.0? ... yes
Ruby version >= 2.3.3 ? ... yes (2.3.3)
Git version >= 2.7.3 ? ... yes (2.13.4)
Active users: ... 11
 
Checking GitLab ... Finished
  
然后稍等一会（如果启动gitlab后，访问出现500，这是因为redis等程序还没完全启动，等一会儿访问就ok了），再次登录Gitlab，就会发现之前误删除的test项目已经恢复了！
  
另外：Gitlab迁移与恢复一样，但是要求两个GitLab版本号一致
```  


```
GItlab只能还原到与备份文件相同的gitlab版本。
假设在上面gitlab备份之前创建了test项目，然后不小心误删了test项目，现在就进行gitlab恢复操作：
  
1）停止相关数据连接服务
[root@code-server backups]# gitlab-ctl stop unicorn
ok: down: unicorn: 0s, normally up
[root@code-server backups]# gitlab-ctl stop sidekiq
ok: down: sidekiq: 1s, normally up
[root@code-server backups]# gitlab-ctl status
run: gitaly: (pid 98087) 1883s; run: log: (pid 194202) 163003s
run: gitlab-monitor: (pid 98101) 1883s; run: log: (pid 194363) 163002s
run: gitlab-workhorse: (pid 98104) 1882s; run: log: (pid 194362) 163002s
run: logrotate: (pid 98117) 1882s; run: log: (pid 5793) 160832s
run: nginx: (pid 98123) 1881s; run: log: (pid 194359) 163002s
run: node-exporter: (pid 98167) 1881s; run: log: (pid 194360) 163002s
run: postgres-exporter: (pid 98173) 1881s; run: log: (pid 194204) 163003s
run: postgresql: (pid 98179) 1880s; run: log: (pid 194365) 163002s
run: prometheus: (pid 98187) 1880s; run: log: (pid 194364) 163002s
run: redis: (pid 98230) 1879s; run: log: (pid 194358) 163002s
run: redis-exporter: (pid 98234) 1879s; run: log: (pid 194208) 163003s
down: sidekiq: 8s, normally up; run: log: (pid 194437) 163001s
down: unicorn: 21s, normally up; run: log: (pid 194443) 163001s
  
2）现在通过之前的备份文件进行恢复（必须要备份文件放到备份路径下，这里备份路径我自定义的/data/gitlab/backups，默认的是/var/opt/gitlab/backups）
[root@code-server backups]# pwd
/data/gitlab/backups
[root@code-server backups]# ll
total 244
-rw-r--r-- 1 git git 245760 Nov 12 15:33 1510472027_2017_11_12_9.4.5_gitlab_backup.tar
  
Gitlab的恢复操作会先将当前所有的数据清空，然后再根据备份数据进行恢复
[root@code-server backups]# gitlab-rake gitlab:backup:restore BACKUP=1510472027_2017_11_12_9.4.5
Unpacking backup ... done
Before restoring the database we recommend removing all existing
tables to avoid future upgrade problems. Be aware that if you have
custom tables in the GitLab database these tables and all data will be
removed.
  
Do you want to continue (yes/no)?
........
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
WARNING:  no privileges were granted for "public"
GRANT
[DONE]
done
Restoring repositories ...
 * treesign/treesign ... [DONE]
 * gateway/gateway ... [DONE]
 * treesign/treesign-doc ... [DONE]
 * qwsign/qwsign ... [DONE]
 * qwsign/qwsign-doc ... [DONE]
 * test/test ... [DONE]
Put GitLab hooks in repositories dirs [DONE]
done
Restoring uploads ...
done
Restoring builds ...
done
Restoring artifacts ...
done
Restoring pages ...
done
Restoring lfs objects ...
done
This will rebuild an authorized_keys file.
You will lose any data stored in authorized_keys file.
Do you want to continue (yes/no)? yes
  
  
Deleting tmp directories ... done
done
done
done
done
done
done
done
[root@code-server backups]#
  
最后再次启动Gitlab
[root@code-server backups]# gitlab-ctl start
ok: run: gitaly: (pid 98087) 2138s
ok: run: gitlab-monitor: (pid 98101) 2138s
ok: run: gitlab-workhorse: (pid 98104) 2137s
ok: run: logrotate: (pid 98117) 2137s
ok: run: nginx: (pid 98123) 2136s
ok: run: node-exporter: (pid 98167) 2136s
ok: run: postgres-exporter: (pid 98173) 2136s
ok: run: postgresql: (pid 98179) 2135s
ok: run: prometheus: (pid 98187) 2135s
ok: run: redis: (pid 98230) 2134s
ok: run: redis-exporter: (pid 98234) 2134s
ok: run: sidekiq: (pid 104494) 0s
ok: run: unicorn: (pid 104497) 1s
[root@code-server backups]# gitlab-ctl status
run: gitaly: (pid 98087) 2142s; run: log: (pid 194202) 163262s
run: gitlab-monitor: (pid 98101) 2142s; run: log: (pid 194363) 163261s
run: gitlab-workhorse: (pid 98104) 2141s; run: log: (pid 194362) 163261s
run: logrotate: (pid 98117) 2141s; run: log: (pid 5793) 161091s
run: nginx: (pid 98123) 2140s; run: log: (pid 194359) 163261s
run: node-exporter: (pid 98167) 2140s; run: log: (pid 194360) 163261s
run: postgres-exporter: (pid 98173) 2140s; run: log: (pid 194204) 163262s
run: postgresql: (pid 98179) 2139s; run: log: (pid 194365) 163261s
run: prometheus: (pid 98187) 2139s; run: log: (pid 194364) 163261s
run: redis: (pid 98230) 2138s; run: log: (pid 194358) 163261s
run: redis-exporter: (pid 98234) 2138s; run: log: (pid 194208) 163262s
run: sidekiq: (pid 104494) 4s; run: log: (pid 194437) 163260s
run: unicorn: (pid 104497) 4s; run: log: (pid 194443) 163260s
 
恢复命令完成后，可以check检查一下恢复情况
[root@code-server backups]# gitlab-rake gitlab:check SANITIZE=true
Checking GitLab Shell ...
 
GitLab Shell version >= 5.3.1 ? ... OK (5.3.1)
Repo base directory exists?
default... yes
Repo storage directories are symlinks?
default... no
Repo paths owned by git:root, or git:git?
default... yes
Repo paths access is drwxrws---?
default... yes
hooks directories in repos are links: ...
5/1 ... ok
6/2 ... ok
5/3 ... repository is empty
12/4 ... ok
12/5 ... ok
Running /opt/gitlab/embedded/service/gitlab-shell/bin/check
Check GitLab API access: OK
Access to /var/opt/gitlab/.ssh/authorized_keys: OK
Send ping to redis server: OK
gitlab-shell self-check successful
 
Checking GitLab Shell ... Finished
 
Checking Sidekiq ...
 
Running? ... yes
Number of Sidekiq processes ... 1
 
Checking Sidekiq ... Finished
 
Checking Reply by email ...
 
Reply by email is disabled in config/gitlab.yml
 
Checking Reply by email ... Finished
 
Checking LDAP ...
 
LDAP is disabled in config/gitlab.yml
 
Checking LDAP ... Finished
 
Checking GitLab ...
 
Git configured correctly? ... yes
Database config exists? ... yes
All migrations up? ... yes
Database contains orphaned GroupMembers? ... no
GitLab config exists? ... yes
GitLab config up to date? ... yes
Log directory writable? ... yes
Tmp directory writable? ... yes
Uploads directory exists? ... yes
Uploads directory has correct permissions? ... yes
Uploads directory tmp has correct permissions? ... yes
Init script exists? ... skipped (omnibus-gitlab has no init script)
Init script up-to-date? ... skipped (omnibus-gitlab has no init script)
Projects have namespace: ...
5/1 ... yes
6/2 ... yes
5/3 ... yes
12/4 ... yes
12/5 ... yes
Redis version >= 2.8.0? ... yes
Ruby version >= 2.3.3 ? ... yes (2.3.3)
Git version >= 2.7.3 ? ... yes (2.13.4)
Active users: ... 11
 
Checking GitLab ... Finished
然后稍等一会（如果启动gitlab后，访问出现500，这是因为redis等程序还没完全启动，等一会儿访问就ok了），再次登录Gitlab，就会发现之前误删除的test项目已经恢复了！
  
另外：Gitlab迁移与恢复一样，但是要求两个GitLab版本号一致
```  
