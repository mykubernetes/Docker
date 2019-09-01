1、通过命令方式实现alert静默
```
# amtool --alertmanager.url=http://192.168.101.69:9093 silence add alertname=InstancesGone service=application1 -c "test1"
```  
- alertname 为告警名称
- service=application1 是告警名称里的标签，用于匹配，key=value
- -c  注释信息
注意：默认创建的静默时间为 1h ，也可以手工指定，匹配两个标签，然后会返回一个静默 ID  

2、 查询当前静默列表
```
# amtool --alertmanager.url=http://192.168.101.69:9093 silence query
```  

3、取消静默模式  
```
# amtool --alertmanager.url=http://192.168.101.69:9093 silence expire 784ac68d-33ce-4e9b-8b95-431a1e0fc268
```  

4、扩展，可以不指定--alertmanager.url=http://192.168.101.69:9093的方式  
amtool 如果不指定 --alertmanager.url默认会在 $HOME/.config/amtool/config.yml 或 /etc/amtools/config.yml 查询  
```
# 创建目录
mkdir -pv $HOME/.config/amtool/

# 将alertmanager.url写入配置文件
# vim /$HOME/.config/amtool/config.yml
alertmanager.url: "http://192.168.101.69:9093"
author: test@example.com
comment_required: true

测试
# amtool silence add --comment "App1 maintenance" alertname=~'Instance.*' service=application1
# amtool silence add --author "test" --duration "2h" alertname=InstancesGone service=application1
```  

