Prometheus监控elk实现  
1、elk环境以及prometheus环境已经部署完成  
2、下载第三方监控插件，实现prometheus与elk的对接  
插件下载地址： https://github.com/vvanholl/elasticsearch-prometheus-exporter/releases  

选择对应的elk版本进行下载，将其解压后存放在/usr/share/elasticsearch/plugins目录下，下载的安装包解压后存在plugin-descriptor.properties才能正确安装，安装好以后重启elasticsearch配置prometheus的配置文件  

添加如下配置  
```
- job_name: 'elasticsearch'
    metrics_path: "/_prometheus/metrics"
    static_configs:
    - targets:[ '172.31.125.104:9200']
```
其中job_name随便定义，如果已经对接grafana，可以去grafana官网下载对应的json模板。  
