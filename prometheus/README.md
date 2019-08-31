curl方式动态加载命令  
```
curl -X POST http://localdns:9090/-/reload
```  
- --web.enable-lifecycle #热载入配置,2.0之后，默认是关闭的，需要开启

kill方式动态加载命令  
```
kill -HUP pid
```  
