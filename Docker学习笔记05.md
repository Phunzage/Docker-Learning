
## Dockerfile 参考

[Dockerfile 编写指南](https://docs.docker.net.cn/reference/dockerfile/)

![[Pasted image 20250513142810.png]]

Dockerfile 相当于 windows 或 linux 的 iso 压缩包，里面封装好了系统环境，需要的软件以及一些启动命令，我们可以将自己的软件和一些启动命令封装进 Dockerfile

一些重要参数

```
FROM 写一些基础环境，比如openjdk

LABEL 可以写一些信息，如作者author

COPY 把当前目录下的一些文件，复制到新镜像\操作系统（Dockerfile就认为是一个新的操作系统）的哪个目录下
EXPOSE 开放的端口（容器内部的端口）

ENTRYPOINT 容器固定启动命令，一般格式为["", "", ""]
```

使用 `docker build -f Dockerfile -t 起名 .` 来制作镜像

> 其中，-f 指定哪个 Dockerfile 文件，-t 起新构建的镜像的名字，最后面还有一个 . ，这个 . 就是 ./ 代表在当前目录构建

## 使用docker制作自己的镜像

> 案例：制作一个前后端分离页面并部署到 docker

思路：

- 服务隔离
将应用拆分为 MySQL 、 Go 后端、 Nginx 三个独立容器，实现服务隔离
MySQL：专注数据存储
Go：处理业务逻辑和API
Nginx：静态资源托管+API反向代理
依赖启动顺序：MySQL -> Go 后端 -> Nginx

- 网络通信设计
自定义Docker网络（rose-network）：
器间通过服务名（如 mysql 、 backend ）直接通信

- 端口规划：
仅暴露 Nginx 的80端口对外
MySQL 3306端口不映射到宿主机，仅限容器间访问

>> 文件结构

```
rose-shop
	backend
		main.go
		go.mod
		go.sum
		Dockerfile
	frontend
		index.html
		style.css
		script.js
	nginx
		nginx.conf
	docker-compose.yml
	init.sql
```

>> main.go 修改

```
// 修改数据库连接部分
func initDB() (err error) {
    // 从环境变量获取配置
    dbUser := os.Getenv("DB_USER")
    dbPass := os.Getenv("DB_PASSWORD")
    dbHost := os.Getenv("DB_HOST")
    dbPort := os.Getenv("DB_PORT")
    dbName := os.Getenv("DB_NAME")
    
    dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?parseTime=true",
        dbUser, dbPass, dbHost, dbPort, dbName)
    // 其他代码保持不变...
}
```

>> nginx.conf

```
server {
    listen 80;
    server_name localhost;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

>> Dockerfile

```
FROM golang:1.23.4-alpine3.20 AS builder

# 设置国内镜像源和Go代理
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    apk add --no-cache git

WORKDIR /app
COPY . .

# 配置Go环境变量
ENV GOPROXY=https://goproxy.cn,direct \
    GO111MODULE=on

RUN CGO_ENABLED=0 GOOS=linux go build -o main .

FROM alpine:3.20
WORKDIR /app
COPY --from=builder /app/main .
CMD ["./main"]
```

>> docker-compose.yml

```
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: Phz81114002
      MYSQL_DATABASE: rose_shop
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - rose-network
    restart: always
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p$${MYSQL_ROOT_PASSWORD}"]
      interval: 5s
      timeout: 10s
      retries: 10
      start_period: 30s

  backend:
    build: ./backend
    container_name: backend
    environment:
      DB_USER: root
      DB_PASSWORD: Phz81114002
      DB_HOST: mysql
      DB_PORT: 3306
      DB_NAME: rose_shop
    ports:
      - "8080:8080"
    networks:
      - rose-network
    restart: always
    depends_on:
      mysql:
        condition: service_healthy

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./frontend:/usr/share/nginx/html
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
    networks:
      - rose-network
    restart: always
    depends_on:
      - backend

volumes:
  mysql_data:

networks:
  rose-network:
    driver: bridge
```

>> init.sql

```
CREATE DATABASE IF NOT EXISTS rose_shop;

USE rose_shop;

CREATE TABLE `rose_prices` (
  `id` int NOT NULL AUTO_INCREMENT,
  `date` date NOT NULL,
  `price` decimal(10,2) NOT NULL,
  PRIMARY KEY (`id`)
);


INSERT INTO `rose_prices` VALUES 
(1,'2025-05-01',50.00),
(2,'2025-05-02',45.00),
(3,'2025-05-03',48.00),
(4,'2025-05-04',52.00),
(5,'2025-05-05',49.00),
(6,'2025-05-06',50.00),
(7,'2025-05-07',49.00);
```

将 rose-shop 文件夹传入远程服务器，在远程服务器的 rose-shop 文件夹下，执行

```
docker-compose up -d --build
```

即可完成容器的部署

---
## Docker 镜像分层存储机制

> 不同镜像共享基础层（如下图 debian 层，openjdk:17层），避免重复存储，

![[Pasted image 20250513154514.png]]

> 容器启动时，在镜像的只读层之上添加一个可写容器层。所有修改（如写入文件）均在此层进行，底层镜像保持不可变，确保一致性。

![[Pasted image 20250513154544.png]]