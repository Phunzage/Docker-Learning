
## 1. 创建云服务器（使用华为云）

使用按需付费，公网使用按流量计费，可拉满带宽
新建安全组开放443、22、80
使用华为云自带远程连接 CloudShell 
或者在本地终端使用 ssh root@公网ip，输入密码后连接

---

## 2. 访问Docker官网配置

Manuals -> Docker Engine -> install 选择对应 linux 版本

阿里云镜像源

```
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装docker及其插件

```
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

国内docker image镜像站

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

## 3. 通过实例学习docker命令

启动一个nginx，并将它的首页改为自己的页面，发布出去，让所有人都能够使用

### 3.1 下载镜像

- docker 搜索镜像

```
docker search nginx
```

- docker 下载镜像(默认最新)

```
docker pull nginx
```

- 下载指定版本

```
docker pull nginx:1.26.0
```

- 列出镜像列表

```
docker images == docker image ls
```

- 删除镜像( 名称+版本号 或 id )

```
docker rmi nginx:latest
```

### 3.2 启动容器

- 运行

```
docker run	//不建议，前台启用，会阻塞进程
docker run -d --name 别名 容器名		//-d：后台启动，--name：起别名
//docker run -d --name mynginx nginx

//端口映射
docker run -d --name mynginx -p 88:80 nginx 
//将主机的88端口映射到容器内的80端口，此时可以通过访问ip加88端口访问nginx页面
//其中88不可以重复，80可以重复，因为80是容器里的端口，容器可以有多个，可以把每个容器想象成一个小的linux系统
```

- 查看

```
docker ps	//查看进行的容器
docker ps -a	//查看所有容器，包括已经停止的
docker ps -aq    //查看当前所有容器的id
```

- 停止

```
docker stop		//可以使用id前三位或容器名
```

- 启动

```
docker start	//可以使用id前三位或容器名
```

- 重启

```
docker restart	//可以使用id前三位或容器名
```

- 状态

```
docker stats	//可以使用id前三位或容器名
```

- 日志

```
docker logs
```

- 进入

```
docker exec

// 以交互模式进入容器
docker exec -it 容器名字（别名） /bin/bash（交互方式，此处使用控制台）
```

- 删除（删除容器与删除镜像不同）

```
docker rm	//可以使用id前三位或容器名
```

### 3.3 修改页面

每个容器都有具体的所在目录地址，可使用

```
docker exec -it 容器名字（别名） /bin/bash（交互方式，此处使用控制台）

//docker exec -it mynginx /bin/bash
```

命令进入文件目录，使用echo简单修改文件内容
如对于nginx的/etc/share/nginx/html目录

- 使用

```
echo "<h1>Hello Nginx</h1>" > index.html
```

可简单修改页面内容

### 3.4 保存镜像

- 提交

```
docker commit

//提交容器并打包
docker commit -m "更改信息" 当前镜像名 打包后镜像名
//docker commit -m "更改了index.html页面" mynginx mynginx:v1.0
```

- 保存

```
docker save

//保存镜像为指定文件，加入-o后缀将文件打包成.tar文件
docker save -o mynginx.tar mynginx:v1.0
```

此时可以将.tar文件发送给其他人，其他用户加载这个文件并运行之后，进入对应浏览器页面可显示“Hello Nginx”

- 加载

```
docker load

//根据压缩包加载镜像
docker load -i mynginx.tar

//加载完会显示 loaded image: mynginx:v1.0
```

对方：

```
docker run -d --name hisnginx -p 80:80 mynginx:v1.0
```

即可查看页面

### 3.5 分享社区

将文件推送到社区，别人使用docker pull即可下载自己的镜像

由于某些原因，不记录此步骤


