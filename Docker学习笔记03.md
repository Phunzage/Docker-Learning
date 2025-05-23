
## Docker内网

在安装 Docker 时，Docker 会给系统添加一条 Docker0 的网卡

使用

```
Docker inspect 容器名    //可查看容器的详细信息，可以找到ip地址 
```

Docker 为每个容器分配唯一 ip ，通过容器 ip 加容器端口可以互相访问

如创建两个容器

```
容器01
docker run -d -p 80:80 --name app01 nginx:1.26.0
容器02
docker run -d -p 88:80 --name app02 nginx:1.26.0
```

查看容器的详细信息（查看 ip ）`docker inspect app02`

> 如app02的 ip 是172.17.0.3

进入 app01 `docker exec -it app01 bash`
通过 `curl ip:port    //此处为172.17.0.3:80` 即可访问 app02 的 80 端口，可看到内容


## 分配自定义网络

可以在容器运行时把容器加入到自定义网络

```
先创建一个自定义网络
docker network create mynet
将新创建的容器加入网络
docker run -d -p 80:80 --name app01 -network mynet nginx1.26.0
docker run -d -p 88:80 --name app02 -network mynet nginx1.26.0

进入 app01 
docker exec -it app01 bash
直接访问 app02
curl app02:port
```

___

## 练习

### Redis主从同步集群

通过分主从机来实现读写分离的效果

```
主机配置
docker run -d -p 6379:6379 \
-v /app/rd1:/bitnami/redis/data \
-e REDIS_REPLICATION_MODE=master \
-e REDIS_PASSWORD=123456 \
--network mynet --name redis01 \
bitnami/redis
```

> 其中 -e 是环境变量，此处指明 redis 是主机，密码为123456
> 加入自建网络， 起别名为 redis01
> 使用 bitnami 的 redis 镜像

```
从机配置
docker run -d -p 6380:6379 \
-v /app/rd2:/bitnami/redis/data \
-e REDIS_REPLICATION_MODE=slave \
-e REDIS_MASTER_HOST=redis01 \
-e REDIS_MASTER_PORT_NUMBER=6379 \
-e REDIS_MASTER_PASSWORD=123456 \
-e REDIS_PASSWORD=123456 \
--network mynet --name redis02 \
bitnami/redis
```

> 此处指明 redis 是从机，它的主机是 redis01，主机端口是 6379 ，
> 主机密码是123456 ，redis02 自身密码是 123456 
> 加入自建网络，起别名为 redis02
> 使用 bitnami 的 redis 镜像

在本地终端，可以使用 `redis-cli -h ip -p port` 连接远程redis（需要先配置 redis.conf ，允许远程连接），通过修改主机文件，从机可同步

___

### MySQL

 > 一个图例

![[Pasted image 20250510213034.png]]

关于容器的一些参数信息（如数据、配置目录），可以通过 `docker inspect 容器名` 或者去 docker hub（镜像站）来获取

启动一个 mysql 容器

```
docker run -d -p 3306:3306 \
-v /app/myconf:/etc/mysql/conf.d \
-v /app/mydata:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name sql01 \
mysql:8.0.42
```

> 挂载 mysql 配置文件和数据文件

在本地终端可以使用 `mysql -hip -Pport -u用户id -p密码`即可远程连接 MySQL 数据库