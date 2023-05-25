## Dcokerfile 常用指令

### FROM

指定 base 镜像

### MAINTAINER

设置镜像的作者。可以是任意的字符

### COPY

将文件从 build context 复制到镜像

COPY 支持两种格式：COPY src dest 和 COPY [“src”,”dest”]

注意：src 只能制动 build context 中的文件或目录即在和 Dockerfile 同目录下才可以

### ADD

与 COPY 类似，从 build context 复制文件到镜像。

不同的是，如果 src 是归档文件（tar,zip,tgz,xz），文件会被自动接要到 dest

### ENV

设置环境变量，环境变量可被后面的指令使用，例如：

ENV name ken RUN echo $name

### EXPOSE

指定容器中的进程会监听某个端口，Docker 可以将该端口暴露出来

### VOLUME

将文件或目录声明为 volume

### WORKDIR

为后面的 RUN,ENTRYPINT,ADD,COPY 指令设置镜像中的当前工作目录

### RUN

在容器中运行指定的命令

### CMD

容器启动时运行指定的命令

dockerfile 中可以多个 CMD 指令，但是只要最后一个生效。CMD 可以被 docker run 之后的参数替换

### ENTRYPOINT

设置容器启动时的命令

dockerfile 中可以有多个 ENTRYPOINT，但是只有最后一个生效。

CMD 或者 docker run 之后的参数会被当做参数传递给 ENTERYPOINT.

## docker 命令

### **docker run ：**创建一个新的容器并运行一个命令

```sh
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

OPTIONS 说明：

- **-a stdin:** 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
- **-d:** 后台运行容器，并返回容器 ID；
- **-i:** 以交互模式运行容器，通常与 -t 同时使用；
- **-P:** 随机端口映射，容器内部端口**随机**映射到主机的端口
- **-p:** 指定端口映射，格式为：主机(宿主)端口:容器端口
- **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- **--name="nginx-lb":** 为容器指定一个名称；
- **--dns 8.8.8.8:** 指定容器使用的 DNS 服务器，默认和宿主一致；
- **--dns-search example.com:** 指定容器 DNS 搜索域名，默认和宿主一致；
- **-h "mars":** 指定容器的 hostname；
- **-e username="ritchie":** 设置环境变量；
- **--env-file=[]:** 从指定文件读入环境变量；
- **--cpuset="0-2" or --cpuset="0,1,2":** 绑定容器到指定 CPU 运行；
- **-m :**设置容器使用内存最大值；
- **--net="bridge":** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
- **--link=[]:** 添加链接到另一个容器；
- **--expose=[]:** 开放一个端口或一组端口；
- **--volume , -v:** 绑定一个卷

### 实例

使用 docker 镜像 nginx:latest 以后台模式启动一个容器,并将容器命名为 mynginx。

```sh
docker run --name mynginx -d nginx:latest
```

使用镜像 nginx:latest 以后台模式启动一个容器,并将容器的 80 端口映射到主机随机端口。

```sh
docker run -P -d nginx:latest
```

使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。

```sh
docker run -p 80:80 -v /data:/data -d nginx:latest
```

绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。

```sh
$ docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
```

使用镜像 nginx:latest 以交互模式启动一个容器,在容器内执行/bin/bash 命令。

```sh
runoob@runoob:~$ docker run -it nginx:latest /bin/bash
root@b8573233d675:/#
```

```sh
 docker ps    // 查看所有正在运行容器
 docker stop containerId    // containerId 是容器的ID

 docker ps -a      // 查看所有容器
 docker ps -a -q  // 查看所有容器ID

 docker start $(docker ps -a -q)  // start启动所有停止的容器
 docker stop $(docker ps -a -q)  // stop停止所有容器

 docker run   // 新建容器并启动
 docker rm $(docker ps -a -q)    // remove删除所有容器

 docker attach id   // 进入容器 （exit 退出） ,但是退出的时候会导致容器停止

 docker exec -it id /bin/bash   // 进入容器 （exit 退出），退出的时候不会导致容器停止（推荐用这个命令）如果进入reids 的话就要在输入 redis-cli 命令

 docker build -t webide:1.42 -f Dockerfile .
```
