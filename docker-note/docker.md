### docker 基本操作

命令帮助文档

```shell
docker --help
docker images --help
```

##### 镜像操作

![docker-note_docker基本操作_镜像操作](F:\github远程仓库\cloud-note\picture\docker-note_docker基本操作_镜像操作.jpg)

案例

```shell
docker pull nginx # 从远程仓库中拉取最新
docker images # 展示已有的
docker save --help
docker save -o nginx.tar nginx:latest # 将镜像保存到指定文件
docker rmi nginx:latest # 移除已有的镜像
docker load --help # 加载文件，得到镜像
docker load -i nginx.tar
```

###### 容器操作

![docker-note_docker基本操作_容器操作](F:\github远程仓库\cloud-note\picture\docker-note_docker基本操作_容器操作.jpg)

案例

```shell
docker run --name containerName -p 80:80 -d nginx # run:运行；--name:起名字；--p:端口映射；-d:后台运行；nginx:要运行的镜像名称
docker ps # 查看正在运行的容器
docker ps -a # 查看所有容器
docker logs containerName # 查看该容器的日志

```

