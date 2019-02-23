docker-compose
============
1、docker-compose安装  
```
wget https://github.com/docker/compose/releases/download/1.24.0-rc1/docker-compose-Linux-x86_64
chmod +x docker-compose-Linux-x86_64
mv docker-compose-Linux-x86_64 /usr/bin/docker-compose
```  

2、编写docker-compose  
```
version: '3'
services:
  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST:  mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-bridge
    depends_on:
      - mysql

  mysql:
    image: mysql
    enviroment:
      MYSQL_ROOT_PASSWORD:  root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      -my-bridge

  volumes:
    mysql-data:
  networks:
    my-bridge:
      driver: bridge
```  
3、启动删除和运行停止docker-compose  
``` 
docker-compose up -f docker-compose.yml
docker-compose up             #启动
docker-compose down           #删除
docker-compose start          #运行
docker-compose stop           #停止
```  
