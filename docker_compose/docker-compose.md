```
version: '3'

services:

  web:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-network
    depends_on:
      - mysql
    deploy:
      mode: replicated
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      update_config:
        parallelism: 1
        delay: 10s

  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-network
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager

volumes:
  mysql-data:

networks:
  my-network:
    driver: overlay
```

命令  
```
启动，日志输出到前台
docker-compose up
启动，日志不输出到前台
docker-compose up -d
启动，指定配置文件方式
docker-compose -f docker-compose up
停止
docker-compose stop
启动
docker-compose start
查看运行的容器
docker-compose ps
查看docker-compose定义的容器
docker-compose images
进入容器
docker-compose exec mysql bash
```  

