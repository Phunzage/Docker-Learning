
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

使用 Docker compose 创建一个 yaml 文件，把配置信息写到里面

```
compose.yaml

name: myblog
services:
	mysql:
		container_name: mysql
		image: mysql:8.0.42
		ports:
			- "3306:3306"
		environment:
			- MYSQL_ROOT_PASSWORD=123456    //可以使用键值对的方式描述
			- MYSQL_DATABASE=wordpress
		volumes:
			- mysql-data:/var/lib/mysql
			- /app/myconf:/etc/mysql/conf.d
		restart: always
		networks:
			- blog
	wordpress:
		image: wordpress
		ports:
			- "80:80"
		environment:
			- WORDPRESS_DB_HOST=mysql
			- WORDPRESS_DB_USER=root
			- WORDPRESS_DB_PASSWORD=123456
			- WORDPRESS_DB_NAME=wordpress
		volumes:
			- "wordpress:/var/www/html"
		restart: always    //总是随服务器的启动而启动
		networks:
			- blog
		depends_on:
			- mysql    //依赖 mysql 服务

volumes:
	mysql-data:
	wordpress:

networks:
	blog:

```

> 其中，启动了两个容器，MySQL 和 wordpress 
> wordpress 的depends_on 是指 wordpress 依赖 MySQL来运行
> volumes 和 networks 表示指明了 MySQL 和 wordpress 的卷挂载和网络，后面不写则存储在 docker 的默认存放位置

使用命令 `docker compose -f compose.yaml up -d` 来启动 dcoker 容器
修改了 compose.yaml 文件之后可以再次使用`docker compose up -d`来批量启动容器
使用 `docker compose -f compose.yaml down` 关闭和删除容器，但是卷挂载和网络会保存

要想删除卷挂载和网络等，使用 `docker compose -f compose.yaml -rmi -v` ,要注意 -rmi 后面可以加 all 即把所有（包括远程镜像、卷挂载、自定义网络）

下次再次使用`docker compose -f compose.yaml up -d`命令，即可快速启动容器





