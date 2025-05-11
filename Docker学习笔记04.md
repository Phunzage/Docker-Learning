
## Docker compose 批量管理容器

### 引出问题

> 一个场景：在服务器部署 MySQL + wordpress

```
创建网络
docker network create blog

启动MySQL
docker run -d -p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-e MYSQL_DATABASE=wordpress \
-v mysql-data:/var/lib/mysql \
-v /app/myconf:/etc/mysql/conf.d \
--restart always --name mysql \    //当服务器启动时，此服务也跟着启动
--network blog \
mysql:8.0.42

启动wordpress
docker run -d -p 8080:80 \
-e WORDPRESS_DB_HOST=mysql \
-e WORDPRESS_DB_USER=root \
-e WORDPRESS_DB_PASSWORD=123456 \
-v wordpress:/var/www/html \
--restart always --name wordpress-app \
--network blog \
wordpress:latest
```

> 一个问题

当我在服务器部署了多个容器应用，如果服务器遇到问题，如升级、换服务器，我需要再次执行这些命令，很麻烦且有出错风险

### 解决问题

使用 Docker compose 


