---
tags: [Docker]
title: Docker 安装和使用
created: '2022-05-28T08:33:29.045Z'
modified: '2022-06-01T02:02:28.603Z'
---

# Docker 安装和使用

最初实现是基于 [LXC](https://linuxcontainers.org/lxc/introduction/)，
从 0.7 版本以后开始去除 LXC，转而使用自行开发的 [libcontainer](https://github.com/docker/libcontainer)，
从 1.11 版本开始，则进一步演进为使用 [runC](https://github.com/opencontainers/runc) 
和 [containerd](https://github.com/containerd/containerd)。
[Docker 入门](https://yeasy.gitbook.io/docker_practice/)

macOS [安装](https://docs.docker.com/desktop/mac/install/)


## 推送镜像到私有仓库

仓库里面，每一个仓库对应一个代码应用

```bash
# 1 登陆到仓库地址
docker login --username=lhxx_2020@rongheyun.net registry.cn-beijing.aliyuncs.com

# 2 创建 tag [LocalImage:LocalTag] [RemoteImage:RemoteTag]
docker tag [py3:v0.0.1] [registry.cn-beijing.aliyuncs.com/mathpool_devlopment/flask:v0.0.1]

# 3 推送
docker push registry.cn-beijing.aliyuncs.com/mathpool_devlopment/flask:v0.0.1
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

## 容器生成镜像

```
docker commit {container-id} {image-name:tag}
```

## 拷贝文件

```
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
docker cp [OPTIONS] DEST_PATH CONTAINER:SRC_PATH
```

## 自动重启

可以使用 `--restart` 参数来控制

* no (default)
* on-failure
* always
* unless-stopped

```
docker run -d --restart unless-stopped redis
docker update --restart unless-stopped redis
docker update --restart unless-stopped $(docker ps -q)
```




