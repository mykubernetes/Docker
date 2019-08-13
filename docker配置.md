1、配置Docker
```
$ mkdir /etc/docker
$ cat << EOF > /etc/docker/daemon.json
{
"insecure-registry": [
    "hub.hipstershop.cn",
    "reg.hipstershop.cn"
],
"registry-mirror": "https://q00c7e05.mirror.aliyuncs.com",
"graph": "/data1/docker",
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
  "max-size": "100m"
},
"storage-driver": "overlay2",
"storage-opts": [
  "overlay2.override_kernel_check=true"
]
}
EOF
```
2、启动Docker  
```
$ systemctl enable docker && systemctl start docker
```  
