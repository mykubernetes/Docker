```
version: '3'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-bridge

  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-bridge

volumes:
  mysql-data:

networks:
  my-bridge:
    driver: bridge
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
