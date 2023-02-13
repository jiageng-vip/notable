---
tags: [Docker]
title: Docker 数据管理
created: '2022-06-01T15:02:21.239Z'
modified: '2022-06-01T15:33:20.239Z'
---

# Docker 数据管理

## 数据卷

数据卷 是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除 数据卷，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的 数据卷。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 docker rm -v 这个命令

* 数据卷 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：
* 数据卷 可以在容器之间共享和重用
* 对 数据卷 的修改会立马生效
* 对 数据卷 的更新，不会影响镜像
* 数据卷 默认会一直存在，即使容器被删除
> 注意：数据卷 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会复制到数据卷中（仅数据卷为空时会复制）。

### 创建数据卷

```
docker volume create [OPTIONS] [VOLUME]

Create a volume

Options:
  -d, --driver string   Specify volume driver name (default "local")                        指定卷驱动程序名称
      --label list      Set metadata for a volume                                           为卷设置元数据
  -o, --opt map         Set driver specific options (default map[])                         设置驱动程序特定选项
```

### 数据卷列表

```
docker volume ls [OPTIONS]

List volumes

Aliases:
  ls, list

Options:
  -f, --filter filter   Provide filter values (e.g. 'dangling=true')                        提供过滤器值
      --format string   Pretty-print volumes using a Go template                            使用 Go 模板打印精美的卷
  -q, --quiet           Only display volume names                                           只显示卷名
```

### 数据卷信息

```
docker volume inspect [OPTIONS] VOLUME [VOLUME...]

Display detailed information on one or more volumes

Options:
  -f, --format string   Format the output using the given Go template
```

### 删除数据卷

```
docker volume rm [OPTIONS] VOLUME [VOLUME...]

Remove one or more volumes. You cannot remove a volume that is in use by a container.

Aliases:
  rm, remove

Examples:

$ docker volume rm hello
hello


Options:
  -f, --force   Force the removal of one or more volumes
```

### 删除未被使用的数据卷

```
docker volume prune [OPTIONS]

Remove all unused local volumes

Options:
      --filter filter   Provide filter values (e.g. 'label=<label>')
  -f, --force           Do not prompt for confirmation
```

### 启动一个挂载数据卷的容器

```
在用 docker run 命令的时候，使用 --mount 标记来将 数据卷 挂载到容器里。在一次 docker run 中可以挂载多个数据卷。

加载一个 数据卷 到容器的 /usr/share/nginx/html 目录
docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```

## 挂载主机目录

### 挂载一个主机目录作为数据卷

使用 --mount 标记可以指定挂载一个本地主机的目录到容器中去

```
docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine
```

加载主机的 /src/webapp 目录到容器的 /usr/share/nginx/html目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，以前使用 -v 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用 --mount 参数时如果本地目录不存在，Docker 会报错。

Docker 挂载主机目录的默认权限是 读写，用户也可以通过增加 readonly 指定为 只读

```
docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html:ro \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly \
    nginx:alpine
```

### 挂载一个本地主机文件作为数据卷

--mount 标记也可以从主机挂载单个文件到容器中

```
docker run --rm -it \
   # -v $HOME/.bash_history:/root/.bash_history \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:18.04 \
   bash

root@2affd44b4667:/# history
1  ls
2  diskutil list
```

