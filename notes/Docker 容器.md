---
tags: [Docker]
title: Docker 容器
created: '2022-05-31T13:56:31.425Z'
modified: '2022-06-01T15:36:20.189Z'
---

# Docker 容器

## 容器命令

### 列出容器

```
docker ps [OPTIONS]

Options:
  -a, --all             Show all containers (default shows just running)                              显示全部容器（默认显示运行的）
  -f, --filter filter   Filter output based on conditions provided                                    根据提供的条件过滤输出
      --format string   Pretty-print containers using a Go template                                   使用 Go 模板打印漂亮的容器
  -n, --last int        Show n last created containers (includes all states) (default -1)             显示 n 个最后创建的容器（默认 -1）
  -l, --latest          Show the latest created container (includes all states)                       显示最新创建的容器
      --no-trunc        Don't truncate output                                                         不要截断输出
  -q, --quiet           Only display container IDs                                                    仅显示容器 ID
  -s, --size            Display total file sizes                                                      显示总文件大小
```

### 停止容器

```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
docker container stop [OPTIONS] CONTAINER [CONTAINER...]

Stop one or more running containers

Options:
  -t, --time int   Seconds to wait for stop before killing it (default 10)                            在杀死它之前等待停止的秒数
```

### 启动容器

```
docker start [OPTIONS] CONTAINER [CONTAINER...]
docker container start [OPTIONS] CONTAINER [CONTAINER...]

Start one or more stopped containers

Options:
  -a, --attach               Attach STDOUT/STDERR and forward signals                                 附加 STDOUT/STDERR 并转发信号
      --detach-keys string   Override the key sequence for detaching a container                      覆盖用于分离容器的键序列
  -i, --interactive          Attach container's STDIN                                                 附加容器的 STDIN
``` 

### 重启容器

```
docker restart [OPTIONS] CONTAINER [CONTAINER...]
docker container restart [OPTIONS] CONTAINER [CONTAINER...]

Restart one or more containers

Options:
  -t, --time int   Seconds to wait for stop before killing the container (default 10)                 在杀死它之前等待停止的秒数
``` 

### 删除容器

```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
docker container rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)                             强制删除正在运行的容器（使用 SIGKILL
  -l, --link      Remove the specified link                                                           删除指定链接
  -v, --volumes   Remove anonymous volumes associated with the container                              删除与容器关联的匿名卷
``` 

### 清理所有处于终止状态的容器

```
docker container prune [OPTIONS]

Remove all stopped containers

Options:
      --filter filter   Provide filter values (e.g. 'until=<timestamp>')                              提供过滤器值
  -f, --force           Do not prompt for confirmation                                                不提示确认
```

### 显示容器的运行进程

```
docker top CONTAINER [ps OPTIONS]
docker container top CONTAINER [ps OPTIONS]

Display the running processes of a container
```

### 在正在运行的容器中运行命令

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background                           分离模式：后台运行命令
      --detach-keys string   Override the key sequence for detaching a container                    覆盖用于分离容器的键序列
  -e, --env list             Set environment variables                                              设置环境变量
      --env-file list        Read in a file of environment variables                                读入环境变量文件
  -i, --interactive          Keep STDIN open even if not attached                                   即使没有连接，也保持 STDIN 打开
      --privileged           Give extended privileges to the command                                赋予命令扩展权限
  -t, --tty                  Allocate a pseudo-TTY                                                  分配一个伪 TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container                                 容器内的工作目录
```

### 导出容器

```
docker export [OPTIONS] CONTAINER
docker container export [OPTIONS] CONTAINER

Export a container's filesystem as a tar archive

Options:
  -o, --output string   Write to a file, instead of STDOUT                                           写入文件，而不是 STDOUT

docker export 7691a814370e > ubuntu.tar
```

### 导入容器

```
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]

Import the contents from a tarball to create a filesystem image

Options:
  -c, --change list       Apply Dockerfile instruction to the created image                            将 Dockerfile 指令应用于创建的镜像
  -m, --message string    Set commit message for imported image                                        为导入的镜像设置提交消息
      --platform string   Set platform if server is multi-platform capable                             如果服务器支持多平台，则设置平台

cat ubuntu.tar | docker import - test/ubuntu:v1.0
docker import http://example.com/exampleimage.tgz example/imagerepo
```

### 返回有关 Docker 对象信息

```
docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format the output using the given Go template
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
```

## run命令

```
# 启动 Redis
docker run \
  -it -d --rm \
  --name=redis1 \
  -p 127.0.0.1:6379:6379 \
  -v /home/john/docker-redis/config/redis1.conf:/etc/redis/redis.conf \
  -v /home/john/docker-redis/data:/data:rw \
  --privileged=true \
  redis:latest \
  redis-server /etc/redis/redis.conf
```

run 常用参数：

* `--name` 容器别名
* `--rm` 容器退出后自动销毁
* `-v` 绑定文件到容器里面
* `-p` 端口映射
* `-d` 容器在后台运行
* `-i` 容器内部是否可以接收 stdin 标准输入
* `-t` 容器内部命令是否在一个 tty 终端里面
* `--link` 用于容器之间互相访问 `--link name:name`


detach `ctrl+p+q`
容器里面必须有一个进程始终在前台挂着，不然容器内部 pid=1 的进程如果没有了，容器就停止了。
对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

* `-d` 一般用在启动一个前台的进程，比如启动 `redis-server --daemon=no` 或者是 `nginx` 都必须启动在前台，可以使用 exit 退出容器，容器不会自动退出。
* `-it` 两个参数一般连用，使用在需要交互的命令里面，比如命令行交互的 `python` 这要需要从客户端获取输入和输出流，必须使用 `-it` 参数，不然启动的命令没法交互。
* 容器内部的初始化命令必须是在前台运行相当于默认占用了 `tty0` 终端，不能退出，如果退出了容器也就销毁了
* 如果使用 `docker attach name` 连接到容器里面，默认就是到 `tty0` 上面，这个时候如果用 `ctrl+c` 就相当于终端进程，容器也就自动退出了
* 如果使用了 `docker exec -it name bash` 连接到容器里面，容器里面其实就有了两个 `tty` 终端了，`tty0` 是容器启动时候 CMD 的进程，`tty1` 就是新的可以执行交互的
* 理解了上面两个参数后，就可以理解 `attach` 和 `exec -it name bash` 的区别了，如果是登陆了 `tty0` 里面，必须用快捷键分离，如果是通过 `exec -it` 登陆后，可以直接使用 `exit` 退出

## run 启动容器 bash

```bash
# 后台运行一个容器
docker run --rm -d {image-name} /bin/bash

# 前台运行一个容器
docker run --rm -it {image-name} /bin/bash

# 后台运行一个空容器
docker run --rm -d {image-name} tail -f /dev/null

# ctrl+p+q 分离容器
# 重新登录容器
docker attach name

# 推荐使用 exec
docker exec -it {container-id} bash
```

