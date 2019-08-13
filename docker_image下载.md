国内拉取google kubernetes镜像
===

微软google gcr镜像源
---
```
#以gcr镜像为例，以下镜像无法直接拉取
docker pull gcr.io/google-containers/kube-apiserver:v1.15.2
#改为以下方式即可成功拉取：
docker pull gcr.azk8s.cn/google-containers/kube-apiserver:v1.15.2
```  

微软coreos quay镜像源  
---
```
#以coreos镜像为例，以下镜像无法直接拉取
docker pull quay.io/coreos/kube-state-metrics:v1.7.2
#改为以下方式即可成功拉取：
docker pull quay.azk8s.cn/coreos/kube-state-metrics:v1.7.2
```  

微软dockerhub镜像源  
---
```
#以下方式拉取镜像较慢
docker pull centos
#改为以下方式使用微软镜像源：
docker pull dockerhub.azk8s.cn/library/centos
docker pull dockerhub.azk8s.cn/willdockerhub/centos
```  

dockerhub google镜像源
---
```
#以gcr镜像为例，以下镜像无法直接拉取
docker pull gcr.io/google-containers/kube-apiserver:v1.15.2
#改为以下方式即可成功拉取：
docker pull mirrorgooglecontainers/google-containers/kube-apiserver:v1.15.2
```  

阿里云google镜像源  
---
```
#以gcr镜像为例，以下镜像无法直接拉取
docker pull gcr.io/google-containers/kube-apiserver:v1.15.2
#改为以下方式即可成功拉取：
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.15.2
```  




