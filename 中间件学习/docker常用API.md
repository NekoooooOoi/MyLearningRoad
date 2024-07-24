#### Docker常用API

容器的定义理解：容器本质上是由命名空间隔离开的受到约束的一系列进程，即 使进程在一个受到约束，受到隔离的环境运行。

容器进程的创建过程：

* 第一次需要申请所需要的网络/PID等的命名空间以创造一个相对隔离的环境；第二次这是找到并使用此命名空间即可。命名空间使该容器的进程与其他进程隔离开。
* 给容器专属的文件系统，即使用chroot等方式使容器的进程只能在当前文件系统下活动。chroot的囚牢使该容器的进程只能访问特定的文件目录，无法对容器外的进程的文件进行操作。
* 创建和容器配置符合的cgroups来限制进程所获得的资源。
* 执行相关可执行文件（加载镜像等）。

容器是一个相对虚拟的概念，通过以上方式，一系列的进程在一个隔离的环境(隔离的命名空间)中，受到一定约束（通过cgroups限制cpu,内存等资源的使用），并且有一个看似独立的文件系统（只能看到自己文件目录），就仿佛是运行在一个容器当中一般。

学习资料：

* https://iximiuz.com/en/posts/containers-learning-docker-with-docker/
* [https://iximiuz.com/en/posts/container-learning-path/?z=10](https://iximiuz.com/en/posts/container-learning-path/?z=100)
* https://github.com/p8952/bocker
* https://github.com/shuveb/containers-the-hard-way
* https://zhuanlan.zhihu.com/p/688123767
* https://docker-practice.github.io/zh-cn/introduction/what.html

在终端里查看docker API

```shell
# docker命令可以直接查看所有的命令
docker
# docker 命令 -help 可以查看具体的解释
docker ps -help
```

容器的使用

获取镜像

```shell
# 如果我们本地没有 ubuntu 镜像，我们可以使用 docker pull 命令来载入 ubuntu 镜像
docker pull ubuntu
```

启动交互式的容器

```shell
runoob@runoob:~$ docker run -itd -p 5000:5000 -v /test:/soft ubuntu:15.10 /bin/bash
2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63
# docker run 运行容器的命令
# -i 允许你对容器内的标准输入 (STDIN) 进行交互。
# -t 在新容器内指定一个伪终端或终端。
# -d 后台运行。
# -p hostPort:containerPort 进行端口的映射
# -v 数据卷绝对路径：容器内绝对路径 --进行数据卷挂载
# ubuntu:15.10 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
# /bin/bash 放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
```

查看容器和退出

```shell
# docker ps -a 查看所有容器
docker ps -a
# docker create 容器名称 -创建一个容器但不运行它
docker create <镜像>
# 使用 docker start 启动一个已经停止的容器
docker start <容器 ID>
# 停止容器
docker stop <容器 ID>
# 重启容器
docker restart <容器 ID>
# 进入容器, docker exec进入容器然后再退出容器不会导致容器的关闭
docker exec -it <容器 ID> /bin/bash
# 退出容器， 我们通过运行 exit 命令或者使用 CTRL+D 来退出容器
exit
# 删除容器, 删除时必须是停止状态
docker rm <容器 ID>
# 查看容器内终端输出， -f可以做到 tail -f 的效果
docker logs -f <容器 ID>
```

镜像的使用

```shell
# 列出主机上的镜像
docker images
# docker search 镜像名 查找某个镜像，也可以去仓库找 https://hub.docker.com/
docker search httpd
# 删除镜像
docker rmi <镜像名>
```

镜像的创建

- 从已经创建的容器中更新镜像，并且提交这个镜像
- 使用 Dockerfile 指令来创建一个新的镜像

**从已创建的容器中更新镜像，并且提交：**

```shell
# 更新镜像之前，我们需要使用镜像来创建一个容器。
runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash
root@e218edb10161:/# 
# 在容器内使用命令，完成后使用 exit 退出
root@e218edb10161:/# apt-get update
root@e218edb10161:/# exit
# 此时 ID 为 e218edb10161 的容器，是按我们的需求更改的容器。我们可以通过命令 docker commit 来提交容器副本
docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
sha256:70bf1840fd7c0d2d8ef0a42a817eb29f854c1af8f7c59fc03ac7bdee9545aff8
# -m 提交的描述信息
# -a 指定镜像作者
# e218edb10161 容器 ID
# runoob/ubuntu:v2 指定要创建的目标镜像名
```

**使用 Dockerfile 指令来创建一个新的镜像**
首先构建一个Dockerfile, 下面是一个简单的例子,每一个指令的前缀都必须是大写的

```shell
FROM    centos:6.7
MAINTAINER      Fisher "fisher@sudops.com"

RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
COPY 		hom* /mydir/

EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```

**FROM**：定制的镜像都是基于 FROM 的镜像
**RUN**：用于执行后面跟着的命令行命令。有以下俩种格式：

```shell
# shell 格式
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令

# exec格式
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline

# 注意：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如下面创建了2层镜像
FROM centos
RUN yum -y install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
# 简化为以下格式只会创建一层
FROM centos
RUN yum -y install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
```

**COPY**: 复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

```shell
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
#<源路径>：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。如：
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

**CMD: **类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在docker run 时运行。
- RUN 是在 docker build。

**作用**：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

```shell
# 推荐第二种格式
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
```

**ENTRYPOINT: **类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

```shell
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

**ARG**：构建参数，与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。`构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。`

**ENV: **设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

```shell
ENV <key> <value>
```

**VOLUME: **定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。
作用：

- 避免重要的数据，因容器重启而丢失，这是非常致命的。
- 避免容器不断变大。

```shell
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

**EXPOSE: **仅仅只是声明端口。
作用：

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

```shell
EXPOSE <端口1> [<端口2>...]
```

**WORKDIR: **指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR 会帮你建立目录。
docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

```shell
WORKDIR <工作目录路径>
```

最后，使用dockerfile构建镜像

```shell
docker build -t nginx:v3 .
# -t ：指定要创建的目标镜像名
# . ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径,也即上下文目录
```

还可以使用 docker compose 来使用yaml文件进行配置，通过一个配置文件来管理多个Docker容器：
Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。
Compose 使用的三个步骤：

- 使用 Dockerfile 定义应用程序的环境。
- 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 docker-compose up 命令来启动并运行整个应用程序。

#### Docker容器不只是进程，还是文件

```
观察docker容器不同阶段file system中内容的变化
```

容器在`docker create`时不会创建logs和runtime相关文件；

容器在'docker start'时才会创建logs和runtime相关文件；

容器在`docker exec`时几乎不产生任何文件变化。

docker容器在运行时进程的运行也反映在文件的变化上。
