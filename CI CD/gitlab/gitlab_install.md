gitlab安装
=======
1、下载镜像  
``` docker pull gitlab/gitlab-ce ```  
2、创建挂载目录  
``` mkdir -pv /opt/gitlab/{config,logs,data}  ```  
3、启动gitlab  
```
docker run -d \
    -p 8443:443 \
    -p 8090:80  \
    -p 2222:22 \
    --name gitlab \
    --restart always \
    -v /opt/gitlab/config:/etc/gitlab \
    -v /opt/gitlab/logs:/var/log/gitlab \
    -v /opt/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce
```  
4、修改配置文件,修改下面两条配置就可以显示了  
```
vim /opt/gitlab/config/gitlab.rb
external_url='192.168.101.66'              #注意添加=号否则可能启动失败

vim /opt/gitlab/data/gitlab-rails/etc/gitlab.yml
  gitlab:
    ## Web server settings (note: host is the FQDN, do not include http://)
    host: 192.168.101.66      #修改成映射的ip地址
    port: 8090                #修改成映射的端口
    https: false
```  

5、可选修改配置
```
vim /opt/gitlab/config/gitlab.rb
修改NGINX监听的端口
nginx['listen_port'] = 8090
如果8080端口被Tomcat占用，会出现502的页面
unicorn['listen'] = 'localhost'
unicorn['port'] = 8080

默认的Gitlab数据存储路径,/data/gitlabData,防止以后数据过大可以修改
###!   path that doesn't contain symlinks.**
# git_data_dirs({
#   "default" => {
#     "path" => "/mnt/nfs-01/git-data"
#    }
# })

git_data_dirs({ "default" => { "path" => "/data/gitlabData" } })
```  
5、初始化配置  
```
docker exec -ti gitlab /bin/bash
gitlab-ctl reconfigure      #花时间比较多
gitlab-ctl restart          #改IP重启就可以了
gitlab-ctl status
```  

6、打开浏览器
http://192.168.101.66:8090
