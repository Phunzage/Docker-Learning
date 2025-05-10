
## 关于安全组

相当于防火墙，云服务器开放了一些端口，如80端口和远程连接的22端口，所以本地端口映射只能映射已开放的端口，
要增加端口，就要去云服务器控制台安全组开放相应的端口

____

## 容器内部

目前容器内部修改有两大麻烦：
1. 使用docker exec进入容器麻烦
2. 容器若出问题，内部已修改的信息有丢失风险

### 解决方法：

使用容器挂载

1. 目录挂载

```
docker run -d -p 80:80 -v /app/nghtml:/usr/share/nginx/html --name mynginx nginx:1.26.0
```

其中

> -v 是将外部文件与内部文件相关联，此处将外部服务器上的 /app/nghtml 与 容器内部的 /usr/share/nginx/html 相关联
> 此时容器以外部文件夹内容为准

使用
```
cd /app/nghtml
touch index.html
echo "<h1>hello world</h1>" > index.html
```

即可编写一个简单的新页面

> 如果删除容器，内部 /app/nghtml 仍然会存在，可供下次使用


2. 卷映射

```
docker run -d -p 80:80 -v /app/nghtml:/usr/share/nginx/html -v ngconf:/etc/nginx --name mynginx nginx:1.26.0
```

其中

> -v ngconf:/etc/nginx 是将nginx的配置文件夹挂载给 ngconf 这个卷，前后不要加 / ，此时 ngconf 内部会有（复制过来）配置文件，
> 此时文件内容要以容器内配置文件为准 

默认存储位置在 /var/lib/docker/volumes/文件名（ngconf），可以使用

```
查看挂载的卷
docker volume ls
新创建卷
docker volume create newconf01
查看详情
docker volume inspect ngconf    //可查看卷所在容器内位置以及创建时间等属性
```

> 如果删除容器，卷也会保留以供下次使用






















