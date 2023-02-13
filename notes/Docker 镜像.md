---
tags: [Docker]
title: Docker 镜像
created: '2022-05-30T13:57:53.576Z'
modified: '2022-05-31T06:02:30.090Z'
---

# Docker 镜像

## 镜像命令
### 拉取镜像
```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
Options:
  -a, --all-tags                Download all tagged images in the repository          下载存储库中的所有tags镜像
      --disable-content-trust   Skip image verification (default true)                跳过镜像验证（默认为 true）
      --platform string         Set platform if server is multi-platform capable      如果服务器支持多平台，则设置平台
  -q, --quiet                   Suppress verbose output                               抑制详细输出
```
### 列出镜像
```
docker image ls [OPTIONS] [REPOSITORY[:TAG]]
Aliases:
  ls, list

Options:
  -a, --all             Show all images (default hides intermediate images)           显示全部镜像
      --digests         Show digests                                                  显示摘要
  -f, --filter filter   Filter output based on conditions provided                    根据提供的条件过滤输出
      --format string   Pretty-print images using a Go template                       使用 Go 模板打印漂亮的图像
      --no-trunc        Don't truncate output                                         不要截断输出
  -q, --quiet           Only show image IDs                                           仅显示图像 ID
```
- 在 mongo:3.2 之后建立的镜像[docker image ls -f since=mongo:3.2]
- 在 mongo:3.2 之前建立的镜像[docker image ls -f before=mongo:3.2]
- 镜像构建时，定义了 LABEL，还可以通过 LABEL 来过滤[docker image ls -f label=com.example.version=0.1]
- 显示无标签镜像也被称为 虚悬镜像(dangling image) [docker image ls -f dangling=true]
```
$ docker image ls --format "{{.ID}}: {{.Repository}}"
5f515359c7f8: redis

$ docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY          TAG
5f515359c7f8        redis               latest
```
### 删除本地镜像
```
docker image rm [OPTIONS] IMAGE [IMAGE...]
Aliases:
  rm, rmi, remove

Options:
  -f, --force      Force removal of the image                                          强制删除镜像
      --no-prune   Do not delete untagged parents                                      不要删除未标记的父母
```
- 删除所有仓库名为 redis 的镜像[docker image rm $(docker image ls -q redis)]
- 删除所有在 mongo:3.2 之前的镜像[docker image rm $(docker image ls -q -f before=mongo:3.2)]
### 移除未使用的镜像
```
docker image prune [OPTIONS]

Options:
  -a, --all             Remove all unused images, not just dangling ones                删除所有未使用的镜像，而不仅仅是虚悬镜像
      --filter filter   Provide filter values (e.g. 'until=<timestamp>')                提供过滤器值
  -f, --force           Do not prompt for confirmation                                  不提示确认
```
### 容器打包成镜像
```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")     作者
  -c, --change list      Apply Dockerfile instruction to the created image              
  -m, --message string   Commit message                                                 提交信息
  -p, --pause            Pause container during commit (default true)                   提交期间暂停容器
```
### 容器系统命令
```
docker system COMMAND

Commands:
  df          Show docker disk usage                                                    查看镜像、容器、数据卷所占用的空间
  events      Get real time events from the server                                      从服务器获取实时事件
  info        Display system-wide information                                           显示系统维度的信息
  prune       Remove unused data                                                        删除未使用的数据
```
## 构建镜像
```Dockerfile
FROM debian:latest
WORKDIR /data01/common_api

EXPOSE 5000

ENV PATH /usr/local/bin:$PATH
ENV LANG en_US.utf8

RUN rm -fr /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free" >> /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free"  >> /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free"  >> /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free" >> /etc/apt/sources.list \
    && apt update && apt upgrade -y \
    && apt install -y redis-tools libmemcached-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install -i https://mirrors.aliyun.com/pypi/simple --no-cache-dir -r requirements.txt

CMD ["python", "main.py"]
```
## Dockerfile 指令详解
### COPY 复制文件
```
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]

<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

COPY package.json /usr/src/app/
COPY hom?.txt /mydir/                                                                   通配符，其通配符规则要满足 Go 的 filepath.Match 规则
COPY --chown=55:mygroup files* /mydir/                                                  --chown=<user>:<group> 选项来改变文件的所属用户及所属组
```
### ADD 更高级的复制文件
```
ADD [--chown=<user>:<group>] <源路径>... <目标路径>
ADD [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]

<源路径> 为一个 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，ADD 指令将会自动解压缩这个压缩文件到 <目标路径> 去。

ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
ADD --chown=55:mygroup files* /mydir/                                                  --chown=<user>:<group> 选项来改变文件的所属用户及所属组
```
### CMD 容器启动命令
```
shell 格式：CMD <命令>
exec 格式：CMD ["可执行文件", "参数1", "参数2"...]

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。

如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行。比如：
CMD echo $HOME
在实际执行中，会将其变更为：
CMD [ "sh", "-c", "echo $HOME" ]
这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解析处理。
```
### ENTRYPOINT 入口点
```
有了 CMD 后，为什么还要有 ENTRYPOINT 呢？这种 <ENTRYPOINT> "<CMD>" 有什么好处么？

场景一：让镜像变成像命令一样使用
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
直接使用 docker run myip -i
存在 ENTRYPOINT 后，CMD 的内容将会作为参数传给 ENTRYPOINT，而这里 -i 就是新的 CMD，因此会作为参数传给 curl，从而达到了我们预期的效果

场景二：应用运行前的准备工作
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]

#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
	find . \! -user redis -exec chown redis '{}' +
	exec gosu redis "$0" "$@"
fi

exec "$@"

根据 CMD 的内容来判断，如果是 redis-server 的话，则切换到 redis 用户身份启动服务器，否则依旧使用 root 身份执行
```
### ENV 设置环境变量
```
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...

这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如 RUN，还是运行时的应用，都可以直接使用这里定义的环境变量

下列指令可以支持环境变量展开： ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD、RUN
```
### ARG 构建参数
```
ARG <参数名>[=<默认值>]

构建参数和 ENV 的效果一样，都是设置环境变量。所不同的是，ARG 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有值的。
Dockerfile 中的 ARG 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 docker build 中用 --build-arg <参数名>=<值> 来覆盖。

ARG 指令有生效范围，如果在 FROM 指令之前指定，那么只能用于 FROM 指令中。
ARG DOCKER_USERNAME=library
FROM ${DOCKER_USERNAME}/alpine
RUN set -x ; echo ${DOCKER_USERNAME}

使用上述 Dockerfile 会发现无法输出 ${DOCKER_USERNAME} 变量的值，要想正常输出，你必须在 FROM 之后再次指定 ARG
# 只在 FROM 中生效
ARG DOCKER_USERNAME=library
FROM ${DOCKER_USERNAME}/alpine
# 要想在 FROM 之后使用，必须再次指定
ARG DOCKER_USERNAME=library
RUN set -x ; echo ${DOCKER_USERNAME}
```
### VOLUME 定义匿名卷
```
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中，后面的章节我们会进一步介绍 Docker 卷的概念。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据
VOLUME /data

这里的 /data 目录就会在容器运行时自动挂载为匿名卷，任何向 /data 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行容器时可以覆盖这个挂载设置
docker run -d -v mydata:/data xxxx
使用了 mydata 这个命名卷挂载到了 /data 这个位置，替代了 Dockerfile 中定义的匿名卷的挂载配置
```
### EXPOSE 暴露端口
```
EXPOSE <端口1> [<端口2>...]

EXPOSE 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口
```
### WORKDIR 指定工作目录
```
WORKDIR <工作目录路径>

WORKDIR 指令使用的相对路径，那么所切换的路径与之前的 WORKDIR 有关
WORKDIR /a
WORKDIR b
WORKDIR c

RUN pwd
```
### USER 指定当前用户
```
USER <用户名>[:<用户组>]

USER 指令和 WORKDIR 相似，都是改变环境状态并影响以后的层。WORKDIR 是改变工作目录，USER 则是改变之后层的执行 RUN, CMD 以及 ENTRYPOINT 这类命令的身份。
注意，USER 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。
```
### HEALTHCHECK 健康检查
```
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

启动容器，初始状态会为 starting，在 HEALTHCHECK 指令检查成功后变为 healthy，如果连续一定次数失败，则会变为 unhealthy。
HEALTHCHECK 支持下列选项：
--interval=<间隔>：两次健康检查的间隔，默认为 30 秒；
--timeout=<时长>：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
--retries=<次数>：当连续失败指定次数后，则将容器状态视为 unhealthy，默认 3 次。
和 CMD, ENTRYPOINT 一样，HEALTHCHECK 只可以出现一次，如果写了多个，只有最后一个生效。

为了帮助排障，健康检查命令的输出（包括 stdout 以及 stderr）都会被存储于健康状态里，可以用 docker inspect 来查看。
$ docker inspect --format '{{json .State.Health}}' web | python -m json.tool
{
    "FailingStreak": 0,
    "Log": [
        {
            "End": "2016-11-25T14:35:37.940957051Z",
            "ExitCode": 0,
            "Output": "<!DOCTYPE html>\n<html>\n<head>\n<title>Welcome to nginx!</title>\n<style>\n    body {\n        width: 35em;\n        margin: 0 auto;\n        font-family: Tahoma, Verdana, Arial, sans-serif;\n    }\n</style>\n</head>\n<body>\n<h1>Welcome to nginx!</h1>\n<p>If you see this page, the nginx web server is successfully installed and\nworking. Further configuration is required.</p>\n\n<p>For online documentation and support please refer to\n<a href=\"http://nginx.org/\">nginx.org</a>.<br/>\nCommercial support is available at\n<a href=\"http://nginx.com/\">nginx.com</a>.</p>\n\n<p><em>Thank you for using nginx.</em></p>\n</body>\n</html>\n",
            "Start": "2016-11-25T14:35:37.780192565Z"
        }
    ],
    "Status": "healthy"
}
```
### ONBUILD 为他人作嫁衣裳
```
ONBUILD <其它指令>

ONBUILD 是一个特殊的指令，它后面跟的是其它指令，比如 RUN, COPY 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

Dockerfile 中的其它指令都是为了定制当前镜像而准备的，唯有 ONBUILD 是为了帮助别人定制自己而准备的。
```
### LABEL 为镜像添加元数据
```
LABEL <key>=<value> <key>=<value> <key>=<value> ...

还可以用一些标签来申明镜像的作者、文档地址等：
LABEL org.opencontainers.image.authors="yeasy"
LABEL org.opencontainers.image.documentation="https://yeasy.gitbooks.io"
```
### SHELL 指令
```
SHELL ["executable", "parameters"]

两个 RUN 运行同一命令，第二个 RUN 运行的命令会打印出每条命令并当遇到错误时退出。
当 ENTRYPOINT CMD 以 shell 格式指定时，SHELL 指令所指定的 shell 也会成为这两个指令的 shell
SHELL ["/bin/sh", "-cex"]
# /bin/sh -cex "nginx"
ENTRYPOINT nginx

SHELL ["/bin/sh", "-cex"]
# /bin/sh -cex "nginx"
CMD nginx
```
