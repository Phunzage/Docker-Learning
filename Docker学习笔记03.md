
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


